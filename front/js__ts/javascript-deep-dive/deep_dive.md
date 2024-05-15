## <span style="color:#802548">_1.JS의 변수 초기화_</span>

- 수학에서 변수는 변하는 수다.
- 프로그래밍에서 변수는 변하는 무언가를 담기 위한 메모리 주소다.
- 그럼 변수를 어떻게 생성할까? 변수를 선언해야 한다.

```javascript
var second;
```

- 변수를 선언했다는 것은, 값을 저장하기 위한 메모리 공간을 확보했다는 것이다.
- 그 외 식별자와 확보된 메모리 공간의 주소를 연결했다는 것이다.
- 그렇다면 이제 값을 할당해야 한다.
- 선언만 하는 경우에는 JS 엔진이 undefined로 초기화해준다.

```javascript
var second;
second = 'B';
```

```javascript
var second;
//second = undefined;
```

```javascript
second();
second(new Rectangle(10,60))
function second(){
    console.log('second');
}

function second(x){
    console.log(x);
}

class Rectangle {
  constructor(height, width) {
    this.height = height;
    this.width = width;
  }
}
```

- 아래와 같은 typescript 문법은 js를 run할 때 hoisting되는 것처럼 보인다.
- 아래에다가 써도 type을 참조할 수 있다.
- 하지만 이것은 hoisting이 아니다. hoisting은 runtime에만 일어난다.
- ts는 기본적으로 transpile 시에만 존재하며, runtime 때는 js 코드로 변형된다.
- 따라서 transpile 시에 미리 type을 확인하는 것일 뿐이다. runtime에 관여하지 않는다.

```javascript
interface second{

}
```

## <span style="color:#802548">_2.JS의 값식문_</span>
- 리터럴이란 사람이 이해할 수 있는 문자, 약속된 기호를 사용해 값을 생성하는 방법이다.
- Java의 예시는 아래와 같다.

```java
String a = "B"; //String은 ""로
char a = 'C'; //char는 ''로
float a = 0.1f; //float는 소수점 + f로
```

- JS도 Java와 비슷하지만 더 확장된 형태고 더 유연한 형태다.

```javascript
var score = "B";
var score = 'C';
var score = 20;
var score = 20.01;
var score = {};
var score = [];
var score = /[A-Z]+/gi;
function score(){

}
var score = true;
```

- 위와 같은 literal을 이용해서 변수를 선언하고 할당할 수 있다.
- 아래 keyword, identifier, literal, semicolon 모두 문장의 구성요소다.
- 문장의 구성요소들이 나뉠 수 없는 순간부터 token으로 취급된다.
- 세미콜론은 statement가 closed되었다는 의미다.

```javascript
const// keyword 선언
const score //keyword, identifier
const score = 50 //keyword, identifier, literal
const score = 50;//keyword, identifier, literal, semicolon
```

- unexpected token이라는 오류는 문장을 구성할 때 순서에 맞춰야 하는데 뭔가 이상하게 끼었다는 뜻이다. 보통 ;를 써야할 때 ,를 써서 난다.

```javascript
const score = 50, //unexpected token const
const score2 = 100


const score = 50, //ok
      score2 = 100
```

- if문, for문, 함수 등의 코드 블록 뒤에는 ;를 붙이지 않는다.
- 그 자체로 종결의 의미를 갖기 때문이다.

```javascript
if(condition1){

}

if(condition2){

}
```

- 여태까지 본 것은 값과 문이었다.
- 이제 식을 살펴보자.
- 식이란 결국 모두 값으로 귀결될 수 있는 statement다.
- 보통은 연산자에서 많이 보인다.

```javascript
x = x + 3;
const var = (a ===3) ? true : false;
```

- 값을 넣어야 하는 곳에 문을 넣을 수는 없지만, 식을 넣을 수는 있다.

```javascript
let result;
const score = {
    math : if(condition1){ //unexpected token if
        result = 10;      
    }
}

const score = {
    math : condition1 ? 10 : undefined; //unexpected token ;. 아직 문장이 끝나지 않았기에 ;를 붙이면 안 된다.
}

const score = {
    math : condition1 ? 10 : undefined 
}
```

## <span style="color:#802548">_3. data type_</span>
- 숫자는 전부 실수로 처리된다. 따라서 js에서는 소수와 정수를 나누지 않는다.

