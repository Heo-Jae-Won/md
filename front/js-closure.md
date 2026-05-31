
## <span style="color:#FFCCFF">lexical scope</span>
```
lexical scope는 함수가 호출된 곳이 아니라, 정의된 곳을 기준으로 한다.
```
- 전역에 정의되었기에 전역 companyName을 가져온다.


```javascript
const companyName = 'openobject';
function getCompanyName() {
  const companyName = 'other';
  getCompanyIndustry();
}

function getCompanyIndustry() {
  console.log(companyName);
}

getCompanyName(); //openobject
```

- 하지만 아래에서는 getCompanyName()안에 정의되었다.
- 따라서 해당 fn scope의 getCompanyName를 가져온다.


```javascript
const companyName = 'openobject';
function getCompanyName() {
  const companyName = 'other';

  function getCompanyIndustry() {
    console.log(companyName);
  }

  getCompanyIndustry();
}


getCompanyName(); //other
```

- 이 lexical scope를 기반으로 closure라는 방식의 함수 활용이 가능해진다.

## <span style="color:#FFCCFF">closure 기본 개념</span>
```
클로저는 중첩함수가 외부함수보다 오래 살아남았을 때 + 상위함수를 참조할 때, closure라고 한다.
즉 상위 함수가 종료되어도 상위 함수의 변수, parameter에 접근할 수 있는 함수다.
```
- 아래는 즉시실행함수를 활용한 closure의 예시다.


```javascript
const counter = (function(){
    let num = 0;
    return {
        increase(){
            return ++num;
        },
        decrease(){
            return --num;
        }
    }
}());

console.log(counter.increase()) //1
console.log(counter.increase()) //2
console.log(counter.decrease()) //1
console.log(counter.decrease()) //0
console.log(counter.decrease()) //-1
console.log(counter.decrease()) //-2
console.log(counter.increase()) //-1
```

- 즉시 실행함수는 한 번만 실행되므로 increase가 호출될 때마다 num 변수가 초기화되는 일은 없다.

- 즉시실행함수는 변수를 그냥 쓰는 것만으로도 함수가 호출된다.


```javascript
const counter = (function(){
   console.log('abc')
}());

counter //abc
```

- 원래로 돌아와서 살펴보면 counter를 호출하는 순간 아래와 같은 객체를 return하는 것을 알 수 있다.


```javascript
const counter = (function(){
    let num = 0;
    return {
        increase(){
            return ++num;
        },
        decrease(){
            return --num;
        }
    }
}());

console.log(counter);
/*
  {
    increase(){
        return ++num;
    },
    decrease(){
        return --num;
    }
  }
*/
```

- 따라서 increase와 decrease라는 method를 활용할 수 있게 된다.
```javascript
console.log(counter.increase()) 
console.log(counter.decrease()) 
```

- 그렇다면 남는 것은 num이다. num은 어떻게 +가 되는 것일까? 
- 위에서 말했듯 closure는 상위함수가 종료되어도 해당 lexical scope를 기억한다.
- 여기서 lexical scope에서 중요한 것은 num이라는 변수다. 바로 이 변수가 기억된다는 의미다.

- 따라서 counter.increase()가 호출되면 num이 1이 더해진 상태로 기억된다.
- 똑같이 counter.decrease()가 호출되면 num이 1이 빠진 상태로 기억된다.


```javascript
console.log(counter.increase()); //1
console.log(counter.decrease()); //0
```

- closure의 장점은 변수를 함수 안에 두고 사용하기 때문에 전역으로 만들 필요가 없다는 점이다.
- 전역으로 만들 필요가 없다는 것은 누군가에 의해 수정될 가능성이 없다는 말과 동일하다. 더 안전하다는 의미다.


- 해당 예시에 즉시실행함수를 쓰지 않았다면 어떻게 될까?
- lexical 환경이 서로 달라 num이라는 변수가 공유되지 않는다.
- num이라는 변수가 공유될 필요가 있다면 IIFE로 만들어 한번만 실행되게 해야 한다.


```javascript
const counter = function(){
    let num = 0;
    return {
        increase(){
            return ++num;
        },
        decrease(){
            return --num;
        }
    }
};

console.log(counter().increase()) //1
console.log(counter().increase()) //1
console.log(counter().decrease()) //-1
console.log(counter().decrease()) //-1
console.log(counter().decrease()) //-1
console.log(counter().decrease()) //-1
console.log(counter().increase()) //1
```

- lexical 환경 공유까지 필요하지 않다면 IIFE로 만들 필요는 없다.

- 아래는 일반함수의 closure 사례로, parameter가 closure를 받는다.


