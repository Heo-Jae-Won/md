## <span style="color:#802548">_1.스프레드 문법, destructuring 문법_</span>
- 하나로 뭉쳐 있는 여러 값의 집합을 펼쳐서 개별 값의 목록으로 만든다.
- iterable 객체만 spread 문법을 활용할 수 있다.
```javascript
console.log(...[1,2,3]); //1,2,3
console.log(...'hello');//h e l l o
console.log(...new Map([['a','1'],['b','2']])); // ['a','1'], ['b','2']
console.log(...new Set([1,2,3])); //1 2 3

console.log(...{a:1, b:2}); //error. Found non-callable @@iterator
```

```javascript
var arr = [1,2,3];
var max = Math.max.apply(null, arr); //3

const arr = [1,2,3];
const max  = Math.max(...arrr); //3
```

- 스프레드 문법은 기존 ES5의 method를 대체할 수 있다.
```javascript
var arr = [1,2].concat([3,4]);
console.log(arr); //[1,2,3,4]

const arr =[...[1,2], ...[3,4]];
console.log(arr); //[1,2,3,4]


var arr1 = [1,4];
var arr2 = [2,3];
arr1.splice(1,0,arr2); //[1,[2,3],4];

Array.prototype.splice.apply(arr1,[1.0].concat(arr2)); //[1,2,3,4]
arr1.splice(1,0,...arr2); //[1,2,3,4];
```

```javascript
function sum() {
    var args = Array.prototype.slice.call(arguments);

    return args.reduce(function (pre, cur){
        return pre + cur;
    }, 0);
}

function sum(){
    return [...arguments].reduce((pre, cur) => pre + cur, 0);
}

const sum = (...args) => args.reduce((pre, cur) => pre + cur, 0);
console.log(sum(1,2,3)); //6
```

```javascript
const arrayLike = {
    0: 1,
    1: 2,
    2: 3,
    length: 3
}

const arr = [...arrayLike];
Array.from(arrayLike); //유사 배열 객체 또는 이터러블을 배열로 변환한다.
```

- 스프레드 문법으로 객체의 property value도 바꿀 수 있다.

```javascript
const obj = {x:1, y:2};
const merged = Object.assign({}, {x: 1, y: 2}, {y: 10, z: 3})
const merged = {...{x: 1, y: 2}, ...{y:10, z:3}}
console.log(merged); // {x:1, y:10, z:3} 나중에 들어온 property가 덮어씀
```

- destructuring 문법도 있다.

```javascript
var arr = [1,2,3];
var one = arr[0];
var two = arr[1];
var three = arr[2];

var [one,two,three] = [1,2,3];

const arr = [1,2,3];
const [one,two,three] = arr;
console.log(one,two,three);
```

- 아래와 같이 쓰면 오류가 나거나 의도하지 않은 결과를 초래할 수 있다.
```javascript
const [x,y] //SyntaxError
const [a,b] = {} //TypeError: {} is not iterable
const [c,d] = 1;
console.log(c,d); //1 undefined
const [e,f] = [1,2,3];
console.log(e,f); //1 2 
```

```javascript
const [a,b,c=3] =[1,2];
console.log(a,b,c) ; //1 2 3. 기본값 적용가능

const [e,f = 10, g = 3] = [1,2];
console.log(e,f,g); // 1 2 3. 할당된 값이 기본값보다 우선
```

- 아래는 destructuring의 사용예제다.
```javascript
function parseURL1(url = ''){
    const parsedURL = url.match(/^(\w+):\/\/([^/]+)\/(.*)$/);
    console.log(parsedURL);

    if(!parsedURL) return{};
    const [protocol,host,path] = parsedURL;
    return {
        protocol,
        host,
        path
    }
}

const parseURL1 = parseURL('https://developer.mozilla.org/ko/docs/Web/JavaScript');
console.log(parseURL1);
/*
protocol: 'https',
host: 'developer.mozilla.org',
path: 'ko/docs/Web/JavaScript'
*/
```

- 이제는 객체의 destructuring의 예시를 살펴보자.
- 아래는 ES5의 예시다.
```javascript
var user = {
    firstName: 'JaeWon',
    lastName: 'Lee'
}

var firstName = user.firstName;
var lastName = user.lastName;
console.log(firstName, lastName);
```

- 아래는 ES6의 예시다.
```javascript
const user = {
    firstName: 'JaeWon',
    lastName: 'Heo'
}
const {lastName, firstName} = user; //key를 기준으로 하기에, 순서를 뒤바꾸는 것은 아무 의미 없다.
console.log(firstName, lastName) //JaeWon Heo
```
- 배열과 마찬가지로 기본값 설정도 가능하다.

```javascript
const {firstName = 'JaeWon', lastName} = {lastName = 'Heo'};
console.log(firstName, lastName) //JaeWon Heo
const {lastName: ln, firstName: fn} = user;
console.log(fn,ln) //JaeWon Heo
```

- 함수의 parameter로 들어갈 때도 가능하다.
```javascript
function printTodo({content, completed}){
    console.log(`할일 ${content} ${completed ? '완료' : '비완료'} 상태입니다.`);
}
printTodo({id:1, content: 'HTML', completed:true});
```

- 중첩 객체의 경우 아래와 같이 사용한다.
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

- rest parameter도 사용가능하다.
```javascript
const {x, ...rest} = {x: 1, y: 2, z: 3};
console.log(x, rest); //1 {y:2, z:3}
```

- 배열의 요소가 객체라면 배열과 객체 모두 destructuring 할당이 가능하다.
```javascript
const todos = [
    {id:1, content: 'HTML', completed: true},
    {id:2, content: 'CSS', completed: false},
    {id:3, content: 'JS', completed: false},
];

const [,{id}] = todos; //2
const [{id}] = todos;  //1
const [,,{id}] = todos; //3
const [{id},{id},{id}] = todos //Uncaught SyntaxError: Identifier 'id' has already been declared 
todos.forEach(element => console.log(element.id));
```

- var 변수로 선언하면 안된다. var는 재선언이 가능하기 때문이다. 그럼 맨 마지막 선언한 id가 되어서 3이된다.
```javascript
var [,{id}] = todos; //2
var [{id}] = todos;  //1
var [,,{id}] = todos; //3
var [{id},{id},{id}] = todos //Uncaught SyntaxError: Identifier 'id' has already been declared 
console.log(id) //3
```

## <span style="color:#802548">_2.rendering과 DOM_</span>
- 브라우저의 렌더링이란, HTML,CSS, JS로 작성된 document를 parsing하여 브라우저에 출력하는 것이다.
- HTML,CSS,JS,image,font 등 렌더링에 필요한 resource 요청
- 브라우저 렌더링엔진이 HTML과 CSS 파싱해 DOM과 CSSOM 생성

- 브라우저 JS 엔진이 JS 파싱하여 AST 생성하고 바이트코드로 변환해 실행.
- 렌더 트리를 기반으로 HTML 레이아웃 계산해 HTML 요소 페인팅

- resource를 요청할 떄는 uri로 하게 되는데, 끝에 확장자가 없는 경우는 /index.html이 생략된 것이다.
- https://naver.com을 누르고 HTML을 파싱하는 도중에 외부 리소스를 로드하는 태그를 만나면, HTML 파싱이 중단되고 해당 리소스를 서버에 요청한다.
- 리소스를 받으면 다시 HTML 파싱이 시작된다.
- 따라서 리소스를 받을 때, 특히 오래 걸리는 이미지는 용량을 압축하는 게 좋다.
- script도 body가 다 만들어진 이후에 import를 하는 것이 좋다. 또는 script에 defer를 쓸 수도 있다.

