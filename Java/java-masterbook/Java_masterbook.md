- valueList를 넣으면, 당시의 요소만 넣는 게 아니라 주소값 자체를 넣는 것이다.
  - 따라서 나중에 추가해도 map.get에 접근이 가능하다!
```java
List<String> list = new ArrayList<>();
list.add("David");
list.add("MinHyunHwang");
list.add("Katya");
list.add("Kale");

Map<Integer, List<String>> map = new HashMap<>();
list.forEach(name -> {
    Integer nameLen = name.length();
    
    List<String> valueList = map.get(nameLen);
    if (valueList == null) {
        valueList = new ArrayList<>();
        map.put(nameLen, valueList);
    }
    valueList.add(name);
    System.out.println("map " + map.get(nameLen));
});
System.out.println(map);
```
- 위와 동일한 식이다.
- lambda의 method를 활용하면 더 간단하게 쓸 수 있다.
```java
List<String> list = new ArrayList<>();
list.add("David");
list.add("MinHyunHwang");
list.add("Katya");
list.add("Kale");

Map<Integer, List<String>> map = new HashMap<>();
list.forEach(name -> {
    Integer nameLen = name.length();
    List<String> valueList = map.computeIfAbsent(nameLen, key -> new ArrayList<>());
    valueList.add(name);
});
System.out.println(map);
```


- argument로 들어오는 것은 엔간하면 수정하지 않는게 좋다.
```java
public static void main(String[] args) throws Exception {
		Entity entity = new Entity();
		entity.value = 1;
		callByReference(entity);
		System.out.println("호출자: " + entity.value); //2
	}
	
	private static void callByReference(Entity entity) {
		entity.value = 2 ;
		System.out.println("수신자 : " + entity.value); //2
	}
```
- 하지만 만약에 argument의 값을 바꿀 때, 새로운 객체를 생성하고 그 객체의 값을 바꾸는 것은 원하지 않는 효과를 일으킨다.
```java
public static void main(String[] args) throws Exception {
		Entity entity = new Entity();
		entity.value = 1;
		callByReference(entity);
		System.out.println("호출자: " + entity.value); //1
	}
	
	private static void callByReference(Entity entity) {
		entity = new Entity();
		entity.value = 2 ;
		System.out.println("수신자 : " + entity.value); //2
	}
```


- Main.java를 Main.class로 compile--> javac Main.java 
- jar를 만들기 --> javv cf my.jar Main.class
- java classPath 설정 --> java -cp my.jar
- 클래스 패스를 설정하는 이유는 import해오는 것들을 돌리려면 classPath가 필요하기 때문이다.
- java -jar my.jar --> jar파일 실행하기. tomcat이나 jenkins 실행할 때 씀.

- 정수형 변수 int는 instance를 선언만 하고 초기화하지 않은 경우, 0이다. 
  - 반면에 Integer는 null이다.
  - 파일이나 통신 등에서 읽은 변수는 초기값 0인지, 실제로 0이라는 값이 지정된지 구별하기 어렵다.
  - 따라서 Integer로 선언하고 null로 초기화하자.

- 익명 클래스는 정의와 인스턴스화를 한꺼번에 한다.

```java
public interface TaskHandler {
    boolean handler(Task task);
}
public class AnnoymousClassSample {
    public static void main(String ...args) {
        TaskHandler taskHandler = new TaskHandler() {
            public boolean handle(Task task) {
                //task에 관한 처리
            }
        };
        Task task = new Task();
        taskHandler.handler(myTask);
    }
}
```

- 동일한 객체인지? 동일성(identity)을 따지는 것.
- 동일한 값인지? 동등성(eqaulity)를 따지는 것.
- 둘 중에 어떤 것을 비교할 지는 hashCode와 equals()를 override하여 결정.
- 그냥 Object는 identity를 비교하지만, String은 eqaulity를 비교함.

- HashSet을 예로 들어보자.
- hashCode를 override하지 않았기 때문에 Object의 hashCode를 가져오게 된다.
- 그렇기에 객체의 주소값이 다른 같은 요소를 가진 hashSet은 서로 다르게 취급된다.
- Set은 중복 key를 허용하면 안되는데, 허용하게 된다.

```java
@RequiredArgsConstructor
public class Employee {
    private int employeeNO;
    private String employeeName;

    @Override
    public boolean equals(Object obj) {
        .
        .
        .
    }
    //hashCode는 override하지 않음.
}

Employee employee1 = new Employee(1, "정시온");
Employee employee2 = new Employee(1, "정시온");
Set<Employee> employees = new HashSet<>();
employees.add(employee1);
employees.add(employee2);
sysout(employees.size()); //2
```

