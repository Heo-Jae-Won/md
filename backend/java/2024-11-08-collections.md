## <span style="color:#802548">_Arrays_</span>
- 배열을 다루는 class며 util method가 많다.

```java
int[] nums = new int[]{1,2,3,4,5};
Arrays.toString(nums) // [1,2,3,4,5]

int[] arr2 = Arrays.copyOf(arr, arr.length); //깊은복사
Arrays.sort(arr2);                                              

List<Integer> numList = Arrays.asList(nums);                    //unmodifiableList. numList를 Collections.sort() 시 unsupportedOperationException
numList.add(6); //error. unmodifiableList.

List<Integer> numList = new ArrayList<>(Arrays.asList(nums));   //변경 가능.  numList를 Collections.sort() 시 ok
```

## <span style="color:#802548">_collection framework_</span>
- 데이터군을 저장하는 클래스들을 표준화한 설계라는 뜻이다.
- 배열은 method가 없는데, 이를 극복하는 겸, 다양한 형태의 자료구조를 구현하는 겸 실현되었다.

- 우리가 실제로 쓰는 map, list, set은 많은 경우 collection의 것을 그대로 가져다 쓴다.
- 그 중 list와 set은 매우 유사하여 Collection interface로 상속 계층도를 구현했다.
- collection interface는 아래와 같은 method를 지니고 있다.

```java
int size();
boolean isEmpty();
boolean add(Object e);
Iterator<E> iterator();
boolean addAll(Collection<? extends E> c);
boolean remove(Object o);
boolean removeAll(Collection<?> c);
void sort(Comparator c);
void clear();
boolean retainAll(Collection<?> c);
default Stream<E> stream() {
    return StreamSupport.stream(spliterator(), false);
}
```

- map은 collection interface에는 포함되어 있지 않다. 
- 구현법이 다르기 때문이다.
- 하지만 사용법은 크게 다르지 않다.

```java
boolean containsKey();
Set entrySet();
Set keySet();
Object put(Object key, Object value);
Object remove(Object key);
int size();
```

## <span style="color:#802548">_collections class_</span>
- Collections class는 collection interface 뿐만 아니라, map을 포함한 collection framework에 관련된 util method를 제공한다.
- multi threading에서 동기화가 핋요할 땐 아래와 같이 synchronized method를 사용한다.
- method의 이름이 같은 것을 보면, overloading되었다는 사실을 알 수 있다.

```java
static Collection synchronizedCollection(Collection c);
static List synchronizedList(List c);
static Set synchronizedSet(Set c);
static Map synchronizedMap(Map c);
static SortedSet synchronizedSortedSet(SortedSet c);
static SortedMap synchronizedSortedMap(SortedMap c);

List<Integer> syncList = (ArrayList<Integer>)Collections.synchronizedCollection(new ArrayList());
List<Integer> syncList = Collections.unmodifiableList(new ArrayList());
```

- 변경불가 컬렉션을 만드는 방법은 아래와 같다.

```java
static Collection unmodifiableCollection(Collection c);
static List unmodifiableList(List list);
static Set unmodifiableSet(Set c);
static Map unmodifiableMap(Map c);
static NavigableSet unmodifiableNavigableSet(NavigableSet c);
static SortedSet unmodifiableSortedSet(SortedSet c);
static NavigableMap unmodifiableNavigableMap(NavigableMap m);
static SortedMap unmodifiableSortedMap(SortedMap m);

Map<String, Object> syncList = (HashMap<String, Object>) Collections.unmodifiableCollection(new HashMap<>());
Map<String, Object> syncList = Collections.unmodifiableMap(new HashMap<>());
```

- collections class에서 자주 쓰는 method는 아래와 같다.

```java
Collections.sort();
Collections.reverse();
Collections.reverseOrder();
Collections.max();
Collections.min();
Collections.emptyList();
Collections.emptyMap();
```

- 자세한 용례를 살펴보기 위해 아래와 같이 list를 만들어보자.
- linked든 Array든 상관없다.

```java
public static void main(String[] args) {
    List<String> aName = new LinkedList<>();

    aName.add("doggy");
    aName.add("Strange");
    aName.add("World");
    aName.add("Alice");
    aName.add("Ironman");


    for (String s : aName) {
        System.out.print(s + " ");
    }
    System.out.println(); // doggy Strange World Alice Ironman 
    //
}
```

- sort는 오름차순 정렬을 natural order로 하며, case-sensitive하게 작동한다.
- doggy가 d라서 A와 I 사이에 있어야 할 거 같지만, 결과는 doggy가 뒤로 간다.
- unicode 상 doggy가 더 값이 크기 때문이다.

```java
// Collection에 있는 객체를 정렬.
System.out.println("=======sort========");
Collections.sort(aName);
for (String s : aName) {
    System.out.print(s + " ");
}
System.out.println(); //Alice Ironman Strange World doggy 
```

- 따라서 case-insensitive하게 정렬하고 싶다면 해당 comparator를 구현해야 한다.
- 다행히 java api 상에 구현되어 있는 comparator가 있다. 해당 comparator를 사용하자.

```java
System.out.println("=======sort========");
Collections.sort(aName, String.CASE_INSENSITIVE_ORDER);
for (String s : aName) {
    System.out.print(s + " ");
}
System.out.println();
```

- reverOrder는 자신이 원하는 정렬기준을 Comparator를 통해 반대로 구현할 수 있다. sort()의 반대다.
- reverseOrder도 마찬가지로 String의 경우 case-sensitive하다.

```java
System.out.println("=====reverse_order=======");
aName.sort(Collections.reverseOrder());
for (String s : aName) {
    System.out.print(s + " ");
}
System.out.println(); //doggy World Strange Ironman Alice
```

- incase-sensitive하게 정렬하면 우리가 생각했던 정렬방식으로 나온다.

```java
System.out.println("=====reverse_order_not_case_sensitive=======");
aName.sort(Collections.reverseOrder(String.CASE_INSENSITIVE_ORDER));
for (String s : aName) {
    System.out.print(s + " ");
}
System.out.println(); //World Strange Ironman doggy Alice 
```

