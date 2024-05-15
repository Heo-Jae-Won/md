### 객체 나누기 코드 사례
```java
public class Application implements OnClickListener {

    private Menu menu1 = new Menu("menu");
    private Menu menu2 = new Menu("menu");
    private Button button1 = new Menu("button1");
    
    private String currentMenu = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
        //button2.setonClickListener(this);
    }

    public void clicked(Component eventSource) {
        if ("menu1".equals(eventSource.getId())) {
            changeUIToMenu1();
        } else if ("menu2".equals(eventSource.getId())) {
            changeUIToMenu2();
        } else if ("button1".equals(eventSource.getId())) {
            if (currentMenu == null) {
                return;
            }
            if ("menu1".equals(currentMenu)) {
                processButton1WhenMenu1();
            } else if ("menu2".equals(currentMenu)) {
                processButton1WhenMenu2();
            }
        }/* else if (eventSource.getId().equals("button2")) {
            if (currentMenu == null) {
                return;
            }
            if (currentMenu.equals("menu1")) {
                processButton1WhenMenu1();
            } else if (currentMenu.equals("menu2")) {
                processButton1WhenMenu2();
            } */
    }


    private void changeUIToMenu1() {
        currentMenu = "menu1";
        sysout("메뉴1 화면으로 전환");
    }

    private void changeUIToMenu2() {
        currentMenu = "menu2";
        sysout("메뉴2 화면으로 전환");
    }

    private void processButton1WhenMenu1() {
        sysout("메뉴1 화면의 버튼1 처리")
    }

    private void processButton1WhenMenu2() {
        sysout("메뉴2 화면의 버튼1 처리")
    }

    /*private void processButton2WhenMenu1() {
        sysout("메뉴1 화면의 버튼2 처리")
    }

    private void processButton2WhenMenu2() {
        sysout("메뉴2 화면의 버튼2 처리")
    }*/

}
```
- interface를 활용하여 코드를 나눈다.
```java
public interface ScreenUI {
    public void show();
    public void handleButton1Click();
}

public class Menu1ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴1 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴1 화면의 버튼1 처리");
    }
}

public class Menu2ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴2 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴2 화면의 버튼1 처리");
    }
}
public class Application implements OnClickListener {

    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
    }

    public void clicked(Component eventSource) {
        String sourceId = eventSource.getId();
        if ("menu1".equals(sourceId)) {
            currentScreen = new Menu1ScreenUI();
            currentScreen.show();
        } else if ("menu2".equals(sourceId)) {
            currentSCreen = new Menu2SCreenUI();
            currentScreen.show();
        } else if ("button1".equals(sourceId)) {
            if(currentScreen == null) {
                return;
            }
            currentScreen.handleButton1Click();
        }
    }
}
```
- 버튼 클릭 처리 코드와 메뉴 클릭 처리 코드의 목적이 다르므로 분리한다.
```java
public interface ScreenUI {
    public void show();
    public void handleButton1Click();
}

public class Menu1ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴1 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴1 화면의 버튼1 처리");
    }
}

public class Menu2ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴2 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴2 화면의 버튼1 처리");
    }
}
public class Application {

    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
    }

    private OnClickListener menuListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            String SourceId = eventSource.getId();
            if("menu1".equals(sourceId)) {
                currentScreen = new Menu1ScreenUI();
            } else if ("menu2".equals(sourceId)) {
                currentScreen = new Menu2ScreenUI();
            }

            currentScreen.show();
        }
    }

    private OnClickListener buttonListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if(currentScreen == null) {
                return;
            }
            String sourceId = eventSource.getId();
            if("button1".equals(sourceId)) {
                currentScreen.handleButton1Click();
            }
        }
    }
}
```
- 버튼2가 추가된다.

- 맨위와 비교하면 정말 간단하게 추가되었음을 알 수 있다.
```java
public interface ScreenUI {
    public void show();
    public void handleButton1Click();
    public void handleButton2Click();
}
public class Menu1ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴1 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴1 화면의 버튼1 처리");
    }
    /*public void handleButton2Click() {
        sysout("메뉴1 화면의 버튼2 처리");
    }*/
}

public class Menu2ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴2 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴2 화면의 버튼1 처리");
    }
    /*public void handleButton2Click() {
        sysout("메뉴2 화면의 버튼2 처리");
    }*/
}
public class Application {

    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        button1.setOnClickListener(this);
    }

    private OnClickListener menuListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            String SourceId = eventSource.getId();
            if("menu1".equals(sourceId)) {
                currentScreen = new Menu1ScreenUI();
            } else if ("menu2".equals(sourceId)) {
                currentScreen = new Menu2ScreenUI();
            }

            currentScreen.show();
        }
    }

    private OnClickListener buttonListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if(currentScreen == null) {
                return;
            }
            String sourceId = eventSource.getId();
            if("button1".equals(sourceId)) {
                currentScreen.handleButton1Click();
            } /*else if(sourceId.equals("button2")) {
                currentScreen.handleButton2Click();
            }*/
        }
    }
}
```
- 메뉴3가 추가되었다고 해보자.
- 메뉴3가 추가되도 이제 버튼에 관한 코드는 건들지 않아도 된다.

- Menu3ScreenUI class만 만들고, Application 생성자에 menu3 관련 clickListener를 달아준다.

```java
public interface ScreenUI {
    public void show();
    public void handleButton1Click();
    public void handleButton2Click();
}
public class Menu1ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴1 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴1 화면의 버튼1 처리");
    }
    public void handleButton2Click() {
        sysout("메뉴1 화면의 버튼2 처리");
    }
}

public class Menu2ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴2 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴2 화면의 버튼1 처리");
    }
    public void handleButton2Click() {
        sysout("메뉴2 화면의 버튼2 처리");
    }
}

public class Menu3ScreenUI implements ScreenUI {
    public void show() {
        sysout("메뉴3 화면으로 전환");
    }
    public void handleButton1Click() {
        sysout("메뉴3 화면의 버튼1 처리");
    }
    public void handleButton2Click() {
        sysout("메뉴3 화면의 버튼2 처리");
    }
}
public class Application {

    private Menu menu1 = new Menu("menu1");
    private Menu menu2 = new Menu("menu2");
    private Button button1 = new Button("button1");

    private ScreenUI currentScreen = null;

    public Application() {
        menu1.setOnClickListener(this);
        menu2.setOnClickListener(this);
        //menu3.setOnClickListener(this);
        button1.setOnClickListener(this);
    }

    private OnClickListener menuListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            String SourceId = eventSource.getId();
            if("menu1".equals(sourceId)) {
                currentScreen = new Menu1ScreenUI();
            } else if ("menu2".equals(sourceId)) {
                currentScreen = new Menu2ScreenUI();
            } /*else if (eventSource.getId().equals("menu3")) {
                currentScreen = new Menu3ScreenUI();
            }*/

            currentScreen.show();
        }
    }

    private OnClickListener buttonListener = new OnClickListener() {
        public void clicked(Component eventSource) {
            if(currentScreen == null) {
                return;
            }
            String sourceId = eventSource.getId();
            if("button1".equals(sourceId)) {
                currentScreen.handleButton1Click();
            } /*else if(sourceId.equals("button2")) {
                currentScreen.handleButton2Click();
            }*/
        }
    }
}
```

### 프로시저의 단점
- 절차지향이란 말은 순서에 따른 프로그래밍이라는 말이 아니다. 프로시저를 이용한 프로그래밍 기법을 의미한다.
- 프로시저는 데이터를 공유하는 방식으로 만들어지기 때문에 데이터를 중심으로 구현된다.
- 프로시저의 단점은 요구사항이 변경되면서 예상치 못한 상태가 추가될 때 드러난다.

- boolean 타입의 이름은 isOn이라는 data가 있다. 아래와 같은 상태를 지닌다.
```
on
off
대기중 상태 추가해달라 ---> 이 데이터 사용하는 모든 프로시저 수정
```

