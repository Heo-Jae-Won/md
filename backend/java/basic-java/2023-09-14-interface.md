## <span style="color:#802548">_재사용이란?_</span>
- 아래와 같이 상속으로 재사용을 하는 경우도 있다.
```java
class Circle extends Shape
```

- 상속받은 자식 class는 부모 클래스의 모든 것을 쓸 수 있다.
- private 멤버변수도 public method로 접근하여 값을 가져오거나 바꿀 수 있다.
- 하지만 private 생성자를 사용하면 어떤 class에서도 접근할 수 없다.
- 따라서 private 생성자가 붙으면 extends가 불가능하다. final class와 같은 효과를 발휘한다.
- private 생성자가 붙은 instance는 그 안에 public static으로 instance를 얻어올 수 있는 method를 만들어야 한다.

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
- 아래 Point를 extends한 자식 class는 int 타입 변수 2개를 그대로 활용할 수 있다.
- 또한 method도 overriding해서 사용할 수 있다.
- @Override를 붙여줄 수도 안 붙여줄 수도 있지만, 대개 붙이는 것이 좋다.
- Override의 조건은 아래와 같다.
  1. 부모 method와 이름이 동일
  2. 매개변수 타입과 갯수가 동일
  3. 반환타입이 동일
- 상속받은 자식의 경우, 접근제어자는 범위가 같거나 확대되어야 한다. 
- 부모 -> protected/ 자식은 protected 혹은 public이어야만 한다.
- 예외의 경우는 반대다. 범위가 구체적으로 축소되어야 한다.
- 부모 -> Exception / 자식은 SQLException이어만 한다.

```java
class Point3D extends Point{
    int z;

    @Override
    String getLocation(){
        //return "x: " + x +", y: " + y + ", z: " + z;
        return super.getLocation() + ",z: " + z; //조상의 method호출로도 가능. 이게 바람직함. 그래야 조상이 바껴도 다시 자식에서 안바꿈.
    }

    Point3D{
        //super()가 생략됨
        this.x = x;
        this.y = y;
        this.z = z;
    }
}
```

- extends한 class는 반드시 생성자 첫줄에서 부모의 생성자 중 하나를 호출해야만 한다.
- 생성자를 넣지 않으면 compiler에서 기본 생성자를 넣어주지만, 만약 조상 class에서 기본 생성자가 없다면?
- 그에 맞춰서 조상 생성자를 명시적으로 호출해야 한다.
- 그 이유는 조상이 가진 멤버변수는 조상의 생성자로 초기화해야하기 때문이다.

- 따라서 위의 Point3D의 생성자는 오류를 낸다. 현재 Point에는 기본생성자가 없기 때문이다. 
- 아래와 같이 부모의 생성자를 넣어 조상의 멤버변수를 초기화해준다.

```java
Point3D{
    super(int x, int y)
    this.z = z;
    }
```

- 아래와 같이 포함으로 재사용을 하는 경우도 있다.
- 특히 Spring의 DI에서 해당 기법이 사용된다.
```java
class Circle{
    int x;
    int y;
    int z;
}

class Circle{
    Point c = new Point(1,1);
    int z;

    Circle(Point p, int z){
        this.p = p;
        this.z = z;
    }
}

class Point{
    int x;
    int y;

    Point(int x, int y){
        this.x = x;
        this.y = y;
    }
}
```

## <span style="color:#802548">_상속의 형변환이란?_</span>

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

- Tv type으로도 참조변수를 만들 수 있다.
- CaptionTv type으로도 참조변수를 만들 수 있다.
- 둘 모두 CaptionTv instance다.
- 그러나 차이가 있다. Tv type의 참조변수 t는 CaptionTv의 멤버를 참조할 수 없다. 즉 text, caption method를 활용할 수 없다.
- 반면에 CaptionTv c는 CaptionTv의 멤버를 참조할 수 있다. 따라서 이를 유의하여야 한다.

```java
CaptionTv c = new Tv();
```
- 위와 같은 형태는 불가능하다.
-  실제 instance인 Tv class의 멤버갯수보다 참조변수 c가 사용할 수 있는 멤버변수가 많아 mapping이 안될 수 있기 때문이다.
- 그래서 애초에 자식 - 부모 관계가 역전된 참조-instance 관계는 생성이 불가능하다.
- 이와 똑같은 이유로 upcasting에는 형변환을 쓰지 않아도 되지만, downcasting에는 형변환이 필요하다.

```java
class Car{
    String color;
    int door;
    void drive(){
        sysout("drive");
    }
    void stop(){
        sysout("stop!");
    }
}

class FireEngine extends Car{
    void water(){
        sysout("water!");
    }
}

class Ambulance extends Car{
    void siren(){
        sysout("siren!");
    }
}
```

