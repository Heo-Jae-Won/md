## <span style="color:#802548">_1. Pom.xml에 Mybatis dependency 넣기_</span>
- 자신의 Spring Framework에 맞는 버전을 가져와야 한다.
  - mybatis
  - mybatis-spring
```xml
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis</artifactId>
    <version>3.2.8</version>
</dependency>
<dependency>
    <groupId>org.mybatis</groupId>
    <artifactId>mybatis-spring</artifactId>
    <version>1.2.2</version>
</dependency>
```
## <span style="color:#802548">_2. Root-context.xml_</span>
- Root-context.xml에 SqlSession bean을 정의한다. 
- 보통은 다음과 같이 나올 것이다.
```
jdbc:mysql://127.0.0.1:3306/project
```
- 아래 url은 localhost url이며 log4jdbc라는 sql을 예쁘게 나오게 하는 dependency를 사용하여 url이 좀 다르다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.1.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.1.xsd">
	<!-- Root Context: defines shared resources visible to all other web components -->
	<bean id="dataSource"
		class="org.springframework.jdbc.datasource.DriverManagerDataSource">
		<property name="driverClassName"
			value="net.sf.log4jdbc.sql.jdbcapi.DriverSpy"></property>
		<property name="url"
			value="jdbc:log4jdbc:mysql://127.0.0.1:3306/projectdb" />
		<property name="username" value="project" />
		<property name="password" value="pass" />
	</bean>
	<bean id="sqlSessionFactory"
		class="org.mybatis.spring.SqlSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="mapperLocations"
			value="classpath:/mapper/**/*.xml" />
	</bean>
	<bean id="sqlSession"
		class="org.mybatis.spring.SqlSessionTemplate">
		<constructor-arg index="0" ref="sqlSessionFactory" />
	</bean>
	<bean id="transactionManager"
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	<tx:annotation-driven />
	<context:component-scan
		base-package="com.example.dao" />

	<context:component-scan
		base-package="com.example.service" />
</beans>
```
- root-context.xml의 namespace에서 mybatis-spring에도 check해준다.
## <span style="color:#802548">_3. mybatis-config.xml_</span>
- mybatis의 환경을 setting해준다. 
- 특히 아래 환경설정은 거의 필수급이다. 해두면 진짜 편하다.

```xml
<settings>
    <setting name="mapUnderscoreToCamelCase" value="true" />
</settings>
```
- 나머지는 package를 전부 다 쓸 필요없게 alias를 설정하는 과정이다.
```xml
<typeAliases>
		<typeAlias alias="user" type="com.example.dto.UserDto" />
		<typeAlias alias="productBoard"
			type="com.example.dto.ProductBoardDto" />
		<typeAlias alias="event" type="com.example.dto.EventDto" />
		<typeAlias alias="eventReply"
			type="com.example.dto.EventReplyDto" />
		<typeAlias alias="notice" type="com.example.dto.NoticeDto" />
		<typeAlias alias="pay" type="com.example.dto.PayDto" />
		<typeAlias alias="productBoardLike"
			type="com.example.dto.ProductBoardLikeDto" />
		<typeAlias alias="review" type="com.example.dto.ReviewDto" />
	</typeAliases>
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<settings>
		<setting name="mapUnderscoreToCamelCase" value="true" />
	</settings>
	<typeAliases>
		<typeAlias alias="user" type="com.example.dto.UserDto" />
		<typeAlias alias="productBoard"
			type="com.example.dto.ProductBoardDto" />
		<typeAlias alias="event" type="com.example.dto.EventDto" />
		<typeAlias alias="eventReply"
			type="com.example.dto.EventReplyDto" />
		<typeAlias alias="notice" type="com.example.dto.NoticeDto" />
		<typeAlias alias="pay" type="com.example.dto.PayDto" />
		<typeAlias alias="productBoardLike"
			type="com.example.dto.ProductBoardLikeDto" />
		<typeAlias alias="review" type="com.example.dto.ReviewDto" />
	</typeAliases>

