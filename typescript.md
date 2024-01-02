- 선언만 한 경우에도 type을 설정할 수 있다.
```javascript
let ab; //type을 설정 안하면 any type.
ab = 'string'; 

let ab : string;
ab = 'string'; //type 설정 시 string type이 됨.
```


- 함수 parameter에 객체 리터럴로 직접 넣을 경우 해당 key만 갖고있는 것으로 넘겨야 한다.
```javascript
function printName(person: { first: string; last: string }): void {
  console.log(`${person.first} ${person.last}`);
}
printName({first: 'key', last: 'value', age: '20'}) //error. 객체 literal 직접 넣으므로 해당 key만 딱 넣어야..

const obj = {
    first: 'key',
    last: 'value',
    age: '20'
}
printNam(obj); //ok. 추가 property가 있어도 문제 없음. 변수로 넣으면 ㄱㅊ
```


- typescript는 맥락을 알아챈다. 자동 추론을 적극 활용하자.
```javascript
const colors = ["red","orange",'yellow'];
colors.map ((color : string) => {
    return color.toFixed() //error.
})

colors.map (color => {
    return color.toFixed() //error. parameter에 type 안 지정해도 맥락으로 알아먹음. colors가 string 배열이기 때문.
})
```


- void는 return undefined고, never는 return 할 기회가 없는 상황이다.
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

- 객체 literal을 type으로 넣기보다 interface를 활용하자.
```javascript
let coordinate: {x: number; y:number} = { x:34, y:2};

function randomCoordinate(): {x: number; y:number} {
    return { x: Math.random(), y:Math.random()};
}
type Point = {
    x: number;
    y: number;
}

let coordinate: Point = {x:34, y: 2};
function randomCoordinate(): Point {
    return { x: Math.random(), y:Math.random()};
}
```

- 중첩 객체를 넣고 싶다면 type을 활용하면 더 직관적일 때도 있다.
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

- type을 2개를 겹쳐서 두 type의 property를 모두 갖는 새로운 type을 만들 수 있다.
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

- 중첩 객체를 가져올 때 destructuring을 쓴다.
- typescript로 destructuring을 쓸 때는 아래와 같이 활용하면 된다.
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

- []를 지정할 때는 type[]과 같은 형식으로 지정해주어야 한다.
```javascript
const activeUsers: [] = []; //이러면 무조건 빈배열만 가능하게 됨.
activeUser.push('string') //error. 빈배열이 아니기 때문.

const activeUsers = []; // any[]와 같음. 아무거나 다 들어갈 수 있음.
const activeUsers: string[] = []; // string만 들어갈 수 있음.

const bools: Array<boolean> = []; 
const bools: boolean[] = [];
```

- 2차원 배열도 type으로 줄 수 있다.
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

- 여러 type을 모두 받는 mixture type도 가능하다.
- mixture type은 union type과는 다르다.
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

- literal도 그 자체로 type으로 줄 수 있다.
- 잘하면 enum처럼 활용이 가능하다.
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

- tuple은 순서를 지켜 넣어야 한다.
```javascript
type HttpResponse = [number, string]; //tuple type.

const goodRes: HttpResponse = ["OK", 200]; //error. 순서를 지켜서 넣어야 함.
goodRes[0] = '200'; //error. [0]은 무조건 number임.
const goodRes: HttpResponse = [200, "OK"];

goodRes.push(123); //정상 작동. length가 3이 되는데도 정상임. tuple의 한계.
goodRes.pop(); //정상 작동. length가 1이 되는데도 정상임. tuple의 한계.
```

- const 변수를 선언하는 것보다 enum을 선언하는 게 더 쓰기 좋다.
- enum은 자동완성을 지원해준다.
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

- method도 interface의 property로 가질 수 있다.
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

- concise method로 쓸 수 있다.
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

- interface도 extends를 쓸 수 있다.
- extends를 쓰면 해당 interface의 property를 모두 갖게 된다. 따라서 둘다 필요하게 된다.
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

