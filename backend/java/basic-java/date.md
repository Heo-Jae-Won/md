## <span style="color:#802548">_Calendar와 Date_</span>
- calendar는 abstract class라 instance를 얻어와야 한다.

```java
Calendar cal = new Calendar();        //error
Calendar cal = Calendar.getInstance();//OK
```

- Calendar -> Date 간 변환은 아래와 같다.

```java
Calendar cal = Calendar.getInstance();/
Date date = new Date(cal.getTimeInMillis());
```

- Date -> Calendar간 변환은 아래와 같다.

```java
Date d = new Date();
Calendar cal = Calendar.getInstance();
cal.setTime(d);
```

- 현재 시간을 얻는 방법이다.

```java
Date now = new Date();
Calendar now = Calendar.getInstance().getTime();
```

- 시간대를 SDF로 문자열로 변형시키는 방법이다.

```java
Date now = new Date();
Calendar now = Calendar.getInstance().getTime();

SimpleDateFormat sdf = new SimpleDateFormat("yyyy년 MM월 dd일 HH시 mm분 ss초")
String formattedNow = sdf.format(now); 
```

- 시간대의 차이를 알고 싶다면 직접 구해야한다.

```java
final int[] TIME_UNIT = {3600, 60, 1}; //큰 단위를 앞에 놓기
final String[] TIME_UNIT_NAME = {"시간", "분", "초"};
Calendar time1 = Calendar.getInstance();
time1.set(Calendar.HOUR_OF_DAY, 10);
time1.set(Calendar.MINUTE, 20);
time1.set(Calendar.SECOND, 30);

Calendar time2 = Calendar.getInstnace();
time2.set(Calendar.HOUR_OF_DAY, 20);
time2.set(Calendar.MINUTE, 30);
time2.set(Calendar.SECOND, 10);

long difference = Math.abs(time2.getTimeInMillis() - time1.getTimeInMillis()) / 1000;

String tmp = "";
for (int i = 0; i < TIME_UNI.length; i++) {
  tmp += difference/TIME_UNIT[i] + TIME_UNIT_NAME[i];
  difference %= TIME_UNIT[i];
}
```


- 시간을 더한다면 add()를 쓴다.

```java
Calendar date = Calendar.getInstance();
date.set(2015,7,31);

date.add(Calendar.DATE, 1);
date.add(Calendar.MONTH, -6);
```

- 해당일이 몇 요일인지 알고 싶다면 get()을 활용한다.

```java
Calendar date = Calendar.getInstance().getTime();
int weekday = date.get(Calendar.DAY_OF_WEEK);
```

- choiceFormate은 범위에 맞게 문자열로 변환시켜준다.
- 단, 정렬 범위는 낮은 값부터 큰 숫자 순으로 적어야 한다.
- <는 해당 범위를 불포함하고, #은 해당 범위를 포함한다.
  - '60#D|70<C'는 60~70, 71부터 그 위까지다.

```java
String pattern = "60#D|70#C|80<B|90#A";
int[] scores = {91, 90, 80, 88, 70, 52, 60};

ChoiceFormat form = new ChoiceFormat(pattern);

for (int i = 0; i < scores.length; i++) {
  System.out.println(score[i]+ ":" + form.formate(scores[i]));
}
/*
  91:A
  90:A
  80:C
  88:B
  70:C
  52:D
  60:D
*/
```

## <span style="color:#802548">_LocalDateTime_</span>
- Temporal, TemporalAccessor, TemporalAdjuster interface를 구현한 경우는
  - LocalDate 등의 class다.
- TemporalAmount interface를 구현한 경우는
  - Period, Duration 등의 class다.


- 먼저 LocalDateTime을 알아보자.
- 현재 시간을 얻어오려면 now를 사용한다.

```java
LocalDateTime dateTime = LocalDateTime.now();
```

- 지정된 시간을 숫자로써 얻어오려면 of를 사용한다.

```java
LocalDateTime dateTime = LocalDateTime.of(2015, 11, 23);
```

- 지정된 시간을 문자로써 얻어오려면 parse를 사용한다.

```java
LocalDateTime LocalDateTime.parse("1999-12-31 23:59:59");
```

- 해당 시간의 년월일을 얻는 방법이다.

