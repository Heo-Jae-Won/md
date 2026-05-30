## <span style="color:#802548">_1. react app 설치_</span>
- 아래와 같이 react project를 설치한다
```
pnpm create vite clean-code-react-app --template react
pnpm i
pnpm dev
```


## <span style="color:#802548">_2. 상태 초기값 설정_</span>
- 올바른 초기값 설정은 중요하다.
  - 초기값은 순간적으로 보여질 수 있기 때문이다.
  - 거기다 처음 undefined면 method가 작동하지 않아 rendering 에러가 난다.
  - 상태를 지울 때도 초기값을 잘 기억해놔야 원상태로 돌아간다.
- list가 undefined라서 제대로 parsing이 안된다.
- 따라서 Array.isArray(list) 같은 쓸데없는 방어적 코드를 넣어야 한다.
- 그보다는 기본값을 넣어주자.
```js
const [count, setCount] = useState('0');
const [list,setList] = useState(); //[] 기본값 넣자.

const resetState = () => {
    setCount(INIT_COUNT_STATE); //INIT_COUNT_STATE가 숫자라면 에러가 난다.
}

return (
    <>
{list.map((item) => (
    <Item item={item} /> 
))}
    </>
)
```

## <span style="color:#802548">_3. 바뀌지 않는 변수는 component 밖으로_</span>
- 아래와 같은 경우 INFO 변수는 굳이 fc가 갖고 있을 필요가 없다.
- 참조를 유지해야 할 필요가 없다. componenet 사이클에 따라 변하지 않기 때문이다.
```js
function NoUpdateValue() : Element {
// 상수를 다루거나 아니면 일반적으로 방치
// 
    const INFO = {
        name: 'MY Component',
        value: 'Clean Code App'
    }    

    return (
        <header>{INFO}</header>
    )


}
```

- 아예 react componenet 외부로 보내자.
```js
const INFO = {
    name: 'MY Component',
    value: 'Clean Code App'
}   

function NoUpdateValue() : Element {
// 상수를 다루거나 아니면 일반적으로 방치

}
```


## <span style="color:#802548">_4. 플래그 값은 state로 쓰지 않는다._</span>
- 플래그 값은 상태로 만들지 않고 component 내부의 변수로 만들자.
- 상태로 만들면 매번 갱신되어야 하는데, flag 값은 처음 받아온 값에서 변하지 않으므로 그럴 필요가 없다.
- 변하는 것을 state로 관리해야 한다.
```js
const isLogin = hasToken &&
                hasCookie &&
                isValiCookie &&
                isNewUser === false &&
```

## <span style="color:#802548">_4. 플래그 값은 state로 쓰지 않는다._</span>
- React에서 state로 선언할 변수는 rendering마다 값이 변하는 것들이다.
- complUserList는 굳이 state로 선언할 필요가 없다.
- 이미 받아온 userList를 가공하는 것이기 때문이다.
```js
const [userList, setUserList] = useState(MOCK_DATA);
const [complUserList, setComplUserList] = useState(MOCK_DATA);

useEffect(() => {
    const newList = complUserList.filter((user) => user.completed === true);
    setUserList(newList);
})
```

- 가공한 것은 state로 선언하지 않아도 충분하다. 어차피 다시 rendering되기 때문이다.
- 따라서 useEffect에서 새로만든 list를 담을 필요도 없다.
```js
const [userList, setUserList] = useState(MOCK_DATA);
const complUserList = userList.filter((user) => user.completed === true);

return (
    <ul>
        {
            complUserList.map((user) => (
                <li>
                    {user.title} / {user.compelted}
                </li>
            ))
        }
    </ul>
)
```


## <span style="color:#802548">_4. useRef를 사용해야 할 때_</span>
- 컴포넌트의 수명과 동일하게 지속되는 정보를 일관적으로 제공해야 하는 경우는 useRef를 사용한다.
- rerendering을 원하지 않을 때 특히 사용하기 좋다.
- 아래처럼 useEffect에 isMount 변수를 변경시키면 설정해놓으면 rerender가 일어난다.
```js
const [isMount, setIsMount] = useState(false);

useEffect(() => {
    if (!isMount) {
        setIsMount(true);
    }
},[isMount])
```

