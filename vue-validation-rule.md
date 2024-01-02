## <span style="color:#802548">_1. ruls 만들기_</span>
- returnType은 form을 준수했냐 안했냐 하는 boolean변수와 false일 때 내보낼 경고문구인 string 모두 포함한다. 
- 이것은 아래 함수의 return type으로 지정할 것이다. const나 let이 아니라 type으로 declare한다.
```js
type ruleReturn = boolean | string;
```

- 아래와 같이 key-value구조를 갖는데, js에서는 함수가 1급객체라 key의 value로 할당이 가능하다. 
- 의도는 아래와 같다. value를 받아서 값이 비어있다면 필수항목입니다 라는 문구를 return한다.
```js
value => {
  if (value === "") {
    return '필수항목 입니다.';
  }
};
```
- 그것에 type을 붙이면 아래와 같이 변한다. 함수의 parameter에 type을 붙였고, 함수의 returntype에 함수를 붙였다. 
```js
(value: string): ruleReturn=> {
  if (value === "") {
    return '필수항목 입니다.';
  }
};
```
- 그것을 module화하여 아래와 같이 나타낼 수 있다. 저기서 export는 JAVA의 pulbic과 같이 전역접근이 가능함을 의미한다.
```js
export const rules = {
  required: (v: string): returnType=> !!v || '필수 항목입니다.',
.
.
.
}
```
- !!라는 문법은 익숙하지 않으니 아래의 사이트로 대체한다. 쉽게 말해 어떤 것이든 boolean으로 변환하며 undefined,"",0 등 falsy하면 false가 된다.

https://hermeslog.tistory.com/279

 
||는 null coalscing 문법으로 ES 6에서 도입되었다고 한다. 데이터가 falsy면 || 뒤의 값으로 대신 지정된다.

https://javascript.plainenglish.io/new-operators-in-ecmascript-2020-3dbb886b2fd9

 

## <span style="color:#802548">_2. vuetify_</span>
- rule을 만들었다면 v-form에 ref값을 넣고, v-text-field에는 rules 규칙을 넣는다.
```html
<v-form ref="form">
<v-text-field :rules="[v => rules.required(v), v => rules.max20(v)]" .../>
</v-form>
```
- html에 넣는 것이 싫다면 script단으로 뺄 수 있다.
```html
<v-text-field :rules="inputRequired".../>
<script>
import { rules } from '@/utils/rule';
const inputRequired= [(v: string) => rules.required(v)];
```
- vuetify가 가진 v-form에 있는 validate()라는 함수를 사용해 validation을 검증하고 통과여부 값을 받아 사용할 것이다. 
- 해당 함수는 유효 여부를 나타내는 boolean 변수 valid와 error를 return한다.
- validate()를 활용하려면 그저 class 대신 쓰이는 ref처럼 HTMLDIvElement로 선언해서는 안된다.

```js
const form = ref<HTMLDIVElement>(null);
```
- validate()를 사용하기 위해서는 HTMLFormElement라는 interface를 사용해야 하기에 아래와 같이 generic type을 설정한다. 또는 아예 빈 칸으로 비워둔다.
```js
const form = ref<HTMLFormElement | null>(null);
const form = ref(null);
```
- validate()는 promise기 때문에 async-await를 사용한다. 
- 그런데 login함수도 axios라서 promise기 때문에 async-await를 사용해야 한다. 
- 따라서 아래와 같이 사용할 수 없다. 그렇게 되면 login axios가 함수 최상단이 아니라서 await를 활용하지 못한다.
```js
    await form.value?.validate().then((result: any) => {
      if (result.valid) {
        const memberStatus = (await onLogin(userId.value, userPassword.value)).data;
        if (memberStatus === 1) {
          alert('로그인이 완료되었습니다.');
          changeLoginMember(userId.value);
          router.push('/');
        }
      }
    });
```
- 결국 ref변수를 따로 하나 만들어서 result.valid 값을 받아와야 한다. 
- validation의 통과여부를 valid가 담고 있다. form.value가 null일 수 있으니 ?.로 써준다. 그래야 error가 안 뜬다.
```js
  const isValidateTrue = ref<boolean>();
    const loginForm = async () => {
      await form.value?.validate().then((result: any) => {
        isValidateTrue.value = result.valid;
      });
.
.
   }
```
## <span style="color:#802548">_3. 자체 interface 생성_</span>
- parameter를 any type으로 그대로 놔두는 것은 좋지 않다. any type을 없애보자.  
- result의 console.log를 찍어보면 아래와 같이 나온다. 
- 처음에는 내장 interface인 PromiseFulfilledResult를 이용하려 했으나 valid라는 property가 없어서 쓸 수가 없었다. 
- 그 이유는 validate()의 결과물이 담기는 형태를 vuetify에서 정해놨기에 아래와 같은 형태로 담기기 때문이었다.
```js
export interface ValidationResult {
  valid: boolean;
  errors: Record<string, string>;
}
```