- 따라서 hashCode를 override해주자. Eclipise에서 해주는 거로 override해주면 된다.
```java
@RequiredArgsConstructor
public class Employee {
    private int employeeNO;
    private String employeeName;

    @Override
    public boolean equals(Object obj) {
        .
        .
        .
    }

    @Override
    public boolean hashCode(Object obj) {
        .
        .
        .
    }

    //hashCode는 override하지 않음.
}
```

- toString을 구현할 때도 eclipse자체에서 해주는 기능으로 해도 된다.
- 아니면 Apache Commons의 Commons Lang을 써도 된다.
```java
@Override
public String toString() {
    return ToStringBuilder.reflectionToString(this);
}
```

- public static final은 타입안전이 아니다.
- 타입안전이란 변수에 타입을 할당함으로써 부정한 동작을 방지한다는 의미다.
```java
public static final String COLOR_BLUE = "blue";
public void processColor(String color) {//상수만 갖고 오고 싶지만.. 상수에 type지정이 불가능.
    //green으로 넣으면 대 참 사
}
```

```java
public enum Color {
    BLUE("파랑"), GREEN("초록"), RED("빨강");

    private final String value;
    
    Color(String value) {
        this.value = value;
    }
    
    public String getValue() {
    	return value;
    }
}

public static void main(String[] args) throws Exception {
    call(Color.RED);
}

private static void call(Color color) {
        String value = color.getValue();
        System.out.println(value);
}
```

- 비교는 Comparator로 수행하는 경우가 많다.
- business logic을 넣은 것들을 비교하려면 Comparable로는 부족하다.
- Comparable은 해당 클래스에 정해진 정렬 방식을 따르기 때문이다.
```java
public class Student {
	private String name;
	private int score;
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public int getScore() {
		return score;
	}
	public void setScore(int score) {
		this.score = score;
	}
}
```

```java
Student[] students = {
    new Student("Ken", 100),
    new Student("Kn", 60),
    new Student("Ke", 80),
}

Comparator<Student> comparator = new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {//o1이 왼쪽이고, o2가 오른쪽인 상황.
        return Integer.compare(o2.getScore(),o1.getScore()); //오른쪽을 앞으로 하면 내림차순, 왼쪽을 앞으로 하면 오름차순 . 여기선 o2인 오른쪽
        //즉 두번째 들어온 인수(여기선 o2)가 compare 시에 앞에 있으면 내림차순, 뒤에 있으면 오름차순
    }
};

Arrays.sort(students, comparaotr);
for (Student student : students) {
    sysout(student.getName() + ":" + student.getScore());
}
```

```java
Student[] students = {
    new Student("Ken", 100),
    new Student("Kn", 60),
    new Student("Ke", 80),
}

Comparator<Student> comparator = new Comparator<Student>() {
    @Override
    public int compare(Student o1, Student o2) {
        return o1.getName().compareTo(o2.getName()); //두 번째 들어온 인수가 compare 시에 앞에 있으면 내림차순, 뒤에 있으면 오름차순
    }
};

Arrays.sort(students, comparaotr);
for (Student student : students) {
    sysout(student.getName() + ":" + student.getScore());
}
```


- 컬렉션은 복수의 데이터를 좀 더 다루기 쉬운 구조를 일컫는다.
- 이러한 컬렉션에 유틸리티까지 포함되면 그것이 컬렉션 프레임워크다.
- Arrays의 asList로 만든 경우, 요소의 추가, 제거, 수정 등이 불가능하다. 읽기 전용이다. iterator 순회도 불가능하다.
```java
List<Integer> integerList = Arrays.asList(1,62,31,1,54,31); //읽기 전용 list.
integerList.add(4); //java.lang.UnsupportedOperationException
List<Integer> integerList = new ArrayList<>(Arrays.asList(1,62,31,1,54,31));
integerList.add(4); //ok
```

- 요소를 살펴보려면 iterator를 사용할 수 있다.
```java
List<Integer> integerList =  new ArrayList<>(Arrays.asList(1,62,31,1,54,31)); //Arrays.asList()로는 안 된다.
integerList.add(4);
System.out.println(integerList.toString());
for(Iterator iteraotr = integerList.iterator(); iteraotr.hasNext();) {
    Integer element = (Integer) iteraotr.next();
    System.out.println(element);
}
```

- thread-safe하게 쓸 때는 CopyOnWriteArrayList class를 쓰기도 한다.
- 여때까지는 ArrayList를 주로 살펴보았다. LinkedList라는 것을 간략하게 살펴보자.
- LinkedList는 for문에서 index와 같이 쓰면 성능이 좋지 않다.


- Map의 경우, get을 통해 얻어올 게 없다면 null을 반환한다.
- Map의 containsValue()는 사용하지 말자. linear search라서 size가 커질수록 오래 걸린다.
  - 대신 containsKey()를 활용하자. 
- thread-safe하게 쓸 때는 ConcurrentHashMap을 사용한다.


