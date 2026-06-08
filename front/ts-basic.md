## <span style="color:#802548">_타입 추론_</span>
- let은 바뀔 수 있고, const는 바뀔 수 없다.
- 따라서 const 원시형 변수는 좀 더 좁게, literal type으로 추론한다.
- 다만 특이한 점은 let 변수에 null과 undefined를 도입하면 any type이 된다.

```js
let abc = 'string';   //string
const abc = 'string'; //string

let n = null; //any type
let a = undefined; //any type
```

- const도 객체일 때는 속성을 바꿀 수 있기 때문에 넓게 추론된다.

```js
const abc = {
    'dd' : 'ss'; 
}
 
//{ dd: string}이다. {dd: 'ss'}가 아니다.
```

- const 객체가 변하지 않는다면 as const를 붙여주자.

```js
const abc = {
    'dd' : 'ss'; 
} as const;
 
//{ readonly dd: 'ss'}
```

- type 표기 시 더 넓은 타입으로 지정하는 것은 문제가 없다.

```js
const str: 'hello' = 'hello';
const str: string  = 'hello';
const str: {}      = 'hello';
```

- {} type은 객체를 의미하지 않고, null과 undefined를 제외한 모든 타입이다.
- 여기서 null과 undefined까지 포함하면 unknown type이다.
- 배열도 type으로 당연히 지정된다.
- 3번째 index는 사실 없다. 하지만 오류가 안 난다.
- array[3]도 number type으로 추론되기 때문이다.

```js
const array =[123, 4, 56]; //number[]
array[3].toFixed(); //no compile error
```

- 그게 싫다면 tuple로 선언해야 한다.

```js
const array: [number, number, number] = [123,4,56];
array[3].toFixed(); //error
```

- 그러나 tuple로도 array에 사용되는 push, pop, unshift, shift 등은 막을 수 없다.
- 그럼 tuple인데도 자릿수가 하나 추가되거나 비는 상황이 발생한다. 
- 그런 상황을 막으려면 readonly로 선언해야 한다.

```js
const tuple: [number, number, number] = [123,4,56];
tuple.push(3); //no compile error

const tuple: readonly [number, number, number] = [123,4,56];
tuple.push(3); //error
```

- tuple은 자릿수가 고정인 배열이 아니다.
- 각 요소 자리에 type이 고정된 배열이다.
  - spread operator로 쓰면 아래와 같이 2개도 넣을 수 있다.
  - 3개도, 아니 무한하게 넣을 수 있다.
  
```js
const strNumBools: [string, number, ...boolean[]] = ['hi', 123, false, true, false]; 
```

- tuple에서도 optional property를 넣을 수 있다.
- optional property 자리에 값이 없다면 undefined로 할당된다.
- 실제로는 아래와 같은 경우, [5, undefined, undefined]라고 생각하면 된다.
- 따라서 없다고 그냥 무시하고 string을 넣으면 error가 난다.

```js
let tuple: [number, boolean?, string?] = [1, false, 'hi'];
tuple = [7, 'no']; //error. undefined가 들어가야함.
// tuple = [7, undefined, 'no'] --> O
```

- spread operator를 써도 typescript의 추론은 작동한다.

```js
const arr1 = ['hi', true];
const arr = [46, ...arr1]; //(string | number | boolean)[]
```

- ts에만 존재하는 type은 아래와 같다.
- any
  - any는 사실상 쓰면 안된다.
  - js -> ts로 migration할 때
  - api에서 무엇을 return하는 지 모를 때
  - 함수 선언문에서 error를 throw할 때
  - 함수 선언문에서 infinite loop를 돌 떄
- never
  - 함수 표현식에서 error를 throw할 때
  - 함수 표현식에서 infinite loop를 돌 때
- {}, Object
  - null, undefined를 제외한 모든 type이 다 들어갈 수 있다.
  - 거의 쓸 일이 없다. method도 없어 사실상 사용이 불가능하다.
  - 아무런 field도 선언하지 않은 interface도 {} type과 동일하다.