- HTML문서는 바이트 형태다. meta태그의 charset을 보고 지정된 인코딩 방식을 기준으로 문자열로 변환한다.
- 변환된 HTML문서는 순수 텍스트이기 때문에, 브라우저가 이해할 수 있는 자료구조(객체)로 변환해 메모리에 저장한다.
- 변환할 때는 Node로 생성해 트리 자료구조인 DOM으로 만든다. 

- CSS파일을 요청하면 그건 DOM과 똑같이 파싱하여 CCSOM을 만든다.
- DOM과 CCSOM을 합치면 렌더링을 위한 렌더 트리로 결합된다.
- 렌더트리는 meta태그, script태그, display:none되는 노드들은 포함되지 않는다.
- 완성된 렌더트리는 레이아웃을 계산하고 페인팅 처리에 사용된다.
- 레이아웃 계산과 페인팅을 다시 실행하는 리렌더링은 성능에 악영향을 준다.
- 리렌더링은 최소로 일어나는 게 좋다.

- JS가 DOM, CCSOM을 변경하면, 렌더 트리를 기반으로 레이아웃과 페인트 과정을 거쳐 브라우저의 화면에 다시 렌더링한다.
- 이를 reflow(레이아웃 다시 계산), repaint(렌더트리 기반으로 다시 페인트)이라고 한다.

- HTML요소는 렌더링 엔진에 의해 파싱돼 요소 노드 객체로 변환된다.
- HTML 요소의 attribute는 attribute node로, 텍스트 콘텐츠는 text node로 변환된다.
- Node는 문서 노드, 요소 노드, 어트리뷰트 노드, 텍스트 노드 등이 있다.
- 문서노드는 document로 DOM의 최상위에 존재한다. 모든 DOM 트리의 노드는 document를 통해야만 한다.
- 요소노드는 HTML요소를 객체로 만든 것이다.
- 어트리뷰트노드는 요소들의 attribute를 가리키는 객체다. 어트리뷰트 노드를 조작하려면 해당 요소노드에 먼저 접근해야 한다.
- 텍스트노드는 HTML요소의 텍스트를 가리키는 객체다. 텍스트는 맨 아랫단 leaf node로 있기 때문에 요소노드에 먼저 접근해야 한다.

```html
<!DOCTYPE html>
<html>
    <body>
        <ul>
            <li id="apple">Apple</li>
            <li id="banana">banana</li>
            <li id="orange">orange</li>
        </ul>
        <script>
            const $elem = document.getElementsById('banana'); //document 문서노드 접근 -> getElementsById() 요소 노드 접근

            $elem.style.color = 'red'; //attribute node 접근하여 값 변경
        </script>
        </body>
</html>
```

```html
<!DOCTYPE html>
<html>
    <body>
        <ul>
            <li id="apple">Apple</li>
            <li id="banana">banana</li>
            <li id="orange">orange</li>
        </ul>
        <script>
            const $elem = document.querySelector('.banana'); //document 문서노드 접근 -> getElementsById() 요소 노드 접근

           $elem.style.color = 'red';
        </script>
        </body>
</html>
```

```html
<!DOCTYPE html>
<html>
    <body>
        <ul>
            <li id="apple">Apple</li>
            <li id="banana">banana</li>
            <li id="orange">orange</li>
        </ul>
        <script>
            const $elems = document.getElementsByTagName('li'); //document 문서노드 접근 -> getElementsById() 요소 노드 접근

            for(let i = 0; i < $elem.length; i++) {
                $elem[i].style.color = 'red'; //attribute node 접근하여 값 변경. 이건 매우 안좋음. live 객체일 수 있기 때문. 아래 두가지 해결책이 있는데 2번이 흔하게 쓰임.
            } 

            for(let i = $elems.length -1; i >= 0; i--){
                  $elem[i].style.color = 'red';  //Java의 ArrayList를 delete할 때처럼 index를 역순으로 돌기
            }
            [...$elems].forEach(item => { //NodeList를 배열로 만들어서 변환하기
                elem.style.color = 'red';
            })
        </script>
        </body>
</html>
```

```html
<!DOCTYPE html>
<html>
    <body>
        <ul>
            <li id="apple">Apple</li>
            <li id="banana">banana</li>
            <li id="orange">orange</li>
        </ul>
        <script>
            const $elems = document.querySelectorAll('ul > li'); //document 문서노드 접근 -> getElementsById() 요소 노드 접근

           $elems.forEach(elem => {
            elem.style.color = 'red';
           })
        </script>
        </body>
</html>
```

- node를 얻기 위한 method는 다음과 같다.

```html
<!DOCTYPE html>
<html>
    <body>
        <ul id="fruits">
            <li id="apple">Apple</li>
            <li id="banana">banana</li>
            <li id="orange">orange</li>
        </ul>
        <script>
            const $fruits = document.getElementById('fruits'); //document 문서노드 접근 -> getElementsById() 요소 노드 접근

            console.log($fruits.childNodes); // NodeList(7) [text, li.apple, text, li.banana, text, li.orange, text]. 
            console.log($fruits.children); // HTMLCollection(3) [li.apple, li.banana, li.orange]. element노드만 포함
            console.log($fruits.firstChild); // #text. text노드와 element노드 포함
            console.log($fruits.lastChild); //#text. text노드와 element노드 포함
            
            console.log($fruits.hasChildNodes()); //true. text노드와 element노드 포함
            console.log($fruits.children.length); //0. text노드 포함 X. element노드만 포함

            console.log($fruits.parentNode); //body
            
        </script>
        </body>
</html>
```

```html
<!DOCTYPE html>
<html>
    <body>
        <ul id="fruits">
            <li id="apple">Apple</li>
            <li id="banana">banana</li>
            <li id="orange">orange</li>
        </ul>
        <script>
            const $fruits = document.getElementById('fruits'); //document 문서노드 접근 -> getElementsById() 요소 노드 접근
            
            const {firstChild} = $fruits; //Element가 안 붙으면 textnode도 포함
            const {nextSibling} = firstChild; //Element가 안 붙으면 textnode도 포함
            const {previousSibling} = nextSibling; //Element가 안 붙으면 textnode도 포함

            const {firstElementChild} = $fruits; //Element가 붙으면 textnode는 미포함. li.apple
            const {nextElementSibling} = firstElementChild; //Element가 붙으면 textnode는 미포함. li.banana
            const {previousElementSibling} = nextElementChild; //Element가 붙으면 textnode는 미포함. li.apple
        </script>
        </body>
</html>
```

