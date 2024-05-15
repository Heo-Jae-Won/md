## <span style="color:#802548">_try_catch란?_</span>

- 에러에는 컴파일 에러, 런타임 에러, 논리적 에러가 존재한다.
- 컴파일 에러는 compile 시에 바로 알아차릴 수 있는 에러다. IDE가 탐지해준다.
- runtime 에러는 compile 시에는 알 수 없고 runtime시에만 알 수 있다. IDE가 탐지할 수 없다.
- 논리적 에러는 언어 상의 에러는 아니지만, 원하는 결과값을 얻지 못하는 경우다.

- 에러처리는 런타임 에러에 대해서만 가능하다.
- 런타임에러는 Error(처리 불가능)과 Exception(처리 가능)으로 나뉜다.
- Error의 대표적인 예가 stackOverFlow와 outOfMemory다. 이 둘은 에러처리가 불가능하다. 언어 단에서 해결이 불가능하다.
- 반면에 Exception은 언어 단에서 처리가 가능하다.
- Exception은 또 반드시 처리해줘야 하는 unchecked와 처리가 선택인 checked로 나뉜다.
- Exception을 처리하는 방식은 2가지가 있다. 먼저 try ~ catch다.

```java
try{
    float b = 5.0;
    int a = 10/b;

}catch(Exception e){
    e.getMessage();
}

String a = "속행점";
sysout("a: " + a);
```

- 위와 같이 정수를 실수로 나누면 AriteMeticException이 나게 된다.
- 그럼 try문의 진행을 멈추고 catch문으로 들어가게되며, 그 뒤로 다시 소스코드가 진행된다.
- 아래와 같이 multi catch문도 가능하다. ArithMeticException이 발생했을 때는 첫째 catch문에 들어가간다.
- 그 외는 전부 둘째 catch문에 들어가게 된다.

```java
try{
    float b = 5.0;
    int a = 10/b;

}catch(ArithmeticException ae){
    ae.getMessage();
}catch(Exception e){
    e.getMessage();
}

String a = "속행점";
sysout("a: " + a);
```

- 실제 Spring에서 쓰는 사례를 들어보자.

```java
@RequestMapping("/url")
public ModelAndView menu(){

    @Autowired
    MenuService menuService;

    ModelAndView mav = new ModelAndView("menu");
    try{
        outputMap = menuService.getMenu();
        mav.addAllObjects(outputMap);

    }catch(Exception e){
        e.getMessage();
    }

    return mav;

}
```

- 만약 Service에서 오류가 난다면 model에 object가 없이 화면이 이동된다.
- 따라서 data가 load되지 못하고 빈 화면으로 노출되게 되는 것이다.
- 그럼 exception이 어디서 넘어온걸까?

## <span style="color:#802548">_throws란?_</span>

- 바로 throws Exception에서 넘어온 것이다.
- method header에 throws를 달면 해당 Exception은 자신을 호출한 method에게 예외처리를 떠넘기게 된다.
- 즉 controller menu()에게 예외처리의 책임을 넘긴 것이다.
- 그래서 menu()에서는 try ~ catch를 사용해서 예외처리를 해줘야만 한다.

```java
public class MenuService{

    public String getMenu() throws Exception{
        ///메뉴를 얻어온다.
    }
}
```

- method header에서 throws Exception을 던지는 것을 배웠다.
- 하지만 이는 블록 내에서 쓰이는 throw new Exception과는 다른 것이다.
- throw new Exception은 책임을 떠넘기는 게 아니라 exception을 고의로 발생시키는 것이다.
- 아래의 경우에는 method header에 throws가 없기 때문에 상위 method로 exception이 전파되지 않는다.
- throws의 경우 runtimeException에서는 필수가 아니다. runtimeException은 에러처리가 필수가 아니기 때문이다. 그래서 Exception class에만 쓰는 경우가 흔하다.


```java
public class MenuService{
    public String getMenu(){
        try{
            new Exception("고의 발생");
        }catch(Exception e){
            sysout("고의 발생함");
        }
    }
}
```

- 하지만 아래와 같이 method header에 throws도 선언했다. 
- 즉 try ~ catch와 throws가 모두 있는 경우다. 이 경우 어떻게 될까?
- 간단하다. 해당 method에서도 에러처리가 일어나고, 상위 method에서도 에러처리가 일어난다.