- Set은 value가 중복되지 않는다.
- List를 Set으로 바꿀 수 있다.
```java
List<Integer> integerList = Arrays.asList(1,62,31,1,54,31); //읽기 전용이어도 Set으로 변환 가능
Set<Intger> integerSet = new HashSet<>(integerList);
```

- thread-safe하게 쓸 때는 ConcurrentHashMap을 사용한다. Set은 Map으로 Set을 만들수 있기 때문에 그렇다.
- Set의 내부에 Map이 존재하며, Set에 추가도니 요소는 Map의 key로 유지된다.
```java
Set<Integer> concurrentHashSet = Collections.newSetFromMap(new ConcurrentHashMap<Integer, Boolean>());
```

- 그 밖에는 Queue가 주로 사용된다.
- Queue는 데이터의 일시 보관 용도로 자주 사용된다.
- 다만 그경우, 보관 처리와 추출처리는 별개의 스레드로 실시해야 한다.

## Stream
- Map interface에는 stream처리에 적절한 게 없다.
- 그래서 Set으로 변환이 필요하다.

```java
Map<String,String> map = new HashMap<>();
map.put("1","Haeun");
map.put("2","Shin");
map.put("3","Shion");

Stream<Entry<String,String>> stream = map.entrySet().stream();
stream.forEach(e -> System.out.println(e.getKey() + ":" + e.getValue()));
```

## Exception handling
- 아래의 try ~ catch처럼 catch로 잡아도 독자 exception으로 wrapping하는 게 좋다.
- 또한 throws Exception은 정말로 엔간하면 피해야 한다.
```java
class Manager {

    public void sendPerson() throws ManagerException {
        try{
            Person person = dao.readPerson();
            Socket socket = getSocket();
            OutputStream os = socket.getOutputStream();
            String personJson = objectMapper.writeValueAs(person);
            os.write(personJson);
        } catch (SQLException | SocketException | OutputStreamException | SerializationException e) {
            throw new ManagerException("error text", e);
        }
    }

}
```

- 람다에서 필요한 예외처리는 람다 안에서 이뤄져야 한다.
- 아래와 같이 try ~ catch를 람다 밖에서 감싸면 parallelStream을 실행할 떄 error catching이 제대로 이뤄지지 않는다.
```java
try(BufferedWriter writer = Files.newBufferedWriter(Paths.get(W_FILENMAE))) {
    lines.forEach(s -> writer.write(s + '\n'));
} catch (IOException ioex) {
    ioex.printStackTrace();
    throw new UncheckedIOException(ioex);
}
```

- 아래와 같이 람다 안에서 error를 catching하게 바꾸자.
```java
try(BufferedWriter writer = Files.newBufferedWriter(Paths.get(W_FILENMAE))) {
    line.forEach(s -> {
        try {
            writer.write(s + '\n');
        } catch (IOException ioex) {
            ioex.printStackTrace();
    throw new UncheckedIOException(ioex);
        }
    })
}
```

## encoding
- 개발 중에 문제가 없었는데 실제 운영에서 encoding 문제가 날 수 있다.
- 그 경우 문자가 꺠지게 된다. 대부분 default encoding의 차이 때문이다.
- 따라서 Java에서 default encoding을 사용하지 않게끔 한다.
- 특시 FileReader와 FileWriter는 쓰면 안 된다. FileInputStream, InputStreamReader로 대체한다.

## 서로게이트 페어
- 2개의 char로 하나의 문자를 표현하는 경우가 있는데, 이를 서로게이트 페어라고 한다.
- 이모티콘의 경우에 특히 2개의 문자로 이뤄져 있을 수 있다. 이 경우 문자열 길이 판정에 특수한 method가 필요하다.
- 만약 서로게이트 페어를 금지하려면 간단하게 아래와 같이 Character class의 method를 활용한다.
```java
char[] chars = str.toCharArray();
for (char c : chars) {
    if (Character.isLowSurrogate(c) || Character.isHighSurrogate(c)) {
        System.out.println("서로게이트 페어가 포함되어 있는 문자열");
        return true;
    }
}

System.out.println("서로게이트 페어가 포함되어 있지 않은 문자열");
return false;
```

- 만약 서로게이트 페어를 허용하려면 String의 codePointCount()를 활용한다.
```java
String str = "코알라(이모티콘)";
System.out.println(str.length());                        //5
System.out.println(str.codePointCount(0, str.length())); //4
```

## Java 7 이후
- java.nio.file을 활용해야 한다. 
- java.io.file을 활용하면 안 된다. 
- 사실 제일 좋은 것은 Paths class를 활용하는 것이다.
```java
Path path1 = Paths.get("C:/work/sample.txt");
System.out.println(path1.getParent());                   //부모 디렉토리 취득. C:/work
System.out.println(path1.resolveSibling("sample2.txt")); //형제 획득.

File file = path1.toFile(); //File instance로 변환
URI uri = path.toUri();     //URI instance로 변환
```

