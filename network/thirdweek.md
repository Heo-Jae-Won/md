## <span style="color:#802548">_쿠키, 세션_</span> 
- 쿠키나 세션이나 모두 stateless를 극복하려는 시도이다.
  - 쿠키는 상태를 client에 저장한다.
  - 세션은 상태를 WAS에 저장한다.
  - 쿠키와 session의 차이는 아래와 같다.
    - 쿠키는 클라에 저장돼 접근이 수월해 보안이 취약하다.
    - 세션은 서버에 저장돼 접근이 어려워 보안이 상대적으로 좋다.
    - 대신 쿠키는 서버 연동이 없으니 속도가 세션보다 속도가 좋다.
    - 세션은 상태 처리를 하는 프로세스(생성-갱신-만료)를 서버에서 처리해서 속도가 느리다.
  - 쿠키와 session의 공통점은 아래와 같다.
    - cookie를 기반으로 한다.
    - cookie 기반이기에 web에서만 사용가능하다. 
      - 핸드폰 native에서 사용불가능하다.
      - 물론 핸드폰으로 여는 web에선 사용가능하다.(이른바 웹뷰)
- 쿠키를 client에서 만들수도 있지만, 인증에서 쓰이는 cookie는 서버에서 만드는 편이다. 작동방식은 아래와 같다.
  - 클라이언트 request
  - 서버는 Set-Cookie header에 넣어 response
  - 차후 request부터 계속 cookie가 header에 담겨감(browser가 담당)

- client-side cookie creation
```js
function setCookie(name, value, days) {
    var expires = "";
    if (days) {
        var date = new Date();
        date.setTime(date.getTime() + (days * 24 * 60 * 60 * 1000));
        expires = "; expires=" + date.toUTCString();
    }
    document.cookie = name + "=" + (value || "") + expires + "; path=/";
}

// Example: Set a cookie named "username" with value "john_doe" that expires in 1 day
setCookie("username", "john_doe", 1);
```

- server-side cookie creation
```java
@GetMapping("/set-cookie")
    public String setCookie(HttpServletResponse response) {
        // Set a cookie named "username" with value "john_doe" that expires in 1 day
        Cookie cookie = new Cookie("username", "john_doe");
        cookie.setMaxAge(24 * 60 * 60); // 1 day in seconds
        response.addCookie(cookie);

        return "Cookie set!";
    }
```


- session의 작동방식은 아래와 같다.
  - 클라이언트가 request
  - 서버는 cookie header 확인하여 session-id가 있나 확인
  - 없으면 서버는 세션키를 생성, 세션키를 이용한 저장소 생성, 세션키를 담은 쿠키 생성
  - 서버가 set-cookie header에 session-id 넣어 response
  - 클라이언트는 향후 cookie header에 session-id값을 넣어 request(browser가 담당)
  - 서버는 session-id를 보고 세션저장소 데이터를 활용

- session은 browser별로 부여되는 것이다.
- 따라서 같은 사용자라도 browser를 바꿔 로그인을 하면 다른 session-id가 생성되어 관리된다.
- 당연히 다른 session-id끼리는 정보가 공유되지 않고 다시 쌓여야 한다.
- username이 늘 같은 key여도 value가 다르게 뽑히는 이유가 바로 session key가 다르게 관리되기 때문이다.
- 해당 session key(Java에선 JSessionId)가 가진 key에 맞는 value가 뽑힌다.
```java
@GetMapping("/create-session")
    public String createSession(HttpServletRequest request) {
        // Get the HttpSession object from the HttpServletRequest
        HttpSession session = request.getSession();

        // Set an attribute in the session
        session.setAttribute("username", "john_doe");

        return "Session created!";
    }
```


## <span style="color:#802548">_URI, URL, URN_</span> 
- uri는 인터넷의 자원을 식별할 수 있는 문자열을 의미한다.
  - 그 중 url은 리소스가 위치한 정보를 사용하는 방식이다.
  - urn은 리소스에 이름을 매핑하여 사용하는 방식이다. 
  - 현실에선 uri나 url이나 잘 구분하지 않고 사용되는 경향이 매우 강하다. urn은 쓰이지 않는다.