- 따라서 아래와 같이 바꾼다.
- rerendering이 불필요하게 일어나지 않는다.
- 
```js
const isMount = useRef(false);

useEffect( () => {
    isMjount.current = true;

    return () => (isMount.current = false);
}, [])
```


## <span style="color:#802548">_5.State 단순화하기_</span>
- finish가 true라면  loading, error는 false여야 한다.
- error가 true라면 loading, finish는 false여야 한다.
- loading이 true라면 finish, error는 false여야 한다.
- 서로 깊게 연관되어 있다. 실수하기 딱 좋다.
```js
function FlatState() {
    const [isLoading, setIsLoading] = useState(false);
    const [isFinish, setIsFinish] = useState(false);
    const [isError, setIsError] = useState(false);

    const fetchData = () => {
        setIsLoading(true);

        fetch(url)
            .then(() => {
                setIsLoading(false);
                setIsFinish(true);
            })
            .catch(() => {
                setIsLoading(false);
                setIsFinish(true);
            })
    }
}
```

- 아래와 같이 하나의 state 변수에 여러 상태를 담게 구성한다.
```js
function FlatState() {
    const [promissState, setPromiseState] = useState('pending');

    const fetchData = () => {
        setPromiseState('loading');

        fetch(url)
            .then(() => {
                setPromiseState('finish');
            })
            .catch(() => {
                setPromiseState('error');
            })
    }

    if(promiseState === 'loaindg') return <
}
```



- string 처리한 것들은 모두 객체로 빼낸다.
```js
const PROMISE_STATE = {
    INIT: 'init';
    LOADING: 'loading';
    FINISH: 'finish';
    ERROR: 'error';
}
function FlatState() {
    const [promiseState, setPromiseState] = useState(PROMISE_STATE.INIT);

    const fetchData = () => {
        setPromiseState(PROMISE_STATE.LOADING);

        fetch(url)
            .then(() => {
                setPromiseState(PROMISE_STATE.FINISH);
            })
            .catch(() => {
                setPromiseState(PROMISE_STATE.ERROR);
            })
    }

    if(promiseState === PROMISE_STATE.LOADING) return <LoadingComponent/>
    if(promiseState === PROMISE_STATE.FINISH) return <FinishComponent/>
    if(promiseState === PROMISE_STATE.ERROR) return <ErrorComponent/>
}
```


- 아래와 같이 하나의 state 변수에 여러 상태를 담게 구성하는 두번쨰는 객체다.
```js
function FlatState() {
    const [fetchState, setFetchState] = useState({
                                            isLoading: false,
                                            isFinish: false,
                                            isError: false,
                                        });
    const fetchData = () => {
        setFetchState({
            isLoading: true,
            isFinish: false,
            isError: false,
        });

        fetch(url)
            .then(() => {
                setFetchState({
                    isLoading: false,
                    isFinish: true,
                    isError: false,
                });
            })
            .catch(() => {
                setFetchState({
                    isLoading: false,
                    isFinish: false,
                    isError: true,
                });
            })
    }
}
```

- 아래와 같이 spread operator를 사용해서 코드 양을 줄여준다.
```js
function FlatState() {
    const [fetchState, setFetchState] = useState({
                                            isLoading: false,
                                            isFinish: false,
                                            isError: false,
                                        });
    const fetchData = () => {
        setFetchState((prevState) =>{
            ...prevState,
            isLoading: true,
        });

        fetch(url)
            .then(() => {
                setFetchState({
                    ...prevState,
                    isFinish: true,
                });
            })
            .catch(() => {
                setFetchState({
                     ...prevState,
                    isError: true,
                });
            })
    }
}
```

