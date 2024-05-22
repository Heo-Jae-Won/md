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
array[3].toFixed(); //no error
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
tuple.push(3); //no error

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

```js
let tuple: [number, boolean?, string?] = [1, false, 'hi'];
```

- optional property 자리에 값이 없다면 undefined로 할당된다.
- 실제로는 아래와 같은 경우, [5, undefined, undefined]라고 생각하면 된다.
- 따라서 없다고 그냥 무시하고 string을 넣으면 error가 난다.

```js
tuple = [5];  //no error
tuple = [7, 'no']; //error. undefined가 들어가야함.
```

- spread operator를 써도 typescript의 추론은 작동한다.

```js
const arr1 = ['hi', true];
const arr = [46, ...arr1]; //(string | number | boolean)[]
```

- union은 type 사이에만 쓸 수 있는 게 아니다.
- type 앞에도 쓸 수 있다.
- 둘 중에 하나를 택해서 쓰면 된다.

```js
type Union1 = string | boolean | number | null;
type Union2 = 
                | string 
                | boolean 
                | number 
                | null;
```


- 구조분해 할당을 할 때는 type과 value를 완전히 구분해야 한다.
- nested를 찾으면 에러가 뜬다. nested라는 속성값을 string이라는 변수 이름에 대입 했기 때문이다.
- string을 찾아야 오류가 안나고 원하는 값이 출력된다.

```js
const { prop: {nested: string}} = {
    prop: {nested: 'hi'}
};

console.log(nested); //error. cannot find name 'nested'
console.log(string); //no error
```

- 만약 type을 굳이, 굳이! 보여주고 싶다면 아래와 같이 쓴다.

```js
const {prop: {nested}}: {prop: {nested: string}} = {
    prop: {nested: 'hi'}
};
console.log(nested);
```

## <span style="color:#802548">_ts에만 존재하는 type_</span>

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



## <span style="color:#802548">_type 별칭_</span>

- type에도 별칭을 붙여줘야 한다.
- 아래는 함수의 예시다.

```js
const func1: (value: number, unit: string) => string = (value, unit) => {
    return value + unit;
}

const func1: (value: nubmer, unit: string) => string = (value, unit) => value + unit;

type ValueWithUnit = (value: number, unit: string) => string;
const func2: ValueWithUnit = (value, unit) => value + unit;
```


- 아래는 객체의 예시다.
- 객체의 type을 미리 빼놓으면 재활용이 가능하다.
- 객체에 type declaration이 간단해지는 장점도 있다.
```js
const person1: {
    name: string,
    age: number,
    married: boolean
} = {
    name: 'zero',
    age: 28,
    married: false
}


type Person = {
    name: string,
    age: nubmer,
    married: boolean
};

const person2: Person = {
    name: 'zero',
    age: 28,
    married: false
}

const person3: Person = {
    name: 'nero',
    age: 32,
    married: true
}
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



## <span style="color:#802548">_excess property checking_</span>

- 변수로 만들면 잉여속성 검사가 발동하지 않는다.
- 객체 리터럴을 그대로 넣으면 발동한다.
- 그 이유는 객체는 대입 가능성을 판단하고, 리터럴은 잉여 속성 검사를 하기 때문이다.
- interface는 ,로도 구분가능하고, ;로도 구분가능하다. 보통은 ;로 하며, 일관되게만 쓰면 된다.

```js
interface Money {
    amount: number;
    unti: string
}

const money = { amount: 1000, unit: 'won', error: '에러 아님'};

function addMoney(money1: Money, money2: Money): Money {
    return {
        amount: money1.amount + money2.amount,
        unit: 'won'
    }
}

