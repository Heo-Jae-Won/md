## <span style="color:#802548">_1.JS의 변수_</span>
- 변수의 키워드를 선언하면 변수의 메모리 공간이 확보된다.
- 선언만 하는 경우에는 undefined로 초기화된다.

- null을 할당해줘야 GC가 수거해간다. undefined로 할당하지 않도록 하자.

```javascript
let str;    //선언. var str = undefined;와 동일
str = 'abc' //할당.

let foo; //GC 수행 X
foo = null; //GC 수행
```

- var, function 등 몇개의 keyword는 hoisting이 된다.
- hoisting이라는 건 코드가 실행 전에 코드 최상위로 끌어올려지는 것을 의미한다.

- 변수를 할당할 때는 생성자, 리터럴이 있다.
```javascript
const str = new String("B"); //생성자 방식
const str = "B";            //literal 방식
const str = 'C';             //literal 방식
const str = `C`;             //literal 방식

const num = new Number(20); //생성자 방식
const num = 20;             //literal 방식
const num = 20.01;          //literal 방식

const obj = new Object();   //생성자 방식
const obj = {};             //literal 방식

const arr = new Array();    //생성자 방식
const arr = [];             //literal 방식

const regex = new RegExp("[A-Z]", "gi");  //생성자방식
const regex = /[A-Z]+/gi;                 //literal 방식

function fn(){

}

var bool = true;
```

- typescript에서 사용하는 interface와 같은 것들도 맨위로 끌어올려지는 것처럼 보인다.

- 하지만 typescript는 runtime이 아니라 compile 시에 이뤄지는 것으로, hoisting과는 무관하다. hoisting은 runtime에 일어나는 것이다.
- 다만 코드가 평가되기 전에 type을 참조하기에, hoisting처럼 어디에 선언해놔도 최상단에 선언한 거나 다름없다.


```javascript
function changeScheduleRequest(data: ScheduleRequest) {
      scheduleRequest.value.scheduleDepartStation = data.scheduleDepartStation;
      scheduleRequest.value.scheduleArriveStation = data.scheduleArriveStation;
      scheduleRequest.value.scheduleDate = data.scheduleDate;
      scheduleRequest.value.scheduleTime = data.scheduleTime;
}

interface ScheduleRequest {
  scheduleNo?: number;
  scheduleDepartStation: string;
  scheduleArriveStation: string;
  scheduleDate: string;
  scheduleTime: string;
}
```

- 하지만 그러한 방식보다는 interface를 따로 js파일로 빼는 편이 더 관리하기 쉽다.
```javascript
export class ScheduleRequest {}
export interface ScheduleRequest {
  scheduleNo?: number;
  scheduleDepartStation: string;
  scheduleArriveStation: string;
  scheduleDate: string;
  scheduleTime: string;
  [key: string]: string | number | undefined;
}

import { ScheduleRequest, ScheduleResponse } from '../model/schedule.model';
.
.
```
- 변수를 선언하고 값을 할당하면 그게 곧 문장이다.
- 아래 keyword, identifier, literal, semicolon 모두 문장의 구성요소다.
- 문장의 구성요소들이 나뉠 수 없는 순간부터 token으로 취급된다.
- 세미콜론은 statement가 closed되었다는 의미다.

- token은 순서를 갖는데, 각 token이 예상치 못한 순서에 오면 unexpected token error가 난다.

```javascript
const// keyword 선언
const score //keyword, identifier
const score = 50 //keyword, identifier, literal
const score = 50;//keyword, identifier, literal, semicolon

const score = {
    math : condition1 ? 10 : undefined; //unexpected token. 아직 문장이 끝나지 않았기에 여기에 ;를 붙이면 안 된다.
}
```

- 값과 문을 보았다면 남은 건 식이다.

- 식이란 결국 모두 값으로 귀결될 수 있는 statement다.
- 보통은 연산자에서 많이 보인다.
- 값을 넣어야 하는 곳에는 식이 들어갈 수는 있지만, 문이 들어갈 수는 없다.
```javascript
x = x + 3;
const var = (a ===3) ? true : false;
const score = {
    math : condition1 ? 10 : undefined
}
const score = {
    math : if(condition1){ //unexpected token if
        result = 10;      
    }
}
```

## <span style="color:#802548">_3. 원시형_</span>
- Number라는 숫자형이다. 또한 2진수, 8진수 등을 써도 모두 10진수로 해석된다.