```js
interface NoProp {}
const obj: NoProp = {
    why: '에러 안 남';
}

const what: NoProp = '이게 되네';
const omg: NoPorp = null; //error. null은 빈 field의 interface type이 될 수 없다.
```

## <span style="color:#802548">_type extraction_</span>

- 객체의 key를 type으로 가져오려면, key의 name을 안 뒤에 typeof를 선언해야한다.
- keyof는 철저하게 type만을 바라보기 때문에 typeof로 type을 가져온 뒤에 keyof를 실행한다.

```js
const obj = {
    hello: 'world',
    name: 'zero',
    age: 28
}

type KeyNames = keyof typeof obj;   //type Keys = 'hello' | 'name' | 'age'
type KeyTypes = typeof obj[KeyNames]; //type Values = string | number
```

- 일부 key의 type만 가져오는 방법도 있다.

```js
const obj = {
    hello: 'world',
    name: 'zero',
    age: 28
}

type KeyTypes = typeof obj['hello' | 'name'];
```

- interface의 type을 가져오는 것도 가능하다.
- Original은 그 자체로 type이기에 typeof를 쓸 필요가 없다.
- 다만 in keyword는 오직 type keyword만 사용가능하다.

```js
interface Original {
    name: string;
    age: number;
    married: boolean;
}

type Copy = {
    [key in keyof Original]: Original[key];
}
//shortcut --> type Copy = Original;


/**
 * type Copy ={
 *  name: string;
 *  age:  number;
 *  married: boolean;
 * }
 */
```


## <span style="color:#802548">_type predicate function_</span>

- type predicate function을 만들면 만능이다.

```javascript
interface Cat {
    name: string,
    numLives: number
}

interface Dog {
    name: string;
    breed: string;
}


function isCat(animal: Cat | Dog): animal is Cat { 
    //parameter가 animal이기 때문에 : 뒤의 단어도 animal
    return (animal as Cat).numLives !== undefined //무조건 return하는 값은 boolean type
}

function isCat(star: Cat | Dog): star is Cat { 
    //parameter가 star기 때문에 : 뒤의 단어도 star
    return (star as Cat).numLives !== undefined //무조건 return하는 값은 boolean type
}

function makeNoise(animal: Cat | Dog): string {
    if (isCat(animal)) {
        animal // 위에서 무조건 animal은 Cat이라고 했으므로 Cat임. animal이 뭐가 오든 Cat.
        return "Meow";
    } else {
        animal //여기는 무조건 Dog
    }
}

function isCat(animal: Cat | Dog){
    return (animal as Cat).numLives !== undefined
}

function makeNoise(animal: Cat | Dog): string {
    if (isCat(animal)) {
        animal // 위에서 무조건 animal은 Cat이라고 했으므로 Cat임. animal이 뭐가 오든 Cat.
        return "Meow";
    } else {
        animal //여기는 무조건 Dog
    }
}

function isRequired<T>(value: T | undefined | null): value is T {
  return value !== undefined && value !== null;
}

function isString(value: unknown): value is string {
  return typeof value === 'string';
}

if (isString(value)) {
  //value의 type은 string으로 narrowing됩니다
  console.log(value.toUpperCase()); // value는 string이기 때문에 toUpperCase()를 호출가능
}
```

## <span style="color:#802548">_duck typing_</span>
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

- NamedVector가 Vector2D를 그대로 갖고 있다면, Vector2D type으로도 취급가능하다.
- 이는 집합과도 비슷하다. 더 큰 집합의 경우, 작은 집합은 큰 집합의 일부로 취급도니다.
- ts에서는 이를 duck typing 특성이라고 한다.

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

- duck typing에는 문제도 도사리고 있다.
- Vector2D를 받았어야 하는데, Vector3D를 type도 받을 수 있다는 점이다.
- 해결 방법은 tagged union이다. 다만 완벽하진 않다. 실수를 줄이는 정도다.

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

- 작은 집합이 큰 집합처럼 행동하는 것은 불가능하다는 의미다.

