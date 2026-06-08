
- reactive로 선언하면 destructuring이 불가능하다.

```js
let form = reactive({});
const fetchPost = () => {
    const data = getPostsById(id);
    //form.value = { ...data };
    form.title = data.title;
    form.content = data.content;
}
```


- 반면에 ref로 선언하면 destructuring이 가능하다.
- paging component는 보통 ref로 선언하는 편이다.

```js
let form = ref({});
const fetchPost = () => {
    const data = getPostsById(id);
    form.value = { ...data };
    //form.title = data.title;
    //form.content = data.content;
}
```


- 보통 router에서 query parameter를 전달받을 때는 useRoute()를 사용한다.

```js
//index.js
{
    path: '/posts/:id',
    name: 'PostDetail',
    component: PostDetailView
}


//PostDetailView.vue
~~~
<script setup>
const route = useRoute();
const router = useRouter();
const id = route.params.id;

const goListPage = () => router.push({name: 'PostList'});

const goEditPage = () => router.push({name: 'PostEdit', params: {id}});
</script>
```


- useRoute()를 쓰지 않고 router를 정의한 곳에서 props로 내릴 수도 있다.
- props를 true로 켜주고, query parameter에 쓰인 문자열을 defineProps에 정의한다.
- defineProps에 정의하는 type들은 모두 wrapper type이어야 한다.

```js
//index.js
{
    path: '/posts/:id',
    name: 'PostDetail',
    component: PostDetailView,
    props: true
}


//PostDetailView.vue
~~~
<script setup>
const props = defineProps({
    id: String,
})
const router = useRouter();
const goListPage = () => router.push({name: 'PostList'});

const goEditPage = () => router.push({name: 'PostEdit', params: {props.id}});
</script>
```


- 함수 형태로 return 할 수도 있다.

```js
//index.js
{
    path: '/posts/:id',
    name: 'PostDetail',
    component: PostDetailView,
    props: router => {
        return {
            id: parseint(route.params.id),
        }
    }
}
```

- 짧게 문장을 가져가고 싶다면 축약할 수 있다.
- {}로 열면 block 첫 start로 해석될 수 있으니, 객체를 return함을 보이기 위해 ()로 감싼다.

```js
//index.js
{
    path: '/posts/:id',
    name: 'PostDetail',
    component: PostDetailView,
    props: router => ({
            id: parseInt(route.params.id),
    })
}

//PostDetailView.vue
~~~
<script setup>
const props = defineProps({
    id: Number,
})
```


