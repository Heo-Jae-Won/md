## <span style="color:#802548">property manipulation prevention</span>
- 생성 이후에 객체의 property 조작을 막으려면 특정한 method를 써야한다.
- 보통 객체조작이 불가능하게 바꿀 때는 lodash library의 deepFreeze()를 사용하는 것이 추천된다.
- 직접 구현도 가능하다. 아래는 인터넷에서 가져온 deepFreeze의 구현 사례다. 
- 구현에 쓰인 method가 궁금하다면 JS의 prototype에 대해서 알아보도록 하자.


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
```

- 객체의 property 조작을 막는 이유 중 하나는 enum처럼 활용하기 위해서이다.
- 아래와 같이 얼려서 조작 불가능하게 만들어 enum으로 쓸 수 있다.

- 그럼 too long이 console에 찍히는 것을 볼 수 있다.


```javascript
const BANNER_LENGTH = Object.freeze({
    MIN: 1,
	  MAX: 5,
});
const bannerName       = "GENESIS"
const bannerNameLength = bannerName.length;
if (bannerNameLength > BANNER_LENGTH.MAX) {
    console.log('too long')
}
```
- typescript에서는 더 간단하게 만들 수 있다. 이 외에도 ts에서는 enum, const enum을 사용할 수 있다.


```javascript
const TICKET_PRICE = {
  ADULT: 10_000,
  CHILD: 5_000,
  ELDER: 7_000,
} as const;
```
- JS에서는 지원되지 않는다. Unexpected identifier 'as' error가 뜬다.

- JS 엔진이 {}뒤에 자동으로 ;를 삽입해준 뒤에 as를 식별자(변수명)로 해석하려 했기 때문이다.


## <span style="color:#802548">data type</span>
- 여태까지 string 원시형 값을 할당할 떄는 ""를 사용했다. 
- ""와 같은 값을 넣기 위한 특정한 형식을 literal이라 한다. 
- 하지만 생성자를 사용할 수도 있다. 다만 꼭 new 연산자가 필수인 것은 아니다.
- 오히려 원시형을 만들고 싶다면 new를 붙여서는 안된다. new를 붙이면 원시형이 아니라 객체의 instance가 된다.

```javascript
const str = String("B");    //생성자 방식
const str = "B";            //literal 방식
const str = 'B';            //literal 방식
console.log(str);           // 모두 "B"라는 원시형 값
```


- new가 필요한 경우도 있다. 원시형은 객체의 property를 활용할 수 없기 때문이다.
- 그러나 원시형이 instance properties에 접근하는 경우, JS엔진이 객체로 wrapping해준다. 따라서 소스코드로 우리가 쓸 필요는 없다. Java에서 쓰이는 일종의 autoBoxing과 비슷한 것으로 보인다.


```javascript
const str = "str"; //원시형. property없음.
console.log(str.length); //new String("str").length;처럼 wrapping됨. 3

const num = 5;
console.log(!num.isNaN); //true
```

- switch문은 원시형 문자를 받기 때문에 아래와 같이 하면 console에 찍히지 않는다.
- 아래에서 new를 지워주면 콘솔에 B가 찍히는 것을 볼 수 있다


```javascript
const str = new String('B');

switch(str) {
  case "A":
    console.log('A');
  break;
  case "B":
    console.log('B');
  break;
  case "C":
      console.log('C');
    break;
  case "D":
      console.log('D');
    break;
  default:
}
```


## <span style="color:#802548">object형 conditional statement</span>
- 만약 요구사항이 계속 바뀌어 추가될 거 같다면 object로 변경하여 관리할 수도 있다.
- 그럼 if문이나 switch문을 늘리지 않고 객체의 property만 늘려나가면 된다.

```javascript
function getUserType(type){
	const USER_TYPE = {
		ADMIN: '관리자',
		INSTRUCTOR: '강사',
		STUDENT: '수강생'
	}

	return USER_TYPE[type] ?? '해당 없음';
}

const userType = getUserType("ADMIN");
```

- 객체도 따로 파일로 빼서 관리한다면 전체 소스코드를 수정할 필요 없이 해당 객체의 property만 수정하면 전체 코드에 적용된다.


```javascript
import USER_TYPE from './constants./...'; //USER_TYPE을 바꿔주면 import하는 모든 js에 적용
function getUserType(type){
	return USER_TYPE[type] ?? USER_TYPE.UN
}
```

# <span style="color:#802548">_객체 literal을 활용한 if문 줄이기_</span>
- if문을 제거해보자.


```js
function executePayment(paymentType) {
    if (paymentType === "KAKAO_PAYMENT") {
        return '카카오 결제 처리';
    } else if (paymentType === "NAVER_PAYMENT") {
        return '네이버 결제 처리';
    } else if (paymentType === "COUPANG_PAYMENT") {
        return '쿠팡 결제 처리';
    } else if (paymentType === "PAYCO_PAYMENT") {
        return '페이코 결제 처리';
    } else if (paymentType === "APPLE_PAYMENT") {
        return '애플 결제 처리'
    }
}
```

- 복잡한 else if 문을 아래처럼 바꿔준다.
```js
const paymentMap = {
    "KAKAO_PAYMENT" : "카카오 결제처리",
    "NAVER_PAYMENT" : "쿠팡 결제 처리",
    "PAYCO_PATYMENT" : "페이코 결제 처리",
    "APPLE_PAYMENT" : '애플 결제 처리'
}

function executePayment(paymentType) {
    return paymentType[paymentType]
}
```

# <span style="color:#802548">_객체 literal의 method를 활용한 if문 줄이기_</span>
- 함수를 실행한다고 해도 동일하게 적용가능하다.


```js
function executePayment(paymentType) {
    if (paymentType === "KAKAO_PAYMENT") {
        payOnKakao();
    } else if (paymentType === "NAVER_PAYMENT") {
        sendLog();
        payOnNaver();
    } else if (paymentType === "COUPANG_PAYMENT") {
        sendLog();
        payOnCouPang();
    } else if (paymentType === "PAYCO_PAYMENT") {
        sendLog();
        payOnPayco();
    } else if (paymentType === "APPLE_PAYMENT") {
        sendLog();
        payOnApple();
    }
}
executePayment("APPLE_PAYMENT");
```


```js
const payment_map = {
    KAKAO_PAYMENT() {
        payOnKakao();
    },
    NAVER_PAYMENT() {
        sendLog();
        payOnNaver();
    },
    COUPANG_PAYMENT() {
        sendLog();
        payOnCouPang();  
    }, 
    PAYCO_PAYMENT() {
        sendLog();
        payOnPayco();
    }, 
    APPLE_PAYMENT() {
        sendLog();
        payOnApple();
    }
}

function executePayment(paymentType) {
    paymentMap[paymentType](); //반환이 함수니까 함수를 호출한 것.
}

