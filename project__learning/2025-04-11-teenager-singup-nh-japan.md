## <span style="color:#802548">_js literal map 활용하기_</span>
- if문으로 값을 받아와 분기처리하는 경우, 아래와 같은 형태일 것이다.

```js
if (payload.responseCode =='0') {
    //logic
    openFailLayer();
} else if(payload.responseCode == '1') {
    //logic
    location.href = '/성공';
} else {
    //logic
    openFailLayer();
}
```

- magic number는 유지보수성을 해치므로 명시적으로 만들어준다.

```js
const IDENTITY_NOT_AUTHENTICATED = '0';
const IDENTITY_AUTHENTICATED = '1';
const IDENTITY_NOT_OWNED = '2';
if (payload.responseCode === IDENTITY_NOT_AUTHENTICATED) {
    //logic
    openFailLayer();
} else if (payload.responseCode === IDENTITY_AUTHENTICATED) {
    //logic
    location.href = '/성공';
} else if (payload.responseCode === IDENTITY_NOT_OWNED) {
    //logic
    openFailLayer();
}
```

- if문도 좋지만, 아래와 같이 객체에 넣어서 좀 더 잘게 쪼개는 것도 가능하다.
- 객체에 넣어서 관리하게 되므로 훨씬 명확하게 어떤 의미인지 알 수 있다.
- 유지보수성에 있어서도 하나의 객체에 모여있어 혹시 해당 객체만 수정하면 되므로 간편하다.

```js
const IDENTITY_NOT_AUTHENTICATED = '0';
const IDENTITY_AUTHENTICATED = '1';
const IDENTITY_NOT_OWNED = '2';
const authenticationMap = {
    IDENTITY_NOT_AUTHENTICATED: function() {
        //logic
        return false;
    },
    IDENTITY_AUTHENTICATED: function() {
        //logic
        return true;
    },
    IDENTITY_NOT_OWNED: function() {
        //logic
        return false;
    }
}

function executePostAuthentication(resultCode) {
    authenticationMap[resultCode]();
}

function authAjax() {
    $.ajax({

    }),
    success: function(res,status,xhr) {
        var payload = JSON.parse(res.payload);

        if (executePostAuthentication(payload.resultCode) == false) {
            openFailLayer();
        }
    }
}
```



## <span style="color:#802548">_중복클릭 방지 고민_</span>
- 과거에 내가 만든 중복클릭 방지는 전역변수, setTimeout을 이용했었다.

```js
var reqFlag = false;

function reqCheck(){
    if(reqCheck == false){
            reqFlag = true;
            return false;
    }

    setTimeout(function(){
            reqFlag = false;
    },3000)

    return true;
}

function handleButtonClick() {
    $('#button').on('click',function() {
        if(reqCheck()) {
            return;
        }
        .
        .
        .
        //ajax 함수
    })
}
```

- 하지만 위의 경우, 혹시 api가 3초를 넘는 경우에는 클릭이 가능하다.
- 서버가 끝나는 타이밍에 맞춰 클릭이 가능한 상태로 변경하려면, 새로운 UI 혹은 프로그래밍을 도입해야 했다.
- 새로운 UI는 내가 결정할 수 없는 영역이었고, 일정에 부담이 컸기 때문에 프로그래밍적으로 구현 가능한 js closure를 사용하려 했다. 

## <span style="color:#802548">_중복클릭 방지 clousre 오류 해결_</span>

- 하지만 원하는대로 작동하지 않았다.

```js
function sendRequest(fn) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            fn(data)
        }
    }
}

function handleButtonClick() {
    $("#button").on('click',function() {
        .
        .
        .
        var sendData = {};
        sendData.payload = id;

        sendRequest(testAjax)(sendData);
    })
}
```

- 문제는 click 시마다 sendRequest()()를 호출하게 되면 매번 새로운 실행 컨텍스트가 생기는 게 문제였다.
- 그래서 실행 컨텍스트를 저장해두는 변수를 만들었다.
- 그랬더니 이제 원하는 대로 작동하기 시작했다!

