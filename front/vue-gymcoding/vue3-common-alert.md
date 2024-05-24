
- AppAlert는 

```js
<template>
	<div v-else>
		<h2>게시글 수정</h2>
		<hr class="my-4" />
		<PostForm
			v-model:title="form.title"
			v-model:content="form.content"
			@submit.prevent="edit"
		>
			<template #actions>
				<button
					type="button"
					class="btn btn-outline-danger"
					@click="goDetailPage"
				>
					취소
				</button>
				<button class="btn btn-primary" :disabled="editLoading">
          수정
				</button>
			</template>
		</PostForm>
    <AppAlert> :show="showAlert"
	</div>
</template>

<script setup>
const route = useRoute();
const router = useRouter();
const id = route.params.id;

const edit = async () => {
	try {
    await updatePost(id, {...form.value});

    vAlert()
  } catch( error ) {
    console.error(error);
    vAlert('네트워크 오류입니다.');
  }
};

const goDetailPage = () => router.push({ name: 'PostDetail', params: { id } });

const showAlert= ref(false);
const alertMessage = ref('');
const vAlert = (message) => {
  setTimeout(() => {
    showAlert.value = true;
    alertMessage.value = message;
  }, 2000); //2초 뒤 수정 alert 사라짐.
  
}
</script>

<style lang="scss" scoped></style>
```


- vAlert component는 받은 message와 show 여부로 노출/비노출된다.

```js
//vAlert.vue
<template>
  <div v-if="show" class="app-alert alert alert-success" role="alert">
   {{ message }}
  </div>
</template>

<script setup>
defineProps({
  show: {
    type: Boolean,
    defeault: false,
  },
  message: {
    type: String,
    required: false,
  },
})
</script>

<style scoped>
.app-alert {
  position: fixed;
  top: 10px;
  right: 10px;
}
</style>
```


- alert 메시지에 type도 넣어서 성공이냐 실패냐를 부모에서 주게끔 한다.
- class를 주고, :class로 동적으로 주면 :class 영역은 조건에 따라 더해진다.

```js
<template>
  <div v-if="show" class="app-alert alert" :classs="styleClass" role="alert">
   {{ message }}
  </div>
</template>

<script setup>
defineProps({
  show: {
    type: Boolean,
    defeault: false,
  },
  message: {
    type: String,
    required: false,
  },
  type: {
    type: String,
    default: '',
    validator: (value) => ['success','error'].includes(value),
  }
});

const styleClass = computed(() => {
  props.type === 'error' ? 'alert-danger' : 'alert-success';
})
</script>

<style scoped>
.app-alert {
  position: fixed;
  top: 10px;
  right: 10px;
}
</style>
```

- 수정을 눌러 성공하게 되면 type까지 받아서 알아서 alert HTML 색깔을 성공으로 띄운다.

```js
<template>
	<div v-else>
		<h2>게시글 수정</h2>
		<hr class="my-4" />
		<PostForm
			v-model:title="form.title"
			v-model:content="form.content"
			@submit.prevent="edit"
		>
			<template #actions>
				<button
					type="button"
					class="btn btn-outline-danger"
					@click="goDetailPage"
				>
					취소
				</button>
				<button class="btn btn-primary" :disabled="editLoading">
          수정
				</button>
			</template>
		</PostForm>
    <AppAlert> :show="showAlert"
	</div>
</template>

<script setup>
const route = useRoute();
const router = useRouter();
const id = route.params.id;

const edit = async () => {
	try {
    await updatePost(id, {...form.value});

    vAlert()
  } catch( error ) {
    console.error(error);
    vAlert('네트워크 오류입니다.', 'success');
  }
};

const goDetailPage = () => router.push({ name: 'PostDetail', params: { id } });

const showAlert= ref(false);
const alertMessage = ref('');
const alertType = ref('error');
const vAlert = (message, type = 'error') => {
  setTimeout(() => {
    showAlert.value = true;
    alertMessage.value = message;
    alertType.value = type;
  }, 2000); //2초 뒤 수정 alert 사라짐.
  
}
</script>

<style lang="scss" scoped></style>
```


- 해당 alert component를 transition을 걸 수도 있다.

```html
<template>
  <Transition>
    <div v-if="show" class="app-alert alert" :classs="styleClass" role="alert">
    {{ message }}
    </div>
  </Transition>
</template>

<script setup>
defineProps({
  show: {
    type: Boolean,
    defeault: false,
  },
  message: {
    type: String,
    required: false,
  },
  type: {
    type: String,
    default: '',
    validator: (value) => ['success','error'].includes(value),
  }
});

const styleClass = computed(() => {
  props.type === 'error' ? 'alert-danger' : 'alert-success';
})
</script>
```

- transition을 걸었다면 아래처럼 예약어 class로 opacity를 준다.
- v-enter-~는 나타날 때, v-leave-~는 사라질 때다.

```html
<style scoped>
.app-alert {
  position: fixed;
  top: 10px;
  right: 10px;
}

//나타날 때
.v-enter-to {
  opacity: 1;
}
.v-enter-from {
  opacity: 0;
}
.v-enter-active {
  transition: opacity 0.5s ease;
}

//사라질 때

.v-leave-to {
  opacity: 0;
}
.v-leave-from {
  opacity: 1;
}
.v-leave-active {
  transition: opacity 0.5s ease;
}

//합쳐서 아래와 같이 표현
.v-enter-to, .v-leave-from  {
  opacity: 1;
}
.v-enter-from, .v-leave-to  {
  opacity: 0;
}
.v-enter-active, .v-leave-active {
  transition: opacity 0.5s ease;
}
</style>
```


- transition에 이름을 줄 수도 있다.
- 이름을 주면 그에 맞춰 class 이름도 바꿔준다.
- 이번엔 위에서 아래로 내려오는 효과도 같이 줬다.
- Transition component는 v-if나 v-show가 있을 때만 사용가능하다.

```html
<template>
  <Transition name="slide">
    <div v-if="show" class="app-alert alert" :classs="styleClass" role="alert">
    {{ message }}
    </div>
  </Transition>
</template>

<script setup>
defineProps({
  show: {
    type: Boolean,
    defeault: false,
  },
  message: {
    type: String,
    required: false,
  },
  type: {
    type: String,
    default: '',
    validator: (value) => ['success','error'].includes(value),
  }
});

const styleClass = computed(() => {
  props.type === 'error' ? 'alert-danger' : 'alert-success';
})
</script>

<style scope>
.slide-enter-to, .slide-leave-from {
  opacity: 1;
  transform: translateY(0px);
}
.slide-enter-from, .slide-leave-to {
  opacity: 0;
  transform: translateY(-30px);
}
.slide-enter-active, .slide-leave-active {
  transition: all 0.5s ease;
}
</style>
```