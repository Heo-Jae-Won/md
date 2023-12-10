```javascript
let ab; //type을 설정 안하면 any type.
ab = 'string'; 

let ab : string;
ab = 'string'; //type 설정 시 string type이 됨.
```


```javascript
function printName(person: { first: string; last: string }): void {
  console.log(`${person.first} ${person.last}`);
}
```
- 함수 parameter에 객체 리터럴로 직접 넣을 경우 해당 key만 갖고있는 것으로 넘겨야 한다.

```javascript
printName({first: 'key', last: 'value', age: '20'}) //error. 객체 literal 직접 넣으므로 해당 key만 딱 넣어야..

const obj = {
    first: 'key',
    last: 'value',
    age: '20'
}
printNam(obj); //ok. 추가 property가 있어도 문제 없음.
```

```javascript
const colors = ["red","orange",'yellow'];
colors.map ((color : string) => {
    return color.toFixed() //error.
})

colors.map (color => {
    return color.toFixed() //error. parameter에 type 안 지정해도 맥락으로 알아먹음. colors가 string 배열이기 때문.
})
```

```javascript
function printTwice(msg: string): void {
  console.log(msg);
  console.log(msg);//return undefined가 바로 void
}

function gameLoop(): never {
  while (true) {
    console.log("GAME LOOP RUNNING!");
  } //return할 기회 조차 없는 게 바로 never
}

function throwError() : never {
    throw new Error("error occured"); //throw error면 return이 불가능. never.
}
```

```javascript
let coordinate: {x: number; y:number} = { x:34, y:2};

function randomCoordinate(): {x: number; y:number} {
    return { x: Math.random(), y:Math.random()};
}
```

```javascript
type Point = {
    x: number;
    y: number;
}

let coordinate: Point = {x:34, y: 2};
function randomCoordinate(): Point {
    return { x: Math.random(), y:Math.random()};
}
```

```javascript
function calculatePayout(song: {title: string, artist: string, 
                                numStreams: number, credits: {producer: string, writer: string}}) {

}

{
    title: 'Unchained Melody',
    artist: 'Righteous Brothers',
    numStreams: 12873321,
    credits: {
        producer: 'Phil Spector',
        wirter: "Alex North"
    }
}
```

```javascript
type Song = {
    title: string,
    artist: string,
    numStreams: number,
    credits: {
                producer: string,
                writer: string
            }
}

function calculatePayout(song: Song) : number {

    return song.numStreams * 0.0033;
}

const mySong = {
    title: 'Unchained Melody',
    artist: 'Righteous Brothers',
    numStreams: 12873321,
    credits: {
        producer: 'Phil Spector',
        wirter: "Alex North"
    }
}

calculatePayout(mySong);
```

```javascript
type Circle = {
    radius: number;
}

type Colorful = {
    color: string;
}

type ColorfulCircle = Circle & colorful;

const happyFace: ColorfulCircle = {
    radius: 4,
    color: 'yellow'
}
```

```javascript
const dune: Movie = {
  title: "Dune",
  originalTitle: "Dune Part One",
  director: "Denis Villeneuve",
  releaseYear: 2021,
  boxOffice: {
    budget: 165000000,
    grossUS: 108327830,
    grossWorldwide: 400671789,
  },
};

const cats: Movie = {
  title: "Cats",
  director: "Tom Hooper",
  releaseYear: 2019,
  boxOffice: {
    budget: 95000000,
    grossUS: 27166770,
    grossWorldwide: 73833348,
  },
};

function getProfit(movie: Movie): number {
    const {grossWorldwide, budget} = movie.boxOffice;
  return grossWorldwide - budget;
}

function getProfit({ boxOffice: { grossWorldwide, budget } }: Movie): number {
  return grossWorldwide - budget;
}
```