- 프로시저의 최악의 단점은 데이터가 서로 다른 의미로 사용되는 경우에 드러난다.
- 서비스 만료일 데이터가 null일 때 오류로 처리하게 만료 확인 프로시저를 만들었다고 하자.
- 그런데 회원 정보 수정 프로시저에서 서비스를 무한정 사용한다는 의미로 서비스 만료일 데이터 값을 null로 설정하게 되면?
- 이전 프로시저들이 모두 오류가 발생하게 된다. null은 오류로 처리하기 때문이다.
- 소리 크기 제어 객체가 있다고 해보자. 해당 객체는 아래 기능을 제공한다.
```
소리 크기 증가
소리 크기 감소
음 소거
```


### 객체의 operation

- 객체가 내부적으로 소리 크기를 어떤 데이터 타입 값으로 보관하는 지는 중요하지 않다. 세 개의 기능을 제공한다는 게 중요하다.
- 객체의 기능을 사용한다는 것은 객체의 operation을 사용한다는 의미고, 그러려면 사용법을 알아야 한다.
- operation의 사용법은 기능 식별 이름, 파라미터와 파라미터 타입, 결과 값을 합쳐 signature라고 한다.
```
method명 - 기능식별이름
parameter명 - parameter
결과값 - return
```
- 이런 객체가 제공하는 모든 operation을 객체의 인터페이스라고 한다.
- 이 interface는 Java 내의 interface가 아니다. 객체를 사용하기 위한 규칙이나 명세라고 생각해야 한다.
- operation의 실행을 요청하는 것을 메시지를 보낸다고 표현한다. 메서드를 호출하는 것이 메시지를 보내는 것이다.

```java
FileInpuStream is = new FileInputStream(fileName);
byte[] data = new byte[512];
int readBytes = is.read(data); // read가 메시지를 전송한 것이다. 즉 메소드 호출.
```

- 객체의 책임이 작아져야 절차지향 방식을 피할 수가 있다.
- 전부 서비스를 나누면 장점은 다음과 같다.
- 파일을 읽어오는 방법을 변경해야 한다면 파일 읽기 책임을 가진 객체의 코드만 수정된다.
- 암호화 알고리즘을 변경해야 하면, byte 암호화 객체의 코드만 수정된다.
- 즉 다른 class에 미치는 영향도가 현격하게 줄어들어 유지보수에 유리하다.


### 의존
- 한 객체가 다른 객체를 생성하거나, 다른 객체의 메서드를 호출할 때, parameter로 전달받을 때, 의존이라고 한다.

```java
public class FloWController {
    private String fileName;

    public FlowController(String fileName) {
        this.fileName = fileName;
    }
    public void process() {
        FileDataReader reader = new FileDataReader(fileName);
        byte[] plainBytes = reader.read();

        ByteEncryptor encryptor = new ByteEncryptor();         //객체 생성
        byte[] encryptedBytes = encryptor.encrypt(plainBytes); //메서드 호출
    }
}

public void process(ByteEncryptor encryptor) {
    //전달받은 파라미터를 사용할 가능성이 높다.
}
```

- 의존의 문제는 의존하는 대상의 코드가 바뀌면 자신의 코드도 바뀔 수 있다는 점이다.
- 아래와 같은 코드가 있다고 해보자. 아래와 같은 코드에서 로그를 남겨달라는 요구가 추가되었다고 해보자.
```java
public class Authenticator {
    public boolean authenticate(String id, String password) {
        Member m = findMemberById(id);
        if  (m == null) {
            return false;
        }

        return m.equalPassword(password);
    }
}

public class AuthenticationHandler {
    public void handleRequest(String inputId, String inputPassword) {
        Authenticator auth = new Authenticator(); //객체 생성. 의존
        if (auth.authenticate(inputId, inputPassword)) {

        } else {

        }
    }
}
```

- 따라서 handler를 Exception을 throw하게 바꿨다.
```java
public class AuthenticationHandler {
    public void handleRequest(String inputId, String inputPassword) {
        Authenticator auth = new Authenticator(); //객체 생성. 의존
        try {
            auth.authenticate(inputId, inputPassword);
        } catch(MemberNotFoundException e) {
            logService.writerLog(e);
            throw e;
        } catch(InvalidPasswordException e) {
            logService.writerLog(e);
            throw e;
        }
    }
}
```
- 그럼 아래와 같이 Authenticator도 코드를 바꿔줘야 한다.
```java
public class Authenticator throws MemberNotFountException, InvalidPasswordException {
    public void authenticate(String id, String password) {
        Member m = findMemberById(id);
        if (m == null) {
            throw new MemberNotFountException();
        }

        if(!m.equalPassword(password)) {
            throw new InvalidPasswordException();
        }
    }
}
```
- 객체지향의 의존이라는 것이 늘 장점만 있는 것은 아니다.

### encapsulation
- 회원 만료 처리를 하는 코드가 아래와 같다고 해보자.
```java
@Getter
public class Member {
    private Date expiryDate;
    private boolean male;
}
.
.
.

if (member.getExpiryDate() != null &&
        member.getExpiryDate().getDate() < System.currentTimeMills()) {
            //만료됐을 때 처리
}
```

- 그런데 서비스를 운영하다가 여성회원은 만료 기간이 지나도 30일 간은 서비스를 사용하게 정책이 변경되었다.
- 그럼 만료가 사용되는 코드를 전부 아래와 같이수정해줘야 한다.
```java
long day30 = 1000 * 60 * 60 * 24 * 30;
if( (member.isMale && member.getExpiryDate ! = null && member.getExpiryDate().getDate() < System.currentTimeMills()) ||
    (!member.isMale() && member.getExpiryDate != null && member.getExpiryDate().getDate() < System.currentTimeMills() - day30)) {

    }
```

- 만료 처리 로직을 캡슐화해준다면 전부 수정하지 않아도 된다.
- 해당 class 내의 로직만 수정해주면 나머지는 알아서 적용된다.
- 아래와 같이 기존 만료 로직이 있다고 해보자.
```java
public class Member {
    private Date expiryDate;
    private boolean male;

    public boolean isExpired() {
        return expiryDate != null &&
                expiryDate.getDate() < System.currentTimeMills();
    }
}

if(member.isExpired) {

}
```

- 기존 로직을 아래와 같이 바꿨다.
- 클래스에 캡슐화해놨기 때문에 서비스의 모든 로직을 수정할 필요가 없다.
```java
public class Member {
    private static final long DAY30 =  1000 * 60 * 60 * 24 * 30;
    private Date expiryDate;
    private boolean male;

    public boolean isExpired() {
        if (male) {
            return expiryDate != null &&
                expiryDate.getDate() < System.currentTimeMills();
        }

        return expiryDate != null &&
                expiryDate.getDate() < System.currentTimeMills() - DAY30;
    }
}

if(member.isExpired()) {

}
```

- 이와 같이 캡슐화의 유명한 원칙으로 두 가지가 있다.

```
Tell, Don't ask
Law of Demeter
```
- 첫째 원칙은 서비스 로직은 data를 요구하지 않는다는 것이다. 객체를 생성하고 매서드를 호출했으면 기능이 알아서 실행되게 하자는 것이다.
- 즉 내부 구현을 감추자는 이야기다.
- 두번째 원칙은 의존을 가진 객체의 method만 호출하자는 것이다.
- 다시말해 의존을 가진 객체의 객체의 method를 호출하지 말자는 의미다.

```java
public void processSome(Member member) {
    if (member.getDate().getTime() < ..) {// getDate에서 또 getTime을 부르면 데메테르 위반

    }
}
```

