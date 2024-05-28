- json server를 깔면 api 값을 배열이나 객체로 만들어서 테스트가 가능하다.

```js
npm i -g json-server

npx json-server --watch db.json

package.json에 db로 json-server --watch db.json --port [번호]복사해 넣기

그 뒤부터는 npm run db 로 json 서버 작동

front 서버 작동은 또 따로 해줘야함.

delay를 주고 싶다면 json-server --watch db.json --port [번호] --delay 1000 로 1초 delay 주기 가능
```

- 설정 완료하면 아래와 같이 두 개의 서버를 모두 실행시킨다.

```js
npm run dev
npm run db
```

- axios를 사용하게 되면 아래와 같이 받을 수 있다.

```js
//axios.js
function getposts() {
    return axios.get(`http:..localhost:5000/posts`);
}

//PostListView.vue
import { getPosts } from './axios.js'
const posts = ref([]);

const fetchPosts = () => {
    getPosts().then( response => {
        conosle.log('response: ', response);
        posts.value = response.data;
    }).catch(error => {
        console.log(error);
    })
}
```


- promise then, catch 대신 async, await를 사용해도 된다.

```js
import { getPosts } from './axios.js'
const posts = ref([]);

const fetchPosts = async () => {
    const response = await getPosts();
    console.dir(response);
}
```

- axios를 async로 받아오면 다양한 방식으로 받아올 수 있다.

```js
const fetchPosts = async () => {
    const { data } = await getPosts();
    posts.value = data;
    console.dir(response);
}

const fetchPosts = async () => {
    const { data: posts.value = data } = await getPosts();
    console.dir(response);
    
}

const fetchPosts = async () => {
    ({ data: posts.value = data } = await getPosts());
    console.dir(response);
}
```

- async의 좋은 점은 try - catch를 사용할 수 있다는 점이다.

```js
const form = ref({
    title: null,
    content: null
});

const createPost = (data) => {
   return axios.post(`http:..localhost:5000/posts`, data);
};

const save = () => {
    try {
        createPost({
            ...form.value,
            createdAt: Date.now(),
        })
    } catch (error) {
        console.error(error);
    }
};

```
