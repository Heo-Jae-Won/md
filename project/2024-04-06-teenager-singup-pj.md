## <span style="color:#802548">_파일을 만들고 권한 주기_</span>

- 옛날버전

```java
File file=new file("/dir/","game");
Runtime.getruntime().exec("chmod 777" + "/dir/game");

filr.createNewFile();
file.setExecutable(true,false)
file.SetReadable(true,false)
file.setWritable(true,false)

FileOutputStream fout = new FileoutputStream(file)
osw = OutputStreamWtiter(fout)
osw.write("cipherkey")
osw.flush()
```

- 최신 버전

```java
public class Main {
    public static void main(String[] args) throws Exception {
        Path path = Paths.get("/dir", "game");

        // Set file permissions
        Files.createFile(path);
        Files.setPosixFilePermissions(path, PosixFilePermissions.fromString("rwxrwxrwx"));

        // Write to file
        String content = "cipherkey";
        Files.write(path, content.getBytes());
    }
}
```

- 더 좋은 버전

```java
public class Main {
    public static void main(String[] args) {
        Path path = Paths.get("/dir", "game");

        // Writing to file
        String content = "cipherkey";
        try (BufferedWriter writer = Files.newBufferedWriter(path)) {
            writer.write(content);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

## <span style="color:#802548">_stream으로 sum하기_</span>
- stream으로 특정 field를 sum할 때

```java
class ConviDto {
    private long count;
    // Other fields and methods...

    public long getCount() {
        return count;
}

List<ConviDto> conviDtoList = new ArrayList<>();

        // Populate conviDtoList with data from the database...

        // Summing the count field using streams
        long totalCount = conviDtoList.stream()
                .mapToLong(ConviDto::getCount)
                .sum();
    // Other getters and setters...
}
```

- stream으로 특정 location 별 count를 sum할 때

```java
public class Main {
    public static void main(String[] args) {
        // Assuming conviDtoList is your ArrayList<ConviDto> containing data from the database
        List<ConviDto> conviDtoList = new ArrayList<>();

        // Populate conviDtoList with data from the database...

        // Grouping counts by location
        Map<String, Long> countByLocation = conviDtoList.stream()
                .collect(Collectors.groupingBy(ConviDto::getLocation,
                        Collectors.summingLong(ConviDto::getCount)));

        // Printing count by location
        countByLocation.forEach((location, count) -> System.out.println(location + ": " + count));
    }
}

class ConviDto {
    private long count;
    private String location;

    public long getCount() {
        return count;
    }

    public String getLocation() {
        return location;
    }

    // Other getters and setters...
}
```

## <span style="color:#802548">_server에서 다른 server로 request_</span>
- resttemplate은 옛날 버전
```java
public class Main {
    public static void main(String[] args) {
        // Create SimpleClientHttpRequestFactory and configure timeouts
        SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
        factory.setConnectTimeout(120 * 1000);
        factory.setReadTimeout(120 * 1000);

        // Create RestTemplate with the SimpleClientHttpRequestFactory
        RestTemplate restTemplate = new RestTemplate(factory);

        // Prepare headers
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.setContentType(MediaType.APPLICATION_JSON);

        // Assuming inputMap is a Map<String, Object> containing JSON data
        // You need to import org.json.JSONObject for this
        JSONObject jsonObject = new JSONObject(inputMap);
        String jsonString = jsonObject.toString();

        // Alternatively, if you have Gson library available:
        //String jsonString = new Gson().toJson(inputMap);

        // Create HttpEntity with JSON data and headers
        HttpEntity<String> httpEntity = new HttpEntity<>(jsonString, httpHeaders);

        // Make HTTP POST request
        ResponseEntity<String> response = null;
        try {
            response = restTemplate.exchange(requestUri, HttpMethod.POST, httpEntity, String.class);
            String result = response.getBody();
            // Process the result
        } catch (HttpClientErrorException e) {
            throw e;
        }
    }
}
```

- 최신 webClient 방식
```java
public class Main {
    public static void main(String[] args) {
        // Create WebClient
        WebClient.Builder webClientBuilder = WebClient.builder();

        // Configure timeouts
        WebClient webClient = webClientBuilder
                .clientConnector(new ReactorClientHttpConnector(HttpClient.create().responseTimeout(java.time.Duration.ofSeconds(120))))
                .build();

        // Prepare headers
        HttpHeaders httpHeaders = new HttpHeaders();
        httpHeaders.setContentType(MediaType.APPLICATION_JSON);

        // Prepare JSON data
        // Assuming inputMap is a Map<String, Object> containing JSON data
        // You need to import org.json.JSONObject for this
        JSONObject jsonObject = new JSONObject(inputMap);
        String jsonString = jsonObject.toString();

        // Alternatively, if you have Gson library available:
        String jsonString = new Gson().toJson(inputMap);

        // Create HttpEntity with JSON data and headers
        HttpEntity<String> httpEntity = new HttpEntity<>(jsonString, httpHeaders);

        // Make HTTP POST request
        Mono<String> response = webClient.post()
                .uri(requestUri)
                .contentType(MediaType.APPLICATION_JSON)
                .bodyValue(jsonString)
                .retrieve()
                .bodyToMono(String.class);

        return response;

        // Process the response
        // response.subscribe(result -> {
        //     // Process the result
        // }, error -> {
        //     // Handle error
        // });
    }
}
```

## <span style="color:#802548">_만 나이 계산_</span>
- 옛날 기준 만나이 계산
```java
public static int getAge(int birthYear, int birthMonth, int birthDay){
    Calendar current = Calendar.getInstance();

    int currentYear  = current.get(Calendar.YEAR);
    int currentMonth = current.get(Calendar.MONTH) + 1;
    int currentDay   = current.get(Calendar.DAY_OF_MONTH);
    
    System.out.println("현재 년 : "+currentYear);
    System.out.println("현재 월 : "+currentMonth);
    System.out.println("현재 일 : "+currentDay);
    
    // 만 나이 구하기 2022-1995=27 (현재년-태어난년)
    int age = currentYear - birthYear;
    // 만약 생일이 지나지 않았으면 -1
    if (birthMonth * 100 + birthDay > currentMonth * 100 + currentDay) 
        age--;
    // 5월 26일 생은 526
    // 현재날짜 5월 25일은 525
    // 두 수를 비교 했을 때 생일이 더 클 경우 생일이 지나지 않은 것이다.
    return age;
}
```

- 최신 기준 만나이 계산
```java
public class Main {
    public static void main(String[] args) {
    LocalDate birthDate = LocalDate.of(1995, 02, 14); // Birth date: April 5, 1996
    LocalDate currentDate = LocalDate.of(2024, 2, 7); // Current date: February 5, 2024

    int age = Period.between(birthDate, currentDate).getYears();

    System.out.println(age);
    }
}
```

## <span style="color:#802548">_enum의 constant_</span>
- enum constant는 subclass이므로 field를 protected로 줘야 한다.
```java
public enum Trash {
    BUS(100){
        @Override
        int fare(int distance) {
            return distance * BASIC_FARE;
        }
    }, 
    
    AIRPLANE(200) {
        @Override
        int fare(int distance) {
            return distance * BASIC_FARE;
        }
    };

    protected final int BASIC_FARE;

    private Trash(int BASIC_FARE){
        this.BASIC_FARE = BASIC_FARE:
    }

    abstract int fare(int distance);
}
```

- 아니면 private field를 주고 public getter로 가져올 수도 있다.
```java
public enum Trash {
    BUS(100){
        @Override
        int fare(int distance) {
            return distance * getBASICFARE();
        }
    }, 
    
    AIRPLANE(200) {
        @Override
        int fare(int distance) {
            return distance * getBASICFARE();
        }
    };

    private final int BASIC_FARE;

    private Trash(int BASIC_FARE){
        this.BASIC_FARE = BASIC_FARE:
    }

    abstract int fare(int distance);

