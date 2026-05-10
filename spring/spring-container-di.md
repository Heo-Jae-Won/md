## <span style="color:#802548">_1.Spring의 container_</span>

### <span style="color:#FFCCFF">Container의 개념</span>
- Spring은 bean을 다루는 것이 핵심이다. 
- 하지만 bean을 보기 이전에 bean을 둘러싼 container에 대해 먼저 이해할 필요가 있다.

- bean은 conatiner에서 관리되기 때문이다.
  - container라는 말만 들으면 뭔가 거창한 느낌이 들지만, 사실 별다른 게 아니다. 
    - 개념으로 말하면 말그대로 그냥 bean이 거주하는 장소다. 
- Spring 식으로 이해해보자. 
  - 우선 Spring은 Java 기반이다. 그렇다면 container라는 것도 결국 class, interface 같은 것들로 구현된 것이라는 의미다.
    - 그럼 구체적으로 어떤 class나 interface가 사용된 것일까? 
    - 바로 ApplicationContext interface와 BeanFactory interface다. 

### <span style="color:#FFCCFF">Web환경에서 쓰이는 Container</span>
- 그 중에서 실제로 쓰이는 것은 대부분 ApplicationContext다. 
- 왜 ApplicationContext만 사용될까?
- BeanFactory interface는 DI 기능만 제공한다.
- ApplicationContext는 transaction 처리, MVC, annotation 설정 등 다양한 편의 기능을 제공해준다.
  - 그런데 왜 ApplicationContext는 class가 아니라 interface인가? ApplicationContext를 다양한 환경에서 구성하기 위해서다.
    - 다양한 환경이라 함은? web환경과 web이 아닌 환경을 모두 아우른다는 의미다. 

- web 환경 내에서는 xml과 annotation 환경이다. 
  - 한국에서는 대부분 Spring이 web 환경에서 쓰인다.
  - 사실상 Applica tionContext의 종류에서도 주로 아래를 많이 사용하게 된다.
    - XmlWebApplicationContext class 
    - AnnotationConfigWebApplicationContext class

### <span style="color:#FFCCFF">실제 XML을 통한 Container</span>

- 하지만 Spring legacy나 boot를 이용하는 사람 대부분은 저 코드를 볼 일이 많지 않을 것이다.
- 보통 다른 우회로를 사용한다.
  - legacy에서는 아래와 같다.
    - web.xml에서 생성
    - implements webInitializer에서 생성.
  - boot는  @SpringBootApplication을 달아준다.
    - 그럼 application.properties 파일이나 yaml 파일로 설정을 읽어 auto-configuration을 진행한다.
- 따라서 위의 class들은 사실 볼일이 많지 않다. 

​
- boot는 legacy를 간소화한 것이기 때문에 legacy를 이해하면 boot는 금방 이해할 수 있다.
- 이제부터는 legacy 기준으로 살펴보겠다. 

- Spring container의 구성요소로 아래와 같은 방법이 있다.
  - xml. 옛날 방식
  - annotation. 요즘 방식
- 여기선 xml이 더 자주 나올 것이다.
- 아래의 xml은 3개의 xml을 context로 선언했다.
  - root-context.xml
  -  servlet-context.xml
  -  security-context.xml
```xml
<context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>
        /WEB-INF/spring/root-context.xml 
        /WEB-INF/spring/security-context.xml
        </param-value>
</context-param>
```
```xml
<servlet>
        <servlet-name>appServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet
        </servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>/WEB-INF/spring/appServlet/servlet-context.xml
            </param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
</servlet>
```
- 그 중 중요한 것은 root-context와 servlet-context다.
## <span style="color:#802548">_2.RootContext_</span>

### <span style="color:#FFCCFF">공유자원과 미공유자원</span>
- 그런데 왜 설정이 두 개 나눠져 있을까? 
- 그냥 한 곳에 모아서 써도 되지 않을까? 나눠놓은 데는 이유가 있다.
- request의 경우 기본적으로 스레드마다 따로 생성되기 때문에 겹쳐서 병목이나 충돌이 일어나지 않는다.

- 그런데 DB커넥션의 경우, 공통으로 쓰이는 과정이다. 
  - 공통으로 쓰이는 것을 스레드마다 따로 생성하는 것은 컴퓨터 자원의 낭비다.
  - 때문에 request 처리 가능 상태가 되기 전에(dispatchServlet load 전에) 미리 contextloaderlistener를 load한다.
  - Security도 마찬가지의 이유로 contextloaderListener에 넣는다.
  - 이 contextloaderListener가 바로 rootApplicationContext가 된다. 
- db와 관련된 것들인 @Service, @Repository가 붙은 class를 bean으로 등록시키는 게 대다수다. 
- rootContext에서 관장하는 bean은 모두 메모리에 load시켜 모든 context에서 공유하는 것이다.
- 따라서 rootContext 혹은 ParentContext라고 불린다.

​

- 아래와 같이 db와 관련된 설정을 만들 수 있다.
- xml 해석설정을 가져오는 site 관련 구성은 아래와 같이 구성된다.

- 참고로 mybatis 기준이라서 실제 기본만 주어지는 rootContext는 이런 식으로 구성되지 않는다.
```xml
?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-3.1.xsd
		http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring-1.2.xsd
		http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.1.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.1.xsd">
```
### <span style="color:#FFCCFF">실제 db관련 xml</span>
- 그럼 실제 RootContext를 설정한 내용이다.
- mybatis 설정에는 sqlSession과 dataSource, sqlSessionFactory가 필요하다.
- 물론 transaction도 사실상 필수라서 TransactionManager까지 달아줘야 한다. 

