- MSA는 cohesion해야 한다.
  - api를 바꿨을 때 연동된 서비스의 소스코드를 바꾸면 MSA를 하는 이유가 없다. 유지보수에 드는 노력이 그대로기 때문이다.
- MSA를 기술적으로만 보고 분리하면 안된다.
  - 그를 유지보수하는 '사람'이 이해할 수 있어야 한다.
  - 또한 유관 업무팀이 이해할 수 있게끔 서비스를 나눠야 한다.
  - 이를 만족하기 위해서는 SRP를 지켜야 한다.
- 그렇다고 너무 많은 service를 나누면 안 된다.
  - 그럼 performance가 심각해진다.
  - 적당히 응집성 있게 service를 나눠야 한다.
  - 서로 결합도가 없어 연관성이 없는 게 최고다.
  - microservice가 작다는 것은, 크기가 작은게 아니라, 연관성이 작아야 한다.

<br >

- MSA를 만들 때는 business capability를 기준으로 만들 수도 있다.
- 온라인 스토어를 상상해보자.
  - 웹페이지   ----> web app service
  - 상품 조회  ----> product service
  - 리뷰 조회  ----> reviews service
  - 주문       ----> orders service
  - 배송       ----> shipping service
  - 재고       ----> inventory service
- service 별로 하나의 책임만을 지기에 SRP를 만족한다.
- SRP를 만족하게 되면 해당 api를 바꾸면 해당 service만 바뀌기에 cohesive도 만족한다.
- looesly coupled도 만족한다.


<br >

- 도메인과 서브 도메인으로 나누는 방법도 있다.
- 개발자에게 좀 더 직관적이다.
  - core  
    - business의 핵심 로직이다.
  - supporting
    - core를 지원한다.
  - generic
    - 공통으로 쓸 수 있는 암호화 같은 것들이다.
    - business랑 관련이 있는 건 아니다.
- online store 사례를 다시 들어보자.
  - core
    - products catalog  ----> products service
  - support         
    - orders            ----> order service
    - inventory         ----> inventory service
    - shipping          ----> shipping service
  - generic 
    - reviews           ----> revies service
    - payment           ----> payments service
    - search            ----> search service
    - image             ----> image service
    - image compression ----> image compression
    - security          ----> sercurity
    - web UI            ----> web app
- image comperssion과 image는 비슷하니 image service로 합쳐준다.
- payments를 order에 합쳐준다. 만약 분리되어야 한다면 다시 분리한다.
- tightly coupled 되지 않게 관리해준다.


<br >

- MSA를 도입하기 위해 새로운 기능 추가를 멈추고 거기에만 집중하는 전략이 있다.
- big bang approach다.
  - 그런데 이 경우에는 개발자가 많아 의견이 많은 만큼 배가 산으로 간다.
  - 거기다 신기능이 추가되지 않는 것은 비즈니스 관점에서 치명타가 된다.
- 그래서 바람직한 MSA는 단계를 밟아 MSA로 전환하는 것이다.
  - 자주바뀌는 service를 먼저 전환한다. merge conflicts를 먼저 해결한다.
  - deadline이 없기 때문에 압박감이 덜해 실수를 덜 하게 된다.
  - 서비스가 계속 발전하기 때문에 사업에 문제가 되지 않는다.


<br >

- MSA 전환을 위해 아래와 같은 단계가 필요하다.
  - code test coverage를 잘 만들어야 한다.
  - 어떤 api를 넣을 지 정해야 한다.
  - 상호의존적인 서비스를 의존하지 않게 만들어야 한다.
- strangle facade를 먼저 만들고, api gateway를 거치게 한다.
- MSA 1을 먼저 만들면 api gate way를 거쳐 monolithic app의 api를 거치지 않고, MSA 1을 거치게 한다.
- 그런식으로 버그가 없는게 확인되면 monolithic application을 삭제한다.
- MSA 전환에 가장 좋은 방법은 code를 최대한 건들지 않고, 기술을 최대한 건들지 않는 것이다. 
- 이미 MSA로 전환 자체가 risky하다.

<br >

- microservice는 각자 DB를 가지고 사용해야 한다.
- 한 microservice에서 다른 db의 data가 필요하다면, 다른 microservice에 api request를 보낸다.
- 만약 다른 microservice가 api를 고치고 있다면, 이를 v2로 만들어 고쳐야 한다. 그래야 서비스는 유지되기 때문이다.
- microservice를 사용하면 어쩔수없이 latency가 늘어난다.
  - DB를 직접 접근하지 않고 request를 보내서 가져오기 때문이다.
  - 그럴 때는 caching을 해두기도 한다. 
  - 이 때 caching은 최신 정보를 유지하지 않아도 되는 REVIEW에 이뤄져야 한다.
  - INVENTORY에 이뤄지게 되면 안된다. inventory는 늘 최신을 유지해야하니 늘 request를 보내게 변경한다.
  - 쓰는 것은 한 service에서만 이뤄져야 한다.
