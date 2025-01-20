## <span style="color:#802548">_function package_</span>
- Supplier는 매개변수가 없고, 반환 값만 있다.

```java
T get();
```

- Consumer는 매개변수만 있고, 반환 값이 없다.

```java
void accept(T t);
```

- Function은 하나의 매개변수를 받으며, 반환 값이 있다.

```java
R apply(T t);
```

- Predicate는 매개변수를 하나 받으며, boolean 변수만 반환한다.

```java
boolean test(T t);
```

## <span style="color:#802548">_일반 식에서 람다식으로_</span>
- 람다식이란 반환값이 없고, 문장도 아니기 떄문에, 세미콜론을 쓰지 않는다.

```java
int max(int a, int b){
    return a > b? a : b;
}
```

- 위에서 return type과 method명을 지워준다.

```java
(int a, int b) -> {
    return a > b? a : b;
}
```

- 여기서 return도 지워준다. return을 지우면 문장이 아닌 식이므로 세미콜론;도 지운다.

```java
(int a, int b) -> {
    a > b? a : b
}
```

- 식이 하나라면 중괄호{}도 생략가능하다.

```java
(int a, int b) -> a > b? a: b
```

- 대부분의 경우 매개변수의 type은 생략가능하다.

```java
(a, b) -> a > b? a : b;
```

- 처음 식은 아래와 같이 변한다.

```java
int max(int a, int b){
    return a > b ? a: b;
}

(a,b) -> a > b ? a : b;
```

- lambda식안에 식이 두 개라면 아래와 같이 쓸 수 있다.

```java
() -> {System.out.println("MyFunction()"); System.out.println("bbb");};
```
## <span style="color:#802548">_익명객체로 구현한 람다식_</span>
- 사실 Java에서는 lambda식은 interface를 구현한 익명 클래스의 객체로 구현한다.
- 따라서 아래와 같은 interface가 필요하다.

```java
interface MyFunction{
    int max(int a, int b)
}
```

- 이 interface로 익명클래스의 객체를 만드는 과정을 간소화하는 게 자바의 람다식이다.

```java
MyFunction f = new MyFunction(){
    @Override
    public int max(int a, int b){//무조건 public 접근제어자.
        return a > b ? a : b;
    }
}
int big = f.max(5,3);


MyFunction f = (a,b) -> a > b ? a : b;
int big = f.max(5,3);
```

- 이렇게 람다식으로 활용하고 싶은 interface는 @FunctionalInterface를 붙여준다.
- 람다식으로 활용하고 싶다면 instance method는 오직 1개만 존재해야 한다. static, default method의 갯수는 상관없다.

```java
@FunctionalInterface
interface MyFunction{
    int max(int a, int b)
}
```

- 람다식을 활용한 실제 사례는 아래와 같다.

```java
Collections.sort(list, new Comparator<String>(){
    public int compare(String s1, String s2){
        return s2.compareTo(s1);
    }
});

Collections.sort(list, (s1, s2) -> s2.compareTo(s1));
```

## <span style="color:#802548">_매개변수로 method를 받는 람다식_</span>
- 람다식을 활용하면 method를 매개변수로 받을 수도 있다.

```java
@FunctionalInterface
interface MyFunction{
    void myMethod();
}

void aMethod(MyFunction f){
    f.myMethod();
}
```

- 가능한 이유는 그것이 익명객체를 주고받는 것이기 때문이다.
- 실제로 주고받는 게 function 자체인 javascript의 1급함수와 같은 속성을 가지게 된 것은 아니다.

```java
MyFunction f = () -> System.out.println("myMethod()");
aMethod(f);

MyFunction f = new MyFunction(){
    @Override
    public void myMethod(){
        System.out.println("myMethod()");
    }
}
aMethod(f);
```

- 참조변수를 아예 람다식으로 넣어버리는 것도 가능하다.

```java
aMethod( () -> System.out.println("myMethod()") );
```

- method의 반환 타입이 함수형 인터페이스라면, 람다식을 가리키는 참조변수와 람다식 모두 반환이 가능하다.
- 위에서는 void를 return type으로 했지만 이번에는 MyFunction을 return type으로 해보자.

