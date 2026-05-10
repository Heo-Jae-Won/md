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


<br >

- unit tests
  - single micro service unit
- integration tests
  - communication between 2 microservice
  - 누가 test 환경을 셋팅할 지 불명확하다.
- functional end-to-end tests
  - real app test
  - 누가 테스트 데이터를 만들지 불명확하다.
  - 누가 에러의 책임을 질지 불명확하다.
  - 누가 test 환경을 셋팅할 지 불명확하다.
  - 비싸다

<br >

- Integration Tests 진행시 환경 setting용 mock을 만든다.
- microservice B가 api producer고 microservice A가 api consumer라면
- response를 뿌리는 producer인 microservice를 mock으로 만든다.
- 반대로 microservice A에 mock을 만들수도 있다.

<img src="/image/integration-test-msa-mock.jpg" >

- 다만 각 팀에서 api를 change했을 때 mock에는 반영되지 않는다.
- 이럴 때 contract test를 도입한다. mock이 아니라 contract testing file에 response와 request를 저장해두어 사용한다.
- contract testing tool로는 Spring Cloud Contract , Pact가 있다.

- real app test가 안되면, 그냥 배포해서 테스트해보기도 한다.
- blue-green deployment, canary testing 등이 있다.


<br >


- MSA의 관찰성은 아래와 같이 나뉜다.
  - monitoring
    - collect, analyz, display 
    - dont tell us problem
    - for monolithic application
  - observability
    - tell us problem
    - can debug
    - for microservice

- 사실상 microservice는 observability를 사용한다.
- 여기엔 3가지 요소가 들어있다.
- distributed logging
  - Event, Action, Exception, Error
  - 대용량을 다루는데, 각 WAS별로 로그파일을 전부 훑어보기는 현실적으로 무리다.
  - 따라서 중앙화된 로깅 시스템이 필요하다.
  - 로그 구조도 기계가 parseable하게, 사람이 readable하게 잘 짜야한다.
    - 예를들어, JSON, XML로 포매팅하는 것이다.
  - 로그 레벨도 fine tuning이 필요하다.
    - 정말 치명적인 것은 ERROR, FATAL로 설정해야 한다.
    - 뭔가 값이 이상하게 들어가는 것들은 WARN으로 설정해야 한다.
    - 나머지 Event 발생들은 DEBUG, TRACE로 놓는다.
  - 또한 처음 진입 request에 id를 부여해서 계속 해당 id로 log를 남기면 좋다.
    - order를 하는 게 request_id가 432면, 해당 주문건에 대한 결제도 request_id가 432로 남고, 배송, 푸시도 똑같은 request_id다.
  - 마지막으로 맥락을 많이 제공해야 한다.
    - timeout을 낸 query와 request
    - 값이 없어 nullpointer가 난 코드 레벨
    - 에러가 난 서비스 이름
    - 에러가 난 시간
    - IP와 user ID(PK)
    - 다만 절대로 절대로 개인 민감 정보는 로깅에 남기면 안 된다.
  - 보통 ELK나 EFK를 사용한다.
- metrics
  - 대쉬보드에 시각화된 signal을 의미한다.
  - 농좆에도 있어서 MSA 전용은 아니다.
  - resources - CPU,Memory,DisskIO,NetWork,Processes
  - App instrumentation -  request time, query time, throuughput, queue size, latency ....
  - 하지만 이 모든 걸 다 모을 수는 없다.
  - 정보를 모으는 데 비용이 너무 비싸다.
  - 또한 다 모아도 뭐가 문제인지 파악하기 어렵다.
  - 따라서 5개의 type만 집중한다.
    - Traffic 
    - Errors
    - Latency
    - Saturation
    - Utilization

- Traffic
  - HTTP requests/ sec
  - Queires/ sec
  - Transactions/ sec
  - Events received/ sec
  - Events delivered/ sec
  - Incoming requests + outgoing reqeuest/ sec