- innerHTML은 사용하지 말고, textContent나 insertAdjacentText를 사용하자.
- innerHTML은 xss공격에 취약하다. 파싱과정에서 그대로 실행될 수 있기 때문이다.
- xss공격을 막기 위해 DOMPurify lib을 사용하는 것도 좋은 선택이다.
```html
<!DOCTYPE html>
<html>
    <body>
        <div id="foo">Hello</div>
    </body>
    <script>
        const $foo = document.getElementById('foo');
        console.log($foo.nodeValue); //null. foo는 요소노드. 요소노드에서 nodeValue property에 접근하면 null반환

        const $textNode = $foo.firstChild;
        console.log($textNode.nodeValue); //Hello

        console.log(document.getElementById('foo').textContent); //이렇게 textContent로 접근 시, 텍스트를 반환 가능. Hello
        document.getElementById('foo').textContent = 'Hi <span>there!</span>'; //HTML은 parsing불가.
        /*
        <html>
            <body>
            'Hi<span>there!</span>'
            </body>
        </html>
        */
```

- dom을 추가하는 것은 아래와 같이 DOM API를 활용할 수 있다.
```html
<!DOCTYPE html>
<html>
    <body>
        <ul id="fruits">
            <li>Apple</li>
        </ul>
    </body>
    <script>
        const $li = document.createElement('li');
        const textNode = document.createTextNode('Banana');
        $li.appendChild(textNode);
        $fruits.appendChild($li);
    </script>
</html>
```

- 복수개의 DOM을 추가할 때는 reflow와 repaint가 n번만큼 발생하지 않게 주의해야 한다.
- 아래처럼 하게 되면 DOM을 1번에 3개를 추가하지 않고, 1개를 3번 추가한다. 
- 따라서 reflow와 repaint가 3번 일어나며 비효율적이다.
```html
<!DOCTYPE html>
<html>
    <body>
        <ul id="fruits">
        </ul>
    </body>
    <script>
        const $fruits = document.createElement('fruits');

        ['Apple','Banana','Orange'].forEach(text => {
            const $li = document.createElement('li');
            const textNode = document.createTextNode(text);

            $li.appendChild(textNode);
            $fruits.appendChild($li);
        })
    </script>
</html>
```

- 하지만 아래와 같이 하면 container div같이 불필요한 element가  많아진다.
```html
<!DOCTYPE html>
<html>
    <body>
        <ul id="fruits">
        </ul>
    </body>
    <script>
        const $fruits = document.createElement('fruits');

        const $container = document.createElement('div');

        ['Apple','Banana','Orange'].forEach(text => {
            const $li = document.createElement('li');
            const textNode = document.createTextNode(text);

            $li.appendChild(textNode);
            $container.appendChild($li); 
        })

        $fruits.appendChild($container); // 맨마지막 자식노드에 추가됨.
        $fruits.appendChild($li, ); // 맨마지막 자식노드에 추가됨.
    </script>
</html>
```

- 그럴 때 DocumentFragment를 도입한다.
```html
<!DOCTYPE html>
<html>
    <body>
        <ul id="fruits">
        </ul>
    </body>
    <script>
        const $fruits = document.createElement('fruits');

        const $fragment = document.createDocumentFragment();

        ['Apple','Banana','Orange'].forEach(text => {
            const $li = document.createElement('li');
            const textNode = document.createTextNode(text);

            $li.appendChild(textNode);
            $fragment.appendChild($li);
        })

        $fruits.appendChild($fragment);
    </script>
</html>
```

- 지정한 위치에 노드를 넣는다. 생성도 되고, 이동도 된다.
```html
<!DOCTYPE html>
<html>
    <body>
        <ul id="fruits">
            <li>Apple</li>
            <!--js로 넣어짐. <li>Orange</li>-->
            <li>Banana</li>
        </ul>
    </body>
    <script>
        const $fruits = document.getElementById('fruits'); 

        const $li = document.createElement('li');
        const textNode = document.createTextNode('Orange');

        $li.appendChild(textNode);
        $fruits.insertBefore($li, $fruits.lastElementChild); //두번째 인자는 $fruits의 자식노드여야만 한다. null이면 맨끝에 추가
    </script>
</html>
```

```html
<!DOCTYPE html>
<html>
    <body>
        <ul id="fruits">
            <li>Apple</li>
        </ul>
    </body>
    <script>
        const $fruits = document.getElementById('fruits'); 
        const $apple = $fruits.firstElementChild;

        const $shallowClone = $apple.cloneNode(); //얕은복사. text복사 X

        $shallow.textContent = 'Banana';
        $fruits.appendChild($shallowClone);
        /*
            <ul id="fruits">
                <li>Apple</li>
                <li>Banana</li>
            </ul>
        */

        const $deepClone = $fruits.clonedNode(true); //깊은복사. text복사 O
        $fruits.appendChild($deepClone);
        /*
             <ul id="fruits">
                <li>Apple</li>
                <li>Banana</li>
                <ul id="fruits">
                    <li>Apple</li>
                    <li>Banana</li>
                </ul>
            </ul>
        */
    </script>
</html>
```

```html
<!DOCTYPE html>
<html>
    <body>
        <ul id="fruits">
            <li>Apple</li>
            <!-- 위의 li는  <li>Banana</li>로 replace됨-->
        </ul>
    </body>
    <script>
        const $fruits = document.getElementById('fruits'); 

        const $newChild = document.createElement('li');
        $newChild.textContent = 'Banana';

        $fruits.replaceChild($newChild, $fruits.firstElementChild);
    </script>
</html>
```

```html
<!DOCTYPE html>
<html>
    <body>
        <ul id="fruits">
            <li>Apple</li>
           <!--오른쪽 Banana는 JS실행되면 사라짐. removeChild--> <li>Banana</li>
        </ul>
    </body>
    <script>
        const $fruits = document.getElementById('fruits'); 

        $fruits.removeChild($fruits.lastElementChild);
    </script>
</html>
```

- 요소노드의 초기 상태는 attribute노드가 관리한다
- 요소노드의 최신 상태는 DOM property가 관리한다.
- 아래는 attribute노드를 바꾼 것이다.

```html
<!DOCTYPE html>
<html>
    <body>
        <input id="user" type="text" value="jaewon">
    </body>
    <script>
        document.getElementById('user').setAttribute('value','foo');
    </script>
</html>
```

- 아래는 DOM property를 바꾼 것이다.
- DOM property를 바꿔도 attribute node의 값은 변하지 않는다. 즉 최신값이 갱신되도 초기 값은 유지된다.
- 둘은 각자 개별로 관리된다는 의미이다.
```html
<!DOCTYPE html>
<html>
    <body>
        <input id="user" type="text" value="jaewon">
    </body>
    <script>
        const $input = document.getElementById('user');

        $input.oninput = () => {
            console.log('value 프로퍼티 값', $input.value);
        }

        $input.value = 'foo';
        console.log('value 어트리뷰트 값', $input.getAttribute('value')); // jaewon
    </script>
</html>
```

- attribute node에 뭔가 부여하고 싶다면, 표준인 data-를 활용하자.
```html
<!DOCTYPE html>
<html>
    <body>
        <ul class="user">
            <li data-user-id="heo" data-role="admin">heo</li>
            <li data-user-id="seo" data-role="subscriber">seo</li>
        </ul>
    </body>
    <script>
        const users = [...document.querySelector('.user').children];

        const user = users.find(user => user.dataset.userId === 'heo');
        console.log(user.dataset.role); //admin
    </script>
</html>
```

- 인라인으로 CSS 스타일을 조작하려면 아래와 같이 할 수 있다.

