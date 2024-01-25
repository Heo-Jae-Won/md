## <span style="color:#802548">_1.프로토타입_</span>
- Js의 상속은 extends가 아니다. Prototype chaining이다.
- prototype chaining은 [[Prototype]] 내부 슬롯의 참조를 따라 자신의 부모 역할을 하는 프로토타입의 프로퍼티를 순차검색한다.
- prototype의 property에 접근하려면 __proto__를 사용하지만 이는 하지 말아야 한다.
```javascript
function Circle(radius){
    this.radius = radius;
    this.getArea = function(){
        return Math.PI * this.radius ** 2;
    }
}

const circle1 = new Circle1(1);
const circle2 = new Circle2(2);
```

- 위와 같이하면 getArea가 모든 property마다 중복 선언되어 불필요하게 많은 메모리를 점유한다.
- 모두가 동일하게 가지는 속성이기 때문에 prototype에 선언해주자.

```javascript
function Circle(radius){
    this.radius = radius;
}

Circle.prototype.getArea = function(){
    return Math.PI * this.radius ** 2;
}

const circle1 = new Circle(1);
const circle2 = new Circle(2);
```
- 아니며 아래와 같이 IIFE를 사용해 만들 수도 있을 것이다.


```javascript
const Person = (function(){
    function Person(name){
        this.name = name;
    }

    Person.prototype.sayHello = function(){
        console.log(`Hi! My name is ${this.name}`);
    }

    return Person;
}());

const me = Person('Lee');
```

- 아래와 같이 직접 prototype에 추가하면 프로토타입 체인에 즉각 반영된다.
```javascript
function Person(name){
    this.name = name;
}

Person.prototype.sayHello = function(){
    console.log(`Hi! my name is ${this.name}`);
    console.log(`Hi! my name is ${name}`); //this.name으로 써야한다.
}

    const person = new Person("Lee");

person.sayHello = function(){
    console.log(`Hi! your name is you`);
}

person.hello(); // Hi! your name is you
```

- prototype에 추가할 때는 아래와 같이 추가할 수도 있다.
- constructor를 추가하지 않으면 prototype이 Person에서 Object로 바뀐다.
```javascript
const Person = (function(){
    function Person(name){
        this.name = name;
    }

    Person.prototype = {
        constructor: Person,
        sayHello(){
            console.log(`Hi! My name is ${this.name}`);
        }
    }

    return Person;
}());

const me = Person('Lee');
```

- 프로토타입을 바꾸면서 상속 관계를 직접 바꾸는 것은 매우 추천되지 않는다.
- 상속관계를 바꾸고 싶다면 class의 extends sugar를 활용하자.

- 만약 객체의 property를 열거하고 싶다면 for in문을 활용한다.

```javascript
const person = {
    name: 'Lee',
    address: 'Seoul'
}

console.log('toString' in person); // true. Object의 prototype에 있는 toString method
for(const key in person){
    console.log(key + ':' + person[key] + ","); //name: Lee, address:Seoul
}
```
- for in문은 모든 프로토타입의 프로퍼티를 순회하며 열거한다.
- 객체 고유의 property만 열거하려면 조건문이 추가된다.
- ES8에서는 더 간단하게 method로 표현할 수 있다.

```javascript
const person = {
    name: 'Lee',
    address: 'Seoul',
    __proto__: {age:20}
}

for(const key in person){
    if(!person.hasOwnProperty(key)){
        continue;
    }
    console.log(key + ":" + person[key] + ","); //name: Lee, address:Seoul
}

console.log(Object.keys(person)); //["name", "address"]
```

- 배열은 for in문 보다는 for of나 forEach를 사용하자.
- 숫자가 아닌 index조차도 열거하여 보여줄 위험이 있기 때문이다. 배열도 객체라서 이상한 index를 key처럼 넣을 수도 있다.
```javascript
const arr = [1,2,3];
arr.x = 10;

for(const i in arr){
    console.log(arr[i]); //1,2,3,10
}
for(let i = 0; i <arr.length; i++){
    console.log(arr[i])//1 2 3
}

for(const value of arr){
    console.log(value); // 1 2 3
}
arr.forEach(v => console.log(v)) // 1, 2, 3
```



## <span style="color:#802548">_2.빌트인 객체_</span>
- JS의 객체는 아래 3가지 분류로 분류한다.
- 표준 빌트인 객체는 전역객체의 프로퍼티로 제공된다.
- 호스트 객체는 JS 실행환경(브라우저, node)에 추가한 것으로 DOM, BOM, fetch, SVG, Web Storage, Web API 등이 있다.
- 나머지는 모두 사용자 정의 객체다.
- 빌트인 객체는 String, Number, Object, Boolean, Math, RegExp 등이 있다.