```java
LocalDateTime dateTime = LocalDateTime.now();
int year = dateTime.getYaer();
int month = dateTime.getMonthValue();
int day = dateTime.getDayOfMonth();
```

- 해당 시간을 변경하는 방법이다.
- 단, plus를 하더라도 now의 자체 값은 절대로 변하지 않는다.
- LocalDateTime은 immutable하다는 것을 인지해야 한다.

```java
LocalDateTime now = LocalDateTime.now();
//2022-05-16T22:08:59.398040

now.plusYears(2) = 2024-05-16T22:08:59.398040
now.plusMonths(5) = 2022-10-16T22:08:59.398040
now.plusDays(10) = 2022-05-26T22:08:59.398040
now.plusHours(5) = 2022-05-17T03:08:59.398040
now.plusMinutes(10) = 2022-05-16T22:18:59.398040
now.plusSeconds(20) = 2022-05-16T22:09:19.398040

now.minusYears(2) = 2020-05-16T22:08:59.398040
now.minusMonths(5) = 2021-12-16T22:08:59.398040
now.minusDays(10) = 2022-05-06T22:08:59.398040
now.minusHours(5) = 2022-05-16T17:08:59.398040
now.minusMinutes(10) = 2022-05-16T21:58:59.398040
now.minusSeconds(20) = 2022-05-16T22:08:39.398040
```

## <span style="color:#802548">_Instant_</span> 
- instant는 epoch time(1970년 1월 1일 0시0분0초 UTC)부터 경과된 시간을 나노초로 표현한다.
- 사람이 알기 쉬운 방식이 아니라 기계를 위한 방식이다. timestamp를 사용한다.

```java
Instnat now = Instant.now();
Instant now2 = Instant.ofEpochSecond(now.getEpochSecond());
Instant now3 = Instant.ofEpochSecond(now.getEpochSecond(), now.getNano());

long epochSec = now.getEpochSecond();
int nano = now.getNano();
long epochMilliSec = now.toEpochMilli();
```

- 다만 UTC +0:00이 표준이라 그냥 쓰긴 어렵고 offsetDateTime으로 써야 한다.

-  Instant -> Date  변환이 가능하다.

```java
Instnat now = Instant.now();
Date date = Date.from(now);
```

- Date -> Instant 변환이 가능하다.

```java
Date now = new Date();
Instant instante = now.toInstant();
```

## <span style="color:#802548">_ZonedDateTime과 OffsetDateTime_</span> 
- LocalDateTime에 시간을 추가하면 ZonedDateTime이 된다.

```java
LocalDateTime now = LocalDateTime.now();
ZoneId zid = ZoneId.of("Asia/Seoul");
ZonedDateTime zdt = now.atZone(zid);
```

- ZoneId에서 더 안전하게 다루는 것이 OffsetDateTime이다.

```java
ZoneOffset krOffset = ZonedDateTime.now().getOffset();
OffsetDateTime odt = OffsetDateTime.of(date, time, krOffset);
OffsetDateTime odt = zdt.toOffsetDateTime();
```

- ZonedDateTime과 Calendar는 서로 변환 가능하다.

```java
LocalDateTime date = Calendar.getInstance().getTime().();
Calendar cal = 
```


## <span style="color:#802548">_Period, Duration_</span> 

- period는 날짜에 관한 것이다.

```java
LocalDate date1 = LocalDate.of(2014, 1, 1);
LocalDate date2 = LocalDate.of(2015, 12, 31);
Period pe = Period.between(date1, date2);

System.out.println("Years: " + period.getYears());
System.out.println("Months: " + period.getMonths());
System.out.println("Days: " + period.getDays());
```

- duration은 시간에 관한 것이다.

```java
LocalTime time1 = LocalTime.of(00,00,00);
LocalTime time2 = LocalTime.of(12,34,56);
Duration du = Duration.between(time1, time2);

System.out.println("Seconds: " + duration.getSeconds());
System.out.println("Nano Seconds: " + duration.getNano());
```

- period를 바로 long으로 바꿀 수 있다.

```java
LocalDate date1 = LocalDate.of(2015, 11, 28);
LocalDate date2 = LocalDate.of(2015, 11, 29);

long period = date2.toEpochDay() - date1.toEpochDay(); //1
```

