## <span style="color:#802548">_1. var와 const_</span>
var를 사용하지 않는 이유
- hoisting과 function scope이기 때문이다.
- 실제로는 대부분 함수안에 끼어넣어서 js 변수를 사용한다.
- 따라서 fn scope 단위이기 떄문에 위와 같은 오류는 흔하게 일어나진 않는다.
- 예시만 참고하자. if문이라는 block에 대해서 scope가 fn이라서 무시되는 모습이다.
```javascript
var name = "abc";
if(true){
	var name = "bbb";
}
console.log(name); //bbb
```

- 재할당과 재선언을 반복해도 error가 나지 않기 때문이다.
- name이라는 변수명이 같은데 오류가 안난다.
- 제일 마지막에 할당한 것이 값이 된다.
```javascript
var name = "abc";
var name = "bbb";
console.log(name); //bbb
```

- 대신 써야할 것은 const와 let인데 그 중에서도 const다.

```javascript
const name = "abc";
if(true){
	name = "bbb"; //error
}
```

```javascript
const name = "abc";
const name = "bbb"; //error
```

- const는 재할당만 금지된다.
- 따라서 reference 값들은 얼마든지 조작할 수 있다.

```javascript
const name = {
	first: "Heo",
	last: "Jaewon"
};

name.first = "Kim";
name.last = "See";
console.log(name.first) // Kim

const name = [{
	first: "Heo",
	last: "Jaewon"
}];

name.push({
	first: "Kim",
	last: "See"
})
``` 

## <span style="color:#802548">_2. window와 global_</span>
- Node 환경에서는 global이고, browser에서는 window가 최상위객체다. 둘이 똑같다.

- 파일을 나눠도 scope가 나뉘지 않는다.
- index.js에 아래와 같이 규정하자.

```javascript
var globalVar = "global";
```

- 그리고 이를 index2.js에서 가져와보자.
- 그럼 가져와진다.
```javascript
console.log(window.globalVar); //global
console.log(globalVar); //global
```

- 이걸 막고 싶다면 근본적으로 const, let을 사용하고 즉시실행함수를 사용해야 한다.

## <span style="color:#802548">_3. 임시객체 없애기_</span>
- 그저 반환만 하는 경우가 있다고 해보자.
- 그럼 임시변수를 쓰지 않고 아래와 같이 바로 return하게 바꿀 수 있다.

```javascript
function getElements(){
	const obj = {};
	obj.title=document.querySelector(".text");

	return obj;
}

function getElements(){
	const obj = {
		title : document.querySelector(".text")
	};

	return obj;
}

function getElements(){
	return {
		title : document.querySelector(".text")
	};
}
```

- 마찬가지로 overloading을 활용해 임시변수를 줄일 수 있다.
- js와 맞지않게 명령형 코드로 만들어져버린다.
- 디버깅도 힘들고 추가 코드를 만들고 싶은 유혹에 빠진다.

```javascript
function getDateTime(targetDate){
	var month = targetDate. getMonth();
	var day = targetDate.getDate();
	var hour = targetDate.Hours();

	month = month >= 10 ? month : '0' + month;
	day = day >= 10 ? day : '0' + day;
	hour = hour >= 10 ? hour : '0' + hour;

	return {
		month,
		day,
		hour
	};
}

function getDateTime(targetDate){
	let month = targetDate. getMonth();
	let day = targetDate.getDate();
	let hour = targetDate.Hours();

	month = month >= 10 ? month : '0' + month;
	day = day >= 10 ? day : '0' + day;
	hour = hour >= 10 ? hour : '0' + hour;

	return {
		month,
		day,
		hour
	};
}

function getDateTime(targetDate){
	return {
		month: month >= 10 ? month : '0' + month,
		day: day >= 10 ? day : '0' + day,
		hour: hour >= 10 ? hour : '0' + hour
	};
}

function getDateTime(){
	const currentDateTime = getDateTime(new Date());

	return {
		month: currentDateTime.month + '분 전',
		day: currentDateTime.day + '일 전',
		hour: currentDateTime.hour + '초 전',
	}

}

function getDateTime(){
	const currentDateTime = getDateTime(new Date());

	return {
		month: computedKrDate(currentDateTime.month),
		day: currentDateTime.day + '일 전',
		hour: currentDateTime.hour + '초 전',
	}

}
```
- 해결책은 위와 같이 함수를 나누는 것, 바로 반환하는 것이다.
- 고차함수도 좋은 방식이다.

```javascript
function getSomeValue(params){
	let tempVal = '';

	for(let index = 0; index< array.length; index++){
		temp = array[index];
		temp += array[index];
		temp += array[index];
		temp += array[index];
	}

	if(temp ??){
		tempVal = ??;
	}else if(temp ??){
		temp += ??
	}

	return temp;
}
```

## <span style="color:#802548">_4. 타입 검사_</span>
- typeof로 볼 수 있는 것은 js의 primitive형이다.
- reference형(array, date, function)은 typeof로 해도 잘 안나온다.
- null이 object로 나와서 굉장히 위험하다.
- 또한 type도 동적이기 때문에 일관된 type이 보장되기 어렵다.
```javascript
function myFunction(){

}
typeof new String("b")	//object
typeof "b" 				//'string'
typeof myFunction		//'function'

typeof null 			//'object'
```

- instanceof도 있는데, 이것도 위험하다.
- 객체는 결국 최상위가 Object이기 때문이다.
- 
```javascript
const arr =[];
const date = new Date();
const func = () => {};

arr instnaceof Array //true
date instnaceof Date //true
func instnaceof Function //true

arr instnaceof Object //true
date instnaceof Object //true
func instnaceof Object //true
```

- typeof와 instanceof 외에 검사할 때는 prototype chain을 활용하는게 좋다.

```javascript
Object.prototype.toString.call(date)	 // [Object Date]; 뒤에거가 제대로 나옴.
Object.prototype.toString.call(arr)	 // [Object Array];
Object.prototype.toString.call(func)	 // [Object Function];
```

## <span style="color:#802548">_5. undefined와 null_</span>
- 값이 명시적으로 없다면 null이다.
```javascript
!null //true
!!null //값을 boolean으로 형변환. false
null + 13 // 13
typeof null // 'Object'
```

- 값을 할당하지 않으면 undefined다.
```javascript
!undefined //true
undefined + 13 // NaN
typeof undefined // 'undefined'
```

- 
```javascript
undefined == null //true
undefined === null //false
!undefined === !null //true
```

## <span style="color:#802548">_6. == 대신 ===쓰기 _</span>
- ==은 값만 비교한다.
- 값이 같으면 자동으로 묵시적 type casting해버린다.
- ===은 값에 씌워진 박스 자체를 비교한다. type casting이 일어나지 않는다.
- 안전한 프로그래밍은 ===으로 비교하는 것이다.

```javascript
'1' == 1 // true
1 == true //true

ticketNumber.value == 0 //비추천
Number(ticketNumber.value) ===	0 //추천. eslint에서 설정해주면 안하면 error 냄
```

## <span style="color:#802548">_7. 명시적으로 형변환하기 _</span>
- 아래와 같이 쓰면 암묵적 형변환이 된다.
```javascript
11 + '문자와 결합'	// '11문자와 결합' 
!!'문자열' //true
!!''	//false
```

- 그것보단 아래와 같이 명시적으로 형변환을 해줘야 유지보수가 좋다.
- parseInt와 Number의 차이는 parseInt는 정수형만, Number는 소수까지 다 가져올 수 있다는 점이다.
```javascript
String(11+ '문자와 결합'); // String
Boolean('문자열'); // true
Boolean('') // false
Number('11') // 11
parseInt('9.9999', 10); //뒤에 10진수라고 꼭 알려주기. 
```

## <span style="color:#802548">_8. js의 숫자 _</span>
- JS에서 다루는 숫자의 끝을 보려면 아래와 같다.

```javascript
Number.MAX_SAFE_INTEGER //9007199254740991. yyyymmddhhmmss의 문자열을 숫자로 형변환해 비교해도 충분.
```

