## <span style="color:#802548">_고려할 경우의 수-시작 시간이 종료 시간 이후가 되지 않게 방지_</span>

- 회의실 예약에 기본적으로 필요한 조건을 내가 생각해서 구현해야했다.
- 처음 고민한 것은 시작 시간이 종료 시간 이후가 되지 않게 하는 것이다.
- 처음 활용했던 것은 Time package의 ZonedDateTime class에 속한 method isBefore(), isAfter()였다. 
- 그러나 이로는 부족했다. isBefore()와 isAfter()라는 method로는 회의가 끝나는 시간에 맞춰 등록하는 경우 중복으로 간주됐다. 
	- 15:00에 끝나면 15:00시작으로 등록하는 것이 대부분의 경우였기 때문에 해당 method는 사용할 수가 없었다. 

```java
//inputStartDateTime: 유저가 선택한 회의 시작 시간
//inputEndDateTime: 유저가 선택한 회의 종료 시간
boolean previousThaninputEndDateTime = inputStartDateTime.atZone(ZoneId.of("Asia/Seoul"))
															.isBefore(inputEndDateTime.atZone(ZoneId.of("Asia/Seoul")));
```


- compareTo로 변경하여 진행하여 15:00 끝나면 15:00에 시작하는 것에 오류를 일으키지 않게 만들었다.

```java
//inputStartDateTime: 유저가 선택한 회의 시작 시간
//inputEndDateTime: 유저가 선택한 회의 종료 시간
public void insertEvent(CalendarEventReq calendarEventReq) throws Exception {
	boolean previousThaninputEndDateTime = inputStartDateTime.atZone(ZoneId.of("Asia/Seoul"))
																.compareTo(inputEndDateTime.atZone(ZoneId.of("Asia/Seoul"))) < 0;
	if (!previousThaninputEndDateTime) {
		throw new BizException(ErrorEnum.CALENDAR_END_BEFORE_START);
	}

}
```

## <span style="color:#802548">_고려할 경우의 수-일정이 중복되지 않게 하기_</span>

- 그 다음 과제는 일정이 중복되지 않게 처리하는 것이었다.
- 생각해봐도 명확한 알고리즘이 나오지 않았다. 하지만 우선 모든 일정이 필요한 게 아니라, 시작 시간 근처로 2개의 일정만 알면 된다는 결론에 이르렀다.
- 따라서 유저가 요청한 시작 시간 전후로 일정을 1개씩 조회한다.

```java
//inputStartDateTime: 유저가 선택한 회의 시작 시간
Events events = service.events().list("ddddd@gmail.com")
								.setTimeMin(inputStartDateTime)
								.setMaxResults(2)
								.setSingleEvents(true)
								.setOrderBy("starttime")
								.execute();
List<Event> items = events.getItems();
```

- 만약 전 후 일정이 없다면 중복을 고려할 필요가 없으니 일정을 만들고 거기서 early return을 진행한다.

```java
// 선택한 시간대가 이후 일정이 없다면 중복 검사를 수행하지 않고 바로 등록한다.
if (items == null) {
	service.events().insert("ddddd@gmail.com", event).execute();
	return;
}
```


- item은 전후 일정으로 갯수가 2개기 때문에 for문에서 조회한다.
- 전의 일정과 중복되도, 후의 일정과 중복되도 안되기 떄문에 둘 중에 하나라도 걸리면 exception을 던진다.

```java
//inputEndDateTime: 유저가 선택한 회의 종료 시간
//listStartDateTime: 기존 회의 시작 시간
ZonedDateTime listStartDateTime = null;
boolean previousThanListStartDateTime = false;
for (Event item : items) {
	listStartDateTime = ZonedDateTime.from(
											Instant.from(DateTimeFormatter.ISO_DATE_TIME.parse(item.getStart().getDateTime().toStringRfc3339()))
													.atZone(ZoneId.of("Asia/Seoul"))
											);

	// 일정 중복 여부를 확인한다. 종료 시간과 겹쳐서 바로 시작하는 경우 중복으로 간주하지 않는다.
	previousThanListStartDateTime = inputEndDateTime.atZone(ZoneId.of("Asia/Seoul"))
													.compareTo(listStartDateTime) <= 0;
	if (!previousThanListStartDateTime) {
		throw new BusinessException(ErrorEnum.CALENDAR_ALREAY);
	}
}
```


