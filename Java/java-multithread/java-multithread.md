## <span style="color:#802548">_thread 생성_</span>
- Thread는 아래와 같이 Runnable interface를 생성자에 넣어 구현할 수 있다.
- name과 priority를 설정하여 start시킨다.
```java
Thread thread = new Thread(new Runnable() {

        @Override
        public void run() {
            System.out.println("We are now in thread " + Thread.currentThread().getName()); //eclipse는 hread에만 breakpoint 잡은 거 아니면 thread 안의 breakpoint가 잡히지 않는다. intelliijay는 잡아준다.
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

- Thread를 생성시켜 작동시킨 예제다.
- 금고 class를 만든다. 
```java
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
```

- 금고를 풀려고 하는 Hacker thread class를 만든다.
- HackerThread는 abstract class이므로 이를 실제로 구현할 Thread class를 따로 만들어준다.
```java
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
```

- 금고를 털려는 Thread를 잡을 경찰 thread도 만든다.
```java
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
```
- 마지막으로 main class는 아래와 같다.
- Random class로 금고의 암호를 만든다.
- 금고를 풀려는 도둑과 경찰 thread를 start시킨다.
```java
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
```


## <span style="color:#802548">_process 정상 종료를 위한 thread interrupt와 daemon Thread_</span>

- thread는 아무것도 안해도 커널과 메모리의 자원을 잡아먹는다. 
- 안 쓰는 스레드는 제거해야 한다.
- thread가 하나만 살아있어도 process가 죽지 않는다. 
  - 따라서 application을 죽이기 전에 모든 thread를 종료시켜야 한다.
  - 거기에 쓰이는 method가 interrupt()다.
    - interrupt는 무한하게 도는 thread에서 interrupt를 감지시켜 thread를 loop에서 빠져나오게 해 thread를 종료시킨다.
    - interrupt 만으로 이미 돌아가던 logic이 멈추진 않는다.
    - Thread.currentThread().isInterrupted() 아니면 setDaemon(true)로 만들어야 한다.
- BlockingTask의 경우, 잠들었기 때문에 interrupt()가 호출되면 InterruptedException을 catch하여 run이 종료되고 스레드가 종료된다.
```java
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
```

- LongComputationTask의 경우, 실행중인 로직이다. 
- Daemon Thread로 만들지 않았기에, interrupt()의 호출을 감지하려면 Thread.isInterrupted()가 필요하다.
- Thread.isInterrupted()로 감지했다고 끝이 아니다. 거기서 loop를 끝낼 로직을 직접 만들어줘야 한다.
  - 여기서는 return BigInteger.ZERO가 그 역할을 했다.
```java
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
            if (Thread.isInterrupted()) {
                System.out.println("Prematurely interrupted computation");
                return BigInteger.ZERO;
            }
            result = result.multiply(base);
        }

        return result;
    }
}
```

- LongComputationTask의 경우, 실행중인 로직이다. 
- interrupt()의 호출을 감지하려면 Thread.isInterrupted()가 필요하다.
- 하지만 해당 로직이 없다. 이 경우 thread 생성시 daemon thread로 만든다.
```java
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

- main method에서 아래와 같이 start()와 interrupt를 해준다.
- interrupt()되면 실행중인 로직이 취소되므로 thread1의 return은 0이 된다.
- daemon thread는 non-daemon thread가 종료되면 같이 종료된다.
- 따라서 Thread가 끝나고, Thread1이 끝나고 main Thread가 끝나면, Thread2도 더이상 발동되지 않는다.

```java
public static void main(String[] args) throws Exception {
    Thread thread = new Thread(new BlockingTask());
    thread.start();
    thread.interrupt();

    Thread thread1 = new Thread(new LongComputationTask(new BigInteger("20000"),new BigInteger("100000")));
    thread1.start();
    thread1.interrupt();
    
    Thread thread2 = new Thread(new LongComputationTask1(new BigInteger("2000"),new BigInteger("10000")));
    thread2.setDaemon(true); 
    thread2.start();
    thread2.interrupt();
}
```

```
Exiting blocking thread             - BlockingThread
0                                   - LongComputationTask1
Prematurely interrupted computation - LongComputationTask
20000^100000 = 0                    - LongComputationTask
1                                   - LongComputationTask1...
2
3
4
5
6
7
8
9
10
11
```



## <span style="color:#802548">_반응성을 개선하는 thread join_</span>

- 오래걸리는 computation logic을 만든다.
- 여기선 factorial이란 재귀함수가 그런 logic이다.
```java
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
```

