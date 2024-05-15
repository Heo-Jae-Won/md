## <span style="color:#802548">_1.component 분리_</span>
- component 분리는 React의 시작이자 끝이다.

<img src="/image/component-seperation.png"   />

- 위와 같이 만들기 위해서 나는 아래처럼 생각했다.
```js
<FilterableProductTable>
    <SearchBar />
    <ProductTable /> 
    <ProducTCategoryRow />
    <ProductRow/>
</FilterableProductTable>
```


- 그러나 실제로는 아래와 같은 형태다.
```js
<FilterableProductTable>
    <SearchBar />
    <ProductTable /> //<ProducTCategoryRow /> <ProductRow/>는 //ProductTable안에..
</FilterableProductTable>
```


- 구체적으로 나타내면 아래와 같다.
- 아래는 mockup data다.
```js
const PRODUCTS = [
  {category: "Fruits", price: "$1", stocked: true, name: "Apple"},
  {category: "Fruits", price: "$1", stocked: true, name: "Dragonfruit"},
  {category: "Fruits", price: "$2", stocked: false, name: "Passionfruit"},
  {category: "Vegetables", price: "$2", stocked: true, name: "Spinach"},
  {category: "Vegetables", price: "$4", stocked: false, name: "Pumpkin"},
  {category: "Vegetables", price: "$1", stocked: true, name: "Peas"}
];
```

- FilterableProductTable componenet 안에 아래 component가 있다
  - SearchBar
  - ProductTable. products를 props로 넘긴다.
```js
export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}

function FilterableProductTable({ products }) {
  return (
    <div>
      <SearchBar />
      <ProductTable products={products} />
    </div>
  );
}
```

- 실제로 반응성있게 작동하게 하려면 state를 만들어야 한다.
  - 기본값을 주는 것을 잊지 말자.
  - SearchBar에는 changeEventHandler도 넘겨준다.
```js
export default function App() {
  return <FilterableProductTable products={PRODUCTS} />;
}

function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar 
        filterText={filterText} 
        inStockOnly={inStockOnly} 
        onFilterTextChange={setFilterText} 
        onInStockOnlyChange={setInStockOnly} />
      <ProductTable 
        products={products} 
        filterText={filterText}
        inStockOnly={inStockOnly} />
    </div>
  );
}
```

- ProdcutTable안에는 아래 같은 component가 있다.
  - ProductCategoryRow
  - ProdcutRow
```js
function ProductTable({ products }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}
```

- props로 내려보낸 products에 문제가 있으면 rendering을 하지 않는다. 빈화면이다
- 문제가 없다면 rendering을 시작해 UI를 채운다.
```js
function ProductTable({ products, filterText, inStockOnly }) {
  const rows = [];
  let lastCategory = null;

  products.forEach((product) => {
    if (
      product.name.toLowerCase().indexOf(
        filterText.toLowerCase()
      ) === -1
    ) {
      return;
    }
    if (inStockOnly && !product.stocked) {
      return;
    }
    if (product.category !== lastCategory) {
      rows.push(
        <ProductCategoryRow
          category={product.category}
          key={product.category} />
      );
    }
    rows.push(
      <ProductRow
        product={product}
        key={product.name} />
    );
    lastCategory = product.category;
  });

  return (
    <table>
      <thead>
        <tr>
          <th>Name</th>
          <th>Price</th>
        </tr>
      </thead>
      <tbody>{rows}</tbody>
    </table>
  );
}
```

- 검색바 component는 아래와 같이 구성한다.
```js
function SearchBar() {
  return (
    <form>
      <input type="text" placeholder="Search..." />
      <label>
        <input type="checkbox" />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}
```

- 검색바에는 props로 받은 changeEventHandler를 등록시켜준다.
- 콜백으로 등록해야하므로 () => 함수명(parameter)로 써주고, jsx라서 js요소는 {}안에 넣어야 한다.
```js
function SearchBar({
  filterText,
  inStockOnly,
  onFilterTextChange,
  onInStockOnlyChange
}) {
  return (
    <form>
      <input 
        type="text" 
        value={filterText} placeholder="Search..." 
        onChange={(e) => onFilterTextChange(e.target.value)} />
      <label>
        <input 
          type="checkbox" 
          checked={inStockOnly} 
          onChange={(e) => onInStockOnlyChange(e.target.checked)} />
        {' '}
        Only show products in stock
      </label>
    </form>
  );
}
```