- case-sensitive하게 만들어 산출한 max 값은 doggy, min 값은 Alice다.
- unicode 값에 의한 결과다. unicode 상의 숫자가 소문자가 더 크기 때문이다.

```java
String Max = Collections.max(aName); //doggy
String Min = Collections.min(aName); //Alice
```

- case-insensitive하게 만들어 산출한 max 값은 World, min 값은 Alice다.
- 사실 max, min은 숫자를 담는 Collections에서 더 자주 쓰인다.

```java
String Max = Collections.max(aName, String.CASE_INSENSITIVE_ORDER); //World
String Min = Collections.min(aName, String.CASE_INSENSITIVE_ORDER); //Alice
```

- 빈 list, map을 return하는 것도 자주 쓴다.
- 오류에 의해 값을 가져오지 못했을 때, exception을 내도 front에는 빈 list, map으로 보내줄 때 유용하다.

```java
Collections.emptyList();
Collections.emptyMap();
```


- reverse는 저장 순서와 반대로 정렬한다. reverse()는 우리가 생각하는 논리적인 sort()의 반대가 아니다.
- reverse를 쓸 일은 거의 없다. business 요건 상에 저장 순서의 반대를 쓸 이유가 없기 때문이다.

## <span style="color:#802548">_List_</span>

- list는 중복을 허용하면서 저장순서가 유지되는 collection framework다.
- list는 ArrayList와 LinkedList, Vector가 있다.
  - Vector는 멀티스레딩에서 쓴다. 하지만 Vector보다는 synchronized arrayList를 구현하는 게 좋다.
- list interface의 method는 list interface가 자체로 갖고 있는 것과 collection interface에서 물려받은 것이 섞여있다.
- list interface가 갖고 있는 것에는 get, subList가 대표적이다.

```java
E get(int index); //E는 <>에 들어오는 원소의 type
List subList(int fromIndex, int toIndex) //from부터 to 이전까지 index에 든 값을 반환
```

- list를 삭제할 때는 유의해야 한다.
- index를 0부터 시작시키는 게 아니라 거꾸로 맨끝에서부터 0으로 이동해야 한다.
- 삭제를 하는 순간 빈 공간을 채우기 위해 나머지 요소들이 index를 이동하기 때문이다.

```java
for(int i = list2.size() - 1; i >= 0; i--){
    if(list1.contains(list2.get(i)))
        list2.remove(i); // AA,B,C
}

for(int i = 0; i < list2.size() - 1; i++){
    if(list1.contains(list2.get(i)))
        list2.remove(i); // Exception return. 위와 같이 i를 --하는 방향으로 해야..
}
```

- 문자열을 10개씩 잘라서 list에 보관하는 예시다.

```java
final int LIMIT = 10;
String source = "0123456789abcdefghijABCDEFGHIJ!@#$%^&*()ZZZ";
int length = source.length();

List<String> list = new ArrayList(length/LIMIT + 10); //크기는 원래 예상보다 10정도 여유있게 잡자. 배열을 이용하기 때문에 용량이 꽉차면 새로운 배열을 생성하고 기존 배열을 복사해서 만드는 형태이므로 시간이 오래 걸린다.

for(int i = 0; i < length; i += LIMIT){
    if(i + LIMIT < length)
        list.add(source.substring(i, i+LIMIT)); //10자씩 끊어먹기
    else
        list.add(source.substring(i)); //끊어먹을 게 10자가 안되면 끝까지 다 넣기
}

for(int i = 0; i < list.size(); i++){
    Sysout(list.get(i));
    //0123456789
    //abcdefghij
    //ABCDEFGHIJ
    //!@#$%^&*()
    //ZZZ
}
```

## <span style="color:#802548">_Vector_</span>

- 아래는 Vector class다.
- 실제 Vector의 구현로직 핵심은 아래와 같다.
- 내부에서는 배열을 이용한다. 삭제는 해당 index 값이후의 배열값을 그 index로 옮기고 마지막 index 값을 null로 만드는 작업이다.
- 따라서 크기를 넘어서는 삽입, 삭제 시에는 새로운 배열을 복사 생성하기 때문에 오래걸린다.
- Vector 보다는 ArrayList를 사용하자.

```java
public class MyVector implements List{
    Object[] data = null;
    int capacity = 0;
    int size = 0;

    public MyVector(int capacity){
        if(capacity < 0){
            throw new IllegalArgumentException("유효하지 않은 값입니다. :" + capacity);
        }

        this.capacity = capacity;
        data = new Object[capacity];
    }

    public MyVector(){
        this(10);
    }

    public void ensureCapacity(int minCapacity){
        if(minCapacity - data.length > 0){
            setCapacity(minCapacity);
        }
    }

    public boolean add(Object obj){
        ensureCapacity(size + 1);
        data[size++]  = obj;
        return true;
    }

    public Object get(int index){
        if(index < 0 || index >= size){
            throw new IndexOutOfBoundsException("범위를 벗어났습니다.");
        }
        return data[index];
    }

    public Object remove(int index){
        Object oldObj = null;

        if(index < 0 || index >= size){
            throw new IndexOfBoundsException("범위를 벗어났습니다.");
        }

        oldObj = data[index];

        if(index != size -1){
            System.arraycopy(data, index+1, data, index, size-index-1);
        }

        data[size-1]= null;
        size--;
        return oldObj;
    }

    public boolean remove(Object obj){
        for(int i = 0; i <size; i++){
            if(obj.equals(data[i])){
                remove(i);
                return true;
            }
        }
    }

    public void trimToSize(){
        setCapacity(size);
    }

    private void setCapacity(int capacity){
        if(this.capacity == capacity){
            return;
        }

        Object[] tmp = new Object[capacity];
        System.arraycopy(data,0,tmp,0,size);
        data = tmp;
        this.capacity = capacity;
    }

    public void clear(){
        for(int i = 0; i < size; i++){
            data[i] = null;
        }
        size = 0;
    }

    public Object[] to Array(){
        Object[] result = new Object[size];
        System.arraycopy(data, 0 ,result, 0, size);

        return result;
    }

    public boolean isEmpty(){
        return size == 0;
    }

    public int capacity(){
        return capacity;
    }

    public int size(){
        return size;
    }
}
```

