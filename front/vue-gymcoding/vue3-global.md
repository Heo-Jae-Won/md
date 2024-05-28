- plugin이란 vue에 전역 수준의 기능을 추가할 때 사용하는 기능이다.
- install로 object를 만들어놓거나 method를 만들어놓는다.

```js
const objPlugin = {
  install(app, options) {
    console.log('objPlugin app: ', app);
    console.log('objPlugin options: ', options);
    // app.component() 전역 컴포넌트
    // app.config.globalProperties 전역 애플리케이션 인스턴스에 속성 추가
    // app.directive 커스텀 디렉티브
    // app.provide 리소스
  }
}
```


- 함수로 만드는 방법도 있다. 다만 install이 더 직관적이라고 생각된다.

```js
function funcPlugin(app. options) {
  console.log('funcPlugin');
}

export default funcPlugins;
```

- use() method를 사용해서 만들어둔 plugin을 넣을 수 있다.

```js
//main.js
import App from './App.vue';
import router from '@/router';
import funcPlugins from './plugins/obj';
import objPlugins from './plugins/obj';

const app = createaApp(app);
app.use(funcplugins);
app.use(objPlugins, {name: '짐코딩'});
app.use(router);
app.mount('#amount');
```


- 아래 예시로 만든 person 객체를 전역에서 쓰고 싶다고 해보자.

```js
//   /plugins/person.js
export default {
  install(app. options) {
    const person = {
      name: '짐코딩',
      say() {
        alert(this.name);
      }
    };
    app.config.globalProperties.$person = person;
  }
}
```


- 만들어둔 install 객체를 가져와서 넣어준다.

```js
import App from './App.vue';
import funcPlugins from './plugins/func';
import person from './plugins/person';

const app = createaApp(App);
app.use(person);
```


- $표시를 붙여서 가져와서 쓸 수 있다.
- 하지만 해당 방식으론 vue3 composition api로는 활용이 불가능하다. 
- 따라서 vue2의 방식을 그대로 가져와서 활용해야 한다.

```html
// HomeView.vue

<script>
export default {
  created() {
    console.log(this.$person.name); //짐코딩;
  }
}
</script>
```


- composition api에서 활용하려면 provide inject를 활용해야 한다.
- app을 만드는 main.js에서 app을 사용했었지만, 여기서도 사용할 수 있다.
- install이라서 가능하다.

```html
<script>
//   /plugins/person.js
export default {
  install(app. options) {
    const person = {
      name: '짐코딩',
      say() {
        alert(this.name);
      }
    };
    app.config.globalProperties.$person = person;
    app.provide('person', person);
  }
}
</script>
```

- app에 등록한 전역 state를 꺼내서 쓰기 위해서 inject를 한다.

```html
// HomeView.vue
<template>
  <button @click="person.say">click person</button>
</template>

<script setup>
  const person = inject('person');
  console.log('name: ', person.name);
</script>
```



- global components도 plugin으로 등록 가능하다.
- 하지만 global component를 등록하게 되면 tree - shaking이 불가능하므로 바람직하지 않다.
- 그냥 기존에 하던대로 local registration으로 사용하자.


```html
//script setup
<script setup>
import ComponentA from './ComponentA.vue'
</script>

<template>
  <ComponentA />
</template>

<script>
//non script setup 
import ComponentA from './ComponentA.js'

export default {
  components: {
    ComponentA
  },
  setup() {
    // ...
  }
}
```



- 예시는 아래와 같다. 
- 우선 import syntax 없이 쓸 global component를 준다.

```js
// global-components.js
import AppAlert from '@/components/app/AppAlert.vue';
import AppCard from '@/components/app/AppCard.vue';
import AppAlert from '@/components/app/AppAlert.vue';
import AppGrid from '@/components/app/AppGrid.vue';
import AppPagination from '@/components/app/AppPagination.vue';

export default {
  install(app) {
    app.component('AppAlert', AppAlert);
    app.component('AppCard', AppCard);
    app.component('AppGrid', AppGrid);
    app.component('AppModal', AppModal);
  }
}
```


- component를 등록했으면 사용하겠다고 한다.
- type에도 등록시켜줘야 한다.

```js
import globalComponents from '@/plugins/global-components';
const app = createApp(App);
app.use(globalComponents);

//components.d.ts
declare module '@vue/runtime-core' {
  export interface GlobalComponents {
    AppAlert: typeof import('./src/componentns/app/AppAlert.vue')['default']
    AppCard: typeof import('./src/componentns/app/AppCard.vue')['default']
    AppAlert: typeof import('./src/componentns/app/AppAlert.vue')['default']
    AppGrid: typeof import('./src/componentns/app/AppGrid.vue')['default']
    AppPagination: typeof import('./src/componentns/app/AppPagination.vue')['default']
    RouterLink: typeof import('vue-router')['RouterLink']
    RouterView: typeof import('vue-router')['RouterView']
  }
}
```



- 이 귀찮은 작업을 대신 해주는 plugin도 있다.
- unplugin-vue-components다.
- npm i unplugin-vue-components -D로 깐다. 
  - js는 type 정보가 필요하지 않아 운영에선 해당 lib가 필요없다.
  - dir을 지정해서 전역으로 쓸 component만 넣어놓자.

```js
//vite.config.js
import { fileURLToPath, URL } from 'url';
import Components from 'unplugin-vue-components/vite';

import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

// https://vitejs.dev/config/
export default defineConfig({
	plugins: [
		vue(),
		Components({
			dirs: ['src/components/app'],
			dts: true,
		}),
	],
	resolve: {
		alias: {
			'@': fileURLToPath(new URL('./src', import.meta.url)),
		},
	},
});
```





