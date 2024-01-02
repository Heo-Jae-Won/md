## <span style="color:#802548">_1. string -> LocalDateTime with parse_</span>
- 아래와 같이 string 으로 front에서 보냈다면, string으로 받게 될 것이다. 
- 단, iso-8601을 준수해야 하므로 빈칸이 있다면 T자로 치환한다. 
- 나는 '2023-05-08 20:42:00'로 yyyy-mm-dd HH;mm:ss 형태로 보냈다.

​
- parse() 안에 parameter로는 charSequence interface가 요구된다.
  - charSequence interface는 String, StringBuilder, StringBuffer class를 구현체로 갖고 있다. 
```java
LocalDateTime.parse(charSequence);
String dateString = scheduleFilteringConditionDto.getScheduleDate(); //YYYYMMDD HHMMSS
LocalDateTime dateTime = LocalDateTime.parse(dateString.replace(" ", "T"));
```
## <span style="color:#802548">_2. string -> LocalDateTime with from_</span>
- 아래는 from(TemporalAccssor)를 써서 LocalDateTime으로 변환한다. 
- TemporalAccessor interface는 아래와 같은 구현체를 갖고 있다.
  - LocalDateTime, 
  - LocalDate, 
  - LocalTime, 
  - ZonedDateTime, 
  - Instant
-  따라서 그 중에 넣으면 된다.

```java
LocalDateTime.from(TemporalAccssor)
```
- 그 중에서 Instant라는 구현 class는 익숙하지 않으니 예시를 보자. 
- 날짜 문자열을 front에서 어떻게 주느냐에 따라서 처리법이 살짝 달라진다.
  - iso-string으로 준다면 replcae 과정 없이 넣으면 되고, 
  - iso-string으로 주지 않는다면 replcae 과정을 거쳐야 한다.
```java
Instant.from(DateTimeFormatter.ISO_DATE_TIME.parse("날짜 문자열"))
```
- 다 이어주면 아래와 같은 소스코드가 완성된다. 
```java
LocalDateTime inputEndDateTime = LocalDateTime
				.from(Instant.from(DateTimeFormatter.ISO_DATE_TIME.parse("날짜 문자열")));
```
- 아래는 DateTimeFormatter의 미리 정의된 예시들이다.나는 ISO_DATE_TIME을 활용했다. 
- ISO_DATE_TIME를 사용한다면 위의 날짜 문자열은 아래와 같이 YYYY-MM-DDTHH:MM:SS.sssZ. 형태여야 한다. 
- 그걸 바로 js의 toISOString()으로 해줄 수 있다. 

​

- 아래와 같은 오류가 나게 될 수 있다.
- 위의 방법이 안된다면 아래 방법을 사용하면 된다. 아래는 KR 타임이다.
```java
Instant instant = Instant.parse(dateString);
LocalDateTime dateTime = LocalDateTime.ofInstant(instant, ZoneId.of("Asia/Seoul"));
```
## <span style="color:#802548">_3. 시간대의 선후 비교_</span>
- 시간대의 선후를 비교하려면 LocalDateTime에서 zonedDateTime class로 변환해야 한다. 
- 그건 간단하게 atZone(zone구역)으로 써준다. 

- 대한민국은 서울이니까 서울로 쓴 것이다. 

```java
.atZone(ZoneId.of("Asia/Seoul"))
```
- 런던은 런던에 맞는 timeZone 값을 넣어주면 된다. 보니까 주요 대도시 기준으로만 있는 것으로 보인다. 

```java
.atZone(ZoneId.of("Europe/London"))
```
- 위와같이 zone을 주면 아래는 ZonedDateTime으로 변환된다. 
```java
inputStartDateTime.atZone(ZoneId.of("Asia/Seoul")) //ZonedDateTime type
```
- LocalDateTime, LocalTime, ZonedDateTime 등에서는 시간대를 비교할 수 있다. 
```java
A.compareTo(B)
```
- A와 B 모두 같은 class로 맞추는 게 좋다.
- 비교한 값이 0보다 작으면 A는 B 이전에 존재하는 것이고, 
- 0과 같으면 둘이 동일한 것이며, 

- 0보다 크면 A가 B 이후의 시간대에 위치한 것이라고 보면 된다. 
- 나는 seoul 기준으로 해야해서 ZonedDateTime을 활용한 것이다. 
- 나는 처음을 시작시간으로 놓고 끝을 종료시간으로 놓는 것을 선호한다.
```java
boolean previousThaninputEndDateTime = 시간.atZone(ZoneId.of("Asia/Seoul"))
				.compareTo(시간.atZone(ZoneId.of("Asia/Seoul"))) < 0;
```
- compareTo가 아니라 직접 isBefore(), isEqual(), isAfter()를 활용할 수도 있다.
- 하지만 숫자로 나오는 걸 boolean으로 판단하는 게 더 편해서 compareTo() method를 활용했다.

​

## <span style="color:#802548">_4. 현재시간 알아내기_</span>
- 아래와 같이 하면 된다. 
```java
LocalDateTime.now()
```
## <span style="color:#802548">_5. LocalDateTime으로 기간 구하기_</span>

- 현재시간과 예정일까지의 기간을 구하는 방법은 Duration class를 활용하는 것이다. 
```java
Duration duration = Duration.between(LocalDateTime.now(), LocalDateTime.now().plusDays(1));
```
## <span style="color:#802548">_6. LocalDateTime -> Timestamp_</span>
- mysql에 시간과 일자를 같이 저장하기 위해 Timestamp 형태로 변형한다. 
- DateTime이라는 db type보다는 timestamp가 바이트가 덜 들어가서 timestamp로 저장했다. 
```java
Timestamp.valueOf(schedule.getScheduleDepartDatetime())
//private LocalDateTime scheduleDepartDatetime;
```