- 데메테르 법칙에 관한 유명한 코드는 아래와 같다.
- 아래 코드로 돈을 받을 수 있지만, 이를 실제 현실로 옮기면 다음과 같은 절차다.
- 지갑을 받는 게 아니라 고객이 돈을 지불하게 바꿔야 한다.
```
고객님 지갑주세요
지갑에 돈있는지 볼게요
돈있으니까 돈 빼갈게요
```
```java
public class Customer {
    private Wallet wallet;

    public Wallet getWallet() {
        return wallet;
    }
}

public class Wallet {
    private int money;
    public int getTotalMoney() {
        return money;
    }
    public void substractMoeny(int debit) {
        moeny -=debit;
    }
}

//돈받는 코드
int payment = 10000;
Wallet wallet = customer.getWallet();
if (wallet.getTotalMoney() >= payment) {
    wallet.substractMoney(payment);
} else {
    //다음에 요금 받게 처리
}
```

- 아래와 같이 class 내부로 캡슐화하면, 객체의 객체의 method를 부르는 일이 없어진다.
- 또한 지갑에서 돈을 받는 게 안리ㅏ 주머니에서 돈을 받는 방식으로 구현이 바뀌어도 getPayment()만 바꿔주면 된다.
```java
public class Customer {
    private Waller wallet;

    public int getPayment(int payment) {
        if (wallet == null) {
            throw new NotEnoughMoneyException();
        }
        if (wallet.getTotalMoney >= payment) {
            wallet.substractMoney(payment);
            
            return payment;
        }

        throw new NotEnoughMoneyException();
    }
}

//돈 받는 코드
int payment = 10000;
try {
    int paidAmount = customer.getPayment(payment);
} catch(NotEnoughMoneyException e) {

}
```


### 상속
```java
public class FlowController {
    private boolean useFile;

    public FlowController(boolean useFile) {
        this.useFile = useFile;
    }

    public void process() {
        byte[] data = null;
        if (useFile) {
            FileDataReader fileReader = new FileDataReader();
            data = fileReader.read();
        } else {
            SocketDataReader socketReader = new SocketDataReader();
            data = socketReader.read();
        }

        Encryptor encryptor = new Encryptor();
        byte[] encryptedData = encryptor.encrypt(data);

        FileDataWriter writer = new FileDataWriter();
        writer.write(encryptedData);
    }
}
```

- 아래와 같이 interface를 사용해 책임을 나누자.
```java
public interface ByteSource {
    public byte[] read();
}

public class FileDataReader implements ByteSource {
    public byte[] read() {

    }

    public class SocketDataReader implements ByteSource {

    }
}

public class FlowController {
    private boolean useFile;

    public FlowController(boolean useFile) {
        this.useFile = useFile;
    }

    public void process() {
       ByteSource source = null;
        if (useFile) {
          source = new FileDateReader();
        } else {
           source = new SocketDataReader();
        }

        byte[] data = source.read();

        Encryptor encryptor = new Encryptor();
        byte[] encryptedData = encryptor.encrypt(data);

        FileDataWriter writer = new FileDataWriter();
        writer.write(encryptedData);
    }
}
```

- 그리고 ByteSource 또한 종류가 변경되도 FlowController가 바뀌지 않게 만든다.
- 그 방법은 class 안에 분류 로직을 만드는 것이다.
- 분류기는 매번 instance를 만들 필요가 없으니 singleton으로 만든다.
```java
public class ByteSourceFactory {
    public ByteSource create() {
        if (useFile) {
            return new FileDataReader();
        } else {
            return new SocketDataReader();
        }
    }

    private boolean useFile() {
        String useFileVal = System.getProperty("useFile");
        
        return useFileVal != null && Boolean.valueOf(useFileVal);
    }

    private static ByteSourceFactory instance = new ByteSourceFactory();
    public static ByteSourceFactory getInstance() {
        return instance;
    }
    private ByteSourceFactory() {}
}

public class FlowController {
    private boolean useFile;

    public FlowController(boolean useFile) {
        this.useFile = useFile;
    }

    public void process() {
        ByteSource source = ByteSourceFactory.getInstance().create();
        byte[] data = source.read();

        Encryptor encryptor = new Encryptor();
        byte[] encryptedData = encryptor.encrypt(data);

        FileDataWriter writer = new FileDataWriter();
        writer.write(encryptedData);
    }
}
```
- FlowController를 위와 같이 바꾸면 새로운 요청 사항이 들어와도, FlowController를 건들필요가 없다.
- HTTPs를 이용해서 데이터를 읽어오려면 ByteSourceFactory의 create에만 관련 내용을 넣어주면 된다.
- 이전 코드를 보면 데이터를 읽어오는 객체를 생성하는 책임, 흐름을 제어하는 책임이 한 객체에 몰아져 있었음을 알 수 있다.

```java
public void process() {
    //데이터 읽기 객체 직접 생성
    FileDataReader reader = new FileDataReader();
    //흐름 제어: 1. 읽기
    byte[] data = reader.read();

    //흐름 제어: 2. 암호화
    Encryptor encryptor = new Encryptor();
    byte[] encryptedData = encryptor.encrypt(data);
    
    //흐름 제어: 3. 쓰기
    FileData Writer writer = new FileDataWriter();
    writer.write(encryptedData);
}
```
- 진행한 두 번의 추상화는 다음과 같다.
```
바이트 데이터 읽기: ByteSource interface 도출
ByteSource 객체 생성하기: ByteSourceFactory 도출
```
- 이러한 추상화를 통해 상위 수준의 controller 로직을 건들지 않고 그대로 놔둘 수 있었다.
- 이처럼 상위 수준(controller) logic은 바뀌지 않게 확장가능한 설계를 짜는 것이 중요하다.
- 그러한 추상화를 위해서 바로 중요한 게 interface를 활용하는 것이다.
- interface는 자바 내부의 interface가 아니라 객체의 스펙, 규격서라는 개념에서의 interface다.
```
program to interface(인터페이스에 대고 프로그래밍하기)  
다만 interface를 쓰면 복잡해지므로 변화가능성이 높은 곳에만 활용해야 한다.
```

- 인터페이스는 인터페이스 사용자 입장에서 만들어야 한다.
- FileDateReader와 SocketFileDataReader class를 모두 아우르는 naming이 필요하다.
- 더 나아가 본질을 포착해 naming을 해야한다. 어쨌든 데이터를 읽어온다는 의미에서 ByteSource라고 지어주는 게 더 좋다.
- FileDateReaderInterface는 바람직하지 않은 naming이다.
- 객체를 나누면 좋은 다른 점은 test가 가능해진다는 점이다.
```java
public class FlowController {
    public void process() {
        FileDataReader reader = new FileDataReader(); //구현못했으면 read() test 불가.. interface를 implements한 class 만들어 대리 test도 불가. IF가 아예 없음. concrete classㅁ나 존재
        byte[] data = reader.read();
    }
}

public void testProcess() {
    FlowController fc = new FlowController();
    fc.process();
}
```

- 아래와 같이 mock 객체를 활용할 수 있다.
- mock 객체를 활용하면 FileDataReader의 read가 구현이 끝나지 않았어도 test가 가능하다.
```java
public class FlowController {
    private ByteSource byteSource;

    public FlowController(ByteSource byteSource) {
        this.byteSource = byteSource;
    }

    public void process() {
        byte[] data = byteSource.read();
    }
}

public void testProcess() {
    ByteSource mockSource = new MockByteSource();
    FlowController fc = new FlowController(mockSource);
    fc.process();
}

class MockByteSource implements ByteSource {
    public byte[] read() {
        byte[] data = new byte[128];

        return data;
    }
}
```

```java
public abstract class Figure {
    private Bounds bounds = new Bounds(); //위임. 원하는 기능이 이미 다른 class에 구현되어 있으면 조립으로 가져온다.
    private void changeSize() {
        bounds.set(x, y, width, height);
    }

    public boolean contains(Point point) {
        return bounds.contains(point.getX(), point.getY());
    }
}

public abstract class Figure {
    public boolean contains(Point point) {
        Bounds bounds = new Bounds(x, y, width, height);

        return bounds.contains(point.getX(), point.getY());
    }
}

public class AService {
    @Autowired
    BService bService; //Spring의 위임. Singletone으로 bean을 만들어 사용.

    public getSth() {
        bService.get();
    }
}
```