```js
interface OnlyName {
    name: string;
}

interface NameWithAge {
    name: string;
    age: number;
}

const onlyName = {
    name: 'zero'
}

const nameWithAge = {
    name: 'nero',
    age: 32
}

const zeroWithoutAge: OnlyName = onlyName;
const zeroWithoutAge: OnlyName = nameWithAge; // no error.
const neroWithAge: NameWithAge = onlyName;    // error.
const neroWithAge: NameWithAge = nameWithAge;
```

- 단, duck typing은 변수로 존재할 때만 가능하다.
- 객체 literal을 직접 넣는 경우에는 정확한 type을 검사하기 때문에 compile error가 난다.

```javascript
function printName(person: { first: string; last: string }): void {
  console.log(`${person.first} ${person.last}`);
}
printName({first: 'key', last: 'value', age: '20'}) //error. 객체 literal 직접 넣으므로 해당 key만 딱 넣어야..

const obj = {
    first: 'key',
    last: 'value',
    age: '20'
}
printNam(obj); //ok. 추가 property가 있어도 문제 없음. 변수로 넣으면 ㄱㅊ
```


- 모든 속성이 동일하면 객체 type 이름이 달라도 동일한 type으로 취급한다.
- 이도 duck typing이 갖고 있는 특성이다.
- Liter와 Money는 type이 달라도 속성이 동일하여 같은 type으로 취급되는 모습이다.

```js
interface Money {
    amount: number;
    unit: string;
}

interface Liter {
    amount: number;
    unit: string;
}

const liter: Liter = { amount: 1, unit: 'liter'};

const circle: Money = liter; //no error
```


- duck typing으로 인해 오류가 날 수도 있다.
- 아래는 분명 오류가 나지 않아야 하지만, 오류가 난다.
- duck typing을 하게 되면 실제로 value 이외에 다른 key도 들어갈 수 있기 때문이다.

```js
interface VO {
    value: any;
}

const returnVO = <T extends VO>(): T => {
    return {
        value: 'test'
    }
}

/**
 * Type '{ value: string; }' is not assignable to type 'T'.
 * '{ value: string; }' is assignable to the constraint of type 'T', 
 * but 'T' could be instantiated with a different subtype of constraint 'VO'.(2322)
 */
```


- 이런 경우에는 그냥 generics를 포기하거나 type assertion을 해야 한다.

```js
interface VO {
    value: any;
}

const f = (): VO => {
    return { value: 'test'};
}

const returnVO = <T extends VO>(): T => {
    return {
        value: 'test' 
    } as T
}
```



## <span style="color:#802548">_type assign_</span>
- type을 2개를 겹쳐서 두 type의 property를 모두 갖는 새로운 type을 만들 수 있다.

```javascript
type Circle = {
    radius: number;
}

type Colorful = {
    color: string;
}

type ColorfulCircle = Circle & colorful;

const happyFace: ColorfulCircle = {
    radius: 4,
    color: 'yellow'
}
```


- []를 지정할 때는 type[]과 같은 형식으로 지정해주어야 한다.

```javascript
const activeUsers: [] = []; //이러면 무조건 빈배열만 가능하게 됨.
activeUser.push('string') //error. 빈배열이 아니기 때문.

const activeUsers = []; // any[]와 같음. 아무거나 다 들어갈 수 있음.
const activeUsers: string[] = []; // string만 들어갈 수 있음.

const bools: Array<boolean> = []; 
const bools: boolean[] = [];
```

- 2차원 배열도 type으로 줄 수 있다.

```javascript
const board: string[][] = [["x",'o','x'], ['x','o','x'],['x','o','x']]
const demo: number[][][] = [[[1]]]
```

```javascript
const ages: number[] = [];
const gameBoard: string[][] = [];

type Product = {
    name: string,
    price: number
}

function getTotal(array : Product[]) : number {
    const sum = array.reduce((acc,cur) => acc + cur.price, 0)
    return sum;
}
```

- 여러 type을 모두 받는 mixture type도 가능하다.
- mixture type은 union type과는 다르다.
- 다만 mixture를 쓰게 되면 어떤 type이 들어오는 지 모르기에, narrowing이나 assertion이 필요하다.