- 파일 복사
```java
Path fromFile = Paths.get("C:/work/sample.dat");
Path toFile = Paths.get("C:/work/copy.dat");

try { 
    Files.copy(fromFile, toFile);
} catch (IOException ex) {
    System.err.println(ex);
}
```

- 파일 삭제

```java
Path path = Paths.get("C:/work/sample.dat");

try {
    Files.delete(path);
} catch (NoSuchFileException ex |
            DirectoryNotEmptyException ex |
            IOException ex) {
    sysout(ex);
}
```

- 파일 작성

```java
Path path = Paths.get("C:/work/new.dat");
try {
    Files.createFile(path);
} catch (FileAlreadyExistsException ex) {
    sysout(ex);
}
```

- 폴더 작성
```java
Path path = Paths.get("C:/work/newDir");
try {
    Files.createDirectory(path);
} catch (NoSuchFileException ex) {
    sysout(ex);
}
```

- 임시파일 작성
```java
Path path = Paths.get("C:/work/newDir");
try {
    Path tempPath = Files.createTempFile(path, "pre", ".tmp");
    sysout(tempPath);
} catch (IOException ex) {
    sysout(ex);
}
```

- 프로퍼티 읽기

```java
Path path = Paths.get("mail.properties");
try (BufferedReader reader = Files.newBufferedReader(path, StandardCharset.UTF_8)) {
    Properties properties = new Properties();
    properties.load(reader);

    String address = properties.getProperty("system.mail.address");
    sysout(address);
} catch (IOException ex) {

}
```

## Date and Time API

- LocalDateTime -> 문자열

```java
LocalDateTime date = LocalDateTime.now();
DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss.SSS").format(date); //2024/01/03 07:18:51
```

- 문자열 -> LocalDateTime

```java
String strDate = "2017/10/14 07:18:51";
TemporalAccessor parsed = DateTimeFormatter.ofPattern("yyyy/MM/dd HH:mm:ss")
                                            .parse(strDate);
LocalDateTime date = LocalDateTime.from(parsed);
```

- ISO_LOCAL_DATE => yyyy-MM-dd
- ISO_LOCAL_DATE_TIME => yyyy-MM-dd'T'HH:mm:ss

```java
TemporalAccessor parsed = DateTimeFormatter.ISO_LOCAL_DATE.parse(strDate);
LOcalDate date = LocalDate.from(parsed);
```

- test를 나중에 하려면 private 가시성보다는, protected가 좋다.


## lifeCycle
- 인스턴스 변수 대신 local 변수를 사용하게 변경
  - 라이프 사이클을 짧게 하여 사고를 방지
```java
@Setter
public class EmployeeService {
    private int id; //instance 변수
    private String name;
    private LocalDate birth;

    public void create() {

    }

    public void get(int id) {

    }
}

public class MainService {
    private EmployeeService employeeService = new employeeService();

    public void register() {
        this.employeeService.setId(1);
        this.employeeService.setName("시온");
        this.employeeService.setBirth(LocalDate.of(1980,2,7));
        this.employeeService.create();
    }
}
```

- Service가 아니라 model class를 따로 만든다.
- 그리고 그 class를 parameter로 가져와 local 변수로 만든다.
```java
public class Employee {
    public int id;
    public String name;
    public LocalDate birth;
}

public class EmployeeService {
    public void create(Employee employee) {
        //employee.id, 
        .
        .
    }

    public Employee get(int id) {

    }
}

public class MainService {
    private EmployeeService employeeService = new EmployeeService();

    public void register() {
        Empployee employee = new Employee();
        employee.id = 1;
        employee.name = "시온";
        employee.birth = LocalDate.of(1980,2,7);
        this.employeeService.create(employee);
    }

}
```

- 라이플 사이클을 오히려 길게 해서 성능을 높일 수도 있다.
- 라이프 사이클을 짧게 하면 GC가 발생횟수가 증가한다. GC는 애플리케이션 성능을 악화시키는 주요 요인이다.
- 특히 변수가 없는 class는 길게 하면 좋다. 그 경우, Utility가 대표적인데, static 변수로 가져오는 것도 좋은 선택이다.

```java
public class StringUtils {
    public boolean isEmpty(String text) {
        return (text == null || text.length() == 0); 
    }
}

public class MainService {
    private static StringUtils stringUtils = new StringUtils();
    
    public void execute(String text) {
        if (StringUtils.isEmpty(text)) {
            
        }
    }
}
```

- 아니면 method를 static으로 만들수도 있다.
```java
public class StringUtils {
    public static boolean isEmpty(String text) {
        return (text == null || text.length() == 0); 
    }
}

public class MainService {
    
    public void execute(String text) {
        if (StringUtils.isEmpty(text)) { //대문자. instance 변수 선언 X
            
        }
    }
}
```

