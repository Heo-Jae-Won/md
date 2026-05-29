## <span style="color:#FFCCFF">Promise</span>
- promise는 비동기처리에 사용되는 객체다. setTimeout의 callback hell이라는 문제점을 극복하고자 하는 시도에서 탄생했다.

- 주로 서버과 통신해 api response를 받아올 떄 쓰인다.
- 문법은 아래와 같다.


```
성공(fulfilled)이라면 resolve()에 비동기처리 결과를 담는다.
실패(rejected)라면 reject()에 비동기처리 결과를 담는다.
```

- 아래는 성공했을 경우에 대한 예시다.


```javascript
function getData() {
  return new Promise(function(resolve, reject) {
    var data = 100;
    resolve(data);
  });
}

// resolve()에 argument로 넣은 data를 callback function의 parameter로 가져온다.
getData().then(function(resolvedData) {
  console.log(resolvedData); // 100
});
```

- 아래는 비동기 처리 결과가 오류일 때에 대한 예시다.


```javascript
function getData() {
  return new Promise(function(resolve, reject) {
    reject(new Error("Request is failed"));
  });
}

// reject()의 결과 값 Error를 err에 받음
getData().then().catch(function(err) {
  console.log(err); // Error: Request is failed
});
```

- 아래는 실제 서버에 요청하는 request를 비동기로 보내는 간단한 예시다.
- status가 200이면 성공이므로 resolve 처리한다.

- resolve(성공)한 데이터를 then()으로 받는다. 


```javascript
const promiseGet = url => {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open('GET', url);
        xhr.send();

        xhr.onload = () => { 
          //onload는 event property 방식의 event handler.
          //event queue에 callback 함수를 push
            if(xhr.status === 200) {
                resolve(JSON.parse(xhr.response));
            } else {
                reject(new Error(xhr.status));
            }
        }
    })
}

promiseGet('https://jsonplaceholder.typicode.com/posts/1').then(res => console.log(res));
/*
{
  "userId": 1,
  "id": 1,
  "title": "sunt aut facere repellat provident occaecati excepturi optio reprehenderit",
  "body": "quia et suscipit\nsuscipit recusandae consequuntur expedita et cum\nreprehenderit molestiae ut ut quas totam\nnostrum rerum est autem sunt rem eveniet architecto"
}
*/
```

- Promise에서는 catch에서 error를 처리한다.

- then은 성공했을 떄의 처리를 담당한다.
- finally는 어떤 경우에도 일어나야만 할 때 필요하다.


```javascript
promiseGet('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => console.log(res))
    .catch(err => console.error(err))
    .finally(() => console.log('Bye!'));
```

- 위에서는 XMLHttpRequest를 사용했지만, 최신 api로 fetch가 있다.

- fetch는 XMLHttpRequest와 기능이 같다. 다만 HTTP error의 경우, reject하지 않는다.
- 대신 resolve()로 받은 response의 ok 값이 false로 온다. 따라서 서버에러를 catch가 아닌 then에서 처리해줘야 한다. 


```javascript
const request = {
    get(url){
        return fetch(url);
    }
}

function getResponse() {
  request.get('https://jsonplaceholder.typicode.com/todos/1')
    .then(response => {
        if(!response.ok) {
            throw new Error(response.statusText); 
        }

        return response.json();
    })
    .then(todos => console.log(todos))  
    // response.json()을 받은게 todos
    .catch(err => console.error(err)); 
    //오프라인 네트워크 장애, CORS 설정 에러 등만 잡힘
}
getResponse();
```

- Promise를 동기처럼 처리하고 싶다면 async - await를 쓸 수도 있다.

- async await의 경우, try ~ catch문에서 error를 처리하는 경우가 흔하다. 
- .catch()보다 가독성이 좋기 때문이라고 한다.


```javascript
const request = {
    get(url){
        return fetch(url);
    }
}

async function getResponse() {
  try{
    const result = await request
                            .get('https://jsonplaceholder.typicode.com/todos/1')
                            .then(res => res.json()).catch(err => err);
    console.log(result);
  } catch(e) {
    console.log(e);
  }
}

getResponse(); 
/*
{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}
*/
```