- 아래와 같이 원시값이 method를 활용하고 있다. 원래 이는 불가능하다.
- 원시값은 객체가 아니라 property를 가질 수 없다.
- 원시값은 마침표, 대괄호 표기법으로 접근할 경우 JS 엔진이 wrapper 객체로 감싸기 때문에 가능하다. 일종의 Java의 autoboxing이다.
- 다만 null과 undefined는 그렇지 않다.
```javascript
const str = 'hello';
//new String('hello');
console.log(str.length); //5
console.log(str.toUpperCase()); // HELLO
```

- 빌트인 객체의 encodeURI, decodeURI, encodeURIComponent, decodeURIComponent가 있다.
- 인코딩이란 URI의 문자들을 이스케이프 처리하는 것을 의미한다.
- 이스케이프 처리는 네트워크를 통해 정보를 공유할 때 어떤 시스템에서도 읽히게끔 아스키 문자셋으로 변환하는 것이다.
- encodeURI와 encodeURIComponent의 차이는 쿼리스트링 구분자로 사용되는 문자(=?&)를 인코딩하지 않냐, 하냐의 차이다.

## <span style="color:#802548">_3.this_</span>
- this binding이란 식별자와 값을 연결하는 과정이다. this라는 식별자와 this가 가리킬 객체가 확보한 메모리 공간의 주소를 binding하는 것이다.
- this binding은 일반함수, 메서드, 생성자함수, call, apply, bind라는 4가지 맥락에 따라 의미가 다르다.
- 일반함수의 경우 lexical scope에 관계없이 무조건 this가 window에 binding된다.
```javascript
var value = 1;

const obj = {
    value: 100,
    function foo(){
    console.log("foo's this: ", this); //method 호출. 객체 자기자신
    console.log("foo's this.value: ", this.value) // 100
    function bar(){
        console.log("bar's this: ", this); //window
        console.log("bar's this: ", this.value); //window, 1
    }
    bar(); //일반함수로 호출됐으면 어디서든 window. callback이라도..
}
}

obj.foo();
```

- callback함수도 일반함수의 형태라면 this가 window에 binding된다.

```javascript
var value = 1;
const obj = {
    value: 100,
    foo(){
        console.log("foo's this: ", this);
        setTimeout(function(){
            console.log("callback's this: ",this); // window
            console.log("callback's this.value: ", this.value) //1
        },100);
    }
}

obj.foo();
```

- method 호출처럼 객체 자기 자신을 binding하려면 아래와 같은 방법으로 가능하다.
- 객체 자기자신 this를 binding할 변수를 할당하여 사용한다.

```javascript
var value = 1;
const obj = {
    value: 100,
    foo(){
        const that = this;

        setTimeout(function(){
             console.log("callback's this: ",that); // obj
            console.log("callback's this.value: ", that.value) //100
        },100);
    }
}

obj.foo();
```

- bind함수를 사용한다.

```javascript
var value = 1; // var는 전역프로퍼티에 들어가게 됨
const obj = {
    value: 100,
    foo(){
        setTimeout(function(){
            console.log("callback's this.value: ", this.value) //100
        }.bind(this),100);
    }
}

obj.foo();
```

- 화살표 함수를 사용한다.
- 화살표함수를 쓰면 this가 상위 스코프에 binding된다.

```javascript
var value = 1; // var는 전역프로퍼티에 들어가게 됨
const obj = {
    value: 100,
    foo(){
        setTimeout(() => console.log(this.value), 100);
    }
}

obj.foo();
```

- method 호출의 경우, 일반 함수처럼 쓸 때 주의해야 한다.

```javascript
const person = {
    name: 'Lee',
    getName(){
        return this.name;
    }
}

console.log(person.getName()) // Lee


const anotherPerson = {
    name: 'Kim'
}

anotherPerson.getName = person.getName;

console.log(anotherPerson.getName()) // Kim

const getName = person.getName;
console.log(getName()) // '' window객체의 name을 참조하기 때문에 빈칸. 일반함수로 바꿔 호출했기 때문에 이렇게 된 것.
```

- call, bind, apply를 사용하면 내부 중첩 함수, 콜백 함수의 this가 불일치하는 문제를 해결할 수 있다.
- 아래 callback 함수의 this.name은 window.name이기에 빈 값이 된다.

