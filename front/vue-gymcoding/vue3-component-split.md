d- 컴포넌트 하나에 모든 걸 몰아서 처리하면 바람직하지 않다.

```js
//PostCreateView.vue
<template>
  <div>
    <h2>게시판 등록</h2>
    <hr class="my-4">
    <form @submit.prevent="save">
      <div class="mb-3">
        <label for="title" class="form-label">제목</label>
        <input
          v-model="form.title"
          type="text"
          class="form-control"
          id="title"
        />
      </div>
      <div class="mb-3">
        <label for="content" class="form-label">내용</label>
        <textarea
          v-model="form.content"
          class="form-control"
          id="content"
          rows="3"
        ></textarea>
      </div>
      <div class="pt-4">
        <button
          type="button"
          class="btn btn-outline-dark me-2"
          @click="goListPage"
          >
          목록
        </button>
        <button class="btn btn-primary">저장</button>
      </div>
    </form>
  </div>
</template>
```

- form 관련은 PostForm.vue로 빼준다.

```js
//PostForm.vue
<template>
  <form>
    <div class="mb-3">
      <label for="title" class="form-label">제목</label>
      <input
        :value="title"
        @input="$emit('update:title', $event.target.value)"
        type="text"
        class="form-control"
        id="title"
      />
    </div>
    <div class="mb-3">
      <label for="content" class="form-label">내용</label>
      <textarea
         :value="content"
        @input="$emit('update:content', $event.target.value)"
        class="form-control"
        id="content"
        rows="3"
      ></textarea>
    </div>
    <div class="pt-4">
      <slot name="actions">
      </slot>
     </div>
  </form>
</template>

<script setup>
defineProps({
  title:String,
  content:String
})
defineEmits(['update:title' , 'update:content']);

</script>
```


- 빼둔 버튼은 slot으로 넣는다.
- 그럼 form을 줄여서 벌써 저만큼 바뀌었다.

```js
//PostCreateView.vue
<template>
  <div>
    <h2>게시판 등록</h2>
    <hr class="my-4">
    <PostForm 
      v-model:title="form.title"
      v-model:content="form.content"
      @submit.prevent="save"
    >
      <template #actions>
        <button
          type="button"
          class="btn btn-outline-dark me-2"
          @click="goListPage"
        >
        목록
        </button>
        <button class="btn btn-primary">저장</button>
      <template #actions>
    </PostForm>
  </div>
</template>
```

- pagination component도 분리해보자.
- 아래는 원본 pagination component다.

```js
<template>
	<div>
		<h2>게시글 목록</h2>
    <form @submit.prevent>
      <div class="row g-3">
        <div class="col">
          <input v-model="params.title_like" type="text" class="form-control">
        </div>
        <div class="col">
          <select v-model="params._limit" class="form-select">
            <option value="3">3개씩 보기</option>
            <option value="6">6개씩 보기</option>
            <option value="9">9개씩 보기</option>
          </select>
      </div>
    </form>
		<hr class="my-4" />
    <div class="row g-3">
      <div v-for=post in posts" :key="post.id" class="col-4">
        <PostItem
          :title="post.title"
          :content="post.content"
          :created-at="post.createdAt"
          @click="goPage(post.id)"
        ></PostItem>
      </div>
    </div>
    <nav aria-label="Page navigation example">
      <ul class="pagination justify-content-center">
        <li class="page-item" :class="{disabled :params._page > 1}">
          <a 
            class="page-link" 
            href="#" 
            aria-label="Previous"
            @click.prevent="--params._page">
            <span aria-hidden="true">$laquo;</span>
          </a>
        </li>
        <li v-for="page in totalPage" :key="page" class="page-tiem" :class="{active: params._page === page}"> //누르는 페이지에 active추가
          <a class="page-link" href="#" @click.prevent="params._page = page" >{{ page }}</a> 
        </li>
        <li class="page-item" :class="{disabled :params._page < totalPage}">
          <a 
            class="page-link" 
            href="#" 
            aria-label="Next"
            @click.prevent="++params._page">
            <span aria-hidden="true">$raquo;</span>
          </a>
        </li>
      </ul>
    </nav>
    <hr class="my-4" />
    <AppCard>
      <PostDetailView :id="previewId"></PostDetailView>
    </AppCard>
		</template>
	</div>
</template>

<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';
import { computed } from '@vue/reactivity';
import { useAxios } from '@/hooks/useAxios';

const router = useRouter();
const posts = ref([]);
const params = ref({
  _sort: 'createdAt',
  _order: 'desc',
  _limit: 3, //페이지 당 보여줄 갯수
  _page: 1, //페이지
  title_like: '' //검색어
});

//pagination
const totalCount = ref(0);
const totalPage = computed(() => Math.ceil(totalCount.value / params.value._limit)); //limit는 page 당 갯수

const fetchposts = async () => {
  try {
    const { data } = await getPosts(params.value);
    posts.value = data;
    totalCount.value = header['x-total-count']; //하이픈이 있어 대괄호로 접근해야 함. json -sersver에서 총량 줄 때 header에 저렇게 담아서 줌.
  } catch (error) {
    console.error(error);
  }
};


watchEffect(fetchposts); //fetchPost()를 넣으면 발동시킨 걸 넣는 거라 안됨.

const goPage = id => {
  router.push({
    name: 'PostDetail',
    params: {
      id,
    }
  })
};
</script>

<style lang="scss" scoped></style>
```