```java
@FunctionalInterface
interface MyFunction{
    void myMethod();
}

MyFunction getSth(){
    MyFunction f = () ->{System.out.println("lambda")};
    return f;
}

MyFunction getSth(){
    return () -> {System.out.println("lambda")};
}
```


## <span style="color:#802548">_stream을 생성하는 여러가지 방법_</span>
- Stream을 생성하는 여러가지 방법이 있다.

```java
String [] strArr = {"aaa", "ddd", "ccc"}; //배열
Stream<String> strStream = Arrays.stream(strArr);
Stream<String> strStream = Stream.of(strArr);

List<String> strList = Arrays.asList(strArr); //리스트
Stream<String> strStream = strList.stream();
Stream<List<String>> strStream = Stream.of(strList);
```

- 기본형은 따로 생성한다.

```java
IntStream intStream = IntStream.range(1,5);
IntStream intStream = IntStream.rangeClosed(1,5);
```

## <span style="color:#802548">_중간연산과 최종연산_</span>
- 스트림의 연산이 중간연산일 경우, 연산 결과가 stream이다. 스트림에 연속해서 중간 연산을 실행할 수 있다.
- 반면에 최종 연산의 경우, 연산 결과가 stream이 아니다. 스트림의 요소를 소모하기 때문에 최종연산은 단 한 번만 가능하다.
- 그런데 사실 최종 연산을 먼저 하고, 중간 연산이 실행되는 것이다.
- 아래의 식을 보면, List interface로 stream을 생성하고, filter를 하는 것처럼 보인다. 그러나 실상은 먼저 list를 만든 뒤에 filtering이 진행된다.

```java
List<Event> filteredItems=items.stream().filter(element -> {
			return !element.getId().equals(calendarEventReq.getEventId());
	    }).collect(Collectors.toList());
```

- 말했다시피 stream을 최종연산을 실행하면 해당 stream은 더이상 사용할 수 없다.

```java
Stream<Event> itemStream=items.stream();
List<Event> items = itemStream.filter(element -> {
			return !element.getId().equals(calendarEventReq.getEventId());
	    }).collect(Collectors.toList());
int count = itemStream.count(); // error
```

- 또한 하나의 stream을 여러 stream으로 나눠서 처리하는 것은 불가능하다.
- 같은 stream으로 객체를 두개로 나눠 중간연산을 각개로 돌리는 것은 불가능하는 것이다.
- 따라서 하나는 주석처리해야 한다. 아니면 stream has already been operated upon or closed 오류가 뜬다. 참고로 주소값 기준이다.

```java
 Stream<String[]> strArrStream = Stream.of(
		            new String[]{"abc","def","jkl"},
		            new String[]{"ABC","DEF","JKL"}
		        );

//Stream<Stream<String>> strStrmStrm = strArrStream.map(Arrays::stream); 
Stream<String> strStrm = strArrStream.flatMap(Arrays::stream); 
```

- 따라서 인간의 눈에 보는 내용물이 같더라도, 주소값이 다른 객체라면 다른 stream으로 취급된다.
- 즉 개별 stream의 연산으로 취급돼기 때문에 stream has already been operated upon or closed 오류가 나지 않는다.


- 중간연산으로 주로 사용되는 것들은 아래와 같다.

```java
map()
sorted()
filter()
```

- 최종연산으로 주로 사용되는 것들은 아래와 같다.

```java
forEach()
reduce()
collect()
```

- map의 사례는 아래와 같다. 

```java
Collection<? extends GrantedAuthority> authorities = Arrays.stream(claims.get("auth").toString().split(","))
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());
```

- filter의 사례는 아래와 같다.

```java
Stream<Event> itemStream=items.stream();
List<Event> items = itemStream.filter(element -> {
			return !element.getId().equals(calendarEventReq.getEventId());
	    }).collect(Collectors.toList());
```

- 만약에 Stream에 값이 없다면 그냥 빈 Stream으로 반환하는 것도 좋은 선택이다.

