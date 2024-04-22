- reentrantLock은 synchronized keyword와 비슷하다. 
- 다만 lock()을 걸고, unlock()을 거는 method가 다르다.
- 또한 thread 별로 공평하게 lock에 접근할 기회를 제공한다.
  - 생성자에 true를 주면 된다.
  - 다만 이렇게 하면 느려진다.
  - 꼭 필요할 때만 사용해야 한다.
- 특히 unlock을 하기 전에 exception을 throw하면 lock이 안 풀려 dealock이 된다.
- 따라서 try ~ finally 구문안에 exception이 날 가능성이 있는 logic을 넣어줘야 한다.
- reentrantLock에서 가장 중요한 기능은 tryLock()이다.
  - tryLock()을 활용하게 되면 lock 객체를 얻지 못한 경우에도 false를 반환해 logic을 진행한다.



- 예시를 살펴보자.
- 가상화폐 다섯개의 가격을 가지고 있다.
- Thread 2개가 접근할 수 있기 때문에 가격이 아무렇게나 바뀌지 않게 동시성 제어를 위해 Lock 객체를 만든다.
```java
public static class PricesContainer {
        private Lock lockObject = new ReentrantLock();

        private double bitcoinPrice;
        private double etherPrice;
        private double litecoinPrice;
        private double bitcoinCashPrice;
        private double ripplePrice;

        public Lock getLockObject() {
            return lockObject;
        }

        public double getBitcoinPrice() {
            return bitcoinPrice;
        }

        public void setBitcoinPrice(double bitcoinPrice) {
            this.bitcoinPrice = bitcoinPrice;
        }

        public double getEtherPrice() {
            return etherPrice;
        }

        public void setEtherPrice(double etherPrice) {
            this.etherPrice = etherPrice;
        }

        public double getLitecoinPrice() {
            return litecoinPrice;
        }

        public void setLitecoinPrice(double litecoinPrice) {
            this.litecoinPrice = litecoinPrice;
        }

        public double getBitcoinCashPrice() {
            return bitcoinCashPrice;
        }

        public void setBitcoinCashPrice(double bitcoinCashPrice) {
            this.bitcoinCashPrice = bitcoinCashPrice;
        }

        public double getRipplePrice() {
            return ripplePrice;
        }

        public void setRipplePrice(double ripplePrice) {
            this.ripplePrice = ripplePrice;
        }
    }
```


- 가격을 update를 해주는 method를 만들자.
- 실제 네트워크를 타는 건 아니기 때문에 Random 객체를 활용한다.
- 또한 logic에 따른 update에 걸리는 시간을 반영해주기 위해 Thread를 1초간 sleep시킨다.
```java
public static class PriceUpdater extends Thread {
        private PricesContainer pricesContainer;
        private Random random = new Random();

        public PriceUpdater(PricesContainer pricesContainer) {
            this.pricesContainer = pricesContainer;
        }

        @Override
        public void run() {
            while (true) {
                pricesContainer.getLockObject().lock();

                try {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                    }
                    pricesContainer.setBitcoinPrice(random.nextInt(20000));
                    pricesContainer.setEtherPrice(random.nextInt(2000));
                    pricesContainer.setLitecoinPrice(random.nextInt(500));
                    pricesContainer.setBitcoinCashPrice(random.nextInt(5000));
                    pricesContainer.setRipplePrice(random.nextDouble());
                } finally {
                    pricesContainer.getLockObject().unlock();
                }

                try {
                    Thread.sleep(2000);
                } catch (InterruptedException e) {
                }
            }
        }
    }
```


- main에서는 javaFX를 활용한다. 그 전에 main에서 활용할 private method를 만든다.
  - 가격을 보여주는 label
  - label이 들어갈 틀인 grid
  - grid의 label에 줄 UI변화
  - grid의 background에 줄 UI 변화