executePayment('KAKAO_PAYMENT');
```

## <span style="color:#802548">고차함수로 loop statement 대체하기</span>
- 반복분에는 for문과 while문이 있다.
- 자주 쓰게 되는 것은 for문이다.

```javascript
let array = [1,2,3];
for (let i = 0; i < array.length; i++) {
  console.log(i);
}
```

- for문을 고차함수로 바꿔 쓰는 것도 가능하다.

- 그 편이 사람이 이해하기도 쉬우며, for, if문 조합으로 로직을 구성하는 것보다 편하다.


```javascript
let array = [1,2,3];
array.forEach(function(element) {
  console.log(element);
})
```

- 그에 관한 아래 사례를 살펴보자.

- for문과 if문의 조합으로 기능을 구현하려면 모두 자신이 직접 구현해야 한다.


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
		someAscendingSortFunc(temp); //sort
	}

	if(orderType === 'DESCENDING'){
		someDescendingSortFunc(temp);
	}
	
	return temp;
}

const intendedPriceList = getWonPrice(price);
```


- 고차함수의 힘을 빌리면 callback function만 넣어주면 쉽게 처리가 가능하다.
- 해당 기능은 js 내에서 이미 구현되어있어 빌리면 되는 것이다.


```javascript
const price = ['2000','1000','3000','5000','4000'];
const suffixWon = (price) => price + '원';
const isOverOneThousand = (price) => Number(price) > 1000;
const ascendingList = (a,b) => a - b;
function getWonPrice(priceList){
	return priceList
		.filter(isOverOneThousand)
		.sort(ascendingList)
		.map(suffixWon);
}

const intendedPriceList = getWonPrice(price); // [2000,3000,4000,5000]
```



# <span style="color:#802548">_비동기는 foreach에서 쓰면 안 된다._</span>
- 같은 반복문이어도 foreach와 for of문은 결과가 다르다.
- 아래 같이 for of 문을 쓰면 순서를 기다려 진행된다.

```js
const baseURL = "https://jsonplaceholder.typicode.com/todos";
const urls = [1,2,5,10];

(async function() {
    for (let url of urls) {
        console.log(`${url}요청 시작~`);
        const res = await fetch(baseURL + url);
        const result = await res.json();
        console.log(`${url}요청 끝`);
    }
    console.log("전체 요청 끝");
})();
/*
1 요청 시작~
1요청 끝~
2요청 시작~ 
2요청 끝
*/
```
- 반면에 foreach를 쓰면 한꺼번에 병렬로 fetch가 실행된다.
- map도 마찬가지다. 따라서 map과 foreach에서는 절대 비동기를 쓰면 안 된다.

```js
const baseURL = "https://jsonplaceholder.typicode.com/todos";
const urls = [1,2,5,10];

(async function() {
    urls.foreach(async (url) => {
        console.log(`${url}요청 시작~`);
        const res = await fetch(baseURL + url);
        const result = await res.json();
        console.log(`${url}요청 끝`);
    }) 
    console.log("전체 요청 끝");
})();
/*
1요청 시작~
2요청 시작~
5요청 시작~
10요청 시작~
전체 요청 끝~

1 요청 끝~
10요청 끝~
5요청 끝~
2요청 끝~
*/
```


# <span style="color:#802548">_고차함수로 for문과 if문을 줄이기_</span>
- for문과 if문을 줄여보자.
- 요구사항은 아래와 같다.

```
1. 0~100
2. 소수점 제거
3. '점' 문자 추가
4. 출력
```

```js
const grades = [80.55, 90, -95, -45, 44.3,  100, 177];

const validGrades = [];
for (let i = 0; i < grades.length; i++) {
    if (grades[i] >= 0 && grades[i] <=100) {
        const validGrade = Math.floor(grades[i]) + '점';
        validGrades.push(validGrade)
    }
}

for (let index = 0; index < validGrades.length; index++) {
    console.log(validGrades[index]);
}
```

- 고차함수를 활용하자.


```js
const validGrades = [];
const positiveGrades = grades.filter( el => el >= 0 && e. <=100);
const newGrades = positiveGrades.map( el => Math.floor(el) + '점');
newgrades.foreach( el => console.log(el));
```

- method chaining을 사용해보자.


```js
const validGrades = grades.filter( el => el >= 0 && e. <=100);
                            .map( el => Math.floor(el) + '점'); // 이걸 2개의 map으로 나눴다는 게 중요
                            .foreach( el => console.log(el));
```


- 이제는 로직에 이름을 부여했다.
- 로직에 이름을 주니 이해하기가 쉽다.


```js
const validScore = el => el >= 0 && e. <=100;
const toInteger = el => Math.floor(el);
const plusSuffix = el => el + '점';
const print = el => console.log(el);

const validGrades = grades.filter(validScore)
                            .map(toInteger)
                            .map(plusSuffix)
                            .foreach(print);
```

## <span style="color:#802548">_spread, destructuring문법_</span>
```
spread는 하나로 뭉쳐 있는 여러 값의 집합을 펼쳐서 개별 값의 목록으로 만든다.
destructuring은 값을 각 변수에 할당한다. 
iterable 객체만 spread 문법을 활용할 수 있다. destructuring은 객체면 사용 가능하다.
```

- 객체 리터럴로 선언한 객체는 iterable이 아니다. string, array, set, map 등이 iterable이다.

- iterable 객체가 되는 조건이 궁금하다면 iterable protocol에 관한 정보를 찾아보면 된다.


```javascript
console.log(...[1,2,3]); //1,2,3
console.log(...'hello');//h e l l o
console.log(...new Map([['a','1'],['b','2']])); // ['a','1'], ['b','2']
console.log(...new Set([1,2,3])); //1 2 3

console.log(...{a:1, b:2}); //error. Found non-callable @@iterator
```

- spread가 없었을 때는 아래와 같이 apply, call, bind 등을 활용해 method를 활용했다.


```javascript
var arr = [1,2,3];
var max = Math.max.apply(null, arr); //3
```

- 비구조화 할당을 사용하면 아래와 같이 바로 활용이 가능하다.


```javascript
const arr = [1,2,3];
const max  = Math.max(...arr); //3
```

- 이제 destructuring 문법의 사례를 살펴보자.

- 기존에는 아래와 같이 index로 받아왔어야 했다.


```javascript
var arr = [1,2,3];
var one = arr[0];
var two = arr[1];
var three = arr[2];
```

- 하지만 비구조화 할당을 활용하면 아래와 같이 간결하게 받아올 수 있다.


```javascript
const arr = [1,2,3];
const [one,two,three] = arr;
console.log(one,two,three); // 1 2 3
```

- 비구조화 할당은 기본값 적용이 가능하다.


```javascript
const [a,b,c=3] =[1,2];
console.log(a,b,c) ; //1 2 3. 기본값 적용가능

const [e,f = 10, g = 3] = [1,2];
console.log(e,f,g); // 1 2 3. 할당된 값이 기본값보다 우선
```

- 아래와 같이 쓰면 오류가 나거나 의도하지 않은 결과를 초래할 수 있다.


