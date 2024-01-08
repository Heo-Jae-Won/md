## <span style="color:#802548">_변수와 상수란?_</span>

- 수학에서는 변하는 수라는 개념이었다. 하지만 프로그래밍에서는 아니다. 프로그래밍에서 변수란, 값을 저장할 수 있는 메모리 상의 공간을 의미한다.
- 변수는 사용 전에 초기화가 필요하다. 메모리는 공용자원이라 다른 프로그램에 의해 저장된 값이 있을 수 있기 때문이다.   즉 다른 프로그램을 위해서, 그리고 지금 프로그램을 위해서 초기화는 필수적이다. 

```java
int a;
```

- 상수는 선언과 초기화(할당)이 동시에 이뤄져야 한다. 상수는 값을 한번만 저장해야 하기 때문에 차후에 값을 바꿀 수가 없다. 

```java
final String a =5; //선언과 할당이 늘 한번에 이뤄짐.
```
​
## <span style="color:#802548">_초기화란?_</span>

- 변수 초기화는 거창한 개념이 아니다. 변수를 사용하기 전에 처음으로 값을 할당하는 것을 의미한다. 
- 지역변수는 사용되기 전에 반드시 초기화가 필요하다. 하지만 클래스변수나 인스턴스변수는 초기화를 생략할 수 있다. 
- int형의 경우 할당하지 않으면 0이 되며, boolean은 false이다.변수는 식별자를 가진다.

```java
int a = 5; // 선언과 할당을 한번에 할 수도 있음. 여기서 바로 a가 식별자.
static int b;
if(true){
    a=b; //가능. 할당이 없으면 0으로 인식하여 처리됨.
}

if(true){
    int c;
    a = c; //불가능. 지역변수는 반드시 초기화가 필요.
}
```
​


## <span style="color:#802548">_data type이란?_</span>

- 값의 종류(data type)란 ? 변수에 저장할 수 있는 값의 타입은 Java에서는 기본형과 참조형이 있다.
- 기본형부터 알아보자. 기본형은 논리, 문자, 숫자가 있다. 논리는 boolean으로 1bit면 되는데, Java의 최소 처리단위가 1byte라서 1byte의 크기를 가진다. 
- 문자에서는 Char가 있다. Char는 2byte의 값을 가진다. 
- 숫자는 정수형과 실수형으로 나뉜다. Int가 기준이 되며, int는 4byte다. 그 아래로 short은 2byte. Byte는 1 byte, long은  8 byte다. Short은 C언어와의 연동 떄문에, byte는 2진수 때문에, long은 큰 수를 다루기 위해 나왔다. 엄청나게 큰 수는 BigInteger로 다룬다. 엄청나게 많은 소수점은 BigDecimal로 다룬다. 
- 정수형의 경우 int면 약 20억까지 표현 가능하고, long은 약 2의 63제곱까지 표현가능하다. 그것을 넘어서면 overflow가 발생한다.
- 실수형의 경우 float면 소수점 아래 6자리, double이면 14자리까지 되는데, 여기서는 overflow와 underflow 모두 발생할 수 있다. 
-  overflow, underflow는 exception이나 error를 의미하지 않기 때문에 처리에 있어 조심해야 한다. 
- Jvm은 기본적으로 4byte 아래의 피연산자는 4 byte로 변환해 연산을 수행한다. 따라서 short, byte를 쓰게 되면 메모리를 아끼지만 CPU 성능이 안좋아져 로딩 속도등이 저하된다. 엔간하면 int,long 이상으로 쓰자.
- 참조형은 객체의 주소값을 반환한다. 참조형의 기본값은 null이다. null의 경우 접근이 불가능하기 때문에 nullPointerException을 조심해야 한다.
- 컴퓨터는 처음에 10진수를 채택했으나 CPU가 매우 불안정하게 움직였다. 결국 껐다와 켰다라는 양분된 상태로 인지하게 하는 bit 체제가 도입되었고, 2진수 기반으로 전압을 성공적으로 처리했다. 
- 즉 컴퓨터는 우리가 10진수로 써도 내부에선 2진수로 저장된다는 의미다. 컴퓨터는 1bit를 읽을 수 있지만, 1bit를 연산하지는 않는다. 8bit까지 모아서 연산을 실행한다. 따라서 8bit, 즉 1byte가 최소 연산 단위다. 이는 효율성을 위해 그렇게 만든 것이다. 
- CPU가 한 번에 처리할 수 있는 최대 크기는 8byte다. 즉 64bit 이상은 다음 CPU 연산 때 처리하게 된다. 따라서 8 byte 이상의 data type을 만들수가 없다.
 