```java
Stream emptyStream = Stream.empty();
long count = emptyStream.count(); // 0
```

- 두 스트림의 연결은 concat으로 한다.

```java
String[] str1 = {"123","456","789"};
String[] str2 = {"ABC","abc","DEF"};

Stream<String> strs1 = Stream.of(str1);
Stream<String> strs2 = Stream.of(str2);
Stream<String> strs3 = Stream.concat(str1, str2);
```

## <span style="color:#802548">_중간연산- flatMap_</span>

- map이외에 flatMap이라는 method도 있다.
- map과 다르게 flat이 붙어있는데, 이는 차원을 평평하게 하는 것이다.

```java
Stream<String[]> arrayStream = Stream.of(
    new String[]{"abc","def","jkl"},
    new String[]{"ABC","DEF","JKL"}
);

Stream<Stream<String>> doubleStream = arrayStream.map(Arrays::stream); // Stream 안의 Stream이 존재. 이 상태에서 forEach해도 값이 제대로 안 뽑힘.
Stream<String> singleStream = arrayStream.flatMap(Arrays::stream);  //Stream 안의 Stream을 전부 합쳐서 동일한 차원에 설정

strStrm.map(String::toLowerCase) 
        .distinct()
        .sorted()
        .forEach(System.out::println);
System.out.println();
```

- flatMap의 단순 예시다.
\
```java
 Stream<String[]> arrayMapStream = Stream.of(
		            new String[]{"abc","def","jkl"},
		            new String[]{"ABC","DEF","JKL"}
		        );
		 
Stream<String[]> arrayFlatMapStream = Stream.of(
        new String[]{"abc","def","jkl"},
        new String[]{"ABC","DEF","JKL"}
);

Stream<Stream<String>> arrayStream = arrayMapStream.map(Arrays::stream); // Stream 안의 Stream이 존재. 이 상태에서 forEach해도 값이 제대로 안 뽑힘.
Stream<String> strStrm = arrayFlatMapStream.flatMap(Arrays::stream);  //Stream 안의 Stream을 전부 합쳐서 동일한 차원에 설정
```

- 평탄화의 과정은 아래와 같다. 두 배열을 새로운 stream 객체로 만들어서 돌리게 된다.
- 대신 Stream을 새로 만드는 만큼, GC가 수거해야 할 객체가 늘어나게 된다.
- 그저 값을 찍는 것이라면 새로운 stream을 만들지 않고 그대로 foreach를 돌리는 편이 나을 것이다.

```java
Stream.of(
    new String[]{"abc","def","jkl"}
    , new String[]{"abc1","def2","jkl3"}
).flatMap(Arrays::stream).forEach((element)->System.out.println(element));

Stream.of("abc","def","jkl","abc1","def2","jkl3").forEach((element)->System.out.println(element));

Stream.of(
            new String[]{"abc","def","jkl"},
            new String[]{"ABC","DEF","JKL"}
        ).forEach((arryas)-> {
            for(String element : arryas) {
                System.out.println(element);
            }
        });
```

- flatMap을 해 평탄화 시킨 뒤 map을 통해 변화시키는 것도 가능하다.
- flatMap의 역할은, 배열 등을 평탄화시켜서 만드는 역할이다. DTO 등으로 mapping하는 것은 그 후의 map()에서 실행한다.

```java
String[] lineStream = {
    "Believe or not It is true",
    "Do or do not There is no try",
};

Stream<String> lineStream = Arrays.stream(lineArr);
lineStream.flatMap(line -> stream.of(line.split(" +"))) //one or more space characters.
//"Believe, or, not, It, is, true" 
            .map(String::toLowerCase)
            .distinct()
            .sorted()
            .forEach(System.out::println);
System.out.println();
```

## <span style="color:#802548">_Optional 객체_</span>

- Optional 객체는 null처리를 안해도 되게 해줘서 편하다.
- of는 nullpointerException이 발생하지만, ofNullable은 NPE가 발생하지 않는다.