## <span style="color:#802548">_LinkedList_</span>

- LinkedList는 배열과 다르게 각 주소값들도 모두 메모리 공간을 차지한다.
- 배열에서는 공간은 0x100 고정에 값이 객체인 경우에 객체기 때문에 메모리 공간을 차지했던 것과는 다른 모양새다.
- 단순 LinkedList는 단방향이라서 이전요소를 접근하는 것이 매우 어렵다.

```java
class Node {
    Node next; //다음 주소만 저장
    Object obj;
}
```

- 그래서 DoubleLinkedList가 나왔고, Java에서 LinkedList class는 DoubleLinkedList로 구현된다.

```java
class Node {
    Node next;     //다음 요소의 주소를 저장
    Node previous; //이전 요소의 주소를 저장
    Object obj;    //데이터를 저장
}
```

- 삽입삭제에 대해서는 LinkedList가 빠르고, 탐색에서는 느리다고 알려져있다.
- 하지만 순차적으로 추가, 삭제하는 경우에는 ArrayList가 더 속도가 빠르다.
- 더군다나 실제로는 객체들이 메모리에서 연속되어 있지 않은 점으로 인해 LinkedList가 예상보다 성능이 떨어진다.
    - 캐시나 버퍼를 통해 이득을 보기 어렵다는 점, 
    - 메모리를 많이 잡아먹는다는 점 등이 작용해 잘 쓰이지 않는다.
- 데이터를 저장할 때는 순차적으로 저장하므로 Linked를 썼다가, 도중 추가 작업 시에 LinkedList로 옮기는 방법도 존재한다.
- collection interface끼리는 서로 생성자에 넣으면 자동 변환된다.

```java
List<String> al = new ArrayList<>();
for (int i = 0; i < 100000; i++) {
    aal.add(i + " ");
}

List<String> ll = new LinkedList<>(al);
for (int i = 0; i < 1000; i++) {
    ll.add(500, "X");
}
```

## <span style="color:#802548">_Stack/Queue_</span>

- list collection framework로 만들 수 있는 것이 바로 stack과 queue다.
- stack은 LIFO고, queue는 FIFO다.
  - stack은 순차적으로 데이터를 추가하고 삭제하기 때문에 ArrayList를 사용한다
- queue는 데이터를 꺼낼때마다 처음 들어온 것을 꺼내기 때문에 배열을 새로 만들어 복사해야 한다.
  - 따라서 ArrayList는 비효율적이고, LinkedList를 쓰는 게 효율적이라 LinkedList로 구현한다.
- stack을 실제로 구현해보자.
  - Stack에 5개를 넣었다고 해보자. 그럼 크기가 5인 배열이 형성된다.
  - stack은 collection framework가 만들어지기 전부터 존재했던 것이라, Vector를 extends한다.

```java
class MyStack extends Vector{
    public Object push(Object item){
        addElement(item);
        return item;
    }

    public Object pop(){
        Object obj = peek();
    }

    public Object peek(){
        int len = size();

        if(len ==0)
            throw new EmptyStackException();

        return elementAt(len -1);
    }

    public boolean empty(){
        return size() == 0;
    }

    public int search(Object o){
        int i = lastIndexOf(o);  //stack은 맨 위에 저장된 객체의 index를 1로 정의한다. 따라서 indexOf가 아닌 lastIndexOf다.
        //A를 찾는다면 i는 3이 return된다.

        if(i >= 0){
            return size() -i; //따라서 searach 시 A의 위치는 2다. 이유는 Stack은 맨위에 쌓인 순서가 0이 아닌 1부터 시작하기 때문이다.
        }

        return -1;
    }
}
```

- queue는 특정 class가 아니라 interface다. 
- 따라서 LinkedList도 queue이므로 LinkedList를 공부하면 queue를 알 수 있다.
- 이제는 stack과 queue를 실제로 사용해보자.

- stack으로 구현한 브라우저의 앞뒤로 가기 예시다.

```java
public class StackEx1{
    public static Stack back = new Stack();
    public static Stack forward = new Stack();

    public static void main(String[] args){
        goURL("1.네이트");
        goURL("2.야후");
        goURL("3.네이버");
        goURL("4.다음");

        printStatus();

        goBack();
        sysout("= '뒤로' 버튼을 누른 후 =");
        printStatus();

        goBack();
        sysout("= '뒤로' 버튼을 누른 후 =");
        printStatus();

        goForward();
        sysout("= '앞으로' 버튼을 누른 후 =");
        printStatus();

        goURL("codechobo.com");
        sysout("= 새로운 주소로 이동 후 =");
        printStatus();
    }

    public void printStatus(){
        sysout("back: "+ back);
        sysout("forward: forward");
        sysout("현재화면은 '" + back.peek() + "' 입니다");
        sysout();
    }

    public static void goURL(String url){
        back.push(url);
        if(!forward.empty()){
            forward.clear();
        }
    }

    public static void goForward(){
        if(!forward.empty()){
            back.push(forward.pop())
        }
    }

    public static void goBack(){
        if(!back.empty()){
            forward.push(back.pop())
        }
    }
}
```

- stack으로 구현한 수식 괄호 검사 예시다.
- 괄호 한 개가 들어오면 저장해두고 있다가 다음 괄호를 만나면 pop으로 비우는 형태다.
- 괄호가 남아있다면 수식이 잘못됐다는 의미다.

