## <span style="color:#802548">_function declaration- generics, extends_</span>
- ts에서는 함수를 2개 이상 overloading하여 구현할 수 없다.

```js
function add(a: number, b: number) {
    return a + b; //compile error. Duplicate function implementation
}

function add(a: string, b: string) {
    return a + b; //compile error. Duplicate function implementation
}
```

- 2개의 overloading된 type을 갖는 fn을 선언은 가능하다.
  - 구현이 아닌 선언만 한다.
  - 구현부는 위에서 말한 것처럼 선택해서 하나만 구현해야 한다.

```js
function add(a: nubmer, b: number) : number;
function add(a: string, b: string) : string;
function add(a,b) {
    return a + b
}
```

- 하나의 parameter가 아니라 다양한 type을 받는 경우에는 union type의 문제으로 선언한다.
- generics와 조건부 타입을 써서 유연하게 받을 수 있게 구성한다.

```js
function add<T extends number | string>(x: T): T extends string ? string : number;
function add(x: any) {
    return x + x
}

function double(x: number) {
    return  add(x); //number로 해석
}
function double(x: number) {
    return  add(x); //string으로 해석
}
function double(x: number | string) {
    return  add(x);// number | string로 해석
}
```

- string을 extends한 것은 string literal 간의 union type도 포함이다.

```js
function getKey<K extends string>(val: any, key: K) {
    ...
}

getKey({}, 'x'); //string
getKey({}, document.title); //string
getKey({}, Math.random < 0.5 ? 'a' : 'b'); //<"a" | "b">. string literal union 
getKey({}, Math.random() < 0.5 ? 'a' : Math.random() < 0.7 ? 'b' : 'c'); //<"a" | "b" | "c">. string literal union 
getKey({}, Math.random() < 0.5 ? 'a' : Math.random() < 0.7 ? 1 : 'c');  //string | number. error
getKey({}, 12); //error. 
```

- number를 extends한 것도 number literal 간의 union type을 포함한다.

```js
function getKey<K extends number>(val: any, key: K) {
   console.log(key);
}

getKey({}, 1);  //number
getKey({}, 2);  //number
getKey({}, Math.random() < 0.5 ? 1 : Math.random() < 0.7 ? 1 : 3);//<1 | 3>
getKey({}, Math.random() < 0.5 ? 1 : Math.random() < 0.7 ? '1' : 3); //error
```

- 위의 extends의 내용을 발전시키면 아래와 같다.
- keyof로 K를 한정하여 가져오면 어떤 interface든 그 조합의 union을 커버할 수 있다.

```js
function sortBy<K extends keyof T, T>(vals: T[], key: K): T[] {
    return vals;
}

interface Point {
    x: number;
    y: number;
    z: number;
}
const pts : Point[] = [{x:1, y:1}, {x:2, y:0}];
sortBy(pts,'x');
sortBy(pts,'y');
sortBy(pts,'z');
sortBy(pts,Math.random() < 0.5 ? 'x' : 'y');
sortBy(pts,Math.random() < 0.5 ? 'x' : 'z');
sortBy(pts,Math.random() < 0.5 ? 'y' : 'z');
sortBy(pts,Math.random() < 0.5 ? 'y' : 't'); //error.Argument of type '"y" | "r"' is not assignable to parameter of type 'keyof Point'. Type '"r"' is not assignable to type 'keyof Point'.
sortBy(pts,Math.random() < 0.5 ? 'x' : Math.random() < 0.7 ? 'y' : 'z');
```

- 만약 fn의 parameter를 object로 받는다면, type을 선언하는 방식은 아래와 같다.

```js
interface emailFunction {
    email(options: {person: Person, subject: string, body: string});
}
```

- email 함수를 위와 같이 선언했다면 비구조화 할당해서 사용한다면 아래와 같다.

```js
function email({person, subject, body} : {person: Person, subject: string, body: string})
```


## <span style="color:#802548">_key순회를 하면서 더하면 안되고, 직접 필요한 property를 더해줘야한다_</span>
- Object.keys()로 객체를 순회하는 경우, key가 문자열로 return된다.
- 따라서 아래 axis는 type이 늘 string이다. 그런데 Vector3D는 string type의 key를 가지지 않는다.

```js

interface Vector3D {
    x: number,
    y: number,
    z: number
}

function calculateLengthL1(v: Vector3D) {
    let length = 0;
    for (const axis of Object.keys(v)) {
        const coord = v[axis]; //Element implicitly has an 'any' type because expression of type 'string' can't be used to index type 'Vector3D'.
                                //No index signature with a parameter of type 'string' was found on type 'Vector3D'.
        length +=Math.abs(coord);
    }
    return length;
}

```

- 해당 상황을 회피하기 위해서 index signiture를 집어넣을 수 있다.
- 그러나 그렇게 되면 아무 key나 들어가도 상관없게 된다.

```js
interface Vector3D {
    x: number,
    y: number,
    z: number
    [key: string]: number,
}

function calculateLengthL1(v: Vector3D) {
    let length = 0;
    for (const axis of Object.keys(v)) {
        const coord = v[axis];
        length +=Math.abs(coord);
    }
    return length;
}

calculateLengthL1({x:123,y:11,z:14,abc:123}) //abc는 유효한 key가 아니지만.. type check 통과
```

- 이 경우에는 element 요소를 순회할 게 아니라 직접 필요한 것만 더해줘야 한다.

```js
interface Vector3D {
    x: number,
    y: number,
    z: number
}

function calculateLengthL1(v: Vector3D) {
    return Math.abs(v.x) + Math.abs(v.y) + Math.abs(v.z);
}

//destructure
function calculateLengthL1({x, y, z}: Vector3D) {
    return Math.abs(x) + Math.abs(y) + Math.abs(z);
}

// 이건 안 됨. 타입을 줘버렸기에 Vector3D가 아니게 됨. 따라서 Vector3D type이 아니므로 compile check 시 오류가 남.
function calculateLengthL1({x: number, y: number, z: number}: Vector3D) {
    return Math.abs(x) + Math.abs(y) + Math.abs(z);
}
```

- 말했듯 Object.keys()는 늘 문자열을 return하기에 문자열 key만 나온다.
- 실제 js runtime에서 key는 늘 문자열 취급이다.
  - a[2]과 a['2']이 똑같다.
- 그러나 ts에서는 다르다. a[2]와 a['2']는 다르다.
  - Object.keys()로 객체의 속성을 살펴보고 이용하려고 하지말자.   
  - 그럴 때는 차라리 for in 문을 쓰자.
  - 만약 배열 객체라면 for of 문을 쓰자.