addMoney(money, {amount: 3000, unit: 'money', error: '에러'}); //객체 리터럴에선 에러 남.
```

- 객체의 대입 가능성을 판단한다는 의미는 ts의 type도 SQL처럼 집합의 관점에서 바라본다는 말과 이어진다.
- 더 넓은 집합이 parameter로 지정되었다면, 더 좁은 대입의 type 변수는 대입가능하다.
- 전체 집합은 unknown이고, 공집합은 never다.
- 현재는 Money interface가 더 적은 field를 가지고 있기에, 더 넓은 집합이라고 볼 수 있다.
  - 전체 인구 집합 중, 남자이며 변호사인 사람보다 남자인 사람의 집합이 더 큰 것을 생각하면 된다.    
  - 따라서 더 넓은 집합 Money interface에 더 좁은 집합인 money 변수를 넣는 것은 아무 문제가 없다.



## <span style="color:#802548">_type extraction_</span>

- 객체의 key를 type으로 가져가는 방법이다.

```js
const obj = {
    hello: 'world',
    name: 'zero',
    age: 28
}

type Keys = keyof typeof obj;   //type Keys = 'hello' | 'name' | 'age'
type Values = typeof obj[Keys]; //type Values = string | number
```

- 일부 key의 type만 가져오는 방법도 있다.

```js
const obj = {
    hello: 'world',
    name: 'zero',
    age: 28
}

type Values = typeof obj['hello' | 'name'];
```

- interface의 type을 가져오는 것도 가능하다.
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

/**
 * type Copy ={
 *  name: string;
 *  age:  number;
 *  married: boolean;
 * }
 */
```

## <span style="color:#802548">_inteface/type 상속_</span>

- interface는 extends로 상속가능하다.
- Dog, Cat interface는 모두 name field를 갖고 있게 된다.

```js
interface Animal {
    name: string;
}

interface Dog extends Animal {
    bark(): void;
}

interface Cat extends Animal {
    meow(): void;
}
```

- type을 상속하는 것은 extends가 아니라 &로 가능하다.

```js
type Animal = {
    name: string
}

type Dog = Animal & {
    bark(): void;
}
```


## <span style="color:#802548">_inteface method 선언_</span>

- 객체의 method 선언 방법이다. interface, type 모두 동일하다.
- 사용하는 데 있어 대부분 호환되지만, 공변성/반공변성 이슈가 있어 대입이 안되면 a 방식을 사용해야 한다.

```js
interface Example {
    a(): void;
    b: () => void;
    c: {
        (): void;
    }
}

type Example = {
    a(): void;
    b: () => void;
    c: {
        (): void;
    }
}
```





## <span style="color:#802548">_duck typing_</span>

- 모든 속성이 동일하면 객체 type 이름이 달라도 동일한 type으로 취급한다.
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

- duck typing은 완전히 엄격하지 않다.
- 어떤 요소를 충분히만 갖고 있다면, 잉여로 더 갖고 있어도 해당 type으로 인정된다.
- nameWithAge는 작은 집합이기 때문에 큰 집합인 OnlyName이 동일한 type으로 취급될 수 있다.
- 반면에 neroWithAge 변수를 onlyName으로 할당하는 것은 type check에서 에러가 난다.

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


## <span style="color:#802548">_inteface generics_</span>

- type과 race는 고정이지만, name과 age만 달라진다.

```js
interface Zero {
    type: 'human';
    race: 'yellow';
    name: 'zero',
    age: 28
}

interface Nero {
    type: 'human';
    race: 'yellow';
    name: 'nero';
    age: 32
}
```


- generics를 이용해 코드 중복을 줄일 수 있다.

```js
interface Person<N,A> {
    type: 'human';
    race: 'yellow';
    name: N,
    age: A
}

interface Zero extends Person<'zero',28>{}
interface Nero extends Person<'nero',32>{}
```

- genercis를 선언하는 위치는 아래와 같다.
- 객체 literal을 return할 때는 단순히 {}만 하지 않고, ({})로 한다.
- block의 start가 아니라 object literal이라는 걸 보여주기 위함이다.

```js
interface Array<T> {
    [key: number]: T;
    length: number
}

class Person<N, A> {
    name: N;
    age: A;
    constructor(name: N, age: A) {
        this.name = name;
        this.age = age;
    }
}

const personFactory = <N,A>(name: N, age: A) => ({
    type: 'human',
    race: 'yellow',
    name,
    age
})

function personFactory<N,A>(name:N, age: A) {
    return ({
        type: 'human',
        race: 'yellow',
        name,
        age
    })
}
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
 * Type '{ value: string; }' is not assignable to type 'T'.'{ value: string; }' is assignable to the constraint of type 'T', 
 * but 'T' could be instantiated with a   different subtype of constraint 'VO'.(2322)
 */
```