```java
public class ExpValidCheck{
    public static void main(String[] args){
        if(args.length != 1){
            sysout("Usage: java ExpValidCheck \"EXPRESSION\"");
            sysout("Example: java ExpValidCheck \"EXPRESSION\"");
            System.exit(0);
        }

        Stack st = new Stack();
        String expression = args[0];

        sysout("expression: " + expression);

        try{
            for (int i = 0; i <expression.length(); i++) {
                char ch = expression.charAt(i);

                if (ch == '(') {
                    st.push(ch + "");
                } else if (ch == ')') {
                    st.pop();
                }
            }
            if (st.isEmpty()) {
                sysout("괄호가 일치합니다.");
            } else {
                sysout("괄호가 일치하지 않습니다.");
            }
        } catch(EmptyStackException e) {
            sysout("괄호가 일치하지 않습니다.");
        }
    }
}
```

- queue로 구현한 최근사용문서 버퍼 예시다.

```java
class QueueEx1 {
    static Queue queue = new LinkedList();
    static final int MAX_SIZE = 5;

    public static void main(String[] args){
        sysout("help를 입력하면 도움말을 볼 수 있습니다.");

        while(true){
            sysout(">>");

            try {
                Scanner s = new Scanner(System.in);
                String input = s.nextline().trim();

                if ("".equals(input)) {
                    continue;
                }

                if (input.equalsIgnoreCase("q")) {
                    System.quit(0);
                } else if (input.equalsIgnoreCase("help")) {
                    sysout(" help - 도움말을 보여줍니다.");
                    sysout(" q 또는 Q - 프로그램을 종료합니다.");
                    sysout(" history - 최근에 입력한 명령어를 "
                                + MAX_SIZE _"개 보여줍니다.");
                } else if ("history".equalsIgnoreCase(input)) {
                    int i = 0;
                    save(input);

                    //queue가 참조 타입이면 list가 아니라서 listIterator를 돌릴 수 없다.
                    //따라서 강제 형변환
                    LinkedList tmp = (LinkedList) queue;
                    ListIterator it = tmp.listIterator();

                    while (it.hasNext()) {
                        sysout(++i + "." + it.next());
                    }
                } else {
                    save(input);
                    sysout(input);
                }
            } catch(Exception e) {
                sysout("입력오류입니다.");
            }
        }
    }

    public static void save(String input){
        if (!"".equals(input)) {
            queue.offer(input);
        }

        if (queue.size() > MAX_SIZE) {
            queue.remove();
        }
    }
}
```

## <span style="color:#802548">_Iterator_</span>

- iterator를 통해 collection interface의 내용을 읽어올 수 있다.

```java
public interface Iterator{
    boolean hasNext();
    Object next();
    void remove();
}

public interface Collection{
    ...
    public Iterator iterator();
}
```

- Map은 collection interface에는 속하지 않아 iterator의 사용이 불가능하다.
- 대신 entrySet, keySet, valueSet 등으로 우회하여 접근할 수 있다.

```java
Map map = new Map<>();

Set entrySet = map.entrySet();
Iterator it = eSet.iterator();
sysout(it.next());
```

- 그러나 iterator를 쓰는 것보다 아래와 같이 쓰는 게 더 빠르다.

```java
Map <String,Object> map = new HashMap<>();
map.put("ddd", "bac4");
map.put("ddf", "ba1");
map.put("dd1", "ba2");
map.put("dd2", "bac3");

StringBuilder result = new StringBuilder();;
for (String key : map.keySet()) {
  String value = (String) map.get(key);
  result.append("key : ")
        .append(key)
        .append(", value: ")
        .append(value)
        .append("\n");
}
System.out.println(result);
```

- list의 경우에는 iterator를 아래와 같이 쓸 수 있다.

```java
Collection c = new ArrayList();
c.add("1");
c.add("2");
c.add("3");
Iterator it = c.iterator(); //iteraotr()도 되지만, list라면 listIterator()가 나음

while(it.hasNext()){
    sysout(it.next()); //[1,2,3]
}
```

- iterator 대신 listIterator를 사용하면 역방향으로 읽는 것까지 가능하다.
- 하지만 List interface를 구현해야만 가능하다.

```java
List list = new ArrayList();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
list.add("5");

ListIterator it = list.listIterator();

while(it.haxNext){
    System.out.print(it.next()); //1,2,3,4,5
}
sysout();

while(it.hasPrevious()){
    System.out.print(it.previous()); //5,4,3,2,1
}

sysout();
```

- 아래는 Iterator를 구현하는 방법에 대한 예시다.
- JDK 11 미만에서는 for문에서 그냥 collection element를 삭제하면 예외가 발생한다.
- 그러나 iterator 중에 삭제한다면 exception이 발생하지 않는다.
  - cursor가 자동으로 옮겨가기 때문이다.

```java
public class MyVector2 extends MyVector implements Iterator {
    int cursor = 0; //앞으로 읽어올 요소의 위치
    int lastRet = -1; //마지막으로 읽어온 요소의 위치

    public MyVector2(int capacity){
        super(capacity);
    }

    public MyVector2(){
        super(10);
    }

    public String toString(){
        String tmp = "";
        Iterator it = iterator();

        for(int i =0; it.hasNext(); i++){
            if(i !=0){
                tmp +=", ";
            }
            tmp += it.next();
        }

        return "[" + tmp + "]";
    }

    public Iterator iterator(){
        cursor = 0;
        lastRet = -1;
        return this;
    }

    public boolean hasNext(){
        return cursor != size();
    }

    public Object next(){
        Object next = get(cursor);
        lastRet = cursor++;
        return next;
    }

    public void remove(){
        if(lastRet == -1){
            throw new IllegalStateException();
        }else{
            remove(lastRet);
            cursor--;
            lastRet = -1;
        }
    }
}
```

## <span style="color:#802548">_Set_</span>

- set은 HashSet과 TreeSet이 있다.
- HashSet은 순서를 유지하지 않는다. 만약 순서를 유지하고 싶다면 LinkedHashSet을 사용해야 한다.
- set은 값이 곧 key다. 따라서 중복된 값은 저장되지 않는다. 똑같아 보인다면 type이 다른 것이다.

```java
Set<Integer> set = new HashSet();

for(int i = 0; i < objArr.length; i++){
    set.add(objArr[i]);
}
```

- set으로 로또 뽑기 같은 문제를 낼 수 있다.

