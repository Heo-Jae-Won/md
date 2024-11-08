## <span style="color:#802548">_Collection_</span>

- 데이터군을 저장하는 클래스들을 표준화한 설계라는 뜻이다.

- 우리가 실제로 쓰는 map, list, set은 많은 경우 collection의 것을 그대로 가져다 쓴다.
- list, set, map이 collection framework를 실제로 구현하는 interface다.
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

- Collections class는 collection에 관련된 util method를 제공한다.
- multi threading에서 동기화가 핋요할 땐 아래와 같이 synchronized method를 사용한다.

```java
static Collection synchronizedCollection(Collection c);
static List synchronizedCollection(List c);
static Set synchronizedCollection(Set c);
static Map synchronizedCollection(Map c);
static SortedSet synchronizedCollection(SortedSet c);
static SortedMap synchronizedCollection(SortedMap c);
List syncList = Collections.synchronizedList(new ArrayList());
```

- 변경불가 컬렉션을 만드는 방법은 아래와 같다.

```java
static Collection unmodifiableCollection(Collection c);
static List unmodifiableList(List list);
static Set unmodifiableCollection(Set c);
static Map unmodifiableCollection(Map c);
static NavigableSet unmodifiableCollection(NavigableSet c);
static SortedSet unmodifiableCollection(SortedSet c);
static NavigableMap unmodifiableCollection(NavigableMap m);
static SortedMap unmodifiableCollection(SortedMap m);
```

- 단 하나의 객체만 저장하는 컬렉션은 아래와 같이 singleton으로 만들 수 있다.

```java
static List singletonList(Object o);
static Set singleton(Object o);
static Map singletonMap(Object key, Object value);
```

```java
import static java.util.Collections.*;
List list = new ArrayList();
addAll(list, ,1,2,3,4,5);
int idx = binarySearch(list, 3); //요소를 얻어오는 게 매우 빠르지만, 정렬되어있을 떄만 사용 가능. value 3의 index는 2.
```


## <span style="color:#802548">_List_</span>

- list는 ArrayList와 LinkedList, Vector가 있다.
  - Vector는 멀티스레딩에서 쓴다.
- list interface의 method는 list interface가 자체로 갖고 있는 것과 collection interface에서 물려받은 것이 섞여있다.
- list interface가 갖고 있는 것에는 get, subList가 대표적이다.

```java
E get(int index); //E는 <>에 들어오는 원소의 type
List subList(int fromIndex, int toIndex) //from부터 to 이전까지 index에 든 값을 반환
```

- ArrayList는 내부에서 Object[]을 사용하기 때문에 어떤 type도 모두 담을 수 있다.

```java
ArrayList<Integer> list1 = new ArrayList(10);
list1.add(5);
list1.add(4);
list1.add(2);
list1.add(0);
list1.add(1);
list1.add(3);

ArrayList<Integer> list2 = new ArrayList(list1.subList(1,4)); //1st index에서 4th index전까지 즉 3번쨰 index까지 반환. 4,2,0
Collections.sort(list1); //0,1,2,3,4,5
Collections.sort(list2); //0,2,4
list1.retainAll(list2); //0,2,4 list1의 요소 중 list2의 요소에 없는 것은 모두 제거한다.

list2.add("B");
list2.add("C");
list2.add(3, "A");
list2.set(3, "AA"); // 0,2,4,AA,B,C
```

- list를 삭제할 때는 유의해야 한다.
- index를 0부터 시작시키는 게 아니라 거꾸로 맨끝에서부터 0으로 이동해야 한다.
- 삭제를 하는 순간 빈 공간을 채우기 위해 나머지 요소들이 index를 이동하기 때문이다.

```java
for(int i = list2.size()-1; i>=0;i--){
    if(list1.contains(list2.get(i)))
        list2.remove(i); // AA,B,C
}

for(int i = 0; i<list2.size()-1;i++){
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

- 아래는 Vector interface다.
- 실제 Vector의 구현로직 핵심은 아래와 같다.
- 내부에서는 배열을 이용한다. 삭제는 해당 index 값이후의 배열값을 그 index로 옮기고 마지막 index 값을 null로 만드는 작업이다.
- 따라서 삽입, 삭제 시에는 새로운 배열을 복사 생성하기 때문에 오래걸린다.

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
- 하지만 실제로는 객체들이 메모리에서 연속되어 있지 않은 점으로 인해 예상보다 성능이 떨어진다.
    - 캐시나 버퍼를 통해 이득을 보기 어렵다는 점, 
    - 메모리를 많이 잡아먹는다는 점 등이 작용해 잘 쓰이지 않는다.

## <span style="color:#802548">_Stack/Queue_</span>

- list collection framework로 만들 수 있는 것이 바로 stack과 queue다.
- stack은 LIFO고, queue는 FIFO다.
  - stack은 순차적으로 데이터를 추가하고 삭제하기 때문에 ArrayList를 사용한다
- queue는 데이터를 꺼낼때마다 처음 들어온 것을 꺼내기 때문에 배열을 새로 만들어 복사해야 한다.
  - 따라서 ArrayList는 비효율적이고, LinkedList를 쓰는 게 효율적이라 LinkedList로 구현한다.
- stack을 실제로 구현해보자.
  - Stack에 5개를 넣었다고 해보자. 그럼 크기가 5인 배열이 형성된다.

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

- queue는 interface로 구현하는 class는 LinkedList다.
- 따라서 LinkedList를 공부하면 queue를 알 수 있다.
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
            for(int i = 0; i <expression.length(); i++){
                char ch = expression.charAt(i);

                if(ch=='('){
                    st.push(ch+"");
                }else if(ch==')'){
                    st.pop();
                }
            }
            if(st.isEmpty()){
                sysout("괄호가 일치합니다.");
            }else{
                sysout("괄호가 일치하지 않습니다.");
            }
        }catch(EmptyStackException e){
            sysout("괄호가 일치하지 않습니다.");
        }
    }
}
```