- DB를 연동하는 방식으로는 다음과 같다.
  - JDBC 방식은 중복 코드가 많아서 Spring에선 거의 직접 사용되지는 않는다.
  - DataSource 중에서도 많은 경우 hikariconfig같은 connection pool을 사용하며, 가끔 JNDI도 사용된다.
  - DriverManager 방식은 test용이 아니면 추천되지 않는다. connection pool이 아니라 성능에 하자가 많다.

- 아래는 connection pool을 이용한 방식이다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:mybatis-spring="http://mybatis.org/schema/mybatis-spring"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:jdbc="http://www.springframework.org/schema/jdbc"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/jdbc http://www.springframework.org/schema/jdbc/spring-jdbc-3.1.xsd
		http://mybatis.org/schema/mybatis-spring http://mybatis.org/schema/mybatis-spring-1.2.xsd
		http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.1.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-3.1.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.1.xsd">

	<!-- Root Context: defines shared resources visible to all other web components -->

	<!-- DB연결 설정 -->
	<bean id="hikariConfig" class="com.zaxxer.hikari.HikariConfig">
		<property name="driverClassName"
			value="org.mariadb.jdbc.Driver"></property>
		<property name="jdbcUrl"
			value="jdbc:mariadb://21ㅌㅌㅌ4:414210/ME2"></property>
		<property name="username" value="h23" />
		<property name="password" value="pass" />
		<property name="dataSourceProperties">
			<props>
				<prop key="cacheprepStmts">true</prop>
				<prop key="prepStmtCacheSize">250</prop>
				<prop key="prepStmtCacheSqlLimit">2048</prop>
			</props>
		</property>
	</bean>

	<bean id="dataSource"        class="com.zaxxer.hikari.HikariDataSource"
		destroy-method="close">
		<constructor-arg ref="hikariConfig" />
	</bean>

	<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="configLocation"  value="classpath:/mybatis-config.xml"/>
		<property name="mapperLocations" value="classpath:/mapper/**/*.xml" />
	</bean>
	
	<bean id="sqlSession"        class="org.mybatis.spring.SqlSessionTemplate"
		destroy-method="clearCache">
		<constructor-arg name="sqlSessionFactory"
			ref="sqlSessionFactory" />
	</bean>
	
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource" />
	</bean>
	
	<tx:annotation-driven />
	
	<context:component-scan
		base-package="com.example.dao" />
	<context:component-scan
		base-package="com.example.service" />
</beans>
```
 - JNDI로 설정하게 되면 DataSource 설정이 아래와 같이 바뀌게 된다. 

 - class를 다른 것을 쓴다. value는 본인이 맞게 구성해야 한다.
```xml
<bean id="DatabaseName" class="org.springframework.jndi.JndiObjectFactoryBean">
    <property name="jndiName" value="java:comp/env/jdbc/DatabaseName"/>
</bean>
```
- Tomcat의 context.xml과 Server.xml에서 바꾸고 싶은 값을 직접 설정하게 된다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context>
    <Resource name="jndi/mysql" 
		  auth="Container" 
		  type="javax.sql.DataSource"
		  driverClassName="com.mysql.cj.jdbc.Driver" 
		  url="jdbc:mysql://localhost:3306/jndi?serverTimezone=UTC"
		  username="root" 
		  password="root"
		  maxTotal="100" 
		  maxIdle="30"
		  maxWaitMillis="10000"/>
          
	...

</Context>
```

### <span style="color:#FFCCFF">실제 security관련 xml</span>
- 아래는 Security 내용이다.

- 간단하게 bcryptpasswordEncoder만 사용했다. 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/security"
            xmlns:beans="http://www.springframework.org/schema/beans"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://www.springframework.org/schema/beans
            http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/security
            http://www.springframework.org/schema/security/spring-security.xsd">
            <beans:bean id="bcryptPasswordEncoder" class="org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder"/>
</beans:beans>
```


## <span style="color:#802548">_3. ServletContext_</span>
- RootContext 설정이 끝나면 web과 관련된 servlet 설정을 읽어들인다. 
- 이는 모두가 공유하지 않으므로 따로 appServlet이라는 이름으로 빼낸 것이다. 
- 보통은 web과 관련된 설정은 1개지만, 복수개를 만든다면 해당 web 설정에 맞는 xml을 읽어들여 적용되게 되는 것이다.

- 만약 만들고 싶다면 아래와 같이 appServlet2라는 name으로 만들 수도 있을 것이다. 
- 물론 그런 경우는 거의 없다.
```xml
<servlet>
    <servlet-name>appServlet2</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/spring/appServlet/servlet-context.xml
        </param-value>
    </init-param>
    <load-on-startup>2</load-on-startup>
</servlet>
```
### <span style="color:#FFCCFF">dispatcherServlet</span>
- serlvet에는 3가지가 있다. 
  - dispatcherServlet. static file과 jsp파일을 제외한 모든 요청을 담당한다.  
  - JspServlet. jsp 파일을 담당한다.
  - DefaultServlet. static file을 담당한다.
- 중요한 것은 dispatcherServlet다. web과 관련된 thread별 별개 설정을 담당하는 applicationContext 구성요소다.
- Spring의 경우에는 internalViewResolver class를 활용하여 Spring 내에서 view를 return하게 된다. 

```xml
<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <beans:property name="prefix" value="/WEB-INF/views/" />
    <beans:property name="suffix" value=".jsp" />
