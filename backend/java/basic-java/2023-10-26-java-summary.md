## <span style="color:#802548">_1. 데이터_</span>

- 기본형: 정수형, 실수형, 논리형, 문자형
  <br/>
  정수형: `overflow 가능성이 있다.` 크면 long. 작으면 int. 너무 크면 정수형이 아닌 BigInt 사용. 곱연산 시에, int와 int 끼리 곱한 것은 무조건 int다.

```java
int a = 1_000_000;
int b = 2_000_000;
int c = a * b; //overflow
long c = a * b; //overflow. a와 b를 곱한 것은 long이 아니라 int다. long으로 선언해도.. 2000000000000이 아니라 -1454759936로 나옴
long c = (long)a * b; // overflow가 나지 않는다. 2000000000000. data가 포괄할 수 있는 더 넓은 범위로 옮겨간다.
long c = a * (long)b; //overflow가 나지 않는다. 2000000000000
```

<br/>

실수형: overflow와 underflow 모두 가능. 소수점 많으면 double, 적으면 float. 처음부터 float로 해놓고 double 형변환해봐야 float 값

```java
float a  = 0.0_000_001f;
float b = 0.0_000_001f;
double c = a * b; //1.000000023372195E-14
double c = (float)a * b; //1.000000023372195E-14
double c = (float)a * (float)b; //9.9999998245167E-15
```

- 배열: 같은 type의 값을 모아둠

```java
int[] arr = new int[5]{10,20,30,40,50}; //생성 당시에 바로 기본 element 할당

System.out.print(arr) //주소값 출력됨

for(int i=0;i<arr.length;i++){
	System.out.print(arr[i]) //값 출력됨
}

System.out.print(Arrays.toString(arr)) //값 출력됨.for문과 동일한데 간편.
```

- 배열은 index가 꽉 차면 새로운 배열을 만들어서 옮겨줘야 한다.

```java
int[] arr = new int[5];
int[] temp = new int[arr.length * 2]
for (int i = 0; i<arr.length; i++) {
	Temp[i]=arr[i]
}
arr = temp

char[] abc = {'A','B','C','D'};
char[] num ={'0','1','2','3','4','5'};
char[] result = new char[abc.length + num.length];
System.arraycopy(abc, 0, result, 0, abc.length); //for문과 동일한데 간편.
System.arraycopy(num, 0, result, abc.length , num.length); //abc와 num 모두 copy
System.out.println(Arrays.toString(result)); //값 출력됨
```

- 대표적인 배열을 이용한 로직 연습은 아래와 같다.

1. 평균내기
2. 최댓값구하기, 최솟값 구하기
3. 랜덤하게 숫자 순서 바꾸기
4. 불연속 값으로 배열 채우기
5. 버블정렬 내림차순, 오름차순
6. 특정문자에 들어가있는 10진수 갯수 세기
7. 내림차순 정렬
8. 2차원 배열

- 그 외 class도 데이터의 형식. class 안에 inner class로 데이터만 담는 구조체를 만드는 경우도 있음. 아주 가끔.. 추천되지 않는 방식

## <span style="color:#802548">_2.선언자_</span>

- 데이터에 맞는 선언을 만들어줘야 한다. int, float 등..
- 사용 전에 초기화하는 습관을 들이자.
- 상수는 값 변경 불가. 선언과 초기화(할당)이 `동시에` 이뤄져야 한다.

```java
int a; //선언. 클래스변수에서는 값이 0
a = 5; //할당
char b ='B'; //초기화(선언 + 할당)
final String a =5; //선언과 할당이 늘 한번에 이뤄짐.
```

- 그 외에 선언자로 private, public, protected, static 등을 사용한다.

```java
private String name;

public setName(String name){
	this.name = name;
}

public getName(){
	return this.name;
}

class Horizon{
	public static final String CIRCLE= "원";  //static method를 제외하면 지역변수로 static 사용 불가
}

void horizon(){
	final String circle = "원";
}

public class ErrorUtil extends RuntimeException{
	protected String ErrorMsg;
}
```

- 데이터가 아닌 곳에도 final이 붙을 수 있다.
- 그경우에도 의미가 있다.
- class에 final이 붙으면 다른 class에서 상속이 불가능하다.
- method에 final이 붙으면 override가 불가능하다.

```java
final class MyClass{
	private String name;

	MyClass(String name){
		this.name = name;
	}
}

class MyClass{
	private String name;

	MyClass(String name){
		this.name = name;
	}

	final public String getName(){
		return name;
	}
}
```

- 생성자에 private이 붙으면 외부에서 접근이 불가능해서 내부에서만 instance를 만들게 된다.
- enum이 이러한 방식을 사용한다.

```java
enum MyClass{
	My(1);

	private int number;

	/*private */MyClass(int number){ //암묵적인 private 생성자. enum은 암묵 default가 아니다.
		this.number = number;
	}
}
```

## <span style="color:#802548">_3.식의 활용_</span>

- 식은 ;로 끝난다.
- 제어문, 반복문이 대표적이다.
- 아래처럼 &&을 사용하는 것은 연산에 악영향을 미치므로 최대한 자제하자.