```javascript
const binary = 0b01000001;
const octal = 0o101;
const hex =0x41;

console.log(binary) //65
console.log(octal) //65
console.log(hex) //65
```

- 숫자는 전부 실수로 처리된다. 따라서 2와 2.0은 동일하다.

```javascript
console.log(1 === 1.0) //true
console.log(2.0 === 4/2); //true
```

- 숫자인지 파악하는 method가 아니라 NaN인지 파악하는 method가 있다.
- 그래서 따로 아래처럼 method로 만드는 게 좋다. 안그러면 혼동이 일어나기 쉽다.
```javascript
function isNumber(variable) {
  return (Number.isNaN(variable) === false && typeof variable === 'number')
}
```

- 문자열은 개행이나 변수를 받을 때는 ``, 그냥 넣을 때는 '', ""를 쓴다.
```javascript
const str = 'hello\nworld';
const template = `<ul>
                    <li><a href="#">Home</a></li>
                  </ul>`;
const url = `/api/naver/${url}`;
```

- raw type의 value는 immutable하다.
- 그럼 아래 script는 어떻게 작동하는 것인가?
```javascript
var str = "a";
str = "c";
```
- 아예 새로운 메모리 주소가 만들어지고, a의 주소 참조값이 그 새로운 메모리 주소로 만들어지며, 거기에 c가 들어간다.
- "a"를 담던 메모리 주소가 이제 참조가 사라졌으니 곧 GC가 수거해갈 것이다.


- 아래도 비슷하다. copy와 number1을 할당하였으나, 원시형이기에 새로운 메모리 주소에 값이 할당된다.
- 따라서 이전의 원시형을 담은 변수인 number1을 변경해도 copy의 값이 변경되지 않는다.
```javascript
var number1 = 10;
var copy = number1;

number1 = 50;

console.log(copy) // 10
```

- 반면에 객체로 만들었다면 변할 수도 있다.

- 객체 property key에 원시값 value를 담은 뒤 이를 변경하면, 변경된 채로 나오는 것을 알 수 있다.
- 기존 객체를 할당하면 새로운 메모리 주소를 갖지 않고 기존 메모리 주소를 공유한다.
```javascript
let match = {
    score : 80
}

let copy = match;

match.score = 100;

console.log(copy.score) //100
```

- 그게 싫다면 새로운 객체를 만들어내는 복사가 필요하다.
- 아래와 같이 Object.assign을 활용할 수도 있다.
```javascript
let match = {
    score: 80
}

let copy = Object.assign({},match);

match.score = 100;
console.log(copy.score) // 80
```

## <span style="color:#802548">_4.조건문_</span>
- 조건문에는 if, switch가 있다.
- if가 if else만 있다면 삼항연산자로 바꾸는 것도 좋은 선택이다.

```javascript
let str;
if(condition) {
  str = "true";
} else {
  str = "false";
}

let str = condition ? 'true' : 'false';
```

- if가 너무 복잡하다면 switch case로 바꾸는 것도 좋은 선택이다.
- switch문은 변수가 문자, 숫자일 때만 사용가능하다는 단점이 있긴 하다.
```javascript
let userType = abc;
if(userType === "관리자") {
    console.log("관리자");
} else if(userType === "강사") {
    console.log("강사");
} else if(userType === "VIP수강생") {
    console.log("VIP수강생");
} else if(userType === "일반수강생") {
    console.log("일반수강생");
} else {
  
}

switch(userType) {
  case "관리자":
     console.log("관리자");
    break;
  case "강사":
     console.log("강사");
    break;
  case "VIP수강생":
     console.log("VIP수강생");
    break;
  case "일반수강생":
     console.log("일반수강생");
    break;
  default: //default는 끝이므로 break;를 굳이 걸필요가 없다.
}
```
- 만약 조건이 같은 경우면 break;문을 오히려 생략하는 게 좋다.
```javascript
let userType = abc;
if(userType === "관리자") {
  console.log("관리자");
} else if(userType === "강사") {
  console.log("관리자");
} else if(userType === "VIP수강생") {
  console.log("관리자");
} else if(userType === "일반수강생") {
  console.log("일반수강생");
} else {
  
}

switch(userType) { //일반수강생 전까지 모두 같은 로직이라면 break;를 걸지 않는다.
  case "관리자":
  case "강사":
  case "VIP수강생":
    console.log("관리자");
    break;
  case "일반수강생":
    console.log("일반수강생");
    break;
  default:
}
```

