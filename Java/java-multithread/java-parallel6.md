## <span style="color:#802548">_SERVICE 2개를 parallel하게 가져오기_</span>
- CompletableFuture를 이용해 앱을 만들어보자.
- 실제 DB접근과 비슷하게 1초 딜레이를 준다.
```java
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
```

- 똑같이 여기도 1초 딜레이를 준다.
```java
public class ReviewService {

    public Review retrieveReviews(String productId) {
        delay(1000);
        LoggerUtil.log("retrieveReviews after Delay");
        return new Review(200, 4.5);
    }
}
```

- 위 두개의 서비스를 sequntial하게 합쳐서 가져온다.
- 그럼 순서대로 실행되므로 1초 + 1초라서 2초가 걸린다.
```java
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

- thenCombine을 이용해서 위의 두 서비스를 parallel하게 합쳐서 가져온다.
- 그럼 2초가 아니라 1초만 걸린다. 각각 1초씩 가져오기 떄문이다.
```java
public class ProductServiceUsingCompletableFuture {
    private ProductInfoService productInfoService;
    private ReviewService reviewService;
    private InventoryService inventoryService;

    public ProductServiceUsingCompletableFuture(ProductInfoService productInfoService, ReviewService reviewService) {
        this.productInfoService = productInfoService;
        this.reviewService = reviewService;
    }

    public Product retrieveProductDetails(String productId) {

        startTimer();

        //productInfo와 reviewInfo를 가져올 준비를 함.
        CompletableFuture<ProductInfo> cfProductInfo = CompletableFuture.supplyAsync(() -> productInfoService.retrieveProductInfo(productId));
        CompletableFuture<Review> cfReview = CompletableFuture.supplyAsync(() -> reviewService.retrieveReviews(productId));


        //thenCombine으로 합치므로 parallel하게 가져오게 됨.
        Product product = cfProductInfo
                .thenCombine(cfReview, (productInfo, review) -> new Product(productId, productInfo, review))
                .join(); // blocks the thread
        timeTaken();
        return product;
    }
}

public static void main(String[] args) {

    ProductInfoService productInfoService = new ProductInfoService();
    ReviewService reviewService = new ReviewService();
    ProductServiceUsingCompletableFuture productService = new ProductServiceUsingCompletableFuture(productInfoService, reviewService);
    String productId = "ABC123";
    Product product = productService.retrieveProductDetails(productId);
    log("Product is " + product);

}
```

## <span style="color:#802548">_thenCombine Junit test_</span>
- test는 아래와 같이 진행한다.
- pscf로 가져오면 parallel하게 가져오므로 그냥 service를 합친 ProductService보다 더 빠르다.
```java
class ProductServiceUsingCompletableFutureTest {

    //test에서는 DI가 안되므로 전부 new로 생성해주어야 한다.
    ProductInfoService pis = new ProductInfoService();
    ReviewService rs = new ReviewService();
    InventoryService is = new InventoryService();
    ProductServiceUsingCompletableFuture pscf = new ProductServiceUsingCompletableFuture(pis, rs, is);

    @Test
    void retrieveProductDetails() {

        //given
        String productId = "ABC123";
        startTimer();

        //when
        Product product = pscf.retrieveProductDetails(productId);
        System.out.println("product:  " + product);

        //then
        assertNotNull(product);
        assertTrue(product.getProductInfo().getProductOptions().size() > 0);
        assertNotNull(product.getReview());

    }
}
```

- 원한다면 CompletableFuture로 return type을 만들 수도 있다.
```java
public CompletableFuture<Product> retrieveProductDetails_CF(String productId) {

        CompletableFuture<ProductInfo> cfProductInfo = CompletableFuture.supplyAsync(() -> productInfoService.retrieveProductInfo(productId));
        CompletableFuture<Review> cfReview = CompletableFuture.supplyAsync(() -> reviewService.retrieveReviews(productId));

        return cfProductInfo
                .thenCombine(cfReview, (productInfo, review) -> new Product(productId, productInfo, review));
    }
