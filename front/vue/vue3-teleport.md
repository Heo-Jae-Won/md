

- 아래는 bootstrap의 modal창이다.
- modal을 일일이 html로 넣기에는 바람직하지 않다.
- 따라서 modal 창을 공통 component로 넣어보자.

```html
//AppModal.vue
<button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#exampleModal">
  Launch demo modal
</button>

<div class="modal-backdrop fade show"></div>
<div 
  id="exampleModal"
  class="modal fade show d-block" 
  tabindex="-1" 
  aria-labelledby="exampleModalLabel" 
  aria-hidden="true">
  <div class="modal-dialog">
    <div class="modal-content">
      <div class="modal-header">
        <slot name="header">
          <h1 class="modal-title fs-5" id="exampleModalLabel">{{title}}</h1>
          <button 
            type="button" 
            class="btn-close" 
            data-bs-dismiss="modal" 
            aria-label="Close"
          ></button>
        </slot>
      </div>
      <div class="modal-body">
        <slot></slot>
      </div>
      <div class="modal-footer">
        <slot name="actions">
          <button 
            type="button"
            class="btn btn-secondary"
            data-bs-dismiss="modal"
          >
            Close
          </button>
        </slot>
        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
        <button type="button" class="btn btn-primary">Save changes</button>
      </div>
    </div>
  </div>
</div>


<script setup>
defineProps({
  show: Boolean,
  title: String,
})
</script>
```

- child component에서 modal event를 emit한다.

```html
//PostItem.vue
<template>
	<AppCard>
		<h5 class="card-title text-truncate">{{ title }}</h5>
		<p class="card-text text-truncate">
			{{ content }}
		</p>
		<p class="text-muted">{{ createdDate }}</p>
		<template #footer>
			<div class="d-flex flex-row-reverse">
				<button class="btn p-1" @click="$emit('modal')">
					<i class="bi bi-emoji-sunglasses"></i>
				</button>
			</div>
		</template>
	</AppCard>
</template>

<script setup>
import { computed, inject } from 'vue';

const props = defineProps({
	title: {
		type: String,
		required: true,
	},
	content: {
		type: String,
	},
	createdAt: {
		type: [String, Date, Number],
	},
});
defineEmits(['modal', 'preview']);
const dayjs = inject('dayjs');
const createdDate = computed(() =>
	dayjs(props.createdAt).format('YYYY. MM. DD HH:mm:ss'),
);
</script>

<style lang="scss" scoped></style>
```

- parent component에서 child component로 emit event를 보낸다.
- 그러나 모달버튼을 클릭해도 모달이 뜨지 않고 상세 페이지로 이동한다.

```html
<template>
	<div>
		<h2>게시글 목록</h2>
		<hr class="my-4" />

			<AppGrid :items="posts" col-class="col-12 col-md-6 col-lg-4">
				<template v-slot="{ item }">
					<PostItem
						:title="item.title"
						:content="item.content"
						:created-at="item.createdAt"
						@click="goPage(item.id)"
						@modal="openModal(item)"
						@preview="selectPreview(item.id)"
					></PostItem>
				</template>
			</AppGrid>
</template>

<script setup>
import PostItem from '@/components/posts/PostItem.vue';
// modal
const show = ref(false);
const modalTitle = ref('');
const modalContent = ref('');
const modalCreatedAt = ref('');
const openModal = ({ title, content, createdAt }) => {
	show.value = true;
	modalTitle.value = title;
	modalContent.value = content;
	modalCreatedAt.value = createdAt;
};
</script>

<style lang="scss" scoped></style>
```


- event가 bubbling되었기 때문이고, 그럴 때, event bubbling을 막아줘야 한다.

```html
<template>
	<AppCard>
		<h5 class="card-title text-truncate">{{ title }}</h5>
		<p class="card-text text-truncate">
			{{ content }}
		</p>
		<p class="text-muted">{{ createdDate }}</p>
		<template #footer>
			<div class="d-flex flex-row-reverse">
				<button class="btn p-1" @click.stop="$emit('modal')">
					<i class="bi bi-emoji-sunglasses"></i>
				</button>
			</div>
		</template>
	</AppCard>
</template>

<script setup>
import { computed, inject } from 'vue';

const props = defineProps({
	title: {
		type: String,
		required: true,
	},
	content: {
		type: String,
	},
	createdAt: {
		type: [String, Date, Number],
	},
});
defineEmits(['modal', 'preview']);
const dayjs = inject('dayjs');
const createdDate = computed(() =>
	dayjs(props.createdAt).format('YYYY. MM. DD HH:mm:ss'),
);
</script>

<style lang="scss" scoped></style>
```


- 이번에는 child component에 props가 아니라 v-model을 주는 형태로 바꿔보자.
- 보내는 v-model은 show 지만, modelValue로 받고, type은 show의 type이다. 즉 Boolean이다.
- v-model로 보낸 값인 show를 쓸 때 또한 modelValue로 쓴다. 