- 계속 조건이 추가될 거 같다면 if, switch문을 벗어나서 object로 처리할 수도 있다.
- 상수를 하나 만들고 이 상수로 계속 처리한다. 상수를 관리하는 js파일을 따로 만들자.

- default 처리는 널병합 연산자가 해준다.
```javascript
const USER_TYPE = {
    ADMIN: '관리자',
    INSTRUCTOR: '강사',
    STUDENT: '수강생',
    .
    .
    .
    .
}

import USER_TYPE from './constants./...';
function getUserType(type) { // 함수는 고정. USER_TYPE만 계속 바꿈.
	return USER_TYPE[type] ?? USER_TYPE.UN
}
```

## <span style="color:#802548">_5. object_</span>
- JS의 객체는 아래 3가지로 분류한다.

```
표준 빌트인 객체 + 호스트 객체 + 사용자 정의 객체
표준 빌트인 객체는 String, Number, Object, Boolean, Math, RegExp, 전역객체의 프로퍼티 등이다.
호스트 객체는 JS 실행환경(브라우저, node)에 추가한 것으로 DOM, BOM, fetch, SVG, Web Storage, Web API 등이 있다.
나머지는 모두 사용자 정의 객체다.
```


- 그런데 원시형은 객체가 아니라 property를 가질 수 없다. 그럼에도 아래를 보면 method가 있는 것처럼 활용하고 있다.
- 원시형은 마침표, 대괄호 표기법으로 접근할 경우 JS 엔진이 wrapper 객체로 감싸기 때문에 가능하다. 일종의 autoBoxing이다.
```javascript
const str = 'hello';
//new String('hello');
console.log(str.length); //5
console.log(str.toUpperCase()); // HELLO
```

- 모든 객체는 [[Prototype]]이라는 내부 슬롯을 갖는다. 
- 내부 슬롯은 원래는 개발자가 접근할 수 없으나, [[Prototype]]만 간접 접근이 가능하다. [[Prototype]]은 __proto__를 통해 일부 접근가능하다.

```javascript
const object = {};
object.[[Prototype]];
object.__proto__ 
```

- js에서 프로퍼티를 생성할 때는 4개 값이 자동 정의된다.
- value, writable, Enumerable, Configurable이다.
- value는 값, writable은 변경가능여부, enumerable은 열거가능여부, configurable은 재정의 가능 여부다.
```javascript
const person = {
  name: 'Lee'
}

console.log(Object.getOwnPropertyDescriptor(person, 'name'));
//{value: 'Lee', writable: true, enumerable: true, configurable: true}
```

- 객체의 상태를 변경하는 것을 막기 위해서는 아래와 같이 활용할 수 있다.
- 다만 그럴 일은 많지 않다. vue의 readonly 속성을 지닌 props정도다.
- 가장 강력한 수단은 freeze다.

```javascript
const person = {name: 'Lee'}
Object.freeze(person);

person.age = 10;
console.log(person); //{name: 'Lee'}

delete person.name; 
console.log(person); //{name: 'Lee'}

person.name = 'Kim';
console.log(person); //{name: 'Lee'}
```

- 아래와 같이 freeze하여 enum처럼 쓸 수 있다.
```javascript
const CAR_NAME_LEN = Object.freeze({
	MIN: 1,
	MAX: 5,
});
```
  
- 다만 property 안의 property까지는 freeze가 안된다.

```javascript
const STATUS = Object.freeze({
	PENDING: 'PENDING',
	SUCCESS:'SUCCESS',
	FAIL:'FAIL',
	OPTIONS:{
		GREEN: 'GREEN',
		RED:'RED'
	}
})

STATUS.OPTIONS.GREEN = 'G';
console.log(STATUS.OPTIONS.GREEN); //G
```

- 그 때는 deepFreeze 함수를 구현하든, lodash에서 deepFreeze를 가져와 쓰든 해줘야 한다.
- 아래는 인터넷에서 가져온 deepFreeze 구현의 예시다.
```javascript
function deepFreeze(target) {
  if (target === null || typeof target !== 'object') {
    return;
  }
  Object.keys(target).forEach((key) => {
    Object.defineProperty(target, key, {
      value: target[key],
      writable: false,
      enumerable: true,
    });
    deepFreeze(target[key]);
  });

  Object.seal(target);
}

STATUS.OPTIONS.GREEN = 'G';
console.log(STATUS.OPTIONS.GREEN); //GREEN
```