```java
if(score >=90){
	Grade=’A’;
}else if(80<= score && score <90){
	Grade =’B’;
}else if(70 <= score && score < 80){
	Grade =’C’;
}else {
	Grade = ‘D’;
}

if(score >=90){
	Grade =’A’;
}else if(score >=80){
	Grade =’B’;
}else if(score  >=70){
	Grade =’C’;
}else {
	Grade =’D’;
}
```

- 숫자가 범위가 아니라 딱 하나의 수라면 아래와 같은 `switch문`도 유효하다. 연산을 if문보다 훨씬 덜한다.
- 하지만 범위라면 아래와 같이 너무 비효율적인 switch문이 만들어진다.

```java

if(menu==1){

}else if(menu == 2){

}else if(menu == 3){

} //비효율적

int menu = Integer.parseInt(tmp);
switch(menu){
	case 1:
		System.out.println("result = " + num * num):
		break; //switch문에서의 break;
	case 2:
		System.out.println("result = " + Math.sqrt(num));
		break; //switch문 끝나고서 진행되는 소스코드있으면 다 진행됨
	case 3:
		System.out.println("result = " + Math.log(num));
		break;
}

int score = request.getParameter('score');
switch(score){
    case 90:
    Grade ='A';
    break;
	case 91:
    Grade ='A';
    break;
	case 92:
    Grade ='A';
    break;
	case 93:
    Grade ='A';
    break;
	case 94:
    Grade ='A';
    break;
	case 95:
    Grade ='A';
    break;
	case 96:
    Grade ='A';
    break;
	case 97:
    Grade ='A';
    break;
	case 98:
    Grade ='A';
    break;
	case 99:
    Grade ='A';
    break;
}

switch (sb.toString()) {
	case "FFD8FFE0": // JPEG image
	case "FFD8FFE1": // JFIF image
		fileExtension = ".jpg";
		break;
	case "89504E47": // PNG image
		fileExtension = ".png";
		break;
	case "47494638": // GIF image
		fileExtension = ".gif";
		break;
	default: // Unknown file type
		fileExtension = "";
		break;
} // break문 처리가 필요하지 않은 경우에는 위와 같이 하지 않을수도 있다. 정처기에 자주 나오는 듯
```

- while문은 아래와 같다.
- while문을 빠져나오려면 `flag를 false`로 만들던지, `break;`를 걸던지 하면 된다.

```java
boolean flag = true; //반복문 전 변수 초기화
while(flag){
	if(num!=0){
		System.out.println(num)
	}else{
		flag=false;
	}
}
while(flag){
	if(num!=0){
		System.out.println(num);
	}else{
		break;
	}
}
```

- 반복문이 안에 하나 더 있으면 break;를 구별해서 주기 위해 위와 같이 `Outer:`로 주기도 한다. 꼭 Outer일 필요는 없다.

```java
Outer:
	for(;;){ //while(true)로 해도 동일하게 잘 적용됨.
		System.out.println("(1) square");
		System.out.println("(2) square root");
		System.out.println("(3) log ");
		System.out.println("원하는 메뉴 선택");

		String tmp = scanner.nextLine();
		int menu = Integer.parseInt(tmp);
		if(menu==0){
			System.out.println("프로그램 종료");
			break;
		}else if(!(1<=menu && menu <=3)){
			System.out.println("메뉴를 잘못 선택했습니다. 종료는 0");
			continue;
		}

		for(;;){
			System.out.println("계산 값 입력. (계산 종료: 00, 전체종료:99)");
			String temp = scanner.nextLine();
			int num = Integer.parseInt(temp);

			if(num==0){
				break; // 해당 if문을 감싼 for문을 벗어남.
			}
			if(num==99){
				break Outer; // 해당 if문을 감싼 for문이 아니라 outer라는 이름을 가진 반복문 전체를 벗어남.
			}

			switch(menu){
				case 1:
					System.out.println("result = " + num * num);
					break; //switch문에서의 break;
				case 2:
					System.out.println("result = " + Math.sqrt(num));
					break; //switch문 끝나고서 진행되는 소스코드있으면 다 진행됨
				case 3:
					System.out.println("result = " + Math.log(num));
					break;
			}
		}
	}
```

- continue Outer도 똑같이 먹힌다.

```java
Outer:
	for(int i = 0; i < 5; i++){
		System.out.println("(1) square");
		System.out.println("(2) square root");
		System.out.println("(3) log ");
		System.out.println("원하는 메뉴 선택");

		String tmp = scanner.nextLine();
		int menu = Integer.parseInt(tmp);
		if(menu==0){
			System.out.println("프로그램 종료");
			break;
		}else if(!(1<=menu && menu <=3)){
			System.out.println("메뉴를 잘못 선택했습니다. 종료는 0");
			continue;
		}

		for(;;){
			System.out.println("계산 값 입력. (계산 종료: 00, 전체종료:99)");
			String temp = scanner.nextLine();
			int num = Integer.parseInt(temp);

			if(num==0){
				break; // 해당 if문을 감싼 for문을 벗어남.
			}
			if(num==99){
				continue Outer; // 해당 if문을 감싼 for문이 아니라 outer라는 이름을 가진 반복문 전체를 벗어남.
			}

			switch(menu){
				case 1:
					System.out.println("result = " + num * num);
					break; //switch문에서의 break;
				case 2:
					System.out.println("result = " + Math.sqrt(num));
					break; //switch문 끝나고서 진행되는 소스코드있으면 다 진행됨
				case 3:
					System.out.println("result = " + Math.log(num));
					break;
			}
		}
	}
```

