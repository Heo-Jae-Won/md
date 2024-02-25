## <span style="color:#802548">_1. axios_</span>

- 나는 아래와 같이 axios를 만들었다.


```js
//src/utils/axios/instance.ts
import { useMemberStore } from '@/module/member.module';
import axios, { AxiosError, AxiosRequestConfig } from 'axios';
import { storeToRefs } from 'pinia';
import { extendSignInStatus } from './api';

const apiRootPath = process.env.VUE_APP_API;
let prevRequest: AxiosRequestConfig | null = null;
export const instance = axios.create({
  timeout: 10 * 1000,
  baseURL: apiRootPath,
});

instance.interceptors.request.use(
  (config: AxiosRequestConfig) => {
    if (config.url !== '/login') {
      const store = useMemberStore();
      const { accessToken } = storeToRefs(store);

      config.headers['authorization'] = `Bearer ${accessToken.value}`;
    }

    prevRequest = config;
    return config;
  },
  error => {
    return Promise.reject(error);
  }
);

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
      const accessTokenResponse = await extendSignInStatus(loginMember.value);
      const accessToken = accessTokenResponse.data;
      updateAccessToken(accessToken);
      const updatedRequestConfig = {
        ...authenticPrevRequest,
        headers: {
          authorization: `Bearer ${accessToken}`,
          'content-type': 'application/json',
        },
      };

      const result = await axios(updatedRequestConfig as AxiosRequestConfig);
      return result;
    }

    if (error.response?.data.message) {
      alert(error.response?.data.message);
    }

    return Promise.reject(error);
  }
);
```

- 그리고 axios call을 모아 둔 file을 별도로 관리했다.
- return instance를 보면 알 수 있다.
```js
import { Member } from '@/model/member.model';
import { PayRequest } from '@/model/pay.model';
import { SchedulePaginationRequest, ScheduleRequest, ScheduleResponse } from '@/model/schedule.model';
import { Seat } from '@/model/seat.model';
import { instance } from './instance';
import { Refund } from '@/model/refund.model';

//로그인
export const login = (loginRequest: Member) => {
  return instance({
    url: '/signin',
    method: 'post',
    data: loginRequest,
  });
};

//로그아웃
export const logout = (memberNo: string) => {
  return instance({
    url: '/logout',
    method: 'post',
    data: memberNo,
    headers: {
      'content-type': 'application/json',
    },
  });
};

//회원가입
export const saveMember = (formData: Member) => {
  return instance({
    url: '/signup',
    method: 'post',
    data: formData,
  });
};
.
.
.

```

- 그 뒤 정보를 이용하려면 아래와 같다.
```js
const scheduleStore = useScheduleStore();
const headcountStore = useHeadcountStore();
const { changeScheduleRequest } = scheduleStore;
const { changeHeadcount } = headcountStore;
function handleMoveWithData() {
    changeScheduleRequest(scheduleInfo.value);
    changeHeadcount(headcountInfo.value);
    router.push(`/schedule`);
}
```

- 반면에 calendar에서는 사용법이 좀 달랐다.
- 개인적으로는 내 방법이 더 낫다고 생각된다.
- calendar 것은 너무 잘게 쪼개져있다.

```js
//src/common/config/config.js
export default {
    BASE_URL: '/api',
    USER_URL: '/user/',
    AUTH_URL: '/user/auth/',
    TEST_URL: '/test/',
    CODE_URL: '/codemanage/',
    TERMS_URL: '/terms/',
    COMMON_URL: '/common/',
    NOTICE_URL: '/notice/',
    FAQ_URL: '/faq/',
    ONEONQ_URL: '/oneonq/',
    DORMANT_URL: '/dormantmember',
    EXCEPTION_URL: '/exception',
    MEMBER_URL: '/member',
    PARTNER_URL: '/partner',
    POINT_URL: '/point/',
    ADMINACCOUNT_URL: '/adminaccount/',
    MENUATH_URL: '/menuath/',
    MENU_URL: '/menu/',
    PROFILE_URL: '/profile/',
    ADMIN_URL: '/admin/',
    PROJECT_URL: '/project/',
    DEPARTMENT_URL: '/department/',
    DEADLINE_URL: '/deadLine/',
    EXPENSES_URL: '/expenses/',
    CORP_URL:'/corpExpenses/',
    RENT_URL:'/rent/',
    ASSET_URL: '/asset/',
    HIWORKS_URL: '/hiworks/',  
    CALENDAR_URL: '/calendar/',
    MAP_URL: '/map/'
}
```