```javascript
const person = {
    name: 'Lee',
    foo(fn callback){
        setTimeout(callback, 100);
    }
};

person.foo(function(){
    console.log(`Hi! my name is ${this.name}`); // Hi! my name is
})
```

- 그럴 때 아래와 같이 bind를 걸어서 person.foo()의 this와 callback fn의 this를 일치시킨다.

```javascript
const person = {
    name: 'Lee',
    foo(fn callback){
        setTimeout(callback.bind(this), 100);
    }
};

person.foo(function(){
    console.log(`Hi! my name is ${this.name}`); // Hi! my name is Lee
})
```

- 또는 아래와 같이 유사배열객체를 실제 배열이 갖는 method를 사용하게 만들 수도 있다.

```javascript
function convertArgsToArray(){
    console.log(arguments);

    const arr = Array.prototype.slice.call(arguments);
    console.log(arr); //[1,2,3]

    return arr;
}
```

- 인수를 넣어서 사용하고 싶다면 아래와 같다.

```javascript
function getThisBinding(){
    console.log(arguments);
    return this;
}

const thisArg = {a:1};

console.log(getThisBinding.apply(thisArg,[1,2,3])); 
```


## <span style="color:#802548">_3.closure_</span>
- 클로저는 중첩함수가, 외부함수보다 오래 살아남았을 때 + 상위함수의 변수를 참조할 때 closure라고 한다.
- 아래가 closure의 예시다.


```javascript
const x = 1;

function outer(){
    const x = 10;
    const inner = function(){
        console.log(x);
    }
    return inner;
}

const innerFunc = outer(); //10
innerFuc();
```

- 아래는 closure가 아니다.
- 상위 함수의 변수를 참조하지 않고 있다.
```javascript
function foo(){
    const x = 1;
    const y = 2;

    function bar(){
        const z = 3;
        debugger;
        console.log(z);
    }

    return bar;
}

const bar = foo();
bar();
```

- 아래는 closure가 아니다.
- 상위 함수 내에서 소멸하고 있다.

```javascript
function foo(){
    const x = 1;
    function bar(){
        debugger;

        console.log(x);
    }
    bar();
}

foo();
```

- 아래가 closure의 다른 예시다.

```javascript
function foo(){
    const x = 1;
    const y = 2;

    function bar(){
        debugger;
        console.log(x);
    }

    return bar;
}

const bar = foo();
bar();
```

- closure는 아래와 같은 경우에 자주 사용된다.

```javascript
let num = 0;
const increase = function(){
    return ++num;;
}

console.log(increase()); //1
console.log(increase()); //2
console.log(increase()); //3
```

- 위처럼 하면 num이 다른 함수에 의해 변경될 수 있는 가능성을 내포하게 된다.
- 따라서 아래처럼 바꾼다.


```javascript
const increase = (function(){
    let num = 0;

    return function(){
        return ++num;;
    }
}());
```

- 여기에 decrease 기능도 넣어보자.

```javascript
const counter = (function(){
    let num = 0;
    return {
        increase(){
            return ++num;
        },
        decrease(){
            return num > 0 ? --num : 0;
        }
    }
}());

console.log(counter.increase())  //1
console.log(counter.increase())  //2
console.log(counter.decrease())  //1
console.log(counter.decrease())  //0
```

- 이걸 종합하면 아래와 같이 생성자를 만드는 것도 가능하다.
- 다만 property가 없다.

```javascript
const Counter = (function(){
    let num = 0;
    function Counter(){
        //this.num = 0; property는 public이라 은닉이 불가능함.
    }

    Counter.prototype.increase = function(){
        return ++num;;
    }

    Counter.prototype.decrease = function(){
        return num > 0 ? --num : 0;
    }

    return Counter;
})
```

- 아래와 같이 생성자함수가 아닌 일반함수로도 closure를 활용할 수 있다.
- 다만 아래와 같이 하면 의도한 대로 되지 않는다.


```javascript
function makeCounter(aux){
    let counter = 0;
    return function(){
        counter = aux(counter);
        return counter;
    }
}

function increase(n){
    return ++n;
}

function decrease(n){
    return --n;
}

const increaser = makeCounter(increase);
console.log(increaser()); //1
console.log(increaser()); //2

const decreaser = makeCounter(decrease);
console.log(decreaser()); //-1
console.log(decreaser()); //-2
```

- increaser과 decreaser이 독립된 lexical 환경을 갖기 때문이다.
- 따라서 자유변수 counter를 공유하게 하려면 IIFE로 만들어야 한다.