- abstract class는 향후 Mock 객체로 만들 떄 활용이 불가능하다.
- 엔간하면 interface로 만들어놓자.
- interface를 쓰면 Java 8부터는 factory class를 굳이 활용하지 않아도 된다.
- 만약 factory class를 쓴다면 아래와 같다.
```java
public class FooFactory {
    public static Foo newInstance(String message) {
        return new DefaultFoo(message);
    }
}

public class ApiClient {
    public staic void main(String ...args) {
        Foo foo = FooFactory.newInstance("Hello Foo!");
        System.out.println(foo.say());
    }
}
```
- Factory class 대신 interface default method를 쓴다면 아래와 같다.
```java
public interface Foo {
    String say();

    static Foo newInstance(String message) {
        return new DefaultFoo(message);
    }
}

class DefaultFoo implements Foo {
    private String message;

    DefaultFoo(String message) {
        this.message = message;
    }

    @Override
    public String say() {
        return this.message;
    }
}

public class ApiClinet {
    public static void main(String ...args) {
        Foo foo = Foo.newInstance("Hello Foo!");

        System.out.println(foo.say());
    }
}
```

## design pattern
- AbstractFacotry 패턴
  - 일련의 인스턴스군을 모아서 생성하기
```
DBMS를 가져올 때 필요한 instance를 모아둔다.
```
```java
//interface로 만들어 어떤 DBMS든 가져올 수 있게끔 만든다.
public interface Facotry {
    Connection getConnection();

    Configuration getConfiguration();
}

//abstrac class로 만들어 어떤 Connection 방식이든 가능하게 한다.
public abstract class Connection {

}

//abstrac class로 만들어 어떤 Configuration 방식이든 가능하게 한다.
public abstract class Configuration {

}

//Factory interface를 구현하여 PostgreSQLFactory와 관련된 connection, configuration을 얻어오게 한다.
public class PostgreSQLFactory implements Facotry {
    @Override
    public Connection getConnection() {
        return new PostgreSQLConnection();
    }

    @Override
    public Confugration getConfiguration() {
        return new PostgreSQLConfiguration();
    }
}

//실제 PostgreConnection을 구현하는 class다.
public class PostreSQLConnection extends Connection {

}

//실제 PostgreSQLConfiguration을 구현하는 class다.
public class PostgreSQLConfiguration extends Configuration {

}
```

- 아래는 Postgre가 아닌 Mysql용으로 만든 것이다.
```java
public class MYSQLFactory implements Factory {
    @Override
    public Connection getConnection() {
        return new MYSQLConnection();
    }

    @Override
    public Configuration getConfiguration() {
        return new MYSQLConfiguration();
    }
}

public class MYSQLConnection extends Connection {

}

public class MYSQLConfiguration extends Configuration {

}
```

- 실제 Factory 사용 class는 아래와 같다.
```java
public class SampleMain {
    public static void main(String ...args) {
        String env = "PostgreSQL";

        Facotry facotry = createFactory(env);
        Connection connection = facotry.getConnection();
        Configuration configuration = factory.getConfiguration();
    }

    private static Factory createFactory(String env) {
        switch (env) {
            case "PostgreSQL":
                return new PostgreSQLFactory();
            case "MYSQL":
                return new MySQLFactory();
            default:
                throw new IllegalArgumentException(env);
        }
    }
}
```

- 즉 순서는 아래와 같다.

```
Factory에 필요한 기능을 확장할 수 있게 abstract class를 만든다. --> abstract class Connection, Configuration
실게 기능을 구현한 class를 만든다. --> class PostreSQLConnection extends Connection, class PostgreSQLConfiguration extends Configuration
Facotry용 interface를 만든다. --> Facotry interface
경우에 맞는 실제 Facotry class를 만든다. --> class MYSQLFactory implements Factory
실제 Factory를 생성한다 --> Facotry facotry = new PostgreSQLFactory();, new MySQLFactory();
```

- Builder 패턴
  - 복합화된 instance의 생성 과정을 은폐한다.

```java
public interface Builder {
    void createHeader();
    
    void createContents();

    void createFooter();

    Page getResult();
}

public class Page {
    private String header;

    private String content;

    private String footer;
}

public class TopPage extends Page {

}

public class Director {
    private Builder builder;

    public Director(Builder builder) {
        this.builder = builder;
    }

    public Page construct() {
        builder.createHeader();
        builder.crateContents();
        builder.createFooter();

        return builder.getResult();
    }
}

public class TopPageBuilder implements Builder {
    private TopPage page;

    public TopPageBuilder() {
        this.page = new TopPage();
    }

    @Override
    public void createHedaer() {
        this.page.setHeader("Header");
    }

    @Override
    public void createContents() {
        this.page.setContent("Contents");
    }

    @Override
    public void createFooter() {
        this.page.setFooter("Footer");
    }

    @Override
    public Page getResult() {
        return this.page;
    }
}
```