```java
Set<Integer> set = new HashSet<>();

for(; set.size() <6;){
    int num = (int) (Math.random() * 45) + 1;
    set.add(Integer.valueOf(num));
}

List<Integer> list = new LinkedList<>(set);
Collections.sort(list);
System.out.println(list); //[7, 11, 17, 18, 24, 28]
```

- 빙고판을 만들 수도 있다.

```java
Set<Integer> set = new HashSet();
Set>Integer> set = new LinkedHashSet();
int[][] board = new int[5][5];

for(int i = 0; set.size() < 25; i++){
    set.add((int)(Math.random() * 50 + 1 + ""));
}

Iterator it = set.iterator();

for(int i = 0; i< board.length; i++){
    for(int j = 0; j < board[i].length; j++){
        board[i][j] = Integer.parseInt((String) it.next());
        sysout((board[i][j] < 10 ? "  " : " " ) + board[i][j]) //10보다 작으면 1자리수라서 비는 한칸을 띄어쓰기로 채움.
    }
}
```

- Set의 경우에는 객체를 넣을 때, 주소값이 달라도 field가 같으면 같은 것으로 판단하게끔한다.
- 따라서 equals와 hashcode()를 override해야한다.
- override하지 않으면 add는 되어도, 같은 객체인지 판단할 수 있수가 없게 되어버린다.
- 해당 override는 자동으로 해주는 IDE의 것을 통해 만들자.


## <span style="color:#802548">_Map_</span>

- 많이 쓰이는 것은 HashMap이다.

```java
public class HashMap extends AbstractMap implements Map, Cloneable, Serializeable{
    transient Entry[] table;
    static class Entry implements Map.Entry{
        final Object key;
        Object value;
    }
}
```

- 아래와 같이도 쓸 수 있지만, 이는 객체지향이 아니다. 따라서 위와 같이 class를 따로 만들어서 사용하는 것이다.

```java
Object[] key;
Object[] value;
```

- HashMap으로 할 수 있는 예는 총점과 평균을 구하는 것이다.

```java
HashMap map = new HashMap<>();
map.put("김자바", Integer.valueOf(100));
map.put("이자바", Integer.valueOf(80));
map.put("강자바", Integer.valueOf(100));
map.put("안자바", Integer.valueOf(90));

Set set = map.entrySet();
Iterator it =set.iterator();

while(it.hasNext()){
    Map.entry e = (Map.Entry)it.next();
    sysout("이름: "+ e.getKey() +", 점수: " +e.getValue());
    //이름: 안자바, 점수: 90
    //이름: 김자바, 점수: 100
    //이름: 강자바, 점수: 100
    //이름: 이자바, 점수: 80
}

set = map.KeySet();
sysout("참가자명단: " + set); //참가자명단: [안자바, 김자바, 강자바, 이자바]

Collection values = map.values();
it = values.iterator();

int total = 0;
while(it.hasNext()){
    Integer i = (Integer)it.next();
    total += i.intValue();
}

sysout("총점: " + total); //총점: 370
sysout("평균: " + (float) total/set.size()); //평균: 92.5
sysout("최고점수: " + Collections.max(values)); //최고점수: 100
sysout("최저점수: " + Collections.min(values)); //최저점수: 80
```

- HashMap으로 할 수 있는 예는 전화번호부를 그룹별로 만드는 것이다.

```java
static HashMap phoneBook = new HashMap();

public static void main(String args[]){
    addPhoneNo("친구", "이지바", "010-111-1111");
    addPhoneNo("친구", "김지바", "010-111-1111");
    addPhoneNo("친구", "김지바", "010-111-1111");
    addPhoneNo("회사", "김대리", "010-111-1111");
    addPhoneNo("회사", "김대리", "010-111-1111");
    addPhoneNo("회사", "박대리", "010-111-1111");
    addPhoneNo("회사", "이과장", "010-111-1111");
    addPhoneNo("세탁", "010-888-8888");
}

static void addPhoneNo(String groupName, String name, String tel){
    addGroup(groupName);
    HashMap group = (HashMap)phoneBook.get(groupName);
    group.put(tel, name); //그룹명이 부여되어있다면 전화번호를 key, 이름을 value로 집어넣음(전화번호가 unique해야하기 때문)
}

static void addGroup(String groupName){
    if(!phoneBook.containsKey(groupName)){
        phoneBook.put(groupName, new HashMap()); //그룹명을 부여
    }
}

static void addPhoneNo(String name, String tel){
    addPhoneNo("기타", name, tel) //그룹명이 없다면 기타로 고정
}

static void printList(){
    Set set = phoneBook.entrySet();
    Iterator it = set.iterator();

    while(it.hasNext()){
        Map.Entry e = (Map.Entry)it.next();

        Set subSet - ((HashMap))e.getValue().entrySet();
        Iterator it = subSet.iterator();

        System.out.println(" * " + e.getKey() + "[" + subSet.size() + "]"); // * 기타[1]

        while(subIt.hasNext()){
            Map.Entry subE = (Map.Entry)subIt.next();
            String telNo = (String)subE.getKey();
            String name = (STring)subE.getValue();
            System.out.println(name + " " + telNo);//세탁 010-888-8888
        }
    }
}
```

- 배열에서 반복된 알파벳의 갯수를 셀 수도 있다.

```java
String[] data = {"A","L","A","K","D","K","A","K","K","K","Z","D"};

HashMap map = new HashMap();

for(int i =0; data.length; i++){
    if(map.containsKey(data[i])){
        Integer value = (Integer)map.get(data[i]);
        mpa.put(data[i], Integer.valueOf(value.intValue()+1));
    }else{
        map.put(data[i], Integer.valueOf(1));
    }
}

Iterator it = map.entrySet().iterator();

while(it.hasNext()){
    Map.Entry entry = (Map.Entry)it.next();
    int value = ((Integer)entry.getValue.intValue());
    System.out.println(entry.getKey() + " : " + printBar('#', value) + " " + value);
    //A: ### 3
    //D: ## 2
    //Z: # 1
    //K: ###### 6
}

public static String printBar(char ch, int value){
    char[] bar = new char[value];

    for(int i = 0; i < bar.length; i++){
        bar[i] = ch;
    }

    return new String(bar);
}
```