```javascript
function add(num1) {
	return function sum(num2) {
		return num1 + num2;
	}
}

const addOne = add(1);

addOne(3); //add(1)으로 add()에 1을 넘긴 상태에서 3을 sum()에 넣은 것. 4
add(1)(3); //위와 동일
```

- 실제 아래와 같이 closure를 활용할 수 있다.

- 원래는 아래와 같이 switch case를 활용하여 분기를 처리했다고 해보자.


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
```

- closure를 이용하면 아래와 같이 바꿀 수 있다.


```javascript
function log(value) {
	return function (fn) {
		fn(value);
	}
}

const logFoo = log;

logFoo('foo')((v) => console.log(v));
logFoo('foo')((v) => console.info(v));
logFoo('foo')((v) => console.warn(v));
logFoo('foo')((v) => console.error(v));
```

- log에 인자를 붙이면 아래와 같이 좀 더 알아보기 쉬운 형태로 바뀐다.


```javascript
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

- 이를 IIFE로 바꾸면 아래와 같다.


```javascript
const logFoo = (function log(value) {
	return function (fn) {
		fn(value);
	}
}('foo'))

logFoo((v) => console.log(v));
logFoo((v) => console.info(v));
logFoo((v) => console.warn(v));
logFoo((v) => console.error(v));
```


- closure를 활용하는 것은 고차함수에도 쓰일 수 있다.

- 아래와 같이 써있는 코드가 있다고 해보자.


```javascript
const arr = [1,2,3,'A','B','C'];
const isNumber = (value) => typeof value === 'number';
const isString = (value) => typeof value === 'string';
arr.filter(isNumber);
```

- 아래와 같이 함수로 빼서 중복을 덜하게 만들 수 있다.


```javascript
function isTypeOf(type, value) {
  return typeof value === type;
}
const isNumber = (value) => isTypeOf('number', value);
const isString = (value) => isTypeOf('string', value);
arr.filter(isNumber);
```

- 거기서 더 나아가 closure를 활용하면 앞으로 isTypeOf()에 string형태의 type만 넣어도 기능이 작동한다.

- type검사가 자주 일어날수록 아래와 같이 바꾸는 것이 더 편할 것이다.