- 아래의 예시를 살펴보자. async 함수 범위안의 console에는 찍히지만, 밖에서 console을 찍어 결과를 가져오면 결과를 가져오지 못한다.
- console.log의 위치를 async 밖으로 빼내면 원하는 데이터가 나오지 않는 이유가 무엇일까?

- 그 이유는 async fn 내부에서만 await 함수가 끝나는 것을 기다리기 때문이다. 
- 즉 getResponse()의 await가 붙은 함수가 실행이 끝나 response가 올 때까지 기다려주지 않는다.


```javascript
const request = {
    get(url){
        return fetch(url);
    }
}

async function getResponse() {
  try{
    const result = await request.get('https://jsonplaceholder.typicode.com/todos/1').then(res => res.json());

    return result;
  } catch(e) {
    console.log(e);
  }
}

console.log(getResponse()); //Promise
```

- 그를 해결하려면 간단하다. getResponse()에도 await를 붙여준다.


```javascript
const request = {
    get(url){
        return fetch(url);
    }
}

async function getResponse() {
  try{
    const result = await request.get('https://jsonplaceholder.typicode.com/todos/1').then(res => res.json());

    return result;
  } catch(e) {
    console.log(e);
  }
}

console.log(await getResponse());
/*
{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}
*/
```


- 이와 똑같은 맥락으로 then문 이전과 이후를 나눠서 변수로 만들면 데이터가 나오지 않는다.


```javascript
const request = {
    get(url){
        return fetch(url);
    }
}

async function getResponse() {
  try{
    const result = await request.get('https://jsonplaceholder.typicode.com/todos/1');
    console.log(result instanceof Response) //true
    console.log(result instanceof Promise) //false

    const data = result.then(res => res.json()); //result.then is not a function
  } catch(e) {
    console.log(e);
  }
}
```

- then문을 나눴다면 이미 resolve되어 Response 객체가 되었으므로 Response라는 prototype에 있는 json()을 통해 결과값을 받아와야 한다.

- 하지만 아래와 같이 하면 그냥 Promise 객체만 도달한다. 결과값을 얻어오는 것도 비동기처리의 일환이라 그렇다.


```javascript
const request = {
    get(url){
        return fetch(url);
    }
}

async function getResponse() {
  try{
    const result = await request.get('https://jsonplaceholder.typicode.com/todos/1');
    const data = result.json(); 
    console.log(data)
  } catch(e) {
    console.log(e);
  }
}

getResponse(); //Promise<pending>
```

- 비동기처리를 동기처럼 바꿔주기 위해 json() 함수에도 await를 붙여주면 값이 정상 출력된다.


```javascript
const request = {
    get(url){
        return fetch(url);
    }
}

async function getResponse() {
  try{
    const result = await request.get('https://jsonplaceholder.typicode.com/todos/1');
    const data = await result.json(); 
    console.log(data)
  } catch(e) {
    console.log(e);
  }
}

getResponse();
/*
{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}
*/
```


- promise를 병렬로 가져오는 방법도 있다.
- 아래의 예시를 살펴보자. 한꺼번에 병렬로 가져오면 1초면 되는 작업이다.

- 하지만 동기로 처리되어 이전의 처리 결과를 기다리게 된다. 그럼 10초가 걸린다.


```javascript
function setTimeoutPromise(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve(), ms);
  });
}

async function fetchAge(id) {
  await setTimeoutPromise(1000);
  console.log(`${id} 사원 데이터 받아오기 완료!`);
  return parseInt(Math.random() * 20, 10) + 25;
}

async function startAsyncJobs() {
  let ages = [];
  for (let i = 0; i < 10; i++) {
    let age = await fetchAge(i);
    ages.push(age);
  }

  console.log(
    `평균 나이는? ==> ${
      ages.reduce((prev, current) => prev + current, 0) / ages.length
    }`,
  );
}

startAsyncJobs();
```