- HashMap은 앞에 hash가 붙어있듯 해싱을 이용한다.
  - 해싱은 해쉬함수를 이용하며, 원하는 데이터를 빠르게 찾을 수 있음을 의미한다.
  - 해싱에서 사용하는 자료구조는 배열과 링크드리스트의 조합으로 이뤄져있다.
- 00년대부터 90년대까지 있다면 00~09, 10~19 등 으로 나눠 배열에 넣는다.
  - 그 안에서 또 나눠서 linkedList에 담는다.
- 즉 key로 003231이 오면 해싱함수를 통해 0이란 값을 얻는다.
  - 그럼 배열의 0번째 index에 가서 해당 key의 value를 찾는 형식이다.
- 보통은 해싱 알고리즘은 Object class의 hashcode()를 이용한다.




## <span style="color:#802548">_Comparable과 comparator_</span>

- Comparable은 class의 기본 정렬 순서를 구현할 때 사용한다.
- TreeSet의 경우 정렬 기준을 generics의 parameterized type인 class 내부에 갖고 있어야 한다.
- 따라서 Comparable 구현이 필수적이다. 구현을 하지 않으면 아예 add가 불가능하다.
- 주의할 점은 그냥 name이 아니라 o.name으로 비교해줘야 한다는 사실이다.

```java
class Member implements Comparable<Member> {
	String name;
	int age;
	
	@Override
	public int compareTo(Member o) {
		// TODO Auto-generated method stub
		return this.name.compareTo(o.name /*name. 그냥 name이 아니라 field로 비교..*/); //역순은 * -1
	}
	
}
```

- this.name or name이 먼저 나오게 되면 오름차순이다.
- 내림차순은 거기서 -1을 곱하면 된다.
    - 하지만 가동성을 위해 -1을 한 식보다는 맨아래처럼 써주는 걸 추천한다.

```java
@Override
public int compareTo(Member o) {
    return this.name.compareTo(o.name);
} //asc

@Override
public int compareTo(Member o) {
    return o.name.compareTo(this.name); 
} //desc

@Override
public int compareTo(Member o) {
    return -(this.name.compareTo(o.name));
} //desc

@Override
public int compareTo(Member o) {
    return -1 * this.name.compareTo(o.name);
} //dsec
```

- 참고로 해당 구현에 따르기 떄문에, 해당 구현 상 정렬이 불가능 하면 add를 하지 않고 false를 return한다.
- 예를 들어, 이름이 같은 경우에 이름만으로 정렬을 하려고 하면 처음 들어간 사람만 들어간다.
- 두번쨰 들어갈 사람은 들어가지 못한다.
- 그런 현상을 방지하기 위해서는 두 번째 정렬 기준은 unique 값으로 잡아주는 것이 좋다.
    - 두번쨰 값을 int 학번이라고 해보자. int는 method가 없어 compareTo()를 사용할 수 없다.
    - 그럴 때 아래처럼 구현하게 되는 경우, 문제가 일어날 수 있다. 아닌 경우 0이면 element가 정렬 불가능으로 판단된다.
   

```java
@Override
public int compareTo(Student o) {
    //오름차순이므로 갈수록 커짐. ex) 가 나 다 .....타 파 하
    //o.name(this.name.compareTo)로 구현하면 내림차순.
    int priority = this.name.compareTo(o.name);
    
    //오름차순이므로 갈수록 커짐
    if (priority == 0) {//int type이라 method가 없음.
        priority = ( this.stdNum > o.stdNum ) ? 1 : 0;
        //( this.stdNum < o.stdNum ) ? 1 : 0; 로 구현하면 이상함
        //두번째 element를 값을 넣었는데, 값이 씹혀버리고 들어가지 않게됨.
    }
    return priority;
}
```

- 따라서 이중 if문을 사용해서 구현한다.
- 이름이 같으면 학번에 따라 오름차순으로 정렬하는 로직이다.

```java
@Override
public int compareTo(Student o) {
    //오름차순이므로 갈수록 커짐. ex) 가 나 다 .....타 파 하
    //o.name(this.name.compareTo)로 구현하면 내림차순.
    int priority = this.name.compareTo(o.name);
    
    //오름차순이므로 갈수록 커짐
    if (priority == 0) {//int type이라 method가 없음.
       // 아래는 학번에 관한 오름차순
        if (priority == 0) {
            if ( this.stdNum > o.stdNum ) {
                priority = 1;
            } else if (this.stdNum < o.stdNum) {
                priority = -1;
            } else {
                priority = 0;
            }
        }

        // 아래는 학번에 관한 내림차순
        if (priority == 0) {
            if ( this.stdNum > o.stdNum ) {
                priority = -1;
            } else if (this.stdNum < o.stdNum) {
                priority = 1;
            } else {
                priority = 0;
            }
        }
    }
    return priority;
}
```

- 하지만 위와 같은 구현은 너무 복잡하다.
- 따라서 아래처럼 간단하게 써준다.

```java
@Override
public int compareTo(Student o) {
    //오름차순이므로 갈수록 커짐. ex) 가 나 다 .....타 파 하
    //o.name(this.name.compareTo)로 구현하면 내림차순.
    int priority = this.name.compareTo(o.name);
    if (priority == 0) {
        //-1을 곱했으니 이건 
        priority = -1 * (this.stdNum - o.stdNum); 
    }

    return priority;
}
```

- Comparator는 기본 정렬 순서 외에 정렬을 하고 싶을 때 구현한다.
- 아래는 generics가 없는 형태라서 전부 Object로 구현되어 있다.

```java
class Descending implements Comparator {
    public int compare(Object o1, Objecet o2){
        if(o1 instanceof Comparable && o2 instanceof Comparable){
            Comparable c1 = (Comparable)o1;
            Comparable c2 = (Comparable)02;

            return c1.compareTo(c2) * -1 //c2.compareTo(c1); //내림차순
        }
    }
}
```