```javascript
function calculateTax(price: number | string, tax: number) {
    price.replace("$", ""); //error. price가 string일 수도 있어서..
}

function calculateTax(price: number | string, tax: number) {
    if (typeof price === 'string') {
        price = parseFloat(price.replace("$", "")); // price를 parseFloat해주면 price는 number가 됨.
    } 

    return price * tax // number라서 *가능!
}

const stuff: (number | string)[] = [1,2,3,"das"]; //number나 string을 받는 배열
const stuff: number[] | string[] = [1,2,3,"das"]; //error. number로 쭉 배열 혹은 string으로 쭉 배열로 담아야함.
```

## <span style="color:#802548">_type assertion & narrowing_</span>
- mixture type 중 null과 |로 이어지는 경우, non-null assertion이 가능하다.
- ! operator, null cehck, optional chaining, typeof 4가지를 보통 사용한다.

```javascript
const btn = document.getElementById("btn")!; 
btn.addEventListener();

function readTodos() {
    const todosJSON = localSTorage.getItem("todos");
    if (todosJSON === null ) { //type narrowing
        return [];
    }

    return JSON.parse(todosJSON);
}

const btn = document.getElementById("btn"); 
btn?.addEventListener();

function triple(value: number | string) {
    if(typeof value === 'string') {
        return value.repeat(3); 
    }

    return value * 3; 
}
```

- truthy비교로도 type narrowing은 가능하다.

```javascript
const el = document.getElementById("idk");
if (el) {
    el //el이 truthy기 때문에 절대 null이 아님. 따라서 HTMLElment type이 됨. HTMLElement | null이라는 union type이 아님.
}

const printLetters = (word: string | null) => {
    for(let char of word) { // word가 null일 수 있어 error가 나옴.
        console.log(char);
    }
}

const printLetters = (word?: string) => {
    for(let char of word) { // word가 undefined일 수 있어 error가 나옴.
        console.log(char);
    }
}

const printLetters = (word?: string) => {
    if(word) {
        for(let char of word) { //type narrowing을 통과했으므로 word는 그냥 string type임.
            console.log(char);
        }
    } else {
        //여기서는 word가 빈값, 즉 falsy일 수 있기 때문에 word가 다시 string | undefined union type이 된다.
        console.log("YOU DID NOT PASS IN A WORD");
    }
    
}
```

- null이 아니라, 특정 type임을 assert할 때는 as를 쓴다.
- 보통은 이러한 type assertion은 보통 DOM조작에 쓰는데, HTML의 property가 여러 interface에 퍼져있기 때문이다.

```javascript
const input = document.getElementById("todoinput")! as HTMLInputElement; //value property는 HTMLInputElement에만 있음.
btn.addEventListener('click',function() {
    alert(input.value); //input does not have a property input. 
    input.value = "";
})
``` 

## <span style="color:#802548">_interface_</span>

- method도 interface의 property로 가질 수 있다.

```javascript
interface Person {
    readonly id: number;
    first: string;
    last: string;
    nickname?: string;
    sayHi: () => string; //sayHi(): string과 동일
}

const thomas: Person = {
    first: "h",
    last: 'd',
    id: 2154,
    sayHi: () => {
        return 5//error. string return해야.
    }
}
```

- concise method로 쓸 수 있다.

```javascript
interface Product {
    name : string,
    price: number,
    applyDiscount(discount/*discount parameter에 type을 지정하지 않아 any type */);
    applyDiscount(discount: number) : number; //아래와 동일
    applyDiscount: (discount: number) => number;
}

const shoes: Product = {
    name: 'f',
    price: 10,
    applyDiscount(amount: number/*string이라고 하면 error */) {
        const newPrice = this.price * ( 1 - amount);
        this.price = newPrice;

        return this.price;
    }
}
```

- interface도 extends를 쓸 수 있다.
- extends를 쓰면 해당 interface의 property를 모두 갖게 된다. 따라서 둘다 필요하게 된다.
- Dog의 속성을 두 번 나눠 interface에 기입해도 똑같은 interface의 속성으로 취급하기에, 4개의 속성이 필요하다.

