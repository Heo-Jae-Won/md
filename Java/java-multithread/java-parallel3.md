- product를 확인하여 특별한 할인을 주는 행사를 한다고 해보자.
- ProductItem은 아래와 같은 field를 가진다.
```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class ProductItem {

    private Integer itemId;
    private String itemName;
    private double rate;
    private Integer quantity;
}
```

- 확인한 productId가 특정한 item이면 true를 return한다.
- 이 때 true를 return한 경우는 특별 할인율을 얻게 된다.
```java
public class EventProductService {
    public boolean isEventItem(ProductItem ProductItem){
        int productId = ProductItem.getItemId();
        log("isEventItem : "+ ProductItem);
        delay(500);
        if (productId == 7 || productId == 9 || productId == 11) {
            return true;
        }
        return false;
    }
}
```



- eventItem에 할인율을 더해주는 service다.
```java
public class DiscountService {

    private EventProductService EventProductService;

    public DiscountService(EventProductService EventProductService) {
        this.EventProductService = EventProductService;
    }

    public double getFinalPrice(List<ProductItem> productItemList) {

        if (productItemList.size() < 0) {
            log("productItem not found");
            throw new DiscountException(ErrorCode.NOT_FOUND, "not found");
        }

        startTimer();
        List<ProductItem> eventItemList = productItemList
                //.stream()
                .parallelStream()
                .map(ProductItem -> {
                    boolean isEventItem = EventProductService.isEventItem(ProductItem);
                    ProductItem.setRate(ProductItem.getRate() + 0.1);
                    return ProductItem;
                })
                .collect(toList());
        timeTaken();
        stopWatchReset();

        if (eventItemList.size() < 0) {
            log("discount item not found");
            throw new DiscountException(ErrorCode.DISCOUNT_ITEM_NOT_FOUND, "discount Item not found");
        }

        //double finalRate = calculateFinalPrice(cart);
        double finalRate = calculateFinalPrice_reduce(cart);

        return finalRate;

    }

    private double calculateFinalPrice(Cart cart) {
        return cart.getProductItemList()
                .parallelStream()
                .map(ProductItem -> ProductItem.getQuantity() * ProductItem.getRate())
                .collect(summingDouble(Double::doubleValue));
        //.mapToDouble(Double::doubleValue)
        //.sum();
    }

    private double calculateFinalPrice_reduce(Cart cart) {
        return cart.getProductItemList()
                .parallelStream()
                .map(ProductItem -> ProductItem.getQuantity() * ProductItem.getRate())
                //.reduce(0.0, (x,y)->x+y);
                .reduce(0.0, Double::sum);
        //Identity for multiplication is 1
        //Identity for addition  is 0
    }
}
```



- parallelism을 Junit으로 test하는 방법도 알아야 할 것이다.
- 우선 아래와 같이 dataSet을 만들 class를 놓는다.
```java
public static Cart createCart(int noOfItemsInCart) {

    Cart cart = new Cart();
    List<CartItem> cartItemList = new ArrayList<>();
    IntStream.rangeClosed(1, noOfItemsInCart)
            .forEach((index) -> {
                String cartItem = "CartItem -".concat(index + "");
                CartItem cartItem1 = new CartItem(index, cartItem, generateRandomPrice(), index, false);
                cartItemList.add(cartItem1);
            });
    cart.setCartItemList(cartItemList);
    return cart;
}
```

- 이 가짜 객체를 가지고 원하는 결과를 얻는지 test해본다.
- checkout은 sequential이므로 0.5초씩 하면 0.5 x 6 하여 3초가량이 걸린다.
- 만약 checkout을 stream()이 아닌 parallelStream()으로 바꾸면 0.5초 가량이 걸린다.
```java
//priceValidatorService.isCartItemInvalid는 0.5초 간 sleep한다.
public void checkout(Cart cart) {

        startTimer();
        List<CartItem> priceValidationList = cart.getCartItemList()
                .stream()
                //.parallelStream()
                .map(cartItem -> {
                    boolean isPriceValid = priceValidatorService.isCartItemInvalid(cartItem);
                    cartItem.setExpired(isPriceValid);
                    return cartItem;
                })
                .filter(CartItem::isExpired)
                .collect(toList());
        timeTaken();
        stopWatchReset();
    }
```

- 아래와 같이 6개에 대해서 parallel하게 수행하면, 0.5초 가량 걸린다.
- 하지만 element 갯수가 적어서 그런 것일 수 있다. Fork/Join은 어쨌든 물리 CPU 갯수에 의존하기 때문이다.
```java
@Test
void checkout_6_items() {

    //given
    Cart cart = DataSet.createCart(6);

    //when
    CheckoutResponse checkoutResponse = checkoutService.checkout(cart);

    //then
    assertEquals(CheckoutStatus.SUCCESS, checkoutResponse.getCheckoutStatus());
    assertTrue(checkoutResponse.getFinalRate()>0);
}
```

- 아래 방법을 통해 실제로 CPU 코어를 최대 몇개까지 사용할 수 있는 지도 볼 수 있다.
- 강사는 12개였는데, 나는 24개다.
```java
@Test
void no_of_cores() {
    System.out.println("no of cores : " +Runtime.getRuntime().availableProcessors());
}
```

- 만약 CPU 코어 수 이상의 item을 잘라서 활용하면 더 느려진다.
- 내 경우는 24개까지는 0.5초 가량이지만, element가 25개부터 48개까지는 1초다.
- 49개부터 72까지는 1.5초일 것이다.
```java
@Test
void checkout_25_items() {

    //given
    Cart cart = DataSet.createCart(25);

    //when
    CheckoutResponse checkoutResponse = checkoutService.checkout(cart);

    //then
    assertEquals(CheckoutStatus.FAILURE, checkoutResponse.getCheckoutStatus());
}
```