## <span style="color:#802548">_3.parameter_</span>

- 함수의 파라미터로 다양한 것이 올 수 있다.
- 그 중에 가변인자는 `맨 마지막`에 와야 한다.

```java
void getSth(String a, int b, MyClass... b);
```

- 가변인자는 다양한 것을 한꺼번에 나타낼 수 있다.

```java
void getSth(String... str);

void getSth(String a);
void getSth(String a, String b);
void getSth(String[] a);
void getSth(null);
void getSth();
```

- interface와 extends한 class 혹은 abstract class를 넣는 경우도 있다.
- 그 경우 해당 interface를 `implements`한 것, 혹은 해당 class를 `extends`한 것을 넣어주면 된다.

```java
UsernamePasswordAuthenticationToken authenticationToken =
		            new UsernamePasswordAuthenticationToken(loginDto.getUserId(), loginDto.getUserPassword());
authenticationManagerBuilder.getObject().authenticate(authenticationToken); //authenticate()의 parameter로 Authentication interface를 implements한 class를 넣는다. 아래와 같다.

Authentication authenticate(Authentication authentication)
			throws AuthenticationException; // Authentication interface를 parameter로 받는다.

public class UsernamePasswordAuthenticationToken extends AbstractAuthenticationToken
public abstract class AbstractAuthenticationToken implements Authentication,
		CredentialsContainer // AbstractAuthenticationToken이 Authentication을 implements하기 때문에 UsernamePasswordAuthenticationToken도 parameter로 들어갈 수 있다.

List<String> strList = new ArrayList<>();
strList.add("bb");
strList.remove("bb");

public boolean remove(Object o) // Object가 parameter라면 모든 class가 들어갈 수 있다. 모든 class는 Object의 자식이다.

```

## <span style="color:#802548">_4.초기화_</span>

- 명시적 초기화 -> 클래스 블록 초기화 -> 인스턴스 블록 초기화 -> 생성자 초기화순이다.
- method를 호출하지 않는 이상 `생성자`에서 초기화한 게 최종이다.

```java
class MyClass{
	String a = "a";
	static int b = 1;

	static {//instance 변수는 static block에서 할당 불가능
		b = 2;
	}
	{//static 변수는 instance block에서 할당 불가능
		a = "b";
	}

	MyClass(String a){ // static은 생성자에서 할당 불가능
		a = "c";
	}
}
```

## <span style="color:#802548">_5.참조변수_</span>

- interface나 abstract class는 자신의 자식을 instance로 생성한다.
- 그런데 자식은 보통 더 많은 method나 멤버변수를 가진다.
- 만약 자식class가 참조변수가 아니라 해당 interface나 abstract class를 참조변수로 가진다면, 해당 method는 사용 불가능하다.

```java
List<String> abc = new ArrayList<>();
abc.get(0); // list interface에도 규정된 method를 arrayList가 implements한 것이라 사용 가능. 즉 override한 것.
abc.grow() //ERROR. override하지 않고 자식인 ArrayList에서 자체로 만든 것이기 떄문
```

- 위와 마찬가지 이유로 형변환 시 downcasting에는 `명시적` 형변환만 가능하다.

```java
String abc = (String)map.get("abc"); //get의 return type이 Object이므로 downcasting
```

## <span style="color:#802548">_6.예외처리_</span>

- 아래와 같이 try ~ catch 문을 사용할 수 있다.
- resources를 IO하는 경우에는 아래와 같이 try() ~ catch를 사용하며, 그 경우 close가 필요없다.

```java
try{
	FileInputStream fis = new FileInputStream("abc.txt");
	fis.read();
	fis.close();
}catch(Exception e){

}catch(IOException e){

}

try(FileInputStream fis = new FileInputStream("abc.txt");){
	fis.read();
}catch(Exception e){

}catch(IOException e){

}
```

- Dao에서 SqlException을 던진다
- Service에서는 SqlException catch가 없다. 대신 Exception catch가 있다.
- SqlException의 상위가 Exception이므로 잡아낼 수 있다.
- 또한 throws Exception이기 때문에 catch를 SqlException으로 해도 Exception을 `상위 호출` method에 던진다.