```javascript
interface Dog {
    name: string
    age: number
}

interface Dog {
    breed: string,
    bark: () => string
}

interface ServiceDog extends Dog {
    job : "drug sniffer"
}
```

- 위처럼 추가가 가능했던 interface와 다르게 type은 향후에 추가를 할수가 없다. 
- 덮어쓰기가 되어버린다.

```javascript
type Chicken = {
    breed: string
}

type Chicken = {
    numEggs: number
}
```


## <span style="color:#802548">_generics_</span>

- method에는 generics로 return type을 명시해야 할 수도 있다.

```javascript
const nums: number[] = [];
const nums: Array<number> = [];

const inputEl = document.querySelector("#username");
console.log(inputEl);
inputEl.value = 'Hacked!' ; //error. querySelector로 가져온 것은 그저 Element고 value라는 property가 없음.

//그럴 때 어떤 type이 method에서 반환되는 지 generics로 알려줌.
const inputEl = document.querySelector<HTMLInputElement>("#username"); //<input id="username">
inputEl.value = 'Hacked!';

const btn = document.querySelector<HTMLButtonElement>(".btn"); //<button class="btn">
```


- generics를 만들지 않을 때는 아래와 같이 일일이 overloading을 해야한다.

```javascript
function numberIdentity(item: number): number {
    return item;
}

function stringIdentity(item: string): string {
    return item;
}

function booleanIdentity(item: boolean): boolean {
    return item;
}

function Identity(item: any): any {
    return item;
}
```

- generics로 만들면 여러 type에 대해 각자 overloading할 필요가 없어 편하다.

```js
function identity<Type>(item: Type): Type {
    return item;
}

identity<boolean>(6) // error. true나 false가 들어와야함.
identity<string>
identity<number>
identity<any>
identity<Cat>([]) //Cat에 맞는 형태가 와야함.


function identity<T>(item: T): T {
    return item;
}
```

- 배열도 똑같다.

```javascript
function getRandomNumberElement(list: number[]) {

}

function getRandomStringElement(list: string[]) {

}

function getRandomBooleanElement(list: boolean[]) {

}

function getRandomNCatElement(list: Cat[]) {

}

function getRandomNAnyElement(list: any[]) {

}
```

- 배열도 똑같이 generics를 쓸 수 있다.
- 자동추론이 가능하면 그렇게 맡겨주자. 그게 code가 더 깔끔하다.

```js
function getRandomTypeElement<T>(list: T[]) : T {
    const randIdx = Math.fllor(Math.random() * list.length);

    return list[randIdx];
}

console.log(getRandomElement<string>(["a","b","c"]));
console.log(getRandomElement<number>([1,2,3,4,5,6]));

console.log(getRandomElement(["a","b","c"])); //확실하면 <>로 type 알려주지 않아도 됨. ts가 추론 가능
console.log(getRandomElement([1,2,3,4,5,6])); //확실하면 <>로 type 알려주지 않아도 됨. ts가 추론 가능
```

- 2번쨰 인수부터는 T가 아니라 U로 쓴다. 
- U 다음은 V로 쓴다.
```javascript
function merge(object1, object2) {
    return {
        ...object1,
        ...object2
    }
}

const comboObj = merge({name: 'colt'}, {pets: ['blue', 'elton']});

function merge<T, U>(object1: T, object2: U) { // T 다음은 U, U 다음은 V
    return {
        ...object1,
        ...object2
    }
}
merge<{name: string}, {pets:string[]}>({name: 'colt'}, {pets: ["blue", "elton"]}) //이렇게 할 필요 없음.
const comboObj = merge({name: 'colt'}, {pets: ["blue", "elton"]}); //자동추론 가능하니 이렇게 쓰면 됨.
```

- generics type을 지정할 때도 extends를 사용할 수 있다.

```javascript
function merge<T extends object, U extends object>(object1: T, object2: U) {}
merge({name: 'Colt'}, 9); //error. object가 들어와야함.
merge({name: 'Colt'}, {num: 9});  //객체로 만드니까 잘 됨.
```