## <span style="color:#802548">_6.useReducer 사용하기_</span>
```js
function FlatState() {
    const [isLoading, setIsLoading] = useState(false);
    const [isFinish, setIsFinish] = useState(false);
    const [isError, setIsError] = useState(false);

    const fetchData = () => {
        setIsLoading(true);

        fetch(url)
            .then(() => {
                setIsLoading(false);
                setIsFinish(true);
            })
            .catch(() => {
                setIsLoading(false);
                setIsFinish(true);
            })
    }

    if (isLoading) return <LoadingComponent/>
    if (isFinish) return <LoadingComponent/>
    if (isError) return <LoadingComponent/>
}
```

- reducer를 사용하면 추상화 수준을 높일 수 있다.
- reducer를 몰라도 type만 보고 대충 이런거 하겠구나 감잡기 좋다.
```js
const INIT_STATE = {
    isLoading: false,
    isSuccess: false,
    isFail : false
}

const ACTION_TYPE = {
    FETCH_LOADING: 'FETCH_LOADING',
    FETCH_SUCCESS: 'FETCH_SUCCESS',
    FETCH_FAIL : 'FETCH_FAIL'
}

const reducer = (state, action) => {
    swithc (action.type) { //if도 사용가능
        case  'FETCH_LOADING':
            return {...state, isLoading: true} //바뀐 상태 명시
        case 'FETCH_SUCCESS' :
            return; {...state, isSuccess: true}
        case 'FETCH_FAIL':
            return; {...state, isFail: true}
        default:
            return INIT_STATE;
    }
}

function FlatState() {
    const [state, dispatch /*setter */] = useReducer(reducer/*바꾸는 함순*/, INIT_STATE/*초기값 */);

    const fetchData = () => {
        dispatch({type: ACTION_TYPE.FETCH_LOADING})

        fetch(url)
            .then(() => {
                dispatch({type: ACTION_TYPE.FETCH_SUCCESS})
            })
            .catch(() => {
                dispatch({type: ACTION_TYPE.FETCH_FAIL})
            })
    }

    if (state.isLoading) return <LoadingComponent/>
    if (state.isFinish) return <LoadingComponent/>
    if (state.isError) return <LoadingComponent/>
}
```


## <span style="color:#802548">_6.customHook으로 fetch도 치우기_</span>
- customHook은 무조건 use로 붙여야 한다.
- UI는 그리는 로직만 남고 나머지는 모두 추상화된다.
```js
const INIT_STATE = {
    isLoading: false,
    isSuccess: false,
    isFail : false
}

const ACTION_TYPE = {
    FETCH_LOADING: 'FETCH_LOADING',
    FETCH_SUCCESS: 'FETCH_SUCCESS',
    FETCH_FAIL : 'FETCH_FAIL'
}

const reducer = (state, action) => {
    swithc (action.type) { //if도 사용가능
        case  'FETCH_LOADING':
            return {...state, isLoading: true} //바뀐 상태 명시
        case 'FETCH_SUCCESS' :
            return; {...state, isSuccess: true}
        case 'FETCH_FAIL':
            return; {...state, isFail: true}
        default:
            return INIT_STATE;
    }
}

const useFetchData = (url) => {
    const [state, dispatch /*setter */] = useReducer(reducer/*바꾸는 함순*/, INIT_STATE/*초기값 */);

    useEffect(() => {
        const fetchData = () => {
            dispatch({type: ACTION_TYPE.FETCH_LOADING})

            fetch(url)
                .then(() => {
                    dispatch({type: ACTION_TYPE.FETCH_SUCCESS})
                })
                .catch(() => {
                    dispatch({type: ACTION_TYPE.FETCH_FAIL})
                })
        };

        fetchData();
    }, [url])
    

    return state;
}

function FlatState() {
    cosnt { isLoading, isFail, isSuccess } = useFetchData('url');

    if (isLoading) return <LoadingComponent/>
    if (isFinish) return <LoadingComponent/>
    if (isError) return <LoadingComponent/>
}
```

## <span style="color:#802548">_7.이전상태 가져와서 udpate하기_</span>
- setter는 비동기적이라서 그냥 바꾸면 timing에 따라 갱신이 안 될 수도 있다.
```js
function PrevState() {
    const [age, setAge] = useState(42);

    function updateState() {
        setAge(age + 1);
        setAge(age + 1);
        setAge(age + 1);
    }
}
```

