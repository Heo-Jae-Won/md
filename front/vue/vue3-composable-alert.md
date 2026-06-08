- 상태 저장 로직을 컴포저블 함수로 분리하고 다른 컴포넌트에서 사용하기 위한 게 composable이다.
- vAlert를 다른 곳에서 활용하기 쉽게끔 만들려면 composable로 묶어주면 좋다.


```js
const alerts = ref([]);

const vAlert = (message, type = 'error') => {
  alerts.value.push({message, type})
  setTimeout(() => {
    alerts.value.shift();
  }, 2000)
}
```


- hooks folder를 만들어서 composable 함수를 모아둔다.
- 작명 규칙은 use를 앞에다가 붙이는 것이다.

```js
// /hooks/alert.js
import { ref } from 'vue'

export function useAlert() {
  const alerts = ref([]);

  const vAlert = (message, type = 'error') => {
    alerts.value.push({message, type})
    setTimeout(() => {
      alerts.value.shift();
    }, 2000)
  };

  const vSuccess = message => vAlert(message, 'success');

  return {
    alerts,
    vAlerts,
    vSuccess
  }
}
```


- 어느 곳에서나 쓸 수 있게 alert 함수를 만들었다.

```html
<!--PostCreateView.vue -->

<script setup>
import { useAlert } from '@/composables/alert';

const { alerts, vAlert, vSuccess } = useAlert();

const save = () => {
  try {
    createPost({
      ...form.value,
      createdAt: Date.now()
      
    });

    vSuccess('등록이 완료되었습니다.');
  } catch (error) {
    vAlert(error.message);
  }
}

</script>
```


- 하지만 수정이 끝나고 다른 경로로 이동을 하게 되는 순간, 노티가 뜨지 않게 되어버린다.
- 노티를 띄우는 component는 PostEditView component의 child인데, router로 이동하여 PostlistView component로 이동했다.
- 그럼 하위 component가 사라지면서 노티를 띄울 수 있는 DOM이 없어지는 것이다.


```html
<!--App.vue-->
<template>
  <TheHeader></TheHeader>
  <TheView></TheView>
</template>

<!-- PostEditView.vue-->
<PostForm> ~~~~ </PostForm>
<AppAlert :items="alerts" />
```

- 해당 현상을 수정하기위해 AppAlert component의 위치를 PostEditView와 동일하게 맞춰주자.
- 그래도 여전히 노출되지 않는다.
- alerts라는 state가 App.vue에서 생성되고, PostEditView.vue에서 또 생성되어 각자 서로의 state를 갖게 되기 때문이다.

```html
<!--App.vue-->
<script setup>
import { useAlert } from '@/composables/alert';
const { alerts } = useAlert();

</script>

<template>
  <TheHeader></TheHeader>
  <TheView></TheView>
  <AppAlert :items="alerts" />
</template>

<!-- PostEditView.vue-->
<PostForm> ~~~~ </PostForm>
```


- 그를 방지하기 위해서는 ref 변수를 함수 밖에 선언해야 한다.
- 그래야 새롭게 만들어지지 않고 component 간에 상태가 공유된다.

```js
// /hooks/alert.js
improt { ref } from 'vue'

const alerts = ref([]);

export function useAlert() {

  const vAlert = (message, type = 'error') => {
    alerts.value.push({message, type})
    setTimeout(() => {
      alerts.value.shift();
    }, 2000)
  };

  const vSuccess = message => vAlert(message, 'success');

  return {
    alerts,
    vAlerts,
    vSuccess
  }
}
```


- 사실 위처럼 ref가 겹치지 않고 공유되게끔 중앙 관리형 store를 사용하는 것이 더 바람직하다.

```js
//useAlertStore.js
import { definedStore } from "pinia";

export const useAlertStore = defineStore('alert', {
  state: () => ({
    alerts = [];
  }),
  actinos: {
    vAlert (message, type = 'error') {
      this.alerts.value.push({message, type})
      setTimeout(() => {
        this.alerts.value.shift();
      }, 2000)
    },
    vSuccess(message) {
      this.vAlert(message, 'success');
    } 

  }
})
```


- store의 변수는 storeToRefs()로, method는 useAlertStore()로 가져온다.

```js
// /hooks/alert.js
import { ref } from 'vue'
import { useAlertStore } from '@/stores/alert';
import { storeToRefs } from 'pinia';

export function useAlert() {
  const { alerts }             = storeToRefs(useAlertStore());
  const { vAlert, vSuccess }   = useAlertStore();
  
  return {
    alerts,
    vAlerts,
    vSuccess
  }
}
```

- 이젠 props로 alerts를 내리지 않는다.
- store에서 관리하기 때문이다.

```html
<!--App.vue-->
<script setup>
import { useAlert } from '@/composables/alert';
</script>

<template>
  <TheHeader></TheHeader>
  <TheView></TheView>
  <AppAlert />
</template>
```



- props를 활용하지 않으니 전부 store에서 가져오게 변경한다.

```html
<!--AppAlert.vue-->
<template>
	<div class="app-alert">
		<TransitionGroup name="slide">
			<div
				v-for="({ message, type }, index) in alerts /* items*/"
				:key="index"
				class="alert"
				:class="typeStyle(type)"
				role="alert"
			>
				{{ message }}
			</div>
		</TransitionGroup>
	</div>
</template>

<script setup>
import { useAlert } from '@/composables/alert';
/*
  defineProps({
    items: Array,
  })
*/
const { alerts } = useAlert();
const typeStyle = type => (type === 'error' ? 'alert-danger' : 'alert-primary');
</script>
```