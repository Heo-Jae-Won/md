```java
Thread thread = new Thread(new Runnable() {

			@Override
			public void run() {
				// TODO Auto-generated method stub
				System.out.println("We are now in thread " + Thread.currentThread().getName()); //eclipse랑 다르게 thread에만 breakpoint 잡은 거 아니면 thread 안의 breakpoint가 잡히지 않는다.
				System.out.println("We are now in thread " + Thread.currentThread().getPriority());

			}
		});
		
		thread.setName("New Worker Thread"); //debugging할 때 나중에 의미 있는 이름이 필요함.
		thread.setPriority(Thread.MAX_PRIORITY); //OS에 thread의 동적 우선순위를 알려줌. 10이고. 제일 우선순위 떨어지는 것.
		System.out.println("we ard in thread: " + Thread.currentThread().getName() + " before starting a new thread");
		thread.start();
		System.out.println("we ard in thread: " + Thread.currentThread().getName() + " after starting a new thread");

		//Thread.sleep(10_000);// OS에 지시하는 것. 이 시간 동안 CPU르 쓰지 않음.
		/*
		 * we ard in thread: main before starting a new thread 
		 * we ard in thread: main after starting a new thread 
		 * We are now in thread Thread-0
		 */
		//start하는 데 시간이 걸려서 main after가 먼저 나옴.
		
		
		Thread thread1 = new Thread(new Runnable() {

			@Override
			public void run() {
				throw new RuntimeException("Internal Exception");
			}
		});
		
		thread1.setName("misbehaving thread");
		thread1.setUncaughtExceptionHandler(new UncaughtExceptionHandler() {
			@Override
			public void uncaughtException(Thread t, Throwable e) {
				System.out.println("A critical Error happened in thread" + t.getName() + " the error is " + e.getMessage());				
			}
		});
		thread1.start();
```

```java
public class Main {
    public static final int MAX_PASSWORD = 9999;

    public static void main(String[] args) {
        Random random = new Random();

        Vault vault = new Vault(random.nextInt(MAX_PASSWORD));

        List<Thread> threads = new ArrayList<>();

        threads.add(new AscendingHackerThread(vault));
        threads.add(new DescendingHackerThread(vault));
        threads.add(new PoliceThread());

        for (Thread thread : threads) {
            thread.start();
        }
    }

    private static class Vault {
        private int password;

        public Vault(int password) {
            this.password = password;
        }

        public boolean isCorrectPassword(int guess) {
            try {
                Thread.sleep(5);
            } catch (InterruptedException e) {
            }
            return this.password == guess;
        }
    }

    private static abstract class HackerThread extends Thread {
        protected Vault vault;

        public HackerThread(Vault vault) {
            this.vault = vault;
            this.setName(this.getClass().getSimpleName());
            this.setPriority(Thread.MAX_PRIORITY);
        }

        @Override
        public void start() {
            System.out.println("Starting thread " + this.getName());
            super.start();
        }
    }

    private static class AscendingHackerThread extends HackerThread {

        public AscendingHackerThread(Vault vault) {
            super(vault);
        }

        @Override
        public void run() {
            for (int guess = 0; guess < MAX_PASSWORD; guess++) {
                if (vault.isCorrectPassword(guess)) {
                    System.out.println(this.getName() + " guessed the password " + guess);
                    System.exit(0);
                }
            }
        }
    }

    private static class DescendingHackerThread extends HackerThread {

        public DescendingHackerThread(Vault vault) {
            super(vault);
        }

        @Override
        public void run() {
            for (int guess = MAX_PASSWORD; guess >= 0; guess--) {
                if (vault.isCorrectPassword(guess)) {
                    System.out.println(this.getName() + " guessed the password " + guess);
                    System.exit(0);
                }
            }
        }
    }

    private static class PoliceThread extends Thread {
        @Override
        public void run() {
            for (int i = 10; i > 0; i--) {
                try {
                    Thread.sleep(1000);
                } catch (InterruptedException e) {
                }
                System.out.println(i);
            }

            System.out.println("Game over for you hackers");
            System.exit(0);
        }
    }
}
```

```java
public class MultiExecutor {
    
    private final List<Runnable> tasks;
 
    /*
     * @param tasks to executed concurrently
     */
    public MultiExecutor(List<Runnable> tasks) {
        this.tasks = tasks;
    }
 
    /**
     * Executes all the tasks concurrently
     */
    public void executeAll() {
        List<Thread> threads = new ArrayList<>(tasks.size());
        
        for (Runnable task : tasks) {
            Thread thread = new Thread(task);
            threads.add(thread);
        }
        
        for(Thread thread : threads) {
            thread.start();
        }
    }
}
```

