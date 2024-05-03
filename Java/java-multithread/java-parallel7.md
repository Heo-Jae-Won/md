- 이전의 helloWorld Service에서 exception이 났을 때 handling하는 method인 handle()만 추가했다.
```java
public String helloWorld_3_async_calls_handle() {
    startTimer();
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello());
    CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world());
    CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
        delay(1000);
        return " HI CompletableFuture!";
    });

    String hw = hello
            .handle((result, e/**exception */) -> { 
                log("result is : " + result);
                return "";
            }) //exception이 나면 "" return
            .thenCombine(world, (h, w) -> h + w)  // " world! "
            .thenCombine(hiCompletableFuture, (previous, current) -> previous + current) // " world! HI CompletableFuture!"
            .thenApply(String::toUpperCase)
            .join();

    timeTaken();

    return hw;
}
```


- test case는 아래와 같다.
- hello()에만 exception을 던지고, world()는 정상 return시킨다. 그러면 assertEquals는 true다.
- 또한 Exception이 일어난 지점에서 log를 찍은 게 나온다.
```java
@ExtendWith(MockitoExtension.class)
class CompletableFutureHelloWorldExceptionTest {


    @Mock
    HelloWorldService helloWorldService = mock(HelloworldService.class);

    @InjectMocks
    CompletableFutureHelloWorldException hwcfe;
    
    @Test
    void helloWorld_3_async_calls_handle() {

        //given
        //exception을 던지는 상황
        when(helloWorldService.hello()).thenThrow(new RuntimeException("Exception Occurred"));

        //정상 return인 상황
        when(helloWorldService.world()).thenCallRealMethod();

        //when
        String result = hwcfe.helloWorld_3_async_calls_handle();

        //then
        String expectedResult = " WORLD! HI COMPLETABLEFUTURE!";
        assertEquals(expectedResult, result);
    }
}
```

- 이번엔 아래와 같이 handle은 매 future마다 걸어줄 수 있다.
```java
public String helloWorld_3_async_calls_handle() {
    startTimer();
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello());
    CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world());
    CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
        delay(1000);
        return " HI CompletableFuture!";
    });

    String hw = hello
            .handle((result, e/**exception */) -> { // this gets invoked for both success and failure
                log("Exception is : " + e.getMessage());
                return ""; //exception이 발동하면 ""로 return
            })
            .thenCombine(world, (h, w) -> h + w) // (first,second)
            .handle((result, e) -> {
                log("Exception Handle after world : " + e.getMessage());
                return "";
            })
            .thenCombine(hiCompletableFuture, (previous, current) -> previous + current)
            .thenApply(String::toUpperCase)

            .join();

    timeTaken();

    return hw;
}
```

- 둘 다 exception이 나게 되면 아래와 같이 " HI COMPLETABLEFUTURE!"만 남게 되어 true가 된다.
```java
@Test
void helloWorld_3_async_calls_handle_2() {

    //given
    when(helloWorldService.hello()).thenThrow(new RuntimeException("Exception Occurred"));
    when(helloWorldService.world()).thenThrow(new RuntimeException("Exception Occurred"));

    //when
    String result = hwcfe.helloWorld_3_async_calls_handle();

    //then
    String expectedResult = " HI COMPLETABLEFUTURE!";
    assertEquals(expectedResult, result);
}
```

- 이번엔 아래와 같이 정상 return만 하게 하자.
- 당연히 true인 줄 알았는데, false다.
```java
@Test
void helloWorld_3_async_calls_handle_2() {

    //given
     when(helloWorldService.hello()).thenCallRealMethod();
     when(helloWorldService.world()).thenCallRealMethod();

    //when
    String result = hwcfe.helloWorld_3_async_calls_handle();

    //then
    String expectedResult = "HELLO WORLD! HI COMPLETABLEFUTURE!";
    assertEquals(expectedResult, result);
}
```