```javascript
isNaN(123) //false. false면 숫자라는 뜻.
isNaN(123 + '테스트'); // false. false면 숫자. 근데 문자를 붙였는데도 false임..
Number.isNaN(123 + '테스트') // true. true면 숫자 X. Number만 붙여주면 엄격한 검사를 함.
Number.isNaN(123) // false. false면 숫자. Number만 붙여주면 엄격한 검사를 함.
```


## <span style="color:#802548">_6. 경계_</span>

- 특별한 flag 값들은 magic number로 두지 말고 변수명으로 바꾸자.

- 그런데 경계값이 애매할 수 있다.
- 그럴 때는 네이밍에 최소값과 최대값을 포함해야 한다. 이상-초과, 이하-미만을 구분할 때 어떻게 naming할 지 생각해보자.
- 

```javascript
if(a > 5){

}

const MIN_NUMBER = 5;
if(a > MIN_NUMBER){ // LIMIT가 없으면 초과?

}

const MIN_NUMBER_LIMIT = 5;
if(a >= MIN_NUMBER_LIMIT){ //LIMIT가 붙으면 이상.

}
```

- 시작일은 포함인데, 끝날은 포함되지 않을 수 있다. 이럴 때 begin과 end를 변수명으로 쓴다. 체크인을 생각해보자. 

```javascript
function reservationDate('2023-11-06','2023-11-07');

const beginDate = "yyyy-mm-dd";
const endDate = "yyyy-mm-dd";
function reservationDate(beginDate, endDate);
```

- 포함된 양 끝을 first와 last를 의미하는데, 보통 연속된 값이 아닐 때 쓴다.

```javascript
const students = ['포코','존','현석'];

function getStudents(firstStudent, lastStudent);
```

- setter, getter, private field 쓰기

```javascript

const language = {
	set current(name){
		this.log.push(name);
	},
	log:	[]
};

const obj = {
	log: ['a','b'],
	get latest(){
		if(this.log.length === 0){
			return undefined;
		}
		return this.log[this.log.length -1]
	}
}
function FactoryFunction(name){
	this._name = name;
}

class FactoryFunction{
	#name = name;
}
```

- 하지만 위의 것보다도 더 중요한 것은 매개변수의 순서다.

```javascript
getRandomNumber(1, 50);
getDates('2021-10-01','2021-10-31');
getShuffleArray(1,5);
```

## <span style="color:#802548">_7. 값식문_</span>

```javascript
ReactDOM.render(
	<div id="msg">Hello World!</div>,
	mountNode
)
 
//Babel 만나면 아래와 같이 변함
ReactDom.render(React.createElement('div',{id:'msg'},'Hello World'), mountNode);
```

- if문을 값에 넣으면 오류가 난다.

```javascript
const obj = {
	id: if(condition){ 
		'msg' //오류남.
	}
}

ReactDOM.render(<div id={condition ? 'msg ' : null}> Hello world! </div>, mountNode);
//오류 안남. 삼항연산자는 식이고 식은 곧 값이 되기 때문.

<p>{this.state.color || 'white'}</p> //error 없음. 논리연산자는 곧 값이 되기 때문.
```

- if문을 사용하려면 어떻게 해야할까?
- 아래와 같이 즉시실행함수를 사용하여 바로 return해주면 값이 되기 떄문에 switch"문"으로도 가능
```javascript
<p>
	{(()=>{
		switch(this.state.color){
			case 'red':
			return '#FF0000';
			case 'green':
			return '#00FF00';
			case 'blue':
			return '#0000FF';
			default:
			return '#FFFFFF';
		}
	})()}
</p>
```

- 하지만 즉시실행함수 코드는 이해하기 어렵고 더럽다.
- 그를 고차함수로 바꿔주면 보기 더 편해진다.
- 아래는 map을 이용했다.
```javascript
function ReactComponent(){
	return (
		<tbody>
			{(() => {
				const rows = [];

				for(let i = 0; i < objectRows.length; i++){
					rows.push(<ObjectRow key={i} data={objectRows[i]} />)
				}
			})}
		</tbody>
	)
}

function ReactComponent(){
	return (
		<tbody>
			{rows.map((obj,i) => <ObjectRows key={i} data={obj} />)}
		</tbody>
	)
}
```

- 또는 아래와 같이 ES6에 도입된 연산자를 이용할 수도 있다.
```javascript
function ReactComponent(){
	return (
		<tbody>
			{(() => {
			if(condition1) return <span>one</span>
			else if(condition2) return <span>two</span>
			})}
		</tbody>
	)
}

function ReactComponent(){
	return (
		<tbody>
			{condition1 && <span>one</span> }
			{condition2 && <span>two</span> }
		</tbody>
	)
}
```

## <span style="color:#802548">_7. 삼항연산자_</span>
- 삼항연산자는 ?가 1개일 때 말고는 최대한 자제하자. 2개까지는 그래도 이해할만 하지만, 3개부터는 그냥 읽기가 어렵다.
- else if가 너무 많다면 switch case도 고려해보자.

```javascript
function example(){
	return condition ? value1 : condition2 ? value2 : condition3 ? value3 : value4;
}

function example(){
	return condition ? value1 
			: condition2 ? value2 
			: condition3 ? value3 
			: value4;
}

function example(){
	if(condition){
		return value1;
	}else if(condition2){
		return value2;
	}else if(condition3){
		return value3;
	}else{
		value4;
	}
}

const temp = str;
function example(){
	switch(temp){
		case "A":
			return value1;
			break;
		case "B":
			return value2;
			break;
		case "C":
			return value3;
			break;
		default:
			value4;
			break;
	}
}
```
 
- 그 안에서 또 비교해서 값을 넣어야 한다면, 괄호를 꼭 사용해서 사람이 알아보기 쉽게 바꿔주자.
```javascript
const example = condition1 ? a===0 ?  'zero' : 'positive' : 'negative';
const example = condition ? 
					a===0 ? 'zero' : 'positive' : 'negative';

const example = condition ?
					(a===0 ? 'zero' : 'positive') :'negative';

const example = condition ? 
					((a===0) ? 'zero' : 'positive') : 'negative';
```

- 삼항연산자는 값을 return할 때 쓰자
- 함수를 return할 때 쓰는 것은 별로 바람직하지 않다.
- 아래는 비추천하는 예시다.
```javascript
const isAdult = age >= 14 ? alert('성인') : alert('성인 아님');
```

- 또한 true나 false 한 가지 조건의 값만 필요할 때도 굳이 삼항연산자를 쓸 필요 없다.
- 그 때는 truthy, falsey 라는 js의 특성을 이용한다.
- falsey는 null, undefined, 숫자 0, 숫자 -0, 문자 "", 논리 false, NaN가 있다.
- truthy는 기억할 필요가 별로 없다.
```javascript
function printName(name){
	if(name === undefined || name ===null){
		return '사람이 없네요';
	}

	return '안녕하세요 ' + name + '님';
}

function printName(name){
	if(!name){
		return '사람이 없네요';
	}

	return '안녕하세요 ' + name + '님';
}
```

- falsy일 때 기본값을 나타내고 싶다면 아래와 같이 ||를 쓰는 게 삼항연산자보다 간단하다.
```javascript
function fetchData(){
	if(state.data){
		return state.data
	}else{
		return 'Fetching';
	}
}

function fetchData(){
	return state.data ? state.data : 'Fetching';
}

function fetchData(){
	return state.data || 'Fetching';
}
```

```javascript
function favoriteDog(someDog){
	let favoriteDog;
	if(someDog){
		favoriteDog = dog;
	}else{
		favoriteDog = '냐옹';
	}

	return favoriteDog + '입니다';
}

function favoriteDog(someDog){
	return (someDog || '냐옹') + '입니다';
}
```

```javascript
const getActiveUserName(user,isLogin){
	if(isLogin){
		if(user){
			if(user.name){
				return user.name;
			}else{
				return '이름없음';
			}
		}
	}
}

const getActiveUserName(user,isLogin){
	if(isLogin && user){
			if(user.name){
				return user.name;
			}else{
				return '이름없음';
			}
	}
}

const getActiveUserName(user,isLogin){
	if(isLogin && user){
		return user.name || '이름없음';
	}
}
```

