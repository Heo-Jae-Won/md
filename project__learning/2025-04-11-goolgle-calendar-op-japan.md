## <span style="color:#802548">_캘린더 중복 방지 로직_</span>

- 처음 활용했던 것은 Time package의 ZonedDateTime class에 속한 method isBefore(), isAfter()였다. 
- 그러나 이로는 부족했다. isBefore()와 isAfter()라는 method로는 회의가 끝나는 시간에 맞춰 등록하는 경우 중복으로 간주됐다. 15:00에 끝나면 15:00시작으로 등록하는 것이 대부분의 경우였기 때문에 이러한 method는 사용할 수가 없었다. 

```java
isAfterPreviousStartDateTime = convertedEndDateTime.atZone(ZoneId.of("Asia/Seoul")).isBefore(dateTime);
```

- 그래서 찾아보니 compareTo라는 method가 있었다. 
- 동일한 경우도 포함하여 이전인지 이후인지 비교할 수 있었다. 그래서 해당 method를 사용했다. 

- 하지만 원하는 결과가 출력되지 않았다. 이전일정과 이후일정이 서로 연동되어있어서 둘을 떼내서 비교하는 것이 너무 어려웠다. 
- 예를 들어 이전일정의 끝나는 시간과 겹치면 안되고 이후 일정의 시작하는 시간과 겹치면 안되는데, for문으로는 이전일정의 시작시간과 끝나는 시간만 가져오기에 둘 중 하나에서 반드시 오류가 났다.

```java
for (Event item : items) {
	previousThanListStartDateTime = inputEndDateTime.atZone(ZoneId.of("Asia/Seoul"))
			.compareTo(listStartDateTime) <= 0;

	if (!previousThanListStartDateTime) {
		throw new Exception("이미 꽉참");
	}
}
```

- 그래서 헤매다가 이전일정 1개와 이후 일정 1개만 비교하는 것으로 범주를 팍 좁혔다. 

```java
Events events = service.events().list("ddddd@gmail.com").setTimeMin(startDateTime).setMaxResults(2)
				.setSingleEvents(true).setOrderBy("starttime").execute();
```
## <span style="color:#802548">_5. 캘린더 CRUD with front_</span>

- 마침내 API를 완성하고 내가 멘토링을 할 때 받은 서버로 배포를 진행한 뒤 front분과 협업을 시작했다. 
- 그런데 여러 가지 추가사항이 들어갔다. 

```java
Events events = service.events().list("ddddd@gmail.com").setTimeMax(lastDay).setTimeMin(firstDay)
				.execute();
```

 - 필요없는 데이터들은 걸러내고 그 중에 필요한 몇가지만 추려서 새롭게 데이터를 만들어서 보낸다던가 하는 식이다.  event를 가져는 경우에 아래와 같이 필요사항만 map에 넣어서 보내는 형태로 진행했다. 

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

- 그리고 front에서 수정의 경우 같은 시간대로 수정하면 중복되지 않게끔 설정해 달라고 했다. 
- 실제로 잘못 눌러서 똑같은 시간대로 수정을 했는데, 수정을 했더니 중복이라고 뜨면 당황스러울 것이다. 

```java
if (inputEndDateTime.atZone(ZoneId.of("Asia/Seoul")).equals(listEndDateTime)
		&& inputStartDateTime.atZone(ZoneId.of("Asia/Seoul")).equals(listStartDateTime)) {
	break;
}
```


## <span style="color:#802548">_7. 캘린더 CRUD  추가된 요구사항_</span>
- 회사의 달력이 멘토분의 핸드폰과 연동되어 있어 밤에 울리지 않게 해달라는 요청을 받았다. 
- 당시에는 설정을 보고서 그런 알림을 끄는 설정이 없었기에 없다고 대답했는데, 없다보다는 한 번 알아보고 답변드리겠습니다 라고 하는 게 옳았다. 
- 네이버에 나중에 다시 치니까 바로 알림 끄는 법이 나왔기 때문이다. 
- 요구사항에 안된다가 아니라 알아보겠습니다라고 답변하는 것으로 방향을 정했다. 그게 더 바람직하다. 