- 특정 속성이 있는 지 확인하려면 그냥 generics가 아니라 extends가 필요하다.
```javascript
function printDoubleLength<T>(thing: T): number {
    return thing.length * 2; //error. 그냥 T는 length라는 properties가 없음.
}

//length라는 property를 가지게끔 interface를 만듦.
interface length {
    length: number;
}

function printDoubleLength<T extends Lengthy>(thing: T): number {
    return thing.length * 2;
}
printDoubleLength('ABCD'); // 4
printDoubleLength(1234) //error. T로 들어오는 type에는 length 속성이 있어야 하기 때문.


function printDoubleLength(thing: Lengthy): number {
    return thing.length * 2;
} //이것도 가능함. 
```

- generics가 있는데도 주지 않으면 unknown type이 된다.
- 하지만 default값을 주면 해당 default type이 된다.

```javascript
function makeEmptyList<T>(): T[] {
    return [];
}

const strings =  makeEmptyList<string>();
strings.push('sth');

const strings =  makeEmptyList(); //strings는 unknown type이 된다.

function makeEmptyList<T = number>(): T[] {
    return [];
}

const nums = makeEmptyList(); //nums는 number type이다. unknown type이 아니다. T는 number로 기본값을 주었기 때문이다.
const strings = makeEmptyList<string>(); //type 값을 제공하면 default값은 무시된다.
```

- generics를 통해 해당 type으로 만든 객체는 해당 type만 받게끔 만들 수도 있다.

```javascript
interface Song {
    title: string;
    artist: string;
}

interface Video {
    title: string;
    creator: string;
    resolution: string;
}

class PlayList<T> {
    public queue: T[] = [];
    add(el: T) {
        this.queue.push(el);
    }
}

const songs = new PlayList<Song>();
const videos = new PlayList<Video>();
videos.add(); //video type만 넣을 수 있음.
songs.add(); //Song type만 넣을 수 있음.
```



## <span style="color:#802548">_class configuration_</span>

- ts에서는 property를 먼저 기술해야만 this로 사용이 가능하다.

```javascript
class Player {
    first: string; //먼저 기술
    last: string;  //먼저 기술
    constructor(first: string, last: string) {
        this.first = first;
        this.last = last;
    }
}
```

- 초기화 값이 정해져있다면 아래와 같이 할 수도 있다.

```javascript
class Player {
    readonly first: string;
    last: string;
    score = 0; //자동 추론
    constructor(first: string, last: string) {
        this.first = first;
        this.last = last;
    }
}
```

- ts에는 public, private같은 keyword 선언도 가능하다.

```javascript
class Player {
    readonly first: string;
    last: string;
    public score = 0; //public은 붙이지 않아도 기본값이 public이지만, 더 알아보기 쉽게 붙인 것.
    constructor(first: string, last: string) {
        this.first = first;
        this.last = last;
    }
}

class Player {
    readonly first: string;
    last: string;
    private score = 0;
    constructor(first: string, last: string) {
        this.first = first;
        this.last = last;
    }
}
```

- class에서 interface를 쓰려면 extends가 아닌 implements로 써야한다.
- 아래와 같이 축약 문법으로 쓸 수도 있다.

```javascript
interface Colorful {
    color: string;
}

class Bike implements Colorful {
    color: string;
    constructor(color: string) {
        this.color = color;
    }
}

class Bike implements Colorful {
    constructor(public color: string) {}
}

class Jacket implements Colorful {
    constructor(public brand: string, public color: string) {}
}

const bike1 = new Bike("red");
const jacket1 = new Jacket("prada", "black")
```

## <span style="color:#802548">_namespace_</span>

- interface가 다른 library랑 겹치지 않게 해주는 영역이다.
- axios library에 Request interface가 있는데, 내가 다시 field를 추가하면 overwrite된다.
- 따라서 namespace를 선언해주고 export하여 사용한다.

```js
namespace Example {
    export interface Inner {
        test: string;
    }
    export type test2 = number;
}

const ex1: Example.Inner = {
    test: 'hello'
}

const ex2: Example.test2 = 123;
```

