## <span style="color:#802548">_1. jjwt dependency add_</span>
- 먼저 jjwt dependency를 넣어준다. 

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId> <!-- or jjwt-gson if Gson is preferred -->
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
```
## <span style="color:#802548">_2. JwtTokenProvider - key 생성_</span>
- JwtTokenProvider instance를 생성할 때 secret을 받게 강제한다. 
- 받은 secret으로 key를 만든다. jjwt library의 HS256은  대칭키 방식이다. 비대칭키 방식은 아니다. 
- 그래서 전자서명에 쓰일 개인키와 암호화에 쓰일 공용키 총 두 개가 아니라 여기서는 그냥 key 하나로 다 이뤄진다. 
```java
 private final Key key;

public JwtTokenProvider(@Value("${secret}") String secretKey) {
    byte[] keyBytes = Decoders.BASE64.decode(secretKey); // base64 복호화.
    this.key = Keys.hmacShaKeyFor(keyBytes);
}
```
- secret은 따로 properties 파일에 저장해두고 @Value로 불러온다.


## <span style="color:#802548">_3. JwtTokenProvider - jwt token생성_</span>
- 토큰을 생성하는 데 필요한 아이디나 권한은 Authentication class에서 받아올 수 있다.  
- 아래는 authentication.getAuthorities()를 통해 Collections을 가져온다. 
- 해당 Colletcion을 Java의 Stream class의 map()를 이용해 내용을 빼낸다. 
- getAuthority는 UserDetails의 getAuthority()이며 Custom해 둔 UserDetails를 구현한 class를 통해 member의 role을 얻어온다. 
- member의 권한이 ROLE_ADMIN, ROLE_MEMBER라면 ROLE_ADMIN과 ROLE_MEMBER로 나타난다. 
- 이를 .collect(Collectors.joining(",")) 를 활용해 콧마(,) 구분자로 이어준다. 

```java
public JwtResponse generateAccessTokenBySignIn(Authentication authentication) {

    String authorities = authentication.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority) 
            .collect(Collectors.joining(","));
    log.info("authorities: {}",authorities);
}
```
- 그럼 authorities는 ROLE_ADMIN,ROLE_MEMBER라고 나타나게 된다. 
```
authorities: ROLE_ADMIN,ROLE_MEMBER
```
- 그 다음으로 얻어온 권한을 활용해 토큰을 생성한다. 
- 토큰의 만료시간은 현재시간으로 부터 10분으로 설정했다. 

- ms 단위기 때문에 1000을 곱하면 1초라고 생각하면 된다. 그래서 나는 accessToken을 10분 단위로 설정했다. 
  - 10분이 지나면 만료되어 다시 로그인이 필요하다. 
  - 전자서명은 위에서 class가 생성될 때 만들어진 key를 활용한다. 
```java
long now = (new Date()).getTime();

    // Access Token 생성. 10분.
Date accessTokenExpiresAt = new Date(now + 60 * 10 * 1000);
String accessToken = Jwts.builder()
        .setSubject(authentication.getName())
        .claim("auth", authorities) // 토큰에 담을 정보
        .setExpiration(accessTokenExpiresAt)
        .signWith(key, SignatureAlgorithm.HS256)
        .compact();
```
- 값이 만들어졌다면 JwtDto instance를 builder()를 활용해 생성한다. 
- JwtDto라는 class를 만들지 않아도 무방하다. 나는 편의상 만든 것일 뿐이다. 

```java
return JwtResponse.builder()
        .grantType("Bearer")
        .accessToken(accessToken)
        .build();
```
- JwtResponse를 왜 builder를 통해 생성하냐고 묻는다면 실수가 덜 나기 때문이다. 아래 내용을 참조하라.

https://jung-story.tistory.com/131
```java
@Builder
@Data
@AllArgsConstructor
public class JwtResponse {
 
    private String grantType;
    private String accessToken;
    private String refreshToken;
}
```
아래가 해당 method의 전체 내용이다.
```java
public JwtResponse generateAccessTokenBySignIn(Authentication authentication) {
    String authorities = authentication.getAuthorities().stream()
            .map(GrantedAuthority::getAuthority)
            .collect(Collectors.joining(","));

    long now = (new Date()).getTime();

    
    Date accessTokenExpiresAt = new Date(now + 60 * 10 * 1000);
    String accessToken = Jwts.builder()
            .setSubject(authentication.getName()) //이용자 이름
            .claim("auth", authorities) //권한
            .setExpiration(accessTokenExpiresAt) //만료기한
            .signWith(key, SignatureAlgorithm.HS256) //서명키
            .compact();

    
    String memberPK = authentication.getName(); // principalDetails의 username으로 저장된 값을 가져온다.
        String accessToken = Jwts.builder()
                .setSubject(memberPK)
                .claim("auth", authorities)
                .setExpiration(accessTokenExpiresAt)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();

        return accessToken;
}
```
## <span style="color:#802548">_4. JwtTokenProvider - jwt token 복호화_</span>
- 먼저 받아온 accessToken을 복호화한다. 
  - jwt token을 복호화하여 usernamePasswordAuthenticationToken을 만드는 게 목적이다. 
  - Authentication 객체가 usernamePasswordAuthenticationToken을 받아 생성되기 때문이다. 
  - 권한이 없으면 잘못된 토큰이기에 Exception을 던져버린다. 
```java
 public Authentication getAuthentication(String accessToken) {
    Claims claims = parseClaims(accessToken);

    if (claims.get("auth") == null) {
        throw new RuntimeException("권한 정보가 없는 토큰입니다.");
    }
 }
 ```
- jjwt에 있는 method를 사용하면 복호화를 알아서 해준다. 
- 여기서도 이전에 만들어 둔 key를  사용한다. 

```java
private Claims parseClaims(String accessToken) {
    try {
        return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(accessToken).getBody();
    } catch (ExpiredJwtException e) {
        return e.getClaims(); 
    }
}
```
- 아래는 권한정보를 가져오는 method이다. UserDetails 객체를 만들어서 Authentication 객체를 return한다.
- 비밀번호는 값이 비어있는데, 이는 accessToken의 claim에 비밀번호가 들어가면 안되기 때문에 넣지 않아서 값을 애초에 가져올 수 없어서 그렇다.
```java
Collection<? extends GrantedAuthority> authorities = 
                 Arrays.stream(claims.get("auth").toString().split(","))
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());

                UserDetails principal = new User(claims.getSubject(), "", authorities);
                log.info("pirincipal memberId: {}",principal.getUsername());

    return new UsernamePasswordAuthenticationToken(principal, "", authorities);