- 따라서 이전 상태를 반드시 가져와서 변경한다.
```js
function PrevState() {
    const [age, setAge] = useState(42);

    function updateState() {
        setAge((prevAge) => prevAge + 1);
        setAge((prevAge) => prevAge + 1);
        setAge((prevAge) => prevAge + 1);
    }
}
```

- setter가 이전 상태를 가져오지 않는다.
- cardNumber handling에서 값이 바뀌면, cardCompany handling에서 예상치 못한 오류가 난다.
```js
const handleCardNumber = (cardNumber) => {
    setCardState({
        ...cardState,
        cardNumber
    });

    if (cardNumber.length === 8) {
        setOpenCardPopup(true);
    }
}

const handleCarCompany = (cardCompany) => {
    setCardState({
        ...cardState,
        ...cardCompany,
    })

    setOpenCardPopup(false);
}

return (
    <div>
        <input
            type="number"
            value={cardState.cardNumber}
            onChange={(e) => handleCardNumber(e.target.value)}
        />
        <input
            type="text"
            value={cardState.cardCompany}
            onChange={(e) => handleCardCompany(e.target.value)}
        />
    </div>
);
```

- 이전상태를 가져오게 하는 것만으로도 에러의 많은 부분을 방지한다.
```js
const handleCardNumber = (cardNumber) => {
    setCardState((prevState) => {
        ...prevState,
        cardNumber
    });

    if (cardNumber.length === 8) {
        setOpenCardPopup(true);
    }
}

const handleCarCompany = (cardCompany) => {
    setCardState((prevState) => {
        ...prevState,
        ...cardCompany,
    })

    setOpenCardPopup(false);
}

return (
    <div>
        <input
            type="number"
            value={cardState.cardNumber}
            onChange={(e) => handleCardNumber(e.target.value)}
        />
        <input
            type="text"
            value={cardState.cardCompany}
            onChange={(e) => handleCardCompany(e.target.value)}
        />
    </div>
);
```

## <span style="color:#802548">_8.props를 state로 선언하지 말자_</span>
- props로 가져온 값은 변하지 않기에 state로 선언할 필요가 없다.
```js
function CopyProps({value}) {
    const [copyValue] = useState(value);
    
    return (
        <div> {copyValue} </div>
    )
}
```

- props로 가져온 값은 state로 선언하지 않고 그냥 쓰면 된다.
```js
function CopyProps({value}) {
    return (
        <div> {value} </div>
    )
}
```

- rerendering될 때마다 연산이 일어나기 때문에, 아래처럼도 쓰면 안 된다.
```js
function CopyProps({value}) {
    const copyValue = 값_비싸고_무거운_연산(value);
    return (
        <div> {value} </div>
    )
}
```

- 만약 props를 받아서 무거운 연산을 하게 된다면, rerendering마다 연산하지 않게 하기 위해 useMemo를 활용하면 된다.
- 제일 좋은 것은 props를 복사하기 전에 이전 componenet에서 연산을 끝내는 것이다.
```js
function CopyProps({value}) {
    const copyValue = useMemo(() => 값비싸고_무거운_연산(value), [value]);
    return (
        <div> {copyValue} </div>
    )
}
```


## <span style="color:#802548">_9.curly braces_</span>
- 그냥 문자열 값은 curly braces를 안 써도 된다. 써도 상관은 없지만 말이다.
- 반면에 연산되는 값들은 모두 curly braces를 써야 한다.
- style 같은 double curly braces는 객체를 inline으로 넣은 것이라 생각하면 된다.
```js
function curlyBraces() {
    return (
        <hedaer
            className="clean-header"
            id="clean-code"
            style={{
                backgroundColor: 'blue',
                width: 1000
            }}
            value={1+2}
            value={isLogin && hasCookie}
            />
    )
}
```

