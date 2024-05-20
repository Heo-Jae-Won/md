## <span style="color:#802548">_class 초기화_</span>

- 클래스 정보가 필요할 때, 클래스는 메모리에 로딩된다.
  - 참조변수의 선언
  - 객체의 생성
- class는 여러 type의 데이터를 담으면서 기능도 갖춘 변수의 최종판이다.

```java
Card c = new Card();

1.     연산자 new ㅡ> 메모리에 card class instance 생성.

2.     생성자 Card() 호출되어 초기화

3.     연산자 new의 결과로 생성된 card instance의 주소가 반환되어 참조변수에 저장한다.
```

- 여기서 this는 instance 자신을 의미한다.
- 그런 이유로 static에서 this는 사용 불가능하다.
- this는 생겨난 instance를 참조하는데, static에서는 Instance가 없을 때도 작동해야 하기 때문이다.

```java
Car(String color, String gearType, int door){
	this.color = color;
	this.gearType = gearType;
	this.door = door;
}
```

- 한 생성자에서 다른 생성자를 호출하면 무조건 첫줄에 해야 한다.
- 초기화 작업을 생성자 이전에 했다면, 무효화되어버리기 때문이다.

```java
Car(){
	this(“white”,”auto”,4); // Car("white","auto",4);와 동일
}
```

- 인스턴스 변수는 선언만해도 기본값이 있어 초기화가 된다.
  - int x는 선언만 하고 할당을 안 해도 초기화값이 부여된다.
  - int 형의 초기값은 0이다.
- 지역변수는 기본값이 없다. 반드시 초기화가 필요하다.

```java
Class initTest(){
	int x;
	int y = x; //0
	void method(){
		int i;
		int j = i; // error. 지역변수는 기본값이 있어야..
	}
}
```

- 초기화 순서는 static variable(class variable)이냐 instance variable이냐에 따라 다르다.
  - static의 경우, 명시적 초기화 -> 클래스 초기화 블록
  - instance의 경우, 명시적 초기화 -> 인스턴스 초기화 블록 -> 생성자

```java
Class BlockTest{
    /**명시적 초기화 */
    static int cv = 1;
    int iv = 1 ;

    /**static 초기화. instance 생성의 경우 타지 않는다. */
    static{
        cv = 2;//클래스 초기화 블록
    }

    /**instance 초기화. static의 경우 타지 않는다. */
    {
        iv = 2;//인스탠스 초기화 블록
    }

    /** 생성자 초기화. static의 경우 타지 않는다. */
    public BlockTest(){
        iv =3;
    }
}
```

- 아래는 초기화 블록을 사용하는 예시다. 아래와 같이 Product를 생성하게 되면 SerialNo가 계속 1씩 증가하는 Product가 찍혀나오게 된다.

```java
Class Product{
	static int count = 0;
	int serialNo;
	{
		++count;
		serialNo = count;
	}
	public Product();
}
```

## <span style="color:#802548">_class 메모리할당_</span>

- class도 메모리 할당이 이뤄진다.

```java
class Tv {
  //property
  String color;
  boolean power;
  int channel;

  void power() { power = !power;}
  void channelUp() { ++channel;}
  void channelDown() { --channel;}
}

class TvTest3 {
  public static void main(String[] args) {
    Tv t1 = new Tv();
    Tv t2 = new Tv();

    System.out.println("t1의 channel값은 " + t1.channel + "입니다.");
    System.out.println("t2의 channel값은 " + t2.channel + "입니다.");
    t2 = t1;
    t1.channel = 7;

  }
}
```

- color, power, channel을 가지고 있다.
- method는 heap에 저장되지 않고 method area에 따로 저장된다.
- t2가 원래 참조하고 있던 0x200은 GC가 잡아먹는다.
  <img src="/image/class-memory-allocation.jpg" >
  <img src="/image/class-memory-allocation2.jpg" >

## <span style="color:#802548">_OOP 구현 수단으로서 class_</span>

- primitive type으로 인자를 늘리는 건 OOP가 무너졌다는 신호일 수도 있다.

```java
int hour;
int minute;
float second;

public void transformTime(int hour, int miniute, float second);
```

- 이름을 주고 method를 주어 class로 만들면 primitive type으로 넣는 것보다 실수가 줄어든다.