- 실제로 사용되는 geneircs가 붙은 Comparator를 살펴보자.
- 왼쪽이 앞에 오면 오름차순, 오른쪽이 앞에 오면 내림차순이다.

```java
class PersonNameComparator implements Comparator<Person> {
    
    @Override
	public int compare(Person o1, Person o2) {
        return o1.name.compareTo(o2.name); //오름차순
	}
}

class PersonNameComparator implements Comparator<Person> {
    
    @Override
	public int compare(Person o1, Person o2) {
        return o2.name.compareTo(o1.name); //내림차순
	}
}

class PersonNameComparator implements Comparator<Person> {
    
    @Override
	public int compare(Person o1, Person o2) {
        return -1 * o1.name.compareTo(o2.name); //내림차순
	}
}
```

- 보통 아래와 같이 익명 객체를 사용하는 형태를 선호된다.
- Comparator는 목적상 1회용인 경우가 흔하기 때문이다.
- Person이라는 class에 Comparable이 구현되어 있지 않으면 아래처럼 Comparator로 정렬 기준을 주면 된다.

```java
Set<Person> set = new TreeSet<Person>(new Comparator<Person>() {

    @Override
    public int compare(Person o1, Person o2) {
        return o1.name.compareTo(o2.name);
    }
});
```

- 보통은 아래처럼 List의 정렬기준으로써 많이 활용된다.

```java
Collections.sort(students, new Comparator<Student>() {

    @Override
    public int compare(Student o1, Student o2) {
        //o1이 먼저나오면 오름차순
        //o2가 먼저나오면 내림차순
        //아래처럼 o1이 먼저 나오고 -1 곱하기 하면 내림차순
        return -1 * (o2.getScore().getSum() - o1.getScore().getSum()) ;
    }
});
```

- 미리 만들어져 있는 Comparator static field들은 아래와 같다.
- 참고로 generics를 만들면 안된다. Comparator interface의 static, default method만 사용하기 때문이다.
- static method는 특정 type에 의존하지 않기 때문에 generics가 붙으면 Comparator interface의 어떤 method에도 접근 불가능하다.

```java
Comparator<Student>. //generics가 생기면 그 순간 모든 method들을 가져올 수 없게됨. 
Comparator.naturalOrder();
Comparator.reverseOrder();
Comparator.comparing(Person::getAge); //natrual order 기준 age field를 기준값으로 잡는다.
Comparator.comparingInt(person -> person.getAge()); //int 값인 경우, Int를 명시해서 natural order기준으로 정렬한다.
Comparator.comparingDouble(Person::getSalary);      //int가 아닌 doubleversion이다.
Comparator.comparingInt(Person::getAge).thenComparing(Person::getName); //두 개이상의 Comparator를 이어줄 때 사용한다.
String.CASE_INSENSITIVE_ORDER;                      //String인데 case-insensitive하게 natural order로 진행한다.
```

- 역순으로 만들어야 한다면, int, double의 경우는 reversed()를 chaining 하여 붙여준다.
- type을 명시하지 않고 그냥 field로 가져오는 경우, reverseOrder()를 안에 넣어줘야 한다.

```java
Comparator<Person> byAgeThenNameDesc = Comparator.comparingInt(Person::getAge)
                                                  .reversed()  // Reverse age order
                                                  .thenComparing(Person::getName, Comparator.reverseOrder()); // Reverse name order
```

- 위의 sort method도 아래와 같이 다시 쓸 수 있다.
- score 안의 sum을 가져오는 것은 method referecne(Student::getScore)로는 가능하지 않다.
- reference는 오직 하나의 method에 대해서만 가능하다. 따라서 (Student::getScore::getSum)은 불가능하다.
- 물론 두번째 방법도 있지만, 이건 가독성이 정말 구려서 비추천이다.

```java
Collections.sort(students, new Comparator<Student>() {

    @Override
    public int compare(Student o1, Student o2) {
        return -1 * (o2.getScore().getSum() - o1.getScore().getSum()) ;
    }
});

Collections.sort(students,  Comparator.comparing(e -> e.getScore().getSum(), Comparator.reverseOrder()) );
Collections.sort(students, Comparator.comparing(Student::getScore, Comparator.comparing(Score::getSum, Comparator.reverseOrder()))); //가독성 ㅈ구림
```

- Comparator를 활용하면 익명 객체를 구현하지 않아도 된다.

```java
Collections.sort(students, new Comparator<Student>() {

    @Override
    public int compare(Student o1, Student o2) {

        return o2.getScore().getSum() - o1.getScore().getSum();
    }
});

Collections.sort(students,  Comparator.comparing(e -> e.getScore().getSum(), Comparator.reverseOrder()) ); //동일
```


## <span style="color:#802548">_method화_</span>
- Student class는 Comparable interface를 구현했다.
- 그 이유는 TreeSet을 구현하기 위해서는 Comparable interface가 필요하기 때문이다.
	- TreeSet은 compareTo로 구현된 기준으로 정렬을 실시해 자동 저장한다.
	- 그 정렬기준은 아래와 같다.

```java
this.name.compareTo(o.name) 		//this가 앞에 오면 오름차순
o.name.compareTo(this.name) 		//parameter 변수가 앞에 오면 내림차순
-1 * (this.name.compareTo(o.name)) 	//2번째의 좀 더 알아보기 쉬운 버전. 내림차순
```

- 그게 아니라면 TreeSet을 생성할 때 new Comparator로 만들어줘야 한다.

```java
Set<Person> set = new TreeSet<Person>(new Comparator<Person>() {

    @Override
    public int compare(Person o1, Person o2) {
        return o1.name.compareTo(o2.name);
}
});
```

- 아래처럼 boolean 변수를 활용해서 확인하게 하였다.
- 그런데, 해당 확인 과정은 여러 번 중복해서 쓰인다.

