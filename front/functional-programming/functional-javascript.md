## <span style="color:#802548">_함수를 인자로 받는 함수_</span>
- 함수형을 paradigm을 사용하게 되면, 함수를 다른 함수의 callback으로 받게 된다.
- pluck을 쓰면 배열은 유지되지만, 객제 배열이 아니라 문자열 배열로 바뀌게 된다.


```js
_.pluck(library, 'title');
// ["SICP", "SICP", "JOY of Clojure"];
```



- 추상화 수준을 유지하면서 데이터를 뽑아줄 수 있게 아래처럼 함수를 구성한다.
- SQL의 select 역할을 하는 함수다.

```js
function project(table, keys) {
  return _.map(table, function(obj) {
    return _.pick.apply(null, construct(obj, keys));
  })
}
//construct 함수는 신경안써도됨.

project(library, ['title','isbn']);
// [{isbn: '0241424', title:'SICP'}, {isbn:"024124", title:"SICP"}]
```


- sql에서 filtering을 하는 함수도 추가해준다.
- edition이 2이상인 것부터만 뽑아오게 된다.

```js
function retstrict(table, pred) {
  return _.reduce(table, function(newTable, obj) {
    if (truty(pred(obj)) {
      return newTable;
    } else {
      return _.without(newTable, obj);
    })
  })
}

restrict(library, function(book){
  return book.ed > 1;
})
// 2이상인 것부터만 뽑아옴.
```

- 아래처럼 project 함수를 넣어 사용하면 함수형 programming이 된다.

```js
restrict(
  project(
    as(objectArray, ['title','isbn','edition']),
    function(book) {
      return book.edition > 1;
    }
  )
)
```

- 위의 함수는 아래 SQL과 동일한다.

```sql
SELECT title, isbn, edition from (
  SELECT ed AS edition FROM library
) EDS
WHERE edition > 1;
```

## <span style="color:#802548">_클로저 예시_</span>

- 클로저는 argument 대상으로도, 자유변수 대상으로도 작동한다.
- FACTOR로 넣은 argument인 10이 끝까지 살아있다.

```js
function createdScaleFunctino(FACTOR) {
  return function(v) {
    return _.map(v, function(n) {
      return (n * FACTOR);
    })
  }
}

var scale10 = createdScaleFunction(10);
scale10([1,2,3]);
```

- 실제로는 아래와 같은 과정을 거친다고 생각하면 된다.

```js
function createdScaleFunctino(FACTOR) {
  return function(v) {
    this['FACTOR'] = FACTOR;
    var captures = this;

    return _.map(v, function(n) {
      return (n * this['FACTOR']);
    }, captrues)
  }
}

var scale10 = createWeirdScaleFunction(10);
scale10.call({}, [5,6,7]);
```


- 108과 0 중 무엇이 우선할까? 정답은 108이다.
  - parameter가 더 우선권이 있기 때문이다.

```js
var shadowed = 0;

function argShadow(shadowed) {
  return ["Value is", shadowed].join(' ');
}

argShadow(108); // Value is 108

argShadow(); // Value is 
```

- closure를 사용하게 되면, 제일 가까운 shadowed 값은 2다.
- 따라서 아래 값은 3이 된다. 109가 아니다.

```js
function captureShadow(SHADOWED) {
  return function(SHADOWED) {
    return SHADOWED + 1;
  }
}

var closureShadow = captureShadow(108); 

console.log(closureShadow(2)); // 109가 아닌 3
```

- 클로저는 비공개 변수를 만드는 데도 탁월하다.
- PRIVATE을 만들어서 다른 함수에서 수정할 수 없게 만들었다.

```js
const pingpong = (function() {
  let PRIVATE = 0;
  
  return {
    inc: function(n) {
      return PRIVATE = PRIVATE + n;
    },
    dec: function(n) {
      return PRIVATE = PRIVATE - n;
    }
  }
})();

console.log(pingpong.inc(10));

console.log(pingpong.dec(3))
```


- 다른 함수를 method로 추가해도 발동된 당시에 없다면 오류가 난다.
- 함수가 호출되는 과정에만 존재하는 변수기 때문에, 차후 추가는 무시된다.
- 매우 안정적으로 프로그래밍을 가능하게 한다.

```js
pingpong.div = function(n) {
  return PRIVATE / n;
}
// Uncaught ReferenceError: PRIVATE is not defined 
```



## <span style="color:#802548">_값 대신 함수를 사용하라_</span> 
- 함수를 사용할 때, 보통 함수를 넘기지 않는다.

```js
function repeat(times, VALUE) {
  return _.map(_.range(times), function() {
    return VALUE;
  })
}

repeat(4, "Major");
//['Major','Major','Major','Major']
```

- 하지만 함수를 넘길 수도 있다.

```js
function repeatedly(time, fun) {
  return _.map(_.range(times), fun);
}

repeatedly(3, function() {
  return Math.floor( (Math.random() * 10) + 1);
})
//[1,3,8]
```

- 함수를 넘기는 것의 장점은, 매우 유연하다는 것이다.
- 2번째 argument로 값만 넘겼을 때와 달리 함수 내부에서 처리할 수 있다.

```js
repeatedly(3, function() { return "Odely";})
//['Odely','Odely','Odely']

repeatedly(3, function(n) {
  const id = 'id' + n;
  $('body').append($("<p>Odelay!</p>").atr('id',id));
  return id;
})
//["id0","id1","id2"]
```

