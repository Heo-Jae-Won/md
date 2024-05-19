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

- 그러나 가변인자는 사용 method는 overload에 이용하지 않는 게 좋다. 헷갈릴 수 있기 때문이다.



- 반면에 아래와 같이 클래스 변수가 아닌 인스턴스변수로 했다면 매번 SerialNo는 0으로 초기화되어 모든 SerialNo는 1로 고정이었을 것이다.
```java
Class Product{
	int count = 0;
	int serialNo;
	{
		++count;
		serialNo = count;
	}
	public Product();
}
```


 - count가 static이라서 모두가 공유하기 떄문에 문서명이 없이 계속 만든다면 제목 없음0, 제목없음1, 제목없음2와 같은 식으로 count가 늘어날 것이다.
 - 반면 제목을 부여한다면 제목문서가 생성되었습니다 라고 뜰 것이다. 예로 로아.txt라고 한다면 로아.txt문서가 생성되었습니다라고 뜬다.
```java
Class Document{
	Static int count = 0;
	String name;
	Document(){
		this(“제목 없음” + ++count);
	}

   Document(String name){
      this.name = name
      sysout(name + "문서가 생성되었습니다.");
   }
}
```