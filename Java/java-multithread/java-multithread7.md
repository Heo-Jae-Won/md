## <span style="color:#802548">_thread를 대기시켰다가 꺠우는 condition_</span>
- interrupt()를 통해 다른 thread에 신호를 보냈다.
- join()을 통해서도 다른 thread에 신호를 보낸다.
- semaphore를 통해서도 다른 thread에 신호를 보낸다.
  - 다만 잘 안 쓰인다.    
- Condition 객체를 사용해서도 다른 thread에 신호를 보낸다.
  - UI thread가 있고, DB 접속하는 thread가 따로 있다.
  - DB 접근 thread는 오래 걸려 반응성이 낮기 때문이다.
  - UI thread에 값이 있으면 DB 접근 Thread가 작동해야 한다.


<br/>

- UI thread에서 singal()을 보낼떄까지 DB thread는 sleep한다.
```java
//DB 접근 thread
Lock lock = new ReentrantLock();
Condition condition = lock.newCondition();
String username = null;
String password = null;

lock.lock();
try {
    while(username == null || password == null) {
        condition.await(); 
    }  
} finally {
    lock.unlock();
}

authenticateUser();
```

- DB접근 thread가 await가 발동하고, sleep 되며 lock을 unlock한다.
- 그 덕에 UI thread는 lock을 얻었다. 그래서 UI에 값을 채운다.
- 그리고 값이 없어 sleep에 빠진 DB 접근 thread를 깨운다. (signal)
- DB 접근 thread가 일어나도 UI thread가 lock을 unlock해야 DB 접근 thread의 lock.lock() 이후의 임계영역 코드가 진행된다.
- unlock되어 진행되게 되면 값이 있다면 while문을 빠져나와 doStuff()가 진행된다.
```java
lock.lock();
try {
    username = userTextbox.getText();
    password = passwordTextbox.getText();
    condition.signal();
} finally {
    lock.unlock();
}
```


## <span style="color:#802548">_thread를 대기시켰다가 꺠우는 wait와 notify_</span>
- wait가 호출되면 CPU를 반환하며 대기상태가 된다.
- notify()를 호출하면 대기중인 모든 thread 중 하나만 선택하여 실행 중 상태로 변경한다.
- notifyAll()을 호출하면 대기중인 모든 thread를 실행중 상태로 변경한다.
- thread1이 start되었을 때, isCompleted가 false이므로 wait()가 발동해 대기상태에 들어간다.
- 이를 다시 꺠우려면 thread2가 flag 변수를 true로 만들고 notify()를 호출하여 대기중인 thread를 실행 중 상태로 변경해주면 된다.
```java
public class SomeClass {
    boolean isCompleted = false;
    
    // Executed by thread1
    public synchronized void declareSuccess() throws InterruptedException {
        while (!isCompleted) {
            wait();
        }
        
        System.out.println("Success!!");
    }
    
    // Executed by thread2
    public synchronized void finishWork() {
       isCompleted = true;
       notify();
    }
}
```


- 조건 변수 대신 wait()와 notify()를 사용한 예제가 아래와 같이 또 있다.
```java
public class SimpleCountDownLatch {
    private int count;
 
    public SimpleCountDownLatch(int count) {
        this.count = count;
        if (count < 0) {
            throw new IllegalArgumentException("count cannot be negative");
        }
    }
 
    /**
     * Causes the current thread to wait until the latch has counted down to zero.
     * If the current count is already zero then this method returns immediately.
    */
    public void await() throws InterruptedException {
        synchronized (this) {
            while (count > 0) {
                this.wait();
            }
        }
    }
 
    /**
     *  Decrements the count of the latch, releasing all waiting threads when the count reaches zero.
     *  If the current count already equals zero then nothing happens.
     */
    public void countDown() {
        synchronized (this) {
            if (count > 0) {
                count--;
                
                if (count == 0) {
                    this.notifyAll();
                }
            }
        }
    }
 
    /**
     * Returns the current count.
    */
    public int getCount() {
        return this.count;
    }
}
```



## <span style="color:#802548">_lock free 구조를 위한 atomic package_</span>
- lock은 동시성 제어에 도움을 준다. 
  - 그러나 deadlock, 기아 문제, lock 경합에 따른 context switching 등의 문제가 있다.
  - 결국 이는 non-atomic 연산으로 인해 벌어진 일이다.
  - Java에서는 count++는 하드웨어 수준에서는 3개의 명령어기에 non-atomic이다.
    - Java에서는 그를 위해 volatile keyword와 atomic package가 있다.
    - 이 중 atomic package를 이용해 lock-free 구조를 만들 수 있다.

