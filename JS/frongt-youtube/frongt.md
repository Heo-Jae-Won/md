# <span style="color:#802548">_함수 return이 closure일 가능성이 높다._</span>
- js를 쓸 때 아래와 같이 decorator로 감싸서 log를 남길 수도 있다.
```javascript
function logTimeDecorator(fn) {
  return function() {
    console.info(`"${fn.name}" function is called`);
    return fn();
  }
}

let foo = () => console.log("frongt");
foo = logTimeDecorator(foo);
foo(); 
/*
"'foo' function is called"
"frongt"
*/
```

- React의 useState는 내부에서 closure를 활용한다.
```javascript
const useState = (initialValue) => {
    let state = initialValue;
    const setState = (fn) => {
        state = fn(state);
        console.log("render : ", state);
    };
    return [state, setState];
}
```

- React의 Router를 만약 코드로 짠다면 아래와 같을 것이다.
- router를 이해하려면 pushState는 꼭 알아두는 게 좋다.
```javascript
class Router {
    constructor() {
        this.routes = [];
        window.addEventListener('popstate', this.handlePopState.bind(this));
    }
    .
    .
    .
    push(path) {
        window.history.pushState({}, '' , path);
        this.render(path);
    }
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


# <span style="color:#802548">_병렬요청은 Promise.all()_</span>
- 병렬 요청이면 Promise.all()을 써야 한다.
- 아래처럼 보내면 첫 res1 함수가 끝나고 나서나 res2가 호출된다.

```js
(async function go() {
    console.time(1);
    const url = [URL+1, URL+2];

    const res1 = await fetch(urls[0]);
    const data1 = await res1.json();

    const res2 = await fetch(urls[1]);
    const data2 = await res2.json();

    console.log(data1, data2);
    console.timeEnd(1);
})
```


-  병렬로 받으려면 Promise.all()을 쓴다.
-  그럼 Network 디버깅을 보면 waterfall이 같이 시작된 것을 볼 수 있다.


```js
(async function go() {
    console.time(1);
    const urls = [URL+1, URL+2];

    const requests = urls.map(url => fetch(url));

    const [res1, res2] = await Promise.all(requests); //배열이 들어가야 한다.
    const [data1, data2] = await Promise.all([res1.json(), res2.json()]);
    console.log(data1, data2);
    console.timeEnd(1);
})
```


# <span style="color:#802548">_비동기에서 error를 다루는 방법은 Promise.reject()_</span>
- error handling을 할 때, throw new error 외 다른 방법도 있다.


```js
async function bar(url) {
    const res = await fetch(url);
    if(!res.ok) {
        throw new Error(`ok가 아니다 ${res.status}`);
    }
    const data = await res.json();
}

async function foo() {
    try {
        const result = await bar("https://httpstat.us/500");
        return result;
    } catch (e) {
        console.log("에러가 있네. 통신에 문제가 있는듯",  e);
    }
}
/*
에러가 있네. 통신에 문제가 있는듯 ok가 아니다 500
*/
```

- 비동기 통신을 할 때는, 특히 Promise.reject()로 에러를 반환하는 게 더 정확하다.


```js
async function bar(url) {
    const res = await fetch(url);
    if(!res.ok) {
        return Promise.reject(`ok가 아니다 ${res.status}`);
    }
    const data = await res.json();
}
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





# <span style="color:#802548">_디버깅 팁_</span>
- 디버깅 팁 HTML

```
HTML inspect하고 scroll into view하면 해당 DOM으로 이동

hover는 force state: hover를 해두면 볼 수 있다.

Elements에서 우하단 property를 보면 해당 HTML의 속성을 모두 볼 수 있다.

클릭 등이나 뭔가 event가 일어나서 css가 바뀔 때, break on에서 attribute modifications를 찍어준다.

만약 event가 html 추가라면, break on: subtree modification일 것이다.
```

- 디버깅 팁 CSS

```
elements computed에서 속성 클릭하면 어떤 속성이 어떤 파일에서 규정됐는 지 확인 가능
```

- 디버깅 팁 네트워크


```
받아오는 요청-응답은 모두 녹화가 가능하다.
페이징이 많은 경우, network tab에서 로그 보기가 어렵다. 그럴 땐 preserve log를 클릭

Disable Cache를 키면 서버에서 값을 반드시 가져오게 함.

네트워크 그래프에 파란선은 DOMCONTENTLOADED고 빨간선은 LOAD선이다. 이게 빠를수록 페이지 반응성이 좋은 것이다.

screenshots을 체크하면 시간별 화면이 보인다.

use large request rows를 클릭하면 압축안된 원본 파일 크기도 보인다.

waterfall을 누르면 서버에서 얼만큼의 시간이 걸렸나도 볼 수 있다. (Waiting for server response)

각 항목들은 우클릭해서 노출하고 싶은 것만 할 수 있다.

js의 경우 오른쪽 클릭해서 open in soucre panel해서 소스코드를 볼 수 있다.

우클릭 후 header options - waterfall - total duration + 우클릭 sort by - waterfall로 하면 오래걸린 순서가 앞으로 나오게 볼 수 있다.
```