- axios call을 하는 전용 class를 만들어서 쓰고 있다.
```js
//src/common/commonApi.js
import axios from 'axios';
import { store } from '~/store/store';
import router from '~/router';
import commonConfig from '~/api/common/config/config';

class apiCommon {
    constructor(path) {
        this.init(path);
    }

    async init() {
        //const module = await import ('../' + path + '/config/config');
        this.config = commonConfig; //가져오는 게 바로 위에 적힌 src/common/config/config.js
        this.axios = new axios.create({
            baseURL: this.config.BASE_URL
        });


        // Add a request interceptor
        this.axios.interceptors.request.use(
            config => {
                if (config.url == '/user/auth/refreshtoken') {
                    const refreshToken = store.getters['auth/getRefreshToken'];
                    config.headers['Authorization'] = 'Bearer ' + refreshToken;
                } else {
                    const token = store.getters['auth/getToken'];
                    config.headers['Authorization'] = 'Bearer ' + token;
                }
                return config;
            },
            error => {
                Promise.reject(error)
            });

        // Add a response interceptor
        this.axios.interceptors.response.use(async (response) => {
            return response;
        }, async(error) => {
            const originalRequest = error.config;
            if (error.response.status === 401 && !originalRequest._retry) {
                originalRequest._retry = true;
                await store.dispatch('auth/refreshToken');
                originalRequest.headers['Authorization'] = 'Bearer ' + store.getters['auth/getToken'];
                return axios(originalRequest);
            }
            if (error.response.status === 403) {
                alert('접근권한이 없습니다');
                router.push('/default/dashboard/ecommerce');
            }
            return Promise.reject(error);
        });

    }


    async get(url, isCommonAlert) {
        try {
            const response = await this.axios.get(url);

            return this.parseResponse(response, isCommonAlert);
        } catch (err) {
            return this.getErrorData(err);
        }

    }

    async post(url, param, isCommonAlert) {
        try {
            const response = await this.axios.post(url, param);

            return this.parseResponse(response, isCommonAlert);
        } catch (err) {
            return this.getErrorData(err);
        }

    }

    parseResponse(response, isCommonAlert) {
        const responseHead = response.data.head;
        const responseBody = response.data.body;

        if (responseHead.resultCode != '0000'& isCommonAlert ) {
            if(responseHead.resultCode != 'EXRS9003' && responseHead.resultCode != 'CMRS9002'){
                alert('Error:: ' + responseHead.resultMessage);
            }
        }

        return {
            resultCode: responseHead.resultCode,
            resultMessage: responseHead.resultMessage,
            data: responseBody
        };

    }

    getErrorData(err) {
        console.log('getErrorData', err);
        return {
            resultCode: 'error',
            resultMessage: err.toString(),
            data: {}
        };
    }
}
export default apiCommon
```

- mapApi 부분이 바로 api를 call하는 부분이다. 좀 별로 같다. 
- 이게 vuex의 문제인 걸 수도 있다. vuex는 actions과 mutation이 따로 있다.
- 따라서 이런 식으로 api를 부르는 action, 정보를 저장하는 mutation이 한꺼번에 묶여버린다.
```js
import { mapApi } from 'Api';
//import Vue from 'vue';
const actions = {
    getMemberAddressList(context, pagenationData) {
        return mapApi.getMemberAddressList(pagenationData).then(res => {
            context.commit('getMemberAddressListSuccess', res);
        }).catch(error => {
            console.log('getMemberAddressListError', error);
        })
    },
}
```

- from Api 같은 import는 아래와 같이 vue.config.js에서 alias 설정을 해주면 된다.
```js
configureWebpack: {
        resolve: {
            alias: {
                '~': path.resolve(__dirname, 'src/'),
                Api: path.resolve(__dirname, 'src/api/'),
                Components: path.resolve(__dirname, 'src/components/'),
                Constants: path.resolve(__dirname, 'src/constants/'),
                Container: path.resolve(__dirname, 'src/container/'),
                Views: path.resolve(__dirname, 'src/views/'),
                Helpers: path.resolve(__dirname, 'src/helpers/'),
                Themes: path.resolve(__dirname, 'src/themes/')
            },
            extensions: ['*', '.js', '.vue', '.json']
        },
        plugins: [
            //jquery plugin
            new webpack.ProvidePlugin({
                $: 'jquery',
                jquery: 'jquery',
                'window.jQuery': 'jquery',
                jQuery: 'jquery'
            }),
        ],
    },
```
## <span style="color:#802548">_2. vue2의 vuex_</span>

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





