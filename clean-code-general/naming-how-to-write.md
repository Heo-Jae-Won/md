
## <span style="color:#802548">_적절한 네이밍의 중요성_</span>
- 애매한 용어는 피하는 게 좋다.
- 아래는 피하면 좋을 단어와, 대안이다. 
  - 기본적으로 get은 시간이 짧고 exception이 필요하지 않은 간단한 DB select다. 
  - 반면 find는 시간이 길게 걸리고 exception이 필요한 간단하지 않은 비즈니스 로직이 포함된 select다.
  - get이라고 늘 DB에서 가져오는 것도 아니고 find도 그렇다. 파일/메모리 등에서도 가져올 수 있다.
- compute
  - 수리적 계산이라면 calculate. caculateAge
  - 유효성 검증이라면 determine. determineValidationResult
  - 감정평가라면 assess. assessAssetPrice
- get
  - 가져오는 시간이 짧다면 fetch/retrieve. fetchUser/retrieveUser
  - 가져오는 시간이 길다면 load. loadUserLogStatistics
- send
  - 메시지 전송이라면 dispatch. dispatchSMS
  - 최적 경로 선택이라면 route
  - 변경사항 공지라면 announce. announce
  - 알림함 이라면 notify. notifyPurchaseEvent
- find
  - db검색이라면 search
  - 가져온 정보에서 특정 정보를 추출하는 거라면 extract
  - 위치를 찾는 것이라면 locate
  - 예외로 JPARepository는 DB검색은 search가 아닌 find를 사용한다.
- start
  - 프로그램 시작이라면 launch
  - 프로세스 생성이라면 create
- make
  - 소스코드를 배포하기 위해 빌드파일을 만든 건 build
  - business logic에 필요한 결과물을 생성한 거면 generate
  - 기존에 있는 data를 조립해서 새로운 형태로 만든거라면 compose
  - 동적으로 html을 생성한다면 append
- stop
  - 다시 되돌릴 수 없다면 kill
  - 다시 되돌릴 수 있다면 pause

<br/>

- 아래와 같이 그냥 get을 쓰면 매우 위험하다.
- Java는 GC라도 있지만 C++은 GC도 없어 memory가 해제되지 않은채로 남는다.
```java
public ObjectOutputStream getOos() throws IOException {
  if(oos == null) {
    oos = new ObjectOutputStream(socket.getOutputStream());
  }
  return oos;
}
```

- 따라서 method명을 아래처럼 바꿔준다.
- get은 특히 조심해야 한다. 이름에 부수 효과를 숨기기 쉽다.
- 이름에 부수 효과를 숨기지 않으려면 context를 모두 제공하는 게 좋다.
```
getOos ㅡㅡㅡ> createOrReturnOos
```
- 맥락을 제거하는 변수명, method명을 쓰지 않는 것은 매우 중요하다.
  - updateMoneyMember라고 해서 moneyMember의 status만 바꾸는 지 알았지만, 변경일자도 같이 변경된다.
  - 

- network programming 관련 예시도 들어보자.
- ServerCanStart()보다는 CanListenOnPort()가 낫다. 
- CanListenOnPort는 보자마자 데몬프로세스가 특정 포트를 열어 들어오는 네트워크에 귀를 기울이고 있다는 사실을 알 수 있다. 
  - 따라서 문제가 발생한 경우는 다른 프로세스가 이미 해당 포트를 점유하거나 포트 접근권한이 부족해서라는 사실을 알기쉽다.

- method만이 문제가 아니라 변수명도 맥락을 제공해야한다.
```
password -->plainPassword 
        ---> encodedPassword

data -> urlEncodedData
```
- 필요없는 맥락은 오히려 제공하지 않는 게 좋다.
- 강타입언어에 타입정보를 식별자에 적을 필요가 없다.
```java
String retStr ="";
HashMap<String, Object> outputMap = new HashMap<>();
```

- 아래와 같이 바꿔주면 된다.
```java
String result = ""; // result 대신 response도 된다..
HashMap<String, Object> output = new HashMap<>();
```

- method 이름은 자신이 하는 일을 나타내야 한다.
- 그 하는 일은 추상적인 add 같은 것 따위로 이름지으면 안 된다.
- 카드번호를 암호화하여 추가하는 method라면 그냥 mnCardAddInfo()로 지으면 안 된다.
  - encryptCardNoAndAddCardInfo로 지어야 했다.
- is로 시작되는 것들은 반드시 boolean type을 return해야 한다.


<br >


- 통일성 또한 중대한 문제다. 
- DB에서 가져오는 걸 get으로 했으면 get으로 다 통일해서 가져온다.
  - get이 아닌 fetch로 했으면 다 fetch로 통일한다.
  - retrieve로 다 했으면 retrieve로 통일한다.
  - get보단 fetch/retrive/load가 낫겠지만 일관적으로 쓰는 게 더 중요하다.