​

- 그래서 이후 휴일도 포함되었으면 한다는 요청이 제기되었을 때 알아보겠다는 답변을 드렸다. 
- 다행히 개발자 분이 천문 쪽 특일이라는 공휴일을 알려주는 공공 API를 활용하면 된다는 정보를 제시해주셨다. 
- 그래서 해당 API를 찾아봤는데 붙이면 google과 공공 모두 데이터를 받아오는 데 시간이 꽤나 걸릴 거 같았다. 
- 그런 와중에 프론트분이 구글 캘린더 api로도 휴일을 받아올 수 있다고 하셨다. 그래서 알아보니 가능하며 대체휴일도 받아올 수 있어 그 방향으로 진행하기로 했다. 
```java
Events events = service.events().list("ko.south_korea#holiday@group.v.calendar.google.com").setTimeMax(lastDay).setTimeMin(firstDay)
				.execute();
```
## <span style="color:#802548">_8. 캘린더 CRUD feedback_</span>
- 그리고 피드백을 받은 결과 map을 쓰는 것은 유지보수에 매우 좋지 않으므로 그것을 Dto나 VO로 바꾸라는 조언을 받았다. 
- 그래서 아래와 같이 VO로 전부 바꾸었다. 
```java
List<CalendarEventResDto> calendarEventList = calendarService.getEventList(firstDayOfMonth, lastDayOfMonth);
List<CalendarHolidayDto> holidayList = calendarService.getHolidayList(firstDayOfMonth, lastDayOfMonth);
CalendarEventListResponse response= new CalendarEventListResponse();
response.setCalendarEventList(calendarEventList);
```
- 어떻게 담아야할 까 고민했는데, 처음엔 req와 res를 구분해서 담지 않고 그냥 담았다. 
- 그런데 req와 res에 받아야하는 data의 type이 서로 달라야 했다. 
- response로 뿌리는 데이터와 front에서 받은 request 데이터로 구글에 보낼 때 서로 다른 data type이 요구됐기 때문이다. 
```java
public class CalendarEventReqDto {
	
	@NotBlank
	private String startDateTime; 
	@NotBlank
	private String endDateTime;
}
```
- 그래서 req와 res를 구분해서 만들었다. 
```java
@Data
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
	private List<CalendarHolidayDto> holidayList;
}
```
- 아래와 같이 set해주면 데이터가 기존에 map으로 만들었던 것과 동일하게 보내진다. 

```java
CalendarEventListResponse response= new CalendarEventListResponse();
response.setCalendarEventList(calendarEventList);
response.setHolidayList(holidayList);
```



## <span style="color:#802548">_9. 회사일정과 휴일 달력 parallel 하게 가져오기_</span>
- 이전에 만든 소스코드는 회사일정과 휴일을 sequential하게 가져왔다. 그래서 느렸다.
- 따라서 회사일정을 가져오면서 동시에 휴일도 가져오게끔 하려했다.
- 그래서 아래와 같이 바꿨다. 그러나 이미 구글의 서비스를 호출할 때 execute를 하기 때문에 아래 구조로는 CompletalbeFuture를 활용해도 sequential이었다.

