- 반응형 객체의 속성을 destructuring해오고 parameter로 넘길 때 reactivty를 잃어버릴 수 있다.
- 그런 경우에, toRef를 활용하

```js
//number.js
export const useNumber = number   => {
  const iOdd = computed(() => number % 2 === 1);
  const isEven = computed(() => !isOdd.value);

  return {
    isOdd,
    isEven,
  }
}
```


- props를 반응형으로 보냈지만, 그 속의 id를 꺼내오면서 반응성이 사라졌다.

```html
<template>
  {{isOdd}}
</template>

<script seupt>
import { useNumber } from '@/composables/number';
const props = defineProps({
  id: [String, Number],
})
  
const { isOdd } = useNumber(props.id);


</script>
```


- 그럴 때 toRef()를 활용한다.

```html
<template>
  {{isOdd}}
</template>

<script seupt>
import { useNumber } from '@/composables/number';
const props = defineProps({
  id: [String, Number],
})
  
const idRef = toRef(props, 'id');
const { isOdd } = useNumber(idRef);
</script>
```


- toRefs()를 사용해서도 가져올 수도 있다.

```html
<template>
  {{isOdd}}
</template>

<script seupt>
import { useNumber } from '@/composables/number';
const props = defineProps({
  id: [String, Number],
})
  
const {id: idRef} = toRefs(props);
const { isOdd } = useNumber(idRef);
</script>
```


- number.js에서 reactive로 보내는 경우에도 반응성이 풀리게 된다.

```js
//number.js
export const useNumber = number   => {
  const iOdd = computed(() => number % 2 === 1);
  const isEven = computed(() => !isOdd.value);

  const state = reactive({
    x: 100,
    y: 1000,
  });

  return state;
}

//PostDetail.vue
const {id: idRef} = toRefs(props);
const { x, y } = useNumber(idRef); //반응성 풀림
```


- 그럴 때 reactive의 변수들을 개별 ref로 만들어서 return한다.
- 다만 여기서 x와 y는 ref라 x.value와 같은 형태로 사용해야 한다.

```js
//number.js
export const useNumber = number   => {
  const iOdd = computed(() => number % 2 === 1);
  const isEven = computed(() => !isOdd.value);

  const state = reactive({
    x: 100,
    y: 1000,
  });

  return toRef(state);
}

//PostDetail.vue
const {id: idRef} = toRefs(props);
const { x, y } = useNumber(idRef); //반응성 유지
```