- pagination을 찢어줍니다.

```js
//paginationView.vue
<template>
  <nav aria-label="Page navigation example">
    <ul class="pagination justify-content-center">
      <li class="page-item" :class="isPrevpage">
        <a 
          class="page-link" 
          href="#" 
          aria-label="Previous"
          @click.prevent="$emit('page',currentPage - 1 )">
          <span aria-hidden="true">$laquo;</span>
        </a>
      </li>
      <li v-for="page in totalPage" :key="page" class="page-tiem" :class="{active: currentPage === page}"> //누르는 페이지에 active추가
        <a class="page-link" href="#" @click.prevent="$emit('page',page)">
        {{ page }}
        </a> 
      </li>
      <li class="page-item" :class="isNextpage">
        <a 
          class="page-link" 
          href="#" 
          aria-label="Next"
          @click.prevent="$emit('page',currentPage + 1)">
          <span aria-hidden="true">$raquo;</span>
        </a>
      </li>
    </ul>
  </nav>
</template>

<script setup>
defineProps({
  currentPage: {
    type: Number,
    required: true,
  },
  totalPage: {
    type: Number,
    required: true,
  }
});
defineEmits(['page']);

const isPrevpage = computed(() => ({ disabled: props.currentPage <= 1 });
const isNextpage = computed(() => ({disabled: props.currentPage > totalPage});
</script>

<style lang="scss" scoped></style>
```


- 기존의 component는 소스코드가 줄어들었다.

```js
<template>
	<div>
		<h2>게시글 목록</h2>
    <form @submit.prevent>
      <div class="row g-3">
        <div class="col">
          <input v-model="params.title_like" type="text" class="form-control">
        </div>
        <div class="col">
          <select v-model="params._limit" class="form-select">
            <option value="3">3개씩 보기</option>
            <option value="6">6개씩 보기</option>
            <option value="9">9개씩 보기</option>
          </select>
      </div>
    </form>
		<hr class="my-4" />
    <div class="row g-3">
      <div v-for=post in posts" :key="post.id" class="col-4">
        <PostItem
          :title="post.title"
          :content="post.content"
          :created-at="post.createdAt"
          @click="goPage(post.id)"
        ></PostItem>
      </div>
    </div>

    <AppPagination 
      :current-page="params._page" 
      :total-page="totalPage" 
      @page="page => (params._page = page) "
    /> 
    <template v-if="posts && posts.length > 0">
      <hr class="my-5" />
      <AppCard>
        <PostDetailView :id="posts[0].id"></PostDetailView>
      </AppCard>
		</template>
	</div>
</template>
```

- 위의 경우 posts[0].id가 number type이다.
- 따라서 내릴 때 props로 받을 때, type을 숫자로도 설정해줘야 한다.

```js
const props = defineProps({
  id: [Number, String]
})
```


- id가 숫자라면, psots.get의 argument를 백틱으로 바꿔 문자열로 변형하면 된다.

```js
//posts.get(id);
posts.get(`/${id}`);
```


- list도 분리해보자.
- list를 갖는 grid component를 먼저 만든다.