- 실제 ProductCategoryRow와 ProductRow는 props를 받아서 UI를 render한다.
```js
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}
```


- productCategoryRow와 productRow는 변화할 게 없다. 그냥 보여주기만 하기 때문이다.
```js
function ProductCategoryRow({ category }) {
  return (
    <tr>
      <th colSpan="2">
        {category}
      </th>
    </tr>
  );
}

function ProductRow({ product }) {
  const name = product.stocked ? product.name :
    <span style={{ color: 'red' }}>
      {product.name}
    </span>;

  return (
    <tr>
      <td>{name}</td>
      <td>{product.price}</td>
    </tr>
  );
}
```


## <span style="color:#802548">_2.self closing tag_</span>
- 명시적으로 닫는 태그가 필요없다.
- mdn 문서에 void element를 보면 된다.
- 기본 태그의 경우 self - closing 태그이므로 />처리가 필요없다.
```
area
br
img
input
hr
```

## <span style="color:#802548">_3.fragment_</span>
- 단일 component가 아니라 에러가 난다.
```js
function Example() {
    return (
        <ChildA />
        <ChildB />
        <ChildC />
    )
}
```

- fragment를 사용하면 된다.
```js
function Example() {
    return (
        <React.Fragement>
            <ChildA />
            <ChildB />
            <ChildC />
        </React.Fragement>
    )       
}
```


- fragment를 명시하지 않고 싶다면 생략가능하다.
```js
function Example() {
    return (
        <>
            <ChildA />
            <ChildB />
            <ChildC />
        </>
    )       
}
```

- fragment를 명시해야 하는 때는 언제인가?
- componenet를 UI render할 때 id 값에 따라 다르게 한다면 필요할 수 있다.
- shortcut은 React 16부터 사용가능하며 shorcut은 바벨 7버전부터 된다.
```js
function ShortCutFragment(items) {items:'any',"": any } {
    return (
        {items.map((item) => (
            <React.Fragment key={id}
                <dt>{item.term}</dt>
                <dd>{item.description}</dd>
            </React.Fragment>
        ))}
    )
}
```

- map 안에서도 비구조할당 수 있다.
```js
function ShortCutFragment(items) {items:'any',"": any } {
    return (
        {items.map(({term,description,id}) => (
            <React.Fragment key={id}
                <dt>{term}</dt>
                <dd>{description}</dd>
            </React.Fragment>
        ))}
    )
}
```


- 현 React는 component에서 string을 return해도 된다.
- React Fragment로 감쌀 필요가 없다.
```js
function StringRender() {
    return <>Clean Code React</>
}

//아래처럼.

function StringRender() {
    return 'Clean Code React'
}

// 아래도 됨.

function StringRender() {
    return ['Clean','Code','React']
}
```

- 아무것도 rendering하지 않을 때도 fragment를 사용한다.
  - 그보다는 null return이 낫다.
  - 그보다는 && 평가가 낫다.
  - 그보다는 평가가 true일 때 dom을 그리는 게 낫다.
```js
function ConditionalRedneringExample({isLoggedIn}) {
    return (
        <div>
            <h1>{isLoggedIn ? 'User' : <></>}</h1>
            <h1>{isLoggedIn ? 'User' : null}</h1>
            <h1>{isLoggedIn && 'User'}</h1>
            {isLoggedIn && <h1>'User'</h1>}
        </div>
    )
}
```


## <span style="color:#802548">_4.component naming_</span>
- html 기본 element는 lower case다.
- component 요소는 Pascal case다.
- component 요소를 kebab case로 쓰면 안 된다.
```js
function ComponentNaming() {
    return (
        <>
            <h1></h1>
            <h2></h2>
            <div></div>
            <input />
            <MyComponent></MyComponent>
            <!--<my_Component></my_component>-->
        </>
    )
}
```

## <span style="color:#802548">_4.함수 component는 직접호출하지 않는다_</span>
```js
function ReturnJSXFunction() {
    const TopRender = () => {
        return (
            <header>
                <h1>Clean Code JavaScript</h1>
            </header>
        )
    }
}

const renderMain = () => {
    return (
        <main>
            <p>Clean Code React</p>
        </main>
    )
}
```