    public int getBASICFARE() {
        return BASIC_FARE;
    }
}
```

## <span style="color:#802548">_if문 refactoring_</span>
- 아래 방식은 코드 반복은 줄어드나, 읽기에 매우 안 좋은 방식이니 비추천.
```java
String deviceOS = dao.selectDevcOS();
int appVersion = dao.selectAppVersion();
boolean isOpenDialog = Y;
if("A".equals(deviceOS) && 310 > appVersion || "I".equals(deviceOS) && 305 > appVersion) {
     boolean isOpenDialog = N;
} else if ("A".equals(deviceOS) && 310 <= appVersion || "I".equals(deviceOS) && 305 <= appVersion) {
        inputMap.put("key", 125);
        inputMap.put("key1", 125);
        inputMap.put("key2", 125);
        inputMap.put("key3", 125);
        inputMap.put("key4", 125);
        inputMap.put("key5", 125);
        inputMap.put("key6", 125);
        inputMap.put("key7", 125);
        inputMap.put("key8", 125);
        inputMap.put("key9", 125);
        inputMap.put("key10", 125);
}
```

- project1에서 nested if를 쓰긴 했지만, named condition statement가 아니면 차라리 깔끔하기라도 한 nested if가 낫다.
```java
boolean isOpenDialog = Y;
if("A".equals(deviceOS)) {
    if( 310 > appVersion) {
        boolean isOpenDialog = N;
    } else {
       inputMap.put("key", 125);
        inputMap.put("key1", 125);
        inputMap.put("key2", 125);
        inputMap.put("key3", 125);
        inputMap.put("key4", 125);
        inputMap.put("key5", 125);
        inputMap.put("key6", 125);
        inputMap.put("key7", 125);
        inputMap.put("key8", 125);
        inputMap.put("key9", 125);
        inputMap.put("key10", 125);
    }
} else {
    if(305 > appVersion) {
       boolean isOpenDialog = N;
    } else {
      inputMap.put("key", 125);
        inputMap.put("key1", 125);
        inputMap.put("key2", 125);
        inputMap.put("key3", 125);
        inputMap.put("key4", 125);
        inputMap.put("key5", 125);
        inputMap.put("key6", 125);
        inputMap.put("key7", 125);
        inputMap.put("key8", 125);
        inputMap.put("key9", 125);
        inputMap.put("key10", 125);
    }
}
```

- 가장 좋은 방식은 not nested naemd condition statement다.
- 기본적으로 nested if는 피하는 게 좋다.
```java
String deviceOS = dao.selectDevcOS();
int appVersion = dao.selectAppVersion();
boolean isOpenDialog = Y;
boolean isAOS = "A".equals(deviceOS);
boolean isNewerAOSVersion = 310 > appVersion;
boolean isIOS = "I".equals(deviceOS);
boolean isNewerIOSVersion = 305 > appVersion;
if((isAOS && isNewerAOSVersion) || (isIOS && isNewerIOSVersion)) {
     boolean isOpenDialog = N;
} else if ((isAOS && !isNewerAOSVersion) || (isIOS && !isNewerIOSVersion )) {
        inputMap.put("key", 125);
        inputMap.put("key1", 125);
        inputMap.put("key2", 125);
        inputMap.put("key3", 125);
        inputMap.put("key4", 125);
        inputMap.put("key5", 125);
        inputMap.put("key6", 125);
        inputMap.put("key7", 125);
        inputMap.put("key8", 125);
        inputMap.put("key9", 125);
        inputMap.put("key10", 125);
}
```

- if문에 관해서 한번 살펴보고 가자.
- 
```
A && B 의 else = !A || !B
A || B의 else = !A && !B

A && B && C의 else = !A || !B || !C
A && B || C의 else = !A || !B && !C
A || B && C의 else = !A && !B || !C
A || B || C의 else = !A && !B && !C

(A && B) && C의 else = !(A && B) || !C = !A || !B || !C
A && (B && C)의 else = !A || !(B && C) = !A || !B || !C 
(A && B) || C의 else = !(A && B) && !C = !A || !B && !C
A && (B || C)의 else = !A || !(B || C) = !A || !B && !C
(A || B) || C의 else = !(A || B) && !C = !A && !B && !C
```

- 실제로는 A나 B는 하나의 조건이 아니라 여러 조건의 합으로 이뤄져 있다.
- 아래가 실제 예제다.
```java
if("A".equals(deviceOS) && 310 > appVersion || "I".equals(deviceOS) && 305 > appVersion) {
    
} else {

}

// 위의 else문은 아래 else if문과 동일하다.
if("A".equals(deviceOS) && 310 > appVersion || "I".equals(deviceOS) && 305 > appVersion) {
    
} else if (!("A".equals(deviceOS) && 310 > appVersion) && !("I".equals(deviceOS) && 305 > appVersion)) {

}

if("A".equals(deviceOS) && 310 > appVersion || "I".equals(deviceOS) && 305 > appVersion) {
    
} else if ((!"A".equals(deviceOS) || 310 <= appVersion) && (!"I".equals(deviceOS) || 305 <= appVersion)) {

}
```

- 아래와 같이 괄호를 다 지워주면 완전히 다른 로직으로 작동하게 된다.
- 따라서 if문을 쓸 때는 괄호를 잘 쳐주는 게 중요하다.
```java
if((!"A".equals(deviceOS) || 310 <= appVersion) && (!"I".equals(deviceOS) || 305 <= appVersion))

if(!"A".equals(deviceOS) || 310 <= appVersion && !"I".equals(deviceOS) || 305 <= appVersion)
```

- 이젠 실제 코드로 살펴보자.
```java
String devcOS = session.getAttribute("devcOS");
int appVersion = session.getAttribute("version");
String versionCheck = "1";
String registered = "";
if ("A".equals(devcOS) ) {
    if(305 > appVersion) {
        isOpenDialog = "N";
    } else {
        if (312 <= appVersion) {
            inputMap.put("name", name);
            outputMap1 = Service1.service(inputMap);
            outputMap2 = Service2.service(inputMap);

            String responseCode = String.valueOf(outputMap1.get("code"));
            String value1       = String.valueOf(outputMap2.get("value1"));
            String value2       = String.valueOf(outputMap2.get("value2"));
            String value3       = String.valueOf(outputMap2.get("value3"));
            versionCheck = "2";

            if("3".equals(value3)) {
                isOpenDialog = "N";
            }

            if("Y".equals(registered) && ("99".equals(responseCode) || "0".eqauls(value1))) {
                if("1".eqauls(value2)) {
                    isOpenDialog = "N";
                } else {
                    isOpenDialog = "Y";
                }
            }
        }
    }
} else {
    if (303 > appVersion) {
        isOpenDialog = "N";
    } else {
        if (309 <= appVersion) {
            inputMap.put("name", name);
            outputMap1 = Service1.service(inputMap);
            outputMap2 = Service2.service(inputMap);

            String responseCode = String.valueOf(outputMap1.get("code"));
            String value1       = String.valueOf(outputMap2.get("value1"));
            String value2       = String.valueOf(outputMap2.get("value2"));
            String value3       = String.valueOf(outputMap2.get("value3"));
            versionCheck = "2";

            if("3".equals(value3)) {
                isOpenDialog = "N";
            }

            if("Y".equals(registered) && ("99".equals(responseCode) || "0".eqauls(value1))) {
                if("1".eqauls(value2)) {
                    isOpenDialog = "N";
                } else {
                    isOpenDialog = "Y";
                }
            }
        }
    }
}
```

- 위의 중첩 if를 보면 정신이 나갈수밖에 없다.
- 고쳐보자. 먼저 if문 303 >appVersion의 else문은 303 <=appVersion이다.
- 따라서 else문의 시작에 존재하는 309 <=appVersion은 있으나마나 한 if문이다. else문의 조건자체가 303 <= appVersion이기 때문이다.
- 즉 304, 305, 306, 307, 308은 애초에 못들어온다. 따라서 지워주자.


```java
String devcOS = session.getAttribute("devcOS");
int appVersion = session.getAttribute("version");
String versionCheck = "1";
String registered = "";
if ("A".equals(devcOS) ) {
    if(305 > appVersion) {
        isOpenDialog = "N";
    } else {
        inputMap.put("name", name);
        outputMap1 = Service1.service(inputMap);
        outputMap2 = Service2.service(inputMap);

        String responseCode = String.valueOf(outputMap1.get("code"));
        String value1       = String.valueOf(outputMap2.get("value1"));
        String value2       = String.valueOf(outputMap2.get("value2"));
        String value3       = String.valueOf(outputMap2.get("value3"));
        versionCheck = "2";

        if("3".equals(value3)) {
            isOpenDialog = "N";
        }

        if("Y".equals(registered) && ("99".equals(responseCode) || "0".eqauls(value1))) {
            if("1".eqauls(value2)) {
                isOpenDialog = "N";
            } else {
                isOpenDialog = "Y";
            }
        }
    }
} else {
    if (303 > appVersion) {
        isOpenDialog = "N";
    } else {
        inputMap.put("name", name);
        outputMap1 = Service1.service(inputMap);
        outputMap2 = Service2.service(inputMap);

        String responseCode = String.valueOf(outputMap1.get("code"));
        String value1       = String.valueOf(outputMap2.get("value1"));
        String value2       = String.valueOf(outputMap2.get("value2"));
        String value3       = String.valueOf(outputMap2.get("value3"));
        versionCheck = "2";

        if("3".equals(value3)) {
            isOpenDialog = "N";
        }

        if("Y".equals(registered) && ("99".equals(responseCode) || "0".eqauls(value1))) {
            if("1".eqauls(value2)) {
                isOpenDialog = "N";
            } else {
                isOpenDialog = "Y";
            }
        }
    }
}
```


- 이제는 조건문을 통합해주자.
- 가만 보면, 중첩 if문의 else문들은 전부 똑같은 내용들이다.
- 조건문만 ||로 통합하면 내용을 겹쳐 쓸 수 있다.
```java
String devcOS = session.getAttribute("devcOS");
int appVersion = session.getAttribute("version");
String versionCheck = "1";
String registered = "";
if( ("A".eqauls(devcOS) && 304 > appVersion) || ("I".eqauls(devcOS) && 303 > appVersion) ) {
    isOpenDialog = "N";
} else if( ("A".eqauls(devcOS) && 303 <= appVersion) || ("I".equals(devcOS) && 303 <= appVersion) ) {
    inputMap.put("name", name);
    outputMap1 = Service1.service(inputMap);
    outputMap2 = Service2.service(inputMap);

    String responseCode = String.valueOf(outputMap1.get("code"));
    String value1       = String.valueOf(outputMap2.get("value1"));
    String value2       = String.valueOf(outputMap2.get("value2"));
    String value3       = String.valueOf(outputMap2.get("value3"));
    versionCheck = "2";

    if("3".equals(value3)) {
        isOpenDialog = "N";
    }

    if("Y".equals(registered) && ("99".equals(responseCode) || "0".eqauls(value1))) {
        if("1".eqauls(value2)) {
            isOpenDialog = "N";
        } else {
            isOpenDialog = "Y";
        }
    }
}
```

- 하지만 위의 조건식은 알아보기 너무 어렵다. 
- 조건식에 이름을 주자.
```java
String devcOS = session.getAttribute("devcOS");
int appVersion = session.getAttribute("version");
String versionCheck = "1";
String registered = "";
boolean isAOS = "A".eqauls(devcOS);
boolean isIOS = "I".eqauls(devcOS);
if( (isAOS && 304 > appVersion) || (isIOS && 303 > appVersion) ) {
    isOpenDialog = "N";
} else if( (isAOS && 303 <= appVersion) || (isIOS && 303 <= appVersion) ) {
    inputMap.put("name", name);
    outputMap1 = Service1.service(inputMap);
    outputMap2 = Service2.service(inputMap);

    String responseCode = String.valueOf(outputMap1.get("code"));
    String value1       = String.valueOf(outputMap2.get("value1"));
    String value2       = String.valueOf(outputMap2.get("value2"));
    String value3       = String.valueOf(outputMap2.get("value3"));
    versionCheck = "2";

    if("3".equals(value3)) {
        isOpenDialog = "N";
    }

    if("Y".equals(registered) && ("99".equals(responseCode) || "0".eqauls(value1))) {
        if("1".eqauls(value2)) {
            isOpenDialog = "N";
        } else {
            isOpenDialog = "Y";
        }
    }
}
```

- 그런데 value1, value2, value3는 해당 로직에서 딱 1번만 쓰인다.
- 거기다 else if안에서 어차피 서비스가 실행되기 때문에, 값을 가지고 연산하는 로직은 밖에서 해도 된다.
- 아래와 같이 따로 빼서 사용할 수 있다.
- 처음에 비하면 엄청나게 깨끗해진 로직이다!
```java
String devcOS = session.getAttribute("devcOS");
int appVersion = session.getAttribute("version");
String versionCheck = "1";
String registered = "";
boolean isAOS = "A".eqauls(devcOS);
boolean isIOS = "I".eqauls(devcOS);
if( (isAOS && 304 > appVersion) || (isIOS && 303 > appVersion) ) {
    isOpenDialog = "N";
} else if( (isAOS && 303 <= appVersion) || (isIOS && 303 <= appVersion) ) {
    inputMap.put("name", name);
    outputMap1 = Service1.service(inputMap);
    outputMap2 = Service2.service(inputMap);
    versionCheck = "2";
}