## <span style="color:#802548">_8. if문_</span>
- else if 처리는 엔간하면 하지 말고 if문을 따로 쓰는 게 대부분 이해하기 쉽다.
- else if가 너무 늘어진다면 switch - case로 바꾸자.

```javascript
const x = 1;
if(x >= 0){
	console.log('x는 0과 같거나 크다');
}else if(x > 0){
	console.log('x는 0보다 크다'); //작동안함. 위의 if문에서 걸린 것.
}

if(x >= 0){
	console.log('x는 0과 같거나 크다');
}else{//else if는 else 문 안의 if와 동일.
	if(x > 0){
	console.log('x는 0보다 크다'); //작동안함. 위의 if문에서 걸린 것.
	}
}

if(x >= 0){
	console.log('x는 0과 같거나 크다');
}

if(x > 0){
	console.log('x는 0보다 크다');
}
```

- else문을 마구잡이로 쓰는 것도 지양해야 한다.
- 특히 2개 이상의 기능을 함수에 담을 때 문제가 될 수 있다.
- 
```javascript
function getHelloCustomer(user){
	if(user.age > 20){ // 청소년도 greeting message를 받아야 하는데, 청소년은 else문에 도달할 수 없는 상황이다.
		report(user);
	}else {
		return '안녕하세요';
	}
}

function getHelloCustomer(user){
	if(user.age > 20){
		report(user);
	}

	return '안녕하세요'; // 청소년도 greeting message를 받을 수 있다.
}
```

- early return으로 if문 내 깊이를 줄일 수도 있다.
```javascript
function loginService(isLogin, user){
	if(!isLogin){
		if(!checkToken()){
			if(!user.nickName){
				return registerUser(user);
			}else{
				refreshToken();

				return '로그인 성공';
			}
		}else{
			throw new Error('No Token');
		}
	}
}

function loginService(isLogin, user){
	if(isLogin){
		return;
	}

	if(!checkToken){
		throw new Error('No Token');
	}

	if(!user.nickname){
		return registerUser(user);
	}

	refreshToken();

	return '로그인 성공';
}

function loginService(isLogin, user){
	if(isLogin){
		return;
	}

	if(!checkToken){
		throw new Error('No Token');
	}

	if(!user.nickname){
		return registerUser(user);
	}

	login();
}

function login(){
	refreshToken();

	return '로그인 성공';
}
```

```javascript
function 오늘하루(condition, weather, isJob){
	if(condition === 'GOOD'){
		공부();
		게임();
		유튜브보기();

		if(weahter === "GOOD"){
			운동();
			빨래();
		}

		if(isJob === "GOOD"){
			야간업무();
			조기취침();
		}
	}
}

function 오늘하루(condition, weather, isJob){
	if(condition !== 'GOOD'){
		return;
	}

	공부();
	게임();
	유튜브보기();

	if(weather === "GOOD"){
		운동();
		빨래();
	}

	if(isJob === "GOOD"){
		야간업무();
		조기취침();
	}
}

function 오늘하루(condition, weather, isJob){
	if(condition !== 'GOOD'){
		return;
	}
	
	공부();
	게임();
	유튜브보기();

	if(weather !== "GOOD"){
		return;
	}

	운동();
	빨래();

	if(isJob !== "GOOD"){
		return;
	}

	야간업무();
	조기취침();
}
```

- 부정연산자는 최대한 덜 활용하자.
- 아래와 같이 3이 숫자인지 검사하려고한다.
- isNaN을 쓰면 !를 붙여서 해야되는데, 이러면 생각을 여러번 해야 한다.
- if - else문으로 쓰지, else - if문으로 쓰지 않는다. true일 떄부터 생각하게 되어있다.

```javascript
if(!isNaN(3)){
	console.log('숫자입니다');
}
```
- !를 조건 앞에 붙이지 말고 method를 만들어 내부 구현으로 숨기자
```javascript
function isNumber(num){
	return !Number.isNaN(num) && typeof num === 'number';
}

if(isNumber(3)){
	console.log('숫자입니다');
}
```

- 부정 조건문은 early return, validation에 쓰는 게 좋다.

## <span style="color:#802548">_8. if문_</span>
- 사용자가 값을 넣지 않았을 때 기본값을 설정하는 행위는 매우 중요하다.
- x와 y를 안넣었을 떄는 1
- height와 width를 안 넣었을 때는 100
```javascript
function sum(x,y){
	x = x || 1;
	y = y || 1;

	return x + y;
}

function createElement(type, height, width){
	const element = document.createElement(type);

	element.style.height = height || 100;
	element.style.width = width || 100;

	return element;
}

function registerDay(userInputDay){
	switch(userInputDay){
		case '월요일';
		case '화요일';
		case '수요일';
		case '목요일';
		case '금요일';
		case '토요일';
		case '일요일';
		default: //오타가 나면 아래와 같이 error 메시지를 뱉어낸다. 그래서 switch - case를 쓴 것이다.
			throw Error('입력값이 유효하지 않습니다.');
	}
}

e.target.value = '월ㄹ요일';
registerDay(e.target.value);
```

- parseInt는 기본값이 10이 아니기 때문에 아래와 같이 10으로 기본값을 정해주는 별도의 method를 만들어주는 것도 좋다.
```javascript
function safeParseInt(number,index){
	return parseInt(number, radix || 10);
}
```

- 연산자의 우선순위는 ()로 눈에 띄게 나타낸다

```javascript
if(isLogin && token || user)

if((isLogin && token) || user)
```

-전위연산자와 후위연산자도 ++이나 --는 덜 쓰자.

```javascript
function increment(number){
	number--;
}

let number;
function increment(){
	number = number- 1;
}
```

- falsy가 아니라 null과 undefined일 때만 작동시키고 싶다면 ||가 아니라 ??를 쓰자.
- 특히 숫자 0의 경우 falsy인데 null은 아니다. 그럴 때 ??를 쓴다.
```javascript
function createElement(type,height,width){
	const element = document.createElement(type || 'div');
}
const el = createElement('div',0,0);
//el은 height와 width가 10이 되버림. 0은 falsy하기 떄문.

function createElement(type,height,width){
	const element = document.createElement(type ?? 'div');
}
const el = createElement('div',0,0);
//el이 의도대로 0으로 출력됨.
```

```javascript
function helloWorld(message){
	if(!message){
		return 'Hello! World';
	}

	return 'Hello! ' | (message || 'World');
}

console.log(helloWorld(0)); // Hello! World; 0이라서 if문에서 false로 걸린다. Hello! 0이 될 수가 없다.

function helloWorld(message){

	return 'Hello! ' | (message ?? 'World');
}

console.log(helloWorld(0)); // Hello! 0; null과 undefined일 때만 falsy하게 하면 Hello! 0으로 출력된다.
```

- 논리연산자와 null coleascing은 같이 쓸 수 없다.
```javascript
console.log(null || undefined ?? "foo"); //error
console.log((null || undefined) ?? "foo") //foo. 괄호로 묶었기 때문에 가능하다. 괄호로 묶으면 값이 나오기 때문이다.
```

```javascript
if(!(isValidToken && isValidUser)) //아래가 더 이해하기 쉽다.
if(!isValidToken || !isValidUser) 

if(!(isValidToken || isValidUser)) //아래가 더 이해하기 쉽다
if(!isValidToken && !isValidUser)
```

## <span style="color:#802548">_8. 배열_</span>

- 배열의 index에 숫자가 아니라 문자열 key도 넣을 수 있다.
- js에서 배열은 객체기 때문이다. 그냥 0,1,2,3이라는 의미없는 key가 들어간 것과 똑같다고 생각할 수 있다
```javascript
const arr = [1,2,3];
arr[3] = 'test';
arr['property'] = 'string value';
arr['obj'] = {};
arr[{}] = [1,2,3];
arr['func'] = function(){
	return 'hello';
}

const obj = {
	arr: [1,2,3],
	3 : 'test',
	property: 'string value',
	obj : {},
	'{}' : [1,2,3],
	fun : function(){
		return 'hello'
	}
}

Array.isArray(arr); //배열임을 검사할 때 쓰는 method.
```