- 만들어줬다면 parameter의 type을 ValidationResult interface로 바꾼다. 그러면 async 1개에 await가 2개가 묶인 함수가 완성된다.
```js
    const loginForm = async () => {
      await form.value?.validate().then((result: ValidationResult) => {
        isValidateTrue.value = result.valid;
      });
      if (!isValidateTrue.value) {
        alert('비정상 입력입니다');
        return;
      }

      const memberStatus = (await onLogin(memId.value, memPw.value)).data;
      if (memberStatus === 1) {
        alert('로그인이 완료되었습니다.');
        changeLoginMember(memId.value);
        router.push('/');
      }
    };
```
- 그럼 아래와 같이 로그인시도를 하면 front에서 가로막아 backend까지 전달되지 않는다. 
- 그리고 alert를 끄면 뭐가 잘못 됐는지 결과를 확인시켜준다. network를 보면 request가 가지 않은 것을 볼 수 있다.

 

- 반면에 정상 로그인 시도인 경우에는 아래와 같이 request가 정상으로 가게 된다. 
- 개발자도구창에는 두번 요청이 갔는데, cors 때문에 그런 것이니 신경쓸 필요 없다. 

https://developer.mozilla.org/ko/docs/Web/HTTP/CORS


## <span style="color:#802548">_4. 공통함수로 빼내기_</span>
- 아래와 같이 만들었다. 하지만 form validation은 앞으로도 자주 쓰게 될 함수다. 
- 따라서 export로 전역에서 사용가능하게 뺴는 것이 좋다.
```js
    const handleLoginClick = async () => {
      await form.value?.validate().then((result: ValidationResult) => {
        isFormValid.value = result.valid;
      });

      if (!isFormValid.value) {
        alert('빈 내용을 채워주세요.');
        return;
      }
      const data = {
        memberId: memberId.value,
        memberPw: memberPw.value,
      };
      await login(data).then(response => {
        alert('로그인이 완료되었습니다.');
        changeLoginMember(memberId.value, response.data?.accessToken, response.data?.refreshToken);
        router.push('/');
      });
    };
```
- 그래서 처음에는 아래와 같이 axios response까지 같이 담아서 보내려고 했다. 
- 하지만 문제가 있었다. 내가 넣고 싶은 것은 return type이 axiosPromise였는데 axiosResponse로 return type이 나왔다. 
```js
export const useValidateForm = async (form: Ref<HTMLFormElement | null>, isFormValid: Ref<boolean>, submitFunction: AxiosResponse<any>) => {
  await form.value?.validate().then((result: ValidationResult) => {
    isFormValid.value = result.valid;
  });
  console.log(isFormValid.value);
  if (!isFormValid.value) {
    alert('양식에 맞춰 작성해주세요.');
    return;
  }
};
```
- 문제를 살펴보니 async-await를 사용해야 하는 axios에서는 await를 붙이는 순간 AxiosResponse로 return type이 바뀐다고 한다. 
- 그래서 parameter로 axios call을 넣으면 axios가 먼저 불려서 등록이 된 뒤에 form validation을 수행했다. 따라서 무의미한 일이었다. 
```js
 const handleMemberSave = async () => {
      useValidateForm(form, isFormValid, await saveMember(formData));

      alert('등록이 완료되었습니다');
      router.push('/login');
    };
```
- 결국 axios call을 공통함수의 parameter로 넣는 것은 포기하고 validation만 딱 넣었다. 사실 그게 원래 목적에도 맞는 것 같아 보인다. useValidateForm 함수면 validateForm 기능만 하면 되기 때문이다. 
```js
export const useValidateForm = async (form: Ref<HTMLFormElement | null>, isFormValid: Ref<boolean>) => {
    await form.value?.validate().then((result: ValidationResult) => {
        isFormValid.value = result.valid;
  });
  console.log(isFormValid.value);
  if (!isFormValid.value) {
      alert('양식에 맞춰 작성해주세요.');
  }

  return isFormValid;
};
```
- parameter로는 form과 isFormValid를 넣었는데 local과 똑같은 이름의 변수로 맞췄다. 그 편이 덜 헷갈릴 것이라 생각했다. local에서 아래와 같이 쓰고 있었다.
```js
const isFormValid = ref<boolean>(false);
const form = ref<HTMLFormElement | null>(null);
```
- 따라서 parameter로 받을 때는 ref라는 value에서 Ref로 type 선언만 해주는 것으로 바꿨다. 
```js
form: Ref<HTMLFormElement | null>, isFormValid: Ref<boolean>
```
- 또한 isFormValid라는 변수도 필요가 없었다. 그냥 useValidateForm() 내에서 true, false를 받으면 되는 것이었다.