```java
    private GridPane createGrid() {
        GridPane grid = new GridPane();
        grid.setHgap(10);
        grid.setVgap(10);
        grid.setAlignment(Pos.CENTER);
        return grid;
    }

    private void addLabelsToGrid(Map<String, Label> labels, GridPane grid) {
        int row = 0;
        for (Map.Entry<String, Label> entry : labels.entrySet()) {
            String cryptoName = entry.getKey();
            Label nameLabel = new Label(cryptoName);
            nameLabel.setTextFill(Color.BLUE);
            nameLabel.setOnMousePressed(event -> nameLabel.setTextFill(Color.RED));
            nameLabel.setOnMouseReleased((EventHandler) event -> nameLabel.setTextFill(Color.BLUE));

            grid.add(nameLabel, 0, row);
            grid.add(entry.getValue(), 1, row);

            row++;
        }
    }

    private Rectangle createBackgroundRectangleWithAnimation(double width, double height) {
        Rectangle backround = new Rectangle(width, height);
        FillTransition fillTransition = new FillTransition(Duration.millis(1000), backround, Color.LIGHTGREEN, Color.LIGHTBLUE);
        fillTransition.setCycleCount(Timeline.INDEFINITE);
        fillTransition.setAutoReverse(true);
        fillTransition.play();
        return backround;
    }

    private Map<String, Label> createCryptoPriceLabels() {
        Label bitcoinPrice = new Label("0");
        bitcoinPrice.setId("BTC");

        Label etherPrice = new Label("0");
        etherPrice.setId("ETH");

        Label liteCoinPrice = new Label("0");
        liteCoinPrice.setId("LTC");

        Label bitcoinCashPrice = new Label("0");
        bitcoinCashPrice.setId("BCH");

        Label ripplePrice = new Label("0");
        ripplePrice.setId("XRP");

        Map<String, Label> cryptoLabelsMap = new HashMap<>();
        cryptoLabelsMap.put("BTC", bitcoinPrice);
        cryptoLabelsMap.put("ETH", etherPrice);
        cryptoLabelsMap.put("LTC", liteCoinPrice);
        cryptoLabelsMap.put("BCH", bitcoinCashPrice);
        cryptoLabelsMap.put("XRP", ripplePrice);

        return cryptoLabelsMap;
    }
```

- 실제 javaFX가 실행되는 main method는 아래와 같다.
  - createGrid
  - createCryptoPriceLabels
  - addLabelsToGrid
  - createBackgroundRectangleWithAnimation
  - primaryStage.show()
```java
public class Main extends Application {
    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        primaryStage.setTitle("Cryptocurrency Prices");

        GridPane grid = createGrid();
        Map<String, Label> cryptoLabels = createCryptoPriceLabels();

        addLabelsToGrid(cryptoLabels, grid);

        double width = 300;
        double height = 250;

        StackPane root = new StackPane();

        Rectangle background = createBackgroundRectangleWithAnimation(width, height);

        root.getChildren().add(background);
        root.getChildren().add(grid);

        primaryStage.setScene(new Scene(root, width, height));
        primaryStage.show();
    }
}
```