```html
<template>
	<Transition>
		<div v-if="modelValue">
			<div class="modal-backdrop fade show"></div>
			<div
				class="modal fade show d-block"
				tabindex="-1"
				aria-labelledby="exampleModalLabel"
				aria-hidden="true"
			>
				<div class="modal-dialog">
					<div class="modal-content">
						<div class="modal-header">
							<slot name="header">
								<h5 class="modal-title" id="exampleModalLabel">{{ title }}</h5>
								<button
									type="button"
									class="btn-close"
									aria-label="Close"
									@click="$emit('update:modelValue', false)"
								></button>
							</slot>
						</div>
						<div class="modal-body">
							<slot></slot>
						</div>
						<div class="modal-footer">
							<slot name="actions"></slot>
						</div>
					</div>
				</div>
			</div>
		</div>
	</Transition>
</template>

<script setup>
defineProps({
	modelValue: Boolean,
	title: String,
});
defineEmits(['close', 'update:modelValue']);
</script>

<style scoped>
.v-enter-from,
.v-leave-to {
	opacity: 0;
}
.v-enter-active,
.v-leave-active {
	transition: all 0.5s ease;
}
.v-enter-to,
.v-leave-from {
	opacity: 1;
}
</style>

```


- v-model에 name을 달아서 보낸 것들은 아래와 같이 받아와서 쓸 수 있다.

```html
//Parent component
<PostFilter 
      v-model:title="params.title_like" 
      v-model:limit="params._limit" 
    />


//Child component
<template>
  <input 
    :value="title" 
    @input="$emit('update:title', $event.target.value)" 
    type="text" 
    class="form-control" 
    placeholder="제목을 입력해주세요" />
  </div>
  <div class="col">
    <select 
      :value="limit" 
      @input="$emit('update:limit', $event.target.value)" 
      class="form-select"
    >
  .
  .
  .
</template>

<script setup>
defineProps({
  title: String,
  limit: Number,
});

defineEmits(['update:title','update:limit']);
</script>
```


- AppModal 아래에 있는 template 조차도 기니까 빼주자.

```html
<AppModal v-model="show" title="게시글">
	<template #default>
		<div class="row g-3">
			<div class="col-3 text-muted">제목</div>
			<div class="col-9">{{ title }}</div>
			<div class="col-3 text-muted">내용</div>
			<div class="col-9">{{ content }}</div>
			<div class="col-3 text-muted">등록일</div>
			<div class="col-9">
				{{ $dayjs(createdAt).format('YYYY. MM. DD HH:mm:ss') }}
			</div>
		</div>
	</template>
	<template #actions>
		<button type="button" class="btn btn-secondary" @click="closeModal">
			닫기
		</button>
	</template>
</AppModal>
```

- postModal이라는 이름으로 component를 만들고 감싸준다.
- 다만 props로 내린 것은 변경 불가능하기 때문에 다시 computed로 감싸준다.
- computed로 빼주려면 props를 변수로 만들어줘야 한다.

```html
<template>
	<AppModal v-model="show" title="게시글">
		<template #default>
			<div class="row g-3">
				<div class="col-3 text-muted">제목</div>
				<div class="col-9">{{ title }}</div>
				<div class="col-3 text-muted">내용</div>
				<div class="col-9">{{ content }}</div>
				<div class="col-3 text-muted">등록일</div>
				<div class="col-9">
					{{ $dayjs(createdAt).format('YYYY. MM. DD HH:mm:ss') }}
				</div>
			</div>
		</template>
		<template #actions>
			<button type="button" class="btn btn-secondary" @click="closeModal">
				닫기
			</button>
		</template>
	</AppModal>
</template>

<script setup>
import { computed } from 'vue';
const props = defineProps({
	modelValue: Boolean,
	title: String,
	content: String,
	createdAt: [String, Number],
});
const emit = defineEmits(['update:modelValue']);

const show = computed({
	get() {
		return props.modelValue;
	},
	set(value) {
		emit('update:modelValue', value);
	},
});
const closeModal = () => (show.value = false);
</script>

<style lang="scss" scoped></style>
```



- 부모는 아래와 같이 간단하게 사용할 수 있다.

```html
//parent component

<PostModal
  v-model="show"
  :title="modalTitle"
  :content="modalContent"
  :created-at="modalCreatedAt"
/>
```


- 문제는 modal 창의 HTML 위치다. 
- modal 창은 list 안에 속한 위치가 아니라 별도로 독립적인 위치에서 뜨는 방식이 맞기 때문이다.
- 그럴 때 Teleport component를 도입해서 css 충돌 현상, UI 버그 등을 해결할 수 있다.

```html
//PostListView.vue

<Teleport to="#modal">
			<PostModal
				v-model="show"
				:title="modalTitle"
				:content="modalContent"
				:created-at="modalCreatedAt"
			/>
		</Teleport>
```

- teleport할 위치는 원래 render하는 app이 아닌 다른 곳에 그려줘야 한다.
- index.html에서 모달창을 위한 새로운 독립 div 영역을 만들어준다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" href="/favicon.ico" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Vite App</title>
  </head>
  <body>
    <div id="app"></div>
    <div id="modal"></div>
    <script type="module" src="/src/main.js"></script>
  </body>
</html>
```