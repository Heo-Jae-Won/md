- CompletableFuture는 supplyAsync와 thenAccept가 있다.
- supplyAsync는 parameter로 supplier 람다식을 넣어야 한다.
  - 반환타입은 CompletableFuture(T)
- thenAccept는 parameter로 Consumer 람다식을 넣어야 한다.
  - 반환타입은 CompletableFuture(void)

- 일반 String return service는 아래와 같다.
```java
public class HelloWorldService {

    public  String helloWorld() {
        delay(1000);
        log("inside helloWorld");
        return "hello world";
    }
}
```

- 가장 간단한 CompletableFuture는 아래와 같이 만들 수 있다.
- 아래와 같이 만들면 result는 log에 찍히지 않는다.
- helloWorldService.helloWorld에서 1초를 쉬는데, main method는 이미 반환을 해서 종료되기 때문이다.
```java
public static void main(String[] args) {
        HelloWorldService helloWorldService = new HelloWorldService();

        //helloWorld는 string을 return
        CompletableFuture.supplyAsync(() -> helloWorldService.helloWorld()) //  runs this in a common fork-join pool
                .thenAccept((result) -> {//result는 helloWorld의 return value임
                    log("result " + result);
                })
        log("Done!");
    }
```

- 따라서 아래와 같이 2초를 쉬게 해주면 CompletableFuture.supplyAsync가 연산을 끝날 때까지 main method가 대기하기 때문에 result 로그가 찍히게 된다.
```java
public static void main(String[] args) {
        HelloWorldService helloWorldService = new HelloWorldService();

        //helloWorld는 string을 return
        CompletableFuture.supplyAsync(() -> helloWorldService.helloWorld()) //  runs this in a common fork-join pool
                .thenAccept((result) -> {//result는 helloWorld의 return value임
                    log("result " + result);
                })
        log("Done!");
        delay(2000);
    }
```

- 2초를 기다리게 할 필요 없이 연산이 끝나면 main method를 종료시키는 방법이 있다. 
- thread에서 썼던 것처럼 join을 하면 호출하면 된다.
```java
public static void main(String[] args) {
        HelloWorldService helloWorldService = new HelloWorldService();

        //helloWorld는 string을 return
        CompletableFuture.supplyAsync(() -> helloWorldService.helloWorld()) //  runs this in a common fork-join pool
                .thenAccept((result) -> {//result는 helloWorld의 return value임
                    log("result " + result);
                })
                .join();
        log("Done!");
    }
```


- thenAccept 외에도 thenApply를 쓰는 것도 가능하다.
- thenApply를 쓰게 되면 map()에서 data를 변경하듯 변경할 수 있다.
- 이런 식으로 method chaning을 계속 inovke하며 stream을 진행시키는 것을 pipeline이라고 한다.
  - supplyAsync
  - thenApply
  - thenAccep
```java
public static void main(String[] args) {
        HelloWorldService helloWorldService = new HelloWorldService();

        //helloWorld는 string을 return
        CompletableFuture.supplyAsync(() -> helloWorldService.helloWorld()) //  runs this in a common fork-join pool
        //CompletableFutuer.supplyAsync(hws::helloWorld)
                //.thenApply((result) -> result.toUpperCase())
                .thenApply(String::toUpperCase)
                .thenAccept((result) -> {//result는 helloWorld의 return value임
                    log("result " + result);
                })
                .join();
        log("Done!");
    }
```

- 이제 CompletableFuture를 Junit으로 test해보자.
- main method에서 test하는 것은 기업 규모에선 불가능하기 때문이다.
- 아래와 같이 thenAccept를 놔둔채로 helloWorld를 구성하면 error가 난다. thenAccep는 value를 consume해서 return type이 void다.
- join도 없애야 된다. join은 CompletableFuture가 끝난 뒤 generics type을 return한다.
```java
public class CompletableFutureHelloWorld {

    private HelloWorldService hws;

    public CompletableFutureHelloWorld(HelloWorldService helloWorldService) {
        this.hws = helloWorldService;
    }

    public CompletableFuture<String> helloWorld() {

        return CompletableFuture.supplyAsync(() -> helloWorldService.helloWorld()) //  runs this in a common fork-join pool
        //CompletableFutuer.supplyAsync(hws::helloWorld)
                //.thenApply((result) -> result.toUpperCase())
                .thenApply(String::toUpperCase)
             /*   .thenAccept((result) -> {
                    log("result " + result);
                })
                .join(); */
    }
}
```