```html
```html
<!DOCTYPE html>
<html>
    <body>
       <div style="color: red">Hello World!</div>
    </body>
    <script>
        const $div = document.querySelector('div');

        console.log($div.style);

        $div.style.color = 'blue';
        $div.style.width = '100px';
        $div.style.height = '20px';
        $div.style.backgroundColor = 'yellow';
        $div.style['background-color'] = 'yellow';
    </script>
</html>
```

- className을 조작하고 싶다면 아래와 같이 할 수 있다.
- className보다는 classList로 하는 게 더 편하다.

```html
<!DOCTYPE html>
<html>
    <body>
       <div class="box red">Hello World!</div>
    </body>
    <script>
        const $box = document.querySelector('.box');

        $box.classList.replace('red','blue'); //box blue
        $box.classList.add('foo'); //box red foo
        $box.classList.add('foo','bar'); //box red foo bar
        $box.classList.remove('foo'); //box red
        $box.classList.remove('box'); // red
        $box.classList.contains('box')//true
        $box.classList.contains('blue')//false
        $box.classList.toggle('foo'); //box red foo
        $box.classList.toggle('foo'); //box red
    </script>
</html>
```


## <span style="color:#802548">_3.에러처리_</span>
- error는 예상치 못한 곳에서 날 수 있다.
```javascript
const $button = document.querySelector('button'); //button이 없으면 여기서 오류가 나야 하지만, null을 반환함.
$button.classList.add('disabled'); //TypeError: cannot read property 'classList' of null
$button?.classList.add('disabled'); 
```

- try ~ catch문에 넣는 게 필요하다.
```javascript
try {

}catch(e) {
    console.error(e);
}finally {

}
```
- 아래와 같이 try ~ catch문에 넣으면 된다.
```javascript
const repeat = (n,f) => {
    if(typeof f !=='function'){
        throw new TypeError('f must be a function');
    }

    for(var i = 0; i < n; i++){
        f(i);
    }
}

try{
    repeat(2,1);
}catch(e){
    console.error(e); //TypeError: f must be a function
}
```

- error는 계속 전파되어 catch가 있는 곳에서 잡힌다.
- Java랑 똑같다. catch하는 곳에서 잡힌다. 
- 다만 setTimeout이나 promise의 then은 호출자가 없기 때문에 throw된 에러를 catch할 수가 없다.
- 따라서 setTimeout이나 promise의 then에서는 try ~ catch를 사용할 수 없다. 사용하면 catch가 안돼 프로그램이 종료된다.
```javascript
const foo = () => {
    throw Error('foo에서 발생한 에러');
}

const bar = () => {
    foo();
}

const baz = () => {
    bar();
}

try {
    baz();
} catch(e) {
    console.error(e);
}
```

## <span style="color:#802548">_4. 모듈_</span>
- 모듈이란 애플리케이션을 구성하는 개별요소로 재사용 가능한 코드 조각이다.
- 모듈은 기능을 기준으로 파일 단위로 분리한다.
- 모듈을 가져다 쓰려면 해당 모듈을 export해야한다.

```javascript
export const pi = Math.PI;

export function square(x) {
    return x * x;
}

export class Person {
    constructor(name) {
        this.name = name;
    }
}
```

```javascript
 const pi = Math.PI;

 function square(x) {
    return x * x;
}

 class Person {
    constructor(name) {
        this.name = name;
    }
}
export {pi, square, Person};
```

- export default도 있다.
- 모듈에서 값을 하나만 export할 때 쓴다.
```javascript
export default x => x * x;
import square from './lib.mjs'; //square라고 안해도 되고 임의의 이름으로. {}만 안 붙이면 됨.
console.log(square(3)); //9
```
- 모듈을 가져다 쓰려면 import로 가져와야 한다.

```javascript
import {pi, square, Person} from './lib.mjs';
console.log(pi); //3.14....
console.log(square(2)); //4
```

```javascript
import * as lib from './lib.mjs';
console.log(lib.pi); //3.14....
console.log(lib.sq(2)); //4
```

```javascript
import {pi as PI, square as sq, Person as p} from './lib.mjs';
console.log(PI); //3.14....
console.log(sq(2)); //4
```



- ES6부터는 모듈이 지원된다.
- type="module"로 주고, 확장자도 ES6Module(ESM)임을 알려주기 위해 mjs로 붙이는 게 좋다.
```html
<script type="module" src="app.mjs"></script>
```

- ESM은 독자 모듈 스코프를 지닌다.
- js 파일마다 식별자가 관리된다는 의미다. 이제 var로 선언해도 window의 프로퍼티가 되지 않는다.
- 물론 그래도 const, let을 사용하도록 하자.

## <span style="color:#802548">_5. 바벨과 웹팩_</span>
- 바벨은 트랜스파일러로, ES6문법이 지원되지 않는 경우, ES5문법으로 대체해준다.
- 그렇게 바벨로 바꿔도 ESM은 사용이 안된다.
- 그럼 모듈로더를 webpack으로 하여 사용하면 된다.

## <span style="color:#802548">_6. 이벤트_</span>
- 이벤트 핸들러는 이벤트가 발생했을 때 호출되는 함수다.
- 이벤트가 발생했을 떄 브라우저에게 이벤트 핸들러의 호출을 위임하는 게 이벤트 핸들러 등록이다
- 즉 함수를 언제 호출할 지 알 수 없으므로 개발자가 명시적으로 함수를 호출하는 게 아니라, 브라우저에게 함수 호출을 위임하는 것이다.
- 이렇게 이벤트 중심으로 제어하는 프로그래밍 방식을 이벤트 드리븐 프로그래밍이라고 한다.
- 마우스 이벤트 이름

```
1. click - 클릭
2. dbclick - 더블클릭
3. mousedown - 마우스 눌렀을 때
4. mouseup - 누르던 마우스 버튼을 놓았을 때
5. mousemove - 마우스 커서를 움직일 때
6. mouseenter - 마우스 커서를 HTML 요소 안으로 이동했을 때 (버블링 X)
7. mouseover - 마우스 커서를 HTML 요소 안으로 이동했을 때 (버블링 O)
8. mouseleave - 마우스 커서를 HTML 요소 밖으로 이동했을 떄 (버블링 X)
9. mouseout - 마우스 커서를 HTML 요소 밖으로 이동했을 때 (버블링O)
```
- 마우스 속성
```
altKey
ctrlKey
shiftKey
screenX/screenY
clientX/clientY
pageX/pageY
offsetX/offsetY
```

- 키보드 이벤트 이름
```
keydown - 한글은 keydown
keypress - deprecated
keyup - 누르던 키를 놓았을 때 발생
```
- 키보드 속성

```
ctrlKey
shiftKey
altKey
key
keyCode
```
- 포커스
```
focus -  HTML요소가 포커스를 잃었을 때(버블링X)
blur -  HTML요소가 포커스를 잃었을 때(버블링X)
focusin - HTML요소가 포커스를 잃었을 때(버블링O)
focusout - HTML요소가 포커스를 잃었을 때(버블링O)
```

- 폼
```
submit - enter key 혹은 submit button 눌렀을 때
reset - 거의 사용되지 않음
```

- 값 변경
```
input - input element, select, textarea에 값이 입력될 때
change - 입력이 종료되면 change 발생. input과 더불어 js로도 값이 바뀌었을 때
```

- DOM 뮤테이션 이벤트
```
DOMContentLoaded - DOM생성 완료 시
```

- 리소스 이벤트
```
load - DOMContentLoaded 이후 리소스의 로딩이 완료됐을 때
unload - 새로운 웹페이지를 요청했을 때
abort - 리소스 로딩이 중단됐을 때
error - 리소스 로딩이 실패했을 때
```

- 이벤트 핸들러를 등록하는 데 3가지 방법이 있다.
- 첫번째는 이벤트 핸들러 어트리뷰트다. 아래와 같은 특징이 있다.
```
HTML에 직접 박아넣는다.
반드시 함수를 반환해야 한다. 값을 반환하는 팩토리 함수는 금지다.
바닐라 자바스크립트에서는 잘 쓰이지 않고, React, Vue 등에서 쓰인다.
두 개의 함수를 넣을 수도 있다.
바닐라 JS에서 event를 받을 때는 event라는 이름이 고정이다.
this를 쓰면 일반함수로 처리돼 전역객체에 binding된다. 단, attribute에 쓰는 this는 event를 바인딩한 DOM 요소로 적용된다.
```
```javascript
<button onclick="sayHi('Lee')">click me!</button>
function sayHi(name) {
    console.log(`Hi! ${name}`);
    console.log(`Hi! ${this}`); //window
}