- 객체의 property를 가져오려고 할 때, null이면 오류가 난다.
- 따라서 객체가 null을 담고 있는지 평가하는 작업이 필요하다.
```javascript
var elem = null;
value = elem.value.data.subject.even; // TypeError: Cannot read property 'value' of null
```

- 그 중에는 다양한 방법이 있다. 하지만 위와 같이 nested object라면?
- 그럴 때 optional chaining을 활용하면 좋다.
- optional chaining 활용하지 않는다면 if문을 중첩으로 만들거나, &&를 많이 늘려야 한다.
```javascript
if(elem) {
    if(elem.value) {
        if(elem.value.data) {
            if(elem.value.data.subject) {
                value = elem.value.data.subject.even;
            }
        }
    }
}

if(elem && elem.value && elem.value.data && elem.value.data.subject) {
    var value = elem.value.data.subject.even; 
}

var value =elem?.value?.data?.subject?.even // elem에 value라는 key가 없을 때 반환 값은 undefined로 고정
```

## <span style="color:#802548">_6. 함수_</span>
- 함수를 불러올 때는 식별자 이름으로 불러올 수 있다. 함수명이 아니다.
```javascript
var add = function add1(x,y) { //add1은 함수 내부에서만 사용 가능하다.
  return x + y;
}

add(5,6); // add1로 불러도 함수가 호출되지 않는다.
```

- 다만 함수의 인자를 넣을 때는 입력 값에 유의해야 한다.
- parameter를 잘못 집어넣으면 undefined나 NaN, null이 나올 수 있다.
```javascript
var add = function(x,y) {
    return x + y
};
add(); //NaN
add(''); // 'undefined'
add(2); //NaN


var add = function(x = 1 ,y = 2) {
  return x + y;
}; 
add(b,c); //b와 c가 null이라면 0 return. 기본값 설정이 100% 안전한 게 아님.
```

- 그래서 아래와 같이 검증이 필요하다.
- 숫자값인지 type 검증을 할 수도 있고, regex로 검증을 할 수도 있다.
```javascript
var add = function(x,y) {
  if(typeof x !== 'number' || typeof y !=='number') {
    throw new TypeError("인수는 모두 숫자값이어야 합니다.");
  }
}

var add function(event){
  const regex = /[0-9]/gi;
  if(!regex.test(event.target.value)) {
      throw new TypeError("인수는 모두 숫자값이어야 합니다.");
  }

    return x + y;
}

var add = function(event){
  const regex = /[0-9]/gi;
  if(!regex.test(event.target.value) ){
      event.target.value = event.target.value.replace(regex,'');
  }
}
```

- typescript를 쓰면 함수가 받을 수 있는 parameter의 type을 아예 지정해버릴수 있다.
```javascript
var add = function(x : number, y: number) {
  return x + y;
}
```

- 표현식 중에는 화살표 함수도 있다.
```javascript
const sum = (a,b) => {
    const result = a + b;
    return result;
};

const create = (id,content) =>({id, content}); //객체 literal 반환 시 소괄호로 감싼다.
//함수의 body인 {}와 헷깔리게 하지 않기 위해서다.

const person = ((name) =>({
    sayHi(){
        return `Hi? My name is ${name}.`;
    }
}))('Lee');
```

- 함수를 객체의 property에 할당하면 method라고 부른다.
- ES6에서는 method에 익명함수표현식을 할당하는 습관을 버려야 한다.

- this가 window로 binding되며, super를 사용할 수가 없다.
```javascript
const parent = {
    name : 'Lee',
    sayHi(){
        return `Hi! ${this.name}`;
    }
}

const child = {
    __proto__:parent,
    sayHi: function(){
        return `${super.sayHi()}. how are you doing?`; //super keyword unexpected here
    }
}

const child = {
    __proto__:parent,
    sayHi: function(){
        return `Hi ${this.name}.`; // window의 name property를 찾음. 있으나 값이 없음. 따라서 Hi만 출력됨.
    }
}

const child = {
    __proto__: parent,
    sayHi(){
        return `${super.sayHi()}. how are you doing?`; //ok
    }
}
```

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
    console.log(key + ":" + person[key] + ","); //name: Lee, address:Seoul. __proto__에 추가된 age는 무시
}