</beans:bean>
```

### <span style="color:#FFCCFF">dispatcherServlet 심화탐구</span>
- 그럼 DispatcherServlet에 대해 더 자세하게 알아보자.
- DispactherServlet은 @Controller가 달린 class만 처리할 수 있는 게 아니다. 
  - 더 다양한 웹 요청을 처리할 수 있다. 
    - 이처럼 다양한 웹 요청을 처리한다는 의미에서, servlet이 아닌 handler라고도 한다. 
- 어떻게 DispactherServlet은 다양한 웹 요청을 처리할 수 있을까?
  - 그것은 DispatcherServlet이 ModelAndView만 return하면 되기 때문이다. 
    - 그런데 jsp를 return할 때 String으로 return하는 경우를 본 적이 있을 것이다.
    - 이 경우에는 String return이라고 생각할 수 있지만, 그렇지 않다.
    - 중간에 HandlerApapter class가 String을 ModelAndView로 전환하여 return한다. 

​
### <span style="color:#FFCCFF">handlerAdapter, handlerMapping</span>
- 물론 HandlerAdapter 이전에 HandlerMapping이 선행된다. 
- 그를 위해선 dispatcherServlet을 관장하는 xml 혹은 class에 특정한 것이 필요하다.
```
<annotation-driven> - xml
@EnableWebMVC - class 
```
- 그러면 handlerAdapter와 handlerMapping이 쌍으로 만들어진다.
  - RequestMappingHandlerMapping과 RequestMappingHandlerAdapter / request
  - SimpleUrlHanlderMapping과 HttpRequestHandlerAdapter / static file
- 그 말인 즉슨 annotation-driven만 설정한다면 된다는 것이다.
- 우리가 굳이 HandlerMapping과 HandlerAdapter를 일일이 구현할 필요가 없다. Spring이 해준다.

​
- 그래도 handleradapter와 mapping의 기본원리는 살펴보도록 하자. web.xml과 관련이 있다.
- HandlerMapping에는 순서가 있다. 
  - RequestMappingHandlerMapping, 즉 @Controller가 먼저다.
  - SimpleUrlHanlderMapping, 즉 정적파일에 대한 것은 나중이다.
    - 정적인 파일이 들어온다면 Request MappingHandlerMapping에 맞는 쌍이 있나 먼저 살펴보고,
    - 없으면 SimpleUrlHandlerMapping이 살펴보고 맞으면 mapping이 이뤄진다는 의미다. 
    - 만약 두 개 다 없으면 최종적으로 defaultServletHt RequestHandler가 살펴보게되며
    - 이마저도 없으면 404를 return한다.
  - 만약 있다면 해당 adapter가 들어온 웹 요청을 처리하게 된다.

​

- web.xml에서 dispatcher경로를  /*로 설정하면 jspServlet(/.jsp), defaultServlet(/.css) 등도 모두 DispatcherServlet이 담당하게 되어 mapping controller가 없어 오류가 난다.
- 따라서 /로 주는 경우가 대부분이다. 
- 때로는 기업의 약칭을 붙이기도 한다. 
  - 아래보면 /*.nhn이 dispatcherServlet 경로로 설정되어 있음을 알 수 있다.
    - 그럼 @RequesteMapping에서 /compantyEthics라고 붙여도 자동으로 /compantyEthics.nhn으로 url이 형성된다. 

```xml
<servlet>
    <servlet-name>appServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/Servlet-context.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

    <!-- 2. 매핑 -->
