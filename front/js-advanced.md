## <span style="color:#802548">_spread, destructuring문법- object_</span>
- spread는 하나로 뭉쳐 있는 여러 값의 집합을 펼쳐서 개별 값의 목록으로 만든다.
- destructuring은 값을 각 변수에 할당한다. 
- iterable 객체만 spread 문법을 활용할 수 있다. destructuring은 객체면 사용 가능하다.
- 객체 리터럴로 선언한 객체는 iterable이 아니다. string, array, set, map 등이 iterable이다.
- iterable 객체가 되는 조건이 궁금하다면 iterable protocol에 관한 정보를 찾아보면 된다.


```javascript
console.log(...[1,2,3]); //1,2,3
console.log(...'hello');//h e l l o
console.log(...new Map([['a','1'],['b','2']])); // ['a','1'], ['b','2']
console.log(...new Set([1,2,3])); //1 2 3

console.log(...{a:1, b:2}); //error. Found non-callable @@iterator
```

- ES6에서는 해당 문법을 사용하여 property에 접근할 수 있다.
- 변수를 할당할 때는 key를 기준으로 한다.


```javascript
const user = {
    firstName: 'JaeWon',
    lastName: 'Heo'
}
const {lastName, firstName} = user; //key를 기준으로 하기에, 순서를 뒤바꾸는 것은 아무 의미 없다. 아무 의미 없기에 바꿔 써도 상관 없다.
console.log(firstName, lastName) //JaeWon Heo


//past
var user = {
    firstName: 'JaeWon',
    lastName: 'Lee'
}

var firstName = user.firstName;
var lastName = user['lastName'];
console.log(firstName, lastName);
```

- 배열과 마찬가지로 기본값 설정도 가능하다.
- 왼쪽과 오른쪽이 = 와 : 로 기호가 다른 것에 주의하자.

```javascript
const {firstName = 'JaeWon', lastName} = {firstName, lastName : 'Heo'};
console.log(firstName, lastName) //JaeWon Heo
```


- 아래와 같이 별칭을 줄 수도 있다.
- 여기서는 기본값을 주는 게 아니라서 = 대신 :를 사용한다.

```javascript
const {lastName: ln, firstName: fn} = {firstName, lastName};
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

## <span style="color:#802548">_spread, destructuring문법- array_</span>
- 비구조화 할당을 활용하면 아래와 같이 간결하게 받아올 수 있다.

```javascript
const arr = [1,2,3];
const [one,two,three] = arr;
console.log(one,two,three); // 1 2 3

//past
var arr = [1,2,3];
var one = arr[0];
var two = arr[1];
var three = arr[2];
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


## <span style="color:#802548">_spread와 destructuring을 이용해 임시객체 없애기_</span>
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

## <span style="color:#802548">_엔간하면 객체는 새로 복사하기_</span>
- Node는 document 내의 모든 객체다.
- elements는 tag에 둘러 싸인 객체다.
- html, body, main, table, thead, th 모두 node다.
- NodeList는 일반 배열로 형변환해주는 게 좋다.

```javascript
const arr = document.querySelectorAll('a'); //NodeList라서 배열의 method는 사용불가. entries(), forEach(), item(), keys(), values()만 존재
const arrFromNode = [...arr];//일반 배열이라서 배열의 method 사용가능.
```

- 불변값으로 return해주는 것도 매우 좋은 선택이다.
- 그럴 때 destructuring이나 spread를 이용한다.

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


## <span style="color:#802548">객체의 property로서의 fucntion - method</span>
- 아래에 썼던 예시에서 getCompanyName이 바로 method다.
- 일반함수의 경우는 this가 window, 화살표 함수의 경우에는 this가 자기 자신이다.

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
- 이 경우는 화살표 함수처럼, this가 자기 자신이다.

```javascript
const obj = {
  name : 'company',
  getCompanyName(someFunc) {
    someFunc
  }
}
```

- method를 변수명에 할당해 일반함수로 만들어 호출할 수도 있다.
- 다만 이는 this와 연관되어 bug를 야기할 수 있다. 
- consie method의 경우, 분리해서 따로 만들면 일반 함수가 되어 this가 window를 가리키기 때문이다.


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

## <span style="color:#802548">this</span>
- this binding이란 식별자와 값을 연결하는 과정이다. 
- this라는 식별자와 this가 가리킬 객체의 메모리 주소를 binding하는 것이다.
- this binding은 일반함수, 메서드, 생성자함수, call-apply-bind라는 4가지 맥락에 따라 의미가 다르다.
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

- property의 method도 비동기 callback안이라면 this가 window에 binding된다.
- settimeout은 비동기콜백이므로, this가 window다.

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

const getName = person.getName; //일반 함수화됨
console.log(getName()) // window객체의 name을 참조한다.
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

- 그러한 종류가 아래처럼 존재한다.
- 이 것들을 다룰 때는 반드시 복사해주자. 아니면 querySelectorAll()를 쓰는 것을 추천한다.

