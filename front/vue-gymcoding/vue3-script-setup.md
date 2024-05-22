
- script setup의 장점은 번거로운 boilderplate를 쓰지 않아도 된다는 점이다.

```js
<template>
    <div class="container py-4"> 
        {{ msg }} 
        <input v-model="message" type="text" >
        <button @click="sayHello">click</button>
        <PostItem 
            type="news" 
            title="제목" 
            contents="내용"
            :is_like="true"
></PostItem>
    </div>
</template>

<script setup>
import { ref } from 'vue';
import PostItem from '@/components';
    const msg ='Hello World';
    const message = ref('');
    const sayHello = () => {
        alert('Hello World');
    }
</script>
```

- props를 script setup에서 사용하면 defineProps()와 defineEmis()를 사용한다.

```js
<template>
  <div class="card">
    <div class="card-body">
      <span class="badge bg-secondary">{{ typeName }}</span>
      <h5 class="card-title red mt-2">{{ title }}</h5>
      <p class="card-text">{{ contents }}</p>
      <a href="#" class="btn" :class="isLikeClass" @click="toggleLike">
        좋아요
      </a>
    </div>
  </div>
</template>

<script setup>
import { computed, defineProps, defineEmits } from 'vue';

const props = defineProps({
  type: {
    type: String,
    default: 'news',
    validator: value => ['news', 'notice'].includes(value)
  },
  title: {
    type: String,
    required: true
  },
  contents: {
    type: String,
    default: ''
  },
  isLike: {
    type: Boolean,
    default: false
  }
});

const { emit } = defineEmits(['toggleLike']);

const isLikeClass = computed(() => props.isLike ? 'btn-danger' : 'btn-outline-danger');

const typeName = computed(() => props.type === 'news' ? '뉴스' : '공지사항');

const toggleLike = () => emit('toggleLike');
</script>

<style></style>
```


- script setup과 script는 같이 쓸 수 있다.
- script에는 한번만 진행해야 하는 로직, 혹은 non-props 설정 등을 넣는다.

```js
<script>
console.log('Normal Script');
export default {
    inheritAttrs: false,
}
</script>


<script setup>
//setup()에서 진행할 실제 로직
</script>
```


- script setup을 하면 최상위 level에서 async 없이 await를 호출할 수 있다.

```js
<script setup>

    const response = await axios(
        'https://~~~~'
    );
</script>
```

- 다만 eslint에서 설정을 추가해줘야 한다.
- 7 버전과 그아래는 아래와 같이 사용한다.

```js
module.exports = {
  parser: 'vue-eslint-parser',
  parserOptions: {
    parser: 'espree', // <-
    ecmaVersion: 2022, // <-
    sourceType: 'module'
  },
}
```

- 8 버전과 그 이상은 아래와 같이 사용한다.

```js
module.exports = {
  parser: 'vue-eslint-parser',
  parserOptions: {
    parser: 'espree', // <-
    ecmaVersion: 2022, // <-
    sourceType: 'module'
  },
}
```