```

- 아래 method는 getAuthentication()의 내용은 아니다만 이어지는 내용이라 가져왔다. 
- 클라이언트와 서버가 통신할 때 아래와 같이 jwt token을 받아 usernamePasswordToken으로 만들어 SecurityContext에 저장하는 용도다. 
- 한번 로그인하고 나서는 JWT를 사용하여 인증한다면 굳이 이러한 SecurityContextHolder에 usernamePasswordToken을 저장하는 작업이 필요할까 의문이 들어 지워봤는데, 401 오류가 뜬다. 매번 contextHolder에 저장해야 하는 것으로 보인다.
```java
 if (token != null && jwtTokenProvider.validateToken(token)) {
    Authentication authentication = jwtTokenProvider.getAuthentication(token);
    SecurityContextHolder.getContext().setAuthentication(authentication);
}
```
- 아래가 전체 method의 내용이다.

```java
public Authentication getAuthentication(String accessToken) {
    Claims claims = parseClaims(accessToken);

    if (claims.get("auth") == null) {
        throw new RuntimeException("권한 정보가 없는 토큰입니다.");
    }

    Collection<? extends GrantedAuthority> authorities = Arrays.stream(claims.get("auth").toString().split(""))
            .map(SimpleGrantedAuthority::new)
            .collect(Collectors.toList());

            UserDetails principal = new User(claims.getSubject(), "", authorities);
            log.info("pirincipal memberId: {}",principal.getUsername());
    return new UsernamePasswordAuthenticationToken(principal, "", authorities);
}
```
## <span style="color:#802548">_5. JwtTokenProvider - jwt token 검증_</span>
- 검증을 하는 것인데 복호화를 하다 오류가 나면 그 오류를 catch하는 형태다. 
- 탈취당한 것이 아니라면 보통 ExpiredJwtException에 걸리게 된다. 
```java
public boolean validateToken(String token) {
    try {
        Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token);
        return true;
    } catch (io.jsonwebtoken.security.SecurityException | MalformedJwtException e) {
        log.info("Invalid JWT Token", e);
    } catch (ExpiredJwtException e) {
        log.info("Expired JWT Token", e);
    } catch (UnsupportedJwtException e) {
        log.info("Unsupported JWT Token", e);
    } catch (IllegalArgumentException e) {
        log.info("JWT claims string is empty.", e);
    }
    return false;
}
```
- 마지막으로 refreshToken에 의해 받는 accessToken method도 추가해준다. generateAccessTokenByRefreshToken다.
- 다만 위에서 accessToken과 겹치지 않게 method명을 좀 바꿨다.
- 또한 refreshToken와 accessToken이 달라야 해서, refresToken에서는 authroity를 넣지 않았다.
```java
@Component
@Slf4j
public class JwtTokenProvider {

    private final Key key;
    private final MemberService memberService;

    public String generateAccessTokenBySignIn(Authentication authentication) {
        String authorities = authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(","));

        Date accessTokenExpiresAt = new Date(new Date().getTime() + 60 * 30 * 1000);

        String accessToken = Jwts.builder()
                .setSubject(authentication.getName())
                .claim("auth", authorities)
                .setExpiration(accessTokenExpiresAt)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();

        return accessToken;
    }

    public String generateAccessTokenByRefreshToken(String refreshToken) {

        Date accessTokenExpiresAt = new Date(new Date().getTime() + 60 * 30 * 1000);

        Claims claims = parseClaims(refreshToken);
        Member memberInfo = memberService.getMemberInfo(claims.getSubject());

        String accessToken = Jwts.builder()
                .setSubject(claims.getSubject())
                .claim("auth", memberInfo.getRoleType().getValue())
                .setExpiration(accessTokenExpiresAt)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();

        return accessToken;
    }

    public String generateRefreshTokenBySignIn(String memberId) {

        String refreshToken = Jwts.builder()
                .setSubject(memberId)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();

        return refreshToken;
    }
}
```
- 아래는 JwtTokenProvider class 전체 내용이다.
```java
@Component
@Slf4j
public class JwtTokenProvider {