```


- test case는 아래와 같다.
```java
@Test
void retrieveProductDetails_CF() {

    //given
    String productId = "ABC123";
    startTimer();

    //when
    CompletableFuture<Product> cfProduct = pscf.retrieveProductDetails_approach2(productId);

    //then
    cfProduct
            .thenAccept((product -> {
                assertNotNull(product);
                assertTrue(product.getProductInfo().getProductOptions().size() > 0);
                assertNotNull(product.getReview());
            }))
            .join();

    timeTaken();

}
```

## <span style="color:#802548">_중첩 Service도 parallel하게 실행하기_</span>
- 여기에 inventoryService도 추가해보자.
```java
public class InventoryService {
    public Inventory retrieveInventory(ProductOption productOption) {
        delay(500);
        return Inventory.builder()
                .count(2).build();

    }
}
```


- inventory는 productOptions의 field다. 따라서 productInfo를 얻어오면서 inventoryService를 호출한다.
```java
private List<ProductOption> updateInventoryToProductOption(ProductInfo productInfo) {
    List<ProductOption> productOptionList = productInfo.getProductOptions()
            .stream()
            .map(productOption -> {
                Inventory inventory = inventoryService.retrieveInventory(productOption);
                productOption.setInventory(inventory);
                return productOption;
            })
            .collect(Collectors.toList());

    return productOptionList;
}
```

- thenApply에서 updateInventoryToProductOption()가 0.5초의 delay를 가진다.
- 따라서 아래 예제에서 cfProductInfo를 parallel하게 가져오는 것처럼 보여도 seqential한 작동이 되어버린다.
- 따라서 productOptions를 얻어와 바꾸는 logic도 parallel하게 바꿔야 한다.
```java
public class ProductServiceUsingCompletableFuture {
    public List<ProductOption> updateInventoryToProductOption_approach2(ProductInfo productInfo) {

        List<CompletableFuture<ProductOption>> productOptionList = productInfo.getProductOptions()
            .stream()
            .map(productOption -> {
                return CompletableFuture.supplyAsync(() -> inventoryService.retrieveInventory(productOption))
                                        .thenApply(inventory -> {
                                        productOption.setInventory(inventory)
                                        return productOption;
                                        });
            })
            .collect(Collectors.toList());

        return productOptionList.stream().map(CompletableFuture::join).collect(Collectors.toList());
    }
}
```


- ProductInfo를 가져올 때 inventory option을 설정해줘야 하기 떄문에 아래와 같이 thenApply에서 updateInventoryToProductOption()를 호출한다.
- supplyAsync에서 호출하는 것은 불가능하다. productInfo element를 가져올 방법이 없다. 따라서 thenApply나 thenAccept에서 호출한다. 
- 여기선 return이 필요하므로 thenApply를 호출한다.
```java
 public Product retrieveProductDetailsWithInventory(String productId) {
    startTimer();
    CompletableFuture<ProductInfo> cfProductInfo = CompletableFuture.supplyAsync(() -> productInfoService.retrieveProductInfo(productId))
            .thenApply((productInfo -> {
                productInfo.setProductOptions(updateInventoryToProductOption_approach2(productInfo) /* updateInventoryToProductOption는 sequential */);
                return productInfo;
            }));

    CompletableFuture<Review> cfReview = CompletableFuture.supplyAsync(() -> reviewService.retrieveReviews(productId));

    Product product = cfProductInfo
                                .thenCombine(cfReview, (productInfo, review) -> new Product(productId, productInfo, review))
                                .join(); // blocks the thread
    timeTaken();

    return product;
}
```



## <span style="color:#802548">_CompletableFuture example- test 2_</span>
- productOption이 설정됐는지 보려면 forEach로 실제로 element를 불러야 한다.
  - ProductInfoService의 retrieveProductInfo가 1초가 걸리고 retrieveProductDetailsWithInventory 안의 inventoryService.retrieveInventory가 0.5초가 걸린다.
  - 만약 updateInventoryToProductOption_approach2가 아닌 updateInventoryToProductOption를 썼다면 productOptions에서 sequential하게 진행된다.
  - 따라서 productOptions의 갯수에 따라 실행시간이 달라진다. ProductionOptions 리스트가 많아질수록 blocking I/O가 많아져 걸리는 시간이 계속 늘어나게 된다. async하지 않은 것이다.
- 하지만 중첩 서비스도 parallel하게 처리했으므로 현재는 문제가 없다.
- 다만 CPU 물리 코어 갯수에 의존한다.
  - CPU 물리코어 갯수가 처리할 수 있는 productOptionslist보다 적다면 0.5초씩 늘어나게 될 것이다.
  - 나는 24개니까 25개부터 0.5초, 49개부터 1초... 이렇게 늘어난다.
```java
@Test
void retrieveProductDetailsWithInventory() {

    //given
    String productId = "ABC123";
    startTimer();

    //when
    Product product = pscf.retrieveProductDetailsWithInventory(productId);
    System.out.println("product:  " + product);

    //then
    assertNotNull(product);
    assertTrue(product.getProductInfo().getProductOptions().size() > 0);
    product.getProductInfo().getProductOptions().forEach(productOption -> {
        assertNotNull(productOption.getInventory());
    });


    assertNotNull(product.getReview());

}
```