```java
class Time {
  int hour;
  int minute;
  float second;
}

public void transformTime(Time time);
```

- business logic이 아니라 class 내에서 validation 코드를 만들 수 있다.
- setter안에서 validation을 진행하지 않았다면, 산재된 field에 모두 if문을 추가해줬어야 했다.

```java
class Time {
  int hour;
  int minute;
  float second;

  public int getHour() {
    return hour;
  }

  public int getMinute() {
    return minute;
  }

  public int getSecond() {
    return second;
  }

  public void setHour(int h) {
    if ( h <0 || h > 23) {
      return;
    }

    hour = h;
  }

  public void setMinute(int m) {
    if ( m <0 || m > 59) {
      return;
    }

    minute = m;
  }

  public void seHour(int s) {
    if ( s < 0 || s > 59.99f) {
      return;
    }

    second = s;
  }
}
```

## <span style="color:#802548">_method_</span>

- method가 호출되면, 호툴스택에 호출된 method를 위한 메모리가 할당된다.
- main method는 프로그램 실행 시 OS에 의해 자동적으로 호출된다.
- 나머지 method는 호출이 되어야만 JVM이 읽어간다.

```java
int result = add(3, 5); //argument. 실제값

int add(int x, int y) { //parameter. 복사본
  int result = x + y;

  return result;
}
```

- 원시형의 경우, byte가 같다면 자동 형변환을 실행해준다.
- double type의 parameter를 받아야하는데, long을 넣었도 오류가 나고 정상적으로 처리된다.

```java
public double divide(double a, double b) {
  return a/ b;
}

double result = divide(5L, 3L); //long이어도 오류 X
```

- method의 경우, 기본형 매개변수와 참조형 매개변수의 작동이 다르다.
- 기본형은 change()라는 stack에서 x 값을 만드는 것이라 main의 d와 무관하다.

```java
public static void main(String[] args) {
  Data d = new Data();
  d.x = 10;
  System.out.println("main() : x = " + d.x);

  change(d.x);
  System.out.println("After change(d.x)");
  System.out.println("main() : x = " + d.x); //10
}

static void change(int x) {
  x = 1000;
  System.out.println("change() : x = " + x); //1000
}
```

<img src="/image/primitive-method-parameter.jpg" >

- 참조형은 change()라는 stack에서 바꿔도 참조로 main()의 값을 바라보고 있어 값이 바뀐다.

```java
public static void main(String[] args) {
  Data d = new Data();
  d.x = 10;
  System.out.println("main() : x = " + d.x);

  change(d);
  System.out.println("After change(d.x)");
  System.out.println("main() : x = " + d.x); //1000
}

static void change(Data d) {
  d.x = 1000;
  System.out.println("change() : x = " + d.x); //1000
}
```

<img src="/image/reference-method-parameter.jpg" >

- 참조형인 배열의 경우도 class와 똑같다.

```java
public static void main(String[] args) {
  int[] x = {10};
  System.out.println("main() : x = " + x[0]);

  change(x);
  System.out.println("After change(d.x)");
  System.out.println("main() : x = " + x[0]); //1000
}

static void change(int[] x) {
  x[0] = 1000;
  System.out.println("change() : x = " + x[0]); //1000
}
```

- method는 call stack을 따른다. call stack에서 pop되면 해당 method가 발동되고 사라진다.
- 이후에도 해당 method의 값을 사용하려면 return이 있어야 한다.
- return하는 것은 객체의 '주소'다. 값이 아니다.
- stack에서 계산 한 뒤에 method가 소멸한 뒤에도 필요하다면, heap에 저장해두는 것이다.

<img src="/image/method-reference-return-heap.jpg" >

## <span style="color:#802548">_static_</span>

- instance 변수가 필요없다면 static을 붙이자.
- static을 붙이면 method 호출 시간이 짧아져 성능이 좋아진다.
- static이 없으면 호출되어야 할 method를 찾는 과정이 추가되 시간이 더 걸린다.
- 아래는 instance 변수를 활용했다.

```java
class MyMath2 {
  long a, b;

  long add() {
    return a + b;
  }

  long substract() {
    return a - b;
  }

  long multiply() {
    return a * b;
  }

  double divide() {
    return a / b;
  }
}

MyMath2 mm = new MyMath2();
mm.a = 200L;
mm.b = 100L;

mm.add();
```