- long taking logic의 값을 가져오려고 아래와 같이 main method를 만들었다.
- thread가 질질 늘어지게 되어버린다. 거기다 long taking logic이라 연산이 완료되지 않아 해당 시점에는 0으로 떠버린다.
  - 그나마 연산이 가장 빠르게 완료되는 23의 경우에만 값이 출력되었다.
```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        List<Long> inputNumbers = Arrays.asList(100000000L, 3435L, 35435L, 2324L, 4656L, 23L, 5556L);

        List<FactorialThread> threads = new ArrayList<>();

        for (long inputNumber : inputNumbers) {
            threads.add(new FactorialThread(inputNumber));
        }

        for (Thread thread : threads) {
            thread.start();
        }


        for (int i = 0; i < inputNumbers.size(); i++) {
            FactorialThread factorialThread = threads.get(i);
            System.out.println(factorialThread.getResult()); 
        }
    }
}
```

```
0
0
0
0
0
25852016738884976640000
0
```


- 너무 오래걸리는 thread는 그냥 종료시키려면 interrupt를 활용할 수 있다. 
- 하지만 interupt로는 정확한 시간만큼 기다리는 것은 불가능하다. daemon thread도 마찬가지다.
- 그렇게 된다면 2초 정도를 기다리면 계산이 완료되는데도 main thread가 가져오는 값은 없을 수 있다.
```java
public class Main {
    public static void main(String[] args) throws InterruptedException {
        List<Long> inputNumbers = Arrays.asList(100000000L, 3435L, 35435L, 2324L, 4656L, 23L, 5556L);

        List<FactorialThread> threads = new ArrayList<>();

        for (long inputNumber : inputNumbers) {
            threads.add(new FactorialThread(inputNumber));
        }

        for (Thread thread : threads) {
            thread.setDaemon(true);
            thread.start();
        }


        for (int i = 0; i < inputNumbers.size(); i++) {
            FactorialThread factorialThread = threads.get(i);
            if (factorialThread.isFinished()) {
                System.out.println("Factorial of " + inputNumbers.get(i) + " is " + factorialThread.getResult());
            } else {
                System.out.println("The calculation for " + inputNumbers.get(i) + " is still in progress");
            }
        }
    }
}
```

```
The calculation for 100000000 is still in progress
The calculation for 3435 is still in progress
The calculation for 35435 is still in progress
The calculation for 2324 is still in progress
The calculation for 4656 is still in progress
Factorial of 23 is 25852016738884976640000
The calculation for 5556 is still in progress
```

- 이를 join을 활용하면 정확히 2초 정도의 시간을 기다리게끔 만들어줄 수 있다.
- join은 현재 실행중인 쓰레드(main thread)가 다른 쓰레드의 작업이 끝날때까지 기다리게 한다.
  - 현재 thread는 대기상태에 돌입하고, 그동안 다른 thread가 연산을 진행하여 2초가 지난 뒤에 연산된 값을 가져온다.
- 그럼 계산이 완료된 logic들의 경우 값을 가져올 수 있게 된다.
```java
public static void main(String[] args) throws InterruptedException {
    List<Long> inputNumbers = Arrays.asList(100000000L, 3435L, 35435L, 2324L, 4656L, 23L, 5556L);

    List<FactorialThread> threads = new ArrayList<>();

    for (long inputNumber : inputNumbers) {
        threads.add(new FactorialThread(inputNumber));
    }

    for (Thread thread : threads) {
        thread.setDaemon(true);
        thread.start();
    }

    for (Thread thread : threads) {
        thread.join(2000);
    }

    for (int i = 0; i < inputNumbers.size(); i++) {
        FactorialThread factorialThread = threads.get(i);
        if (factorialThread.isFinished()) {
            System.out.println("Factorial of " + inputNumbers.get(i) + " is " + factorialThread.getResult());
        } else {
            System.out.println("The calculation for " + inputNumbers.get(i) + " is still in progress");
        }
    }
}
```