```javascript
const [a,b] = {} //TypeError: {} is not iterable
const [c,d] = 1;
console.log(c,d); //1 undefined
const [e,f] = [1,2,3];
console.log(e,f); //1 2. 3이 없다. 
```

- 아래는 배열의 destructuring의 사용예제다.


```javascript
function parseURL(url = ''){
    const parsedURL = url.match(/^(\w+):\/\/([^/]+)\/(.*)$/);

    if(!parsedURL) {
      return {};
    }

    const [url1, protocol,host,path] = parsedURL;
    return {
        url1,
        protocol,
        host,
        path
    }
}

const parsedURL = parseURL('https://developer.mozilla.org/ko/docs/Web/JavaScript');
console.log(parsedURL);
/*{
  "url1": "https://developer.mozilla.org/ko/docs/Web/JavaScript",
  "protocol": "https",
  "host": "developer.mozilla.org",
  "path": "ko/docs/Web/JavaScript"
}*/
```

- 이제는 객체의 destructuring의 예시를 살펴보자.
- ES5에서는 해당 문법이 없어 property에 접근할 시 .이나 []를 활용 한다.


```javascript
var user = {
    firstName: 'JaeWon',
    lastName: 'Lee'
}

var firstName = user.firstName;
var lastName = user['lastName'];
console.log(firstName, lastName);
```

- 하지만 ES6에서는 해당 문법을 사용하여 property에 접근할 수 있다.

- 변수를 할당할 때는 key를 기준으로 한다.


```javascript
const user = {
    firstName: 'JaeWon',
    lastName: 'Heo'
}
const {lastName, firstName} = user; //key를 기준으로 하기에, 순서를 뒤바꾸는 것은 아무 의미 없다. 아무 의미 없기에 바꿔 써도 상관 없다.
console.log(firstName, lastName) //JaeWon Heo
```

- 배열과 마찬가지로 기본값 설정도 가능하다.
- 다만 왼쪽에 기본값을 주는 것은 가능해도, 오른쪽에는 안된다.

- 하지만 아래와 같이 쓰면  오류가 난다.
- 오른쪽에는 정해진 값이 할당되어야 하기 때문이다.


```javascript
const {firstName = 'JaeWon', lastName} = {firstName, lastName = 'Heo'};
//Invalid shorthand property initializer
```

- =를 :로 바꿔 객체를 할당하는 올바른 초기화 token을 써주자.


```javascript
const {firstName = 'JaeWon', lastName} = {firstName, lastName : 'Heo'};
console.log(firstName, lastName) //JaeWon Heo
```

- 아래와 같이 별칭을 줄 수도 있다.

- 여기서는 기본값을 주는 게 아니라서 = 대신 :를 사용한다.


```javascript
const {lastName: ln, firstName: fn} = {firstName: 'JaeWon', lastName : 'Heo'};
console.log(fn,ln) //JaeWon Heo
```

- 함수의 parameter로 들어갈 때도 비구조화 할당이 가능하다.


```javascript
function printTodo({content, completed}){
    console.log(`할일 ${content} ${completed ? '완료' : '비완료'} 상태입니다.`);
}
printTodo({id:1, content: 'HTML', completed:true}); //할일 HTML 완료 상태입니다.
```

- nested property를 parameter로 하여 가져올 때도 비구조화 할당으로 가져올 수 있다.


```javascript
const cats = {
  title: "Cats",
  director: "Tom Hooper",
  releaseYear: 2019,
  boxOffice: {
    budget: 95_000_000,
    grossUS: 27_166_770,
    grossWorldwide: 73_833_348,
  },
};

function getProfit({ boxOffice: { grossWorldwide, budget } }) {
  return grossWorldwide - budget;
}
```

- 비구조화 할당을 이용해서 아래와 같이 필수값은 받도록 검사할 수 있다.

- 필수값을 넘기지 않았다면 error를 던져주면 개발자가 더 안전하게 해당 함수를 사용할 수 있다.


```javascript
function createCar(name,{brand,color,type}){ 
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

createCar('GENESIS',{});
createCar('GENESIS',{brand: 'Hyundai'});
/*
  {
    "name": "GENESIS",
    "brand": undefined,
    "color": undefined,
    "type": undefined
  }
/*
/*
  {
    "name": "GENESIS",
    "brand": "Hyundai",
    "color": undefined,
    "type": undefined
  }
/*
```

- 중첩 객체의 property를 가져올 때는 위의 중첩된 객체를 parameter로 분해할 때와 동일하게 가져올 수 있다.


```javascript
const user = {
    name: 'Lee',
    address: {
        zipCode: '03068',
        city: 'Seoul'
    }
}

const {address:{city}} = user;
console.log(city) //Seoul
```

- spread와 destructuring은 같이 사용가능하다.


```javascript
const {x, ...rest} = {x: 1, y: 2, z: 3};
console.log(x, rest); //1 {y:2, z:3}
```

- 배열을 받는 것도 아래와 같이 구조분해할당을 이용할 수 있다.

```javascript
function operateTime(inputs, operators, is){//배열의 index로 접근
	inputs[0].split('').forEach((num) =>{
		cy.get('.digit').contains(num).click();
	})

	inputs[1].split('').forEach((num) =>{
		cy.get('.digit').contains(num).click();
	})
}

function operateTime(inputs, operators, is){//구조 분해 할당으로 접근
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
- 차례로 0번쨰, 1번쨰, 2번쨰 index로 인식된다.

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

- 배열을 구조분해 할당으로 받아오는 또다른 예시다.

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

- 객체의 일부인 배열도 당연히 destructuring이 가능하다.

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


## <span style="color:#802548">property로 function 쓰기</span>
```
property의 value로 함수가 들어가면 이를 method라고 칭한다.
```
- 위에 썼던 예시에서 getCompanyName이 바로 method다.


```javascript
const obj = {
  name : 'company',
  getCompanyName: function(someFunc) {
    someFunc
  }
}

const obj = {
  name : 'company',
  getCompanyName: (someFunc) => someFunc
}
```

- method는 아래와 같이 간결하게 쓸 수도 있다.
- 이를 concise method라고 한다.

```javascript
const obj = {
  name : 'company',
  getCompanyName(someFunc) {
    someFunc
  }
}
```

- method를 변수명에 할당해 일반함수로 만들어 호출할 수도 있다.
- 다만 이는 this와 연관되어 bug를 야기할 수 있다. this 부분에서 살펴보자.


```javascript
const person = {
    name: 'Lee',
    getName(){
        return this.name;
    }
}

const getName = person.getName;
console.log(getName()) // 일반 브라우저에선 ''. codepen에서 test하면 CodePen
```




## <span style="color:#802548">parameter</span>
```
parameter는 함수에 넘기는 추상적인 인자다.
반면에 argument는 실제로 넘겨지는 구체적인 값이다.
```
- 아래에서 name은 parameter고, argument는 openobject다.


```javascript
function getCompanyName(name) {
  console.log(name);
}

console.log('openobject');
```

- argument가 몇개가 들어올 지 모를 때는 ...으로 축약하여 사용가능하다.


```javascript
function getSomething(...args) {
  console.log(args); 
}