## <span style="color:#802548">_리터럴이란?_</span> 

- 리터럴이란 ? 리터럴은 거창한 게 아니라 변수가 아닌 값이다. 213, “아”, ‘A’, 2.0f, 1.0d 등이 모두 리터럴이다. 다만 단독으로 쓰일 수 없고 변수에 대입되는 형태로만 사용된다. 즉 변수의 type과 동일한 type의 리터럴을 사용해야 한다. 
- char라면 ‘’를, int라면 213을, float라면 2.0f를 말이다. 다만 값을 담기 위해 변수가 존재하는 것이기 때문에, 리터럴의 type이 변수의 type을 결정한다. 

```java
String a = "A";
char a = 'A';
int b = 5;
float c = 1.5f;
```

- int형은 10진수가 대부분이지만 원한다면 2진수나 16진수로도 나타낼 수 있다. 

```java
Int hexNum=0x10 //A(16)
Int binNum = 0b10 //10(2)
```
 
숫자의 경우, 출력을 다양한 형태로 바꿀 수 있다. 바로 원하는 자리 수만큼 0을 채워넣거나 빈칸을 넣는 형태다. 변수가 10을 담고 있다고 해보자.
```java
%5d   //[   10]으로 앞의 3자리 빈칸
%-5d  //[10   ]로 뒤의 3자리가 비어 출력
%05d  //[00010]으로 빈 자리를 0으로 채우게
%-05d //[10000]은 10이 아니라 만
```
## <span style="color:#802548">_문자형이란?_</span> 

- 문자형이란 ? char는 문자이긴 하지만, 앞에서 말했듯, 컴퓨터는 0과 1밖에 모르기 때문에 문자가 아닌 2진수로 저장된다. 그 숫자로 저장되는 약속 중 가장 흔한 게 바로 UTF-8이다. 
- UTF-16은 영어와 숫자도 2byte로 처리해야 한다는 점에서 영어권 국가에서 이점이 없어 거의 쓰이지 않게 되었다. 
- UTF-8을 쓰게 되면 영어와 숫자는 1byte, 한글은 3byte를 잡아 처리한다. 문자에는 음수가 없어 -를 고려하지 않기 때문에 표현폭이 더 넓다. 

## <span style="color:#802548">_정수형이란?_</span> 

- 정수형이란 ? 정수형에서 int는 -까지 포함해 표현해야 해서 표현 폭이 좁다. 
- 2의 8제곱이면 -128부터 127까지다. 정수형을 실수형으로 바꾸는 것은 매우 위험할 수 있다. 소수점 이하 정보는 제대로 표현될 수 없을 수 있기 때문이다. 
- 정수형에는 overflow가 존재하는데, 해당 data type에게 주어진 byte로 표현하지 못하는 큰 수의 경우에는 뒷자리가 짤려나가기 때문이다. 
- 대부분의 IDE에서는 The literal 3_100_000_000 of type int is out of range 오류를 보여준다. 하지만 문제는 그 자체로는 아니어도 더하면 int type의 범위를 넘어갈 때이다. 이 때는 IDE의 도움을 받을 수 없다. 특히 변수일 때 그러하다.
  
```java
int a = 3_100_100_100; //int는 31억은 담을 수 없어 뒷자리들이 짤리기 때문에 overflow 오류. 값이 원하는 형태로 출력되지 않음. 

long a = 3_100_100_100; //20억넘어가면 long으로 type을 주어야 함.
```