<servlet-mapping>
    <servlet-name>appServlet</servlet-name>
    <url-pattern>/*.nhn</url-pattern>
</servlet-mapping>
```
```
https://www.nhn.com/ko/company/companyEthics.nhn
```
 
- 파일의 경우에는 web에서 받을 때 별도의 resolver가 필요하다.
- 해당 class 이름이 multipartResolver다.

- web.xml에 추가한다면 아래와 같다.
```xml
<multipart-config>
<location>C:\temp</location>
<max-file-size>377759039</max-file-size>
<max-request-size>-1</max-request-size>
<file-size-threshold>1024</file-size-threshold>
</multipart-config>
```

- dispatcherServlet에 추가한다면 아래와 같다.
```xml
<beans:bean id="multipartResolver"
class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
<beans:property name="maxUploadSize" value="377759039" />
</beans:bean>
```
​
### <span style="color:#FFCCFF">DefaultServlet</span>
- 다음으로 DefaultServlet은 tomcat의 web.xml에도 있다.
- 보통 rest api를 사용할 때는 dispatcherServlet에서도 root를 /로 설정한다.
- 따라서 tomcat의 default servlet설정인 /가 덮어씌워진다.

- 그래서 정적파일들을 처리하지 못한다. 
- 이러한 경우를 대비해 Spring 내에서는 아래와 같은 annotation 처리가 필요하다.
- 그래야 정적인 파일들이 default servlet에서 처리된다.
```
<mvc:default-servlet />

or

<mvc:resources  mapping="/resources/**"  location="/resources/"/>
```
- 만약 DispatcherServlet with name 'disaptcher' processing GET request for [/img/button.jpg]라는 오류가 뜬다면?

- 정적파일을 dispatcherServlet에서 처리하게끔 하고 있다는 의미다.
  - 정적파일을 처리하는 controller는 대부분 따로 만들지 않는다.
  - 저 오류가 난다는 것은 defaultServlet mapping이 되지 않아 오류가 난다는 의미다.
  - 즉 defaultServlet과 관련된 Spring 내 설정이 되어있지 않다는 것이다.
    - 그렇다면 정적파일을 처리해줄 defaultServlet에 위임시키야 한다.
    - 그게 바로 위의 annotation의 역할이다.
​

### <span style="color:#FFCCFF">context xml에 관한 정리</span>
- 정리하면 contextLoadListener로 규정된 xml(root-context)의 설정값을 먼저 읽어들여 전 servlet에서 공유된다.
- 그 다음으로 dispatcherServlet로 규정된 xml의 설정값을 읽어들여 request를 처리할 준비를 마친다.
- 당연히 root에서는 dispatchservlet에서 만들어진 깂인 viewresolver, interceptor, multipartresolver 등 웹과 관련된 component에 접근할 수 없다. 
  - 아직 읽어들이지 않아 존재하지 않기 때문이다.
  - 반면에 dispatchservlet에서는 root-context.xml에서 생성된 값들(service, dao, vo 등)을 활용할 수 있다.
    - 즉 controller에서는 db에서 받아온 데이터들을 담는 service를 활용할 수 있다.
    - 그래서 Controller에 Service를 호출하게 된다. Service에서 Controller의 bean을 호출할 수 없게 구조가 짜였다.
    - 컴퓨팅 자원을 아끼기 위한 고충이 MVC에 들어간 셈이다.

​
## <span style="color:#802548">_4. annotation-config, component-scan, annotation-driven 차이_</span>


- Annotation-config는 @Autowired, @Component 등의 bean을 만들어주는 annotation을 활성화시킨다.
  - 하지만 scan 기능은 없다. 따라서 일일이 전부 해당 class를 xml에 적어주어야 한다.
- 반면에 component-scan은 해당 class를 xml에 적지 않아도 @component라고 붙였다면 scan을 하여 bean으로 만들어준다. 
  - 그러면서 Annotation-config의 기능을 다 갖고 있다. 
  - 즉 component-scan만 선언하면 annotation-config는 필요없다.
- Annotation-driven은 mvc냐 tx냐에 따라 기능이 다르다. 
  - mvc의 경우 HandlerMapping과 HandlerAdapter를 bean으로 등록시켜준다. 
    - HandlerMapping과 HandlerAdapter의 존재 덕분에 별도의 adapter와 mapping이 필요없다.
    - 바로 @Controller를 붙여 사용하면 된다.
  - tx의 경우 @Transactional 애노테이션을 사용할 수 있다.
- 사실 대부분 annotation-config보다는 component-scan을 쓰는게 좋다. 
- 또한 annotation-driven도 annotation을 이용하려면 필수에 가깝다.
  - 그러한 이유로 component-scan과 annotation-driven은 root-context와 servlet-context 모두에서 같이 쓰인다.  

​

- root-context에서는 annotation-driven을 쓰면 @Service 등의 annotation이 활성화된다.
- Servlet-context에서는 annotation-driven을 쓰면 messageConverter 등이 등록된다.
    - messageConverter를 별도로 등록하면 기본으로 등록되는 messageConverter들이 등록되지 않는다. 
    - StringHttpMessageConverter, MappingJackson2 HttpMessageConverter 등록이 별도로 요구된다. 
    - messageConverter는 굳이 custom하지 않는 것을 추천한다. 

## <span style="color:#802548">_5. bean_</span>
### <span style="color:#FFCCFF">bean 형성</span>
- container를 web.xml을 통해 형성했다면 이제 bean을 넣을 차례다.
- 사실 우리는 이미 container에서 bean을 형성하는 것을 보았다.

- 아래와 같은 경우 bean 이름이 dataSource인데, 해당 bean의 class는 HikariDataSource다. 
- 이 bean은 hikariConfig라는 bean 이름(id)을 가진 bean을 참조하여야 하기 때문에 이 bean이 만들어지기 전에 hikariConfig가 있어야 한다.
```xml
<bean id="dataSource"        class="com.zaxxer.hikari.HikariDataSource"
        destroy-method="close">
        <constructor-arg ref="hikariConfig" />
</bean>
```
- hikariConfig라는 id를 가진 bean은 아래와 같이 bean이 만들어질 때 속성 값을 받는다.
- 받는 속성 값은 아래와 같다- driverClassName, jdbcUrl, username, password, dataSourceProperties 
```xml
<bean id="hikariConfig" class="com.zaxxer.hikari.HikariConfig">
        <property name="driverClassName"
            value="org.mariadb.jdbc.Driver"></property>
        <property name="jdbcUrl"
            value="jdbc:mariadb://2133300/MEN323></property>
        <property name="username" value="h2313w" />
        <property name="password" value="pass" />
        <property name="dataSourceProperties">
            <props>
                <prop key="cacheprepStmts">true</prop>
                <prop key="prepStmtCacheSize">250</prop>
                <prop key="prepStmtCacheSqlLimit">2048</prop>
            </props>
        </property>
</bean>
```
- Annotation으로 쓰면 아래와 같이 될 것이다.

```java
@Bean
    public DataSource dataSource() {
        HikariConfig hikariconfig=new HikariConfig();
        hikariconfig.setDriverClassName("org.mariadb.jdbc.Driver");
        hikariconfig.setJdbcUrl("jdbc:mariadb://2141240/ME123234");
        hikariconfig.setUsername("h125125");
        hikariconfig.setPassword("pa124s");
        
        return new HikariDataSource(hikariconfig);
    }

```
- bean은 위와 같이 꼭 명시적으로 작성하지는 않는다.

- 예를 들어 component를 scan하라고 지정할 수도 있다. 
```xml
<context:component-scan base-package="com.example.controller" />
```

- 그 경우 @Controller, @Service, @Component, @Repository를 가진 class는 bean으로 전환된다. 

- 이 때 id는 class명이 첫글자만 소문자로 바뀌게 되는 형태다. 

​
### <span style="color:#FFCCFF">xml 형식과 class 형식 bean</span>
- 예를들어 아래와 같이 PrincipalDetailsService class가 있다고 해보자.
```java
@Service
@RequiredArgsConstructor
public class PrincipalDetailsService implements UserDetailsService {

    private final MemberRepository memberRepository;

    /**
     * @description: 유저 정보를 조회해 Spring Security 규격에 맞게 UserDetails 객체 생성
     */
    @Override
    public UserDetails loadUserByUsername(String memberId) throws UsernameNotFoundException {
        Member member = memberRepository.findByMemberId(memberId);
        if (member != null) {
            return new PrincipalDetails(member);
        }
        return null;
    }

}
```
- 해당 클래스는 아래와 같이 쓸 수 있을 것이다.
```xml
<bean id="principalDetailsService class="com.example.service.PrincipalDetailsService">
  <constructor-arg ref="memberRepository">