getSomething('4','5'); //4,5
getSomething('4','5','6'); //4,5,6
```

- parameter에는 기본값을 부여할 수 있다.
- 하지만 parameter에 기본값을 넣은 것으로는 falsy값이 들어오는 것을 막을 수 없다.
- 그럴 때 아래와 같이 || 연산자를 활용하여 기본값을 줄 수도 있다.

- falsy가 아니라 null, undefined만 거를 거라면 ??를 활용할 수도 있다.


```javascript
function sum(x = 1, y = 2){

	return x + y;
}

function sum(x,y){
	x = x || 1;
	y = y || 1;

	return x + y;
}

function sum(x,y){
	x = x ?? 1;
	y = y ?? 1;

	return x + y;
}
```

- 기본값 부여로도 충분치 않다면 아래와 같이 직접 타입검사를 실행할 수도 있다.


```javascript
var add = function(x,y) {
  if(typeof x !== 'number' || typeof y !=='number'){
    throw new TypeError("인수는 모두 숫자값이어야 합니다.");
  }
}
```

- typescript에서는 더 간단하게 해결이 가능하다.
- 아래와 같이 parameter가 들어올 type을 한정해줄 수 있다. 입력을 되돌리지는 못한다.


```javascript
var add = function(x: number,y: number) {

}
```
- 입력 자체를 되돌리고 싶다면 정규식을 사용할 수 있다.


```javascript
var validateInputNumber = function(char){
  const regex = /[^0-9]/gi;
  if( regex.test(char) ){
      char = char.replace(regex,'');
   }
  console.log(char)
}

validateInputNumber('b'); // ''
validateInputNumber('9'); //'9'
```

## <span style="color:#802548">this</span>
- this binding이란 식별자와 값을 연결하는 과정이다. 

```
this라는 식별자와 this가 가리킬 객체의 메모리 주소를 binding하는 것이다.
this binding은 일반함수, 메서드, 생성자함수, call-apply-bind라는 4가지 맥락에 따라 의미가 다르다.
```

- 일반함수의 경우 scope에 관계없이 무조건 this가 window에 binding된다.


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
      
      bar(); //일반함수로 호출됐으면 어디서든 window.
    }
}

obj.foo();
```

- property의 method도 일반함수의 형태라면 this가 window에 binding된다.


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

- callback 함수를 상위 스코프에 binding하려면 아래와 같은 방법으로 가능하다.


```javascript
var value = 1;
const obj = {
    value: 100,
    foo(){
        const that = this; //this를 다른 변수에 할당

        setTimeout(function(){
             console.log("callback's this: ",that); // obj
            console.log("callback's this.value: ", that.value) //100
        },100);
    }
}

obj.foo();
```

- 상위스코프에 연동하기 위해, apply, bind, call 등의 함수를 사용할 수도 있다.


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

- 또 다른 방법으로는 화살표함수를 사용하는 것이다.


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

- 참고로 프로퍼티에 규정할 함수 호출의 경우, 일반 함수처럼 쓰지 않는 게 좋다.
- 변수명에 할당하는 일반 함수로 바뀌게 되면 this가 window에 binding되는 것으로 바뀐다.


```javascript
const person = {
    name: 'Lee',
    getName(){
        return this.name;
    }
}

console.log(person.getName()) // person객체의 name을 참조한다.

const getName = person.getName;
console.log(getName()) // window객체의 name을 참조한다.
```

- this를 setTimeout같은 객체에 감싸게 되면 undefined가 뜨는 경우가 있다.

```js
const myObj = {
    name: 'frongt',
    info() {
        setTimeout(function(){
            console.log(this.time);
        },1000);
    }
}

myObj.info(); //undefined
```

- js에서 this는 실행 timing에 결정되서 그런 것이다.
- 따라서 this가 값이 있는 scope에서 지역변수로 저장하는 것도 방법이다.
```js
const myObj = {
    name: 'frongt',
    info() {
        const that = this; //여기서 this는 name값이 frongt다.
        setTimeout(function(){
            console.log(that.time);
        },1000);
    }
}

myObj.info(); //frongt
```


- 아니면 this 값을 특정한 형태로 강제하는 bind함수를 쓸 수도 있다.
- 그러면 실행타이밍이 아니라 확실한 고정된 값에 묶인다.
- function에 bind(this)를 하면 this가 myObj라는 객체에 묶인 새로운 함수를 반환한다.
```js
const myObj = {
    name: 'frongt',
    info() {
        setTimeout(function(){
            console.log(this.name);
        }.bind(this),1000);
    }
}

myObj.info(); //frongt
```


- 아니면 ES6를 활용한 화살표함수를 쓰면 된다. 사실 제일 간단한 해결법이다.
```js
const myObj = {
    name: 'frongt',
    info() {
        setTimeout(() => {
            console.log(this.name)
        },1000);
    }
}

myObj.info(); //frongt
```


## <span style="color:#802548">live와 static </span>
- 요소 노드의 static 상태는 attribute노드가 관리한다
- 요소 노드의 live 상태는 DOM property가 관리한다.
- 둘을 바꿔보면 차이점을 느낄 수 있다. 아래는 attribute노드를 바꾼 것이다.

- DOM property를 바꿔도 attribute node의 값은 변하지 않는다. 둘은 각자 개별로 관리된다는 의미이다.


```html
<!DOCTYPE html>
<html>
    <body>
        <input id="user" type="text" value="car">
    </body>
    <script>
        const $input = document.getElementById('user');

        $input.oninput = () => {
            console.log('value 프로퍼티 값', $input.value);
        }

        $input.value = 'foo';
        console.log('value 어트리뷰트 값', $input.getAttribute('value')); 
        // car. foo로 나오지 않음.
    </script>
</html>
```

- DOM을 가져오는 특정 method의 경우, live상태를 바꿀 수 있는 객체를 반환한다. 이 경우, 주의하여 다룰 필요가 있다.
- 아래 예시를 살펴보자. 의도한 것은 className이 전부 txt-red가 되는 것인데, 실제로 그렇지 않게 되는 버그가 발생한다.

- DOM property를 바꾸는 순간, getElementsByClassName()의 반환 값인 HTMLCollection이 실시간으로 변동해 for문의 index가 엉키게 된다.
- 1번이 txt-red로 바뀌면, 그 다음 2번이 바뀌어야 하겠지만, live객체라서 2번이 1번자리인 [0]으로 내려오기 때문이다.
- [1]이었어야 할 2번이 [0]이 되어 아래와 같은 방식으로는 접근이 불가능하다.


```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      .txt-blue { color: blue; }
      .txt-red { color: red; }
    </style>
  </head>
    <body>
        <ul class="list">  
          <li class="txt-blue">1번</li>  
          <li class="txt-blue">2번</li>  
          <li class="txt-blue">3번</li>
        </ul>  
        <script>
           const $li = document.getElementsByClassName('txt-blue'); 
           for (let i = 0; i < $li.length; i++) {  
              $li[i].className = 'txt-red';
           }
        </script>
        </body>
</html>
```