```java
String str = "abc";
Optional<String> optVal = Optional.of(str); 
Optional<String> optVal = Optional.ofNullable(str); // str이 null일 가능성이 있다면 ofNullable로!
```

- Optional 객체를 초기화할 때는 아래와 같이 empty()를 사용한다

```java
String str = "abc";
Optional<String> optVal = null; //이거 아니다
Optional<String> optVal = Optional.<String>empty(); //<String>empty()로 적는 이유는 return type이 Optional<T>이기 때문.

Optional<Integer> optVal = Optional.<Integer>empty();
Optional<Integer> optVal = Optional.empty(); //generics type은 생략 가능.
```

- Optional 객체의 값을 가져오는 데는 보통 2가지를 사용한다.

```java
String str1 = optVal.get(); //NPE 발생 가능. 잘 안 씀.
String str2 = optVal.orElseGet(String::new); //optVal이 null이면 새로운 String으로 대체
String str3 = optVal.orElseThrow(NullPointerException); //optVal이 null이면 원하는 exception을 던짐.
```

- 여기서 orElseGet이나 orElseThrow나 모두 함수형 인터페이스, 즉 람다식을 사용해야 한다.

```java
Optional<String> abc = Optional.ofNullable(str);
String str1 = abc.orElseGet("bbb"); //불가능. 여기는 반드시 람다식이 들어가야 함. 람다식이 ()->new String()일수도 있고, 매써드 참조형태의 String::new일 수도 있음.
String optionalStr = abc.orElseGet(()->"bbb"); // String optionalStr = abc.orElseGet(()->new String("bbb"));과 동일.
System.out.println(optionalStr);
System.out.println(str1 == optionalStr); // false
```

- 참고로 람다식 내에서 만들어지는 String 변수는 String constant pool에 들어가는 게 아니라 heap으로 들어간다.
- "bbb"로 만들어도 new String("bbb")로 만드는 것과 동일하며 늘 새로운 메모리 주소를 점유하게 된다.
- 따라서 str1과 OptionalStr의 value가 같아도 둘은 서로 다른 객체다.

```java
String str = null;
String str1 = "bbb";
Optional<String> abc = Optional.ofNullable(str);
String optionalStr = abc.orElseGet(()->"bbb"); // String optionalStr = abc.orElseGet(()->new String("bbb"));과 동일.
System.out.println(optionalStr);
System.out.println(str1 == optionalStr); // false
System.out.println(str1.equals(optionalStr)); //true
```

- orElseGet()은 공백도 value로 인식한다.

```java
String str = "";
String optionalStr = abc.orElseGet(()->"bbb");
System.out.println(optionalStr); //공백. 공백도 값이 있는 것이기 때문에 null일 때처럼 "bbb"를 return하지 않는다.
```

- Optional 객체의 값이 있는지 확인하는 방법은 아래와 같다.

```java
if(str!=null){
    System.out.println(str);
} //기존

if(Optional.ofNullalbe(str).isPresent()){
    System.out.println(str);
} //Optional 객체를 활용한 형태

Optional.ofNullable(str).ifPresent(System.out::println); // Optional 객체와 lambda식을 활용한 형태. 만약 값이 있다면 sysout을, 값이 없다면 
```

- stream의 연산에도 아래와 같이 Optional 객체를 사용할 수 있다.
- 만약에 stuStream에 값이 없으면 빈 Optional 객체를 반환한다. 
- 빈 Optional 객체는 안에 null을 저장해두고 있다.

```java
Stream<Student> stuStream = Stream.empty();
boolean noFailed = stuStream.anyMatch(s->s.getTotalScore() <= 100);
Optional<Student> stu = stuStream.filter(s->s.getTotalScore() <= 100).findFirst(); 
```

- 그냥 stream class를 이용한 경우는 아래와 같이 int type으로 타입을 지정한다. 

```java
int count = intStream.reduce(0,(a,b)->a+1);
int sum = intStream.reduce(0,(a,b)->a+b);
int max = intStream.reduce(Integer.MIN_VALUE,(a,b)->a>b ? a: b);
int min = intStream.reduce(Integer.MAX_VALUE, (a,b)->a<b ? a: b);
```