- 위와 같이 상속관계를 만들었다고 해보자.
- 아래와 같은 형식의 형변환은 불가능하다. 참조형의 형변환은 자식-부모 관계에 있을 때만 가능하다. 

```java
Ambulance a =(Ambulance)f; //불가능
FireEngine f = (FireEngine)a; //불가능
```

- 아래와 같은 형변환은 가능하다.
- 다만 자식 ->부모의 형변환은 자식이 반드시 부모를 포함해 더 많은 멤버를 가지므로 형변환을 쓰지 않아도 된다.
- 그러나 부모 ->자식으로의 형변환은 부모는 자식보다 덜 갖고 있기 때문에 형변환을 명시적으로 써야만 한다.
```java
Car car = null;
FireEngine fe = new FireEngine();
FireEngine fe2 = null;
car = fe; //(Car) 생략 가능하다. upcasting이기 때문이다. 
car.water(); // error. car type의 참조변수는 water()를 호출할 수 없다.
fe2 = (FireEngine) car; //downcasting이기 때문에 반드시 형변환을 명시한다.
```

- downcasting의 실제 사례는 아래와 같다.

```java
HashMap<String,Object> map = new HashMap<>();
map.put("ddd","ddd");

String ddd = (String)map.get("ddd"); // (String)이 필요한 이유는 map.get으로 꺼내오는 return type이 Object이기 때문. Object(부모) ->String(자식)이면 downcasting!
```

- 자식은 늘 부모 instance의 대상이다.
```java
FireEngine f = new FireEngine();
if(f instanceof Object) // true
if(f instanceof Car) // true
if(f instanceof FireEngine) //true
```

## <span style="color:#802548">_다형성이란?_</span>

```java
class Product{
    int price;
    int bonusPoint;
}

class Tv extends Product{}
class Computer extends Product{}
class Audio extends Product{}
```

- 위와 같은 형태의 Prodcut class와 그를 extends한 class들이 있다.

- 그렇다면 아래와 같이 Product를 extends한 모든 class를 매개변수로 지정할 수도 있다.

```java
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

## <span style="color:#802548">_abstract란?_</span>

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
- 그리고 나서 아래와 같이 활용할 수 있다.
- 이것이 바로 다형성이다. 각 instance에 맞는 method를 어떻게 호출할 지 고민하지 않아도 된다.
- Unit class는 abstract class다. move도 abstract method로 method body가 없다. 이 경우 실제 들어온 instance에서 구현한 move method가 호출된다. 
- 
```java
Unit[]  group = new Unit[5];
group[0] = new Marine();
group[1] = new Tank();
group[2] = new Dropship();

for(int i=0;i<group.length;i++){
    group[i].move(2,2); //0번째엔 Marine의 move가, 1번째엔 Tank의 move가, 2번째엔 Dropship의 move가 호출된다.
}
```

## <span style="color:#802548">_interface란?_</span>

- interface는 method와 멤버변수를 가진다. 다만 멤버변수는 모두 public static final이어야 한다. Method 또한 public abstract가 있어야만 한다. 

```java
public interface caculatable{
	public static final Double pi = 3.14;
	public abstract Void add(int x, int y);
}
```

- 하지만 아래와 같이 멤버변수에 public static final은 생략가능하다. 
- method에서도 public abstract는 생략가능하다.
- 생략해도 compiler에서 알아서 붙여준다. 

```java
Public interface caculatable{
	double pi = 3.14;
	void add(int x, int y);
	void add(int… args); // override가능한 method의 경우, 가변인자로 쓰는 것은 지양
}
```

```java
class employee implements Caculatable{
	int x;
	int y;
}
```

- Staic, default method의 경우 접근제어자 생략은 불가능하다. 다만 잘 쓰이지는 않는다.

- interface 또한 상속과 동일하게 조상타입의 참조변수로 설정하고 instance는 자손 class로 할당할 수 있다.  
- 또한 이러한 경우에는 상속과 동일하게 조상타입의 method만 사용가능함.
- 아래와 같이 참조변수는 Fightable interface인데, instance는 Fighter class다.
- 이 경우에는 Fightable type으로 참조변수가 생겼기 때문에 Fighter class에만 있는 멤버를 활용할 수 없다.
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
- 아래에서 Fightable type의 매개변수는 interface인 Fightable을 넣으라는 게 아니라 그를 구현한 class를 넣으라는 뜻이다.

```java
Void attack(Fightable f){
	…
}