</configuration>
```
## <span style="color:#802548">_4. Data Transfer Object_</span>
- lombok을 적용해 아래와 같이 만들어준다. setter를 뺀다면 builder를 대신 넣어도 된다.
```java
@Data
public class EventVO {

	@JsonFormat(shape = JsonFormat.Shape.STRING, pattern = "yyyy-MM-dd", timezone = "Asia/Seoul")
	private Date regDate;
	private int ecode;
	private String etitle;
	private String econtent;
	private String ewriter;
}
```
## <span style="color:#802548">_5. sql mapper xml_</span>
- mybatis에서 xml을 쓸 때 변수는 #{}로 표기한다.
- 변수값에 따라 쿼리문을 변화시키고 싶다면 if test와 같은 조건문을 사용할 수 있다. 
- 쿼리는 select, update, delete, insert가 있다. 

​

- 그외에도 selectKey 등을 활용하여 방금 만들어낸 값을 가져와서 소스코드내에서 활용할 수도 있다. 
- 그에 대해서는 아래 사이트를 참고하라.

​

https://yookeun.github.io/java/2014/07/11/mybatis-selectkey/

- 아래는 현재 resultMap이 적용되지 않은 상태다.
- 이경우 VO를 camelCase가 아니라 snake로 써야해서 자바의 convetion이 깨진다.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.EventMapper">
	 <select id="list"
		resultType="com.example.domain.EventVO">
		select
		ecode, etitle, econtent,
		ewriter,regDate
		from tbl_event
		<if test='searchType != null &amp;&amp; searchType.equals("제목")'>
			WHERE etitle LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null &amp;&amp; searchType.equals("내용")'>
			WHERE econtent LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null &amp;&amp; searchType.equals("제목과 내용")'>
			WHERE etitle LIKE concat('%', #{keyword}, '%')
			or econtent
			LIKE concat('%', #{keyword}, '%')
		</if>

		limit #{start}, #{num}
	</select>

	<select id="getTotal" resultType="int">
		select count(*) from tbl_event
		<if test='searchType != null &amp;&amp; searchType.equals("제목")'>
			WHERE etitle LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null &amp;&amp; searchType.equals("내용")'>
			WHERE econtent LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null &amp;&amp; searchType.equals("제목과 내용")'>
			WHERE etitle LIKE concat('%', #{keyword}, '%')
			or econtent
			LIKE concat('%', #{keyword}, '%')
		</if>
	</select>

 	  <select id="read" resultType="com.example.domain.EventVO">
	 	select * from tbl_event where ecode=#{ecode};
 	 </select>

 	 <delete id="delete">
 	 	delete from tbl_event where ecode=#{ecode}
 	 </delete>

 	 <insert id="insert">
 	 	insert into tbl_event(ecode,etitle,econtent,ewriter)
 	 	values(#{ecode},#{etitle},#{econtent},#{ewriter})
 	 </insert>

 	 <update id="update">
 	 	update tbl_event set etitle=#{etitle} , econtent=#{econtent}
 	 	where ecode=#{ecode}
 	 </update>

</mapper>
```
- 그래서 resultmap을 적용하여 camelCase로 변환한다.
- 그럴 떄 바로 위의 mybatis-config.xml에 설정해둔 것을 활용할 수 있다. 