```js
//AppGrid.vue
<template>
  <div class="row g-3">
    <div v-for="(item, index) in items" :key="index" class="col-4">
      <slot :item="item" :index="index"></slot>
    </div>
  </div>
</template>

<script setup>
  defineProps({
    items: {
      type: Array,
      required: false,
    }
  })
</script>

<style lang="scss" scoped>

</style>
```

- grid를 적용하면 slot으로 받아온다.
- react에서 array의 element만큼 map으로 component를 만드는 것이랑 비슷해보인다.

```js
<template>
	<div>
		<h2>게시글 목록</h2>
    <form @submit.prevent>
      <div class="row g-3">
        <div class="col">
          <input v-model="params.title_like" type="text" class="form-control">
        </div>
        <div class="col">
          <select v-model="params._limit" class="form-select">
            <option value="3">3개씩 보기</option>
            <option value="6">6개씩 보기</option>
            <option value="9">9개씩 보기</option>
          </select>
      </div>
    </form>

		<hr class="my-4" />

    <AppGrid :items="posts">
      <template v-slot="{item}">
        <PostItem
          :title="item.title"
          :content="item.content"
          :created-at="item.createdAt"
          @click="goPage(item.id)"
        ></PostItem>
      </template>
    </AppGrid>

    <AppPagination 
      :current-page="params._page" 
      :total-page="totalPage" 
      @page="page => (params._page = page) "
    /> 
    <template v-if="posts && posts.length > 0">
      <hr class="my-5" />
      <AppCard>
        <PostDetailView :id="posts[0].id"></PostDetailView>
      </AppCard>
		</template>
	</div>
</template>
```

- filter도 분리해주자.
- 아래가 원본이다.

```js
<template>
	<div>
		<h2>게시글 목록</h2>
    <form @submit.prevent>
      <div class="row g-3">
        <div class="col">
          <input v-model="params.title_like" type="text" class="form-control">
        </div>
        <div class="col">
          <select v-model="params._limit" class="form-select">
            <option value="3">3개씩 보기</option>
            <option value="6">6개씩 보기</option>
            <option value="9">9개씩 보기</option>
          </select>
      </div>
    </form>

		<hr class="my-4" />

    <AppGrid :items="posts">
      <template v-slot="{item}">
        <PostItem
          :title="item.title"
          :content="item.content"
          :created-at="item.createdAt"
          @click="goPage(item.id)"
        ></PostItem>
      </template>
    </AppGrid>

    <AppPagination 
      :current-page="params._page" 
      :total-page="totalPage" 
      @page="page => (params._page = page) "
    /> 
    <template v-if="posts && posts.length > 0">
      <hr class="my-5" />
      <AppCard>
        <PostDetailView :id="posts[0].id"></PostDetailView>
      </AppCard>
		</template>
	</div>
</template>
```

- 필터링을 위한 component를 분리했다.

```js
//PostFilter.vue
<template>
  <form @submit.prevent>
    <div class="row g-3">
      <div class="col">
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

          <option value="3">3개씩 보기</option>
          <option value="6">6개씩 보기</option>
          <option value="9">9개씩 보기</option>
        </select>
    </div>
  </form>
</template>

<script setup>
defineProps({
  title: String,
  limit: Number,
});

defineEmits(['update:title','update:limit']);
</script>

<style lang="scss" scoped></style>
```


- 분리한 component를 아래와 같이 넣어준다.

```js
<template>
	<div>
		<h2>게시글 목록</h2>
    <PostFilter 
      v-model:title="params.title_like" 
      v-model:limit="params._limit" 
    />

		<hr class="my-4" />

    <AppGrid :items="posts">
      <template v-slot="{item}">
        <PostItem
          :title="item.title"
          :content="item.content"
          :created-at="item.createdAt"
          @click="goPage(item.id)"
        ></PostItem>
      </template>
    </AppGrid>

    <AppPagination 
      :current-page="params._page" 
      :total-page="totalPage" 
      @page="page => (params._page = page) "
    /> 
    
    <template v-if="posts && posts.length > 0">
      <hr class="my-5" />
      <AppCard>
        <PostDetailView :id="posts[0].id"></PostDetailView>
      </AppCard>
		</template>
	</div>
</template>
```