- dynamic component용으로 우선 component를 만든다.
- 우선 사과다.

```js
<template>
	<AppCard> 사과 </AppCard>
</template>

<script setup>
import AppCard from './AppCard.vue';
</script>
```

- 그 다음은 바나나다.

```js
<template>
	<AppCard> 바나나 </AppCard>
</template>

<script setup>
import AppCard from './AppCard.vue';
</script>
```

- dynamic component를 구성 할 때는 :is를 활용한다.

```js
<component :is="currentComp" />
```


- 버튼을 누르면 component가 바뀌게끔 함수를 구성할 때, shallowRef를 사용한다.
- shallowRef는 값의 일부가 바뀌어도 전체가 바뀌지 않으면 rendering되지 않기 때문이다.
- dynamic component 용도라고 보면 된다.

```js
<div class="container py-4">
    <button
        class="btn btn-primary me-2"
        @click="changeCurrentComp(DynamicApple)"
    >
        사과
    </button>
    <button class="btn btn-danger" @click="changeCurrentComp(DynamicBanana)">
        바나나
    </button>
    <hr />
    <component :is="currentComp" />
</div>

import DynamicApple from './DynamicApple.vue';
import DynamicBanana from './DynamicBanana.vue';
const currentComp = shallowRef(DynamicApple);
const changeCurrentComp = comp => (currentComp.value = comp);
```



```js
<template>
	<div class="container py-4">
		<button
			class="btn btn-primary me-2"
			@click="changeCurrentComp(DynamicApple)"
		>
			사과
		</button>
		<button class="btn btn-danger" @click="changeCurrentComp(DynamicBanana)">
			바나나
		</button>
		<hr />
		<component :is="currentComp" />
		<p>{{ obj1 }}</p>
		<p>{{ obj2 }}</p>
	</div>
</template>

<script setup>
import { ref, shallowRef } from 'vue';
import DynamicApple from './DynamicApple.vue';
import DynamicBanana from './DynamicBanana.vue';

const currentComp = shallowRef(DynamicApple);
const obj1 = ref({ name: '홍길동' });
const obj2 = shallowRef({ name: '김길동' });
const changeCurrentComp = comp => (currentComp.value = comp);
</script>

<style lang="scss" scoped></style>
```



- 이전에 눌렀던 탭의 정보를 유지하려면 아래 같이 keep alive를 선언한다.

```js
<keep-alive>
  <component :is="currentComp" />
</keep-alive>
```


- 만약 특정 탭은 상태 유지에서 제외시키고 싶다면 exclude를 선언한다.

```js
<keep-alive exclude="DynamicApple, DynamicBanana">
  <component :is="currentComp" />
</keep-alive>
```

- 특정 탭들만 상태 유지를 시키고 싶다면 include를 선언한다.

```js
<keep-alive :include="['DynamicApple', 'DynamicBanana']">
  <component :is="currentComp" />
</keep-alive>
```
