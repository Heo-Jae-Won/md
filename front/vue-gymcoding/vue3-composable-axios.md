- axios를 추상화해 사용할 수 있는 로직을 만든다.

```js
//useAxios.js
import axios from 'axios';

axios.defaults.baseURL = import.meta.env.VITE_APP_API_URL;

export const useAxios = (url, config = {}) => { //config를 {}로 할당해놔야 desturcturing 시 error X
  const data = ref([]);
  const error = ref(null);
  const loading = ref(false);


  loading.value = true;

  axios(url, config)
    .then(res => {
      data.value = res.data;
    }).catch(err => {
      error.value = err;
    }).finally(() => {
      loading.value = false;
    })

  try {
    loading.value = true;
    const { data, headers } = await getPosts(params.value);
    posts.value = data;
    totalPage.value = headers['x-total-count'];
  } catch( err ) {
    error.value = err;
  } finally {
    loading.value = false;
  }

  return {
    data,
    error,
    loading,
  }
}
```


- 

```js
//PostListView.vue
const params = ref({
	_sort: 'createdAt',
	_order: 'desc',
	_page: 1,
	_limit: 6,
	title_like: '',
});

const {
	response,
	data: posts,
	error,
	loading,
} = useAxios('/posts', { method: 'get', params });
```


- proxy로 넘어와서 config가 제대로 인식되지 않는다.
- params를 비구조할당으로 가져와서 unref한다.
- 확장성있게 data를 활용가능하게 response 전체를 넘긴다.

```js
//useAxios.js
import axios from 'axios';

axios.defaults.baseURL = import.meta.env.VITE_APP_API_URL;

export const useAxios = (url, config = {}) => { //config를 {}로 할당해놔야 desturcturing 시 error X
  const data = ref([]);
  const response = ref(null);
  const error = ref(null);
  const loading = ref(false);

  const { params} = config;


  loading.value = true;

  axios(url, {
    ...config,
    params: unref(params),
  })
    .then(res => {
      response.value = res;
      data.value = res.data;
    }).catch(err => {
      error.value = err;
    }).finally(() => {
      loading.value = false;
    })

  try {
    loading.value = true;
    const { data, headers } = await getPosts(params.value);
    posts.value = data;
    totalPage.value = headers['x-total-count'];
  } catch( err ) {
    error.value = err;
  } finally {
    loading.value = false;
  }

  return {
    config,
    data,
    error,
    loading,
  }
}
```

- response를 받아올 때도 response를 추가해준다.
- 그런데 paging이 실행되지 않는다.
- useAxios에서는 watchEffect logic이 없기 때문이다.


```js
//PostListView.vue
const {
  response,
  data: posts,
  error,
  loading
} = useAxios('/posts', { method: 'get', params});

const totalCount = computed(() => response.value.headers['x-total-count']);

```

- 따라서 params가 변경되었을 때 다시 실행되게 useAxios()를 변경해주자.
- default method를 get으로 추가해주자.

```js
//useAxios.js
import axios from 'axios';

axios.defaults.baseURL = import.meta.env.VITE_APP_API_URL;

const defaultMethodConfig = {
  method: 'get'
}

export const useAxios = (url, config = {}) => { //config를 {}로 할당해놔야 desturcturing 시 error X
  const data = ref([]);
  const response = ref(null);
  const error = ref(null);
  const loading = ref(false);

  const { params } = config;

  loading.value = true;

  const execute = () => {  
    data.value = null;
    error.value= null;
    loading.value = true;
    
     axios(url, {
      ...defaultMethodConfig,
      ...config,
      params: unref(params),
    })
      .then(res => {
        response.value = res;
        data.value = res.data;
      }).catch(err => {
        error.value = err;
      }).finally(() => {
        loading.value = false;
      })

    }
    
   

  try {
    loading.value = true;
    const { data, headers } = await getPosts(params.value);
    posts.value = data;
    totalPage.value = headers['x-total-count'];
  } catch( err ) {
    error.value = err;
  } finally {
    loading.value = false;
  }

  watchEffect(execute);

  return {
    config,
    data,
    error,
    loading,
  }
}
```

- params를 object로 넘기든, reactivity 변수로 넘기든 상관없이 정상작동하게 처리하자.
- isRef 조건문을 추가해준다.

