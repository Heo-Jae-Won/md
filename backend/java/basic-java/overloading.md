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