```javascript
console.log(1 === 1.0) //true
console.log(2.0 === 4/2); //true
```


- 모두 Number라는 숫자형이다. 또한 2진수, 8진수 등을 써도 모두 10진수로 해석된다.

```javascript
const binary = 0b01000001;
const octal = 0o101;
const hex =0x41;

console.log(binary) //65
console.log(octal) //65
console.log(hex) //65
```

- 숫자인지 아닌지 판단하려면 아래와 같은 방법이 있다.

```javascript
Number.isNaN(variable) === false && typeof variable === 'number';

function isNumber(variable){
  return (Number.isNaN(variable) === false && typeof variable === 'number')
}
```

- 문자열은 모두 UTF-16의 집합이며, '' 와 ""와 ``로 표현한다.
- 그냥 따옴표들은 개행이 적용되지 않지만, ``는 개행이 적용된다.
- 백틱은 html을 쓸 때 자주 사용하며, 그외에는 변수값을 가져올 때 자주 사용한다.
- 백틱에서 변수값은 모두 문자열로 취급된다.

```javascript
const str = 'hello
world'; // invalid or unexpected token;

const str = 'hello\nworld';
```

```javascript
const template = `hello
world`;

const template = `<ul>
                    <li><a href="#">Home</a></li>
                  </ul>`;

const url = `/api/naver/${url}`;
```


- undefined는 선언만 했을 때 할당해주는 값이다.
- JS는 쓰레기값을 만들지 않기 위해 선언만 한 경우 undefined로 할당한다.
- 값이 없음을 알리고 싶다면 null을 할당해줘야 한다.
- 그러면 GC가 수행된다. undefined일 때는 GC가 작동하지 않는다.

```javascript
var foo; //GC 수행 X
var foo = 'Lee';

foo = null; //GC 수행
```

- 함수가 유효한 값을 반환할 수 없을 때도 null을 반환하는 경우도 있다.
- 그 이유는 함수 스펙이 그런 식으로 되어있기 때문이다.
```javascript
var element = document.querySelector('.myClass'); //null
querySelector<K extends keyof HTMLElementTagNameMap>(selectors: K): HTMLElementTagNameMap[K] | null;
```

- null이 값인지 확인할 때는 typeof를 사용하면 안 된다.

```javascript
var element = document.querySelector('.myClass'); //null
typeof element === null; //false. null을 typeof로 하면 object. 
element === null; // true.
```

## <span style="color:#802548">_4.control flow statement_</span>
- block statement, conditional statement, loop statement가 있다.
- 일반적으로 블록문만 단독으로 쓰이는 일은 거의 없다.
- 예시를 위한 예시다.
```javascript
{
  var score = 10;
}
```

- 조건문에는 if, switch가 있다.
- if가 if else만 있다면 삼항연산자로 바꾸는 것도 좋은 선택이다.

```javascript
let str;
if(condition){
  str = "true";
}else{
  str = "false";
}

let str = condition ? 'true' : 'false';
```

- if가 너무 복잡하다면 switch case로 바꾸는 것도 좋은 선택이다.
- switch문은 변수가 문자, 숫자일 때만 사용가능하다는 단점이 있긴 하다.
```javascript
let script = abc;
if(script === "condition1"){

}else if(script === "condition2"){

}else if(script === "condition3"){

}else if(script === "condition4"){

}else{
  
}

switch(script){
  case "condition1":
    break;
  case "condition2":
    break;
  case "condition3":
    break;
  case "condtion4":
    break;
  default: //default는 끝이므로 break;를 굳이 걸필요가 없다.
}


let script = abc;
if(script === "condition1"){
  console.log("condition1");
}else if(script === "condition2"){
  console.log("condition1");
}else if(script === "condition3"){
  console.log("condition1");
}else if(script === "condition4"){
  console.log("condition2");
}else{
  
}

switch(script){ //condition4전 까지 모두 같은 로직이라면 break;를 걸지 않는다.
  case "condition1":
  case "condition2":
  case "condition3":
    break;
  case "condtion4":
    break;
  default: //default는 끝이므로 break;를 굳이 걸필요가 없다.
}
```


```javascript
const str = 'Hello World';
const searchedWord = 'l';
let index;

for(var i = 0; i < str.length; i++){
  if(str[i] === searchedWord){
    index = i;
    break;
  }
}

console.log(index); //2
```

