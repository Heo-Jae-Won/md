- 자바는 Thread를 통해 동시성을 달성한다.
- Threading을 지원하는 도구로 아래와 같은 것들이 있다. atomic package는 예외다.
  - synchronized
  - Reentrant Lock
  - condition
  - Atomic package



- 자바는 병행성도 있다.
- 병행성은 parallelism인데, task 한 개를 subtask로 나눈다 (fork)
- 그 뒤 subtask의 결과를 모은다 (join)
- concurrency와 parallelism 모두 결과를 빠르게 얻어오는 방법이지만, 방식이 다르다.
- Stream api에서는 parallelStream()가 멀티스레딩과 비슷한 기능을 담당한다.


<img src="/image/parallelism.jpg" />

- parallelStream을 쓰는 간단한 예시는 아래와 같다.
```java
List<String> namesList = List.of("Bob", "Jamie","Jill","Rick");

List<String> namesListUpperCase = namesList.parallelStream()
                                            .map(String::toUpperCase)
                                            .collect(Collectors.toList());
```

- productInfoService를 실행하면 1초가 걸린다.
- ReviewService 또한 실행하면 1초가 걸린다.
- 현재 ProductService는 parallel이 아니므로 Blocking I/O로 인해 2초가 걸리게 된다.
```java
public class ReviewService {
    public Review retrieveReviews(String productId) {
        delay(1000);
        LoggerUtil.log("retrieveReviews after Delay");
        return new Review(200, 4.5);
    }
}

public class ProductInfoService {

    public ProductInfo retrieveProductInfo(String productId) {
        delay(1000);
        List<ProductOption> productOptions = List.of(new ProductOption(1, "64GB", "Black", 699.99),
                new ProductOption(2, "128GB", "Black", 749.99));
        LoggerUtil.log("retrieveProductInfo after Delay");
        return ProductInfo.builder().productId(productId)
                .productOptions(productOptions)
                .build();
    }
}

public class ProductService {
    private ProductInfoService productInfoService;
    private ReviewService reviewService;

    public ProductService(ProductInfoService productInfoService, ReviewService reviewService) {
        this.productInfoService = productInfoService;
        this.reviewService = reviewService;
    }

    public Product retrieveProductDetails(String productId) {
        stopWatch.start();

        ProductInfo productInfo = productInfoService.retrieveProductInfo(productId); // blocking call
        Review review = reviewService.retrieveReviews(productId); // blocking call

        stopWatch.stop();
        log("Total Time Taken : "+ stopWatch.getTime());
        return new Product(productId, productInfo, review);
    }
}


public static void main(String[] args) {
    ProductInfoService productInfoService = new ProductInfoService();
    ReviewService reviewService = new ReviewService();
    ProductService productService = new ProductService(productInfoService, reviewService);
    String productId = "ABC123";
    Product product = productService.retrieveProductDetails(productId);
    log("Product is " + product);
}
```


- 그러나 multi-Thread를 사용해 반응속도를 개선할 수 있다.
- Product의 info와 review의 info를 같이 가져오게 만든다.
- 그럼 2초에서 1초로 줄어든다.
- 다만 문제는 Thread를 쓰기 위해 코드가 엄청나게 길어진다. 관리하기 어려워진다는 의미다.
  - Thread 생성자 코드, Thread를 start시키고 연산이 끝나 값을 받아올 때까지 무한히 대기시키는 join
  - Thread 안의 내용물을 구성하기 위한 RunnableImpl class, 거기서 또 정보를 얻어오려는 private field...
```java
public class ProductServiceUsingThread {
    private ProductInfoService productInfoService;
    private ReviewService reviewService;

    public ProductServiceUsingThread(ProductInfoService productInfoService, ReviewService reviewService) {
        this.productInfoService = productInfoService;
        this.reviewService = reviewService;
    }

    
    private class ProductInfoRunnable implements Runnable {
        private ProductInfo productInfo;
        private String productId;

        public ProductInfoRunnable(String productId) {
            this.productId = productId;
        }

        public ProductInfo getProductInfo() {
            return productInfo;
        }

        @Override
        public void run() {

            productInfo = productInfoService.retrieveProductInfo(productId);
        }
    }

    private class ReviewRunable implements Runnable {
        private String productId;

        public Review getReview() {
            return review;
        }

        private Review review;

        public ReviewRunable(String productId) {
            this.productId = productId;
        }

        @Override
        public void run() {
            review = reviewService.retrieveReviews(productId);
        }
    }

    public Product retrieveProductDetails(String productId) throws InterruptedException {
        stopWatch.start();
        ProductInfoRunnable productInfoRunnable = new ProductInfoRunnable(productId);
        Thread productInfoThread = new Thread(productInfoRunnable);

        ReviewRunable reviewRunnable = new ReviewRunable(productId);
        Thread reviewThread = new Thread(reviewRunnable);

        productInfoThread.start();
        reviewThread.start();

        productInfoThread.join();                                       
        reviewThread.join();
 
        ProductInfo productInfo = productInfoRunnable.getProductInfo(); //Thread가 start됐으므로 runnable에서 productInfo를 가져올 수 있다. join을 해놨으니 가져오는 것. join없으면 아무값도 못가져옴..
        Review review = reviewRunnable.getReview();                     //Thread가 start됐으므로 runnable에서 productInfo를 가져올 수 있다. join을 해놨으니 가져오는 것. join없으면 아무값도 못가져옴..

        stopWatch.stop();
        log("Total Time Taken : " + stopWatch.getTime());
        return new Product(productId, productInfo, review);
    }

    public static void main(String[] args) throws InterruptedException {

        ProductInfoService productInfoService = new ProductInfoService();
        ReviewService reviewService = new ReviewService();
        ProductServiceUsingThread productService = new ProductServiceUsingThread(productInfoService, reviewService);
        String productId = "ABC123";
        Product product = productService.retrieveProductDetails(productId);
        log("Product is " + product);

    }
}
```