## <span style="color:#802548">_10.shorthand props_</span>
- props를 비구조 할당을 쓰지 않고도 쓸 수 있다.
```js
function ShortHandProps(props) {
    return (
        <header
            className="clean-header"
            title="Clean Code React"
            isDarMode={true}
            isLogin={true}
            hasPadding={true}
            isFixed={true}
            isAdmin={true}
            />
    )
}
```

- hasPadding, isFixed, isAdmin이 항상 boolean type이고 true면 shorthand property를 사용하면 편한다.
```js
function ShortHandProps({isDarMode, isLogin, ...props}) {
    return (
        <header
            className="clean-header"
            title="Clean Code React"
            isDarMode={isDarMode}
            isLogin={isLogin}
            hasPadding
            isFixed
            isAdmin
            />
    )
}
```


## <span style="color:#802548">_11. 따옴표_</span>
- html은 큰 따옴표를 쓴다.
- js는 작은 따옴표를 쓴다.
```js
const obj = {
    hello: 'world'
}

<a href="https://~~~~" />
```

- 신경쓰지 않으려면 eslint, prettier rule에다가 jsx관련 quote 룰을 추가한다.



## <span style="color:#802548">_12.props naming_</span>
- 문제는 없지만, 잘못된 표식을 가진 props다.
```js
function PropsNaming() {
    return (
        <ChildComponent
            class="mt-0"
            Clean="code"
            clean_code="react"
            otherComponent={OtherComponent}
            isShow={true}
        />
    )
}
```

- class -> className이다
- props가 대문자면, componenet를 기대한다
- props도 key라서 camelCase로 써야 한다.
- isShow가 늘 true라면 shorthand로 쓰면 된다.
```js
function PropsNaming() {
    return (
        <ChildComponent
            className="mt-0"
            clean="code"
            cleanCode="react"
            OtherComponent={OtherComponent}
            isShow
        />
    )
}
```

## <span style="color:#802548">_12.inline CSS_</span>
- jsx에서는 문자열로 inline에 넣을 수 없다. 작동하지 않는다.
```js
function InLineSylte() {
    return (
        <button style="background-color: 'red'; font-size: '14px';">
            Clean Code
        </button>
    )
}
```

- 객체로 바꾼 것 같지만 아니다. style을 객체로 넣어야 하기 때문에 중괄호를 하나 더 집어넣어야 한다.
```js
function InLineSylte() {
    return (
        <button style={backgroundColor: 'red'; fontSize: '14px'}>
            Clean Code
        </button>
    )
}
```

- 중괄호로 inline으로 넣으면 나중에 수정할 때 번거롭다.
```js
function InLineSylte() {
    return (
        <button style={{backgroundColor: 'red'; fontSize: '14px'}}>
            Clean Code
        </button>
    )
}
```

- rendering될 때마다 평가되므로 component 밖으로 빼야한다.
```js
function InLineSylte() {
    const myStyle = {backgroundColor: 'red'; fontSize: '14px'};
    return (
        <button style={myStyle}>
            Clean Code
        </button>
    )
}
```

- button에 따른 css를 다르게 만들어달라는 확장 요건이 올 수 있다.
- 해당 점을 고려해 다시 한번 key - value를 만들어주자.
```js
const myStyle = {backgroundColor: 'red'; fontSize: '14px'};
function InLineSylte() {
    return (
        <button style={myStyle}>
            Clean Code
        </button>
    )
}
```


- 확장성있는 css가 완성됐다.
- 그러나 바람직한 건 css 파일을 사용하는 것이다.
```js
const myButtonStyle = {
    warning: {backgroundColor: 'yellow'; fontSize: '14px'}
    danger: {backgroundColor: 'red'; fontSize: '24px'}
};

function InLineStyle() {
    return (
        <button style={myButtonStyle.warning}>Warning Code Click!</button>
        <button style={myButtonStyle.danger}>Danger Code Click!</button>
    )
}
```

## <span style="color:#802548">_12.inline CSS 지양하기_</span>