- test는 일반 test와는 좀 다르게 구성한다.
- 검증을 thenAccept 안에서 진행한다. 
- 그런데 아래와 같이 join을 호출하지 않으면 어떤 값을 비교하든 true가 된다.
```java
class CompletableFutureHelloWorldTest {

    HelloWorldService hws = new HelloWorldService();
    CompletableFutureHelloWorld cfhw = new CompletableFutureHelloWorld(hws);

    @Test
    void helloWorld() {

        //given
        //when
        CompletableFuture<String> completableFuture = cfhw.helloWorld();

        //then
        completableFuture
                .thenAccept(s -> {
                    //assertEquals("hello world", s);
                    assertEquals("HELLO WORLD", s);
                })
    }
}
```


- 아래와 같이 join을 호출하면 false가 되어 exception이 떨어진다.
- 제대로 된 테스트가 진행된 것이다.
```java
class CompletableFutureHelloWorldTest {

    HelloWorldService hws = new HelloWorldService();
    CompletableFutureHelloWorld cfhw = new CompletableFutureHelloWorld(hws);

    @Test
    void helloWorld() {

        //given
        //when
        CompletableFuture<String> completableFuture = cfhw.helloWorld();

        //then
        completableFuture
                .thenAccept(s -> {
                    //assertEquals("hello world", s);
                    assertEquals("HELLO WORLD1", s);
                })
    }
}
```

- size까지 같이 보려면 아래와 같이 만들 수 있다.
```java
public CompletableFuture<String> helloWorld_withSize() {

    return CompletableFuture.supplyAsync(() -> helloWorldService.helloWorld())//  runs this in a common fork-join pool
            .thenApply((s) -> s.length() + " - " + s);
}
```

- thenCombine()은 별개의 CompletableFuture를 합칠 때 쓴다.
- service가 2개인 경우, 두 개를 병렬로 호출하여 하나의 서비스가 마무리 된 후에 다른 서비스를 호출하지 않게 된다. 그만큼 빨라진다.
- hello와 world를 호출하는 서비스를 합쳐보자.
```java
public class HelloWorldService {

    public  String hello() {
        delay(1000);
        log("inside hello");
        return "hello";
    }

    public  String world() {
        delay(1000);
        log("inside world");
        return " world!";
    }
}
```

- 실제 async하게 service를 호출하여 combine하려면 아래와 만든다.
- join을 까먹으면 안 된다. join을 까먹으면 아무것도 안된다.
- 그럼 hello와 world가 각각 1초가 걸리지만, 실제로 연산이 끝나는데는 1초 가량이 걸리는 것을 볼 수 있다.
- parallel하게 진행되기 때문이다. 뒤에 배울 thenCompose는 sequential하게 진행된다.
```java
public class CompletableFutureHelloWorld {

    private HelloWorldService hws;

    public CompletableFutureHelloWorld(HelloWorldService helloWorldService) {
        this.hws = helloWorldService;
    }

    public String helloWorld_multiple_async_calls() {
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello());
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world());

        String hw = hello
                .thenCombine(world, (h, w) -> h + w) // (first,second). 여기선 hello string과 world string
                .thenApply(String::toUpperCase)
                .join();

        return hw;
    }
}
```


- test case도 만들어보자.
- thenAccept()를 쓰지 않고 그냥 assertEquals를 호출하면 된다.
- 그 이유는 join을 호출해 CompletableFuture(String)이 아닌 String을 return했기 때문이다.
```java
@Test
void helloWorld_multiple_async_calls() {

    //given
    //when
    String hw = cfhw.helloWorld_multiple_async_calls();

    //then
    assertEquals("HELLO WORLD!", hw);

}
```


- 이젠 2개가 아니라 3개로 늘려보자.
- thenCombine을 2개로 늘려주면 된다.
```java
public String helloWorld_3_async_calls() {
        startTimer();
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello());
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world());
        CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
            delay(1000);
            return " HI CompletableFuture!";
        });

        String hw = hello
                .thenCombine(world, (h, w) -> h + w) // (first,second). hello world!
                .thenCombine(hiCompletableFuture, (previous, current) -> previous + current) //hello world! HI CompletableFuture!
                .thenApply(String::toUpperCase) //HELLO WORLD! HI COMPLETABLEFUTURE!
                .join();

        timeTaken();

        return hw;
    }
```

- test case는 아까와 완전히 동일하다.
```java
@Test
void helloWorld_3_async_calls() {

    //given
    //when
    String hw = cfhw.helloWorld_3_async_calls();

    //then
    assertEquals("HELLO WORLD! HI COMPLETABLEFUTURE!", hw);

}
```


- then combine과 비슷하게 thenCompose도 사용가능하다.
- 다만 sequential하게 진행된다.
```java
public class HelloWorldService {

    public CompletableFuture<String> worldFuture(String input) {
        return CompletableFuture.supplyAsync(()->{
            delay(1000);
            return input+" world!";
        });
    }

}
```