console.log(Object.keys(person)); //ES8. ["name", "address"]
```

## <span style="color:#802548">_7. this_</span>
- this binding이란 식별자와 값을 연결하는 과정이다. this라는 식별자와 this가 가리킬 객체가 확보한 메모리 공간의 주소를 binding하는 것이다.
- this binding은 일반함수, 메서드, 생성자함수, call, apply, bind라는 4가지 맥락에 따라 의미가 다르다.

- 일반함수의 경우 lexical scope에 관계없이 무조건 this가 window에 binding된다.
- method라면 객체 자기자신이 binding된다.
```javascript
var value = 1;

const obj = {
    value: 100,
    function foo(){
    console.log("foo's this.value: ", this.value) 
    function bar(){
        console.log("bar's this: ", this.value);
    }
    bar(); //1 ---일반함수로 호출됐으면 어디서든 window. callback이라도..
}
}

obj.foo(); // 100
```

- callback함수라 할지라도 일반함수의 형태라면 this가 window에 binding된다.
```javascript
var value = 1;
const obj = {
    value: 100,
    foo(){
        console.log("foo's this: ", this);
        setTimeout(function(){
            console.log("callback's this: ",this); // window
            console.log("callback's this.value: ", this.value) //1
        }, 100);
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
            console.log("callback's this: ",that); 
            console.log("callback's this.value: ", that.value) 
        },100);
    }
}

obj.foo(); //100
```

- bind함수를 사용한다.
```javascript
var value = 1; // var는 전역프로퍼티에 들어가게 됨
const obj = {
    value: 100,
    foo(){
        setTimeout(function(){
            console.log("callback's this.value: ", this.value) 
        }.bind(this),100);
    }
}

obj.foo(); //100. --bind로 this를 묶으면 객체 자기자신이 binding됨
```

- 화살표 함수를 사용한다.
- 화살표함수를 쓰면 this가 상위 스코프에 binding된다.
```javascript
var value = 1; 
const obj = {
    value: 100,
    foo(){
        setTimeout(() => console.log(this.value), 100);
    }
}

obj.foo();
```

- method 호출의 경우, 일반 함수로 만들어서 호출하면 this가 window에 binding되는 문제가 똑같이 발생한다. 따라서 method를 굳이 일반함수로 만드는 건 좋지 않다.
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

- 위에서 보았듯, call, bind, apply를 사용하면 내부 중첩 함수, 콜백 함수의 this가 불일치하는 문제를 해결할 수 있다.
- call은 this 문제만 해결하는 게 아니다. 아래와 같이 유사배열객체를 실제 배열이 갖는 method를 사용하게 만들 수도 있다.
```javascript
function convertArgsToArray(){
    console.log(arguments);

    const arr = Array.prototype.slice.call(arguments);
    console.log(arr); //[1,2,3]

    return arr;
}
```

## <span style="color:#802548">_8.생성자함수, class, prototype_</span>
- Js는 OOP를 Prototype으로 구현한다.

- 아래와 같이 하면 Circle이라는 Prototype이 만들어진 것이다.
- Circle은 Object라는 부모를 갖게 된다. 그리고 prototype 상에 편입된다.
- 따라서 Object의 prototype에 있는 method를 사용할 수 있다.
```javascript
function Circle(radius){
    this.radius = radius;
    this.getArea = function(){
        return Math.PI * this.radius ** 2;
    }
    //생성자 함수는 return을 쓰면 안 된다.
}

const circle1 = new Circle1(1);
const circle2 = new Circle2(2);
circle1.toString(); // Circle에는 없지만, Object의 prototype에 있는 것이라 사용 가능.
```

- new 없이 Circle()을 넣으면 일반함수로 처리된다.

- return이 없는 일반함수는 undefined를 return한다.
- 하지만 new를 넣은 것처럼 처리할 수 있다.
```javascript
function Circle(radius){
  if(!new.target){ // ES6방식
    return new Circle(radius);
  }
  if(!(this instanceof Circle)){ //ES5방식. babel을 쓰면 위처럼 써도 transpile 시 자동 변환
    return new Circle(radius);
  }

  this.radius = radius;
  this.getDiameter = function(){
    return 2 * this.radius;
  }
}
const circle1 = new Circle(5); //{radius:5, getDiameter:function(){}}
const circle2 = Circle(5); //{radius:5, getDiameter:function(){}} 둘다 똑같음.
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

    Person.prototype = { //위와 동일. Constructor 반드시 명시. ObjectX
        constructor: Person,
        sayHello(){
            console.log(`Hi! My name is ${this.name}`);
        }
    }

    return Person;
}());