```javascript
const counter = (function(){
    let counter = 0;
    return function(aux){
        counter = aux(counter);
        return counter;
    }
}());

function increase(n){
    return ++n;
}

function decrease(n){
    return --n;
}

console.log(counter(increase)) //1
console.log(counter(increase)) //2
console.log(counter(decrease)) //1'
console.log(counter(decrease)) //0
```


- closure는 아래와 같이 var로 사용하면 실수가 날 때 유용하다.
- 의도환 대로 012가 나와야하는데, 333이 나온다.
```javascript
var funcs = [];
for(var i = 0; i < 3; i++){
    funcs[i] = function(){
        return i;
    };
}

// funcs[j]는 모두 전역변수 i를 return 한다.
for(var j = 0; j < funcs.length;j++){
    console.log(funcs[j]()); // 3 3 3
}
```

- 여기서 closure를 활용하면 좋다.
- IIFE에 arguments를 pass하면 아래와 같이 된다.
```javascript
var funcs = [];
for(var i = 0; i < 3; i++){
    funcs[i] = (function(id){ //argument 받음.
        return function(){
            return id;
        }
    }(i)); //argument pass
}

for(var index = 0; index < 3; index++){
    funcs[index] = (function(a){
        return function(){
            return a;
        }
    }(index));
}

for(var j = 0; j < funcs.length; J++){
    console.log(funcs[j]()); //012
}
```

- closure를 이용하지 않고 let으로만 바꿔도 의도한대로 된다.
```javascript
var funcs = [];
for(let i = 0; i < 3; i++){
    funcs[i] = function(){
        return i;
    };
}

for(var j = 0; j < funcs.length;j++){
    console.log(funcs[j]()); // 0 1 2
}
```



## <span style="color:#802548">_4.class_</span>
- 기존 ES5의 문법은 아래와 같이 생성자 함수로 instance를 만들었다.
```javascript
var Person = (function(){
    function Person(name){
        this.name = name;
    }

    Person.prototype.sayHi = function(){
        console.log(`Hi! my name is ${this.name}`);
    }
    return Person;
})();

var me = new Person('Lee');
me.sayHi(); // Hi! my name is Lee
```

- ES6에 도입된 class로는 아래와 같이 만들 수 있다.
```javascript
class Person{
    constructor(firstName, lastName){ // 1개만 가능. 생략하면 묵시적 생성자로 간주
        this.firstName = firstName;
        this.lastName = lastName;
    }

    get fullName(){
        return `${this.firstName} ${this.lastName}`;
    }

    set fullName(){
        [this.firstName, this.lastName] = name.split('');
    }

    sayFirstName(){
        console.log(`Hi! my name is ${this.firstName}`);
    }

    static sayLastName(){
        console.log(`Hi! my name is ${this.lastName}`);
    }
}

const me = new Person('Ungmo', 'Lee');

me.fullName = 'Heegun Lee'; //할당 시 setter의 fullName
console.log(me.fullName) //조회 시 getter의 fullName Heegun Lee
```

- private field의 경우 아래와 같이 property에 직접 접근이 불가능하다.
```javascript
class Person{
    #name = '';

    constructor(name){
        this.#name = name;
    }
}

const me = new Person('Lee');
console.log(me.#name); //private field라서 외부참조불가.
```

- getter를 사용하여 private field값을 가져올 수 있다.
```javascript
class Person{
    #name = '';
    
    constructor(name){
        this.#name = name;
    }

    get name(){
        return this.#name.trim();
    }
}

const me = new Person('Lee');
console.log(me.name) // Lee. private field지만 getter를 통해 접근가능.
```

- Java의 extends와 거의 동일하다.
- constructor()가 subclass에 없으면 super의 것을 받아오는 것으로 하여 transpile 시에 집어넣는다.
```javascript
class Animal{
    constructor(age, weight){
        this.age = age;
        this.weight = weight;
    }

    eat(){
        return 'eat';
    }

    move(){
        return 'move';
    }
}

class Bird extends Animal{
    //constructor(... args){super(...args)} 
    fly(){
        return 'fly';
    }
}

class Bird1 extends Animal{
    constructor(age, weight,length){
        super(age, weight) //constructor가 있으면 반드시 호출
        this.length = length;
        } 
    fly(){
        return 'fly';
    }

    eat(){
        return `${super.eat()}`;
    }

    move(){
        return super.move();
    }
}

const bird = new Bird(1,5);
const bird = new Bird1(1,5,20);
```