- 실제 price update를 반영하기 위해 아래와 같이 main method를 변경해준다.
- 여기서 핵심 lock logic은 AnimationTimer안에 있다.
```java
public class Main extends Application {
    public static void main(String[] args) {
        launch(args);
    }

    @Override
    public void start(Stage primaryStage) {
        primaryStage.setTitle("Cryptocurrency Prices");

        GridPane grid = createGrid();
        Map<String, Label> cryptoLabels = createCryptoPriceLabels();

        addLabelsToGrid(cryptoLabels, grid);

        double width = 300;
        double height = 250;

        StackPane root = new StackPane();

        Rectangle background = createBackgroundRectangleWithAnimation(width, height);

        root.getChildren().add(background);
        root.getChildren().add(grid);

        primaryStage.setScene(new Scene(root, width, height));

        PricesContainer pricesContainer = new PricesContainer();

        PriceUpdater priceUpdater = new PriceUpdater(pricesContainer);

        AnimationTimer animationTimer = new AnimationTimer() {
            @Override
            public void handle(long now) {
                //tryLock 사용. lock을 얻을 수 있으면 true 반환하고 lock() 작동. 따로 lock() 안 써도 됨.
                //lock을 못얻으면 false 반환. 반응성이 좋아짐.
                if (pricesContainer.getLockObject().tryLock()) {
                    try {
                        Label bitcoinLabel = cryptoLabels.get("BTC");
                        bitcoinLabel.setText(String.valueOf(pricesContainer.getBitcoinPrice()));

                        Label etherLabel = cryptoLabels.get("ETH");
                        etherLabel.setText(String.valueOf(pricesContainer.getEtherPrice()));

                        Label litecoinLabel = cryptoLabels.get("LTC");
                        litecoinLabel.setText(String.valueOf(pricesContainer.getLitecoinPrice()));

                        Label bitcoinCashLabel = cryptoLabels.get("BCH");
                        bitcoinCashLabel.setText(String.valueOf(pricesContainer.getBitcoinCashPrice()));

                        Label rippleLabel = cryptoLabels.get("XRP");
                        rippleLabel.setText(String.valueOf(pricesContainer.getRipplePrice()));
                    } finally {
                        pricesContainer.getLockObject().unlock();
                    }
                }
            }
        };

        addWindowResizeListener(primaryStage, background);

        animationTimer.start();

        priceUpdater.start();

        primaryStage.show();
    }

    private void addWindowResizeListener(Stage stage, Rectangle background) {
        ChangeListener<Number> stageSizeListener = ((observable, oldValue, newValue) -> {
            background.setHeight(stage.getHeight());
            background.setWidth(stage.getWidth());
        });
        stage.widthProperty().addListener(stageSizeListener);
        stage.heightProperty().addListener(stageSizeListener);
    }
}
```


- 핵심 lock logic을 더 살펴보자.
- lock을 쥐고 있다면 text 변화를 실행하고, 아니면 실행하지 않는다.
```java
AnimationTimer animationTimer = new AnimationTimer() {
            @Override
            public void handle(long now) {
                //tryLock 사용
                if (pricesContainer.getLockObject().tryLock()) {
                    try {
                        Label bitcoinLabel = cryptoLabels.get("BTC");
                        bitcoinLabel.setText(String.valueOf(pricesContainer.getBitcoinPrice()));

                        Label etherLabel = cryptoLabels.get("ETH");
                        etherLabel.setText(String.valueOf(pricesContainer.getEtherPrice()));

                        Label litecoinLabel = cryptoLabels.get("LTC");
                        litecoinLabel.setText(String.valueOf(pricesContainer.getLitecoinPrice()));

                        Label bitcoinCashLabel = cryptoLabels.get("BCH");
                        bitcoinCashLabel.setText(String.valueOf(pricesContainer.getBitcoinCashPrice()));

                        Label rippleLabel = cryptoLabels.get("XRP");
                        rippleLabel.setText(String.valueOf(pricesContainer.getRipplePrice()));
                    } finally {
                        pricesContainer.getLockObject().unlock();
                    }
                }
            }
        };
```


- tryLock()을 실행하지 않는다면 lock을 얻지 못하면 계속 기다린다.
  - 따라서 UI 변화가 연속적으로 일어나지 못하고 lock을 얻는 순간 전부 갱신되어 UI가 뚝뚝 끊기는 느낌을 받게 된다.
  - 그래서 lock객체를 활용할 때는 tryLock()을 거의 필수적으로 활용하게 된다.