```
HTMLCollectiondocument.getElementsByClassName()
document.getElementsByTagName()
element.childrenNodeList
element.childNodes
NamedNodeMapelement.attributes
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
                            .map( el => Math.floor(el) + '점'); 
                            .foreach( el => console.log(el));
```


- 이제는 로직에 이름을 부여했다.
- 로직에 이름을 주니 이해하기가 쉽다.
- 이해하기 쉽게 SRP를 따라서 map도 2개로 나눠준다.

```js
const validScore = el => el >= 0 && e. <=100;
const toInteger = el => Math.floor(el);
const plusSuffix = el => el + '점';
const print = el => console.log(el);

const validGrades = grades.filter(validScore)
                            .map(toInteger) // 이걸 2개의 map으로 나눴다는 게 중요
                            .map(plusSuffix)
                            .foreach(print);
```


## <span style="color:#802548">_고차함수로 event parameter 대체하기_</span>
- 이를 이용하면 event를 이용한 handler를 쓸 때도 parameter를 번잡하게 쓸 필요가 없다.
- 아래는 일반 함수 정의다.

```js
const handler = (e,id) => {
    console.log(`${id} 클릭함 ${e.offsetX}`);
}

document.addEventListener('click',(e) => handler(e,'frongt'));
```


- 잃반 함수 정의가 아니라, 함수를 return하는 함수를 정의한다.


```js
const handler = id => e => {
     console.log(`${id} 클릭함 ${e.offsetX}`);
}

document.addEventListener('click', handler('frongt'));
```


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


# <span style="color:#802548">_setTimeoue을 Promise로 만들기_</span>
- refactoring해보자. 아래는 원본이다.

```js
function orderCoffee(el, orderList) {
    if (el) {
        if (Array.isArray(orderList)) {
            el.addEventListener('click',function () {
                setTimeout(function() {
                    for(let i = 0; i < orderList.length; i++) {
                        document.querySelector('#log').innerHTML += `${orderList[i]}가 완료됐습니다.<br/>`;
                    }
                },2000);
            })
        }
    }
}
```


- 아래는 내가 한 것이다.

```js
function orderCoffee(el, orderList) {
    if (el && Array.isArray(orderList)) {
        el.addEventListener('click', notifyOrderSucccess(orderList))
    }
}

function notifyOrderSucccess(orderList) {
    setTimeout(function() { 
        for(const order of orderList) {
            document.querySelector('#log').textContent += `${order}가 완료됐습니다.<br/>`;
        }       
    },2000);
}
```


- 아래는 프롱트에서 보여준 것이다.

```js
function delay(time) { //setTimeout을 한줄로 간단하게 나눠주기 위한 함수..
    return new Promise((resolve) => setTimeout(()=> resolve(),time)); // () => resolve()는 익명함수를 정의한 것과 동일하다. 다시말해 time만큼이 지나면 그냥 resolve()가 된다. 완료된다는 의미다.
}

async function notifyOrderSucccess(orderList) {
    await delay(2000);
    orderList.forEach((order) => document.querySelector('#log').textContent += `${order}가 완료됐습니다.<br/>`); // for문을 한줄로 줄이기 위한 고차함수
}

function orderCoffee(el, orderList) {
    if(!el || !Array.isArray(orderList)) { // early return
        return;
    }

    el.addEventListener('click', () => notifyOrderSucccess(orderList)) //parameter를 전달해줄 때는 그냥 notifyOrderSuccess(orderList)로 전달해줄 수 없다. 그러면 함수가 바로 실행된다. 
}
```

# <span style="color:#802548">_callback 함수는 함수 정의를 넣는 것이지, call하는 게 아니다._</span>
- 왜 그냥 notifyOrderSucccess(orderList)는 안될까?
- 아래의 구문은, 이미 만들어진 함수를 call하는 것이라 eventListener로 등록되는 데 그치지 않고 즉시 실행된다. 그래서 문제인 것이다.

```js
el.addEventListener('click', notifyOrderSucccess(orderList))
 ```

- 내가 eventListenr에 주어야 하는 것은 함수의 선언식, 표현식이다. 이미 만들어진 함수가 아니다.
- 즉 callback함수를 정의해줘야 하는 것이지, 이미 만들어진 함수를 호출하라는 게 아니다.

```js
el.addEventListener('click', function() {
    notifyOrderSucccess(orderList)
});

el.addEventListener('click', () => notifyOrderSucccess(orderList))
```

- parameter가 없는 경우도 살펴보자.
- 아래와 같이는 사용할 수 있다.

```js
 el.addEventListener('click', function() {
    notifyOrderSucccess()
});

 el.addEventListener('click', notifyOrderSucccess); //함수를 call한 게 아니다. 이미 정의된 함수를 넘긴 것이다.

 el.addEventListener('click', () => notifyOrderSucccess());

```

- 하지만 아래와 같이는 사용할 수 없다.
- 정의된 함수를 가져온 게 아니다. 


```js
el.addEventListener('click', notifyOrderSucccess()); //함수를 call했다. 이건 절대 안된다.
```