    private final Key key;

    
    public JwtTokenProvider(@Value("${secret}") String secretKey) {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey); // base64 복호화.
        this.key = Keys.hmacShaKeyFor(keyBytes);
    }

    /*
     * 토큰을 생성한다. 필요한 것은 권한, 만료시간, key, 토큰에 담을 정보(claim), 토큰 제목-보통 user이름-(subject)이다.
     */
   public String generateAccessTokenBySignIn(Authentication authentication) {
        String authorities = authentication.getAuthorities().stream()
                .map(GrantedAuthority::getAuthority)
                .collect(Collectors.joining(","));

        Date accessTokenExpiresAt = new Date(new Date().getTime() + 60 * 30 * 1000);

        String accessToken = Jwts.builder()
                .setSubject(authentication.getName())
                .claim("auth", authorities)
                .setExpiration(accessTokenExpiresAt)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();

        return accessToken;
    }

    public String generateAccessTokenByRefreshToken(String refreshToken) {

        Date accessTokenExpiresAt = new Date(new Date().getTime() + 60 * 30 * 1000);

        Claims claims = parseClaims(refreshToken);
        Member memberInfo = memberService.getMemberInfo(claims.getSubject());

        String accessToken = Jwts.builder()
                .setSubject(claims.getSubject())
                .claim("auth", memberInfo.getRoleType().getValue())
                .setExpiration(accessTokenExpiresAt)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();

        return accessToken;
    }

    public String generateRefreshTokenBySignIn(String memberId) {

        String refreshToken = Jwts.builder()
                .setSubject(memberId)
                .signWith(key, SignatureAlgorithm.HS256)
                .compact();

        return refreshToken;
    }

    /*
     * 권한 정보를 가져오기
     */
    public Authentication getAuthentication(String accessToken) {

        Claims claims = parseClaims(accessToken);

        if (claims.get("auth") == null) {
            throw new RuntimeException("권한 정보가 없는 토큰입니다.");
        }

        // 클레임에서 권한 정보 가져오기
        Collection<? extends GrantedAuthority> authorities = Arrays.stream(claims.get("auth").toString().split(","))
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());

                // UserDetails 객체를 만들어서 Authentication 리턴. token에 password를 담을 수는 없으므로 빈칸.
                UserDetails principal = new User(claims.getSubject(), "", authorities);
                log.info("pirincipal memberId: {}",principal.getUsername());
        return new UsernamePasswordAuthenticationToken(principal, "", authorities);
    }

    /*
     * 토큰 검증
     */
    public boolean validateToken(String token) {
        try {
            Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token);
            return true;
        } catch (io.jsonwebtoken.security.SecurityException | MalformedJwtException e) {
            log.info("Invalid JWT Token", e);
        } catch (ExpiredJwtException e) {
            log.info("Expired JWT Token", e);
        } catch (UnsupportedJwtException e) {
            log.info("Unsupported JWT Token", e);
        } catch (IllegalArgumentException e) {
            log.info("JWT claims string is empty.", e);
        }
        return false;
    }

    /*
     * 토큰 복호화
     */
    private Claims parseClaims(String accessToken) {
        try {
            return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(accessToken).getBody();
        } catch (ExpiredJwtException e) {
            return e.getClaims(); //뭐가 return되는 지는 잘 모르겠음.
        }
    }
}
```
## <span style="color:#802548">_6. JwtAuthenticationFilter - token 추출_</span>
- JwtTokenProvider class를 만들어 Jwt와 관련된 util은 전부 갖춰졌다. 이제 Filter를 만들면 된다. 
- Filter는 JwtTokenProvider class를 가지고 생성되게 한다. 
- JwtTokenProvider class에 @Component를 붙였기에 DI가 가능하다. 
```java
@Slf4j
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

 private final JwtTokenProvider jwtTokenProvider;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        String token = resolveToken((HttpServletRequest) request);
        log.info("token: {}", token);