- inline css라서 보기 지저분하다.
```js
export function Card({title, children}) {
    return (
        <div css ={css`
            background-colodr: white;
            border: 1px solid #eee;
            border-radius: 0.5rem;
            padding: 1rem;
        `}>
            <h5 css={`
                font-size: 1.25rem;
            `}>
                {title}
            </h5>
            {childeren}
        </div>
    )
}
```

- 늘 그렇듯 외부의 객체로 뺀다.
- rendering될 때마다 계산되지 않게끔 바꿔줄 수 있다.
```js
const cardCss = { 
    self: css`
            background-colodr: white;
            border: 1px solid #eee;
            border-radius: 0.5rem;
            padding: 1rem;
        `,

    title: css`
            font-size: 1.25rem;
    `
}
export function Card({title, children}) {
    return (
        <div css ={cardCss.css}>
            <h5 css={cardCss.title}>
                {title}
            </h5>
            {childeren}
        </div>
    )
}
```


- 자동완성 등에 도움을 받기 위해 jsx 문법을 사용하는 것으로 바꾼다.
- 외부로 빼면 export해서 다른 component에서 쓰기도 좋다.
- 특히 js에서 css 변경은 성능에 민감하기 때문에 엔간하면 안 하는 게 좋다. 외부로 빼자.
```js
const cardCss = { 
    self: css{{
            backgroundColodr: white,
            border: 1px solid #eee,
            borderRadius: 0.5rem,
            padding: 1rem,
    }},
    title: css{{
        fontSize: 1.25rem,
    }}
}
export function Card({title, children}) {
    return (
        <div css ={cardCss.css}>
            <h5 css={cardCss.title}>
                {title}
            </h5>
            {childeren}
        </div>
    )
}
```


## <span style="color:#802548">_13. 객체 props 지양하기_</span>
- dependency array에서 비교할 때 쓰는게 Object.is()다
- 그런데 값이 같아도 객체면 주소값이 달라서 false가 된다. 따라서 rerendering이 된다.
```js
Object.is(
    {hello:'world'},
    {hello:'world'},
) //false

Object.is(
    ["hello"], 
    ["hello"], 
) //false
```

- 따라서 객체를 쓰면 안 좋다.
```js
function SomeComponent() {
    return (
        <ChildComponent
            propObj={{hello:'World'}}
            propArr={["hello","hello"]}
    )
}
```

- 필요한 값만 보낸다.
- 문자열로 뺄 수도 있고, useState로 사용할수도 있다.
```js
function SomeComponent() {
    const [propArr, setPropArr]=useState(["hello","hello"]);

    return (
        <ChildComponent
            hello='world'
            hello2={propArr.at(0)}
    )
}
```

- 정말 무거운 연산의 경우에는 useMemo를 쓸 수도 있다.
```js
function SomeComponent({heavyState}) {
    const [propArr, setPropArr]=useState(["hello","hello"]);

    const computedState = useMemo(() => {
        computedState: heavyState
    }, [heavyState]);

    return (
        <ChildComponent
            hello='world'
            hello2={propArr.at(0)}
            computedState={{
                heavyState: heavyState
            }}
    )
}
```


## <span style="color:#802548">_14.HTML attribute 주의_</span>
- props 선언 시에 html attribute와 겹치는지 살펴보아야 한다.
- js 기본 keyword도 쓰지 않아야 한다.
```js
function HTMLDefaultAttribute() {
    const MyButton = ({ children, ...rest}) => (
        <button {...rest}>{children}</button>
    )

    return (
        <>
        <MyButton className="mt-0" type="submit">
            Clean Code
        </MyButton>

        <input type="number" maxlength={99}
        </>
    )
}
```


## <span style="color:#802548">_14.너무 많은 props 걸러내기_</span>
```js
const ParentComponent = (props) => {
    return <ChildOrHOCComponent {...props} />
}

class ParentComponent extends React.Component {
    render() {
        return <ChildOrHOCComponent {...this.props} />
    }
}
```