<button onclick="console.log('Hi');console.log('Lee');">click me!</button>
<button onclick="handleClick(event)">click me!</button>
<button onclick="handleClick(this)">click me!</button> //this는 button 자신
<button onclick={sayHi('Lee')}>click me!</button>
<button @onclick="sayHi('Lee')">click me!</button>
```

- 두번째는 이벤트 핸들러 프로퍼티다.
```
하나의 이벤트 핸들러만 바인딩 가능하다.
this를 쓰면 이벤트를 바인딩한 DOM요소를 가리킨다.

```
```javascript
<button>click me!</button>
const $button = document.querySelector('button');

$button.onclick = function() { //$button은 eventTarget. onclick은 on + 이벤트type
    console.log('button click'); // function은 event handler
}

$button.onclick = null; //event 핸들러 제거
```

- 세번쨰 addEventListener method
```
하나의 이벤트 핸들러 이상 바인딩이 가능하다.
this를 쓰면 이벤트를 바인딩한 DOM요소를 가리킨다.
remove를 하려면 함수에 식별자가 있어야 한다.
```
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
}) //제거 불가.

$button.removeEventListener('click',onClick);
```

- 이벤트핸들러를 function으로 선언한 뒤 제거하는 것은 안된다.
- 하지만 1회만 호출하는 것은 가능하다.
```javascript
$button.addEventListener('click',function foo() {
    console.log('button click');

    $button.removeEventListener('click',foo);
})
```

- 이벤트 phase는 캡처링, 타깃, 버블링이 있다.
- 캡처링은 window -> document -> html -> body -> ul -> li로 내려오는 과정이다.
- 타깃은 event target에 이벤트가 실제 일어나는 단계다.
- 버블링은 다시 li -> ul -> body -> html -> document -> window로 간다.

- 버블링의 특성을 이용하면 이벤트 핸들러를 많이 등록하지 않고 똑같이 event를 걸 수 있다.
- 아래는 버블링의 특성을 이용하지 않고 이벤트 핸들러를 걸었다.
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

- 이제 이벤트 위임을 사용해보자.
- 상위 DOM에 걸면 하위 DOM에는 걸 필요가 없다.

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
                if(!target.matches('#fruits > li')){
                    return;
                }

                [...$fruits.children].forEach($fruit => {
                    $fruit.classList.toggle('active', $fruit === target);
                    $msg.textContent = target.id;
                })
            }

            //$fruits.onclick = activate;
            $fruits.addEventListener('click', activate);
        </script>
    </body>
</html>
```


- preventDefault()는 DOM 요소의 기본 동작을 중단한다.
```html
<a href="https://www.google.com">go</a>
<input type="checkbox">
<script>
    document.querySelector('a').onclick = e => e.preventDefault(); // a태그를 눌러도 이동X
    document.querySelector('input[type=checkbox]').onclick = e => e.preventDefault //체크박스 눌러도 체크X
</script>
```

- stopPropagation()는 이벤트 전파를 중지시킨다.

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
                if(!target.matches('.container > button')) { // container 하위 요소 중 button만 대상
                    return;
                }
                target.style.color = 'red';
            }

            document.querySelector('.btn2').addEventListener('click',(e) => {
                e.stopPropagation(); // 그 버튼 중 btn2는 버블링 전파 불가.
                e.target.style.color = 'blue'; //따라서 red로 컬러가 바뀌지 않음. blue로 바뀜
            })
        </script>
    </body>
</html>
```

- 다만 class에서 this를 binding하는 경우에 DOM과 같이 쓴다면 주의해야 한다.

```html
<!DOCTYPE html>
<html>
<body>
    <button class="btn">0</button>
    <script>
        class App {
            constructor() {
                this.$button = document.querySelector('.btn');
                this.count = 0;

                this.$button.onclick = this.increase;
                //this.$button.onclick = this.increase.bind(this);
            }

            increase() {
                this.$button.textContent = ++this.count; //여기서 this는 생성할 instance가 아님.
                //this는 this.$button임. 그걸 의도하진 않기 때문에 위를 bind로 바꿔줘야.
            }
        }

        new App();
    </script>
</body>
</html>
```

- 커스텀 이벤트를 만들 수도 있다.
- 기본값은 버블링X, cancelableX(preventDefault 호출해도 적용X)다

```javascript
const customEvent = new MouseEvent('click');
const customEvent = new MouseEvent('click',{
    bubbles: true,
    cancelable: true
    clientX: 50,
    clientY: 100
})
const keyboardCustomEvent = new KeyBoardEvent('keyup',{key: 'Enter'});
```

- 기존 event type이라면 CustomEvent로 생성하지 않고 기존의 생성자를 가져온다.
```html
<!DOCTYPE html>
<html>
<body>
    <button class="btn">Click me</button>
    <script>
        const $button = document.querySelector('.btn');

        $button.addEventListener('click',e => {
            console.log(e);
            alert(`{e} Clicked`);
        })

        const customEvent = new MouseEvent('click');
        $button.dispatchEvent(customEvent); //그냥 addEventListerner 등록만으로는 안 됨
    </script>
</body>
</html>
```

- 새로운 event type이라면 CustomEvent로 생성한다.
- 기존의 Event에는 detail이라는 instance properties가 없지만, CustomEvent에는 존재한다.

```html
<!DOCTYPE html>
<html>
<body>
    <button class="btn">Click me</button>
    <script>
        const $button = document.querySelector('.btn');

        $button.addEventListener('click',e => {
            console.log(e);
            alert(`{e} Clicked`);
        })

        const customEvent = new CustomEvent('foo', {
            detail: {
                message: 'Hello'
            }
        });
        $button.dispatchEvent(customEvent); //그냥 addEventListerner 등록만으로는 안 됨
    </script>
</body>
</html>
```

- 아래는 mdn의 CustomEvent example이다.
- 기존에 있는 event에 또 엮어서 customEvent를 만드는 것 같다.