- instance 변수를 쓰지 않고 parameter로 받게 바꾸면 쓰기도 쉽다.
- Math class가 static을 활용하는 사례다.

```java
class MyMath2 {
  static long add(long a, long b) {
    return a + b;
  }

  static long substract(long a, long b) {
    return a - b;
  }

  static long multiply(long a, long b) {
    return a * b;
  }

  static double divide(double a, double b) {
    return a / b;
  }
}

MyMath2.add(200L, 100L);
```

- static 변수는 모든 instance가 공유한다.
- 문서명이 없이 계속 만든다면 제목 없음0, 제목없음1, 제목없음2와 같은 식으로 count가 늘어날 것이다.
- 반면 제목을 부여한다면 제목문서가 생성되었습니다 라고 뜰 것이다.

```java
Class Document{
	Static int count = 0;
	String name;
	Document(){
		this("제목 없음" + count);
    count++;
	}

   Document(String name){
      this.name = name
      sysout(name + "문서가 생성되었습니다.");
   }
}
```

- static method는 instance method, instanc variable을 쓸 수 없다.
  - static method는 instance와 무관하게 작동가능해야 하기 때문이다.
  - static에서는 Instance가 없을 때도 작동해야 하기 때문에 instance를 가리키는 this도 쓸 수 없다.

## <span style="color:#802548">_overloading_</span>

- 매개변수 개수를 다르게 하거나, 매개변수 타입을 다르게 하거나, 그 순서를 바꾸는 것을 의미하여 같은 이름으로 새로운 method를 만드는 것을 의미한다.

```java
add(int a,long b)
```

- 매개변수의 순서를 바꾸면 오버로딩이 된다. 타입이 차례마다 다르게 들어가기 때문이다.

```java
add(long a, int b)
```

- 아래는 오버로딩이 아니다. 변수의 타입과 개수가 똑같기 때문이다.

```java
add(int b, int c)
```

- 매개변수의 개수가 다르기 때문에 아래는 오버로딩이다.

```java
 Add(int a)
```

- 다만 override가능한 method의 경우, 가변인자로 쓰는 것은 지양해야 한다.
- 정확히 어떤 게 들어갈 지 명시하는 게 바람직하다.

## <span style="color:#802548">_가변인자_</span>

- 만약에 type이 같은 여러개의 매개변수를 넣는다면 어떤 방식이 좋을까? 그럴 때는 가변인자를 쓰면 편리하다.
- 다만 가변인자인지 판별을 하기 위해 가변인자는 맨 마지막 매개변수로 들어가야 한다.
- 예시로 살펴보자.

```java
String concat(String a);
String concat(String a, String b);
String concat(String a, String b, String c);
```

- 이렇게 똑같은 type의 매개변수를 여러 개로 만들어서 오버로딩해야 한다면, 이 만큼의 코드를 다 써야해서 매우 불편할 것이다.
- 가변 변수를 사용한다면 여러개의 method를 overriding할 필요가 없다. 가변인자를 쓴다면 아래와 같이 간단해진다.

```java
String concat(String… str){…}
```

- 가변인자의 또다른 장점은 인자가 실제로 들어오지 않든, 인자를 n개를 넣든, 인자가 배열이든 상관없다는 것이다.
- 그저 같은 type이기만 하면 된다. 그렇다. 배열도 들어올 수 있다. 그렇다면 가변인자 대신 직접 배열을 넣는다면 어떨까?

```java
String concat(String[] str){…}
```

- 직접 배열을 넣게 되면 null일 때의 처리가 포함되지 않는다.
- 들어오지 않을 떄의 처리도 포함되지 않는다.
- 반면 위의 가변인자 식은 아래와 같이 복수 개의 method가 override된 것과 동일한 효과를 발휘한다.

```java
concat(String[] strArray);
concat(null)
concat();
concat(String a);
concat(String a, String b);
concat(String a, String b, String c);
```

## <span style="color:#802548">_extends_</span>

- 아래와 같이 상속으로 재사용을 하는 경우도 있다.

```java
class Circle extends Shape
```

