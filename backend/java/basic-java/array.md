## <span style="color:#802548">_배열이란?_</span> 
- 배열은 같은 타입의 여러 변수를 하나의 묶음으로 다룬다.
- int[] score =new int[5]라고 하면 5개의 길이를 가진 int type의 배열이 생성되며, 배열은 기본형으로 type을 갖더라도 무조건 참조형 변수이다. 
  - 여기서 score라는 식별자는 배열을 다루는 데 필요한 참조변수이지, 값을 저장하기 위한 공간이 아니다. 
  - 즉 그저 객체의 주소일 뿐이다. 실제 저장 값은 주소 안에 존재한다. 
  - 배열의 초기화 값은 int는 0으로 초기화되어 값이 존재한다. 그게 싫다면 본인이 초기화값을 넣으면 된다. 
  - boolean은 false이며, String은 null이다. 참조변수는 전부 자동 초기화값이 null이다. 
- 초기화 값을 정하고 싶다면 아래와 같이 정할 수 있다.
```java
int[] score = new int[5];
String[] name = {"Kim", "Park", "Yi"};
```

- 메모리 할당은 아래와 같이 이뤄진다.
<img src="/image/memory-allocation1.jpg">

- 실제는 아래와 같이 이뤄진다고 보면 된다.
<img src="/image/memory-allocation2.jpg">


- arr라는 식별자는 위에서 말했듯 값이 아니라 주소만 담고 있다.
- 따라서 위와같이 쓰면 I@16자리해쉬값과 같은 형태로 나오게 된다. 
  - 값을 꺼내려면 for문으로 아래와 같이 써준다.
  - 아니면 Arrays class의 toString을 써도 된다.
```java
for(int i=0;i<arr.length;i++){
	Sysout(arr[i])
}
Sysout(Arrays.toString(arr))
``` 


## <span style="color:#802548">_배열의 복제란?_</span> 
- 배열은 한 번 만들어지면 공간이 확정되고 공간을 바꿀 수 없다. 공간이 부족하면 새로운 배열에 복사해줘야 한다.  
- for문으로 복사해줄 수 있다. 참조가 사라진 temp는 GC가 나중에 잡아먹는다.
```java
int[] arr = new int[5]; 
int[] temp = new int[arr.length * 2] 
for(int i = 0; i<arr.length; i++){
	Temp[i]=arr[i]
}
arr = temp;
```


- 그 안에 담긴 로직은 아래와 같다.
- 참조를 잃은 arr의 이전 주소값 0x100은 GC가 치워버린다.
<img src="/image/array-copy-memory-address1.jpg" >
<img src="/image/array-copy-memory-address2.jpg" >



- 하지만 배열의 복사는 for문을 사용하기 보다는 System.arraycopy()를 사용하는 게 더 효율적이다. 
- 요소를 1개씩 이동시키지 않고 통째로 카피가 가능하기 때문이다. 아래와 같이 쓰면 전부 복사가 가능하다.
```java
Char[] abc = {'A','B','C','D'};
Char[] num ={'0','1','2','3','4','5'};
Char[] result = [abc.length + num.length];
System.arraycopy(abc, 0, result, 0, abc.length); //abc배열의 0번 index부터 복사해서 result 배열의 0번 index에 채워넣음. 복사할 element의 수는 abc.lenght만큼.
System.arraycopy(num, 0, result, 0, num.length);
```

## <span style="color:#802548">_배열의 활용이란?_</span> 
- 평균내기 
  - score는 int형이지만 float형과 나눌 경우 해당 값은 float가 된다.
```java
int[] score = {100,88,100,100,90};
int sum=0;
float average = 0f;
for(int i =0; i<score.length; i++){
	Sum += score[i]
}
average = sum / (Float)score.length;
```


- 최대최소값 가져오기
```java
int[] score = {79,88,91,33,100,55,95};
int max = score[0];
int min = score [0];
for(int i =0; i<score.length; i++){
   if(max <score[i]){
	   max = score[i]
   }

   if(min > score[i]){
	   min = score[i]
   }
}
```