- queue로 구현한 최근사용문서 버퍼 예시다.

```java
class QueueEx1{
    static Queue q = new LinkedList();
    static final int MAX_SIZE = 5;

    public static void main(String[] args){
        sysout("help를 입력하면 도움말을 볼 수 있습니다.");

        while(true){
            sysout(">>");

            try{
                Scanner s = new Scanner(System.in);
                String input = s.nextline().trim();

                if("".equals(input)){
                    continue;
                }

                if(input.equalsIgnoreCase("q")){
                    System.quit(0);
                }else if(input.equalsIgnoreCase("help")){
                    sysout(" help - 도움말을 보여줍니다.");
                    sysout(" q 또는 Q - 프로그램을 종료합니다.");
                    sysout(" history - 최근에 입력한 명령어를 "
                                                            + MAX_SIZE _"개 보여줍니다.");
                }else if(input.equalsIgnoreCase("history")){
                    int i = 0;
                    save(input);

                    LinkedList tmp = (LinkedList)q;
                    ListIterator it = tmp.listIterator();

                    while(it.hasNext()){
                        sysout(++i + "." + it.next());
                    }
                }else{
                    save(input);
                    sysout(input);
                }
            }catch(Exception e){
                sysout("입력오류입니다.");
            }
        }
    }

    public static void save(String input){
        if(!"".equals(input)){
            q.offer(input);
        }

        if(q.size() > MAX_SIZE){
            q.remove();
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
Iterator it = c.iterator();

while(it.hasNext()){
    sysout(it.next()); //[1,2,3]
}
```

- 하지만 Collection interface가 아닌 List interface를 사용하면 역방향으로 읽는 것까지 가능하다.

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
public class MyVector2 extends MyVector implements Iterator{
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

## <span style="color:#802548">_comparator란?_</span>

```java
class Descending implements Comparator{
    public int compare(Object o1, Objecet o2){
        if(o1 instanceof Comparable && o2 instanceof Comparable){
            Comparable c1 = (Comparable)o1;
            Comparable c2 = (Comparable)02;

            return c1.compareTo(c2) * -1 //c2.compareTo(c1); //내림차순
        }
    }
}
```

- 문자열의 오름차순은 공백 -> 숫자 ->대문자 ->소문자 순으로 정렬됨을 의미.

## <span style="color:#802548">_Set_</span>

- set은 HashSet과 TreeSet이 있다.
- HashSet은 순서를 유지하지 않는다. 만약 순서를 유지하고 싶다면 LinkedHashSet을 사용해야 한다.
- set은 값이 곧 key다. 따라서 중복된 값은 저장되지 않는다. 똑같아 보인다면 type이 다른 것이다.

```java
Object[] objArr = {"1", Integer.valueOf(1), "2", "2", "3", "3", "4", "4", "4"};
Set set = new HashSet();

for(int i = 0; i < objArr.length; i++){
    set.add(objArr[i]);
}

sysout(set); //[1,1,2,3,4]
```

- 위와 같은 불상사가 일어나지 않게끔 type을 generics로 강제해주자.
- 그럼 IDE에 오류가 드러난다.
  - The method add(Integer) in the type Set(Integer) is not applicable for the arguments (Object)

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
    int num =(int) ( Math.random() * 45)+1;
    set.add(Integer.valueOf(num));
}

List<Integer> list = new LinkedList<>(set);
Collections.sort(list);
System.out.println(list); //[7, 11, 17, 18, 24, 28]
```

- 빙고판을 만들 수도 있다.

```java
Set set = new HashSet();
Set set = new LinkedHashSet();
int[][] board = new int[5][5];

for(int i = 0; set.size() < 25; i++){
    set.add((int) (Math.random()*50 +1 + ""));
}

Iterator it = set.iterator();

for(int i =0; i< board.length; i++){
    for(int j =0; j <board[i].length; j++){
        board[i][j] = Integer.parseInt((String) it.next());
        sysout((board[i][j] < 10 ? "  " : " " ) + board[i][j]) //10보다 작으면 1자리수라서 비는 한칸을 띄어쓰기로 채움.
    }
}
```

- Set의 경우에는 객체를 넣을 때, 주소값이 달라도 field가 같으면 같은 것으로 판단하게끔한다.
- 따라서 equals와 hashcode()를 override해야한다.


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