## <span style="color:#802548">_index signature_</span>

- index signature는 특별한 경우가 아니면 쓰지 않는 것이 좋다.
- 아래와 같은 interface가 있다고 해보자.

```js
interface Schedule {
    date: string,
    headCount: number
}
```

- 만약 해당 속성에 대괄호로 접근하려 한다면 오류가 난다.

```js
const a: Schedule = {
    date:'2023-01-21',
    heaCount:5
}

a['date']; // Element implicitly has an 'any' type because expression of type 'string' can't be used to index type
```

- 그 때 index signature를 도입하면 문제가 해결된다. 하지만 임시방편이다.
- 아래와 같은 index signature는 string 형식의 key면 무엇이든 받는다.
  - 그것이 실제 Schedule interface에 존재하지 않아도 말이다.

```js
interface Schedule {
    date: string,
    headCount: number
    [key: string]: string | number
}
```

- key에 유효하지 않은 값을 넣었지만 type check가 성공한다. 이건 명백한 오류다.
```js
const a: Schedule = {
    date:'2023-01-21',
    heaCount:5
}
a['ddd']; //error 안 남.
```

- 해당 현상을 해결하기 위해서는 아래와 같은 방법을 모색할 수 있다.
- as keyof를 사용하는 것이다.
- 그럼 모든 변수에서가 아니라, 딱 저기에서만 as를 사용하기에 더 나은 선택이다.

```js
interface Schedule {
    date: string,
    headCount: number
}

 <v-select v-for="element in stationFilteringItems" :label="element.label" :key="element.label" :items="stationItems" v-model="scheduleRequest[element.value]"></v-select>
```

- 하지만 이 역시 element.value가 headcountInfo의 key가 아닌 게 들어와도 type check를 해주지 않는다.
- scheduleArriveStation인데 scheduleArriveStatino으로 오타를 내도 as를 썼기에 오류가 없다.
- as를 쓰면 type check가 작동하지 않기 때문이다. 오타에 취약하다.

```js
const stationFilteringItems = [
  { label: '출발역', value: 'scheduleDepartStation' },
  { label: '도착역', value: 'scheduleArriveStatino' },
];
```


- 따라서 value를 대괄호에 들어갈 수 있는 key로 규정해줘야 한다.
- 그것은 즉 keyof로 규정해줘야 문제를 해결할 수 있다는 것이다.
- 바로 type check를 통해 오타를 인지하는 모습을 볼 수 있다.
- 또한 자동완성 서비스도 받을 수 있다. value라고 치고 컨트롤 엔터를 누르면 ScheduleRequest의 key가 자동완성 추천에 뜬다.

```js
interface StationFilteringItem {
  label: string;
  value: keyof ScheduleRequest; // Assuming ScheduleRequest is the type for scheduleRequest
}
const stationFilteringItems: StationFilteringItem[] = [
  { label: '출발역', value: 'scheduleDepartStation' },
  { label: '도착역', value: 'scheduleArriveStatino' }, //type '"scheduleArriveStatino"' is not assignable to type 'keyof ScheduleRequest'. Did you mean '"scheduleArriveStation"'?
];
```

- 원래 있던 key를 가져오는 경우는 as keyof로 바로 해결가능하다.
- 아래와 같은 경우다. axis는 무조건 해당 객체의 key이기 때문에 as를 써도 된다.
- 내가 직접 객체로 넣는 것도 아니기에 오타를 걱정할 필요도 없다.

```js
for (const axis of Object.keys(v)) {
        const coord = v[axis as keyof Vector3D];
        length += Math.abs(coord);
}
```


- index signature를 써야만 할 때도 있다.
- 데이터가 어떻게 들어오는 지 모르는 것들일 때다.
- 예를 들자면 excel, csv 등이다. 어떤 column 이름이 들어올 지 모르기 때문이다.

```js
function parseCSV(input: string): {[columnName: string]: string}[] {
    const lines = input.split('\n');
    const [header, ...rows] = lines;
    const headerColumns = header.split(',');
    return rows.map(rowStr => {
        const row: {[columnName:string]: string} = {};
        rowStr.split(',').forEach((cell,i) => {
            row[headerColumns[i]] = cell;
        })
        return row;
    })
}
```

- 만약에 열 이름을 안다면 interface를 만들면 된다.

```js
interface ProductRow {
    productId: string;
    name: string;
    price: string;
}

let csvData = '';
const products = parseCSV(csvData) as unknown as ProductRow[];
```

- 더 안전하게 할 수도 있다. 
- 아래와 같이 undeinfed type을 return type으로 추가해준다.

```js
function safeParseCSV(input: string): {[columnName:string]: string | undefined}[] {
    return parseCSV(input);
}

const prices:{[proodut: string]: number} = {};
let csvData = '';
const safeRows = safeParseCSV(csvData);
for(const row of safeRows) {
    if(row.productId) {//row.productId가 undefined일 수 있어 type check 필요
        prices[row.productId] = Number(row.price);
    }
}
```

- index signature가 가진 또 하나의 단점은 예약어에 취약하다는 점이다.

```js
function countWords(text: string) {
    const counts: {[word: string] : number} = {};
    for ( const word of text.split(/[\s,.]+/)) {
        counts[word] = 1 + (counts[word] || 0);
    }

    return counts;
}

console.log(countWords('Objects have a constructor'));
```

- constructor와 같은 예약어를 쓰면 예기치 않은 결과를 초래한다.

```
{
    Objects: 1,
    have: 1,
    a: 1,
    constructor: "1function Object() { [native code]}";
}
```

- map과 set을 이용하면 예기치 않은 결과를 피할 수 있다.

```js
function countWords(text: string) {
    const counts = new Map<string, number>();
    for ( const word of text.split(/[\s,.]+/)) {
        counts.set(word, 1 + (counts.get(word) || 0));
    }

    return counts;
}

console.log(countWords('Objects have a constructor'));
[LOG]: Map (4) 
{
    "Objects" => 1, 
    "have" => 1, 
    "a" => 1, 
    "constructor" => 1
} 
```


## <span style="color:#802548">_typeof, type assertion_</span>
- ts에서 type은 값들의 집합이라고 생각하면 이해하기 쉽다.
  - 아래 오류는 해당 type은 다른 type의 부분집합이 될 수 없다는 의미라고 이해할 수 있다.
  - 즉 type check에 실패했기 때문에 고쳐야한다는 의미다.
```
'12' 형식의 인수는 'string' 형식의 매개변수에 할당될 수 없습니다.
'number[] 타입은' '[number, number]' 타입의 0, 1 속성에 없습니다.
```