- 디버깅 팁 콘솔로그


```
콘솔로그를 이제 그만 쓰자.

반복문에서 특정 값을 확인해야 한다면 add conditional breakpoint다.

attribute가 바뀔 때 break on: attribute modification을 걸어놨다면,
소스코드가 열린다. 그 때 어떤 함수가 해당 함수를 호출했는 지 궁금하다면,
call stack을 보면 된다.

특정 URL로 들어가는 경우 멈추라고하고 싶다면, sources - XHR/fetch Breakpoints를 입력하자. 그럼 해당 url이 호출될 때 debugging이 가능하다.

특정 event가 발생했을 때 멈추라고 하고 싶다면, sources - event listener breakpoints에서 debugging하고 싶은 event를 선택한다.
```


- 배포가 아닌 개발은 source code map을 볼 수 있다.

```
sources - 좌상단 점3개 클릭 - Group by folder, Group by Authroed/Deployed 클릭. Deployed는 복잡한 build된 코드니까 보지 말자.
Hide ignore-listed sources를 하고, 오른쪽상단에 톱니바퀴 클릭 - ignore list -node_modules 추가. 그러면 source map에서 node_module 안보이게 할 수 있음. 불필요한 파일 생략
다만 single file component(.vue)는 vue.config.js에 devtools: 'source-map' 옵션이 있어야 볼 수 있다.
```


- 소스안에 하드코딩은 줄이고, 매개변수를 활용하자.


```js
function timer() {
    setTimeout(() => {
        alert("time done")
    }, 3000);
}

function timer(time) {
    setTimeout(() => {
        alert("time done")
    }, time);
}
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

/* function getImages() {
    for (section in sections) {
            return section.image;
    }
} */

function getImages() {
    const images = [];
    for (const section of sections) {
        images.push(section.image);
    }

    return images;
}
```
# <span style="color:#802548">_magic number를 금지하라._</span>
- 열거형 값을 암묵적으로 처리하면 안 된다.
- 열거형 값은 차후에 추가가 가능하다. 그걸 늘 가정해야 한다.


```js
enum RESULT {
    BUST = "BUST",
    OK = "OK"
}

function predict(cond) {
    if (cond === RESULT.BUST) {
        return false;
    }

    return true;
}


enum RESULT {
    BUST = "BUST",
    OK = "OK",
    WORLD_END = "World end"
}

function predict(cond) {
    if (cond === RESULT.BUST) { //세상이 망했는데... 예측이 되네?
        return false;
    }

    return true;
}
```

- 열거형은 switch case로 써주자.


```js
function predict(cond) {
    switch (cond) {
        case RESULT.BUST:
            return faslse;
        case RESULT.OK:
            return true;
        default: throw new Error("이 케이스가 존재하지 않습니다.");
    }
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

# <span style="color:#802548">_객체나 배열을 변화시킬 때, 복사해서 써라_</span>
- 객체나 배열을 가져다 쓸 때는 복사해서 쓰는 게 좋을 수 있다.


```js
const originalObject = { key1: 'value1', key2: 'value2' };

// Copying the object using spread syntax
const copiedObject = { ...originalObject };

// Modifying the copied object
copiedObject.key3 = 'value3';

console.log(originalObject); // { key1: 'value1', key2: 'value2' }
console.log(copiedObject); 
```

- 아래와 같이 spreading operator를 이용하면 shallow copy를 만든다.
- shallow라고 해도 다른 객체라 copiedArray를 바꿔도 originalArray에 영향이 없다.
- 객체도 마찬가지다.


```js
const originalArray = [1, 2, 3, 4];

// Copying the array using spread syntax
const copiedArray = [...originalArray];

// Modifying the copied array
copiedArray.push(5);

console.log(originalArray); // [1, 2, 3, 4]
console.log(copiedArray);   // [1, 2, 3, 4, 5]
```


- 조건이 길다면 변수로 빼라.
- 의미를 부여할 수 있다.

```js
if (permission === 'customer' && level === '2' && expired ===false) {

}

const isDislayed = (permission === 'customer' && level === '2' && expired ===false);
if (isDislayed) {

}
```

# <span style="color:#802548">_변수의 이름은 늘 중요하다._</span>
- 변수의 이름은 중요하다.
- order에 따라 상품을 보여주는 쇼핑몰이라고 해보자.
- 구체적으로 써라.

```js
const id = 'asdf';