- 상속받은 자식 class는 부모 클래스의 모든 것을 쓸 수 있다.
- 하지만 private 생성자를 사용하면 어떤 class에서도 접근할 수 없다.
- 따라서 private 생성자가 붙으면 extends가 불가능하다. final class와 같은 효과를 발휘한다.
  - private 생성자가 붙은 class를 가져오는 방법은 내부에 getter를 만드는 것이다.
  - 사실상 private 선언을 하게 되면 singleton처럼 하나의 instance만 사용하게 된다.

```java
final class Singleton{
    private static singleton s = new Singleton();

    private Singleton(){}


    public static Singleton getInstance(){
        return s;
    }
}
```

- 그럼 이제는 상속에 관해 더 살펴보자. 아래는 상속시켜줄 부모 class다.

```java
class Point{
    int x;
    int y;

     String getLocation(){
        return "x: " + x +", y: " + y;
    }

    Point(int x, int y){
        this.x = x;
        this.y = y;
    }
}
```

- 상속받은 자식의 경우, 접근제어자는 범위가 같거나 확대되어야 한다.
  - 부모가 protected라면, 자식은 protected 혹은 public이어야만 한다.
- 예외의 경우는 반대다. 범위가 구체적으로 축소되어야 한다.
  - 부모 Exception이라면 자식은 SQLException이어만 한다.
- extends한 class는 반드시 생성자 첫줄에서 부모의 생성자 중 하나를 호출해야만 한다.
  - 다만 기본형이라면 생략가능하다. 생성자를 넣지 않으면 compiler에서 기본 생성자를 넣어준다.

```java
class Point3D extends Point{
    int z;

    @Override
    String getLocation(){
        //return "x: " + x +", y: " + y + ", z: " + z;
        return super.getLocation() + ",z: " + z; //조상의 method호출로도 가능. 이게 바람직함. 그래야 조상이 바껴도 다시 자식에서 안바꿈.
    }

    Point3D() {
        //super()가 생략됨
        this.x = x;
        this.y = y;
        this.z = z;
    }
}
```

- 만약 조상 class가 기본형 생성자가 없다면, super를 반드시 호출해야한다.
  - 그 이유는 조상이 가진 멤버변수는 조상의 생성자로 초기화해야하기 때문이다.

```java
class Point3D extends Point{
    int z;

    @Override
    String getLocation(){
        //return "x: " + x +", y: " + y + ", z: " + z;
        return super.getLocation() + ",z: " + z; //조상의 method호출로도 가능. 이게 바람직함. 그래야 조상이 바껴도 다시 자식에서 안바꿈.
    }

    Point3D() {
      super(int x, int y)
      this.z = z;
    }
}

```

## <span style="color:#802548">_instance와 참조변수의 차이_</span>

- Tv type으로도 참조변수를 만들 수 있다.
- CaptionTv type으로도 참조변수를 만들 수 있다.
- 하지만 둘 모두 CaptionTv instance다.
  - 그러나 차이가 있다. Tv type의 참조변수 t는 CaptionTv의 멤버를 참조할 수 없다. 
  - 즉 text, caption method를 활용할 수 없다.
- 반면에 CaptionTv c는 CaptionTv의 멤버를 참조할 수 있다.

```java
class Tv{
    boolean power;
    int channel;

    void power(){power !=power}
    void channelUp(){++channel}
    void channelDown(){--channel}
}

class CaptionTv extends Tv{
    String text;
    void caption(){};
}

CatpionTv c = new CaptionTv();
Tv t = new CaptionTv();
```

- 실제 instance인 Tv class의 멤버갯수보다 참조변수 c가 사용할 수 있는 멤버변수가 많다.
- 따라서 자식 - 부모 관계가 역전된 참조-instance 관계는 생성이 불가능하다.
- 이와 똑같은 이유로 upcasting에는 형변환을 쓰지 않아도 되지만, downcasting에는 형변환이 필요하다.

```java
CaptionTv c = new Tv(); //error
```

- 위와 같이 상속관계를 만들었다고 해보자.
- 아래와 같은 형식의 형변환은 불가능하다. 참조형의 형변환은 자식-부모 관계에 있을 때만 가능하다.
- downcasting의 실제 사례는 아래와 같다.
- (String)이 필요한 이유는 map.get으로 꺼내오는 return type이 Object이기 때문이다. 
  - Object(부모) ->String(자식)이면 downcasting에 해당한다.