.
.   
    }
}
```
 
- 토큰은 Authorization header에서 값을 추출한다. 
- 따라서 매번 통신마다 프론트에서 보내줘야 한다. 그 처리는 다른 글에서 다룰 예정이다. 
- substring(7)은 Bearer라는 말이 6글자라서 그렇다.
  - 6글자 + 띄어쓰기 이후부터는 token값이기에 띄어쓰기까지 포함해서 7글자 이후부터 값을 받아오게 하는 것이다. 참고로
  - bearerToken.startsWith에서 Bearer라고 딱 쓰면 안되고 띄어쓰기 한 칸 해줘야 한다. 아니면 아래와 같은 오류가 난다. 
```
Bearjava.lang.StringIndexOutOfBoundsException: String index out of range: -1
```

```java
private String resolveToken(HttpServletRequest request) {
    String bearerToken = request.getHeader("Authorization");
    if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
        return bearerToken.substring(7);
    }
    return null;
} 
```
## <span style="color:#802548">_7. JwtAuthenticationFilter - SecurityContext에 저장_</span>
- 아까 이야기했던 내용이다. (4)를 참고하라. 
- 간략하게 말하자면 jwt를 복호화해 얻은 claim 값으로 usernamePasswordToken을 만들어 SecurityContext에 저장하는 것이다. context가 휘발성이라 매번 저자오디어야 한다.
```java
// 2. validateToken 으로 토큰 유효성 검사
if (token != null && jwtTokenProvider.validateToken(token)) {
    // 토큰이 유효할 경우 토큰에서 Authentication 객체를 가지고 와서 SecurityContext 에 저장
    Authentication authentication = jwtTokenProvider.getAuthentication(token);
    SecurityContextHolder.getContext().setAuthentication(authentication);
}
filterChain.doFilter(request, response);
```
- 아래는 전체 내용이다. 
- OncePerRequestFilter를 extends한 이유는 필터가 1번만 실행되도록 하기 위해서다.
```java
@Slf4j
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtTokenProvider jwtTokenProvider;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        // 1. Request Header 에서 JWT 토큰 추출
        String token = resolveToken((HttpServletRequest) request);
        log.info("token: {}", token);

        // 2. validateToken 으로 토큰 유효성 검사
        if (token != null && jwtTokenProvider.validateToken(token)) {

            // 토큰이 유효할 경우 토큰에서 Authentication 객체를 가지고 와서 SecurityContext 에 저장
            Authentication authentication = jwtTokenProvider.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
        filterChain.doFilter(request, response);
    }

    // Request Header 에서 토큰 정보 추출
    private String resolveToken(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer")) {
            return bearerToken.substring(7);
        }
        return null;
    }
}
```
## <span style="color:#802548">_8. JwtExceptionFilter - 예외처리_</span>
- 이 Filter는 그저 예외처리를 위한 filter다. 
- 사실 이 filter는 request가 들어올 때는 그냥 거수기처럼 통과시켜주고, JwtAuthenticationFilter에서 오류가 터졌을 때 상위로 전파된 오류를 잡아서 프론트에 보내주는 역할을 한다. 
- Filter는 Spring Container의 영역이 아니기 때문에 ExceptionHandler가 적용되지 않는다.
- 따로 static method를 만들어 error를 front에 보낼수도 있지만, front에서 error message를 처리하게 한다면 필요없다.
```java
public class JwtExceptionFilter extends OncePerRequestFilter {

	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws ServletException, IOException {
		try {
			chain.doFilter(request, response); // JwtAuthenticationFilter로 이동
		} catch (JwtException ex) {
			// JwtAuthenticationFilter에서 예외 발생하면 바로 setErrorResponse 호출. 
			CommonErrorCode.errorResponse(request, response, ex.getMessage(), 401);
		}
	}
}
```

- static method로 만든 것은 아래와 같다. HTTPStatus, message, 오류가 난 URI를 front에 보내주는 역핧이다.
```java
public static void errorResponse(HttpServletRequest request,
        HttpServletResponse response, String error, int status) throws JsonGenerationException, JsonMappingException, IOException{
                response.setContentType(MediaType.APPLICATION_JSON_VALUE);
                response.setStatus(status);
                final Map<String, Object> body = new HashMap<>();
                body.put("status", status);
                body.put("message",error);
                body.put("path", request.getServletPath());
                final ObjectMapper mapper = new ObjectMapper();
                mapper.writeValue(response.getOutputStream(), body);
}
```
## <span style="color:#802548">_9. CosrFilter 생성_</span>
- CorsConfig는 아래와 같이 bean으로 만든다. header나 method를 다 열어두긴 했는데, 제어 가능하다. 
- api용 백엔드로서 모든 request에 대해 corsFilter를 적용하려면 registerCorsConfiguration에 "/api/**"라고 써주면 된다. 
- origin은 하드코딩보다는 @Value로 박아두는 것이 훨씬 관리하기 편하다. 나는 간단한 프로젝트라서 이렇게 진행한 것이다.
```java
@Configuration
public class CorsConfig {

    @Bean
    public CorsFilter corsFilter() {
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        CorsConfiguration config = new CorsConfiguration();
        config.setAllowCredentials(true);
        config.addAllowedOrigin("http://localhost:3000");
                                                          
        config.addAllowedHeader("*");
        config.addAllowedMethod("*");
        source.registerCorsConfiguration("/api/**", config);
        return new CorsFilter(source);
    }
}
```
 ## <span style="color:#802548">_10.SecurityConfig - 필터 설정_</span>
- filter는 아래와 같이 만들어준다. 
- 그러면 기존의 Spring Security가 수행하는 Filter들 사이에 등록이 된다. 
- 참고로 UsernamePasswordAuthenticationFilter 말고 BasicAuthenticationFilter 앞에다가 등록하는 것도 가능하다.
```java
http.addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider),
                      UsernamePasswordAuthenticationFilter.class)
                    .addFilter(corsFilter);
