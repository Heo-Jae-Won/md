- 아래는 vue2의 vuex3다.
- 당연히 vue3의 vuex4와는 문법이 다르다.
- store는 store.js로 따로 선언한다. vue3의 vuex4는 개별 store를 createStore로 선언한다.
```js
//src/store/store.js
.
.
.
.
import map from './modules/map'; //index.js는 생략가능.
Vue.use(Vuex);

export const store = new Vuex.Store({
  modules: {
      auth,
      chat,
      settings,
      ecommerce,
      mail,
      sidebar,
      sample,
      user,
      profile,
      admin,
      project,
      department,
      deadLine,
      expenses,
      commonCode,
      corporate,
      rent,
      asset,
      confRoomReserve,
      map,
  },
  plugins: [createPersistedState({
      paths: ["auth"],
  })],
})
```

- store는 아래와 같이 state, actions, mutations로 이뤄진다.
- state는 중앙저장소에 둘, 공유변수다.
- actions는 주로 api 등 비동기 함수용 method다.
- mutations는 주로 동기용 함수 method다.

```
state에 선언한다.
actions에 함수를 concise method로 선언한다.
첫 argument는 무조건 context, 두번째 인수는 payload다.
return은 보통 axios instance다. axios instance에서 then chaining을 한다. 그 안에서 commit을 한다. commit은 mutation 함수를 부르는 것이다. 보통 mutation은 state를 저장하는 역할이다.
then에서 res를 받으면 그 res를 mutation으로 넘긴다. 
```

- 코드로 살펴보자.
- state에 선언한다.

```js
const state = {
    memberAddressList: [],
    memberPageCount: {},
    memberListSize: {},
}
```
- actions에 함수를 concise method로 선언한다.
```js
const actions = {
    getMemberAddressList() {
    }
}
```
- 첫 argument는 무조건 context(store와 동일), 두번째 인수는 payload다.
```js
const actions = {
    getMemberAddressList(context, pagenationData) {
    }
}
```
- return은 보통 axios instance다. axios instance에서 then chaining을 하면 그 안에서 commit을 한다. commit은 mutation 함수를 부르는 것이다.
- 보통 mutation은 state를 저장하는 역할이다.
```js
const actions = {
    getMemberAddressList(context, pagenationData) {
        return mapApi.getMemberAddressList(pagenationData).then(res => {
            context.commit('getMemberAddressListSuccess', res); //mutation에 getMemberAddressListSuccess를 불러온다.
        }).catch(error => {
            console.log('getMemberAddressListError', error);
        })
    },
}

const mutations = {
    getMemberAddressListSuccess(state, res) {
        if (res.resultCode == '0000') {
            state.memberAddressList = res.data.memberAddressList;
        }
    },
}
```

- 다 쓰면 아래와 같다.
```js
//src/modules/map/index.js


import { mapApi } from 'Api';
//import Vue from 'vue';

const state = {
    memberAddressList: [],
    memberPageCount: {},
    memberListSize: {},
}

const actions = {
    getMemberAddressList(context, pagenationData) {
        return mapApi.getMemberAddressList(pagenationData).then(res => {
            context.commit('getMemberAddressListSuccess', res);
        }).catch(error => {
            console.log('getMemberAddressListError', error);
        })
    },
    getMemberListSize(context, pagenationData) {
        return mapApi.getMemberListSize(pagenationData).then(res => {
            context.commit('getMemberListSizeSuccess', res);
        }).catch(error => {
            console.log('getMemberListSizeError', error);
        })
    },
}

const mutations = {
    getMemberAddressListSuccess(state, res) {
        if (res.resultCode == '0000') {
            state.memberAddressList = res.data.memberAddressList;
        }
    },
    getMemberListSizeSuccess(state, res) {
        if (res.resultCode == '0000') {
            state.memberListSize = res.data.memberListSize;
            state.memberPageCount = res.data.memberPageCount;
        }
    },
}
export default {
    namespaced: true, //독립적인 모듈 작성을 위해 모듈을 매핑할 때 namespaced: true 옵션을 줘서 네임스페이스를 지정할 수 있다. 필수.
    state,
    actions,
    mutations,
}
```

- store가 완성되었으므로 이제 실제 component에서 사용하면 된다.
- action을 부를 때는 dispatch로 부른다.
- 또한 store를 부를 때도 this.store가 아니라 this.$store다.
- this.pagenationData가 위 actions의 getMemberAddressList함수의 두번째 인수인 pagenationData가 된다.
```js
//src/views/map/components/MapByAddress.vue
.
.
.
methods: {
    getMemberAddressList() {
        this.$store.dispatch('map/getMemberAddressList', this.pagenationData).then(() => { 
            this.memberList = this.$store.state.map.memberAddressList;
            this.memberListLoading = false;
            this.selected = []
            this.dialog = false;
        });
    },
}
```

- 위를 보면 dispatch의 첫 인수는 type인데, getMemberAddressList가 아니라 map/getMemberAddressList인 이유는 store 이름이 map이기 때문이다.
```js
import map1 from './modules/map'; //index.js는 생략가능.
Vue.use(Vuex);

export const store = new Vuex.Store({
    modules: {
        map1,
  },
  plugins: [createPersistedState({
      paths: ["auth"],
  })],
})
```

- 만약 아래와 같이 map1으로 바꾸면 map1으로 dispatch type도 바뀐다.
- state에서 가져올 때도 map1으로 바꿔줘야 한다.
```js
methods: {
    getMemberAddressList() {
        this.$store.dispatch('map/getMemberAddressList', this.pagenationData).then(() => { 
            this.memberList = this.$store.state.map1.memberAddressList;
            this.memberListLoading = false;
            this.selected = []
            this.dialog = false;
        });
    },
}
```