### SOLID
```
SRP- 클래스는 단 하나의 책임을 가진다.
```

- HttpClient을 이용해서 HTML 응답 문자열을 받아 화면을 load하는 형식이다.
```java
public class DataViewer {

    public void display() {
        String data = loadHtml();
        updateGui(data);
    }

    public String loadHtml() {
        HttpClient client = new HttpClient();
        client.connect(url);
        
        return client.getResponse();
    }

    private void updateGui(String data) {
        GuiData guiModel = parseDataToGuiData(data);
        tableUI.changeData(guiModel);
    }

    private GuiData parseDataToGuiData(String data) {

    }
}
```
- SocketClient를 이용해서 byte[]를 받아 화면을 load하는 형식으로 바뀌었다.
- 즉 데이터를 읽어오는 책임의 기능이 변경된 것이다. 
- 데이터를 보여주는 책임의 기능은 그대로지만, 데이터를 읽어오는 책임의 기능이 같이 있어 코드가 수정된다.
- 단일 책임을 지키지 않아 코드 수정이 많아진다.
```java
public class DataViewer {

    public void display() {
        byte[] data = loadHtml();
        updateGui(data);
    }

    public byte[] loadHtml() {
        SocketClient client = new SocketClient(); //변화
        client.connect(server, port);             //변화
        
        return client.read();                     //변화
    }

    private void updateGui(byte[] data) {          //변화
        GuiData guiModel = parseDataToGuiData(data);
        tableUI.changeData(guiModel);
    }

    private GuiData parseDataToGuiData(byte[] data) {
        //파싱 코드도 변경
    }
}
```

- OCP
```
기능을 변경하거나 확장할 수 있으면서 기능을 사용하는 코드는 수정하지 않는다.
```

- 아래처럼 만들고 상속을 하면 OCP를 지킬 수 있다.
```java
public class ResponseSender {
    private Data data;
    public ResponseSender(Data data) {
        this.data = data;
    }

    public Data getData() {
        return data;
    }

    public void send() {
        sendHeader();
        sendBody();
    }

    protected void sendHeader() {
        //헤더 데이터 전송        
    }

    protected void sendBody() {
        //text로 데이터 전송
    }
}
```

```java
public class ZippedREsponseEnder extends ResponseSender {
    public ZippedResponseSender(Data data) {
        super(data);
    }

    @Override
    protected void sendBody() {
        //데이터 압축 처리
    }
}
```

- 또는 이전의 ByteSource처럼 interface 등으로 추상화한다.

- OCP가 깨지는 주요 증상은 downcasting이다.
- 특정 class인지 확인하는 처리가 있다면, Character class가 확장될 때 함께 수정될 가능성이 높다.
- 이럴 떄는 drawSpecific()을 Missile이 아닌 Character에 추가하여 추상화한 뒤 사용하는 게 좋다. 
```java
public void drawCharacter(Character character) {
    if (character instanceof Missile) {
        Missile missile = (Missile) character;
        missile.drawSpecific();
    } else {
        character.draw();
    }
}
```

- 비슷한 if - else 블록이 존재해도 의심해야 한다

```java
public class Enemy extends Character {
    private int pathPattern;

    public Enemy(int pathPattern) {
        this.pathPattern = pathPattern;
    }

    public void draw() {
        if (pathPattern == 1) {
            x += 4;
        } else if (pathPattern == 2) {
            y += 10;
        } else if (pathPattern == 4) {
            x += 4;
            y += 10;
        }
        //그려주는 코드
    }
}
```

- int 값을 받는 게 아니라, int를 PathPattern이라는 class로 만들어 객체를 받게 변경한다.
```java
public class Enemy extends Character {
    private PathPattern pathPattern;

    public Enemy(PathPattern pathPattern) {
        this.pathPattern = pathPattern;
    }

    public void draw() {
        int x = pathPattern.nextX();
        int y = pathPattern.nextY();

        //그려주는 코드
    }
}
```

- 새로운 패턴이 생겨도, nextX()와 같은 구현 method만 바꿔주면 된다.
```java
public class Enemy extends Character {
    private PathPattern pathPattern;

    public Enemy(PathPattern pathPattern) {
        this.pathPattern = pathPattern;
    }

    public void draw() {
        int x = pathPattern.nextX();
        int y = pathPattern.nextY();

        //그려주는 코드
    }
}
```

- LSP
```
LSP - 상위 타입 객체를 하위 타입 객체로 치환해도 정상으로 작동해야 한다.
```

```java
public void someMethod(SuperClass sc) {
    sc.someMethod();
}

someMethod(new SubCLass()); //subclass로 넣어도 잘 돌아가야 함.
```

```java
public class Rectangle {
    private int widht;
    private int height;

    public void setWidth(int width) {
        this.width = width;
    }

    public void setHeight(int height) {
        this.height = heigth;
    }

    public int getWidth() {
        return width;
    }

    public int getHeight() {
        return heigth;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWIdth(int width) {
        super.setWidth(width);
        super.setHeight(width);
    }

    @Override
    public void setHeight(int height) {
        super.setWidth(height);
        super.setHeight(height);
    }
}
```

- Rectangle을 넣으면 무리없이 작동하지만, Rectangle을 상속한 Square를 넣으면 의도와 다르게 된다.
- 따라서 LSP가 지켜지지 않은 것이다.
```java
public void increaseHeight(Rectangle rec) {
    if (rec.getHeight <= rec.getWidth()) {
        rec.setHeight(rec.getWidth() + 10);
    }
}
```

- 이를 커버하기 위해 instanceof로 확인을 시도할 수 있지만, 이 자체로 LSP가 지켜지지 않는다는 의미다.
- 이처럼 우리의 개념 상으로 상속 관계여도, 프로그램 상에서는 상속 관계가 아닐 수 있다.
- 그럴 때 Square class는 Rectangle을 extends하지 않고 별도로 구현해야 한다.

- LSP를 어기게 되는 다른 예는 상위 타입에서 지정한 return 값의 범위를 지키지 않았을 때다.

```java
public class CopyUtil {
    public static void copy(InputStream in, OutputStream out) {
        byte[] data = new byte[512];
        int len = -1;

        while( (len = in.read(data)) != -1) {
            out.write(data, 0, len);
        }
    }
}
```
- read한 결과가 0이게 하위 class를 바꾸면 무한 루프를 돌아버리게 된다.
```java
public class SatanInputStream implements InputStream {
    public int read(byte[] data) {
        .
        .
        .
        
        return 0;
    }
}
```

- LSP가 위반될 가능성이 높은 경우는 아래와 같다.
```
subclass가 명세에서 벗어난 값을 return할 떄
명세에서 벗어난 exception을 throw할 때
명세에서 벗어난 기능을 수행할 떄
```

```java
public class Coupon {
    public int calculateDiscountAmount(item, item) {
        return item.getPrice() * discountRate;
    }
}

public class Coupon {
    public int calculateDiscountAmount(Item item) {
        if (item instanceof SpecialItem) {
            return 0;
        }

        return item.getPrice() * discountRate;
    }
}
```

```java
public class Item {

    public boolean isDiscountAvailable {
        return true;
    }
}

public class SpecialItem extends Item {
    @Override
    public boolean isDiscountAvailable {
        return false;
    }
}

public class Coupon {
    public int calculateDiscountAmount(Item item) {
        if (!item.isDiscountAvailable()) { //어떤 instance인지 확인할 필요가 없다. 다형성으로 parameter의 class type을 check해서 알아서 해당 class에 맞는 method가 call된다.
            return 0;
        }

        return item.getPrice() * discountRate;
    }
}
```