```javascript
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

## <span style="color:#802548">_fetch closure- execution context_</span>

- 중복 클릭을 방지하기 위한 closure 적용을 fetch문으로 만들었다.
- 이전과 다르게 if문으로 먼저 early return하는 방식으로 변경했다.
- 하지만 아무리 해도 closure가 적용되지 않았다.

```js
function createToggleBookmarkWithThen() {
    let isClicked = false; // This persists across multiple clicks

    return function toggleBookmark() {
        if (isClicked) {
            Swal.fire({
                icon: 'error',
                title: '이미 클릭하셨습니다.',
                text: '서버에서 프로세스가 진행중입니다.',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        isClicked = true; // Disable further clicks until the API is done

        fetch('/recipe/history/save', {
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ title: "Test Recipe" }),
            method: 'POST'
        })
        .then((response) => {
            if (!response.ok) {
                throw new Error('API Error');
            }
            return response.text(); // This will resolve to the response text
        })
        .then((responseText) => {
            console.log("API Response:", responseText);
			isClicked = false;
        })
        .catch((error) => {
            console.error("Error:", error);
			isClicked = false;
        })
    };
}
```

- html안에 ()()로 실제 function이 발생하게 하였다.
- 하지만 제대로 작동하지 않았다. 초기화가 되지 않았다.

```html
<button class="bookmark-btn" onclick="toggleBookmark()()">
```

- chatGpt 검색 결과, 위와 같은 방식으로 만들 경우, 실행 context가 매번 새로만들어져서, 값이 초기화된다.
- 따라서 실행컨텍스트를 먼저 만들고 이를 eventListener로 callback을 달아주라고 한다.

```js
const toggleBookmarkwithThen = createToggleBookmarkWithThen();
document.querySelector('.bookmark-btn').addEventListener('click', toggleBookmarkwithThen);
```

## <span style="color:#802548">_fetch closure async한 then과 catch_</span>


- 하지만 여전히 원하는대로 작동하지 않았다.
- 그래서 원인이 async - await인가 싶어서 async await로 진행되는 방식으로 바꿔보았다.

```js
function createToggleBookmark() {
    let isClicked = false; // This will persist between clicks

    return async function toggleBookmark() {
        console.log(isClicked);
        if (isClicked) {
            Swal.fire({
                icon: 'error',
                title: '이미 클릭하셨습니다.',
                text: '서버에서 프로세스가 진행중입니다.',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        isClicked = true; // Disable further clicks until the API is done

        try {
            const response = await fetch('/recipe/history/save', {
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ title: "Test Recipe" }),
                method: 'POST'
            });

            if (!response.ok) {
                throw new Error('API Error');
            }

            const responseText = await response.text(); // Get the response text
            console.log("API Response:", responseText);

        } catch (error) {
            console.error("Error:", error);
        } 
    };
}

const toggleBookmark = createToggleBookmark();
document.querySelector('.bookmark-btn').addEventListener('click', toggleBookmark);
```

- 하지만 여전히 작동하지 않았다.
- 원인은 then과 catch가 async하기 때문이었다. 오직 finally 구문만이 Promise가 resolve or reject 된 이후 발동한다.
- 따라서 실행컨텍스트에서 자유 변수인 isClicked가 변경되기 위해선 finally 구문에서 값을 변경해야 했다.

```js
function createToggleBookmark() {
    let isClicked = false; // This will persist between clicks

    return async function toggleBookmark() {
        console.log(isClicked);
        if (isClicked) {
            Swal.fire({
                icon: 'error',
                title: '이미 클릭하셨습니다.',
                text: '서버에서 프로세스가 진행중입니다.',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        isClicked = true; // Disable further clicks until the API is done

        try {
            const response = await fetch('/recipe/history/save', {
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ title: "Test Recipe" }),
                method: 'POST'
            });

            if (!response.ok) {
                throw new Error('API Error');
            }

            const responseText = await response.text(); // Get the response text
            console.log("API Response:", responseText);

        } catch (error) {
            console.error("Error:", error);
        } finally {
			 isClicked = false; 
		}
    };
}
```

- then 구문으로 하게 되어도 finally만 잘 적어주면 작동한다.
- closure의 작동은 이로써 완료다.
- 중요한 것은 Promise의 개념이었다.
	- Promise가 resolve 혹은 reject되는 것을 보장할 수 있는 것이 finally라는 개념을 깨달았다.
	- then, catch, 그냥 async - await는 모두 async하고 Promise가 resolve 혹은 reject가 확정되지 않은 상태이다.
	- 따라서 변수도 초기화되지 않는다.

```js
function createToggleBookmarkWithThen() {
    let isClicked = false; // This persists across multiple clicks

    return function toggleBookmark() {
        console.log(isClicked);
        if (isClicked) {
            Swal.fire({
                icon: 'error',
                title: '이미 클릭하셨습니다.',
                text: '서버에서 프로세스가 진행중입니다.',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        isClicked = true; // Disable further clicks until the API is done

        fetch('/recipe/history/save', {
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ title: "Test Recipe" }),
            method: 'POST'
        })
        .then((response) => {
            if (!response.ok) {
                throw new Error('API Error');
            }
            return response.text(); // This will resolve to the response text
        })
        .then((responseText) => {
            console.log("API Response:", responseText);
        })
        .catch((error) => {
            console.error("Error:", error);
        })
        .finally(() => {
            isClicked = false; // Always re-enable clicking after the API call finishes
        });
    };
}
```

- 백엔드에서는 아래와 같이 4초 이상이 걸린다고 로직만 가정하였다.

```java
@PostMapping("/recipe/history/save")
@ResponseBody
public void saveRecipeHistory(@RequestBody RecipeHistroyRequsetDTO recipeHistroyRequsetDTO) throws InterruptedException {

	log.info("rrrr");
	Thread.sleep(4000);
	//recipeHistoryService.saveRecipeHisotry(recipeHistroyRequsetDTO);
}
```


## <span style="color:#802548">_fetch closure finally가 아닌 곳에서 활용하기_</span>

- 저장하기를 누르면, 서버에서 오래걸리는 경우, 오래 걸리는 것만큼 중복 클릭을 하지 못하게 막았다.
- 하지만, 위의 경우로는 이미 저장된 이후에 다시 저장하게 하는 것을 막는 것이 안 된다.
- 따라서 아래와 같이 자유 변수를 하나 더 추가했다.
    - 추가하고서 await로 서버에서 받은 값을 저장하면 막을 수 있다.

```js
function createToggleBookmark() {
    let isClicked = false; // This will persist between clicks
    let recipeSeq = 0; // Initially, there's no recipe sequence

    return async function toggleBookmark() {
        let bookmarkBtn = document.querySelector('.bookmark-btn');
        let bookmarkIcon = document.getElementById('bookmarkIcon');


        if (!bookmarkBtn.classList.contains('bookmarked')) {
            bookmarkBtn.classList.add('bookmarked');
            bookmarkIcon.src = '/image/star2.png'; 
        }

        console.log(isClicked);
        if (isClicked) {
            Swal.fire({
                icon: 'error',
                title: '이미 클릭하셨습니다.',
                text: '서버에서 프로세스가 진행중입니다.',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        // Block further requests if recipeSeq is already set (meaning data has been saved)
        if (recipeSeq !== 0) {
            Swal.fire({
                icon: 'warning',
                title: '이미 저장된 데이터입니다.',
                text: '저장은 이미 완료되었습니다.',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });
            return;
        }

        isClicked = true; // Disable further clicks until the API is done
        const title = document.querySelector(".recipe-title").textContent;
        const outputContent = document.querySelector(".recipe-info").innerHTML; // HTML itself as a string
        const usage = document.querySelector("input[name='usage']").value;
        const menu = document.querySelector("input[name='menu']").value;
        const taste = document.querySelector("input[name='taste']").value;
        const level = document.querySelector("input[name='level']").value;

        const data = {
            title,
            recipeCondition: { usage, menu, taste, level },
            outputContent
        };

        try {
            const response = await fetch('/recipe/history/save', {
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(data),
                method: 'POST'
            });

            if (!response.ok) {
                removeBoomark();
                throw new Error('API Error');
            }

            recipeSeq = await response.json(); // Store the server response (recipeSeq)

            Swal.fire({
                icon: 'success',
                title: '저장 성공!',
                text: '저장이 완료되었습니다.',
                confirmButtonColor: '#ff7f50',
                confirmButtonText: '확인',
            });

        } catch (error) {
            removeBoomark();
            console.error("Error:", error);
        } finally {
            isClicked = false; // Re-enable clicking after the response is handled (success or failure)
        }
    };
}

// Attach the function to the button event
const toggleBookmark = createToggleBookmark();
document.querySelector('.bookmark-btn').addEventListener('click', toggleBookmark);
```


## <span style="color:#802548">_fetch closure- passowrdCheck를 통해 얻은 자유변수 값을 다른 함수에서 사용할 수 있게 하기_</span>

- 전역변수로 되어있는 isPasswordCorrect 아래처럼 안에 넣어 로컬 변수로 바꿔준다.

```js
const createPasswordChecker = () => {
    let isPasswordCorrect = false;

    async function checkPw() {
        if (password.value.length === 0) {
            isPasswordCorrect = false;
            pwIconBox.innerHTML = "";
            changePwBtn.disabled = true;
            validateForm();
            return Promise.resolve(false); // Resolve immediately with false if input is empty
        }

        return fetch("/mypage/checkPassword", {
            method: "POST",
            headers: { "Content-Type": "application/json" },
            body: JSON.stringify({ currentPassword: password.value })
        })
        .then(response => response.json())
        .then(data => {
            isPasswordCorrect = data;

            // Update UI
            pwIconBox.innerHTML = isPasswordCorrect
                ? `<i class="fa-solid fa-circle-check pwCheckIcon" style="color: #5cd85a;"></i>`
                : `<i class="fa-solid fa-circle-xmark pwCheckIcon" style="color: #f55735;"></i>`;

            changePwBtn.disabled = !isPasswordCorrect;
            validateForm();

            return isPasswordCorrect; // Return the updated value
        })
        .catch(error => {
            console.error('Error:', error);
            return false; // In case of error, assume password is incorrect
        });
    }
   
    return { checkPw};
};
```

- 만약 passsword 입력에 debounce를 주어야 한다면 createPasswordChecker를 debounce로 감싸지 않는다.
- 아래처럼 나중에 별도의 변수를 만들어 감싸준다.

```js
const createPasswordChecker = () => {
    let isPasswordCorrect = false;

    async function checkPw() {
        .
        .
    }

    const debounceCheckPw = debounce(checkPw, 300);

    return {checkPw: debounceCheckPw}
}
```


- 그럼 password를 입력할 때, 실행컨텍스를 만든다.
- 해당 isPasswordValid 변수를 checkPw()도 return하게 한다.

```js
const passwordChecker = createPasswordChecker();

password.addEventListener("keyup", async () => { //debounce 적용 필요
    const result = await passwordChecker.checkPw();
    console.log("Password check complete:", result);
});

```

- 해당 변수를 다른 곳에서 사용하기 위해서 변수만을 return하는 함수도 별도로 만들어준다.

```js
const createPasswordChecker = () => {
    let isPasswordCorrect = false;
    async function checkPw() {
        .
        .
        .
        //isPasswordCorrect를 서버의 응답에 따라 변경함.

    }

    function isPasswordValid() {
        return isPasswordCorrect;
    }

    return { checkPw, isPasswordValid };
 
}
```



- 아래와 같이 별도의 함수에서도 clousre에서 변경시킨 자유 변수를 사용할 수 있게 된다.
    - passwordChecker.isPasswordValid()를 보면 이전에 저장된 자유 변수 값들이 그대로 활용가능하다.

```js
function validateForm() {
    const isNickNameValid = nickName.value.trim().length >= 2 && nickName.value.trim().length <= 11;
    const isNewPwValid = newPW.value.length > 0 && newPW.value === newPWCheck.value;

    const nickNameLengthMessage = document.getElementById("nickNameLengthMessage");
    nickNameLengthMessage.style.display = isNickNameValid ? "none" : "block";

    if (newPasswordSection.style.display === "none") {
        changeInfoBtn.disabled = !(passwordChecker.isPasswordValid() && isNickNameValid);
    } else {
        changeInfoBtn.disabled = !(isNickNameValid && isNewPwValid);
    }

    changePwBtn.disabled = !(passwordChecker.isPasswordValid() && isNickNameValid);
}
```




## <span style="color:#802548">_closure와 고차함수_</span>

- closure를 이용하면 고차함수를 사용할 수 있다.
- 아래와 같은 형태의 함수가 있다고 해보자.

```js
function delay(time) { 
    return new Promise((resolve) => setTimeout(()=> resolve(undefiend),time)); 
}

const notifyOrderSucccess = async (orderList) => {
    await delay(2000);
    orderList.forEach((order) => document.querySelector('#log').textContent += `${order}가 완료됐습니다.<br/>`); 
}

function orderCoffee(el, orderList) {
    if(!el || Array.isArray(orderList)) { 
        return;
    }

    el.addEventListener('click', () => notifyOrderSucccess(orderList))
}
```

- 위의 함수는 고차함수로 만들지 않고 eventListner의 function을 직접 () =>로 달았다.
- 아래처럼 closure를 이용한 고차함수로 만들면 () => 없이 그냥 달아줄 수 있다.
- orderCoffee의 orderList parameter는 free variable로서 활용된다.

```js
function delay(time) { 
    return new Promise((resolve) => setTimeout(()=> resolve(undefiend),time)); 
}

function notifyOrderSucccess(orderList) {
    return async function() {
        await delay(2000);
        orderList.forEach((order) => document.querySelector('#log').textContent += `${order}가 완료됐습니다.<br/>`); 
    }
}

function orderCoffee(el, orderList) {
    if(!el || Array.isArray(orderList)) {
        return;
    }

    el.addEventListener('click', notifyOrderSucccess(orderList)) 
}
```

- async의 위치도 중요하다. await가 포함되어 있는 function 바로 상위에 async를 달아줘야 한다.
- 즉 notifyOrderSucccess()에다가 async를 달아주게 되면 await가 제대로 작동하지 않게 된다.
- async는 아래 익명함수에다가 작성해줘야 한다.

```js
function delay(time) { 
    return new Promise((resolve) => setTimeout(()=> resolve(undefiend),time)); 
}

async function notifyOrderSucccess(orderList) { //async는 아래 
    return function() {
        await delay(2000);
        orderList.forEach((order) => document.querySelector('#log').textContent += `${order}가 완료됐습니다.<br/>`); 
    }
}

function orderCoffee(el, orderList) {
    if(!el || Array.isArray(orderList)) {
        return;
    }

    el.addEventListener('click', notifyOrderSucccess(orderList)) 
}
```


- 화살표 함수로 바꿔주면 아래와 같은 형태로 바꿔주면 된다.
- orderList가 free variable로 남아서 계속 사용할 수 있는 변수로 남게 된다.

```js
function delay(time) { 
    return new Promise((resolve) => setTimeout(()=> resolve(),time)); 
}

const notifyOrderSucccess = async (orderList) => () => {
    await delay(2000);
    orderList.forEach((order) => document.querySelector('#log').textContent += `${order}가 완료됐습니다.<br/>`); 
}

function orderCoffee(el, orderList) {
    if(!el || Array.isArray(orderList)) { 
        return;
    }

    el.addEventListener('click', notifyOrderSucccess(orderList))
}
```


- 이러한 closure의 속성을 이용하면 event를 이용한 handler를 쓸 때도 parameter를 번잡하게 쓸 필요가 없다.
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



## <span style="color:#802548">_closure를 사용해서 중복클릭 방지하기_</span>
- 과거에 내가 만든 중복클릭 방지는 전역변수, setTimeout을 이용했었다.
```js
var reqFlag = false;

function reqCheck(){
    if(reqCheck == false){
            reqFlag = true;
            return false;
    }

    setTimeout(function(){
            reqFlag = false;
    },3000)

    return true;
}

function handleButtonClick() {
    $('#button').on('click',function() {
        if(reqCheck()) {
            return;
        }
        .
        .
        .
        //ajax 함수
    })
}
```

- 저번에도 closure로 바꿔보고 싶었지만 실패했었다.
- 그래서 이번에 한번 바꿔봤다.
- 처음엔 아래와 같이 만들었다.
- 그런데 문제는 의도한대로 되지 않는다는 점이었다.
- 내가 의도한 것은 함수가 결과를 받아올 때까지 버튼이 눌리지 않는 것인데, 잘 작동하지 않았다.
- 계속 clicked가 false로 처리되었다.
- var의 문제인가 했지만 그런 것은 아니었다. 즉시실행함수로 바꿔도 달라지는 점은 없었다.
```js
function sendRequest(fn) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            fn(data)
        }
    }
}

function handleButtonClick() {
    $("#button").on('click',function() {
        .
        .
        .
        var sendData = {};
        sendData.payload = id;

        sendRequest(testAjax)(sendData);
    })
}

function testAjax() {
    $.ajax({
        .
        .
        .
        success: function(res, status, xhr){
            if(res.statusCode === '200') {
                location.href = '/teenAgerCardIssue';
            }
        },
        error: function(error) {
            alert(error);
        } ,
        complete: function() {
            checkMobileDeviceOs();
        }
    })
}
```



- 그래서 원인을 알아냈는데, 그건 비동기이면서, event Listener에 달고, closure를 써서 복잡하게 꼬여있던 것이었다.
- closure의 자유변수를 활용하려면 하나의 함수 안에서 이뤄져야 했다. 
- success callback, error callback, complete callback에서 모두 clicked를 다시 false로 바꿔줘야 했다.
- 먼저 함수를 parameter로 넘기지 않고, ajax를 분리하지 않고 closure가 쓰인 함수로 옮겼다.
```js
function sendRequest() {
    var clicked = false;
    return function() {
        if(!clicked) {
            clicked = true;
            var sendData = {};
            sendData.payload = id;
            $.ajax({
                .
                .
                .
                success: function(res, status, xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                },
                error: function(error) {
                    alert(error);
                     clicked = false;
                } ,
                complete: function() {
                    checkMobileDeviceOs();
                     clicked = false;
                }
            })

        }
    }
}

function handleButtonClick() {
    $("#button").on('click',function() {
       sendRequest()();
    })
}
```

- 그럼에도 여전히 clicked가 false로 초기화됐다.
- 원인을 알아보니 EventLisnter가 문제는 아니었다.
- 문제는 click 시마다 sendRequest()()를 호출하게 되면 매번 새로운 실행 컨텍스트가 생기는 게 문제였다.
- 그래서 실행 컨텍스트를 저장해두는 변수를 만들었다.
- 그랬더니 이제 원하는 대로 작동하기 시작했다!
```js
function sendRequest() {
    var clicked = false;
    return function() {
        if(!clicked) {
            clicked = true;
            var sendData = {};
            sendData.payload = id;
            $.ajax({
                .
                .
                .
                success: function(res, status, xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                },
                error: function(error) {
                    alert(error);
                     clicked = false;
                } ,
                complete: function() {
                    checkMobileDeviceOs();
                     clicked = false;
                }
            })

        }
    }
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
       fetchData();
    })
}
```

- 이제 기본적인 작동은 되니 함수를 분리해주고 싶었다.
- 그래서 다시 parameter로 주는 방법을 생각해보았다.
- 우선 data부터 parameter로 넘겨주기로 했다.
- 외부함수의 parameter보단 내부함수의 parameter에서 넣는 게 더 좋다 판단하였다.
- 처음에 실행컨텍스트를 불러올 때 함수를 parameter를 받을 수 있기 때문에, 그건 함수에게 넘겨주려 생각했다.
```js
function sendRequest() {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
             $.ajax({
                .
                .
                data:data
                success: function(res, status, xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                },
                error: function(error) {
                    alert(error);
                     clicked = false;
                } ,
                complete: function() {
                    checkMobileDeviceOs();
                     clicked = false;
                }
            })

        }
    }
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
        fetchData(sendData);
    })
}
```

- 이제는 ajax 함수를 바깥으로 빼보고 싶었다.
- 그랬더니 clicked가 closure함수가 속한 범위에서 벗어나 버렸다.
- 그래서 이를 해결하기 위해 parameter로 넘겨봤지만 아무 소용이 없었다.
- 계속 clicked가 false로 초기화됐다.
```js
function sendRequest() {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            textAjax(data, clicked);

        }
    }
}