- ts의 typeof를 쓰면 값과 타입에 대해 다른 연산이 일어난다.
- type의 typeof는 js코드로 사용이 불가능하다.
- const같은 값의 typeof는 js코드로 사용이 가능하다.
  - 참고로 class는 js에서 함수로 구현되므로 값의 관점에서 typeof를 하면 function
  - class로 만든 instance의 typeof는 object다.
```js
type t1 = typeof p; //js 코드로 변환 불가능. Person 같이 사용자 정의 type도 노출됨.
const t1 = typeof p; //js runtime의 type 정보. string, number, object, function 등 6개 type 중 1개. interface나 type은 아님

class ScienceFacility {
    constructor() {
        this.units = ['Science Vessl'];
    }

    getUnits() {
        return this.units;
    }
}

let scienceFacility = new ScienceFacility();
scienceFacility.getUnits();

const t1 = typeof ScienceFacility;  // class
const t2 = typeof scienceFacility ; // instance
console.log(t1); //fucntion
console.log(t2); //object
```

- 아래의 type은 모두 instance가 지닌 값이라는 관점이 아니라, ts에서 다루는 type과 관련된다.
- 타입 단언과 타입 선언은 아래와 같이 쓴다.
```js
const bob = {} as Person /* const bob = <Person>{} */   //타입 단언
/* const bob =ref<>();와는 다르다. 해당 문법은 generics를 사용한 것이다.*/
const bob: Person = {};                                 //타입 선언
```

- 참고로 객체형 type은 선언하면 안 된다. 
```
String X / string O
Number X / number O
Boolean X / boolean O
```

- 타입 선언이 좋은 이유는 함수에서 더 확실하게 드러난다.
- type선언을 해놓고 쓰면 함수 표현식에 해당 return type으로 표시해준다.
- 그럼 parameter에 type을 적지 않아도 context가 제공되니까 상관이 없다.

```js
function rollDice(sides: number): number {}

type DiceRollFn = (sides: number) => number;
const rollDice: DiceRollFn = sides => {};
```

- 그럼 의미에서는 함수 선언보다는 함수 표현식이 좋다.
- 함수 선언은 parameter에 type을 다 넣어줘야 한다.
- 함수 표현식으로 쓰고 함수 전체에 type을 쓰면 좋은 이유가 더 있다.
- 함수가 호출된 곳이 아니라, 함수가 정의한 곳에서 오류가 나서 오류 해결이 빨라진다.
```js
async function checkedFetch(input: RequestInfo, init?: RequestInit) {
    const response = await fetch(input, init);
}

const checkedFetch: typeof fetch = async(input, init) => {
    const response = await fetch(input, init);
}
```

- 이제 타입 단언을 알아보자.
  - 가장 큰 단점으로는 잉여속성 체크가 안 된다는 점이다.
  - type check가 부실하다는 의미다.
```js
interface Options {
    title: string;
    darkMode?: boolean;
}

const o = {
    darkmode: true,
    title: 'Ski Free'
} as Options;   //darkmode라고 했으면 원래 잉여 속성 체크가 돼서 darkmode라는 key가 없다고 떠야한다.
                //하지만 타입 선언을 했으므로 안 나온다.
```

- return variable에 타입 단언을 하는 것은 type check를 속이는 것일 뿐이다.
  - runtime에서는 무의미하다. 
  - 그럴 땐 ts가 아닌 js의 형변환을 사용하는 것이 좋다.
```js
//bad
function asNumber(val: number | string) {//val은 string도 될 수 있으니 as number로 썼다.
    return val as number;
}

//good
function asNUmber(val: number | string) : boolean {
    return typeof(val) === 'string' ? Number(val) : val;
}
```

- 다만 type 단언이 필요한 경우가 있다.
- 바로 DOM 객체를 다룰 때다.
  - element는 HTMLElement를 return한다.
  - 그러나 HTMLElement가 아니라 HTMLInputElment를 return해야 한다.
  - 그래야 value라는 property에 접근 가능하기 때문이다.
- 그 때 타입 단언을 사용한다.
```js
const element = document.querySelector('#input');

const element = document.querySelector('#input') as HTMLInputElement;
```

## <span style="color:#802548">_임시변수 쓰지 않기_</span>
- 변수 대입으로 해도 잉여속성 체크가 안된다.
- 임시 변수를 사용하는 것은 피해야 한다.
```js
interface Options {
    title: string;
    darkMode?: boolean;
}

const intermediate = {
    darkmode: true,
    title: 'Ski Free'
} 

const o : Options = intermediate; //darkmode라서 key가 없다는 오류가 떠야하지만 뜨지 않는다.
```

- 객체의 속성을 임시변수를 꺼내서 사용하는 것도 위험하다.
- polygon.bbox 값이 변경되면, bbox와 다른 값을 참조할 수 있다.
- 그게 type마저 변화시키는 것이라면 혼란을 초래할 수 있다.
```js
const {bbox} = polygon; //임시변수로 꺼냄.
if (!bbox) {
    calculatePlygonBbox(polygon);
}
```

- 계산 로직 등으로 값, type 변화가 일어난다면, 변수를 아예 만들지 않는게 낫다.
```js
if (!polygon.bbox) {
    calculatePlygonBbox(polygon);
}
```

## <span style="color:#802548">_interface_</span>
- interface는 중복을 제거하는 데 있어서도 꽤나 중요하다.
```js
function distance(a: {x: number, y: number} , b: {x: number, y: number}) {
    return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
}
```

- 그냥 x, y보다는 강타입으로 묶을 수 있고, 이름을 부여할 수 있게 interface를 만든다.
```js
interface Point2D {
    x: number,
    y: number
}
function distance(a:Point2D, b:Point2D) {}
```

- 만약 몇 개의 property는 optional 하게 받고 싶다면, 해당 property를 optional로 만들 수도 있지만, interface의 extends를 활용할 수도 있다.
- PersonWithBirthDate는 firstName, lastName, birth 3개를 갖게 된다.

```js
interface Person {
    firstName: string;
    lastName: string;
}

interface PersonWithBirthDate extends Person {
    birth: date
}
```

- 이를 type으로 보여주면 아래와 같다.
```js
type PersonWithBirthDate = Person & {birth: Date};
```

- interface를 사용하면 더 알아보기 쉽게 만들 수 있다.
- people의 return type은 {name:'string'}[]이다. 이게 뭔지 이해하기 어렵다.
```js
const people = ['alice','bob','jan'].map((name) => ({name}));
```

