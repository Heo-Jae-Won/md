## <span style="color:#802548">_generics란?_</span>

- 컴파일 시의 타입을 체크하는 기능이다. T, K, V, E 등이 있다.
- 기호의 종류만 다를 뿐 임의의 참조형 타입을 의미하는 것은 같다.

```java
class Box<T>{ //제너릭 클래스. T box라고 읽는다. T는 타입 매개변수, Box는 원시 타입이다.
    T item;

    void setItem(T item){
        this.item = item;
    }
    T getItem();
}

Box<String> b = new Box<String>;
b.setItem(new Object());
b.setItem("ABC");
String item = b.getItem; // generics로 Element의 type T를 String으로 정했기 때문에 (String) 형변환 필요 없음.
```

- 다시말해 아래 식은 아래와 같은 class를 선언한 것이다.

```java
Box<String b = new Box<String>;

class Box<String>{
    String item;

    void setItem(String item){
        this.item = item;
    }
    String getItem();
}
```

- generics는 compile time에 미리 type을 check하여 안정성을 높이기 위한 것이다.
    - static은 T가 들어오기 전에 class type이 명확하게 주어져있어야 한다. 가장 먼저 읽혀 올라가기 떄문이다.
    - generics를 배열로 선언하는 것도 불가능하다. 배열은 runtime에 구체화되기 때문이다.
    - new로 쓰는 것도 불가능하다. new는 runtime에 구체화되기 때문이다.

```java
class Box<T>{
    static T item; //error
    List<String>[] lists = new ArrayList[10];  //error
    T[] itemArr; //ok

    T[] toArray(){
        T[] temArr = new T[itemArr.length]; //error
    }
}
```

- generics type끼리는 늘 동일해야 한다.

```java
Box<Apple> b = new Box<Fruit> f; //error
```

- generics에는 특수한 extends 기능이 있다.
- 아래와 같이 T extends 특정 클래스라면, 특정클래스와 그 아래 자손 class만 generics의 type으로 선언 가능하다.

```java
class FruitBox<T extends Fruit>{
    .
    .
    .

}
```

- 반면에 T super Fruit이라면, 특정클래스와 그 위 조상 class만 generics의 type으로 선언 가능하다.

```java
class FruitBox<T super Fruit>{
    .
    .
    .

}
```

- interface를 implements하는 것일지라도 extends라는 용어를 사용해야만 한다.
- 아래는 &로 묶여서 Fruit과 자손, Eatable과 자손의 교집합의 경우에만 type으로 들어올 수 있음을 의미한다.

```java
class FruitBox<T extends Fruit & Eatable>{
    .
    .
    .

}
```

## <span style="color:#802548">_generics의 whildcard란?_</span>

- 와일드카드를 사용하면 한층 자유도가 높아진다. 물론 더 어려워지는 것도 맞다.
- whildcard를 사용한 Collections의 sort()를 살펴보자.

```java
static void sort(List<T> list, Comparator<? super T> comp)
```

- 위와 같은 <T>나 <? super T>는 사실 모든 것이 들어갈 수 있다는 의미다.
- 즉 Object와 동일하다. 다만 Object로 쓰지 않는 이유는 형변환 때문이다.
- 제너릭과 원시타입 간의 형변환은 반드시 제너릭의 type이 맞아야 한다.
- 다만 예외적으로 와일드카드만이 type이 달라도 허용된다.

```java
Box<String> box = (Box<String>)new Box<Object> // error
Box<String> box = (Box<String>)new Box<? extends Object> //ok
```

- 이제는 sort()를 살펴보자.

```java
static void sort(List<T> list, Comparator<? super T> comp)
```

- 왜 sort는 매개변수 Comparator의 type을 ? super T로 강제한걸까?
- 그 이유는 유지보수와 재활용성 때문이다.
- 만약 아래와 같이 와일드카드를 쓰지 않았다고 해보자.

```java
static void sort(List<T> list, Comparator<T> comp)
```

