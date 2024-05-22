- vue emit event 발생은 아래와 같은 과정으로 하면 된다.

- 부모 component에 자식 component에 전달할 emit 용 함수를 만든다.

```js
<script>
export default {
	components: {
		PostItem,
		PostCreate,
		LabelInput,
	},
	setup() {
		const createPost = newPost => {
			console.log('newPost: ', newPost);
			posts.push(newPost);
		};
	},
};
</script>
```

- 부모 component에서 자식 component에 emit 용 함수를 전달한다.

```js
<template>
	<main>
		<div class="container py-4">
			<PostCreate @create-post="createPost"></PostCreate>
		</div>
	</main>
</template>
```


- 자식 component가 emit을 받아올 때는 html에 아래같이 직접 넣어줄 수도 있다.

```js
<template>
	<div class="row g-3">
		<!-- @click="" -->
		<div class="col col-2 d-grid">
			<button class="btn btn-primary" @click="$emit('createPost', 1, 2, 3, '김길동')">추가</button>
		</div>
	</div>
</template>
```

- 보통은 js에서 한번 거치게 된다.

```js
<template>
	<div class="row g-3">
		<!-- @click="$emit('createPost', 1, 2, 3, '김길동')" -->
		<div class="col col-2 d-grid">
			<button class="btn btn-primary" @click="createPost">추가</button>
		</div>
	</div>
</template>

<script>
setup(props, { emit }) {
		const type = ref('news');
		const title = ref('');
		const createPost = () => {
			const newPost = {
				type: type.value,
				title: title.value,
			};

			emit('createPost', newPost);
			console.log('1','2','4');
		};

		return { createPost, type, title };
	},

</script>
```

- emit을 받아올 때 validation check도 가능하다.
- validation check를 할 생각이 없다면 주석 처럼 만들면 된다.
- validation check 해봐야 console에 경고등 뜨는 게 전부다.

```js
export default {
    emits: {
        //createPost: null;
		createPost: newPost => {
			if (!newPost.type) {
				return false;
			} else if (!newPost.title) {
				return false;
			}
			return true;
		},
	},
}
```