## <span style="color:#802548">_고려할 경우의 수-수정 시에는 자기 자신을 수정하는 경우는 허용하기_</span>

- 입력의 경우와 똑같은 알고리즘을 사용하려고 했다.
- 하지만 수정의 경우 같은 시간대로 놓고 수정하면 중복으로 판단하는 결과를 일으켰다.
- 실제로 잘못 눌러서 똑같은 시간대로 수정을 했는데, 수정을 했더니 중복이라고 뜨면 당황스러울 것이다. 
- 이 경우 중복이어도 문제가 되지 않기 때문에 그대로 수정되게끔 알고리즘을 수정했다.

```java
List<Event> filteredItems=items.stream().filter(element -> {
	return !element.getId().equals(calendarEventReq.getEventId());
}).collect(Collectors.toList());

for (Event item : filteredItems) { //not All item, but filteredItem
	//duplication check logic
}
```

## <span style="color:#802548">_고려할 경우의 수- 주말에는 insert하지 않게_</span>

- 일단, front에서 주말의 경우 별도의 색깔로 칠해놓아 입력할 때 경고를 주었으나, 그거로 충분하지 않다는 생각이 들었다.
- 따라서 주말에 일정을 등록하지 못하게 주말 여부인지 확인하기로 했다.
- 들어온 값이 주말이면 굳이 Google cloud db를 활용할 필요도 없으므로 맨 처음에 검사하도록 추가했다.

```java
private boolean isWeekend(String isoDateTimeString) {
	LocalDateTime dateTime = LocalDateTime.parse(isoDateTimeString);
	DayOfWeek dayOfWeek = dateTime.getDayOfWeek();
	return ((dayOfWeek == DayOfWeek.SATURDAY) || (dayOfWeek == DayOfWeek.SUNDAY));
}

public void insertEvent(CalendarEventReq calendarEventReq) throws Exception {
	if(isWeekend(calendarEventReq.getStartDateTime())) {
		throw new BusinessException(ErrorEnum.CALENDAR_WEEKEND);
	}
}
```


## <span style="color:#802548">_고려할 경우의 수- 공휴일에는 insert하지 않게_</span>

- 주말을 포함했으니, 휴일도 제외시키는 것으로 결정하였다.
- 다만 주말이 아닌 휴일을 가져오는 방법은 JAVA API에는 없기 때문에 별도로 찾아보아야 했다.
- 방법은 국가에서 제공하는 API 혹은 현재 사용하는 구글 Calendar API를 사용하는 것이었다.
- 유사성이 높은 구글 Calendar API를 사용하기로 결정했다.
- 아래와 같이 두 개를 받아오게 한 결과, 반응성이 악화되었다. 처음 우리가 만든 일정 select가 완료된 뒤에 휴일 일정을 select하는 형태였기 때문이다.

```java
public void insertEvent(CalendarEventReq calendarEventReq) throws Exception {
	Events events = service.events().list("ddddd@gmail.com")
									.setTimeMin(startDateTime).setMaxResults(2)
									.setSingleEvents(true)
									.setOrderBy("starttime")
									.execute();
	List<Event> items = events.getItems();

	Events holidays = service.events()
							.list("ko.south_korea#holiday@group.v.calendar.google.com")
							.setTimeMax(lastDay)
							.setTimeMin(firstDay)
							.execute();
	List<Event> holidays = events.getItems();

	items.add(holidays);
}
```

- 해당 부분을 고치려면 동시에 가져오는 행위가 필요했다.
- 그 중에 CompletableFuture를 활용했다. 
- parallelStream, ExceutorService + Future, Spring Reactive 등의 선택지가 있었다.
	- parallelStream나 Future class는 오류처리가 어려웠다.
	- Spring Reactive는 우리 프로젝트에 없는 기술인데다가, 배우는 데 오래 걸린다는 평가가 많았다.
	- 따라서 오류처리가 간편하며, fucntional chaining도 가능한 CompletableFuture를 사용하기로 했다.
- 처음에는 Oauth service를 사용하는 Calendar 객체를 각 요청이 공유하여 사용하는 형태로 진행했다.

```java
public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = getServiceAuth();

	CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(() -> { 
									return service.events().list("fffff@gmail.com")
															.setTimeMax(lastDay)
															.setTimeMin(firstDay)
															.execute(); 
								});
	CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(() -> { 
									return service.events().list("ko.south_korea#holiday@group.v.calendar.google.com")
															.setTimeMax(lastDay)
															.setTimeMin(firstDay)
															.execute(); 
								});


	List<Event> items =  companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> {
																						cur.putAll(prev);
																						return cur;
																					}
													).join().getItems();							
}
```