```java
int a = 2_100_000_000;
int b = 2_100_000_000;

sysout(a + c); //4_200_000_000을 예상하지만 결과는 -94967296로 나옴. 뒷자리가 짤려 overflow가 일어난 것.
```

## <span style="color:#802548">_실수형이란?_</span> 

- 실수형은 2진수로 완전한 표현이 불가능하여 오차가 있다. 다시말해 0.1과 0.1f나 0.1d는 값이 다르다는 것이다. 실제로는 0.1은 늘 float거나 double이기 때문에 비교할 일은 없다. 하지만 0.1f나 0.1d끼리는 비교가 가능하다. 
- 그 중에서 정밀도가 더 높은 것은 double이다. 소수점 아래 값을 저장하기 위해 더 많은 저장 공간, 8 byte를 소모하기 때문이다. 따라서 값이 달라진다. float는 소수점 이하 6자리까지만 인지하고 값을 계산한다. 따라서 float형으로 값이 계산된 뒤 double로 형변환해도 값이 변하지 않는다.

```java
float f = 9.1234567f; 
double d = 9.1234567;
``` 

- 위 두개의 변수는 값이 다르다. 정밀도가 다르기 때문이다.

```java
float f = 9.1f; 
double d = 9.1;
``` 

- 위 두개의 변수도 값이 다르다. 정밀도가 다르기 때문이다.

```java
double d2 = (double) f //값이 달라진 상태에서 형변환을 해봐야 float로 값이 계산이 끝남.
```

- 형변환을 해도 d와 d2는 다르다. 
- 따라서 처음부터 double로 type을 정해서 계산해야 한다. 아니면 double을 float로 형변환해줘야 한다. 
- float와 double은 숫자가 같아도 2진수로 변환돼 저장되는 지점에서 정밀도가 다르기 때문에 실제 값이 다르다는 점을 이해해야 한다

- 물론 아래와 같이 .0의 경우에는 값이 같다.소수점이 없는 것과 똑같기 때문이다.

```java
float f = 9.0f; 
double d = 9.0;
``` 


## <span style="color:#802548">_형변환이란?_</span> 

- 형변환은 큰 byte에서 작은 byte로 갈 때는 명시적으로 casting을 씌워야 한다. 값을 잃어버릴 가능성이 존재하기 때문이다. 
- 8 byte짜리가 4 byte가 되면 남은 4byte가 담던 정보가 사라지기 때문이다. 특히 Long ㅡ> int 혹은 double ㅡ> float은 굉장히 위험하다. 
- Int ㅡ> float이나 float ㅡ> int는 가능하다. 다만 실수형에서 정수형으로 변환할 때는 소수점 이하 자리가 싹 다 사라지고 정수형 정보만 남게 되므로 이 점에 유의해야 한다. 
- 작은 byte에서 큰 byte로 갈 때는 묵시적 형변환이 허용된다. 따라서 따로 casting을 하지 않는다. 
- 다만 변수가 사용된 경우에는 명시적인 casting이 반드시 필요하다. 이유는 compiler가 리터럴 간의 연산은 다르게 처리하기 때문이다. 
- 리터럴 간의 연산은 runtime동안 변할 수 가 없는 상수기에 compiler에서 바로 계산을 실행하여 char c2 = ‘b’로 바꿔치기한다. 
- 하지만 변수를 사용했다면 값이 바뀔 수 있어 compiler가 계산하지 않는다. 특정 조건 하에 c1이 ‘b’가 될 수도 있기 때문이다. 
- 따라서 c1 + 1은 data type과 literal type이 맞지 않는다. char와 int가 섞여있기 때문이다. Runtime 때 실행해서 값이 들어와야 알 수  있다. 따라서 char로 형변환을 명시적으로 해야한다.

```java
Char c1 = 'a'; 
Char c2 = c1 + 1; // error. Char c2 =  (char) (c1 + 1);로 써줘야 함.
Char c2 = 'a' +1; //정상.
```

## <span style="color:#802548">_식이란?_</span> 
​