```java
AnimationTimer animationTimer = new AnimationTimer() {
            @Override
            public void handle(long now) {
                pricesConatiner.getLockObject().lock(); //일반 lock 사용
                try {
                    Label bitcoinLabel = cryptoLabels.get("BTC");
                    bitcoinLabel.setText(String.valueOf(pricesContainer.getBitcoinPrice()));

                    Label etherLabel = cryptoLabels.get("ETH");
                    etherLabel.setText(String.valueOf(pricesContainer.getEtherPrice()));

                    Label litecoinLabel = cryptoLabels.get("LTC");
                    litecoinLabel.setText(String.valueOf(pricesContainer.getLitecoinPrice()));

                    Label bitcoinCashLabel = cryptoLabels.get("BCH");
                    bitcoinCashLabel.setText(String.valueOf(pricesContainer.getBitcoinCashPrice()));

                    Label rippleLabel = cryptoLabels.get("XRP");
                    rippleLabel.setText(String.valueOf(pricesContainer.getRipplePrice()));
                } finally {
                    pricesContainer.getLockObject().unlock();
                }
            }
        };
```

- 그 외에 ReentrantReadWriterLock도 있다.
- 읽기와 쓰기 lock을 합친 것이다.
- race condition에 관해 다시 살펴보자.
  - update를 할 때는 막아야만 한다.
  - 그러나 read만 하는 것은 lock을 걸어 막으면 오히려 반응성에 안 좋다.
  - 그럴 때 ReentrantReadWriterLock을 활용한다.
    - 그럼 여러 thread가 읽기에 관해서는 상호 배타 lock이 아니라 접근이 가능하다.
    - 하지만 여러 thread가 쓰기에 관해서는 상호 배타라서 접근을 막는다.
    - writerLock이 걸린 경우에도 readLock이 접근할 수 없다.



- 실제 사례를 살펴보자.
- DB에서 가져오겠지만, 여기서는 DB call을 하지 않으니 유사하게 만든 것이다.
- 사실 DB call을 하게 되면 DB가 동시성 제어를 해야 한다.
  - 똑같은 기능을 이중화한 서버에서는 synchronized 등이 별 의미가 없기 때문이다.
  - 오직 자기 WAS에서 발급한 Thread들 내부에서만 synchronized가 걸린 객체를 알 수 있다.
  - MSA는 다른 기능을 다른 WAS에 넣는 것이라 경우가 다르다.
```java
public class Main {
    public static class InventoryDatabase {
        private TreeMap<Integer, Integer> priceToCountMap = new TreeMap<>(); //트리맵을 쓴 이유는 이진 트리로서 최대값, 최소값 가져오는 시간복잡도로서 나쁘지 않아서로 보임..

        // 가격의 하한가와 상한가를 지정하고, 주어진 가격 범위에서 상품 갯수를 count
        public int getNumberOfItemsInPriceRange(int lowerBound, int upperBound) {
            //최저가를 treeMap에서 찾기
            Integer fromKey = priceToCountMap.ceilingKey(lowerBound);   

            //최고가를 treeMap에서 찾기
            Integer toKey = priceToCountMap.floorKey(upperBound);       

            if (fromKey == null || toKey == null) {
                return 0;
            }

            //원하는 가격대의 tree를 기록
            NavigableMap<Integer, Integer> rangeOfPrices = priceToCountMap.subMap(fromKey, true, toKey, true); 


            //해당 범위의 가격대에서 상품의 총갯수를 반환
            int sum = 0;
            for (int numberOfItemsForPrice : rangeOfPrices.values()) {
                sum += numberOfItemsForPrice;
            }

            return sum;
        }

        //해당 가격의 아이템 갯수 늘리기
        public void addItem(int price) {
            Integer numberOfItemsForPrice = priceToCountMap.get(price);
            if (numberOfItemsForPrice == null) {
                priceToCountMap.put(price, 1);
            } else {
                priceToCountMap.put(price, numberOfItemsForPrice + 1);
            }
        }


        //해당 가격의 아이템 갯수 줄이기
        public void removeItem(int price) {
            Integer numberOfItemsForPrice = priceToCountMap.get(price);
            if (numberOfItemsForPrice == null || numberOfItemsForPrice == 1) {
                priceToCountMap.remove(price);
            } else {
                priceToCountMap.put(price, numberOfItemsForPrice - 1);
            }
        }
    }
}
```