```java
class Dao{
	MenuVO selectItem() throws SqlException{
		sqlSession.selectOne();
	};
}

class Service{
	MenuVO getItem() throws Exception{
		Dao dao = new Dao();
		try{
			dao.selectItem();
		}catch(Exception e){ //Exception이 먼저오면 다 여기로 먼저 빠지기 때문에 늘 마지막에 Exception을 catch한다.
			logService.writerLog("Service Exception");
			throw e
		}catch(SqlException e){
			logService.writerLog("Service Exception");
			throw e
		}
	}
}

class Service{
	MenuVO getItem() throws Exception{
		try{
			Dao dao = new Dao();
			dao.selectItem(); // dao의 selectItem()이 throws Exception이기 때문에 try ~ catch로 처리하거나, throws Exception으로 처리가 필요하다.
		}catch(SqlException e){
			logService.writerLog("Service Exception");
			throw e
		}catch(Exception e){
			logService.writerLog("Service Exception");
			throw e
		}
	}
}

class controller{
	@GetMapping
	public String menuItem(){
		try{
			MenuVO vo = Service.getItem(); // service의 getItem()이 throws Exception이기 때문에 try ~ catch로 처리해야한다. throws Exception은 controller에서는 엔간하면 만들지 않는다.

		}catch(Exception e){
			logService.writerLog("Controller Exception");
			throw e;
		}
	}
}
```

- 고의로 Exception을 낼 수도 있다.
- 주로 business logic에서 필요할 때 사용한다.
- 아래와 같이 checked를 unchecked Exception인 RuntimeException으로 감싸면 unchecked처럼 처리도 가능하다.
- 참고로 기능을 구현하지 않을 override의 경우 아래와 throw new Exception으로 처리한다.

```java
if (readDto == null || !passwordEncoder.matches(loginDto.getUserPassword(), readDto.getUserPassword())) {
	throw new ServiceException(CommonErrorCode.AUTH_BAD_CREDENTIALS.getMessage());
}

if(true){
	throw new RuntimeException(new Exception());
}


public class ArrayList<E>
@Override
E get(){
	throw new UnsupportedException();
};
```

## <span style="color:#802548">_7.주요 class_</span>

- Object class는 모든 class가 조상으로 갖는 class다.
- 주로 쓰는 것은 equals, hashCode, toString이다. 보통 각 class에서 자신에 맞게 바꿔쓴다.
- 바꾸지 않는다면 모두 객체의 주소값을 비교하거나, 주소값을 return하고 주소값을 문자열로 반환한다.

- List interface에서 주로 쓰는 것은 add, get, contains, isEmpty, remove, clear, size 등이다.

```java
List<String> strList = new ArrayList<>();
if(strList.isEmpty()){
	strList.add("b");
}

String b = strList.get(0); //list에서 get은 map과 다르게 index로만 가져올 수 있다.
if(strList.contains(b)){
	strList.remove("b"); //list에서 remove는 element를 직접 넣어줘도 된다. Object를 parameter로 받기 때문이다.
	//strList.remove(0); index로도 삭제 가능
}

Collections.emptyList(); // 빈 list를 만들 때 쓰는 방법. new ArrayList<>();보다 간단.
//js에서는 const list = [];로 간단하게 만들 수 있다.

List<String> abc = new ArrayList<>();
abc.add("1");
abc.add("2");
abc.add("3");

//아래와 같이 하면 cursor가 자동이동되지 않기 때문에 무조건 오류가 난다.
for(int i = 0; i < abc.size(); i++){
	if(abc.get(0) == "1"){
		abc.remove() //IndexOufOfRange
	}
}

//아래와 같이 위에서부터 거꾸로 거슬러 올라가면 cursor가 이동하기에 오류가 나지 않는다.
for(int i = abc.size()-1; i >=0; i--){
	if(abc.get(0) == "1"){
		abc.remove()
	}
}
```

- String class에서 주로 쓰는 것은 equals, length, format, lastIndexOf, subString, trim, replace

```java
if(type.equals("1")) // type == "1"; 이러면 bug가 날 것이다

String hello = "Hello";
hello.length() //5

String.format("%02d", 1) // 01.  %d면 2번째 parameter로 int type이 들어와야 한다.

String abc = "cad@naver.com";
String BeforeAt = abc.subString(0, abc.lastIndexOf("@")); // cad
String AfterAt = abc.subString(abc.lastIndexOf("@")+1);// naver.com

String abc = "    ";
abc.trim();
if(abc.isBlank()) // true

String abc = "dr";
abc = abc.replace("d", "a"); //ar
//replaceAll은 특수문자로 치환이 어려움.
```

- Map interface에서 주로 쓰이는 것은 get, put, putAll, remove, clear다.

```java
Map<String,Object> map = new HashMap<>();

map.put("number", 5);
String abc = (long)map.get("number"); // 숫자를 map에 넣으면 int가 아닌 long으로 형변환

MaP<String,Object> inputMap = new HashMap<>();
mpa.put("list", Collections.emptyList());
map.putAll(inputMap);

if(map.containsKey("list")){
	map.remove("list");
}
map.clear();
//근데 보통 map은 debugger로 찍는다.
for (Map.Entry<String, String> entry : map.entrySet()) { // Map 내부에 규정된 inner interface Entry를 가져온다.
	System.out.println("[key]:" + entry.getKey() + ", [value]:" + entry.getValue());
}
```

- Arrays class에서 주로 쓰이는 것은 asList, sort, toString, stream이다.
- toString은 해당 class에 구현된 것을 보고, 없으면 Object의 toString()을 가져온다. 즉 주소값이 찍힌다.

