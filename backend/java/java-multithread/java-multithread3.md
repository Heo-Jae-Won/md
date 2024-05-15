## <span style="color:#802548">_Thread와 Process_</span>
- Process는 Context와 동일하다.
- Process는 Main Thread와 하위 thread를 가진다.
- 그 외에 process는 Heap와 Code도 가진다.
- Thread는 stack을 가진다. stack은 method를 호출할 때 필요한 영역이다.
- thread 간 stack은 공유되지 않는다. 
- heap은 모든 Thread가 공유하는 공간이다.
- heap에는 모든 객체가 저장된다. 
  - String
  - Object
  - Collection
  - static variable
  - memeber variable of classes


- allNames는 reference로서 stack에 저장되지만, 이 reference가 가리키는 ArrayList-String 유형의 객체는 heap에 할당된다.
```java
public List<String> getAllNames() {
    int count = idToNameMap.size();
    List<String> allNames = new ArrayList<>();
    
    allNames.addAll(idToNameMap.values());
    
    return allNames;
}
```

- 아래와 같이 start()와 join()을 순서대로 thread마다 실행하면 0이 나온다. 
```java
public class Main {
    public static void main(String[] args) 
            throws InterruptedException {
        InventoryCounter inventoryCounter = new InventoryCounter();
        IncrementingThread incrementingThread = new IncrementingThread(inventoryCounter);
        DecrementingThread decrementingThread = new DecrementingThread(inventoryCounter);

        incrementingThread.start();
        incrementingThread.join();

        decrementingThread.start();
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
        private int items = 0;

        public void increment() {
            items++;
        }

        public void decrement() {
            items--;
        }

        public int getItems() {
            return items;
        }
    }
}

```

## <span style="color:#802548">_race condition_</span>
- 하지만 start시켜놓고 join을 몰아서 하면 예상하지 않은 결과가 도출된다.
  - static class이므로 inventoryCounter가 thread 간 공유되는 Heap에 저장되며, items도 static class의 멤버변수이므로 동일하다.
  - items에 관한 연산인 items++, items--가 atomic operation이 아니다. all or nothing이 이뤄지지 않은 것이다.
```java
public static void main(String[] args) 
        throws InterruptedException {
    InventoryCounter inventoryCounter = new InventoryCounter();
    IncrementingThread incrementingThread = new IncrementingThread(inventoryCounter);
    DecrementingThread decrementingThread = new DecrementingThread(inventoryCounter);

    incrementingThread.start();
    decrementingThread.start();

    incrementingThread.join();
    decrementingThread.join();

    System.out.println("We currently have " + inventoryCounter.getItems() + " items"); //-176도 나오고, -851도 나오고...
}
```

- 그럼 왜 items++는 원자적 연산이 아닌가? 이는 Java에서는 단일 연산으로 보이지만, 실제론 3개 연산을 같이 진행하기 때문이다.
  - items의 현재 value를 가져오고
  - 1을 더한 뒤
  - items에 해당 value를 저장한다
- 이 3가지 과정이 multi thread 간에 작동하면 OS의 스케쥴링에 따라 아래와 같은 시나리오도 가능하다.
  - (1) IncrementingThread에서  items 값을 가져온다.
  - (2) IncrementingThread에서  items 값을 1을 더한다.
  - (3) 아직 저장을 하지 않았는데, DecrementingThread에서 items 값을 가져오니 0이다.
  - (4) DecrementingThread에서 해당 items 값에 -1을 한다.
  - (5) DecrementingThread에서 items 값에 -1한 새로운 값을 저장한다.
  - (6) IncrementingThread에서에서 새로운 items 값은 1이기 때문에, 1로 저장한다.
  - (7) heap에서 items는 -1이었다가 1로 바뀌게 된다.
- 이 3가지 과정이 multi thread 간에 작동하면 OS의 스케쥴링에 따라 아래와 같은 시나리오도 가능하다.
  - (1) IncrementingThread에서  공유된 items 값을 가져온다.
  - (2) IncrementingThread에서  items 값을 1을 더한다.
  - (4) DecrementingThread에서  공유된 items 값을 가져온다. 아직 IncrementingThread에서 값을 저장하지 않아 0이다.
  - (5) DecrementingThread에서 items 값에 -1연산을 한다.
  - (6) IncrementingThread에서 새로운 items 값은 1이기 때문에, 1로 저장한다.
  - (3) DecrementingThread에서 새로운 items 값은 -1이기 때문에 -1로 저장된다.
  - (7) heap에서 items는 1이었다가 -1로 바뀌게 된다.

## <span style="color:#802548">_lock걸기- synchronized_</span>
- 위와 같은 현상을 race condition이라고 하며, heap, file system, DB 등 공유자원을 어느 떄나 접근할 수 있게 만들어서 생긴 현상이다.
- 이를 막기 위해선 critical section을 만들어줘야 한다. critical section을 만들면 method의 특정 연산 영역을 묶어 lock을 걸수 있다.
- 그 방법 중 하나가 바로 synchronized다. 다른 thread가 접근하지 못하게 막는 기능을 한다.
```java
void aggregateFunction() {
    entry 임계점 (critical Section)
    operation1();
    operation2();
    operation3();
    exit critical section
}
```