```java
HashMap<String,Object> map = new HashMap<>();
map.put("ddd","ddd");

String ddd = (String)map.get("ddd"); // 
```


## <span style="color:#802548">_다형성_</span>

- Prodcut class와 그를 extends한 class들이 있다.
- buy()를 overloading해서 각각 다른 Product를 정확하게 받는 method를 구현할 수도 있다.
```java
class Product{
    int price;
    int bonusPoint;
}

class Tv extends Product{}
class Computer extends Product{}
class Audio extends Product{}

class Buyer{
    int money = 0;
    int bonusPoint = 0;

    void buy(Tv t){
        money = money - t.price;
        bonusPoint = bonusPoint + t.bonusPoint;
    }

    void buy(Computer c){
        money = money - c.price;
        bonusPoint = bonusPoint + c.bonusPoint;
    }
}
```

- 하지만 아래와 같이 해당 부모 class를 받는 형태로 진행하면 똑같은 method를 반복해서 만들 필요가 없어진다.
- 이것이 바로 다형성이다. 

```java
class Buyer{
    int money;
    int bonusPoint;
    void buy(Product p){
        money = money - p.price;
        bonusPoint = bonusPoint + p.bonusPoint;
    }
}
```

## <span style="color:#802548">_abstract class_</span>

- abstract class는 멤버변수와 method를 선언할 수 있다.
- abstract class는 그냥 method도 가질 수 있고 abstract method도 가질 수 있다.
- abstract가 붙은 method는 method body를 가질 수 없다. 오직 header만 존재한다.
- abstract class를 계승한 class는 abstract method를 반드시 override해야한다.

```java
abstract class Player{
    boolean pause;
    int currentPos;

    Player(){
        pause = false;
        currentPos = 0;
    }

    abstract void play(int pos);
    abstract void stop();

    void play(){
        play(currentPos);
    }
}
```

- abstract class를 만드는 예시를 살펴보자.

```java
class Marine{
    int x, int y;
    void move(int x, int y){};
    void stop(){};
    void stimpack(){};
}

class Tank{
    int x, int y;
    void move(int x, int y){};
    void stop(){};
    void changeMode(){};
}

class Dropship{
    int x, int y;
    void move(int x, int y){};
    void stop(){};
    void unload(){};
}
```

- 위를 보면 중복된 method와 멤버변수가 많다.
- 그럴 때 abstract class를 활용할 수 있다.

```java
abstract class Unit{
    int x;
    inx y;
    abstract void move(int x, int y);
    void stop();
}

class Marine extends Unit{
    void move(int x, int y){};
    void stimpack();
}

class Tank extends Unit{
    void move(int x, int y);
    void changeMode();
}

class Dropship extends Unit{
    void move(int x, int y);
    void unload();
}
```

- 위와 같이 중복은 abstract class로 빼고, 이를 계승하게 만들면 된다.
- 각 instance에 맞는 method를 어떻게 호출할 지 고민하지 않아도 된다.
  - Unit class는 abstract class다. move도 abstract method로 method body가 없다. 
  - 이 경우 실제 들어온 instance에서 구현한 move method가 호출된다.

```java
Unit[]  group = new Unit[5];
group[0] = new Marine();
group[1] = new Tank();
group[2] = new Dropship();

for(int i = 0; i < group.length; i++){
    group[i].move(2,2); //0번째엔 Marine의 move가, 1번째엔 Tank의 move가, 2번째엔 Dropship의 move가 호출된다.
}
```

## <span style="color:#802548">_interface_</span>

- interface는 method와 멤버변수를 가진다. 
  - 다만 멤버변수는 모두 public static final이어야 한다. 
  - Method 또한 public abstract가 있어야만 한다.

```java
public interface caculatable{
	public static final Double pi = 3.14;
	public abstract Void add(int x, int y);
}
```

- 하지만 아래와 같이 멤버변수에 public static final은 생략가능하다.
- method에서도 public abstract는 생략가능하다.
- 생략해도 compiler에서 알아서 붙여준다.
- interface에 쓰는 method는 가변인자를 지양하고 정확한 매개변수를 지정해주는 게 좋다.