```
## <span style="color:#802548">_11. SecurityConfig - 기본 설정_</span>
- 이제는 exception을 handling할 차례다. 
- csrf는 disable 했지만, 실제 프로젝트에선 disable하면 안되고 구현해야 한다. csrf는 기본이다.
```java
 http.csrf().disable()
            .httpBasic().disable()
            .exceptionHandling()
            .accessDeniedHandler(webAccessDeniedHandler) // 권한이 없는 사용자 접근 시
            .authenticationEntryPoint(webAuthenticationEntryPoint) // 인증이 없는 사용자 접근 시
            .and()
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);
```
- CustomAuthenticationEntryPoint는 아래와 같이 만든다. 
- 이것도 filter 단에서 작동하기 때문에 exceptionHandler를 거치지 않는다. 
- 따라서 위에 작성해두었던 public static method인 errorResponse()를 호출하여 error를 프론트에 던진다. 
```java
@Component
@Slf4j
public class WebAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
            AuthenticationException authException) throws IOException, ServletException {
        log.warn("인증되지 않은 이용자입니다.");

        response.sendError(401);
    }
}
```
- CustomAccessDeniedHandler는 아래와 같이 만든다. 
- 이것도 filter 단에서 작동하기 때문에 exceptionHandler를 거치지 않는다. 
```java
@Component
@Slf4j
public class WebAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
            AccessDeniedException accessDeniedException) throws IOException, ServletException {
        log.warn("권한이 없습니다.");

        response.sendError(403);
    }

}
```
## <span style="color:#802548">_12. SecurityConfig - 인증, 인가 설정_</span>
- PUBLIC_URI라는 상수에 포함된 url을 제외하고는 모두 인증이 필요하다. 
- 그 중에서도 권한이 MEMBER인 경우에만 결제 내역쪽에 들어갈 수 있다. 
- hasRole()의 경우 자동으로 접두사 ROLE_을 붙이기 때문에 db에 ROLE_MEMBER로 저장하고 hasRole()안의 argument는 MEMBER로 작성해야 한다. 
```java
// 권한 관련
http.authorizeRequests()
                .antMatchers(PUBLIC_URI).permitAll() // 전체 접근 허용
                .antMatchers("/api/member/**").authenticated() // 인증된 사용자만 접근 허용
                .antMatchers("/api/payment").hasRole("MEMBER") // ROLE_MEMBER 권한을 가진 사용자만 접근
                .anyRequest().authenticated();
```

- 인증을 무시하는 방법은 2가지가 있다.
  - 몇 url들은 filter를 거치면서 통과되어야 한다.
    - web.ignoring()은 정적 파일 혹은 filter를 전혀 거치지 않게 만들 것만 넣는다. 
  - 또 몇 url들은 아예 filter를 거치면 안된다.
    - filter를 거치면서 통과해야 하는 건 permitAll()에 넣는다.
    - 대신 NPE같은 오류가 나지 않게 해야 한다. 
- 이번 경우에는 web.ignoring을 쓸 url은 없다.
```java

private static final String[] PUBLIC_URI = {};

String[] permitUri = { "/api/signup", "/api/logout", "/api/signin", "/api/auth/refreshToken",
                        "/api/schedule/**",
                        "/api/seat/**" };

        @Override
        public void configure(WebSecurity web) throws Exception {
                web.ignoring().antMatchers(PUBLIC_URI);
        }
- 그래서 permitAll method를 활용하게 한다. 
- 이렇게 하면 SecurityFilterChain은 타게 되어 JwtAuthenticationFilter를 타긴 하지만 uri는 통과된다. 

 http.authorizeRequests()
    .antMatchers(permitUri).permitAll()
    .antMatchers("/api/payment").authenticated() // 인증된 사용자만 접근 허용
    .antMatchers("/api/member/**").hasAuthority("회원") // 회원(ROLE_MEMBER) 권한을 가진 사용자만 접근
    .anyRequest().authenticated();
```
- web.ignore()에 회원가입을 넣는다면 회원가입에서 db관련 exception이 터지는 경우, WebAuthenticationEntryPoint로 넘어가 401 error가 떨어진다. 
- permitAll()에 회원가입을 넣는다면 500오류가 떨어진다. 
- 따라서 front에서 401 error를 받아서 accessToken을 재발급하려는 경우에도 permitAll()을 사용해야 한다. web.ignoring에 넣으면 무한 loop가 일어나게 된다. 
```java
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig extends WebSecurityConfigurerAdapter {

        private final WebAccessDeniedHandler webAccessDeniedHandler;
        private final WebAuthenticationEntryPoint webAuthenticationEntryPoint;
        private final CorsFilter corsFilter;
        private final JwtTokenProvider jwtTokenProvider;

        private static final String[] PUBLIC_URI = {};

        String[] permitUri = { "/api/signup", "/api/logout", "/api/signin", "/api/auth/refreshToken",
                        "/api/schedule/**",
                        "/api/seat/**" };

        @Bean
        public PasswordEncoder passwordEncoder() {
                return new BCryptPasswordEncoder();
        }

        @Override
        public void configure(WebSecurity web) throws Exception {
                web.ignoring().antMatchers(PUBLIC_URI);
        }

        // 스프링 시큐리티 규칙
        @Override
        protected void configure(HttpSecurity http) throws Exception {

            http.formLogin().disable();

            web.ignoring().antMatchers(PUBLIC_URI);
            // filter 관련.
            http.addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider),                UsernamePasswordAuthenticationFilter.class)
                .addFilter(corsFilter);

            // 설정 관련
            http.csrf().disable()
                .httpBasic().disable()
                .exceptionHandling()
                .accessDeniedHandler(webAccessDeniedHandler) // 권한이 없는 사용자 접근 시
                .authenticationEntryPoint(webAuthenticationEntryPoint) // 인증이 없는 사용자 접근 시
                .and()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);

            // 권한 관련
            http.authorizeRequests()
                .antMatchers(permitUri).permitAll()
                .antMatchers("/api/payment").authenticated() // 인증된 사용자만 접근 허용
                .antMatchers("/api/member/**").hasAuthority("회원") // 회원(ROLE_MEMBER) 권한을 가진 사용자만 접근
                .anyRequest().authenticated();
        }

}
```
## <span style="color:#802548">_13. SignController_</span>
- filter를 거쳐 controller에 도착하면 바로 service를 호출한다. 
- 사실 첫 로그인 때는 filter에서 하는 거 없이 그냥 바로 거수기마냥 통과된다. 
- JwtAuthenticationFilter는 로그인 이후부터 제대로 동작한다고 보면 된다. 
```java
@PostMapping("/api/signin")
    @ResponseBody
    public JwtResponse login(@RequestBody SignInRequest signInRequestDto)
            throws Exception {
        JwtResponse jwtResponse = signService.login(signInRequestDto.getMemberId(), signInRequestDto.getMemberPw());

        return jwtResponse;
    }