- 이러한 버그를 예방하려면 for문을 역순으로 끝에서부터 도는 것이 좋다.

- 아니면 배열로 만들어 live 상태가 바뀌지 않게끔 방지한 뒤 className을 바꾸면 된다.


```javascript
for(let i = $li.length -1; i >= 0; i--){
  $li[i].className = 'txt-red';
}

[...$li].forEach(item => { 
    $li[i].className = 'txt-red';
})
```

## <span style="color:#802548">innerHTML</span>
- innerHTML은 사용하지 말고, textContent나 insertAdjacentText를 사용하자.

- innerHTML은 xss공격에 취약하다. 파싱과정에서 그대로 실행될 수 있기 때문이다.
- 다만 textContent는 그 아래 자식 노드를 전부 지워버리고 text만 추가해주기 때문에 조심해서 사용해야 한다.


```html
<!DOCTYPE html>
<html>
    <body>
        <div id="foo">Hello</div>
    </body>
    <script>
        const $foo = document.getElementById('foo');

        console.log(document.getElementById('foo').textContent); //Hello
        document.getElementById('foo').textContent = 'Hi <span>there!</span>'; //HTML parsing이 되지 않기 때문에 xss공격에 안전.
        /*
        <html>
            <body>
            'Hi<span>there!</span>' //tag 달고 그대로 출력됨
            </body>
        </html>
        */
    </script>
</html>
```


- 자식노드를 살리면서 요소를 추가하고 싶다면 insertAdjacentHTML 사용하자.
- 위치는 beforebegin, afterbegin, beforeend, afterend으로 다양하게 넣을 수 있다. 


```html
<!DOCTYPE html>
<html>
    <body>
        <div id="foo">Hello</div>
    </body>
    <script>
        const $foo = document.getElementById('foo');

        $foo.insertAdjacentHTML('beforebegin','Hi <span>there!</span>'); 
        /*
        <html>
            <body>
            Hi there!
            Hello
            </body>
        </html>
        */
    </script>
</html>
```

## <span style="color:#802548">data property</span>
- 과거에는 자신이 원하는 형식의 attribute를 사용할 때 아래와 같이 사용했다. 


```html
<!DOCTYPE html>
<html>
    <body>
        <div id="foo" mapId="mapId">Hello</div>
    </body>
    <script>
        const $foo = document.getElementById('foo').getAttribute("mapId"); 
        //$("#foo").attr("mapId");와 같이 Jquery로도 쓸 수 있다.
        console.log($foo); //mapId
    </script>
</html>
```

- 이제는 규격화되어 html에 내가 정의한 attribute를 넣고 싶다면 data-를 활용한다.

- data-로 시작하는 kebab case로 html에 넣고, js로 가져올 때는 dataset.camelCase로 가져온다.


```html
<!DOCTYPE html>
<html>
    <body>
        <div id="foo" data-map-id="mapId">Hello</div>
    </body>
    <script>
        const $foo = document.getElementById('foo').dataset.mapId;
        console.log($foo); //mapId
    </script>
</html>
```

## <span style="color:#802548">eventListener 등록시 이름 있는 함수로</span>
- 이벤트 핸들러를 등록할 때 바닐라 js에서 가장 많이 쓰이는 방식으로 callback fn 형태로 받는다.
- 함수가 형태가 완전히 똑같아도, 함수를 내부에서 익명으로 선언하면 새로운 함수로 취급되어 메모리상에서는 서로 다른 주소를 가진 별개의 객체가 된다.
- 따라서 event를 등록할 때는 반드시 이름이 있는 함수로 등록해주자.

```javascript
$button.addEventListener('click',function() {
    console.log('button click1');
})
const onClick = () => {
    console.log('button click2')
}

$button.addEventListener('click',onClick)
$button.removeEventListener('click', function() {
    console.log('button click1')
}) //변수로 callback을 넣는 게 아니면 제거 불가.

$button.removeEventListener('click',onClick); //변수로 callback을 넣어서 제거 가능
```

## <span style="color:#802548">bubbling</span>
- 이벤트 phase는 캡처링, 타깃, 버블링이 있다.
- 캡처링은 window -> document -> html -> body -> ul -> li로 event가 내려오는 과정이다.
- 타깃은 event target에 이벤트가 실제 일어나는 단계다.
- 버블링은 다시 li -> ul -> body -> html -> document -> window로 올라가는 과정이다.

- 버블링의 특성을 이용하면 이벤트 핸들러를 많이 등록하지 않고 똑같이 event를 걸 수 있다.
- 아래는 버블링의 특성을 이용하지 않고 이벤트 핸들러를 걸었다.
- 따라서 eventListener를 모든 li마다 등록해야한다.


```html
<!DOCTYPE html>
<html>
    <body>
        <nav>
            <ul id="fruits">
                <li id="apple" class="active">Apple</li>
                <li id="banana">Banana</li>
                <li id="orange">Orange</li>
            </ul>
        </nav>
        <div>선택된 내비게이션 아이템:
            <em class="msg">apple</em>
        </div>
        <script>
            const $fruits = document.getElementById('fruits');
            const $msg = document.querySelector('.msg');

            function activate({target}) {
                [...$fruits.children].forEach($fruit => {
                    $fruit.classList.toggle('active', $fruit === target);
                    $msg.textContent = target.id;
                })
            }

            document.getElementById('apple').onclick = activate; //nav마다 eventListener 등록..
            document.getElementById('banana').onclick = activate;//nav마다 eventListener 등록..
            document.getElementById('orange').onclick = activate;//nav마다 eventListener 등록..
        </script>
    </body>
</html>
```

- 이제 이벤트 위임을 사용해 중복을 제거해보자.
- 버블링을 이용하면 상위 DOM에 걸면 하위 DOM에는 걸 필요가 없다.

- 하위 DOM에도 event를 걸면 둘다 발동된다. 
- span은 li가 아니라 원하는 target이 아니므로 무시한다.


```html
<!DOCTYPE html>
<html>
    <body>
        <nav>
            <ul id="fruits">
                <li id="apple" class="active">Apple</li>
                <li id="banana">Banana</li>
                <li id="orange">Orange</li>
                <span>hi</span>
            </ul>
        </nav>
        <div>선택된 내비게이션 아이템:
            <em class="msg">apple</em>
        </div>
        <script>
            const $fruits = document.getElementById('fruits');
            const $banana = document.getElementById('banana');
            const $msg = document.querySelector('.msg');

            function activate({target}) {
                if(!target.matches('#fruits > li')){ //span은 무시
                    return;
                }

                [...$fruits.children].forEach($fruit => {
                    $fruit.classList.toggle('active', $fruit === target);
                    $msg.textContent = target.id;
                })
            }

            function bananaActivate() {
               console.log("event not ignored");
            }

            $fruits.addEventListener('click', activate);
            $banana.addEventListener('click', bananaActivate);
        </script>
    </body>
</html>
```