- 그렇게 된다면 List<Apple> 정렬하기 위해서는 Comparator<Apple>이 필요하다는 뜻이다.

```java
class AppleComp implements Comparator<Apple>{
    public int compare(Apple t1, Apple t2){
        return t2.weight - t1.weight;
    }
}

Collections.sort(appleBox.getList(), new AppleComp());
```

- 저기까진 괜찮은데 만약 Grape도 비교해야 한다면 아래와 같은 똑같은 내용의 comp를 또 만들어줘야 한다.

```java
class GrapeComp implements Comparator<Grape>{
    public int compare(Grape t1, Grape t2){
        return t2.weight - t1.weight;
    }
}

Collections.sort(grapeBox.getList(), new GrapeComp());
```

- 매번 비교대상이 늘어날 때마다 이렇게 comparator를 implements한 class를 만들기는 매우 불편하다.
- 따라서 ? super T로 generics를 설정하고, FruitComp라는 상위 class를 만든다.

```java
class FruitComp implements Comparator<Fruit>{
    public int compare(Fruit t1, Fruit t2){
        return t1.weight - t2.weight;
    }
}

Collections.sort(appleBox.getList(), new FruitComp());
Collections.sort(grapeBox.getList(), new FruitComp());
```

- 물론 Fruit class로 list를 만들고 FruitComp를 반들 수도 있다.
- 하지만 그 경우에는 아래와 같이 Apple만 모으고 싶을 때, Apple만 모으게 compile 시에서부터 강제가 불가능하다.
- 즉 유지보수에 좋지 않다. 따라서 Comparator class는 generics로 ? super T를 쓰는 것이다.
- 쓰지 않았다면 Fruit class만 타입 매개변수로 받는 list가 강제되어 apple만 따로, grape만 따로 모을 수가 없다.

```java
List<Apple> list = new0 ArrayList<>();
    list.add(new Grape("파란", 0));//Fruit이라서 Grape도 담음. 하지만 error가 아님.
    list.add(new Apple("빨간",1));//Fruit이라서 Apple도 담음. 하지만 error가 아님.
    Collections.sort(list, new FruitComp());
    for(Fruit f : list) {
        System.out.println(f.toString());
    }
```

- 매개변수에서 generics를 온갖 것을 다집어넣는 <?>를 준다고 해도, class가 늘 기준이다.
- class가 <T extends Fruit>이기 때문에 FruitBox는 Fruit과 자손 class만 들어감을 compiler가 인지한다.
- 그럼 참조변수 box가 FruitBox<? extends Object>라고 해도 그 element인 f는 늘 Fruit와 자손 class만을 갖게 된다.
- 따라서 Fruit f를 받을 수 있다.

```java
class FruitBox<T extends Fruit> extends Box<T>{}

static Juice makeJuice(FruitBox<? extends Object> box){
    String tmp = "";

    for(Fruit f: box.getList()) //ok
        tmp += f + " ";

    return new Juice(tmp);
}
```

## <span style="color:#802548">_generics method란?_</span>

- method의 return type 앞에 붙는 <>generics는 참조변수에 붙는 것과는 완전히 다른 의미다.
- 이 경우, 매개변수의 type을 지정하는 의미다.
- 아래 두 식은 동일한 식이다. 매개변수에 다 몰아넣냐, 매개변수의 type에 대한 설명을 앞으로 빼냐의 차이다.

```java
static Juice makeJuice(FruitBox<? extends Fruit> box)
static <T extends Fruit> Juice makeJuice(FruitBox<T> box)
```

- generics method를 호출할 때는 아래와 같다.

```java
FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
FruitBox<Apple> appleBox = new FruitBox<Apple>();

sysout(Juicer.<Fruit>makeJuice(fruitBox));
sysout(Juicer.<Apple>makeJuice(appleBox));
```

- generics의 type에 관한 설명인 <Fruit>은 대부분 생략할 수 있다.