- 이전에 했던 Inventory 예제를 생각해보면, synchronized method를 사용했었다.
```java
public class Main {
    public static void main(String[] args) 
            throws InterruptedException {
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
        private int items = 0; // method에 synchronized가 필수라 변수를 volatile keyword를 붙일 필요가 없다.

        public synchronized void increment() { 
            items++;
        }

        public synchronized void decrement() {
            items--;
        }

        public int getItems() {//synchronized 추가 필요 없음
            return items;
        }
    }
}
```

- 이를 atomic으로 바꾸면 아래와 같이 바꿀 수 있다.
- synchronized keyword가 사라졌다.
```java
private static class InventoryCounter {
    private AtomicInteger items = new AtomicInteger(0);

    public void increment() {
        items.incrementAndGet();
    }

    public void decrement() {
        items.decrementAndGet();
    }

    public int getItems() {
        return items.get();
    }
}
```

- atomic이기에 무조건 원자적일거 같지만, method를 섞어 쓰면 원자성이 깨진다.
- 아래에서 incrementAndGet하고 addAndGet을 같이 썼기에 각자는 원자적이나 method 단위로는 실제로는 비원자적 연산이 된다.
  - 하나의 변수는 읽고 쓰는 걸 한 method 안에서 하면 안된다.
  - 이는 두개의 변수도 마찬가지다. 한 method 안에서 읽거나 쓰는 것은 하면 안된다.
```java
int initialValue = 0;
AtomicInteger atomicInteger = new AtomicInteger(initialValue);

atomicInteger.incrementAndGet();
atomicInteger.addAndGet(-5);
```

- 두 개 변수인 count와 sum을 하나의 method에서 읽고 수정하면 각각은 원자적이다.
- 그러나 두개를 같이 연산하게 되는 CPU의 입장에서는 원자적인 연산이 아니게 된다.
- 그 말인 즉슨 addSample 함수는 atomic하지 않다는 의미이며, 다른 방식이 필요하다는 의미다.
- 그 방식은 AtomicLong이 아닌 AtomicReference의 CAS 함수다.
```java
public class Metric {
    private AtomicLong count = new AtomicLong(0);
    private AtomicLong sum = new AtomicLong(0);

    public void addSample(long sample) {
        sum.addAndGet(sample);
        count.incrementAndGet();
    }

    public double getAverage() {
        double average = (double)sum.get()/count.get();
        reset();
        return average;
    }

    private void reset() {
        count.set(0);
        sum.set(0);
    }
}
```

- compareAndSet은 all or nothing을 보장하여 원자적 연산으로 이뤄지게 해준다.
- 그러나 기억해야 할 점은 multi-threading이 늘 그렇듯, 성능이 반드시 좋아진다는 보장은 없다.
- 그래도 보통은 LockFree가 더 빠른 편이다. 실제로 구현해보자.
  - 다른 thread에 의해 바뀌지 않았다면 if 조건이 true가 되어 break된다.
  - 반면에 다른 thread에 의해 바뀌었다면 compareAndSet이 false를 반환한다.
  - 그 후 else문에서 1ms간 대기했다가 다시 while문을 읽어나간다.
```java
public static class LockFreeStack<T> {
    private AtomicReference<StackNode<T>> head = new AtomicReference<>();
    private AtomicInteger counter = new AtomicInteger(0);

    public void push(T value) {
        StackNode<T> newHeadNode = new StackNode<>(value);

        while (true) {
            StackNode<T> currentHeadNode = head.get();
            newHeadNode.next = currentHeadNode;

            //CAS 발동
            if (head.compareAndSet(currentHeadNode, newHeadNode)) {
                break;
            } else {
                LockSupport.parkNanos(1);
            }
        }
        counter.incrementAndGet(); //atomic 변수 연산
    }

    public T pop() {
        StackNode<T> currentHeadNode = head.get();
        StackNode<T> newHeadNode;

        while (currentHeadNode != null) {
            newHeadNode = currentHeadNode.next;

            //CAS 발동
            if (head.compareAndSet(currentHeadNode, newHeadNode)) {
                break;
            } else {
                LockSupport.parkNanos(1);
                currentHeadNode = head.get();
            }
        }
        counter.incrementAndGet();
        return currentHeadNode != null ? currentHeadNode.value : null;
    }

    public int getCounter() {
        return counter.get();
    }
}
```


- atomic package를 이용하지 않은 thread-safe stack class는 아래와 같다.
  - synchronized keyword가 반드시 필수적이다.
  - 다만 getter는 그 자체로 원자적이기에 synchronized keyword가 필요없다.