- 비슷한 예시를 살펴보자.
- li를 클릭 했을 때 해당 li의 text를 출력한다고 해보자.
- element마다 달아주면 일일이 달아줘야 한다.


```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      .txt-blue { color: blue; }
      .txt-red { color: red; }
    </style>
  </head>
    <body>
        <ul class="list">  
          <li class="txt-blue">1번</li>  
          <li class="txt-blue">2번</li>  
          <li class="txt-blue">3번</li>
          <span>hi</span>
        </ul>  
        <script>
          const $li = document.querySelectorAll('li'); 
          for(let i = 0; i < $li.length; i++){
             $li[i].addEventListener('click', function() {
              console.log(this.textContent)
           })
          }
          
        </script>
      </body>
</html>
```

- bubbling을 이용하면 아래와 같이 달아준다.
- 다만 this가 아니라 target이다. this로 하면 ul이 this라 원하는 대로 나오지 않는다.


```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      .txt-blue { color: blue; }
      .txt-red { color: red; }
    </style>
  </head>
    <body>
        <ul class="list">  
          <li class="txt-blue">1번</li>  
          <li class="txt-blue">2번</li>  
          <li class="txt-blue">3번</li>
          <span>hi</span>
        </ul>  
        <script>
          const $li = document.querySelector('ul'); 
          $li.addEventListener('click', function({target}) {
            if(!target.matches('.list > li')){ //span은 무시된다.
                  return;
            }

            console.log(target.textContent) 
          })
          
        </script>
        </body>
</html>
```

- 버블링을 막는 방법도 있다.
- stopPropagation()는 이벤트 버블링을 중단시킨다.
- 따라서 이벤트 위임 방식으로는 eventListener가 작동하지 않는다.


```html
<!DOCTYPE html>
<html>
    <body>
        <div class="container">
            <button class='btn1'>Button 1</button>
            <button class='btn2'>Button 2</button>
            <button class='btn3'>Button 3</button>
        </div>
        <script>
            document.querySelector('.container').onclick = ({target}) => {
                if(!target.matches('.container > button')) { 
                    return;
                }
                target.style.color = 'red';
            }

            document.querySelector('.btn2').addEventListener('click',(e) => {
                e.stopPropagation();            
                // 버튼 중 btn2는 버블링 전파 불가.
                e.target.style.color = 'blue';  
                //따라서 red로 컬러가 바뀌지 않음. blue로 바뀜 
            })
        </script>
    </body>
</html>
```


## <span style="color:#802548">call stack, task queue, event loop</span>
- 우선 브라우저의 구성에 대해 이해할 필요가 있다.


```
JS 엔진은 단 하나의 call stack을 가진다. 
JS 엔진은 위와 같이 싱글 스레드로 동작하지만 브라우저는 멀티 스레드로 동작한다.
비동기함수(setTimeout, promise 등)의 callback함수는 call stack에 바로 들어가지 않고 event queue에 들어간다.
event queue에 들어간 callback함수는 call stack이 비어있을 때 event loop에 의해 call stack으로 불려와 호출된다.
push된 함수는 call stack에서 실행이 끝나면 pop된다.
```

- 코드로 이해하는 것이 빠를 것이다. 아래 코드를 보자.

- console에 찍힌 것은 step1, step2, step3이 아니다. step1, step3, step2다.


```javascript
function step1() {
    console.log('step1');
}

function step2() {
  setTimeout(function() {
    console.log('step2');
  },0);
}

function step3() {
    console.log('step3');
}

step1();
step2();
step3();
/*
step1
step3
step2
*/
```

- 그 원리를 설명하자면 아래와 같다.
- step1, step2, step3 순서대로 call stack에 push된다.
- step1이 가장 먼저 push되어 실행되고 pop된다.

- step2가 그 다음 call stack에 push된다. setTimeout이라는 함수가 실행되고, callback fn이 timer로 이동한다.
- step2가 call stack에서 사라졌으니 step3가 push된다.
- 4ms정도(0초라도 실제론 지연시간이 있다) 지난 step2의 callback함수가 event queue로 이동한다. step 3도 실행되어 pop된다.
- step3가 call stack에서 사라지고 나서 callback함수가 event loop에 의해 call stack으로 이동한다.
- callback 함수인 console.log('step2')가 실행되고 call stack에서 pop된다.

## <span style="color:#802548">setTimeout</span>

- rendering엔진은 JS엔진이 소스코드 평가-실행 상태인 경우 동작하지 않는다. 즉 call stack에 함수가 있으면 작동하지 않는다.
- setTimeout을 사용하게 되면 event queue에서 callback 함수가 call stack으로 이동하는 시간 동안 call stack이 비어있다.

- 그 때 rendering 엔진이 작동하여 UI 변화가 그려진다.
- 더 자세한 내용은 https://www.youtube.com/watch?v=8aGhZQkoFbQ를 참고하자.
- 쉽게 이해할 수 있는 예시를 들어보자.
- 아래와 같은 eventListener가 있다고 해보자.


```javascript
$(".btn").click(function() {
  showLoadingBar();
  takesManyTime();
  displayNoneLoadingBar();
  updateHTML();
});
```
- 안에서 다음과 같은 일이 벌어진다.


```
1. 로직이 오래 걸리는 함수인데, 해당 함수 이후에 UI가 변화해야한다.
2. 기다리는 시간이 오래걸리므로 loading bar를 띄우게끔 했다.
3. 하지만 예상과 다르게 loading bar가 떠오르지 않는다.
4. showLoadingBar 함수의 실행이 끝나고 렌더링 엔진이 렌더링을 하려해도,태스크 큐에서 takesManyTime()가 실행되고 있다. 
5. 렌더링 엔진은 JS엔진이 작동할 때 대기상태이므로 JS코드가 전부 실행될 때까지 기다리게 된다.
6. 그럼 takesManyTime이 끝나고,  displayNoneLoadingBar가 끝나고, updateHTML가 끝난 뒤에야 rendering 엔진이 작동하며, 이 때는 loading bar가 이미 hide 되었다.
7. 따라서 repaint할 loading bar 관련 DOM 요소가 없다. JS에 의해 영향을 받은 DOM요소가 있어야 렌더링 엔진이 render가 이뤄지는데, 해당 시점에서 loading bar DOM요소가 없다.
8. 따라서 JS를 통해 text나 attribute가 변경된 DOM만 repaint가 이뤄진다. takesManyTime이후 변경되는 DOM들이 다시 rendering되어 화면이 출력된다. 거기엔 loading bar는 없다.
```
- 사라진 loading bar를 띄워보려면 어떻게 할까?
- 간단하게 아래와 같이 바꿀 수 있다.

- setTimeout을 이용하는 것이다.


```javascript
$(".btn").click(function() {
  showLoadingBar();
  setTimeout(function() {
    takesManyTime();
    displayNoneLoadingBar();
    updateHTML();
  }, 0);
});
```
- setTimeout을 적용하면 어떤 일이 벌어질까?