- ISP
```
인터페이스를 사용하는 사용자를 기준으로 분리해야 한다.
```

- 사용자를 기준으로 분리해야 기능 변경의 여파를 최소화할 수 있다.


- DIP

```
추상타입이 있다면, 추상타입이 총괄해야 한다.
```

- 의존 역전 원칙은 소스코드에서의 의존을 역전시키는 원칙이다. runtime에서의 의존을 말하는 것이 아니다.
```java
public class FlowController {
    public void process() {
        FileDataReader reader = new FileDataReader(); 
        //runtime 의존은 FlowController가 FileDataReader에 의존
    }
}

public class FileDataReader implements ByteSource {
    .... //상세 구현에서 추상 타입에 의존
//소스코드에서의 의존은 FlowController가 ByteSource에 의존
}
```

### service locator
- 사용할 객체를 제공하는 책임을 갖는 객체를 Service Locator라고 한다.

```java
public class Worker {
    public void run() {
        JobQueue jobQueue = new JobQueue();
        Transcoder transcoder = new Transcoder();

        while(someRunningCondition) {
            jobData jobData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget());
        }
    }
}

public class JobCLI {
    public void interact() {
        printInputSourceMessage();
        String source = getSourceFromConsole();
        printInputTargetMessage();
        String target = getTargetFromConsole();

        JobQueue jobQueue = ...;
        jobQueue.addJob(new JobData(source, target));
    }
}
```

- locator를 transcoder라는 package 안에 넣은 이유는 패키지 간 순환 의존을 발생시키지 않기 위해서다.
- Locator가 package locator라면 transcoder package는 Locator.getInstance()에서 보듯 locator 패키지에 의존
- locator 패키지는 JobQueue에서 보듯 transcoder package에 의존
- 이렇게 하면 서로 맞물리는 의존 관계가 되어버린다.
- 순환 의존은 package의 변경이 다른 패키지에 영향을 줄 수 있다.
- 그럼 배포를 할때도 전혀 변경된 게 없어도 의존성이 있는 package 때문에 다시 배포를 해야 한다.
- 독립 배포가 불가능하게 된다.
```java
package transcoder
public class Worker {
    public void run() {
        JobQueue jobQueue = Locator.getInstance().getJobQueue();
        Transcoder transcoder = Locator.getInstance().getTranscoder();

        while(someRunningCondition) {
            jobData jobData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget());
        }
    }
}
package ui
public class JobCLI {
    public void interact() {
        printInputSourceMessage();
        String source = getSourceFromConsole();
        printInputTargetMessage();
        String target = getTargetFromConsole();

        JobQueue jobQueue = Locator.getInstance().getJobQueue();
        jobQueue.addJob(new JobData(source, target));
    }
}
package transcoder
public class Locator {
    private static Locator instance;
    private static Locator getInstance() {
        return instance;
    }

    public static void init(Locator locator) {
        this.instance = locator;
    }

    private JobQueue jobQueue;
    private Transcoder transcoder;
    public Locator(JobQueue jobQueue, Transcoder transcoder) {
        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }

    public JobQueue getJobQueue() {
        return jobQueue;
    }

    public Transcoder getTranscoder() {
        return transcoder;
    }
}
```

- 그럼 Locator 객체는 누가 초기화할까?
- 그 때 main 영역이 초기화 작업을 수행한다.

```java
public class Main {
    public static void main(String[] args) {
        JobQueue jobQueue = new FileJobQueue(); //new StreamingJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();// new mpegTranscoder();

        Locator locator = new Locator(jobQueue, transcoder);
        Locator.init(locator);

        final Worker worker = new Worker();
        Thread t = new Thread(new Runnable() {
            public void run() {
                worker.run(); //Locator는 FileJobQueue, FfmpegTranscoder를 사용
            }
        });
        JobCli cli = new JobCLI();
        cli.interact();
    }
}
```

- 이 경우 main -> application으로 의존성을 갖게 된다.
- main 영역을 변경해도 application은 변경되지 않는다.
- main 영역에서 application에 쓰일 객체를 교체하는 것에 부담이 적어진다.

- 만약 ServiceLocator를 만들 때, 타입만 다르게 구조가 같은 Locator 클래스를 만들어야 할 수도 있다. 그럴 때는 generics를 활용한다.

```java
public class Locator {
    private static Map<Class<?>, Object> objectMap = 
        new HashMap<Class<?>, Object>;

    public static <T> T get(Class<T> klass) {
        return (T) objectMap.get(klass);
    }

    public static void regist(Class<?> klass, Object obj) {
        objectMap.put(klass, obj);
    }
}

public static void main(String[] args) {
    Locator.regist(JobQueue.class, new FileJobQueue());
    Locator.regist(Transcoder.class), new FfmpegTranscoder());

    JobCLI jobCli = new JobCLI();
    jobCli.interact();
}

public class JobCli {
    public void interact() {
        .
        .
        .
        JobQueue jobQueue = ServiceLocator.get(JobQueue.class);
    }
}
```
### DI
- 생성자 방식으로 보통 만든다.
- setter 방식은 필요한 dependency를 생략했다면 compile 시가 아니라 runtime에 오류가 난다.
```java
public class Worker {
    private JobQueue jobQueue;
    private Transcoder transcoder;

    public Worker(JobQueue jobQueue, Transcoder transcoder) {
        if (jobQueue == null) {
            throw new IllegalArgumentException();
        }

        this.jobQueue = jobQueue;
        this.transcoder = transcoder;
    }

    public void run() {
        while(someRunningCondition) {
            JobData jobData = jobQueue.getJob();
            transcoder.transcode(jobData.getSource(), jobData.getTarget())
        }
    }
}
```
- 조립기를 별도로 분리해주면 소스코드를 변경할 때 편리하다.

```java
public class Assembler {
    public void createAndWire() {
        JobQueue jobQueue = new FileJobQueue();
        Transcoder transcoder = new FfmpegTranscoder();
        this.worker = new Worker(jobQueue, transcoder);
        this.jobCLI = new JobCLI(jobQueue);

        public Worker getWorker() {
            return this.worker;
        }

        public JobCLI getJobCLI() {
            return this.jobCLI;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        Assembler assembler = new Assembler();
        assembler.createAndWire();
        final Worker worker = assembler.getWorker();
        JobCLI jobCli = assembler.getJobCLI();
    }
}
```
- 위와 같이 객체 조립을 분리하면 XML로 따로 뺴서 객체를 조립해도 된다.
- 그게 바로 Spring legacy다.


- setter방식은 test할 때 많이 사용된다.
```java
@Test
public void shouldRunSuccessFully() {
    JobQueue mockJobQueue = Mock 객체;
    Transcoder mockTranscoder = Mock 객체;
    Worker worker = new Worker();
    worker.setJobQueue(mockJobQueue);
    worker.setTranscoder(mockTranscoder);
    worker.run():
}
```

- 주요 디자인 패턴

### strategy

- 아래와 같이 구현된 코드가 있다. target에 따른 if - else 분기문이다.
```java
public class Calculator {
    public int calculate(boolean firstGuest, List<Item> items) {
        int sum = 0;
        for (Item item: items) {
            if (firstGuest) {
                sum += (int) (item.getPrice() * 0.9); //첫 손님 10% 할인
            } else if (! item.isFresh()) {
                sum += (int) (item.getPrice() * 0.8); //덜 신선한 것 20% 할인
            } else {
                sum += item.getPrice();
            }
        }
    }
}
```

- ui 코드는 아래와 같이 될 것이다.
```java
public void onCalculationButtonClick() {
    Calculator cal = new Calculator();
    int price = cal.calculate(firstGuest, items);
}
```


- if - else문의 가장 큰 단점은 추가될 때마다 늘어난다는 점,
- 로직이 같이 추가되면 분석이 너무 어려워진다는 점이다.