```js
function sendRequest() {
    var clicked = false;
    return function() {
        if(!clicked) {
            clicked = true;
            var sendData = {};
            sendData.payload = id;
            $.ajax({
                .
                .
                .
                success: function(res, status, xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                },
                error: function(error) {
                    alert(error);
                     clicked = false;
                } ,
                complete: function() {
                    checkMobileDeviceOs();
                     clicked = false;
                }
            })

        }
    }
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
       fetchData();
    })
}
```



## <span style="color:#802548">_중복클릭 방지 clousre 읽기 쉽게 변경하기_</span>

- 이제는 ajax 함수를 바깥으로 빼보고 싶었다. 요청함수가 너무 길어지지 않게 하고 싶었다.
- 그랬더니 clicked가 closure함수가 속한 scope에서 벗어나 버렸다.
- 그래서 이를 해결하기 위해 parameter로 넘겨봤지만 아무 소용이 없었다.
- 계속 clicked가 false로 초기화됐다.

```js
function sendRequest() {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            textAjax(data, clicked);

        }
    }
}

function testAjax(data, clicked) {
    $.ajax({
        .
        .
        data:data
        success: function(res, status, xhr){
            if(res.statusCode === '200') {
                location.href = '/teenAgerCardIssue';
            }
            clicked = false;
        },
        error: function(error) {
            alert(error);
            clicked = false;
        } ,
        complete: function() {
            checkMobileDeviceOs();
            clicked = false;
        }
    })
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```

- 결국 과거의 jquery ajax 활용법으론 불가능하다고 판단했다.
- Promise와 비슷하게 .then()처럼 이어가는 방식의 ajax를 사용하기로 했다.
- 이렇게 하면 ajax를 분리하면서 자유변수는 그대로 살릴 수 있었다.

```js
function sendRequest() {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            textAjax(data)
                .done(function(res,status,xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```

- 이제는 여러가지 ajax 함수를 받을 수 있게 함수를 parameter로 받으려 했다.
- ajaxFn으로 받는 것은 아래와 같이 간단하게 바꿀 수 있었다.
- 대신 처음 실행컨텍스트를 정할 때 원하는 ajax 함수를 같이 넘기게 바꿔야 한다.

```js
function sendRequest(ajaxFn) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            ajaxFn(data)
                .done(function(res,status,xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

var fetchData = sendRequest(testAjax);

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```

## <span style="color:#802548">_중복클릭 방지 clousre 공통화하기_</span>
- 그런데 successCallback도 같이 ajax와 넣어두면 이걸 공통함수로 뺄 수 있겠다는 생각이 들었다.
- 그래서 successCallback도 같이 parameter로 받게 바꿨다.
- successCallback 함수는 res를 parameter로 받아야한다.

```js
function sendRequest(ajaxFn, successCallback) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            ajaxFn(data)
                .done(function(res,status,xhr){
                    successCallback(res)
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

function successCallback(res) {
    var popupText = "";
    var popupHtml = "";
    if(res.statusCode === '200') {
        location.href = '/teenAgerCardIssue';
    } else if (res.statusCode === '199') {
        popupText = "비밀번호가 1회 오류입니다."
    } else if (res.statusCode === '198') {
        popupText = "비밀번호가 2회 오류입니다."
    } else {
        location.href = '/teenAgerError'
        return;
    }

    poupHtml = $("#errorGuide").text(popupText);
    popupHtml.html(popupHtml.html().replace(/\n/g,'<br>'));

    Layer.open("#errorPopup");
}

var fetchData = sendRequest(testAjax, successCallback);

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```


- 모듈화를 해놨으니 다른 공통 함수로 빼놓을 수 있다.
- 공통함수 js에 넣을 수 있다는 의미다.
- 공통 사용을 건의해봤지만 아쉽게도 이해하기 어렵다는 이유로 거부당했다.