```java
public static class StandardStack<T> {
    private StackNode<T> head;
    private int counter = 0;

    public synchronized void push(T value) {
        StackNode<T> newHead = new StackNode<>(value);
        newHead.next = head;
        head = newHead;
        counter++;
    }

    public synchronized T pop() {
        if (head == null) {
            counter++;
            return null;
        }

        T value = head.value;
        head = head.next;
        counter++;
        return value;
    }

    public int getCounter() {
        return counter;
    }
}
```

- 데이터를 저장하는 Node class는 아래와 같다.
```java
private static class StackNode<T> {
    public T value;
    public StackNode<T> next;

    public StackNode(T value) {
        this.value = value;
        this.next = next;
    }
}
```

- 이제 실제 main thread에서 thread를 열어주면 된다.
- StandardStack은 1억개를 연산하지만, LockFree는 8억개를 연산한다.
  - LockFree로 구현하면 보통은 성능 상에 이점이 있다.
  - 그 외에는 lock을 구현하기 위한 try catch 등을 쓰지 않아 소스코드가 가독성이 좋아진다.
```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        //StandardStack<Integer> stack = new StandardStack<>();
        LockFreeStack<Integer> stack = new LockFreeStack<>();
        Random random = new Random();

        for (int i = 0; i < 100000; i++) {
            stack.push(random.nextInt());
        }

        List<Thread> threads = new ArrayList<>();

        int pushingThreads = 2;
        int poppingThreads = 2;

        for (int i = 0; i < pushingThreads; i++) {
            Thread thread = new Thread(() -> {
                while (true) {
                    stack.push(random.nextInt());
                }
            });

            thread.setDaemon(true); //interrupt를 구현하지 않으려고 그냥 daemon처리 해버림..
            threads.add(thread);    //thread는 daemon thread기 때문에 main thread가 끝나면 같이 종료됨.
        }

        for (int i = 0; i < poppingThreads; i++) {
            Thread thread = new Thread(() -> {
                while (true) {
                    stack.pop();
                }
            });

            thread.setDaemon(true);
            threads.add(thread);   //thread는 daemon thread기 때문에 main thread가 끝나면 같이 종료됨.
        }

        for (Thread thread : threads) {
            thread.start();
        }

        Thread.sleep(10000);

        System.out.println(String.format("%,d operations were performed in 10 seconds ", stack.getCounter()));
    }
}
```


- atomicReference class가 필요한 예시를 하나 더 살펴보자.
- 아래와 같이 AtomicLong class로 만들면, 원자적 연산을 보장할 수가 없다.
- Atomic원시형들은 연산을 읽고 쓰는 연산을 한번만 해야한다.
```java
public class Metric {
    private AtomicLong count = new AtomicLong(0);
    private AtomicLong sum = new AtomicLong(0);

    public void addSample(long sample) {
        sum.addAndGet(sample);
        count.incrementAndGet();
    }

    public double getAverage() {
        double average = (double)sum.get()/count.get();
        reset();
        return average;
    }

    private void reset() {
        count.set(0);
        sum.set(0);
    }
}
```

- 따라서 long같은 일반 원시형으로 변수를 만들고, atomicReference로 wrapping한다.
- 그리고 해당 값을 꺼내어 연산한 뒤에 compareAndSet을 통해 다른 thread에 의한 변경여부를 확인한다.
  - 변경되었다면 바꾸지 않고, 변경되지 않았다면 값을 바꾼다. 따라서 thread-safe하다.
  - 또한 do문 안에 있는 일련의 4개의 computation 과정이 모두 undo됐다.
  - 즉, all or nothing을 충족한다. 따라서 atomic operation이기도 하다.
  - 이런한 atomicity는 RDBMS의 transaction의 atomicity가 all or nothing인 것과 동일하다.
```java
private static class Metric {
    private static class InternalMetric{
        public long count;
        public long sum;
    }
    
    private AtomicReference<InternalMetric> internalMetric = new AtomicReference<>(new InternalMetric());
 
    public void addSample(long sample) {
        InternalMetric currentState;
        InternalMetric newState;
        do {
            currentState = internalMetric.get();
            newState = new InternalMetric();
            newState.sum = currentState.sum + sample;
            newState.count = currentState.count + 1;
        } while (!internalMetric.compareAndSet(currentState, newState));
    }
 
    public double getAverage() {
        InternalMetric newResetState = new InternalMetric();
        InternalMetric currentState;    
        double average;
        do {
            currentState = internalMetric.get();            
            average = (double)currentState.sum / currentState.count;
        } while (!internalMetric.compareAndSet(currentState,  newResetState));
        
        return average;
    }        
}
```
