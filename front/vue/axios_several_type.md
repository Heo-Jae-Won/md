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

- 반면에 회사 PMS에서는 사용법이 좀 달랐다.
- 개인적으로는 내 방법이 더 낫다고 생각된다.
- 회사 것은 너무 잘게 쪼개져있다.

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