```javascript
const form = document.querySelector("form");
const textarea = document.querySelector("textarea");

form.addEventListener("awesome", (e) => console.log(e.detail.text()));
textarea.addEventListener("input", function () {
  // Create and dispatch/trigger an event on the fly
  // Note: Optionally, we've also leveraged the "function expression" (instead of the "arrow function expression") so "this" will represent the element
  this.dispatchEvent(
    new CustomEvent("awesome", {
      bubbles: true,
      detail: { text: () => textarea.value },
    }),
  );
});
```

## <span style="color:#802548">_7. 비동기_</span>
- setTimeout은 비동기처리로 동작한다.
- setInterval은 비동기처리로 동작한다.

```javascript
setTimeout(() => console.log('Hi!')/*,0*/); 
setTimeout(() => console.log('Hi!'), 1000); 
setTimeout((name) => console.log(`${name}`), 1000, 'Lee'); 

const timer = setTimeout(() => console.log('Hi!',1000));
clearTimeout(timer);

let count = 1;
const timeoutId = setInterval(() => {
    console.log(count)
    if(count === 5){
        clearInterval(timeoutId);
    }
    count = count + 1;
    },1000); //
```

- scroll, resize, input, mousemove는 짧은 시간에 연속 발생한다.
- 따라서 이벤트 핸들러가 과도하게 노출되지 않게 디바운스와 스로틀 처리를 해주면 좋다.

```html
<!DOCTYPE html>
<html>
<body>
    <button>click me</button>
    <pre>일반 클릭 이벤트 카운터
        <span class="normal-msg">0</span>
    </pre>
     <pre>디바운스 클릭 이벤트 카운터
        <span class="debounce-msg">0</span>
    </pre>
     <pre>스로틀 클릭 이벤트 카운터
        <span class="throttle-msg">0</span>
    </pre>
    <script>
        const $button = document.querySelector('button');
        const $normalMsg = document.querySelector('normal-msg');
        const $debounceMsg = document.querySelector('debounce-msg');
        const $throttleMsg = document.querySelector('throttle-msg');
    
    const debounce = (callback, delay) => {
        let timerId;
        return (...args) => {
            if(timerId) {
                clearTimeout(timerId);
            }
            timerId = setTimeout(callback, delay, ...args);
        }
    }

    const throttle = (callback, delay) => {
        let timerId;
        return (...args) => {
            if(timerId) {
                return;
            }
            timerId = setTimeout(() => {
                callback(...args);
                timerId = null;
            }, delay)
        }
    }

    $button.addEventListener('click',() => {
        $normalMsg.textContent = +$normalMsg.textCont + 1;
    })

    $button.addEventListener('click', debounce(() => {
        $debounceMsg.textContent = +$debounceMsg.textCont + 1;
    }, 500))

    $button.addEventListener('click',throttle(() => {
        $throttleMsg.textContent = +$throttleMsg.textContent + 1;
    },500))
    </script>
</body>
</html>
```

- input에 0.3초 내 입력이 없으면 debounce 내의 callback이 발동된다.
- 0.3초 내에 입력이 생기면 새로운 타이머를 재설정한다.
- 따라서 이벤트핸들러 자체는 1번만 호출되는 셈이다.
- 다만 이거보다는 lodash의 debounce 함수를 사용하는 것이 좋다. 아래는 단순하게 만들어진 예시다.
```html
<!DOCTYPE html>
<html>
<body>
    <input type="text">
    <div class="msg"></div>
    <script>
        const $input = document.querySelector('input');
        const $msg = document.querySelector('.msg');
        
        const debounce = (callback, delay) => {
            let timerId;
            return (...args) => {
                if(timerId) {
                    clearTimeout(timerId);
                }
                timerId = setTimeout(callback,delay, ...args);
            }
        }

        $input.oninput = debounce((e) => {
            $msg.textContent = e.target.value;
        },300)

    </script>
</body>
</html>
```

- scroll 이벤트는 연속발생하는데, 일정 시간 단위로 이벤트 핸들러가 호출되도록 한다.
- delay 시간 간격으로 콜백 함수가 호출된다.
```html
<!DOCTYPE html>
<html>
<head>
    <style>
        .container {
            width: 300px,
            height: 300px,
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
                if(timerId) {
                    return;
                }
                timerId = setTimeout(() => {
                    callback(...args);
                    timerId = null;
                }, delay);
            }
        }

        let normalCount = 0;
        $container.addEventListener('scroll', () => {
            $normalCount.textContent = ++normalCount;
        });

        let throttleCount = 0;
        $container.addEventListener('scroll',throttle(() => {
            $throttleCount.textContent = ++throttleCount; 
        }, 100));
    </script>
</body>
</html>
```

- JS 엔진은 단 하나의 call stack을 가진다. 최상위 요소인 실행 중인 실행 컨텍스트만 실행된다.
- 실행 대기 중인 태스크를 제외하면 모두 대기중이다. 현재 실행중인 함수가 종료되어야만 다른 함수가 실행된다.
- 이를 회피하려면 비동기로 처리하는 setTimeout, setInterval을 실행해야 한다.
```
JS엔진은 싱글 스레드로 동작하지만 브라우저는 멀티 스레드로 동작한다.
브라우저는 JS엔진 외에도 렌더링 엔진과 Web API를 제공한다.
timeout 설정과 timeout태스트 큐에 등록하는 처리는 JS 엔진이 아니라 브라우저가 실행한다.
```
- 브라우저 환경이 task queue안에 비동기 함수의 콜백 함수, 이벤트 핸들러를 일시로 보관한다.
- 콜 스택이 비어있다면 이벤트 루프가 task queue 안에 함수들을 call stack으로 이동시킨다.

- 아래와 같이 진행한다면, 0초를 준다고 해도, foo함수가 task queue에 등록된다. 
- 즉 bar가 먼저 call stack에 push되고, bar가 pop되면 이벤트 루프가 task queue에서 foo를 가져와 call stack에 push한다.
- promise의 후속처리(then)은 task queue보다 우선순위가 높은 microtask queue에 저장된다. 
```javascript
function foo() {
    console.log('foo');
}

function bar() {
    console.log('bar');
}

setTimeout(foo,0);
bar();
```

- 비동기 통신의 최초는 XMLHttpRequest다.

```javascript
const xhr = new XMLHttpRequest();
xhr.open('GET','/users'); //초기화
xhr.setRequestHeader('content-type','application/json')//request 헤더 설정
xhr.send(); //request send

const xhr = new XMLHttpRequest();
xhr.open('POST','/users');
xhr.setRequestHeader('content-type','application/json');
xhr.send(JSON.stringify({id: 1, content: 'HTML', completed: false})) 
xhr.onreadystatechange = () => { 
    if(xhr.readState !== XMLHttpRequest.DONE) { //요청 상태 확인
        return;
    }

    if(xhr.status === 200) { //성공
        console.log(JSON.parse(xhr.response));
    } else { // 실패
        console.error('Error', xhr.status, xhr.statusText);
    }
}
```

- 비동기 처리를 위한 콜백 패턴의 단점은, 비동기 함수가 종료될 때까지 기다리지 않고 반환해버린다는 점이다.
```javascript
let g;
setTimeout(() => {
    g = 100;
}, 0);
console.log(g); //undefined
```
```javascript
let g;
let g;
setTimeout(() => {
    g = 100;
}, 0);
setTimeout(() => {
    console.log(g); //100
}, 0);
```