- 위처럼 Builder를 만들어두면 복잡한 instance를 만드는 과정이 전부 은폐되고 3줄짜리로 instance를 만드는 것처럼 보이게 된다.
```java
public class SampleMain {
    public static void main(String ...args) {
        Builder builder = new TopPageBuilder();
        Director director = new Director(builder);

        Page page = director.construct();
    }
}
```

- Adapter 패턴
  - 인터페이스에 호환성이 없는 클래스들을 조합시키기
  - 기존 시스템과 새로운 시스템이 완전히 다른 method를 사용하는 경우 사용할 수 있다.
```java
public class OldSystem {
    public void oldProcess() {
        //기존처리
    }
}

public abstract class Target {
    abstract void process();
}

public class Adapter extends Target {
    private OldSystem oldSystem;

    public Adapter() {
        this.oldSystem = new OldSystem();
    }

    @Override
    public void process() {
        this.oldSystem.oldProcess();
    }
}

public class SampleMain {
    public static void main(String[] args) {
        Target target = new Adapter();
        target.process();
    }
}
```

- Composite 패턴
  - 재귀적 구조 쉽게 처리
  - 파일 시스템에 쓰기 좋다.

- 아래는 실제 Java의 File, Directory class는 아니다. sudo다.
```java
public interface Entry {
    void add(Entry entry);

    void remove();

    void rename(String name);
}

public class File implmenets Entry {
    private String name;

    public File(String name) {
        this.name = name;
    }

    @Override
    public void add(Entry entry) {
        throw new UnsupportedOpertaionException();
    }

    @Override
    public void remove() {
        System.out.println(this.name + "를 삭제했다.");
    }

    @Override
    public void rename(String name) {
        this.name = name;
    }
}

public class Directory implements Entry {
    private String name;

    private List<Entry> list;

    public Directory(String name) {
        this.name = name;
        this.list = new ArrayList<>();
    }

    @Override
    public void add(Entry entry) {
        list.add(entry);
    }

    @Override
    public void remove() {
        Iterator<Entry> itr = list.iterator(); //Entry를 구현한 class는 다 들어올 수 있다. 그게 File과 Directory다.
        while (itr.hasNext()) {// 한마디로 File인지, Directory인지 신경안쓰고 모두 remove가 가능하다.
            Entry entry = itr.next();
            entry.remove();
        }
        System.out.println(this.name + "을 삭제했다.");
    }

    @Override
    public void rename(String name) {
        this.name = name;
    }
}
```

- Command 패턴
  - 명령을 instance로 취급한다.
  - 할인 정책 같은 곳에 많이 쓰인다.

```java
@Setter
public abstract class Command {
    protected Book book;

public abstract void execute();
}

@Getter
@Setter
@RequiredArgsConstructor
public class Book {
    private double amount;
}

public class DiscountCommand extends Command {
    @Override
    public void execute() {
        double amount = book.getAmount();
        book.setAmount(amount * 0.9);
    }
}

public class SpecialDiscountCommand extends Command {
    @Override
    public void execute() {
        double amount = book.getAmount();
        book.setAmount(amoutn * 0.7);
    }
}

public class SampleMain {
    public static void main(String... args) {
        Book comic = new Book(5000);

        Book technicalBook = new DiscountCommand();

        Command discountCommand = new DiscountCommand();

        Command specialDiscountCommand = new SpecialDiscountCommand();

        discountCommand.setBook(comic);
        discountCommand.execute();
        System.out.println("할인 후 금액은 " + comic.getAmount() + "원");

        discountCommand.setBook(technicalBook);
        discountCommand.execute();
        System.out.println("할인 후 금액은 " + technicalBook.getAmount() + "원");

        speicalDiscountCommand.setBook(technicalBook);
        speicalDiscountCommand.execute();
        System.out.println("할인 후 금액은" + technicalBook.getAmount() + "원");
    }
}
```

- Strategy 패턴
  - 처리 알고리즘(전략)을 간단하게 전환
  - 처리 조건에 따라 전략을 바꿀 때 좋음. ex) 할인

```java
public interface Strategy {
    void discount(Book book);
}

@RequiredArgsConstructor
@Setter
@Getter
public class Book {
    private double amount;
}

public class DiscountStrategy implements Strategy {
    @Override
    public void discount(Book book) {
        double amount = book.getAmount();
        book.setAmount(amount * 0.9);
    }
}

public class SpecialDiscountStrategy implements Strategy {
    @Override
    public void discount(Book book) {
        double amount = book.getAmount();
        book.setAmount(amount * 0.7);
    }
}

public class Shop {
    private Strategy strategy;

    public Shop(Strategy strategy) {
        this.strategy = strategy;
    }

    public void setStrategy(Strategy strategy) {
        this.strategy = stratgey;
    }

    public void sell(Book book) {
        this.stratgey.discount(book);
    }
}

public class SampleMain {
    public static void main(String... args) {
        Book comic = new Book(5000);

        Book technicalBook = new Book(25_000);

        Strategy discountStrategy = new DiscountStrategy();

        Shop shop = new Shop(discountStrategy);
        shop.sell(comic);
        System.out.println("할인 후 금액은 " + comic.getAmount() + "원" );

        shop.setStrategy(specialDiscountStrategy);
        shop.sell(technicalBook);
        System.out.println("할인 후 금액은" + technicalBook.getAmount() + "원");
    }
}
```

