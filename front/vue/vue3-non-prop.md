- non props는 class, style, id와 같은 것들이다.
- 기본적으론 부모 component에서 class명을 내려보내면, child component에도 상속된다.
- non props는 모두 자동 상속된다고 보면 된다. 최상위 요소에 상속된다.


```html
<!--App.vue-->
<template>
  <MyButton
    class="large"
  />
</template>


<!--MyButton.vue -->
<button>click me</button>
```

- 실제 rendering이 끝난 browser 상에서는 아래처럼 class가 붙는다.

```html
<button class="large">click me</button>
```

- 만약 원래 button에 class가 있었다면 병합된다.


```html
```html
<!--App.vue-->
<template>
  <MyButton
    class="large"
  />
</template>


<!--MyButton.vue -->
<button class="btn">click me</button>

<!--browser-->
<button class="btn large">click me</button>
```


- eventListener도 상속된다.
- class와 마찬가지로 listener도 merge되어 기존 button의 것, 내려온 것 두 개 다 나간다.

```html
<!--App.vue-->
<MyButton @click="counter++" />


<!--MyButton.vue -->
<button @click="counter--" />


<!--browser-->
<button  @click="counter++, counter--" >click me</button>
```


- 자동 상속을 막으려면 옵션을 주어야 한다.

```html
<!--MyButton.vue -->
<template>
  <button class="btn" data-link="hello">click me</button>
</template>
<script>
export default {
  inheritAttrs: false,
};
</script>
```

- 분명 최상위 요소에 상속된다고 했다.
- 그런데 상속을 최상위 요소가 아니라 하위 특정요소에 하고 싶을 수 있다.
- 그 경우 $attrs를 활용한다. $attrs는 props, emetis, class, style, v-on 중 선언되지 않은 모든 것을 가지는 짬뽕이다.

```html
<!--MyButton.vue -->
<template>
  <label>
    이름:
    <input type="text" v-bind="$attrs" />
  </label>
</template>
<script>
export default {
  inheritAttrs: false,
};
</script>

<!--Parent component-->
<MyInput
  class="my-input"
  placeholder="didj"
  @keyup="onKeyup"
  data-message="Hello World!"
/>
```

- 만약 setup에서 쓰려한다면 아래와 같이 쓸 수 있다.

```html
<script setup>
import { useAttrs } from 'vue'

const attrs = useAttrs()
</script>
```


```js
export default {
  setup(props, ctx) {
    // fallthrough attributes are exposed as ctx.attrs
    console.log(ctx.attrs)
  }
}
```