```java
String[] strArr = {"2","1"};
List<String> strList = Arrays.asList(strArr);
Arrays.sort(strList); //error. sort()의 parameter는 정수, 실수형이 들어가야 함.

class MenuVo{
	private String menu;

	MenuVo(String menu){
		this.menu = menu;
	}
}

MenuVo[] menuVoArr = {new MenuVo("12"), new MenuVo("34")};
String str = Arrays.toString(menuVoArr);
System.out.println(str); //[spring.aop.Program$1MenuVo@668bc3d5, spring.aop.Program$1MenuVo@3cda1055]
System.out.println(Arrays.toString(strArr)); // [2,1]

class MenuVo{
	private String menu;

	MenuVo(String menu){
		this.menu = menu;
	}

	@override
	public String toString() { //toString을 재정의
		return menu;
	}
}

System.out.println(str); // [12,34]
```

- Collections class에서 주로 쓰이는 것은 sort와 emptyList다.

```java
List<String> abc = Collections.emptyList();

Collections.sort(list, new Comparator(){
	@Override
	int compare(String s1, String s2){
		return s2.compareTo(s1);
	}
})
```

## <span style="color:#802548">_8. generics_</span>

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

- 위에 말했듯 컴파일 시에 타입을 알아야 한다.
- 따라서 배열에 제너릭을 줄 수 없고, static에도 제너릭을 줄 수 없다.
- generics는 instance를 기준으로 작동한다. static은 T라는 instance 변수와 공존할 수 없다.
- 배열 또한 불가능하다. new로 생성을 할 때 T가 어떤 타입일지 알 수가 없기 때문이다.

```java
class Box<T>{
    static T item; //error
    static int compare //error
    T[] itemArr;

    T[] toArray(){
        T[] temArr = new T[itemArr.length]; //불가능. new를 썼기 때문.
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
List<Apple> list = new ArrayList<>();
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
static Juice makeJuice(FruitBox<? extends Fruit> box) 		//FruitBox class에 들어갈 type은 Fruit class를 extends한 class만 가능하다.
static <T extends Fruit> Juice makeJuice(FruitBox<T> box)	//위와 동일
```

- generics method를 호출할 때는 아래와 같다.

```java
FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
FruitBox<Apple> appleBox = new FruitBox<Apple>();

sysout(Juicer.<Fruit>makeJuice(fruitBox)); 		//거의 이렇게는 안 쓴다.
sysout(Juicer.<Apple>makeJuice(appleBox));	 	//거의 이렇게는 안 쓴다.
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

## <span style="color:#802548">_9. enum_</span>

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
```

- enum을 쓰면 아래와 같이 바뀐다.

```java
@ReArgsConstructor
class Card{
    enum Kind { CLOVER, HEART, DIAMOND, SPADE}
    enum value { TWO, THREE, FOUR}
}
```

- 하지만 보통 아래와 같이 enum을 따로 만드는 편이다.

```java
public enum Suit {
    CLOVER,
    HEART,
    DIAMOND,
    SPADE
}

public enum Rank {
    TWO,
    THREE,
    FOUR
}

public class Card {
    final Suit suit;
    final Rank rank;

    public Card(Suit suit, Rank rank) {
        this.suit = suit;
        this.rank = rank;
    }
}
```