- hello를 호출하며 1초, worldFuture를 호출하며 1초가 걸린다.
- thenCompose와 thenCombine와 달리, 이전의 연산이 완료될 때까지 대기한다. 따라서 아래의 연산은 1초가 아니라 2초가 걸린다.
```java
public class CompletableFutureHelloWorld { 
    HelloWorldService hws = new HelloWorldService();

    public CompletableFuture<String> helloWorld_thenCompose() {

        CompletableFuture<String> helloWorldFuture = CompletableFuture.supplyAsync(hws::hello) //hello return
                .thenCompose(previous -> hws.worldFuture(previous))

        return helloWorldFuture;

    }
}
```

- 바꿔 쓰면 아래와 같이도 쓸 수 있다.
```java
public class CompletableFutureHelloWorld { 
    public CompletableFuture<String> helloWorld_thenCompose() {

        CompletableFuture<String> helloWorldFuture = CompletableFuture.supplyAsync(() -> this.hws.hello())
                .thenCompose(previous -> hws.worldFuture(previous))
                .thenApply(String::toUpperCase);

        return helloWorldFuture;

    }
}
```

- thenCompose의 test case는 아래와 같이 작성한다.
- 여기서 2초가 걸린다는 점을 확인할 수 있다.
```java
class CompletableFutureHelloWorldTest {

    HelloWorldService hws = new HelloWorldService();
    CompletableFutureHelloWorld cfhw = new CompletableFutureHelloWorld(hws);


    @Test
    void helloWorld_thenCompose() {

        //given
        //when
        startTimer();

        CompletableFuture<String> completableFuture = cfhw.helloWorld_thenCompose();

        //then
        completableFuture
                .thenAccept(s -> {
                    //assertEquals("hello world", s);
                    assertEquals("HELLO WORLD!", s);
                })
                .join();
        timeTaken();
    }
}
```





```java
class CompletableFutureHelloWorldTest {

    HelloWorldService hws = new HelloWorldService();
    CompletableFutureHelloWorld cfhw = new CompletableFutureHelloWorld(hws);


    @Test
    void helloWorld_thenCompose() {

        //given
        //when
        startTimer();

        CompletableFuture<String> completableFuture = cfhw.helloWorld_thenCompose();

        //then
        completableFuture
                .thenAccept(s -> {
                    //assertEquals("hello world", s);
                    assertEquals("HELLO WORLD!", s);
                })
                .join();
        timeTaken();
    }

    @Test
    @Disabled
    void helloWorld_complete() {

        //given
        //when
        startTimer();

        CompletableFuture<String> completableFuture = cfhw.complete("hello world!");

        //then
        completableFuture
                .thenAccept(s -> {
                    //assertEquals("hello world", s);
                    assertEquals("12 - HELLO WORLD!", s);
                })
                .join();
        timeTaken();


    }

    @Test
    void allOf() {

        //given

        //when
        String result = cfhw.allOf();

        //then
        assertEquals("Hello World", result);
    }

    @Test
    void anyOf() {

        //given

        //when
        String result = cfhw.anyOf();

        //then
        assertEquals("Hello World", result);
    }
}
```