- js 배열의 length는 요소의 갯수가 아니라 마지막 index를 알려줄 뿐이다.

```javascript
const arr = [1,2,3];
arr.length = 10;

console.log(arr.length); //10. 요소가 3개밖에 없지만, 마지막 index가 9가 되어 길이가 10이 된다.

Array.prototype.clear = function(){
	this.length = 0;
}

function clearArray(array){
	array.length = 0;

	return array;
}

console.log(arr) //[]; const arr = [];와 같은 초기화는 length를 0으로 만들면서도 가능.
```

- 배열을 받는 것도 아래와 같이 구조분해할당을 이용할 수 있다.
```javascript
function operateTime(input, operators, is){//배열의 index로 접근
	inputs[0].split('').forEach((num) =>{
		cy.get('.digit').contains(num).click();
	})

	inputs[1].split('').forEach((num) =>{
		cy.get('.digit').contains(num).click();
	})
}

function operateTime(input, operators, is){//구조 분해 할당으로 접근
	const [firstInput,secondInput] = inputs
	
	firstInput.split('').forEach((num) =>{
		cy.get('.digit').contains(num).click();
	})

	secondeInput.split('').forEach((num) =>{
		cy.get('.digit').contains(num).click();
	})
}

function operateTime([firstInput, secondeInput], operators, is){//parameter에 구조분해할당으로 넣기
	firstInput.split('').forEach((num) =>{
		cy.get('.digit').contains(num).click();
	})

	secondeInput.split('').forEach((num) =>{
		cy.get('.digit').contains(num).click();
	})
}
```

- index를 처리하지 않아도 배열을 구조분해할당하면 된다.
```javascript
function clickGroupButton(){
	const confirmButton = document.getElementsByTagNam('button')[0];
	const cancelButton = document.getElementsByTagNam('button')[1];
	const resetButton = document.getElementsByTagNam('button')[2];
}

function clickGroupButton(){
	const [confirmButton, cancelButton, resetButton] = document.getElementsByTagNam('button');
}
```

```javascript
function formatDate(targetDate){
	const date = targetDate.toISOString().split('T').[0];

	const [year, month, day] = date.split('-');

	return `${year}년 ${month}월 ${day}일`;
}

function formatDate(targetDate){
	const [date] = targetDate.toISOString().split('T');

	const [year, month, day] = date.split('-');

	return `${year}년 ${month}월 ${day}일`;
}

function head(arr){
	return arr[0] ?? '' //배열에 요소가 없는데 [0]같이 index에 접근하면 error가 나니까 기본값 셋팅
}

function formatDate(targetDate){
	const date = head(targetDate.toISOString().split('T'));

	const [year, month, day] = date.split('-');

	return `${year}년 ${month}월 ${day}일`;
}
```

- 유사 배열 객체도 있다.
- 유사 배열 객체는 배열이 아니다. 하지만 for문으로 돌릴 수 있다.
- 하지만 function 안에만 있는 map, forEach 등은 사용이 불가능하다.

```javascript
function generatePriceList(){
	for(let i = 0; i < arguments.length; i++){
		const element = arguments[index];

		console.log(element); //100, 200, 300, 400, 500, 600 for문은 순회가능.
	}
}

generatePriceList(100, 200, 300, 400, 500, 600);

function generatePriceList(){
	for(let i = 0; i < arguments.length; i++){
		return arguments.map((arg) => arg + '원'); // arguments.map is not a function
												   // prototype에 없음. 배열이 아니라서.
	}
}

function generatePriceList(){
	for(let i = 0; i < arguments.length; i++){
		return Array.from(arguments).map((arg) => arg + '원'); //가능. array로 변경했기 때문.
	}
}

//arguments의 내부는 아래와 같이 생김. 이건 array로 변경 가능!
const arrayLikeObject = {
	0: 'HELLO',
	1: 'WORLD',
	length:2,
};

const arr = Array.from(arrayLikeObject);

console.log(Array.isArray(arrayLikeObject))// true
console.log(arr) // [HELLO, WORLD]
console.log(arr.length)	// 2
```

- 객체의 메모리 주소값을 같게 해놓고 요소를 추가하면 원본 배열도 영향을 받는다.
- 따로 새로운 배열을 만들어야 한다.
```javascript
const originArray = ['123','345','789'];

const newArray = originArray;

originArray.push(10);
originArray.push(11);
originArray.push(12);
originArray.unShift(0);

console.log(newArray) //[0,'123','345','789',10,11,12];
```

- 새로운 배열을 기존의 배열 값을 복사해서 만들면, 새로운 배열의 값을 바꿔도 불변이다.
```javascript
const originArray = ['123','345','789'];

const newArray = [...originArray];

originArray.push(10);
originArray.push(11);
originArray.push(12);
originArray.unShift(0);

console.log(newArray) //['123','345','789'];
```

- for문 대신 고차함수를 활용해라!


```javascript
const price = ['2000','1000','3000','5000','4000'];
function getWonPrice(priceList){
	let temp = [];

	for(let i = 0; i <priceList.length; i++){
		if(priceList[i] > 1000){
				temp.push(priceList[i] + '원'); //원화표기
		}
	}

	if(orderType === 'ASCENDING'){
		someAscendingSortFunc(temp);
	}

	if(orderType === 'ASCENDING'){
		someDescendingSortFunc(temp);
	}
	
	return temp;
}
```

- 아래처럼 고차함수를 활용해 바꿀 수 있다.
```javascript

function getWonPrice(priceList){

	return return priceList.map((price) => price + '원');
}

const suffixWon = (price) => price + '원';
function getWonPrice(priceList){

	return return priceList.map(suffixWon);
}

const suffixWon = (price) => price + '원';
const isOverOneThousand = (price) => Number(price) > 10000;
function getWonPrice(priceList){
	const isOverList = priceList.filter(isOverOneThousand); //원화표기 + 1000원 이상만 노출

	return isOverList.map(suffixWon);
}

const suffixWon = (price) => price + '원';
const isOverOneThousand = (price) => Number(price) > 10000;
const ascendingList = (a,b) => a - b;
function getWonPrice(priceList){
	const isOverList = priceList.filter(isOverOneThousand); //원화표기 + 1000원 이상만 노출
	const sortList = isOverList.sort(ascendingList);

	return sortList.map(suffixWon);
}

const suffixWon = (price) => price + '원';
const isOverOneThousand = (price) => Number(price) > 10000;
const ascendingList = (a,b) => a - b;
function getWonPrice(priceList){
	return priceList.filter(isOverThousand).sort(ascendingList).map(suffixWon);
}

const suffixWon = (price) => price + '원';
const isOverOneThousand = (price) => Number(price) > 10000;
const ascendingList = (a,b) => a - b;
function getWonPrice(priceList){
	return priceList
		.filter(isOverThousand)
		.sort(ascendingList)
		.map(suffixWon);
}

const result = getWonPrice(price);

console.log(result); // ['1000원', '2000원','3000원','4000원','5000원'];
```

- forEach는 값을 return하지 않고 element를 순회하며 내부 함수를 실행한다.
- map은 새로운 배열을 return한다.

```javascript
const prices = ['1000','2000','3000'];

const newPricesForEach = prices.forEach(function(price){
	return price + '원';
})

const newPricesMap = prices.map(function(price){
	return price + '원';
})

console.log(newPricesForEach); //undefined
console.log(newPricesMap); //['1000원','2000원','3000원']
```

