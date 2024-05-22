- props drilling을 해결하기 위한 용도가 바로 provide, inject다.

- 부모 component에서 provide를 통해 특정한 값을 주입한다.
- 아래에서는 staticMessage, message, count를 주입했다.

```js
//ParenCompoennt.vue
<template>
	<div class="container py-4">
		<div class="card">
			<div class="card-header">ProvideInejct Comp</div>
			<div class="card-body">
                <button @click="count++">Click</button>
				<Child></Child>
			</div>
		</div>
	</div>
</template>

<script>
import { provide, ref } from 'vue';
import Child from './Child.vue';
export default {
	components: {
		Child,
	},
	setup() {
		const staticMessage = 'static message';
		const message = ref('');
		const count = ref(10);
		provide('static-message', staticMessage);
		provide('message', message);
		provide('count', count);
		return {};
	},
};
</script>

<style lang="scss" scoped></style>
```

- childComponent에서는 prop만을 내리기 위한 코드를 억지로 작성하지 않는다.
- provide로 넣은 것이 있으니까 prop으로 내릴 필요가 없기 때문이다.

```js
//ChildComponent.vue
<template>
	<div class="card">
		<div class="card-header">Child Component</div>
		<div class="card-body">
			<DeepChild></DeepChild>
		</div>
	</div>
</template>

<script>
import { inject } from 'vue';
import DeepChild from './DeepChild.vue';
export default {
	components: {
		DeepChild,
	},
	setup() {
	},
};
</script>

<style lang="scss" scoped></style>
```


- 해당 prop을 꺼내올 손자 component에서 inject하여 사용한다.
- 실제 ParentComponent의 button을 눌러서 count가 증가하면 자동으로 GrandsonComponent에도 반영된다.
- 만약 ParentComponent에서 값을 넘기지 않는다면 두번째 인자로 default 값을 전달할 수 있다.

```js
//GrandsonComponent.vue
<template>
	<div class="card">
		<div class="card-header">Deep Child Component</div>
		<div class="card-body">
			<p>staticMessage: {{ staticMessage }}</p>
			<p>message: {{ message }}</p>
			<p>count: {{ count }}</p>
		</div>
	</div>
</template>

<script>
import { inject } from 'vue';

export default {
	setup() {
		const staticMessage = inject('static-message', 'default message');
		const { message } = inject('message');
		const count = inject('count');
		return { staticMessage, message, count };
	},
};
</script>

<style lang="scss" scoped></style>
```


- 가장 최상위 component인 App.vue를 만드는 main.js에서 주입하면 어느 component든 사용 가능하다.

```js
//main.js
import 'bootstrap/dist/css/bootstrap.min.css';
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);

// app.component('AppCard', AppCard);

app.provide('app-message', 'app message 입니다');
//
app.config.globalProperties.msg = 'hello';
app.provide('msg', 'hello msg');
app.mount('#app');
import 'bootstrap/dist/js/bootstrap.js';
```


- App.vue에 주입하여 childComponent에서 사용한다.

```js
//ChildComponent.vue
<template>
	<div class="card">
		<div class="card-header">Child Component</div>
		<div class="card-body">
			<p>appMessage: {{ appMessage }}</p>
			<DeepChild></DeepChild>
		</div>
	</div>
</template>

<script>
import { inject } from 'vue';
import DeepChild from './DeepChild.vue';
export default {
	components: {
		DeepChild,
	},
	setup() {
		const appMessage = inject('app-message');
		return { appMessage };
	},
};
</script>

<style lang="scss" scoped></style>
```

- ref를 provide로 전달할 경우, 값을 자식 component에서 바꾸지 않고, 바꿀 함수도 같이 전달하여야 한다.
- 만약 값을 바꾸지 못하게 하려면 readonly로 감싼다.


```js
//Provider(ParentComponent.vue)
const message = ref('Hello World');
const updateMessage = () => {
    message.value = 'world!';
}

provide('message',{ message, updateMessage});
provide('message', readonly(count));
provide('message',{ message: readonly(count), updateMessage});
```

- 주입 받은 childComponent에서는 아래와 같이 사용한다.

```js
//Injector(ChildComponent.vue)
const { message, updateMessage} = inject('message');
```


- message.value = message.value + '!';와 같이 직접 바꾸는 형태는 바람직하지 않다.
- provide에서 주입한 setter 함수를 사용해서 reactivity를 올바르게 관리하자.

```js
//child Component
<template>
	<div class="card">
		<div class="card-header">Deep Child Component</div>
		<div class="card-body">
			<p>staticMessage: {{ staticMessage }}</p>
			<p>message: {{ message }}</p>
			<p>count: {{ count }}</p>
		</div>
	</div>
</template>

<script>
import { inject } from 'vue';

export default {
	setup() {
		const staticMessage = inject('static-message', 'default message');
		const { message, updateMessage } = inject('message');
		updateMessage('!');
		//message.value = message.value + '!';
		const count = inject('count');
		return { staticMessage, message, count };
	},
};
</script>

<style lang="scss" scoped></style>
```


- key로 넣는 것은 문자열이 아니라 symbol type이 좋다.
- 큰 규모일 수록 문자열이 겹치는 사례가 많고, 겹칠 경우 overwrite가 일어난다.

```js
//keys.js
export const myInjectionKey = Symbol();

//provider component
import { provide } from 'vue';
import { myInjectionKey } from './keys.js';

provide(myInjectionKey, {

})

//injector component
import { inject } from 'vue';
import { myInjectionKey } from './keys.js';

const injected = inject(myInjectionKey);
```


- provide, inject를 이용하지 않고도 주입할 수 있는 기능도 있다.
- main에서 app 시작 시에 globalProperties에 msg property를 집어넣는다.
- app.provide()가 더 편해서 보통 그 방식이 많이 채택된다.

```js
//main.js
import 'bootstrap/dist/css/bootstrap.min.css';
import { createApp } from 'vue';
import App from './App.vue';

const app = createApp(App);

// app.component('AppCard', AppCard);

app.provide('app-message', 'app message 입니다');
//
app.config.globalProperties.msg = 'hello';
app.provide('msg', 'hello msg');
app.mount('#app');
import 'bootstrap/dist/js/bootstrap.js';
```

- 주입한 속성은 this를 붙여 꺼낼 수 있다.
- 주입한 this는 dom이 mount된 이후부터 사용할 수 있다.

```js
<template>
	<div class="container py-4">
		<div class="card">
		</div>
	</div>
</template>

<script>
import { provide, ref } from 'vue';
import Child from './Child.vue';
export default {
	components: {
		Child,
	},
	setup() {
		console.log('this arg: ', this); //undefined

		onMounted(() => {
			console.log('this arg: ', this.msg); //'hello'
		})
		return {};
	},
};
</script>
```