- 연산자와 피연산자가 합쳐진 것을 식이라고 한다. 
- 식을 계산하는 것이 식을 평가하는 것이며, 식은 ;로 끝마친다. 즉 ;까지가 하나의 문장인 셈이다. 
- Or 연산(||)에서는 참일 확률이 높은 피연산자를 왼쪽에 넣는 게 중요하다. 일단 if문에서 왼쪽이 참이면 오른쪽은 평가를 하지 않기 때문이다. 그럼 그만큼 CPU를 아끼게 될 것이다. 

```java
int a = 5;
int b = 0;
Sysout("a=%d, b=%d%n", a, b); //a =5 , b = 0
Sysout("a!=0 || ++b!=0 =%b%n", a!=0 || ++b!=0) //a=5, b=0;
Sysout("a=%d, b=%d%n", a, b); //a=5, b=0
```
- ++b ! =0 식이 평가되지 않음을 볼 수 있다. 왼쪽이 true였기에 ++b!=0은 평가되지 않았다. 해당 식이 평가되었다면 b가 1로 출력되었을 것이다.
-  method 안에서 쓰인 전위증감이기 때문이다.

```java
Sysout("a!=0 || ++b!=0 =%b%n", ++b!=0 || a!=0 ) //a=5, b=1;
```

## <span style="color:#802548">_증감연산자란?_</span> 

- 증감연산자는 독립적으로 쓰이면 후위든 전위든 차이가 없다. 
```java
i++;
++i;
```

- 하지만 매개변수로서 혹은 식을 구성하는 요소로서 같이 쓰이면 의미가 달라진다.
```java
i=i++; // 다음번에 i가 증가된 상태
i=++i; // 지금 당장 i가 증가된 상태

Sysout("++b != 0") //a=5, b=0; //당장 지금 i가 증가된 상태로 평가됨
Sysout("b++ != 0") //a=5, b=1; //다음번에 i가 증가된 상태로 평가됨
```
- 아래의 예시를 본다면 더욱 이해가 빠를 것이다.
```java
int i = 1;
while(i--!=0){
	Sysout(i)
}
```
```java
while(i!=0){
	i--; //--i;도 가능
	sysout(i);
}
```
위의 두 식은 동일하다.

```java
while(--i!=0){
	Sysout(i)
}
```
```java
while(i!=0){
	--i; //i--;도 가능
	Sysout(i)
}
```
- 위의 두 식은 동일하지 않을 수 있다. 조건식이 false로 변했을 수도 있기 때문이다.

- 증감연산자가 식 혹은 매개변수일 때 전위증감과 후위증감이 작동하고, 독립적인 i++;의 경우 후행이라는 말을 했다. 
- for문에서도 마찬가지다. i++이든 ++i이든 후위증감의 효과로 나타난다. 
- for문에서 ++i;를 하든 i++;을 하든 독립적으로 쓰인 것이기 때문에 for문이 한바퀴 돌은 이후에 i가 ++이 된다. 아래 두 개의 식이 동일한 결과를 가져온다.

```java
for(int i=0;i<array.length;i++);
```

```java
int i =0;
while(i<array.length){
   i++
};
```

## <span style="color:#802548">_곱연산자란?_</span> 

- 곱연산 시에, int와 int 끼리 곱한 것은 무조건 int다. 

```java
int a = 1_000_000;
int b = 2_000_000; 
long c = a * b; //a와 b를 곱한 것은 long이 아니라 int다
```
- 리터럴이 data type을 따라가는 게 아니라 리터럴이 data type을 결정한다는 사실을 상기해보자. 
- a와 b를 곱한 것이 int type이었기 때문에 int 값으로 20억을 지닐 수 없다.
- 따라서 변수가 long type으로 선언되어있어도 overflow가 난다.

## <span style="color:#802548">_제어문의 활용이란?_</span> 

- If-else로 변환가능한 상반된 조건의 2개 if문은 if-else로 변환하는 것이 좋다. 그럼 처음 if문을 평가하면 두번째 if문은 평가하지 않아도 되기 때문에 효율적이다. 
- &&의 경우에는
```java
 if(score >=80 && score <=90) //가독성 나쁨. 
 ```
 
 ```java
 if(80 <=score && score <90) //가독성 좋음. 
 ```