function testAjax(data, clicked) {
    $.ajax({
        .
        .
        data:data
        success: function(res, status, xhr){
            if(res.statusCode === '200') {
                location.href = '/teenAgerCardIssue';
            }
            clicked = false;
        },
        error: function(error) {
            alert(error);
            clicked = false;
        } ,
        complete: function() {
            checkMobileDeviceOs();
            clicked = false;
        }
    })
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```

- 결국 과거의 jquery ajax 활용법으론 불가능하다고 판단했다.
- Promise와 비슷하게 .then()처럼 이어가는 방식의 ajax를 사용하기로 했다.
- 이렇게 하면 ajax를 분리하면서 자유변수는 그대로 살릴 수 있었다.
```js
function sendRequest() {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            textAjax(data)
                .done(function(res,status,xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```

- 이제는 여러가지 ajax 함수를 받을 수 있게 함수를 parameter로 받으려 했다.
- ajaxFn으로 받는 것은 아래와 같이 간단하게 바꿀 수 있었다.
- 대신 처음 실행컨텍스트를 정할 때 원하는 ajax 함수를 같이 넘기게 바꿔야 한다.
```js
function sendRequest(ajaxFn) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            ajaxFn(data)
                .done(function(res,status,xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

var fetchData = sendRequest(testAjax);

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```

- 그런데 successCallback도 같이 ajax와 넣어두면 이걸 공통함수로 뺄 수 있겠다는 생각이 들었다.
- 그래서 successCallback도 같이 parameter로 받게 바꿨다.
- successCallback 함수는 res를 parameter로 받아야한다.
- 그러나 sendRequest에서 굳이 parameter를 명시하지 않아도 알아서 잘 받아진다. 이유는 모르겠다.
```js
function sendRequest(ajaxFn, successCallback) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            ajaxFn(data)
                .done(function(res,status,xhr){
                    successCallback(res)
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

function successCallback(res) {
    var popupText = "";
    var popupHtml = "";
    if(res.statusCode === '200') {
        location.href = '/teenAgerCardIssue';
    } else if (res.statusCode === '199') {
        popupText = "비밀번호가 1회 오류입니다."
    } else if (res.statusCode === '198') {
        popupText = "비밀번호가 2회 오류입니다."
    } else {
        location.href = '/teenAgerError'
        return;
    }

    poupHtml = $("#errorGuide").text(popupText);
    popupHtml.html(popupHtml.html().replace(/\n/g,'<br>'));

    Layer.open("#errorPopup");
}

var fetchData = sendRequest(testAjax, successCallback);

function handleButtonClick() {
    $("#button").on('click',function() {
        var sendData = {};
        sendData.payload = id;
       fetchData(sendData);
    })
}
```


- 모듈화를 해놨으니 다른 공통 함수로 빼놓을 수 있다.
- 공통함수 js에 넣을 수 있다는 의미다.
```js
//common.js

function sendRequestOnlyOnce(ajaxFn, successCallback) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            ajaxFn(data)
                .done(function(res,status,xhr){
                    successCallback(res)
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}
```

- 이제 ajax들은 모두 common.js에서 가져온 sendRequestOnlyOnce 함수로 wrapping할 수 있다.
- 그러면 별다른 처리 없이 간단하게 아래와 같은 순서로 하면 중복클릭방지가 저절로 들어가는 것이다.
- script import --> ajax 함수 정의 --> ajax의 successCallback 정의 --> 실행컨텍스트 저장을 위한 변수 정의 --> data collection --> eventListener에서 함수실행
- 다만 여기서 ajax에 절대로 async:false 옵션이 있으면 안 된다.
- 그럼 어떤 방식(버튼 disabled, 전역변수 flag 등..)을 쓰든 중복클릭 방지가 작동하지 않는다.
```js
//teenAger.jsp
<script src ="/my/common.js"/>



//원하는 ajax함수
function testAjax(data) {
    var leftDay = "${leftDay}";
    if (parseInt(leftDay) >= 1) {
        preventRequestAndShowPopup(parseInt(lefyDay));
        return;
    }

    return $.ajax({
                url:'~~',
                method:'post',
                dataType:'json',
                data:data
            })
}

//원하는 ajax의 successCallback
function successCallback(res) {
    .
    .
}

// 실행컨텍스트 저장을 위해 wraaping하는 함수
var fetchData = sendRequestOnlyOnce(testAjax, successCallback);


function handleButtonClick() {
    $("#button").on('click',function() {
        var isDisabled = $(this).hasClass('disabled');
        if(isDisabled) {
            return;
        }

        //data collecting
        var sendData = {};

        //실제 함수 실행
       fetchData(sendData);
    })
}
```


- 해당 내용을 axios/ts로 변환해보자.
- 나는 ajax의 형태를 아래와 같이 사용했다.
```js
const apiRootPath = process.env.VUE_APP_API;
let prevRequest: AxiosRequestConfig | null = null;
export const instance = axios.create({
  timeout: 10 * 1000,
  baseURL: apiRootPath,
});

instance.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    if (config.url !== '/login') {
      const store = useMemberStore();
      const { accessToken } = storeToRefs(store);

      config.headers['authorization'] = `Bearer ${accessToken.value}`;
    }

    prevRequest = config;
    return config;
  },
  error => {
    return Promise.reject(error);
  }
);

instance.interceptors.response.use(
  response => {
    return response;
  },
  async (error: AxiosError) => {
    if (error.response?.status === 401) {
      const store = useMemberStore();
      const { loginMember } = storeToRefs(store);
      const { updateAccessToken } = store;
      const authenticPrevRequest = prevRequest;
      const accessTokenResponse = await extendSignInStatus(loginMember.value);
      const accessToken = accessTokenResponse.data;
      updateAccessToken(accessToken);
      const updatedRequestConfig = {
        ...authenticPrevRequest,
        headers: {
          authorization: `Bearer ${accessToken}`,
          'content-type': 'application/json',
        },
      };

      const result = await axios(updatedRequestConfig as AxiosRequestConfig);
      return result;
    }

    if (error.response?.data.message) {
      alert(error.response?.data.message);
    }

    return Promise.reject(error);
  }
);
```

- 실제 ajax 함수는 아래와 같이 만들수 있다.
```js
export const login = (loginRequest: Pick<Member,'memberId' | 'memberPassword'>) => {
  return instance({
    url: '/signin',
    method: 'post',
    data: loginRequest,
  });
};
```

- 그럼 위와 같은 ajax 함수를 받아오기 위한 형태로 아래와 같이 만들어준다.
- parameter는 뭐로 들어올지 몰라 any로 해놨다. 나중에 규칙이 추가된다면 any가 아닌 다른것으로 바꿔보자.
```js
export const sendRequestOnlyOnce = (ajaxFn: (parameter: any) => AxiosPromise<any>, successCallback: (response: AxiosResponse) => void) => {
    let clicked = false;
    return (parameter: any) => {
        console.log(clicked);
        if(!clicked) {
            clicked = true;
            ajaxFn(parameter)
                .then(res => {
                    successCallback(res)
                    clicked = false;
                }).catch(function(error){
                    alert(error);
                    clicked = false;
                }).finally(function(){
                    clicked = false;
                })

        }
    }
}
```

- successCallback은 아래와 같이 적용해준다.
```js
const successCallback = (response: AxiosResponse): void => {
    if (response.status === 200) {
    const accessToken = response.data.accessToken;
    const refreshToken = response.data.refreshToken;
    const memberNo = response.data.memberNo;

    alert('로그인이 완료되었습니다.');
    changeLoginMember(memberId.value, accessToken, refreshToken, memberNo);
    router.push('/');
    } 
}
```

- 그럼 실제 closure는 아래와 같이 호출한다.
- 아직 실전 test는 안해봤는데, 개인 프로젝트 하게 되면 test를 해보려고 한다.
```js
const fetchData = sendRequestOnlyOnce(login, successCallback);

async function handleLoginClick() {
    const isValid = (await useValidateForm(form, isFormValid))?.value;
    if (isValid) {
        const data = {
            memberId: memberId.value,
            memberPassword: memberPassword.value,
        };

        fetchData(data);
    }
}
```


- 물론 아래와 같이 전역변수를 쓰는 것도 가능하다.
- async는 false로 쓰면 절대 안된다.
- async는 false로 쓰면 ajax의 success에서 return한 값을 쓰겠다는 의미다.
- 이런 식으로 쓰면 안되고, 콜백함수를 활용해줘야 한다.
```js
var dualExistsFlag = false;

function ajax(){
    if($(this).hasClass('disabled')) {
        return;
    }

    if(dualExistsFlag) {
        return;
    }
    dualExistsFlag = true;

    $.ajax({
        url:'ddd',
        method:'post',
        data:data,
        dataType:'json',
        success:function(res,status,xhr){
            .
            .
            .

        },
        error:function(){
            .
            .
            .

        },
        complete:function(){
            dualExistsFlag = false;
        }
    })
}
```

## <span style="color:#802548">_ajax closure_</span>

- fetch가 아닌 ajax로 적용한 경우 아래처럼 만들 수 있다.

```js
function sendRequest() {
    let clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            textAjax(data)
                .done(function(res,status,xhr){
                    if(res.statusCode === '200') {
                        location.href = '/teenAgerCardIssue';
                    }
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}
```