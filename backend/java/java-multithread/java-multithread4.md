## <span style="color:#802548">_multiThread 사용 시에도 atomic한 경우_</span>
- 안타깝게도 대부분의 연산은 non-atomic하다.
- 그러나 몇개의 연산은 언제나 atomic하다.
  - 모든 참조에 대한 할당은 atomic하다.
  - 또한 long과 double을 제외한 원시형의 할당 또한 atomic하다.
    - long과 double이 예외인 것은 64 bit라 Java가 32 bit로 둘을 나눠 연산할 가능성이 높기 때문이다.
    - 대신 double이나 long을 volatile을 달아주면 무조건 atomic해진다.
    - 따라서 getter연산은 synchronized가 필요없다.
    - setter의 경우는 참조에 대한 할당처럼 작동할 때만, 즉 연산이 없을 때만 atomic하다.

- 실제 아래와 같은 getter는 synchronized를 안 달아도 원자적 연산이다.
- 여기서 setter는 단순한 참조에 대한 할당이므로 syncrhonized가 필요없다.
```java
public class SharedClass {
        private String name;
 
        public void updateString(String name) {
            this.name = name;
        }
        
        public String getName() {
            return name;
        }
    }
```


- long이나 double은 다른 원시형과 다르게 volatile을 달아주어야만 atomic하다.
- 32bit로 나눠서 처리되기 때문에 그러한 것이다.
- 또한 연산이 들어가는 경우 반드시 synchronized 를 달아줘야 한다.
```java
public class Main {
    public static class Metrics {
        private long count = 0;
        private volatile double average = 0.0;

        public synchronized void addSample(long sample) {
            double currentSum = average * count;
            count++;
            average = (currentSum + sample) / count;
        }

        public double getAverage() {
            return average;
        }
    }
}
```


- 실제 사례를 살펴보자.
- business logic을 만드는데, 여기에서는 try ~ catch에 들어간게 business logic이다. BusinessLogic class다. 
- 시간이 그만큼 걸리니 sleep으로 아래와 같이 만든 것이다.
```java
public class Main {

    public static class BusinessLogic extends Thread {
        private Metrics metrics;
        private Random random = new Random();

        public businessLogic(Metrics metircs) {
            this.metrics = metrics;
        }

        @Override
        public void run() {

            while(true) {
                long start = System.currentTimeMillis();
                try {
                    Thread.sleep(random.nextInt(10));
                } catch (InterruptedException e) {
                }

                long end = System.currentTimeMillis();
                metrics.addSample(end - start);
            }
        }
    }

    public static class Metrics {
        private volatile long count = 0;
        private volatile double average = 0.0;

        public synchronized void addSample(long sample) {
            double currentSum = average * count;
            count++;
            average = (currentSum + sample) / count;
        }

        public double getAverage() {
            return average;
        }
    }
}
```


- 이제 businessLogic을 연산해보자.
- main method에 businessLogic을 넣는다.
- 그럼 실제로 대부분 5 ms 정도의 실행시간 평균값을 보이게 된다.
```java
public class Main {

    public static void main(String[] args) {
        Metrics metrics = new Metrics();

        BusinessLogic businessLogicThread1 = new BusinessLogic(metrics);

        BusinessLogic businessLogicThread2 = new BusinessLogic(metrics);

        businessLogicThread1.start();
        businessLogicThread2.start();
    }
}
```

- getter와 setter를 이용하여 새로운 값이 올 때 최대, 최솟값을 계속 바꿔가는 예시는 아래와 같다.
- long에 volatile을 달아준다. 이를 통해 getter에는 synchronized keyword를 쓰지 않을 수 있다.
- 그 다음엔 필요한 부분에만 synchronized 키워드를 달아준다.

```java
public class MinMaxMetrics {
 
    private volatile long minValue;
    private volatile long maxValue;
 
    /**
     * Initializes all member variables
     */
    public MinMaxMetrics() {
        this.maxValue = Long.MIN_VALUE;
        this.minValue = Long.MAX_VALUE;
    }
 
    /**
     * Adds a new sample to our metrics.
     */
    public void addSample(long newSample) {
        synchronized (this) {
            this.minValue = Math.min(newSample, this.minValue);
            this.maxValue = Math.max(newSample, this.maxValue);
        }
    }
 
    /**
     * Returns the smallest sample we've seen so far.
     */
    public long getMin() {
        return this.minValue;
    }
 
    /**
     * Returns the biggest sample we've seen so far.
     */
    public long getMax() {
        return this.maxValue;
    }
}
```

## <span style="color:#802548">_synchronized와 volatile이 빠진 예시_</span>
- race condition은 thread 스케줄링의 순서나 시점에 따라 결과가 달라지는 현상을 의미한다.
- race condition을 해결하기 위해 위처럼 synchronized를 활용하게 된다.
- 공유 리소스(heap data)에 비원자적 연산(OS 입장)이 실행될 때 흔하게 발견되는 문제 현상이다.
- items++ 연산은 비원자적연산이기에 synchronized keyword 없이는 main method의 getItems() 결과가 매번 달라지게 된다. 맨 처음 예시다.
  - synchronized를 덜 쓰는 방법은 getter, setter로 만들고 클래스의 멤버변수로 원시형변수를 사용하는 것이다.
  - long이나 double의 경우에만 volatile을 붙여주면 된다. 단 비원자적 연산들은 volatile만으로는 안되고 synchronized도 같이 붙여줘야 한다.
```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        InventoryCounter inventoryCounter = new InventoryCounter();
        IncrementingThread incrementingThread = new IncrementingThread(inventoryCounter);
        DecrementingThread decrementingThread = new DecrementingThread(inventoryCounter);

        incrementingThread.start();
        decrementingThread.start();

        incrementingThread.join();
        decrementingThread.join();

        System.out.println("We currently have " + inventoryCounter.getItems() + " items"); //0
    }

    public static class DecrementingThread extends Thread {

        private InventoryCounter inventoryCounter;

        public DecrementingThread(InventoryCounter inventoryCounter) {
            this.inventoryCounter = inventoryCounter;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                inventoryCounter.decrement();
            }
        }
    }

    public static class IncrementingThread extends Thread {

        private InventoryCounter inventoryCounter;

        public IncrementingThread(InventoryCounter inventoryCounter) {
            this.inventoryCounter = inventoryCounter;
        }

        @Override
        public void run() {
            for (int i = 0; i < 10000; i++) {
                inventoryCounter.increment();
            }
        }
    }

    private static class InventoryCounter {
        private int items = 0; //volatile 추가 필요 없음

        public void increment() { //synchronized 추가 필요
            items++;
        }

        public void decrement() {//synchronized 추가 필요
            items--;
        }

        public int getItems() {//synchronized 추가 필요 없음
            return items;
        }
    }
}
```