- uri는 scheme://host/path?query#fragment와 같은 형식으로 구성된다.
  - scheme은 protocol이고, host는 domain이다. 
  - fragment는 자주 쓰이진 않지만 웹페이지의 특정 구역으로 이동하게 할 때 사용한다.


## <span style="color:#802548">_Restful api_</span> 
- Restful api는 자원의 표현을 통한 상태를 전달하는 데 초점을 둔 HTTP 활용 규약이다.
  - 1990년대의 HTTP api는 GET-POST로 모든 것을 처리했기 때문에 수정,삭제,생성 등이 별도의 method로 구분되지 않았다.
  - uri만으로 api를 통해 자원이 어떻게 변화하는지 파악하기도 어려웠다.
  - 그에따라 uri, http method를 특정 원칙을 준수하며 사용하자는 주장이 나왔는데 그게 바로 restful api다.
  - 복잡한 원칙은 제쳐두고 간단한 몇가지 원칙을 나열하면 아래와 같다. 거의 uri 중 path와 query에 적용된다.
    - 하이픈 허용. 언더바 X
    - 파일확장자 X
    - 소문자 uri
    - uri의 path는 동사가 아닌 명사로
    - uri의 paht는 위계질서를 갖게 구성
  - 이제 복잡한 제약조건을 살펴보자.
    - client - server 분리
      - HTTP api를 쓰는 방법과 동일하다.
    - stateless
      - session을 쓰지 말라는 의미다. 
      - restful api를 준수하려면 jwt같은 인증을 사용해야 한다.
    - cachable
      - 캐시를 사용해야 한다.
      - HTTP api를 쓰는 방법과 동일하다.
      - cache-control header를 사용하면 된다.
    - layered system
      - client는 오로지 rest api서버만 바라보면 되게 서버 아키텍쳐를 구성한다.
      - HTTP api를 쓰는 방법과 동일하다.
    -  Code on Demend
       - 선택조건으로 서버가 보내준 js 코드를 즉시 실행한다.
       - 솔직히 어떻게 작동하는건지 이해 못했다. 써본 적이 없다..
    - uniform interface
      - 리소스 식별
        - 특정 리소스는 오직 하나의 URL만 가져야 한다.
      - 표현을 통한 자원 조작 
        - accept 등 콘텐츠 협상헤더를 사용하여 동일한 uri로 새로운 리소스 표현을 제공할 수 있어야 한다.
        - 보통 text/plain, application/json 등이 자주 쓰이는 리소스 표현 방식이다.
      - 자기서술적 메시지
        - header, body만 보고 HTTP message가 파악될 수 있어야 한다.
      - 애플리케이션의 상태가 Hyperlink를 이용해 전이
        - 관련 있는 resource로 이동할 수 있는 링크가 제공되어야 한다.
        - 기존 HTTP api와는 많이 동떨어져있다.

<img src='/image/hateoas.png' />



## <span style="color:#802548">_sop와 cors_</span> 
- 맨처음 SSR의 시대에는 client를 다른 domain으로 쓰지 않았다.
- 따라서 브라우저 resource 정책으로 sop(same origin policy)를 적용해 다른 domain은 다 막아버리면 됐다.
- 하지만 front framework가 등장하고 CSR의 시대가 오면서 프론트에서 백엔드로부터 데이터만 받아 직접 render하는 형식이 됐다.
- CSR에서는 front 서버가 생겨나면서 backend 서버와 완전히 분리되면서 port도 분리되었다. 서로 다른 origin(출처)로 여겨지게 된 것이다.
- 이제는 sop를 적용할 수가 없었다. 그런 상황에서 cors(cross-origin-resources-sharing)가 등장했다. 
- client에서 보낸 origin header의 값이 server에서 보낸 acess-control-allow-origin header의 값에 포함되면 브라우저는 server에서 온 response를 차단하지 않았다.
- 보통 4가지 header를 설정해서 server에서 보내게 된다.
  - access-control-allow-origin     
  - access-control-allow-credentials
  - access-control-allow-header
  - access-control-allow-method