```java

	//thread는 아무것도 안해도 커널과 메모리의 자원을 잡아먹는다. 안 쓰는 스레드는 제거해야 한다.
	//thread가 하나만 살아있어도 process가 죽지 않는다. 따라서 application을 죽이기 전에 모든 thread를 종료시켜야 한다.
	//거기에 쓰이는 method가 interrupt()다.
	public static void main(String[] args) throws Exception {
		Thread thread = new Thread(new BlockingTask());
		thread.start();
		thread.interrupt();
		//System.out.println("thread is over"); 이렇게 써도 start가 먼저시작되지 않으면 이놈이 먼저 뜸.

		Thread thread1 = new Thread(new LongComputationTask(new BigInteger("20000"),new BigInteger("100000")));
		thread1.start();
		thread1.interrupt(); //이것만으로는 부족. 해당 thread의 run method에서 실제 오래 걸리는 작업에 flag를 붙여줘야 함.
		//따라서 아래 pow()의 for문 안에 if문으로 flag를 줌. 그럼 interrupted가 된 것을 알고 thread 정지  
		
		
		Thread thread2 = new Thread(new LongComputationTask1(new BigInteger("2000"),new BigInteger("10000")));
		thread2.setDaemon(true); //만약 if문으로 어떤 처리를 따로 할 필요가 없다고 생각한다면 간단하게 이렇게 처리 가능. 그럼 interrupt됨.
		thread2.start();
		thread2.interrupt();
	}
	
	private static class BlockingTask implements Runnable {
		
		@Override
		public void run() {
			try {
				Thread.sleep(50000);
			} catch(InterruptedException e) {
				System.out.println("Exiting blocking thread");
			}
		}
	}
	
	private static class LongComputationTask implements Runnable {
        private BigInteger base;
        private BigInteger power;

        public LongComputationTask(BigInteger base, BigInteger power) {
            this.base = base;
            this.power = power;
        }

        @Override
        public void run() {
            System.out.println(base + "^" + power + " = " + pow(base, power));
        }

        private BigInteger pow(BigInteger base, BigInteger power) {
            BigInteger result = BigInteger.ONE;

            for (BigInteger i = BigInteger.ZERO; i.compareTo(power) != 0; i = i.add(BigInteger.ONE)) {
                if (Thread.currentThread().isInterrupted()) {
                    System.out.println("Prematurely interrupted computation");
                    return BigInteger.ZERO;
                }
                result = result.multiply(base);
            }

            return result;
        }
    }
	
	private static class LongComputationTask1 implements Runnable {
        private BigInteger base;
        private BigInteger power;

        public LongComputationTask1(BigInteger base, BigInteger power) {
            this.base = base;
            this.power = power;
        }

        @Override
        public void run() {
            System.out.println(base + "^" + power + " = " + pow(base, power));
        }

        private BigInteger pow(BigInteger base, BigInteger power) {
            BigInteger result = BigInteger.ONE;

            for (BigInteger i = BigInteger.ZERO; i.compareTo(power) != 0; i = i.add(BigInteger.ONE)) {
                result = result.multiply(base);
            }

            return result;
        }
    }
```

```java
//맞습니다. 애플리케이션을 프로그램적으로 중단할 수 있는 유일한 방법은 스레드를 데몬으로 만드는 것입니다. 안타깝게도 System.in.read()는 Thread.interrupt();에 응답하지 않습니다.
 public static void main(String [] args) {
        Thread thread = new Thread(new WaitingForUserInput());
        thread.setName("InputWaitingThread");
		//thread.setDaemon(true);
        thread.start();
		//thread.interrupt();

    }
 
    private static class WaitingForUserInput implements Runnable {
        @Override
        public void run() {
            try {
                while (true) {
                    char input = (char) System.in.read();
                    if(input == 'q') {
                        return;
                    }
                }
            } catch (IOException e) {
                System.out.println("An exception was caught " + e);
            };
        }
    }
```

```java
    public static void main(String [] args) {
        Thread thread = new Thread(new SleepingThread());
        thread.start();
        thread.interrupt();
				// thread.interrupt()는 다른 스레드에게 중단하라는 신호를 보내는 방법일 뿐입니다. 중단이 가능한 경우, 중단을 하는 건 사용자의 몫입니다.
				// 아래 catch문에 return을 추가할 필요가 있습니다.
    }
 
    private static class SleepingThread implements Runnable {
        @Override
        public void run() {
            while (true) {
                try {
                    Thread.sleep(1000000);
                } catch (InterruptedException e) {
                }
            }
        }
    }
```