- 상품 갯수를 select/update thread를 만들자.
- writer는 update고, reader는 select다.
```java
 public static void main(String[] args) throws InterruptedException {
    public static final int HIGHEST_PRICE = 1000;

    public static void main(String[] args) throws InterruptedException {
        InventoryDatabase inventoryDatabase = new InventoryDatabase();

        Random random = new Random();
        for (int i = 0; i < 100000; i++) {
            inventoryDatabase.addItem(random.nextInt(HIGHEST_PRICE));
        }

        //상품 갯수 update하는 thread
        Thread writer = new Thread(() -> {
            while (true) {
                inventoryDatabase.addItem(random.nextInt(HIGHEST_PRICE));
                inventoryDatabase.removeItem(random.nextInt(HIGHEST_PRICE));
                try {
                    Thread.sleep(10);
                } catch (InterruptedException e) {
                }
            }
        });

        writer.setDaemon(true);
        writer.start();

        //reader thread는 7개
        int numberOfReaderThreads = 7;
        List<Thread> readers = new ArrayList<>();

        for (int readerIndex = 0; readerIndex < numberOfReaderThreads; readerIndex++) {
            Thread reader = new Thread(() -> {
                for (int i = 0; i < 100000; i++) {
                    int upperBoundPrice = random.nextInt(HIGHEST_PRICE);
                    int lowerBoundPrice = upperBoundPrice > 0 ? random.nextInt(upperBoundPrice) : 0;
                    inventoryDatabase.getNumberOfItemsInPriceRange(lowerBoundPrice, upperBoundPrice);
                }
            });

            reader.setDaemon(true); //Daemon thread로 만드는 이유는 Daemon 스레드는 JVM이 일반 스레드들이 모두 종료되면 자동으로 종료되기 떄문이다.
            readers.add(reader);    //만약 Daemon으로 만들지 않고 일반 스레드로 만든다면, main 스레드가 종료되어도 reader 스레드들이 계속 실행되며 프로그램이 종료되지 않을 수 있다.
        }

        //상품 갯수 read하는 thread
        long startReadingTime = System.currentTimeMillis();
        for (Thread reader : readers) {
            reader.start();
        }

        for (Thread reader : readers) {
            reader.join();
        }

        long endReadingTime = System.currentTimeMillis();

        System.out.println(String.format("Reading took %d ms", endReadingTime - startReadingTime));
    }
 }
```


- read는 lock을 걸지 않고, writer만 lock을 걸려면 아래와 같이 활용한다.
- 여기서 중요한 것은 그냥 ReentrantLock이 아닌 reentrantReadWriteLock을 활용한다는 점이다.
  - ReentrantLock인 경우에는 모든 thread가 read연산이 끝나는데 3초가 걸린다.
  - reentrantReadWriteLock인 경우에는 모든 thread가 read연산이 끝나는데 1초가 걸린다.
  - lock을 바꾸기만 해도 속도가 매우 빨라졌다. 여기서 tryLock까지 섞어주면 더 좋을 것이다.