- interface를 사용해서 바꿔보자.
- name이란 element가 아니라 map()의 return type을 Person으로 해줘야 한다.
```js
interface Person {
    name: string    
}

const people = ['alice','bob','jan'].map((name) => {
    const person: Person = {name};
    return person;
});

const people = ['alice','bob','jan'].map((name: Person) => {name}); //error

const people = ['alice','bob','jan'].map((name) : Person => ({name})); //ok
```

- interface에 필수 속성이 추가됐을 때, 자동으로 오류가 나게 하는 방법이 있다.
- 아래와 같이 규정된 interface와 변수가 있다.

```js
interface ScatterProps {
    x: number[];
    y: number[];
    onClick (x: number, y: number, index:number) => void;
}

const REQUIRES_UPDATES = {
    x: true,
    y: true,
    onClick: false
}
```

- 여기서 interface에 doubleClick 속성을 추가했다.
- 만약 REQUIRES_UPDATES 변수가 ScatterProps의 interface key를 가져오려했다면, 의도한 대로 동작하지 않는다.
- 현재 onDoubleClick이 추가됐지만, 변수에는 추가되지 않았다. 그래도 오류가 없다. 변수는 ScatterProps type이 아니기 때문이다. 
- 주석으로 어디어디에 수정해야한다고 알려줄 수도 있지만, 매우 나쁜 방법이다.
```js
interface ScatterProps {
    x: number[];
    y: number[];
    onClick (x: number, y: number, index:number) => void;
    // 더 추가되면 REQUIRES_UPDATES 변수 수정 요구됨.
    onDoubleClick: ~~
}

const REQUIRES_UPDATES= {
     x: true,
    y: true,
    onClick: false
    //error 나지 않음.
}
```

- 그렇다고 ScatterProps type으로 줄 수도 없다. 결과값들은 boolean으로 받고 싶기 때문이다. 
- 그럴 때 아래와 같이 keyof를 사용할 수 있다.

```js
const REQUIRES_UPDATES:{[k in keyof ScatterPropㄵ3s]: boolean} = {
    x: true,
    y: true,
    onClick: false,
    //Property 'onDoubleClick' is missing in type 
}

```


## <span style="color:#802548">_특정 interface pattern_</span>
- 기존의 interface에서 일부 property만 가져오고 싶을 떄 사용하는 pattern이다.
- 보통의 interface를 아래와 같이 쓴다.
```js
interface State {
    userId: string;
    pageTitle: string;
    recentFiles: string[];
    pageContents: string;
}

interface TopNavState {
    userId: string;
    pageTitle: string;
    recentFiles: string;
}
```


- 저 상태에선 부모 interface property의 type이 바뀌면 자식도 일일이 찾아서 바꿔줘야 한다.
- 그걸 하지 않게 매핑시켜주자.

```js
interface TopNavState {
    userId: State['userId'];
    pageTitle: State['pageTitle'];
    recentFiles: State['recentFiles'];
}
```

- 이걸 추상화 시켜주자.
```js
type TopNavState = {[k in 'userId' | 'pageTilte' | recentFiles]: State[k]}
```


- 이걸 다시한번 구체적 상황이 아닌 일반 상황으로 추상화 시켜주자.
```js
type Pick<T,K> = {[k in K]: T[k]}
```

- K가 아무거나 들어가면 안되니까 T의 key여야 한다.
```js
type Pick<T,K extends keyof T> = {[P in K]: T[P]}
```

- 따라서 아래와 같이 쓸 수있다.
```js
type TopNavState = Pick<State,'userId' | 'pageTitle' | 'recentFiles'>;
```



- 특정 interface의 property를 모두 optional property가 되게 하는 방법도 있다.
```js
interface Options {
    width: number;
    height: number;
    color: string;
    label: string;
}

interface OptionsUpdate {
    width?: number;
    height?: number;
    color?: string;
    label?: string;
}
```

- 하지만 이건 역시 type mapping이 안 된다.
- 자동으로 받아오게 해보자.
```js
type OptionsUpdate = {[k in keyof Options]?: Options[k]};
```

- 이걸 아래와 같이 추상화할 수 있다.
```js
type Partial<T> = {[P in keyof T]?: T[P]};
```

- 이를 이용해서 아래와 같이 쓸 수 있다.
```js
type OptinosUpdate = Partial<Options>
```


- 특정한 값으로만 이뤄지는 type을 손쉽게 만드는 방법도 있다.
- 바로 Record다.
- 아래와 같이 interface를 만들었는데, 이게 일회용이다.
```js
interface point3D {
    x: number;
    y: number;
    z: number;
}
```

- 그럼 더 간단하게 Record를 쓰면 된다.
```js
type Point3D = Record<'x' |'y'|'z',number>;
```

- Record 패턴은 아래와 같다.
- 모든 key가 들어갈 수 있지만, 각 key의 type은 하나로 고정이다.
```js
type Record<K extends keyof any, T> = {
    [P in K]: T;
};
```

- 만약 함수의 returnType을 명명된 type으로 하고 싶다면?
- 직접 쓸 수도 있다. 아래와 같이 api 정보를 모두 알아와야 한다.
```js
function getUserInfo(userId: string) {

    return {
        userId,
        name,
        age,
        height,
    }
}

type getUserInfo = {
    userId: string;
    name: string;
    age: number;
    height: number
}
```

- 하지만 이건 너무 노가다다. 
- 노가다 대신 ReturnType을 쓸 수 있다.
```js
type getUserInfo = ReturnType<typeof getUserInfo>;
```

- ReturnType은 아래와 같다. 
- parameter는 아무거나 받고, return type도 아무거나 받을 수 있다.
```js
type ReturnType<T extends (...args: any) => any> = T extends (...args: any) => infer R ? R : any;
```

## <span style="color:#802548">_타입 추론 차이- let과 const_</span>
- const의 경우, let보다 더 정밀하게 type을 추론한다.
- 값이 바뀔 수 없기 때문이다.
```js
let x ='x'; //string type
const x = 'x'; // 'x' type
```

- 이는 강점이 될 수도 있지만, 너무 엄격한 type checking은 방해가 될 때도 있다.
- 배열은 (string | number)[]로 타입이 check되는데, tuple을 쓰려고 의도했다.
```js
const mixed = ['1',2];
```

- 그럴 때 명시적 type 선언을 해주자.
```js
const x: string = 'x';

const mixed: [string, number] = ['1',2];
```

- 물론 명시적 type을 남발하는 게 좋은 것은 아니다.
- 그런 코드가 많을수록 refactoring이 어려워진다.
- typescript의 자동추론에 맡기는 게 좋다.