```javascript
const activeUsers: [] = []; //이러면 무조건 빈배열만 가능하게 됨.
activeUser.push('string') //error. 빈배열이 아니기 때문.

const activeUsers = []; // any[]와 같음. 아무거나 다 들어갈 수 있음.
const activeUsers: string[] = []; // string만 들어갈 수 있음.

const bools: Array<boolean> = []; 
const bools: boolean[] = [];
```

```javascript
const board: string[][] = [["x",'o','x'], ['x','o','x'],['x','o','x']]
const demo: number[][][] = [[[1]]]
```

```javascript
const ages: number[] = [];
const gameBoard: string[][] = [];

type Product = {
    name: string,
    price: number
}

function getTotal(array : Product[]) : number {
    const sum = array.reduce((acc,cur) => acc + cur.price, 0)
    return sum;
}
```

```javascript
function calculateTax(price: number | string, tax: number) {
    price.replace("$", ""); //error. price가 string일 수도 있어서..
}

function calculateTax(price: number | string, tax: number) {
    if (typeof price === 'string') {
        price = parseFloat(price.replace("$", "")); // price를 parseFloat해주면 price는 number가 됨.
    } 

    return price * tax // number라서 *가능!
}

const stuff: (number | string)[] = [1,2,3,"das"]; //number나 string을 받는 배열
const stuff: number[] | string[] = [1,2,3,"das"]; //error. number로 쭉 배열 혹은 string으로 쭉 배열로 담아야함.
```

```javascript
let zero: 0 = 0; //literal type이 annotation임.
zero = 4;//error. 0만 가능.
const mood: "Happy" | "Sad" = "Happy";
mood = "Sad"; //error. Happy나 sad만 가능.

type DayOfWeek= "Monday" | "TuesDay" | "Wednesday" | 0;
let today: DayOfWeek  = "Weds" ;//error. Wednesday라고 써야함.
```


- 거의 쓸일은 없지만, Tuple도 있다.
- Tuple은 array인데, index마다 들어오는 type과 length가 정해져 있다.
```javascript
let myTuple: [number, string]; //[0]은 무조건 number type. [1]은 string type. length는 2 고정.
myTuple = [1,5]; //error. [1]이 string이여야 함.
myTuple = [1,'5',5];//error. [2]는 length를 초과했음. 
const color: [number,number,number] = [255,0,255];
```

```javascript
type HttpResponse = [number, string]; //tuple type.

const goodRes: HttpResponse = ["OK", 200]; //error. 순서를 지켜서 넣어야 함.
goodRes[0] = '200'; //error. [0]은 무조건 number임.
const goodRes: HttpResponse = [200, "OK"];

goodRes.push(123); //정상 작동. length가 3이 되는데도 정상임. tuple의 한계.
goodRes.pop(); //정상 작동. length가 1이 되는데도 정상임. tuple의 한계.
```

```javascript
const PENDING = 0;
const SHIPPED = 1;
const DELIVERED = 2;

if (status === DELIVERED) {

}


enum OrderStatus {
    PENDING, //0
    SHIPPED, //1
    DELIVERED,//2
    RETURNED //3
}

enum OrderStatus {
    PENDING = 10,
    SHIPPED, //11
    DELIVERED,//12
    RETURNED //13
}

enum OrderStatus {
    PENDING   = 10,
    SHIPPED   = 41,
    DELIVERED = 153,
    RETURNED  = 524
}

enum ArrowKeys {
    UP = "up",
    DOWN = "down",
    LEFT = "left",
    RIGHT = "right",
    ERROR = 500
}// 보통은 같은 type으로 다 맞춰주는 게 좋다.

const myStatus = OrderStatus.DELIVERED;
function isDelivered(status: OrderStatus) {

    return status === OrderStatus.DELIVERED
}

isDelivered(OrderStatus.RETURNED);
```

