- exception 처리도 test를 해보자.
- 이전의 ReviewService와 ProductInfoService, ServiceProductSerivce에 exceptionally() method chaning을 추가한다.
```java
public Product retrieveProductDetailsWithInventory_approach2(String productId) {

    startTimer();
    CompletableFuture<ProductInfo> cfProductInfo = CompletableFuture.supplyAsync(() -> productInfoService.retrieveProductInfo(productId))
            .thenApply((productInfo -> {
                productInfo.setProductOptions(updateInventoryToProductOption_approach2(productInfo));
                //  productInfo.setProductOptions(updateInventoryToProductOption_approach3(productInfo));
                return productInfo;
            }));

    CompletableFuture<Review> cfReview = CompletableFuture.supplyAsync(() -> reviewService.retrieveReviews(productId))
            .exceptionally((ex) -> {
                log("Handled the Exception in review Service : " + ex.getMessage());
                return Review.builder() //기본값을 적용. assertNotNull test 시 null이 아니게 되는 문제가 생김.
                        .noOfReviews(0).overallRating(0.0)
                        .build();
            });

    Product product = cfProductInfo
            .thenCombine(cfReview, (productInfo, review) -> new Product(productId, productInfo, review))
            .whenComplete((prod, ex) -> { //exception이 나면 기본값 recover X
                log("Inside whenComplete : " + prod + "and the exception is " + ex);
                if (ex != null) {
                    log("Exception in whenComplete is : " + ex);
                }
            })
            .join(); // blocks the thread
    timeTaken();
    return product;
}
```


- test에 필요한 service를 DI로 부를 땐 new가 아니라 @InjectMocks로 불러온다.
- @InjectMocks로 호출한 service에 필요한 DI는 @Mock으로 불러온다.
```java
@ExtendWith(MockitoExtension.class)
class ProductServiceUsingCompletableFutureExceptionTest {
    @Mock
    ProductInfoService pisMock;
    @Mock
    ReviewService rssMock;
    @Mock
    InventoryService isMock;

    @InjectMocks
    ProductServiceUsingCompletableFuture pscf;
}
```

- 우선 data부터 셋팅한다.
- data를 select하는 service를 호출할 때 이전에 만든 mock 객체를 활용한다.
- data setting 시 나는 exception을 test하기 위해 thenCallRealMethod와 thenThrow를 적절히 섞어준다.
```java
@Test
void retrieveProductDetails_reviewServiceError() {

    //given
    String productId = "ABC123";
    when(pisMock.retrieveProductInfo(any())).thenCallRealMethod();
    when(rssMock.retrieveReviews(any())).thenThrow(new RuntimeException("Exception Occurred"));
    when(isMock.retrieveInventory(any(ProductOption.class))).thenCallRealMethod();
}
```

- exception이 나도 review는 기본값이 있기 때문에 null은 아니다.
- 따라서 review의 size를 비교하는 로직을 추가해야 제대로 된 test가 가능하다.
```java
@Test
void retrieveProductDetails_reviewServiceError() {

    //given
    String productId = "ABC123";
    when(pisMock.retrieveProductInfo(any())).thenCallRealMethod();
    when(rssMock.retrieveReviews(any())).thenThrow(new RuntimeException("Exception Occurred"));
    when(isMock.retrieveInventory(any(ProductOption.class))).thenCallRealMethod();


    //when
    Product product = pscf.retrieveProductDetailsWithInventory_approach2(productId);

    //then
    assertNotNull(product);
    assertTrue(product.getProductInfo().getProductOptions().size() > 0);
    product.getProductInfo().getProductOptions().forEach(productOption -> {
        assertNotNull(productOption.getInventory());
    });
    assertNotNull(product.getReview());
    assertEquals(0, product.getReview().getNoOfReviews());

    long count = product.getProductInfo().getProductOptions().stream()
            .count();
    System.out.println("count : "+count);
}
```

- error exception이 제대로 던져지는 지도 test가 가능하다.
- Assertions.assertThrows가 그러한 역할을 한다.
- 우리는 retrieveProductInfo()를 호출하며 runtimeException을 뱉고, test가 true로 처리된다. 
- pscf.retrieveProductDetailsWithInventory_approach2() 안에 pisMock.retrieveProductInfo이 있기 때문이다.
```java
@Test
void retrieveProductDetails_productInfoServiceError() {

    //given
    String productId = "ABC123";
    when(pisMock.retrieveProductInfo(any())).thenThrow(new RuntimeException("Exception Occurred"));
    when(rssMock.retrieveReviews(any())).thenCallRealMethod();
    //when(isMock.retrieveInventory(ans())).thenCallRealMethod(); 쓰지 않는 mock을 쓰게 되면 unneccesaryStubbingException 에러가 뜬다. 주석처리

    //then
    // exception이 던져졌는지 확인함. 첫번쨰 인자는 던져진 exception, 두번째 인자는 해당 exception을 던진 서비스를 람다로 호출
    Assertions.assertThrows(RuntimeException.class, ()->pscf.retrieveProductDetailsWithInventory_approach2(productId));

}
```


- CompletableFuture를 쓸 때는 common fork/join pool을 사용한다.
- 따라서 아래와 같이 thenCombine, thenApply에 log를 넣고 thread 이름을 확인해보면 ForkJoinPool.commonPool로 명시되어있다.
```java
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
```

- 문제는 api call이 많아지는 경우, thread 고갈 문제가 일어날 가능성이 존재한다는 것이다.
- 또한 시간이 오래 걸리는 작업을 진행하면 Thread가 점거당하게 된다.
- 따라서 별개의 Fork/Join Pool을 만들어서 진행해야 할 필요가 있다.
- 그 때 ExecutorService를 활용한다.
- 그리고 CompletableFuture를 만들 때 두번째 인자로 만들어둔 ExecucotrService를 집어넣는다.
- 그럼 ForkJoinPool.commonPool이 아닌 다른 thread명이 보이게 된다.
```java
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
```

- async하게 결과를 가져오는 것은 supplyAsync(), runAsync()에서 일어나는 작업이지 그 아래 chainingMethod에선 그렇지 않다.
  - supplyAsync는 return type이 있고, runAsync()는 return type이 없다.
- thenCombine, thenCombine, thenApply 등의 method chaining은 모두 같은 Thread에서 여태까지 일어난다. 그런데 이것도 다른 Thread에서 일으키는 것도 가능하다. 
- 다만 context switching 비용이 커서 간단하게 CPU만 이용해서 합치는 작동에 대해선 굳이 그렇게 하지 않는 것 뿐이다.
- 만약 thenCombine 작업에서 I/O가 오래걸려 async 옵션이 필요한 경우는, 다른 thread에 맡길 수 있다. 
- 그냥 thenCombien, thenApply, thenCompose, thenAccept 등의 method명에 Async만 추가해주면 된다.
```java
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
```


- 마찬가지로 custom ThreadPool에서도 async하게 진행할 수 있다.
```java
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
```