- 객체의 경우에는 as const를 활용할 수 있다.
- as const를 쓰면 key-value 조차 절대 바뀌지 않는 상수가 된다.
```js
const v3 = {
    x:1,
    y:2
}
```

## <span style="color:#802548">_객체의 타입형성- object_</span>
- js와 달리 ts는 객체를 한꺼번에 만들어야 한다.
- 그 이유는 ts는 처음 객체를 만들 때 type을 결정하는 경우가 대다수기 때문이다.
- 나중에 property를 동적으로 추가하면 안 된다.
```js
const abc = {};
abc.x = 'y'; //error. abc는 {} type이기 때문에 property를 추가할 수 없다.
```

- 객체 전개를 통해 동적으로 추가가 가능하긴 하다.
```js
let hasMiddle: boolean;
const firstLast = {first: 'Harry', last: 'Truman'};
const president = {...firstLast, ...(hasMiddle ? {middle: 'S'} : {})};
```

- 그럼 type이 두 개중에 하나게 된다.
```js
president: {
    middle?: string;
    first: string;
    last: string
}

////

president: {
    first: string;
    last: string;
} |
president: {
    middle: string;
    first: string;
    last: string
}
```

- 만약 optional property가 되길 원했다면 아래와 같이 helper 함수를 써야 한다.
```js
function addOptional<T extends object, U extends object>(a: T, b: U | null): T & Partial<U> {
    return {...a, ...b}
}

const firstLast = {first: 'Harry', last: 'Truman'};

const president = addOptional(
    firstLast,
    hasMiddle ? {middle: 'S'} : null
)
```

- union type이 아니라 optional property를 가진 type으로 나온다.
```
president {
    middle?: string
    first: string;
    last: string;
}
```

## <span style="color:#802548">_객체의 타입형성- function_</span>
- 함수의 returnType을 명시해두면, 호출된 곳이 아니라, 함수가 정의된 곳에서 오류가 난다.
- 그럼 오류를 잡기가 좋다.

- 아래는 return type을 명시하지 않았을 때다.
- number 형식에 then 속성이 없습니다라는 오류가 뜬다.
- 함수를 호출한 곳에 알아먹기 어려운 오류가 뜬다. 


```js
function getQuote(ticker: string) {
    .
    .
    .
    return cache[ticker]
}
getQuote('MSFT').then(consiedBuying); //number 형식에 then 속성이 없습니다
```

- 그럼 return type을 줘보자.
- 그러자 이제 제대로 표시된다.
- number 형식은 Promise(number)형식에 할당 불가능하다라는 오류가 정의부에서 뜬다.
```js
function getQuote(ticker: string) : Promise<number>{
    .
    .
    .

    return cahce[ticker]; // number 형식은 Promise<number>형식에 할당 불가능하다.
}
```

## <span style="color:#802548">_객체의 타입 판별 함수 만들기_</span>
- type을 판별하는 함수를 도입할 수 있다.
- 아래와 같은 형태다.
```js
function isCertainType(parameter: wantedType): parameter is wantedType {
    return must-have property in parameter; //무조건 boolean return
}
```

- 실제 예시는 아래와 같다.


```js
function isInputElement(el: HTMLElement): el is HTMLInputElment {
    return 'value' in el;
}
function getElementContent(el: HTMLElement) {
    if(isInputElment) {
        return el.value; // isInputElement가 true인 경우, HTMLInputElement type이다.
    }
    
    return el.textContent; //isInputElement가 false인 경우, 그냥 HTMLElement다.
}
```

- 이같은 type guard 함수를 사용하면, 고차함수에서도 undefined를 거를 수 있다.
- 아래처럼 filter 안에서 !== undefined 조건절로는 type이 string[]로 만들 수 없다.
```js
const jackson5 = ['Jackie', 'Tito', 'Jermnire','marion','Michael'];
const members = ['Janet', 'Michael'].map(who => jackson5.find(n => n === who)); // 이 경우 members의 type은 (string | undefined)[]
const filteredMembers = ['Janet', 'Michael'].map(who => jackson5.find(n => n === who).filter(who => who !== undefined)); // 이 경우 members의 type은 (string | undefined)[]
```

- 아래와 같은 type guard 함수를 넣어줘야 한다.
```js
function isDefined<T>(x: T | undefined): x is T {
    return x !== undefined
}
const filteredMembers = ['Janet', 'Michael'].map(who => jackson5.find(n => n === who).filter(isDefined)); // 이 경우 members의 type은 string[]
```



## <span style="color:#802548">_객체나 배열로 감싸서 필수값을 check하라_</span>
- 어느 경우에도 필요한 값 같은 경우도 따로 변수를 두지 않고 배열, 객체로 포괄해서 만들자.
- max가 없을 때 if문을 만드는 것을 까먹고 넣지 않았다.
- 그러자 undefined 오류가 뜬다.
```js
function extent(nums: number[]) {
    let min, max;
    for (const num  of nums) {
        if (!min) {
            min = num;
            max = num;
        } else {
            min = Math.min(min, num);
            max = Math.max(max, num);
        }
    }
    return [min, max];
}

const [min, max] = extent([0,1,2]);
const span = max - min; //max is possibly undefined
                        //min is possibly undefined
```


- 그걸 아래와 같이 배열로 바꿔주자.
```js
function extent(nums: number[]) {
    let result: number[] | null = null;
    for (const num  of nums) {
        if (!result) { // null check해줘야 함. 안 해주면 아래 result[0]에서 null일수 있다는 오류가 남.
            result = [num,num];
        } else {
            result = [Math.min(num,result[0]),Math.max(num,result[1])];
        }
    }
    return result;
}

const [min, max] = extent([0,1,2])!; // null 없음 단언 해줘야 함. 아니면 Type '[number, number] | null' is not an array type.
const span = max - min;
```

- 객체도 마찬가지다.
- 아래와 같이 birth의 경우 place와 date를 받으려고 한다.
```js
interface Person {
    name: string;
    placeOfBirth?: string;
    dateOfBirth?: date;
}
```


- 그런데 사실 두개를 한꺼번에 받거나, 아예 안받거나다.
- 그 경우 아래와 같이 interface를 통해 해당 방식으로 객체 구현을 강제할 수 있다.
```js
interface Person {
    name: string;
    birth?: {
        place: string;
        date: Date;
    }
}

cosnt alanT: Person = {
    name: 'Alana Turing';
    birth: {
        place: 'London';
    }
} // error. birth 안에 date 속성이 없습니다.
```