String responseCode = String.valueOf(outputMap1.get("code"));
String value1       = String.valueOf(outputMap2.get("value1"));
String value2       = String.valueOf(outputMap2.get("value2"));
String value3       = String.valueOf(outputMap2.get("value3"));

if("3".equals(value3)) {
    isOpenDialog = "N";
}

if("Y".equals(registered) && ("99".equals(responseCode) || "0".eqauls(value1)) ) {
    if("1".eqauls(value2)) {
        isOpenDialog = "N";
    } else {
        isOpenDialog = "Y";
    }
}
```

## <span style="color:#802548">_파일을 하나만 등록하고 여러곳에서 사용하는 query_</span>
- 등록해 둔 배너 이미지, 내용을 3개의 다른 곳에서 사용할 수 있게 조회하는 쿼리
```sql
select RN as BNNR_DS,
        BNNR_ST_DTM,
        BNNR_ED_DTM
        , CASE WHEN RN = 1
            THEN '로그인 배너'
            WHEN RN = 2 
            THEN '메인 배너'
            WHEN RN = 3
            THEN '전체 메뉴 배너'
            END AS BNNR_DS_NM,
        BNNM,
        CASE WHEN RN = 1
            THEN  BNNR1_IMG_URL
            WHEN RN = 2
            THEN BNNR2_IMG_URL
            WHEN RN = 3
            THEN BNNR3_IMG_URL
            END as BNNR_IMG_URL,
        CASE WHEN RN = 1
            THEN BNNR_CNTN1
            WHEN RN = 2
            THEN BNNR_CNTN2
            WHEN RN = 3
            THEN BNNR_CNTN3
            END as BNNR_CNTN,
        LK_URL, TYPE, APP_DSC
        FROM BNNR_INF
        inner join(select rownum RN from dual connect by rownum <= 3)
        on to_char(sysdate, 'YYYYMMDDHH24MISS') BETWEEN BNNR_ST_DTM AND BNNR_ED_DTM
```


## <span style="color:#802548">_날짜는 char가 아닌 date type으로_</span>
- 사실 날짜는 db에 char 타입으로 저장하는 게 매우 좋지 않다.
- 오늘 날짜와 비교하려는 경우, to_char와 같이 불필요한 function이 들어간다.
- date type이 char type보다 index를 활용할 때 더 optimizer가 잘 최적화해준다.
- date type의 날짜를 활용한다면, 아래와 같이 불필요한 to_char를 call하지 않아도 된다.
```java
select RN as BNNR_DS,
        BNNR_ST_DTM,
        BNNR_ED_DTM
        , CASE WHEN RN = 1
            THEN '로그인 배너'
            WHEN RN = 2 
            THEN '메인 배너'
            WHEN RN = 3
            THEN '전체 메뉴 배너'
            END AS BNNR_DS_NM,
        BNNM,
        CASE WHEN RN = 1
            THEN  BNNR1_IMG_URL
            WHEN RN = 2
            THEN BNNR2_IMG_URL
            WHEN RN = 3
            THEN BNNR3_IMG_URL
            END as BNNR_IMG_URL,
        CASE WHEN RN = 1
            THEN BNNR_CNTN1
            WHEN RN = 2
            THEN BNNR_CNTN2
            WHEN RN = 3
            THEN BNNR_CNTN3
            END as BNNR_CNTN,
        LK_URL, TYPE, APP_DSC
        FROM BNNR_INF
        inner join(select rownum RN from dual connect by rownum <= 3)
        on sysdate BETWEEN BNNR_ST_DTM AND BNNR_ED_DTM
```

## <span style="color:#802548">_특정 값 먼저노출되게 정렬하는 query_</span>
- 자기 회사 우선하여 가져오기(여기선 project1)
```sql
SELECT UP_C_ID,
        COMN_C_ID,
        COMN_CNM,
        RMK_CNTN,
        RMK_CNTN2,
FROM (
    SELECT UP_C_ID,
            COMN_C_ID,
            COMN_CNM,
            RMK_CNTN1,
            RMK_CNTN2,
            CASE COMN_C_ID
            WHEN 'B1_011' /*농좆*/
            THEN RMK_CNTN2 || '01'
            WHEN 'B1_012'
            THEN RMK_CNTN2 || '02'
            WHEN 'B2_012'
            THEN RMK_CNTN2 || '03'
            ELSE RMK_CNTN2 || '99'
            END AS ORDER_NO
    FROM O_COMMON_CODE
    WHERE UP_C_ID = 13
    AND COMN_C_UYN = 'Y'
) A
ORDER BY ORDER_NO, COMN_CNM, COMN_C_ID, RMK_CNTN2
```

- 위에는 굳이 subquery를 썼는데, 사실 subquery를 쓸 필요가 별로 없다.
- 아래와 같이 그냥 select 문에서 CASE WHEN을 써도 된다.
```sql
SELECT UP_C_ID,
       COMN_C_ID,
       COMN_CNM,
       RMK_CNTN1,
       RMK_CNTN2,
       CASE COMN_C_ID
           WHEN 'B1_011' THEN RMK_CNTN2 || '01'
           WHEN 'B1_012' THEN RMK_CNTN2 || '02'
           WHEN 'B2_012' THEN RMK_CNTN2 || '03'
           ELSE RMK_CNTN2 || '99'
       END AS ORDER_NO
FROM O_COMMON_CODE
WHERE UP_C_ID = 13
  AND COMN_C_UYN = 'Y'