attack(new Fighter());
```

- 마찬가지로 return type이 interface일 수 있다. 이 경우 해당 interface를 구현한 class를 return해주면 된다. 
- Parseable interface를 return type으로 지정했다. 그리고 실제 return되는 class는 Parseable interface를 구현한 XMLPasrse()와 HTMLParser()이다.

```java
public static Parseable getParser(String type){
	if(type.equals("XML"){
		return new XMLParser():
	}else {
		Parseable p =new HTMLParser():
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

```java
interface Repairable(){}
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
		Return “Tank”;
	}
}

class Dropship extends AirUnit implements Repairable{
	Dropship(){
	super(125);
		hitPoint = MAX_HP;
	}

	Public String toString(){
		Return “Dropship”;
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
- 그런데 repair할 대상은 Repairable interface를 구현한 대상으로 한정한다.
- 그 방법은 Repairable interface를 구현한 class만 매개변수로 받게끔 하는 것이다.
- 즉 해당 repairable interface를 매개변수로 설정하는 것이다.
- Unit의 instance인지 보는 이유는 건물도 수리가능하기 때문이다. 따라서 이에 맞춰 형변환이 필요한 것이다.


```java
class SCV extends GroundUnit implements Repairable{
	SCV(){
		Super(60);
	}

	Void repair(Repairable r){
		if(r instanceof Unit){
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
		}else if(r instanceof Building){
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
- 만약 모든 class마다 굳이 개별적으로 구현할 필요가 없다면, 아래와 같이 공통 구현 class를 만드는 것도 좋은 선택지다.

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
- Spring에서 Db connector를 만들 때 이런 방식을 활용한다.

```java
class InterfaceManager{
	private InterfaceManager(){

	}
	public static I getInstance(){
		return new B(): //여기만 나중에 new C(); new D();로 바꿔주면 됨.
	}
}

interface I{
	public abstract void method(); //public abstract는 생략 가능
}

class B implements I{
	public void method(){
		sysout("MethodB in B class");
	}
	public String toString(){
		return "class B";
	}
}
```
- 아래는 실제 test 사례다.

```java
class A{
	void methodA(){
		I i = new InstanceManager.getInstance():
		i.methodB();
	}
}

class InterfaceTest{
	public static void main(String[] args){
		A a = new A();
		a.methodA();	
	}
}
```

- default method는 약간 유의해야 한다.
- Child class가 두 개의 interface를 implements했는데 둘 모두 같은 이름의 default method가 있을 경우가 있다.
- 이 때는 그냥 Child class에서 해당이름으로 override해서 사용하는 것이 좋다.

```java
class DefaultMethodTest{
	public static main(String[] args){
		Child c = new Child();
		c.method();
		c.method2(); //method2는 Child에는 없어서 조상 class`에 있나 보고 조상 class의 method 호출. Child class 참조변수는 조상클래스 것은 전부 쓸수있다는 사실을 잊지말자.
		MyInterface.staticMethod();
		MyInterface.staticMethod();
	}
}

class Child extends Parent implements MyInterface, MyInterface2{
	public void method1(){ //override된 것. @Override가 없지만, 이건 필수가 아니라서 그럼.
		Sysout("method1() in Child"):
	}
}

class Parent{
	public void method2(){
		sysout("method2Op in Parent");
	}
}

interface MyInterface{
	default void method1(){
		Sysout(“method1() in Myinterface”);
	}
	default void method2(){
		Sysout(“method2() in MyInterface”);
	}
	static void staticMethod(){
		Sysout(“staticMethod() in MyInterface”):
	}
}

interface MyInterface2{
	default void method1(){ //default method라서 method body가 존재
		Sysout(“method1() in MyInterface2”);
	}

	static void staticMethod{ //static method라서 method body가 존재
		Sysout(“staticMethod() in MyInterface”):
	}
}
```

```java
class InnerEx{
	private int outerIv = 0;
	static int outerCv = 0;
	
	class InstanceInner{
		Int iiv = outerIv; // 내부 class는 외부 class의 private 변수를 사용할 수 있다. 외부 class의 instance 안에서 존재하기 때문이다.
		Int iiv2 = outerCv;
	}

	static class StaticInner{
		//int siv = outerIv; static 내부 class의 경우에는 외부 class의 멤버변수에 접근할 수 없다. static이라서 외부 class없이도 존재할 수 있기 때문이다.
		static int scv = outerCv;
	}

	void myMethod(){
		int iv = 0; //지역변수의 경우, method 내부의 지역 class에 쓰인다면 반드시 final이 붙어야 한다. 하지만 1.8이후로는 생략 가능하다. 생략해도 final이 붙어있는 것이다.
		final int LV = 0;
		
		class LocalInner{
			int liv = outerIv;
			int liv2 = outerCv;

			int liv3 = iv;
			int liv4 = LV;
		}
	}
}
```

- 아래는 익명클래스의 예시다.
- Spring에서는 내부 클래스 중에서 익명 클래스말고는 잘 활용되지 않는다.
- 위의 방식은 Android에서 주로 활용된다.
```java
class InnerEx8{
	public static void main(String[] args){
		Button b = new Button("start"):
		b.addActionListener(new AcitionListener(){
			public void actionPerformed(ActionEvent e){ //overriding?
				sysout(“ActionEvent occurred!!”);
			}
		})
	}
}
```