- handle()은 exception이 있든 없든 무조건 호출되기 때문이다.
- 따라서 service를 아래와 같이 exception이 null인지 확인하게 바꿔줘야 한다.
- 그러고 나서 test를 다시 진행하면 true가 된다.
```java
public String helloWorld_3_async_calls_handle() {
        startTimer();
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello());
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world());
        CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
            delay(1000);
            return " HI CompletableFuture!";
        });

        String hw = hello
                .handle((result, e) -> { // this gets invoked for both success and failure
                    log("result is : " + result);
                    if (e != null) {
                        log("Exception is : " + e.getMessage());
                        return "";
                    }
                    return result;

                })
                .thenCombine(world, (h, w) -> h + w) // (first,second)
                .handle((result, e) -> { // this gets invoked for both success and failure
                    log("result is : " + result);
                    if (e != null) {
                        log("Exception Handle after world : " + e.getMessage());
                        return "";
                    }
                    return result;
                })
                .thenCombine(hiCompletableFuture, (previous, current) -> previous + current)
                .thenApply(String::toUpperCase)

                .join();

        timeTaken();

        return hw;
    }
```


- handle()을 사용하면 코드가 좀 많이 더러워진다.
- 그래서 exceptionally()를 사용하는 게 좋다.
- 코드의 가독성을 더럽히는 exception null check if문을 전부 걷어낼 수 있다.
- exceptionally는 exception이 일어날 때만 호출되기 때문에 handle()과 같이 정상 return에도 값을 덮어씌우는 상황을 걱정을 할 필요가 전혀 없다.
```java
public String helloWorld_3_async_calls_exceptionally() {
    startTimer();
    CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello());
    CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world());
    CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
        delay(1000);
        return " HI CompletableFuture!";
    });

    String hw = hello
            .exceptionally((e) -> { // this gets invoked for both success and failure
                    log("Exception is : " + e.getMessage());
                return "";
            })
            .thenCombine(world, (h, w) -> h + w) // (first,second)
            .exceptionally((e) -> { // this gets invoked for both success and failure
                    log("Exception Handle after world : " + e.getMessage());
                    return "";
            })
            .thenCombine(hiCompletableFuture, (previous, current) -> previous + current)
            .thenApply(String::toUpperCase)

            .join();

    timeTaken();

    return hw;
}
```


- 실제로 test를 해보자.
- 모두 정상인 경우, helloWorld_3_async_calls_exceptionally()의 assert는 true가 되어야 한다.
- helloWorld_3_async_calls_exceptionally_2()의 assert도 true가 되어야 한다.
- 실제로 둘다 모두 true가 나온다. 
- exceptionally를 사용하면 정상 return일 때는 무시하고, exception이 날때만 호출됨이 증명된 것이다.
```java
@Test
void helloWorld_3_async_calls_exceptionally() {

    //given
     when(helloWorldService.hello()).thenCallRealMethod();
     when(helloWorldService.world()).thenCallRealMethod();

    //when
    String result = hwcfe.helloWorld_3_async_calls_handle();

    //then
    String expectedResult = "HELLO WORLD! HI COMPLETABLEFUTURE!";
    assertEquals(expectedResult, result);
}

@Test
void helloWorld_3_async_calls_exceptionally_2() {

    //given
    when(helloWorldService.hello()).thenThrow(new RuntimeException("Exception Occurred"));
    when(helloWorldService.world()).thenThrow(new RuntimeException("Exception Occurred"));

    //when
    String result = hwcfe.helloWorld_3_async_calls_exceptionally();

    //then
    String expectedResult = " HI COMPLETABLEFUTURE!";
    assertEquals(expectedResult, result);
}
```