ORDER BY ORDER_NO, COMN_CNM, COMN_C_ID, RMK_CNTN2;
```

## <span style="color:#802548">_tomcat bio과 nio_</span>
- tomcat connector는 nio가 더 좋다.
- bio는 예전 시대의 산물이다.
- nio를 써야 동접자(CCU)가 많아도 처리가 가능하다. 반응성도 높아진다. 다만 비동기인지라 예측하지 못하는 상황도 생긴다.
- 다만 bio를 여전히 많이 쓰는데 그 이유는 그냥 그렇게 써왔기 때문이며, 비동기 io는 여러가지 추가로 학습이 요구되기 때문이다.
- 가령 inputStream,OutputStream class는 Blocking I/O에서 쓰인다.
- nio 즉 비동기 방식으로는 Selector and Channel class을 써야 한다. ByteBuffer, and CompletionHandler도 있다.
- 대부분의 Spring WebFlux 방식은 nio connector와 써야 한다. bio랑 쓰면 예기한 만큼의 효과를 얻지 못한다.
- 특히 streaming APIs, and server-sent events (SSE) 같이 webFlux module을 사용할 때는 nio connector를 써야 한다.
- nio와 bio의 선택은 자바의 멀티스레딩과는 관련이 없다. 그냥 thread를 다루는 방식이다.
- bio는 사람 한명 당 한 스레드를 부여한다. nio는 여러명이 한 스레드를 공유한다.
- 쓰레드가 1개의 stack을 가지는데, 여러명이 한 스레드를 공유하게 되면 문제가 생기지 않을까 생각할 수 있다.
- 그런데 nio는 그러한 문제가 없는데, 그 이유는 connection마다 method call과 관련된 정보를 저장하는게 stack이 아니기 때문이다.
- connection과 관련된 다른 곳에 method call 정보를 저장해둔다. 전통적 stack 개념을 생각해선 안 된다.
- Jetty, 톰캣은 bio, nio 둘다 있는데 보통 요새는 거의 nio이며, Netty or Undertow는 nio방식만 있다.

- Selector class의 예시다.
```java
public class NioServerExample {

    public static void main(String[] args) throws IOException {
        // Create a Selector
        Selector selector = Selector.open();

        // Create a ServerSocketChannel
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
        serverSocketChannel.bind(new InetSocketAddress("localhost", 8080));
        serverSocketChannel.configureBlocking(false);

        // Register ServerSocketChannel with Selector for Accept operation
        serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);

        while (true) {
            // Select ready channels
            int readyChannels = selector.select();

            // Get selected keys
            Set<SelectionKey> selectedKeys = selector.selectedKeys();
            Iterator<SelectionKey> keyIterator = selectedKeys.iterator();

            // Iterate over selected keys
            while (keyIterator.hasNext()) {
                SelectionKey key = keyIterator.next();

                // Check if key is ready for Accept operation
                if (key.isAcceptable()) {
                    ServerSocketChannel serverChannel = (ServerSocketChannel) key.channel();

                    // Accept the connection
                    SocketChannel clientChannel = serverChannel.accept();
                    clientChannel.configureBlocking(false);

                    // Register clientChannel with Selector for Read operation
                    clientChannel.register(selector, SelectionKey.OP_READ);
                } 
                // Check if key is ready for Read operation
                else if (key.isReadable()) {
                    SocketChannel clientChannel = (SocketChannel) key.channel();
                    ByteBuffer buffer = ByteBuffer.allocate(1024);

                    // Read data from client
                    int bytesRead = clientChannel.read(buffer);
                    if (bytesRead == -1) {
                        // Client closed connection
                        clientChannel.close();
                    } else if (bytesRead > 0) {
                        buffer.flip();
                        byte[] data = new byte[buffer.limit()];
                        buffer.get(data);
                        System.out.println("Received: " + new String(data));
                    }
                }

                // Remove the key from the selected key set
                keyIterator.remove();
            }
        }
    }
}
```

- 위의 비동기를 아래와 같이 동기로 바꿀 수 있다.
- inpustream을 사용한 예시다.
```java
public class BioServerExample {

    public static void main(String[] args) throws IOException {
        ServerSocket serverSocket = new ServerSocket(8080);

        while (true) {
            // Accept incoming connections
            Socket clientSocket = serverSocket.accept();
            System.out.println("Client connected: " + clientSocket.getInetAddress());

            // Create input and output streams for the client socket
            InputStream inputStream = clientSocket.getInputStream();
            OutputStream outputStream = clientSocket.getOutputStream();

            // Read data from client
            byte[] buffer = new byte[1024];
            int bytesRead;
            StringBuilder receivedData = new StringBuilder();
            while ((bytesRead = inputStream.read(buffer)) != -1) {
                receivedData.append(new String(buffer, 0, bytesRead));
            }

            // Print received data
            System.out.println("Received: " + receivedData.toString());

            // Echo back the received data to the client
            outputStream.write(receivedData.toString().getBytes());

            // Close streams and socket
            inputStream.close();
            outputStream.close();
            clientSocket.close();
        }
    }
}
```


## <span style="color:#802548">_파일 비동기로 다루기_</span>
- 위의 예시는 network에 관한 예시다.
- file은 사실 Java에서는 대부분 BIO방식을 쓴다.
- 하지만 비동기 IO도 가능하다.
- Java의 nio.file은 아쉽게도 비동기 IO는 아니다.
- 우선 동기 방식의 코드부터 보자. 매우 익숙할 것이다.
```java
public class FileIOExample {

