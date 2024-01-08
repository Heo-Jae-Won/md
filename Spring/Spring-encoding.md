## <span style="color:#802548">_1. Java로 해결_</span>
​

- 참고로 이것은 특수한 경우다. 
- InputStreamReader를 활용하게 되면 아래와 같이 UTF-8 설정을 해줘야 한다. 
- 안해주면 default charset이 적용되는데 윈도우에선 MS949라서 대다수의 UTF-8과 충돌이 일어나게 된다. InputStreamReader자체에서 문제가 생기는 것이라 아래 해결법들은 소용이 없다.
```java
rd = new BufferedReader(new InputStreamReader(conn.getInputStream(),"UTF-8"));
```
- 다른 경우 아래처럼 String 자체를 UTF-8로 만들 수 있는데 추천하지 않는다.  매 번 저런 식으로 만들면 관리하기가 어렵다. 따라서 2번의 해결법을 참고하자.
```java
String error="에러";
error.getBytes(Charset.forName("UTF-8"))
```
## <span style="color:#802548">_2. Spring으로 해결_</span>
- 때로는 Spring의 기본 HttpMessageConverter가 ISO-8859-1이기 때문에 한글이 깨지기도 한다. 
- 그럴 때는 아래와 같이 MessageConverter에 charset을 추가해주자. 
```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
  StringHttpMessageConverter stringConverter = new StringHttpMessageConverter();
    stringConverter.setSupportedMediaTypes(Arrays.asList(new MediaType("text", "plain", Charset.forName("UTF-8"))));
    converters.add(stringConverter);
}
```
- 때로 위의 설정만으로 부족할 경우도 있는데, Exception을 던질 때 그러한 상황이 일어난다. 
- ExceptionHandler를 통해 Dto class에 담아 Exception code나 message를 front에 던지게 되면 @ResponseBody를 거치지 않기 때문에 Dto object가 json으로 변환되지 못한다. 
- no converter found 오류가 일어나게 된다는 것이다. 그럴 땐 MappingJackson2HttpMessageConverter도 추가해야 한다. 
```java
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
  StringHttpMessageConverter stringConverter = new StringHttpMessageConverter();
    stringConverter.setSupportedMediaTypes(Arrays.asList(new MediaType("application", "json", Charset.forName("UTF-8"))));
    converters.add(stringConverter);
    converters.add(new MappingJackson2HttpMessageConverter());
} 
```
## <span style="color:#802548">_(3) maven으로 해결_</span>
​

- 그게 아니면 build 자체에서 utf-8을 주어야 하는 경우도 있다. 
- jenkins에서 build할 때 기본 default 인코딩이 주어져있지 않은 경우 ANSI로 설정하게 되는데 messageconverter를 거치지 않는 Exception의 경우, default charset이 맞지 않아 한글이 깨지는 경우가 생길 수 있다. 
- legacy에서는 아래 설정이 필요했는데 boot에서는 아래 설정없이도 자동으로 utf-8 build가 이뤄지는 것으로 보인다. 이유는 잘 모르겠다. 
```xml
<project.build.sourceEncoding>utf-8</project.build.sourceEncoding>
```
​

## <span style="color:#802548">_4. Tomcat으로 해결_</span>
​

- 다른 사람들은 효과를 봤다길래 일단 써놓는데, 나는 효과를 못봤다. 일단 저렇게 써놓으면 logback이 아닌 catalina.out에서 vm arguments의 옵션들을 살펴볼 수 있다. 
```xml
JAVA_OPTS="$JAVA_OPTS -Dfile.encoding=UTF-8 -Dfile.client.encoding=UTF-8 -Dclient.encoding.o
```