```java
sysout(Juicer.makeJuice(fruitBox));
sysout(Juicer.makeJuice(appleBox));
```

- 그럼 이중 generic을 해석해보자.

```java
public static <T extends Comparable<? super T>> void sort(List<T> list)
//list는 반드시 T타입의 원소를 가져야 한다.
//T는 Comparable interface의 구현체 혹은 상속체여야 한다.
// Comparable은 T 타입과 조상을 대상으로 한다
```

- 아래와 같이 Optional에도 static에 <T>가 붙어있다.
- 그런데 여기는 parameter가 없기 때문에 parameter generics는 아니다.

```java
public static<T> Optional<T> empty() {
    @SuppressWarnings("unchecked")
    Optional<T> t = (Optional<T>) EMPTY;
    return t;
}
Optional<String> optVal = Optional.<String>empty();
Optional<String> optVal = Optional.empty(); //위에서 말했듯 generics의 type에관한 설명인 <String>은 생략 가능.
```

## <span style="color:#802548">_enum이란?_</span>

- enum을 쓰면 아래 class를 간단하게 만들어 줄 수 있다.
- Java의 enum은 type까지 관리하기 때문에 타입에 안전하다.

```java
class Card{
    static final int CLOVER = 0;
    static final int HEART = 1;
    static final int DIAMOND = 2;
    static final int SPADE = 3;

static final int TWO = 0;
static final int THREE = 1;
static final int FOUR = 2;

final int kind;
final int num;
}

class Card{
    enum Kind { CLOVER, HEART, DIAMOND, SPADE}
    enum value { TWO, THREE, FOUR}

    final Kind kind;
    final Value value;
}
```

```java
enum Direction{
    EAST(1), SOUTH(5), WEST(-1), NORTH(10);

    private final int value;

    Direction(int value){ // enum의 생성자는 암묵적으로 private
        this.value = value;
    }

    public getValue(){
        return value;
    }
}

Dirction d = new Direction(); //error. 생성자는 private
```

- enum에 새로운 member를 추가하는 것도 가능하다.

```java
enum Direction{
    EAST(1, ">"), SOUTH(5, "V"), WEST(-1,"<"), NORTH(10,"^"); //열거형 상수 하나하나가 모두 static 상수객체

    private final int value;
    private final String symbol;

    Direction(int value, String symbol){ // enum의 생성자는 암묵적으로 private
        this.value = value;
        this.symbol = symbol;
    }

    public getValue(){
        return value;
    }
}
```

- enum에 새로운 abstract method를 추가하는 것도 가능하다.
- 아래와 같은 enum class가 있다. 여기에 method를 추가해주자.

```java
enum Transportation{
    BUS(100), TRAIN(150), SHIP(100), AIRPLANE(300);

    private final int BASIC_FARE;

    private Transportation(int basicFare){
        BASIC_FARE = basicFare;
    }

    int fare(){
        return BASIC_FARE;
    }
}
```

- 거리에 따른 요금 산정방식이 운송수단마다 다르기 때문에 abstract method를 선언해주고, 이를 열거형 상수가 구현하게 한다.

```java
enum Transportation{
    BUS(100){
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    TRAIN(150){
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    SHIP(100){
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    AIRPLANE(300){
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    abstract int fare(int distance);

    protected final int BASIC_FARE;

    Transportation(int basicFare){
        BASIC_FARE = basicFare;
    }

    public int getBasicFare(){
        return BASIC_FARE;
    }
}
```

```java
System.out.println("bus fare =  " + Transportation.BUS.fare(100)); //100 * 100
System.out.println("train fare =  " + Transportation.TRAIN.fare(100)); //150 * 100
System.out.println("ship fare =  " + Transportation.SHIP.fare(100)); // 100 * 100
System.out.println("airplane fare =  " + Transportation.AIRPLANE.fare(100)); //300 * 100
```

- 아래와 같이 switch - case문에서도 사용이 가능하다.
-