- 필요한 method에만 걸어주면 된다.
- 아래와 같이 synchronized를 걸면, items의 result가 중구난방으로 나오는 현상이 해결된다.
```java
private static class InventoryCounter {
        private int items = 0;

        public synchronized void increment() {
            items++;
        }

        public synchronized  void decrement() {
            items--;
        }

        public int getItems() {
            return items;
        }
    }
```


- start, start, join, join임에도 items는 의도한대로 0으로 나오는 것을 볼 수 있다.

```java
public static void main(String[] args) 
        throws InterruptedException {
    InventoryCounter inventoryCounter = new InventoryCounter();
    IncrementingThread incrementingThread = new IncrementingThread(inventoryCounter);
    DecrementingThread decrementingThread = new DecrementingThread(inventoryCounter);

    incrementingThread.start();
    decrementingThread.start();

    incrementingThread.join();
    decrementingThread.join();

    System.out.println("We currently have " + inventoryCounter.getItems() + " items"); //-176이 나오지 않고, 무조건 0이 나오게 된다.
}
```

- method의 일부에만 synchorinzed를 거는 것도 가능하다.
- 필요한 영역만 lock을 걸게 되어 좀 더 반응성이 높아진다.
- method와 무관한 logic이 있다면 해당부분까지는 동시실행이 가능해 실행시간 상에서 이득이 있다.
```java
private static class InventoryCounter {
        private int items = 0;

        Object lock = new Object();

        public synchronized void increment() {
            synchronized(this.lock) {
                items++;
            }
        }

        public synchronized  void decrement() {
            synchronized(this.lock) {
                items--;
            }
        }

        public int getItems() {
            synchronized(this.lock) {
               return items;
            }
        }
    }
```


- thread1의 sharedObject가 increment()를 하는 동안, thread2의 sharedObject가 decrement()할 수 없다.
- 그 이유는 같은 객체의 경우, synchronized가 걸리면 다른 synchronized가 붙은 method도 같이 block되기 때문이다.
- 즉, 객체 단위로 lock이 걸리는 것이다.
```java
public class Main {
    public static void main(String [] args) {
        SharedClass sharedObject = new SharedClass();
 
        Thread thread1 = new Thread(() -> {
            while (true) {
                sharedObject.increment();
            }
        });
 
        Thread thread2 = new Thread(() -> {
            while (true) {
                sharedObject.decrement();
            }
        });
 
        thread1.start();
        thread2.start();
    }
 
    static class SharedClass {
        private int counter = 0;
 
        public synchronized void increment() {
            this.counter++;
        }
 
        public synchronized void decrement() {
            this.counter--;
        }
    }
}
```


- 마찬가지의 이유로 sharedObject1과 sharedObject2는 서로 다른 객체다.
- 따라서 increment()가 각자 동시에 실행된다. 서로를 block하지 않는다.
```java
public class Main {
    public static void main(String [] args) {
        SharedClass sharedObject1 = new SharedClass();
        SharedClass sharedObject2 = new SharedClass();
 
        Thread thread1 = new Thread(() -> {
            while (true) {
                sharedObject1.increment();
            }
        });
 
        Thread thread2 = new Thread(() -> {
            while (true) {
                sharedObject2.increment();
            }
        });
 
        thread1.start();
        thread2.start();
    }
 
    static class SharedClass {
        private int counter = 0;
 
        public synchronized void increment() {
            this.counter++;
        }
    }
}
```

- 그와 동일한 이유로 아래와 같이 lock을 거는 객체가 서로 다르면 독립적으로 취급된다.
- 따라서 동시에 실행이 가능하다. 위의 예제는 class를 두개로 만들었다.
  - 그러나 실제론 static class는 하나의 객체만 사용하고, static class 안에서 lock 객체를 다르게 쓸 것이다.
```java
public class Main {
    public static void main(String [] args) {
        SharedClass sharedObject = new SharedClass();
 
        Thread thread1 = new Thread(() -> {
            while (true) {
                sharedObject.incrementCounter1();
            }
        });
 
        Thread thread2 = new Thread(() -> {
            while (true) {
                sharedObject.incrementCounter2();
            }
        });
 
        thread1.start();
        thread2.start();
    }
 
    static class SharedClass {
        private int counter1 = 0;
        private int counter2 = 0;
 
        private Object lock1 = new Object();
        private Object lock2 = new Object();
 
        public void incrementCounter1() {
            synchronized (lock1) {
                this.counter1++;
            }
        }
 
        public void incrementCounter2() {
            synchronized (lock2) {
                this.counter2++;
            }
        }
    }
}
```