```js
//useAxios.js

.
.

const execute = () => {
  .
  .

}

if (isRef(params)) {
  watchEffect(execute);
} else {
  execute();
}
```


- 만든 axios 함수는 다른 view에서도 사용가능하다.

```html
<!-- PostDetailView.vue -->

<script setup>

/
/
/
const { error, loading, data: post } = useAxios(`/posts/${props.id}`);

// const fetchPost = async () => {
//   try {
//     loading.value = true;
//     const { data }  = await getPostsById(props.id);
//     setPost(data);
//   } catch( err ) {
//     error.value = err;
//   } finally {
//     loading.value = false;
//   }
// };

// const setPost = ({title, content, createdAt}) => {
//   post.value.title = title;
//   post.value.content = content;
//   post.value.createdAt = createdAt;
// }

// fetchPost();
```
  



- 등록화면에서도 쓰일 수 있다.

```js
const save = async () => {
  const { error: error.value, loading } = useAxios('/posts', {
    method:'post'},
    data:{ ...form.value, createdAt: Date.now() });
}
//비구조 할당 하여 아래와 같이 받을 수있다.

const save = async () => {
  ({ error: error.value, loading: loading.value } = useAxios('/posts', {
    method:'post',
    data:{ ...form.value, createdAt: Date.now() }
  }));
}

// 원본은 아래와 같다.
const save = async () => {
	try {
		loading.value = true;
		await createPost({
			...form.value,
			createdAt: Date.now(),
		});
		router.push({ name: 'PostList' });
		vSuccess('등록이 완료되었습니다!');
	} catch (err) {
		vAlert(err.message);
		error.value = err;
	} finally {
		loading.value = false;
	}
};
```

- 그런데 alert 내는 처리를 async 내에서 할수가 없어서 callback으로 useAxios에 넘겨야 한다.
- onError, onSuccess callback을 넘겨주자.

```js
//useAxios.js
import axios from 'axios';

axios.defaults.baseURL = import.meta.env.VITE_APP_API_URL;

const defaultMethodConfig = {
  method: 'get'
}

export const useAxios = (url, config = {}, options = {}) => { //config를 {}로 할당해놔야 desturcturing 시 error X
  const data = ref([]);
  const response = ref(null);
  const error = ref(null);
  const loading = ref(false);

  const { onSuccess, onError } = options;
  const { params } = config;

  loading.value = true;

  const execute = () => {  
    data.value = null;
    error.value= null;
    loading.value = true;
    
     axios(url, {
      ...defaultMethodConfig,
      ...config,
      params: unref(params),
    })
      .then(res => {
        response.value = res;
        data.value = res.data;
        if (onSuccess) {
          onSuccess(res);
        }
      }).catch(err => {
        error.value = err;
        if (onError) {
          onError(err);
        }
      
      }).finally(() => {
        loading.value = false;
      })

    }
    
   

  try {
    loading.value = true;
    const { data, headers } = await getPosts(params.value);
    posts.value = data;
    totalPage.value = headers['x-total-count'];
  } catch( err ) {
    error.value = err;
  } finally {
    loading.value = false;
  }

 if (isRef(params)) {
    watchEffect(execute);
  } else {
    execute();
  }

  return {
    config,
    data,
    error,
    loading,
  }
}
```

- 그럼 아래와 같이 save 함수에 callback options가 추가된다.
- 다만 이 경우, 잔상이 생긴다.

```js
const save = async () => {
  ({ error: error.value, loading: loading.value } = useAxios('/posts', {
    method:'post',
    data:{ ...form.value, createdAt: Date.now() }
  },
  {
    onSuccess: () => {
      router.push({name:'PostList'});
      vSuccess('등록이 완료되었습니다!');
    },
    onError: () => {
      vAlert(err.message);
    }
  }));
}
```

- 잔상을 제거하기 위해 함수를 바꿔준다.
- error와 loading을 save 안에 두지 않고 밖으로 뺀다.
- data는 save 버튼을 눌렀을 때 채워지는 방식으로 변경된다.


