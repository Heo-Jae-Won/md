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




