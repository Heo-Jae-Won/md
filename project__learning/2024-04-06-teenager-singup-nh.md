
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
            WHEN 'B1_011' /*모 회사*/
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





## <span style="color:#802548">_느린 네트워크에서 CSS 무너짐_</span>
- 3층 환경이 AP 기기가 너무 느렸다. 
- api 한번 response 받는데 6초가 걸렸다. render는 25초 걸렸다.
- 그런데 알 수 없는 이유로 자꾸 앱에서 CSS가 무너졌다.
- 로컬에서는 절대 그런일이 없었다. 그런데 자꾸 무너지니까 미칠거 같았다. 디버깅해도 문제가 보이지 않았다.
- 그러다 혹시 느려서 그런건 아닐까 하고, 개발자 도구에서 네트워크 환경을 느린 3G로 바꾸니까 실제 내가 겪는 문제처럼 나왔다.
- 그렇다. 느린 네트워크로 인해 뭔가 문제가 생긴 것이었다.
- 그래서 원인을 보니, font가 원래 지정된 font가 아니라 default font가 적용되고 있었다.
- font를 가져오는 게 가장 느렸는데, 그래서 font를 적용하지 않고 그냥 DOM을 그려버렸다.
- 그래서 원하는 CSS가 안나왔던 것이다. 실제로 가끔 빠르게 화면이 뜨면 정상적으로 입력 CSS가 한줄에 나왔는데, 그런 이유였던 것이다.
- 만약 느린 환경까지 고려한다면, 폰트를 최대한 압축하는 것도 중요하겠다는 생각이 들었다.
  






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


## <span style="color:#802548">_jquery의 dom life cycle_</span>
- Jquery에도 DOM의 lifecycle을 관리할 수 있는 method들이 있다.
- 해당 method들을 무시하면 page 관리에서 오류가 날 수 있다.
- 난 모 회사에서 웹뷰의 title을 바꾸는데, $(window).load()와 $().ready()의 차이를 몰라서 해결하는 데 오래 걸렸다.
  - 참고로 아래가 ready()와 동일한 method다.
  - ready는 DOM 생성이 끝나면 발생한다. vue의 onMounted나 react의 useEffect와 같다. DOM 생성과 함께 js가 읽히기 때문에 변수가 undefined일 위험은 없다.
  - load는 DOM만이 아니라 DOM의 content까지 모두 완료되었을 때 발동한다. 따라서 SSR용코드라고 보면된다.
- 아래와 같은 상황에서 init이 먼저 발동하여 title이 hide 되지만, load가 뒤이어 발동하면서 title이 change되면서 show된다.
  - 팝업에 header가 있다면 title이 중복으로 뜨는 것과 같은 효과를 내게 된다.
```js
$(window).on('load',function() {
    location.href = '앱스키마://titleChange?title=A'
})

$(function(){
    init();
})

function init() {
    const isValid = computedValidationResult(accidentCode);
    if(!isValid) {
        location.href = '앱스키마://titleHide'
        openPopup('사고코드가 있어 페이지 진입이 불가능합니다');
    }
}
```

- 그럴 때 아래와 같이 순서를 바꿔준다.
- 페이지 진입과 동시에 title이 바뀌어야 하므로 load가 아닌 ready에 title을 change하는 함수를 먼저 명시하는 것이다.
  - load는 지워버린다.
  - 그럼 먼저 title이 바뀌고, 뒤이어 init함수 내에서 조건을 만족하면 title이 hide되게 된다.
  - 팝업에 header가 있다면 title이 popup의 header만 있기 때문에 1개만 유일하게 뜨게 된다.
```js
$(function(){
    location.href = '앱스키마://titleChange?title=A'
    init();
})

function init() {
    const isValid = computedValidationResult(accidentCode);
    if(!isValid) {
        location.href = '앱스키마://titleHide'
        openPopup('사고코드가 있어 페이지 진입이 불가능합니다');
    }
}
```

- setTimeout 등을 주는 것은 아무런 의미가 없다.
  - 그건 UI 변화가 event loop와 연관있을 때 가능한 trick이다.
  - 따라서 아래와 같이 setTimeout을 줘도 타이틀이 중복으로 뜨는 현상을 막을 수는 없다.
```js
$(window).on('load',function() {
    location.href = '앱스키마://titleChange?title=A'
})

$(function(){
    init();
})

function init() {
    const isValid = computedValidationResult(accidentCode);
    if(!isValid) {
        setTimeout(function() {
            location.href = '앱스키마://titleHide'
        }, 500);
        openPopup('사고코드가 있어 페이지 진입이 불가능합니다');
    }
}
``` 



