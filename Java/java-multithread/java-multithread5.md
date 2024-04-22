- 아래와 같이 한 thread는 increment를 하고, 다른 thread는 data race를 기록한다.
- 그럼 우리의 예상과는 다르게 불변성이 유지되지 않는 것이 보인다.
- 이것이 바로 data race 현상이다.
- data race는 CPU의 비순차적 명령어 처리에서 기인한다.
- compiler와 CPU가 효율성이 높게 데이터를 처리하기 위한 최적화의 과정이다.
- 다만 논리결과를 어기지 않을 때만 일어난다.
```java
public class Main {
    public static void main(String[] args) {
        SharedClass sharedClass = new SharedClass();
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < Integer.MAX_VALUE; i++) {
                sharedClass.increment();
            }
        });

        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < Integer.MAX_VALUE; i++) {
                sharedClass.checkForDataRace();
            }

        });

        thread1.start();
        thread2.start();
    }

    public static class SharedClass {
        private int x = 0;
        private int y = 0;

        public void increment() {
            x++;
            y++;
        }

        public void checkForDataRace() {
            if (y > x) { //CPU에 따라 multithread인 경우 y++을 먼저 할수도 있음. 따라서 아래 if문이 나오게 됨.
                System.out.println("y > x - Data Race is detected");
            }
        }
    }
}
```

- 만약 아래와 같이 이전 논리연산에 의존하고 있다면 y > x 조건을 만족하는 경우는 일어나지 않는다.
```java
 public static class SharedClass {
        private int x = 0;
        private int y = 0;

        public void increment() {
            x = 1;
            y = x + 2;
            z = y + 10;
        }

        public void checkForDataRace() {
            if (y > x) {
                System.out.println("y > x - Data Race is detected");
            }
        }
    }
```


- 아래는 compiler나 cpu가 data race가 있다고 해도, 똑같은 로직이기 때문에 똑같은 method라고 판단하게 된다.
- 문제는 single thread일 때는 몰라도, multihtread일 때 문제가 된다는 점이다. 
```java
 public static class SharedClass {
        private int x = 0;
        private int y = 0;

        public void increment1() {
           x++;
           y++;
        }

        public void increment2() {
            y++;
            x++;
        }
    }
```

- 다른 콩에서 실행되는 스레드를 인지하지 못하고, 동일 변수를 읽고 특정 처리 순서에 의존한다는 점이다. 그래서 예상치 못한 잘못된 결과를 낳게 된다.
- Java에서는 이러한 현상을 방지하기 위해서 synchroinized를 활용했다.
- 또는 volatile을 활용했다. volatile이 synchronized보다 나은 점은 잠금 오버헤드가 줄어드는 이득이 있다는 점이다. 
  - 아래에선 long이나 int의 경우지만, 그 외의 경우에도 data race인 상황에선 volatile이 공유변수에 lock을 걸어주게 된다. 
```java
public class Main {
    public static void main(String[] args) {
        SharedClass sharedClass = new SharedClass();
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < Integer.MAX_VALUE; i++) {
                sharedClass.increment();
            }
        });

        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < Integer.MAX_VALUE; i++) {
                sharedClass.checkForDataRace();
            }

        });

        thread1.start();
        thread2.start();
    }

    public static class SharedClass {
        private volatile int x = 0; //volatile을 추가하여 y > x인 상황 없애버림.
        private volatile int y = 0;

        public void increment() {
            x++;
            y++;
        }

        public void checkForDataRace() {
            if (y > x) {
                System.out.println("y > x - Data Race is detected");
            }
        }
    }
}
```


- CPU의 명령어 재배열로 인해 다양한 시나리오가 만들어진다.
- 예시를 살펴보자.
- 아래의 경우에서, 지역변수인 local1과 local2가 2와 1이 되는 경우의 수가 있다.
```java
public static void main(String[] args) {
        SharedClass sharedClass = new SharedClass();
        
        Thread thread1 = new Thread(() -> sharedClass.method1());
        Thread thread2 = new Thread(() -> sharedClass.method2());
 
        thread1.start();
        thread2.start();
    }
 
   private static class SharedClass {
        int a = 0;
        int b = 0;
 
        public void method1() {
            int local1 = a;
            this.b = 1;
        }
 
        public void method2() {
            int local2 = b;
            this.a = 2;           
        }       
   }
```

