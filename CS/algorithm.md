- 오름차순 정렬

- i = 0 일때 i가 -1이라서 접근해올 떄 exception
```java
int[] numberList = {45,34,21,611,53,55,73};
		
int temp = 0;
for(int i = 1; i <= numberList.length  ; i++) {
    if (numberList[i] < numberList[i -1]){
        temp = numberList[i-1];
        numberList[i] = numberList[i-1];
        numberList[i-1] = temp;
    }
}

for(int element : numberList) {
    System.out.println(element);
}
```


- 7번쨰 index를 접근해오려고 해서 exception
```java
int[] numberList = {45,34,21,611,53,55,73};

int temp = 0;
for(int i = 1; i <= numberList.length; i++) {
    if (numberList[i+1] < numberList[i]){
        temp = numberList[i];
        numberList[i] = numberList[i+1 ];
        numberList[i+1] = temp;
    }
}
for(int element : numberList) {
    System.out.println(element);
}
```

- 정렬이 제대로 수행되지 않음
```java
int[] numberList = {45,34,21,611,53,55,73};

int temp = 0;
for(int i = 1; i < numberList.length; i++) {
    if (numberList[i] < numberList[i]){
        temp = numberList[i];
        numberList[i] = numberList[i+1 ];
        numberList[i] = temp;
    }
}
for(int element : numberList) {
    System.out.println(element);
}
```


- 오름차순 정렬은 실행회차마다 정렬 대상 element가 늘어난다.
- element갯수만큼 그 이전 element 전부 똑같이 다시 실행해줘야 한다
- 따라서 아래 같이 이중 for문으로 만듦
```java
int[] numberList = {45,34,21,611,53,55,73};
		
int temp = 0;
for(int i = 0; i < numberList.length; i++) {
    for(int j = i; j > 0; j--) {
        if (numberList[j] < numberList[j-1]){
            temp = numberList[j-1];
            numberList[j-1] = numberList[j];
            numberList[j] = temp;
        }
    }
}

for(int element : numberList) {
    System.out.println(element);
}
```

- i와 j로는 뜻을 나타내기 어려우므로 변경
  - 정렬횟수는 iterationCount, 정렬 포인트시작점은 sortStartIndex
- 처음 반복 시에는 sort 시작지점이 34.
- 그 다음 반복 시에는 sort 시작지점이 21.
  - 이렇게 반복 회차마다 정렬에 포함되는 element가 많아진다.
```java
int[] numberList = {45,34,21,611,53,55,73};
		
int temp = 0;
for(int iterationCount = 0; iterationCount < numberList.length; iterationCount++) {
    for(int sortStartIndex = iterationCount; sortStartIndex > 0; sortStartIndex--) {
        if (numberList[sortStartIndex] < numberList[sortStartIndex-1]){
            temp = numberList[sortStartIndex-1];
            numberList[sortStartIndex-1] = numberList[sortStartIndex];
            numberList[sortStartIndex] = temp;
        }
    }
}

for(int element : numberList) {
    System.out.println(element);
}
```