- 여전히 100000000L은 계산중이라 값을 가져올 수 없다.
- eclipse에서는 이유는 모르겠는데 100000000L, 3435L은 콘솔에서 지워지고 그 이후부터 출력된다.
```
The calculation for 100000000 is still in progress
Factorial of 35435 is 386588992032151251667007583813348702360432931100560868954226257629442643373231427226867559462578302509283089665244215909384571828146597432482783406603858090226007951016972443408362423971173167574438923082698334573145838648096788185680106428957986499780489200968997490302464862469292999028824193670828568407530556434212631480406995268603279354662349037768693011626007099108989845481867838003729114572126627560365220962674456731133207689540784702065638623177113845158608923369304050318505189001248164915371687556472646128582314257947040270714178729837000098190856785622373322497855919537192744678414013755741053302987702456398500223358867939367998167588716676449062888956046405186995355116604966407309359623161874035026325944850251507183143525155082350419315247885305805138501059288920498
.
.
.
Factorial of 2324 is 82148407165048181056827200552545437572223953583509883674946458054844446908120819016663854631403295507160347782511867449764724613729073475534332618412046688619217115831734898578436991590722684312995775994377705545589415840329990011405456594973089194689831937182701483346122322683280925821345500267154027688945651366235905205525562181337221675915297049723776443122588580469788006052161083899603696111780175296314243496391664161292935638128912133642029824981810811933315833911655981380687746385462882291826575719431499196817362734588961155120404631698165759061357383833947808636323655117870477358408450631859239182150923341090557797970549588013677387299982600745313554083074783731093332248699091062001762719984861007137243510396808991474603117711655147857401869867645724372181888901254023736773223049710680942623520311926483206750263866740286556041721016669871242717985370409365147489199745723321948000416755175606191706079917267031314598610070718563296955173643912934181529012634909247136087104977087287621453711858978753099482263624554531266031384321920330747243261844347089235974522311808597576828190859240725889778033409791300296058521739916945304687582469160856895497855727875432646287648093850151794746418421370687969330855297792345021237836599867482082380740470536695257256758713343392789641375030392198797.
.
.
Factorial of 4656 is 2669269449393821055486755804950239900551819660012792816830778583528312560038271395810866072945947790638821320968540760808049457201882378844211206639875242602358683764249379990637148268819825230693673039553110526203766057557271417282375370453558386818973649457923447741149546925837275389755810201614258068644886935909491601791895623658769460156172871828920333099360353728320706803707965842774198391757162658019677855026008957318857105864181507691362619780232598342066871270108430179141932524874914080485013846117583414994218067689224338817750895459992513600642541951543868519311853455372886118880750747266159462773982702556074910619439953729243930394835136816144529419141089146686889202138633865939913363560776556351684076095311916058913863480184354286044889246441936429866658293966215618342057685603375110949309947308946552246814690363298977308520015132997407374741664932093548563892082273727876142696597241974734318093390289826178092772817252953313691689160062689352859236663685515642402664713589470417962113647789917266044349821102714417541904623936779887477214936074924122737232250187595538853754672244084910086609828869509534998280670795782205294496106355215086654694497807018906719457200635474993835747315698763943428800220662699903007373095390943280679177492440448257407735719555231108556721333436865808093775973909926504325577929379755091462286136397264738211422777792000909011242389174815515831210449777464940318327251130202831279299760088004290901562272916880274136643145045959505131182191825193938643795478743461967094065217872093844179845808996232098667135377918301167445107310705600465331901108505776150629950402395597377285083056070324394026792156529543066538496971577445197745512198514234657207290836264151881847405782570756481748541544937862060850206185991055345874055367660721563479731200521805921827496439856058909773476292187703103584956190689204549431590029073392970457711694268761428440941650093816571543770659836218372095697481187507218932939414333835881661630099344184583444387607173921872981080097008146375782723621759245109224463994452755632899983394312064492186600993405670473645505539372678847803585465834812557376172830991191971990421995135961191267848534712446508759216356664591818981836258735850634821812806226227995799660048022415595588443932558566508848438736991239281975036673581174232657637986845298458203334291014156779519575651010796470453765080702505615848047356799261584815187489250418089027665884360521299451923299272365652256933820740802409576432933735418254976155674063692008655594423497819908684101778376177966202451465226233152524977001536798447695278483156685703125219595582343.
.
.
Factorial of 23 is 25852016738884976640000
Factorial of 5556 is 1699515964436519756709361069052680097268578402372263360131377466557861020609544297030935865023310828261601267993719610472479951447158200702473126886048427947689288472913128938960327881146341123573408010461704780915685451588386179037342030358372260357745461444920553880455816389768968372028009103467018055283457803464487372799427997682857401234411138774597439082989283785680560378326728861264159795338911584485238686999776031089526118992061573124316665075300133631329939088233454690361408494339300839086278379960818590896338689918450923904138143665029103033642584736460569636444179049255100797339456952738476073537010019579793038392331090112591504347550858622273710477332795317702210233004312236946787736335097831209927952511349354333036000964938266142843186521622203074915567409827187765666096696344793268849757808366458992514687190340789799467037967008336592767193482866553510020345839247246604413674020800830046597667774942830527776011920729472112048288762492808398430902660895772549887736689011707464663681580275240562668549426297769011043752144.
.
```



- join을 활용하게 되면 2초 정도 기다리는 시간안에 logic이 완료될 때까지 시간을 벌 수 있다.
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


- thread를 복수개를 만들어 join을 활용하게 되면 어떻게 될까?
- 
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