</bean>
```
### <span style="color:#FFCCFF">qualifier</span>
- 만약에 id가 다르고 class type이 같은 bean을 만들어야 하는 경우가 있다면, class가 같아서 오류가 나게 된다.
- 그러한 경우에는 @Qualifer로 값을 지정하여 서로 unique함을 보장하여 문제를 해결한다.
```xml
<bean id="dataSource"        class="com.zaxxer.hikari.HikariDataSource"
        destroy-method="close">
        <constructor-arg ref="hikariConfig" />
</bean>
<bean id="dataSource"        class="com.zaxxer.hikari.HikariDataSource"
        destroy-method="close">
        <constructor-arg ref="hikariConfig" />
</bean>
```

### <span style="color:#FFCCFF">bean 초기화, 소멸</span>
- bean은 생성될 때 다른 작업들도 수반되어야 하는 경우가 있다.
  - 이런 경우 InitializingBean을 implements하여 구현한다.
    - afterPropertiesSet() method를 override해 수반되어야 하는 작업들을 넣어주면 된다.
```java
public class ConnPoll implements InitializingBean, DisposableBean {

    @Override
    public void afterPropertiesSet() throws Exception{

    }

}
```
- annotation만 해결하려한다면 @PostConstruct를 사용할 수 있다. 
  - 혹시 InitializingBean과 @PostConstruct를 모두 사용했다면?
  -  @PostConstruct가 먼저 적용되고 나중에 InitializingBean의 method가 덮어씌워진다.
- 즉 최종으로는 InitializingBean의 afterPropertiesSet() method가 적용된다.
```java
public class ConnPoll{

    @PostConstruct
    public void initPool() throws Exception{

    }

}
```
- bean이 소멸할 때 해주고 싶은 작업이 있다면  DisposableBean이나 @PreDestory를 사용할 수 있다. 
```java
public class ConnPoll implements DisposableBean {

    @Override
    public void destroy() throws Exception{
        
    }
}
public class ConnPoll{
    
    @PreDestroy
    public void destroyPool() throws Exception{
        
    }
}
```
## <span style="color:#802548">_5. DI_</span>
- DI는 주로 쓰이는 Constructor와 @Autowired를 중심으로 설명하겠다. 
- DI는 Dependency Injection의 줄임말이다. 

​

- 먼저 @Autowired에 대해 살펴보자. 
- @Autowired는 bean이 있다면 해당 bean을 inject해주는 annotation이다. 
- <annotation-driven>이라는 태그를 활성화시키면 @Autowired를 사용할 수 있다. 
```java
@Autowired  
private ConnPoll conpoll
```
- 혹시 똑같은 bean id와 class가 있다면, 아래와 같이 Qualifier를 찾게하여 unique한 bean으로 만들어줘야 한다. 
```java
@Autowired  @Qualifier("m1")
private ConnPoll conpoll
```
- xml에서 qualifier 설정도 되어 있어야 한다.
```xml
<bean id="conpoll"        class="com.example.connection.ConnPoll"
        destroy-method="destroy">
        <qualifier value=”m1” />
