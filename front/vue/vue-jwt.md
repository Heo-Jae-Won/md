## <span style="color:#802548">_1. pinia 설치_</span>
- 우선 pinia라는 중앙집중형 관리소를 설치해준다. 
- 새로고침에도 값이 날라가지 않게 persistedstate plugin도 같이 설치한다.
```
npm i pinia
npm i pinia-plugin-persistedstate
```
설치했다면 아래와 같이 설정해준다.

```js
//main.ts
createApp(App)
    .use(createPinia()
    .use(piniaPluginPersistedState))
    .use(router).use(vuetify).mount('#app');
```
## <span style="color:#802548">_2. store 정의와 스토어 내 변수 설정_</span>
- pinia는 아래와 같이 기본 틀이 구성된다. component setup() 문법과 매우 유사하다.
```js
defineStore([storename],
()=>{
     return {}
   },{
option
}
)
```
- 나는 아래와 같이 썼다. 
```js
//member.module.ts
export const useMemberStore = defineStore(
  'memberStore',
  () => {
    const loginMember = ref('');
    const accessToken = ref('');
    const refreshToken = ref('');
    const getLoginMember = computed(() => loginMember.value);

    function changeLoginMember(memberId: string, access: string, refresh: string) {
      loginMember.value = memberId;
      accessToken.value = access;
      refreshToken.value = refresh;
    }

    function updateAccessToken(access: string) {
      accessToken.value = access;
    }

    function resetLoginMember() {
      loginMember.value = '';
      accessToken.value = '';
      refreshToken.value = '';
    }

    return { loginMember, getLoginMember, changeLoginMember, accessToken, refreshToken, resetLoginMember, updateAccessToken };
  },
  {
    persist: true,
  }
);
```
- persist 옵션을 true로 설정해야 새로고침을 해도 pinia의 값이 날라가지 않는다.
## <span style="color:#802548">_3. store 내 변수를 component에 꺼내 쓰기_</span>
- store에 있는 변수를 활용하려면, 함수의 경우에는 store에서 destructuring을 하면 된다. 
- 함수는 반응성이 없기 때문이다. 반면에 reactivity를 가진 변수들은 아래와 같이 storeToRefs(store)를 통해 반응성을 가진 상태를 유지시켜 사용한다.

```js
//*.vue
const store = useMemberStore();
const { changeLoginMember } = store;
 const { loginMember } = storeToRefs(store);
axios로 api를 호출해 로그인이 성공하면 그 결과값을 가져와 accessToken과 refreshToken에 넣고, store에 값을 저장한다.

//로그인을 하여 JWT를 반환한다
    async function handleLoginClick() {
      const isValid = (await useValidateForm(form, isFormValid))?.value;
      if (isValid) {
        const data = {
          memberId: memberId.value,
          memberPassword: memberPassword.value,
        };

        const result = (await login(data)).data;
        const accessToken = result.accessToken;
        const refreshToken = result.refreshToken;

        alert('로그인이 완료되었습니다.');
        changeLoginMember(memberId.value, accessToken, refreshToken);
        router.push('/');
      }
    }
```
- 저장해 둔 값은 아래와 같이 꺼내서 사용할 수 있다. 
- template에서도 활용하려면 늘 그렇듯 return문 안에 포함시켜줘야 한다.
```html
<v-toolbar-title v-if="loginMember !== ''" @click="$router.push(`/myInfo/` + loginMember)">
    {{ loginMember }}
</v-toolbar-title>
```
```js
//<script lang="ts>
export default defineComponent({
setup(){
  const { loginMember } = storeToRefs(store);
.
.
    return {
      loginMember,
.
.
    }
  },
})
//<script>
```
## <span style="color:#802548">_4. axios interceptor 활용_</span>
​

