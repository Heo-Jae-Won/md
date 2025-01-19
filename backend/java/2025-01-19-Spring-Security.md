## <span style="color:#802548">_Spring Security 구축시, Controller는 login의 역할이 거의 없다_</span>
- Security가 login을 관리하게 되면, login의 경우에는 Controller에서 return이 쓸모가 없게 된다.
- Spring Security가 돌아갈 페이지를 지정하기 때문이다.
- 따라서 아래와 같이 return문을 호출할 필요가 없다.

```java
@PostMapping("/signin")
public String signin(@ModelAttribute LoginDTO signIndDTO, @RequestParam(name = "name", required = false) String name) {

    LoginUserDetailDTO loginUserDetailDTO = (LoginUserDetailDTO) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
    String password = loginUserDetailDTO.getPassword();
    System.out.println("password: " + password);

    userService.signin(signIndDTO);

    return "redirect:/"
}
```

- 또한 signin처럼 비밀번호, 아이디 비교 로직또한 필요없다.
- 그러한 로직이 Security에 담겨있기 때문이다.
- 아래는 아예 필요없는 로직이다.

```java
public void signin(SignIndDTO signIndDTO) {
    Optional<UserEntity> temp = userRepository.findById(signIndDTO.getId());
    UserEntity userEntity = temp.orElseThrow(()->new RuntimeException("없음"));

    if (userEntity == null || !signIndDTO.getPass().equals(userEntity.getUserPW())) {
        throw new UsernameNotFoundException("아이디가 혹은 비밀번호가 다릅니다.");
    } 
}
```

- 해당 부분을 제대로 이해하지 못하면 redundant한 network call이 일어나게 된다.
- 그럼 어떻게 바꿔야 할까? 아래처럼 바꾸면 된다.
- ajax로 보내든, form submit으로 보내든 from login의 경우 어차피 return을 하지 않는다. Spring Security에서 담당하기 때문이다.
- 따라서 MVC pattern으로서 껍데기만 존재하면 된다.

```java
@PostMapping("/signin")
public void signin() {

}
```


## <span style="color:#802548">_Spring Security flowㅡㅡ> UsernamePasswordAuthenticationFilter_</span>

- Spring Security는 Controller에서 login 인증이 이뤄지는 게 아니다.
- 아래처럼 별도의 공간에서 process가 흘러간다.

```
- Login Form Submission from front
- UsernamePasswordAuthenticationFilter 
- AuthenticationManager interface - AuthenticationProviders 
- DaoAuthenticationProvider
- UserDetailsService - loadUserByUsername
```

- UserNamePasswordAuthenticationFilter에서 AuthenticationManager interface에 authenticate 책임을 위임한다.
    - return 문의 return this.getAuthenticationManager().authenticate(authRequest)가 해당 코드다.

```java
//UsernamePasswordAuthenticationFilter class
@Override
public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
        throws AuthenticationException {
    if (this.postOnly && !request.getMethod().equals("POST")) {
        throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
    }
    String username = obtainUsername(request);
    username = (username != null) ? username.trim() : "";
    String password = obtainPassword(request);
    password = (password != null) ? password : "";
    UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,
            password);
    // Allow subclasses to set the "details" property
    setDetails(request, authRequest);
    return this.getAuthenticationManager().authenticate(authRequest);
}
```

- 이 때 Filter는 front에서 받은 값을 token 형태로 저장해둔다. 
    - UsernamePasswordAuthenticationToken.unauthenticated()가 그 과정이다.
    - UsernamePasswordAuthenticationToken 값은 향후 UserDetailsDTO의 값과 비교하는 작업이 이뤄진다. 

```java
@Override
	public Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException {
		if (this.postOnly && !request.getMethod().equals("POST")) {
			throw new AuthenticationServiceException("Authentication method not supported: " + request.getMethod());
		}
		String username = obtainUsername(request);
		username = (username != null) ? username.trim() : "";
		String password = obtainPassword(request);
		password = (password != null) ? password : "";
		UsernamePasswordAuthenticationToken authRequest = UsernamePasswordAuthenticationToken.unauthenticated(username,
				password);
		// Allow subclasses to set the "details" property
		setDetails(request, authRequest);
		return this.getAuthenticationManager().authenticate(authRequest);
	}
```

- principal은 id가, credentials는 비밀번호가 들어간다.

```java
public static UsernamePasswordAuthenticationToken unauthenticated(Object principal, Object credentials) {
    return new UsernamePasswordAuthenticationToken(principal, credentials);
}
```

## <span style="color:#802548">_Spring Security flowㅡㅡ> AuthenticationManager_</span>

- AuthenticationManager는 UserNamePasswordAuthenticationFilter class에서 setter를 찾아볼 수 없다.
- AuthenticationManager class는 SpringSecurity config에서 넣는다. Spring boot에서 autoconfigure 해주는 것이다.
    - auto configure가 어떻게 이뤄지는 지에 관한 logic은 공개되어 있지 않은 것으로 보인다.
