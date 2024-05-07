## <span style="color:#802548">_중복코드 제거_</span>
- 아래와 같은 if문이 여러군데 중복된다면 한군데만 고치면 if문을 사용하는 다른 곳에서는 빵꾸가 날 수 있다.
```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
if ((employee.flgas && HOURLY_FLAG) && (employee.age > 65))
```

- 따라서 아래와 같이 class안에 method로 묶어주자.
- 만들어둔 isEligibleForFullBenefits logic을 이제 계속 활용하면 된다.
- if문은 Employee class 안으로 encapsulation된다.
- 해당 방식은 DDD에선 가능한데, mybatis에서는 어려울 수도 있다. 
  - 특히 dto class도 없는 곳에선... 불가능하다.

```java
@Data
public class Employee {
    private boolean flags;
    private int age;

    public boolean isEligibleForFullBenefits() {
        if( (this.flags && HOURLY_FLAG) && (this.age > 65) ) {
            return true;
        } 

        return false;
    }

}

public class EntryController {

    @GetMapping("/employees/{employeeId}")
    public EmployeeDto checkDuplication(@PathVariable String employeeId) {
        Employee employee = EmployeeService.retreiveEmployee(employeeId);
        if (emplyee.isEligibleForFullBenefits()) {
            //business logic
        }
    }
}
```


- class를 못쓰는 js convention이라면 어떤 의미인지 알기 쉽게 named boolean 변수로라도 만들어주자.
```js
const isEligibleForFullBenefits = ((employee.flgas && HOURLY_FLAG) && (employee.age > 65));
```


## <span style="color:#802548">_어떻게 주석을 써야 할까_</span>
- 주석으로 쓸 내용은 아래와 같다.
  - code가 특정 방식으로 작성된 이유
  - TODO, HACK, FIX
  - 상수가 특정 값으로 정해진 이유. 
    - check_age2 보다는 teenAgerStartAge라고 하고 아동시작나이라고 주석으로 쓴다.
    - check_age1 보다는 youthAgerStartAge라고 하고 청소년시작나이라고 주석으로 쓴다.


## <span style="color:#802548">_API 문서화_</span>
- 코드 문서화의 경우, 공개 API에 관해서만 진행한다.
- 또한 내부 동작방식을 설명하면 안 된다.
- 반드시 3인칭으로 진행한다.
  - gets the label (o)
  - get the label (x)
- 메소드 설명은 동사구로 시작한다.
  - gets the label of this button

- 클래스/인터페이스/필드는 주어 생략
  - A button label(o)
  - This field is button label (x)
- 현재 클레스에서 생성된 객체를 지칭할 때는 the 대신 this
  - gets the toolkit for this componenet (o)
  - gets the toolkit for  the component (x)



- 잘쓴 javadoc API문서의 사례는 아래와 같다.
```java
/**
* Registers the text to display in a tooltip.
* This text displays when the cursor lingers
* over the component.
*
*
* @param text the string to display. If the text is
*                    null, the tool tip is turned off for
*                    this component.
*
*/
```


## <span style="color:#802548">_git commit message 남기는 법_</span>
- commit은 how 대신 what과 why로 적는 것이다. 
  - 맨처음은 요약 50글자
    - 그래야 git log --oneline 으로 명령으로 요약이 전부 출력된다.
  - 본문은 70글자 정도..
  - 마지막은 참고사항을 적는다.
- git commit -m "Fix typo in introduction to use guide" 같이 -m 옵션은 쓰면 안된다.
  - vscode는 source control에서 컨트롤 엔터하면 vim처럼 쓸 수 있게 제공해준다.
  - 자동 행갈이가 없으니 enter를 쳐줘야 한다.
- commit은 아래와 같이 구성해서 쓴다.
```
Change default confirm taget from 2 to 6  //1. 요약 50자짜리 commit
//한줄 띄고

//바꾼 이유
Recent discussion(in IRC meetings, and e.g. #8989) has shown a preference for the default confirm target for smartfees to be 6 instead of 2, to avoid overpaying fees for questionable gain.
//한줄 띄고

//참고사항
6 is also a compromise between the GUI's pre-#8989 value of 25 and the bitconid ... 
```