- 만약 기존 interface를 수정할 수 없는 환경이라면?
- 잘게 쪼개주고 union type으로 만들어야 한다.
```js
interface Name {
    name: string;
}
interface PersonWithBirth extends Name {
    placeOfBirth?: string;
    dateOfBirth?: date;
}
type Person = Name | personWithBirth;

function getPersonIfBirth(p: Person) {
    if ('placeOfBirth' in p) {
        const {dateOfBirht} = p //dateOfBirth는 Date type
    }
}
```


## <span style="color:#802548">_좋은 type- 유효한 상태를 발견하라_</span>
- 좋은 type을 설계하려면, 유효한 상태만 표현해야 한다.
- 가령 아래와 같은 interface가 있다고 해보자.
- 아래의 interface는 loading이 true면서 error가 true인 상황을 처리할 수 없다.

```js
interface State {
    pageText: strting; 
    isLoading: boolean;
    error?: string
}
```

- 위의 interface에는 아래와 같은 함수가 나올 수 밖에 없다.


```js
function rednerPage(state: State) {
    if (state.error) {
        return `Error! Unable to load ${currentPage} : ${state.error}`;
    } else if (state.isLoading) {
        return `Loading ${currentPage}`;
    }

    return `<h1>${currentPage}</h1>\n${state.pageText}`
}

async function changePage(state: State, newPage: string) {
    state.isLoading = true;
    try {
        const response = await fetch(getUrlForPage(newPage));
        if (!response.ok) {
            throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
        }
        const text = await response.text();
        state.isLoading = false;
        state.pageText = text;
    } catch (e) {
        state.error = '' + e;
    }
}
```

- 위의 changePage 함수는 아래와 같은 문제가 있다.
  - 오류가 발생했을 때 state.isLoading = false로 바꿔주지 않는다.
  - state.error를 초기화하지 않았기에, 페이지 전환 중에 로딩 메시지 대신 과거의 오류 메시지를 보여준다.
  - 페이지 로딩 중에 사용자가 페이지를 바꾸면, 벌어질 일을 예상하기 어렵다.
    - 새 페이지에 오류가 뜰 수도 있다. 아니면
    - 응답이 오는 순서에 따라 두번쨰 페이지가 아닌 첫번쨰 페이지로 전환될 수 있다
- 그럼 이제 type을 유효하게 나눠보자.


```js
interface RequestPending {
    state: 'pending'
}
interface RequestError {
    state: 'error',
    error: string;
}
interface RequestSuccess {
    state: 'ok',
    pageText: string;
}

type RequestState = RequestPending | RequestError | RequestSuccess;
interface State {
    currentPage: string;
    requests: {[page: string]: RequestState}; //index signature
}

function renderPage(state: State) {
    const {currentPage} = state;
    const requestState = state.requests[currentPage];
    const {state} = requestState;
    switch (requestState.state) {
        case 'pending':
            return `loading ${currentPage}`;
        case 'error':
            return `Error! Unable to load ${currentPage}: ${requestState.error}`;
        case 'ok':
            return `<h1>${currentPage}</h1>\n${requestState.pageText}`;
    }
}

async function changePage(state: State, newPage: string) {
    state.requests[newPage] = {state: 'pending'};
    state.currentPage = newPage;
    try {
        const response = await fetch(getUrlForPage(newPage));
        if (!response.ok) {
            throw new Error(`Unable to load ${newPage}: ${response.statusText}`);
        }
        const pageText = await response.text();
        state.requests[newPage] = {state: 'ok', pageText};
    } catch (e) {
        state.requests[newPage] = {state: 'error', error: '' + e}
    }
}
```

- 또 다른 예시를 들어보자.
- 아래처럼 union을 합친 interface를 만들었다면, 뭔가 type 설계가 잘못 된 것이다.
```js
interface Layer {
    layout: FillLayout | LineLayout | PointLayout;
    paint: FillPaint | LinePaint | PointPaint;
}
```

- 여러 interface의 union으로 type을 지정해야 한다.
```js
interface FillLayer {
    layout: FillLayout;
    paint: FillPaing;
}
interface LineLayer {
    layout: LineLayout;
    paint: LinePaint;
}
interface PointLayer {
    layout: PointLayout;
    paint: PointPaint;
}
type Layer = FillLayer | LineLayer | PointLayer;
```

- 만약 runtime에서도 interface 정보를 활용하고 싶다면 tagged union으로 만들면 된다.
```js
interface FillLayer {
    kind: 'fill';
    layout: FillLayout;
    paint: FillPaing;
}
interface LineLayer {
    kind: 'line';
    layout: LineLayout;
    paint: LinePaint;
}
interface PointLayer {
    kind: 'point';
    layout: PointLayout;
    paint: PointPaint;
}

function drawLayer(layer: Layer) {
    if (layer.kind === 'point') {
        cosnt {paint} = layer; //paint의 type은 PointPaint
    } else if (layer.kind === 'fill') {
        const {paint} = layer; //paint의 type은 FillPoint
    }
}
```

## <span style="color:#802548">_좋은 type- 사용할 떄는 너그럽게, 생성할 땐 엄격하게_</span>
- 생성할 떄 모두 optional property로 줘서 쉽게 만들 수도 있다.
- 하지만, 이것은 좋지 않다. 차라리 interface를 원본과 option으로 나누자.


```js
interface LngLat {
      lng: number;
      lat: number;
    }

type LngLatLike = LngLat | {lon: number; lat: number; } | [number, number];

interface Camera {
    center: LngLat;
    zoom: number;
    bearing: number;
    pitch: number;
}

interface CamerOptions extends Omit<Partial<Camera>, 'center'> {
    center?: LngLatLike;
}
/*
interface CamerOptions {
  center?: LngLatLike;
  zoom?: number;
  bearing?: number;
  pitch?: number;
}
*/
```



## <span style="color:#802548">_좋은 type- string 보다 구체적인 type을 사용하자._</span>
- string type은 어떤 값이 들어올 지 모르는 문자열에 대해서만 활용하자.
- 만약 들어올 값이 정해져 있다면, literal type을 활용해 구체화 하자.
```js
interface Album {
    artist: string;
    title: string;
    releaseDate: string;
    recordingType: string;
}

const kindOfBlue: Album = {
    artist: 'Miles Davis',
    title: 'Kind of Blue',
    releaseDate: 'August 17th, 1959',
    recordingType: 'Studio'
}

type RecordingType = 'studio' | 'live';
interface Album {
    artist: string;
    title: string;
    releaseDate: Date;
    recordingType: RecordingType;
}

const album: Album = {
    artist: 'abc',
    title: 'ad',
    releaseDate: new Date('2014-02-11'),
    recordingType: 'Studio' //error. studio라고 써야 하는데, 대문자로 씀. 그럼 literal type은 catch 가능!
}
```