```java
switch (Transportation) {
    case BUS: //...
    case TRAIN: //...
    case SHIP: //...
    case AIRPLANE:
}


```

- 내부 구현원리는 아래와 같다.

```java
abstract class MyEnum<T extends MyEnum<T>> implements Comparable<T>{
    static int id = 0; //enum객체에 붙일 일련번호
    int ordinal = 0;
    String name = "";

    public int ordinal(){
        return ordinal;
    }

    MyEnum(String name){
        this.name = name;
        ordinal = id++; //this.안붙여도 되나봄. 매개변수로 int ordinal이 안들어와서 this.ordinal과 동일한 의미인듯.
    }

    public int compareTo(T t){
        return ordinal - t.ordinal(); //T의 type에 ordinal이 있나 확인을 안해도 되는 이유는 generics가 T extends MyEnum<T>이기 때문.
                                      //만약 abstract class MyEnum<T>였다면 ordinal - t.ordinal()과 같이 간단하게 처리 불가능함. if(t instanceof MYEnum)이 필요함.
    }
}

abstract class MyTransportation extends MyEnum{
    static final MyTransportation BUS = new MyTransportation("BUS", 100){ //enum은 compile 타임에 값이 정해져있는 상수이므로 static final로 구현됨
        @Override //MyTransportation 객체를 생성했으면, abstract method인 fare를 반드시 구현해야 함.
        int fare(int distance){
            return distance * BASIC_FARE; // BASIC_FARE는 객체를 생성할 때 반드시 같이 넣어줘야 함.
        }
    };

     static final MyTransportation SHIP = new MyTransportation("SHIP", 100){
        @Override
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    };

     static final MyTransportation TRAIN = new MyTransportation("TRAIN", 150){
        @Override
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    };

     static final MyTransportation AIRPLANE = new MyTransportation("AIRPLANE", 300){
        @Override
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    };

    abstract int fare(int distance);

    protected final int BASIC_FARE;

    private MyTransportation(String name, int basicFare){
        super(name);
        BASIC_FARE = basicFare;
    }

    public String name(){
        return name;
    }

    public String toString(){
        return name;
    }
}
```

- 아래 ordinal은 enum이 선언된 순서를 나타내며, 사실 개발하면서 쓰면 안된다.
- ordianl()을 기반으로 하게된다면 상수의 순서를 변경하거나, 중간에 끼워넣기라도 하게 되면 모두 문제가 발생한다. ordinal 값이 바뀌기 떄문이다.

```java
class EnumEx4{
    MyTransportation t1 = MyTransportation.BUS;
    MyTransportation t2 = MyTransportation.BUS;
    MyTransportation t3 = MyTransportation.TRAIN;
    MyTransportation t4 = MyTransportation.SHIP;
    MyTransportation t5 = MyTransportation.AIRPLANE;

System.out.println("t1=%s, %d%n  " + t1.name(), t1.ordinal()); //t1=BUS, 0
System.out.println("t2=%s, %d%n  " + t2.name(), t2.ordinal()); //
System.out.println("t3=%s, %d%n  " + t3.name(), t3.ordinal());
System.out.println("t4=%s, %d%n  " + t4.name(), t4.ordinal());
System.out.println("t5=%s, %d%n  " + t5.name(), t5.ordinal());
}
```

- 메타 annotation
- annotation을 위한 annotation이다.
- annotation의 target, 유지기간을 지정하는 데 사용한다.

```java
@Target({FIELD, TYPE, TYPE_USE})
public @interface MyAnnotation{}

@MyAnnotation
class MyClass{} // TYPE_USE

@MyAnnotation
int i; //FIELD

@MyAnnnotation
MyClass mc; // TYPE
```

```java
@Documented //javadoc에 포함
@Retention(RetentionPolicy.RUNTIME) //runtime 시에도 annotation에 대한 정보가 존재
@Target(ElementType.TYPE) //
public @interface FunctionalInterface{}
```