```js
const { error, loading } = useAxios('/posts', {
    method:'post',
    data:{ ...form.value, createdAt: Date.now() }
  },
  {
    immediate: false,
    onSuccess: () => {
      router.push({name:'PostList'});
      vSuccess('등록이 완료되었습니다!');
    },
    onError: () => {
      vAlert(err.message);
    }
  })

const save = async () => {
  execute({...form.value, createdAt: Date.now()}
  )
}
```


- immediate 속성을 반영할 수 있게 useAxios()를 변경해준다.
- immediate가 true일 때만 axios를 호출한다.

```js
//useAxios.js
import axios from 'axios';

axios.defaults.baseURL = import.meta.env.VITE_APP_API_URL;

const defaultMethodConfig = {
  method: 'get'
}

const defaultOptions = {
  immediate: true
}

export const useAxios = (url, config = {}, options = {}) => { //config를 {}로 할당해놔야 desturcturing 시 error X
  const data = ref([]);
  const response = ref(null);
  const error = ref(null);
  const loading = ref(false);

  const { onSuccess, onError, immediate } = {
    ...defaultOptions,
    ...options
  };

  const { params } = config;

  loading.value = true;

  const execute = body => {  
    data.value = null;
    error.value= null;
    loading.value = true;
    
     axios(url, {
      ...defaultMethodConfig,
      ...config,
      params: unref(params),
      data: body
    })
      .then(res => {
        response.value = res;
        data.value = res.data;
        if (onSuccess) {
          onSuccess(res);
        }
      }).catch(err => {
        error.value = err;
        if (onError) {
          onError(err);
        }
      
      }).finally(() => {
        loading.value = false;
      })

    }
    
   

  try {
    loading.value = true;
    const { data, headers } = await getPosts(params.value);
    posts.value = data;
    totalPage.value = headers['x-total-count'];
  } catch( err ) {
    error.value = err;
  } finally {
    loading.value = false;
  }

  if (isRef(params)) {
    watchEffect(execute);
  } else {
    if (immediate) {
      execute();
    } 
  }

  return {
    config,
    data,
    error,
    loading,
    execute
  }
}
```


- post에서 body는 객체로 넘긴다. 객체가 아닌 경우 기본값으로 빈 객체를 주도록 하자.

```js
//useAxios.js
.
.
.
axios(url, {
  ...defaultMethodConfig,
  ...config,
  params: unref(params),
  data: typeof body === 'object' ? body : {},
})
```

- 이젠 수정도 바꿔보자.

```js
//기존
const editError = ref(null);
const editLoading = ref(false);
const edit = async () => {
  try {
    editLoading.value = true;
    await updatePost(id, {...form.value});
    router.push({name: 'PostDetail', params: {id}});
    vSuccess('수정이 완료되었습니다.');
  } catch (err) {
    vAlert(err.rmessage);
    editError.value = err;
  } finally {
    editLoading.value = false;
  }
}

//useAxios 적용
const {
  error: editError,
  loading: editLoading,
} = useAxios(`/posts/${id}`, {
  method: 'patch'
  },
  {
    immediate: false,
    onSuccess: () => {
      vSuccess('수정이 완료되었습니다.');
      router.push({name: 'PostDetail', params: {id}});
    },
    onError: err => {
      vAlert(err.message);
    }
  }
);

const edit = () => {
  exeucte({
    ...form.value
  })
}
```


- composable 함수를 만들 때는, ref 파라미터가 들어올 수 있기에, ref인 경우 처리가 필요하다.

```js
function useFeacture(maybeRef) {
  //만약 mayRef가 실제로 ref라면, .value를 반환한다.
  //그렇지 않으면 있는 그대로 반환한다.
  const value = unref(maybeRef);
}
```


- 또한 구조분해할당 사용이 가능하도록 reactive를 쓰지 않고 ref를 쓰는 것이 바람직하다.
- 아래와 같이 자주 구조분해 할당된다.

```js
const { x, y } = useMouse();
```


- composable함수에서 반환된 것을 변수로 destructuring 하지 않고 싶을 수 있다.
- 그 때는 reactive로 묶은 뒤에 객체의 속성을 꺼내야 reactivity를 유지한다.

```js
// mouse.x is linked to original ref
const mouse = reactive(useMouse());
console.log(mouse.x);

// mouse.x lose link to original ref
const mouse = useMouse();
console.log(mouse.x);
```