- enum에 새로운 member를 추가하는 것도 가능하다.

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
		@Override
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    TRAIN(150){
		@Override
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    SHIP(100){
		@Override
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    AIRPLANE(300){
		@Override
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
System.out.println("bus fare =  " + Transportation.BUS.fare(100)); //100 * 100. Transportation.BUS(100).fare(100);이 아닌 이유는 전자의 100은 enum class가 생성될 때 이미 들어가지기 때문.
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

## <span style="color:#802548">_10. IOStream_</span>

- OutputStream의 예시는 아래와 같다.
- 거의 대부분 BufferedStream을 불러서 사용한다. 그게 효율적이기 때문이다.
- try ~ catch 혹은 try ~ with ~ resources 혹은 throws Exception이 필수다.

```java
void transferFile(){
    byte[] decodedBytes = Base64.getUrlDecoder().decode(insertDto.getFile().substring(insertDto.getFile().lastIndexOf(",") +1)
                                            .replace('+', '-').replace('/', '_'));
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 4; i++) { // Consider first 4 bytes as file signature
        sb.append(String.format("%02X", decodedBytes[i]));
    }

    try{
        File file = new File("/upload/" + fileName + fileExtension);
        FileOutputStream fileOutputStream = new FileOutputStream(file);
        BufferedOutputStream bos = new BufferedOutputStream();
        bos.write(decodedBytes);

        bos.close();
    }catch(IOException e){
        throw e;
    }
}

void transferFile(){
    byte[] decodedBytes = Base64.getUrlDecoder().decode(insertDto.getFile().substring(insertDto.getFile().lastIndexOf(",") +1)
                                            .replace('+', '-').replace('/', '_'));
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 4; i++) { // Consider first 4 bytes as file signature
        sb.append(String.format("%02X", decodedBytes[i]));
    }

        File file = new File("/upload/" + fileName + fileExtension);
    try(FileOutputStream fileOutputStream = new FileOutputStream(file);
        BufferedOutputStream bos = new BufferedOutputStream();){

        bos.write(decodedBytes);
    }catch(IOException e){
        throw e;
    }
}

void transferFile() throws IOException{
    byte[] decodedBytes = Base64.getUrlDecoder().decode(insertDto.getFile().substring(insertDto.getFile().lastIndexOf(",") +1)
                                            .replace('+', '-').replace('/', '_'));
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 4; i++) { // Consider first 4 bytes as file signature
        sb.append(String.format("%02X", decodedBytes[i]));
    }
    File file = new File("/upload/" + fileName + fileExtension);
    FileOutputStream fileOutputStream = new FileOutputStream(file);
    BufferedOutputStream bos = new BufferedOutputStream();
    bos.write(decodedBytes);

    bos.close();
}
```

- input을 output으로 새로운 파일을 만드는 예제다.

```java
try{
    FileInputStream fis = new FileInputStream("123.txt");
    BufferedInputStream bis = new BufferedInputStream();
    FileOutputStream fos = new FileOutputStream("456.txt");
    BufferedOutputStream bos = new BufferedOutputStream();
    int data = 0;
    while((data = bis.read()) != -1){
        bos.write(data); //123.txt의 내용이 그대로 456.txt라는 새 파일에 들어간다.
    }

    bis.close();
}catch(IOException e){
    throw e;
}
```

## <span style="color:#802548">_11. Stream_</span>

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

- 다만 그냥 interface로는 람다식을 쓸 수 없다.
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
Stream<String> stream = Stream.of(strArr);

List<String> strList = Arrays.asList(strArr); //리스트
Stream<String> strStream2 = strList.stream();
Stream<List<String>> stream2 = Stream.of(strList);
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

```java
 Stream<String[]> strArrStream = Stream.of(
		            new String[]{"abc","def","jkl"},
		            new String[]{"ABC","DEF","JKL"}
		        );

		 Stream<String[]> strArrStrea1 = Stream.of(
		            new String[]{"abc","def","jkl"},
		            new String[]{"ABC","DEF","JKL"}
		        );

		        Stream<Stream<String>> strStrmStrm = strArrStrea1.map(Arrays::stream); // Stream 안의 Stream이 존재. 이 상태에서 forEach해도 값이 제대로 안 뽑힘.
		        Stream<String> strStrm = strArrStream.flatMap(Arrays::stream);  //Stream 안의 Stream을 전부 합쳐서 동일한 차원에 설정
```

- 중간연산으로 주로 사용되는 것들은 아래와 같다.

```
map()
sorted()
filter()
```

- 최종연산으로 주로 사용되는 것들은 아래와 같다.

```
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

- map이외에 flatMap이라는 method도 있다.
- map과 다르게 flat이 붙어있는데, 이는 차원을 평평하게 하는 것이다.
- 아래와 같이 String[]을 stream으로 생성하면 Stream<String>이 아니라 Stream<Stream<String>>이 되어버린다.
- 그럴 때 flatMap을 써서 동일한 차원에 있게 만들어준다.

```java
class StreamEx4{
    public static void main(String[] args){
        Stream<String[]> strArrStream = Stream.of(
            new String[]{"abc","def","jkl"},
            new String[]{"ABC","DEF","JKL"}
        );

        Stream<Stream<String>> strStrmStrm = strArrStream.map(Arrays::stream); // Stream 안의 Stream이 존재. 이 상태에서 forEach해도 값이 제대로 안 뽑힘.
        Stream<String> strStrm = strArrStream.flatMap(Arrays::stream);  //Stream 안의 Stream을 전부 합쳐서 동일한 차원에 설정

        strStrm.map(String::toLowerCase) //그 뒤에 forEach를 해야 원하는 대로 요소들이 뽑혀서 출력됨.
                .distinct()
                .sorted()
                .forEach(System.out::println);
        System.out.println();

        String[] lineStream = {
            "Believe or not It is true",
            "Do or do not There is no try",
        };

        Stream<String> lineStream = Arrays.stream(lineArr);
        lineStream.flatMap(line -> stream.of(line.split(" +")))
                    .map(String::toLowerCase)
                    .distinct()
                    .sorted()
                    .forEach(System.out::println);
        System.out.println();

        Stream<String> strStrm1 = Stream.of("AAA","BBB","CCC","DDD"); //of의 매개변수는 가변인자. 따라서 저렇게 String을 여러개 집어넣으면 배열로 취급. Stream<String> strStrm1 = Stream.of(new String[]{"AAA","BBB","CCC","DDD"});과 동일.
        Stream<String> strStrm2 = Stream.of("bbb","aaa","ccc","dd");

        Stream<Stream<String>> strStrmStrm = Stream.of(strStrm1, strStrm2);
        Stream<String> strStream = strStrmStrm.map(s -> s.toArray(String[]::new))
                                                .flatMap(Arrays::stream);

        strStream.map(String::toLowerCase)
                    .distinct()
                    .forEach(System.out::println);
                    // bbb
                    // aaa
                    // bbb
                    // ccc
                    // ddd
                    // dd
    }
}
```

- collect()는 Stream에서 가장 많이 쓰이는 method다.

```java
List<String> names = stuStream.map(Student::getName).collect(Collectors.toList());
ArrayList<String> list = names.Stream().collect(Collectors.toCollection(ArrayList::new));
Map<String,Person> map = PersonStream.collect(Collections.toMap(p->p.getRegId(), p->P));
```

- stream을 두 개 그룹으로 나눌 때는 groupingBy()로 쓰고 그 이상의 그룹으로 나눌 때는 partioningBy();를 사용한다.

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

Map<Student.Level, Long> stuByLevel = stuStream.collect(groupingBy(s->{
    if(s.getScore() >= 200)
        return Student.Level.HIGH;
    else if(s.getScore() >= 100)
        return Student.Level.MID;
    else
        return Student.Level.LOW;
},counting())) //[MID] - 8명, [HIGH] - 8명, [LOW] - 2명

Map<Integer,Map<Integer,List<Student>>> stuByHakAndBan = stuStream.collect(
				Collectors.groupingBy(Student::getHak,
						Collectors.groupingBy(Student::getBan))); //학년별로 그룹화 후 반별로 다시 그룹화
Map<Integer, Map<Integer,Student>> topStuByHakAndBan = stuStream.collect(groupingBy(Student::getHak, groupingBy(Student::getBan, collectingAndThen(maxBy(comparingInt(Student::getScore),Optional::get))))) //각 반의 1등을 출력
Map<Integer, Map<Integer,Set<Student.Level>>> stuByHakAndBan = stuStream.collect(GroupingBy(Student::getHak,groupingBy(Student::getBan,mapping(s->{
    if(s.getScore() >= 200)
        return Student.Level.HIGH;
    else if(s.getScore() >= 100)
        return Student.Level.MID;
    else
        return Student.Level.LOW;
}, toSet()))))
```

- 물론 위에처럼 쓰면 안되고, 아래처럼 써야한다.
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

## <span style="color:#802548">_12. Optional_</span>

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
String optionalStr = abc.orElseGet("bbb"); //불가능. 여기는 반드시 람다식이 들어가야 함. 람다식이 ()->new String()일수도 있고, 매써드 참조형태의 String::new일 수도 있음.
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
    sysout(str);
} //기존

if(Optional.ofNullalbe(str).isPresent()){
    sysout(str);
} //Optional 객체를 활용한 형태

Optional.ofNullable(str).ifPresent(System.out::println); // Optional 객체와 lambda식을 활용한 형태. 만약 값이 있다면 sysout을, 값이 없다면
```

- stream의 연산에도 아래와 같이 Optional 객체를 사용할 수 있다.
- 만약에 stuStream에 값이 없으면 빈 Optional 객체를 반환한다. 빈 Optional 객체는 안에 null을 저장해두고 있음.

```java
Stream<Student> stuStream = Stream.empty();
boolean noFailed = stuStream.anyMatch(s->s.getTotalScore() <= 100);
Optional<Student> stu = stuStream.filter(s->s.getTotalScore() <= 100).findFirst();
```

- 그냥 stream class를 이용한 경우는 아래와 같이 int type으로 타입을 지정한다. Stream<Integer>

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

## <span style="color:#802548">_13. network_</span>

- url을 만들어 거기에 있는 정보를 가져올 때는 URL instance를 활용한다.
- stream을 열어야 하는데, 그것은 openStream()으로 가능하다.

```java
public class NetWorkEx4 {
    public static void main(String[] args) {
        URL url = null;
        BufferedReader input = null;
        String address = "http://www.codechobo.com/sample.hello.html";
        String line = "";

        try{
            url = new URL(address);
            input = new BufferedReader(new InputStreamReader(url.openStream()));

            while( (line = input.readLine()) != null) {
                sysout(line);
            }
            input.close();
        } catch(Exceptione e) {
            e.printStackTrace();
        }
    }
}
```

- try catch보다는 try with resources로 바꿔주자.

```java
 try( BufferedReader input = new BufferedReader(new InputStreamReader(new URL(address).openStream())) ){
    while( (line = input.readLine()) != null) {
        sysout(line);
    }
}
```

- Spring을 쓴다면 WebClinet class를 쓸 수도 있다.

```java
public class WebClientExample {
    public static void main(String[] args) {
        String url = "https://example.com";

        WebClient webClient = WebClient.create();
        String responseBody = webClient.get()
                .uri(url)
                .retrieve()
                .bodyToMono(String.class)
                .block();

        System.out.println(responseBody);
    }
}
```

- URL은 인터넷 상의 주소여야 한다.
- 하지만 그걸 출력할 Stream은 반드시 문자열일 필요가 없다.
- 아래와 같이 파일로 만들 수도 있다.

```java
public class NetWorkEx5 {
    public static void main(String[] args) {
        URL url = null;
        InputStream out = null;
        FileOutputStream out = null;
        String address = "http://www.codechobo.com/book/src/javajungsuk3_src.zip";

        int ch = 0;

        try(in = new URL(address).openStream(); out = new FileOutputStream("javajungsuk3_src.zip")) {
            while( (ch = in.read()) != -1) {
                out.write(ch);
            }
        }
    }
}
```

- WebClient로도 충분히 파일을 만들 수 있다.

```java
public class FileDownloadExample {

    public static void main(String[] args) {
        String fileUrl = "https://example.com/path/to/file.zip";

        WebClient webClient = WebClient.create();

        webClient.get()
                .uri(fileUrl)
                .retrieve()
                .bodyToFlux(byte[].class)
                .doOnNext(FileDownloadExample::saveToFile)
                .blockLast(); // block to wait for completion, you might want to handle it differently in a real application
    }

    private static void saveToFile(byte[] content) {
        Path destination = Paths.get("local/path/to/save/file.zip");

        try (FileOutputStream fos = new FileOutputStream(destination.toFile())) {   //Path를 지정해준다.
            fos.write(content);                                                     //content는 위의 bodyToFlux(byte[].class)에서 가져온 것이다.
            System.out.println("File downloaded and saved to: " + destination);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

- RestTemplate으로도 가능하다. 하지만 webClient가 추천된다.
- webClient는 비동기도 지원되지만, RestTemplate은 동기만 가능하다.
- 하지만 RestTemplate은 HttpConnection(url.openConnection)을 추상화한 것 뿐이라 성능 문제는 똑같다.
- 엔간하면 webClient를 쓰자. https://velog.io/@jmjmjmz732002/Spring-%EC%99%B8%EB%B6%80-API-%ED%86%B5%EC%8B%A0-%EB%A6%AC%ED%8C%A9%ED%86%A0%EB%A7%81%EC%9D%84-%ED%95%98%EA%B2%8C-%EB%90%9C-%EC%9D%B4%EC%9C%A0 참조

- 다만 % 인코딩의 문제가 있으니 만약 문제가 일어난다면, 아래와 같이 encodingmode를 바꿔줘야 한다.

```java
DefaultUriBuilderFactory factory = new DefaultUriBuilderFactory();
factory.setEncodingMode(DefaultUriBuilderFactory.EncodingMode.NONE);

 // baseUrl e.g. https://apis.data.go.kr
 WebClient webClient = WebClient.builder()
 				.uriBuilderFactory(factory)
                .baseUrl(baseUrl)
                .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                .build();

// serviceKey : {SERVICE_KEY_HERE}
String result = webClient.get()
					.uri(uriBuilder -> uriBuilder
                    			.path("/B551011/KorService1/locationBasedList1")
    							.queryParam("serviceKey", serviceKey)
                        		.build())
                        .retrieve()
                        .bodyToMono(String.class)
                        .block();
```

- Webclient가 채택한 리액티브 패턴은 전염성이 있어서 최적의 성능을 내려면 처음부터 끝까지 모든 구간을 비동기로 짜야한다.
- 컨트롤러의 return 타입이 Mono나 Flux 타입이 되야한다.
- 어디선가 block을 한번 호출하면 그 시점부터 비동기의 이점이 없어지기 때문에 별 차이가 없는 것처럼 느껴질 수 있다.
- Mono와 Flux의 차이는 Mono는 말그대로 one이고, Flux는 다수라는 차이다. 물론 둘다 요소가 없을 수도 있다.
```java
 @GetMapping("/data")
    public Flux<String> getData() {
        return webClient.get()
                        .uri("/api/data")
                        .accept(MediaType.APPLICATION_JSON)
                        .retrieve()
                        .bodyToFlux(String.class);
    }

//

@RestController
public class MyController {

    @GetMapping("/data")
    public Flux<String> getData() {
        // Simulate streaming data
        return Flux.just("Data 1", "Data 2", "Data 3");
    }
}
```

- Mono는 아래와 같다.
```java
@RestController
public class MyController {

    @Autowired
    private WebClient webClient;

    @GetMapping("/data")
    public Mono<String> getData() {
        return webClient.get()
                        .uri("/api/data")
                        .accept(MediaType.APPLICATION_JSON)
                        .retrieve()
                        .bodyToMono(String.class);
    }
}
```

- front에서도 기존과 동일한 방식으로 받으면 된다.
```js
axios.get('/data')
  .then(response => {
    const data = response.data;
    console.log("Received data:", data);
  })
  .catch(error => {
    console.error("Error:", error);
  });
```

- 다만 Flux를 ServerSentEvent로 사용할 경우, axios나 fetch 같은 전통 방식을 활용할 수가 없다.
```java
@RestController
public class MyController {

    @GetMapping(value = "/data", produces = "text/event-stream")
    public Flux<ServerSentEvent<String>> getData() {
        // Simulate streaming data
        return Flux.interval(Duration.ofSeconds(1))
                   .map(sequence -> ServerSentEvent.<String>builder()
                           .id(String.valueOf(sequence))
                           .data("Data " + sequence)
                           .build());
    }
}
```

- 위와 같이 SSE인 경우, 어쩔수없이 다른 방식으로 front에서 받아야 한다.
- streaming을 열어서 받아야 한다는 의미다.
- EventSource는 ES5에서도 사용가능하다.
```js
const eventSource = new EventSource("/data");

eventSource.onmessage = function(event) {
    const data = JSON.parse(event.data);
    console.log("Received data:", data);
};

eventSource.onerror = function(error) {
    console.error("EventSource error:", error);
};
```