- 구체적인 commit message를 쓰는 사항은 아래와 같다.
  - 주제행 첫글자는 대문자가 되어야 한다.
  - 행 마지막에 점을 찍으면 안 된다
  - 주제 행을 명령문으로 만들어라
    - fixed bug -->fix bug
    - changing behavior -> change behaviour
    - sweet new API -> release version 1.0.0
    - more fix for broken stuff -> remove deprecated methods


- 잘 쓴 예시는 아래와 같다.
```
Convert template to US-ASCII to fix error

I introduced some tests in a feature branch to match the contents of
`/etc/nginx/router_routes.conf`. They worked fine when run with `bundle exec
rake spec` or `bundle exec rspec modules/router/spec`. But when run as
`bundle exec rake` each should block failed with:

    ArgumentError:
      invalid byte sequence in US-ASCII

I eventually found that removing the `.with_content(//)` matchers made the
errors go away. That there weren't any weird characters in the spec file. And
that it could be reproduced by requiring Puppet in the same interpreter with:

    rake -E 'require "puppet"' spec

That particular template appears to be the only file in our codebase with an
identified encoding of `utf-8`. All others are `us-ascii`:

    dcarley-MBA:puppet dcarley$ find modules -type f -exec file --mime {} \+ | grep utf
    modules/router/templates/routes.conf.erb:text/plain; charset=utf-8

Attempting to convert that file back to US-ASCII identified the offending
character as something that looked like a whitespace:

    dcarley-MBA:puppet dcarley$ iconv -f UTF8 -t US-ASCII modules/router/templates/routes.conf.erb 2>&1 | tail -n5
      proxy_intercept_errors off;

      # Set proxy timeout to 50 seconds as a quick fix for problems
      #
    iconv: modules/router/templates/routes.conf.erb:458:3: cannot convert

After replacing it (by hand) the file identifies as `us-ascii` again:

    dcarley-MBA:puppet dcarley$ file --mime modules/router/templates/routes.conf.erb
    modules/router/templates/routes.conf.erb: text/plain; charset=us-ascii

Now the tests work! One hour of my life I won't get back..
```

- commit의 맨 앞에 나오는 전형적인 단어들은 아래와 같다.
  - fix -> 올바르지 않은 동작을 고침
  - add -> 기능, test, 문서를 추가
  - remove -> 코드를 삭제
  - use -> 뭔가를 사용해 구현
  - refactor -> 리팩토링
  - simplify -> refactor보단 약한 수정( if문 위치 옮기기.. field명만 바꾸기.. )
  - update -> 수정/추가/보완
  - improve -> 호환성/테스트커버리지/접근성 향상이 있을 때
  - implement -> 클래스/모듈 단위로 코드를 추가
  - revise -> 문서를 개정
  - correct -> 문법 오류, 타입 변경, 이름 변경
  - ensure  -> 오류처리가 없었는데 추가 등 기능을 보장
  - prevent -> stackoverflow나 out of memory 등이 일어날 수 있는데 이를 막기 위해 코드를 수정한 경우


<br/>


- 파일 수정의 목적이 다르면 각자 개별 commit으로 넣어야 한다.
  - 기능추가
  - 주석보완
  - import정리
    - 위 3개는 모두 파일 수정 목적이 다르다.
    - 개별 commit으로 넣어야 한다..


## <span style="color:#802548">_좋은 error message 만들기_</span>
- 좋은 오류메시지 구성요소는 다음과 같다.
  - 오류가 생긴 원인
  - 어떤오류인지
  - 오류를 극복한 시도
- 실제 예시로 들면 아래와 같다.
- 아래같이 오류가 생긴 원인은 알았다.
- 그러나 해당 메시지로는 어떤 오류인지는 알기 어렵다.
```
couldnt parse config file
```

- 어떤 파일 때문에 오류가 났는지 명확하게 적는다.
```
couldnt parse config file ---> couldnt parse config file: /etc/sample-config.properties
```

- 해당 파일의 어떤 부분 때문에 오류가 났는지 구체적으로 적는다.
```
couldnt parse config file: /etc/sample-config.properties---> couldnt parse config file: /etc/sample-config.properties; given snapshot mode 'nevr' isn't valid
```


- 오류를 고치기 위해서 할 수 있는 간단한 solution도 추가해준다.
```
couldnt parse config file: /etc/sample-config.properties; given snapshot mode 'nevr' isn't valid---> couldnt parse config file: /etc/sample-config.properties; given snapshot mode 'nevr' isn't valid(must be one of initial, always, never)
```

- 서버에서 뿌려줄 때 메시지만이 아니라 코드도 같이 주는 게 훨씬 좋다.
- 메시지는 변경될 수 있기 때문에 코드와 묶어주는 게 좋다.
  - ORA-01476: divisor is equal to zero

- 오류메시지를 쓰는 법은 아래와 같다.
  - 태는 통일한다.
    - couldnt parse config file (능동태)
    - config file couldnt be parsed (수동태)
    - 하나 골라서 모든 오류 메시지를 동일한 태로 쓴다.
  - 민감한 정보는 노출하면 안 된다.
  - 오류는 가장 빨리 일어난 시점에서 출력되어야 한다. 실패시점에 바로 던지는게 좋다.

<br/>

- 오류 메시지를 쓰는 법을 추천 예시와 비추천 예시를 통해 살펴보자.
- 오류 원인을 정확히 파악하는 메시지가 좋다.
```
not recommended
bad directory

recommended
the specified directory exists but is not writable. To add files to this directory the directory must be writable.
```

- 사용자의 유효하지 않은 입력을 파악해서 알려주는 메시지가 좋다.
```
not recommended
invalid postal code

reommended
The postal code for the US must consist of either five or nine digits. The psecified postal code (input) contained sevent digits.
```

- 요구사항과 제약 사항을 정확하게 명세하는 메시지가 좋다.
```
not recommended
The combined size of the accatchments is too big.

recommended
The combines size of the attachments (14MB) exceeds the allowed limit (10MB). 
```
- 예제를 제공하는 메시지가 좋다.
```
Not recommended
Invalid email address.

recommended
The specified email address (robin) is missing an @Sign and a domain name. For example: robin@example.com
```
- 오류메시지는 간결해야한다.
```
Not recommended
The resource was not found and cannot be differentiated. What you selected doesnt exists in the cluster.

recommended
Resource<name> isn't in cluseter <name>.
```

- 이중부정은 읽기 어려우니 쓰지 말자.
- 아래같이 excep와 unless는 같이 쓰지 않는다.
```
Not reco 
The App Engine Service account must have permissions on the image, except the Storage Object Viewer role, unless the Storage Object Admin role is available.

recommended

the App Engine service account must have one of the following roles:
 1. Storage Object Admin
 2. Storage Object Creator
```
- 사용자에 맞는 맞춤형 오류가 필요
```
not recommended
A server dropped ur clients' request because the server farm is running at 92% CPU capacity. Retry in finve minutes.

recommended
So many People are shopping right now that our system can't complete ur purchase. Dont worry-- we won't lose ur shopping cart. Plz retry ur purchase in five minutes.
```
- 오류 메시지는 일관되게 사용한다.
```
Not recommended
Cant connect to cluster at 127.0.0.1:56. Check whether minikube is running.

recommended
Cant connect to minikube at 127.0.0.1:56. Check whether minikube is running.
```
- 긍정적 표현을 사용하자
```
Not recommended
you entered an invalid postal code.

recommended
Enter a valid postal code.
```
- 과도한 사과를 피하자
```
Not recommended
we're sorry, a server error occurred and we're temporarilty unable to load ur spreadsheet. we apologize for the inconvenience. plz wait a while and try again.

recommended
Google Docs is temporarily unable to open ur spreadsheet. In the meantime, try right-clicking the spreadsheet in the doc list to download it
```
- 사용자를 비난하지말자
```
Not recommended
You specifed a printert that's offline

recommended
The specified printer is offline.
```
 
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


- 통일성 또한 중대한 문제다. 
- DB에서 가져오는 걸 get으로 했으면 get으로 다 통일해서 가져온다.
  - get이 아닌 fetch로 했으면 다 fetch로 통일한다.
  - retrieve로 다 했으면 retrieve로 통일한다.
  - get보단 fetch/retrive/load가 낫겠지만 일관적으로 쓰는 게 더 중요하다.



