## <span style="color:#802548">_1. Window 설치 및 설정 파일_</span>
- 아래 사이트에서 redis를 다운받으면 된다. window에서는 redis를 더이상 유지보수해주지 않는다. 
https://github.com/microsoftarchive/redis/releases


- redis-server.exe가 아니라 redis-cli.exe를 실행해야 redis-server가 시작된다. 이유는 모르겠다. 
- 설정파일은 redis.windows-service.conf이며 여기서 수정해주면된다. 
- 다만 user의 권한이 없다고 뜨는 경우, redis.windows-service.conf 파일 우클릭 -속성- 보안- Users에게 모든 권한 부여를 통해 파일을 수정할 수있다. 
- 편집툴은 VsCode를 사용했다.


​

- 이로써 local에서도 test가 가능하지만, 사실 핵심은 linux다. 
- redis도 unix 계열만 지원된다. redis를 window에서도 쓸 수있게 MS가 개조를 한 것 뿐이다.

## <span style="color:#802548">_2. Linux 설치 및 설정 파일_</span>
- 리눅스에서는 아래 명령어로 redis를 설치한다.
```sh
apt-get update
apt-get install redis
```
- apt로 깔았다면 /etc/redis/redis.conf에서 설정을 수정할 수 있다. 
- 같은 서버 내에서 redis를 돌리기 때문에 자신인 localhost로 ip를 설정한다.
- 아래와 같이 비밀번호를 요구하게 정할 수 있다.
- 근데 나는 비밀번호를 요구하도록 변경했는데도 설정이 적용되지 않아 직접 command로 비밀번호를 받게끔 하였다.
```sh
CONFIG SET requirepass 비밀번호
```
- 그럼 아래와 같이 redis-cli로 redis 조작을 시작할 때 비밀번호를 요구하게 된다.
- redis-cli를 나가는 법은 exit를 치면 되고, redis를 끄고 싶다면 shutdown을 치면 된다. 
- 시작하고 싶다면 redis-server를 치면 된다. 
- 참고로 뒤에 &를 안 붙이면 process를 background로 못 돌리니까 꼭 &를 붙여주자.
- 값을 소스코드가 아닌 직접 조작하려면 아래와 같은 방식으로 쓰면 된다. redis-cli에 접속한 뒤 아래 명령어를 활용한다.
```sh
set [key] [value]

get [key]

del [key]
```
## <span style="color:#802548">_3. Configuration_</span>
- 먼저 dependency를 넣어준다.
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
- 그리고 아래와 같이 configuration을 구성한다. 
- 비밀번호는 걸어도 그만 안 걸어도 그만이지만 host와 port는 필수다.
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
## <span style="color:#802548">_4. Repository_</span>
- 모두 RedisTemplate의 opsForValue()라는 method를 통해 구현된다.
```java
private final RedisTemplate<String, String> redisTemplate;
​```

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
## <span style="color:#802548">_5. Service_</span>
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