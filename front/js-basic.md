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
- btn2 button을 클릭한다고 가정해보자. 그러면 btn2가 먼저 이벤트가 발동하고 그 뒤 상위로 전파된다.
- 그런데 btn2 button을 눌렀을 때 stopPropagation을 만나기 때문에 bubbling 되지 않는다.

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
- falsy일 때 가능하다.

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

- falsy는 아래의 값들이다.
- 빈 배열은 falsy가 아니라 truthy 값이다.
- 공백만 있는 문자열도 truthy 값이다.

```text
false 
0
-0 
0n
""
null
undefined
variable.NaN
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
		return user.name ? user.name : '이름없음';
	}
}

const getActiveUserName(user,isLogin){
	if(isLogin && user){
		return user.name || '이름없음';
	}
}
```


## <span style="color:#802548">_arrow function in class_</span>
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



