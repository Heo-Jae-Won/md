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