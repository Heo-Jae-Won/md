## <span style="color:#802548">_1. logging을 위한 entity class_</span>
- entity를 만들기 위해 아래와 같이 column을 구성한다.

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
@Table(name = "MEMBER_LOGGER")
public class MemberLogger {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long loggerNo;
    private String ip;
    private String memberId;
    private String requestMethod;
    private String requestUrl;
    private String requestUri;
    private String queryString;
    private String refererPage;
    private String userAgent;
    @CreationTimestamp
    private LocalDateTime loggedTime;

    public MemberLogger(String ip, String memberId, String requestMethod, String requestUri, String queryString,
            String refererPage, String userAgent) {
        this.ip = ip;
        this.memberId = memberId;
        this.requestMethod = requestMethod;
        this.requestUri = requestUri;
        this.queryString = queryString;
        this.refererPage = refererPage;
        this.userAgent = userAgent;
    }
}
```
## <span style="color:#802548">_2. logging을 위한 repository class_</span>
- logging은 저장 기능밖에 없기 때문에 저장하는 persist()만 만들어놓는다.

```java
@Repository
public class MemberActivityRepository {

    @PersistenceContext
    private EntityManager entityManager;

    public void save(MemberLogger userLogger) {
        entityManager.persist(userLogger);
    }

}
```

## <span style="color:#802548">_3. logging을 위한 service class_</span>
- repository를 이용해 logging entity를 저장하는 service class를 만든다.

```java
@Service
@RequiredArgsConstructor
@Transactional
public class MemberActivityService {

	private final MemberActivityRepository memberActivityRepository;

	public void save(String ip, String memberId, String requestMethod, String requestURI,
			String refererPage,
			String queryString,
			String userAgent) {

		MemberLogger userActivity = new MemberLogger(ip, memberId, requestMethod, requestURI, queryString,
				refererPage, userAgent);

		memberActivityRepository.save(userActivity);
	}

}
```

## <span style="color:#802548">_4. logging을 위한 util class_</span>
- 아래와 같이 request에 값이 있는지 확인하고 가져온다.

```java
if (RequestContextHolder.getRequestAttributes() == null) {
	return "";
}

HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
				.getRequest();
```

- 아래와 같이 proxy ip가 있으면 proxy ip를 반환하고, 그게 아니라면 자신의 ip를 반환한다.
- localhost는 0.0.0.0을 반환한다.

```java
private static final String[] IP_HEADER_CANDIDATES = { "X-Forwarded-For", "Proxy-Client-IP", "WL-Proxy-Client-IP",
			"HTTP_X_FORWARDED_FOR", "HTTP_X_FORWARDED", "HTTP_X_CLUSTER_CLIENT_IP", "HTTP_CLIENT_IP",
			"HTTP_FORWARDED_FOR", "HTTP_FORWARDED", "HTTP_VIA", "REMOTE_ADDR" };

public static String getClientIpAddress() {

		if (RequestContextHolder.getRequestAttributes() == null) {
			return "0.0.0.0";
		}

		HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
				.getRequest();

		for (String header : IP_HEADER_CANDIDATES) {
			String ipList = request.getHeader(header);
			if (ipList != null && ipList.length() != 0 && !"unknown".equalsIgnoreCase(ipList)) {
				String ip = ipList.split(",")[0];
				return ip;
			}
		}

		return request.getRemoteAddr();
	}
```

- userId의 경우 SecurityContext에서 얻어온다. 
- 만약에 SecurityContextHolder에 값이 없다면 빈값을 반환한다. 
- principal의 경우 로그인을 하지 않은 상태에서 SecurityContextHolder에서 꺼내면 String 형변환을 해야 하며, 값은 anonymousUser로 로그인이 된 상태라면 User로 형변환해야 하고, 값은 object라서 한 차례 더 들어가서 꺼내야 한다. 

```java
public static String getUserId() {
		String userId = null;
		if (RequestContextHolder.getRequestAttributes() == null) {
			return "";
		}

			Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		if (authentication != null && authentication.getPrincipal() instanceof User) {
			User user = (User) authentication.getPrincipal();
			userId = user.getUsername();
		} else {
			userId = "";
		}

		return userId;

	}
