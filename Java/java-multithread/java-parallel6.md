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

    public static void main(String[] args) {

        ProductInfoService productInfoService = new ProductInfoService();
        ReviewService reviewService = new ReviewService();
        ProductService productService = new ProductService(productInfoService, reviewService);
        String productId = "ABC123";
        Product product = productService.retrieveProductDetails(productId);
        log("Product is " + product);

    }
}
```

- thenCombine을 이용해서 위의 두 서비스를 parallel하게 합쳐서 가져온다.
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

    public static void main(String[] args) {

        ProductInfoService productInfoService = new ProductInfoService();
        ReviewService reviewService = new ReviewService();
        ProductServiceUsingCompletableFuture productService = new ProductServiceUsingCompletableFuture(productInfoService, reviewService);
        String productId = "ABC123";
        Product product = productService.retrieveProductDetails(productId);
        log("Product is " + product);

    }
}
```


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


- ProductInfo를 가져올 때 inventory option을 설정해줘야 하기 떄문에 아래와 같이 thenApply에서 updateInventoryToProductOption()를 호출한다.
- supplyAsync에서 호출하는 것은 불가능하다. productInfo element를 가져올 방법이 없다. 따라서 thenApply나 thenAccept에서 호출한다. 
- 여기선 map과 같은 느낌이므로 thenApply를 호출한다.
```java
 public Product retrieveProductDetailsWithInventory(String productId) {

        startTimer();
        CompletableFuture<ProductInfo> cfProductInfo = CompletableFuture.supplyAsync(() -> productInfoService.retrieveProductInfo(productId))
                .thenApply((productInfo -> {
                    productInfo.setProductOptions(updateInventoryToProductOption(productInfo));
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


- test case는 아래와 같다.
- 실제로 productOption이 설정됐는지 보려면 forEach로 실제로 element를 불러야 한다.
- ProductInfoService의 retrieveProductInfo가 1초가 걸리고 retrieveProductDetailsWithInventory 안의 inventoryService.retrieveInventory가 0.5초가 걸리는데, 두개의 서비스가 sequential하게 진행되기 때문에 실제로는 2초 가량이 걸린다.
- ProductionOptions 리스트가 많아질수록 blocking I/O가 많아져 걸리는 시간이 계속 늘어나게 된다. async하지 않은 것이다.
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


- 그럼 async하게 만들어줘야 한다.
- 아래와 같이 inventoryService.retrieveInventory를 CompletableFuture로 감싸준다.
```java
private List<ProductOption> updateInventoryToProductOption_approach2(ProductInfo productInfo) {

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
```

- retrieveProductDetailsWithInventory에서는 아래와 같이 updateInventoryToProductOption만 approach2 method로 변경하면 된다.
```java
 public Product retrieveProductDetailsWithInventory(String productId) {

        startTimer();
        CompletableFuture<ProductInfo> cfProductInfo = CompletableFuture.supplyAsync(() -> productInfoService.retrieveProductInfo(productId))
                .thenApply((productInfo -> {
                    productInfo.setProductOptions(updateInventoryToProductOption_approach2(productInfo) /*여기만 approach2 method로 변경 */);
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


- test case를 작성해보자. 사실 test case 또한 retrieveProductDetailsWithInventory 대신 retrieveProductDetailsWithInventory_approach2로 method를 바꾸는 게 다다.
- method를 바꾸게 되면 이제는 2초가 아니라 1.5초가 나온다.
- inventory를 parallel하게 가져오기 때문이다.
- productOptions 요소가 많아져도 1.5초로 고정이다.
```java
@Test
void retrieveProductDetailsWithInventory_approach2() {

    //given
    String productId = "ABC123";
    startTimer();

    //when
    Product product = pscf.retrieveProductDetailsWithInventory_approach2(productId);
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
