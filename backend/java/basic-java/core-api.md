## <span style="color:#802548">_Object_</span>
- 모든 class는 Object class를 extends한 것이다.
- Object class에는 hashcode(), equals(), toString(), clone() method가 있다. 
- hashcode()는 객체 주소값으로 해시코드를 만들어서 반환한다. 
  - 똑같은 객체라면 hashcode()를 했을 때 똑같은 값을 반환하게 하는 것이다.
  - 64bit JVM에선 중복 되는 해쉬코드가 나올 수도 있다. 따라서 해쉬코드가 같다고 객체 주소값이 같다고 생각하면 안된다.
  - 객체주소값을 완전하게 유일하게 가져가는 hashcode는 System.identityHashcode()다.
- equals도 마찬가지로 객체의 주소값이 같다면 같은 객체로 판단한다
  - 하지만 각 class마다 hashcode와 equals를 자신의 목적에 따라 override해서 사용하기도 한다.
  - String class는 객체의 주소값이 아니라 문자열이 같은지를 비교하기 위해 equals를 overriding했다.
  - 그에 따라 hashcode()도 같이 override되어있다. 즉 문자열이 같으면 hashcode도 같은 값을 지닌다. 그것이 다른 객체라고 해도 말이다.
- toString은 Object에서는 객체의 해쉬코드로 된 주소값을 반환한다.
  - 하지만 String, Date, Arrays class는 toString()으로 문자열을 반환한다.
  - 또한 커맨드 객체의 경우, 보통 toString()을 만들면 자신의 field 값을 반환하게 만든다.
- clone()의 경우 얕은 복사를 실행한다. 
  - 다만 Clonealbe interface를 구현한 class만 사용가능한 method다.
  - clone()으로 배열이나 list, 커맨드 객체를 복사할 때 특히 조심해야 한다. 
  - 기본형 배열은 참조가 없어 얕은 복사와 깊은 복사가 똑같다.
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
        }catch(CloneNotSupportedException e){

        }

        return (Circle)obj;
 
    }

    public Circle deepCopy(){
        Object obj = null;
        
        try{
            obj = super.clone();
        } catch (CloneNotSupportedException e){

        }

        Circle c = (Circle)obj;
        c.p = new Point(this.p.x, this.p.y);

        return c;
    }
}
```

- c1 등 instance는 각자 주소값을 지닌다. 복사한 클래스의 참조변수의 주소값은 모두 다르다.
  - 얕은 복사의 경우, 그 안에 들어있는 원소 p의 주소값은 전부 동일하다.
  - 깊은 복사의 경우, 그 안에 들어있는 원소 p의 주소값도 원본과 달라진다.
- 따라서 객체를 복사하려면 대부분 깊은 복사를 실행해야 한다.
```java
Circle c1 = new Circle(new Point(1,1),2.0); //주소값 0x100, p의 주소값 0x200
Circle c2 = c1.shallowCopy(); //주소값 0x300, p의 주소값 0x200
Circle c3 = c1.shallowCopy(); //주소값 0x400, p의 주소값 0x200
Circle c4 = c1.deepCopy(); //주소값 0x500, p의 주소값 0x600
```

<img src="/image/deep-copy.jpg" >



## <span style="color:#802548">_java의 autoBoxing란?_</span>
- 기본형 변수와 wrapper class 간에 자유롭게 오갈 수 있게 하는 JVM의 기능이다.
```java
int i = 5;
Integer iObj = new Integer(7); // new Integer는 이제 사용불가능하다. Integer.valueOf()를 사용한다.
int sum = i +iObj; //12
```

- int와 Integer를 더하려면 아래와 같이 unboxing이 필요하다.
```java
int sum = i + iObj.intValue(); //wrapper를 기본형으로 unboxing했다.
```

- 하지만 해당 작업을 sourcode에서 하지 않아도 compiler가 알아서 대신해준다.
- 0은 int라 Integer에 넣을 수 없지만, Integer로 compiler가 알아서 감싸서 넣는 소스코드로 바꿔 build한다.
```java
ArrayList<Integer>list = new ArrayList<Integer>();
list.add(0);
```

- 아래처럼 JVM이 변환해주는다는 의미다.
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

- for문에서 매번 boxing과 unboxing이 일어나지 않게 해야 한다.
- 간단하다. for 문 시작전에 int 값으로 변환해주면 된다.
```java
long sum = 0L;
int max = Integer.MAX_VALUE; // autoBoxing은 따로 뺴서 먼저 진행한다.

for(long i=0 ; i < max ; i++) { // for문에서는 int 기본타입끼리 비교한다. Integer비교는 시간과 메모리가 많이 든다.
    sum +=i;
}
System.out.println(sum);
```













































