- 랜덤하게 숫자 순서 바꾸기
```java
int[] numArr = { 0,1,2,3,4,5,6,7,8,9};
for(int i =0; i<numArr.length;i++){
	int n = (int)(Math.randow * 10) // 0~9
	int tmp = numArr[i];
	numArr[i] = numArr[n];
	numArr[n] = tmp;
} 
```


- 불연속 값으로 배열 채우기
  - 실행마다 결과가 다르지만 10개의 길이 배열은 반드시 code의 element로 구성되게 된다.
```java
int[] code = {-4,-1,3,6,11};
int[] arr= new int[10];
for(int i =0; i<arr.length;i++){
	int temp = (int)(Math.random() * code.length); //﻿ code 배열에 있는 모든 값이 원소가 될 가능성이 있게 한 것이다.
	int arr[i] = code[temp];
} 
```


- 버블정렬.오름차순
  - 할당해서 numArr이 {1,3,4,4,2,1,3,8,4,3}가 되었다면
    - 1342134438
    - 1321344348
    - 1213343448
    - 1123334448
  - 위처럼 오름차순 정렬된다. 오름차순은 큰수는 뒤에 가게된다.
  - 내리참순 정렬은 if문을 numArr[j] < numArr[j+1]로 바꿔주면 된다.
```java
int[] numArr = new int[10];
for(int i =0; i < numArr.length; i++){
	Sysout(numArr[i] = (int)(Math.random() * 10)); //﻿numArr에 값을 할당하면서 동시에 출력한다.
}
Sysout();
for(int i =0; i<numArr.length - 1; i++){ //10개의 숫자는 9번의 정렬 필요.
	boolean changed = false;
	For(int j=0;j<numAArr.length-1-i;i++){ //1번째 정렬은 9번을 비교함/ 2번째 정렬은 8번을 비교. 1번씩 줄어드는 이유는 맨 마지막 값은 늘 정렬을 마쳐 최댓값이 들어오기 때문임.
		if(numArr[j] > numArr[j+1]){ // 0-1, 1-2, 2-3 등 인접한 숫자끼리 비교해 바꾼다. 
			int temp = newArr[j];
			numArr[j] = numArr[j+1];
			numArr[j+1] = temp;
			changed = true;
		}
	}
	if(!changed)
		break;
	for(int k=0; k<numArr.length;k++){
		Sysout(numArr[k])
	}
	Sysout();
}
```


- element 갯수 세기
```java
int[] numArr = new int[10];
int[] counter = new int[10];

for(int i =0; i<numArr.length; i++){
	numArr[i]= (int) (Math.random() * 10);
	sysout(numArr[i]);
}
Sysout();
for(int i=0; i<numArr.length; i++){
	Counter[numArr[i]]++
}

for(int i=0; i<numArr.length;i++){
	Sysout(i+"의 개수: "+ counter[i]);
}
```


## <span style="color:#802548">_2차원 배열의 활용이란?_</span> 
- 2차원 배열의 가장 큰 특징은 유연성이다. 가변배열이 가능하다.
```java
int[][] arr = new int[][]{{1,2,3},{4,5,6}};
```


- 다만 보기 쉽게 아래와 같이 쓸 수도 있다. 
```java
int[][] arr = new int[][]{
                            {1,2,3},
                            {4,5,6}
                        };
```						

- 가변배열은 아래와 같이 2차원의 값을 지정하지 않고 차후에 지정함으로써 가변배열을 만들 수 있다.
```java
int[][] score = new int[5][];
score[0] = new int[4];
score[1] = new int[3];
score[2] = new int[2];
score[3] = new int[2];
score[4] = new int[5];
```

- 위와 아래는 동일한 갯수의 가변배열이다. 
```java
int[][] score = {
		{100,100,100,100},
		{20,20,20},
		{30,30},
		{40,40},
		{10,20,30,40,50}
}
```


​

​

​


