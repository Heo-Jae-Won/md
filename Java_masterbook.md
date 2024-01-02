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