```
서로 다른 계산 정책이 한 코드에 섞여 있다.
가격 정책이 추가될 때마다 calculate method 수정이 어려워진다.
```

- 할인과 관련된 책임은 할인 interface를 구현한 class에 맡긴다.
```java
public class Calculator {
    private DiscountStrategy discountStrategy;

    public Calculator(DiscountStrategy discountStrategy) {
        this.discountStrategy = discountStrategy;
    }

    public int calculate(List<Item> items) {
        int sum = 0;
        for (Item item: items) {
            sum += discountStrategy.getDiscountPrice(item);
        }
    }
}
```

- 하나의 할인정책에 전체금액과 아이템 할인 정책을 method로 가져올 수도 있다. 
```java
public interface DiscountStrategy {
    int getDiscountPrice(Item item);        //아이템 별 할인 정책
    int getDiscountPrice(int totalPrice);   //전체 금액 할인 정책
}
```

- 전체금액 할인 정책과 아이템 별 할인 정책을 분리할 수도 있다.
```java
public interface ItemDiscountStrategy {
    int getDiscountPrice(Item item);
}

public interface TotalPriceDiscountStrategy {
    int getDiscountPrice(int totalPrice);
}
```

```java
public class FirstGuestDiscountStrategy implements DiscountStrategy {

    @Override
    public int getDiscountPrice(Item item) {
        return (int) (item.getPrice() * 0.9);
    }
}
```

- ui 코드는 아래와 같다.
```java
private DiscountStrategy strategy;

public void onFirstGuestButtonClick() {
    strategy = new FirstGuestDiscountStrategy();
}

public void onLastGuestButtonClick() {
    strategy = new LastGuestDiscountStrategy();
}

//firstGuest버튼을 누르면 해당 할인 정책이 적용된다.
public void onCalculationButtonClick() {
    Calculator cal = new Calculator(strategy);
    int price = cal.calculate(items);
}
```

### template

- 일부를 빼고 동일한 절차를 가진 코드를 작성하기도 한다. 게시판에 파일을 insert하거나 update할 때, 특히 그러하다.
```java
public class DbAuthenticator {
    public Auth authenticate(String id, String pw) {
        User user = userDao.selectById(id);
        boolean auth = user.equalPassword(pw);
        if (!auth) {
            throw createException();
        }

        return new Auth(id,user.getName());
    }

    private AuthException createException() {
        
        return new AuthException();
    }
}

public class LdapAuthenticator {
    public Auth authenticate(String id, String pw) {
        boolean lauth = ldapClient.authenticate(id, pw);
        if(!auth) {
            throw createException();
        }

        LdapContext ctx = ldapClient.find(id);
            
        return new Auth(id, ctx.getAttribute("name"));
    }

    private AuthException createException() {
        
        return new AuthException();
    }
}
```

- template method는 하위 class가 아닌 상위 class가 기능 사용 여부를 결정한다.
```java
public abstract Authenticator {
    public Auth authenticate(String id, String pw) {
        if (!doAuthenticate(id, pw)) {
            throw createException();
        }

        return createAuth();
    }

    protected abstract boolean doAuthenticate(String id, String pw);

    private RuntimeException createException() {
        throw new AuthException();
    }

    protected abstract Auth createAuth(String id);
}

public class LdapAuthenticator extends Authenticator {
    @Override
    protected boolean doAuthenticate(String id, String pw) {
        return ldapClient.authenticate(id, pw); 
    }

    @Override
    protected Auth createAuth(String id) {
        LdapContext ctx = ldapClient.find(id);
        
        return new Auth(id, ctx.getAttribute("name"));
    }
}
```

- 템플릿 메서드와 전략 패턴을 함께 사용할 수도 있다.

```java
public <T> T execute(TransactionCallback<T> action) throws TransactionException {
    TransactionStatus status = this.transactionManager.getTransaction(this);
    T result;

    try {
        result = action.doInTransaction(status);
    } catch (RuntimeException ex) {
        rollbackOnException(status, ex);
        throw ex;
    }

    this.transactionManager.commit(status);
    return result;
}

transactionTemplate.execute(new TransactionCallback<String>() {
    public String doInTransaction(TransactionStatus status) {
        //트랜잭션 범위 안에서 실행될 코드
    }
})
```

### state
- 아래와 같은 요구가 있다고 가정해보자.

```
동작        조건            실행               결과
동전넣음    동전X           금액증가            제품선택가능하게변경
동전넣음    제품선택가능     금액증가            제품선택가능
제품선택    동전X           동작X               동전없음유지
제품선택    제품선택가능     제품주고잔액감소     잔액있으면제품선택가능,잔액없으면동전없음상태로 변경   
```

```java
public class VendingMachine {
    public static enum State {NOCOIN, SELECTABLE}

    private State state = State.NOCOIN;

    public void insertCoin(int coin) {
        switch (state) {
            case NOCOIN:
                increaseCoin(coin);
                state = State.SELECTABLE;
                break;
            case SELECTABLE:
                increaseCoin(coin);
        }

        public void select(int productId) {
            switch (state) {
                case NOCOIN:
                    break;
                case SELECTABLE:
                    provideProduct(productId);
                    decreaseCoin();
                    if (hasNoCoin()) {
                        state = State.NOCOIN;
                    }
            }
        }
    }
}
```

- 그러다가 자판기에 제품이 없는 경우에 동전을 넣으면 바로 동전을 되돌려달라는 요구가 들어왔다.
- 그럼 아래와 같이 state가 추가된다.
```java
public class VendingMachine {
    public static enum State {NOCOIN, SELECTABLE, SOLDOUT}

    private State state = State.NOCOIN;

    public void insertCoin(int coin) {
        switch (state) {
            case NOCOIN:
                increaseCoin(coin);
                state = state.SELECTABLE;
                break;
            case SELECTABLE:
                increaseCoin(coin);
                break;
           /* case SOLDOUT:
                returnCoin(); */
        }
    }

    public void seleect(int productId) {
        switch(state) {
            case NOCOIN:
                break;
            case SELECTABLE:
                provideProduct(productId);
                decreaseCoin();
                if (hasNoCoin()) {
                    state = state.NOCOIN;
                }
           /* case SOLDOUT: */
        }
    }
}
```

- 상태에 따라 동일한 기능 요청의 처리를 다르게 한다는 의미가 담겨있다.
- 이를 state 패턴으로 구현한다.
```java
public class VendingMachine {
    private State state;

    public VendingMachine() {
        state = new NoCoinState();
    }

    public void insertCoin(int coin) {
        state.increaseCoin(coin, this); //상태 객체에 위임
    }

    public void select(int productId) {
        state.select(productId, this); //상태 객체에 위임
    }

    public void changeState(State newState) {
        this.state = newState;  
    }
}

public class NoCoinState implements State {

    @Override
    public void increaseCoin(int coin, VendingMachine vm) {
        vm.increaseCoin(coin);
        vm.changeState(new SelectableState());
    }

    @Override
    public void select(int productId, VendingMachine vm) {
        SoundUtil.beep();
    }
}

public class SelectableState implements State {

    @Override
    public void increaseCoin(int coin, VendingMachine vm) {
        vm.increaseCoin(coin);
    }

    @Override
    public void select(int productId, VendingMachine vm) {
        vm.provideProduct(productId);
        vm.decreaseCoin();

        if (vm.hasNoCoin())  {
            vm.changeState(new NoCoinState());
        }
    }
}

```

- State 패턴의 장점은 class는 증가해도, if - else 조건문이 늘어나진 않아 본 코드가 복잡해지지 않는다는 점이다.
- 그리고 상태 별 class가 따로 존재하므로 해당 상태별 코드 수정을 건드리려면 해당 class만 건들면 된다는 점이다. 흩뿌려진 method를 수정하지 않아도 된다.
- 위에서는 State 객체에서 상태를 변경했다.
- 아래에서는 컨텍스트에서 상태를 변경해보자.