- DB join이 이뤄지기가 힘들다. DB간에 서로 다르다면, join을 할 수가 없다.
  - WAS단에서 이를 통합하여 join된 object를 만들어줘야 한다.
- transaction이 분리되어 rollback하기가 어려워진다.

<br >

- dry는 Dont repeat yourself 원칙이다.
- 반복되면 공통 method로 만들어둬야 한다는 의미다.
- DRY는 microservice에서 만큼은 적용되지 않을 수 있다.
- 공유되는 로직이 있다면 DRY가 원칙이나, util은 그냥 복제해서 각자 만드는 게 낫다.
- 공유되는 로직 중 DRY를 해야 하는 것은 logging, retrying, pattern matching 등이다. 


<br >

- msa의 핵심은 각 팀이 기술 스택을 자유롭게 선택하는 게 아니다.
- 이것은 그냥 허구다. 
  - microservice간 api가 통일되지 않는다. 
    - Rest면 Rest로 통일
    - graphQL이면 graphQL로 통일
    - gRPC면 gRPC로 통일해야 한다
  - 러닝커브가 너무 크다.
    - 다른 팀의 기술스택을 파악해서 문제를 추적해야 한다.
    - 배워야 할 게 기하급수적으로 많아진다.
- 특히 아래 infra 사항은 MSA를 써도 microservice 간에 달라선 안된다.
  - CI/CD, monitoring, alerting
  - api guideline, best practice
  - security, data compliance(정합성)
- 좀 자유로워도 되는 영역은 아래와 같다.
  - 프로그래밍 언어, DB다.
- 완전히 자유로워도 되는 것은
  - local testing, testing, 신입 멘토링 이다.
  - 사실상 개발에 관해서는 기술선택이 자유롭지 않다.

<br >

- 여태까지 알아본 것은 MSA로 백엔드 용이었다.
- MSA로 각 백엔드가 서비스가 나뉜다고 해도, 프론트에서 병목현상이 일어나면 무의미해진다. 
- MFA는 각 화면을 다루는 것을 서비스로 나눈다.
- 페이지의 각 영역을 서로 다른 searchbar는 react로 만들고, 추천 영역은 vue로 만든다.
- MFA로 만든 microFrontservice를 container application에서 동시에 실행시킨다.


<br >

- MSA는 기존 request - response model로는 해결이 어렵다.
- MSA에서 request를 순차적으로 성공코드를 받게 설계하면 latency가 너무 높아진다.

<img src="/image/request-response-msa.jpg" >

- 또한 도중에 실패하면 transaction rollback도 어렵다.
- subscription은 success인데, recommendation을 받지 못하는 것이다.

<img src="/image/side-effect-request-response.jpg" >

- 따라서 parallel하게 진행되게끔 오케스트라 service를 만들어줄 수 있다.

<img src="/image/upgrade-request-response-msa.jpg" >

- 대신 서비스 하나가 잠깐 장애를 잃는 경우에, 별거 아닌데 고객이 아예 사용불가능하다. 무조건 rollback되기 때문이다.


<img src="/image/side-effect-upgrade-request-response-msa.jpg" >


- 여러 결함에 따라 request-response가 아니라 event - driven architecture로 만들어준다.
- producer가 event를 만들고, message broker가 event를 consumer에 보낸다.
  - request-response와 달리 event-driven은 async하다. 
  - event-driven은 message broker가 transaction의 책임을 지닌다. producer가 책임을 지지 않는다. inversion of control이다.
  - loose-coupling 또한 가능하다.

<img src="/image/event-driven1.jpg" >
<img src="/image/event-driven2.jpg" >
<img src="/image/event-driven3.jpg" >
<img src="/image/event-driven4.jpg" >
<img src="/image/event-driven5.jpg" >
<img src="/image/event-driven6.jpg" >

<br >

