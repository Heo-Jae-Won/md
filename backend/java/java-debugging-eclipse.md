## <span style="color:#802548">_debugging 걸기_</span>
- line에 toggle breakpoint를 건다
- 디버깅모드로 실행한다.



## <span style="color:#802548">_필요없는 package 무시하기_</span>
- 자바 관련 package 내부구현은 볼 필요가 없다. 내가만든 함수만 보면 된다.
- window - preference - debug - step filtering에서 자바 api 관련 package를 모두 체크해준다.


<img src="/image/debuggin-step-filtering.png" />

## <span style="color:#802548">_if문 조건에 따른 디버깅_</span>
- if문이 true인 경우에만 값을 보고 싶을 수 있다. 
- 그러한 경우 if문에 breakpoint를 걸고 breakpoint properties를 들어간다.
- conditional을 체크하고 true일 때만 breakpoint가 걸리게 할 수 있다.
- 그렇게 하면 for문을 도는 경우, if문이 true이전에는 breakpoint가 걸리지 않고, true가 될 때 걸린다.
- true인 조건이 끝나고 resume 시 for문을 index마다 돌지 않고 그대로 모든 연산을 완료하고 종료된다.
- 다만 step over시에는 for문을 index마다 여전히 돈다.


## <span style="color:#802548">_특정 값 추적하여 확인_</span>
- variables에는 너무 많은 변수가 있다. 
- 따라서 내가 원하는 특정 변수만 확인하고 싶다면 expression에 추가해서 사용한다.


<img src="/image/not-all-variables-want-expression.png" />



## <span style="color:#802548">_for문 확인_</span>
- for문이 선언된 곳에도 breakpoint를 걸 수 있다.
  - for문에 breakpoint를 걸고 resume을 누르면 해당 index인 경우의 연산이 종료되고, 다음 index에 breakpoint가 걸린다.
- for문이 선언된 곳에는 breakpoint properties에서 conditional을 걸 수 없다. i 변수가 인식되지 않는다.
  - 그 아래에서 걸어줘야 한다. 보통 i == 2와 같은 형태로 index에 대해 거는 경우가 많다.
  - foreach문으로 하면 element == 3435L과 같이 조건을 주면 된다.
- 여기서도 마찬가지로 conditional을 지정했다면 resume을 누르면 for문의 남은 index마다 breakpoint가 걸리지 않는다.
  - 그 경우 모든 index를 한번에 연산을 끝내고 for문을 탈출한다.
- 그 외에 for문을 5번 돈 뒤에 breakpoint를 걸고 싶을 떄는 breakpoint properties에서 Hit count를 5로 설정한다.
  - 그럼 index 5번째가 아니라, index로 따지면 4번쨰를 다 돌고난 뒤 5번쨰 index, 6번째 element가 for문을 돌게 된다.
  - 여기서도 resume을 누르게 되면 남은 연산을 모두 한꺼번에 진행하여 for문을 탈출한다.
  - 만약 for문 내 연산이 필요없어 skip하고 싶다면 hitcount를 for문이 가질 수 있는 횟수보다 더 높게 주면 된다. 
  - 사실 그보다 좋은건 그냥 for문 연산이 끝난 이후에 breakpoint를 거는 것이다.



## <span style="color:#802548">_DB 안타고 값만 확인하기_</span>
- DB로 들어가는 method에서 step into를 하고, 변수명만 살펴본다.
- 그 뒤에 terminate 시키면 변수가 제대로 들어오는 지는 확인하고, DB call은 호출하지 않을 수 있다.