```java
public void insertStudent() {
    System.out.println("\n[ 학생등록 ]");
    System.out.print("이름 입력 > ");
    String name = scan.next();
    int java = 0;
    int db   = 0;
    int web  = 0;
    boolean isValidJavaScore = false; 
    boolean isValidDbScore = false; 
    boolean isValidWebScore = false; 
    do {
        System.out.print("*Java : ");
        java = scan.nextInt();
        isValidJavaScore = (0 <= java && java <= 100);
        System.out.print("*DB : ");
        db = scan.nextInt();
        isValidDbScore = (0 <= db && db <= 100);
        System.out.print("*WEB : ");
        web = scan.nextInt();
        isValidWebScore = (0 <= web && web <= 100);
        
        if (!isValidJavaScore || !isValidDbScore || !isValidWebScore) {
            System.out.println("점수는 0~100점 사이로 입력하세요.");
        }
    } while (!isValidJavaScore || !isValidDbScore || !isValidWebScore );
    
    Score score = new Score( java, db, web);
    Student student = new Student(name, score);
    studentList.add(student);
    System.out.println("등록되었습니다.");
    
}
```

- 따라서 아래와 같이 method로 바꿔준다.
- Java 하나만 한정지을 게 아니라, 다양한 과목의 점수를 모두 받아 유효성을 check하게 한다.
- 따라서 isValidScore라는 method이름이 되는 것이다. insertScore()와 insertStudent() 모두에서 쓰므로 바꿔줄 수 있다.

```java
private boolean isValidScore(int score) {
    
    return (0 <= score && score <=100);
}

insertScore();
insertStudent();
```

- 아래 Student의 정보를 가져오는 방식도 바꿔줄 수 있다.
- 학생 정보를 가져오는 방식을 여러군데에서 쓰고 있으니 method로 빼주자.

```java
Student student = null;
boolean isExisted = false;
for (Student element: studentList) {
    if (element.getStdNum() == stdNum) {
        isExisted = true;
        student = element;
    }
}

if (!isExisted) {
    System.out.println("학생정보가 없습니다.");
    return;
}
```

- method로 빼는 것의 장점은 이름이 달린다는 것이다.
- return type은 Student지만 null도 return 가능하다.

```java
private Student findStudent(TreeSet<Student> studentList, int stdNum) {
    Student student = null;
    for (Student element: studentList) {
        if (element.getStdNum() == stdNum) {
            return student;
        }
    }
    return null;
}
```

- 더 간단해진다.
- method명을 보고 역할도 쉽게 유추가 가능하다.

```java
Student foundStudent = findStudent(studentList, stdNum);
			
if (foundStudent == null) {
    System.out.println("학생정보가 없습니다.");
    return;
}
```

- 반면에 student가 여러 명이 조회되어야 하는 경우도 있다.
- 기존에는 아래와 같은 코드를 사용했지만, 위의 method를 활용해보자.

```java
List<Student> students = new ArrayList<Student>();
boolean isExisted = false;
for (Student element: studentList) {
    if (element.getName().equals(name)) {
        isExisted = true;
        students.add(element);
    }
}

if (!isExisted) {
    System.out.println("학생정보가 없습니다.");
    return;
}
```

- 만약, paramter type이 바뀌지 않고 return type만 바뀐다면 overloading은 불가능했다.
- 하지만 이름은 String type이라 parameter가 바뀌므로 overloading 취급된다.
- 대신 name은 String이므로 ==이 아닌 equals로 비교한다.
- 초기값도 null이 아닌 new로 생성한 arraylist로 주면 된다.

```java
private List<Student> findStudent(TreeSet<Student> studentList, String name) {
    List<Student> students = new ArrayList<>();
    for (Student element: studentList) {
        if (element.getName().equals(name)) {
            students.add(element);
        }
    }
    return students;
}
```

- overloading을 활용했고, 이름을 알기 쉽다.
- == null이 아니라 size가 0인지 확인한다.

```java
List<Student> foundStudents = findStudent(studentList, name);
		
if (students.isEmpty()) {
    System.out.println("학생정보가 없습니다.");
    return;
}
```

- TreeSet은 index를 드러낼 수가 없다.
- TreeSet의 headSet()은 ArrayList 마냥 저장 순서를 드러내는 정확한 index 개념이 아니다.
- 아래같이 IntStream으로는 treeSet의 element에 접근할 수 있는 방법이 없다. get()이 없다.
    - 그렇다고 stream을 열면 의도하지 않게 매번 indeX마다 전체 조회를 하게 된다.


```java
int i = 1;
IntStream.range(0, studentList.size()).forEach(idx -> {
        System.out.println(studentList.headSet(idx).size() );
    });
```

- 그렇다고 stream을 TreeSet collection에서 열어도, 순위를 구해올 수가 없다.
- headSet은 index의 개념이 아니다.

```java
System.out.println(studentList.size());
studentList.stream().forEach(e -> {
    System.out.printf("[ %d위], %s_%d : 합계(%3d), 평균(%3.2f)", 
            studentList.headSet(e).size() + 1, e.getName(), e.getStdNum(), 
                        e.getScore().getSum(), e.getScore().getAvg());
    System.out.println();
    
    } 
);
```

- for문을 열어도 element에 접근할 수 가 없다.
- get() 방식이 없기 때문이다.

```java
//treeMap은 get으로 index 불러오는 것처럼 못함.
// get() 자체가 없음.
for(int i = 0; i <studentList.size(); i++) {
}
```

- 따라서 index를 가지는 ArrayList로의 변환이 필요하다.
- 다만 unmodifiableList로 변환하면 sort가 불가능하다.
- unmodifiableList는 요소의 변경이 허락되지 않은 collection이다.
- 따라서 아래와 같은 방식으론 sort가 불가능하다. unsupportedOperationException이 뜬다.

```java
List<Student> students = studentList.stream().toList();
List<Student> students = List.copyOf(studentList);
```

- 수정이 가능한 List로 만들려면 아래와 같이 변환해야 한다.

```java
List<Student> students = new ArrayList<>(studentList);
List<Student> students = studentList.stream().collect(Collectors.toList());
```