```
## <span style="color:#802548">_14. SignService_</span>
- SecurityConfig에 @EnableWebSecurity를 달아주면 AuthenticationManagerBuilder와 AuthenticationManager와 AuthenticationProvider가 자동으로 Spring Container에 등록된다. 
- 물론 이건 default기 때문에 추가 사항을 더 구현하려면 interface를 override하여 구현해야 한다. 
- 하지만 나는 굳이 그럴 필요가 없어서 override 하지 않았다. 
- 보통 AuthenticationProvider interface만 override하는 경우가 많은 것으로 보인다. 
```java
public class SignService {

    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtTokenProvider jwtTokenProvider;
    private final AuthenticationManagerBuilder authenticationManagerBuilder; 
}
```
- JPARepository를 extends한 memberRepository에서 해당 member의 정보를 가져온다. 
- JPARepository는 지정된 method의 경우 지정된 쿼리문을 보낼 수 있어 편리하다. 
- findBy~(조회), save(저장), deleteBy~(삭제) 등이 그러한 예시다. 
- 회원정보가 없을 경우를 먼저 처리하고, 회원정보가 있으나 비밀번호가 일치하지 않는 경우를 처리한다. 
- 자세한 정보를 알려주기 싫으면 둘 다 묶어 퉁쳐서 아이디나 비밀번호가 잘못되었다는 오류를 넘기면 된다. 
```java
public JwtResponse login(String memberId, String memberPw) throws Exception {
    Members memberInfo = memberRepository.findByMemberId(memberId);

    if (memberInfo == null) {
        throw new Exception("회원정보가 없습니다.");
    }
    //아래와 같이 암호화 비밀번호는 안된다. 평문 비밀번호를 써줘야 한다. 
    boolean isEqualPassword = passwordEncoder.matches(memberPw, memberInfo.getMemberPw());
    if (!isEqualPassword) {
        throw new Exception("비밀번호가 일치하지 않습니다.");
    }
}
```
- 여기서는 id와 password를 받아서 UsernamePasswordAuthenticationToken을 만들어 Spring Security의 인증을 받는다.
- 인증을 받는 방법은 바로 authenticate()다. authenticate()를 호출하면 UsersDetailService의 loadUserByUsername()를 호출하여 회원정보가 있는지 확인한다. 
- UsersDetailService의 경우 구현체가 있다면 해당 구현체를 참조한다. loadUserByUsername()의 parameter는 프론트에서 보내는 body의 key와 같아야 한다. 그래서 memberId로 주었다. 
```java
@Service
@RequiredArgsConstructor
public class PrincipalDetailsService implements UserDetailsService {
	 private final MemberRepository memberRepository

	
	@Override
	public UserDetails loadUserByUsername(String memberId) throws UsernameNotFoundException {
	   Members memberInfo = memberRepository.findByMemberId(memberId);
		if (userDto != null) {
			return new PrincipalDetail(memberInfo);
		}
		return null;
	}
}
```
- PrincipalDetailsService가 return한 class 또한 UserDetails class를 implements하여야 한다. Spring Security의 규칙이다. 
- Spring Security는 authenticationManager가 authenticationProvider에게 권한을 가져오라고 명하고, authenticationProvider는 authenticat()를 찾는다. 
- authenticate()가 호출되면 UserDetailsService를 implements한 class에서 loadUserByUsername를 호출하고 db에서 값을 얻어와 UserDetails를 생성한다. 
```java
@Slf4j
@RequiredArgsConstructor
public class PrincipalDetail implements UserDetails{

	private final Members memberInfo;
	
	//해당 유저의 권한을 return함. 근데 나는 user의 role을 설정안했는데 어떻게 해야할까? 그냥 아래와 같이 authority를 하드코딩해줬다.
	@Override
	public Collection<? extends GrantedAuthority> getAuthorities() {
		Collection<GrantedAuthority>collect=new ArrayList<>();
		collect.add(new GrantedAuthority() {
			
			@Override
			public String getAuthority() {
				log.info("role {}",memberInfo.getUserRole());;
				return memberInfo.getUserRole();
			}
		});
		return collect;
	}

	@Override
	public String getPassword() {
		// TODO Auto-generated method stub
		return memberInfo.getUserPassword();
	}

