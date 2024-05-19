## <span style="color:#802548">_try-catch란?_</span>

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


// 농좆 exception 실패 사례 적어주기