- 다만 module 파일은 대부분 namespace가 없다.
- namespace를 import 해올 때 지정할 수 있기 때문이다.
- interface 명이 같으니까 namespace로 지정해야 할 거 같지만, 그럴 필요가 없다는 의미다.

```js
//module1.ts
export interface Test {
    name: string;
}

export default function() {
    console.log('default export');
}

//module2.ts
export interface Test {
    name2: string;
}
```

- namepsace를 지정해 import 해오면, 정확히 해당 모듈의 interface가 인식된다.
- module1과 module2가 namespace가 없더라도, 해당 파일에서는 module1과 module2를 namespace로서 자동으로 인식하는 것이다.

```js
import * as module1 from './moudle1';
import * as moudle2  from './module2';

const ex1: mouodle1.Test = {
    name: 'hi',
    name2: 'error', // 해당 속성은 없다.
}

const ex2: module2.Test = {
    name: 'error', // 해당 속성은 없다.
    name2: 'hi'
}

module1.default(); //module1.ts의 default function
```


## <span style="color:#802548">_enum과 const enum대신 추천되는 방법_</span>
- enum은 아래처럼 js로 compile 되면 쓸데없는 코드를 만든다
- 이는 쓸데없이 파일 크기를 늘리며, tree-shaking을 막을 수 있다
    - IIFE의 경우, 지워도 되는 지 bundler가 확신하기 어렵다
    - property들의 mutation이 많이 발생해서 지워도 되는 지 bundler가 확신하기 어렵다
    - 최종 결과물은 아래 주석인데, 여기서 어떤 key가 필요한 지 모르기에 지워도 되는 property를 bundler가 확신하기 어렵다
```js
var Lengths;
(function (Lengths) {
    Lengths[Lengths["MaxLength"] = 7] = "MaxLength";
    Lengths[Lengths["MinLength"] = 2] = "MinLength";
})(Lengths || (Lengths = {}));

/*
{
    "MaxLength": 7,
    "MinLength": 2,
    "7": "MaxLength",
    "2": "MinLength"
}
*/
```

- const enum도 문제를 품고 있다.
- bundler들이 build 시에는 one single file at a time의 isolatedModules 방식이다
- const enum이 파일 내에서 쓰이면 아래와 같이 Config라는 enum 정보가 사라지고 일반 변수로 변환된다.

```js
const enum Config {
    Max = 7,
    Min = 2
}
const current = Config.Max;

//compile 이후에는 let current = 7;로 변경됨.
```

- const enum의 경우, 위에서 보았듯 transpilation 시 Status라는 것 자체가 사라지고 그냥 일반 변수로 변경된다
- 따라서 다른 파일에서 참조하려하는 Status는 이미 사라진 상태다
- 즉, const enum의 경우 JavaScript object를 만들지 않기 때문에, 다른 파일에서 참고하려는 객체를 참고할 수가 없다
- 한 파일이 끝나고 다른 파일을 transcompilation하기 때문에 A 파일의 transpilation 이전 당시의 정보를 알 수가 없는 것이다.
- 따라서 js로 변환 시에 정보가 소실되어 runtime에서 error가 나게 되는 것이다

```js
export const enum Status { Active = 1 }

import { Status } from './FileA';
console.log(Status.Active); //crush
```


- enum처럼 쓸 수 있는 방법으론 아래가 추천된다.
- 이건 그냥 magic number를 대신하는 as const다.

```javascript
const TICKET_PRICE = {
  ADULT: 10_000,
  CHILD: 5_000,
  ELDER: 7_000,
} as const;
```

- 만약 함수의 parameter로 쓰고 싶다면, 아래처럼 key를 string으로 받는 방식이 추천된다.

```js
const obj = {
    MaxLength: 7,
    MinLength: 2,
    defaultLength: 5
} as const;

type KeyNames = keyof typeof obj;
function setLength(length: KeyNames) {
    const numericValue = obj[length]; // This equals 7, 2, or 5
    console.log(numericValue);
}

setLength("MaxLength");
setLength("MinLength");
setLength("defaultLength");
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