- js method를 활용하면 for, if를 안쓰고 아래와 같이 깔끔하게 한줄로 요약된다.
```javascript
console.log(str.indexOf(searchedWord)); //2
```

- 마찬가지다. 
```javascript
const str = 'Hello wolrd';
const searchedWord = 'l';
let count = 0;

for(let i = 0; i < str.length; i++){
  if(str[i] !== searchedWord){
    continue;
  }

  count = count + 1;
}
console.log(count) //3

const regex = new RegExp(searchedWord, 'g');
console.log(str.match(regex).length); //3
```

## <span style="color:#802548">_5.immutable_</span>
- raw type의 value는 immutable하다.
- 그렇다면 새로운 원시값을 어떻게 생성하는 것일까? 이는 Java와 동일하다.

```java
String a = "b";

a = "c";
```

- 위와 같이 Java에서 초기화를 하면 처음 선언하고 할당한 a의 메모리주소에 b 대신 c가 들어가는 게 아니다.
- 아예 새로운 메모리 주소가 만들어지고, a의 주소 참조값이 그 새로운 메모리 주소로 만들어지며, 거기에 c가 들어간다.
- js도 마찬가지다. "a"를 담던 메모리 주소가 이제 참조가 사라졌으니 곧 GC가 수거해갈 것이다.


```javascript
var str = "a";
str = "c";
```

- 아래도 비슷하다. copy와 number1이 같다고 하였으나, 원시값이기에 메모리 주소가 같은 게 아니라 값이 같은 것이다.
- 따라서 이전의 원시값을 담은 변수인 number1을 변경해도 copy의 값이 변경되지 않는다.
```javascript
var number1 = 10;
var copy = number1;

number1 = 50;

console.log(copy) // 10
```

- 반면에 객체로 만들었다면 변할 수도 있다.
- 객체 property key에 원시값 value를 담은 뒤 이를 변경하면, 변경된 채로 나오는 것을 알 수 있다.
- 객체는 불변이 아니기 때문이다.
```javascript
let score = {
  football : 80
}

let copy = score;

score.football = 100;

console.log(copy.football) //100
```


## <span style="color:#802548">_6. object_</span>
- JS에서는 원시형을 제외한 모든 게 객체다. class, function, array, set, date 등..
- 객체는 key - value 구조로 이뤄진다.

```javascript
const object = {
  key: value
}

const month = 10;
const elem = {
  month : month
}

const elem = {
  month
}

const elem = {
  month : function(){ // property의 value로 쓰인 fn은 method라고 한다.
    console.log("month"); 
  }
}

elem.hello = 'world';
elem['hello'] = 'world';

var key = 'hello';
elem[key] = 'world';
```

- 객체의 property는 아래와 같이 가져올 수 있다.
- 하지만 null은 property가 없기 때문에 오류가 난다.
- 따라서 객체가 null을 담고 있는지 평가하는 작업이 필요하다.
- &&평가는 null을 elem을 반환하고, 
```javascript
var elem = null;
value = elem.value; // TypeError: Cannot read property 'value' of null
if(elem){
  value = elem.value;
}

var value = elem && elem.value; //null. elem이 undefined 초기화라면 undefined. 하지만 undefined로 초기화는 매우 나쁨.

var value = elem?.value; // elem에 value라는 key가 없을 때 반환 값은 undefined로 고정
```

- 한번 생성된 문자열은 읽기 전용 값으로 변경이 불가능하다. 문자열은 JS에서 원시값이다.
- 원시값이 index를 가지는 것도 이상하겠지만, 유사배열 객체면 가능하다. index가 있고 for문도 순회한다.
- 함수의 인자는 arguments도 있다.
```javascript
var str = 'string';

str[0] = 'S';

console.log(str) //string
```

- person1과 person2는 key와 value가 같다고 해도, 서로 메모리주소가 다른 객체다
- 따라서 identity를 비교한다면 false다. 하지만 그 안의 property의 원시값은 동일하기에 둘을 비교하면 true다.
```javascript
var person1 = {
  name : 'Lee'
}

var person2 = {
  name : 'Lee'
}

console.log(person1 === person2); //false
console.log(person1.name === person2.name); //true 
```