```java 
public class MenuService throws Exception{
    public String getMenu(){
        try{
            new Exception("고의 발생");
        }catch(Exception e){
            sysout("고의 발생함");
        }
    }
}
```

- 이경우는 보통 Spring에서 aop를 사용하지 않고 log를 남길 때 이런 식으로 많이 쓴다.
- 그럼 Service에서도 로그가 남고, Controller에서도 로그가 남는다.
- 한 Service를 다양한 곳에서 호출하기 때문에 Service와 Controller 모두에서 log를 남기는 것이다.


```java 
public class MenuService throws Exception{
    public String getMenu(){
        try{
            if(/*특정조건*/){
                new Exception("에러 발생");
            }
        }catch(Exception e){
            int logNumber = 난수;
            logService.transferLog(logNumber);
        }
    }
}

@RequestMapping("/url")
public ModelAndView menu(){

    @Autowired
    MenuService menuService;

    ModelAndView mav = new ModelAndView("menu");
    try{
        outputMap = menuService.getMenu();
        mav.addAllObjects(outputMap);

    }catch(Exception e){
        e.getMessage();
        int logNumber = 난수;
        logService.transferLog(logNumber);
    }

    return mav;

}
```

- throw를 통해 checked를 unchecked로 바꿀 수도 있다.
- checked의 경우 throws 혹은 try ~ catch가 필요하다.


```java
public class MyException extends Exception{
    super(String msg);
}
.
.
public class MenuService{
    public void getMenu(){
        try{
            throw new MyException(); //checked인 Exception class로 예외를 던졌기 때문에 예외처리가 반드시 필요하다.
        }catch(Exception e){
            sysout("예외처리 필수 checked");
        }
    }
}
```
- 아래처럼 runtimeException으로 덮어씌우면 unchecked가 된다.
- 이 경우 예외처리가 필수가 아니라 구현하지 않아도 된다.


```java
public class MenuService{
    public void getMenu(){
        throw new RuntimeException(MyException("")); //RuntimeException으로 덮어씌워 checked를 unchecked로 바꿨다.
    }
}
```

- finally는 exception이 던져짐에도 해당 로직 진행이 필요할 때 사용된다.
- finally는 쓸지 말지 선택이다.


```java
public class MenuService throws Exception{
    public String getMenu(){
        try{
            if(/*특정조건*/){
                new Exception("에러 발생");
            }
        }catch(Exception e){
            int logNumber = 난수;
            logService.transferLog(logNumber);
        }
    }
}
```

## <span style="color:#802548">_try ~ with ~ resources란?_</span>

- 입출력 관련 클래스를 대상으로 사용되는 try ~ catch문이다.
- AutoCloseable interface를 구현한 class만 사용가능한 타게팅 try ~ catch문이다.
- FileInputStream을 예로 들어보자.
- FileInputStream은 inputStream을 extends한다.
- InputStream은 Closeable interface를 implements한다.
- Closeable interface는 Autocloseable interface를 extends한다.
- 따라서 FileInputStream은 Autoclosealbe의 instance인 것이다.
- InputStream과 OutputStream을 extends한 모든 class가 try ~ catch ~ resources를 사용할 수 있다.

- 아래는 한개의 입출력만 사용했을 때이다.

```java
try(FileInputStream fis = new FileInputStream("txt.txt"))
```

- 만약 두개를 사용한다면 ;로 이어준다.
- 이렇게 쓰면 fis.close()를 따로 굳이 해주지 않아도 된다.
- Stream이 알아서 닫히게 되는 것이다.

```java
try(FileInputStream fis = new FileInputStream("txt.txt");
    DataInputStream dis = new DataInputStream(fis)){

}catch(Exception e){

}
```

## <span style="color:#802548">_java의 Object class란?_</span>
- 모든 class는 Object class를 extends한 것이다.
- Object class에는 hashcode(), equals(), toString(), clone() method가 있다. 
- hashcode()는 객체 주소값으로 해시코드를 만들어서 반환한다. 64bit JVM에선 중복 되는 해쉬코드가 나올 수도 있다. 따라서 해쉬코드가 같다고 객체 주소값이 같다고 생각하면 안된다.
- 객체주소값을 완전하게 유일하게 가져가는 hashcode는 System.identityHashcode()다.
- 그렇다면 hashcode()는 무엇을 위한 용도일까? 바로 똑같은 객체라면 hashcode()를 했을 때 똑같은 값을 반환하게 하는 것이다.
- equals도 마찬가지로 객체의 주소값이 같다면 같은 객체로 판단한다