</bean>
```
- 다만 @Autowired의 문제점은 순환참조에 취약하다는 것이다.
  - 순환참조임에도 compile 때 애플리케이션이 아무런 오류나 경고없이 구동된다는 것이다.
  - 그러다 runtime에 오류가 발생하면 멈춰버린다.
  - 즉 실제 코드가 호출되기 전까지 문제를 발견할 수 없다. 
- 따라서 constructor 방식이 더 추천된다.
​

- 생성자의 경우에는 아래와 같은 형식으로 만든다.

```java
public class db{
   public db(ConnPoll conpoll){
      this.conpoll=conpoll
   }
}
```

- ​lombok을 활용하면 다음과 같이 간소하게 쓸 수 있다.
- boot에서는 lombok이 자동지원된다.

```java
@RequiredArgsConstructor
public class db{
   private final ConnPoll conpoll
}
```
- 생성자 주입의 장점은 final을 사용할 수 있어 혹시 inject를 실행하는 class의 속성(여기서는 ConnPoll) 이 변경되는 현상을 막을 수 있다는 점이다.

(1)IOC

IOC란 개발자가 아니라 spring이 객체를 생성시키는 것을 의미한다. 개발자가 new로 객체 새로 생성하지 말고 Spring의 관리 감독 하에 두라는 것이다. new로 객체를 매번 생성한다면 객체가 새로 생성돼서 객체가 공유되지 않지만, Spring의 관리 감독 아래 두면 singletone 객체로서 관리되기에 처음 생성된 거로 계속 가져올 수 있다. 이 점은 메모리에 부담을 덜 주며 context가 공유된다는 장점이 있다. 

​

(2)서로 다른 Container

Spring에서 쓰는 모든 코드가 Spring에 속한 것은 아니다. 대표적으로 web container인 tomcat이 적용하는 filter는 spring 이전의 영역에 속한다. tomcat의 filter는 spring context에 종속되지 않는다. 

​

Tomcat의 filter 이후에 동작하는 spring의 거름망은 interceptor라고 부른다. filter에서 먼저 거르고, 그 다음에 spring의 interceptor가 다시 한번 거른다. filter는 전역 보안, 문자열 인코딩, 이미지나 데이터 압축 등에 쓰이고 interceptor는 client의 요청과 관련된 인가 작업, api 호출정보 기록등에 쓰인다고 한다. 그런데 jwt는 interceptor가 아닌 filter로 구현하는데, 그 이유는 나도 잘 모르겠다. 

​

(3)@ResponseBody, @RequestBody의 인코딩 방식

영어 한문자를 8bit로 표현가능하다. 1바이트가 8bit다. 그래서 1바이트가 통신단위다. 한글은 16비트, 중국어는 24비트다. 그래서 유니코드는 중국어같이 큰 것에 맞춰서 3바이트짜리가 된다. 다만 유니코드는 변환이 가능하다. 이러한 바이트를 받는 것이 inputstream이다. 그런데 바이트로 문자를 처리하면 단점이 많아서 문자열로 바꾼다. 거기에 처음 썼던게 inputstreamReader다. 하지만 inputStreamReader로 쓰면 바이트를 정해놔야 해서 정보량이 적은 글자를 보낼 때 손해가 많다. 글자를 많이 안보낼수록 그만큼 바이트가 낭비된다. 그래서 가변적인 bufferedreader를 쓴다.

​

자바에서는 HttpServletRequest 안에서 getReader()를 할 때 BufferedReader()로 가져오는거 BufferedReader는 printWriter 혹은 JSP 내장 객체 out을 사용한다. 하지만 스프링에선 @ResponseBody를 사용하면 BufferedWriter고 @RequestBody를 사용하면 BufferedReader를 사용하게 되는 것이다.

​

(10)웹서버와 WAS

웹서버는 소스코드를 이해하지 못함. 그래서 소스코드를 이해할 수 있는 WAS가 필요하게 된다. WAS인 톰캣은 웹서버가 이해하지 못하는 JSP를 컴파일하여 html로 만들어 웹서버에게 돌려주는 역할을 한다. html로 와야만 웹브라우저가 이해할 수 있다. 웹브라우저는 html, js, css, avi 등 정적 콘텐츠만을 이해할 수 있기 때문이다. spring은 URL 접근 요청을 다 막혀있다. 특정한 파일을 요청할 때는 반드시 자바를 거쳐야 함. 따라서~~~~/a.png(URL)로는 안되고, ~~~/picture/a(URI)로 써줘야함. 따라서 URI로만 실체를 얻음. 물론 webapp에 넣으면 URL로도 접근은 가능하지만, 보안 상 권장되지 않는다.

​

(11)Servlet

톰캣은 html, css가 아니라 java자원에 관한 요청이 최초로 들어오면 servlet 객체를 init()하고, 스레드를 생성한 뒤 그 스레드에서 doget, dopost 등 service()를 호출한다. get이라면 get에서 DB연결하고 데이터를 받아 html로 응답한다. 최초가 아니면 이미 생성된 servlet 객체를 사용하여 새로운 스레드를 만들어 요청을 처리한다. 

​

그런데 스레드 수는 한계가 있다. 예로 20개까지만 스레드를 생성할 수 있다면 20명 이후에도 응답이 안끝났는데

10명이 추가로 요청이 들어오면 10명은 대기를 해야한다. 그리고 스레드는 요청이 끝난다고 지워지지 않고 대기자의 요청을 처리하게 된다. 예로 스레드1이 1번을 끝냈다면,  이제는 21번의 요청을 처리하게 된다. 즉 서블릿객체는

반드시 1개고, 스레드가 복수개다. 이렇게 스레드를 여러개로 만들어 처리하는 방법을 풀링기술이라고 한다. 컴퓨터 성능을 올려 스레드를 20개에서 100개로 만드는 건 스케일 업, 그러한 서버 자체 대수를 늘리는 건 스케일 아웃이라고 한다.

​

(12)web.xml

web.xml은 인증(session), 필터링(filter), 리스너(event) 등이 있음. 그런데 web.xml에 너무 많은 내용이 들어가 있으면 버벅인다. 그래서 특정 uri가 들어오면 FrontController로 넘겨버린다. 예를 들어 a.do를 낚아채라고 하면, FrontController가 a.do 요청을 낚아채서 내부적으로 그 자원을 request하는 요청이 만들어진다. 그런데 A라는 client의 기존 request를 덮어씌워 지워버리면 안되기 때문에, requestDispatcher를 이용한다.그러면 기존의 request에 있는 데이터를 유지하면서 재사용 가능하게 된다.

​

A클라가 a.jsp를 B서버에 요청하면 B서버는 a.html을 response한다. 만약 다음 페이지 버튼을 누르면 b.jsp를 요청한다면, b.html을 resp하게 된다. requestDispatcher를 사용하면 a.jsp을 request했을 때 client가 보냈던 데이터를  b.jsp와 관련한 response에서도 서버가 사용할 수 있다. 즉 requestDispatcher를 이용해야만 페이지 간 데이터 이동이 가능하다.

​

(13)dispatcherServlet(servlet-context)

그런데 FrontController와 requestDispatcher를 묶어논 것이 바로 DispatchServlet이다. 따라서 직접구현하지 않아도된다. jsp는 직접 짜야 한다. 보통은 dispatchServlet이 생성될 때 component가 등록된다.

​

request가 오면, web.xml을 탄 뒤 DispatchServlet으로 가게 된다. 그럼 dispatchServlet은 component scan을 수행하여, 해당 request를 처리할 수 있는 memory에 떠있는 class를 탐색한다. 따라서 DispatchServlet은 우선 memory에 올린다. src 폴더 내부에 있는 파일을 스캔한다. 즉 해당 프로젝트의 웹과 관련된 자바파일을 뒤져서 필요한 것들을 메모리에 올린다. 
필요한 것은 annotation으로 정해져있다. @Controller, @RestController, @Component 등이다. 이것이 붙어있다면 메모리에 load시킨다. spring은 new를 사용해서 띄우지 않는 걸 원칙으로 하기 때문에 IOC를 위해 annotation으로 띄우는 것이다. 


(15)Bean Factory

applicationContext(root와 servlet)말고 bean factory에서도 IOC로  메모리 load가 가능하다. @configuration이 붙어있는 class 안에서 객체를 return 하는 method에 @Bean을 붙이면 된다. 그러한 @Bean은 처음부터 compile할 때 memory에 load시키지 않고 필요할 때만 불러서 load된다. 즉 lazy-loading이 가능하다. runtime loading이다.

​



## <span style="color:#802548">_in case of x-www-form-urlencoded, how to generate immutable DTO in Spring_</span>
- 불변성을 유지하는 것은 무엇보다 중요하다.
- Enttiy class들은 어쩔수없지만, DTO class들은 immutability를 유지하는 방향으로 가보자.
- 아래가 일반적으로 사용되는 DTO다.

```java
@Getter
@Setter
@ToString
@NoArgsConstrutor
public class CarResponseDTO {
    private  Long carSeq;
    private  String carType;
    private  boolean carStatus;

}
```

- immutability로 만들려면 아래와 같은 조건을 만족해야 한다.
    - final class
    - final field
    - only creation in Consturctor, no setter method
    - when changing, using new ()


- 따라서 아래와 같이 바꿔준다.
- class와 field를 모두 final로 바꿨다.

```java
@Getter
@Setter
@ToString
@NoArgsConstrutor
public final class CarResponseDTO {
    private final Long carSeq;
    private final String carType;
    private final boolean carStatus;

}
```

- 하지만 안타깝게도 Spring에서는 Data를 받으면　@NoArgsConstructor가 필요하다.
    - @RequestBody로 받을 때는 Jackson이 DTO mapping 시에 @NoArgsConsturctor를 필요로 한다.
        - 
    - @ModelAttribute로 받을 때는 Spring 차원에서 @NoArgsConstructor를 필요한다.
        - reflection으로 data를 binding하는데, 그 때 쓰이는 Class.newInstance()는 NoArgsConstrucotr를 필요로 한다.
- immutability를 활용하려면 결국 Reflection api의 기본 동작인 NoArgs를 활용하면 안 된다.

```
Exception ~~~~~~~
```


- Reflection api에는 다행히 NoArgs 말고 Parametetized constructor를 활용할 수 있다.
- 대신 Reflection api에 binding할 name을 전부 명시해주어야 한다.
- 그럼 Class.newInstance()가 아니라 Constructor.newInstance()가 발동한다.
    - Parametetized constructor를 활용하는 것이다.

```java
@Getter
@ToString
public final class CarResponseDTO {
    private final Long carSeq;
    private final String carType;
    private final boolean carStatus;