## <span style="color:#802548">_7. 함수_</span>
- 두 식은 모두 동일하다. 다만 add라는 함수의 이름만이 차이다. 함수의 이름을 주지 않으면 익명함수가 된다.
- 물론 함수 이름은 함수 내부에서만 쓰인다. 함수를 불러올 때는 식별자 이름으로 불러올 수 있다.
```javascript
var add = function add(x,y){
  return x + y;
}

var add = function (x,y){
  return x + y;
}

add(5,6);
```

- 다만 함수의 인자를 넣을 때는 입력 값에 유의해야 한다.
- 특히 빈 string + 안 넣은 경우가 조합되면 무조건 값이 undefined로 나오게 된다.
- 인자가 있음에도 넣지 않으면 undefined로 초기화되기 때문이다.

```javascript
add();
var add = function(undefined,undefined){
  return undefined + undefined
};

add('');
var add = function('',undefined){
  //return '' + undefined;
  return 'undefined';
}

add(2);
var add = function(2,undefined){
  //return 2 + undefined;
  return NaN;
}


var add = function(x = 0 ,y = 0){
  return x + y;
}; //단 이 방법은 x와 y가 입력값이 없는 undefined일 때만 통하고, null을 넣으면 안 통함
// null을 넣으면 아래와 같이 됨.

var add = function(x = 0 ,y = 0){
  return null + 0;
}; 

var add = function(x,y){
  if(typeof x !== 'number' || typeof y !=='number'){
    throw new TypeError("인수는 모두 숫자값이어야 합니다.");
  }
}

var add function(event){
  const regex = /[0-9]/gi;
  if(!regex.test(event.target.value) ){
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

- 그러한 이유로 typescript를 많이 사용하는 추세다.
- 함수가 받을 수 있는 parameter의 type을 아예 지정해버리는 것이다.
```javascript
var add = function(x : number, y: number){
  return x + y;
}
```


- 함수는 함수 리터럴 표현식, 함수 리터럴 선어문, new 생성자로 만들 수 있다.
- 이 중에 추천하는 방식은 함수 리터럴 표현식이다.
- 의도치 않은 hoisting을 막을 수 있고, 변수명과 함수명이 동일한 경우를 미연에 방지할 수 있다.


```javascript
var add = function(x, y){
  return x + y;
}

function add(x,y){
  return x + y;
}

var add = new Function('x','y','return x + y');
```

```javascript
function foo(){
  return; // return undefined;
}

function foo(){
 // return undefined;
}

console.log(foo()); //undefined
```

- 함수를 여러번 쓰는 것보다 함수형태를 쓰고 함수를 규정해서 집어넣을 수도 있다.
- 함수가 객체라 parameter에 우겨넣는 게 가능한 js의 특징이다.
- 이와 같이 함수의 parameter로 넘어가는 함수가 곧 callback함수다.
```javascript
function repeat(n,f){
  for(let i = 0; i < n; i++){
    f(i);
  }
}

let logAll = function(i){
  console.log(i);
}

repeat(5,logAll); // 0,1,2,3,4

let logOdds = function(i){
  if(i % 2){
    console.log(i);
  }
}

repeat(5,logOdds); //1,3,

let arr = arr.filter((element) => element ! == data);
```

- 비순수함수보다는 순수함수로 만드는 게 변경을 막는데 유리하다.
- 아래는 비순수함수다. 외부 변수인 count를 함수 내에서 직접 참조하고 있다.
```javascript
var count = 0;

function increase(){
  return ++count;
}

increase();
console.log(count); //1

increase();
console.log(count); //2
```

- 아래는 순수함수다. 외부 변수를 매개변수로 받아서 내부에서 값을 변화시킨다.
```javascript
var count = 0;
function increase(n){
  return ++n;
}

count = increase(count);
console.log(count); // 

count = increase(count);
console.log(count);
```
- 함수는 lexical scope인데, 자신이 정의된 곳의 환경을 기억한다.
- 아래에서 보면 bar()는 foo()안에서 호출되었다.
- 하지만 정의된 곳은 그 밖이며, 그곳의 x값은 1이기 때문에 1이 console에 출력된다.
```javascript
var x = 1;
function foo(){
  var x = 10;
  bar();
}

function bar(){
  console.log(x);
}