```js
export const useValidateForm = async (form: Ref<HTMLFormElement | null>) => {
    let isFomrValid = false;
    await form.value?.validate().then((result: ValidationResult) => {
        isFomrValid = result.valid;
  });
  if (!isFormValid) {
      alert('양식에 맞춰 작성해주세요.');
  }

  return isFormValid;
};
```
- aysnc가 붙으면 무조건 Promise 객체를 return하고, 거기에 await를 달아줘야 Promise가 resolve되는 구조라서 이런 방식으로 구성했다. 
- 그럼 form validate 값이 true인지 false인지에 따라 등록을 할 지 말지가 정해진다. 
```js
    const handleMemberSave = async () => {
      const status = (await useValidateForm(form)).value;
      if (status) {
        await saveMember(formData);
        alert('등록이 완료되었습니다');
        router.push('/login');
      }
    };
```
- 위는 저장이고 아래는 수정이다. 모두 form validation이 필요한데, 이제는 간단하게 공통함수로 뺄 수 있다.  
```js
    const handleMemberUpdate = async () => {
      const status = (await useValidateForm(form)).value;
      if (status) {
        await updateMember(formData.value, memberId);
        alert('수정이 완료되었습니다');
      }
    };
```
- 참고로 formvalidation에 async를 안 붙이고도 해보려고 했지만, 그러니까 처음할 때는 유효성 검사 여부에 상관없이 무조건 초기값이 들어가고 그 이후부터 form validation 값이 제대로 반영되는 문제가 있어서 폐기했다. 
- validate()함수도 async-await 관계에 묶여야 하는 함수로 보인다. promise를 return하는 듯하다. 

```js
 async function handleLoginClick() {
      const isValid = (await useValidateForm(form))?.value;
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

## <span style="color:#802548">_5. rules_</span>
```js
type rulesType = boolean | string;

export const rules = {
  required: (v: string): rulesType => !!v || '필수 항목입니다.',
  requiredCustom:
    (t: string) =>
    (v: string): rulesType =>
      !!v || t,
  min:
    (n: number) =>
    (v: string): rulesType =>
      (typeof v == 'string' && v.length >= n) || '최소 ' + n + '글자 이상 입력해주세요.',
  birth: (v: string): rulesType => (typeof v === 'string' && v.length === 10) || '생년월일을 10글자로 입력해주세요.',
  min8: (v: string): rulesType => (typeof v === 'string' && v.length >= 8) || '최소 8글자 이상 입력해주세요.',
  min10: (v: string): rulesType => (typeof v === 'string' && v.length >= 10) || '최소 10글자 이상 입력해주세요.',
  max20: (v: string): rulesType => (typeof v === 'string' && v.length <= 20) || '최대 20글자까지 입력가능합니다.',
  max45: (v: string): rulesType => (typeof v === 'string' && v.length <= 45) || '최대 45글자까지 입력가능합니다.',
  max50: (v: string): rulesType => (typeof v === 'string' && v.length <= 50) || '최대 50글자까지 입력가능합니다.',
  max60: (v: string): rulesType => (typeof v === 'string' && v.length <= 60) || '최대 60글자까지 입력가능합니다.',
  max150: (v: string): rulesType => (typeof v === 'string' && v.length <= 150) || '최대 150글자까지 입력가능합니다.',
  krEn: (v: string): rulesType => /^[a-zA-Z가-힣ㄱ-ㅎㅏ-ㅣ\s/(/)]+$/i.test(v) || '숫자, 특수문자는 사용할 수 없습니다.',
  krEnNum: (v: string): rulesType => /^[0-9a-zA-Z가-힣ㄱ-ㅎㅏ-ㅣ\s/(/)]+$/i.test(v) || '한글, 영문, 숫자만 입력가능합니다.',
  email: (v: string): rulesType => /^[0-9a-zA-Z]([-_.]?[0-9a-zA-Z])*@[0-9a-zA-Z]([-_.]?[0-9a-zA-Z])*\.[a-zA-Z]{2,3}$/i.test(v) || '이메일 형식으로 입력해주세요.',
  allowComma: (v: string): rulesType => /[0-9.]$/i.test(v) || '숫자와 .만 입력가능합니다.',
  phone: (v: string): rulesType => /^01([0|1|6|7|8|9])-?([0-9]{3,4})-?([0-9]{4})$/i.test(v) || '01x-xxxx-xxxx형식으로 입력해주세요.',
  enNum: (v: string): rulesType => /^[0-9a-z]+$/.test(v) || '영소문자, 숫자만 입력가능합니다.',
  passwordCheck: (v: string): rulesType => /^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[$@$!%*?&])[A-Za-z\d$@$!%*?&]{8,}/.test(v) || '8자리 이상의 영대소문자와 숫자, 특수문자를 포함해야 합니다.',
};
```