- enum은 ts에서 compile 되고 나서 사라지지 않고 남는다. js에 영향을 미친다.
- 그래서 그냥은 잘 쓰이지 않는다. const를 붙여서 사용한다.
- 그러면 eunm을 사용하면서 자동완성의 이점도 누리고, js에도 영향을 미치지 않을 수 있다.
```javascript
const enum ArrowKeys {
    UP = "up",
    DOWN = "down",
    LEFT = "left",
    RIGHT = "right",
    ERROR = 500
}

const ArrowKeys = {
    UP = "up",
    DOWN = "down",
    LEFT = "left",
    RIGHT = "right",
    ERROR = 500
} as const
```

- type
```javascript
interface Person {
    readonly id: number;
    first: string;
    last: string;
    nickname?: string;
    sayHi: () => string; //sayHi(): string과 동일
}

const thomas: Person = {
    first: "h",
    last: 'd',
    id: 2154,
    sayHi: () => {
        return 5//error. string return해야.
    }
}
```

```javascript
interface Product {
    name : string,
    price: number,
    applyDiscount(discount/*discount parameter에 type을 지정하지 않아 any type */);
    applyDiscount(discount: number) : number; //아래와 동일
    applyDiscount: (discount: number) => number;
}

const shoes: Product = {
    name: 'f',
    price: 10,
    applyDiscount(amount: number/*string이라고 하면 error */) {
        const newPrice = this.price * ( 1 - amount);
        this.price = newPrice;

        return this.price;
    }
}
```

```javascript
interface Dog {
    name: string
    age: number
}

interface Dog {
    breed: string,
    bark: () => string
}
//Dog의 속성을 두 번 나눠 기입해도 똑같은 interface의 속성으로 취급. 따라서 4개의 속성이 필요함.

//DOg의 속성이 모두 필요하고 추가로 기입한 속성도 필요. 다중 상속 가능!
interface ServiceDog extends Dog {
    job : "drug sniffer"
}

const chewy: ServiceDog = {
    name: "Chewy",
    age: 4.5
    breed: 'Lab',
    bark() {
        return "bark!"
    },
    job: "drug sniffer"
}
```

- interface는 객체의 형태만을 묘사. type은 모든 형태를 묘사할 수 있다.
```javascript
type Chicken = "chicken" | "turkey";
```
- type은 향후에 추가를 할수가 없다. interface는 가능하다. extends도 가능하다.
```javascript
type Chicken = {
    breed: string
}

type Chicken = {
    numEggs: number
}
```


- typescript에선 document에 Document라는 type이 있다.
- 그 type을 인지하는 옵션은 ts.config의 lib이다.
- lib에 dom이 있어야 한다.
- 또한 타입 정의 파일인 ~~d.ts도 있어야 한다. 그런데 해당 타입 정의 파일은 target이 된 버전을 사용한다.
- 즉 es5의 type 정의를 사용한다는 의미다. 따라서 ES2021에 있는 replaceAll 같은 method는 사용불가능하다.
- Web Worker도 쓰고 싶다면 lib에 넣어야 한다.
```javascript
"lib": ["esnext", "dom", "dom.iterable", "scripthost"]
```

```javascript
const btn = document.getElementById("btn")!; //non-null. 대신 이거 엔간하면 쓰지 말자.
btn.addEventListener();

const btn = document.getElementById("btn"); //optional chaining으로 가자.
btn?.addEventListener();
```

- type assertion은 보통 DOM조작에 쓴다.
```javascript
let mystery: unknown = 'Hello World';
const numChars = mystery as string.length; //이렇겐 안됨 괄호로 묶어야함.
const numChars = (mystery as string).length;


const btn = document.getElementById("btn")!;
const input = document.getElementById("todoinput");

btn.addEventListener('click',function() {
    alert(input.value); //input does not have a proerty input. 
})

const input = document.getElementById("todoinput")! as HTMLInputElement; //value property는 HTMLInputElement에만 있음.
btn.addEventListener('click',function() {
    alert(input.value); //input does not have a proerty input. 
    input.value = "";
})

const input = document.getElementById("todoinput");
btn.addEventListener('click',function() {
    alert(input.value); //input does not have a proerty input. 
    (<HTMLInputElement>input).value = ""; //JSX에선 작동안함. React에선 불가.
        //(input).value와 동일.
})
``` 