- IntStream을 이용한 경우는 아래와 같이 OptionalInt로 타입을 지정한다.
- 기본형으로 바꾸고 싶다면 아래와 같이 getAsInt()로 바꿔주면 된다.


```java
OptionalInt max = intStream.reduce((a,b)->a > b ? a : b);
OptionalInt min = intStream.reduce((a,b)->a < b ? a : b);
OptionalInt max = intStream.reduce(Integer::max);
OptionalInt min = intStream.reduce(Integer::min);
int maxValue = max.getAsInt();
```

## <span style="color:#802548">_collect_</span>
- collect()는 Stream에서 가장 많이 쓰이는 method다.

```java
List<String> names = stuStream.map(Student::getName).collect(Collectors.toList());
ArrayList<String> list = names.stream().collect(Collectors.toCollection(ArrayList::new));
Map<String,Person> map = PersonStream.collect(Collections.toMap(p->p.getRegId(), p->P));
```

- stream을 두 개 그룹으로 나눌 때는 groupingBy()로 쓴다.
- 그 이상의 그룹으로 나눌 때는 partioningBy();를 사용한다.

## <span style="color:#802548">_grouping, partioningby_</span>

- groupingBy와 partioningBy를 쓰기 위한 재료 class다.

```java
public class Student{
    String name;
    boolean isMale;
    int hak;
    int ban;
    int score;

    Student(String name, boolean isMale, int hak, int ban, int score){
        this.name = name;
        this.isMale = isMale;
        this.hak = hak;
        this.ban = ban;
        this.score = score;
    }

    String getName(){
        return name;
    }

    boolean isMale(){
        return isMale;
    }

    int getHak(){
        return hak;
    }

    int getBan(){
        return ban;
    }

    int getScore(){
        return score;
    }

    public String toString(){
        return String.format("[%s, %s, %d학년 %d반, %3d점]", name, isMale ? "남" : "여", hak, ban, score);
    }

    enum Level{HIGH, MID, LOW}
}
```

- class로 stream을 만들었다.

```java
Stream<Student> stuStream = Stream.of(
    new Student("나자바",true,1,1,300),
    new Student("김지미",false,1,1,250),
    new Student("김자바",true,1,1,200),
    new Student("이지미",false,1,2,150),
    new Student("남자바",true,1,2,100),
    new Student("안지미",false,1,2,50),
    new Student("황지미",false,1,3,100),
    new Student("강지미",false,1,3,150),
    new Student("이자바",true,1,3,200),

    new Student("나자바",true,2,1,300),
    new Student("김지미",false,2,1,250),
    new Student("김자바",true,2,1,200),
    new Student("이지미",false,2,2,150),
    new Student("남자바",true,2,2,100),
    new Student("안지미",false,2,2,50),
    new Student("황지미",false,2,3,100),
    new Student("강지미",false,2,3,150),
    new Student("이자바",true,2,3,200)
);
```

- 해당 stream으로 구성해서 값을 분류하여 collection에 넣는다.