	@Override
	public String getUsername() {
		// TODO Auto-generated method stub
		return memberInfo.getUserId();
	}
.
..
	

}
```
- 생성된 UserDetails의 비밀번호와 UsernamePasswordAuthenticationToken의 비밀번호를 비교한다. 
- 비밀번호 검증은 DaoAuthenticationProvider가 해주는데, passwordencoder를 사용하기 때문에 db의 암호화된 비밀번호가 아니라 dto로 받은 평문 비밀번호를 token에 넣어줘야 한다. 내부 로직이 그러하다.
- 안 그러면 계속 401 오류가 나게 된다. 비록 이전에 id와 비밀번호를 다 검증했더라고 SpringSecurity에서도 검증을 통과해야 401 오류가 안나기 때문이다. 
- 따라서 id과 비밀번호 검증이 끝났음에도 다시 UsernamePasswordAuthenticationToken를 가져와 비교하는 것이다. 
- 그래서 쓸데없이 db를 더 조회하는 일이 생긴다. 그게 싫다면 DaoAuthenticationProvider를 직접 구현해야 한다.
```java
   UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(memberId,
                memberPw);
```
이를 거쳐서 살아남았다면 아래와 같이 jwt를 return한다. 
```java
JwtResponse jwt = jwtTokenProvider.generateToken(authentication);
    return jwt;
```
- 합치면 아래와 같이 된다.
```java
   //memberInfo에서 가져온 비밀번호가 아니라 dto로 받은 비밀번호를 설정해야 한다. 즉 암호화된 비밀번호를 여기다가 넣으면 안된다.
        UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(memberId,
                memberPw);
        Authentication authentication = authenticationManagerBuilder.getObject().authenticate(token);
        // authenticate 메서드가 실행이 될 때 CustomUserDetailsService에서 만들었던 loadUserByUsername
        // 메서드가 실행됨
        JwtResponse jwt = jwtTokenProvider.generateToken(authentication);
        return jwt;
```
- 아래는 전체 합본이다. 
```java
public class SignService {

    private final MemberRepository memberRepository;
    private final PasswordEncoder passwordEncoder;
    private final JwtTokenProvider jwtTokenProvider;
    private final AuthenticationManagerBuilder authenticationManagerBuilder; 

    public JwtResponse login(String memberId, String memberPw) throws Exception {
        Members memberInfo = memberRepository.findByMemberId(memberId);

        if (memberInfo == null) {
            throw new Exception("회원정보가 없습니다.");
        }

        boolean isEqualPassword = passwordEncoder.matches(memberPw, memberInfo.getMemberPw());
        if (!isEqualPassword) {
            throw new Exception("비밀번호가 일치하지 않습니다.");
        }

        //memberInfo에서 가져온 비밀번호가 아니라 dto로 받은 비밀번호를 설정해야 한다. 즉 암호화된 비밀번호를 여기다가 넣으면 안된다.
        UsernamePasswordAuthenticationToken token = new UsernamePasswordAuthenticationToken(memberId,
                memberPw);
        Authentication authentication = authenticationManagerBuilder.getObject().authenticate(token);
        // authenticate 메서드가 실행이 될 때 CustomUserDetailsService에서 만들었던 loadUserByUsername
        // 메서드가 실행됨
        String accessToken = jwtTokenProvider.generateAccessTokenBySignIn(authentication);
        String refreshToken = jwtTokenProvider.generateRefreshTokenBySignIn(memberId);

        JwtResponse jwt = JwtResponse.builder()
                .grantType("Bearer")
                .accessToken(accessToken)
                .refreshToken(refreshToken)
                .build();

        return jwt;
    }
}
```
## <span style="color:#802548">_15. redis를 활용해 refreshToken 관리_</span>
- 자 이제 그럼 refreshToken은 어디서 관리할까?
- RDB도 가능하지만, Redis에서 관리하면 더 빠르게 access가 가능하다.
- redis를 활용해서 처음 로그인할 때 refreshToken을 redis에 넣는다. 
```java
@PostMapping("/api/signin")
    public JwtResponse login(@RequestBody SignInRequest signInRequestDto, HttpSession session)
            throws Exception {
        JwtResponse jwtResponse = signService.login(signInRequestDto.getMemberId(),
                signInRequestDto.getMemberPassword());

        redisService.setRefreshToken(signInRequestDto.getMemberId(), jwtResponse.getRefreshToken());

        return jwtResponse;
    }
```
- refreshToken은 key를 통해 redis에서 가져올 수 있다. 그럼 가져온 refreshToken으로 새로운 accessToken을 만든다.
```java
@PostMapping("/api/auth/refreshToken")
    public String extendSingIn(@RequestBody String memberId) {
        String refreshToken = redisService.getRefreshToken(memberId);
        String newAccessToken = signService.extendSingInStatus(refreshToken);

        return newAccessToken;
    }
```
- refreshToken은 하루가 지나면 사라지기 때문에 없는 경우에 오류를 뱉어낸다.
```java
public String getRefreshToken(String memberId) {
        String refreshToken = redisRepository.getValues(memberId);
        if (refreshToken == null) {
            throw new SignException(ErrorEnum.NO_REFRESHTOKEN);
        }

        return refreshToken;
    }
```
- 로그아웃을 눌렀을 시에는 해당 memberId를 key로 하는 refreshToken을 redis에서 깔끔하게 지워버린다. 
```java
@PostMapping("/api/logout")
    public void logout(@RequestBody String memberId) {

        redisService.deleteRefreshToken(memberId);
    }
 public void deleteRefreshToken(String memberId) {
        redisRepository.deleteValues(memberId);
    }