const orderId = 'asdf';
```

- 주석이 설명을 대신해주지 않는다.
- 주석은 현행화되지 않는 경우가 흔하다.

- 함수가 하는 일은 정확하게 구체적으로 묘사하라

- render라고 쓰지 않아서 value만 가져오니까 얼마 안 걸릴거라고 오해할 수 있다.


```js
function getValue() {
    render() // 시간이 걸리는 작업
    return value;
}

function renderAndGetValue() {
    render()
    return value;
}
```


- 코드 줄수 줄이는 게 중요하지 않다. 읽기 좋아야 한다.
- 삼중 tenary같은거 쓰지 말자.


- 상수는 이름을 줘야 한다.


```js
if (price < 1000) {
    invalid = true;
}

const MIN_ORDER_PRICE = 1000;
if (price < MIN_ORDER_PRICE) {
    invalid = true;
}
```

# <span style="color:#802548">_early return_</span>
- 깊이 있는 중첩은 피하자.
- early return이 가능하면 활용하자.

```js
const getOwnersAddress = (vehicle) => {
    let buyer;
    if (vehicle.hasBeenScraped()) {
        return SCRAPED_ADDRESS
    } else {
        const mostRecentPurchase = vehicle.getMostRecentPurchase()
        if (mostRecentPurchase === null) {
            return SHOWROOM_ADDRESS;
        } else {
            buyer = mostRecentPurchase.getBuyer(); 
            if (buyer !== null) {
                return buyer.getAddress()
            }
        }
    }
    return null;
}

const getOwnersAddress = (vehicle) => {
    if (vehicle.hasBeenScraped()) {
        return SCRAPED_ADDRESS
    }  

    const mostRecentPurchase = vehicle.getMostRecentPurchase()
    if (mostRecentPurchase === null) {
        return SHOWROOM_ADDRESS;
    } 

    let buyer = mostRecentPurchase.getBuyer(); 
    if (buyer !== null) {
        return buyer.getAddress()
    }
    
    return null;
}
```

# <span style="color:#802548">_parameter로 넣을 떄, 객체로 만들어 넣자._</span>
- 명명된 매개변수(key가 보이게)를 사용하자.
- 위도, 경도는 진짜 헷갈린다.
- 이럴 때 int, int type이 아니라 key가 보이는 객체를 parameter로 받자.


```js
const getLocation = (lat, lon) => {
    const locationName = googleMap(lat,lon);
    return locationName;
}

getLocation(2,1); //앗 1,2인데 헷갈려서 2,1로 넣음. 같은 type이니까 헷갈릴 수밖에..


const getLocation = ({lat, lon}) => {
    const locationName = googleMap(lat,lon);
    return locationName;
}

getLocation({lat:2,lon:1}); //key와 value를 개발자가 직접 구성하게 하니 헷갈리지 않음.
```


# <span style="color:#802548">_decorator pattern_</span>
- 디자인 패턴 중 decorator를 살펴보자.
- ScienceFacility는 바꾸지 않고 기능을 확장하는 방법이다.

```js
class ScienceFacility {
    constructor() {
        this.units = ['Science Vessl'];
    }

    getUnits() {
        return this.units;
    }
}

let scienceFacility = new ScienceFacility();
scienceFacility.getUnits(); // ['Science Vessel']


class PysicsLab {
    constructor(baseBuiliding) {
        this.baseBuiliding = baseBuiliding;
        this.units = ['Battlecruiser'];
    }

    getUnits() {
        return [...this.baseBuliding.getUnits() /* 변수로 넣은 원래 변수의  unit. 여기선 Science Vessel */, ...this.units /*여기선 Battlecruiser*/]; 
    }
}
scienceFacility = new PysicsLab(scienceFacility);
scienceFacility.getUnits(); // ['Science Vessel', 'Battlecruiser']

class CovertOps {
    constructor(baseBuiliding) {
        this.baseBuiliding = baseBuiliding;
        this.units = ['Ghost'];
    }

    getUnits() {
        return [...this.baseBuliding.getUnits() /* 변수로 넣은 원래 변수의  unit. 여기선 Science Vessel */, ...this.units /*여기선 Battlecruiser*/]; 
    }
}

scienceFacility = new CovertOps(scienceFacility);
scienceFacility.getUnits(); // ['Science Vessel', 'Ghost']
```


- 겹치는 부분은 상속을 통해 제거한다.


```js
class ScienceFacility {
    constructor() {
        this.units = ['Science Vessl'];
    }

    getUnits() {
        return this.units;
    }
}

let scienceFacility = new ScienceFacility();
scienceFacility.getUnits(); // ['Science Vessel']