- 물론 &&를 최대한 안 쓰는 게 좋다. 평가할 식이 늘어나기 때문이다.

```java
If(score >=90){
	Grade=’A’;
}else if(80<= score && score <90){
	Grade =’B’;
}else if(70 <= score && score < 80){
	Grade =’C’;
}else {
	Grade = ‘D’;
}
```

```java
If(score >=90){
	Grade =’A’;
}else if(score >=80){
	Grade =’B’;
}else if(score  >=70){
	Grade =’C’;
}else {
	Grade =’D’;
}
```
- 위의 두 식 중 후자가 더 낫다. 평가할 식이 줄어들기 때문이다.
- switch문을 활용할 수 있다면 if문보다 switch문이 더 효율적이다. switch문에서는 평가할 조건식이 단 하나이기 때문이다.  
- 다만 case 값은 정수나 문자열 litereal이여먄 한다는 제약이 있다. case값에는 변수를 넣을 수없고 다른 data type도 넣을 수 없다. 
- 또 아래를 보면 알 수 있듯, 범위를 따지는 조건이라면 switch문을 쓰면 매우 불편하다. 
```java
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
.
.
.
```
- switch문을 쓰면 90일때, 91일때, 92일 때 모두 A처리를 따로 해야해 매우 불편하다.
- 따라서 아래와 같은 if문이 더 편하다.
```java
	if(grade >= 90){
		Grade ='A';
	}
```



## <span style="color:#802548">_반복문의 활용이란?_</span> 
- while문의 loop 탈출방법이란 ? break; 혹은 while문의 조건을 false로 만들면 된다.

```java
while(flag){
	if(num!=0){
		Sysout(num)
	}else{
		Flag=false;
	}
}
while(flag){
	if(num!=0){
		Sysout(num);
	}else{
		Break;
	}
}
```
- 아래식과 동일한 결과다. Break;로 빠져나가든 조건식을 false로 만들어 빠져나가든 동일하다.

- while문의 최종 정리는 아래와 같다. outer라는 큰 반복문이 있다. 그 아래로 for문과 같은 outer에 속한 반복문이 있다. 
- for문 밖에서 하는 break는 outer를 빠져나가지만, for문 안에서의 break;는 for문을 빠져나가지 outer를 빠져나가지 않는다. 
- outer를 빠져나가고 싶다면 break outer를 써야한다. 
- for문안의 switch문은 break를 쓰면 switch문을 빠져나가는 것이다. 그 뒤에 logic이 있다면 for문을 계속 순회하게 된다. 
- 반면에 if문은 break를 쓰면 for문을 빠져나간다. if와 switch는 같은 조건문이지만 break;에서 차이가 존재하니 주의를 요구한다.

```java
Outer:
while(true){
	Sysout("(1) square");
	Sysout("(2) square root"):
	Sysout("(3) log ");
	Sysout("원하는 메뉴 선택");
	
	String tmp = Scanner.nextLine();
	Int menu = Integer.parseInt(tmp);
	if(menu==0){
		Sysout("프로그램 종료");
		Break;
	}else if(!(1<=menu && menu <=3)){
		Sysout("메뉴를 잘못 선택했습니다. 종료는 0");
		continue;
	}

	for(;;){
		Sysout("계산 값 입력. (계산 종료: 00, 전체종료:99)");
		Tmp = Scanner.nextLine();
		Num = Integer.parseInt(tmp);
		
		if(num==0){
			break; // 해당 if문을 감싼 for문을 벗어남.
		}
		if(num==99){
			Break outer; // 해당 if문을 감싼 for문이 아니라 outer라는 이름을 가진 반복문 전체를 벗어남.
		}

		switch(menu){
			case 1:
				Sysout("result = " + num * num):
				break; //switch문에서의 break;
			case 2:
				Sysout("result = " + Math.sqrt(num));
				break; //switch문 끝나고서 진행되는 소스코드있으면 다 진행됨
			case 3: 
				Sysout("result = " + Math.log(num));
				break;
		}
	}
}
```