- 따라서 비동기 작업이 순차를 기다리지 않아도 되는 경우에는 아래와 같이 Promise.all을 사용하는 것이 더 좋다.


```javascript
function setTimeoutPromise(delay) {
  return new Promise((resolve) => setTimeout(resolve, delay));
}

async function fetchAge(id) {
  await setTimeoutPromise(1000);
  console.log(`${id} 사원 데이터 받아오기 완료!`);
  return Math.round(Math.random() * 20) + 25;
}

async function startAsyncJobs() {
  const ids = Array.from({ length: 10 }).map((_, index) => index);

  const promises = ids.map(fetchAge);

  const ages = await Promise.all(promises);

  console.log(
    `평균 나이는? ==> ${
      ages.reduce((prev, current) => prev + current, 0) / ages.length
    }`,
  );
}

startAsyncJobs();
```

- async - await는 비동기 함수 중 Promise 함수를 사용했을 때만 사용가능하다.
- async 함수가 붙으면 반드시 resolve된 Promise를 return해야 하기 때문이다.

- async keyword가 붙으면 await도 써주는 게 좋다.
- 아래를 보면 await가 붙어있지 않다. 그러자 catch문에 error가 catch되지 않는다.
- 색깔도 warn이라 노란색이어야 하는데 빨간색으로 나온다.


```javascript
async function thisThrows() {
    throw new Error("Thrown from thisThrows()");
}

async function myFunctionThatCatches() {
    try {
        return thisThrows();
    } catch (e) {
        console.warn(e,'catch');
    } finally {
        console.log('We do cleanup here');
    }
    return "Nothing found";
}

async function run() {
    const myValue = await myFunctionThatCatches();
    console.log(myValue);
}

run(); //We do cleanup here
/*
  Uncaught (in promise) Error: Thrown from thisThrows()
  at thisThrows (<anonymous>:2:11)
  at myFunctionThatCatches (<anonymous>:7:16)
  at run (<anonymous>:17:27)
  at <anonymous>:21:1
*/
```

- await를 쓰면 아래에서 error가 catch되는 모습을 볼 수 있다. 
- 색깔도 warn으로 노란색이다.


```javascript
async function thisThrows() {
    throw new Error("Thrown from thisThrows()");
}

async function myFunctionThatCatches() {
    try {
        return await thisThrows();
    } catch (e) {
        console.warn(e, 'catch');
    } finally {
        console.log('We do cleanup here');
    }
    return "Nothing found";
}

async function run() {
    const myValue = await myFunctionThatCatches();
    console.log(myValue);
}

run();  
/*
  Error: Thrown from thisThrows()
  at thisThrows (<anonymous>:2:11)
  at myFunctionThatCatches (<anonymous>:7:22)
  at run (<anonymous>:17:27)
  at <anonymous>:21:1 'catch'
*/
//We do cleanup here
//Nothing Found

```

- 즉 async를 쓰면 await도 써야 try ~ catch에서 error catch가 정상으로 작동한다고 볼 수 있다.


## <span style="color:#802548">_Promise와 sync_</span>
- Promise 객체는 callback의 지옥을 벗어나기 위해 만들어진 logic이다.
- sync하게 사용할 경우 callback funciton없이 parameter만 받는다.

```js
var sync1 = function(param){ return param　*　2; }
var sync2 = function(param){ return param　*　3; }
var sync3 = function(param){ return param　*　4; }
var start = 1;
console.log(sync3(sync2(sync1(start)))); // 24
```

- 그걸 Promise 객체로 바꾸면 아래와 같이 변경할 수 있다.
- 아래는 new Promise()를 사용하였다.

```js
function async1 (param) {    return new Promise(function(resolve, reject) {        resolve(param　*　2);    });}
function async2 (param) {    return new Promise(function(resolve, reject) {        resolve(param　*　3);    });}
function async3 (param) {    return new Promise(function(resolve, reject) {        resolve(param　*　4);    });}
var start = 1;
async1(start)    
    .then(async2)    
    .then(async3)    
    .then(result => {
        console.log(result); // 24    
    });
```