```javascript
const orders = ['first', 'second', 'third'];

orders.forEach(function(order){
	if(order === 'second'){
		break; // error. 고차함수에서는 break;나 continue를 쓸 수 없다. 
	}

	console.log(order); 
})

try{
	orders.forEach(function(order){
	if(order === 'second'){
		throw new Error("error");
	}

	console.log(order); 
})
}catch(e){

}

for(let i = 0; i <array.length; i++){
	const element = array[index];
	if(array[i] === 'second'){
		break;
	}
}

for(const element of orders){
	if(element === 'second'){
		break;
	}

	console.log(element)//'first'
}

for(const key in orders){
	if(orders[key] === 'second'){
		break;
	}
  
  console.log(orders[key])//'first'
}

//흐름제어는 위의 for문이나 try ~ catch말고 아래와 같이도 활용할 수 있다.
Array.prototype.every() 	// truthy. &&와 비슷하게 작동
Array.prototype.some()  	// falsy. ||와 비슷하게 작동
Array.prototype.find()  	// 특정 item을 찾으면 종료
Array.prototype.findIndex() //특정 index를 찾으면 종료
```

## <span style="color:#802548">_9. 객체를 다루는 기술_</span>
- shorthand property

```javascript
const firstName = 'poco';
const lastName = 'jang';
const person = {
	firstName : 'poco',
	lastName : 'jang',
	getFullName: function(){ 
		return this.firstName + '' + this.lastName;
	}
}

const person = {
	firstName,
	lastName,
	getFullName(){ //concise method 
		return this.firstName + '' + this.lastName;
	}
}
```

- computed property name는 객체의 key를 동적으로 넣을 때 쓴다.
- value만 동적으로 넣는 게 아니라 key도 동적으로 넣을 수 있게 된 것이다.

```javascript
const [state,setState] = useState({
	id: '',
	password:''
})
const handleChange = (e) => {
	setState({
		[e.target.name]: e.target.value, //event.target에 동적으로 속성을 넣을 수 있다.
	})
}

return ( 
	<React.Fragment>
		<input {state.id} onChange={handleChange} name="id">
		<input {state.id} onChange={handleChange} name="password">
)
```


```javascript
const reducer = handleActions(
	{
		[noop]: (state, action) =>({ //noop도 computed property name. 동적으로 key가 생성됨.
			counter: state.counter + action.payload,
		}),
	}
)

const store = new Vuex.Store({
	state:{

	},
	mutations: {
		[SOME_MUTATION](state){}, //[SOME_MUTATION]도 computed property name
	}
})

const funcName0 = 'func0';
const funcName1 = 'func1';
const funcName2 = 'func2';

const obj = {
	[funcName0](){
		console.log('func0');
	}
	[funcName1](){
		console.log('func1');
	}
	[funcName2](){
		console.log('func2');
	}
}
for(let i = 0; i < 3; i++){
	console.log(obj[funcName + i]()); //func0, func1, func2
}
for(let i = 0; i < 3; i++){
	console.log(obj['func' + i]());
}
for(let i = 0; i < 3; i++){
	console.log(obj[`func${i}`]());
}
```


```javascript
function func1(a, b) {
  return a + b;
}
function func2() {
  return 'hello';
}

let obj = {
  [`key${func1(5,8)}`] : 'result is 13',
  [func2()] : 'hi'
}

// obj = {
//   key13 : 'value is 13',
//   hello: 'hi'
// }

const prefix = 'prop';
let i = 0;

const obj = {
    [`${prefix}-${++i}`]: i,
    [`${prefix}-${++i}`]: i,
    [`${prefix}-${++i}`]: i
};

console.log(obj); // { 'prop-1': 1, 'prop-2': 2, 'prop-3': 3 }
```

- else if가 너무 길어지면 switch문을 써보자.
- switch문으로도 계속 추가될 거 같다면 lookup table을 사용해보자.
```javascript

function getUserType(type){
	if(type ==='ADMIN'){
		return '관리자';
	}else if(type === 'INSTRUCTOR'){
		return '강사';
	}else if(type === 'STUDENT'){
		return '수강생';
	}.
	.
	.
	.
	.

}
```

```javascript
function getUserType(type){
	const USER_TYPE = {
		ADMIN: '관리자',
		INSTRUCTOR: '강사',
		STUDENT: '수강생'
	}

	return USER_TYPE[type] ?? '해당 없음';
}

function getUserType(type){
	return (
		{
			ADMIN: '관리자',
			INSTRUCTOR: '강사',
			STUDNET:'수강생'
		}[type] ?? '해당없음'
	)
}

function getUserType(type){
	const USER_TYPE = {
		ADMIN: '관리자',
		INSTRUCTOR: '강사',
		STUDENT: '수강생',
		UN: '해당 없음'
	}

	return USER_TYPE[type] ?? USER_TYPE.UN
}

import USER_TYPE from './constants./...';
function getUserType(type){ // 함수는 고정. USER_TYPE만 계속 바꿈.
	return USER_TYPE[type] ?? USER_TYPE.UN
}
```

- 객체 비구조할당을 하면 parameter의 순서를 신경쓰지 않아도 된다.
- default처리는 parameter가 아니라 함수 내부에서 이뤄진다.
```javascript
function Person(name, age, location){
	this.name = name;
	this.age = age;
	this.location = location;
}

const poco = new Person('poco',30,'Korea');


function Person({name, age, location}){
	this.name = name ?? '이름 없음';
	this.age = age ?? '0';
	this.location = location ?? '인천';
}

const poco = new Person({
	name:'poco',
	age:30,
	location:'korea'
});
```

- 필수값은 반드시 받아야 한다면 아래와 같이 썼을 것이다.

```javascript
function Person(name, options){
	this.name = name;
	this.age = options.age;
	this.location = options.location
}

const person = new Person('재원',30,'인천');
```

- 객체비구조 할당을 이용하면 더 편해진다.
```javascript
function Person(name, {age,location}){
	this.name = name;
	this.age = age;
	this.location = location
}

const pocoOptions = {
	age: 30,
	location: '인천'
}

const person = new Person('재원',pocoOptions);
const person = new Person('선호', {
	age: 50,
	location: '서울'
})
```

```javascript
const orders = ['First', 'Second', 'Third'];

const st = orders[0];
const rd = orders[2];
console.log(st); //First
console.log(rd); //Third

const [st1, , rd1] = orders;
console.log(st1); //First
console.log(rd1); //Third

const {0:st2, 2:rd2} = orders;
console.log(st2); //First
console.log(rd2);  //Third
```

- React의 props를 내려줄 때도 비구조할당을 쓴다.
```javascript
function welcome({name}){
	return <h1> Hello, {props.name}</h1>;
}
```

- Object를 freeze할 때는 재귀로 해야 한다.
- value 안의 object까지는 freeze가 되지 않는다.
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
STATUS.OPTIONS.YELLOW = 'Y';
console.log(STATUS)
// {
// 	PENDING: 'PENDING',
// 	SUCCESS:'SUCCESS',
// 	FAIL:'FAIL',
// 	OPTIONS:{
// 		GREEN: 'G',
// 		RED:'RED'
// 	}
// }
```

- deepFreeze는 lodash 혹은 직접 구현, typescript readonly로 할 수 있다.
```javascript
const deepFreeze(){

	Object.keys(targetObj).forEach(key =>{
		if(/*객체가 맞다면 */){
			deepFreeze(targetObj[key])
		}
	})
}
```

- prototype 조작은 하지말자.
- JS는 매우 많이 발전했고, JS 빌트인 객체를 건드는 게 위험하기 때문이다.
- 특히 기존 JS의 빌트인 method는 절대 건들지 말자.
- 특정 method를 지원하지 않는 브라우저에서만 polyfill정도 만드는 게 좋다.
```javascript
function Car(name,brand){
	this.name = name;
	this.brand = bran;
}

Car.Prototype.sayName = function(){
	return this.brand + '-' + this.name;
}
```


- 해당 객체에 property가 있는지 확인할 때는 그냥 hasOwnProperty를 쓰면 안된다.
- call을 써야한다. 
```javascript
const person = {
	name: 'jaewon'
}
person.hasOwnProperty('name'); //true

const foo = {
	hasOwnProperty: function(){
		return 'hasOwnProperty';
	},
	bar: 'string'
}
foo.hasOwnProperty('bar'); //hasOwnProperty
Object.prototype.hasOwnProperty.call(foo, 'bar') //true
```

- 매번 하기 귀찮다면 아래와 같이 util함수로 만드는 것도 좋다.
```javascript
function hasOwnProp(targetObj, targetProp){
	 return Object.prototype.hasOwnProperty.call(targetObj,targetProp);
}