- accessToken은 매 api마다 보내져야 하므로 아래와 같이 request interceptor에서 가로채서 header에 accessToken값을 추가하여 request를 전송한다. 
- store는 전역으로 규정될 수 없고 아래와 같이 블록 안에서 규정되어야 한다. 그렇지 않으면 오류가 난다.
```java
instance.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    if (config.url !== '/login') {
      const store = useMemberStore();
      const { accessToken } = storeToRefs(store);

      config.headers['authorization'] = `Bearer ${accessToken.value}`;
    }

    return config;
  },
  error => {
    return Promise.reject(error);
  }
);
```
- accessToken이 30분짜리라면, 30분 이후에는 만료처리되어 사용이 불가능하다. 
- 따라서 refreshToken을 통해 갱신이 필요하다. refreshToken을 다시 보내는 것은 response로 401이 내려올 때 해주면 된다. 
- 로직은 사람마다 다를 것이다. 내 경우에는 Spring redis를 통해 memberId를 key로 하여 refreshToken을 저장했기 때문에 memberId를 인자로 받은 것이다. 
```js
const store = useMemberStore();
const { loginMember } = storeToRefs(store);
const { updateAccessToken } = store;
const authenticPrevRequest = prevRequest;
const accessToken = (await extendSignInStatus(loginMember.value)).data;
```
- 그렇다면 이전의 request를 어떻게 기억하는 것일까? 따로 값을 저장해두어야 한다. 
- 전역변수로 request를 초기화시킨 상태로 설정하고 request 시에 값을 저장해 둔다. 
```java
let prevRequest: AxiosRequestConfig | null = null;

instance.interceptors.request.use(
  (config: AxiosRequestConfig) => {
.
.
    prevRequest = config;
    return config;
  },
  error => {
    return Promise.reject(error);
  }
);
```
- 그리고 refreshToken을 받아오는 api를 호출하기 전에 이전의 request를 상수로 저장하여 변경이 불가능하게 바꾼다.
- refreshToken 이후에 이전 request를 저장하면 늘 refreshToken을 호출하는 api가 저장되므로 순서를 뒤바꿔서 써야 한다.
```js
const accessToken = (await extendSignInStatus(loginMember.value)).data;
const authenticPrevRequest = prevRequest;
//뒤바뀐 순서가 바로 아래이다.

instance.interceptors.response.use(
  response => {
    return response;
  },
  async (error: AxiosError) => {
    if (error.response?.status === 401) {
      const store = useMemberStore();
      const { loginMember } = storeToRefs(store);
      const { updateAccessToken } = store;
      const authenticPrevRequest = prevRequest;
      const accessToken = (await extendSignInStatus(loginMember.value)).data;
    }
  }
)
```
- accessToken을 store에서 갈아 끼운다. 이번 재요청되는 request 이후부터 오게 되는 request는 갈아끼워진 accessToken을 갖고 backend로 날아가게 된다.

## updateAccessToken(accessToken);
- 우리는 갈아끼운 store의 토큰을 가져올 필요 없이 바로 axios에서 받은 accessToken을 header에 주입하여 사용할 수 있다. 
- content-type도 application/json으로 넣어준다. 
- 이유는 모르겠으나 axios로 처음 보낼 때는 application/json 설정으로 가는데, refreshToken을 호출한 이후 전송되는 재요청의 경우 content-type이 자꾸x-www-urlencoded로 가게되어 아래와 같이 content-type을 설정했다. 
- 다만 문제가 multipart/form-data의 경우 어떻게 대처해야 할 지 고민해야 할 것 같다.
- accessToken과 content-type을 갈아낀 request를 다시 axios로 호출하여 result를 저장한다.
```js
const updatedRequestConfig = {
        ...authenticPrevRequest, 
        headers: {
          authorization: `Bearer ${accessToken}`,
          'content-type':'application/json'
        },
      };
const result = await axios(updatedRequestConfig as AxiosRequestConfig);
  return result;
  ```
- 아래가 합본이다. 
```js
instance.interceptors.response.use(
  response => {
    return response;
  },
  async (error: AxiosError) => {
    if (error.response?.status === 401) {
      const store = useMemberStore();
      const { loginMember } = storeToRefs(store);
      const { updateAccessToken } = store;
      const authenticPrevRequest = prevRequest;
      const accessToken = (await extendSignInStatus(loginMember.value)).data;
      updateAccessToken(accessToken);
      const updatedRequestConfig = {
        ...authenticPrevRequest, 
        headers: {
          authorization: `Bearer ${accessToken}`,
          'content-type':'application/json'
        },
      };

      const result = await axios(updatedRequestConfig as AxiosRequestConfig);
      return result;
    }

    return Promise.reject(error);
  }
);
​```