- 아래처럼 해도 그려지지만, 이것이어떤 의도인지 파악하기 어렵다.
```js
return (
    <div>
        {TopRender()}
        {renderMain()}
    </div>
)
```

- props를 내릴 수 있는 공인된 형태로 쓰자.
```js
return (
    <div>
        <TopRender />
        <renderMain />
    </div>
)
```

## <span style="color:#802548">_5.컴포넌트 내부에 컴포넌트를 선언하지 않는다_</span>
- component 내부에 만들면 component를 만들면 결합도가 늘어난다.
  - 나중에 상위 component를 고칠 때 하위 component에 보내는 state들이 문제가 된다.
- 상위 component가 rerender될 때 하위 component도 재생성된다.
  - 성능저하가 생길 수밖에 없다.
```js
function OuterComponent() {
    const InnerComponent = () => {
        return (
            <div>
                Inner component
            </div>
        )
    }

    return (
        <div>
            <InnerComponent />
        </div>
    )
}
```

- 따로 뺴서 만들고 import해서 써야 한다.
```js
const InnerComponent = () => {
        return (
            <div>
                Inner component
            </div>
        )
    }

function OuterComponent() {
    return (
        <div>
            <InnerComponent />
        </div>
    )
}
```

- 성급한 분리인지도 고민해봐야 한다.
- 성급한 분리라면 아래처럼 안으로 옮겨준다.
```js
function OuterComponent() {
    return (
        <div>
           'Inner component'
        </div>
    )
}
```


## <span style="color:#802548">_6.displayName을 사용해 React dev tool debugging 하기_</span>
- displayName을 설정해두면 React dev tools에 component 이름이 명시된다.
```js
const InputText = forwardRef((props, ref) => {
    return <input type="text" ref={ref} />

})

InputText.displayName = 'InputText'
```

- 또다른 예시다.
```js
const withRouter = (Component) => {
    const WithRouter = (props) => {
        const location = useLocation();
        const navigate = useNavigate();
        const params = useParams();
        const navigationType = useNavigationType();

        return (
            <Component
                {...props}
                location = {location}
                navigate = {navigate}
                params = {params}
                navigationType = {navigationType}
            />
        )
    }
}
WithRouter.displayName = Component.displayName ?? Component.name ?? 'WithRouterComponent';
```

## <span style="color:#802548">_7.추천 component 구성_</span>
- 변하지 않는 상수는 component 외부로 뺀다.
- interface도 다른 파일에 빼거나, component 외부로 뺀다.
```js
const DEFAULT_COUNT = 100;

interface SomeComponentProps {
    
}
```


- 코드 최상위에 react 관련 함수를 놓는다.
  - react 관련 use붙은 hook 중 외부 library는 최상위에 놓는다.
  - customhook을 그 아래 놓고, 그 아래 useState, usereducer, useRef를 놓는다.
```js
const DEFAULT_COUNT = 100;

interface SomeComponentProps {
    
}

const SomeComponent = (props) => {
    const location = useLocation();
    const queryClient = useQueryClient();
    const state = useSelector((state) => state);

    const state = useCustomHooks((state) => state);

    const [state,setState] = useState("someState");
    const ref = useRef(null);

    const onClose => handlerClose();
}
```

- props는 받아와서 비구조할당을 실행한다.
```js
const DEFAULT_COUNT = 100;

interface SomeComponentProps {
    
}

const SomeComponent = (props) => {
    const {unusedProps, prop1, prop2, ...props} = props;
    const location = useLocation();
    const queryClient = useQueryClient();
    const state = useSelector((state) => state);

    const state = useCustomHooks((state) => state);

    const [state,setState] = useState("someState");
    const ref = useRef(null);

}
```

- eventhandler 함수들은 component 외부에 뺀다.
- state를 받을 때는 parameter로 받으면 된다.
```js
const DEFAULT_COUNT = 100;

interface SomeComponentProps {
    
}

const handlerClose = (state) => {
    //business logic...
}

const SomeComponent = (props) => {
    const {unusedProps, prop1, prop2, ...props} = props;
    const location = useLocation();
    const queryClient = useQueryClient();
    const state = useSelector((state) => state);

    const state = useCustomHooks((state) => state);

    const [state,setState] = useState("someState");
    const ref = useRef(null);

    const onClose = () => handleClose();
    let isHold = false;
}
```