hasOwnProp(person, 'name'); //true
const person = {
	name: 'jaewon'
}
const foo = {
	hasOwnProperty: function(){
		return 'hasOwnProperty';
	},
	bar: 'string'
}
hasOwnProp(foo, 'hasOwnProperty'); //true
```


- 직접 접근하는 것을 지양하자.
- 아래는 변수의 property에 직접접근하는 나쁜 예시다.
```javascript
const model = {
	isLogin: false,
	isValidToken: false,
}

function login(){
	model.isLogin = true;
	model.isValidToken = true;
}

function logout(){
	model.isLogin = false;
	model.isValidToken = false;
}
```

- 함수의 parameter로 넣어서 바꾸자. 접근자 함수를 만들자. 순수함수로 바꾸자는 것이다.
- 장점은 변화를 최소화하고, 새로운 기능을 쉽게 추가할 수 있다는 점이다.
```javascript
function setValidToken(bool) {
	model.isValidToken = bool;
	ServerAPI.log();
}

function setIsLogin(bool){
	model.isLogin = bool;
	ServerAPI.log();
}

function login(){
	setIsLogin(true)
	setValidToken(true)
}

function logout(){
	setIsLogin(false)
	setValidToken(false)
}
```

- 서버에서 response를 받을 때 optional chaining을 쓰자.
- runtime에서 오류가 나는 것을 방지할 수 있다.
```javascript
const response = {
	data:{
		userList:[
			{
				name: 'Jang',
				info:{
					tel: '010',
					email: 'jang@gmail.com'
				}
			}
		]
	}
}

function getUserEmailByIndex(userIndex){
	return response.data.userList[userIndex].info.email
}

getUserEmailByIndex(0); //userList이 서버에서 제대로 오지 않았을 때, 그냥 return하면 error뜸.

function getUserEmailByIndex(userIndex){
	if(response.data.userList){
		return response.data.userList[userIndex].info.email
	} 
}
//근데 data도 없다면? 아래와 같이 if문 중첩으로 막아버림
function getUserEmailByIndex(userIndex){
	if(response.data){
		if(response.data.userList){
			if(response.data.userList[userIndex]){
				return response.data.userList[userIndex].info.email
			}
		}
	}
}
//그게 보기 싫어서 &&로 바꿈.
function getUserEmailByIndex(userIndex){
	if(response.data && response.data.userList && response.data.userList[userIndex]){
		return return response.data.userList[userIndex].info.email
	}
}


// &&가 길기 때문에 optional chaining으로 바꿈.
function getUserEmailByIndex(userIndex){
	if(response?.data?.userList?.[userIndex]?.info?.email){
		return return response.data.userList[userIndex].info.email
	}
}

return '알 수 없는 에러가 발생했습니다'; 

```

## <span style="color:#802548">_10. 추상화_</span>
```javascript
setTimeout(()=>{
	scrollToTop();
},  3 * 60 * 1000)

const COMMON_DELAY_MS = 3 * 60 * 1000;
setTimeout(()=>{
	scrollToTop();
}, COMMON_DELAY_MS)


const PRICE = {
	MIN: 1000000,
	MAX: 100000000
}
const PRICE = {
	MIN: 1_000_000,
	MAX: 100_000_000
}

getRandomPrice(0,10);
getRandomPrice(PRICE.MIN,PRICE.MAX);


function isValidName(name){
	return carName.length >= 1 && carName.length <=5;
}

const CAR_NAME_LEN = Object.freeze({
	MIN: 1,
	MAX: 5,
});
const CAR_NAME_LEN = {
	MIN: 1,
	MAX: 5
} as const;

function isValidName(name){
	return carName.length >= CAR_NAME_LEN.MIN && carName.length <= CAR_NAME_LEN.MAX;
}
```

- DOM API 접근 추상화

```javascript
export const loader = () => {
	const el = document.createElement("div)");
	el.setAttrtibute("class",'loading d-flex justify-center mt-3');

	const el2 = document.createElement('div');
	el2.setAttribute("class","relative spinner-container");

	const el3 = document.createElement("div");
	el3.setAttribute('class','material spinner');

	el.append(el2);
	el2.append(el3);
}


const createLoader = () => {
	const el = document.createElement("div)");
	const el2 = document.createElement('div');
	const el3 = document.createElement("div");

	return {
		el, el2, el3
	}
}


const createLoaderStyle = ({el,el2,el3}) => {
	el.setAttribute("class",'loading d-flex justify-center mt-3');
	el2.setAttribute("class","relative spinner-container");
	el3.setAttribute('class','material spinner');

	return {
		newEl: el, 
		newEl2: el2, 
		newEl3: el3
	}
}
const loader = () => {
	const {el, el2, el3} = createLoader();
	const {newEl, newEl2,newEl3} = createLoaderStyle();

	newEl.append(newEl2);
	newE2.append(newEl3);
}

const myModal = () => {
	return document.querySelector("#modal");
}

const changeColor = (element) => {
	element.style.backgroundColor = "black";
}

const openModal = (element) => {
	element.classList.add('--open');
}

const closeModal = (element) => {
	element.classList.remove('--open')
}

openModal(myModal);
changeColor(myModal);
closeModal(myModal);
```

## <span style="color:#802548">_11. 에러 다루기_</span>
- 서버에 에러를 덜 내기 위해선 validation이 필요하다.
- validation은 HTML 단위, JS 단위에서 등 가능한 단위에서 다 해줘야 된다.
- front에서 했으면 backend에서도 해야된다.


- try ~ catch는 1개만 쓰자. 물론 catch는 복수 개 쓸 수도 있다.
- catch를 하면 개발자를 위한 예외처리, 사용자를 위한 예외처리, 사용자에게 사용을 제안, 로그수집을 해야 한다.
- 그냥 console.log는 안된다.
```javascript
function handleSubmit(input){
	try {

	}catch(error) {
		//1. 개발자를 위한 예외처리
		console.warn(error);
		//2. 사용자를 위한 예외처리
		//alert(error), alert('404', not Found);
		alert('잠시만 기다려주세요. ~~문제가 발생했습니다.');
		//3. 사용자를 위한 제안
		history.back();
		history.go('안전한 어디 url');
		clear();
		element.focus();
		//4. 로그
		sentry.전송();
		//5. 만약에 필요하다면 재귀호출
		handleSubmit();
	}finally {
		//데이터분석을 위한 로그
	}
}
```

## <span style="color:#802548">_12. semantic tag 지켜쓰기_</span>
<html>
	<body>

<!-- header -->
	<div>
		<div>
			<span></span>
		</div>
	</div>

<!-- contents-->
	<div>
		<span></span>
	</div>

<!-- footer-->
	<div></div>

	</body>
	<script></script>
</html>

<html>
	<body>

<!-- header -->
	<header>
	</header>

<!-- contents-->
	<main>
		<article>
			<section>
			</section>
		</article>

		<article>
			<section>
			</section>
		</article>
	</main>

<!-- footer-->
	<footer></footer>

	</body>
	<script></script>
</html>


## <span style="color:#802548">_13. NodeList_</span>
- Node는 document 내의 모든 객체다.
- elements는 tag에 둘러 싸인 객체다.
- html, body, main, table, thead, th 모두 node다.
- NodeList는 일반 배열로 형변환해주는 게 좋다.
```javascript
const arr = document.querySelectorAll('a'); //NodeList라서 배열의 method는 사용불가. entries(), forEach(), item(), keys(), values()만 존재
const arrFromNode = [...arr];//일반 배열이라서 배열의 method 사용가능.
```

## <span style="color:#802548">_13.innerHTML_</span>
- innerHTML은 쓰지 말자. xss 공격에 매우 취약하다.
```javascript
document.querySelector('a').innerHTML = '<h1>hello</h1>'
```

- html을 넣을 때는 insertAdjacentHTML을 쓰자.
- text를 넣을 떄는 textContent를 쓸 수있다.
- text의 위치를 세밀하게 넣어야 한다면 insertAdjacentText를 쓰자.

```html
<!--beforebegin-->
<p>
<!--afterbegin-->
	foo