- 비동기로 보낸 함수의 return을 보면, undefined다.
```javascript
const get = url => {
    const xhr = new XMLHttpRequest();
    xhr.open('GET', url);
    xhr.send();

    xhr.onload = () => {
        if(xhr.status === 2000) {
            return JSON.parse(xhr.response);
        }
        console.error(`${xhr.status} ${xhr.statusText}`);
    };
}

const response = get('https://jsonplaceholder.typicode.com/posts/1');
console.log(response)//undefined
```

- 상위 스코프의 변수에 할당한다음에 바꿔도 똑같이 undefined다.
```javascript
let todos;

const get = url => {
    const xhr = new XMLHttpRequest();
    xhr.open('GET',url);
    xhr.send();

    xhr.onload = () => {
        if(xhr.status === 200) {
            todos = JSON.parse(xhr.response); //onload event는 밑의 console.log보다 더 늦는다. 비동기 event라서 task queue에 들어갔기 때문이다. 
        } else {
            console.error(`${xhr.status} ${xhr.statusText}`);
        }
    }
}

get('https://jsonplaceholder.typicode.com/posts/1');
console.log(todos); //undefined. onload가 아직 실행되지 않았기 때문이다. console.log가 끝나야 xhr이 task queue에서 call stack으로 push되어 todos가 값을 갖게 된다. 그럼 그 때는 todos가 할당되게 된다.
```

```javascript
const get = (url, successCallback, failureCallback) => {
    const xhr = new XMLHttpRequest();
    xhr.open('GET', url);
    xhr.send();

    xhr.onload = () => {
        if(xhr.status === 200) {
            successCallback(JSON.parse(xhr.response));
        } else {
            failureCallback(xhr.status);
        }
    }
}

get('https://jsonplaceholder.typicode.com/posts/1', console.log, console.error); // data 정상 넘어옴.
```

- 위에서 보았듯, callback은 함수를 넘겨 받아서 계속 이어나가야 한다.
- 변수를 할당하는 것으로는 문제 해결이 안된다.
- 그러다보니 아래와 같이 callback 안의 callback으로 계속 이어져버린다.
```javascript
const get = (url, callback) => {
    const xhr = new XMLHttpRequest();
    xhr.open('GET', url);
    xhr.send();

    xhr.onload = () => {
        if(xhr.status === 200) {
            callback(JSON.parse(xhr.response));
        } else {
            console.error(`${xhr.status} ${xhr.statusText}`);
        }
    }
}

const url = 'https://jsonplaceholder.typicode.com';
get(`${url}/posts/1`, ({userId}) => {
    console.log(userId);

    get(`${url}/users/${userId}`, userInfo => {
        console.log(userInfo);
    })
})
```

- callback의 다른 문제는 error catch가 안 된다는 점이다.
- setTimeout이 호출 -> setTimeout은 비동기 함수라서 콜백함수 호출되는 것 기다리지 않고 종료
- setTimeout의 callback 함수는 task queue로 push
- 문제는 setTimeout은 이미 종료된 함수라서 callback함수를 호출한 것은 브라우저라는 점

- 따라서 콜백함수에서 난 error가 setTimeout으로 전파되지 않고, error가 catch되지 않는다.
- catch되지 않은 error가 있어 프로그램이 강제 종료된다.
```javascript
try { 
    setTimeout(() => {
        throw new Error('Error!');
    }, 1000)
} catch(err) {
    console.error('캐치한 에러',err);
}
```

- 이런 문제점을 해결하려고 Promise가 도입됐다.

```javascript
const promise = new Promise((resolve, reject) => {
    if(/* 비동기 처리 성공 */) {
        resolve('result');
    } else { /* 비동기 처리 실패 */
        reject('failure reason');
    }
})

const promiseGet = url => {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open('GET', url);
        xhr.send();

        xhr.onload = () => {
            if(xhr.status === 200) {
                resolve(JSON.parse(xhr.response));
            } else {
                reject(new Error(xhr.status));
            }
        }
    })
}

promiseGet('https://jsonplaceholder.typicode.com/posts/1');
```

```javascript
new Promise(resolve => resolve('fulfilled'))
    .then(v => console.log(v) , e => console.error(e)); // 비동기 처리 성공시 console.log('fulfilled')

new Promise((_, reject) => reject(new Error('rejected')))
    .then(undefined, e => console.error(e))   // 비동기 처리 실패시 console.error(new Error('rejected'))
```

```javascript
const promiseGet = url => {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest():
        xhr.open('GET', url);
        xhr.send();

        xhr.onload = () => {
            if(xhr.status === 200) {
                resolve(JSON.parse(xhr.response));
            } else {
                reject(new Error(xhr.status));
            }
        }
    })
}

//처음 cb fn에서 err 뜬 것을 두번째  fn에서 인지 못함.
promiseGet('https://jsonplaceholder.typicode.com/posts/1').then(
    res => console.log(res),
    err => console.error(err)
)

//catch 대신에 then으로 쓰면 아래와 같음. 성공 callback은 undefined로 설정
promiseGet('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => console.log(res))
    .then(undefined, err => console.error(err));

// then 대신에 catch문으로 넣으면 undefined를 굳이 주지 않아도 된다. 더 편하다.
promiseGet('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => console.log(res))
    .catch(err => console.error(err))
    .finally(() => console.log('Bye!'));
```

- 아까 썼던 callback 지옥을 promise로 바꿔보자
- 여전히 계쏙 이어 써야 하는 점은 변하지 않는다.

```javascript
const url = 'https://jsonplaceholder.typicode.com';

promiseGet(`${url}/posts/1`)
    .then(({userId}) => promiseGet(`${url}/users/${userId}`))
    .then(useInfo => console.log(userInfo))
    .catch(error => console.error(error));
```

- 아래는 다른 방식으로 동일한 Promise를 생성한 것이다.
```javascript
const resolvedPromise = new Promise(resolve => resolve([1,2,3]));
const resolvedPromise = Promise.resolve([1,2,3]);
resolvedPromise.then(console.log); //[1,2,3]

const rejectedPromise = new Promise((_, reject) => reject(new Error('Error!')));
const rejectedPromise = Promise.reject(new Error('Error!'));
rejectedPromise.catch(console.log); //Error: error!
```

- 아래는 순차적으로 promise를 chaining한 것이다.
- 그래서 6초가 걸린다.
```javascript
const requestData1 = () => new Promise(resolve => setTimeout(() => resolve(1), 3000));
const requestData2 = () => new Promise(resolve => setTimeout(() => resolve(2), 2000));
const requestData3 = () => new Promise(resolve => setTimeout(() => resolve(3), 1000));

const rest = [];
requestData1()
    .then(data => {
        res.push(data);
        return requestData2();
    })
    .then(data => {
        res.push(data);
        return requestData3();
    })
    .then(data => {
        res.push(data);
        console.log(res); //[1,2,3] 6초 소요
    })
    .catch(console.error);
```

- 하지만 아래와 같이 all로 걸면 비동기로 진행되어 3초면 된다.
```javascript
Promise.all([requestData1(), requestData2(), requestData3()])
    .then(console.log) //[1,2,3] 3초 소요
    .catch(console.error);
```