## <span style="color:#802548">_운영 에러로그 만드는 법_</span>
- 아래와 같이 쓰는 게 모 회사의 방식인데, 정상인 경우에는 괜찮다.
```java
public class Main {
	public static void main(String[] args) throws InterruptedException {
		char[] chars = {'a','b','c','d'};
		try {
			calculate(chars, 6, i -> i[0]==1 && i[1] == 1 && i[2] == 0);	//9 line
		} catch (Exception e) {
			StackTraceElement[] stackTrace = e.getStackTrace();
			StackTraceElement element = stackTrace[0];
			System.out.println("lineNumber: " + element.getLineNumber() + "\nmessage: " + e.getMessage());
		}
		
		
	}
	
	static void calculate(char[] a, int k , Predicate<int[]> decider) {
		int n = a.length;
		if (k <1 || k > n) {
			throw new IllegalArgumentException("Forbidden"); //23 line
		}
		
		int[]	 indexes = new int[n];
		int total = (int) Math.pow(n,  k);
		
		while (total-- > 0) {
			for (int i = 0; i < n - (n-k); i++) {
				System.out.println(a[indexes[i]]);
			}
			System.out.println();
			
			if (decider.test(indexes)) {
				System.out.println("gotya");
				//break;
			}
			
			for (int i = 0; i < n; i++) {
				if (indexes[i] >= n-1) {
					indexes[i] = 0;
				} else {
					indexes[i]++;
					break;
				}
			}
		}
	}
}
```

- [0]으로 exception line을 가져오면 현재는 error를 던진 곳이 line이 찍히고 있다.
- 정상 출력되고 있는 셈이다. error를 명시적으로 throw할 때는 아무 문제가 없다.

```
java.lang.IllegalArgumentException
	at com.example.Main.calculate(Main.java:23)
	at com.example.Main.main(Main.java:9)
lineNumber: 23
```


- 이번에는 명시적인 throw를 하지 않았는데, java api 활용에서 실수를 냈다.
- substring을 할 때 indexoutofboundsexception이 났다.
- try catch를 넣으면 아까와는 사뭇 양상이 다르다.

```java
public class Main {
	public static void main(String[] args) throws Exception {
		char[] chars = {'a','b','c','d'};
		calculate(chars, 6, i -> i[0]==1 && i[1] == 1 && i[2] == 0);	//8 line
	}
	
	static void calculate(char[] a, int k , Predicate<int[]> decider) {
		
		try {
			int n = a.length;
			String abc ="fff";
			abc = abc.substring(0,12); // 16 line
			
			int[]	 indexes = new int[n];
			int total = (int) Math.pow(n,  k);
			
			while (total-- > 0) {
				for (int i = 0; i < n - (n-k); i++) {
					System.out.println(a[indexes[i]]);
				}
				System.out.println();
				
				if (decider.test(indexes)) {
					System.out.println("gotya");
					//break;
				}
				
				for (int i = 0; i < n; i++) {
					if (indexes[i] >= n-1) {
						indexes[i] = 0;
					} else {
						indexes[i]++;
						break;
					}
				}
			}
		} catch (Exception e) {
			e.printStackTrace();
			StackTraceElement[] stackTrace = e.getStackTrace();
			StackTraceElement element = stackTrace[0];
			System.out.println("lineNumber: " + element.getLineNumber() + "\nmessage: " + e.getMessage()+ "\nclass: " + element.getClassName());
		}
		
	}
}
```

- String class가 맨 첫줄에 exception에 찍힌다.
- 우리가 기대한 것은 calculate의 16 line이었다. 거기서 실제 exception이 났다.
- 근데 얼탱이없이 String class가 exception line에 찍힌다.
- 명시적인 throw를 하지 않았기 때문이다.
```
java.lang.StringIndexOutOfBoundsException: begin 0, end 12, length 3
	at java.base/java.lang.String.checkBoundsBeginEnd(String.java:4604)
	at java.base/java.lang.String.substring(String.java:2707)
	at com.example.Main.calculate(Main.java:16)
	at com.example.Main.main(Main.java:8)
lineNumber: 4604
message: begin 0, end 12, length 3
class: java.lang.String
```


