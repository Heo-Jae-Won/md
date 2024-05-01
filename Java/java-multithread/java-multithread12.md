- Collection framework의 data structure에서 sequential과 parallel도 비교해보자.
- boolean 변수를 주어 parallel일 때와 아닐 떄를 비교해보자.
```java
public class ArrayListSpliteratorExample {
    public List<Integer> muultiplyEachValue(ArrayList<Integer> inputList, int multiplyValue, boolean isParallel) {

    startTimer();
    Stream<Integer> integerStream = inputList.stream();

    if(isParallel) {
        integerStream.parallel();
    }

    List<Integer> resultList = integerStream
                                    .map(integer -> integer * multiplyValue)
                                    .collect(Collectors.toList());

    timeTaken();

    return resultList;
    }
}
```

- 이제 만든 class를 test해보자.
- Test를 처음할 때보다 할때마다 더 빨라진다. 그를 보여주기 위해 @RepeatedTest를 썼다.
- 더 빨라지는 CPU가 cache를 활용하기 때문이다.
- 그리고 data만 나누는 경우에는, ArrayList를 쓰면 parallel이 더 빨라지긴 하지만, sequential에 비해 그정도로 빠르진 않다.
```java
class ArrayListSpliteratorExampleTest {
    ArrayListSpliteratorExample arrayListSpliteratorExample = new ArrayListSpliteratorExample();


    //@Test
    @RepeatedTest(5)
    void arrayListSpliteratorExample() {
        //given
        int size = 1_000_000;
        ArrayList<Integer> inputList = DataSet.generateArrayList(size);

        //when
        List<Integer> resultList = arrayListSpliteratorExample.muultiplyEachValue(inputList, true /*false */);

        //then
        assertEquals(size, resultList.size());
    }
}
```


- 그런데 linkedList를 쓰면 완전히 달라진다.
- parallelStream으로 linkedList를 쓰면 data를 split하는 게 어렵기 때문에 성능이 크게 저하된다.
- linkedList를 쓸 때는 sequential stream을 쓰는게 낫다.
```java
class ArrayListSpliteratorExampleTest {
    LinkedSpliteratorExample linkedSpliteratorExample = new LinkedSpliteratorExample();


    //@Test
    @RepeatedTest(5)
    void LinkedSpliteratorExample() {
        //given
        int size = 1_000_000;
        LinkedLIst<Integer> inputList = DataSet.generateLinkedList(size);

        //when
        List<Integer> resultList = linkedSpliteratorExample.muultiplyEachValue(inputList, true /*false */);

        //then
        assertEquals(size, resultList.size());
    }
}
```


- data collection에 따라 최종연산의 방식도 달라진다.
- list를 아래와 같이 parallelstream에 넣어 돌려도 order는 유지된다.
```java
public class ParallelStreamResultOrder {

    public List<Integer> listOrder(List<Integer> inputList) {
        inputList.parallelStream()
                .map(integer -> integer * 2)
                .collect(Collectors.toList());
    }

    public static void main(String[] args) {
        List<Integer> inputLIst = List.of(1,2,3,4,5,6,7,8);
        log("inputList: " + inputList); //[1.2.3.4.5.6.7.8]
        List<Integer> result = listOrder(inputList);
        log("result: " + result);       //[2.4.6.8.10.12.14.16]
    }
}
```

- 반면에 set은 그렇지 않다.
- parllelStream의 경우, set은 order 유지를 보장해주지 않는다.
```java
public List<Integer> setOrder(List<Integer> inputList) {
        inputList.parallelStream()
                .map(integer -> integer * 2)
                .collect(Collectors.toSet());
    }

public static void main(String[] args) {
        Set<Integer> inputLIst = Set.of(1,2,3,4,5,6,7,8);
        log("inputList: " + inputList); //[1.2.3.4.5.6.7.8]
        Set<Integer> result = SetOrder(inputList);
        log("result: " + result);       //[16.2.4.6.8.10.12.14]
    }
```

- collect와 reduce는 같은 결과를 같게끔 만들 수 있다.
- 그러나 메모리 관점에서는 collect는 mutable이며, reduce는 immutable이다.
- reduce는 중간연산도 immutable하기 때문에 전부 쓰레기 데이터로 남는다.
- 나중에 GC가 잡아먹기야 하겠지만.. colelct도 가능하다면 reduce보단 collect를 쓰는 게 더 낫다.
```java
public class CollectVsReduce {

    public static String collect() {
        List<String> list = DataSet.namesList();

        String result = list.
                parallelStream().
                        collect(Collectors.joining());

        return result;
    }

    public static String reduce() {

        List<String> list = DataSet.namesList();

        String result = list.
                parallelStream().
                reduce("", (s1, s2) -> s1 + s2);

        return result;
    }

    public static void main(String[] args) {

        log("collect : "+ collect());
        log("reduce : "+ reduce());
    }
}
```

- parallelStream은 sort 연산, boxing, unboxing 연산이 진행되도 느려진다. 그 때는 sequential로 쓰는게 낫다.



