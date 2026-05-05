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



## <span style="color:#802548">_debugging 걸기_</span>
- line에 toggle breakpoint를 건다
- 디버깅모드로 실행한다.

## <span style="color:#802548">_breakpoint 걸린 곳 확인하기_</span>
- 왼쪽 사이드바 메뉴의 run and debug에서 breakpoint가 확인가능하다.
  - 맨 아래에 있다.

- 그 외 watch, variables, call stack가 있다.
- variables는 너무 변수가 많으니 보고 싶은 변수를 watch에 넣자.


## <span style="color:#802548">_step into에 필요없는 package 무시하기_</span>
- step into로 들어갈 떄, 자바 관련 package 내부구현은 볼 필요가 없다. 
- 내가만든 함수만 보면 된다. workspace setting에 아래와 같이 setting해준다.

```sh
 "java.debug.settings.stepping.skipClasses": [
    "org.springframework.*",
    "$JDK",
    "javax.*",
    "java.*",
    "com.fasterxml.*",
    "com.sun.*",
    "org.apache.tomcat.*",
    "org.hibernate.*"
]
```
- step into에서만 가능하다. call stack 자체를 무시하는 것은 안돼서 step over로 계속 지나쳐야한다.

## <span style="color:#802548">_if문 conditional 디버깅_</span>
- if문이 true인 경우에만 값을 보고 싶을 수 있다. 
- 그러한 경우 edit breakpoint를 걸고 true or false 조건을 준다.


```java
if (projections.isEmpty()) { //edit breakpoint에
    throw new RuntimeException("No such movie + review");
}
```

- expression은 간단하게 true or false다.
- 아래처럼 쓰면 안 된다.

```sh
projections.isEmpty() == true ## 이렇게 쓰면 안됨.
```

- true나 false로만 쓴다.

```sh
true
```

## <span style="color:#802548">_for문 expression conditional 디버깅_</span>
- for문이 선언된 곳에도 breakpoint를 걸 수 있다.
  - for문에 breakpoint를 걸고 continue를 누르면 해당 index인 경우의 연산이 종료되고, 다음 index에 breakpoint가 걸린다.
  - for문의 로직을 일일이 확인하기보단, 로직이 끝난 뒤 값을 보고 싶다면, 그냥 continue로 쭉쭉 누르면 된다는 의미다.
- 하지만 conditional을 걸면 condition에 맞을 떄만 debugger가 걸린다.

- for문이 선언된 곳에는 breakpoint properties에서 conditional을 걸 수 없다. i 변수가 인식되지 않는다.
  - 그 아래에서 걸어줘야 한다. 보통 i == 2와 같은 형태로 index에 대해 거는 경우가 많다.


```java
i == 2
```

- 하지만 범위에 대해서 활용하고 싶다면 아래처럼 범위로 걸어준다.
- 프로그래밍 java 언어와 똑같은 조건식 형태로 걸어주자. 

```java
0 < i && i < 3
// 0 < i < 3 --->수식으로 걸면 안 된다.
```

- foreach문을 쓸 경우에 index 기준으로 breaking을 걸고 싶다면?
- for문의 선언줄에 걸지 않고 그 아래 줄에 conditional break를 걸어준다.

```java
/* 여기다 걸면 안됨! */for (ReviewDTO.ReviewResponse reviewResponse : reviews) {
/* 그 아래에다가 걸어야됨. */    if (reviewResponse.reviewNum() == null &&
            reviewResponse.reviewDate() == null &&
            reviewResponse.reviewText() == null &&
            reviewResponse.reviewerNickName() == null &&
            reviewResponse.score() == null) {
        reviews = Collections.emptyList();
    }
}
```

- 아래처럼 걸어주면 2번째 element에서 breaking point가 발동하게 된다.
- conditional 조건은 아래와 같이 적어준다.

```java
reviews.indexOf(reviewResponse) == 1
```

- foreach문으로 값 기준으로 하고 싶다면 아래와 같다.

