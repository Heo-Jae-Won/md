## <span style="color:#802548">_class_</span>
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

Car(){
	this(“white”,”auto”,4); // Car("white","auto",4);
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