    @Builder
    @ConstructorProperties({"carSeq","carType", "carStatus"})
    public CarResponseDTO(Long carSeq, String carType, boolean carStatus) {
        this.carSeq = carSeq;
        this.carType = carType;
        this.carStatus = carStatus;
    }
}
```


## <span style="color:#802548">_x-www-form-urlencoded일 경우 response와 request DTO분리하기_</span>

- request용과 response용은 서로 쓰이는 field가 다르다.
    - 그런데 final class는 값이 생성자를 만들때까지는 반드시 설정되어야 한다.   
    - 따라서 null로 초기값을 설정해야 하는데, 그러면 해당 field를 response용에선 쓸 수 없다.
    - 서로를 분리해주자.

```java
@Getter
@ToString
public class CarInsertRequestDTO {
    private final Long carSeq;
    private final String carType;

    @Builder
    @ConstructorProperties({"carSeq","carType"})
    public CarInsertRequestDTO(Long carSeq, String carType) {
        this.carSeq = carSeq;
        this.carType = carType;
    }
}
```

- response에서는 data를 Spring이 binding하지 않는다. 
    - @ModelAttribute라던가 @RequestBody 등의 annotation이 없다.
- 따라서 @ConstructorProperties가 불필요하다. 
    - 혹시 response에서도 binding이 필요하다면 써야 한다.

```java
@Getter
@ToString
public class CarResponseDTO {
    private final Long carSeq;
    private final String carType;
    private final boolean carStatus;