```java
reviewResponse.reviewNum() != null
```

- if문의 조건이 여러개가 되어 아래로 내려가면, 그 아래 조건이 시작되는 곳에 breakpoint를 걸자.
- 따라서 기존의 줄은 조건이 아무것도 없어 breakpoint가 발동되지 않는다.

```java
/* 여기다 걸면 안됨! */for (ReviewDTO.ReviewResponse reviewResponse : reviews) {
/* unverified breakpoint */    if (
/* 여기다가  breakpoint 해줘야.. */           reviewResponse.reviewNum() == null &&
            reviewResponse.reviewDate() == null &&
            reviewResponse.reviewText() == null &&
            reviewResponse.reviewerNickName() == null &&
            reviewResponse.score() == null) {
        reviews = Collections.emptyList();
    }
}
```


## <span style="color:#802548">_for문 hit count conditional 디버깅_</span>

- hit count 기준으로 breakpoint를 걸 수도 있다.

```java
10 → Breaks only on the 10th hit.
>= 5 → Breaks on the 5th hit and onwards.
% 3 == 0 → Breaks every 3rd time.
```

- 근데 몇 번 걸리고 부터는 먹통이다.
- 그냥 쓰지 말고 expression으로 쓰는 게 좋을 듯 하다.


## <span style="color:#802548">_for문이 아닌 stream() expression conditional 디버깅_</span>

```java
List<ReviewDTO.ReviewResponse> reviews = projections.stream().map(ReviewDTO.ReviewResponse::TODTO).toList();
```

```java
List<ReviewDTO.ReviewResponse> reviews = new ArrayList<>();
projections.forEach(projection -> {
    ReviewDTO.ReviewResponse response = ReviewDTO.ReviewResponse.TODTO(projection);
    System.out.println("Debug: " + response); // Add a breakpoint here
    reviews.add(response);
});
```




## <span style="color:#802548">_watch- 특정 값 추적하여 확인_</span>
- variables에는 너무 많은 변수가 있다. 
- 따라서 내가 원하는 특정 변수만 확인하고 싶다면 watch에 추가해서 사용한다.


## <span style="color:#802548">_step over, step into, step out, continue_</span>
- 기본적으로 파고 들 이유가 없는 로직이라면 step over를 사용한다.
- 다시말해 단순히 계산된 결과 값만 필요하면 step over를 사용한다.
  - getter, setter 용도의 method 들은 step over를 사용한다.
  - step into를 하면 쓸데없이 해당 class로 이동되어 불편함만 야기된다.
- 만약 debugging에 도움되지 않는 jdk, spring, jpa 등의 class로 debugging이 이동했다면 step out으로 빠져나오자.
- 거긴 어차피 내가 고칠 수 있는 영역이 아니니까 그냥 무시해야 한다.

- continue는 로직을 진행하며 다음 breaking point까지 이동하게 한다.
- 만약 breaking point가 한개라면, 그냥 바로 디버깅모드가 끝난다.
- for문안에서 debugging이 걸린경우, continue를 누르면 for문을 1번 도는 행위를 수행한다.
  - step over등으로 for문 안으로 들어갔다고 해도, debugging이 걸려있지 않으면, continue를 누르면 모든 for문의 순회를 끝마친다.
  - 따라서 for문안의 변수 값 변화를 확인하려면 안에다가 반드시 debugger를 찍고 continue를 눌러야 한다.




## <span style="color:#802548">_DB 안타고 값만 확인하기_</span>
- DB로 들어가는 method에서 step into를 하고, 변수명만 살펴본다.
- 그 뒤에 terminate 시키면 변수가 제대로 들어오는 지는 확인하고, DB call은 호출하지 않을 수 있다.

## <span style="color:#802548">_JPA proxy 객체 값 확인하기_</span>
- JPA 객체는 proxy로 이뤄져 있어 값을 확인하는 게 불가능하다.
- 따라서 DTO로 변환한 뒤에 그 값을 확인해야 한다.