- 그래서 JDK 1.5에서 ThreadPool과 ExecutorService가 탄생했다.
- ThreadPool이 있으면 코드에서 start join을 전부 쓸 필요가 없다.
- ExecutorService를 쓰려면 3가지 요소가 필요하다.
  - ThreadPool이 필요하다.
  - WorkQueue도 필요하다.
  - completionQueue도 필요하다.

- 위의 Thread를 start()하고 join()했던 코드는 아래와 같이 양이 확 줄어든다.
- 다만 Future는 합치기가 어렵고, productInfoFuture가 먼저 값을 얻어와야 비로소 review에 값을 할당할 수 있다.
```java
public class ProductServiceUsingExecutor {

    static ExecutorService executorService= Executors.newFixedThreadPool(Runtime.getRuntime().availableProcessors());
    private ProductInfoService productInfoService;
    private ReviewService reviewService;

    public ProductServiceUsingExecutor(ProductInfoService productInfoService, ReviewService reviewService) {
        this.productInfoService = productInfoService;
        this.reviewService = reviewService;
    }

    public Product retrieveProductDetails(String productId) throws ExecutionException, InterruptedException, TimeoutException {
        stopWatch.start();

        Future<ProductInfo> productInfoFuture = executorService.submit(()->productInfoService.retrieveProductInfo(productId));
        Future<Review> reviewFuture = executorService.submit(()->reviewService.retrieveReviews(productId));

        //ProductInfo productInfo = productInfoFuture.get();
        ProductInfo productInfo = productInfoFuture.get(2, TimeUnit.SECONDS); //끝날때까지 2초간 기다려준다.무한하게 기다리지 않는다. 만약 대기시간을 넘어서면 TimeoutException이 터진다.
        Review  review = reviewFuture.get();                                  //끝날떄까지 무한하게 기다려준다.

        stopWatch.stop();
        log("Total Time Taken : "+ stopWatch.getTime());
        return new Product(productId, productInfo, review);
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException, TimeoutException {

        ProductInfoService productInfoService = new ProductInfoService();
        ReviewService reviewService = new ReviewService();
        ProductServiceUsingExecutor productService = new ProductServiceUsingExecutor(productInfoService, reviewService);
        String productId = "ABC123";
        Product product = productService.retrieveProductDetails(productId);
        log("Product is " + product);
        executorService.shutdown(); //method가 끝났으니 shutdown한다.

    }
}
```


- ThreadPool/ExecutorSerivce 이후에는 Fork/Join framework가 1.7에 나왔다.
- Fork/Join과 ExecutorService의 차이는 Fork/join은 data parallelism이라는 사실이다.
- ExecutorService는 task base parallelism이다.
  - 아래와 같이 service가 task를 완료하는 데 집중한다.
```java
Future<ProductInfo> productInfoFuture = executorService.submit(()->productInfoService.retrieveProductInfo(productId));

Future<Review> reviewFuture = executorService.submit(()->reviewService.retrieveReviews(productId));
```

- 반면에 Fork/Join은 data based parallelism이다.
- Fokr/join은 3가지 요소로 구성된다.
  - Shared Work Queue
  - Worker Thread
  - Work Queue per Thread
  - Work stealing
    - 놀고 있는 Thread는 일이 밀린 Thread의 work queue에서 일을 가져온다.