- early return도 가능하다.
```js
const DEFAULT_COUNT = 100;

interface SomeComponentProps {
    
}

const handlerClose = (state) => {
    //business logic...
}

const SomeComponent = (props) => {
    const {unusedProps, prop1, prop2, ...props} = props;
    const location = useLocation();
    const queryClient = useQueryClient();
    const state = useSelector((state) => state);

    const state = useCustomHooks((state) => state);

    const [state,setState] = useState("someState");
    const ref = useRef(null);

    const onClose = () => handleClose();
    let isHold = false;

    if (!isHolde) {
        return <div>데이터가 없는데요;;</div>
    }
}
```

- useEffect는 최하단에 HTML과 가깝게 놓는다.
  - 엔간하면 한개만 있는 게 좋다. 
- HTML과 useEffect는 개행을 한칸은 무조건 준다.
```js
const DEFAULT_COUNT = 100;

interface SomeComponentProps {
    
}

const handlerClose = (state) => {
    //business logic...
}

const SomeComponent = (props) => {
    const {unusedProps, prop1, prop2, ...props} = props;
    const location = useLocation();
    const queryClient = useQueryClient();
    const state = useSelector((state) => state);

    const state = useCustomHooks((state) => state);

    const [state,setState] = useState("someState");
    const ref = useRef(null);

    const onClose = () => handleClose();
    let isHold = false;

    if (!isHolde) {
        return <div>데이터가 없는데요;;</div>
    }

    useEffect( () => {

    }, []);

    return (
        <div className="tooltip">
            <div className="msg">Hello World</div>
            <button
                className="close"
                type="button"
                onClick={onClose}
            />
        </div>
    )
}
```

- styled component는 export default 바로 위에 HTML과 가깝게 놓는다.
```js
const DEFAULT_COUNT = 100;

interface SomeComponentProps {
    
}

const handlerClose = (state) => {
    //business logic...
}

const SomeComponent = (props) => {
    const {unusedProps, prop1, prop2, ...props} = props;
    const location = useLocation();
    const queryClient = useQueryClient();
    const state = useSelector((state) => state);

    const state = useCustomHooks((state) => state);

    const [state,setState] = useState("someState");
    const ref = useRef(null);

    const onClose = () => handleClose();
    let isHold = false;

    if (!isHolde) {
        return <div>데이터가 없는데요;;</div>
    }

    useEffect( () => {
        //fetch api
    }, []);

    return (
        <div className="tooltip">
            <div className="msg">Hello World</div>
            <button
                className="close"
                type="button"
                onClick={onClose}
            />
        </div>
    )
}

const Button = styled.a<{$primary? boolean;}>`
    padding: 0.5rem 0;
    transition: all 200ms ease-in-out;
    width: 11rem;

    $:hover {
        filter: brightness(0.85)
    }
`

export default SomeComponent;
```

## <span style="color:#802548">_8.jsx 공백처리_</span>
- &nbsp;로 공백을 넣을 필요 없다.
- {}안에 공백 문자열을 넣으면 된다.
```js
return (
    <div>
        Welcome Clean Code{' '}
)
```


## <span style="color:#802548">_9.JSX에서는 0이 유효하다._</span>
- js에서는 0이 falsy하다.
- jsx에서는 falsy해도 rendering의 대상이 된다.
- 아래처럼 쓰면 items가 없어도 rendering된다. 0으로..
```js
return <div>
    {items.length && items.map((item) => <Item item={item} />)
    </div>};
```

- render를 막으려면 0보다 클 때로 바꿔줘야 한다.
- truthy나 falsy가 아니라 true나 false냐를 명확하게 조건으로 만들어주는 습관을 들이자.
```js
return (
    <div>
        {items.length > 0 && items.map( (item) => <Item item={item}/> )}
    </div>;
    )
```

```js
return (
    <div>
        {items.length > 0 ? items.map( (item) => <Item item={item}/> ): null}
    </div>;
    )
```


## <span style="color:#802548">_10.list의 rendering unique key_</span>
- virtual dom이라서 list에서 DOM을 찍어내면 구분이 어렵다.
- 따라서 key를 주어야만 한다.
```js
function KeyInList() {
    return (
        <>
            <ul>
                {list.map((item) => {
                    <li>{item}</li>
                })}
            </ul>
        </>
    )
}
```