- handle()과 exceptionally() 말고 whenComplete()도 있다.
- handle()과 exceptionally()는 whenComplete()는 exception이 난 경우에 기본값으로 바꿔 return하는 기능은 없다.
- whenComplete()는 BiConsumer 함수형 인터페이스를 parameter로 받는데, 해당 람다는 값을 return하지 않는다.
- 그런 이유로 사실 거의 쓰이지 않는 exception handling method다.
- whenComplete()도 handle()과 같이 정상 return이든, 아니든 무조건 발동한다. 또한 exception이 한번 발동하면 recover가 불가능하므로 thenCombine()을 찾아가는 게 아니라 그 다음 handle()/exceptionally()/whenComplete()를 찾아간다.
```java
public String helloWorld_3_async_whenComplete() {
        startTimer();
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello());
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world());
        CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
            delay(1000);
            return " HI CompletableFuture!";
        });

        String hw = hello
                .whenComplete((result, e) -> { // this gets invoked for both success and failure
                    log("result is : " + result);
                    if (e != null) {
                        log("Exception is : " + e.getMessage());
                        //return ""; return이 불가능하다. whenComplete는 recover 기능이 없다.
                    }
                })
                .thenCombine(world, (h, w) -> h + w) // (first,second)
                .whenComplete((result, e) -> { // this gets invoked for both success and failure
                    log("result is : " + result);
                    if (e != null) {
                        log("Exception Handle after world : " + e.getMessage());
                    }
                })
                .thenCombine(hiCompletableFuture, (previous, current) -> previous + current)
                .thenApply(String::toUpperCase)

                .join();

        timeTaken();
        return hw;
    }
```

- whenComplete()의 test case를 쓰면 아래와 같다.
- 처음 정상 return일 때는 true를 return해 test를 통과한다.
- 그러나 두번째 method에서는 exception을 하나라도 내면, whenCompelte()가 호출되고, 그 뒤부터는 실패경로의 method들(handle, exceptionally, whenComplete)만 호출된다. 
- 위에서는 whenComplete()만 계속 호출했으므로 exception이 throw되는 것으로 마무리된다.
- 따라서 두번째 case는 true도 false도 내지 않는다. exception이 나버려 test가 fail된다.
```java
@Test
void helloWorld_3_async_whenComplete() {

    //given
    when(helloWorldService.hello()).thenCallRealMethod();
    when(helloWorldService.world()).thenCallRealMethod();

    //when
    String result = hwcfe.helloWorld_3_async_whenComplete();

    //then
    String expectedResult = "HELLO WORLD! HI COMPLETABLEFUTURE!";
    assertEquals(expectedResult, result);
}


@Test
void helloWorld_3_async_whenComplete_2() {

    //given
    when(helloWorldService.hello()).thenThrow(new RuntimeException("Exception Occurred"));
    when(helloWorldService.world()).thenCallRealMethod();

    //when
    String result = hwcfe.helloWorld_3_async_whenComplete();

    //then
    String expectedResult = " HI COMPLETABLEFUTURE!";
    assertEquals(expectedResult, result);
}
```


- 따라서 true를 받아 test를 통과하려면 맨 마지막에는 value를 recover하는 handle() 혹은 exceptionally()가 필요하다.
- 기왕이면 더 간단한 exceptionally를 넣어준다. 
- 이제 맨마지막에 값을 ""로 recover했으므로 실제 결과가 " HI COMPLETABLEFUTURE!"가 되어 true로 test를 통과하게 된다.
```java
public String helloWorld_3_async_whenComplete() {
        startTimer();
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello());
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world());
        CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
            delay(1000);
            return " HI CompletableFuture!";
        });

        String hw = hello
                .whenComplete((result, e) -> { // this gets invoked for both success and failure
                    log("result is : " + result);
                    if (e != null) {
                        log("Exception is : " + e.getMessage());
                        //return ""; return이 불가능하다. whenComplete는 recover 기능이 없다.
                    }
                })
                .thenCombine(world, (h, w) -> h + w) // (first,second)
                .whenComplete((result, e) -> { // this gets invoked for both success and failure
                    log("result is : " + result);
                    if (e != null) {
                        log("Exception Handle after world : " + e.getMessage());
                    }
                }) //아래가 추가된 exceptionally. 이게 있으면 값이 recover된다.
                .exceptionally((e) -> { // this gets invoked for both success and failure
                    log("Exception Handle after world : " + e.getMessage());
                    return "";
                })
                .thenCombine(hiCompletableFuture, (previous, current) -> previous + current)
                .thenApply(String::toUpperCase)

                .join();

        timeTaken();
        return hw;
    }
```