- 참고로 UserDetailsSerivce 구현체가 있어야만 AuthenticationManager 구현체 중 DaoAuthenticationProvider로 등록된다.
- 그외 custom은 아래처럼 custom을 만들어 만들어 등록해야 한다.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfiguration {
    
    @Bean
    public AuthenticationProvider customAuthenticationProvider() {
        return new DaoAuthenticationProvider() {{
            setUserDetailsService(customUserDetailsService());
            setPasswordEncoder(passwordEncoder());
        }};
    }

}
```

## <span style="color:#802548">_Spring Security flowㅡㅡ> DaoAuthenticationProvider_</span>

- 그럼 UsernamePasswordAuthenticationFilter class가 getAuthenticationManager().authenticate()를 호출하면 DaoAuthenticationProvider의 authenticate()가 호출되게 된다.
    - 해당 logic은 DaoAuthenticationProvider class 자체가 갖고 있지 않다.
    - 상위 abstract class인 AbstractUserDetailsAuthenticationProvider class가 가지고 있다.
    - 중요한 것은 중간의 retrieveUser()다.

```java
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,
            () -> this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.onlySupports",
                    "Only UsernamePasswordAuthenticationToken is supported"));
    String username = determineUsername(authentication);
    boolean cacheWasUsed = true;
    UserDetails user = this.userCache.getUserFromCache(username);
    if (user == null) {
        cacheWasUsed = false;
        try {
            user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
        }
        catch (UsernameNotFoundException ex) {
            this.logger.debug("Failed to find user '" + username + "'");
            if (!this.hideUserNotFoundExceptions) {
                throw ex;
            }
            throw new BadCredentialsException(this.messages
                .getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
        }
        Assert.notNull(user, "retrieveUser returned null - a violation of the interface contract");
    }
    try {
        this.preAuthenticationChecks.check(user);
        additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
    }
    catch (AuthenticationException ex) {
        if (!cacheWasUsed) {
            throw ex;
        }
        // There was a problem, so try again after checking
        // we're using latest data (i.e. not from the cache)
        cacheWasUsed = false;
        user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
        this.preAuthenticationChecks.check(user);
        additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
    }
    this.postAuthenticationChecks.check(user);
    if (!cacheWasUsed) {
        this.userCache.putUserInCache(user);
    }
    Object principalToReturn = user;
    if (this.forcePrincipalAsString) {
        principalToReturn = user.getUsername();
    }
    return createSuccessAuthentication(principalToReturn, authentication, user);
}
```

- retrieveUser는 abstract method이므로 구현체가 구현한다.
- 다시말해 DaoAuthenticationProvider class에서 구현한다.
- 여기서 중요한 것은 loadUserByUsername()를 호출한다는 사실이다.

```java
@Override
protected final UserDetails retrieveUser(String username, UsernamePasswordAuthenticationToken authentication)
        throws AuthenticationException {
    prepareTimingAttackProtection();
    try {
        UserDetails loadedUser = this.getUserDetailsService().loadUserByUsername(username);
        if (loadedUser == null) {
            throw new InternalAuthenticationServiceException(
                    "UserDetailsService returned null, which is an interface contract violation");
        }
        return loadedUser;
    }
    catch (UsernameNotFoundException ex) {
        mitigateAgainstTimingAttack(authentication);
        throw ex;
    }
    catch (InternalAuthenticationServiceException ex) {
        throw ex;
    }
    catch (Exception ex) {
        throw new InternalAuthenticationServiceException(ex.getMessage(), ex);
    }
}
```

## <span style="color:#802548">_Spring Security flowㅡㅡ> UserDetailsService_</span>

- loadUserByUsername()는 우리가 구현한 class의 method를 호출하게 된다.
- UserDetailsService를 구현했다면 loadUserByUsername()를 통해 db에서 가져온 id와 pass를 Spring session에 id와 pass 값을 넣는다.
- db조회 시 id가 아예 없다면 UsernameNotFoundException을 떨어뜨리면 된다.

```java
@Service
@RequiredArgsConstructor
public class LoginUserDetailsService implements UserDetailsService {

    private final UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        Optional<UserEntity> temp = userRepository.findById(username);
        //supplier면 그냥 null이 아니라, () -> null로 줘야 됨.
        UserEntity userEntity = temp.orElseGet(() -> null);
        if (userEntity == null) {
            //여기서 throw exception을 해도 front에서 200으로 code가 전달됨.--------> 302로 redirect 되는 것임. form 인증의 경우 기본 로직.
            throw new UsernameNotFoundException("아이디가 없습니다.");
        } 

        // LoginUserDetailDTO loginUserDetailDTO = LoginUserDetailDTO.TODTO(userEntity);
        // return LoginUserDetailDTO.TODTO(userEntity);