```java
public interface caculatable{
	double pi = 3.14;
	void add(int x, int y);
	//void add(int… args); override가능한 method의 경우, 가변인자로 쓰는 것은 지양
}
```

```java
class employee implements Caculatable{
	int x;
	int y;

  public void add(int x, int y) {
    ...
  }
}
```

- static, default method의 경우 접근제어자 생략은 불가능하다. 다만 잘 쓰이지는 않는다.
  - interface 또한 상속과 동일하게 조상타입의 참조변수로 설정할 수 있다.
  - 또한 이러한 경우에는 상속과 동일하게 조상타입의 method만 사용가능하다.
- 아래 경우가 그러한 사례다. Fightable type으로 참조변수가 생겼다.
  - Fighter class에만 있는 method인 escap()를 활용할 수 없다.

```java
interface Fightable{
	void getReady();
}

class Fighter implements Fightable{
	@Override
	void getReady(){
		...
	}

	void escape(){

	}
}


Fightable f = new Fighter(); //Fighter의 escape는 쓸 수 없다.
```

- interface를 매겨변수 type으로 넘길 수도 있다. 이 경우 해당 interface를 구현한 class를 넘겨야 함을 의미한다.

```java
Void attack(Fightable f){
	…
}

attack(new Fighter());
```

- 마찬가지로 return type이 interface일 수 있다. 
- 이 경우 해당 interface를 구현한 class를 return해주면 된다.
  - Parseable interface를 return type으로 지정했다. 
  - 그리고 실제 return되는 class는 Parseable interface를 구현한 XMLParser()와 HTMLParser()이다.

```java
public static Parseable getParser(String type){
	if ( type.equals("XML") ) {
		return new XMLParser():
	} else {
		Parseable p = new HTMLParser():
		Return p;
	}
}
```

- 이제는 interface와 extends를 어떻게 활용하는 지 아래의 예시를 살펴보자.
- 우선 대분류 유닛을 만들고, 공중유닛과 지상유닛을 나눈다.

```java
class Unit {
	int hitPoint;
	final int MAX_HP;

	Unit(int hp){
		MAX_HP = hp;
	}
}

class GroundUnit extends Unit{
	GroundUnit(int hp){
		Super(hp);
	}
}

class AirUnit extends Unit{
	AirUnit(int hp){
		Super(hp)
	}
}
```

- 이제 개별 유닛을 해당 분류에 집어넣는다. 코드로는 extends다.

```java
class Tank extends GroundUnit {
	Tank(){
		Super(150);
		hitPoint = MAX_HP;
	}

	public String toString(){
		Return "Tank";
	}
}

class Dropship extends AirUnit {
	Dropship(){
	super(125);
		hitPoint = MAX_HP;
	}
	public String toString(){
		Return "Dropship";
	}
}

class Marine extends GroundUnit{
	Marine(){
	super(40);
		hitPoint = MAX_HP;
	}

	public String toString(){
		Return "Marine";
	}
}

class SCV extends GroundUnit {
	SCV(){
		Super(60);
	}

	public String toString(){
		Return "SCV";
	}
}
```

- 그러나 이로는 부족하다. 수리 같은 경우, Unit에게 달려 있는 특성이 아니라 수리가 가능한 Unit이 따로 존재한다.
- 그럴 때 extends가 아닌 수리가능이라는 interface를 추가한다.
- default method의 경우, body를 가질 수 있다.

```java
interface Repairable(){
  default void method1(){
		Sysout("method1() in Myinterface");
	}
}
```

- 이제 해당 interface를 구현한 class만 수리가능하게 해주면 된다.
- Tank, Dropship, SCV는 수리 가능하고, Marine은 수리 불가능하다.

```java
class Tank extends GroundUnit implements Repairable{
	Tank(){
		Super(150);
		hitPoint = MAX_HP;
	}

	Public String toString(){
		Return "Tank";
	}
}

class Dropship extends AirUnit implements Repairable{
	Dropship(){
	super(125);
		hitPoint = MAX_HP;
	}

	Public String toString(){
		Return "Dropship";
	}
}

class Marine extends GroundUnit{
	Marine(){
	super(40);
		hitPoint = MAX_HP;
	}
}
```