- 작은 project면 그냥 index를 써도 된다.
```js
<>
    <ul>
        {list.map((item, index) => {
            <li key={index}>{item}</li>
        })}
    </ul>
</>
```

- 아니면 좀 더 React가 구분하기 쉽게 prefix를 사용하기도 한다.
```js
 <>
    <ul>
        {list.map((item) => {
            <li key={'card-item-' + index}>{item}</li>
        })}
    </ul>
</>
```

- 최악은 아래와 같은 형태다.
- rendering될 때마다 고유한 값을 가져와야 돼서 느려진다.
```js
 <>
    <ul>
        {list.map((item) => {
            <li key={new Date().toString()}>{item}</li>
        })}
    </ul>
</>
```

- 마찬가지로 아래처럼해도 느려진다. 고유한 값인지만 중요한 게 아니다.
- 성능도 중요하다.
```js
 <>
    <ul>
        {list.map((item) => {
            <li key={uuidv4()} >{item}</li>
        })}
    </ul>
</>
```


- setItem이 되는 순간 rerender가 된다. state가 바뀌었기 때문이다.
  - useEffect는 그자체로는 re-render와 관련있진 않다. 
  - 단지 보통 useEffect 안에 setter function이 있을 뿐이다.
- useEffect의 deps array argument는 늘 state여야한다.
  - 그래야 state change가 감지되어 useEffect가 발동된다.
- 또한 list에서 unique한 key값을 만드는 시점이 rendering 시점이면 안된다.
  - rendering은 component마다 2회부터 10회까지 매우 빈번하게 일어나기 때문이다.
  - 따라서 list를 rendering하기 전에 미리 unique한 key값인 id를 만들어 넣어주자.
```js

function KeyInList() {
    const [items, setItems] = useState([]);
    const [page,setPage] = useState(1);
    const handleAddItem = (value) => {
        setItems((prev) => [
            ...prev,
            {
                id: crypto.randomUUID(),
                ...value
            }
        ]);
    };

    const handlePageUp = () => {
        setPage((prevPage) => prevPage + 1);
    }

    useEffect(() => {
        const fetchData = async () => {
            const response = await getScheduleList(page);
            response.data.scheduleList = [
                                            {'scheduleNo':15,'stationNo':10,'departTime':'2024-01-13 06:15:00'},
                                            {'scheduleNo':14,'stationNo':10,'departTime':'2024-01-13 06:15:00'},
                                            {'scheduleNo':13,'stationNo':10,'departTime':'2024-01-13 06:15:00'}
                                            {'scheduleNo':12,'stationNo':10,'departTime':'2024-01-13 06:15:00'}
                                            {'scheduleNo':11,'stationNo':10,'departTime':'2024-01-13 06:15:00'}
                                        ]
            response.data.scheduleList.forEach(item => {
                handleAddItem(item);
            });
        };

        fetchData();
    }, [page]);

    return (
        <>
        <button onClick={handlePageUp}>page up</button> 
            <ul>
                {list.map( (item) => (
                    <li key={item.id} onClick={() => handleDelete(item.id)}>
                        {item}
                    </li>
                ))}
            </ul>
        </>
    )
}
```

## <span style="color:#802548">_11.안전한 rawHTML_</span>

- inline html을 그대로 넣으면 위험하다.
```js
const SERVER_DATA = '<p>some raw html</p>';

function DangerouslySetInnerHTMLExampl() {
    const post = {
        content: `<img src="" onerror='alert("you were hacked")'
    };
}
```

- 서버에서 받아온 HTML을 그대로 넣는 것은 더 위험하다.
```js
const SERVER_DATA = '<p>some raw html</p>';

function DangerouslySetInnerHTMLExampl() {
    const markup = {_html: SERVER_DATA};

    return <div>{markup}</div>
}
```

- dangerouslySetInnerHTML이라도 활용하자.
```js
const SERVER_DATA = '<p>some raw html</p>';

function DangerouslySetInnerHTMLExampl() {
    const markup = {_html: SERVER_DATA};

    return <div dangerouslySetInnerHTML={markup} />
}
```

- rawHTML은 꼭 소독하자..
- 유저가 수정할 수 있는 것들은 dompurify library를 활용하자.
- 혹시 까먹었을 수 있으니 eslint-plugin-risxx plugin을 깔아서 주의를 기울이지 않고 기계가 오류를 내게 자동화하자.
```js
const SERVER_DATA = '<p>some raw html</p>';

