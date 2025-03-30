## <span style="color:#802548">_node 깔기_</span>

- eslint는 npm을 통해 깔 수 있다. 이는 package.json이 깔려있어야한다는 의미다.
- 그를 위해서 우선, node를 먼저 깔아줘야 한다.
- 그래야 npm도 사용 가능하다.
- node를 초콜레티로 깔아주고서, 컴퓨터를 껐다가 켜준다.
    - 컴퓨터를 껐다가 키지 않으면 node, npm 명령어가 듣지 않는다.

## <span style="color:#802548">_eslint 깔기_</span>
- cmd에서 node -v와 npm -v 시 버전이 출력되면, 프로젝트에서 아래 명령어를 실행해준다.
    
```sh
npm install eslint --save-dev
npx eslint --init
```



- 그럼 설치 명령어들이 나오게 된다.
- 아래 순서대로 쳐준다.

```
to check syntax and problem
none of these
none of these
no
browser
yes
npm
```

## <span style="color:#802548">_gitignore 추가_</span>
- gradle project기 떄문에 굳이 pacakge.json을 달고 다닐 필요는 없다.
- 관련 파일들을 .gitignore에 넣어준다.

```
package-lock.json
package.json
eslint.config.mjs
node_modules/
```


## <span style="color:#802548">_eslint extension 추가_</span>
- eslint를 활성화시키려면 extension에서 eslint를 추가해줘야 한다.
- ms 거로 설치해주자.


## <span style="color:#802548">_eslint 설정_</span>
- 그럼 eslint.config.js가 설정된다.
- 참고로 과거 버전은 eslintrc.json이었는데, v9로 올라가며 바뀌었다. eslintrc.json에 추가해도 아무런 소용이 없다.

- eslint.config.js는 아래와 같이 구성한다.
- globals는 HTML 안에서 script 형태로 구성되기 때문에 글로별 변수로 넣어줘야만 한다.
- rule은 원하는 것들을 추가해주면 된다.
- 이렇게 하면 SSR에서도 no-def 같은 문제를 찾아내기 매우 수월해진다.


```js
import { defineConfig } from "eslint/config";
import globals from "globals";
import js from "@eslint/js";

export default defineConfig([
  {
    files: ["**/*.{js,mjs,cjs}"],
    languageOptions: {
      sourceType: "script",
      globals: {
        ...globals.browser,
        $: true,        // ✅ Add $ global
        jQuery: true,   // ✅ Add jQuery global
        Swal: true,
        Dropzone: true,
        Quill: true
      }
    },
    plugins: { js },
    rules: {
      "no-undef": "error",
      "no-console": "warn",
      "indent": ["error", 4] 
    }
  }
]);
```
