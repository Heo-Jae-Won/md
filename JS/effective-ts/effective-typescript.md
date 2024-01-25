# ts 실력을 키우기 위한 기본 설정
- noImplicitAny 설정
- strictNullChecks 설정
- NoImplictthis 설정
- no_inferred-types 설정

```
만약 ts 예제 오류 재현이 안된다?
tsconfig.json을 살펴보자.
```

# ts 오류는 실행불가능한 오류가 아니다.
- typescript 오류는 compile 오류가 아니다.
  - type check 오류다.
  - 실제 runtime에서 일어나는 오류가 compile 오류다.
  - 그런데 runtime에는 type check가 불가하다.
- 그럴 때 type을 속성으로 받는다.
- 이를 tagged union이라고 한다.


# tagged union
```js
interface Square {
    kind: 'square';
    width: number;
}

interface Rectangle {
    kind: 'rectangle';
    height: number;
    width: number;
}

type Shpae = Square | Rectangle;
function calculateArea(shape: Shape) {
    if (shape.kind ==='rectangle') { //shpae instanceof Rectangle 사용 불가. instance는 runtime에만 존재. ts의 type은 runtime에 존재하지 않는다.
        shape //Rectangle type
    } else {
        shape //Square type
    }
}
```

# 형변환은 여전히 필요하다
- return variable에 type assertion을 하는 것은 type check를 속이는 것일 뿐이다.
  - runtime에서는 무의미하다. 
  - 그럴 땐 ts가 아닌 js의 형변환을 사용하는 것이 좋다.


```js
//bad
function asNumber(val: number | string) {//val은 string도 될 수 있으니 as number로 썼다.
    return val as number;
}

//
function asNUmber(val: number | string) : boolean {
    return typeof(val) === 'string' ? Number(val) : val;
}
```

# 함수 오버로딩
- ts에서는 함수를 2개 이상 overloading하여 구현할 수 없다.



```js
function add(a: number, b: number) {
    return a +b; //compile error. Duplicate function implementation
}

function add(a: string, b: string) {
    return a +b; //compile error. Duplicate function implementation
}
```

- 2개의 overloading된 type을 갖는 fn을 선언하자. 
  - 구현이 아닌 선언만 한다.
  - 구현부는 하나로 한다.



```js
function add(a: nubmer, b: number) : number;
function add(a: string, b: string) : string;
function add(a,b) {
    return a + b
}
```

- 다만 위와 같이 해도, union type의 문제가 남아있다.
- 그 때 조건부 타입을 활용한다.



```js
function add<T extends number | string>(x: T): T extends string ? string : number;
function add(x: any) {
    return x+x
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

# duck typing
- typescript는 duck typing의 언어다.
- 객체가 다른 객체의 변수와 method를 가지면, 해당 객체 타입도 다른 객체 type으로 간주된다.
  - runtime에는 쓸모가 없지만, type check 시에 유용하다.


```js
interface NamedVector {
    name: string,
    x: number,
    y: number
}

interface Vector2D {
    x: number,
    y: number
}
```

- 위와 같이 NamedVector가 Vector2D를 그대로 갖고 있다면, Vector2D type으로 취급가능하다.
- Java의 다형성과 비슷하다.


```js
function calculateLength(v: Vector2D) {
    ....
}

const v: NamedVector = { x: 3, y: 4, name: 'Zee'};
calculateLength(v); //분명 Vector2D type으로 parameter가 들어와야 하지만, 괜찮다. 왜? duck typing이니까.
```

- 이를 활용해 test code를 쉽게 짤 수 있다.
- duck typing을 의식하지 않고 만든 코드다.
- DB의 type이 구체적으로 박혀있어서 다른 DB로 test할 때 방해가 된다.

```js
interface Author {
    first: string,
    last: string,
}

function getAuthors(database: PostgreDB) : Author[] {
    const authorRows = database.runQuery('SELECT FIRST, LAST...');
}
```

- 이를 duck typing을 이용해 바꿔보자.
- runQuery를 key로 갖고, any[]를 value로 갖는다면 어떤 DB로든 갈아끼울 수 있다.

```js
interface Author {
    first: string,
    last: string,
}

interface DB {
    runQuery(sql: string): any[]
}

function getAuthors(database: DB) : Author[] {
    const authorRows = database.runQuery('SELECT FIRST, LAST...');
}
```


# duck typing의 문제
- duck typing이 좋은 것만은 아니다.
- 떄때로 compiler를 혼동시켜 type error가 뜨게 한다.
- 특히 Object.keys()로 객체를 순회하는 경우, key가 문자열로 return된다.
- 따라서 아래 axis는 type이 늘 string이다.

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

calculateLengthL1({x:123,y:11,z:14,abc:123}) //abc는 key가 아니지만.. type check 통과
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
```

- 혹은 타입 단언을 활용해야 한다.
- axis가 string으로 온다고 해도, Vector3D의 key임은 변함이 없다.
- 따라서 Vector3D의 key임을 명확하게 알린다.


```js
const calculateLengthL1 = (v: Vector3D) => {
      let length = 0;
      for (const axis of Object.keys(v)) {
        const coord = v[axis as keyof Vector3D];
        length += Math.abs(coord);
      }
      alert(length);
    };
```