```

- 참고로 controller에서 그냥 Principal principal로 값을 가져온다면 로그인을 하지 않았을 때 값이 null로 나온다. 
- 즉, Authentication을 거치지 않은 경우에는 SecurityContextHolder를 거치면 annoymousUser, 안 거치면 null로  값이 달라진다.
- 아래가 합본이다. 간단한 내용은 추가 설명하지 않았다.

```java
package com.example.reservation.activityLogging;

import javax.servlet.http.HttpServletRequest;

import org.springframework.security.core.Authentication;
import org.springframework.security.core.context.SecurityContextHolder;
import org.springframework.security.core.userdetails.User;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;

public final class HttpRequestResponseUtils {

	private HttpRequestResponseUtils() {
	}

	private static final String[] IP_HEADER_CANDIDATES = { "X-Forwarded-For", "Proxy-Client-IP", "WL-Proxy-Client-IP",
			"HTTP_X_FORWARDED_FOR", "HTTP_X_FORWARDED", "HTTP_X_CLUSTER_CLIENT_IP", "HTTP_CLIENT_IP",
			"HTTP_FORWARDED_FOR", "HTTP_FORWARDED", "HTTP_VIA", "REMOTE_ADDR" };

	public static String getClientIpAddress() {

		if (RequestContextHolder.getRequestAttributes() == null) {
			return "0.0.0.0";
		}

		HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
				.getRequest();

		for (String header : IP_HEADER_CANDIDATES) {
			String ipList = request.getHeader(header);
			if (ipList != null && ipList.length() != 0 && !"unknown".equalsIgnoreCase(ipList)) {
				String ip = ipList.split(",")[0];
				return ip;
			}
		}

		return request.getRemoteAddr();
	}

	public static String getRequestUri() {

		if (RequestContextHolder.getRequestAttributes() == null) {
			return "";
		}

		HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
				.getRequest();

		return request.getRequestURI();
	}

	public static String getRefererPage() {

		if (RequestContextHolder.getRequestAttributes() == null) {
			return "";
		}

		HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
				.getRequest();

		String referer = request.getHeader("Referer");

		return referer != null ? referer : request.getHeader("referer");
	}

	public static String getPageQueryString() {

		if (RequestContextHolder.getRequestAttributes() == null) {
			return "";
		}

		HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
				.getRequest();

		return request.getQueryString();
	}

	public static String getUserAgent() {

		if (RequestContextHolder.getRequestAttributes() == null) {
			return "";
		}

		HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
				.getRequest();

		String userAgent = request.getHeader("User-Agent");

		return userAgent != null ? userAgent : request.getHeader("user-agent");
	}

	public static String getRequestMethod() {

		if (RequestContextHolder.getRequestAttributes() == null) {
			return "";
		}

		HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes())
				.getRequest();

		return request.getMethod();
	}

	public static String getUserId() {
		String userId = null;
		if (RequestContextHolder.getRequestAttributes() == null) {
			return "";
		}

		Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
		if (authentication != null && authentication.getPrincipal() instanceof User) {
			User user = (User) authentication.getPrincipal();
			userId = user.getUsername();
		} else {
			userId = "";
		}

		return userId;

	}

}
```

## <span style="color:#802548">_5. interceptor 활용하기_</span>
- 아래와 같이 HandlerInterceptor를 implements한 class를 만들고 @Component를 붙여 bean으로 만든다. 
- logging할 것들은 아래와 같은 것으로 만들었다. 

```java
@Component
@RequiredArgsConstructor
@Slf4j
public class MemberActivityInterceptor implements HandlerInterceptor {

    private final MemberActivityService userActivityService;