```js
//common.js

function sendRequestOnlyOnce(ajaxFn, successCallback) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            ajaxFn(data)
                .done(function(res,status,xhr){
                    successCallback(res)
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}
```

- 이제 ajax들은 모두 common.js에서 가져온 sendRequestOnlyOnce 함수로 wrapping할 수 있다.
- 그러면 별다른 처리 없이 간단하게 아래와 같은 순서로 하면 중복클릭방지가 저절로 들어가는 것이다.
- script import --> ajax 함수 정의 --> ajax의 successCallback 정의 --> 실행컨텍스트 저장을 위한 변수 정의 --> data collection --> eventListener에서 함수실행
- 다만 여기서 ajax에 절대로 async:false 옵션이 있으면 안 된다.
- 그럼 어떤 방식(버튼 disabled, 전역변수 flag 등..)을 쓰든 중복클릭 방지가 작동하지 않는다.

```js
//teenAger.jsp
<script src ="/my/common.js"/>



//원하는 ajax함수
function testAjax(data) {
    var leftDay = "${leftDay}";
    if (parseInt(leftDay) >= 1) {
        preventRequestAndShowPopup(parseInt(lefyDay));
        return;
    }

    return $.ajax({
                url:'~~',
                method:'post',
                dataType:'json',
                data:data
            })
}

//원하는 ajax의 successCallback
function successCallback(res) {
    .
    .
}

// 실행컨텍스트 저장을 위해 wraaping하는 함수
var fetchData = sendRequestOnlyOnce(testAjax, successCallback);

function handleButtonClick() {
    $("#button").on('click',function() {
        //data collecting
        var sendData = {};

        //실제 함수 실행
       fetchData(sendData);
    })
}
```


## <span style="color:#802548">_더 나은 에러로그_</span>
- 개발 이후 close - beta에서 에러가 나서 에러를 찾느라 고생한 적이 있다.
- 해당 현장은 에러를 처리하는 메뉴얼이 있었는데, 그 메뉴얼이 잘못되어서 고생하였다. 

```java
public class Main {
	public static void main(String[] args) throws InterruptedException {
		char[] chars = {'a','b','c','d'};
		try {
			calculate(chars, 6, i -> i[0]==1 && i[1] == 1 && i[2] == 0);	//9 line
		} catch (Exception e) {
			StackTraceElement[] stackTrace = e.getStackTrace();
			StackTraceElement element = stackTrace[0];
			System.out.println("lineNumber: " + element.getLineNumber() + "\nmessage: " + e.getMessage());
		}
	}
	
	static void calculate(char[] a, int k , Predicate<int[]> decider) {
		int n = a.length;
		if (k < 1 || k > n) {
			throw new IllegalArgumentException("Forbidden"); //23 line
		}
	}
}
```

- 0번쨰 element로 exception line을 가져오면 현재는 error를 던진 곳이 line이 찍히고 있다.
- 정상 출력되고 있는 셈이다. error를 명시적으로 throw할 때는 아무 문제가 없다.

```
java.lang.IllegalArgumentException
	at com.example.Main.calculate(Main.java:23)
	at com.example.Main.main(Main.java:9)
lineNumber: 23
```


- 이번에는 명시적인 throw를 하지 않고, java api 활용에서 실수를 냈다.
- substring을 할 때 indexoutofboundsexception이 났다.
- try catch를 넣으면 아까와는 사뭇 양상이 다르다.

```java
public class Main {
	public static void main(String[] args) throws Exception {
		char[] chars = {'a','b','c','d'};
		calculate(chars, 6, i -> i[0]==1 && i[1] == 1 && i[2] == 0);	//8 line
	}
	
	static void calculate(char[] a, int k , Predicate<int[]> decider) {
		
		try {
			int n = a.length;
			String abc ="fff";
			abc = abc.substring(0,12); // 16 line
		} catch (Exception e) {
			e.printStackTrace();
			StackTraceElement[] stackTrace = e.getStackTrace();
			StackTraceElement element = stackTrace[0];
			System.out.println("lineNumber: " + element.getLineNumber() + "\nmessage: " + e.getMessage()+ "\nclass: " + element.getClassName());
		}
		
	}
}
```