- 관련 없는 props는 미리 발라낼 수 있다면 발라낸다.
```js
const ParentComponent = (props) => {
    const {관련_없는_props, 관련_있는_props, ...나머지_props} = props;
    return <ChildOrHOCComponent 
                관련_있는_props={관련_있는_props}
                {...나머지_props}
            />
}

class ParentComponent extends React.Component {
    const {관련_없는_props, 관련_있는_props, ...나머지_props} = props;
    render() {
        return <ChildOrHOCComponent 
                관련_있는_props={관련_있는_props}
                {...나머지_props}
                />
    }
}
```


## <span style="color:#802548">_14.너무 많은 props 걸러내기_</span>
- component 분리가 먼저다.
```js
const App = () => {
    return (
        <JoinForm
            user={user}
            auth={auth}
            location={location}
            favorit={favorite}
            handleSubmit={handleSubmit}
            handleReset={handleReset}
            handleCancel={handleCancel}
        />
    )
}
```

```js
const App = () => {
    return (
        <JoinForm
            onSubmit={handleSubmit}
            onReset={handleReset}
            onCancel={handleCancel}
        >
            <UserForm user={user} />
            <AuthForm auth={auth} />
            <LocationForm location={location} />
            <FavoriteForm favorite={favorite} />
        </JoinForm>
    )
}
```


- 확장성을 위해 도메인 로직은 다른 곳으로 모아 넣는다.
- context api나 customHook을 사용해서 도메인 로직을 만들어놓으면 된다.
```js
const App = () => {
    return (
        <JoinForm
            onSubmit={handleSubmit}
            onReset={handleReset}
            onCancel={handleCancel}
        >
            <CheckBoxForm formData={user} />
            <CheckBoxForm formData={auth} />
            <RadioButtonForm formData={location} />
            <SectionForm formData={favorite} />
        </JoinForm>
    )
}
```


- ChatGPT 추천 결과 아래와 같이 context api를 작성할 수 있다고 한다.. 
- React 플젝 이제 슬슬 한번 해봐야겠다.
```js
import React, { createContext, useContext, useState } from 'react';

// Step 1: Create a context to hold the form data
const FormDataContext = createContext();

// Step 2: Create a custom hook to access the context
const useFormData = () => useContext(FormDataContext);

// Step 3: Wrap your JoinForm with the FormDataContext.Provider
const App = () => {
    // State to hold form data
    const [formData, setFormData] = useState({});

    // Handlers for form actions
    const handleSubmit = () => {
        // Handle form submission
    };

    const handleReset = () => {
        // Handle form reset
    };

    const handleCancel = () => {
        // Handle form cancel
    };

    return (
        <FormDataContext.Provider value={{ formData, setFormData }}>
            <JoinForm
                onSubmit={handleSubmit}
                onReset={handleReset}
                onCancel={handleCancel}
            >
                {/* Pass formData to each child component */}
                <CheckBoxForm />
                <RadioButtonForm />
                <SectionForm />
            </JoinForm>
        </FormDataContext.Provider>
    );
};

// In each child component, use the useFormData hook to access the form data
const CheckBoxForm = () => {
    const { formData, setFormData } = useFormData();

    // Access formData.user and formData.auth here

    return (
        // Your CheckBoxForm JSX here
    );
};

const RadioButtonForm = () => {
    const { formData, setFormData } = useFormData();

    // Access formData.location here

    return (
        // Your RadioButtonForm JSX here
    );
};

const SectionForm = () => {
    const { formData, setFormData } = useFormData();

    // Access formData.favorite here

    return (
        // Your SectionForm JSX here
    );
};

export default App;
```


## <span style="color:#802548">_14.객체보다는 간단한 props_</span>
- props를 객체로 가져와버리면 불필요한 객체속성도 들어오게 된다.
```js
function UserInfo({user}) {
    return (
        <div>
            <img src={user.avatarImgUrl}/>
            <h3>{user.userName}</h3>
            <h4>{user.email}</h4>
        </div>       
    )
}
```

- 처음 보낼 때 필요한 것만 보내자.
```js
function UserInfo({avatarImgUrl, userName, email}) {
    return (
        <div>
            <img src={avatarImgUrl}/>
            <h3>{userName}</h3>
            <h4>{email}</h4>
        </div>       
    )
}
```