function DangerouslySetInnerHTMLExample() {
    const [contentHTML, setContentHTML] = useState('');
    const sanitizeContent = {_html: DOMPurify.sanitize(SERVER_DATA)};
    setContentHTML(DOMPurify.sanitize(SERVER_DATA));
    return <div dangerouslySetInnerHTML={sanitizeContent} />
}
```


## <span style="color:#802548">_12.기명함수로 useEffect 사용하기_</span>

- 함수가 익명이라 무엇을 하는 지 알기 어렵다.
```js
useEffect( () => {
  if (isInView(someRef.current)) {
    //some logic
  }
}, [isInView]);
```

- useEffect로 등록하는 callback 함수 안에도 이름을 주자.
```js
useEffect( function onPopState() => {
  if (isInView(someRef.current)) {
    //some logic
  }
}, [isInView]);
```


- 사실 아래는 만들지 않는 게 좋은 함수지만... 
- 굳이 쓴다면 함수 이름을 달아주자.
```js
useEffect(function onInit() => {
  if (!isInit) {
    return;
  }

  setIsInit(false);
}, [isInit]);
```


- 여기도 이름이 없다.
```js
useEffect( () => {
  docuent.addEventListner();

  return () => {
    document.removeEventListerner();
  }
},[]);
```

- 이름을 달아주면 좋은 장점은 뭘까?
- 로그로 보았을 때 함수에 이름이 달려서 call stack으로 나온다는 사실이다.
```js
useEffect( function addEvent() {
  docuent.addEventListner();

  return function removeEvent() {
    document.removeEventListerner();
  }
},[]);
```


- 보통 useEffect안에서는 data fetch가 자주 이뤄지니 아래 같이 사용할 수도 있다.
- dom이 unmount되면 request를 보냈어도 response를 handle하지 않는다.
- response를 handle하지 않으므로 없는 dom에 대한 code가 진행되지 않나 error가 나지 않을 것이다.
```js
useEffect( function fetchData() {
    // set our variable to true
    let isApiSubscribed = true;
    axios.get(API).then((response) => {
        if (isApiSubscribed) {
            // handle success
        }
    });
    return function cleaup() {
        // cancel the subscription
        isApiSubscribed = false;
    };
}, []);
```

- 위를 axios 공식 api를 사용하게 바꾼 것이다.
```js
useEffect( function fetchData() {
  const CancelToken = axios.CancelToken;
  const source = CancelToken.source();
  axios
    .get(API, {
      cancelToken: source.token
    })
    .catch((err) => {
      if (axios.isCancel(err)) {
        console.log('successfully aborted');
      } else {
        // handle error
      }
    });
  return function cancelRequest() {
    // cancel the request before component unmounts
    source.cancel();
  };
}, []);
```

- 위를 fetch api를 사용하게 바꾼 것이다.
```js
import React, { useState, useEffect } from "react";
export default function Post() {
  const [posts, setPosts] = useState([]);
  const [error, setError] = useState(null);

  useEffect( function fetchData() {
    const controller = new AbortController();
    const signal = controller.signal;
    fetch("https://jsonplaceholder.typicode.com/posts", { signal: signal })
      .then((res) => res.json())
      .then((res) => setPosts(res))
      .catch((err) => setError(err));
  }, []);

  return (
    <div>
      {!error ? (
        posts.map((post) => (
          <ul key={post.id}>
            <li>{post.title}</li>
          </ul>
        ))
      ) : (
        <p>{error}</p>
      )}
    </div>
  );
}
```

- xmlHttpRequest를 abort해도 server는 여전히 response를 준다.
- 다만 abort()를 호출하면 browser에서 해당 response를 무시할 뿐이다.

## <span style="color:#802548">_13.useEffect에는 하나의 역할만_</span>
- useEffect에 넘기는 함수에 기명함수를 적어보고 그에 맞는 코드인지 확인하자.
- deps array에 여러 state가 들어가면 하나의 역할인지 확인하자.
- 
```js
function LoginPage({page, newPath}) {
  useEffect( () => {
    redirect(newPath);

    const userInfo = setLogin(token);
    //login business logic

  }, [token, newPath])
}
```

- token이 바꼈을 떄, newPath가 바꼈을 때 useEffect를 분리해주자.
- 둘은 다른 로직이기에 분리가 맞다.
```js
function LoginPage({page, newPath}) {
  useEffect( () => {
    const userInfo = setLogin(token);

  }, [token])

  useEffect( () => {
    redirect(newPath);
  }, [newPath])
}
```

## <span style="color:#802548">_14.비구조 할당으로 customHook을 다양하게 써보기_</span>
- customHook이라고 해서 반드시 tuple을 반환하는 게 아니다.
- 다양하게 반환이 가능하다.
  - 튜플
  - 단일 변수값
  - 객체
- 다만 튜플 반환 시 주의점이 있다.
- 관습에 따라 왼쪽에 getter, 오른쪽에 setter를 넣자.
```js
const useSomeHooks = (bool) => {
  return [state, setState]; //[setState, state] X
}

