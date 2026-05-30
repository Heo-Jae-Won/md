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

- 아래처럼 사용하면 된다.

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


# <span style="color:#802548">_if문 줄이기_</span>
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
    if(!el || Array.isArray(orderList)) { // early return
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


- 아래처럼 notifyOrderSucccess(parameter)를 바로 eventListener에 넣어주려면 아래와 같이 return을 함수로 하면 된다.

```js
function delay(time) { //setTimeout을 한줄로 간단하게 나눠주기 위한 함수..
    return new Promise((resolve) => setTimeout(()=> resolve(),time)); // () => resolve()는 익명함수를 정의한 것과 동일하다. 다시말해 time만큼이 지나면 그냥 resolve()가 된다. 완료된다는 의미다.
}

const notifyOrderSucccess = async (orderList) => () => {
    await delay(2000);
    orderList.forEach((order) => document.querySelector('#log').textContent += `${order}가 완료됐습니다.<br/>`); // for문을 한줄로 줄이기 위한 고차함수
}

function orderCoffee(el, orderList) {
    if(!el || Array.isArray(orderList)) { // early return
        return;
    }

    el.addEventListener('click', notifyOrderSucccess(orderList)) //parameter를 전달해줄 때는 그냥 notifyOrderSuccess(orderList)로 전달해줄 수 없다. 그러면 함수가 바로 실행된다. 
}
```

- 아래는 함수 선언 방식으로 짰을 때다. 해당 방식에서는 async를 notifyOrderSuccess에 넣으면 안되고, return할 function에 넣어야 한다.

```js
function delay(time) { //setTimeout을 한줄로 간단하게 나눠주기 위한 함수..
    return new Promise((resolve) => setTimeout(()=> resolve(),time)); // () => resolve()는 익명함수를 정의한 것과 동일하다. 다시말해 time만큼이 지나면 그냥 resolve()가 된다. 완료된다는 의미다.
}

function notifyOrderSucccess(orderList) {

    return async function() {
        await delay(2000);
        orderList.forEach((order) => document.querySelector('#log').textContent += `${order}가 완료됐습니다.<br/>`); // for문을 한줄로 줄이기 위한 고차함수
    }
    
}

function orderCoffee(el, orderList) {
    if(!el || Array.isArray(orderList)) { // early return
        return;
    }

    el.addEventListener('click', notifyOrderSucccess(orderList)) //parameter를 전달해줄 때는 그냥 notifyOrderSuccess(orderList)로 전달해줄 수 없다. 그러면 함수가 바로 실행된다. 
}
```

# <span style="color:#802548">_함수를 return하는 화살표 함수_</span>
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

# <span style="color:#802548">_배열이나 객체를 활용하자._</span>
- 이 경우, 기본 값을 주는 게 재활용성에 방해가 될 수 있다.
- 만약 다른 사람은 기본체를 바탕으로 하고 싶었다면 아래 함수를 활용할 수가 없다.

```js
function setPreferedFont(font) {
    if (font) {
        return font;
    }

    return Font.Arial;
}

function setPreferedFont(font) {
    if (font) {
        return font;
    }
}
```

- 지나치게 좁은 경우의 상정은 재사용성은 해친다.
- 이미지를 하나만 return하고 싶어서 아래와 같이 if문을 넣었다.
- 하지만 이후에 이미지를 여러개 return해야 한다면?
- 그런 경우도 생각한다면 배열에 push해서 배열을 return하는 게 낫다.
- 성능은 조금안좋다고 해도 말이다.
- 섣부른 최적화보단 재사용성이 높게 만들자.


```js
function getImage() {
    for (section in sections) {
        if (section.containsImage()) {
            return section.image;
        }
    }
}


function getImages() {
    const images = [];
    for (const section of sections) {
        images.push(section.image);
    }

    return images;
}
```

# <span style="color:#802548">_기본값은 함부로 달면 안 된다._</span>
- 값이 없으면 null이나 error를 반환해야 한다.
- age가 없는 경우 0으로 처리해보자.
- 당장은 문제가 없지만 그 함수를 가지고 2차로 함수를 만들면 문제가 있다.
- 나이가 없는 사람은 0으로 처리되어 평균이 극히 낮아진다.
- 오류가 나면 어디서 오류가 났는 지 찾기가 매우 힘들다.
- 이럴 땐 나이가 0이면 filter를 하던가 해야된다.

```js
function getAge() {
    if (!this.age) {
        return 0;
    }

    return this.age;
}

function getAvgAge(userList) {
    const sumOfAge = userList.reduce((acc,cur) => acc + cur.getAge(), 0)
    /*for (const user in userList) {
        sumOfAge +=parseInt(user.getAge())
    }*/

    return sumOfAge / userList.length
}
```

- 아니면 아래와 같이 그냥 null을 return해버릴 수도 있다.


```js
function getAge() {
    if (!this.age) {
        return null;
    }

    return this.age;
}

function getAvgAge(userList) {
    const sumOfAge = userList.reduce((acc,cur) => (cur ? acc + cur.getAge() : acc), 0);
    const length =  userList.reduce((acc,cur) => (cur ? acc + 1 : acc), 0);
    
    /*for (const user in userList) {
        sumOfAge +=parseInt(user.getAge())
    }*/

    const averageAge = length > 0 ? sumOfAge / length : 0;

    return averageAge /*sumOfAge / length 혹시 0이면 0으로 나눠 error가 나므로 그 처리를 위에서 해준다.*/ 
}
```








# <span style="color:#802548">_strategy pattern_</span>
- 전략이 바뀌어도 getDialogues()는 바뀌지 않아도 된다.

```js
const Units = {
    캐리어() {
        return 'Carrier has arrived'
    },
    다크템플러() {
        return 'Adun Toridas'
    },
    커세어() {
        return 'It is a good day to die!'
    }
}

const getDialogues = (unitName) => {
    return Units[unitName]();
}
```

- 실제 예시로는 chart가 있다.
- 도넛 차트, 꺾은선 차트 등...필요한 걸 골라야 할 것이다.
- 그 때는 각각 차트를 만들 수도 있다.

- 통합적으로 컨트롤이 안된다.


```js
const chart = new BarChart();
chart.render(data);

const char2 = new DonutChart();
chart.render(data);

const chart3 = new LineChart();
chart.render(data);
```

- Chart라는 껍데기를 만들어주고, 그 안에 넣는 strategy만 바꾼다.
- 근데 strategy가 많지 않다면 굳이 안 써도 될거 같다.


```js
class Chart {
    constructor(strategy) {
        this.chartType = strategy;
    }
    render(data) {
        this.chartType.render(data);
    }
}
cosnt chart = new Chart(new BarChart());
chart.render(data);

chart = new Chart(new DonutChart());
chart.render(data);
```