class AddOnDecorator {
    constructor(baseBuilding) {
        this.baseBuilding = baseBuilding;
    }
}

class PysicsLab extends AddOnDecorator{
    constructor(baseBuiliding) {
        super(baseBuilding);
        this.units = ['Battlecruiser'];
    }

    getUnits() {
        return [...this.baseBuliding.getUnits() /* 변수로 넣은 원래 변수의  unit. 여기선 Science Vessel */, ...this.units /*여기선 Battlecruiser*/]; 
    }
}
scienceFacility = new PysicsLab(scienceFacility);
scienceFacility.getUnits(); // ['Science Vessel', 'Battlecruiser']

class CovertOps extends AddOnDecorator{
    constructor(baseBuiliding) {
        super(baseBuilding);
        this.units = ['Ghost'];
    }

    getUnits() {
        return [...this.baseBuliding.getUnits() /* 변수로 넣은 원래 변수의  unit. 여기선 Science Vessel */, ...this.units /*여기선 Battlecruiser*/]; 
    }
}

scienceFacility = new CovertOps(scienceFacility);
scienceFacility.getUnits(); // ['Science Vessel', 'Ghost']
```

# <span style="color:#802548">_class가 아닌 function_</span>
- class가 아니라 함수형으로 해도 똑같이 만들 수 있다.
- 함수 합성을 통해 가능하다.
```js
const pipe = (...fns) => (x) => fns.reduce((y,f) => f(y), x); //와 근데 ㄹㅇ 하나도 안 읽힌다.

const scienceFacility = () => {
    const units = ['Science Vessel'];
    return units;
}

const physicsLab = (baseUnits) => {
    const units = ['Battlecruiser'];
    return [...baseUnits, ...units];
}

const physicsLab2 = (baseUnits) => {
    const units = ['Ghosts'];
    return [...baseUnits, ...units];
}

const scienceFacilityWithPhsicsLab = pipe(
    scienceFacility,
    physicsLab,
    physicsLab2
)(); // scienceFacility 함수를 돌리고, 그 다음에 그 값을 받아서 physicsLab 함수를 돌린다.  

console.log(scienceFacilityWithPhsicsLab)
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

# <span style="color:#802548">_facade_</span>
- 추상화 패턴은 facade도 알아보자.
- 4개의 함수가 있다고 해보자.

```js
function checkShippingAddress(address) {
    //지역 확인 로직..
    if (address.includes('서울') || address.includes('경기')) {
        return 0;
    } else {
        return 3000;
    }
}

export function checkStock(productId) {
    //...재고 확인 로직
    return true; //재고가 있을 경우 true, 없을 경우 false 반환
}

export function checkDiscount(productId) {
    //... 할인정보 확인 로직 ...
    return 0.1;
}

export function checkout(productId, paymentInfo) {
    // ...결제 로직...
    return true;
}
```

- 아래의 4개 함수를 모두 쓰는 비즈니스 로직이다. 
- 4개의 함수를 모두 쓰기에 전부 알아야 한다.
```js
import { checkShippingADdress, checkStock, checkDiscount, checkout} from './myFunctions'

const productId = 'ABC123';
const paymentInfo = {
    ...
}

const address = '서울특별시 강남구';
const shippingCost = checkShippingAddress(address);

if (!checkStock(productId)) {
    console.error('Out of stock');
} else {
    const discount = checkDiscount(productId);
    
    try {
        const result = checkout(productId, paymentInfo);
        console.log("결제 성공: ", result);
        //결제가 성공했을 경우 처리 로직
    } catch (error) {
        console.error('결제 실패: ', error.message);
        //결제가 실패했을 경우 처리 로직
    }
}
```

- 위의 service.js에 모듈을 두어 추상화하자.
- 그럼 아래와 같이 아주 간단하게 service.js가 바뀐다.
```js
const productId = 123;
const paymentInfo = {...};
const address = '서울특별시 강남구 역삼동';

try {
    const success = checkout(productId, paymentInfo, address);
    console.log(success);
} catch(error) {
    console.error(error);
}
```

- checkout 함수는 중간 모듈인 payment.js에 있는 것이다.


```js
import { checkShippingADdress, checkStock, checkDiscount, checkout} from './myFunctions'

export function checkout(productId, paymentInfo, address) {
    if (!checkStock(productId)) {
        throw new Error('Out of Stock');
    }

     const discount = checkDiscount(productId);
     const shippingCost = checkShippingAddress(address);

     //... 결제 로직 ...
     return true;
}
```


- network debugging 시 오류 난 api를 우클릭 해 copy - copy as CURL(cmd)로 복사해서 보면 header, url 정보를 볼 수 있다.