```
1. showLoadingBar가 호출된다.
2. 그 이후 setTimeout이 호출되어 call stack이 잠시 비게 된다.
3. 그동안 rendering 엔진이 UI를 repaint한다. 즉 loading bar가 나타난다.
4. 렌더링이 끝나고 event queue에 있던 3개의 함수가 차례로 call stack에 push된다.
5. takesManyTime가 호출되고, fetch가 끝나면 displayNoneLoadingBar가 호출되며 마지막으로 updateHTML이 호출된다.
6. 따라서 loading bar가 사라지고, 바뀐 data로 repaint하여 HTML이 렌더링된다.
```
- 위는 이해를 돕기 위한 간단한 예시고, 실제 구현을 살펴보자.
- 시작을 누르고 진행 확인을 누르면 반응이 없는 것을 볼 수 있다.

- 또한 percentage가 한번에 확 올라가버린다.


```html
<html>
	<head>
	<style>
		#progressBar { position: relative; width: 100%; height: 30px; 
           background-color: #ddd; }
		#progressPercent { position: absolute; width: 0%; height: 100%;
           background-color: #ff9933; }
	</style>
	</head>

	<body>
		<button type="button"  onclick="alert('실행중');">진행확인</button>
		<button type="button"  onclick="execute();">시작</button>

		<div id="progressBar" style="margin-top:20px;">
			<div id="progressPercent"></div>
		</div>
	</body>
	<script>
    let dataList = [[]];
    let availableDataList = [];
		let targetList = [1];
    let allCount = 100_000;
		let remainCount = 100_000 ;
		

    //longTakingLogic을 위해 data를 만든다
		function initData() {
			for (let i = 0; i < 10; i++) {
				dataList[i] = [];

				for (let j = 0; j < 2000; j++) {
					dataList[i][j] = (i + j);
				}
			}
		}

		// 오래 걸리는 작업
		function longTakingLogic() {
			for (let i = 0; i < targetList.length; i++) {
				for (let j = 0; j < dataList.length; j++) {
					for (let k = 0; k < dataList[j].length; k++) {
						if (dataList[j][k] == targetList[i]) {
							availableDataList.push(targetList[i]);
						}
					}
				}
			}
		}

    //버튼을 누르면 실행되는 작업
		function execute() {
			let progressElement = document.getElementById("progressPercent");
			let isCompleted;
			
			initData();
			
			for (let i = 0; i < allCount; i++) {
				longTakingLogic();
				remainCount = remainCount - 1;
				isCompleted = (i+1) % 10000 == 0;
				
				if (isCompleted) {
					// progress 표시
					progressElement.style.width = ((allCount - remainCount) / allCount) * 100 + '%';
				}
			}
		}

		</script>
</html>
```
- 왜 이런 일이 벌어지는지 로직을 살펴보자.


```
1. 시작을 누르면, execute()가 call stack에 push된다.
2. execute 안에서 initData()가 call stack에 push된다.
3. for문을 돌면서 longTakingLogic()이 push된다. if문의 로직이 실행되어 DOM property가 변경된다.
4. 하지만 곧바로 longTakingLogic()이 다시 push된다. JS엔진이 실행 중이라 if문의 로직이 실행되어 DOM property가 변경되어도 렌더링 엔진이 repaint할 수 없다. 
5. 맨 마지막에 isCompleted가 false가 되면 for문을 빠져나오고 execute()가 종료되고 call stack이 빈다.
6. 그럼 마지막 Dom property 조작만 rendering 엔진이 인식하여 repaint한다.
7. 따라서 한꺼번에 쫙 차오르게 된다. 또한 실행중 버튼을 눌러도 call stack이 차있어 event queue에서 event handler인 alert함수가 대기하게 된다.
8. 마찬가지로 call stack이 비게 되면 alert가 뜨게 된다. 따라서 repaint도, alert(실행중)도 execute()가 끝나기 전까지 작동하지 않는다.
9. 눌러놨던 숫자만큼 alert()가 call stack에 push되고 실행된다.
```

- 아래와 같이 setTimeout을 넣으면 의도한대로 작동하게 된다. 

- percentage가 계속 순차적으로 올라가고, 진행확인을 누르면 alert가 반응한다.


```javascript
//버튼을 누르면 실행되는 작업
function execute() {
    let progressElement = document.getElementById("progressPercent");
    let isCompleted;
    
    initData();
    
    for (let i = 0; i < allCount; i++) {
      //오래 걸리는 작업, UI를 변화시키는 DOM 조작을 모두 setTimeout에 집어넣음.
      setTimeout(function(){
        longTakingLogic();
         console.log(i);
    console.log(isCompleted);
        remainCount = remainCount - 1;
        isCompleted = (i+1) % 10000 == 0;
          progressElement.style.width = ((allCount - remainCount) / allCount) * 100 + '%';
      },0)
    }
  }
```

- 이건 왜 작동할까? 로직을 살펴보자.


```
1. 시작을 누르면, execute()가 call stack에 push된다.
2. execute 안에서 initData()가 call stack에 push된다.
3. for문을 돌면서 setTimeout()이 call stack에 push된다. 
4. setTimeout에 있는 callback function이 timer를 거쳐 event queue에 push된다.
5. 그동안 call stack이 비어있다. 따라서 변경된 DOM property에 맞게 렌더링 엔진이 repaint한다. 
6. rendering 엔진이 repaint가 끝나면 JS 엔진이 나머지 for문을 다시 돈다.
7. 이러한 과정이 반복된다.
```
- 아래와 같이 중간(별표주석자리)에 console.log를 넣어주면 UI가 update되는 속도가 현저히 느려진다. 

- console.log가 추가되면서 JS엔진이 더 오랜 시간을 점유하고 있기에 렌더링 엔진이 repaint하는 시점이 그만큼 느려진 것이다.


```javascript
function execute() {
    let progressElement = document.getElementById("progressPercent");
    let isCompleted;
    
    initData();
    for (let i = 0; i < allCount; i++) {
      setTimeout(function(){
        longTakingLogic();
        remainCount = remainCount - 1;
        isCompleted = (i+1) % 10000 == 0;
        progressElement.style.width = ((allCount - remainCount) / allCount) * 100 + '%';
      },0)
     /**/ console.log((allCount - remainCount) / allCount); 
    }
  }
```


- 아래와 같이 setTimeout에 감싸지 않고 promise로 바깥으로 뺄 수도 있다.
- 하지만 그렇게 하면 속도가 많이 느려진다.
- if문이 있으면 percent가 계속 바뀌지 않는다.

```js
 function delay(time) { 
      return new Promise((resolve) => setTimeout(()=> resolve(),time)); 
    }
    //버튼을 누르면 실행되는 작업
		async function execute() {
      let progressElement = document.getElementById("progressPercent");
      let isCompleted;

      initData();

      for (let i = 0; i < allCount; i++) {
        //오래 걸리는 작업, UI를 변화시키는 DOM 조작을 모두 setTimeout에 집어넣음.
        await delay(0);
        paintElement(progressElement,i);
      }
  }

  function paintElement(progressElement,i){
        longTakingLogic();
        remainCount = remainCount - 1;
        progressElement.style.width = ((allCount - remainCount) / allCount) * 1000 + '%';
    }
```