- 서버에서는 filter가 따로 없다면 request를 받으면 response를 하기 때문에 resource가 바뀌게 될 위험이 있다.
- 따라서 browser는 resource가 바뀌는 걸 방지하기 위해 cors 위반 여부를 먼저 검사하는 request를 날리게 된다.
- 그 때 method는 options이며, 이를 preflight라고한다. preflight는 status Code가 중요하지 않다. 도메인이 맞냐 안맞냐만 browser가 따진다.
- Spring에서는 별도의 CorsConfig를 만들지 않으면 전부 허용하게끔 되어있기 때문에 만드는 게 좋다.
```java
@Configuration
@Slf4j
public class CorsConfig {
	
	@Bean 
	public CorsFilter corsFilter()	{
        .
        .
        .
		config.addAllowedOrigin("http://localhost:3000");
		config.setAllowCredentials(true); 
		config.addAllowedMethod("GET");
		config.addAllowedMethod("POST");
		config.addAllowedMethod("DELETE");
		config.addAllowedMethod("PUT");
		config.addAllowedMethod("PATCH");
		config.addAllowedMethod("*");
        .
        .
        .
	}
	
}
```


## <span style="color:#802548">_csrf_</span> 

- CSRF는 인증된 사용자가 web-app에 특정 request를 보내게 유도하는 보안 공격이다.
- 사용자가 인증한 경우, 요청이 사용자의 동의를 받았는지 확인이 어려운 web의 특성을 악용한 것이다.
- 아래는 간단한 예시다.
```
특정 은행계좌에서 해커의 계좌로 송금하라는 api를 만듦
해당 api를 호출하는 하이퍼링크를 웹사이트에 뿌려둠
해당 링크를 누르면 송금하는 api가 호출되어 송금이 이뤄짐
```

- CSRF를 방어하는 코드를 만드는 방법으로는 대표적으로 세 개가 있다.
  - referrer 검증
    - host와 referrer가 같지 않은 경우는 대부분 악의적인 경우에 의해 위조된 요청이기에 그냥 걸러버린다.
    - 피싱 사이트가 아니라 해당 웹사이트 내에서 일어난 일이면 취약하다. 
  - captcha: 사용자가 의도한 요청인지, 모르게 작동한 요청인지 거를 수 있다. 원인을 모르는 capcha에 대해서도 그냥 다 해주는 소비자의 경우는 무의미하다.
  - CSRF 토큰
    - 서버에서 만든 중요한 request의 경우에는 csrf token을 만들어서 hidden 값으로 넣어두고 서버에서 검증한다. 
    - sop나 cors를 프론트서버로만 설정하면 attack들은 script로 csrf token을 가져올 수 없으니 서버에 저장된 session과 input hidden의 csrf token이 달라 걸러진다.
    - 가장 안전하지만 가장 귀찮은 방식이다. 매 요청마다 해줘야 하기 때문이다.


- referrer 검증을 Java소스코드로 쓰면 아래와 같다.
```java
public class ReferrerCheck implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String referer = request.getHeader("Referer");
        String host = request.getHeader("host");
        if (referer == null || !referer.contains(host)) {
            response.sendRedirect("/");
            return false;
        }
        return true;
    }
}
```

- csrf토큰을 Java소스코드로 쓰면 아래와 같다.
```java
session.setAttribute("CSRF_TOKEN", UUID.randomUUID().toString());

public class CsrfTokenInterceptor implements HandlerInterceptor {
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        HttpSession httpSession = request.getSession();
        String csrfTokenParam = request.getParameter("CSRF_TOKEN");
        String csrfTokenSession = (String) httpSession.getAttribute("CSRF_TOKEN");
        if (csrfTokenParam == null || !csrfTokenParam.equals(csrfTokenSession)) {
            response.sendRedirect("/");
            return false;
        }
        return true;
    }
}
```


## <span style="color:#802548">_xss_</span> 
- 악의적인 사용자가 공격하려는 사이트에 스크립트를 넣어 정보를 탈취하는 공격이다.
- 간단한 예시는 아래와 같다.
```
악의적인 사용자가 보안이 취약한 사이트를 발견했습니다.
보안이 취약한 사이트에서 사용자 정보를 빼돌릴 수 있는 스크립트가 담긴 URL을 만들어 일반 사용자에게 스팸 메일로 전달합니다.
일반 사용자는 메일을 통해 전달받은 URL 링크를 클릭합니다. 일반 사용자 브라우저에서 보안이 취약한 사이트로 요청을 전달합니다.
일반 사용자의 브라우저에서 응답 메시지를 실행하면서 악성 스크립트가 실행됩니다.
악성 스크립트를 통해 사용자 정보가 악의적인 사용자에게 전달됩니다.

```