- 몇가지 다른 점은 동적인 상속 대상 결정이 가능하다는 점 + class만이 아니라 생성자 함수를 상속받을 수도 있다는 점이다.

```javascript
function Base1(a){
    this.a = a;
}
class Base2{}

let condition = true;

class Derived extends ( condition ? Base1 : Base2){}
const derived = new Derived(1);
console.log(derived) //{"a": 1}
```

- 다만 염두에 두어야 할 것은, field에 함수를 할당한다면 ES6 축약 method를 사용해야한다는 점이다.
- 일반 표현식으로 쓰면 instance method가 되어버린다.
```javascript
class Person{
    name = 'Lee',
    //sayHi = () => console.log(`Hi ${this.name}`);
    sayHi() {
        console.log(`Hi ${this.name}`);
    }
}
```

## <span style="color:#802548">_5.ES6_</span>
- 화살표함수도 도입되었다. 여러모로 화살표함수를 쓰는 게 좋다.

```javascript
const sum = (a,b)=>{
    const result = a + b;
    return result;
};

const create = (id,content) =>({id, content}); //객체 literal 반환 시 소괄호로 감싼다.

const person = ((name) =>({
    sayHi(){
        return `Hi? My name is ${name}.`;};
    }
}))('Lee');
```

- ES6에서는 method에 익명함수표현식을 할당하는 습관을 버려야 한다.
- 그 방식으로는 더이상 super를 쓸수가 없기 때문이다.
- 화살표함수 일때도 이 문제는 동일하며, this 떄문이라도 method에서는 일반함수(화살표든, 표현식이든, 선언문이든)가 아닌 ES6 축약 method를 써야한다.
```javascript
const base = {
    name : 'Lee',
    sayHi(){
        return `Hi! ${this.name}`;
    }
}

const derived = {
    __proto__:base,
    sayHi: function(){
        return `${super.sayHi()}. how are you doing?`; //super keyword unexpected here
    }
}

const derived = {
    __proto__:base,
    sayHi: function(){
        return `Hi ${this.name}.`; // window의 name property를 찾음. 있으나 값이 없음. 따라서 Hi만 출력됨.
    }
}

const derived = {
    __proto__: base,
    sayHi(){
        return `${super.sayHi()}. how are you doing?`; //ok
    }
}
```

## <span style="color:#802548">_6.Array_</span>
- 아래는 array의 일반함수다.
```javascript
const arr = Array.of(1,2,2,3);
arr.isArray(); //true;
arr.indexOf(2); //1
arr.includes(2) //true. indexOf보다 더 나은 방식이다. ES7에서부터 활용가능

arr.push(3); //[1,2,2,3,3];
arr[arr.length] = 3; //이게 push보다 빠르다. 직접 원본 배열을 변경한다.
const newArr = [...arr,3]; //새로운 배열을 반환한다.
arr.concat(3); // 새로운 배열을 반환. push와 동일

arr.pop(); //원본변경한다. 맨마지막 요소 반환하고 삭제
arr.unshift(); //원본변경한다. 맨앞 요소 추가하고 length 반환
arr.shift(); //원본 배열에서 첫 번째 요소를 제거하고 제거한 요소를 반환한다. 
const newArr = [3, ...arr];

arr.splice(1,2,20,30); //원본변경한다. index 1부터 2개 요소를 지우고 그 자리에 20,30을 넣는다.
arr.splice(1,0,20); //index 1부터 지우지 않고 그 자리에 20을 넣는다.
arr.splice(i,1); //index  i에서 1개 요소를 지운다.
arr.splice(1); //index 1부터 모든 요소를 지운다.

arr.slice(0,1); //arr[0]부터 arr[1]이전까지 복사하여 반환한다. 원본 변경 X

arr.join(); //'1,2,2,3' 문자열로 변환
arr.join(""); //1234
```

- 아래는 array의 고차함수다.
- 고차함수는 callback fn을 화살표함수로 써주는 게 좋다.
```javascript
const numbers = [1,2,3];
const pows = [];

numbers.forEach(item => pows.push(item ** 2));
console.log(pows);

class Number{
    numberArray = [];

    multiply(arr){
        arr.forEach(function(item){
            this.numberArray.push(item * item);
        }, this);
    }
}

class Numbers{
    numberArray = [];
    multiply(arr){
        arr.forEach(item => this.numberArray.push(item * item));
    }
}

const numbers = new Numbers();
numbers.multiply([1,2,3]);
console.log(numbers.numberArray);
```