- Errors
  - Error rate and Error Types
  - HTTP response status code
  - response exceeding latency thresholds
  - Faild Events
  - Failed Delivery
  - Aborted transaction
- Latency
  - not just average, with Latency distribution
  - seperate successful operations from failed operations
- Saturation
  - 너무 많은 서비스가 하나의 서버에 집중되는지
  - DB scale out, server scale out이 필요할수 있다.
- Utilization
  - CPU, memory, disk
  - 사용률이 80%가 넘으면 scale out이 필요하다.
  - 10 min average가 아니라 1 min average 단위로도 봐야..
  - 길게 잡으면 평균값에 수렴하게 되어 edge case를 놓치게 된다.


<img src="/image/metrics.jpg" >


- 마지막 요소는 distributed tracing
- 고객이 주문을 했는데, 노티가 하루 뒤에 메일로 날라왔다.
- 그럼 Order가 문제였을까? noti가 문제였을까? payments가 문제였을까?
- 이걸 전부 metrics를 보거나, logging을 보기 어렵다.
- 무엇보다 해당 microservice가 어떤 microservice를 부르는 지 전부 아는 것이 불가능에 가깝다.
- 따라서 분산된 api 로그들을 전부 하나로 통합해서 trace를 하게끔 하는 작업이 필요하다.
 - front end에서 trace context에 traceId를 보내면, 모든 microservice에서 해당 traceId를 계속 보내서 사용한다.
 - spring에서 logback으로 기록하면, 같은 host에서 돌아가는 별도 프로세스(elasticsearch??) 다 모은다. 그럼 tracing data를 nosql DB에 넣는다.
 - 그런데 이러한 로깅이 전부 손으로 우리가 만들어야 한다.
 - 제대로 다루지 못하면 기록되지 않는 api들이 생긴다.
 - 로그 추적용도의 빅 nosql은 유지비가 많이 든다.
  - 따라서 1000개 중 1개의 request만 넣는 1/1000 sampling을 하기도 한다. 그러나 샘플링 비율이 낮을수록 디버그가 힘들다.
-  trace가 너무 큰 것도 문제다. 
-  UI에 다 담기지 않는다.
 - spring cloud sleth + Zipkin을 많이 사용한다.



<img src="/image/distributed-tracing1.jpg" >
<img src="/image/distributed-tracing2.jpg" >




<br >

- MSA는 cloud vm 환경에서 이뤄진다.
- cloud vm에선 vm회사가 전부 칼같이 잘라서 다른 회사에 팔 수 있다.
  - 따라서 좀 싸고 traffic에 따라 가격을 받아 합리적이다.
  - 그런데 실제로는 칼같이 resource를 자르는 게 어렵다. 네트워크 대역폭, 인터널 버스 등 통제불가능한 내부 요인과도 연관이 있기 떄문이다.
  - 따라서 정말로 엄청나게 빠른 속도를 요구하는 산업에서도 사용되기 어렵다.
  - 특히 게이밍, 동영상 스트리밍, 트레이딩 시스템에선 안된다.
  - 또한 해당 host의 다른 vm이 뚫리면 내 vm도 뚫려버린다. 보안문제가 있다.
  - 특히 금융, 헬스케어, 공공, 국방은 쓸수가 없다.
  - 해당 방식은 multi-tenancy라고 부른다.
  - 아마존 elastic comput cloud(EC2) instnace가 그 예시다.
- multi-tenant가 아니면 single-tenant다. server 하나 전체를 하나의 회사만이 사용한다.
  - 대신 돈을 더 많이 내야 한다. 효율적으로 잘라서 다른 회사에 팔 수 있는 resource가 없기 때문이다.
  - AWS 전용 EC2 instance가 그 예시다.


<br >

- 서버조차 없는 serverless 배포도 있다.
  - 동영상 스트리밍 vod가 있다고 해보자.
  - 티켓을 사는데 1달에 딱 하루 1번이 열리고, 해당 시간에만 request가 몰린다.
  - 그런데 cloud vm에 올리면 no traffic인데도 돈을 내야 한다!
  - load balancing까지 하면 돈을 더블로 낸다!

