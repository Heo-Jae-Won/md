## <span style="color:#802548">_collections란?_</span>

- Student class는 Comparable interface를 구현했다.
- 그 이유는 TreeSet을 구현하기 위해서는 Comparable interface가 필요하기 때문이다.
	- TreeSet은 compareTo로 구현된 기준으로 정렬을 실시해 자동 저장한다.
	- 그 정렬기준은 아래와 같다.

```java
this.name.compareTo(o.name) 		//this가 앞에 오면 오름차순
o.name.compareTo(this.name) 		//parameter 변수가 앞에 오면 내림차순
-1 * (this.name.compareTo(o.name)) 	//2번째의 좀 더 알아보기 쉬운 버전. 내림차순
```

- method 기준으로는 compareTo는 

```java
int priority = ( this.stdNum > o.stdNum ) ? 1 : 0;

if (priority == 0) {
	if ( this.stdNum > o.stdNum ) {
		priority = 1;
	} else if (this.stdNum < o.stdNum) {
		priority = -1;
	} else {
		priority = 0;
	}
}

```


```java
public class Student implements Comparable<Student> {
// compareTo()
	@Override
	public int compareTo(Student o) {
		int priority = -1 * (this.name.compareTo(o.name)) ;
		
		//아래는 가능한 식.
		//priority = ( this.stdNum > o.stdNum ) ? 1 : 0;
		
		//아래는 삽입불가능한 식. 내림차순 시 무조건 false가 뜨니까 0이 되는데,
		//0이 되면 같다고 판단되어 treeMap에서 넣을 수가 없다!!
		//priority = ( o.stdNum > this.stdNum ) ? 1 : 0;
		
		// 아래는 학번에 관한 오름차순
//		if (priority == 0) {
//			if ( this.stdNum > o.stdNum ) {
//				priority = 1;
//			} else if (this.stdNum < o.stdNum) {
//				priority = -1;
//			} else {
//				priority = 0;
//			}
//		}
		
		// 아래는 학번에 관한 내림차순
//		if (priority == 0) {
//			if ( this.stdNum > o.stdNum ) {
//				priority = -1;
//			} else if (this.stdNum < o.stdNum) {
//				priority = 1;
//			} else {
//				priority = 0;
//			}
//		}
		//하지만 this.stdNum - o.stdNum 처럼 하면 더 간단!
		//this가 먼저 나오면 오름차순
		//parameter가 먼저 나오면 내림차순
		// o.stdNum - this.stdNum보다는, -1 * (this.stdNum - o.stdNum)로 쓰는 게 좋을듯.
		
		if (priority == 0) {
			priority = -1 * (this.stdNum - o.stdNum); 
		}
		return priority;
	}
}
```

```java
public class Student implements Comparable<Student> {

	// 멤버변수
	private static final String ACADEMY = "DSA";
	private static int NUMBER = 46;
	private static int serialNum = 20241000;
	private String name;
	private int stdNum;
	private Score score;
	
	
	{
		serialNum = serialNum + 1;
		this.stdNum = serialNum;
	}
	
	// 생성자
	public Student(String name, Score score) {
		this.name = name;
		this.score = score;
	}
	
	@Override
	public boolean equals(Object obj) {
		if (this == obj)
			return true;
		if (obj == null)
			return false;
		if (getClass() != obj.getClass())
			return false;
		Student other = (Student) obj;
		return stdNum == other.stdNum;
	}
	
	// compareTo()
	@Override
	public int compareTo(Student o) {
		// this.name.compareTo(o.name); --> 이름 오름차순
		// o.name.compareTo(this.name); --> 이름 내림차순. 하지만 아래처럼 써주는 게 더 가독성 좋음.
		int priority = -1 * (this.name.compareTo(o.name)) ;
		
		//아래는 가능한 식.
		//priority = ( this.stdNum > o.stdNum ) ? 1 : 0;
		
		//아래는 삽입불가능한 식. 내림차순 시 무조건 false가 뜨니까 0이 되는데,
		//0이 되면 같다고 판단되어 treeMap에서 넣을 수가 없다!!
		//priority = ( o.stdNum > this.stdNum ) ? 1 : 0;
		
		// 아래는 학번에 관한 오름차순
//		if (priority == 0) {
//			if ( this.stdNum > o.stdNum ) {
//				priority = 1;
//			} else if (this.stdNum < o.stdNum) {
//				priority = -1;
//			} else {
//				priority = 0;
//			}
//		}
		
		// 아래는 학번에 관한 내림차순
//		if (priority == 0) {
//			if ( this.stdNum > o.stdNum ) {
//				priority = -1;
//			} else if (this.stdNum < o.stdNum) {
//				priority = 1;
//			} else {
//				priority = 0;
//			}
//		}
		//하지만 this.stdNum - o.stdNum 처럼 하면 더 간단!
		//this가 먼저 나오면 오름차순
		//parameter가 먼저 나오면 내림차순
		// o.stdNum - this.stdNum보다는, -1 * (this.stdNum - o.stdNum)로 쓰는 게 좋을듯.
		
		if (priority == 0) {
			priority = -1 * (this.stdNum - o.stdNum); 
		}
		return priority;
	}

	// toString() 사용
	@Override
	public String toString() {
		
		return String.format(
							"[%s-%sth] 이름: %s, 학번: %d, 점수: JAVA(%3d), DB(%3d), WEB(%3d)", 
							Student.ACADEMY, Student.NUMBER, this.getName(), this.getStdNum(), 
							this.score.getJava(), this.score.getDatabase(), this.score.getWeb()
						);
	}
}
```