- 고차함수의 경우, index만 있고 값이 없는 경우에는 무시하고 순회하지 않는다.
```javascript
const arr = [1,,3];
for(let i = 0; i <arr.length; i++){
    console.log(arr[i]); //1, undefined, 3
}

arr.forEach(v => console.log(v)); // 1,3
```

- forEach문이나 map은 break;나 continue;가 불가능하다. 전부 순회해야 한다.
```javascript
[1,2,3].forEach(item => {
    console.log(item);
    if(item > 1)
        break; // 
})

const numbers = [1,4,9];

const roots = numbers.map(item => Math.sqrt(item)); // [1,2,3]
```

- filter는 콜백함수의 반환값이 true인 요소로만 구성된 새로운 배열을 반환한다.

```javascript
const numbers = [1,2,3,4,,5];
const odds = numbers.filter(item => item % 2);
console.log(odds) //[1,3,5]
```


- reduce는 다양한 예제를 보고 감을 익히는 것이 좋다.
- 사실 map, filter 등은 모두 reduce로 구현가능하지만, 굳이 그럴필요가 없다.
```javascript
const values = [1,2,3,4,5,6];

const average = values.reduce((acc,cur,i {length}) => { 
    return i === length -1 ? (acc + cur) / length : acc + cur;
})

const values = [1,2,3,4,5,6];

const max = values.reduce((acc,cur) => (acc > cur ? acc : cur), 0);
console.log(max);

const fruits = ['banana','apple','orange','orange','apple'];
const count = fruits.reduce((acc,cur)=>{
    acc[cur] = (acc[cur] || 0) + 1;
    return acc;
},{})

const values = [1,[2,3],4,[5,6]];

const flatten = values.reduce((acc,cur) => acc.concat(cur), []);
//[1] -> [1,2,3] -> [1,2,3,4] ->[1,2,3,4,5,6]

const values = [1,2,1,3,5,4,5,3,4,4];
const result = values.reduce( //중복제거
    (unique, val, i, _values) =>
    _values.indexOf(val) === i ? [...unique, val] : unique,
    []
)
const result = values.filter((val,i,_values) => _values.indexOf(val) === i);
const result = [...new Set(values)];


console.log(result) //[1,2,3,5,4]
```

- reduce에서 가장 중요한 것은 초기값을 전달하는 것이다.
- 적절한 초기값을 주지 않으면 undefined나 NaN에 도달하기 쉽다.
```javascript
const products = [
    {id: 1, price: 100},
    {id: 2, price: 200},
    {id: 3, price: 300}
]

const priceSum = products.reduce((acc,cur) => acc.price + cur.price); //초기값을 안줘서 {}로 초기값이 설정된다.
console.log(priceSum) //NaN. 2번째 순회시에는 acc가 {}가 아니라 300이다. 따라서 acc.price는 undefined이며, undefined를 계속 더하면 NaN이다.

const priceSum = products.reduce((acc,cur) => acc + cur.price,0);
console.log(priceSum) //600
```

- 그 외 some, every, find, findIndex, flatMap 등이 있다.
- 한번이라도 조건을 만족하면 true, 한번도 안되면 false를 return한다. 하나만 있으면 된다.
- every는 some과 반대다. 하나만 틀려도 false다.
- find는 요소를 반환하며, findIndex는 요소의 index를 반환한다.
- flatMap은 1단계 평탄화밖에 안되기 때문에 단계를 지정하려면 map과 flat을 따로 사용해야한다.
```javascript
[5,10,15].some(item => item > 10); //true
[5,10,15].every(item => item > 10); //false


const users = [
    {id: 1, name : 'Lee'},
    {id: 2, name : 'Kim'},
    {id: 3, name : 'Choi'},
    {id: 4, name : 'Park'},
]

user.find(user => user.id === 2); //{id:2, name: 'Kim'}
user.findIndex(user => user.id === 2); //1

const arr = ['hello', 'world'];
arr.map(x => x.split(''))
// [
//     ["h","e","l","l","o"],
//     ["w","o","r","l","d"]
// ]
arr.map(x => x.split('').flat()); //["h","e","l","l","o","w","o","r","l","d"]
arr.flatMap(x => x.split(''));    //["h","e","l","l","o","w","o","r","l","d"]
arr.map((str, index) => [index, [str, str.length]]) 
// [
//     [0,["hello",5]],
//     [1,["world",5]]
// ]
arr.map((str, index) => [index, [str, str.length]]).flat(1);
// [
//    0,["hello",5],
//    1,["world",5]
// ]
arr.map((str, index) => [index, [str, str.length]]).flat(2); //[0,"hello",5,1,"world",5]
```