```java
public class CompletableFutureHelloWorld {

    public String helloWorld_3_async_calls_log() {
        startTimer();
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello());
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world());
        CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
            delay(1000);
            return " HI CompletableFuture!";
        });

        String hw = hello
                // .thenCombine(world, (h, w) -> h + w) // (first,second)
                .thenCombine(world, (h, w) -> {
                    log("thenCombine h/w ");
                    return h + w;
                }) // (first,second)
                //.thenCombine(hiCompletableFuture, (previous, current) -> previous + current)
                .thenCombine(hiCompletableFuture, (previous, current) -> {
                    log("thenCombine , previous/current");
                    return previous + current;
                })
                //.thenApply(String::toUpperCase)
                .thenApply(s -> {
                    log("thenApply");
                    return s.toUpperCase();
                })
                .join();

        timeTaken();

        return hw;
    }

    public String helloWorld_3_async_calls_log_async() {
        startTimer();
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello());
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world());
        CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
            delay(1000);
            return " HI CompletableFuture!";
        });

        String hw = hello
                // .thenCombine(world, (h, w) -> h + w) // (first,second)
                .thenCombineAsync(world, (h, w) -> {
                    log("thenCombine h/w ");
                    return h + w;
                }) // (first,second)
                //.thenCombine(hiCompletableFuture, (previous, current) -> previous + current)
                .thenCombineAsync(hiCompletableFuture, (previous, current) -> {
                    this.hws.hello();
                    log("thenCombine , previous/current");
                    return previous + current;
                })
                //.thenApply(String::toUpperCase)
                .thenApplyAsync(s -> {
                    this.hws.hello();
                    log("thenApply");
                    return s.toUpperCase();
                })
                .join();

        timeTaken();

        return hw;
    }


    public String helloWorld_3_async_calls_custom_threadPool() {

        ExecutorService executorService = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

        startTimer();
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello(), executorService);
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world(), executorService);

        CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
            delay(1000);
            return " HI CompletableFuture!";
        }, executorService);

        String hw = hello
                // .thenCombine(world, (h, w) -> h + w) // (first,second)
                .thenCombine(world, (h, w) -> {
                    log("thenCombine h/w ");
                    return h + w;
                }) // (first,second)
                //.thenCombine(hiCompletableFuture, (previous, current) -> previous + current)
                .thenCombine(hiCompletableFuture, (previous, current) -> {
                    log("thenCombine , previous/current");
                    return previous + current;
                })
                //.thenApply(String::toUpperCase)
                .thenApply(s -> {
                    log("thenApply");
                    return s.toUpperCase();
                })
                .join();

        timeTaken();

        return hw;
    }

    public String helloWorld_3_async_calls_custom_threadpool_async() {

        ExecutorService executorService = Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());

        startTimer();
        CompletableFuture<String> hello = CompletableFuture.supplyAsync(() -> this.hws.hello(), executorService);
        CompletableFuture<String> world = CompletableFuture.supplyAsync(() -> this.hws.world(), executorService);

        CompletableFuture<String> hiCompletableFuture = CompletableFuture.supplyAsync(() -> {
            // delay(1000);
            return " HI CompletableFuture!";
        }, executorService);

        String hw = hello
                // .thenCombine(world, (h, w) -> h + w) // (first,second)
                .thenCombineAsync(world, (h, w) -> {
                    log("thenCombine h/w ");
                    return h + w;
                }, executorService) // (first,second)

                /*  .thenCombineAsync(world, (h, w) -> {
                      log("thenCombine h/w ");
                      return h + w;
                  }) // with no executor service as an input*/
                //.thenCombine(hiCompletableFuture, (previous, current) -> previous + current)
                .thenCombineAsync(hiCompletableFuture, (previous, current) -> {
                    log("thenCombine , previous/current");
                    return previous + current;
                }, executorService)
                //.thenApply(String::toUpperCase)
                .thenApply(s -> {
                    log("thenApply");
                    return s.toUpperCase();
                })
                .join();

        timeTaken();

        return hw;
    }

    public CompletableFuture<String> helloWorld_thenCompose() {

        CompletableFuture<String> helloWorldFuture = CompletableFuture.supplyAsync(() -> this.hws.hello())
                .thenCompose(previous -> hws.worldFuture(previous))
                //.thenApply(previous -> helloWorldService.worldFuture(previous))
                .thenApply(String::toUpperCase);

        return helloWorldFuture;

    }

    public String allOf() {
        startTimer();

        CompletableFuture<String> cf1 = CompletableFuture.supplyAsync(() -> {
            delay(1000);
            return "Hello";
        });

        CompletableFuture<String> cf2 = CompletableFuture.supplyAsync(() -> {
            delay(2000);
            return " World";
        });

        List<CompletableFuture<String>> cfList = List.of(cf1, cf2);
        CompletableFuture<Void> cfAllOf = CompletableFuture.allOf(cfList.toArray(new CompletableFuture[cfList.size()]));
        String result = cfAllOf.thenApply(v -> cfList.stream()
                .map(CompletableFuture::join)
                .collect(joining())).join();

        timeTaken();

        return result;

    }

    public String anyOf() {
        startTimer();

        CompletableFuture<String> db = CompletableFuture.supplyAsync(() -> {
            delay(1000);
            log("response from db");
            return "Hello World";
        });

        CompletableFuture<String> restApi = CompletableFuture.supplyAsync(() -> {
            delay(2000);
            log("response from restApi");
            return "Hello World";
        });

        CompletableFuture<String> soapApi = CompletableFuture.supplyAsync(() -> {
            delay(3000);
            log("response from soapApi");
            return "Hello World";
        });

        List<CompletableFuture<String>> cfList = List.of(db, restApi, soapApi);
        CompletableFuture<Object> cfAllOf = CompletableFuture.anyOf(cfList.toArray(new CompletableFuture[cfList.size()]));
        String result =  (String) cfAllOf.thenApply(v -> {
            if (v instanceof String) {
                return v;
            }
            return null;
        }).join();

        timeTaken();
        return result;
    }


    public String helloWorld_1() {

        return CompletableFuture.supplyAsync(() -> hws.helloWorld())//  runs this in a common fork-join pool
                .thenApply(String::toUpperCase)
                .join();

    }

    public CompletableFuture<String> complete(String input) {

        CompletableFuture<String> completableFuture = new CompletableFuture();
        completableFuture = completableFuture
                .thenApply(String::toUpperCase)
                .thenApply((result) -> result.length() + " - " + result);

        completableFuture.complete(input);

        return completableFuture;

    }
}
```