- 하지만 각 class마다 hashcode와 equals를 자신의 목적에 따라 override해서 사용하기도 한다.
- String class는 객체의 주소값이 아니라 문자열이 같은지를 비교하기 위해 equals를 overriding했다.
- 그에 따라 hashcode()도 같이 override되어있다. 즉 문자열이 같으면 hashcode도 같은 값을 지닌다. 그것이 다른 객체라고 해도 말이다.
- 해당 hashcode는 Map에서 key로 String을 넣을 때 구분하기 위한 용도로 쓰인다.

```java
String a = new String("ddd");
		String b = "ddd";
		System.out.println(a.hashCode()); //99300
		System.out.println(b.hashCode()); //99300
		System.out.println(a==b); //false
```

- toString은 Object에서는 객체의 해쉬코드로 된 주소값을 반환한다.
- 하지만 String, Date, Arrays class는 toString()으로 문자열을 반환한다.
- 또한 커맨드 객체의 경우, 보통 toString()을 만들면 자신의 field 값을 반환하게 만든다.

```java
Strin abc ="abc";
sysout(abc.toString()); //abc
Date today = new Date(); 
System.out.println(today.toString()); //Sat Sep 16 21:06:27 KST 2023
LocalDateTime today1 = LocalDateTime.now(); //Date class는 1.8 이상부터는 거의 쓰이지 않는다.
```

- clone()의 경우 얕은 복사를 실행한다. 다만 Clonealbe interface를 구현한 class만 사용가능한 method다.
- clone()으로 배열이나 list, 커맨드 객체를 복사할 때 특히 조심해야 한다. 기본형 배열은 참조가 없어 얕은 복사와 깊은 복사가 똑같다.
- 하지만 객체를 원소로 가지는 경우, 얕은 복사만 하게 되면 복사본 배열의 값을 변경하면 원본도 변경된다.
- 따라서 둘을 분리시키는 깊은 복사가 필요하다.

```java
Class Circle implements Cloneable{
    Point p;
    double r;

    Circle(Point p, double r){
        this.p = p;
        this.r = r;
    }

    public Circle shallowCopy(){
        Object obj =null;
        try{
            obj = super.clone();
        }catch(CloneNotSupportedException e){}

        return (Circle)obj;
 
    }

    public Circle deepCopy(){
        Object obj = null;
        
        try{
            obj = super.clone();
        }catch(CloneNotSupportedException e){}

        Circle c = (Circle)obj;
        c.p = new Point(this.p.x, this.p.y);

        return c;
    }
}

```java
Circle c1 = new Circle(new Point(1,1),2.0); //주소값 0x100, p의 주소값 0x200
Circle c2 = c1.shallowCopy(); //주소값 0x300, p의 주소값 0x200
Circle c3 = c1.shallowCopy(); //주소값 0x400, p의 주소값 0x200
Circle c4 = c1.deepCopy(); //주소값 0x500, p의 주소값 0x600
```

- 얕은 복사의 경우, 복사한 클래스의 참조변수의 주소값은 다 다를지라도 그 안에 들어있는 원소 p의 주소값은 전부 동일하다.
- 깊은 복사의 경우, 복사한 클래스의 참조변수 주소값도 다르고, 그 안에 들어있는 원소 p의 주소값도 원본과 달라진다.
- 따라서 객체를 복사하려면 대부분 깊은 복사를 실행해야 한다.

## <span style="color:#802548">_java의 String class란?_</span>

- String은 immutable하다. 즉 불변이기 때문에 값이 변하지 않는다.
- 그런데 우리는 +로 계속 String 값을 바꾸고 있었다.

```java
String a = "a"; //주소값 0x100
String b = "b"; //주소값 0x200
a = a + b; //주소값 0x100이 아니라 0x300. immutable하기 때문에 새로 생성되는 것.
```

- 그럼 기존의 0x100은 어떻게 되는 걸까? 참조를 잃고 나중에 GC에 의해 수거된다.
- 즉 + 결합은 문자열은 매번 새로운 instance를 생성하는 것이다. 
- 따라서 새로운 instance를 덜 만들려면 StringBuffer(multithread)나 StringBuilder를 사용하는 것이 좋다.
- StringBuffer를 살펴보자.

```java
StringBuffer sb = new StringBuffer("abc");
StringBuffer sb2 = sb.append("123"); // append는 객체의 주소값을 반환한다. 따라서 기존 abc에다가 123을 덧대면서 새로운 instance를 만들지 않는다.