- 아래는 Promise.resolve()를 사용하였다.
- 위와 똑같은 효과가 나타난다.

```js
function async1 (param) {    return Promise.resolve(param * 2) };
function async2 (param) {    return Promise.resolve(param * 3) };
function async3 (param) {    return Promise.resolve(param * 4) };
var start = 1;
async1(start)    
    .then(async2)    
    .then(async3)    
    .then(result => {
        console.log(result); // 24    
    });
```

## <span style="color:#802548">_Promise와 async_</span>

- Promise를 사용하지 않은 async 코드는 아래와 같이 callback 안에서 callback을 다시 한 번 호출한다.

```js
var async1 = function(param, callback) { callback(param * 2); }
var async2 = function(param, callback) { callback(param * 3); }
var async3 = function(param, callback) { callback(param * 4); }
var start = 1;
 async1(start, result => {
    async2(result, result => {
            async3(result, result => {
                    console.log(result); // 24        
        });    
    });
});
```

- new Promise()를 사용한 코드로 async code를 고쳐보았다.
- 다른 게 없어 보이지만, fetch와 같이 ajax call의 경우는 문제가 발생한다.

```js
function async1 (param) {    return Promise.resolve(param * 2) };
function async2 (param) {    return Promise.resolve(param * 3) };
function async3 (param) {    return Promise.resolve(param * 4) };
var start = 1;
async1(start)    
    .then(async2)    
    .then(async3)    
    .then(result => {
        console.log(result); // 24    
    });
```

- ajax call에 Promise.resolve()를 쓰게 되면 result가 undefined가 찍히게 된다.
- xhr.send()를 보내면 Promise의 contexnt가 끝나서 xhr.send의 response를 받을 시점에는 함수가 이미 끝나있다.
    - 즉 Promise.resolve()는 callback으로서 exceution을 잠깐 미뤄놓은 것일 뿐, await의 기능은 실행하지 못한다.
    - 따라서 resolve()가 pending인 상태에서 then()이 실행된다.
    - 따라서 undefined가 나오게 되는 것이다.

```js
function request (param) {    
    return Promise.resolve()        
            .then(function () {            
                var xhr = new XMLHttpRequest();
                xhr.open('GET', 'http://google.co.kr/', true);            
                xhr.onreadystatechange = function () {                
                    if (xhr.readyState == 4 && xhr.status == 200) {                    
                        return Promise.resolve(xhr.response);                
                    }            
                };            
                xhr.send(null);        
            });
} 
Promise.resolve()    
    .then(request)    
    .then(result => {        
        console.log('ok');        
        console.log(result);    
    });
```


- 이를 new Promise로 바꾸면 request가 response를 얻어올 때 까지 대기한다.
- request가 complete되면 response를 resolve한다.
- 따라서 바로 then이 호출되지 않고 pending되었다가 request가 complete되는 순간 then()이 발동한다.

```js
function request (param) {    
    return new Promise((resolve, reject) => {            
        var xhr = new XMLHttpRequest();
        xhr.open('GET', 'http://google.co.kr/', true);            
        xhr.onreadystatechange = function () {                
            if (xhr.readyState == 4 && xhr.status == 200) {                    
                return Promise.resolve(xhr.response);                
            }            
        };            
        xhr.send(null);        
    });
} 
Promise.resolve()    
        .then(request)    
        .then(result => {        
            console.log('ok');        
            console.log(result);    
        });
```

- 안타깝게도 xhr은 await 문법을 지원하지 않기 때문에 then()의 형태로만 사용 가능하다.
- 아래처럼 억지로 async - await로 바꿔도 간결해지지 않기 때문에 그냥 then()으로 쓰자.