- union type에서 type narrowing을 하는 방법으로 non-null assertion도 있다.
- 다른 방법으론 optional chaining도 있다.
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

- querySelector로 할 때는 id나 class보다는 element를 주면 좋다.
- interface를 ts가 알아서 추론해주어 그 interface의 method를 쓸 수 있다.
- id나 class를 쓰면 any type이라서 generics로 덮어 써야 method를 쓸 수 있다.
```javascript
const form = document.querySelector('form'); 
const form = document.querySelector<HTMLFormElement>('#form');

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
- 그래서 stringify로 바꿔서 넣고, parse로 꺼내 쓴다.
```javascript
localStorage.setItem('todos', JSON.Stringify(toods));
```

- 하지만 localStorage에서 가져온 건 null도 되는 union type이다. type narrowing이 필요하다.
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

- class에서 interface를 쓰려면 extends가 아닌 implements로 써야한다.
- 아래와 같이 축약 문법으로 쓸 수도 있다.
```javascript
interface Colorful {
    color: string;
}

class Bike implements Colorful {
    color: string;
    constructor(color: string) {
        this.color = color;
    }
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

- implements는 다중으로 가능하다.
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

class Jacket implements Colorful, Printable {
    constructor(public brand: string, public color: string) {}

    print() {
        console.log(`${this.brand}, ${this.color}`);
    }
}

const bike1 = new Bike("red");
const jacket1 = new Jacket("prada", "black")
```

- method에는 generics로 return type을 명시해야 할 수도 있다.
```javascript
const nums: number[] = [];
const nums: Array<number> = [];

const inputEl = document.querySelector("#username");
console.log(inputEl);
inputEl.value = 'Hacked!' ; //error. querySelector로 가져온 것은 그저 Element고 value라는 property가 없음.

//그럴 때 어떤 type이 method에서 반환되는 지 generics로 알려줌.
const inputEl = document.querySelector<HTMLInputElement>("#username"); //<input id="username">
inputEl.value = 'Hacked!';

const btn = document.querySelector<HTMLButtonElement>(".btn"); //<button class="btn">
```


- generics를 만들면 여러 type에 대해 각자 overloading할 필요가 없어 편하다.
```javascript
function numberIdentity(item: number): number {
    return item;
}

function stringIdentity(item: string): string {
    return item;
}

function booleanIdentity(item: boolean): boolean {
    return item;
}

function Identity(item: any): any {
    return item;
}

function identity<Type>(item: Type): Type {
    return item;
}

identity<boolean>(6) // error. true나 false가 들어와야함.
identity<string>
identity<number>
identity<any>
identity<Cat>([]) //Cat에 맞는 형태가 와야함.


function identity<T>(item: T): T {
    return item;
}
```

- 배열도 똑같이 generics를 쓸 수 있다.
- 자동추론이 가능하면 그렇게 맡겨주자. 그게 code가 더 깔끔하다.
```javascript
function getRandomNumberElement(list: number[]) {

}

function getRandomStringElement(list: string[]) {

}

function getRandomBooleanElement(list: boolean[]) {

}

function getRandomNCatElement(list: Cat[]) {

}

function getRandomNAnyElement(list: any[]) {

}


function getRandomTypeElement<T>(list: T[]) : T {
    const randIdx = Math.fllor(Math.random() * list.length);

    return list[randIdx];
}

console.log(getRandomElement<string>(["a","b","c"]));
console.log(getRandomElement<number>([1,2,3,4,5,6]));

console.log(getRandomElement(["a","b","c"])); //확실하면 <>로 type 알려주지 않아도 됨. ts가 추론 가능
console.log(getRandomElement([1,2,3,4,5,6])); //확실하면 <>로 type 알려주지 않아도 됨. ts가 추론 가능
```


- tsx에서 화살표함수로 generics를 쓰면 무조건 ,를 붙여줘야 한다.
- tsx에서만 이런 현상이 일어난다.
```javascript
function getRandomTypeElement<T>(list: T[]) : T {
    const randIdx = Math.fllor(Math.random() * list.length);

    return list[randIdx];
}

//Tsx파일에서 화살표 함수로 generics를 쓸 떄는 Type뒤에 무조건 ,를 붙여줘야 한다.
//아니면 html로 생각해서 오류가 난다.
const getRandomTypeElement = <T,>(list: T[]) T => {
    const randIdx = Math.fllor(Math.random() * list.length);

    return list[randIdx];
}
```


- 2번쨰 인수부터는 T가 아니라 U로 쓴다. 
- U 다음은 V로 쓴다.
```javascript
function merge(object1, object2) {
    return {
        ...object1,
        ...object2
    }
}

const comboObj = merge({name: 'colt'}, {pets: ['blue', 'elton']});

function merge<T, U>(object1: T, object2: U) { // T 다음은 U, U 다음은 V
    return {
        ...object1,
        ...object2
    }
}
merge<{name: string}, {pets:string[]}>({name: 'colt'}, {pets: ["blue", "elton"]}) //이렇게 할 필요 없음.
const comboObj = merge({name: 'colt'}, {pets: ["blue", "elton"]}); //자동추론 가능하니 이렇게 쓰면 됨.
```

- generics type을 지정할 때도 extends를 사용할 수 있다.
```javascript
function merge<T extends object, U extends object>(object1: T, object2: U) {}
merge({name: 'Colt'}, 9); //error. object가 들어와야함.
merge({name: 'Colt'}, {num: 9});  //객체로 만드니까 잘 됨.
```

- 특정 속성이 있는 지 확인하려면 그냥 generics가 아니라 extends가 필요하다.
```javascript
function printDoubleLength<T>(thing: T): number {
    return thing.length * 2; //error. 그냥 T는 length라는 properties가 없음.
}

//length라는 property를 가지게끔 interface를 만듦.
interface length {
    length: number;
}

function printDoubleLength<T extends Lengthy>(thing: T): number {
    return thing.length * 2;
}
printDoubleLength('ABCD'); // 4
printDoubleLength(1234) //error. T로 들어오는 type에는 length 속성이 있어야 하기 때문.


function printDoubleLength(thing: Lengthy): number {
    return thing.length * 2;
} //이것도 가능함. 
```

- generics가 있는데도 주지 않으면 unknown type이 된다.
- 하지만 default값을 주면 해당 default type이 된다.
```javascript
function makeEmptyList<T>(): T[] {
    return [];
}

const strings =  makeEmptyList<string>();
strings.push('sth');

const strings =  makeEmptyList(); //strings는 unknown type이 된다.

function makeEmptyList<T = number>(): T[] {
    return [];
}

const nums = makeEmptyList(); //nums는 number type이다. unknown type이 아니다. T는 number로 기본값을 주었기 때문이다.
const strings = makeEmptyList<string>(); //type 값을 제공하면 default값은 무시된다.
```

- generics를 통해 해당 type으로 만든 객체는 해당 type만 받게끔 만들 수도 있다.
```javascript
interface Song {
    title: string;
    artist: string;
}

interface Video {
    title: string;
    creator: string;
    resolution: string;
}

class PlayList<T> {
    public queue: T[] = [];
    add(el: T) {
        this.queue.push(el);
    }
}

const songs = new PlayList<Song>();
const videos = new PlayList<Video>();
videos.add(); //video type만 넣을 수 있음.
songs.add(); //Song type만 넣을 수 있음.
```

- typeof로 type narrowing이 가능하다.
```javascript
function triple(value: number | string) {
    if(typeof value === 'string') {
        return value.repeat(3); //string인지 확인 안하면 ts는 이게 어떤 type인지 몰라 오류 일으킴.
        //그래서 typeof guard를 쓴 것.
    }

    return value * 3; //typeof로 string을 걸렀기 때문에 여기선 무조건 number type으로 확정되는 것을 ts가 암. 따라서 * 연산자에도 오류가 안 남.
}
```

- truthy로 type narrowing이 가능하다.
```javascript
const el = document.getElementById("idk");
if (el) {
    el //el이 truthy기 때문에 절대 null이 아님. 따라서 HTMLElment type이 됨. HTMLElement | null이라는 union type이 아님.
}

const printLetters = (word: string | null) => {
    for(let char of word) { // word가 null일 수 있어 error가 나옴.
        console.log(char);
    }
}

const printLetters = (word?: string) => {
    for(let char of word) { // word가 undefined일 수 있어 error가 나옴.
        console.log(char);
    }
}

const printLetters = (word?: string) => {
    if(word) {
        for(let char of word) { //type narrowing을 통과했으므로 word는 그냥 string type임.
            console.log(char);
        }
    } else {
        //여기서는 word가 빈값, 즉 falsy일 수 있기 때문에 word가 다시 string | undefined union type이 된다.
        console.log("YOU DID NOT PASS IN A WORD");
    }
    
}
```

- 객체의 property가 있는지 검사하려면 in을 사용하자.
```javascript
function someDemo(x: string | number, y: string | boolean) {
    if (x===y) {
        x.toUpperCase();
    }
}

interface Movie {
    title : string,
    duration: number,
}

interface TVShow {
    title: string,
    numEpisodes: number,
    episodeDuration: number
}
function getRuntime(media: Movie | TVShow) {
    media.numEpisodes //error. numEpisodes가 Movie에는 없어서 type narrowing 필요.
}

function getRuntime(media: Movie | TVShow) {
    if ("numEpisodes" in media) {
        return media.numEpisodes * media.episodeDuration   //numEpisodes는 TVShow에만 존재.
        //type narrowing 수행한 것.
    }

    return media.duration; //if문에서 TVShow가 걸러졌기에 여기는 반드시 Movie type. 
    //따라서 duration을 오류없이 그냥 가져오는 게 가능함. ts가 추론해줌.
}

getRuntime({title: "Amadeus", duration: 140})
getRuntime({title: "Spongebob", numEpisodes: 80, episodeDuration: 30})
```

- class, built in object에는 instanceof를 쓰자.
```javascript
function printFullDate(date: string | Date) {
    if (date instanceof Date) {
        console.log(date.toUTCString()); //Date면 toUTCString() 사용 가능
    } else {
        console.log(new Date(date).toUTCString()); //Date가 아니면 string이기 때문에 Date로 만들어줘야함.
    }
}


class User {
    constructor(public username: string) {}
}
class Company {
    constructor(public name: string) {}
}

function printName(entity: User | Company) {
    if (entity instanceof User) {
        entity //User class의 instance
    } else {
        entity //Company class의 instance
    }
}
```

- type predicate function을 만들면 만능이다.
```javascript
interface Cat {
    name: string,
    numLives: number
}

interface Dog {
    name: string;
    breed: string;
}


function isCat(animal: Cat | Dog): animal is Cat { 
    //parameter가 animal이기 때문에 : 뒤의 단어도 animal
    return (animal as Cat).numLives !== undefined //무조건 return하는 값은 boolean type
}

function isCat(star: Cat | Dog): star is Cat { 
    //parameter가 star기 때문에 : 뒤의 단어도 star
    return (star as Cat).numLives !== undefined //무조건 return하는 값은 boolean type
}

function makeNoise(animal: Cat | Dog): string {
    if (isCat(animal)) {
        animal // 위에서 무조건 animal은 Cat이라고 했으므로 Cat임. animal이 뭐가 오든 Cat.
        return "Meow";
    } else {
        animal //여기는 무조건 Dog
    }
}

function isCat(animal: Cat | Dog){
    return (animal as Cat).numLives !== undefined
}

function makeNoise(animal: Cat | Dog): string {
    if (isCat(animal)) {
        animal // 위에서 무조건 animal은 Cat이라고 했으므로 Cat임. animal이 뭐가 오든 Cat.
        return "Meow";
    } else {
        animal //여기는 무조건 Dog
    }
}

function isRequired<T>(value: T | undefined | null): value is T {
  return value !== undefined && value !== null;
}

function isString(value: unknown): value is string {
  return typeof value === 'string';
}

if (isString(value)) {
  //value의 type은 string으로 narrowing됩니다
  console.log(value.toUpperCase()); // value는 string이기 때문에 toUpperCase()를 호출가능
}
```

- 다 똑같은 property를 interface가 공유한다면?
```javascript
interface Rooster {
    name: string;
    weight: number;
    age: number;
}

interface Cow {
    name: string;
    weight: number;
    age: number; 
}

interface Pig {
    name: string;
    weight: number;
    age: number;  
}
```

- 그럼 아래와 같이 분별을 위한 property를 추가해준다.
```javascript
interface Rooster {
    name: string;
    weight: number;
    age: number;
    kind: "rooster" //name과 weight와 age가 모두 똑같아 추가된 discriminate(분별)을 위해 추가한 type
}

interface Cow {
    name: string;
    weight: number;
    age: number; 
    kind: "cow"   //name과 weight와 age가 모두 똑같아 추가된 discriminate(분별)을 위해 추가한 type
}

interface Pig {
    name: string;
    weight: number;
    age: number;  
    kind: "pig"   //name과 weight와 age가 모두 똑같아 추가된 discriminate(분별)을 위해 추가한 type
}
```

- 그리고 이를 switch로 꺼내 type narrowing을 한다.
```javascript
type FarmAnimal = Pig | Rooster | Cow;  
function getFarmAnimalSound(animal: FarmAnimal) {
    switch(animal.kind) { //분별을 위한 property를 통해 type narrowing
        case "pig":
            return "Oink!";
        case "cow":
            return "Mooo!";
        case "rooster":
            return "Cockadoodledoo!";
    }
}

    
const stevie: Rooster = {
    name: "Stevie Chicks",
    weight: 2,
    age: 1.5,
    kind: "rooster"
}

console.log(getFarmAnimalSound(stevie)); //Cockadoodledoo!
```

- 새로운 interface를 추가하고 FarmAnimal에 넣었다고 해보자. 그럼 아래에서는 case가 없어서 return을 하는 게 없다.
```javascript
interface Sheep {
    name: string;
    weight: number;
    age: number;  
    kind: "sheep"
}
type FarmAnimal = Pig | Rooster | Cow | Sheep;  
function getFarmAnimalSound(animal: FarmAnimal) {
    switch(animal.kind) { //sheep case가 없음.
        case "pig":
            return "Oink!";
        case "cow":
            return "Mooo!";
        case "rooster":
            return "Cockadoodledoo!";
    }
}
```

- 그럴 때 자동으로 error를 던지게끔 하려면 default가 필요하다.
```javascript
type FarmAnimal = Pig | Rooster | Cow | Sheep;  
function getFarmAnimalSound(animal: FarmAnimal) {
    switch(animal.kind) { //sheep case가 없음.
        case "pig":
            return "Oink!";
        case "cow":
            return "Mooo!";
        case "rooster":
            return "Cockadoodledoo!";
        default:
            const _exhaustiveCheck: never = animal; //Type Sheep is not assignable to type 'never'
    }
}
```

- default까지 내려가게 되면 무조건 error가 난다.
```javascript
type FarmAnimal = Pig | Rooster | Cow | Sheep;  
function getFarmAnimalSound(animal: FarmAnimal) {
    switch(animal.kind) { //sheep case가 없음.
        case "pig":
            return "Oink!";
        case "cow":
            return "Mooo!";
        case "rooster":
            return "Cockadoodledoo!";
        case "sheep":
            return "Baaa!";
        default:
            const _exhaustiveCheck: never = animal; //Type Sheep is not assignable to type 'never'
        return  _exhaustiveCheck; //never는 return이 불가능하므로 확실하게 에러가 난다.
    }
}
```

- type declaration file도 있다.
- 이건 .d.ts files다.
- lib.dom.d.ts가 대표적이다.
- 여긴 interface, class, type만 있다.
- library를 만들 때도 .d.ts.files가 반드시 필요하다.
- axios는 npm install 시에 index.d.ts를 같이 생성해 ts가 type 해석에 필요한 정보를 제공한다.
```javascript


//generics가 있다. 써서 
get<T = any, R = AxiosResponse<T>, D =any>(url: string, config?: AxiosRequestConfig<D>) : Promise<R>
export interface AxiosResponse<T = any, D = any> {
    data: T,
    status: number;
    statusText: string;
    headers: AxiosResponseHeaders;
    config: AxiosRequestConfig<D>;
    request?: any;
}

axios.get<User> /* return type을 지정할 수 있다. */("https://jsonplaceholder.typicode.com/users/1")
.then((res) => {
    const {data} = res;
    printUser(data);
})

axios.get<User[]> /* return type을 지정할 수 있다. */("https://jsonplaceholder.typicode.com/users/1")
.then((res) => {
    const {data} = res;
    res.data.forEach(printUser); //배열이면 foreach로.
})


function printUser({user: {name, email, phone} }: User) {
    console.log(name);
    console.log(email);
    console.log(phone);
}
```

- Lodash를 깔아보면, package.json에 types가 없다.
- types가 없다는 것은 type을 규정해줄 ~d.ts가 없다는 의미다.
- cannot find module 오류는 lodash 자체를 안 깔아서 나는 오류다.
- types가 없어서 나는 오류는 cannot find a declaration file for module 'lodash'다.
- 그럼 types를 깔면 된다. npm i --save-dev @types/lodash로 깔아준다.


- ts는 export keyword가 없으면 모두 global scope로 간주한다.
- export를 써야 Module이 작동한다.
- module을 commonjs로 해두면, 브라우저의 JS 엔진은 modules.exports에 대해 모른다. 그래서 export, import를 지우고 <script src=>로 가져와야 제대로 작동한다.
- module을 ES6로 해두고, <script type='module' src=>로 바꿔두면 export가 제대로 작동한다.



```javascript
import User, {userHelper, sample as randomSample} from "./User.js"; //한꺼번에 export default, export import 가능!
const sample = 122_342_333; //error. sample이라는 변수명이 이미 있음.
const randomSample = 122_342_333; //ok. as로 받아온 변수명은 안 겹쳐서 가능!
```

```javascript
export default class User {
    ....
}

export const userHelper = () => {
    ....
}
```

- type, interface를 import할 때는 그냥 import는 별로다.
- 아래와 같이 해준다.

```javascript
import type {Person} from "./types.js"
```

- webpack을 쓰면 dependency 등이 모두 차례에 맞게 정렬되게끔 모든 js파일과 리소스를 한 js에 묶어준다.
- 대신 소스코드가 정말 복잡하다.
```javascript
const path = require('path'); //path.resolve(__dirname, 'dist')를 위해 필요함.
module.exports = {
    entry: "./src/index.ts"; 
    module: {
        rules: [
            {
                test: /\.tsx?$/, //tsx파일이면 아래 laoder를 사용
                use: "ts-loader",
                exclude: /node_modules/ //node_modules은 type이 섞인 경우가 많아 대부분 제외
            }
        ]
    },
    resolve: {
        extensions: [".tsx",".ts",'.js'] //webpack이 포함해야 할 js형식
    },
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist') //__dirname은 node 설정으로 보인다.
    }
}
```

- webpack을 쓰고 webpack으로 script src를 가져온다면 <script type='module'>을 쓸 필요가 없다.
- 그냥 <script src="dist/bundle.js">등으로 가져오면 된다.

- webpack에서 source map을 설정해주면 bundling되지 않은 소스코드를 볼 수 있다.
- webpack_ts쪽에 있다.