function ReturnCustomHooks() {
  const [setValue, value] = useSomeHooks(true);
}
```

- 하나만 return한다면 getter, setter tuple을 쓸 필요가 없다.
- 그냥 단일한 변수로 return하면 된다.
```js
const useSomeHooks = (bool) => {
  return state; //[state, setState] X
}

function ReturnCustomHooks() {
  const value = useSomeHooks(true);
}
```

- 배열을 비구조할당해 customHook으로 만드는 건 유지보수에 좋지 않다.
- 배열의 요소가 늘어나면 대응이 되지 않는다.
- 안 가져다 쓸 거는 억지로 _ 처럼 뭔가를 넣어야 한다.
```js
const useSomeHooks = (bool) => {
  return [0,1,2,3 state, setState];
}

function ReturnCustomHooks() {
  const [firstValue, secondeValue, _, thirdValue, _, _] = useSomeHooks(true);
}
```

- 객체를 비구조할당해서 customHook으로 만드는 게 더 유지보수에 좋다.
```js
const useSomeHooks = (bool) => {
  return {
    first,
    second, 
    third,
    rest
  }
}

function ReturnCustomHooks() {
  const {first: firstValue, second: secondeValue, rest: thirdValue} = useSomeHooks(true);
}
```

- 비구조 할당은 여러모로 도움이 된다.
- ES6 이전의 문법이다.
```js
const query =useQuery({ queryKey: ['hello'], queryFn: getHello});
const query2 =useQuery({ queryKey: ['hello2'], queryFn: getHello});

const data = query.data;
const refetch = query.refetch;
const isSuccess = query.isSuccess;
```

- ES6에서는 비구조할당할 수 있다.
```js
const {data, refetch, isSuccess} =useQuery({ queryKey: ['hello'], queryFn: getHello});
const query2 =useQuery({ queryKey: ['hello2'], queryFn: getHello});

const hello2Data = query.data;
const hello2Refetch = query.refetch;
const hello2IsSuccess = query.isSuccess;
console.log(data);
console.log(refetch);
console.log(isSuccess);
```

- 비구조할당을 하면 변수명도 바꾸는 게 가능하다.
```js
const {
  data: helloData, 
  refetch: helloRefetch, 
  isSuccess: helloIsSuccess
} = useQuery({ queryKey: ['hello'], queryFn: getHello});
const query2 =useQuery({ queryKey: ['hello2'], queryFn: getHello});

const hello2Data = query.data;
const hello2Refetch = query.refetch;
const hello2IsSuccess = query.isSuccess;
console.log(helloData);
console.log(helloRefetch);
console.log(helloIsSuccess);
```

## <span style="color:#802548">_15.useEffect 내에서 비동기는 쓰지 말자_</span>
- useEffect보다는 사실 tanStackQuery를 쓰자.
- useEffect 콜백에는 async를 달 수 없다.
- useEffect는 promise를 return하지 않고 함수나 undefined만 return해야 한다.
```js
useEffect(async () => {
  //비동기
  const result = await fetchData();
}, []);
```

- 번거로워도 한단계 거치는 방법으로 바꿔야 한다.
```js
useEffect( () => {
  //비동기
  const fetchData = async () => {
    const result = await someFetch();
  }
  
  fetchData();
}, []);
```

## <span style="color:#802548">_16.import React 필요없다_</span>
- v17 이전엔 import 구문이 필요했다.
- 아래 jsx가 React.createElement로 transpile 됐기 때문이다.
```js
function App() {
  return <h1>Hello World</h1>;
}