```java
public class VendingMachine {
    private State state;

    public VendingMachine() {
        state = new NoCoinState();
    }

    public void insertCoin(int coin) {
        state.increaseCoin(coin, this); //상태 객체에 위임
        if (hasCoin()) {
            changeState(new SelectableState());
        }
    }

    public void select(int productId) {
        state.select(productId, this); //상태 객체에 위임
        if (state.isSelectable() && hasNoCoin()) {
            changeState(new NoCoinState());
        } 
    }

    private void changeState(State newState) {
        this.state = newState;  
    }

    private boolean hasCoin() {

    }

    private boolean hasNoCoin() {
        return !hasCoin();
    }
}

public class SelectableState implements State {

    @Override
    public void increaseCoin(int coin, VendingMachine vm) {
        vm.increaseCoin(coin);
    }

    @Override
    public void select(int productId, VendingMachine vm) {
        vm.provideProduct(productId);
        vm.decreaseCoin();
    }
}
```

### 데코레이터
- 상속으로만 하면 여러 기능을 함께 제공할 때 계층 구조가 복잡해진다.
- 따라서 상속이 아닌 위임을 사용하는 decorator 패턴으로 바꾼다.

```java
public abstract class Decorator implements FileOut {
    private FileOut delegate; //위임 대상
    public Decorator(FileOut delegate) {
        this.delegate = delegate;
    }

    protected void doDelegate(byte[] data) {
        delegate.writer(data); //delegate에 쓰기 위임
    }
}

public class EncryptionOut extends Decorator {
    public EncryptionOut(FileOut delegate) {
        super(delegate);
    }

    public void write(Byte[] byte) {
        byte[] encryptedData = encrypt(data);
        super.doDelegate(encryptedData);
    }

    private byte[] encrypt(byte[] data) {

    }
}
```

- 데코레이터는 조합할 수 있다는 데 장점이 있다.
- 압축을 한 뒤 암호화를 해서 파일에 쓰고 싶다면 아래와 같이 Zipout class를 넣어주면 된다.
```java
FileOut delegate = new FileOutImpl();
FileOut fileOut = new EncryptionOut(delegate);
fileOut.write(data);

FileOut delegate = new FileOutImpl();
FileOut fileOut = new EncryptionOut(new ZipOut(delegate));
fileOut.write(data);
```

- 기능 적용 순서 변경도 쉽다.

```java
FileOut fileOut = new BufferedOut(EncryptionOut(new ZipOut(delegate))); // 버퍼 -> 암호화 -> 압축 -> 파일쓰기
FileOut fileOut = new EncryptionOut(new ZipOut(new BufferedOut(delegate))); //암호화 -> 압축 -> 버퍼 -> 파일 쓰기
```

- 다만 decorator 패턴은 기능 갯수가 적을 때 쓰는 게 좋다. 기능이 많으면 너무 복잡해진다
- 또한 데코레이션 객체가 비정상으로 동작할 때 어떻게 처리할 지 정해야 한다. 보통은 throw Exception보다는 로그를 남긴다.


### proxy 

- 제품 목록을 구성할 떄, 모든 이미지를 로딩시키면 불필요하게 메모리를 사용한다.
- 스크롤해서 보이는 것만 보여주면 되는데, 안보이는 것까지 로딩하기 때문이다.
- 또한 이미지 리소스를 로딩하는 데 시간이 더 오래걸려 반응성이 안 좋아진다.

- 그럴 때 Image class의 DynamicLoadingImage class를 사용할 수 있다.
- 하지만 이미지 로딩 방식을 변경하면 ListUI 코드를 변경해야 한다.
- 이 때 ListUI 코드를 변경하지 않고 로딩 방식을 교체할 수 있다.

```java
public class ProxyImage implements Image {
    private String path;
    private RealImage image;

    public ProxyImage(String path) {
        this.path = path;
    }

    public void draw() {
        if (image == null) {
            image = new RealImage(path); //최초 접근 시 객체 생성
        } 
        image.draw();
    }
}
```

```java
public class ListUI {
    private List<Image> images;
    public ListUI(List<Image> images) {
        this.images = images;
    }

        
    public void onScroll(int start, int end) {
        for (int i = start; i <= end; i++) {
            Image image = images.get(i);
            image.draw();
        }
    }
}
```

```java
List<String> paths = ...//이미지 경로 목록을 가져옴
List<Image> images = new ArrayList<Image>(paths.size());
for (int i = 0; i < paths.size(); i++) {
    if (i <= 4) { //상위 4개는 바로 이미지를 로딩
        images.add(new RealImage(paths.get(i)));
    } else {      //나머지는 이미지가 필요할 때 load
        images.add(new ProxyImage(paths.get(i)));
    }
}

ListUI listUI = new ListUI(images);
```

### adapter

- 게시글이 많아지면서 like의 성능이 떨어졌다.
- 검색 속도 문제 해결을 위해 Tolr라는 오픈소스 검색 서버를 도입했다.
- 문제는 TolrClient가 제공하는 interface가 내가 만든 SearchService와 맞지 않다는 점이다.
- 그럼 전부 TolrClient에 맞게 변경해야하는데, 작업량이 만만치 않다.
- 그럴 때 아래와 같이 adapter를 사용한다.
```java
public class SearchServiceTolrAdapter implements SearchService {
    private TolrClient tolrClient = new TolrClient();

    public searchResult search(String keyword) {
        TolrQuery tolrQuery = new TolrQuery(keyword);
        QueryResponse response = tolrClient.query(tolrQuery);
        SearchResult result = convertToResult(response);
        
        return result;
    }

    private SearchResult convertToResult(QueryResponse response) {
        List<TolrDocument> tolrDocs = response.getDocumentList().getDocuments();
        List<SearchDocument> docs = new ArrayList<SearchDocument>();
        for (TolrDocument tolrDoc : tolrDocs) {
            docs.add(new SearchDocument(tolrDoc.getId(), ...));
        }

        return new SearchResult(docs);
    }
}
```

- adapter pattern을 적용한 것이 바로 SLF4J다. 
- 자바 로깅, log4j, log4j2, logback 등의 프레임워크를 선택하여 사용할 수 있게 해준다.


### observer
- 웹 사이트의 상태를 확인해 응답 속도가 느리거나, 연결이 안되면 모니터링 담당자에게 이메일로 통지하는 시스템을 만든다고 해보자.

```java
public class StatusChecker {
    private EmailSender emailSender;

    public void check() {
        Status status = loadStatus();

        if (status.isNotNormal()) {
            emailSender.sendEmail(status);
        }
    }
}
```

- 여기서 긴급한 메시지는 SMS로 바로 알려달라는 요건이 추가되었다.
- 이를 반영해보자.

```java
public class StatusChecker {
    private EmailSender emailSender;
    private SmsSender smsSender;

    public void check() {
        Status status = loadStatus();

        if (status.isNotNormal()) {
            emailSender.sendEmail(status);
            smsSender.sendSms(status);
        }
    }
}
```

- 만약 회사 내부에서 사용하는 메신저로도 메시지를 보내달라는 요구가 들어오면?
```java
public class StatusChecker {
    private EmailSender emailSender;
    private SmsSender smsSender;
    private Messenger messengerSender;
    public void check() {
        Status status = loadStatus();

        if (status.isNotNormal()) {
            emailSender.sendEmail(status);
            smsSender.sendSms(status);
            messengerSender.sendMessenger(status);
        }
    }
}
```

- StatusChecker class가 변경되지 않게 observer pattern을 활용해보자.

```java
public abstract class StatusSubject {
    private List<StatusObserver> observers = new ArrayList<StatusObserver>();

    public void add(StatusObserver observer) {
        observers.add(observer);
    }

    public void remove(StatusObserver observer) {
        observers.remove(observer);
    }

    public void notifyStatus(Status status) {
        for (StatusObserver observer: observers) {
            observer.onAbnormalStatus(status);
        }
    }
}
```