    @Builder
    //@ConstructorProperties({"carSeq","carType", "carStatus"})
    public CarResponseDTO(Long carSeq, String carType, boolean carStatus) {
        this.carSeq = carSeq;
        this.carType = carType;
        this.carStatus = carStatus;
    }z
}
```

- 서로 분리해주었다면, 이제는 기본형은, 프론트에서 값을 무조건 주는 형태로 진행되게 될 것이다. 
- 따라서 객체형이 아닌 기본형으로 변경가능하다. 별건 아니지만, 성능이 좀 더 좋아진다.

```java
@Getter
@ToString
public class CarInsertRequestDTO {
    private final long carSeq;
    private final String carType;

    @Builder
    @ConstructorProperties({"carSeq","carType"})
    public CarInsertRequestDTO(long carSeq, String carType) {
        this.carSeq = carSeq;
        this.carType = carType;
    }
}
```


## <span style="color:#802548">_x-www-form-urlencoded일 경우  domain 별로 모은 static inner class화_</span>

- domain 별로 명확하게 구분짓기 위해서 static inner class를 활용하는 것도 좋은 선택이다.
- top level class는 1개로 설정 하는 대신, 아무런 method와 field를 가지지 않는다.

```java
public class CarDTO {
    
    @Getter
    @ToString
    public static final class CarRequestDTO {
        private final Long carSeq;
        private final String carType;

        @Builder
        @ConstructorProperties({"carSeq","carType"})
        public CarRequestDTO(Long carSeq, String carType) {
            this.carSeq = carSeq;
            this.carType = carType;
        }
    }

    @Getter
    @ToString
    public static final class CarResponseDTO {
        private final Long carSeq;
        private final String carType;
        private final boolean carStatus;

        @Builder
        public CarResponseDTO(Long carSeq, String carType, boolean carStatus) {
            this.carSeq = carSeq;
            this.carType = carType;
            this.carStatus = carStatus;
        }
    }
}
```

## <span style="color:#802548">_Json type일 경우 immutable DTO class_</span>
- JSON일 경우, Jackson library로 parsing을 진행하기 때문에, 다른 annotation을 써야 한다.
    - JSON serialization/deserialization와 java beans의 serialization/deserialization의 구조가 다르기 때문이다.
    - JavaBeans는 DTO class같은 Java의 객체들이다. 주로 www-x-form-urlencoded로 content를 front와 주고받을 때 사용한다.
    - Json은 주로 application/json으로 content를 front와 주고 받을 때 사용한다. 

```java
@Getter
@ToString
public final class CarResponseDTO {
    private final Long carSeq;
    private final String carType;
    private final boolean carStatus;

    @Builder
    @JsonCreator
    public CarResponseDTO(@JsonProperty("carSeq")       Long carSeq, 
                          @JsonProperty("carType")      String carType, 
                          @JsonProperty("carStatus")    boolean carStatus) {
        this.carSeq = carSeq;
        this.carType = carType;
        this.carStatus = carStatus;
    }
}
```

## <span style="color:#802548">_더 간단하게, record class_</span>
- record를 쓰면 여러가지 고충을 덜을 수 있다.
    - final class로 선언할 필요X
    - final field 선언할 필요 X
    - getter, setter, NoArgs, ReArgs, ConstructorProperties 선언할 필요 X
    - content type 변경에 따른 class field parsing annotation 교체 X
        - front framework를 사용하게 되면 www-x-form-urlencoded에서 application/json으로 변경된다.

```java
public record CarRequestDTO(Long carSeq, String carType){

}
```



## <span style="color:#802548">_domain 별로 모은 static record화_</span>
- record의 경우 top level이면 당연히 static이다.
- 그러나 inner에서는 static을 붙여줘야 한다.

```java
public class CarDTO {
    
    @Builder
    public static record CarRequestDTO(Long carSeq, String carType) {
        public static CarRequestDTO TODTO(CarEntity carEntity) {
            return CarRequestDTO.builder().carSeq(carEntity.getCarSeq())
                                            .carType(carEntity.getCarType())
                                            .build();
        }
    }

    @Builder
    public static record CarResponseDTO(Long carSeq, String carType, boolean carStatus) {
        public static CarResponseDTO TODTO(CarEntity carEntity) {
            return CarResponseDTO.builder()
                                    .carSeq(carEntity.getCarSeq())
                                    .carType(carEntity.getCarType())
                                    .carStatus(carEntity.getYnEnum().getBooleanValue()) 
                                    .build();
                            
        }
    }
}
```


## <span style="color:#802548">_요약_</span>
- form 방식은 아래와 같다.

```
front content-type: www-x-form-urlencoded
controller annotation: @ModelAttribute
serialization/deserialization mechanism: form binding (like JavaBeans serialization/deserialization)
Dto annotation for immutability: @ConstructorProperties
```

- 변환은 Spring이 알아서 해준다.

```java
public class MyDto {
    private String name;
    private int age;

    // Getters and Setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

//name=John&age=25
```

- Json 방식은 아래와 같다.
```
front content-type: application/json
controller annotation: @RequestBody
serialization/deserialization mechanism: Jackson library (JSON serialization/deserialization)
DTO annotation for immutability: @JsonCreator, @JsonProperties
```

- 변환은 Spring이 아니라 Jackson이 알아서 해준다.

```java
public class MyDto {
    private String name;
    private int age;

    // Getters and Setters
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

/*
    {
        "orderSeq": 123,
        "carSeq": 456
    }
/*
```