```java
public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = getServiceAuth();

	// 선택한 날짜의 달을 기준으로 초일(firstDay)부터 말일(lastDay)까지 회사 google calendar 목록을 조회
	List<CalendarEventRes> list = new ArrayList<>();
	DateTime firstDay = new DateTime(firstDayOfMonth);
	DateTime lastDay = new DateTime(lastDayOfMonth);
	Events events = service.events().list("ffff@gmail.com").setTimeMax(lastDay).setTimeMin(firstDay)
			.execute();

	Events events1 = service.events().list("ko.south_korea#holiday@group.v.calendar.google.com").setTimeMax(lastDay)
			.setTimeMin(firstDay).execute();

	CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(() -> { return events; });
	CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(() -> { return events1; });


	List<Event> items =  companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> {
																						prev.putAll(cur);
																						return prev;
																					}).join().getItems();							
	//events.getItems();

	for (Event event : items) {
		CalendarEventRes calendarEventDto = new CalendarEventRes();
		calendarEventDto.setSummary(event.getSummary());
		calendarEventDto.setDescription(event.getDescription());
		calendarEventDto.setStartDateTime(event.getStart());
		calendarEventDto.setEndDateTime(event.getEnd());
		calendarEventDto.setEventId(event.getId());

		list.add(calendarEventDto);
	}
	return list;

}
```

- 그런 이유로 아래와 같이 execute를 CompletableFuture에서 실행하게 변경했다.
```java
public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = getServiceAuth();

	// 선택한 날짜의 달을 기준으로 초일(firstDay)부터 말일(lastDay)까지 회사 google calendar 목록을 조회
	List<CalendarEventRes> list = new ArrayList<>();
	DateTime firstDay = new DateTime(firstDayOfMonth);
	DateTime lastDay = new DateTime(lastDayOfMonth);

	CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(() -> { return service.events().list("fffff@gmail.com").setTimeMax(lastDay).setTimeMin(firstDay)
			.execute(); });
	CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(() -> { return service.events().list("ko.south_korea#holiday@group.v.calendar.google.com").setTimeMax(lastDay)
			.setTimeMin(firstDay).execute(); });


	List<Event> items =  companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> 
																					{
																						cur.putAll(prev);
																						return cur;
																					}
													).join().getItems();							

	for (Event event : items) {
		CalendarEventRes calendarEventDto = new CalendarEventRes();
		calendarEventDto.setSummary(event.getSummary());
		calendarEventDto.setDescription(event.getDescription());
		calendarEventDto.setStartDateTime(event.getStart());
		calendarEventDto.setEndDateTime(event.getEnd());
		calendarEventDto.setEventId(event.getId());

		list.add(calendarEventDto);
	}
	return list;

}
```

- 안타깝게도 매 service 실행마다 authtoken이 날라가야 할까? 날라가야 하면 아래처럼....
- 따라서 auth 인증 서비스도 각 completableFuture마다 들어가줘야 한다.
```java
public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = getServiceAuth();

	// 선택한 날짜의 달을 기준으로 초일(firstDay)부터 말일(lastDay)까지 회사 google calendar 목록을 parallel하게 조회 
	
	List<CalendarEventRes> list = new ArrayList<>();
	DateTime firstDay = new DateTime(firstDayOfMonth);
	DateTime lastDay = new DateTime(lastDayOfMonth);
	CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(() -> 
																					{ 
																						Calendar service = getServiceAuth();
																						return service.events().list("openobjectnet@gmail.com").setTimeMax(lastDay).setTimeMin(firstDay)
																																									.execute(); 
																					}
																			);
	CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(() -> 
																					{ 
																						Calendar service = getServiceAuth();
																						return service.events().list("ko.south_korea#holiday@group.v.calendar.google.com").setTimeMax(lastDay)
																																											.setTimeMin(firstDay).execute(); 
																					}
																			);
	List<Event> items =  companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> 
																					{
																						cur.putAll(prev);
																						return cur;
																					}
													).join().getItems();							

	for (Event event : items) {
		CalendarEventRes calendarEventDto = new CalendarEventRes();
		calendarEventDto.setSummary(event.getSummary());
		calendarEventDto.setDescription(event.getDescription());
		calendarEventDto.setStartDateTime(event.getStart());
		calendarEventDto.setEndDateTime(event.getEnd());
		calendarEventDto.setEventId(event.getId());

		list.add(calendarEventDto);
	}


	return list;

}
```