- 함수 이름을 정확하게 나타내주자.

```js
function executeRpeatedAction(iterationCount, actionFunction) {

}
```

## <span style="color:#802548">_커링과 비슷하게 부분 적용_</span> 
- 커링은 js에서만 쓰는 건 아니고, 함수형 언어에서는 모두 사용되는 기법이다.
- 커링을 쓰지 않으면 일반적인 OOP와 비슷하게 전개된다.

```js
const todos = [
  { id: 3, content: 'HTML', completed: false },
  { id: 2, content: 'CSS', completed: true },
  { id: 1, content: 'Javascript', completed: false }
];

const getTodosIdArr = todos => todos.map(todo => todo.id);
const getTodosContentArr = todos => todos.map(todo => todo.content);
const getTodosCompletedArr = todos => todos.map(todo => todo.completed);

console.log(getTodosIdArr(todos)); // [ 3, 2, 1 ]
console.log(getTodosContentArr(todos)); // [ 'HTML', 'CSS', 'Javascript' ]
console.log(getTodosCompletedArr(todos)); // [ false, true, false ]
```

- 커링을 사용하게 되면 중간에 함수 매개체가 parameter로 하나 끼어들게 된다.

```js
const todos = [
  { id: 3, content: 'HTML', completed: false },
  { id: 2, content: 'CSS', completed: true },
  { id: 1, content: 'Javascript', completed: false }
];

const get = property => object => object[property];

const getId = get('id');
const getContent = get('content');
const getCompleted = get('completed');

const getTodosIdArr = todos => todos.map(getId);
const getTodosContentArr = todos => todos.map(getContent);
const getTodosCompletedArr = todos => todos.map(getCompleted);

console.log(getTodosIdArr(todos)); // [ 3, 2, 1 ]
console.log(getTodosContentArr(todos)); // [ 'HTML', 'CSS', 'Javascript' ]
console.log(getTodosCompletedArr(todos)); // [ false, true, false ]
```

- 커링의 다른 예시는 아래와 같다.

```js
function multiply(x, y, z) {
  return x * y * z;
}

console.log(multiply(2, 3, 4)); // Output: 24

function multiply(x) {
  return function(y) {
    return function(z) {
      return x * y * z;
    };
  };
}

console.log(multiply(2)(3)(4)); // Output: 24
```


- 또 다른 예제는 log를 찍는 것이다.

```js
function log(date, importance, message) {
  alert(`[${date.getHours()}:${date.getMinutes()}] [${importance}] ${message}`);
}

const log = _.curry(log);

log(new Date(), "DEBUG", "some debug"); // log(a, b, c)
log(new Date())("DEBUG")("some debug"); // log(a)(b)(c)
```


- 1단계 parameter를 받아서 거기까지만 저장하여 한꺼풀 벗겨낸다. 
  - 여기선 날짜다. 현재날짜로 로그를 찍는다.
- 그 뒤 로그 수준을 정한다.
- 마지막으로 메시지를 정한다. 

```js
// logNow는 log의 첫 번째 인수가 고정된 partial
const logNow = log(new Date());
logNow("INFO", "message"); // [HH:mm] INFO message

const debugNow = logNow("DEBUG");
debugNow("message"); // [HH:mm] DEBUG message
```

- validation 기능을 위해 validate 함수를 만들었다.

```js
function validator(message, fun) {
  const f = function() {
    return fun.apply(fun, arguments);
  };

  f['message'] = messaage;
  return f;
}
```


- 아직 커링이 적용되지 않은 상태의 함수다.

```js
const zero = validator("cannot be zero", function(n) { return 0 === n});
const number = validator("arg must be a number", _.isNumber);

function sqr(n) {
  if (!number(n)) 
    throw new Error(number.message);
  if (zero(n))
    throw new Error(zero.message); 

  return n * n;
}
```

- 커링을 이용해서 validate를 사용하면 좀 더 보기 좋은 형태가 된다

```js
function condition1(validatorFn /* validator*/) {
  const validators = _.toArrray(arguments);

  return function(fun, arg) {
    const errors = mapcat(function(isValid) {
      return isValid(arg) ? [] : [isValid.message];
    }, validators)

    if (!_.isEmpty(errors) {
      throw new Error(errors.join(", "));
    })

    return fun(arg);
  }
}

const sqrPre = condition1(
  validator("arg must not be zero", complement(zero)),
  validator("arg must be a number", _.isNumber)
);
```

## <span style="color:#802548">_순수함수와 불변객체_</span> 

- 함수를 사용할 때는, 순수함수를 쓰고, 객체를 사용할 때는 불변 객체를 쓰자.

- 순수함수는 다음 조건을 만족하는 함수다.
  - 인자만을 이용해 계산 결과를 만든다
  - 외부에서 데이터를 받지 않는다.
  - 외부의 데이터를 변화시키지 않는다.
- 다시말해 순수함수는 함수 내에서만 모든 과정이 처리되게끔 하는 함수다.
- 당연히 api call, db read, write는 pure function이 될 수 없다.
- 하지만 그 외 함수는 최대한 pure하게 만드는 것이 버그를 줄일 수 있다.


- 불변의 객체는 다음 조건을 만족하는 경우다.
 - 객체는 생성될 때 자신의 값을 채운 뒤로 변화하지 않는다.
 - 모든 동작의 결과가 새로운 객체로 반환된다.
 