```javascript
const form = document.querySelector('form'); //아이디나 class보다는 안 겹친다면 element명으로 ㄱㄱ
//그러면 ts가 알아서 추론해서 type 추론이 더 좋음. #id는 any type이기 때문에 안좋음.

form.addEventListener("submit",function(e){//e가 뭘 의미하는 지 암. submit에 addEventLister기 때문.
    e.preventDefault();
    console.log("SUBMITTED");
})

const handleSubmit = function(e){ //e가 뭘의미하는 지 ts가 모름. context가 없기 때문. error
//e: subMitEvent()로 써줘야 함.
    e.preventDefault();
    console.log("SUBMITTED");
}
form.addEventListener("submit",handleSubmit); 
```

- localStorage는 문자열만 갖고 있다. 그래서 객체를 넣을 수가 없다.
```javascript
localStorage.setItem('todos', JSON.Stringify(toods));
```

- localStorage에서 가져온 건 union type이다. type narrowing이 필요하다.

```javascript
function readTodos() {
    const todosJSON = localSTorage.getItem("todos");
    if (todosJSON === null ) { //type narrowing
        return [];
    }

    return JSON.parse(todosJSON);
}
```

- ts에서는 property를 먼저 기술해야만 this로 사용이 가능하다.
```javascript
class Player {
    first: string;
    last: string;
    constructor(first: string, last: string) {
        this.first = first;
        this.last = last;
    }
}
```

- 초기화 값이 정해져있다면 아래와 같이 할 수도 있다.
```javascript
class Player {
    readonly first: string;
    last: string;
    score = 0; //자동 추론
    constructor(first: string, last: string) {
        this.first = first;
        this.last = last;
    }
}
```

- ts에는 public, private같은 keyword 선언도 가능하다.
```javascript
class Player {
    readonly first: string;
    last: string;
    public score = 0; //public은 붙이지 않아도 기본값이 public이지만, 더 알아보기 쉽게 붙인 것.
    constructor(first: string, last: string) {
        this.first = first;
        this.last = last;
    }
}

class Player {
    readonly first: string;
    last: string;
    private score = 0;
    constructor(first: string, last: string) {
        this.first = first;
        this.last = last;
    }
}

const elton = new Player("Elton", "Steele");
elton.score = 10; //private을 쓰면 property의 이름이 바뀌지 않는다. 물론 접근 불가다.

class Player {
    readonly first: string;
    last: string;
    #score = 0; //자동 추론
    constructor(first: string, last: string) {
        this.first = first;
        this.last = last;
    }
}
elton.#score = 10; //#을 써도 private과 같지만, property의 이름이 바뀐다. 물론 접근 불가다.
```

- 아래 class의 instance method를 class 밖에서 쓰면 오류가 난다.
- 하지만 무시하고 compile해도 돌아간다. 왜냐하면 keyword private이 지워지면 저 함수 자체는 구동가능한 게 맞기 때문이다.
```javascript
private secretMethod() {
    console.log("operated!!!");
}
```

```javascript
interface Colorful {
    color: string;
}

class Bike implements Colorful {
    constructor(public color: string) {}
}

class Jacket implements Colorful {
    constructor(public brand: string, public color: string) {}
}

const bike1 = new Bike("red");
const jacket1 = new Jacket("prada", "black")
```


```javascript
interface Colorful {
    color: string;
}

interface Printable {
    print(): void;
}
class Bike implements Colorful {
    constructor(public color: string) {}
}

class Jacket implements Colorful {
    constructor(public brand: string, public color: string) {}

    print() {
        console.log(`${this.brand}, ${this.color}`);
    }
}

const bike1 = new Bike("red");
const jacket1 = new Jacket("prada", "black")
```