<!--beforeend-->
</p>
<!--afterend-->
```

```javascript

document.querySelector('p').insertAdjacentHTML('beforeBegin','<h1>hello</h1>');
//innerHTML X
document.querySelector('p').insertAdjacentHTML("afterbegin",'<h1>hello</h1>');
document.querySelector('p').insertAdjacentHTML("beforeend",'<h1>hello</h1>');
document.querySelector('p').insertAdjacentHTML("afterend",'<h1>hello</h1>');

document.querySelector('p').textContent = 'Hello World';
document.querySelector('p').insertAdjacentHText("afterbegin",'예');
document.querySelector('p').insertAdjacentHText("afterbegin",'아니오');
//innertText X
```

## <span style="color:#802548">_14. data attribute_</span>
- 비표준 속성을 쓰지 않게 생긴 표준

```html
<html>
	<main
	id="elect"
	data-columns="3"
	data-index-number="1234"
	data-parent-world="cars">
	main
</main>
</html>
```

```javascript
console.log(main.dataset.columns); //3
console.log(main.dataset.indexNumber); //1234
console.log(main.dataset.parentWorld); //cars
```

## <span style="color:#802548">_15. black box eventListener_</span>
- callback 함수의 이름을 고민없이 만들어 넣으면 무슨 기능이 실행되는 지 알 수가 없다.
```javascript
const button = document.querySelector('button');
button.addEventListener('click',onClick); //onClick 대신 더 제대로 된 이름을..likebutton이라면?
const onclickLikeButton = () => {};

button.addEventListener('click',handleClick); //handleClick 대신 뭘 제출하는 지 써놓자
const handleClick = (e) : void =>	{
	//1. input을 받는 코드
	//2. 유효성 검사를 하는 코드
	//3. form을 전송하는 코드
}

//만약 검색이라면 click event가 아니니까 search라는 본래 기능을 보여주기 위해 아래와 같이 이름을 준다.
const handleSearch = (e) : void =>	{
	//1. input을 받는 코드
	//2. 유효성 검사를 하는 코드
	//3. 전송하는 코드
}
button.addEventListener('click',handleSearch); 
button.addEventListener('keyup',handleSearch); 
button.addEventListener('onsubmit',handleSearch); 
```


## <span style="color:#802548">_16. 공백을 잘 쓰자_</span>
- eslint에서 padding-line-between-statements option을 추가하자.
```javascript
const loadingElements = () => {
	//1. 선언
	const el = document.createElement('div');
	const el2 = document.createElement('div');
	const el3 = document.createElement('div');

	//2. setting
	el.setAttribute('class', 'loading-1')
	el2.setAttribute('class', 'loading-2')
	el3.setAttribute('class', 'loading-3')


	//3. 로직
	el.append(el2);
	el2.append(el3);
	if(el2){
		return el2;
	}

	//4. return
	return el3;
}
```

## <span style="color:#802548">_17. 중첩 줄이기_</span>
- early return
- callback 대신 promise, promise 대신 async-await
- 고차함수
- 함수를 나누고 추상화하기
- method chaining

- eslint에서 indent 정하기
```javascript
{
	"indent" : ["error", "tab"];
}
```
## <span style="color:#802548">_13.함수 관리하기_</span>
- 매개변수가 많을 때는 object형태로 바꿔 넣는 것을 고려해보자.
```javascript
function createCar(name,brand,color,type){
    return {
        name,
        brand,
        color,
        type 
    }
}
createCars('name','brand','color','type');

function createCar(options){
    var name = options.name;
    var brand = options.brand;
    var color = options.color;
    var type = options.type;
    return {
        name: options.name,
        brand: options.brand,
        color: options.color,
        type: options.type
    }
}
createCars({
    name:'name',
    brand:'brand',
    color:'color',
    type:'type'
})

function createCar({name,brand,color,type}){
    var name = options.name;
    var brand = options.brand;
    var color = options.color;
    var type = options.type;
    return {
        name,
        brand,
        color,
        type 
    }
}
function createCar({brand,name,color,type}){ //객체로 만들면 순서가 상관없어서 좋다.
    var name = options.name;
    var brand = options.brand;
    var color = options.color;
    var type = options.type;
    return {
        name,
        brand,
        color,
        type 
    }
}

function createCar(name,{brand,color,type}){ //필수값은 객체밖으로 빼도 된다.
    return {
        name,
        brand,
        color,
        type 
    }
}

createCars('차량이름',{})//안넣은 건 undefined
createCars('차량이름',{
    color:'color',
    brand:'brand',
    type:'type'
})

createCars('차량이름',{ //brand가 없는 차량으로 만듦.
    color:'color',
    type:'type'
})

function createCar(name,{brand,color,type}){ //필수값을 넘기지 않았다면 error를 던져주면 좋다.
    if(!name){
        throw new Error("name은 필수값입니다.");
    }
    return {
        name,
        brand,
        color,
        type 
    }
}

function createCar(name,{brand,color,type}){ //문자열 필수값이면서 공백은 무시하려면 아래와 같이..
    if(typeof name !== 'string' || !name){
        throw new Error("name은 문자열 필수값입니다.");
    }
    return {
        name,
        brand,
        color,
        type 
    }
}

function createCar(name,{brand,color,type}){ //객체 options에서도 필수값이 있다면 아래와 같이..
    if(typeof name !== 'string' || !name.trim()){
        throw new Error("name은 문자열 필수값입니다.");
    }

	if(typeof name !== 'string' || !brand.trim()){
		throw new Error("name은 문자열 필수값입니다.");
	}
    return {
        name,
        brand,
        color,
        type 
    }
}
```
- 기본값을 넣어주자

```javascript
function createCarousel(options){
	options = options || {};
	var margin = options.margin || 0;
	var center = options.center || false;
	var navElement = options.navElement || 'div';

	return {
		margin,
		center,
		navElement
	}
}

createCarousel(); //{ margin: 0, center: false, navElement: 'div'}

function createCarousel({
	margin = 0,
	center = false,
	navElement = 'div',
} = {}){// parameter에 아무것도 안들어오면 빈 객체를 할당.

	return {
		margin,
		center,
		navElement
	}
}) 

const required = (argName) => {
	throw new Error('required is ' + argName); 
}

function createCarousel({ //필수옵션을 넘기지 않을 때, default parameter를 이용한다면, 함수를 써서 error를 return
	items = required('items'),
	margin = 0,
	center = false,
	navElement = 'div'
} = {}){

	return {
		margin,
		center,
		navElement
	}
}
```
- 매개변수가 정해지지 않았다면 rest parameter를 쓰자.
- rest parameter는 arguments와 달리 배열이기 때문에 바로 배열 method를 쓸 수 있다.
```javascript
function sumTotal(initValue, bonusValue, ...args){ //rest parameter는 무조건 맨 마지막에
	console.log(initValue); //100
	console.log(bonusValue); //99

	return args.reduce(
		(acc,cur) => acc + cur, initValue
	)
}
console.log(100,99,1,2,3,4,5);
```

- void 함수에는 return을 넣지 말자.
- void도 undefined를 반환한다.
```javascript
function handleClick(){
	return setState(false);
}

function showAlert(message){
	return alert(message);
}


function handleClick(){
	setState(false);
}

function showAlert(message){
	alert(message);
}

function testVoidFunc(){
	arr.push(10);
}

function testFunc(){
	const arr = [1,2];
	return arr.push(10); //3. array의 push는 늘어난 이후의 length를 반환함.
}
```
- 화살표함수를 잘 사용하자. 맹목적으로 사용하면 안 된다.

```javascript
const user = {
	name: 'Poco',
	getName: () => {
		return this.name;
	}
}

console.log(user.getName()) //undefined

const user = { // method는 concise method로..
	name: 'Poco',
	getName(){
		return this.name;
	}
}

console.log(user.getName()) //Poco

const user = { // method는 concise method로..
	name: 'Poco',
	getName(){
		return this.name;
	}
	newFriends: () =>{
		const newFriendList = Array.from(arguments); //화살표함수에선 arguments를 쓸 수 없다.
		return this.name + newFriendList;
	}
}