- string을 formatting하는 예시를 들어보자.
- 전통 방식으로는 아래와 같이 forEach를 사용한다.
- 만약 오래걸리는 방식이었다면 0.5초씩 list의 element를 곱해야 한다.
- 2초를 조금 넘게 걸릴 것이다. element가 4개라서 그렇다.
```java
public class StringTransformExample {

    public static void main(String[] args) {

        List<String> resultList = new ArrayList<>();
        List<String> names = DataSet.namesList(); //[Bob, Jamie, Jill, Rick]

        names.forEach((name)->{
            String newValue = addNameLengthTransform(name);
            resultList.add(newValue);
        });
    }


    private static String addNameLengthTransform(String name) {
        delay(500);
        return name.length()+" - "+name ;
    }
}
```

- 여태까지 했던 Thread, ExecutorService는 모두 task based parallelism이다. 
- 하지만 Fork/join은 data based다. 
- ForkJoinPool이 invoke 되면 compute method가 발동된다.
```java
public static void main(String[] args) {

        ForkJoinPool forkJoinPool = new ForkJoinPool();
        ForkJoinUsingRecursion forkJoinExampleUsingRecursion = new ForkJoinUsingRecursion(DataSet.namesList());
        stopWatch.start();

        // Start things running and get the result back, This is blocked until the results are calculated.
        List<String> resultList = forkJoinPool.invoke(forkJoinExampleUsingRecursion); // invoke -> Add the task to the shared queue from which all the other qu

        log("resultList : " + resultList);

        stopWatch.stop();
        log("Total time taken : " + stopWatch.getTime());
    }
```

- 만들려면 RecursiveTask를 extends하고 return할 type을 generics로 지정한다. 
- 생성자에는 나눌 data collection을 집어넣는다.
```java
public class ForkJoinUsingRecursion extends RecursiveTask<List<String>> {

    public ForkJoinUsingRecursion(List<String> inputList) {
        this.inputList = inputList;
    }
}
```


- ForkJoinTask의 generics type도 RecursiveTask의 generics와 같아야 한다.
- 놀고 있는 worker thread가 받아가게끔 일을 나눈다.
```java
@Override
    protected List<String> compute() {
        int midPoint = inputList.size() / 2;
        ForkJoinTask<List<String>> leftInputList = new ForkJoinUsingRecursion(inputList.subList(0, midPoint)) //left side of the list
                .fork(); // 1. asynchronously arranges this task in the deque,
        inputList = inputList.subList(midPoint, inputList.size()); //right side of the list
    }
```


- 나눌수 없는 수준까지 data를 나눠서 compute한다.
- list의 크기가 1이 되게 되면 거기서는 더 나눌 수 없는 수준이다. 
- 이같이 if문으로 어느 수준까지 나눌 지를 정해준다.
```java
 @Override
    protected List<String> compute() {
        if (this.inputList.size() <= 1) {
            List<String> resultList = new ArrayList<>();
            inputList.forEach(name -> resultList.add(transform(name)));
            return resultList;
        }

        int midPoint = inputList.size() / 2;
        ForkJoinTask<List<String>> leftInputList = new ForkJoinUsingRecursion(inputList.subList(0, midPoint)) //left side of the list
                .fork(); // 1. asynchronously arranges this task in the deque,
        inputList = inputList.subList(midPoint, inputList.size()); //right side of the list
    }
```

- 일을 나눠 각자 thread에서 계산된 결과를 합쳐준다.
```java
 @Override
    protected List<String> compute() {
        if (this.inputList.size() <= 1) {
            List<String> resultList = new ArrayList<>();
            inputList.forEach(name -> resultList.add(transform(name)));
            return resultList;
        }

        int midPoint = inputList.size() / 2;
        ForkJoinTask<List<String>> leftInputList = new ForkJoinUsingRecursion(inputList.subList(0, midPoint)) //left side of the list
                .fork(); // 1. asynchronously arranges this task in the deque,
        inputList = inputList.subList(midPoint, inputList.size()); //right side of the list

        List<String> rightResult = compute(); //recursion happens
        List<String> leftResult = leftInputList.join();
        log("leftResult : "+ leftResult);
        leftResult.addAll(rightResult);
        return leftResult;
    }
```


- compute가 발동되면 0.5초씩 x 4 만큼 걸리지 않는다.
- 0.5초를 조금 넘기는 시간만큼만 걸리게 된다.
```java
public static void main(String[] args) {

    ForkJoinPool forkJoinPool = new ForkJoinPool();
    ForkJoinUsingRecursion forkJoinExampleUsingRecursion = new ForkJoinUsingRecursion(DataSet.namesList());
    stopWatch.start();

    // Start things running and get the result back, This is blocked until the results are calculated.
    List<String> resultList = forkJoinPool.invoke(forkJoinExampleUsingRecursion); // invoke -> Add the task to the shared queue from which all the other qu

    log("resultList : " + resultList);

    stopWatch.stop();
    log("Total time taken : " + stopWatch.getTime());
}
```