- 그 외로 never type이 문제가 될 수도 있다.
- T는 boolean이라 false도 되어야 하지만, never extends boolean도 참이라서, 에러가 난다.

```js
function onlyBoolean<T extends boolean>(arg: T = false): T {
    return arg;
}

/**
 * Type 'boolean' is not assignable to type 'T'.
  'boolean' is assignable to the constraint of type 'T', but 'T' could be instantiated with a different subtype of constraint 'boolean'.(2322)
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

## <span style="color:#802548">_ts의 건망증치료_</span>

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

## <span style="color:#802548">_method 구성 살펴보기_</span>

- forEach를 직접 만들어보자.
- 겹치면 안되므로 아래와 같이 my를 붙여주자.
- 그럼 에러가 난다. Array interface에 없기 때문이다.

```js
[1,2,3].myForEach(()=>{});
```

- myForEach를 Array interface에 merge해준다.
- 이번엔 인자가 없다는 에러가 뜨게 된다.

```js
interface Array<T> {
    myForEach(): void;
}
```

- callback을 받으므로 callback fn을 인자로 넣는다.

```js
interface Array<T> {
    myForEach(callback: () => void): void;
}
```

- 그럼 이제 실제 테스트를 해보자.
- myForEach method는 매개변수로 콜백을 받지만, 콜백의 매개변수가 지정되지 않았다.
- 따라서 에러를 내뱉게 될 수밖에 없다.

```js
[1,2,3].myForEach(()=>{});
[1,2,3].myForEach((v,i,a) => {console.log(v, i, a)});   //error
[1,2,3].myForEach((v,i)=>console.log(v));               //error
[1,2,3].myForEach((v)=>3);                              //error
```

- interface type 정보를 다시 수정해주자.

```js
interface Array<T> {
    myForEach(callback: (v: number, i: number, a: number[]) => void): void;
}
```


- number type이 아닌 string type으로 배열으 만들어 foreach를 호출해보자.
- v를 number로 지정했기에 slice method가 없어 에러가 난다.
- if문의 경우, v는 무조건 number type이라 string type은 될 수 없어 never로 지정되어 slice가 없어 에러가 난다.

```js
['1','2','3'].myForEach((v) => {console.log(v.slice(0))}) //error

[true,2,'3'].myForEach((v) => {
    if (typeof v === 'string') {
        v.slice(0); //error
    } else {
        v.toFixed(); 
    }
})
```

- number로 한정하지 않고 generics type으로 바꾼다.
- 그럼 모두 error가 나지 않는다.

```js
interface Array<T> {
    myForEach(callback: (v: T, i: number, a: T[]) => void): void;
}

[1,2,3].myForEach(()=>{});
[1,2,3].myForEach((v,i,a) => {console.log(v, i, a)});   
[1,2,3].myForEach((v,i)=>console.log(v));               
[1,2,3].myForEach((v)=>3);   
['1','2','3'].myForEach((v) => {console.log(v.slice(0))}) //error
[true,2,'3'].myForEach((v) => {
    if (typeof v === 'string') {
        v.slice(0); //error
    } else {
        v.toFixed(); 
    }
})                           
```

- 실제 lib.es5.d.ts에서의 구현을 살펴보자.
- 아까와 거의 비슷한데, thisArg가 추가되어 있다.
- this 선언문을 사용할 때 값을 바꿀 수 있기 때문에 추가된 것이다.
- 이마저도 완벽하진 않다. 중요한 것은 100% type check를 보장하는 게 아니라, 쓸만한 게 type을 만드는 것이다.

```js
interface Array<T> {
    forEach(callbackFn: (value: T, index: number, array: T[]) => void, thisArg?: any): void;
}
```


- 우리가 만든 myForEach method는 ts를 type check에서 속일 수 있다.
- 하지만 실제로 구현한 함수는 없기 때문에 js에서는 발동하지 않는다.
- 즉 runtime에서는 무의미하다. ts를 속여 type error가 없더라도 늘 실행되진 않는다.