- Observer pattern
  - 어떤 인스턴스의 상태가 변화할 때 그 인스턴스 자신이 상태의 변화를 통지하는 구조를 제공한다.
  - 다른 시스템으로부터 데이터를 수신, 사용자가 버튼을 누름 -> 상태 변화의 계기. 이걸 감지하여 처리하는 프로그램 구현
  - 상태가 바뀔 때 필요한 처리 호출하면 상태 보관 클래스와 호출 클래스가 결합도가 높아짐. 확장성 떨어짐

```java
public interface Observer {
    void update(Subject subject);
}

public abstract class Subject {
    private List<Observer> observers = new ArrayList<>();

    public void addObserver(Observer observer) {
        this.observers.add(observer);
    }

    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(this);
        }
    }
    public abstract void execute();
}

public class Client implements Observer {
    @Override
    public void update(Subject subject) {
        System.out.println("통지를 수신했다.");
    }
}

public class DataChanger extends Subject {
    private int status;

    @Override
    public void execute() {
        status++;
        System.out.println("상태가" + status + "로 바뀌었다.");
        notifyObservers(); 
    }
}

public class SampleMain {
    public static void main(String... args) {
        Observer observer = new Client();        // 상태 변경을 감시
        Subject dataChanger = new DataChanger(); // 상태 변경을 통지

        dataChagnger.addObserver(observer);
        for (int count = 0; count < 10; count ++){
            dataChanger.execute();

            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

- 핵심은 Subject는 통지할 Observer를 보관한다.
- notifyObserver method가 호출되면 Observer에 통지한다. (update method 호출)
- 정보를 '통지'하는 구조가 Observer interface와 Subject 추상 클래스에서 제공된다.
- 실제 처리는 각각을 구현하고 상속한 class에서 이뤄진다.


## Thread safe
- thread safe하게 소스코드를 짜는 게 필요하다.
- Thread safe란 아래와 같은 의미를 지닌다.
```
여러 쓰레드에서 읽거나 써도 데이터가 파괴되지 않는다.
여러 쓰레드에서 읽거나 써도 오류가 없다.
여러 쓰레드에서 읽거나 써도 deadlock이 없다.
```

- thread - safe가 아닌 대표적인 상황들이다.
```
int 증가처리
SimpleDateFormate의 parse method
HashMap의 put
ArrayList의 add, remove
long으로의 대입
```

- int 증가처리
```java
public class IntIncrement {
    public static void main(String ...args) {
        IntHolder holder = new IntHolder();
        Thread th1 = new Thread(new IntIncrementer("thread-1", holder));
        Thread th2 = new Thread(new IntIncrementer("thread-2", holder));
        th1.start();
        th2.start();

        try {
            th1.join();
            th2.join();
            int result = holder.getResult();
            System.out.println("result: " + result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public class IntHolder {
    private int intNum = 0;

    public int getResult() {
        return intNum;
    }

    public void increment() {
        intNum++;
    }
}

public class IntIncrementer implements Runnable {
    private String name;
    private IntHolder holder;

    public IntIncrementer(String argName, intHolder argHolder) {
        name  = argName;
        holder = argHolder;
    }

    @Override
    public void run() {
        System.out.println("[" + name + "] started.");
        for (int counter = 0; counter < 1000000; counter++) {
            holder.increment();
        }
        System.out.println("[" + name + "] finished. ");
    }
}
```

```
[thread-2] started.
[thread-1] started.
[thread-1] finished.
[thread-2] finished.
result: 1097061 //매번 달라짐
```

- 원래 정상적으로 작동했다면 위의 소스코드는 result가 2_000_000이 나왔어야 한다.
- 그런데 한참 못미친다. 그 이유는 두 단계로 이뤄지기 때문이다.
```
1- 현재 값 취득
2- 취득 값에 1을 더해서 기록
```

- 그런데 이 두 단계가 서로의 스레드에 의해 간섭을 받는다.
- 그럼 1을 더해서 기록해도 덮어 씌워진다. 증가시키는 행위 자체가 없던 거처럼 되는 것이다.

- SDF의 format도 synchronized로 보호받지 않기 때문에 가장 마지막에 수정한 날짜로 format된다.

- HashMap을 여러 스레드에서 동시에 접근하여 put하면 무한 루프가 발생하기도 한다.
```java
public class HashMapLoop implements Runnable {
    final Map<Integer, Integer> map = new HashMap<>();

    @Override
    public void run() {
        for (int i = 0; i < 100000000; i++) {
            int key = i % 1_000_000_000;
            if (map.containsKey(ke)) {
                map.remove(key);
            } else {
                map.put(key,i);
            }
        }
    }

    public void runLoop() throws InterruptedException {
        Thread th1 = new Thread(this);
        Thread th2 = new Thread(this);
        System.out.println("start.");
        th1.start();
        th2.start();
        th1.join();
        th2.join();
        System.out.println("finished.");
    }

    public static void main(String ...args) throws Exception {
        new HashMapLoop().runLoop();
    }
}
```

- long타입으로의 대입은 32 bit JVM을 쓰는 경우 문제가 된다.
- long이 64bit기 때문에, 상위 32bit와 하위 32bit를 개별조작하기 때문에 조작이 중복되면 예상치 못한 값이 되기 때문이다.
```java
public class IncrementLongSample {
    public static void main(String ...args) {
        LongHolder holder = new LongHolder();
        Thread th1 = new Thread(new LongPlusSetter("thread -1", holder));
        Thread th2 = new Thread(new LongPlusSetter("thread -2", holder));
        th1.start();
        th2.start();

        try {
            th1.join();
            th2.join(); 
            long result = holder.getResult();
            System.out.println("result: " + result);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}


public class LongHolder {
    private long longNum = 0;

    public long getResult() {
        return longNum;
    }

    public void setPlus() {
        longNum = 1;
        check(longNum);
    }

    public void setMinus() {
        longNum = 1;
        check(longNum);
    }

    public void check(long longNum) {
        if (longNum != 1 && longNum != 1-) {
            throw new RuntimeException("longNum: " + longNum);
        }
    }
}

public class LongPlusSetter implements Runnable {
    private String name;
    private LongHolder holder;

    public LongPlusSetter(String argName, LongHolder argHolder) {
        name = argName;
        holder = argHolder;
    }

    public void run() {
        System.out.println("[" + name + "] started.");
        for (int counter = 0; counter < 1_000_000; counter++) {
            holder.setPlus();
        }
    }
}

public class LongMinusSetter implements Runnable {
    private String name;
    private LongHolder holder;

    public LongMinusSetter(String argName, LongHolder argHolder) {
        name = argName;
        holder = argHolder;
    }

    public void run() {
        System.out.println("[" + name "] started.");
        for (int counter = 0; counter < 1_000_000; counter++) {
            holder.setMinus();
        }
    }
}
```

- 리스트에서 하나의 longHolder instance를 LongPlusSetter와 LongMinuseSetter를 Thread에서 셋팅해준다.
- long은 32 bit가 상위, 하위가 나뉜다. thread에서 만약 long을 조작할 때 1로 상위를 조작하고서 다른 thread가 가로채어 -1로 하위를 조작한다면?
- 그럼 예측하지 못한 결과가 튀어나온다.
```
1의 경우
상위 32bit = 0x00000000
하위 32bit = 0x00000001

-1의 경우
상위 32bit = 0xffffffff
하위 32bit = 0xffffffff

longNum = 0x00000000_ffffffffL; 1의 상위 32 bit + -1의 하위 32bit //4294967295
longNum = 0xffffffff_00000001L; -1의 상위 32 bit + 1의 하위 32bit //-4294967295
```

- 이러한 예측불가능한 상황을 벗어나는 좋은 방법은?
  - 변수를 쓰지 않는 것이다.

- 아래 식은 변수를 Map<>으로 받는다.
- 이를 없애려면?
```java
public class BadPractice {
    private Map<String,String> map = new HashMap<>();

    public void doSomething(String value) {
        map.put("foo", value);
        doInternal();
    }

    private void doInternal() {
        System.out.println(map.get("foo"));
    }
}
```

- doSomething method에서만 map을 쓰므로 해당 영역에서 만들어준다.
```java
public class GoodPractice {
    public void doSomething(String value) {
        Map<String,String> map = new HashMap<>();
        map.put("foo", value);
        doInternal(map);
    }

    private void doInternal(Map<String,String> map) {
        System.out.println(map.get("foo"));
    }
}
```

```java
public class CallbackSample {
    public static void main(String... args) {
        final ExecutorService executor = Executors.newSingleThreadExecutor();
        AsyncProcess proc = new AsyncProcess(new AsyncCallback() {
            public void notify(String message) {
                System.out.println("callback message: " + message);
                executor.shutdown();
            }
        });
        executor.execute(proc);
        System.out.println("AsyncProcess is started.");

    }
}

public class AsyncProcess implements Runnable {
    private AsyncCallback callback;

    public AsyncProcess(AsyncCallback asyncCallback) {
        this.callback = asyncCallback;
    }

    public void run() {
        try {
            Thread.sleep(1000L);
        }
    }
}