    public static void main(String[] args) {
        // Path to the file
        Path filePath = Paths.get("example.txt");

        // Write data to the file
        try (OutputStream outputStream = Files.newOutputStream(filePath)) {
            String data = "Hello, world!";
            byte[] bytes = data.getBytes();
            outputStream.write(bytes);
        } catch (IOException e) {
            e.printStackTrace();
        }

        // Read data from the file
        try (InputStream inputStream = Files.newInputStream(filePath)) {
            byte[] buffer = new byte[1024];
            int bytesRead = inputStream.read(buffer);
            if (bytesRead != -1) {
                String data = new String(buffer, 0, bytesRead);
                System.out.println("Read from file: " + data);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

- 다음으로는 비동기 방식이다.
```java
public class AsyncFileIOExample {

    public static void main(String[] args) throws IOException, InterruptedException, ExecutionException {
        Path filePath = Paths.get("example.txt");

        // Open file asynchronously
        AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(filePath);

        // Allocate buffer for reading data
        ByteBuffer buffer = ByteBuffer.allocate(1024);

        // Initiate read operation asynchronously
        Future<Integer> future = fileChannel.read(buffer, 0);

        // Perform other tasks while waiting for read operation to complete
        // ...

        // Wait for read operation to complete and handle result
        int bytesRead = future.get();
        if (bytesRead != -1) {
            buffer.flip();
            byte[] data = new byte[buffer.limit()];
            buffer.get(data);
            System.out.println("Read from file: " + new String(data));
        }

        // Close file channel
        fileChannel.close();
    }
}
```

- 하지만 위의 코드는 retry logic이 없다.
- 비동기인만큼 retry가 필요하다. 보면 recursive로 구현했다.
```java
public class AsyncFileUploadRetryExample {

    private static final int MAX_RETRIES = 3;
    private static final long RETRY_DELAY_MS = 1000; // 1 second

    public static void main(String[] args) throws IOException, InterruptedException, ExecutionException {
        Path filePath = Paths.get("example.txt");

        // Retry uploading the file
        boolean uploadSuccessful = uploadFile(filePath, 0);
        if (uploadSuccessful) {
            System.out.println("File upload successful!");
        } else {
            System.out.println("File upload failed after maximum retries.");
        }
    }

    private static boolean uploadFile(Path filePath, int attempt) throws IOException, InterruptedException, ExecutionException {
        if (attempt >= MAX_RETRIES) {
            return false; // Maximum retries reached
        }

        AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(filePath);

        ByteBuffer buffer = ByteBuffer.allocate(1024);

        Future<Integer> future = fileChannel.read(buffer, 0);

        try {
            int bytesRead = future.get();
            if (bytesRead != -1) {
                buffer.flip();
                byte[] data = new byte[buffer.limit()];
                buffer.get(data);
                System.out.println("Read from file: " + new String(data));
            }

            // Upload successful
            return true;
        } catch (ExecutionException e) {
            // Upload failed, retry after delay
            System.out.println("File upload attempt " + (attempt + 1) + " failed. Retrying...");
            Thread.sleep(RETRY_DELAY_MS);
            return uploadFile(filePath, attempt + 1); // Retry upload
        } finally {
            fileChannel.close();
        }
    }
}
```


- 하지만 retry를 너무 많이 하는 것도 별로다. 
- 100번의 횟수제한을 주자.
```java
public class AsyncFileUploadRetryExample {

    private static final int MAX_RETRIES = 100;
    private static final long RETRY_DELAY_MS = 1000; // 1 second

    public static void main(String[] args) throws IOException, InterruptedException, ExecutionException {
        Path filePath = Paths.get("example.txt");

        // Retry uploading the file
        boolean uploadSuccessful = uploadFile(filePath, 0);
        if (uploadSuccessful) {
            System.out.println("File upload successful!");
        } else {
            System.out.println("File upload failed after maximum retries.");
        }
    }

    private static boolean uploadFile(Path filePath, int attempt) throws IOException, InterruptedException, ExecutionException {
        if (attempt >= MAX_RETRIES) {
            return false; // Maximum retries reached
        }

        AsynchronousFileChannel fileChannel = AsynchronousFileChannel.open(filePath);

        ByteBuffer buffer = ByteBuffer.allocate(1024);

        Future<Integer> future = fileChannel.read(buffer, 0);

        try {
            int bytesRead = future.get();
            if (bytesRead != -1) {
                buffer.flip();
                byte[] data = new byte[buffer.limit()];
                buffer.get(data);
                System.out.println("Read from file: " + new String(data));
            }

            // Upload successful
            return true;
        } catch (ExecutionException e) {
            // Upload failed, retry after delay
            System.out.println("File upload attempt " + (attempt + 1) + " failed. Retrying...");
            Thread.sleep(RETRY_DELAY_MS);
            return uploadFile(filePath, attempt + 1); // Retry upload
        } finally {
            fileChannel.close();
        }
    }
}
```

## <span style="color:#802548">_다른 상위 분류 선택시 하위 분류 선택 css 초기화_</span>
- 핸드폰 팝업에서 select 같이 보이는 것들 선택했을 떄, 다른 선택 css 처리 초기화
```js
$('.popup').each(function() {
    var idVal = $(this).attr('id');

    $('.popup.show .select_item_wrap').find('li').on('click',function() {
        //결제수단을 바꾼 경우, 그 아래에 있는 모든 css 선택 처리 초기화를 의도. 그러나 의도대로 잘 되지 않는 경우가 있어음.
        $(this).childeren().addClass('selected ui-pop-close').parent('li').siblings().children().removeClass('selected ui-pop-close');

        //따라서 아래와 같이 추가.
        var txt = $(this).find('strong').html();
        var selectedDomArray = ['#crdReasonPop', '#actReasonPop', '#cashReasonPop'];
       
        if (idVal == "wayPop") {
            if (txt == "카드") {
                selectedDomArray = selectedDomArray.filter(function(item) {
                    return item !== '#crdReasonPop';
                })
            } else if (txt == "계좌") {
                selectedDomArray = selectedDomArray.filter(function(item) {
                    return item !== '#actReasonPop';
                })
            } else if (txt == "현금") {
                selectedDomArray = selectedDomArray.filter(function(item) {
                    return item !== '#cashReasonPop';
                })
            }
        }
        //만약 결제수단을 변경하면 분류 선택 css를 초기화함.  다만, 자기 자신을 누른 경우는 제외해야 하므로 위에서 filtering 한것.
        //그게 selectedDomArray.toString()
        $('.btn-select').parents().find('#container').siblings(selectedDomArray.toString()).find('ul').children().each(function() {
            $(this).find('div').removeClass('selected ui-pop-close');
        })
    })
})
```

## <span style="color:#802548">_서로 다른 내용 합치는 query refactoring_</span>
- 아래 서브쿼리는 서브쿼리가 아주 깊다.
- 안내
```sql
SELECT
    DECODE(new_ds, 1, '안내', 2, '혜택') AS new_nm,
    sub.*
FROM
    (
        SELECT
            '1' AS new_ds,
            bbrd_sqno AS new_id,
            CASE
                WHEN (TO_CHAR(SYSDATE, 'yyyymmdd') - TO_CHAR(TO_DATE(rg_dtm, 'yyyymmddhh24miss'), 'yyyymmdd')) <= 7 THEN 'y'
                ELSE 'n'
            END AS new_yn
        FROM
            (
                SELECT
                    rg_dtm,
                    bbrd_sqno
                FROM
                    board
                WHERE
                    bltn_yn = 'y'
                    AND TO_CHAR(SYSDATE, 'yyyymmddhh24miss') <= ed_dtm
                ORDER BY
                    rg_dtm DESC NULLS LAST
            )
        WHERE
            ROWNUM <= 1
        UNION ALL
        SELECT
            '2' AS new_ds,
            bbrd_sqno AS new_id,
            CASE
                WHEN (TO_CHAR(SYSDATE, 'yyyymmdd') - TO_CHAR(TO_DATE(req_dtm, 'yyyymmddhh24miss'), 'yyyymmdd')) <= 7 THEN 'y'
                ELSE 'n'
            END AS new_yn
        FROM
            (
                SELECT
                    req_dtm
                FROM
                    push
                WHERE
                    cusno = '10'
                    AND del_yn = 'n'
                ORDER BY
                    req_dtm DESC NULLS LAST
            )
        WHERE
            ROWNUM <= 1
    ) sub;
```

- 위의 서브쿼리를 아래와 같이 window function을 이용해 단순하게 만들자.
- 그리고 WITH까지 사용하면 매우 깔끔해진다.
```java
WITH board_cte AS (
    SELECT
        bbrd_sqno AS new_id,
        TO_DATE(rg_dtm, 'yyyymmddhh24miss') AS rg_date,
        ROW_NUMBER() OVER (ORDER BY rg_date DESC NULLS LAST) AS rn_board
    FROM
        board
    WHERE
        bltn_yn = 'y'
        AND TO_CHAR(SYSDATE, 'yyyymmddhh24miss') <= ed_dtm
),
push_cte AS (
    SELECT
        bbrd_sqno AS new_id,
        TO_DATE(req_dtm, 'yyyymmddhh24miss') AS req_date,
        ROW_NUMBER() OVER (ORDER BY req_date DESC NULLS LAST) AS rn_push
    FROM
        push
    WHERE
        cusno = '10'
        AND del_yn = 'n'
)
SELECT
    DECODE(new_ds, 1, '안내', 2, '혜택') AS new_nm,
    sub.*
FROM
    (
        SELECT '1' AS new_ds, new_id, CASE WHEN rg_date >= SYSDATE - 7 THEN 'y' ELSE 'n' END AS new_yn
        FROM board_cte WHERE rn_board = 1

        UNION ALL

        SELECT '2' AS new_ds, new_id, CASE WHEN req_date >= SYSDATE - 7 THEN 'y' ELSE 'n' END AS new_yn
        FROM push_cte WHERE rn_push = 1
    ) sub;
```

## <span style="color:#802548">_효율적인 Java method 사용_</span>
- 그저 해당 문자열이 있는지만 비교할 거라면, substring.eqauls를 쓸 필요가 없다.
- indexOf로 가져오면 된다.
```java
string.substring().equals() //substring creation overhead
string.indexOf() >= 1       //better
```



## <span style="color:#802548">_js literal map 활용하기_</span>
- if문으로 전개하면 아래와 같은 형태일 것이다.
```js
if (payload.responseCode =='0') {
    //logic
    openFailLayer();
} else if(payload.responseCode == '1') {
    //logic
    location.href = '/성공';
} else {
    //logic
    openFailLayer();
}
```

- magic number를 피해주자.
```js
const IDENTITY_NOT_AUTHENTICATED = '0';
const IDENTITY_AUTHENTICATED = '1';
const IDENTITY_NOT_OWNED = '2';
if (payload.responseCode == IDENTITY_NOT_AUTHENTICATED) {
    //logic
    openFailLayer();
} else if (payload.responseCode == IDENTITY_AUTHENTICATED) {
    //logic
    location.href = '/성공';
} else if (payload.responseCode == IDENTITY_NOT_OWNED) {
    //logic
    openFailLayer();
}
```

- if문도 좋지만, 아래와 같이 객체에 넣어서 좀 더 잘게 쪼개는 것도 가능하다.
```js
const IDENTITY_NOT_AUTHENTICATED = '0';
const IDENTITY_AUTHENTICATED = '1';
const IDENTITY_NOT_OWNED = '2';
const authenticationMap = {
    IDENTITY_NOT_AUTHENTICATED: function() {
        //logic
        return false;
    },
    IDENTITY_AUTHENTICATED: function() {
        //logic
        return true;
    },
    IDENTITY_NOT_OWNED: function() {
        //logic
        return false;
    }
}

function executePostAuthentication(resultCode) {
    authenticationMap[resultCode]();
}

function authAjax() {
    $.ajax({

    }),
    success: function(res,status,xhr) {
        var payload = JSON.parse(res.payload);

        if (executePostAuthentication(payload.resultCode) == false) {
            openFailLayer();
        }
    }
}
```



## <span style="color:#802548">_controller가 아닌 service에서 refacotring_</span>
- 가입 조건이 바뀌어 로직을 추가해야 했다.
- 그러나 갑사의 convention에 묶여 controller단에서는 중첩 if가 강제였다.
- finally에서 encPayload를 해서 return을 해야했기 때문이다. early return이 불가능했다.
```java
HashMap<String, Object> checkPasswordMap = new HashMap<>();
if( "2".equals(dscCode) ) {
    checkPasswordMap = Service.checkPassword(payloadMap);
    String responseCode = String.valueOf(checkPasswordMap.get("RT_CD"));
    if( "0000".equals(responseCode) ) {
        outputMap = Service.requestCard(inputMap);
    } else {
        outputMap.putAll(checkPasswordMap);
    }
} else {
    outputMap = Service.requestCard(inputMap);
}
```

- convention을 준수하면서 소스코드를 좀 더 깔끔하게 쓰려면 Service단으로 분기 로직을 옮겨야 했다.
- 그럼 early return을 활용해 좀 더 깔끔하게 코드를 쓸 수 있다.
```java
HashMap<String, Object> retMap = new HashMap<>();
String responseCode = "";
if( "1".equals(dscCode) ) {
    retMap = IF_A();
    
    return retMap
} else if( "2".equals(dscCode)) {
    retMap = Service.checkPassword(inputMap);
    responseCode = String.valueOf(retMap.get("RT_CD"));
}

if( "0000".equals(responseCode)) {
    retMap = IF_A();
}

return retMap;
```

## <span style="color:#802548">_closure를 사용해서 중복클릭 방지하기_</span>
- 과거에 내가 만든 중복클릭 방지는 전역변수, setTimeout을 이용했었다.
```js
var reqFlag = false;

function reqCheck(){
    if(reqCheck == false){
            reqFlag = true;
            return false;
    }

    setTimeout(function(){
            reqFlag = false;
    },3000)

    return true;
}

function handleButtonClick() {
    $('#button').on('click',function() {
        if(reqCheck()) {
            return;
        }
        .
        .
        .
        //ajax 함수
    })
}
```

- 저번에도 closure로 바꿔보고 싶었지만 실패했었다.
- 그래서 이번에 한번 바꿔봤다.
- 처음엔 아래와 같이 만들었다.
- 그런데 문제는 의도한대로 되지 않는다는 점이었다.
- 내가 의도한 것은 함수가 결과를 받아올 때까지 버튼이 눌리지 않는 것인데, 잘 작동하지 않았다.
- 계속 clicked가 false로 처리되었다.
- var의 문제인가 했지만 그런 것은 아니었다. 즉시실행함수로 바꿔도 달라지는 점은 없었다.
```js
function sendRequest(fn) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            fn(data)
        }
    }
}

function handleButtonClick() {
    $("#button").on('click',function() {
        .
        .
        .
        var sendData = {};
        sendData.payload = id;

        sendRequest(testAjax)(sendData);
    })
}

function testAjax() {
    $.ajax({
        .
        .
        .
        success: function(res, status, xhr){
            if(res.statusCode === '200') {
                location.href = '/teenAgerCardIssue';
            }
        },
        error: function(error) {
            alert(error);
        } ,
        complete: function() {
            checkMobileDeviceOs();
        }
    })
}
```



- 그래서 원인을 알아냈는데, 그건 비동기이면서, event Listener에 달고, closure를 써서 복잡하게 꼬여있던 것이었다.
- closure의 자유변수를 활용하려면 하나의 함수 안에서 이뤄져야 했다. 
- success callback, error callback, complete callback에서 모두 clicked를 다시 false로 바꿔줘야 했다.
- 먼저 함수를 parameter로 넘기지 않고, ajax를 분리하지 않고 closure가 쓰인 함수로 옮겼다.
```js
function sendRequest() {
    var clicked = false;
    return function() {
        if(!clicked) {
            clicked = true;
            var sendData = {};
            sendData.payload = id;
            $.ajax({
                .
                .
                .
                success: function(res, status, xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                },
                error: function(error) {
                    alert(error);
                     clicked = false;
                } ,
                complete: function() {
                    checkMobileDeviceOs();
                     clicked = false;
                }
            })

        }
    }
}

function handleButtonClick() {
    $("#button").on('click',function() {
       sendRequest()();
    })
}
```

- 그럼에도 여전히 clicked가 false로 초기화됐다.
- 원인을 알아보니 EventLisnter가 문제는 아니었다.
- 문제는 click 시마다 sendRequest()()를 호출하게 되면 매번 새로운 실행 컨텍스트가 생기는 게 문제였다.
- 그래서 실행 컨텍스트를 저장해두는 변수를 만들었다.
- 그랬더니 이제 원하는 대로 작동하기 시작했다!
```js
function sendRequest() {
    var clicked = false;
    return function() {
        if(!clicked) {
            clicked = true;
            var sendData = {};
            sendData.payload = id;
            $.ajax({
                .
                .
                .
                success: function(res, status, xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                },
                error: function(error) {
                    alert(error);
                     clicked = false;
                } ,
                complete: function() {
                    checkMobileDeviceOs();
                     clicked = false;
                }
            })

        }
    }
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
       fetchData();
    })
}
```

- 이제 기본적인 작동은 되니 함수를 분리해주고 싶었다.
- 그래서 다시 parameter로 주는 방법을 생각해보았다.
- 우선 data부터 parameter로 넘겨주기로 했다.
- 외부함수의 parameter보단 내부함수의 parameter에서 넣는 게 더 좋다 판단하였다.
- 처음에 실행컨텍스트를 불러올 때 함수를 parameter를 받을 수 있기 때문에, 그건 함수에게 넘겨주려 생각했다.
```js
function sendRequest() {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
             $.ajax({
                .
                .
                data:data
                success: function(res, status, xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                },
                error: function(error) {
                    alert(error);
                     clicked = false;
                } ,
                complete: function() {
                    checkMobileDeviceOs();
                     clicked = false;
                }
            })

        }
    }
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
        fetchData(sendData);
    })
}
```

- 이제는 ajax 함수를 바깥으로 빼보고 싶었다.
- 그랬더니 clicked가 closure함수가 속한 범위에서 벗어나 버렸다.
- 그래서 이를 해결하기 위해 parameter로 넘겨봤지만 아무 소용이 없었다.
- 계속 clicked가 false로 초기화됐다.
```js
function sendRequest() {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            textAjax(data, clicked);

        }
    }
}

function testAjax(data, clicked) {
    $.ajax({
        .
        .
        data:data
        success: function(res, status, xhr){
            if(res.statusCode === '200') {
                location.href = '/teenAgerCardIssue';
            }
            clicked = false;
        },
        error: function(error) {
            alert(error);
            clicked = false;
        } ,
        complete: function() {
            checkMobileDeviceOs();
            clicked = false;
        }
    })
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```

- 결국 과거의 jquery ajax 활용법으론 불가능하다고 판단했다.
- Promise와 비슷하게 .then()처럼 이어가는 방식의 ajax를 사용하기로 했다.
- 이렇게 하면 ajax를 분리하면서 자유변수는 그대로 살릴 수 있었다.
```js
function sendRequest() {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            textAjax(data)
                .done(function(res,status,xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```

- 이제는 여러가지 ajax 함수를 받을 수 있게 함수를 parameter로 받으려 했다.
- ajaxFn으로 받는 것은 아래와 같이 간단하게 바꿀 수 있었다.
- 대신 처음 실행컨텍스트를 정할 때 원하는 ajax 함수를 같이 넘기게 바꿔야 한다.
```js
function sendRequest(ajaxFn) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            ajaxFn(data)
                .done(function(res,status,xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

var fetchData = sendRequest(testAjax);

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```

- 그런데 successCallback도 같이 ajax와 넣어두면 이걸 공통함수로 뺄 수 있겠다는 생각이 들었다.
- 그래서 successCallback도 같이 parameter로 받게 바꿨다.
- successCallback 함수는 res를 parameter로 받아야한다.
- 그러나 sendRequest에서 굳이 parameter를 명시하지 않아도 알아서 잘 받아진다. 이유는 모르겠다.
```js
function sendRequest(ajaxFn, successCallback) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            ajaxFn(data)
                .done(function(res,status,xhr){
                    successCallback(res)
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

function successCallback(res) {
    var popupText = "";
    var popupHtml = "";
    if(res.statusCode === '200') {
        location.href = '/teenAgerCardIssue';
    } else if (res.statusCode === '199') {
        popupText = "비밀번호가 1회 오류입니다."
    } else if (res.statusCode === '198') {
        popupText = "비밀번호가 2회 오류입니다."
    } else {
        location.href = '/teenAgerError'
        return;
    }

    poupHtml = $("#errorGuide").text(popupText);
    popupHtml.html(popupHtml.html().replace(/\n/g,'<br>'));

    Layer.open("#errorPopup");
}

var fetchData = sendRequest(testAjax, successCallback);

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```


- 모듈화를 해놨으니 다른 공통 함수로 빼놓을 수 있다.
- 공통함수 js에 넣을 수 있다는 의미다.
```js
//common.js

function sendRequestOnlyOnce(ajaxFn, successCallback) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            ajaxFn(data)
                .done(function(res,status,xhr){
                    successCallback(res)
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}
```

- 이제 ajax들은 모두 common.js에서 가져온 sendRequestOnlyOnce 함수로 wrapping할 수 있다.
- 그러면 별다른 처리 없이 간단하게 아래와 같은 순서로 하면 중복클릭방지가 저절로 들어가는 것이다.
- script import --> ajax 함수 정의 --> ajax의 successCallback 정의 --> 실행컨텍스트 저장을 위한 변수 정의 --> data collection --> eventListener에서 함수실행
- 다만 여기서 ajax에 절대로 async:false 옵션이 있으면 안 된다.
- 그럼 어떤 방식(버튼 disabled, 전역변수 flag 등..)을 쓰든 중복클릭 방지가 작동하지 않는다.
```js
//teenAger.jsp
<script src ="/my/common.js"/>



//원하는 ajax함수
function testAjax(data) {
    var leftDay = "${leftDay}";
    if (parseInt(leftDay) >= 1) {
        preventRequestAndShowPopup(parseInt(lefyDay));
        return;
    }

    return $.ajax({
                url:'~~',
                method:'post',
                dataType:'json',
                data:data
            })
}

//원하는 ajax의 successCallback
function successCallback(res) {
    .
    .
}

// 실행컨텍스트 저장을 위해 wraaping하는 함수
var fetchData = sendRequestOnlyOnce(testAjax, successCallback);


function handleButtonClick() {
    $("#button").on('click',function() {
        var isDisabled = $(this).hasClass('disabled');
        if(isDisabled) {
            return;
        }

        //data collecting
        var sendData = {};

        //실제 함수 실행
       fetchData(sendData);
    })
}
```


- 해당 내용을 axios/ts로 변환해보자.
- 나는 ajax의 형태를 아래와 같이 사용했다.
```js
const apiRootPath = process.env.VUE_APP_API;
let prevRequest: AxiosRequestConfig | null = null;
export const instance = axios.create({
  timeout: 10 * 1000,
  baseURL: apiRootPath,
});

instance.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    if (config.url !== '/login') {
      const store = useMemberStore();
      const { accessToken } = storeToRefs(store);

      config.headers['authorization'] = `Bearer ${accessToken.value}`;
    }

    prevRequest = config;
    return config;
  },
  error => {
    return Promise.reject(error);
  }
);

instance.interceptors.response.use(
  response => {
    return response;
  },
  async (error: AxiosError) => {
    if (error.response?.status === 401) {
      const store = useMemberStore();
      const { loginMember } = storeToRefs(store);
      const { updateAccessToken } = store;
      const authenticPrevRequest = prevRequest;
      const accessTokenResponse = await extendSignInStatus(loginMember.value);
      const accessToken = accessTokenResponse.data;
      updateAccessToken(accessToken);
      const updatedRequestConfig = {
        ...authenticPrevRequest,
        headers: {
          authorization: `Bearer ${accessToken}`,
          'content-type': 'application/json',
        },
      };

      const result = await axios(updatedRequestConfig as AxiosRequestConfig);
      return result;
    }

    if (error.response?.data.message) {
      alert(error.response?.data.message);
    }

    return Promise.reject(error);
  }
);
```

- 실제 ajax 함수는 아래와 같이 만들수 있다.
```js
export const login = (loginRequest: Pick<Member,'memberId' | 'memberPassword'>) => {
  return instance({
    url: '/signin',
    method: 'post',
    data: loginRequest,
  });
};
```

- 그럼 위와 같은 ajax 함수를 받아오기 위한 형태로 아래와 같이 만들어준다.
- parameter는 뭐로 들어올지 몰라 any로 해놨다. 나중에 규칙이 추가된다면 any가 아닌 다른것으로 바꿔보자.
```js
export const sendRequestOnlyOnce = (ajaxFn: (parameter: any) => AxiosPromise<any>, successCallback: (response: AxiosResponse) => void) => {
    let clicked = false;
    return (parameter: any) => {
        console.log(clicked);
        if(!clicked) {
            clicked = true;
            ajaxFn(parameter)
                .then(res => {
                    successCallback(res)
                    clicked = false;
                }).catch(function(error){
                    alert(error);
                    clicked = false;
                }).finally(function(){
                    clicked = false;
                })

        }
    }
}
```

- successCallback은 아래와 같이 적용해준다.
```js
const successCallback = (response: AxiosResponse): void => {
    if (response.status === 200) {
    const accessToken = response.data.accessToken;
    const refreshToken = response.data.refreshToken;
    const memberNo = response.data.memberNo;

    alert('로그인이 완료되었습니다.');
    changeLoginMember(memberId.value, accessToken, refreshToken, memberNo);
    router.push('/');
    } 
}
```

- 그럼 실제 closure는 아래와 같이 호출한다.
- 아직 실전 test는 안해봤는데, 개인 프로젝트 하게 되면 test를 해보려고 한다.
```js
const fetchData = sendRequestOnlyOnce(login, successCallback);

async function handleLoginClick() {
    const isValid = (await useValidateForm(form, isFormValid))?.value;
    if (isValid) {
        const data = {
            memberId: memberId.value,
            memberPassword: memberPassword.value,
        };

        fetchData(data);
    }
}
```


- 물론 아래와 같이 전역변수를 쓰는 것도 가능하다.
- async는 false로 쓰면 절대 안된다.
- async는 false로 쓰면 ajax의 success에서 return한 값을 쓰겠다는 의미다.
- 이런 식으로 쓰면 안되고, 콜백함수를 활용해줘야 한다.
```js
var dualExistsFlag = false;

function ajax(){
    if($(this).hasClass('disabled')) {
        return;
    }

    if(dualExistsFlag) {
        return;
    }
    dualExistsFlag = true;

    $.ajax({
        url:'ddd',
        method:'post',
        data:data,
        dataType:'json',
        success:function(res,status,xhr){
            .
            .
            .

        },
        error:function(){
            .
            .
            .

        },
        complete:function(){
            dualExistsFlag = false;
        }
    })
}
```

## <span style="color:#802548">_post 방식으로 페이지 이동하며 한글값 가져가기_</span>
- 개인정보 보호법을 보면, 개인정보는 URL에 노출시키는 것이 금지되어있다.
- 따라서 get 방식 query string으로 직접 주소값을 넣을 수가 없다.
- 암호화를 해야한다. 하지만 보통 암호화까지 하기는 부담스럽다.
- 그런데 개인정보를 페이지 내에 보여줘야할 때가 있다. DB에서 조회하지 않고 이전 페이지 값을 통해서 말이다.
- 카드를 발급했을 때 쓴 주소를 그 다음 페이지에 보여주는데, db에 주소 값을 저장하지는 않는 상황이라면 더더욱 그러하다.
- 그럴땐 post를 사용해서 페이지를 이동하며, body data로 개인정보를 갖고 간다.
- 이동이 시작되는 페이지에선 아래와 같이 구성한다.
- 한글의 경우, 인코딩 문제가 있을 수 있기 때문에 아래와 같이 URIComponent로 변환해서 가져간다.
```js
const data = encodeURIComponent($("#firstAddress")+ $("#secondeAddress"));
$("#address").val(data);
const form = document.form.addressForm;
form.action = '/cardIssueDone';
form.method = 'post'
form.submit();

<form id="addressForm" name="addressForm" accept-charset='utf-8'>
    <input type="hidden" id="address" name="address" value="" />
</form>
```

- URI componenet는 encoding에 따라 값이 달라지지 않아 encoding 문제가 없다.
- controller에서는 해당 값을 꺼내 돌려줄 페이지에 셋팅해준다.
```java
@Controller
public moveCompletePage(HttpServletRequest request) {
    String address = request.getParameter('address');
    mav.addObject("address", address);
}
```

- 이동한 페이지에서 아래와 같이 꺼내서 쓰면 된다.
- URIComponenet로 encoding했으니 다시 decode하는 것이다.
```js
const address = decodeURIComponent("${address}")
```

- 인터넷에서 말하는 여러가지 설정 다해봤지만 encoding 문제가 해결이 안된다.
- URIEncoding, request.setCharacterEncoding 등 전부 안돼서 위와 같이 조금 byte를 먹더라고 저렇게 처리하는 게 맘 편하다.
- web.xml은 하청업체가 건드리기도 어려우니까 애초에 선택지도 아니었다.


- 조금 복잡하게 객체로 바꿔서 보내보자.
```js
const payload = {
    address: encodeURIComponent($("#firstAddress")+ $("#secondeAddress"));
}
$("#payload").val(payload);
const form = document.form.addressForm;
form.action = '/cardIssueDone';
form.method = 'post'
form.submit();

<form id="addressForm" name="addressForm" accept-charset='utf-8'>
    <input type="hidden" id="payload" name="payload" value="" />
</form>
```

- URI componenet는 encoding에 따라 값이 달라지지 않아 encoding 문제가 없다.
- controller에서는 해당 값을 꺼내 돌려줄 페이지에 셋팅해준다.
```java
@Controller
public moveCompletePage(HttpServletRequest request) {
    HashMap<String, Object> payloadMap = (HashMap<String,Object>)request.attribute('payload');
    String address = (String) payloadMap.get("address");

    mav.addObject("address", address);
}
```


- 그런데 SPA에서는 해당 방식이 작동하지 않는다.
- SPA에서 form을 submit하면 페이지 이동을 하는게 아니라, server에 데이터를 받아오는 것이기 때문이다.
- 따라서 SPA에서는 페이지 이동을 get으로 한다 생각하고 만들어야 한다.
- 그럼 어떻게 해야할까? client단에 암/복호화 library를 사용하는 게 일단 가장 간단한 해결방법이다.
- 암호화를 해서 get 쿼리 스트링으로 보낸다.
```js
import {encrypt} from 'enc/dec';

const basicAddress = ref('');
const additionalAddress = ref('');
async function cardIssue(){
    const cardIssueResponse = await cardIssue(data);
    if(cardIssueResponse.status !== 200) {
        alert('시발 에러야..')
        return;
    }

    const encryptAddress = encrypt(basicAddress.value + additionalAddress.value);
    router.push(`/cardIssueDone?address=${encryptAddress}`)
}

<input v-model="basicAddress" />
<input v-model="additionalAddress" />
<button type="button" id="cardIssue" @click="cardIssue" />
```

- page가 load될 때 복호화를 해서 원래 값으로 보여준다.
```js
const address =ref('');
onMounted(async ()=>{
   const decryptAddress = decrypt(router.param.address);
   address.value = decryptAddress;
})

<input v-model="address" />
```

- 아니면 서버단에 post로 데이터 암호화 요청을 날려서 query string으로 붙인다.
```js
const basicAddress = ref('');
const additionalAddress = ref('');
function encryptAddress() {
    const address = basicAddress.value + additionalAddress.value;
    const result = axios.post('/encryptData', address);

    return result;
}

async function cardIssue(){
    const cardIssueResponse = await cardIssue(data);
    if(cardIssueResponse.status !== 200) {
        alert('시발 에러야..')
        return;
    }

    const addressResponse = await encryptAddress();
    const encryptedAddress = addressResponse.data.encryptedAddress;
    router.push(`/cardIssueDone?address=${encryptAddress}`)
}

<input v-model="basicAddress" />
<input v-model="additionalAddress" />
<button type="button" id="cardIssue" @click="cardIssue" />
```

- 이동한 뒤에 page를 처음 load할 때 복호화 요청을 날려서 해당 주소가 복호화된 값으로 보이게 한다.
```js
const address =ref('');
onMounted(async ()=>{
   const decryptedAddressResponse = await axios.post('/decryptAddress',router.param.address);
   address.value = decryptedAddressResponse.data.decryptAddress;
})

<input v-model="address" />
```

- post방식의 이동은 SPA의 router로는 불가능하니 시도하지 말자.

## <span style="color:#802548">_느린 네트워크에서 CSS 무너짐_</span>
- 3층 환경이 AP 기기가 줫구려서 너무 느렸다. 
- api 한번 response 받는데 6초가 걸렸다. render는 25초 걸렸다.
- 그런데 알 수 없는 이유로 자꾸 앱에서 CSS가 무너졌다.
- 로컬에서는 절대 그런일이 없었다. 그런데 자꾸 무너지니까 미칠거 같았다. 디버깅해도 문제가 보이지 않았다.
- 그러다 혹시 느려서 그런건 아닐까 하고, 개발자 도구에서 네트워크 환경을 느린 3G로 바꾸니까 실제 내가 겪는 문제처럼 나왔다.
- 그렇다. 느린 네트워크로 인해 뭔가 문제가 생긴 것이었다.
- 그래서 원인을 보니, font가 원래 지정된 font가 아니라 default font가 적용되고 있었다.
- font를 가져오는 게 가장 느렸는데, 그래서 font를 적용하지 않고 그냥 DOM을 그려버렸다.
- 그래서 원하는 CSS가 안나왔던 것이다. 실제로 가끔 빠르게 화면이 뜨면 정상적으로 입력 CSS가 한줄에 나왔는데, 그런 이유였던 것이다.
- 만약 느린 환경까지 고려한다면, 폰트를 최대한 압축하는 것도 중요하겠다는 생각이 들었다.


## <span style="color:#802548">_에러코드_</span>
- request가 서비스가 두 개를 한꺼번에 처리할 때, 성공은 같은 코드로 주어도 실패는 서로 다르게 가져가야 한다.
- 실패를 서로 같은 코드로 주게 되면 각 서비스 실패에 관한 분기 메시지 처리가 불가능해지기 때문이다.
- 다만 에러코드를 바라보는게 아니라, 에러 메시지를 바라보게 처리할 경우, 이러한 제약은 무의미해진다.
- 서버에서 에러 메시지를 관리하는 경우라면 별로 문제가 없다. 따라서 서버에서 enum으로 에러 메시지를 관리할 떄, 사용자가 이해할 만한 메시지로 써야 한다.
  


## <span style="color:#802548">_jsp include directive_</span>
- footer와 header를 jsp에서 넣을 땐 보통 jsp:include를 쓴다.
- 반면에 원 jsp와 타 jsp의 HTML 및 js 소스코드가 합쳐져야 하는 경우도 있다.
- vue로 따지면 component를 분리한 뒤 emit으로 함수를 보내는 형태다.
- 그 땐 include directive를 쓴다.

```js
<% include file="/web-inf/jsp/transfer/financialSelectPop.jsp" %>
```

- 어디서나 쓸 수 있는 공통 오픈뱅킹 팝업 render함수를 위의 financialSelectPop.jsp에 넣는다.
- 해당 jsp를 include할 pair jsp에서는 render 이후 일어질 일들을 프로세스처리에 필요한 함수를 규정한다.


- financalSelectPop.jsp는 아래와 같이 만든다.
- render함수에서 DOM의 이미지 안에 onClick 함수를 만들어놓는다.
```html
<html>
    .
    .
    .
    <a href="#bank">은행</a>
    <a href="#security">증권사</a>
    <div id="bank">
        <ul class = "bank_popup"></ul>
    </div>
    <div id="security">
        <ul class = "security_popup"></ul>
    </div>
</html>
<script>
    $.ajax({
        .
        .
        .
        success:function(res){
            const data = res
            renderPopup(data);
        }
    })

    function renderPopup(data){
        const bankHtml = [];
        const securityHtml = [];
        for(const element of data.list){
            html.push(element.bankCode);
            html.push(element.bankName);
            .
            .
            html.push('<a onClick="financialSelect(this,\"+bankCode+'\',\"+  ');
        }
    }
</script>
```

- 해당 onClick함수는 process를 처리하는 함수이므로, include directive를 사용한 jsp에서 정의한다.
- 예를 들자면 계좌등록 화면이다.
```js
function financialSelect(obj, bankCode,...) {
    //logic
}
```

## <span style="color:#802548">_실행계획과 index 조회_</span>
- oracle execution plan은 아래와 같이 볼 수 있다.
```sql
EXPLAIN PLAN FOR [query문]

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY)
```

- mysql의 execution plan은 아래와 같이 볼 수 있다.
```sql
explain [query]
```

- oracle의 index 구성은 아래와 같이 볼 수 있다.
- comment를 포함하여 보는 방법이다.
```sql
SELECT a.table_name
    ,  a.index_name
    ,  a.column_name
    ,  b.comments
FROM all_ind_columns a
    , all_col_comments b
WHERE a.table_name = 'emp'
AND a.table_owner = b.owner
AND a.table_name = b.table_name
AND a.column_name = b.column_name
ORDER BY a.index_name,
        a.column_position
```

- oracle에서 comment 없이 index 구성만 보는 법은 아래와 같다.
```sql
SELECT a.table_name 
     , a.index_name 
     , a.column_name 
  FROM all_ind_columns a 
 WHERE a.table_name = 'EMP' 
 ORDER BY a.index_name
        , a.column_position
```

- mysql은 아래와 같이 간단하다.
```sql
show index from [table이름]
```

## <span style="color:#802548">_암호화 풀기_</span>
- 암호화의 경우, get parameter로 갈 때도 이용된다.
  - 그래서 주의할점이 바로 url-safe한 지 봐야 한다는 점이다.
  - 특히 +(띄어쓰기)나 /(path)나 =(query string)를 유의해야 한다.
  - 따라서 암호화한 값을 다시 base64 url-safe하게 encoding하는 게 좋다.
- Java에서 하는 방식과 Android에서 하는 방식이 다르지 않을 확률이 높다.
  - base64 url-safe는 공식 스펙으로 지정되어있기 때문이다.
  - JDK 1.8이라면 Base64 package를 사용하면 되지만, 1.7에서는 공식으로 지원되지 않는다.
  - 따라서 org.apache.commons.codec.binary.Base64 package를 사용해야한다.
  - 소스코드는 아래와 같다.
```java
String encryptedData = (String)req.getParameter("encryptedString");
byte[] encodedBytes = Base64.encodeBase64URLSafe(encryptedData.getBytes());
String encodedString = new String(encodedBytes);

byte[] decodedBytes = Base64.decodeBase64(encodedString.getBytes());
String decodedString = new String(decodedBytes);
```

- 여기의 경우, urlsafe의 문제는 아니었다.
- 그것보단 split의 문제였다. 우리 서버에서 저쪽으로 보낼 때, plain data가 구분자를 |로 하여 가져갔었다.
- 주민번호|ci|고객번호|고객명|....과 같은 식이다. 그런데 빈값은 그냥 보냈다.
- 따라서 |||고객명||...과 같은 식으로 연속 ||이 붙은 경우가 생겼다.
- 이 경우 split으로 가져올 때 무시되어 8개만 파싱됐다.
- 따라서 빈 값의 경우, "empty"로 주기로 결정했다.
```java
String[] plainDataArray= {"empty", ciNo, cusNm, phoneNo, "empty", "empty"};
List<String> plainDataList = new ArrayList<>();
for (String element:plainDataArray) {
    plainDataList.add(element);
}

String entryString = StringUtils.join(plainDataList, "|");
```

- 인터넷을 찾아보니 join은 List interface만이 아니라 배열에도 가능하다고 한다.
- 아래와 같이 수정해도 될 거 같다.
```java
String[] plainDataArray= {"empty", ciNo, cusNm, phoneNo, "empty", "empty"};
String entryString = StringUtils.join(plainDataList, "|");
```