- 함수에 특히 parameter를 string으로 주는 것은 좋지 않다.
- 아래와 같은 함수가 있다고 해보자.
```js
function pluck(records, key) {
    records.map(r => r[key])
}
```

- 해당 함수를 아래와 같이 바꿀 수있다.
```js
function pluck(records: any[], key: string): any[] {
    return records.map(r => r[key]);
}
```

- 해당 함수에 generics를 써서 string을 감춘다.
- 그러나 undefined type도 같이 딸려온다.
- T[keyof T]면 T[string], T[Date], T[RecordingType]이 가능하다.
- 이중 T[string]의 경우, 해당하는 key가 없어 undefined를 return할 수 있다. 
- 따라서 (string | undefined)[]로 나온 것이다.
```js
function pluck<T>(records: T[], key: keyof T): T[keyof T] {
    return records.map(r => r[key])
}

const releaseDates = pluck(albums, 'releaseDate')// type이 (string | undefined)[]
```

- undefined가 딸려오지 않게 extends를 활용한다.
- 그럼 K는 절대로 아무런 문자열 key를 받을 수 없게 된다.
- 무조건 T interface 안에 있는 key를 받아오게 되므로 undefined는 사라진다.
- 더불어 key가 잘못되면 오류도 잡을 수 있다.
```js
function pluck<T, K extends keyof T>(records: T[], key: K): T[K][] {
    return records.map(r => r[key])
}

pluck(albums, 'releaseDate');   //타입이 Date[]. undefined[]는 있을 수 없다.
pluck(albums, 'recordingDate'); //error. recordingDate property is not a type of album.
```

## <span style="color:#802548">_any 지양- parameter로서 any를 최소화 적용하라_</span>
- any는 기본적으로 최소화해야 한다.
- 아래처럼 x에 any type으로 선언하면 안 된다.
```js
function f1() {
    const x: any = expresssionReturningFoo();
    processBar(x);
}
```

- 호출되는 parameter를 직접 넣을 때 any로 바꿔줘야 한다.
```js
function f2() {
    const x = expressionReturningFoo();
    processBar(x as any);
}
```

- 객체도 마찬가지다.
- 아래같이 전체 객체를 any로 감싸면 type정보가 다 사라진다.
```js
const config: Config = {
    a: 1,
    b: 2,
    c: {
        key: value
    }
} as any
```

- 모르는 부분만 any 단언을 해준다.
```js
const config: Config = {
    a: 1,
    b: 2,
    c: {
        key: value as any
    }
}
```

- 함수를 정의할 때도 적어도 배열인 것만 알아도, 그 정보를 제공하자.
  - 그냥 any가 아니라 any[]와 같이 써주자.
- any로 쓰면 length를 return할 때 number가 아니라 any type이 된다.
    - any[]로 쓰면 length를 return할 때 number type이 된다.
```js
function getLengthBad(array: any) {
    return array.length; //any type
}
```


- 어쨌든 any[]라서 length 속성은 number type이다.
```js
function getLengthBad(array: any[]) {
    return array.length; // number type
}
```

- 객체를 정의할 때도 마차간지다.
- 만약 value를 모른다면, 그냥 any로 쓰지 말자.
```js
function hasTwelveLetterKey(o: any) {
    for (const key in o) {
        if (key.length === 12) {
            return true;
        }
    }
    return false;
}
```

- 아래와 같이 object 형태를 명시해주자.
```js
function hasTwelveLetterKey(o: {[key: string]: any}) {
    for (const key in o) {
        if (key.length === 12) {
            console.log(key, o[key]); //o[key]에 접근 가능함.
            return true;
        }
    }
    return false;
}
```

- 위의 type은 object와 동일하다.
- 다만 object type으로는 속성에 접근할 수가 없다.
```js
function hasTwelveLetterKey(o: object) {
    for (const key in o) {
        if (key.length === 12) {
            console.log(key, o[key]); //o[key]에 접근 불가능함.
            return true;
        }
    }
    return false;
}
```

- 객체로 속성에 접근할 필요가 있다면, 위처럼 {[key: string]: any}나 unknown을 쓰자.
```js
function hasTwelveLetterKey(o: unknown) {
    for (const key in o) {
        if (key.length === 12) {
            console.log(key, o[key]); //o[key]에 접근 가능함.
            return true;
        }
    }
    return false;
}
```

## <span style="color:#802548">_any 지양- return은 any가 아닌 unknown_</span>
- any를 return type으로 하지 말아야할 이유가 또 있다.
- any를 쓰면 잉여 속성 체크가 발동하지 않는다.
  - 그러니 모른다면 차라리 unknown으로 하자.
  - 그럼 타입 단언으로 원하는 객체로 바꿀 수 있다.
```js
function parseYAML(yaml: string): any {
    .....
}

function safeParseYAML(yaml: string): unknown /* null이나 undefined를 return하지 않을 확신이 있다면 {}도 괜찮다. 하지만 eslint에서 막으니까 쓰지 말자.*/{
    return parseYAML(yaml); // any는 unknown type으로 바뀔 수 있다.
}
const book = safeParseYAML(`
    name: Villette
    author: Charlotte Bronte
`)
alert(book.title);  //type check 통과. 그러나 runtime에서 error 뜸.
```


- 그럼 호출하는 쪽에서 원하는 타입으로 바꿔줄 수 있다.
```js
interface Book {
    name: string;
    title: string;
}

const book = safeParseYAML(`
    name: Villette
    author: Charlotte Bronte
`) as Book;
alert(book.title);  //error. Book 형식에는 title 속성이 없습니다.
```

- generics를 이용하는 경우도 있다.
- 하지만 호출하는 쪽이 아니라 정의하는 쪽에서 강제하는 것이라 쓰지 말자
- 사용자가 원하는 객체로 바꿀 수 있게 해주는 쪽이 바람직하다.
```js
function safeParseYAML<T>(yaml: string): T {
    return parseYAML(yaml);
}
```

- parameter를 unknown으로 받을 때 타입을 고정시키는 것은 꼭 타입 단언만 있지 않다.
- 아래와 같이 helper, instanceof를 사용할 수도 있다.
```js
function isBook(val: unknown): val is Book {
    return (
        typeof(val) === 'object' && val !== null && 'name' in val && 'author' in val
    )
}

function processValu(val: unknown) {
    if (isBook(val)) {
        val; //type이 Book
    }
}

function processValue(val: unknonw) {
    if (val instanceof Date) {
        val // 타입이 Date
    }
}
```