- 간단한 것들은 그냥 request - response model을 이용하면 된다.
- 복잡한 것들만 event drivent architecture를 사용한다.
- 이벤트를 전달하는 방법은 아래와 같다.
  - event streaming
    - message broker가 event storage로 이용된다.
    - event를 보관하기 때문에 정해진 시간동안은 언제든 다시 retry 가능하다.
  - pub/sub 
    - subscriber는 새로운 event에만 접근이 가능하다.
    - 나중에 subscriber가 추가되어도, 추가된 시점 이후에 새롭게 등록된 event에만 접근 가능하다.


<br >

- publisher가 message broker에 event를 보내고, message broker가 저장했다고 해도, 
- at-most-once
  - message broker가 ack을 안 주면 resend하지 않는다. event duplication을 피하기 위함이다.
  - 중요하지 않은 data loss는 허용하는 정책이다.
  - lowest latency, cost-effective
- at-least-once
  - message broker가 ack을 안 주면 resend한다. retry로직을 만드는 것이다. data replication을 허용하게 된다.
  - data loss를 절대 허용하지 않는 정책이다.
  - increaed latency
- exactly-once
  - retry 로직이 있지만, id를 확인해서 실제로 DB에 transaction 완료 여부를 확인한다.
  - transaction이 완료되었다면 ack만 날리게 처리해야 하며, 이것은 message broker가 아닌 개발자가 직접 구현해야 하는 것이다.
  - highest overhead / latency
  - kafka는 exactly-once를 지원한다.


<img src="/image/exactly-once1.jpg" >
<img src="/image/exactly-once2.jpg" >
<img src="/image/exactly-once3.jpg" >
<img src="/image/exactly-once4.jpg" >
<img src="/image/exactly-once5.jpg" >


- exactly-once는 금융에서 필수적이다.


<img src="/image/exactly-once-example1.jpg" >
<img src="/image/exactly-once-example2.jpg" >
<img src="/image/exactly-once-example3.jpg" >



<br >

- saga pattern1
  - workflow orchestration service를 둔다.
  - sync한 compensation transaction --> 실패 시 transaction rollback에 준하는 행위가 필요하다.


<img src="/image/workflow-orchestration1-undo.jpg" >
<img src="/image/workflow-orchestration2-undo.jpg" >

- saga pattern2
  - event driven model
  - workflow orchestration service를 두지 않는다.
  - asyn한 ccompensation transaction --> 실패 시 transaction rollback에 준하는 행위가 필요하다.


<img src="/image/saga-event-driven1.jpg" >
<img src="/image/saga-event-driven2.jpg" >
<img src="/image/saga-event-driven3.jpg" >


- CQRS (command and query responsibility segration)
  - command는 insert, update, delete 등이다.
    - command는 command 용 database를 갖고 original data를 다룬다.
    - permission, business logic, input validation은 여기에 놓는다.
    - 보통 RDB로 만들어진다.
  - query는 select다.
    - query는 query 용 database를 갖고, data copy를 다룬다.
    - command가 바껴도 query는 바뀌지 않고, 따라서 재배포할 필요가 없다.
    - 보통 NoSql로 만들어진다.
  - CQRS는 microservice의 SRP 원칙을 지키는  데 적합하다.
  - 각 type 별로 scale out 하기도 쉽다.
  - microservice 별로 table을 별도로 갖는 경우보다 join을 하기도 쉽다.


<img src="/image/cqrs1.jpg" >
<img src="/image/cqrs2.jpg" >
<img src="/image/cqrs3.jpg" >
<img src="/image/cqrs4.jpg" >



- 실제 예시로 식당 리뷰 앱을 살펴보자.
-  리뷰는 자주 바뀐다.
   -  business logic을 위한 많은 validation도 필요하다.
-  레스트토랑은 자주 바뀌지 않는다.
-  고객은 레스토랑과 리뷰를 join한 데이터를 보아야 한다. 
-  그럴 때 CQRS를 도입 해 read 전용 db를 만든다.


<img src="/image/cqrs5.jpg" >
<img src="/image/cqrs6.jpg" >
<img src="/image/cqrs7.jpg" >



- event sourcing pattern
- history를 통해 최신 정보를 가져온다.
- account balance를 딸랑 최신 정보만 보여주면 당황스러울 것이다.
- 그런 경우 history를 보여주면서 최신 정보를 알려주어야 한다.
  - event를 store하는 것은 database가 있다.
  - message broker도 있다.
- 그렇다고 모든 history를 보여줄 수는 없다.
- 따라서 달마다의 transaction을 보여준다. snapshot 전략이다.
- CQRS 전략을 쓰기도 한다.
- event sourcing + CQRS 전략이 가장 인기가 많다.
- 다만 eventually consistency만 가능하다. 실시간 정보 갱신은 안된다.