- method1과 method2는 내부 연산에서 각각 독립적이므로 CPU에 의해 명령어 재배치가 가능하다.
- 따라서 method1이 아래와 같이 재배치될 수 있다.
```java
public void method1() {
    this.b = 1;
    int local1 = a;
}
```

- 그렇게 되면 아래와 같은 실행순서가 만들어지는 경우가 생긴다.
```
Thread 1: b = 1
Thread 2: local = b (1)
Thread 1: a = 2
Thread 2: local1 = a (2)
```


- locking 기법을 택할 때는 촘촘 vs 느슨 방법이 있다.
- 느슨하게 하면 병렬성이 낮아지는 느려진다.
- 촘촘하게 하면 병렬성이 높아져 빠른 대신 deadlock이 일어난다.
- deadlock은 아래와 같은 상황이다.
  - Thread 1은 Thread 2의 B lock이 풀리길 기다린다.
  - Thread 2는 Thread 1의 A lock이 풀리길 기다린다.
```
Thread1             Thread 2
1. lock(A)           
                    1. lock(B)
                    2. lock(A)
2. lock(B)
```


- 코드로 나타내면 아래와 같다.
```java
public class Main {
    public static void main(String[] args) {
        Intersection intersection = new Intersection();
        Thread trainAThread = new Thread(new TrainA(intersection));
        Thread trainBThread = new Thread(new TrainB(intersection));

        trainAThread.start();
        trainBThread.start();
    }

    public static class TrainB implements Runnable {
        private Intersection intersection;
        private Random random = new Random();

        public TrainB(Intersection intersection) {
            this.intersection = intersection;
        }

        @Override
        public void run() {
            while (true) {
                long sleepingTime = random.nextInt(5);
                try {
                    Thread.sleep(sleepingTime);
                } catch (InterruptedException e) {
                }

                intersection.takeRoadB();
            }
        }
    }

    public static class TrainA implements Runnable {
        private Intersection intersection;
        private Random random = new Random();

        public TrainA(Intersection intersection) {
            this.intersection = intersection;
        }

        @Override
        public void run() {
            while (true) {
                long sleepingTime = random.nextInt(5);
                try {
                    Thread.sleep(sleepingTime);
                } catch (InterruptedException e) {
                }

                intersection.takeRoadA();
            }
        }
    }

    public static class Intersection {
        private Object roadA = new Object();
        private Object roadB = new Object();

        public void takeRoadA() {
            synchronized (roadA) {
                System.out.println("Road A is locked by thread " + Thread.currentThread().getName());

                synchronized (roadB) {
                    System.out.println("Train is passing through road A");
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                    }
                }
            }
        }

        public void takeRoadB() {
            synchronized (roadB) {
                System.out.println("Road B is locked by thread " + Thread.currentThread().getName());

                synchronized (roadA) {
                    System.out.println("Train is passing through road B");

                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                    }
                }
            }
        }
    }
}
```
```
Road B is locked by thread Thread-1
Train is passing through road B
Road B is locked by thread Thread-1
Train is passing through road B
Road A is locked by thread Thread-0
Train is passing through road A
Road B is locked by thread Thread-1
Train is passing through road B
Road A is locked by thread Thread-0
Train is passing through road A
Road B is locked by thread Thread-1
Road A is locked by thread Thread-0
```

- deadlock을 발생하는 경우는 아래와 같다.
  - (1). 상호 배제
  - (2). Hold and wait
  - (3). wait until release
  - (4). circualr wait
- deadlock을 일어나지 않게 하는 건 4개 중 하나라도 안되게 하면 된다.
  - circualr wait를 막으려면 lock의 순서를 동일하게 적용해주면 된다.
  - unlock의 순서는 중요하지 않다.
  - 아래와 같다.
```
Thread1             Thread 2
1. lock(A)           
                    1. lock(A)
                    2. lock(B)
2. lock(B)
```

