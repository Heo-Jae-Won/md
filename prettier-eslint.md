- 오늘 자꾸 prettier를 설정한 이후 자꾸 const가 initialize가 안되고 , declare module에서 semi colon을 찾는 오류가 나서 시간을 날리는 바람에 이번에 대략 어떻게 적용하는 지 간략하게 정리했다. 

​

## <span style="color:#802548">_1. eslint 적용_</span>
- 먼저 eslint는 프로젝트마다 다를 수 있다고 해서 extension이 아니라 dependency로 깔았다. 
```
npm install --save-dev eslint
```
- 그 뒤에 eslint를 npx로 실행한다. 
```
npx eslint --init
```
- 그럼 모듈이 저장되지는 않으나 답변에 따라 .eslintrc.json이라는 파일이 생성된다. 
- 맨 마지막 추가 plugin들을 깔게되면 충돌이 잘 나는 아래 plugin은 지워주자. 이상하게 저것만 prettier하고 자꾸 충돌이 난다.

```
npm remove @vue/eslint-config-typescript
```
- 그 뒤 .eslintrc파일에 parser 내용을 채워야 한다. 
- 그러려면 module이 필요하기 때문에 모듈을 깔아주자. 
- i는 install의 줄임말이고 -D는 --save-dev의 줄임말로 개발단계에서만 쓰는 dependency라는 뜻이다. 
```
npm i -D vue-eslint-parser
```
- npm i -D로 깔면 개발용, npm i -S(--save)로 깔면 배포용이다. 
- 그런데 npm 5버전부터는 npm i가 곧 npm i -S이기 때문에 개발용으로만 깔 게 아니면 npm i로 그냥 깔면 된다. 
- 아래 사이트를 보면 npm install의 여러 옵션이 설명되어 있다.

https://docs.npmjs.com/cli/v9/commands/npm-install

- 나는 아래와 같이 파일을 만들었다. 
- async가 있는데 await가 없으면 에러를 내고, ;를 두개이상 붙이면 error를 내고, 사용되지 않는 것들을 놔두면 에러가 나는 형식이다. 
- 그러한 규칙은 rule에 추가한다. 
```js
{
  "env": {
    "browser": true,
    "es2021": true
  },
  "extends": ["eslint:recommended", "plugin:vue/essential", "plugin:@typescript-eslint/recommended"],
  "parserOptions": {
    "ecmaVersion": 12,
    "parser": "@typescript-eslint/parser",
    "sourceType": "module"
  },
  "plugins": ["vue", "@typescript-eslint"],
  "rules": {
    "no-extra-semi": "error",
    "require-await": "error",
    "no-unused-expressions": [
      "error",
      {
        "allowTernary": true, // a || b
        "allowShortCircuit": true, // a ? b : 0
        "allowTaggedTemplates": true
      }
    ]
  },
  "parser": "vue-eslint-parser"
}
```
eslint에 관한 구성은 아래 사이트에 잘 설명되어있다.

https://velog.io/@kyusung/eslint-config-2


 
rule 옵션은 아래 공식홈페이지를 참조하자.

https://eslint.org/docs/latest/rules/

- 참고로 vue 프로젝트기 때문에 vue parser를 넣은 것이다. 안하면 parser가 없어서 vue파일인지 되지 않아 아래 같은 오류가 난다. 
```
 Eslint Vue 3 Parsing error: '>' expected.eslint 
```
​

- 따라서 parser 설정은 필수다. 
- 그래서 위에서 해당 moduel을 npm에서 받은 것이다.
```
"parser": "vue-eslint-parser"
 ```
- 그러고 나서 lint 명령어를 실행하면 아래와 같이 error와 warn이 뜬다. 이걸 보고 고치면 된다. 참고로 그냥 소스코드 쓸 때도 rule에 따라 error 표시를 해준다. 


- eslint가 무시했으면 하는 설정파일은 .eslintignore에 적어둔다. 참고로 js파일이면 .js라는 확장자까지 모두 적어줘야 한다. node_modules는 필수로 들어가야 한다. 아니면 node_modules도 검사하기 때문에 에러문구가 너무 많이 뜬다.


## <span style="color:#802548">_2. prettier 설정_</span>
- prettier는 코딩 스타일에 관한 것이라 전역적으로 사용하려고 그냥 extension을 깔았다.  

- prettier를 깔았다면 설정파일을 만들어주자. 
- 설정파일 이름은 .prettierrc.json으로 만들었다. .
- prettierrc로도 만들수있다. eslint+rc, prettier+rc로 뒤에 rc가 붙는데  자동 실행될 스크립트인 runcom file에서 유래됐다고 한다.

- 한마디로 run commands 의 약자인데, 여기서는 runtime configuration이라고 생각된다. 
```
{
  "singleQuote": true,
  "bracketSpacing": true,
  "bracketSameLine": true,
  "arrowParens": "avoid",
  "printWidth": 120,
  "tabWidth": 2
}
```
- 저장 시에 자동으로 파일들을 prettier 규칙에 맞게 바꾸고 싶다면 아래와 같은 방법으로 설정해줄 수 있다.
- 잘 적용이 됐나 보기 위해 ''로 string 처리한 것을 ""로 바꾸고 저장을 눌러보자. 
- 그럼 저절로 ''로 변환되면서 저장됨을 확인할 수 있다. 나는 tabWidth도 10으로 바꿔놔서 아래와 같이 tab이 어마어마하게 늘어났다. 참고로 저장만이 아니라 자동정렬 기능(기본 alt shift f)을 사용할 때도 prettier가 적용된다.

