## <span style="color:#802548">_parallelStream_</span>
- java 8에서는 fork/join보다 더 간단한 stream api로 병행성을 달성할 수 있다.
- 순서대로 한다면 아래와 같을 것이다. 그럼 0.5초씩 x4라 2초 가량 걸린다.
```java
public List<String> stringTransform_upperCase(List<String> namesList){
    return namesList. //["Bob","Jamie","Jill","Rick"]
            stream()
            .map(String::toLowerCase)
            .collect(Collectors.toList());
}
```

- parallelism하게 수행하려면 아래와 같이 parallelStream()으로 열어준다.
- parallelism으로 stream을 열어주면 0.5초 가량 걸린다.
```java
public List<String> stringTransform_upperCase(List<String> namesList){
    return namesList
            .parallelStream()
            .map(String::toLowerCase)
            .collect(Collectors.toList());
}
```


## <span style="color:#802548">_parallelStream Junit Test_</span>
- parallelism stream을 unit test하고 싶다면 Junit을 활용한다.
```java
class ParallelismExampleTest {

    ParallelStreamsExample parallelismExample = new ParallelStreamsExample();

    @Test
    void stringTransform() {

        //given
        List<String> inputList = DataSet.namesList();

        //when
        startTimer();
        List<String> stringList = parallelismExample.stringTransform(inputList);
        timeTaken();

        //then
        assertEquals(4, stringList.size());
        stringList.forEach((name) -> {
            assertTrue(name.contains("-")); //*를 넣으면 exception이 난다. *가 아니라 -를 넣었기 때문..
        });
    }
}
```

- parallelStream으로 만들 수도 있지만 아래와 같이 만들수도 있다.
- 다만 코드를 읽는 데 일관성이 떨어지니 추천되지는 않는다.
```java
public List<String> stringTransform(List<String> namesList){
    return namesList
            .stream()
            .map(this::transform)
            .parallel()
            .collect(Collectors.toList());
}
```

- paralleStream을 sequential로 바꾸고 parallel로 바꾸면 결국 parallelism 진행이다. 
- 맨 마지막 method가 전체 방향을 결정한다. 다만 이런 식으로 쓰면 일관성이 떨어지니 parallelStream()이나 stream()으로 쓰자.
```java
public List<String> stringTransform(List<String> namesList){
    return namesList
            .parallelStream()
            .map(this::transform)
            .sequential()
            .parallel()
            .collect(Collectors.toList());
}
```


- parameterizedTest를 사용하면 같은 test를 다른 parameter를 사용하여 복수의 test를 진행할 수 있다.
- boolean에 false나 ture를 주고, @ValueSource를 준다.
- 그리고 사용할 변수명을 parameter로 지정한다.
```java
public List<String> stringTransform_1(List<String> namesList, boolean isParallel) {

    Stream<String> nameStream = namesList.stream();

    if(isParallel)
        nameStream.parallel();

    return nameStream
            .map(this::transform)
            .collect(Collectors.toList());
}


@ParameterizedTest
@ValueSource(booleans = {false, true})
void stringTransform_1(boolean isParallel) {

    //given
    startTimer();
    List<String> inputList = DataSet.namesList();

    //when
    List<String> stringList = parallelismExample.stringTransform_1(inputList,isParallel);
    timeTaken();
    stopWatchReset();

    //then
    assertEquals(4, stringList.size());
    stringList.forEach((name) -> {
        assertTrue(name.contains("-"));
    });
}
```