foo(); //1
bar(); //1
```

## <span style="color:#802548">_8. 프로퍼티 어트리뷰트_</span>
- 모든 객체는 [[Prototype]]이라는 내부 슬롯을 갖는다. 
- 내부 슬롯은 원래는 개발자가 접근할 수 없으나, [[Prototype]]만 간접 접근이 가능하다.

```javascript
const object = {};
object.[[Prototype]];
object.__proto__ //Object.prototype
/*constructor
: 
ƒ Object()
hasOwnProperty
: 
ƒ hasOwnProperty()
isPrototypeOf
: 
ƒ isPrototypeOf()
propertyIsEnumerable
: 
ƒ propertyIsEnumerable()
toLocaleString
: 
ƒ toLocaleString()
toString
: 
ƒ toString()
valueOf
: 
ƒ valueOf()
__defineGetter__
: 
ƒ __defineGetter__()
__defineSetter__
: 
ƒ __defineSetter__()
__lookupGetter__
: 
ƒ __lookupGetter__()
__lookupSetter__
: 
ƒ __lookupSetter__()
__proto__
: 
(...)
get __proto__
: 
ƒ __proto__()
set __proto__
: 
ƒ __proto__()*/
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

- 그 다음은 seal이다. property 값 갱신은 가능하다.

```javascript
const person = {name: 'Lee'}
Object.seal(person);

person.age = 10;
console.log(person); //{name: 'Lee'}

delete person.name; 
console.log(person); //{name: 'Lee'}

person.name = 'Kim';
console.log(person); //{name: 'Kim'}
```

- 그 다음은 preventExtensions이다. property 추가는 불가능하다.

```javascript
const person = {name: 'Lee'}
Object.preventExtensions(person);

person.age = 10;
console.log(person); //{name: 'Lee'}

person.name = 'Kim';
console.log(person); //{name: 'Kim'}

delete person.name; 
console.log(person); //{}
```

## <span style="color:#802548">_9. Object 생성자_</span>
- Object를 생성하는 함수는 return이 함수단위로 있으면 안 된다.
- return이 없어야 본래 생성자함수로서 기능한다.
- 생성자함수와 일반 함수간에 형식차이가 따로 없기 때문에 생성자는 파스칼케이스로 명명한다.


```javascript
function Circle(radius){
  this.radius = radius;
  this.getDiameter = function(){
    return 2 * this.radius;
  }
}
const circle1 = new Circle(5); //{radius:5, getDiameter:function(){}}
const circle1 = Circle(5); //undefined
```

- 일반함수에 new를 붙이는 경우, return이 원시값이면 빈 객체가 return된다.
- 만약 parameter가 필요한 일반함수에 new를 안붙이면 undefined 혹은 NaN이 return될 수 있다.


```javascript
function add(x,y){
return x + y;
}

const dis = new add(); // {}
const dis = add(); //NaN
```


- new 없이 Circle()을 넣었을 때도 new를 넣은 것처럼 처리할 수 있다.
- 아래는 ES6에서 쓸 수있는 method다.

```javascript
function Circle(radius){
  if(!new.target){ // ES6
    return new Circle(radius);
  }

  this.radius = radius;
  this.getDiameter = function(){
    return 2 * this.radius;
  }
}
const circle1 = new Circle(5); //{radius:5, getDiameter:function(){}}
const circle2 = Circle(5); //{radius:5, getDiameter:function(){}}}
```

- ES5에서는 아래와 같이 쓸 수 있다.
```javascript
function Circle(radius){
  if(!(this instanceof Circle)){ //ES5
    return new Circle(radius);
  }

  this.radius = radius;
  this.getDiameter = function(){
    return 2 * this.radius;
  }
}

const circle1 = new Circle(5); //{radius:5, getDiameter:function(){}}
const circle2 = Circle(5); //{radius:5, getDiameter:function(){}}}
```

- 생성자 안에 함수를 넣어놓으면 생성되는 instance마다 해당 함수를 넣을 메모리 공간이 필요하다.
- 따라서 아래와 같이 prototype에 getter 함수를 넣는다.
```javascript
function Circle(radius){
  if(!(this instanceof Circle)){ //ES5
    return new Circle(radius);
  }

  this.radius = radius;
}

Circle.prototype.getDiameter = function(){
  return 2 * this.radius;
}

const circle = new Circle(5);
console.log(circle.getDiameter());
```







