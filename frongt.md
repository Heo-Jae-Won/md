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