## <span style="color:#802548">_any 지양- 암시적 any는 아예 쓰지 말자_</span>
- 변수를 암시적 any로 쓰는 것도 매우 안 좋다.
- 아래같이 쓰면 any[]이 된다.
- 그런데 값을 넣으면 type이 계속 확장된다.
```js
const array = [];   //any[] type
array.push('a');    //string[] type
array.push(1);      //(string | number)[] type
```


- 이건 매우 좋지 않다. 배열을 사용할 때는 처음부터 type을 명시하고 사용하자.
```js
const list: number[] = [];
list.push('11')     //error.  rgument of type 'string' is not assignable to parameter of type 'number'.
```


- 분기문에서도 암시적 any의 진화가 일어난다.
```js
let val = null; // any type
try {
    val = 12; 
    val         // number type
} catch(e) {
    console.warn('alas!');
}
val; //number | undefined
```

- 따라서 선언만 할 때도 type을 명시하자.
```js
let val: number;    // number;
try {
    val = 12;       // number
} catch(e) {
    console.log('alas!');
}
val;                // number
```

## <span style="color:#802548">_any 지양- any가 쓰인 곳 찾아내기_</span>
- any가 project의 어디에 쓰였는지 알 수 있는 방법으로 type-coverage package가 있다.
```
npm i type-coverage
npx type-coverage --detail
```

ㅌ

## <span style="color:#802548">_function parameter this binding_</span>
- class 내에서 this를 쓰는 경우, 아래와 같이 보통 bind를 쓴다.
```js
class ResetButton {
    constructor() {
        this.onClick = this.onClick.bind(this);
    }

    render() {
        return makeButton({text: 'Reset', onClick: this.onClick});
    }

    onClick() {
        alert(`Reset ${this}`);
    }
}
```

- 아니면 화살표함수를 쓴다.
```js
class ResetButton {
    constructor() {
        this.onClick = this.onClick(this);
    }

    render() {
        return makeButton({text: 'Reset', onClick: this.onClick});
    }

    onClick = () => {
        alert(`Reset ${this}`);
    }
}
```

- 아니면 좀 옛날 방식으론 대리변수를 쓴다.
```js
class ResetButton {
    constructor() {
        var self = this;
        this.onClick = function() {
           alert("Reset" + self);
        }
    }

    render() {
        return makeButton({text: 'Reset', onClick: this.onClick});
    }
}
```

- 그런데 parameter의 경우가 문제다. 여기서 this를 쓰려하면 문제가 발생한다.
```js
function addKeyListener(el: HTMLElement, fn: (e: KeyboardEvent) => void) {
    el.addEventListener('keydown', e => {
        fn.call(e); //call이 무조건 있어야 한다. fn.call()로 불러야 한다. 하지만 그럼 오류가 난다. this가 먼저 규정되어야 하기 때문이다.
    })
}
const element = document.querySelector('input') as HTMLElement;
addKeyListener(element,function(e) {
    console.log(e, this.innerHTML); //'this' implicitly has type 'any' because it does not have a type annotation.
})
```


- 그 경우, this를 함수 정의에 포함시키고, call로 부른다. 
- call()은 this를 첫번째 parameter로 받는다. 여기서 this의 type을 지정해준다.
- 또한 callback함수의 첫번쨰 parameter와 원함수의 첫번쨰 parameter의 type은 동일해야 한다.
```js
function addKeyListener(el: HTMLElement, fn: (this: HTMLElement, e: KeyboardEvent) => void) {
    el.addEventListener('keydown', e => {
        fn.call(el, e); //call이 무조건 있어야 한다.
    })
}
const element = document.querySelector('input') as HTMLElement;
addKeyListener(element,function(e) {
    console.log(e, this.innerHTML);
})
```

- 이 경우, 화살표 함수를 사용하기 어렵다는 단점이 있다.
- listener들은 뭔가 return 하는 경우가 없어 void인데, 화살표함수는 상위 컨텍스트에 this를 binding한다.
- 그런데 innerHTML이 상위 컨텍스트에 없기 때문에 오류가 난다.
```js
function addKeyListener(el: HTMLElement, fn: (this: HTMLElement, e: KeyboardEvent) => void) {
    el.addEventListener('keydown', e => {
        fn.call(el, e); //call이 무조건 있어야 한다.
    })
}
const element = document.querySelector('input') as HTMLElement;
addKeyListener(element,(e) => {
    console.log(e, this.innerHTML); //Property 'innerHTML' does not exist on type 'void'.
})
```


## <span style="color:#802548">_enum이 아닌 as const_</span>

- const가 아닌 enum을 쓰면 자동완성을 지원해준다.

```JS
enum OrderStatus {
    PENDING   = 10,
    SHIPPED   = 41,
    DELIVERED = 153,
    RETURNED  = 524
}

const myStatus = OrderStatus.DELIVERED;
function isDelivered(status: OrderStatus) {

    return status === OrderStatus.DELIVERED
}

isDelivered(OrderStatus.RETURNED);
```

- 하지만, enum은 ts에서 compile 되고 나서 사라지지 않고 남아 js에 영향을 미친다.
- 따라서 as const를 많이 사용한다. 

```javascript
const enum ArrowKeys {
    UP = "up",
    DOWN = "down",
    LEFT = "left",
    RIGHT = "right",
    ERROR = 500
}

const ArrowKeys = {
    UP = "up",
    DOWN = "down",
    LEFT = "left",
    RIGHT = "right",
    ERROR = 500
} as const
```


## <span style="color:#802548">_try catch 시 error 다루기_</span>

- 에러를 다룰 때 error는 unknown type이다.
- if문으로 걸러도 {} type이 되기 때문에 property가 없다.

```js
try {
     
} catch( error) {
    if (error) {
        error.message; //property message does not exist on type '{}'
    }
}
```


- error를 if문에서 Error로 바꿔도 어차피 까먹는다.

```js
try {

} catch (error) {
    if( error as Error) {
        error.message; //error is of type 'unknown'
    }
}
```


- 그럴 땐 변수로 만들어 놔야 한다.
- type assertion은 변수에 적용해야만 타입이 assertion한 type으로 유지된다.

```js
try {

} catch( error) {
    const err = error as Error;
    if (err) {
        err.message; //no error
    }
}
```

- as를 안 쓰는 게 더 좋다.

```js
try {

} catch ( error) {
    if ( error instanceof Error) {
        error.message;
    }
}
```