```java
public class StatusChecker extends StatusSubject {
    public void check() {
        Status status = loadStatus();

        if (status.isNotNormal()) {
            super.notifyStatus(status);
        }
    }

    private Status loadStatus() {

    }
}
```

```java
public interface StatusObserver {
    void onAbnormalStatus(Status status);
}

public class StatusEmailSender implements StatusObserver {

    @Override
    public void onAbnormalStatus(Status status) {
        sendEmail(status);
    }

    private void sendEmail(Status status) {

    }
}
```


- 옵저버로 등록하였고, 시스템이 비정상이 되면 StatusChecker -> StatusEmailSender -> onAbnormalStatus 호출해 상태 정보 통지
```java
StatusChecker checker = new StatusChecker();
checker.add(new StatusEmailSender());
```

- observer를 이용하면 아래와 같이 FaultStatusSMSSender로 바꿔도 StatusChecker class는 바뀌지 않는다.
- 또한 기존에 있던 StatusEmailSender도 똑같이 넣어서 작동시킬 수 있다.
```java
StatusChecker checker = ...;
StatusObserver faultObserver = new FaultStatusSMSSender();
checker.add(faultObserver);
checker.add(new StatusEmailSender());
```

- 옵저버 객체가 기능을 수행할 떄, 주제 객체의 상태가 필요할 수 있다.
- FaultStatusSMSSender class는 장애일 때만 SMS를 전송하고, 응답속도가 느려지는 장애 이외의 비정상 상태는 메시지를 전송하지 않게 구현한다고 해보자.
- 이 떄 FaultStatusSMSSender는 상태값을 확인해야 한다.
```java
public abstract class StatusSubject {
    private List<StatusObserver> observers = new ArrayList<StatusObserver>();

    public void notifyStatus(Status status) {
        for (StatusObserver observer : observers) {
            observer.onAbnormalStatus(status); //상태를 옵저버에 전달
        }
    }
}

public class FaultStatusSMSSender implements StatusObserver {
    public void onAbnormalStatus(Status status) { //상태값 전달받음
        if (status.isFault()) { //전달받은 상태값 사용
            sendSMS(status);
        }
    }
}
```

- 만약 onAbnormalStatus()를 호출할 떄 전달한 객체만으로 부족하다면?
- 그럴 때는 직접 구현해야 한다. statusChecker를 받아 statusChecker.isContinuousFault() 조건이 추가된다.  
```java
public class SpecialStatusObserver implements StatusObserver {
    private StatusChecker statusChecker;
    private Siren siren;

    public SpeicalStatusObserver(StatusChecker statusChecker) {
        this.statusChecker = statusChecker;
    }

    public void onAbnormalStatus(Status status) {
        if (status.isFault() && statusChecker.isContinuousFault()) {
            siren.begin();
        }
    }
}
```

- GUI 프로그래밍에서 observer가 많이 사용된다.
- Button class의 상위타입인 View가 존재하고, View class가 옵저버 객체를 관리하기 위한 기능을 제공한다.
```java
public class MyActivity extends Activity implements View.OnClickListener {
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        ...
        Button loginButton = getViewById(R.id.main_loginbtn);
        loginButton.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        login(id, password);
    }
}

public class MyActivity extends Activity implements View.OnClickListener {
    public void onCreate(Bundle savedInstanceState) {
         super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        ...
        Button loginButton = (Button) findViewById(R.id.main_loginbtn); //동일한 OnClickListener 객체 등록
        loginButton.setOnClickListener(this);
        Button logoutButton = (Button) findViewById(R.id.main_logoutbtn); //동일한 OnClickListener 객체 등록
        logoutButton.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        //주제 객체를 구분하기 위한 분기. 동일한 onClickListener라도 구분 가능.
        if (v.getId() == R.id.main_loginbtn) {
            login(id, password);
        } else if (v.getId() == R.id.main_logoutbtn) {
            logout();
        }
    }
}
```

### mediator
- ??? ㅈㄴ 어렵다. 잘 모르겠다..
```java
private VideoMediator videoMediator;

public void onSelectedItem(int selectedIdx) {
    VideoInfo videoInfo = videoList.get(selectedIdx);
    videoMediator.selectVideo(videoInfo.getFile());
}
```

### abstract Factory
- 게임 플레이를 진행하는 Stage를 class로 나타내보자.
- 몇 단계인지 에따라 장애물, 보스, 적이 달라진다.

```java
public class Stage {
     ....   
    private void createEnemies() {
        for (int i = 0; i <= ENEMY_COUNT; i++) {
            if (stageLevel == 1) {
                enemies[i] = new DashSmallFlight(1,1); //공격/수비 1
            } else if (stageLevel == 2) {
                enemies[i] = new MissileSmallFlight(1,1);
            }
        }
        if (stageLevel == 1) {
            boss = new StrongAttackBoss(1, 10); //공격/수비 1/10
        } else if (stageLevel == 2) {
            boss = new CloningBoss(5, 20);
        }
    }

    private void createObstacle() {
        for (int i = 0; i <OBSTACLE_COUNT; i++) {
            if (stageLevel == 1) {
                obstacles[i] = new RockObstacle();
            } else {
                obstacles[i] = new BombObstacles();
            }
        }
    }
}
```

- if - else 문이 많아 꽤나 복잡하다.
- 이를 abstract Factory를 이용해 바꿔보자.

```java
public abstract class EnemyFactory {
    public static EnemyFactory getFactory(int level) {
        if (level == 1) {
            return EasyStageEnemyFactory(); //1레벨도 빡세면 EasyStageEnemyFactory를  SuperEasyStageEnemyFactory로 바꿀 수있다.
        } else {
            return HardEnemyFactory();
        }
    }

    public abstract Boss createBoss();
    public abstract SmallFlight createSmallFlight();
    public abstract Obstacle createObstacle();
}

public class EasyStageEnemyFactory extends EnemyFactory {
    public Boss createBoss() {
        return new StrongAttackBoss();
    }

    public abstract SmallFlight createSmallFlight() {
        return new DashSmallFlight();
    }

    public abstract Obstacle createObstacle() {
        return new RockObstacle();
    }
}

public class HardEnemyFactory extends EnemyFactory {
    public Boss createBoss() {
        return new CloningAttackBoss();
    }

    public abstract SmallFlight createSmallFlight() {
        return new MissileSmallFlight();
    }

    public abstract Obstacle createObstacle() {
        return new BombObstacle();
    }
}
```

```java
public class Stage {
   private EnemyFactory enemyFactory;
   private Enemies enemies;
   private Boss boss;
   private Obstacles obstacles;

   public class Stage(int level) {
        enemyFactory = EnemyFactory.getFactory(level);
   }   

   private void createEnemies() {
    for (int i = 0; i <= ENEMY_COUNT; i++) {
        enemies[i] = enemyFactory.createSmallFlight();
    }

    boss = enemyFactory.createBoss();
   }

    private void createObstacle() {
        for (int i = 0; i <OBSTACLE_COUNT; i++) {
           obstacles[i] = enemyFactory.createObstacle();
        }
    }
}
```

```java
public class Stage {
   private EnemyFactory enemyFactory;
   private Enemies enemies;
   private Boss boss;
   private Obstacles obstacles;

   public class Stage(int level, EnemyFactory enemyFactory) {
        this.level = level;
        this.enemyFactory = EnemyFactory enemyFactory;
   }   

   private void createEnemies() {
    for (int i = 0; i <= ENEMY_COUNT; i++) {
        enemies[i] = enemyFactory.createSmallFlight();
    }

    boss = enemyFactory.createBoss();
   }

    private void createObstacle() {
        for (int i = 0; i <OBSTACLE_COUNT; i++) {
           obstacles[i] = enemyFactory.createObstacle();
        }
    }
}
```