```js
async function request(param) {
    return new Promise((resolve, reject) => {
        var xhr = new XMLHttpRequest();
        xhr.open("GET", "http://google.co.kr/", true);

        xhr.onreadystatechange = function () {
            if (xhr.readyState === 4) {
                if (xhr.status === 200) {
                    resolve(xhr.response); // ✅ Correctly resolve the promise
                } else {
                    reject(new Error(`Request failed with status ${xhr.status}`)); // ❌ Properly handle errors
                }
            }
        };

        xhr.onerror = function () {
            reject(new Error("Network error")); // ❌ Handle network errors
        };

        xhr.send(null);
    });
}

// ✅ Async function to call `request` with await
async function fetchData() {
    try {
        const result = await request();
        console.log("ok");
        console.log(result);
    } catch (error) {
        console.error("Error:", error.message);
    }
}

// Call the async function
fetchData();
```



## <span style="color:#802548">_Promise와 fetch-then_</span>

- fetch는 기본적으로 ajax를 call하는 Promise()이며 server에서 받은 response를 resolve한다.
- response를 아래와 같이 명시적으로 Promise.resolve()안에 return을 할 수도 있다.

```js
fetch("/mypage/checkPassword", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ currentPassword: password.value })
})
.then(response => return Promise.resolve(response.json()))
```

- 하지만 Promise.resolve()는 암묵적으로 씌워진다. 
- 따라서 그것을 쓰지 않아도 된다.
- 동시에 return도 1줄짜리면 생략이 가능하다.

```js
fetch("/mypage/checkPassword", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ currentPassword: password.value })
})
.then(response => response.json())
```

- 2줄 이상이 된다면 return을 반드시 써줘야 한다.
- Promise.resolve()로 감싸지 않아도 정상 작동한다.

```js
fetch("/mypage/checkPassword", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ currentPassword: password.value })
})
.then(response => {
    console.log(response);
    return Promise.resolve(response.json());
})
```

- Promise.resolve()는 안 감싸도 된다.
- 자동으로 감싸지기 때문이다.

```js
fetch("/mypage/checkPassword", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ currentPassword: password.value })
})
.then(response => {
    console.log(response);
    return response.json();
})
```

- 서버에서 난 오류는 catch문에서 처리하지 않는다.
- then 문에서 response.ok로 분기하여 처리해야 한다.

```js
fetch("/mypage/checkPassword", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ currentPassword: password.value })
})
.then(response => {
   if (response.ok) {
        return Promise.resovle(response.json()); // Must return the parsed JSON
    } else {
        console.error(`Server error: ${response.status} ${response.statusText}`);
        return Promise.resolve(false); // Return false for server errors
    }
})
```

- network에서 오류가 난 것은 catch를 실행한다.
- 따라서 reject는 catch에서 실행한다.

```js
.catch(error => {
    console.error('Error:', error);
    return Promise.reject(new Error("Network error, please check your internet connection."));
});
```


## <span style="color:#802548">_Promise와 fetch-async_</span>
- fetch의 경우, async await를 사용한 경우에는 더더욱 간단해진다.
- 아래처럼 async - await를 사용해도 then을 사용할 수는 있지만, 이 경우 문제가 있다.
    - fetch가 실패했는지, json()이 실패했는지 error trace를 찾기가 어렵다.

```js
async function checkPw() {
    if (password.value.length === 0) {
        isPasswordCorrect = false;
        return false; // ✅ No need for Promise.resolve(false)
    }

    try {
        const response = await fetch("/mypage/checkPassword", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ currentPassword: password.value })
        }).then((res) => res.json())

        if (!response.ok) {
            console.error(`Server error: ${response.status} ${response.statusText}`);
            return false; // ✅ No Promise.resolve needed
        }

        isPasswordCorrect = data;

        return isPasswordCorrect;
    } catch (error) {
        console.error('Error:', error);
        throw new Error("Network error, please check your internet connection."); // ✅ No Promise.reject needed
    }
}
```

- 따라서 response를 따로 await하여 별도의 객체로 만든다.
- 그리고 그것을 다시 await하여 별도의 객체로 만든다.