    private static final List<String> PUBLIC_URI = Arrays.asList(
            "/api/signin", "/api/signup", "/api/logout", "/api/auth/refreshToken");

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
            throws Exception {

        String uri = request.getRequestURI();
        if (PUBLIC_URI.stream().anyMatch(uri::startsWith)) {
            return true;
        }

        final String ip = HttpRequestResponseUtils.getClientIpAddress();
        final String memberId = HttpRequestResponseUtils.getUserId();
        final String requestURI = HttpRequestResponseUtils.getRequestUri();
        final String refererPage = HttpRequestResponseUtils.getRefererPage();
        final String queryString = HttpRequestResponseUtils.getPageQueryString();
        final String userAgent = HttpRequestResponseUtils.getUserAgent();
        final String requestMethod = HttpRequestResponseUtils.getRequestMethod();

        userActivityService.save(ip, memberId, requestMethod, requestURI, refererPage, queryString,
                userAgent);

        return true;
    }

}
```

- 그리고 implements WebMvcConfigurer를 한 class에 addInterceptor() method를 override하여 custom interceptor를 구현한다.

```java
@Configuration
@EnableWebMvc
@RequiredArgsConstructor
public class WebConfig implements WebMvcConfigurer {

    private final MemberActivityInterceptor memberActivityInterceptor;

@Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(memberActivityInterceptor);
    }
}
```
## <span style="color:#802548">_6. aop 활용하기_</span>
- 일단 pom.xml에 aop를 넣어준다.

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aop</artifactId>
</dependency>
```

- main class에 @EnableAspectJAutoProxy를 달아준다. 그럼 이제 aop를 활용할 수 있다.

```java
@SpringBootApplication
@EnableAspectJAutoProxy
@EnableJpaAuditing
public class ReservationApplication {

	public static void main(String[] args) {
		SpringApplication.run(ReservationApplication.class, args);
	}

}
```
- aop를 활용하기 위한 기본 개념으로 Pointcut과 advice가 있다. 
- pointcut은 내가 적용할 method 혹은 package를 인자로 넣고, advice에는 적용될 pointcut의 method name을 annotation의 인자로 넣어준다. 
- 그리고 그 아래에는 실제 곁다리로 실행할 작업을 전개한다. 대표적인 게 오늘 하는 logging 작업이다. 

​

- within으로 package 전체를 대상으로 만든 뒤에, 내가 원하는 method만 따로 뺀다. 
- method를 빼는 것은 !execution으로 한다. 맨 앞의 *는 return type인데, 뭐가 올 지 모르니 whildcard로 달아줬다. 
- 한 칸 띄고 해당 method의 name을 써준다. 뒤에 (..)은 parameter의 갯수인데, 몇 개가 들어올 지 모르니 ..으로 달아준다.

```java
@Pointcut("within(com.example.reservation.controller.*) && !execution(* com.example.reservation.controller.SignController.login(..))")
```
- 그리고 advice의 인자로는 위에서 서술했던 exceptSignIn()으로 넣어준다.

```java
@Before("exceptSignIn()")
```

- 아래와 같은 형태다. 지금은 JoinPoint를 활용하지 않는다. 필요가 없기 때문이다.

```java
@Pointcut("within(com.example.reservation.controller.*) && !execution(* com.example.reservation.controller.SignController.login(..))")
    public void exceptSignIn() {}

@Before("exceptSignIn()")
    public void beforeRequest(JoinPoint joinPoint) {
        final String ip = HttpRequestResponseUtils.getClientIpAddress();
        final String memberId = HttpRequestResponseUtils.getUserId();
        final String requestURI = HttpRequestResponseUtils.getRequestUri();
        final String refererPage = HttpRequestResponseUtils.getRefererPage();
        final String queryString = HttpRequestResponseUtils.getPageQueryString();
        final String userAgent = HttpRequestResponseUtils.getUserAgent();
        final String requestMethod = HttpRequestResponseUtils.getRequestMethod();

        userActivityService.save(ip, memberId, requestMethod, requestURI, refererPage, queryString,
                userAgent);

    }
```
- 하지만 이제는 Joinpoint를 사용한다. Joinpoint를 활용해서 advice에 포함된 method의 argument를 가져온다. 
- 따라서 아래 SignInRequest class와 HttpSession calss를 가져온다고 생각하면 된다.