- all로 걸었을 때 Promise가 하나라도 rejected가 되는 경우, 나머지 프로미스가 fulfilled가 되는 것을 기다리지 않고 즉시 종료한다.
```javascript
Promise.all([
    new Promise((_,reject) => setTimeout(() => reject(new Error('Error 1'), 3000))),
    new Promise((_,reject) => setTimeout(() => reject(new Error('Error 2'), 2000))),
    new Promise((_,reject) => setTimeout(() => reject(new Error('Error 3'), 1000)))
])
    .then(console.log)
    .catch(console.log); //Error: Error 3. 맨마지막 promise가  제일 빨리 실행됐고, rejected되었으므로 다른 promise들의 결과상태를 보지 않고 catch문으로 빠진다.

Promise.all([
    new Promise((_,reject) => setTimeout(() => reject(new Error('Error 1'), 1000))),
    new Promise((_,reject) => setTimeout(() => reject(new Error('Error 2'), 2000))),
    new Promise((_,reject) => setTimeout(() => reject(new Error('Error 3'), 3000)))
])
    .then(console.log)
    .catch(console.log); //Error: Error 3. 맨마지막 promise가  제일 빨리 실행됐고, rejected되었으므로 다른 promise들의 결과상태를 보지 않고 catch문으로 빠진다.
```

- 깃허브 아이디 정보를 가져온다.
```javascript
const promiseGet = url => {
    return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open('GET', url);
        xhr.send();

        xhr.onload = () => {
            if(xhr.status === 200) {
                resolve(JSON.parse(xhr.response));
            } else {
                reject(new Error(xhr.status));
            }
        }
    })
}

const githubIds = ['jeresig', 'ahejlsberg', 'ungmo2'];

Promise.all(githubIds.map(id => promiseGet(`https://api.github.com/users/${id}`)))
    .then(users => users.map(user => user.name))
    .then(console.log)
    .catch(console.error);
```

- fetch는 웹서버와 소통하는 api인데, 내부에서 Promise를 활용한다.
- 다만 HTTP error code는 reject하지 않기 때문에 그러한 에러는 then에서 잡아줘야 한다.


```javascript
const request = {
    get(url){
        return fetch(url);
    },
    post(url, payload) {
        return fetch(url,{
            method: 'POST',
            headers: {'content-type' : 'application/json'},
            body: JSON.stringify(payload)
        })
    },
    patch(url, payload) {
        return fetch(url, {
            method: 'PATCH',
            headers: {'content-type' : 'application/json'},
            body: JSON.stringify(payload)
        })
    },
    delete(url) {
        return fetch(url, {method: 'DELETE'});
    }
}

request.get('https://jsonplaceholder.typicode.com/todos/1')
    .then(response => {
        if(!response.ok) {
            throw new Error(response.statusText); // 서버에러는 catch가 아니라 여기서 잡힘
            //fetch의 promise는 404, 500 등의 에러에 대해 에러를 reject하지 않고 ok를 false로 잡고 resolve하기 때문
        }

        return response.json();
    })
    .then(todos => console.log(todos))  // response.json()을 받은게 todos
    .catch(err => console.error(err)); //오프라인 네트워크 장애, CORS 설정만 잡힘
```

```javascript
async function foo() {
    const res= await Promise.all([
        new Promise(resolve => setTimeout(() => resolve(1), 3000)),
        new Promise(resolve => setTimeout(() => resolve(2), 2000)),
        new Promise(resolve => setTimeout(() => resolve(3), 1000))
    ]);

    console.log(res); //[1,2,3]
}

foo(); //3초 걸림
```

- 계속 이어서 동기적인 처리가 필요하다. 이전의 promise 값을 받아서 진행하고 있다.
```javascript
async function bar(n) {
    const a = await new Promise(resolve => setTimeout(() => resolve(n) ,3000));
    const b = await new Promise(resolve => setTimeout(() => resolve(a+1),2000));
    const c = await new Promise(resolve => setTimeout(() => resolve(b + 1), 1000));

    console.log([a,b,c]);
}

bar(1);
```

## <span style="color:#802548">_8. Set과 Map_</span>
- Set객체는 중복되지 않는 값의 집합이다.

```javascript
const set1 = new Set([1,2,2,3]);
console.log(set1); //{1,2,3}

const set2 = new Set('hello');
console.log(set2) //{"h","e","l","o"}
```

- 따라서 set으로 만들면 filter를 걸 필요가 없다.

```javascript
const uniq = array => array.filter((v, i, self) => self.indexOf(v) === i);
console.log(uniq(2,1,1,3,2,4));// [2,1,3,4]

const uniq = array => [...new Set(array)]; //set이 아니라 array이다.
console.log(uniq(2,1,1,3,2,4));// [2,1,3,4]
```

- Set의 size property는 readonly기 때문에 변경이 불가능하다.
```javascript
const set = new Set([1,2,3])
set.size = 10;
console.log(set.size); //3
```

- set에 element를 넣을 때는 add로 넣는다.
```javascript
const set = new Set();
set.add(1); //1이 추가된 새로운 Set을 반환
set.add(1).add(2);//1이 추가된 새로운 Set에 또 2를 추가한 새로운 Set을 반환
console.log(set); 
```

- set이 요소를 포함하고 있는지 보려면 has로 본다.
```javascript
const set = new Set([1,2,3]);
console.log(set.has(2)); //true
console.log(set.has(4)); //false
```

- set이 요소를 제거하려면 delete다.
- 존재하지 않는 요소를 제거해도 에러가 나지 않고 무시된다.
- delete는 연속 호출이 불가능하다. return 값이 boolean이다.
```javascript
const set = new Set([1,2,3]);
set.delete(2);
console.log(set); //{1,3}
set.delete(2).delete(1) //TypeError:delete is not a function
```

- 요소를 일괄 삭제하려면 clear다.
```javascript
const set = new Set([1,2,3]);
set.clear(); //return은 undefined
```

- 요소를 순회하려면 forEach를 사용한다.
```javascript
const set = new Set([1,2,3]);
set.forEach((v) => console.log(v)); // 1 2 3
```

- set으로 집합 연산을 구현할 수 있다.
- 아래는 교집합이다.
```javascript
Set.prototype.intersection = function(set) {
    const result = new Set();

    for(const value of set) {
        if(this.has(value)){
            result.add(value);
        }
    }

    return result;
}

Set.prototype.intersection = function (set) {
    return new Set([...this].filter(v => set.has(v)));
}

const setA = new Set([1,2,3,4]);
const setB = new Set([2,4]);

console.log(setA.intersection(setB)); //{2,4};
console.log(setB.intersection(setA)); //{2,4};
```

- 아래는 합집합이다.
```javascript
Set.prototype.union = function(set) {
    const result = new Set(this);

    for(const value of set) {
        result.add(value);
    }

    return result;
}

Set.prototype.union = function (set) {
    return new Set([...this, ...set]);
};

const setA = new Set([1,2,3,4]);
const setB = new Set([2,4]);

console.log(setA.union(setB)); // {1,2,3,4}
console.log(setB.union(setA));// {2,4,1,3}
```

- 아래는 차집합이다.
```javascript
Set.prototype.difference = function(set) {
    const result = new Set(this);

    for(const value of set) {
        result.delete(value);
    }

    return result;
}

Set.prototype.difference = function (set) {
    return new Set([...this].filter(v => !set.has(v)));
};

const setA = new Set([1,2,3,4]);
const setB = new Set([2,4]);

console.log(setA.difference(setB)); // {1,3}
console.log(setB.difference(setA));// {}
```