```java
Map<Boolean, List<Student>> stuBySex = stuStream.collect(partitioningBy(Student::isMale));
List<Student> maleStudent = stuBySex.get(true);    //Map에서 남성 목록을 얻어온다.
List<Student> femaleStudent = stuBySex.get(false); //Map에서 여성 목록을 얻어온다.

Map<Boolean,Long> stuNumBySex = stuStream.collect(partitioningBy(Student::isMale, counting()));
System.out.println("남학생 수: " + stuNumBySex.get(true)); //8
System.out.println("여학생 수: " + stuNumBySex.get(false)); //10

Map<Boolean,Optional<Student>> topScoreBySex = stuStream.collect(partitioningBy(Student::isMale, maxBy(comparingInt(Student::getScore))));
System.out.println("남학생 1등: " + topScoreBySex.get(true)); //Optional[[나자바,남,1,1,300]]. maxBy()는 Optional<Student>이 반환타입
System.out.println("여학생 1등: " + topScoreBySex.get(false));//Optional[[김지미,여,1,1,250]]

Map<Boolean,Optional<Student>> topScoreBySex = stuStream.collect(partitioningBy(Student::isMale, collectingAndThen(maxBy(comparingInt(Student::getScore), Optional::get))));
System.out.println("남학생 1등: " + topScoreBySex.get(true)); //남학생 1등: [나자바, 남, 1, 1, 300]. collectingAndThen을 추가해주면 Student가 반환타입.
System.out.println("여학생 1등: " + topScoreBySex.get(false));//여학생 1등: [김지미, 여, 1, 1, 250]

Map<Boolean, Map<Boolean,List<Student>>> failedStuBySex = stuStream.collect(partitioningBy(Student::isMale, partitioningBy(s -> s.getScore() < 150))); //partitioningBy안에서 또 partitioningBy를 호출해서 조건을 두개를 &&로 연결
List<Student> failedMaleStu = failedStuBySex.get(true).get(true); // 남성 중 성적이 150점 이하인 사람
List<Student> failedMaleStu = failedStuBySex.get(false).get(true); //여성 중 성적이 150점 이하인 사람

Map<Integer, List<Student>> stuByBan = stuStream.collect(groupingBy(Student::getBan, /*toList()*/)); //toList()는 생략가능
Map<Integer, HashSet<Student>> stuByHak = stuStream.collect(groupingBy(Student::getHak, toCollection(HashSet::new)));
```


- 좀 더 복잡한 예시로 점수에 따라 레벨을 나눠 그룹핑한다.

```java
Map<Student.Level, Long> stuByLevel = stuStream.collect(groupingBy(s->{
    if(s.getScore() >= 200)
        return Student.Level.HIGH;
    else if(s.getScore() >= 100)
        return Student.Level.MID;
    else
        return Student.Level.LOW;
},counting())) //[MID] - 8명, [HIGH] - 8명, [LOW] - 2명
```

- counting 하지 않고, Set으로 바꾼다.

```java
Map<Integer, Map<Integer,Set<Student.Level>>> stuByHakAndBan = stuStream.collect(GroupingBy(Student::getHak,groupingBy(Student::getBan,mapping(s->{
    if(s.getScore() >= 200)
        return Student.Level.HIGH;
    else if(s.getScore() >= 100)
        return Student.Level.MID;
    else
        return Student.Level.LOW;
}, toSet()))))
```


- 좀 더 복잡한 예시로 학년 별 그룹화 -> 반 별 그룹화를 진행한다.

```java
Map<Integer,Map<Integer,List<Student>>> stuByHakAndBan = stuStream.collect(
				Collectors.groupingBy(Student::getHak, 
						Collectors.groupingBy(Student::getBan)));
```

- maxBy의 return typ이 Optional이다.
- Optional(Student)가 아니라 그냥 Student로 바꿔주기 위해 추가해준다.
    - collectingAndThen
    - Optional::get

```java
Map<Integer, Map<Integer,Student>> topStuByHakAndBan = stuStream.collect(
    groupingBy(Student::getHak, groupingBy(
        Student::getBan, collectingAndThen(
            maxBy(comparingInt(Student::getScore),Optional::get)
            )
        )
    )
) //각 반의 1등을 출력
```

- 위는 static import를 실행했을 때의 모습이다.
- 그냥 import로는 class명을 전부 기입해줘야 한다.