- SCV가 수리를 해주므로 repair method는 SCV에 둔다.
- repair할 대상은 Repairable interface를 구현한 대상으로 한정한다.
  - 그 방법은 Repairable interface를 구현한 class만 매개변수로 받게끔 하는 것이다.

```java
class SCV extends GroundUnit implements Repairable{
	SCV(){
		Super(60);
	}

	Void repair(Repairable r){
		if (r instanceof Unit) {
			Unit u = (Unit)r;
			while(u.hitPoint ! = u.MAX_HP){
				u.hitPoint++;
			}

			Sysout(u.toString() + "의 수리가 끝났습니다.");
		}
	}
}
```

- 자 그럼 이제 test를 해보자.
- marine은 repairable interface를 implements하지 않았다.
- 그런 이유로 repair()의 parameter type인 Repairable이 아니다.

```java
class RepairableTest{
	public static void main(String[] args){
		Tank tank = new Tank();
		Dropship dropship = new Dropship();

		Marine marine = new Marine();
		SCV scv = new SCV();
		Scv.repair(tank); //ok
		Scv.repair(dropship); //ok
		Scv.repair(marine); //error
		}
}
```


- repair 가능한 것은 기계만이 아니라, 건물도 있다.

```java
class Building {
	int hitPoint;
	final int MAX_HP;

	Building(int hp){
		MAX_HP = hp;
	}
}

class Barrack extends Building{
	super(1500);

	public String toString(){
		return "Barrack";
	}
}
```



- 따라서 아래같이 building instance일 때의 else문이 추가된다.
- 사실 instanceof가 활용되는 것은 바람직하지 않은 상황이다.
- building과 unit으로 나눈 설계가 과연 유의미한지 검토가 필요할 수 있따.

```java
class SCV extends GroundUnit implements Repairable{
	SCV(){
		Super(60);
	}

	Void repair(Repairable r){
		if(r instanceof Unit){
			Unit u = (Unit)r; //r.hitPoint는 불가능. Repairable interface에 hitPoinnt라는 멤버변수가 없음. 그래서 형변환하는 것.
							 //형변환도 특정 구현 class가 아닌 Unit으로 하였음. 그럼 특정 구현 class마다 if문을 만들 필요가 없음.
							 //Repairable interface가 형변환이 가능한 이유는 instance로는 특정 구현 class가 들어오기 때문
							 //해당 구현 class가 Unit class와 자식 - 부모 관계라면 형변환 가능.
			while(u.hitPoint ! = u.MAX_HP){
				u.hitPoint++;
			}
			Sysout(u.toString() + "의 수리가 끝났습니다.");
		}else if (r instanceof Building){
			Building b = (Building)r;  //Barrack이 들어왔다고 해보자. Barrck은 부모 Building class의 멤버변수 hitPoint를 쓰기 때문에
									   //Building으로 형변환해서 해당 멤버변수를 활용해도 된다.
									   //하지만 Barrack에만 있는 멤버변수는 이런 식으로 활용이 불가능하다.
			while(b.hitPoint != b.MAX_HP){
				b.hitPoint++;
			}
			sysout(b.toString() + "의 수리가 끝났습니다.");
		}
	}
}
```

- interface의 장점은 extends에 묶이지 않는다는 것이다.
- 행위를 추가하고 싶다면 interface를 구현하면 된다.
- 건물을 들어올리는 것은 extends가 아닌 아래와 같은 interface를 구현하여 해결한다.

```java
interface Liftable{
	void liftoff();
	void move();
	void stop();
	void land();
}

class LiftableImpl implements Liftable{
	public void liftoff(){};
	public void move(int x, int y){};
	public stop(){};
	public void land(){};
}

class Barrack extends Building implements Liftable{
	LiftableImpl l = new Liftable();
	void liftoff(){ l.lifteoff()};
	void move(int x, int y){ l.move(x,y)};
	void stop(){l.stop()};
	void land(){l.land()};
	void makeTank(){};
}
```

- interface는 아래와 같이도 활용할 수 있다.
- B class의 instance가 아니라 D로 return하고 싶다면, interface I를 구현하는 D class를 만들어주면 된다.

```java
static class InterfaceManager{
	private InterfaceManager(){

	}

	public static I getInstance(){
		return new B(): //여기만 나중에 new C(); new D();로 바꿔주면 됨.
	}
}
```