```java
 public JwtResponse login(@RequestBody SignInRequest signInRequestDto, HttpSession session)
            throws Exception
```
- returning은 result라고 명시하면 아래 method에서도 parameter 이름을 result로 똑같이 맞춰야 한다. 

```java
@AfterReturning(pointcut = "signIn()", returning = "result")
    public void AfterRequest(JoinPoint joinPoint, Object result) {
    }
```
- 로그인은 SingInRequest class가 첫번째로 들어오니 0번째 index를 가져온다. 
- Object class라서 형변환이 필요하다. 해당 class로 형변환해주자. 나는 그게 SignInRequest class다.

```java
    Object[] args = joinPoint.getArgs();
    SignInRequest signInRequest = (SignInRequest) args[0]; 
```

- 아래가 합본이다.

```java
@Pointcut("execution(* com.example.reservation.controller.SignController.login(..))")
public void signIn() {
}

@AfterReturning(pointcut = "signIn()", returning = "result")
public void AfterRequest(JoinPoint joinPoint, Object result) {
    Object[] args = joinPoint.getArgs();
    SignInRequest signInRequest = (SignInRequest) args[0]; 

    final String ip = HttpRequestResponseUtils.getClientIpAddress();
    final String memberId = signInRequest.getMemberId();
    final String requestURI = HttpRequestResponseUtils.getRequestUri();
    final String refererPage = HttpRequestResponseUtils.getRefererPage();
    final String queryString = HttpRequestResponseUtils.getPageQueryString();
    final String userAgent = HttpRequestResponseUtils.getUserAgent();
    final String requestMethod = HttpRequestResponseUtils.getRequestMethod();

    userActivityService.save(ip, memberId, requestMethod, requestURI, refererPage, queryString,
            userAgent);

}
```

- 아래는 전체 합본이다.

```java
@Component
@Aspect
@RequiredArgsConstructor
public class LogAspect {

    private final MemberActivityService userActivityService;

    @Pointcut("within(com.example.reservation.controller.*) && !execution(* com.example.reservation.controller.SignController.login(..))")
    public void exceptSignIn() {
    }

    @Before("exceptSignIn()")
    public void beforeRequest(JoinPoint joinPoint) {
        final String ip = HttpRequestResponseUtils.getClientIpAddress();
        final String memberId = HttpRequestResponseUtils.getUserId();
        final String requestURI = HttpRequestResponseUtils.getRequestUri();
        final String refererPage = HttpRequestResponseUtils.getRefererPage();
        final String queryString = HttpRequestResponseUtils.getPageQueryString();
        final String userAgent = HttpRequestResponseUtils.getUserAgent();
        final String requestMethod = HttpRequestResponseUtils.getRequestMethod();

        userActivityService.save(ip, memberId, requestMethod, requestURI, refererPage, queryString,
                userAgent);

    }


    @Pointcut("execution(* com.example.reservation.controller.SignController.login(..))")
    public void signIn() {
    }

  
    @AfterReturning(pointcut = "signIn()", returning = "result")
    public void AfterRequest(JoinPoint joinPoint, Object result) {
        Object[] args = joinPoint.getArgs();
        SignInRequest signInRequest = (SignInRequest) args[0]; 
        final String ip = HttpRequestResponseUtils.getClientIpAddress();
        final String memberId = signInRequest.getMemberId();
        final String requestURI = HttpRequestResponseUtils.getRequestUri();
        final String refererPage = HttpRequestResponseUtils.getRefererPage();
        final String queryString = HttpRequestResponseUtils.getPageQueryString();
        final String userAgent = HttpRequestResponseUtils.getUserAgent();
        final String requestMethod = HttpRequestResponseUtils.getRequestMethod();

        userActivityService.save(ip, memberId, requestMethod, requestURI, refererPage, queryString,
                userAgent);

    }

}
```