## <span style="color:#802548">_3. Jpa Specification_</span>
- 아래와 같은 jparepository의 method명이 있다.
- 쿼리를 날려보자.


```java
Page<User> findByRoles_ruleSeqNotAndRoles_ruleSeqNotAndUserNmContainingOrRoles_ruleSeqNotAndUserProfile_prflCmpnyGradeContainingOrRoles_ruleSeqNotAndUserProfile_prflDprtmContaining(long ruleSeq, long ruleSeq2, String userNm, long ruleSeq3, String userEmailAddr, long ruleSeq4, String prflDprtm, Pageable pageable);
```


```sql
 select
        user0_.USER_SEQ as user_seq1_21_,
        user0_.USER_EMAIL_ADDR as user_ema2_21_,
        user0_.USER_ID as user_id3_21_,
        user0_.USER_NM as user_nm4_21_,
        user0_.USER_PW as user_pw5_21_,
        user0_.USER_USE_YN as user_use6_21_
    from
        TB_USER user0_
    left outer join
        TB_USER_ROLES roles1_
            on user0_.USER_SEQ=roles1_.USER_SEQ
    left outer join
        TB_USER_ROLE userrole2_
            on roles1_.RULE_SEQ=userrole2_.RULE_SEQ
    left outer join
        TB_USER_PROFILE userprofil3_
            on user0_.USER_SEQ=userprofil3_.USER_SEQ
    where
        userrole2_.RULE_SEQ<>3
        and userrole2_.RULE_SEQ<>3
        and (
            user0_.USER_NM like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_CMPNY_GRADE like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_DPRTM like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_ADDR like '%%'
        )
    order by
        user0_.USER_NM asc limit 100;
```

- 위에서 address도 검색가능하게 추가했다.
- 쿼리를 날려보자.


```java
Page<User> findByRoles_ruleSeqNotAndRoles_ruleSeqNotAndUserNmContainingOrRoles_ruleSeqNotAndUserProfile_prflCmpnyGradeContainingOrRoles_ruleSeqNotAndUserProfile_prflDprtmContainingOrRoles_ruleSeqNotAndUserProfile_prflAddrContaining(long ruleSeq, long ruleSeq2, String userNm, long ruleSeq3,String userEmailAddr, long ruleSeq4, String prflDprtm, long ruleSeq5, String prflAddr, Pageable pageable);
```


```sql
 select
        user0_.USER_SEQ as user_seq1_21_,
        user0_.USER_EMAIL_ADDR as user_ema2_21_,
        user0_.USER_ID as user_id3_21_,
        user0_.USER_NM as user_nm4_21_,
        user0_.USER_PW as user_pw5_21_,
        user0_.USER_USE_YN as user_use6_21_
    from
        TB_USER user0_
    left outer join
        TB_USER_ROLES roles1_
            on user0_.USER_SEQ=roles1_.USER_SEQ
    left outer join
        TB_USER_ROLE userrole2_
            on roles1_.RULE_SEQ=userrole2_.RULE_SEQ
    left outer join
        TB_USER_PROFILE userprofil3_
            on user0_.USER_SEQ=userprofil3_.USER_SEQ
    where
        userrole2_.RULE_SEQ<>3
        and userrole2_.RULE_SEQ<>3
        and (
            user0_.USER_NM like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_CMPNY_GRADE like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_DPRTM like '%%'
        )
    order by
        user0_.USER_NM asc limit 100;
```


- 위에서 비활성화계정을 제외하기 위해 method명을 바꿨다.
- 쿼리를 날려보자.

```java
Page<User> findByUserUserYnAndRoles_ruleSeqNotAndRoles_ruleSeqNotAndUserNmContainingOrRoles_ruleSeqNotAndUserProfile_prflCmpnyGradeContainingOrRoles_ruleSeqNotAndUserProfile_prflDprtmContainingOrRoles_ruleSeqNotAndUserProfile_prflAddrContaining(CommonEnumYN isYn, long ruleSeq, long ruleSeq2, String userNm, long ruleSeq3,String userEmailAddr, long ruleSeq4, String prflDprtm, long ruleSeq5, String prflAddr, Pageable pageable);
```