- 자동 mapping 설정해둔 설정을 적용하자. 
- 그럼 곤란하게 jdbcType이나 javaType을 전부 손으로 다 쓸 필요도 없다. 
```xml
//mybatis-config.xml
<settings>
		<setting name="mapUnderscoreToCamelCase" value="true" />
	</settings>
<configuration>
	<typeAliases>
		<typeAlias alias="event" type="com.example.dto.EventDto" />
	</typeAliases>
</configuration>
```
- 그럼 아래와 같이 깔끔하게 줄어든다. 
- type이 event라는 것은 바로 eventDto 형태를 return해야함을 의미한다.
```xml
<resultMap id="eventInfo" type="event"/>

<select id="list" resultMap="eventInfo">
    select
        event_code, event_title, event_content, event_writer,event_regDate
    from event
    <if test='searchType != null &amp;&amp; searchType.equals("제목")'>
        WHERE event_title LIKE concat('%', #{keyword}, '%')
    </if>
    <if test='searchType != null &amp;&amp; searchType.equals("내용")'>
        WHERE event_content LIKE concat('%', #{keyword}, '%')
    </if>
    <if test='searchType != null &amp;&amp; searchType.equals("제목과 내용")'>
        WHERE event_title LIKE concat('%', #{keyword}, '%')
        or event_content
        LIKE concat('%', #{keyword}, '%')
    </if>
    limit #{start}, #{num}
</select>
```
## <span style="color:#802548">_6. Dao(repository)_</span>
```java
public interface EventDAO {
	public List<EventVO> list(int page, int num, String searchType, String keyword) throws Exception;
	public EventVO read(int ecode);
	public void insert(EventVO vo);
	public void update(EventVO vo);
	public void delete(int ecode);
	public int getTotal(String searchType, String keyword);
}
```
- Dao를 implement class가 실제 구현체인데, SqlSession class를 이용해 db와 소통한다. 
```java
@Autowired
SqlSession session;
```
- 이 SqlSession class가 bean으로 존재해야만 하며, root-context.xml에 등록되어 있어야만 한다.
```xml
<bean id="sqlSession"
    class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg index="0" ref="sqlSessionFactory" />
</bean>
```
- 아래 namespace 값은 sql mapper xml에서 규정한 mapper namespace와 동일해야 한다. 
```java
String namespace="com.example.mapper.EventMapper";
```
- session은 selectOne(row 1개), selectList(row 복수개), udpate, insert, update, delete를 사용할 수 있다. 
- DML은 다 쓸 수있다. xml에 들어갈 변수는 모두 map에 담아준다. 
- 인수로 한 개만 가져갈 수 있어 그런 방식을 사용할 수 밖에 없다. 

```java
@Override
	public List<EventVO> list(int page, int num, String searchType, String keyword) throws Exception {
		HashMap<String, Object> map = new HashMap<String, Object>();
		map.put("start", (page - 1) * num);
		map.put("num", num);
		map.put("searchType", searchType);
		map.put("keyword", keyword);
		return session.selectList(namespace + ".list", map);
	};
```
- 다만 위에 보면 알겠지만 .list와 같이 쓰는 저 id이름(list)이 sql mapper xml에 실제로 존재해야 한다.
```xml
 <select id="list"
		resultType="com.example.domain.EventVO">
<select id="getTotal" resultType="int">
```

```java
@Repository
public class EventDAOImpl implements EventDAO{

	@Autowired
	SqlSession session;
	String namespace="com.example.mapper.EventMapper";

	@Override
	public List<EventVO> list(int page, int num, String searchType, String keyword) throws Exception {
		HashMap<String, Object> map = new HashMap<String, Object>();
		map.put("start", (page - 1) * num);
		map.put("num", num);
		map.put("searchType", searchType);
		map.put("keyword", keyword);
		return session.selectList(namespace + ".list", map);
	};

	@Override
	public int getTotal(String searchType, String keyword) {
		HashMap<String, Object> map = new HashMap<String, Object>();
		map.put("searchType", searchType);
		map.put("keyword", keyword);
		return session.selectOne(namespace + ".getTotal", map);
	}

	@Override
	public EventVO read(int ecode) {
		return session.selectOne(namespace+".read",ecode);
	}

	@Override
	public void insert(EventVO vo) {
		session.insert(namespace+".insert",vo);
	}

	@Override
	public void update(EventVO vo) {
		session.update(namespace+".update",vo);

	}

	@Override
	public void delete(int ecode) {
		session.delete(namespace+".delete",ecode);

	}

}
```
## <span style="color:#802548">_7. Service_</span>
- 그럼 아래와 같이 Service에서 dao를 활용할 수 있다.
```java
@Service
public class EventServiceImpl implements EventService {

	@Autowired
	EventDAO edao;

	@Autowired
	EreplyDAO erdao;

	@Transactional
	@Override
	public void eventDelete(int ecode) {
		erdao.allDelete(ecode);
		edao.delete(ecode);
	}
}
```