- error를 throw하고 catch로 잡아서 다시 exception을 던지는 경우에는 어떻게 될까?
- 사실 이게 찐 모 회사 운영 에러 로그 방식이다.
```java
public class Main {
	public static void main(String[] args) throws InterruptedException {
		char[] chars = {'a','b','c','d'};
		try {
			calculate(chars, 6, i -> i[0]==1 && i[1] == 1 && i[2] == 0);	//9 line
		} catch (Exception e) {
			e.printStackTrace();
			StackTraceElement[] stackTrace = e.getStackTrace();
			StackTraceElement element = stackTrace[0];
			System.out.println("lineNumber: " + element.getLineNumber() + "\nmessage: " + e.getMessage());
		}
		
		
	}
	
	static void calculate(char[] a, int k , Predicate<int[]> decider) throws Exception {

        try {
            int n = a.length;
            if (k <1 || k > n) {
                throw new IllegalArgumentException("Forbidden"); //23 line
            }
            
            int[]	 indexes = new int[n];
            int total = (int) Math.pow(n,  k);
            
            while (total-- > 0) {
                for (int i = 0; i < n - (n-k); i++) {
                    System.out.println(a[indexes[i]]);
                }
                System.out.println();
                
                if (decider.test(indexes)) {
                    System.out.println("gotya");
                    //break;
                }
                
                for (int i = 0; i < n; i++) {
                    if (indexes[i] >= n-1) {
                        indexes[i] = 0;
                    } else {
                        indexes[i]++;
                        break;
                    }
                }
            }
        } catch (Exception e) {
            throw new Exception("0001"); //52 line
        }
	}
}
```

- calculate의 23 line에서 exception이 났지만, 잡아서 52 line에서 던지기 때문에 바꿔치기된다.
- 따라서 어디서 exception이 났는지 도저히 알수가 없게 된다.
- 명시적으로 throw를 던진다면 catch로 잡아서 다시 던지면 안된다는 의미다.

```
java.lang.Exception: 0001
	at com.example.Main.calculate(Main.java:52)
	at com.example.Main.main(Main.java:9)
lineNumber: 52
message: 0001
```

- java api 활용에서 오류가 났을 때도 마찬가지다.
- substring() 활용에서 exception이 났다.

```java
public class Main {
	public static void main(String[] args) throws Exception {
		char[] chars = {'a','b','c','d'};
		
		try {
			calculate(chars, 6, i -> i[0]==1 && i[1] == 1 && i[2] == 0);	//10 line
		} catch(Exception e) {
			e.printStackTrace();
			StackTraceElement[] stackTrace = e.getStackTrace();
			StackTraceElement element = stackTrace[0];
			System.out.println("lineNumber: " + element.getLineNumber() + "\nmessage: " + e.getMessage()+ "\nclass: " + element.getClassName());
		}
		
	}
	
	static void calculate(char[] a, int k , Predicate<int[]> decider) throws Exception {
		
		try {
			int n = a.length;
			String abc ="fff";
			abc = abc.substring(0,12); // 25 line
			
			int[]	 indexes = new int[n];
			int total = (int) Math.pow(n,  k);
			
			while (total-- > 0) {
				for (int i = 0; i < n - (n-k); i++) {
					System.out.println(a[indexes[i]]);
				}
				System.out.println();
				
				if (decider.test(indexes)) {
					System.out.println("gotya");
					//break;
				}
				
				for (int i = 0; i < n; i++) {
					if (indexes[i] >= n-1) {
						indexes[i] = 0;
					} else {
						indexes[i]++;
						break;
					}
				}
			}
		} catch (Exception e) {
			throw new Exception("fffff"); //51 line
		}
		
	}
}
```

- error는 실제로 26 line substring()에서 났는데, 해당 에러는 stackTrace에 찍히지도 않았다.
- api 실수를 잡아서 error를 던진 곳이 stackTrace에 잡히게 된다.
- 어디서 오류가 났는지 모르게 되는 상황이다. 심지어 printStackTrace를 찍었는데도 말이다.
- 거기다 error message가 스프링에서 제공해주는 message가 아니라 유저가 넣은 메시지가 되어버린다.
- 에러 정보를 아예 얻을 수가 없게 되는 것이다. 정말 최악이 아닐수가 없다.

```
java.lang.Exception: fffff
	at com.example.Main.calculate(Main.java:51)
	at com.example.Main.main(Main.java:10)
lineNumber: 51
message: fffff
class: com.example.Main
```



- 위와 같은 방식으로는 exception이 난 line을 파악하는 게 불가능에 가깝다.
- 기본적으로 StackTrace의 line은 호출스택이 경우에 따라서 달라지기 때문에 exception이 난 line을 아는 것은 어렵다.
- 따라서 무조건 error log를 java의 stackTrace로 쌓는 곳이라면 line number가 아니라, exception의 message를 아는 것이 중요하다.
- 정말 중요한 로직이라면 logger.error를 찍어서 확인하는 것이 필요하다.
- 가장 중요한 것은, catch를 한 뒤에 또 exception을 던지면 안 된다는 점이다. 그런 식으로 하면 stackTrace가 꼬인다.
- 어느 line에서 error가 났는지를 순전히 모든 코드를 분석해보면서 때려맞춰야 한다.