- 하지만 구글에서 만든 Calendar class가 thread-safe인지 확신이 없었다.
- thread-safe한 class가 아니면 multi-threading 환경에서 문제를 일으킬 수 있었다.
- thread-safe하지 않은 공유 객체이기 때문에 race-condition에 빠질수 있다고 판단됐다.
- 따라서 공유 객체가 아닌 각각의 thread에서 별개로 object를 만드는 형태로 변경했다.

```java
public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = getServiceAuth();

	CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(() -> { 
									Calendar service = getServiceAuth();
									return service.events().list("fffff@gmail.com")
															.setTimeMax(lastDay)
															.setTimeMin(firstDay)
															.execute(); 
								});
	CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(() -> { 
									Calendar service = getServiceAuth();
									return service.events().list("ko.south_korea#holiday@group.v.calendar.google.com")
															.setTimeMax(lastDay)
															.setTimeMin(firstDay)
															.execute(); 
								});


	List<Event> items =  companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> {
																						cur.putAll(prev);
																						return cur;
																					}
													).join().getItems();							
}
```





## <span style="color:#802548">_필요한 정보만 json데이터로 보내기_</span>

- 마침내 API를 완성하고 내가 멘토링을 할 때 받은 서버로 배포를 진행한 뒤 front분과 협업을 시작했다. 
- 프론트 입장에서 필요없는 데이터들은 받아도 눈에 거슬릴 뿐이며, 트래픽이 높아지기 때문에 필요한 값만 보내달라는 이야기를 받았다.
- HashMap을 통해 해당 요구사항을 구현했다.

```java
public HashMap<String, Object> getEvent(String eventId)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = serviceAuth();

	Event event = service.events().get("ddddd@gmail.com", eventId).execute();
	HashMap<String, Object> map = new HashMap<String, Object>();
	map.put("summary", event.getSummary());
	map.put("description", event.getDescription());
	map.put("start", event.getStart());
	map.put("end", event.getEnd());
	map.put("id", event.getId());
	
	return map;
}
```

## <span style="color:#802548">_map 대신 DTO 사용하기_</span>
- 그리고 map으로 front에 return하는 것은 유지보수에 매우 좋지 않으므로 그것을 Dto나 VO로 바꾸라는 조언을 받았다. 
- 특히 front와의 의사소통에 Swagger를 사용할 경우, map은 지원되지 않기 때문에 VO로 바꿔달라는 이야기를 들었다.
- 어떻게 담아야할 까 고민했는데, 처음엔 req와 res를 구분해서 담지 않고 그냥 담았다. 
	- 어떤 게 요청이고 어떤 게 응답용인지 변수명에 따른 구분이 많이 어려웠다. 

```java
public class CalendarEventDto {
	
	@NotBlank
	private String startDateTime; 
	@NotBlank
	private String endDateTime;
	@NotBlank
	private EventDateTime startEventDateTime; 
	@NotBlank
	private EventDateTime endEventDateTime;
}
```


- 거기다 req와 res에 받아야하는 data의 type이 서로 달라야 했다. 
- response로 뿌리는 데이터와 front에서 받은 request 데이터로 구글에 보낼 때 서로 다른 data type이 요구됐기 때문이다. 
- 따라서 응답과 요청 DTO를 분리해서 만들었다.

```java
public class CalendarEventReqDto {
	
	@NotBlank
	private String startDateTime; 
	@NotBlank
	private String endDateTime;
}

public class CalendarEventResDto {

	@NotBlank
	private EventDateTime startDateTime; 
	@NotBlank
	private EventDateTime endDateTime;

}
```

- 그리고 response는 휴일과 이벤트일정을 동시에 보내야 해서 아래와 같이 새로운 클래스에 담아서 보냈다. 

```java
public class CalendarEventListResponse {
	private List<CalendarEventResDto> calendarEventList;
	private List<CalendarHolidayResDto> holidayList;
}
```

- 아래와 같이 set해주면 데이터가 기존에 map으로 만들었던 것과 동일하게 보내진다. 

```java
CalendarEventListResponse response= new CalendarEventListResponse();
response.setCalendarEventList(calendarEventList);
response.setHolidayList(holidayList);
```
