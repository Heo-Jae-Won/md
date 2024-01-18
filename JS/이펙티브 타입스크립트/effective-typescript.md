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






