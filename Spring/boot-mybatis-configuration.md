## <span style="color:#802548">_1. Pom.xml에 Mybatis dependency 넣기_</span>

​

- pom.xml에 아래와 같이 1개만 넣으면 된다.
```xml
<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>2.1.2</version>
		</dependency>
```
## <span style="color:#802548">_2. properties 파일에 db정보 작성하기_</span>

​

- root-context.xml 대신 application.properties 파일에 db정보를 적으면 된다.
- setting정보나 sql mapper xml의 위치도 아래와 같이 자신이 직접 지정할 수 있다. 
- classpath는 src/main/java 혹은 src/main/resources 경로라고 생각하면 된다. 

```xml
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://211.118.245.244:4100/MENTO_HJW
spring.datasource.username=root
spring.datasource.password=open1404
mybatis.mapper-locations=classpath:mapper/**/*.xml
mybatis.config-location=classpath:mybatis-config.xml
```

- 내 경우에는 resources 아래이며 대부분 그렇게 한다.


## <span style="color:#802548">_3. mybatis-config.xml_</span>
- 아래는 기본 설정은 아니다. 근데 무조건 쓰는 게 좋다. 없으면 실수가 많이 날 수밖에 없다.
- 해당 설정을 해두면 resultMap을 쓸 떄 매우 간결하게 쓸 수 있다.
```xml
<settings>
		<setting name="mapUnderscoreToCamelCase" value="true" />
	</settings>
<configuration>
	<typeAliases>
		<typeAlias alias="user" type="com.example.demo.domain.UserDto" />
		<typeAlias alias="board" type="com.example.demo.domain.BoardDto" />
	</typeAliases>
</configuration>
```

## <span style="color:#802548">_4. sql mapper xml_</span>
- 위의 mapUnderscoreToCamelCase를 안 쓰면 아래같이 property를 전부 일일이 지정해주고, jdbcType과 javaType을 맞춰줘야 한다.
```xml
<mapper namespace="com.example.demo.mapper.BoardMapper">

	<resultMap id="boardInfo" type="board">
		<result column="board_no" property="boardNo" 
			jdbcType="INTEGER" javaType="int" />
		<result column="board_title" property="boardTitle" 
			jdbcType="VARCHAR" javaType="String" />
		<result column="board_content" property="boardContent"
			jdbcType="CHAR" javaType="String" />
		<result column="board_writer" property="boardWriter"
			jdbcType="VARCHAR" javaType="String" />
		<result column="board_register_date" property="boardRegisterDate"
			jdbcType="DATE" javaType="java.sql.Date" />
		<result column="board_photo" property="boardPhoto"
			jdbcType="VARCHAR" javaType="String" />
		<result column="board_view" property="boardView" 
			jdbcType="INTEGER" javaType="int" />
	</resultMap>
	.
    .
    .
```


- 하지만 configuration을 사용하면 resultMap이 아래와 같이 간략하게 줄어든다.

```xml
<resultMap id="boardInfo" type="board"/>
```

- 아래는 실제 sql문이다.
```xml
<select id="read" resultMap="boardInfo">
		select board_no, board_title, board_content,
		board_writer, board_register_date, board_photo, board_view
		from board 
		where board_no=#{boardNo}
	</select>

	<select id="list" resultMap="boardInfo">
		select board_no, board_title, board_content,
		board_writer, board_register_date, board_photo, board_view
		from board
		<if test='searchType != null and searchType.equals("제목")'>
			WHERE board_title LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("내용")'>
			WHERE board_content LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("작성자")'>
			WHERE board_writer LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("제목과 내용")'>
			WHERE board_title LIKE concat('%', #{keyword}, '%') 
			or board_content LIKE concat('%', #{keyword}, '%') 
		</if>
		limit #{start},6
	</select>

	<select id="total" resultType="int">
		select count(board_no)
		from board
		<if test='searchType != null and searchType.equals("제목")'>
			WHERE board_title LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("내용")'>
			WHERE board_content LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("작성자")'>
			WHERE board_writer LIKE concat('%', #{keyword}, '%')
		</if>
		<if test='searchType != null and searchType.equals("제목과 내용")'>
			WHERE board_title LIKE concat('%', #{keyword}, '%') 
			or board_content LIKE concat('%', #{keyword}, '%') 
		</if>
	</select>
```



## <span style="color:#802548">_5. mapper(repostiroy)_</span>
- mapper는 위에 규정된 sql mapper xml의 namespace와 실제 경로가 동일해야 한다.
- src/main/java까지는 classpath라 생략된 것이고, 그 이후부터의 경로가 동일해야 한다.
```xml
<mapper namespace="com.example.demo.mapper.BoardMapper">
```

 - interface의 method는 아래와 같은 조건을 만족해야 한다.
   - sql mapper xml의 id와 동일해야 한다.
  - parameter로 들어가는 변수명이 복수 개일 경우, @Param으로 지정한다. SqlSession과 다르게 매개변수 복수 개를 직접 써넣을 수 있다. hashMap을 쓰지 않아도 된다.

```java
List<BoardDto> list(@Param("start") int page, @Param("searchType") String searchType, @Param("keyword")String keyword);
```
- 한 개짜리는 @Param을 달 필요가 없다. 
```java
void increaseView(int boardNo);
```
- class는 자동으로 mapping돼서 @Param을 달지 않아도 된다. 
```java
void insert(BoardDto boardDto);
```
- returntype도 동일해야 한다. 
- sql mapper에서 나오는 result의 type이 int라면, 아래와 같이 interface의 method도 int type이어야 한다. 
```xml
<select id="total" resultType="int">
```

```java
int total(@Param("searchType") String searchType, @Param("keyword")String keyword);
```
- list type을 return하는 경우에는 sql mapper xml에 limit와 같이 복수개의 row를 출력하는 결과를 산출하는 select 문이 필요하다.
```xml
<select id="list" resultMap="boardInfo">
select * from board
.
.
.
limit #{start},6
```
- mapper로 db와 통신하려면 아래에 @Mapper를 달아줘야 하며 interface type으로 만들어야 한다. 
```java
@Mapper
public interface BoardMapper {

	List<BoardDto> list(@Param("start") int page, @Param("searchType") String searchType, @Param("keyword")String keyword);

	int total(@Param("searchType") String searchType, @Param("keyword")String keyword);

	BoardDto read(int boardNo);

	void insert(BoardDto boardDto);
	
	void delete(int boardNo);
	
	void update(BoardDto boardDto);
	
	void increaseView(int boardNo);
}
```
- mapper를 써놨다면 아래와 같이 MapperScan을 통해 Mapper를 bean으로 등록시켜준다. 
```java
@MapperScan(basePackages ={"com.example.demo.mapper"})
public class DemoApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
```
## <span style="color:#802548">6. Service</span>
- 여기서는 start를 직접 만들어서 가져가야 한다. 
- legacy의 경우 dao에서 start 값을 만들어서 db에 뿌렸지만, 여기서는 service에서 start 값을 만들어서 가져간다. 
```java
public List<BoardDto> getList(int page, String searchType, String keyword) {
		int start = (page - 1) * 6;
		List<BoardDto> list = boardMapper.list(start, searchType, keyword);

		return list;
	}
```