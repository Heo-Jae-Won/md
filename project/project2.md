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

- 그저 해당 문자열이 있는지만 비교할 거라면, substring.eqauls를 쓸 필요가 없다.
- indexOf로 가져오면 된다.
```java
string.substring().equals() //substring creation overhead
string.indexOf() >= 1       //better
```