```java
public static class InventoryDatabase {
    private TreeMap<Integer, Integer> priceToCountMap = new TreeMap<>();
    private ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
    private Lock readLock = reentrantReadWriteLock.readLock();
    private Lock writeLock = reentrantReadWriteLock.writeLock();
    private Lock lock = new ReentrantLock();

    public int getNumberOfItemsInPriceRange(int lowerBound, int upperBound) {
        //lock.lock(); ReentrantLock. 
        readLock.lock();
        try {
            Integer fromKey = priceToCountMap.ceilingKey(lowerBound);

            Integer toKey = priceToCountMap.floorKey(upperBound);

            if (fromKey == null || toKey == null) {
                return 0;
            }

            NavigableMap<Integer, Integer> rangeOfPrices = priceToCountMap.subMap(fromKey, true, toKey, true);

            int sum = 0;
            for (int numberOfItemsForPrice : rangeOfPrices.values()) {
                sum += numberOfItemsForPrice;
            }

            return sum;
        } finally {
            readLock.unlock();
            //lock.unlock();
        }
    }

    public void addItem(int price) {
        //lock.lock();
        writeLock.lock();
        try {
            Integer numberOfItemsForPrice = priceToCountMap.get(price);
            if (numberOfItemsForPrice == null) {
                priceToCountMap.put(price, 1);
            } else {
                priceToCountMap.put(price, numberOfItemsForPrice + 1);
            }

        } finally {
            writeLock.unlock();
            /// lock.unlock();
        }
    }

    public void removeItem(int price) {
        //lock.lock();
        writeLock.lock();
        try {
            Integer numberOfItemsForPrice = priceToCountMap.get(price);
            if (numberOfItemsForPrice == null || numberOfItemsForPrice == 1) {
                priceToCountMap.remove(price);
            } else {
                priceToCountMap.put(price, numberOfItemsForPrice - 1);
            }
        } finally {
            writeLock.unlock();
            // lock.unlock();
        }
    }
}
```

- 그럼 아래와 같이 될 것이다.
- readLock.lock()이 아니라 if(readLock.tryLock())으로 변환하면 된다.
```java
public static class InventoryDatabase {
    private TreeMap<Integer, Integer> priceToCountMap = new TreeMap<>();
    private ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
    private Lock readLock = reentrantReadWriteLock.readLock();
    private Lock writeLock = reentrantReadWriteLock.writeLock();
    private Lock lock = new ReentrantLock();

    public int getNumberOfItemsInPriceRange(int lowerBound, int upperBound) {
        //lock.lock(); ReentrantLock. 
        if(readLock.tryLock()) {
            try {
                Integer fromKey = priceToCountMap.ceilingKey(lowerBound);

                Integer toKey = priceToCountMap.floorKey(upperBound);

                if (fromKey == null || toKey == null) {
                    return 0;
                }

                NavigableMap<Integer, Integer> rangeOfPrices = priceToCountMap.subMap(fromKey, true, toKey, true);

                int sum = 0;
                for (int numberOfItemsForPrice : rangeOfPrices.values()) {
                    sum += numberOfItemsForPrice;
                }

                return sum;
            } finally {
                readLock.unlock();
                //lock.unlock();
            }
        }
        
    }

    public void addItem(int price) {
        //lock.lock();
        if(writeLock.tryLock()) {
            try {
                Integer numberOfItemsForPrice = priceToCountMap.get(price);
                if (numberOfItemsForPrice == null) {
                    priceToCountMap.put(price, 1);
                } else {
                    priceToCountMap.put(price, numberOfItemsForPrice + 1);
                }

            } finally {
                writeLock.unlock();
                /// lock.unlock();
            }
        }
    }

    public void removeItem(int price) {
        //lock.lock();
        if(writeLock.tryLock()) {
            try {
                Integer numberOfItemsForPrice = priceToCountMap.get(price);
                if (numberOfItemsForPrice == null || numberOfItemsForPrice == 1) {
                    priceToCountMap.remove(price);
                } else {
                    priceToCountMap.put(price, numberOfItemsForPrice - 1);
                }
            } finally {
                writeLock.unlock();
            // lock.unlock();
            }
        }
    }
}
```


- 다른 예시를 들면 아래와 같다.
- 굉장히 길지만 핵심은 readLock과 writerLock을 분리하는 것이다.
  - 멤버변수로 lock을 미리 만들어 둔다.
  - lock을 반환할 named method를 만들어 둔다.
  - read용도에는 readLock을 건다.
  - writer용도에는 writeLock을 건다.