const user = { //rest parameter를 활용하자.
	name: 'Poco',
	getName(){
		return this.name;
	}
	newFriends: (...args) =>{
		return this.name + args;
	}
}

const Person = (name,city) => {
	this.name = name;
	this.city = city;
}
const person = new Person('name','city'); //Person is not a constructor;

function Person(name,city) {//생성자함수는 일반 함수로 쓰던가, class로 만들자.
	this.name = name;
	this.city = city;
}
class Person { 
	this.name = name;
	this.city = city;
}


class Person { 
	this.name = name;
	this.city = city;

	parentMethod() {
		console.log('parentMethod');
	}

	parentMethodArrow = () => { // super로 child에서 호출이 불가능하다. 생성자함수 내부에서 바로 초기화되기 때문이다.
		console.log('parentMethodArrow');
	}

	overrideMethod = () => { //child에서 override해도 덮어씌워지지 않는다.
		return 'Parent';
	}
}

new Child().childMethod(); // error. (intermediate value).parentMethodArrow is not a function
new Child().overrideMethod(); //Parent

class Person { 
	this.name = name;
	this.city = city;

	parentMethod() {
		console.log('parentMethod');
	}

	parentMethodArrow(){ //concise method로 쓰면 된다.
		console.log('parentMethodArrow');
	}

	overrideMethod(){ //concise method로 쓰면 된다.
		return 'Parent';
	}
}

class Child extends Parent {
	childMethod(){
		super.parentMethodArrow()
	}

	overrideMethod(){
		return 'Child';
	}
}

new Child().childMethod(); //parentMethodArrow
new Child().overrideMethod(); //Child
```

```javascript
function* gen() {
	yield () => ?? // arrow fn에 yield를 쓸 수 없다.
}

function* gen() {
	yield function name(params){

	}
}
```

- callback fn은 함수의 실행권을 다른 함수에 위임하는 것이다.
- 아래는 addEventListener의 내부 구현 예시다.
```javascript
someElement.addEventListener('click',function(){
	console.log(someElement + '이 클릭이 되었습니다.');
})

DOM.prototype.addEventListener = function(eventType,cbFunc) {
	if(eventType === 'click'){
		const clickEventObject = {
			target: {},
		}
	}

	cbFunc(clickEventObject);
}
```

- 그럼 callback function으로 어떻게 소스코드를 간결하게 만들 수 있을까?
```javascript
function register() {
	const isConfirm = confirm('회원가입에 성공했습니다.');

	if(isConfirm) {
		redirectUserInfoPage();
	}
}

function login() {
	const isConfirm = confirm("로그인에 성공했습니다.");

	if(isConfirm) {
		redirectIndexPage();
	}
}
```

- message와 callbackfn을 넘겨주는 형태로 바꾼다.
```javascript
function confirmModal(message, callbackFn) {
	const isConfirm = confirm(message);

	if(isConfirm && callbackFn){
		callbackFn();
	}
}
function register() {
	confirmModal('회원가입에 성공했습니다.', redirectIndexPage) //callback은 함수를 실행시켜 넘기지 않고 그 자체를 넘긴다. redirectIndexPage()로 넘기면 절대 안된다. 넘어가면 그 넘어간 후에 실행되는 것이다.
}

function login() {
	confirmModal('로그인에 성공했습니다.', redirectIndexPage)
}
```


## <span style="color:#802548">_14. 순수함수_</span>	
```javascript
let num1 = 10;
let num2 = 20;
function impureSum1() {
	return num1 + num2;
}

function impureSum2(newNum) {
	return num1 + newNum;
}

impureSum1(); //30

num1 = 100;
impureSum2(30); //130. 누가 저 변수를 변경하면 값이 변화해버림.

function pureSum(num1, num2) { //인자로 넣어서 외부 변화를 신경쓰지 않고 진행하게 변경. 이건 원시형에만 해당
	return num1 + num2;
}

pureSum(10, 20); //30
num1 = 100;
pureSum(10, 20); //30
num1 = 50;
pureSum(10, 20); //30
```

- 객체의 property는 parameter로 받고 조작하면 바뀐다.
- 따라서 아예 새로운 객체를 return해야한다.
```javascript
const obj = { one: 1};
function changeObj(targetObj) { //객체는 원시형처럼 바꾼다고 하면 바뀌어 버림.
	targetObj.one = 100;

	return targetObj;
}
changeObj(obj); 
console.log(obj); // {one: 100}

function makeNewObj(targetObj) { //새로운 객체를 반환하게 변경
	return {...targetObj, one: 100};
}

makeNewObj(obj); //{one: 100}
console.log(obj); //{one: 1}
```

## <span style="color:#802548">_15. 클로저_</span>
- 클로저는 외부의 context를 기억한다. parameter도 기억한다는 뜻이다.
```javascript
function add(num1) {
	return function sum(num2) {
		return num1 + num2;
	}
}

const addOne = add(1);
const addTwo = add(2);

addOne(3); //add(1)으로 add()에 1을 넘긴 상태에서 3을 sum()에 넣은 것. 4
add(1)(3); //addOne(3)과 동일
```

- 3개로 겹쳐있어도 closure는 잘 돌아간다.
```javascript
function add(num1) {
	return function (num2) {
		return function (calculateFn) {
			return calculateFn(num1, num2);
		}
	}
}

function sum(num1, num2) {
	return num1 + num2;
}

function multiple(num1, num2) {
	return num1 * num2;
}
 
const addOne = add(1)(2);
const addTwo = add(5)(2);
const sumAddOne = addOne(sum); //3
const sumAddOne = add(1)(2)(sum); //3
const sumMultipleOne = addOne(multiple); //2
const sumAddTwo = addTwo(sum); //7
const sumMultipleTwo = addTwo(multiple); //10
const sumMultipleTwo = add(5)(2)(multiple); //10
```

- if문 떡칠 혹은 switch문을 아래와 같이 closure를 활용해 바꿀 수도 있다.
```javascript
function log(value, level) {
	switch(level) {
		case "log":
			console.log(value);
			break;
		case "info":
			console.info(value);
			break;
		case "warn":
			console.warn(value);
			break;
		case "error":
			console.error(value);
			break;
	}
}


function log(value) {
	return function (fn) {
		fn(value);
	}
}

const logFoo = log('foo');

logFoo((v) => console.log(v));
logFoo((v) => console.info(v));
logFoo((v) => console.warn(v));
logFoo((v) => console.error(v));
```

- closure를 활용하는 것은 고차함수에도 쓰일 수 있다.
```javascript
const arr = [1,2,3,'A','B','C'];
const isNumber = (value) => typeof value === 'number';
const isString = (value) => typeof value === 'string';
arr.filter(isNumber);

function isTypeOf(type, value) {
	return typeof value === type;
}
const isNumber = (value) => isTypeOf('number', value);
const isString = (value) => isTypeOf('string', value);
arr.filter(isNumber);

function isTypeOf(type) {
	return function (value) {
		return typeof value === type;
	}
}
function isTypeOf(type) {
	return (value) => typeof value === type;
}
const isNumber = isTypeOf('number');
const isString = isTypeOf('string');
arr.filter(isNumber); //[1,2,3]
arr.filter(isString);//['A','B','C']
```

- fetch api에서도 쓰일 수 있다.
```javascript
function fetcher(endpoint) {
	return function(url, options) {
		return fetch(endpoint + url, options)
			.then((res) => {
				if(res.ok) {
					return res.json();
				} else {
					throw new Error(res.error);
				}
			})
			.catch((err) => console.log(err))
	}
}

const naverApi = fetcher('http://naver.com');
const daumApi = fetcher('http://daum.net');
 
 daumApi('/webtoon').then((res) => res);
 nvaerApi('/webtoon').then((res) => res);
```

- DOM eventListener에도 쓰일 수 있다.
```javascript
someElement.addEventListener('click',debounce(handleClick, 500));
someElement.addEventListener('click',throttle(handleClick, 500));
```