function App() {
  return React.createElement('h1',null,'Hello world');
}
```

- 하지만 v17 부터는 import 구문이 필요없다.
- 다만, class형 component는 필요하다.
```js
import {Component} from 'react';

class Welcome extends Component {
  render() {
    return <h1>Hello</h1>
  }
}
```


## <span style="color:#802548">_16.folder structure_</span>
- 결합도가 높은 component 이름은 굳이 preFix를 다르게 하지 않는다.
```
components/
TodoList.vue
TodoItem.vue
TodoButton.vue

components/
SearchSiderbar.vue
NavigationForSearchSidebard.vue
```

- 결합되어있는 component이름을 그대로 prefix로 써준다.
```
components/
TodoList.vue
TodoListItem.vue
TodoListButton.vue

components/
SearchSiderbar.vue
SearchSiderbarNavigation.vue
```

- 연관된 component라면 prefix를 앞에 달아주자.
- 폴더구로조만 depth를 늘리는 것은 바람직하지 않다.
```
components/
ClearSearchButton.vue
ExcludeFromSearchInput.vue
LaunchOnStartupCheckbox.vue
RunSearchButton.vue
SearchInput.vue
TermsCheckbox.vue
```

- 연관된 component면 one depth로 표현하고 component naming을 잘 만들자.
```
components/
SearchButtonClear.vue
SearchBUttonRun.vue
SearchButtonQuery.vue
SearchInputExcludeGlob.vue
SettingsCheckboxTerms.vue
SettingsCheckboxLaunchOnStartup.vue
```

- 굳이 js라 folder 나누고, jsx라 또 나누고, css라 또 나누지 말자.
- 업무 단위로 폴더를 만들고 거기다가 모두 박아도 된다.
```
Componentns
 ㄴ @shared
  ㄴ 공통 컴포넌트
 ㄴ Todo
  ㄴ Todo.jsx
  ㄴ Todo.hook.js
  ㄴ Todo.css
```


- 아니면 hooks나 style이나 components로 나눌 수도 있다.
```
Hooks/
ㄴ useTodo.js
Style/
ㄴ Todo.css
Components/
ㄴ Todo.jsx
```

## <span style="color:#802548">_17.새로고침 자제하자_</span>
- reload를 쓰는 것은 SPA에는 위험한 발상이다.
```js
const [isLoggedIn, setIsLoggedIn] = useState(false);

const handleLogin = async () => {
  try {
    if (isSuccess === true)  {
      setIsLoggedIn(true);
      //SPA 입장에서는 앱을 완전히 종료하고 다시 실행하는 행위다.
      //reload하면 앱이 온전하지 않은 상태일 수 있다.
      window.location.reload();
    }
    
  } catch (error) {
    alert('로그인에 실패했습니다.');    
  }
}
```

- state, rouer 등이 모두 날라간다. SSR이 아니라 CSR이기에 js에서 전부 rendering을 담당하고 있기 때문이다. reload는 쓰면 안 된다.
```js
const [isLoggedIn, setIsLoggedIn] = useState(false);

const handleLogin = async () => {
  try {
    if (isSuccess === true)  {
      setIsLoggedIn(true);
      //SPA 입장에서는 앱을 완전히 종료하고 다시 실행하는 행위다.
      //reload하면 앱이 온전하지 않은 상태일 수 있다.
      window.location.reload();
    }
    
  } catch (error) {
    alert('로그인에 실패했습니다.');    
  }
}
```

## <span style="color:#802548">_18.domain과 무관하게 component naming_</span>
- 확장성 있게 component를 쓰려면 domain name이 아니라 UI를 묘사하자.
```
TodoList
TodoItem
```

- radix UI를 보고 만들면 naming하면 좋다.
```
CardList
List
Item
AlbutList
Button
```

- div, span으로 떡칠하는 건 최소화하자.
- semantic을 지켜서 HTML을 만들자.
- 그럴 때 interface와 React에서 제공하는 HTMLAttribute가 도움된다.
```js
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {

}

const Button = (props: ButtonProps) => {
  return (
    <button {...props}>
      {children}
    </button>
  )
}
```