```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        List<Long> inputNumbers = Arrays.asList(100000000L, 3435L, 35435L, 2324L, 4656L, 23L, 5556L);

        List<FactorialThread> threads = new ArrayList<>();

        for (long inputNumber : inputNumbers) {
            threads.add(new FactorialThread(inputNumber));
        }

        for (Thread thread : threads) {
            thread.setDaemon(true); //join으로 기다리지 않고 결과를 받아오게 하면, 계속 running임. 따라서 죽일 수 있게 설정.
                                    //아니면 interrupt 설정을 해줘야 함.
            thread.start();
        }

        //join이 없이는 race condition이 되어 매우 비효율적으로 cpu 자원을 소모.
        //이걸 해야 main thread가 계승 thread가 끝날 때까지 기다리게 함.
        //그러나 thread가 너무 시간이 오래 걸릴 수도 있음. 큰 수를 계산할 때 그러함. 그럴 때 기다리는 시간을 줌.
        //thread.join(2000);이 그 예시
        // for (Thread thread : threads) {
        //     thread.join(2000);
        // }

        for (int i = 0; i < inputNumbers.size(); i++) {
            FactorialThread factorialThread = threads.get(i);
            if (factorialThread.isFinished()) {
                System.out.println("Factorial of " + inputNumbers.get(i) + " is " + factorialThread.getResult());
            } else {
                System.out.println("The calculation for " + inputNumbers.get(i) + " is still in progress");
            }
        }
    }

    public static class FactorialThread extends Thread {
        private long inputNumber;
        private BigInteger result = BigInteger.ZERO;
        private boolean isFinished = false;

        public FactorialThread(long inputNumber) {
            this.inputNumber = inputNumber;
        }

        @Override
        public void run() {
            this.result = factorial(inputNumber);
            this.isFinished = true;
        }

        public BigInteger factorial(long n) {
            BigInteger tempResult = BigInteger.ONE;

            for (long i = n; i > 0; i--) {
                tempResult = tempResult.multiply(new BigInteger((Long.toString(i))));
            }
            return tempResult;
        }

        public BigInteger getResult() {
            return result;
        }

        public boolean isFinished() {
            return isFinished;
        }
    }
}
```

```java
import java.math.BigInteger;
import java.util.List;
import java.util.ArrayList;

public class ComplexCalculation {
    public BigInteger calculateResult(BigInteger base1, BigInteger power1, BigInteger base2, BigInteger power2) {
        BigInteger result;
        List<PowerCalculatingThread> threads = new ArrayList<>();
        PowerCalculatingThread thread1 = new PowerCalculatingThread(new BigInteger((Long.toString(21))) ,new BigInteger((Long.toString(5))));
        PowerCalculatingThread thread2 = new PowerCalculatingThread(new BigInteger((Long.toString(10))) ,new BigInteger((Long.toString(3))));
        threads.add(thread1);
        threads.add(thread2);
        
        for (Thread thread: threads) {
            thread.setDaemon(true);
            thread.start();
        }
        
        for (Thread thread: threads) {
            try {
                 thread.join(2000);
            } catch(InterruptedException e) {
                System.out.println("error is: " + e);
            }
           
        }
    
        result = thread1.getResult().add(thread2.getResult());
        return result;
    }

    private static class PowerCalculatingThread extends Thread {
        private BigInteger result = BigInteger.ONE;
        private BigInteger base;
        private BigInteger power;
    
        public PowerCalculatingThread(BigInteger base, BigInteger power) {
            this.base = base;
            this.power = power;
        }
    
        @Override
        public void run() {
          result =  base.pow(power.intValue());
        }
    
        public BigInteger getResult() { return result; }
    }
}
```

```java

public class ComplexCalculation {
    public BigInteger calculateResult(BigInteger base1, 
                                      BigInteger power1, 
                                      BigInteger base2, 
                                      BigInteger power2) {
        BigInteger result;
        PowerCalculatingThread thread1 = new PowerCalculatingThread(base1, power1);
        PowerCalculatingThread thread2 = new PowerCalculatingThread(base2, power2);
 
        thread1.start();
        thread2.start();
 
        try {
            thread1.join();
            thread2.join();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
 
        result = thread1.getResult().add(thread2.getResult());
        return result;
    }
 
    private static class PowerCalculatingThread extends Thread {
        private BigInteger result = BigInteger.ONE;
        private BigInteger base;
        private BigInteger power;
 
        public PowerCalculatingThread(BigInteger base, BigInteger power) {
            this.base = base;
            this.power = power;
        }
 
        @Override
        public void run() {
            for(BigInteger i = BigInteger.ZERO;
                i.compareTo(power) !=0;
                i = i.add(BigInteger.ONE)) {
                result = result.multiply(base);
            }
        }
 
        public BigInteger getResult() {
            return result;
        }
    }
}
```

- 중요한 건 latency(지연시간)과 throughput(처리량)이다.
- web에서는 그게 중요하다.
- task를 subtask로 줄이고 싶을 떄는, 현재 돌아가는 process의 수와 CPU 코어의 수를 알아야 한다.
- task를 나누는 데에도 컴퓨터 자원이 필요하고, 나눠진 task를 종합하는 데도 컴퓨터 자원이 필요하다.
- 따라서 늘 task를 나누는 게 좋은 결과를 초래하진 않는다.
- 다만 기존 작업이 무거울수록 나누는 게 좋고, 별것도 아닌것이라면 나누지 않는 게 좋다는 것이 현실에서 밝혀졌다.