## <span style="color:#802548">_7.그 외 prototype_</span>
- 소수를 비교할 때는 부동소수점이기 산술 연산이라 제대로 비교가 안된다.
- 그래서 아래와 같이 비교해준다.

```javascript
function isEqual(a,b){
    return Math.abs(a-b) < Number.EPSILON;
}

isEqual(0.1 +0.2, 0.3); //true
```

- Number의 method는 아래와 같다.

```javascript
Number.isFinite();
Number.isInteger();
Number.isNaN();
Number.toFixed(3); //인수자리수까지 반올림. 1234.5789 -> 1234.579
```

- Math는 Number와 달리 수학 상수와 함수를 위한 property와 method를 제공한다.
```javascript
Math.PI;
Math.abs(); //절대값
Math.round();
Math.ceil(); //올림하여 정수 반환
Math.floor(); //내림하여 정수반환
Math.sqrt(); //제곱근 반환
Math.random(); //랜덤 정수 반환 0<= b < 1
const random = Math.floor(Math.random() * 10 + 1); // 1 < = a < 11. 1부터 10까지 정수
Math.pow(2,8)  //제곱. 2 ** 8(ES6)
Math.max(); //최대값
Math.max.apply(null,[1,2,3]) //3
Math.max(...[1,2,3]);
Math.min(); //최소값
```

- Date는 날짜 객체다

```javascript
const now = new Date();
const specificDate = new Date('2020/07/24/10:00:00');
const specificDate = new Date('2020-07-24T10:00:00'); 
const specificDate = new Date('2020-07-24T10:00:000Z');
now.setFullYear(2000);
now.setFullYear(2000,0,1); //2000년 1월 1일
now.setMonth(0); //1월
now.setDate(11,1); //12월 1일
now.getDate(); //25
now.getDay(); //0부터 6까지. 0 = 일요일 ... 6= 토요일
now.setHours(now.getHours() + 12); //12시간 뒤
now.setMinutes(now.getMinutes() + 20); //20분 뒤
now.setMinutes(5,10,999); //HH:05:10:999 밀리세컨드까지 지정
```

```javascript
(function printNow(){
    const today = new Date();

    const dayNames = [
        '(일요일)',
        '(월요일)',
        '(화요일)',
        '(수요일)',
        '(목요일)',
        '(금요일)',
        '(토요일)'
    ]

    const day = dayNames[today.getDay()];
    const year = today.getFullYear();
    const month = today.getMonth() + 1;
    const date = today.getDate();
    let hour = today.getHours();
    let minute = today.getMinutes();
    let second = today.getSeconds();
    const ampm = hour >= 12 ? 'PM' : 'AM';

    //12시간제
    hour %= 12;
    hour = hour || 12; //hour가 0이면 12를 재할당

    minute = minute < 10 ? '0' + minute : minute;
    second = second < 10 ? '0' + second : second;

    const now = `${year}년 ${month}월 ${date}일 ${day} ${hour}:${second}:${ampm}`;

    console.log(now);

    setTimeout(printNow,1000);
})();
```

- 문자열의 method는 아래와 같다.
```javascript
const str = 'Hello World';

str.indexOf('l') //2
str.indexOf('or') //7. 첫번째 o의 index가 반환된다.
if(str.indexOf('Hello') !== -1){

}
if(str.includes('Hello')){

}
if(str.startsWith('W', 6)){ // index 6부터 W로 시작하는지

}
if(str.endsWith('d')){

}
if(str.endsWith('lo', 5)){ // 문자열 처음부터 5자리까지 lo로 끝나는지

}

const str = 'jaewon@jaewon.com';
str.substring(4,1); //str.substring(1,4)로 작동
str.substring(-2); //str.substring(0)으로 작동
str.substring(0,str.indexOf('@') + 1); //email id
str.substring(str.indexOf('@') + 1, str.length); //email host

const str = 'hello world';
str.substring(0,5); //hello
str.slice(0,5); //hello
str.slice(-5); //substring()거의 비슷한데, slice는 음수로 할 시 거꾸로 끝에서부터 가져올 수 있음. 뒤에서 5자리를 가져옴.

const str = "How are you doing?";
str.split(" ");  //['How','are','you','doing'] 한칸 띄어쓰기만 기준으로 나눔.
str.split(/\s/); //['How', 'are', 'you', 'doing'] \t, \n 등 여러 공백문자를 기준으로 반환
str.split(''); //["H","o","w"...] 전체 문자를 각 element로 하는 배열을 반환
spt.split(); //["How are you doing?"]
```