## <span style="color:#802548">debounce와 throttle</span> 
- setTimeout을 활용해 event handler를 덜 호출하게 만들 수 있다.
- 아래와 같이 input이 바뀔 때마다 text가 바뀌게끔 해보자.

- 그럼 매번 input이 바뀔 때마다 browser가 repaint하여 rendering이 이뤄진다.
- 이렇게 event handler가 과도하게 호출되면 성능 손실로 이뤄진다.


```html
<!DOCTYPE html>
<html>
  <body>
    <input type="text">
    <div class= "msg"></div>
    <script>
      const $input = document.querySelector('input');
      const $msg = document.querySelector('.msg');

      const debounce = (callback, delay) => {
        let timerId;
        return (...args) => {
          if (timerId) {
            clearTimeout(timerId);
          }
          timerId = setTimeout(callback, delay, ...args);
        }
      }

      $input.addEventListener('input', e => {
        $msg.textContent = e.target.value;
      },300)
    </script>
  </body>
</html>
```

- scroll, input, mousemove가 특히 그러하다. 이 경우에, 일정 시간 경과 이후 event handler를 호출하게끔 할 수있다. 이것을 debounce라고 한다.

- 0.3 초 내에 입력이 다시 이뤄지면 다시 timer를 등록하고, 0.3초가 지나도록 입력이 없으면 event handler가 호출된다.


```html
<!DOCTYPE html>
<html>
  <body>
    <input type="text">
    <div class= "msg"></div>
    <script>
      const $input = document.querySelector('input');
      const $msg = document.querySelector('.msg');

      const debounce = (callback, delay) => {
        let timerId;
        return (...args) => {
          if (timerId) {
            clearTimeout(timerId);
          }
          timerId = setTimeout(callback, delay, ...args);
        }
      }

      $input.addEventListener('input', debounce(e => {
        $msg.textContent = e.target.value;
      },300))
    </script>
  </body>
</html>
```

- 아래는 input이 아닌 scroll의 예시다.
- scroll은 delay 시간이 경과했을 때 event가 발생하면 callback 함수를 호출하고 다시 timer를 재설정한다.

- throttle이 없는 함수에서는 delay 시간 간격이 아니라 매번 event handler가 호출된다.
- throttle이 있는 함수에서는 delay 시간 간격마다 event handler가 호출된다.


```html
<!DOCTYPE html>
<html>
  <head>
    <style>
      .container {
        width: 300px;
        height: 300px;
        background-color: rebeccapurple;
        overflow: scroll;
      }

      .content {
        width: 300px;
        height: 1000vh;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <div class="content"></div>
    </div>
    <div>
      일반 이벤트 핸들러가 scroll 이벤트를 처리한 횟수:
      <span class="normal-count">0</span>
    </div>
    <div>
      스로틀 이벤트 핸들러가 scroll 이벤트를 처리한 횟수:
      <span class="throttle-count">0</span>
    </div>

    <script>
      const $container = document.querySelector('.container');
      const $normalCount = document.querySelector('.normal-count');
      const $throttleCount = document.querySelector('.throttle-count');

      const throttle = (callback, delay) => {
        let timerId;

        return (...args) => {
          if (timerId) {
            return;
          }

          timerId = setTimeout(()=> {
            callback(...args);
            timerId = null;
          }, delay);
        };
      };

      let normalCount = 0;
      $container.addEventListener('scroll',() => {
        $normalCount.textContent = ++normalCount;
      })

      let throttleCount = 0;
      $container.addEventListener('scroll', throttle(() => {
        $throttleCount.textContent = ++throttleCount;
      },100))
    </script>
  </body>
</html>
```

- 다만 실무에서는 직접 구현보다는 lodash의 debounce()와 throttle()을 활용하는 게 더 좋다고 한다. 


## <span style="color:#802548">주의점</span>
- setTimeout에서 주의할 점은 try ~ catch로 감싸면 안 된다는 점이다.
- setTimeout은 비동기 함수라서 콜백함수가 호출되는 것을 기다리지 않고 종료된다.
- setTimeout의 callback 함수는 event queue로 push되어 call stack으로 브라우저에 의해 push된다.

- 따라서 callback 함수의 호출자가 없고, 콜백함수에서 난 error가 setTimeout()으로 전파될 수도 없다. 이미 종료되어 call stack에서 pop되었기 때문이다. 
- 다시 말해 상위 호출자인 setTimeout으로 error가 전파되지 않아 error가 catch되지 않는다.
- catch되지 않은 error가 있어 프로그램이 강제 종료된다.


```javascript
try { 
    setTimeout(() => {
        throw new Error('Error!');
    }, 1000)
} catch(err) {
    console.error('캐치한 에러',err); //catch 되지 않는다. 따라서 console에 찍히지 않는다.
}
```



## <span style="color:#802548">_임시객체 없애기_</span>
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

- 임시 객체를 없애는 다른 예시다.

```javascript
function getDateTime(targetDate){
	let month = targetDate.getMonth();
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
```

- overloading을 활용해 유연한 코드를 만들 수도 있다.
- 위에는 argument가 있지만, 여기는 없다. 없으면 기본으로 현재일자가 들어간다.

```js
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


## <span style="color:#802548">_type- 명시적으로 형변환하기_</span>
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



## <span style="color:#802548">_Number.isNaN_</span>
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


## <span style="color:#802548">_falsy operator_</span>
- 기본값을 주는 또다른 예시다.

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

- 아래는 여러 연산자들을 활용해 중첩 if문을 제거하는 사례다.

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


## <span style="color:#802548">_objet- shorthand property_</span>
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

## <span style="color:#802548">_object- property check_</span>
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


## <span style="color:#802548">_추상화_</span>
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
	const {newEl, newEl2,newEl3} = createLoaderStyle({el, el2, el3});

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


## <span style="color:#802548">_HTML- NodeList_</span>
- Node는 document 내의 모든 객체다.
- elements는 tag에 둘러 싸인 객체다.
- html, body, main, table, thead, th 모두 node다.
- NodeList는 일반 배열로 형변환해주는 게 좋다.

```javascript
const arr = document.querySelectorAll('a'); //NodeList라서 배열의 method는 사용불가. entries(), forEach(), item(), keys(), values()만 존재
const arrFromNode = [...arr];//일반 배열이라서 배열의 method 사용가능.
```


## <span style="color:#802548">_function- pure function, parameter use_</span>
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

- 순수함수의 또다른 예시다.

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

- 불변값으로 return해주는 것도 매우 좋은 선택이다.

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


## <span style="color:#802548">_function- arrow function_</span>
- 화살표함수는 class를 만들 때도 거의 쓰지 않는다.
- 정상 작동이 되지 않는다.

```js
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
```


- concise method로 활용해야 한다.

```js
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