```java
Map<Integer,Map<Integer,List<Student>>> stuByHakAndBan = stuStream.collect(
    Collectors.groupingBy(Student::getHak, 
        Collectors.groupingBy(Student::getBan)
    )
); //학년별로 그룹화 후 반별로 다시 그룹화

for(Map<Integer,List<Student>> hak : stuByHakAndBan.values()){
for(List<Student> ban : hak.values()){
    System.out.println();
    for(Student s : ban){
        System.out.println(s);
    }
} 
} 
// [나자바, 남, 1학년 1반, 300점]
// [김지미, 여, 1학년 1반, 250점]
// [김자바, 남, 1학년 1반, 200점]

// [이지미, 여, 1학년 2반, 150점]
// [남자바, 남, 1학년 2반, 100점]
// [안지미, 여, 1학년 2반,  50점]

// [황지미, 여, 1학년 3반, 100점]
// [강지미, 여, 1학년 3반, 150점]
// [이자바, 남, 1학년 3반, 200점]

// [나자바, 남, 2학년 1반, 300점]
// [김지미, 여, 2학년 1반, 250점]
// [김자바, 남, 2학년 1반, 200점]

// [이지미, 여, 2학년 2반, 150점]
// [남자바, 남, 2학년 2반, 100점]
// [안지미, 여, 2학년 2반,  50점]

// [황지미, 여, 2학년 3반, 100점]
// [강지미, 여, 2학년 3반, 150점]
// [이자바, 남, 2학년 3반, 200점]


Map<Integer, Map<Integer,Student>> topStuByHakAndBan = stuStream.collect(
    Collectors.groupingBy(Student::getHak, 
            Collectors.groupingBy(Student::getBan, 
                    Collectors.collectingAndThen(
                            Collectors.maxBy(Comparator.comparingInt(Student::getScore)),
                                                                        Optional::get
                            )
                    )
            )
    ); //각 반의 1등을 출력


for(Map<Integer,Student> ban: topStuByHakAndBan.values())
for(Student s : ban.values())
    System.out.println(s);
    // [나자바, 남, 1학년 1반, 300점]
    // [이지미, 여, 1학년 2반, 150점]
    // [이자바, 남, 1학년 3반, 200점]
    // [나자바, 남, 2학년 1반, 300점]
    // [이지미, 여, 2학년 2반, 150점]
    // [이자바, 남, 2학년 3반, 200점]


Map<Integer, Map<Integer,Set<Student.Level>>> stuByHakAndBan = stuStream.collect(
    Collectors.groupingBy(Student::getHak,
        Collectors.groupingBy(Student::getBan,
            Collectors.mapping(s->{
                if(s.getScore() >= 200)
                    return Student.Level.HIGH;
                else if(s.getScore() >= 100)
                    return Student.Level.MID;
                else
                    return Student.Level.LOW;
                    }, Collectors.toSet()
            )
        )
    )
);

Set<Integer> keySet2 = stuByHakAndBan.keySet();

for(Integer key : keySet2){
    System.out.println("[" + key + "]" + stuByHakAndBan.get(key)); //key안에 담긴 요소는 map
}
// [1]{1=[HIGH], 2=[MID, LOW], 3=[MID, HIGH]}. 1학년 1반은 HIGH, 2반은 MID,LOW, 3반은 MID,HIGH
// [2]{1=[HIGH], 2=[MID, LOW], 3=[MID, HIGH]}
```


- 마지막 예시의 경우 아래와 같이 String을 key로 사용하는 Set으로 고칠 수 있다.


```java
Map<String,Set<Student.Level>> stuByScoreGroup = stuStream.collect(
                                                                Collectors.groupingBy(
                                                                    s->s.getHak() + "-" + s.getBan(), // 1-1, 1-2, 1-3... 등을 key로 아래와 같이 mapping됨.
                                                                    Collectors.mapping(s -> {
                                                                        if(s.getScore() >= 200)
                                                                            return Student.Level.HIGH;
                                                                        else if(s.getScore() >= 100)
                                                                            return Student.Level.MID;
                                                                        else
                                                                            return Student.Level.LOW;
                                                                    }, Collectors.toSet())
                                                                )
                                                        );

Set<String> keySet2 = stuByScoreGroup.keySet();
for(String key: keySet2){
    System.out.println("[" + key + "]" + stuByScoreGroup.get(key)); //key안에 담긴 요소는 Set collection
}
// [1-1][HIGH]. 1학년 1반은 HIGH
// [2-1][HIGH]
// [1-2][MID, LOW]
// [2-2][MID, LOW]
// [1-3][MID, HIGH]
// [2-3][MID, HIGH]
```