- String class가 맨 첫줄에 exception에 찍힌다.
- 우리가 기대한 것은 calculate의 16 line이었다. 거기서 실제 exception이 났다.
- 근데 String class가 exception line에 찍힌다.
- 명시적인 throw를 하지 않았기 때문이다.

```
java.lang.StringIndexOutOfBoundsException: begin 0, end 12, length 3
	at java.base/java.lang.String.checkBoundsBeginEnd(String.java:4604)
	at java.base/java.lang.String.substring(String.java:2707)
	at com.example.Main.calculate(Main.java:16)
	at com.example.Main.main(Main.java:8)
lineNumber: 4604
message: begin 0, end 12, length 3
class: java.lang.String
```


- 그렇다면 error를 throw하고 catch로 잡아서 다시 exception을 던지면 된다고 생각할 수 있다.

```java
public class Main {
	public static void main(String[] args) throws InterruptedException {
		char[] chars = {'a','b','c','d'};
		try {
			calculate(chars, 6, i -> i[0]==1 && i[1] == 1 && i[2] == 0);	//9 line
		} catch (Exception e) {
			e.printStackTrace();
			StackTraceElement[] stackTrace = e.getStackTrace();
			StackTraceElement element = stackTrace[0];
			System.out.println("lineNumber: " + element.getLineNumber() + "\nmessage: " + e.getMessage());
		}
	}
	
	static void calculate(char[] a, int k , Predicate<int[]> decider) throws Exception {

       try {
			int n = a.length;
			String abc ="fff";
			abc = abc.substring(0,12); // 16 line
        } catch (Exception e) {
            throw new Exception("0001"); //18 line
        }
	}
}
```

- calculate의 16 line에서 exception이 났지만, 잡아서 18 line에서 던지기 때문에 바꿔치기된다.
- 따라서 어디서 exception이 났는지 도저히 알수가 없게 된다.
- 짧으니까 다행이지만, 길어지면 어디서 오류가 났는지 찾을 수가 없다.
- 명시적으로 throw를 던진다면 catch로 잡아서 다시 던지면 미궁으로 빠져버리게 되는 것이다.

```
java.lang.Exception: 0001
	at com.example.Main.calculate(Main.java:18)
	at com.example.Main.main(Main.java:9)
lineNumber: 18
message: 0001
```

- 가장 중요한 것은, catch를 한 뒤에 또 exception을 던지면 안 된다는 점이다. 그런 식으로 하면 stackTrace가 꼬인다.
- 그 다음으로는 line이 아닌, exception의 message를 아는 것이 중요하다.
- 서비스 로직이라면, log.error를 활용하는 것이 좋은 선택이다.
- 서비스의 실패가 아닌 Java api에서 예상치 못한 error를 막을 때를 대비해 전체에 try ~ catch를 씌워준다.
- 전체는 맨 마지막에 try catch를 처리하는 Controller에서만 진행되면 충분하다.

```java
public class Main {
	public static void main(String[] args) throws InterruptedException {
		char[] chars = {'a','b','c','d'};
		try {
			calculate(chars, 6, i -> i[0]==1 && i[1] == 1 && i[2] == 0);	//9 line
		} catch (Exception e) {
			e.printStackTrace();
			StackTraceElement[] stackTrace = e.getStackTrace();
			StackTraceElement element = stackTrace[0];
			System.out.println("lineNumber: " + element.getLineNumber() + "\nmessage: " + e.getMessage());
		}
	}
	
	static void calculate(char[] a, int k , Predicate<int[]> decider) throws Exception {
        int n = a.length;
		if (k < 1 || k > n) {
            log.error("business logic 오류입니다.")
			throw new BuisinessException("Forbidden"); //23 line
		}
        String abc ="fff";
        abc = abc.substring(0,12); // 16 line
    }
}
```