<img src="/image/cloud-vm-drawback-cost.jpg" >

- 다른 예시를 들어보자. 어떤 보고서를 파는 회사가 한달에 한번만 cron으로 pdf를 보낸다고 해보자.
  - 그때를 위해 cloud vm을 쓰는 인프라 비용은 너무 비싸다.
  - 또한 개발비용도 너무 비싸다. HTTP request-response 받고, 배포하고...
- 이런 이유로 비용을 줄이기 위해 매우 가끔씩 일어나는 event를 Function as a service로 쓴다.
  - 이것은 전형적인 request-response가 아니라 event-driven이다.
  - event가 trigger 되면 api gateway를 타고 온다.
  - api gateway가  특정 event를 처리하기 위한 java 소스를 packaging해 jar로 만다.
  - 그걸 하드웨어에 바로 deploy하여 실행시켜 원하는 서비스를 발동시킨다.

<img src="/image/serverless.jpg" >

- 만약 event가 많이 일어나는 경우면 cloud 제공자가 scale out해준다.


<img src="/image/serverless2.jpg" >


- 보통 가격정책은 달마다의 요청 건수, 실행시간, 메모리다. 
  - 따라서 request가 없으면 비용이 0이다.
  - 대신 트래픽이 골고루 오게 되면 과금이 엄청나게 비싸질 수 있다.
  - 또한 business logic이 복잡해질수록 메모리를 잡아먹고 실행시간도 많이 걸려 과금이 비싸진다.
- 퍼포먼스를 예측하기 어렵다는 것도 단점이다.
  - latency가 중요하면 쓸 수가 없다.
- 또한 multi-tenant라 보안이 문제다.
- 보통 AWS 람다를 사용하거나, GCP의 cloud function을 사용한다.



<br >

- 사실 요새는 vm이 아니라 컨테이너를 사용하는 추세다.
- vm을 기피하게 된 근본적인 이유는 dev와 prod 환경이 다른 경우가 대부분이라서다.
  - dev에서는 작동하는데, prod에서는 안돌아가버린다.
- 그래서 dev를 prod와 유사한 vm에 만들수도 있다.
  - 그러나 매 microservice마다 vm에 넣으면 vm이 하나 더 추가되어 느려진다. 
  - 그런데 모든 microservice마다 vm에 따른 os가 추가되니 매우 느려진다.

<img src="/image/vm-drawback.jpg" >


- vm에서도 환경을 묶어 image화하려는 시도가 있었다. 그러나 cloud vender마다 다르다는 것이다. 
  - 서로 다른 vender의 image를 통합하는 것도 어렵다.
  - 거기다 cloud마다 다르게 운용한다면, 더더욱 통합이 어렵다.
- 따라서 container가 도입되었다. MSA는 대부분 container를 제공하는 Docker와 같이 간다.
- containter image는 jar binary code, run command, dependencies로 OS에 무관하게 작동하게끔 최소한으로 이뤄진다. 
- 각 container는 파일시스템, runtime, network interface를 독립적으로 가지지만, OS kernel은 모두 공유된다.


<img src="/image/container.jpg" >


- 다만 Docker를 사용전에 해결해야되는 게 있다.
- infra layer와 container layer 간의 문제다.
  - 실제 infra 수준에서 server와 container가 말린 곳을 트래픽에 따라 직접 조절해야 한다.
  - 새로운 api가 도입된 microservice를 반영하기 위해 해당 api가 있는 microservice가 말린 container를 찾아야 한다.
  - 이걸 위해서 containers orchestration layer를 추가한다. 
- 여기서 쿠버네티스가 등장했다.


<img src="/image/container-infra1.jpg" >
<img src="/image/container-infra2.jpg" >
<img src="/image/container-infra3.jpg" >

