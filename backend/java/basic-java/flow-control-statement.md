## <span style="color:#802548">_조건문_</span> 
- nestd if를 쓸 때는 꼭 block을 달아주는 게 좋다.
```java
if (num >= 0) 
	if (num != 0) 
		sign = '+';
else
	sign = '-';
```

- 위 식은 아래처럼 해석된다.
```java
if (num >= 0) {
	if (num != 0) {
		sign = '+';
	} else {
		sign = '-';
	}
}
```

- 원래 의도대로라면 아래와 같이 썼어야 했을 것이다.
- 그런데 block을 적지 않아 실수를 알아채지 못한 것이다.
```java
if (num >= 0) 
	if (num != 0) 
		sign = '+';
	else
		sign = '-';

//

if (num >= 0) {
	if (num != 0) {
		sign = '+';
	} 
} else {
	if (num != 0) {
		sign = '-';
	} 
}
```

- 사실 위의 식은 early return을 해주면 더 좋다.
```java
if (num ! = 0) {
	throw new Exception("0은 안됨 ㅇㅇ");
}

if(num > 0) {
	sign = '+';
} else {
	sign = '-';
}
```

- &&의 경우에는 수학의 비교처럼 보이게 바꿔주자.
```java
 if (score >=80 && score <=90) //가독성 나쁨. 
 if (80 <=score && score <90) //가독성 좋음. 
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


- &&를 안 쓸 수 있나 고민해보자.
- 평가할 식이 줄어들면 성능 이득이다. 얼마 안되지만..
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


- if문 말고 switch문도 조건문 중에 하나다.
  - switch문에는 문자, 숫자, enum만 들어올 수 있다.
//https://velog.io/@leon/TIL5-%EC%9E%90%EB%B0%94-switch%EB%AC%B8-%ED%99%9C%EC%9A%A9%ED%95%98%EA%B8%B0
- switch문은 같은 결과를 원하는 case라면 case만 적어주면 된다.
- default를 적어주는 것을 늘 잊지말자.
- default가 끝이므로 default에는 break;가 필요없다.
```java
String regNo = "9502141";
char gender = regNo.charAt(7);

switch(gender) {
	case '1': case '3':
		sysout("당신은 남자입니다.");
		break;
	case '2': case '4':
		sysout("당신은 여자입니다.");
		break;
	default:
		sysout("유효하지 않은 주민번호입니다.");
}
```

- nested if처럼, nested switch도 가능하다.
- 다만 inner switch가 끝나면 break;를 꼭 걸어줘야 한다.
```java
String regNo = "9502141";
char gender = regNo.charAt(7);

switch(gender) {
	case '1': case '3':
		switch(gender) {
			case '1':
				sysout("당신은 2000년 이전에 출생한 남자입니다.");
			case '3':
				sysout("당신은 2000년 이후에 출생한 남자입니다.");
		}
		break;
	case '2': case '4':
		switch(gender) {
			case '2':
				sysout("당신은 2000년 이전에 출생한 여자입니다.");
			case '4':
				sysout("당신은 2000년 이후에 출생한 여자입니다.");
		}
		break;
	default:
		sysout("유효하지 않은 주민번호입니다.");
}
```


- switch문을 활용할 수 있다면 if문보다 switch문이 더 효율적이다. 
- switch문에서는 평가할 조건식이 단 하나이기 때문이다.  
  - 다만 case 값은 정수나 문자열 litereal이여먄 한다는 제약이 있다. 
  - case값에는 변수를 넣을 수없고 다른 data type도 넣을 수 없다. 
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
```


- switch문을 쓰면 90일때, 91일때, 92일 때 모두 A처리를 따로 해야해 매우 불편하다.
- 따라서 아래와 같은 if문이 더 편하다.
```java
	if(grade >= 90){
		Grade ='A';
	}
```



## <span style="color:#802548">_반복문_</span> 
- while문의 최종 정리는 아래와 같다. while이라는 큰 반복문이 있다. 
  - 그 안에 for문이 속해있다. 
    - for문 밖 while문에서 break는 while문(outer)을 빠져나간다.
    - for문 안에서 break는 for문을 빠져나간다.
    - for문 안에서 while문을 빠져나가고 싶다면 break outer를 써야한다. 
      - for문 안 if문에서 break는 for문을 빠져나간다. while loop는 그대로다.
      - for문 안 switch문에서 break는 switch문을 빠져나간다. for loop는 그대로다.


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
		break;
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
			break outer; // 해당 if문을 감싼 for문이 아니라 outer라는 이름을 가진 반복문 전체를 벗어남.
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

- 이차원 배열의 for문을 열 때는 조심해야 한다.
```java
int[][] score = {
					{100, 100, 100}
					,{20, 20, 20}
					,{30, 30, 30}
					,{40, 40, 40}
};

int sum = 0;

for (int i = 0; i < score.length; i++) {
	for(int j = 0; j < score[i].length; j++) {
		System.out.printf("score[%d][%d] = %d%n", i, j, score[i][j]);
	}
}

/*아래처럼 하면 error. 2차원 배열 score의 각 요소는 1차원 배열.
for (int i : score) {
	sum +=i;
}

*/

//따라서 해당 배열을 flat할 for문을 한 번 더 열어야 한다.
for (int[] tmp : score) {
	for( int i : tmp) {
		sum += i;
	}
}

System.out.println("sum=" + sum);
```