```sql
 select
        user0_.USER_SEQ as user_seq1_21_,
        user0_.USER_EMAIL_ADDR as user_ema2_21_,
        user0_.USER_ID as user_id3_21_,
        user0_.USER_NM as user_nm4_21_,
        user0_.USER_PW as user_pw5_21_,
        user0_.USER_USE_YN as user_use6_21_
    from
        TB_USER user0_
    left outer join
        TB_USER_ROLES roles1_
            on user0_.USER_SEQ=roles1_.USER_SEQ
    left outer join
        TB_USER_ROLE userrole2_
            on roles1_.RULE_SEQ=userrole2_.RULE_SEQ
    left outer join
        TB_USER_PROFILE userprofil3_
            on user0_.USER_SEQ=userprofil3_.USER_SEQ
    where
        user0_.USER_USE_YN=1
        and userrole2_.RULE_SEQ<>3
        and userrole2_.RULE_SEQ<>3
        and (
            user0_.USER_NM like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_CMPNY_GRADE like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_DPRTM like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_ADDR like '%%'
        )
    order by
        user0_.USER_NM asc limit 100;
```


- 그런데 오류가 난다. 
- userUseYn이 N인 것 하나가 노출된다.
- 그래서 hibernate가 생산한 쿼리를 직접 client에서 돌려봤다.
- 아래 쿼리로 돌리면 오류가 나지 않는다.

```sql
 select
        user0_.USER_SEQ as user_seq1_21_,
        user0_.USER_EMAIL_ADDR as user_ema2_21_,
        user0_.USER_ID as user_id3_21_,
        user0_.USER_NM as user_nm4_21_,
        user0_.USER_PW as user_pw5_21_,
        user0_.USER_USE_YN as user_use6_21_
    from
        TB_USER user0_
    left outer join
        TB_USER_ROLES roles1_
            on user0_.USER_SEQ=roles1_.USER_SEQ
    left outer join
        TB_USER_ROLE userrole2_
            on roles1_.RULE_SEQ=userrole2_.RULE_SEQ
    left outer join
        TB_USER_PROFILE userprofil3_
            on user0_.USER_SEQ=userprofil3_.USER_SEQ
    where
        user0_.USER_USE_YN=1
        and userrole2_.RULE_SEQ<>3
        and userrole2_.RULE_SEQ<>3
        and (
            user0_.USER_NM like '%%'
        )
    order by
        user0_.USER_NM asc limit 100;
```


- 그런데 그 이후부터 문제가 났다.
- or and 처리에서 괄호가 문제가 생겼던 셈이다.


```sql
select
        user0_.USER_SEQ as user_seq1_21_,
        user0_.USER_EMAIL_ADDR as user_ema2_21_,
        user0_.USER_ID as user_id3_21_,
        user0_.USER_NM as user_nm4_21_,
        user0_.USER_PW as user_pw5_21_,
        user0_.USER_USE_YN as user_use6_21_
    from
        TB_USER user0_
    left outer join
        TB_USER_ROLES roles1_
            on user0_.USER_SEQ=roles1_.USER_SEQ
    left outer join
        TB_USER_ROLE userrole2_
            on roles1_.RULE_SEQ=userrole2_.RULE_SEQ
    left outer join
        TB_USER_PROFILE userprofil3_
            on user0_.USER_SEQ=userprofil3_.USER_SEQ
    where
        user0_.USER_USE_YN=1
        and userrole2_.RULE_SEQ<>3
        and userrole2_.RULE_SEQ<>3
        and (
            user0_.USER_NM like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_CMPNY_GRADE like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_DPRTM like '%%'
        )
        or userrole2_.RULE_SEQ<>3
        and (
            userprofil3_.PRFL_ADDR like '%%'
        )
    order by
        user0_.USER_NM asc limit 100;
```

- 그래서 그냥 extends JPARepository로 괄호를 조절할 수 있나 찾아보았다.
- 하지만 불가능했다. parenthesis를 넣으려면 JPA specifications를 써야한다고 한다.
- 그래서 specification을 도입했다.