- 종류는 3가지이다.
  - reflected xss: 
    - 악의적인 사용자가 악성 스크립트가 담긴 URL을 만들어 일반 사용자에게 전달하는 경우
    - 검색키워드를 통해 url script 심기가 통하는지 본 뒤에, 통하면 url 단축기술을 악용해 악성 url 생성
  - stored xss:    
    - 보안이 취약한 서버에 악의적인 사용자가 악성 스크립트를 저장하는 경우
    - 대표적으로 게시글 작성을 악용
  - dom-based xss: 
    - 보안에 취약한 JavaScript 코드로 DOM 객체를 제어하는 경우
    - 대표적으로 url hash를 악용

- 방어법은 아래와 같다.
  - '<', '>' 와 같이 태그에 사용되는 기호를 엔티티코드로 변환
  - 서버에서 유효성 검증 수행
  - SCP 헤더 설정
  - 취약한 js 코드 사용 금지

- 아래는 Java에서 악성 요청이 들어온 경우 이를 엔티티코드로 변환하기 위한 filter class다.
- 해당 filter class를 wrapper로 감싸서 만들어 준다.
```java
package blog.in.action.filter;

import org.springframework.stereotype.Component;

import javax.servlet.*;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletRequestWrapper;
import java.io.IOException;

@Component
public class XssAttackFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        filterChain.doFilter(new RequestWrapper((HttpServletRequest) servletRequest), servletResponse);
    }

    @Override
    public void destroy() {

    }

    private class RequestWrapper extends HttpServletRequestWrapper {

        public RequestWrapper(HttpServletRequest request) {
            super(request);
        }

        @Override
        public String[] getParameterValues(String parameter) {
            String[] values = super.getParameterValues(parameter);
            if (values == null) {
                return null;
            }
            int count = values.length;
            String[] encodedValues = new String[count];
            for (int i = 0; i < count; i++) {
                encodedValues[i] = cleanXSS(values[i]);
            }
            return encodedValues;
        }

        @Override
        public String getParameter(String parameter) {
            String value = super.getParameter(parameter);
            if (value == null) {
                return null;
            }
            return cleanXSS(value);
        }

        @Override
        public String getHeader(String name) {
            String value = super.getHeader(name);
            if (value == null) {
                return null;
            }
            return cleanXSS(value);
        }

        private String cleanXSS(String value) {
            value = value.replaceAll("&", "&amp;");
            value = value.replaceAll("<", "&lt;").replaceAll(">", "&gt;");
            value = value.replaceAll("\\(", "&#40;").replaceAll("\\)", "&#41;");
            value = value.replaceAll("/", "&#x2F;");
            value = value.replaceAll("'", "&#x27;");
            value = value.replaceAll("\"", "&quot;");
            return value;
        }
    }
}
```

- 아래는 취약하다고 일컬어지는 js코드다.
```js
document.write()
document.writeln()
document.domain
element.innerHTML
element.outerHTML
element.insertAdjacentHTML
element.onevent
```

- 아래는 CSP 헤더 설정이다.
- script-src는 엄격하지 않은 정책이라 nonce나 hash를 쓰는 걸 구글은 추천하고 있다.
- 다만 어떤 값의 nonce나 hash를 쓰는지 드러나면 무용지물이기 때문에 source code 난독화 혹은 bundling 없이는 쓰기 힘들어 보인다.


## <span style="color:#802548">_출처_</span> 
https://jaeseongdev.github.io/development/2021/06/15/REST%EC%9D%98-%EA%B8%B0%EB%B3%B8-%EC%9B%90%EC%B9%99-6%EA%B0%80%EC%A7%80/ - RESTFUL api
https://www.bugbountyclub.com/pentestgym/view/47 - csrf 
https://devscb.tistory.com/123 - csrf
https://junhyunny.github.io/information/security/spring-mvc/reflected-cross-site-scripting/ - xss
https://portswigger.net/web-security/cross-site-scripting/dom-based - xss
https://developer.chrome.com/docs/lighthouse/best-practices/csp-xss?hl=ko - csp