- replace는 입력값을 바꿀 떄 특히 많이 쓰여서 별도로 빼서 쓴다.

```javascript
function camelToSnake(camelCase){
    return camelCase.replace(/.[A-Z]/g, match =>{
        console.log(match);
        return match[0] + '_' + match[1].toLowerCase();
    })
}

const camelCase = 'helloWorld';
camelToSnake(camelCase) //hello_world


function snakeToCamel(snakeCase){
    return snakeCase.replace(/_[a-z]/g, match => {
        console.log(match);
        return match[i].toUpperCase();
    })
}


const snakeCase = 'hello_world';
snakeToCamel(snakeCase);
```

- Symbol은 거의 쓰이지 않는데, js에서 enum을 구현할 때 쓴다고 한다.

```javascript
const Direction = Object.freeze({
    UP: Symbol('up'),
    DOWN: Symbol('donw'),
    LEFT: Symbol('left'),
    RIGHT: Symbol('right')
})

const myDirection = Direction.UP;

if(myDirection === Direction.UP){

}
```

- 객체의 프로퍼티 키가 절대 중복되지 않게 하는 데도 쓰인다.

```javascript
const obj = {
    [Symbol.for('mySymbol')] : 1
}

obj[Symbol.for('mySymbol')]; //1
```

## <span style="color:#802548">_8.이터러블_</span>
- 배열, 문자열, Map, Set 등이 iterable이다.
- 이터러블은 for...of문으로 순회가 가능하며, 스프레트 문법과 배열 디스트럭쳐링 할당의 대상으로 사용할 수 있다.

```javascript
const array = [1,2,3];

for(const item of array){
    console.log(item);
}
console.log([...array]);

const [a, ...rest] = array;
console.log(a, rest); //1,[2,3]
```
- 이터레이터는 이터러블과 비슷하지만, for of문을 사용할 수 없고, 배열 디스트럭쳐링도 불가능하다.
- 단 Symbol.iterator를 활용해 next를 계속 불러올 수 있다. Java의 iterator와 비슷하다.
```javascript
const array = [1,2,3];
const iterator = array[Symbol.iterator]();

console.log(iterator.next()); // { value: 1, done: false}
console.log(iterator.next()); // { value: 2, done: false}
console.log(iterator.next()); // { value: 3, done: false}
console.log(iterator.next()); // { value: undefined, done: true}
```

- for of문은 내부구현이 아래와 같다.
```javascript
const iterable = [1,2,3];
const iterator = iterable[Symbol.iterator]();
for(;;){
    const res = iterator.next();
    if(res.done){
        break;
    }

    const item = res.value;
    console.log(item);
}
```

- iterable 객체는 아래와 같이 우리가 직접 만들어 줄 수도 있다. value와 done을 property로 갖게 하면 된다.
```javascript
const iterable = { //iterator 반환
    [Symbol.iterator](){
        let cur = 1;
        return {
            next(){
                return {value : cur ++, done: cur > max + 1};
            }
        }
    }
}

for (const num of iterable){
    console.log(num); // 1 2 3 4 5
}
```

- 사용자 정의 이터러블로 피보나치 수열을 만든 것이다.
```javascript
const fibonacci = {
    [Symbol.iterator](){
        let [pre,cur] = [0,1];
        const max =10;

        return {
            next() {
                [pre, cur] = [cur, pre + cur];
                return {
                    value: cur, done: cur >= max
                }
            }
        }
    }
}

for(const num of fibonacci){
    console.log(num); // 1 2 3 5 8
}

const arr = [...fibonacci];
console.log(arr); //[1,2,3,5,8]

const [first, second, ...rest] = fibonacci;
console.log(first, second, rest); // 1 2 [3, 5, 8]
```

- 이터러블을 생성하는 함수로 바꿔서 만든다면 재활용성이 좋아진다.

```javascript
const fibonacciFunc = function (max) {
    let [pre,cur] = [0,1];
    return {
        [Symbol.iterator]() {
            return {
                next() {
                    [pre,cur] = [cur, pre + cur];
                    return {
                        value: cur,
                        done: cur >= max
                    }
                }
            }
        }
    }
}

for(const num of fibonacciFunc(10)){
    console.log(num);
}
```