```java
public class UserSpecifications {
    private static final int ADMIN_ROLE_VALUE = 3;
    private static final int USER_ACTIVATED = 1;
    

    public static Specification<User> customConditions(String keyword) {
        return (root, query, cb) -> {
            // Joining tables
            root.join("roles", javax.persistence.criteria.JoinType.LEFT);
            root.join("userProfile", javax.persistence.criteria.JoinType.LEFT);

            // Building the conditions
            Predicate namePredicate = cb.like(root.get("userNm"), "%" + keyword + "%");
            Predicate companyGradePredicate = cb.like(root.get("userProfile").get("prflCmpnyGrade"),  "%" + keyword + "%");
            Predicate departmentPredicate = cb.like(root.get("userProfile").get("prflDprtm"),  "%" + keyword + "%");
            Predicate addressPredicate = cb.like(root.get("userProfile").get("prflAddr"),  "%" + keyword + "%");
            Predicate ruleSeqPredicate = cb.notEqual(root.join("roles").get("ruleSeq"), ADMIN_ROLE_VALUE);
            Predicate userUseYnPredicate = cb.equal(root.get("userUseYn"), USER_ACTIVATED);

            // Combining the conditions with AND
            Predicate andPredicate = cb.and(
                    cb.or(namePredicate,companyGradePredicate, departmentPredicate, addressPredicate),
                    ruleSeqPredicate,
                    userUseYnPredicate
            );

            // Adding the WHERE clause
            query.where(andPredicate);

            // Adding the ORDER BY clause
            query.orderBy(cb.asc(root.get("userNm")));

            return null; // CriteriaQuery root is not used
        };
    }

    public static Specification<User> countCustomConditions(String keyword) {
        return (root, query, cb) -> {
            // Joining tables
            root.join("roles", javax.persistence.criteria.JoinType.LEFT);
            root.join("userProfile", javax.persistence.criteria.JoinType.LEFT);

            // Building the conditions
            Predicate namePredicate = cb.like(root.get("userNm"), "%" + keyword + "%");
            Predicate companyGradePredicate = cb.like(root.get("userProfile").get("prflCmpnyGrade"), "%" + keyword + "%");
            Predicate departmentPredicate = cb.like(root.get("userProfile").get("prflDprtm"), "%" + keyword + "%");
            Predicate addressPredicate = cb.like(root.get("userProfile").get("prflAddr"), "%" + keyword + "%");
            Predicate ruleSeqPredicate = cb.notEqual(root.join("roles").get("ruleSeq"), ADMIN_ROLE_VALUE);
            Predicate userUseYnPredicate = cb.equal(root.get("userUseYn"), USER_ACTIVATED);

            // Combining the conditions with AND
            Predicate andPredicate = cb.and(
                    cb.or(namePredicate, companyGradePredicate, departmentPredicate, addressPredicate),
                    ruleSeqPredicate,
                    userUseYnPredicate
            );

            // Adding the WHERE clause
            query.where(andPredicate);

            // Adding the SELECT clause for counting
            query.multiselect(cb.count(root.get("userSeq")));

            return null; // CriteriaQuery root is not used
        };
    }
}
```

- userUseYn을 int 값이 아니라 enum으로 활용할 수도 있다.
```java
public class UserSpecifications {
    private static final int ADMIN_ROLE_VALUE = 3;
    

    public static Specification<User> customConditions(String keyword) {
        return (root, query, cb) -> {
            // Joining tables
            root.join("roles", javax.persistence.criteria.JoinType.LEFT);
            root.join("userProfile", javax.persistence.criteria.JoinType.LEFT);

            // Building the conditions
            Predicate namePredicate = cb.like(root.get("userNm"), "%" + keyword + "%");
            Predicate companyGradePredicate = cb.like(root.get("userProfile").get("prflCmpnyGrade"),  "%" + keyword + "%");
            Predicate departmentPredicate = cb.like(root.get("userProfile").get("prflDprtm"),  "%" + keyword + "%");
            Predicate addressPredicate = cb.like(root.get("userProfile").get("prflAddr"),  "%" + keyword + "%");
            Predicate ruleSeqPredicate = cb.notEqual(root.join("roles").get("ruleSeq"), ADMIN_ROLE_VALUE);
            Predicate userUseYnPredicate = cb.equal(root.get("userUseYn"), CommonYnEnum.Y); //여기 바꿈

            // Combining the conditions with AND
            Predicate andPredicate = cb.and(
                    cb.or(namePredicate,companyGradePredicate, departmentPredicate, addressPredicate),
                    ruleSeqPredicate,
                    userUseYnPredicate
            );

            // Adding the WHERE clause
            query.where(andPredicate);

            // Adding the ORDER BY clause
            query.orderBy(cb.asc(root.get("userNm")));

            return null; // CriteriaQuery root is not used
        };
    }

    public static Specification<User> countCustomConditions(String keyword) {
        return (root, query, cb) -> {
            // Joining tables
            root.join("roles", javax.persistence.criteria.JoinType.LEFT);
            root.join("userProfile", javax.persistence.criteria.JoinType.LEFT);

            // Building the conditions
            Predicate namePredicate = cb.like(root.get("userNm"), "%" + keyword + "%");
            Predicate companyGradePredicate = cb.like(root.get("userProfile").get("prflCmpnyGrade"), "%" + keyword + "%");
            Predicate departmentPredicate = cb.like(root.get("userProfile").get("prflDprtm"), "%" + keyword + "%");
            Predicate addressPredicate = cb.like(root.get("userProfile").get("prflAddr"), "%" + keyword + "%");
            Predicate ruleSeqPredicate = cb.notEqual(root.join("roles").get("ruleSeq"), ADMIN_ROLE_VALUE);
            Predicate userUseYnPredicate = cb.equal(root.get("userUseYn"), CommonYnEnum.Y); //여기 바꿈

            // Combining the conditions with AND
            Predicate andPredicate = cb.and(
                    cb.or(namePredicate, companyGradePredicate, departmentPredicate, addressPredicate),
                    ruleSeqPredicate,
                    userUseYnPredicate
            );

            // Adding the WHERE clause
            query.where(andPredicate);

            // Adding the SELECT clause for counting
            query.multiselect(cb.count(root.get("userSeq")));

            return null; // CriteriaQuery root is not used
        };
    }
}
```