sysout(sb.toString()); // abc123
```

- String과 달리 StringBuffer는 equals와 ==비교가 동일한 결과를 낳는다. equals를 overriding하지 않아 문자열이 아닌 객체의 주소값으로 비교하기 때문이다.
- 다만 toString()은 override되어있어 객체의 주소값이 아닌 문자열을 반환한다.

- 참고로 띄어쓰기가 필요할 떄는 아래와 같이 적어줄 수 있다.

```java
String a = "광역시";
String b = "구";
String c = "동";
StringBuilder sb = new StringBuilder(a) + sb.append(" " + b) + sb.append(" " + c); // 너무 기니까 아래와 같이 증감연산자를 이용
StringBuilder sb = new StringBuilder(a);
              sb += sb.append(" " + b);
              sb += sb.append(" " + c);

## <span style="color:#802548">_java의 Math class란?_</span>
- Math에는 instance 변수가 없다. 전부 static method이며 매개변수로 값을 받는다.
- 원하는 소수번째 자리에서 반올림하는 방법만 간단하게 짚어보자.
- round는 long, 즉 정수형으로 반환한다. 원하는 소수번째 자리를 얻으려면 아래와 같이 바로 round()를 써서는 안 된다.
- 둘째자리까지 반올림한 수를 얻으려면 100을 곱한 수를 round에 매개변수로 넣고 다시 100을 나눠준다.

```java
double a = 1.4352;
double b = Math.round(a); // 1
```

```java
double a =1.4352;
double b = Math.round(a * 100) / 100; //143.52에서 round -> 144 /100  --->1.44
```

```java
double a =1.4332;
double b = Math.round(a * 100) / 100; //143.32에서 round -> 143 /100  --->1.43
```

- 그 외 중요사항으론 add()가 아니라 addExact로 Exact가 붙어있어야 overflow가 날 때 exception을 던진다.


## <span style="color:#802548">_java의 Big.. class란?_</span>
- BigInteger나 BigDecimal은 long과 double을 뛰어넘는 수를 계산할 때 사용한다.
- 생성 매개변수로는 보통 문자열이 들어간다. double이나 long이 들어가면 값이 long이나 double처럼 제한돼 BigInteger나 BigDecimal을 쓰는 이유가 없기 때문이다.


## <span style="color:#802548">_java의 autoBoxing란?_</span>
- 기본형 변수와 wrapper class 간에 자유롭게 오갈 수 있게 하는 JVM의 기능이다.

```java
int i = 5;
Integer iObj = new Integer(7); // new Integer는 이제 사용불가능하다. Integer.valueOf()를 사용한다.
int sum = i +iObj; //12
```

- int와 Integer를 더해도 알아서 int형으로 값이 나온다. Jvm이 아래와 같이 알아서 변환해주기 떄문이다. 

```java
int sum = i + iObj.intValue(); //wrapper를 기본형으로 unboxing했다.
```

```java
ArrayList<Integer>list = new ArrayList<Integer>();
list.add(0);
```

- Integer만 element로 받는데 int를 넣었다. 하지만 Jvm이 알아서 boxing해주기 때문에 문제가 없다.
- 참고로 generics에는 int같은 기본형은 넣을 수 없다. 객체만 들어갈 수 있다.

```java
ArrayList<Integer>list = new ArrayList<Integer>();
list.add(Integer.valueOf(10));
```

- autoBoxing은 주의해서 해야 한다. 아니면 계속 boxing과 unboxing을 반복하기 때문에 느려진다.
- 아래가 바로 잘못된 autoBoxing의 사례다.
```java
Long sum = 0L;

for(long i=0 ; i < Integer.MAX_VALUE ; i++) { // i는 기본형인데 Integer와 비교하기 때문에 계속 boxing, unboxing을 for문 index 하나하나 반복. 느려짐
    sum +=i;
}
System.out.println(sum);
```

```java
long sum = 0L;
int max = Integer.MAX_VALUE; // autoBoxing은 따로 뺴서 먼저 진행한다.

for(long i=0 ; i < max ; i++) { // for문에서는 int 기본타입끼리 비교한다. Integer비교는 시간과 메모리가 많이 든다.
    sum +=i;
}
System.out.println(sum);
```













































