- 소스코드로는 아래와 같다.
```java
public static class Intersection {
        private Object roadA = new Object();
        private Object roadB = new Object();

        public void takeRoadA() {
            synchronized (roadA) {
                System.out.println("Road A is locked by thread " + Thread.currentThread().getName());

                synchronized (roadB) {
                    System.out.println("Train is passing through road A");
                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                    }
                }
            }
        }

        public void takeRoadB() {
            synchronized (roadA) { //roadB를 roadA로 바꿔 Thread1과 같은 lock 순서 유지
                System.out.println("Road B is locked by thread " + Thread.currentThread().getName());

                synchronized (roadB) { //roadA를 roadB로 바꿔 Thread1과 같은 lock 순서 유지
                    System.out.println("Train is passing through road B");

                    try {
                        Thread.sleep(1);
                    } catch (InterruptedException e) {
                    }
                }
            }
        }
    }
```

- lock이 적을 때는 아래와 같이 lock 순서를 유지해주는 게 최고다.
- 그러나 만약 lock이 많다면 tryLock, thread interrupt 같은 기술을 사용해야 한다. 다만 tryLock이나 thread interrupt는 synchronized와 같이 사용될 수 없다.
- thread interrupt는 interrupt()와 다른 것이다.


- 아래와 같은 방식으로 모든 method에 synchronized를 걸게 되면 suspend가 흔하게 걸리게 된다.
```java
public class Metrics {
        private long count;
        private double average;
        private long max;
 
        public synchronized void addSample(long sample) {
            average = (average * count + sample) / (++count);
            max = Math.max(max, sample);
        }
 
        public synchronized void reset() {
            count = 0;
            max = Integer.MIN_VALUE;
            average = 0.0;
        }
 
        public synchronized long getCount() {
            return count;
        }
 
        public synchronized long getMax() {
            return max;
        }
 
        public synchronized double getAverage() {
            return average;
        }
    }

- 그래서 단계별로 lock을 걸면 deadlock에 빠지게 된다.
  - reset method가 maxLock을 쥐고 있고 countLock을 기다린다.
  - addSample method가 countLock을 쥐고 있고 maxLock을 기다린다.  
- 따라서 아래와 같이도 안 된다.
```java
public class Metrics {
        private long count;
        private double average;
        private long max;
 
        private Object countLock = new Object();
        private Object averageLock = new Object();
        private Object maxLock = new Object();
 
        public void addSample(long sample) {
            synchronized (countLock) {
                synchronized (averageLock) {
                    synchronized (maxLock) {
                        average = (average * count + sample)/(++count);
                        max = Math.max(max, sample);
                    }
                }
            }
        }
        
        public void reset() {
            synchronized (maxLock) {
                synchronized (averageLock) {
                    synchronized (countLock) {
                        count = 0;
                        max = Integer.MIN_VALUE;
                        average = 0.0;
                    }
                }
            }
        }
 
        public long getCount() {
            synchronized (countLock) {
                return count;
            }
        }
 
        public long getMax() {
            synchronized (maxLock) {
                return max;
            }
        }
 
        public double getAverage() {
            synchronized (averageLock) {
                return average;
            }
        }
    }
```

- 그렇다고 아래와 같이 synchronized를 없애면 멀티스레딩에서 data race와 race condition에 의해 예상하지 않은 결과를 얻게 된다.
```java
  public class Metrics {
        private long count;
        private double average;
        private long max;
 
        public synchronized void addSample(long sample) {
            average = (average * count + sample) / (++count);
            max = Math.max(max, sample);
        }
 
        public synchronized void reset() {
            count = 0;
            max = Integer.MIN_VALUE;
            average = 0.0;
        }
 
        public long getCount() {
            return count;
        }
 
        public long getMax() {
            return max;
        }
 
        public double getAverage() {
            return average;
        }
    }
```


- 아래와 같은 방식이 가장 바람직하다.
  - getter setter에는 synchronized를 붙이지 않는다.
  - 비원자적 연산인 addSample에는 synchronized가 필요하다. 
  - reset은 사실상 setter기 때문에 synchronized를 붙일 필요가 없다. 원자적 연산이다.
```java
public class Metrics {
    private volatile long count;
    private volatile double average;
    private volatile long max;

    public synchronized void addSample(long sample) {
        average = (average * count + sample) / (++count);
        max = Math.max(max, sample);
    }

    public synchronized void reset() {
        count = 0;
        max = Integer.MIN_VALUE;
        average = 0.0;
    }

    public long getCount() {
        return count;
    }

    public long getMax() {
        return max;
    }

    public double getAverage() {
        return average;
    }
}
```