- 아래와 같은 sql문이 형성된다.
```sql
 select
        user0_.USER_SEQ as user_seq1_21_,
        user0_.USER_EMAIL_ADDR as user_ema2_21_,
        user0_.USER_ID as user_id3_21_,
        user0_.USER_NM as user_nm4_21_,
        user0_.USER_PW as user_pw5_21_,
        user0_.USER_USE_YN as user_use6_21_ 
    from
        TB_USER user0_ 
    left outer join
        TB_USER_ROLES roles1_ 
            on user0_.USER_SEQ=roles1_.USER_SEQ 
    left outer join
        TB_USER_ROLE userrole2_ 
            on roles1_.RULE_SEQ=userrole2_.RULE_SEQ 
    left outer join                     /**걸은적이 없는 join이?? */
        TB_USER_PROFILE userprofil3_ 
            on user0_.USER_SEQ=userprofil3_.USER_SEQ 
    inner join                          /** 걸은 적이 없는 join이?? */
        TB_USER_ROLES roles4_ 
            on user0_.USER_SEQ=roles4_.USER_SEQ 
    inner join TB_USER_ROLE userrole5_  /** 걸은 적이 없는 join이?? */
            on roles4_.RULE_SEQ=userrole5_.RULE_SEQ 
    where
        (
            user0_.USER_NM like ? 
            or userprofil3_.PRFL_CMPNY_GRADE like ? 
            or userprofil3_.PRFL_DPRTM like ? 
            or userprofil3_.PRFL_ADDR like ?
        ) 
        and userrole5_.RULE_SEQ<>3 
        and user0_.USER_USE_YN=1

```

- user_profile은 걸은 join이 없는데 걸렸다.
- JPA의 entity 설계 때문에 그렇다.
```java
@Data
@Entity
@Table(name = "TB_USER", uniqueConstraints = { @UniqueConstraint(columnNames = "USER_ID") })
public class User implements Cloneable {
    .
    .
    .
    @OneToOne(mappedBy = "user")
	@JoinColumn(name="PRFL_SEQ")
	@JsonManagedReference
	private UserProfile userProfile;
}
```


- inner join도 마찬가지의 이유다.
- join과 inversejoin으로 인한 것이다.
```java
@Data
@Entity
@Table(name = "TB_USER", uniqueConstraints = { @UniqueConstraint(columnNames = "USER_ID") })
public class User implements Cloneable {
    .
    .
    .

    @ManyToMany(fetch = FetchType.LAZY)
	@JoinTable(name = "TB_USER_ROLES", joinColumns = @JoinColumn(name = "USER_SEQ"), inverseJoinColumns = @JoinColumn(name = "RULE_SEQ"))
	private Set<UserRole> roles = new HashSet<>();
}
```

- enum class의 경우, @Enmerated를 붙이지 않으면 자동으로 ORDINAL이 된다.
- 그럼 들어온 순서대로 0,1,2...로 DB에 저장된다.
```java
@Column(name = "USER_USE_YN", nullable = false)
//@Enumerated(EnumType.ORDINAL) 안쓰면 ORDINAL이 default
	private CommonYNEnum userUseYn;
```