```java
import java.util.*;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ProductReviewsService {
    private final HashMap<Integer, List<String>> productIdToReviews;
    
    // Create your member variables here
    private ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();
    private Lock readLock = reentrantReadWriteLock.readLock();
    private Lock writeLock = reentrantReadWriteLock.writeLock();
    

/********* DO NOT MODIFY THIS SECTION **************/
    
    public ProductReviewsService() {
        this.productIdToReviews = new HashMap<>();
    }

    /**
     * Adds a product ID if not present
     */
    public void addProduct(int productId) {
        Lock lock = getLockForAddProduct();
        
        lock.lock();
        
        try {
            if (!productIdToReviews.containsKey(productId)) {
                productIdToReviews.put(productId, new ArrayList<>());
            }
        } finally {
            lock.unlock();
        }
    }

    /**
     * Removes a product by ID if present
     */
    public void removeProduct(int productId) {
        Lock lock = getLockForRemoveProduct();
        
        lock.lock();
        
        try {
            if (productIdToReviews.containsKey(productId)) {
                productIdToReviews.remove(productId);
            }
        } finally {
            lock.unlock();
        }
    }

    /**
     * Adds a new review to a product
     * @param productId - existing or new product ID
     * @param review - text containing the product review
     */
    public void addProductReview(int productId, String review) {
        Lock lock = getLockForAddProductReview();
        
        lock.lock();
        
        try {
            if (!productIdToReviews.containsKey(productId)) {
                productIdToReviews.put(productId, new ArrayList<>());
            }
            productIdToReviews.get(productId).add(review);
        } finally {
            lock.unlock();
        }
    }

    /**
     * Returns all the reviews for a given product
     */
    public List<String> getAllProductReviews(int productId) {
        Lock lock = getLockForGetAllProductReviews();
        
        lock.lock();
        
        try {
            if (productIdToReviews.containsKey(productId)) {
                return Collections.unmodifiableList(productIdToReviews.get(productId));
            }
        } finally {
            lock.unlock();
        }
        
        return Collections.emptyList();
    }

    /**
     * Returns the latest review for a product by product ID
     */
    public Optional<String> getLatestReview(int productId) {
        Lock lock = getLockForGetLatestReview();
        
        lock.lock();
        
        try {

            if (productIdToReviews.containsKey(productId) && !productIdToReviews.get(productId).isEmpty()) {
                List<String> reviews = productIdToReviews.get(productId);
                return Optional.of(reviews.get(reviews.size() - 1));
            }
        } finally {
            lock.unlock();
        }
        
        return Optional.empty();
    }

    /**
     * Returns all the product IDs that contain reviews
     */
    public Set<Integer> getAllProductIdsWithReviews() {
        Lock lock = getLockForGetAllProductIdsWithReviews();
        
        lock.lock();
        
        try {
            Set<Integer> productsWithReviews = new HashSet<>();
            for (Map.Entry<Integer, List<String>> productEntry : productIdToReviews.entrySet()) {
                if (!productEntry.getValue().isEmpty()) {
                    productsWithReviews.add(productEntry.getKey());
                }
            }
            return productsWithReviews;
        } finally {
            lock.unlock();
        }
    }

/********* END OF UNMODIFIABLE SECTION **************/


    Lock getLockForAddProduct() {
        return writeLock;
    }

    Lock getLockForRemoveProduct() {
        return writeLock;
    }

    Lock getLockForAddProductReview() {
        return writeLock;
    }

    Lock getLockForGetAllProductReviews() {
        return readLock;
    }

    Lock getLockForGetLatestReview() {
       return readLock;
    }
    
    Lock getLockForGetAllProductIdsWithReviews() {
        return readLock;
    }
}
```