- 또 다른 문제는 Vector2D를 받았어야 하는데, Vector3D를 받아버리는 것과 같은 실수를 할 수 있다는 점이다.
- 다른 해결 방법은 tagged union이다.
- 다만 완벽하진 않다. 실수를 줄이는 정도다.



```js

interface Vector3D {
    x: number,
    y: number,
    z: number,
    _brand: '3D'
}
interface Vector2D {
    x: number,
    y: number,
    _brand: '2D'
}

function calculateLengthL1(v: Vector3D) {
     return Math.abs(v.x) + Math.abs(v.y) + Math.abs(v.z);
}

const vector2D = {
    x: 1,
    y: 2,
}
calculateLengthL1(vector2D); //error. 개발자가 직접 _brand:'3D'넣어줘야..
```


# keyof

```
keyof (A&B)는 (keyof A) | (keyof B)와 동일
keyof (A | B)는 (keyof A) & (keyof B)와 동일
```

# extends

- string을 extends한 것은 string 간의 union type도 포함이다.


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

- 이는 number도 마찬가지다.


```js
function getKey<K extends number>(val: any, key: K) {
   console.log(key);
}

getKey({}, 1);  //number
getKey({}, 2);  //number
getKey({}, Math.random() < 0.5 ? 1 : Math.random() < 0.7 ? 1 : 3);//<1 | 3>
getKey({}, Math.random() < 0.5 ? 1 : Math.random() < 0.7 ? '1' : 3); //error
```



# 주어진 interface로 union type의 조합들까지 커버하기
- 위의 extends의 내용을 발전시키면 아래와 같다.
- keyof로 K를 한정하여 가져오면 어떤 interface든 그 조합의 union을 커버할 수 있다.

```js
function sortBy(K extends keyof T, T)(vals: T[], key: K): T[] {
    return vals;
}

interface Point {
    x: number;
    y: number;
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


# typeof
- ts의 typeof를 쓰면 값과 타입에 대해 다른 연산이 일어난다.
- type의 typeof는 js코드로 사용이 불가능하다.
- const같은 값의 typeof는 js코드로 사용이 가능하다.
- 참고로 class는 js에서 함수로 구현되므로 값의 관점에서 typeof를 하면 function.
- class로 만든 instance의 typeof는 object
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

const t1 = typeof ScienceFacility;
const t1 = typeof scienceFacility ;
console.log(t1); //fucntion
console.log(t2); //object
```


# fn의 parameter에 object literal를 넣는 경우
- fn의 parameter를 object로 받는다면, type을 선언하는 방식은 아래와 같다.


```js
function email(options: {person: Person, subject: string, body: string});
```

- email 함수를 위와 같이 선언했다면 비구조화 할당해서 사용한다면 아래와 같다.


```js
function email({person, subject, body} : {person: Person, subject: string, body: string})
```


# 잉여속성 체크. 
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

- 그럼 의미에서 함수 선언보다는 함수 표현식이 좋다.
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

- 이제 타입 선언을 알아보자.
- 타입선언은 매우 구리다.
- 잉여속성 체크가 안 된다.


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

- 다만 type 단언이 필요한 경우가 있다.
- 바로 DOM 객체를 다룰 때다.


```js
const 

```

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

# index signature

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

- 그 때 index signature를 도입하면 문제가 해결된다.
- 하지만 임시방편이다.
- 아래와 같은 index signature는 string 형식의 key면 무엇이든 받는다.
  - 그것이 실제 Schedule interface에 존재하지 않아도 말이다.

```js
interface Schedule {
    date: string,
    headCount: number
    [key: string]: string | number
}
```

- 따라서 아래와 같이 써도 오류가 나지 않게 된다.
- 이건 명백한 오류다. 'ddd'라는 property는 존재하지 않기 때문이다.
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

# interface는 중복제거로도 좋다.
- interface는 중복을 제거하는 데 있어서도 꽤나 중요하다.


```js
function distance(a: {x: number, y: number} , b: {x: number, y: number}) {
    return Math.sqrt(Math.pow(a.x - b.x, 2) + Math.pow(a.y - b.y, 2));
}
```


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

# Pick 패턴

- 기존의 interface에서 일부 property만 가져오고 싶을 떄 사용하는 pattern이다.

- 아래는 노가다 interface다.
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


# Partial 패턴
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

# key가 아닌 값을 type으로(엔간하면 이렇게 쓰면 안 된다)

- 값을 type으로 받고 싶을 수도 있다.
- 예를 들어 아래와 같은 형태의 변수가 있다고 해보자.


```js
const INIT_OPTIONS = {
    width: 641,
    height: 'F100FF00',
    label: 'VGA'
}
```

- 저 변수의 형태를 type으로 갖고 싶다면 interface를 만들수도 있다.


```js
interface Options {
    width: number,
    height: string,
    label: string
}
```


- 하지만 간단하게 아래와 같이 만들 수도 있다.
- 이런 방식은 설계상 나쁘다. type이 있은 뒤에 값을 그 type에 맞게 쓰는 것을 연습하자.

```js
type Options = typeof INIT_OPTIONS;
```

- 만약 함수의 returnType을 명명된 type으로 하고 싶다면?
- 직접 쓸 수도 있다.

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


# Record 패턴

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