        //GC가 최대한 덜 일어나게 아래처럼 사용. 이게 더 직관적인 것이기도 함.
        return LoginUserDetailDTO.TODTO(userEntity);
    }
}
```

## <span style="color:#802548">_Spring Security flowㅡㅡ> DaoAuthenticationProvider_</span>
- 아까 말한 것처럼 DaoAuthenticationProvider class는 abstract class인 AbstractUserDetailsAuthenticationProvider의 authenticate()를 호출했다.
- 그럼 이제 return된 것이 AbstractUserDetailsAuthenticationProvider class의 authenticate()에서 check를 거치게 된다.
    - this.preAuthenticationChecks.check(user)
    - additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication)
    - this.postAuthenticationChecks.check(user);  부분이다.

```java
@Override
public Authentication authenticate(Authentication authentication) throws AuthenticationException {
    Assert.isInstanceOf(UsernamePasswordAuthenticationToken.class, authentication,
            () -> this.messages.getMessage("AbstractUserDetailsAuthenticationProvider.onlySupports",
                    "Only UsernamePasswordAuthenticationToken is supported"));
    String username = determineUsername(authentication);
    boolean cacheWasUsed = true;
    UserDetails user = this.userCache.getUserFromCache(username);
    if (user == null) {
        cacheWasUsed = false;
        try {
            user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
        }
        catch (UsernameNotFoundException ex) {
            this.logger.debug("Failed to find user '" + username + "'");
            if (!this.hideUserNotFoundExceptions) {
                throw ex;
            }
            throw new BadCredentialsException(this.messages
                .getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
        }
        Assert.notNull(user, "retrieveUser returned null - a violation of the interface contract");
    }
    try {
        this.preAuthenticationChecks.check(user);
        additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
    }
    catch (AuthenticationException ex) {
        if (!cacheWasUsed) {
            throw ex;
        }
        // There was a problem, so try again after checking
        // we're using latest data (i.e. not from the cache)
        cacheWasUsed = false;
        user = retrieveUser(username, (UsernamePasswordAuthenticationToken) authentication);
        this.preAuthenticationChecks.check(user);
        additionalAuthenticationChecks(user, (UsernamePasswordAuthenticationToken) authentication);
    }
    this.postAuthenticationChecks.check(user);
    if (!cacheWasUsed) {
        this.userCache.putUserInCache(user);
    }
    Object principalToReturn = user;
    if (this.forcePrincipalAsString) {
        principalToReturn = user.getUsername();
    }
    return createSuccessAuthentication(principalToReturn, authentication, user);
}
```

- preAuthenticationChecks와 postAuthenticationChecks logic은 아래와 같다.
- lock, enable, expire, sessionexpire 여부를 확인한다.

```java
private class DefaultPreAuthenticationChecks implements UserDetailsChecker {

    @Override
    public void check(UserDetails user) {
        if (!user.isAccountNonLocked()) {
            AbstractUserDetailsAuthenticationProvider.this.logger
                .debug("Failed to authenticate since user account is locked");
            throw new LockedException(AbstractUserDetailsAuthenticationProvider.this.messages
                .getMessage("AbstractUserDetailsAuthenticationProvider.locked", "User account is locked"));
        }
        if (!user.isEnabled()) {
            AbstractUserDetailsAuthenticationProvider.this.logger
                .debug("Failed to authenticate since user account is disabled");
            throw new DisabledException(AbstractUserDetailsAuthenticationProvider.this.messages
                .getMessage("AbstractUserDetailsAuthenticationProvider.disabled", "User is disabled"));
        }
        if (!user.isAccountNonExpired()) {
            AbstractUserDetailsAuthenticationProvider.this.logger
                .debug("Failed to authenticate since user account has expired");
            throw new AccountExpiredException(AbstractUserDetailsAuthenticationProvider.this.messages
                .getMessage("AbstractUserDetailsAuthenticationProvider.expired", "User account has expired"));
        }
    }

}

private class DefaultPostAuthenticationChecks implements UserDetailsChecker {

    @Override
    public void check(UserDetails user) {
        if (!user.isCredentialsNonExpired()) {
            AbstractUserDetailsAuthenticationProvider.this.logger
                .debug("Failed to authenticate since user account credentials have expired");
            throw new CredentialsExpiredException(AbstractUserDetailsAuthenticationProvider.this.messages
                .getMessage("AbstractUserDetailsAuthenticationProvider.credentialsExpired",
                        "User credentials have expired"));
        }
    }

}
```

- additionalAuthenticationChecks() logic은 아래와 같다.
    - front에서 가져온 id, password를 저장한 UsernamePasswordAuthenticationToken과 db값을 저장한 userDetails을 비교한다.
    - 여기선 id가 있는지만이 아니라, password 비교도 이뤄진다.
    - 따라서 Controller에서의 service를 통한 비밀번호 비교 등이 의미가 없다는 것이다.

```java
@Override
@SuppressWarnings("deprecation")
protected void additionalAuthenticationChecks(UserDetails userDetails,
        UsernamePasswordAuthenticationToken authentication) throws AuthenticationException {
    if (authentication.getCredentials() == null) {
        this.logger.debug("Failed to authenticate since no credentials provided");
        throw new BadCredentialsException(this.messages
            .getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
    }
    String presentedPassword = authentication.getCredentials().toString();
    if (!this.passwordEncoder.matches(presentedPassword, userDetails.getPassword())) {
        this.logger.debug("Failed to authenticate since password does not match stored value");
        throw new BadCredentialsException(this.messages
            .getMessage("AbstractUserDetailsAuthenticationProvider.badCredentials", "Bad credentials"));
    }
}
```