const me = Person('Lee');
```

- 다만 polyfill을 제외하면 이렇게 prototype에 추가하는 것은 매우 위험하다.
- JS에서 제공하는 원래 prototype 내용, 즉 소스코드를 덮어씌우는 overriding이 일어날 수 있기 때문이다.
```javascript
String.prototype.indexOf = function() {
    return 'a';
}

const str = 'abc';
str.indexOf(); //'a'
```

- prototype method를 작성했어도 instance method로 override할 수 있다.
```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.sayHello = function() {
    console.log(`Hi! my name is ${this.name}`);
    console.log(`Hi! my name is ${name}`); //this.name으로 써야한다.
}
const person = new Person("Lee");

person.sayHello = function() {
    console.log(`Hi! your name is you`);
}

person.sayHello(); // Hi! your name is you
```

- prototype을 직접 건들게끔 유도하는 생성자함수는 좋지 않다.
- 생성자 함수로 만들어 Prototype을 건들기 보다는 class를 활용하자.
```javascript
class Person {
    constructor(firstName, lastName) { // 1개만 가능. 생략하면 묵시적 생성자로 간주
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
```

- extends를 넣어서 상속관계를 걸 수도 있다.
```javascript
class Animal {
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

class Bird extends Animal {
    //constructor(... args){super(...args)} 
    fly(){
        return 'fly';
    }
}

class Bird1 extends Animal {
    constructor(age, weight,length) {
        super(age, weight) //constructor가 있으면 반드시 호출
        //서브클래스는 수퍼클래스가 인스턴스를 생성하게 위임하기 때문
        this.length = length;
        } 
    fly() {
        return 'fly';
    }

    eat() {
        return `${super.eat()}`;
    }

    move() {
        return super.move();
    }
}
```

- Java와 몇가지 다른 점은 동적인 상속 대상 결정이 가능하다는 점 및 class만이 아니라 생성자 함수를 상속받을 수도 있다는 점이다.
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
- concise method를 쓰면 생성자함수로 활용이 불가능하다.

- class 안의 method는 생성자 함수로 쓸 이유가 전혀 없기 때문에 애초에 원천 차단하는 것이다.
```javascript
class Person{
    name = 'Lee',
    //sayHi = () => console.log(`Hi ${this.name}`);
    sayHi() {
        console.log(`Hi ${this.name}`);
    }
}
```

- concise method를 쓰는 장점은 super도 활용 가능하다는 점이다.
```javascript
const Person = {
    greeting () { return 'hello'; }
};

const friend = {
    greeting () {
        return 'hi, ' + super.greeting();
    }
};

Object.setPrototypeOf(friend, Person); // prototype chain 안에 넣기. extends와 동일. setPrototypeOf(child, parent);
friend.greeting(); // hi, hello
```

## <span style="color:#802548">_9.이터러블과 이터레이터_</span>
- 배열, 문자열, Map, Set 등이 iterable이다.
- 이터러블은 for...of문으로 순회가 가능하며, 스프레트 문법과 배열 디스트럭쳐링 할당의 대상으로 사용할 수 있다.

```javascript
const array = [1,2,3];

for(const item of array) {
    console.log(item);
}
console.log([...array]);

const [a, ...rest] = array;
console.log(a, rest); //1,[2,3]
```
- 이터레이터는 이터러블과 비슷하지만, for of문을 사용할 수 없고, 배열 디스트럭쳐링도 불가능하다.

- 단 Symbol.iterator를 활용해 next를 계속 불러올 수 있다. Java의 iterator와 비슷하다.
- done과 value를 property로 가진다.
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

- iterable 객체는 아래와 같이 우리가 직접 만들어 줄 수도 있다. 
- value와 done을 property로 갖게 하면 된다.
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

- 아래 예시는 사용자 정의 이터러블로 피보나치 수열을 만든 것이다.
```javascript
const fibonacci = {
    [Symbol.iterator](){//done이 true가 될 때까지 진행
        let [pre,cur] = [0,1]; //spread문법
        const max =10;

        return {
            next() {
                [pre, cur] = [cur, pre + cur];
                return {
                    value: cur, 
                    done: cur >= max
                }
            }
        }
    }
}

console.log(fibonacci) // {}. 아무 value도 안 나온다. 아래처럼 순회해야 나온다.
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

for(const num of fibonacciFunc(10)) {
    console.log(num); // 1 2 3 5 8
}

for(const num of fibonacciFunc(15)) {
    console.log(num); // 1 2 3 5 8 13
}
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



## <span style="color:#802548">_10.스프레드 문법, destructuring 문법_</span>
- 하나로 뭉쳐 있는 여러 값의 집합을 펼쳐서 개별 값의 목록으로 만든다. 
- 즉 대괄호 하나를 벗기는 것과 같다.
```javascript
console.log(...[1,2,3]); //1,2,3
console.log(...'hello');//h e l l o
console.log(...new Map([['a','1'],['b','2']])); // ['a','1'], ['b','2']
console.log(...new Set([1,2,3])); //1 2 3

console.log(...{a:1, b:2}); //error. Found non-callable @@iterator
```

```javascript
var arr = [1,2,3];
var max = Math.max.apply(null, arr); //3

const arr = [1,2,3];
const max  = Math.max(...arr); //3
```

- 스프레드 문법은 기존 ES5의 method를 대체할 수 있다.
```javascript
var arr = [1,2].concat([3,4]);
console.log(arr); //[1,2,3,4]

const arr =[...[1,2], ...[3,4]];
console.log(arr); //[1,2,3,4]


var arr1 = [1,4];
var arr2 = [2,3];
arr1.splice(1,0,arr2); //[1,[2,3],4]; //index 1부터 0개 요소를 지우고 arr2를 넣는다.

Array.prototype.splice.apply(arr1,[1.0].concat(arr2)); //[1,2,3,4]
arr1.splice(1,0,...arr2); //[1,2,3,4];
```

- 아래와 같이 기존에는 call을 썼다면, 유사배열 객체도 spread로 배열로 만들 수 있다.
```javascript
function sum() {
    var args = Array.prototype.slice.call(arguments);

    return args.reduce(function (pre, cur){
        return pre + cur;
    }, 0);
}

function sum(){
    return [...arguments].reduce((pre, cur) => pre + cur, 0);
}

const sum = (...args) => args.reduce((pre, cur) => pre + cur, 0);
console.log(sum(1,2,3)); //6
```

```javascript
const arrayLike = {
    0: 1,
    1: 2,
    2: 3,
    length: 3
}

const arr = [...arrayLike];
Array.from(arrayLike); //유사 배열 객체 또는 이터러블을 배열로 변환한다.
```

- 스프레드 문법으로 객체의 property value도 바꿀 수 있다.
```javascript
const obj = {x: 1, y: 2};
const merged = Object.assign({}, {x: 1, y: 2}, {y: 10, z: 3})
const merged = {...{x: 1, y: 2}, ...{y:10, z:3}}
console.log(merged); // {x:1, y:10, z:3} 나중에 들어온 property가 덮어씀
```

- destructuring 문법도 있다.

- ES5의 문법으론 아래와 같이 index로 접근했다.
```javascript
var arr = [1,2,3]; 
var one = arr[0];
var two = arr[1];
var three = arr[2];
```

- ES6에서는 비구조화 할당을 실행하면 된다.
```javascript
var arr = [1,2,3];
var [one,two,three] = [1,2,3];
const arr = [1,2,3];
const [one,two,three] = arr;
console.log(one,two,three);
```

- 아래와 같이 쓰면 오류가 나거나 의도하지 않은 결과를 초래할 수 있다.
```javascript
const [x,y] //SyntaxError
const [a,b] = {} //TypeError: {} is not iterable
const [c,d] = 1;
console.log(c,d); //1 undefined
const [e,f] = [1,2,3];
console.log(e,f); //1 2 
```

- 아래와 같이 destructuring에 기본값 적용도 가능하다.
```javascript
const [a,b,c=3] =[1,2];
console.log(a,b,c) ; //1 2 3. 기본값 적용가능

const [e,f = 10, g = 3] = [1,2];
console.log(e,f,g); // 1 2 3. 할당된 값이 기본값보다 우선
```

- 아래는 destructuring의 사용예제다.
```javascript
function parseURL(url = ''){
    const matchedURL = url.match(/^(\w+):\/\/([^/]+)\/(.*)$/);
    console.log(matchedURL);
//["https://developer.mozilla.org/ko/docs/Web/JavaScript","https","developer.mozilla.org","ko/docs/Web/JavaScript"]
    if(!matchedURL) {
        return {};
    }
    const [protocol,host,path] = matchedURL;
    return {
        protocol,
        host,
        path
    }
}
const parsedURL = parseURL('https://developer.mozilla.org/ko/docs/Web/JavaScript');
console.log(parsedURL);
/*
protocol: 'https',
host: 'developer.mozilla.org',
path: 'ko/docs/Web/JavaScript'
*/
```

- 이제는 객체의 destructuring의 예시를 살펴보자.
- 아래는 ES5의 예시다.
```javascript
var user = {
    firstName: 'JaeWon',
    lastName: 'Lee'
}

var firstName = user.firstName;
var lastName = user.lastName;
console.log(firstName, lastName);
```

- 아래는 ES6의 예시다.
```javascript
const user = {
    firstName: 'JaeWon',
    lastName: 'Heo'
}
const {lastName, firstName} = user; //key를 기준으로 하기에, 순서를 뒤바꾸는 것은 아무 의미 없다.
console.log(firstName, lastName) //JaeWon Heo
```
- 배열과 마찬가지로 기본값 설정도 가능하다.
```javascript
const {firstName = 'JaeWon', lastName} = {lastName: 'Heo'}; //오른쪽은 실제 객체라서 기본값 제공 불가능!
console.log(firstName, lastName) //JaeWon Heo
const {firstName = 'JaeWon', lastName} = {lastName = 'Heo'}; //SyntaxError: Invalid shorthand property initializer 
const {lastName: ln, firstName: fn} = user;
console.log(fn,ln) //JaeWon Heo
```

- 함수의 parameter로 들어갈 때도 가능하다.
```javascript
function printTodo({content, completed}){
    console.log(`할일 ${content} ${completed ? '완료' : '비완료'} 상태입니다.`);
}
printTodo({id:1, content: 'HTML', completed:true});
```

- 중첩 객체의 경우 아래와 같이 사용한다. 한번 더 감싸준다.
- address가 중첩 객체를 property로 갖는 할당은 address를 정의한 것이 아니라 식별자가 없는 것이다.
- 다시 써서 지정해줘야 한다.
```javascript
const user = {
    name: 'Lee',
    address: {
        zipCode: '03068',
        city: 'Seoul'
    }
}

const {address:{city}} = user;
console.log(address) //Reference Error: address is not defined 
console.log(city) //Seoul

const {address} = user;
console.log(address)
/*
{
    zipCode: '03068',
    city: 'Seoul'
}
*/
console.log(city) //Seoul
```

```javascript
const user = {
    name: 'Lee',
    address: {
        zipCode: '03068',
        city: 'Seoul'
    }
}

const {address} = user;
console.log(address) 
/*
{
    zipCode: '03068',
    city: 'Seoul'
}
*/
```

- rest parameter도 사용가능하다.
```javascript
const {x, ...rest} = {x: 1, y: 2, z: 3};
console.log(x, rest); //1 {y:2, z:3}
```

- 배열의 요소가 객체라면 배열과 객체 모두 destructuring 할당이 가능하다.
```javascript
const todos = [
    {id:1, content: 'HTML', completed: true},
    {id:2, content: 'CSS', completed: false},
    {id:3, content: 'JS', completed: false},
];

const [,{id}] = todos; //2
const [{id}] = todos;  //1
const [,,{id}] = todos; //3
const [{id},{id},{id}] = todos //Uncaught SyntaxError: Identifier 'id' has already been declared 
todos.forEach(element => console.log(element.id)); //1, 2, 3
```

- 단, 이때 var 변수로 선언하면 안된다. var는 재선언이 가능하기 때문이다. 그럼 맨 마지막 선언한 id가 되어서 3이된다.
```javascript
var [,{id}] = todos; 
console.log(id) //2
var [{id}] = todos;  
console.log(id) //1
var [,,{id}] = todos; 
console.log(id) //3
var [{id},{id},{id}] = todos //Uncaught SyntaxError: Identifier 'id' has already been declared 
console.log(id) //3
```

## <span style="color:#802548">_11.흔히 쓰이는 method_</span>
- 아래는 array의 일반함수다.
```javascript
const arr = Array.of(1,2,2,3); //array로 만든다.
const arr = Array.from(1,2,2,3); //이터러블이나 유사배열객체만 array로 변환해줌.
Array.isArray(arr); //true;
arr.indexOf(2); //1
arr.includes(2) //true. indexOf보다 더 나은 방식이다. ES7에서부터 활용가능

arr.push(3); //[1,2,2,3,3];
arr[arr.length] = 3; //이게 push보다 빠르다. 직접 원본 배열을 변경한다.
const newArr = [...arr,3]; //새로운 배열을 반환한다.
arr.concat(3); // 새로운 배열을 반환. 기능은 push와 동일

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



## <span style="color:#802548">_12.closure_</span>
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


