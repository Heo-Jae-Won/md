- 등록할 v-directive 함수를 선언한다.

```js
// focus.js
export default {
  mounted: el => {
    el.foucs();
  }
}
```


```js
// main.js
import App from './App.vue';
import foucs from '@/directives/foucs';

const app = createApp(App);
app.directive('focus');
```


- directive에 등록한 것은 v-~로 쓸 수 있다.
- 화면이 로딩되면 해당 input가 늘 focus된다.

```html
<!--PostFrom.vue-->

<input
  :value="title"
  v-foucs
  @input="$emit(~~~)"
  />
```


- 우리가 배운 plugin을 활용해서도 directive 등록이 가능하다.
- custom directive는 mounted, updated 시에 모두 호출된다.

```js
// global-directives.js
import foucs from '@/directives/focus';

export default {
  install(app) {
    app.directive('focus', focus);
  }
}

// main.js
import globalDirectives from './plugins/global-directives';
const app = createApp(App);
app.use(globalDirectives);
```


- custom directive도 인자를 받을 수 있다.

```js
// color.js
function color(el, binding) {
  el.style.color = binding.value;
}

export default color;
```

- input


```js
<input
 v-color="'yellow'" 
>
```

- 객체 리터럴로 전달도 가능하다.

```js
<div v-demo="{ color: 'white' , text: 'hello!'}"> </div>

app.directive('demo', (el, binding) => {
  console.log(binding.value.color); // white
  console.log(binding.value.text);  // hello!
})
```