```js
async function checkPw() {
    if (password.value.length === 0) {
        isPasswordCorrect = false;
        return false; // ✅ No need for Promise.resolve(false)
    }

    try {
        const response = await fetch("/mypage/checkPassword", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ currentPassword: password.value })
        });

        if (!response.ok) {
            console.error(`Server error: ${response.status} ${response.statusText}`);
            return false; // ✅ No Promise.resolve needed
        }

        const data = await response.json();
        isPasswordCorrect = data;

        return isPasswordCorrect;
    } catch (error) {
        console.error('Error:', error);
        throw new Error("Network error, please check your internet connection."); // ✅ No Promise.reject needed
    }
}
```

- 해당 함수를 호출할 때는 async가 붙어있기 때문에 그냥 return하면 Promise 객체가 return된다.
- 아래와 같이 checkPw를 호출하면 Promise를 뱉기 때문에 result가 제대로 사용되기 어렵다.

```js
const password = document.querySelector('#password');
password.addEventListener("keyup", () => { //debounce 적용 필요
    const result = passwordChecker.checkPw();
    console.log("Password check complete:", result);
});
```

- 따라서 eventListener 등의 상위 호출 함수에도 async를 붙여주고, await를 붙여준다.
- 핵심은 해당 함수를 await를 붙여 resolve시키는 것이다.
- await는 혼자서는 쓰일 수 없기 때문에 async도 상위 함수에 같이 붙여줘야 한다.

```js
const password = document.querySelector('#password');
password.addEventListener("keyup", async () => { //debounce 적용 필요
    const result = await passwordChecker.checkPw();
    console.log("Password check complete:", result);
});
```

## <span style="color:#802548">_Promise와 setTimeout_</span>

- setTimeout은 callback과 delay가 같은 코드에 섞여 있다.

```js
function notifyCallback(orderList) {
    orderList.forEach((order) => document.querySelector('#log').textContent += `${order}가 완료됐습니다.<br/>`); // 
}

function notifyOrderSucccess(orderList, time) { 
    setTimeout(() => notifyCallback(orderList),time); 
}

function orderCoffee(el, orderList) {
    if(!el || Array.isArray(orderList)) { // early return
        return;
    }

    el.addEventListener('click', () => notifyOrderSucccess(orderList)) //parameter를 전달해줄 때는 그냥 notifyOrderSuccess(orderList)로 전달해줄 수 없다. 그러면 함수가 바로 실행된다. 
}
```

- Promise를 이용하면 setTimeout에서 delay와 callback을 분리할 수 있다.
- 우선 setTimeout의 delay만 Promise로 return하게 만든다.

```js
function delay(time) { //setTimeout을 한줄로 간단하게 나눠주기 위한 함수..
    return new Promise((resolve) => setTimeout(()=> resolve(),time)); // () => resolve()는 익명함수를 정의한 것과 동일하다. 다시말해 time만큼이 지나면 그냥 resolve()가 된다. 완료된다는 의미다.
}
```

- resolve에 아무 것도 쓰지 않으면 undefiend를 return한다.
- 아래 세 개는 전부 동일한 의미다.

```js
function delay(time) { 
    return new Promise((resolve) => setTimeout(()=> resolve(undefiend),time)); 
}

function delay(time) { 
    return new Promise((resolve) => setTimeout(()=> resolve(),time)); 
}

function delay(time) { 
    return new Promise((resolve) => setTimeout(resolve,time)); 
}
```

- Promise.resolve()를 사용하면 아래와 같이 변하지만, 여기선 바람직하지 않다.
- 쓸데 없이 한 번 더 묶는 꼴이 되기 때문이다.

```js
function delay(time) {
    return Promise.resolve().then(() => new Promise(resolve => setTimeout(resolve, time)));
}
```

- delay를 분리했다면, 아래와 같이 로직도 분리하여 줄 수 있다.


```js
const notifyOrderSucccess = async (orderList) => {
    await delay(2000);
    orderList.forEach((order) => document.querySelector('#log').textContent += `${order}가 완료됐습니다.<br/>`); // for문을 한줄로 줄이기 위한 고차함수
}

function orderCoffee(el, orderList) {
    if(!el || Array.isArray(orderList)) { // early return
        return;
    }

    el.addEventListener('click', () => notifyOrderSucccess(orderList))
}
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