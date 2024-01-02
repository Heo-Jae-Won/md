## <span style="color:#802548">_1. 구글 API 사용_</span>
- 처음 구글 calendar API를 활용해 회의실 일정 예약 시스템을 만들어보라는 업무 지시가 떨어졌을 때, 솔직히 두려움이 매우 앞섰다. 
- 프론트와의 협업은 그게 처음이었기 때문이다. 역시나 난관에 매우 빠르게 봉착했다. 
- 구글의 API를 활용하려면 공식 문서 상에서는 client ID라는 방식으로 인증토큰을 발급받으라고 되어있었다.

​

- 그런데 client ID방식은 동의 화면을 구성해야 하고, 거기에는 약관, 개인정보 처리 방침 등 다양한 기업 상품과 관련된 설명들이 포함되어 있었다. 
- 그러나 나는 client ID방식을 쓰면 안됐다. 남에거에다 하는 게 아니었다. 그래서 다른 방식을 알아봤다. 나는 공식문서가 client ID로 되어있어서 반드시 그것으로 진행해야하는 줄 알았는데 다행히 그렇지 않았다. 서비스 계정 방식이라는 게 있었다.

- 그래서 찾아본 결과 service 계정을 만드는 방식으로도 인증이 가능했다. 
- 아래 scope는 client ID 인증인 경우에만 중요하고 service account의 경우에는 크게 상관없는 것으로 보인다. 
- 그렇게 인증에 성공했고 api가 정상으로 작동한다.

https://velog.io/@minwest/%EC%84%9C%EB%B9%84%EC%8A%A4-%EA%B3%84%EC%A0%95%EC%9C%BC%EB%A1%9C-Google-Calendar-API-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0

 
## <span style="color:#802548">_2. 캘린더 설정_</span>
- 그런데 캘린더에 따로 또 설정을 해줘야만 인증이 service account가 제역할을 할 수 있었다. 
- 다행히 아래에서 잘 설명되어 있어 아래를 보고 따라했다. 
- 나는 JWT를 따로 보내지 않아도 됐다. JSON 파일에 이미 token정보가 있어 그를 바탕으로 인증을 진행하기 때문이었다.

https://kibua20.tistory.com/87

## <span style="color:#802548">_3. 캘린더 기본 CRUD_</span>
- 이제 캘린더 CRUD를 개발하면 된다! setting에 시간을 거의 25%는 쓴 것 같다. 
- 기본 CRUD 자체도 그렇게 오래 걸리지는 않았다. 내 local에서 postman으로 찌르는 형식으로 진행했다. 

- 그런데 시간대의 문제가 있었다. 내가 써놓은 시간대는 세계시 기준으로 들어가서 내가 의도한 바와 맞지 않게 들어갔다. 
- 아래와 같이 프론트에서 ISO스트링으로 보낸다고 해보자.
```
startTime: 2023-03-11T15:00:00.000Z
endTime: 2023-03-11T15:45:00.000Z
```
- 그럼 내 달력에는 2023-03-12T00:00:00.000Z가 시작시간이고 끝나는 시간은 00:45:00이었다. 
- 9시간 뒤에 등록된 셈이다. 그래서 알고보니 한국시간대는 +09:00여서 그랬던 것이었다. 
- 이러한 시간대의 오류를 알고서는 서버에서 알아서 바꿔서 보내도록 했고 정상처리됐다. 

​

## <span style="color:#802548">_4. 캘린더 CRUD 추가 로직_</span>
- 그러나 기본 CRUD로는 충분하지 않았다. 
- 우선 내가 판단하기에 이전 일정은 예약에는 필요가 없으므로 보여줄 필요가 없다고 생각했다. 
- 그래서 아래와 같이 현재 시간 이후의 일정만을 가져오게 했다. 
```java
isAfterNow = dateTime.atZone(ZoneId.of("Asia/Seoul")).isAfter(now);
```
- 또한 일정이 중복되면 안되기에 중복 방지가 필요했다. 
- 그래서 이 부분을 고안하는 데 엄청나게 오랜 시간이 걸렸다. 
- 내가 한 번도 쓰지 않았던 Java의 Time package를 활용해야했기 때문이다. 

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
- 하지만 이것으로는 부족했다. 그러다 실수로 몇개 내용을 지우면서 돌렸는데, 너무 잘되는 것이었다. 
- 그래서 디버거로 무슨 일이 일어나나 관찰했더니 겹치는 일정이 되는 순간 이전 일정이 겹치는 일정이 되고, 겹치지 않는 일정이 되는 순간 비교로직이 잘 수행되는 마법같은 일이 일어났다. 

- 그래서 다행스럽게도 문제를 해결했다. 그 로직을 발견하고서야 이해했으니 그야말로 우연의 산물이다. 
- 이 우연이 아니었으면 여태 끙끙 앓고 있었을 지도 모른다. 그래도 내가 범주를 좁혀갔기 때문에 이런 우연도 발견한 것이니까 내 노력이이 없는 것은 아니라고 생각한다. 
```java
Events events = service.events().list("ddddd@gmail.com").setTimeMin(startDateTime).setMaxResults(2)
				.setSingleEvents(true).setOrderBy("starttime").execute();
```
## <span style="color:#802548">_5. 캘린더 CRUD with front_</span>

- 마침내 API를 완성하고 내가 멘토링을 할 때 받은 서버로 배포를 진행한 뒤 front분과 협업을 시작했다. 
- 그런데 여러 가지 추가사항이 들어갔다. 1달치로 리스트를 끊어서 받아온다던가,
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
## <span style="color:#802548">_6. 캘린더 CRUD with 사내 project_</span>
- 내가 여태 했던 것은 내 개인 local이었고 이제는 사내 project 구조에 맞게 바꿔 갈아껴야 했다. 
- 그래서 사내 프로젝트 구조를 내 local에 그대로 재현해서 실험했다. 
- 그런데 서로 환경이 달랐다. 나는 legacy + eclipse였고, 사내 프로젝트는 vscode + boot 환경이었다. 그래서 dependecy를 추가하지 못하고 있었다.

- 하지만 언젠가는 dependency 충돌 실험이 필요하다는 것을 과거 경험에서 느꼈던 바라서 결국 vscode로 boot를 진행했다.
- Extension pack for java, spring boot extension, lombok annotation을 깔아서 진행했다. 
- 나는 boot 프로젝트를 다뤄본 적이 없어서 내장톰캣이 어떤 것인지 몰랐다. 
- 그래서 community connector server를 깔아서 tomcat을 넣었다. 


- 나는 그게 내장 tomcat인줄 알았다. 
- 그래서 maven으로 install하고 jar파일을 톰캣에 deploy하고 publish했다. 
- 그런데 아무 일도 일어나지 않았다. 계속 404 오류만이 무한하게 떴다. 
- 그래서 나중에 알고보니  community connector server로 깔아서 쓰는 것은 외장 톰캣이고, boot project는 main class에서 그냥 run하면 내장 톰캣을 돌리는 구조였다. 그걸 몰라서 정말 오랫동안 헤맸다. 

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
- pull request를 처음에 잘못 이해해서 push로 넣어버려서 주눅들어있다가 pull request를 두번째에는 잘 했다. 
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