```
- 혹시 redis가 꺼져있다면 runtime에러가 나므로 application을 처음 시작할 때 connection 여부를 확인한다.
```java
@Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new StringRedisSerializer());
        template.setConnectionFactory(redisConnectionFactory);

        try {
            RedisConnection connection = redisConnectionFactory.getConnection();
            if (connection != null && connection.ping().equals("PONG")) {
                connection.close();
            }
        } catch (Exception e) {
            throw new RedisConnectionFailureException(ErrorEnum.NOT_CONNECTED_REDIS.getMessage());
        }

        return template;
    }
```

## <span style="color:#802548">_16. RedisConfiguration_</span>
​- 물론 Redis를 쓰기 전에 먼저 configuration을 해야한다.
- 먼저 dependency를 넣어준다.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
- 그리고 아래와 같이 configuration을 구성한다. 비밀번호는 걸어도 그만 안 걸어도 그만이지만 host와 port는 필수다.
```java
redisStandaloneConfiguration.setHostName(redisHost);
redisStandaloneConfiguration.setPort(redisPort);
redisStandaloneConfiguration.setPassword(password);
```

- key와 value가 제대로 인식되게 하기 위해서 직렬화 도구를 넣는다. 
```java
template.setKeySerializer(new StringRedisSerializer());
template.setValueSerializer(new StringRedisSerializer());
```
- 아래가 합본이다.
```java
@Configuration
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String redisHost;

    @Value("${spring.redis.port}")
    private int redisPort;

    @Value("${spring.redis.password}")
    private String password;

    @Bean
    public RedisConnectionFactory redisConnectionFactory() {
        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration();
        redisStandaloneConfiguration.setHostName(redisHost);
        redisStandaloneConfiguration.setPort(redisPort);
        redisStandaloneConfiguration.setPassword(password);
        LettuceConnectionFactory lettuceConnectionFactory = new LettuceConnectionFactory(redisStandaloneConfiguration);
        return lettuceConnectionFactory;
    }

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        template.setKeySerializer(new StringRedisSerializer());
        template.setValueSerializer(new StringRedisSerializer());
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}
```
## Repository
- 모두 RedisTemplate의 opsForValue()라는 method를 통해 구현된다.
```
private final RedisTemplate<String, String> redisTemplate;
```

- key와 value를 넣는 method인 set을 활용한다.
```java
public void setValues(String key, String data) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set(key, data);
    }
```
- key와 value를 넣을 때 duration도 같이 설정한다. 이렇게 하면 duration 이후에는 해당 데이터가 날라간다.
```java
public void setValues(String key, String data, Duration duration) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set(key, data, duration);
}
```
- key를 입력하면 value를 얻어온다.
```java
public String getValues(String key) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        return values.get(key);
}
```
- key를 입력하면 해당 데이터를 지운다.
```java
public void deleteValues(String key) {
        redisTemplate.delete(key);
}
```
- 아래가 합본이다.
```java
@Repository
@RequiredArgsConstructor
public class RedisRepository {

    private final RedisTemplate<String, String> redisTemplate;

    public void setValues(String key, String data) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set(key, data);
    }

    public void setValues(String key, String data, Duration duration) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        values.set(key, data, duration);
    }

    public String getValues(String key) {
        ValueOperations<String, String> values = redisTemplate.opsForValue();
        return values.get(key);
    }

    public void deleteValues(String key) {
        redisTemplate.delete(key);
    }

}
```
## Service
- 내가 redis를 쓴 이유는 refreshToken을 가볍고 손쉽게 구현하기 위해서였다.
- refreshToken을 duration과 함께 넣으면 duration이 지나면 자동으로 사라진다. 
```java
 public void setRefreshToken(String memberId, String refreshToken) {
        Duration duration = Duration.between(LocalDateTime.now(), LocalDateTime.now().plusDays(1));
        redisRepository.setValues(memberId, refreshToken, duration);
    }
```
- 로그아웃을 하는 경우에는 직접 지워준다.
```java
public void deleteRefreshToken(String memberId) {
        redisRepository.deleteValues(memberId);
}
```
- accessToken이 만료되었을 때 refreshToken이 있는 지 확인하고, 있다면 다시 accessToken을 발급하고, 없으면 다시 로그인하라는 오류를 낸다. 
```java
 public String getRefreshToken(String memberId) {
        String refreshToken = redisRepository.getValues(memberId);
        if (refreshToken == null) {
            throw new SignException(ErrorEnum.NO_REFRESHTOKEN);
        }

        return refreshToken;
    }
```
- 아래는 합본이다.
```java
@Service
@RequiredArgsConstructor
public class RedisService {

    private final RedisRepository redisRepository;

    public void setRefreshToken(String memberId, String refreshToken) {
        Duration duration = Duration.between(LocalDateTime.now(), LocalDateTime.now().plusDays(1));
        redisRepository.setValues(memberId, refreshToken, duration);
    }

    public void deleteRefreshToken(String memberId) {
        redisRepository.deleteValues(memberId);
    }

    public String getRefreshToken(String memberId) {
        String refreshToken = redisRepository.getValues(memberId);
        if (refreshToken == null) {
            throw new SignException(ErrorEnum.NO_REFRESHTOKEN);
        }

        return refreshToken;
    }

}
```