- mysql cli에서 한글 깨질 떄 아래와 같이 써준다.
```sh
ALTER DATABASE your_database_name CHARACTER SET utf8 COLLATE utf8_general_ci;

mysql -u your_user -p  --port 3355 --default-character-set=utf8 databaseName
```

## <span style="color:#802548">_SQL 해석 과정_</span>
- SQL 실행 -> 파서 ->전처리 ->옵티마이저 실행 계획 -> 스토리지 엔진에서 데이터 획득 ->결과 전달
  - 파서는 sql syntax 오류를 점검한다.
  - 전처리는 해당 db 오브젝트(table, view, column, index, 권한)이 실제로 있는지 검사한다.
  - 옵티마이저는 실행계획을 수립한다.
  - 스토리지 엔진에서 데이터를 가져온다. join, filter의 작업도 여기서 이뤄진다.
- DB 오브젝트 중 PK는 클러스터형 인덱스다. PK들끼리는 서로 가까워서 index 중에서도 더 빠르다.


## <span style="color:#802548">_서브쿼리의 종류_</span>
- 스칼라 서브쿼리
  - 결과값이 1행 1열의 구조다.
  - 보통 집계함수(max, min, avg, sum, count)와 많이 쓰인다.
  - SELECT 절에 있어야 한다.
```sql
SELECT 이름,
    (SELECT count(*) FROM 학생 ) AS count,
    FROM 학생2
```

- 인라인 뷰
  - 결과값이 1행 1열이다 이런 게 아니다.
  - FROM 절에 있어야 한다. table의 대용으로 쓰인다.
  - 내부적으로 메모리나 disk에 임시테이블을 만드는 것이다.
```sql
SELECT 학생2.학번, 학생2.이름
    FROM  (SELECT *
            FROM 학생
            WHERE 성별 = '남') AS 학생2;
```

- 중첩 서브쿼리
  - filtering을 위한 서브쿼리다.
  - WHERE절에 있어야 한다.
  - IN, NOT IN, EXISTS, NOT EXISTS, 비교연산자와 자주 활용된다.
```sql
SELECT * 
    FROM 학생
    WHERE 학번 = (SELECT MAX(학번) FROM 학생)
```

- 비상관 서브쿼리
  - outer query와 inner query가 서로 독립적이다.
  - 서브쿼리가 먼저 실행되고, 그 다음 메인쿼리가 실행된다.
```sql
SELECT *
    FROM 학생
    WHERE 학번 IN (SELECT 학번
                    FROM 학생
                    WHERE 성별 = '남')
```


- 상관 서브쿼리
  - outer query와 inner query가 서로 연관되어 있다.
  - main query가 먼저 실행되고, 서브쿼리가 실행된다. 그리고 다시 메인쿼리를 실행한 뒤 결과를 출력한다.
```
1. 학생 테이블의 학번 결과를 서브쿼리로 전달
2. 지도교수 테이블의 학번화 학생 테이블의 학번이 동일할 때만 서브쿼리 resultSet 도출
3. 서브쿼리의 resultSet인 학번 결과를 메인쿼리의 학번과 비교해 최종 결과 출력
```
```sql
SELECT * FROM 학생
    WHERE 학번 IN (SELECT 학번
                    FROM 지도교수
                    WHERE 성별 = 지도교수.학번 = 학생.학번)
```

## <span style="color:#802548">_scan_</span>
- table full scan
  - WHERE절에서 활용할 index가 없는 경우, index가 있어도 선택률이 높은 경우 활용된다.
  - sequential access, db file scattered read이다. 메모리 공간에서는 분산되어 놓이기 때문이다.
  - 물론 다 읽어야 해서 성능 면에서는 좋지 않다.

- 아래는 index scan의 종류다.
- index scan은 디스크 헤드를 랜덤하게 접근하여 띄엄띄엄 읽는다.
- sequential access보다 비효율적이지만, 읽는 데이터가 적어 성능은 좋다.
- random access이며, db file sequential read다. 메모리 공간에는 연속으로 놓이기 때문이다.
- index range scan
  - where에서 index를 사용한 field가 있지만, 검색값이 여러개 인 경우다.
  - 주로 BETWEEN, LIKE, <>와 같은 연산자를 할 때 많이 쓰인다.

- index full scan
  - where에서 index를 사용한 field가 없을 때 주로 쓰인다.
  - covering index의 방식이라고 생각하면 된다. SELECT에만 index를 거는 것이다.

- index unique scan
  - unique index로 where 조건문을 거는 것이다.
  - UK, PK, UI column인 경우에 해당한다.

- index loose scan
  - GROUP BY, MAX, MIN이 포함되면 작동한다.
  - 인덱스에서 필요한 부분만 골라 scan한다.


## <span style="color:#802548">_execution plan_</span>
- mysql 에서는 아래와 같이 실행계획을 볼 수 있다.
```sql
EXPLAIN query문
```

```
id      |select_type        |table                |type       |possible_keys      |key            |ref     |extra
1       |SIMPLE             |사원                 |range      |primary            |primary         |NULL   |Using Where
```

- id
  - id는 SQL문이 수행되는 차례다.
  - ID의 숫자가 작을수록 먼저 수행된 것이다.
  - ID의 숫자가 같으면 join이 이뤄진 것이다.
```sql
SELECT 사원.사원번호, 사원.이름, 사원.성, 급여.연봉,
        (SELECT MAX(부서번호) 
            FROM 부서사원_매핑 AS 매핑 WHERE 매핑.사원번호 = 사원.사원번호) AS 카운트
FROM 사원, 급여
WHERE 사원.사원번호 = 10001
AND 사원.사원번호 = 급여.사원번호; /**oracle join은 from의 앞이 driving table이다. */
```
```
id      |select_type         |table        |type      |possible_keys       |key                 |ref       |extra
1       |PRIMARY             |사원         |const      |primary            |primary             |const     |null
1       |PRIMARY             |급여         |ref        |primary            |primary             |const     |null
2       |DEPENDENT SUBQUERY  |사원         |null       |null               |null                |NULL      |Select tables optimized away
```

## <span style="color:#802548">_selecttype_</span>
- select_type
  - select문의 유형을 출력한다.
  - SIMPLE ---> 단순한 select문이다.
  - PRIMARY ---> 서브쿼리가 포함된 SQL문에서 메인쿼리 or UNION에서 첫번쨰 select
  - subquery ----> 독립적으로 실행가능한 subquery
  - dependent subquery ----> 독립적으로 실행불가능한 서브쿼리
  - derived ----> FROM 절에 작성된 subquery. inline view를 의미
  - union -----> UNION 및 UNION ALL에서 두번째 이후부터의 select문
  - union result ----> union all이 아닌 union으로 select문을 결합한다. 중복을 삭제해주는 과정이 추가된다.
  - dependent union ----> ??
  - uncachheable subquery ---> RAND(), UUID() 등의 함수를 사용해 조회하는 경우
  - materialized -----> IN 절에 subquery로 임시 테이블을 만드는 경우



- SIMPLE의 예시다.
```sql
SELECT * FROM 사원
```
```
	id	select_type	  table		type	possible_keys	key		ref	  Extra
	1	  SIMPLE	      사원		ALL		 null			    null  null   null
```

- subquery인데도 SIMPLE로 뜨는 경우도 있다.
- 아마 optimizer가 view merging을 한 것으로 생각된다.
```sql
SELECT 사원.사원번호, 사원.이름, 사원.성, 급여.연봉
    FROM 사원,
            (SELECT 사원번호, 연봉
                FROM 급여
                WHERE 연봉 > 80000) AS 급여
    WHERE 사원.사원번호 = 급여. 사원번호
    AND 사원.사원번호 BETWEEN 10001 AND 10010;
```
```
	id	select_type	table		  type	    possible_keys	  key	           ref	                    Extra
	1	  SIMPLE	    사원	    range	    PRIMARY	        PRIMARY	       null                     Using where
	1	  SIMPLE	    급여	    ref	      PRIMARY	        PRIMARY	       tuning.사원.사원번호	     Using where
```
- PRIMARY의 첫번쨰 예시다.
```sql
SELECT 사원.사원번호, 사원.이름, 사원.성, 급여.연봉, /** 사원과 급여가 메인쿼리*/
        (SELECT MAX(부서번호) 
            FROM 부서사원_매핑 AS 매핑 WHERE 매핑.사원번호 = 사원.사원번호) AS 카운트
    FROM 사원, 급여
```
```
	id	select_type	              table	    type	      possible_keys	  key	   	      ref	  	              Extra
	1	  PRIMARY	                  사원		  null        ALL		          nuil	        null	                null
	1	  PRIMARY	                  급여		  null        ALL	            null          null                  Using join buffer (hash join)
	2	  DEPENDENT SUBQUERY	      매핑		  ref	        PRIMARY	        PRIMARY	      tuning.사원.사원번호	 Using index
```

- PRIMARY의 두번쨰 예시다.
- union에서 처음에 나온 SELECT문이 PRIMARY고, 뒤를 이은 게 UNION이다.
```sql
SELECT 사원1.사원번호, 사원1.이름, 사원1.성
    FROM 사원 AS 사원1
WHERE 사원1.사원번호 = 100001

UNION ALL

SELECT 사원2.사원번호, 사원2.이름, 사원2.성
    FROM 사원 AS 사원2
WHERE 사원2.사원번호 = 100001
```
```
	id	select_type	table	  type	  possible_keys	     key	     ref	    		Extra
	1	  PRIMARY	    사원1	  const	  PRIMARY	           RIMARY	   const	  	  null
	2	  UNION	      사원2	  const	  PRIMARY	           PRIMARY	 const	      null
```

- subquery의 예시다.
```sql
SELECT (SELECT COUNT(*)
            FROM 부서사원_매핑 as 매핑
        ) AS 카운트,
        (SELECT MAX(연봉)
            FROM 급여
        ) AS 급여;
```

```
	id	select_type	table		  type	 possible_keys	key	          ref	      Extra
	1	  PRIMARY			null      null   null           null          null  	  No tables used
	3	  SUBQUERY	  급여	    ALL	   null           null         	null      null
	2	  SUBQUERY	  매핑      index	 null	          I_부서번호	   null		   Using index
```

- derived의 예시다.
```sql
SELECT 사원.사원번호, 급여.연봉
    FROM 사원,
        (SELECT 사원번호, MAX(연봉) as 연봉
            FROM 급여
            WHERE 사원번호 BETWEEN 10001 AND 20000 GROUP BY 사원번호) AS 급여
WHERE   사원.사원번호 = 급여.사원번호;
```

- id가 1인 것이 2개라면, join이 이뤄진 것으로 생각할 수 있다.
- dervied2는 id가 2번이면서 select_type이 derived인 것이다. 즉 사실상 급여 table이다.
- subquery로 가져온 것을 다시 쓰는 것이다.
```
	id	select_type	table		      type	  possible_keys	      key	       ref	          Extra
	1	  PRIMARY	    <derived2>		 ALL		null			          null       null      
	1	  PRIMARY	    사원		       eq_ref	PRIMARY	            PRIMARY	   급여.사원번호    Using index
	2	  DERIVED	    급여		       range	PRIMARY,I_사용여부	 PRIMARY	  null            Using where
```


- derived인데도 SIMPLE로 뜨는 경우도 있다.
- optimizer가 view merging을 한 것으로 생각된다.
```sql
SELECT 사원.사원번호, 사원.이름, 사원.성, 급여.연봉
    FROM 사원,
            (SELECT 사원번호, 연봉
                FROM 급여
                WHERE 연봉 > 80000) AS 급여
    WHERE 사원.사원번호 = 급여. 사원번호
    AND 사원.사원번호 BETWEEN 10001 AND 10010;
```

- union result의 예시다.
```sql
SELECT 사원_통합.*
  FROM (
        SELECT MAX(입사일자) AS 입사일자
          FROM 사원 AS 사원1
          WHERE 성별 = 'M'

          UNION

          SELECT MIN(입사일자) as 입사일자
          FROM 사원 as 사원2
          WHERE 성별 = 'M'
  ) AS 사원_통합;
```

- 아래 UNION RESULT에서 table이 union1이 아니라 union 2,3인 이유는 union1은 보통 UNION 이전의 select문을 의미하기 때문이다.
- 여기서는 FROM 절 이전의 SELECT가 되겠다.
- UNION 중복값 삭제는 temp table에서 이뤄지기 때문에 Extra가 Using temporary다.
```
	id	select_type	  table	    	type	possible_keys	  key	  	    ref	    Extra
	1	  PRIMARY	      <derived2>  ALL		null			      null        null    
	2	  DERIVED	      사원1		    ref	  I_성별_성	       I_성별_성   const	
	3	  UNION	        사원2		    ref	  I_성별_성	       I_성별_성   const	
	4	  UNION RESULT	<union2,3>  ALL	  null				    null        null    Using temporary
```

- union의 예시다.
- 위의 query에서 union을 union all로만 바꾸었다.
```sql
SELECT 사원_통합.*
  FROM (
        SELECT MAX(입사일자) AS 입사일자
          FROM 사원 AS 사원1
          WHERE 성별 = 'M'

          UNION ALL

          SELECT MIN(입사일자) as 입사일자
          FROM 사원 as 사원2
          WHERE 성별 = 'M'
  ) AS 사원_통합;
```

- 여기서는 FROM 절 이전의 SELECT가 되겠다.
- UNION ALL은 중복값이 없다는 가정 하에 합치는 것이라서 중복값을 제거하는 과정이 없다.
- 따라서 UNION RESULT가 빠진다. 
```
	id	select_type	  table	    	type	possible_keys	key	      	ref	  	Extra
	1	  PRIMARY	      <derived2>  ALL		null			    null        null     null
	2	  DERIVED	      사원1		    ref	  I_성별_성	     I_성별_성	 const	  null
	3	  UNION	        사원2		    ref	  I_성별_성	     I_성별_성   const    null	
```


- dependent union의 예시다.
```sql
SELECT 관리자.부서번호,
 (
        SELECT 사원1.이름
			FROM 사원 AS 사원1
          WHERE 성별 = 'F'
          AND 사원1.사원번호 = 관리자.사원번호

          UNION ALL

            SELECT 사원2.이름
			FROM 사원 AS 사원2
          WHERE 성별 = 'M'
          AND 사원2.사원번호 = 관리자.사원번호
  ) AS 이름
  FROM 부서관리자 AS 관리자;
  ```

```
id	select_type	          table	     	 type	    possible_keys	      key	    	  ref	                 	Extra
1	    PRIMARY	            관리자	     index		I_부서번호	         null   		 null                  Using index
2	    DEPENDENT SUBQUERY	사원1		     eq_ref	  PRIMARY,I_성별_성	  PRIMARY	    tuning.관리자.사원번호  Using where
3	    DEPENDENT UNION	    사원2	       eq_ref	  PRIMARY,I_성별_성	  PRIMARY	    tuning.관리자.사원번호  Using where
```

- materialized의 예시다.
```sql
SELECT *
  FROM 사원
  WHERE 사원번호 IN (SELECT 사원번호 FROM 급여 WHERE 시작일자 > '2020-01-01');
```

```
	id	select_type	  table	        	type	  possible_keys	        key	                  ref	          	Extra
	1	  SIMPLE	      사원		        ALL	    PRIMARY				         null                 null          	
	1	  SIMPLE	      <subquery2>		  eq_ref	<auto_distinct_key>	  <auto_distinct_key>	  사원.사원번호		
	2	  MATERIALIZED	급여		        index	  PRIMARY	              I_사용여부	           null         	 Using where; Using index
```

## <span style="color:#802548">_type_</span>
- type은 테이블의 데이터를 어떻게 찾을지에 관한 것이다.
  - system ---> 데이터가 없거나, 한개만 있는 경우
  - const ----> 조회되는 데이터가 단 1건인 경우
  - eq_ref ----> join 수행 시, unique index인 column을 사용해 비교하는 경우에, on조건절에 따른 row가 각 테이블에 한개씩만 존재할 때 그렇다. join 중에는 가장 성능이 좋다.
  - ref   -----> join 수행 시, unique index인 column을 사용해 비교하는 경우에, on조건절에 따른 row가 각 테이블에 여러개 존재할 때 그렇다.
  - ref   -----> index로 쓸만한 적절한 column이 존재할 떄.
  - ref_or_null ----> index로 쓸만한 적절한 column에 IS NULL 구문이 쓰일 때
  - range -----> 연속된 데이터 범위를 조회
  - index -----> index full scan
  - ALL  ------> table full scan

  <br/>
- eq-ref의 예시다.
```sql
SELECT 매핑.사원번호, 부서.부서번호, 부서.부서명
from 부서사원_매핑 as 매핑, 부서
Where 매핑.부서번호 = 부서.부서번호
AND 매핑.사원번호 BETWEEN 100001 AND 100010;
```
- 사원번호, 부서번호가 on조건절에 쓰인다.
- 사원번호, 부서번호가 모두 겹치는 row가 없다. 따라서 조건별로 1개씩만 있는 셈이다.
```
사원번호  부서번호
100001	d005
100001	d008
100002	d008
100003	d005
100004	d007
100005	d007
100006	d004
100007	d007
100008	d005
100009	d007
100010	d004
100010	d009
```
```
	id	select_type	table		type	  possible_keys	          key	    	 ref	        	 Extra
	1	    SIMPLE	  매핑	  range	  PRIMARY,I_부서번호	     PRIMARY    null        	  Using where; Using index
	1	    SIMPLE	  부서    eq_ref	PRIMARY	                PRIMARY	   매핑.부서번호
```

- ref의 예시다.
- join을 할 때 실행된다.
```sql
SELECT 사원.사원번호, 직급.직급명
FROM 사원, 직급
WHERE 사원.사원번호 = 직급.사원번호
AND 사원.사원번호 BETWEEN 10001 AND 10010;
```
- 사원.사원번호만이 on 조건절에 쓰인다.
- 그런데 아래와 같이 두개의 row가 나온다.
```
사원번호 직급명
10005	Senior Staff
10005	Staff
```
```
	id	select_type	table		type	possible_keys	  key	    	  ref	        	  Extra
	1	  SIMPLE	    사원	  range	PRIMARY	        PRIMARY	    null             Using where; Using index
	1	  SIMPLE	    직급	  ref	  PRIMARY	        PRIMARY	    사원.사원번호     Using index
```

- ref의 다른 예시다.
- index column과 비교 연산자로 filtering할 때 쓰인다.
```sql
SELECT * 
FROM 사원
WHERE 입사일자 = '1985-11-21'
```
```
	id	select_type	table		type	possible_keys	 key	        ref	  	    Extra
	1	  SIMPLE	    사원	   ref	 I_입사일자	    I_입사일자	  const
```


- ref_or_null의 예시다.
- null은 가장 앞쪽에 정렬된다. 검색할 null 데이터 양이 적다면 이득이다.
```sql
SELECT *
FROM 사원출입기록
WHERE 출입문 IS NULL
OR 출입문 = 'A';
```
```
	id	select_type	table	       type	          possible_keys	  key	      ref	   Extra
	1	    SIMPLE	사원출입기록	  ref_or_null	   I_출입문	        I_출입문  const	 Using index condition
```

- range의 에시다.
```sql
SELECT *
FROM 사원
WHERE 사원번호 BETWEEN 10001 AND 100000;
```

```
	id	select_type	table	type	possible_keys	key	    	ref	        Extra
	1	    SIMPLE	  사원  range	PRIMARY	      PRIMARY	  null        Using where
```


## <span style="color:#802548">_keyLen_</span>
- 인덱스를 사용할 때 사용된 바이트의 수
  - UTF - 8 기준 int는 단위 당 4바이트, VARCHAR는 단위당 3바이트다.

```sql
SELECT 사원번호
FROM 직급
WHERE 직급명 = 'Manager'
```
- 직급 table은 사원번호, 직급명, 시작일자를 묶어 PK로 만든 자연키다.
- 여기서 사원번호는 int고, 직급명은 varchar(50)이다.
- 위의 sql은 사원번호와 직급명만 사용되었다. 사원번호는 4byte고, 직급명은 (50 + 1) x3이므로 159로 과 동일하다.
- 또한 PK를 사용한 조회도 아니었으므로 type이 index로 나온다. 효율적인 ref가 아니라 index full scan을 할수밖에 없었다.
```
	id	select_type	table	  type	  possible_keys	 key	    ref	 Extra
	1	      SIMPLE	직급	  index	   PRIMARY       PRIMARY	null  Using where; Using index
```


- 만약 직급명에 index를 만들면 아래와 같이 실행계획이 바뀐다.
- type이 index가 아니라 ref가 된다. index를 효율적으로 scan할 수 있게 된 것이다.
```
	id	select_type	table		type	possible_keys	            key	            ref	  	Extra
	1	  SIMPLE	    직급	  ref	  PRIMARY,idx_직급_직급명	   idx_직급_직급명  const   Using index
```

## <span style="color:#802548">_ref_</span>
- 조건절에 쓰이는 indexing 처리가 된 column이거나 상수값이다.
  - WHERE일수도, ON일수도 있다.
  - driven table이 join을 수행할 때, on 조건절에 쓰인 field를 의미한다.
  - const인 경우도 있다.
```sql
SELECT 사원.사원번호, 직급.직급명
FROM 사원, 직급
WHERE 사원.사원번호 = 직급.사원번호
AND 사원.사원번호 BETWEEN 10001 AND 10100;
```

- driving table인 사원의 경우, ref가 null이다. index를 사용해 특정 범위의 데이터를 모두 가져온다.
- driven table인 직급의 경우, on절에 쓰인 field를 기준으로 데이터를 검색한다. 따라서 사원번호가 ref에 쓰인다.
```
	id	select_type	table	  type	  possible_keys	  key	     ref	        	 Extra
	1	    SIMPLE	  사원	  range	  PRIMARY	        PRIMARY	 null            Using where; Using index
	1	    SIMPLE	  직급	  ref	    PRIMARY	        PRIMARY	 사원.사원번호	  Using index
```

- const인 경우를 보자.
- 특정한 문자명을 상수로 제공했기 때문이다.
```sql
SELECT 사원번호
FROM 직급
WHERE 직급명 = 'Manager'
```
```
	id	select_type	table		type	possible_keys	key	        ref	  	Extra
	1	  SIMPLE	    사원		ref	  I_입사일자	   I_입사일자   const	
```

- WHERE 조건문을 range로 바꾸면 ref는 null이 된다.
```sql
select 사원번호
from 급여
WHERE 사원번호 >=10010;
```

```
	id	select_type	table	  type	possible_keys	      key	          ref	    Extra
	1	  SIMPLE	    급여	  range	PRIMARY,I_사용여부	 I_사용여부	    null  	Using where; Using index
```

- range가 아니라 명확한 조건을 =으로 넣으면 다시 ref가 나온다.
```sql
select 사원번호
from 급여
WHERE 사원번호 =10010;
```

```
	id	select_type	table	  type	possible_keys	     key	      ref	    Extra
	1	  SIMPLE	    급여	  ref	  PRIMARY,I_사용여부	PRIMARY	   const   Using index
```


## <span style="color:#802548">_extra_</span>
- Distinct ----> distinct나 union
- Using where ---> where조건절 존재
- Using Temporary ---> 중간 결과 저장을 위해 임시 테이블 생성. 
  - 저장, 정렬 행위 필요할 때.
  - 보통 DiSTINCT, GROUP BY, ORDER BY
  - TEMP 탈락 시 성능 저하
- Using index ---> index only scan. covering index
- Using filesort ----> 정렬이 필요한 데이터를 메모리에 올리고 정렬 작업 수행
  - index를 사용하지 못할 때 필요. index가 있다면 이미 데이터가 정렬됨
- Using join buffer ---> driving table data에 먼저 접근한 결과를 join buffer에 담음
  - 조인 버퍼와 드리븐 테이블 간에 일치하는 조인 key를 찾음
- Using union----> OR 구문으로 작성된 경우
- Using intersect---> AND 구문으로 작성된 경우
- Using sort_union ---> OR 구문이 동등조건이 아닐 때
- Using index condition ----> Using where에서 optimizer가 더 최적화
  - mysql이 아닌 storage 엔진에서 filtering 실행해 mysql 엔진으로 오는 데이터양을 줄임



- using index의 예시다.
- 직급 table은 PK가 (사원번호, 직급명, 시작일자)로 구성되어 있다.
- 시작일자가 없어도 Using index로 충분하다.
```sql
select 직급명
from 직급
where 사원번호 = 100000;
```
```
	id	select_type	table		type	  possible_keys	             key	      	    ref	   Extra
	1	  SIMPLE	    직급	  ref	     PRIMARY,idx_직급_직급명	  PRIMARY	         const	Using index
```

- 급여 table은 PK가 (사원번호, 시작일자)로 되어있다.
- 시작일자가 없어도 Using index로 충분하다. covering index가 된다는 의미다.
- PK가 아니라 I_사용여부로 조회하기 때문이다.
```sql
select 사원번호
from 급여;
```

```
	id	select_type	table		type	  possible_keys	  key	      	    ref	    Extra
	1	    SIMPLE	  급여	  index	  null            I_사용여부       null	   Using index
```


## <span style="color:#802548">_sql 성능 튜닝_</span>
## <span style="color:#802548">_where에 함수X_</span>
- 사원번호가 1100으로 시작하면서 사원번호가 5자리인 사람을 출력해봅시다.

```sql
SELECT * 
FROM 사원
WHERE SUBSTRING(사원번호,1,4) = 1100
AND LENGTH(사원번호) = 5
```
```
	id	select_type	table		type	possible_keys	key		ref         Extra
	1	  SIMPLE	    사원		ALL				                            Using where
```
- WHERE절에 함수를 쓰지 않습니다.
- 함수를 쓰면 index를 타지 않습니다. 그 결과 type이 range에서 ALL이 되었습니다.
- 아래와 같이 바꿔줍니다.
```sql
SELECT *
FROM 사원
WHERE 사원번호 BETWEEN 11000 AND 11009
```
```
	id	select_type	  table		type	    possible_keys	    key		      ref	  Extra
	1	  SIMPLE	      사원		range	    PRIMARY	          PRIMARY	    null   Using where
```

## <span style="color:#802548">_IFNULL이 필요한지 고민_</span>
- 성별이 null이면 No Data라고 출력해봅시다.
```sql
SELECT IFNULL(성별, 'NO DATA') AS 성별, COUNT(1) 건수
FROM 사원
GROUP BY IFNULL(성별, 'NO DATA')
```
```
	id	select_type	table		type	  possible_keys	  key		      ref	    Extra
	1	  SIMPLE	    사원		index	  I_성별_성	       I_성별_성	  null    Using index; Using temporary;
```


- 성별 column은 not null 컬럼입니다.
- IFNULL을 쓰면 해당 처리를 위한 temp table이 생성됩니다. 그 결과 Using temporary가 추가되었습니다.
```sql
SELECT 성별, COUNT(1) 건수
FROM 사원
GROUP BY 성별
```
```
	id	select_type	  table		type	  possible_keys	    key		      ref	  Extra
	1	  SIMPLE	      사원		index	  I_성별_성	        I_성별_성	   null	  Using index
```

## <span style="color:#802548">_묵시적형변환을 없애라_</span>
- 유효한 급여 정보만 조회해봅시다.
```sql
SELECT COUNT(1)
FROM 급여
WHERE 사용여부 = 1
```
```
	id	select_type	table		type	possible_keys	  key		    ref	    	Extra
	1	  SIMPLE	    급여		index	I_사용여부	     I_사용여부	null		  Using where; Using index
```

- 사용여부는 varchar입니다.
- 따라서 그냥 1로 비교하면 '1'로 바꾸기 위한 묵시적 형변환이 일어나게 됩니다. 
- 형변환 함수를 쓰면 적절한 index를 타지 않습니다. 그 결과 type이 ref가 아닌 index가 되었습니다.
```sql
SELECT COUNT(1)
FROM 급여
WHERE 사용여부 = '1'
```
```
	id	select_type	table		type	possible_keys	   key		      ref	    Extra
	1	  SIMPLE	    급여		ref	  I_사용여부	      I_사용여부	  const		Using index
```

## <span style="color:#802548">_PK가 포함된 row는 DISTINCT X_</span>
- 부서 관리자의 사원번호와 이름, 성, 부서번호 데이터를 중복 제거하여 조회해봅시다.
```sql
SELECT DISTINCT 사원.사원번호, 사원.이름, 사원.성, 부서관리자.부서번호
FROM 사원
JOIN 부서관리자
ON (사원.사원번호 = 부서관리자.사원번호)
```
```
	id	select_type	  table		      type	    possible_keys	      key		       ref	                      Extra
	1	  SIMPLE	      부서관리자		 index	   PRIMARY	           I_부서번호	   null                     	Using index; Using temporary
	1	  SIMPLE	      사원		      eq_ref	  PRIMARY	            PRIMARY	      tuning.부서관리자.사원번호	
```

- 사원번호는 PK입니다. 따라서 row들은 중복이 있을 수가 없습니다.
- 그럼에도 DISTINCT를 사용하는 바람에 정렬- 삭제 작업을 위한 temp table이 추가됐습니다.
- 실행계획에서 Using temporary가 추가되었습니다.
```sql
SELECT 사원.사원번호, 사원.이름, 사원.성, 부서관리자.부서번호
FROM 사원
JOIN 부서관리자
ON (사원.사원번호 = 부서관리자.사원번호)
```


## <span style="color:#802548">_겹치지 않는다면 UNION ALL_</span>
- 성별이 남자이며 성이 Baba인 경우, 성이 Baba이며 성별이 여자인 경우를 조회해봅시다.
```sql
SELECT 'M' AS 성별, 사원번호
FROM 사원
WHERE 성별 = 'M'
AND 성 = 'Baba'

UNION

SELECT 'F', 사원번호
FROM 사원
WHERE 성별 = 'F'
AND 성 = 'Baba'
```
```
	id	select_type	      table		      type	possible_keys	    key		       ref	  Extra
	1	  PRIMARY	          사원		       ref	I_성별_성	        I_성별_성	    const	  Using index
	2	  UNION	            사원		       ref	I_성별_성	        I_성별_성	    const 	Using index
	3	  UNION RESULT	    <union1,2>  	 ALL							                           Using temporary
```

- UNION은 DISTINCT와 마찬가지입니다.
- 정렬 후 중복 삭제를 위해 temp table을 만듭니다.
- 그래서 UNION RESULT select_type이 추가되었습니다.
- 그러나 성별이 남자이면서 여자인 경우는 없습니다. 중복제거가 무의미한 셈입니다.
- 아래와 같이 단순히 합치는 걸로 바꿔주면 UNION RESULT는 사라집니다.
```sql
SELECT 'M' AS 성별, 사원번호
FROM 사원
WHERE 성별 = 'M'
AND 성 = 'Baba'

UNION ALL

SELECT 'F', 사원번호
FROM 사원
WHERE 성별 = 'F'
AND 성 = 'Baba'
```
```
	id	select_type	table		type	  possible_keys	    key		       ref		   Extra
	1	  PRIMARY	    사원		ref	    I_성별_성	         I_성별_성		const	    Using index
	2	  UNION	      사원		ref	    I_성별_성	         I_성별_성		const     Using index
```

## <span style="color:#802548">_index 순서대로 GROUP BY해라_</span>
- 성과 성별로 묶어서 조회해보자.
```sql
SELECT 성, 성별, COUNT(1) AS 카운트
FROM 사원
GROUP BY 성, 성별
```

```
	id	select_type	  table		type	      possible_keys	    key		          ref	   Extra
	1	  SIMPLE	      사원		index	      I_성별_성	         I_성별_성	     null   Using index; Using temporary
```

- index를 활용하면 보통 temp table이 필요없습니다. 그런데도 Using temporary가 있습니다.
- 그 이유는 I_성별_성 index가 성별, 성 순으로 정렬되었기 때문입니다.
- show index from 사원으로 보면 알 수 있습니다.
- 즉 group by를 할 때 성먼저하고 성별이 아니라, 성별 먼저 하고 성이어야 합니다.
- 그렇지 않으면 다시 임시테이블을 생성해 group by에 쓰인 순서에 맞게 정렬하는 과정이 필요한 것입니다.

```
	Table	Non_unique	Key_name	Seq_in_index	Column_name	Collation	Cardinality	Sub_part	Packed	Null	Index_type	Comment	Index_comment	Visible	Expression
	사원	0	PRIMARY	1	사원번호	A	299379				BTREE			YES	
	사원	1	I_입사일자	1	입사일자	A	4645				BTREE			YES	
	사원	1	I_성별_성	1	성별	A	1				BTREE			YES	
	사원	1	I_성별_성	2	성	A	3154				BTREE			YES	
```

- 아래와 같이 바꿔줍니다.
- using temp가 사라졌습니다.
```sql
SELECT 성, 성별, COUNT(1) AS 카운트
FROM 사원
GROUP BY 성별, 성
```
```
	id	select_type	  table		type	  possible_keys	    key		         ref	  Extra
	1	  SIMPLE	      사원		index	  I_성별_성	         I_성별_성	    null    Using index
```

## <span style="color:#802548">_where 조건문은 access와 filter로 나뉜다_</span>
- 입사일자와 사원번호를 기준으로 filtering을 해 조회해보자.
- 아래와 같이 두개를 쓸 수 있다.
```sql
SELECT 사원번호
FROM 사원
WHERE 입사일자 LIKE '1989%'
AND 사원번호 > 100000;
```
```sql
SELECT 사원번호
FROM 사원
where 사원번호 > 100000;
and 입사일자 LIKE '1989%'
```

- 두 개의 조건 중 어떤 걸 앞으로 놓는게 좋을까?
- index를 쓰는 column 혹은 more selective(row가 작은 것)이 좋다.
- 그를 알기 위해서는 아래와 같이 select나 show가 필요하다.
```sql
show index from 사원;
```
- cardinality가 높은 게 좋다. 
- 그럼 more selective하기 때문이다. 그만큼 unique하다는 것이다. 사실 PK라서 높은 것이다.
- 다만 여기선 =가 아니라 >로 비교하기 때문에 index range scan이 되어 이점이 깎인다.
```
	Table	Non_unique	Key_name	  Seq_in_index	Column_name	 Collation	Cardinality	Sub_part	Packed	Null	Index_type	Comment	Index_comment	Visible	Expression
	사원	0	          PRIMARY	    1	            사원번호	    A	          299379				                      BTREE			                        YES	
	사원	1	          I_입사일자	 1	           입사일자	     A	         4645				                         BTREE			                       YES	
	사원	1	          I_성별_성	   1	           성별	        A	           1				                          BTREE			                         YES	
	사원	1	          I_성별_성	   2	           성	          A	           3154				                        BTREE			                         YES	
```

- 이젠 filtering을 얼마나 많이 해주냐를 봐야 한다.
- 입사일자 LIKE '1989%'가 더 많이 filtering해준다.
```sql
SELECT COUNT(1) FROM 사원
WHERE 입사일자 LIKE '1989%'; 	
```
```
COUNT(1)
	28394
```
```sql
SELECT COUNT(1) FROM 사원
WHERE 사원번호 > 100000;
```
```
COUNT(1)
	210024
```

- 이 경우에는 많이 filtering하는 조건이 더 좋을 것이다.
- 입사일자도 index가 있기 때문이다.
```sql
SELECT 사원번호
FROM 사원
WHERE 입사일자 LIKE '1989%'
AND 사원번호 > 100000;
```
```
	id	select_type	  table		type	  possible_keys	      key		         ref	  Extra
	1	  SIMPLE	      사원		range	  PRIMARY,I_입사일자	 PRIMARY	      null   Using where
```
- LIKE 후방일치보다는 아래와 같이 조건식을 써주는 게 좋다.
- 그럼 Using index가 추가된다. type은 range여도, table scan이 아니라 index를 이용해 가져오게 바뀐다.
- Using index라는 것은 테이블에 접근하지 않고 스토리지 엔진에서 I_입사일자(key) 라는 index에 있는 데이터만 가져옴을 의미한다.
- 그 뒤 mysql 엔진에서 사원번호에 대한 필터 조건으로 데이터를 추출한다.
```sql
SELECT 사원번호
FROM 사원
WHERE '1989-01-01' <= 입사일자 AND 입사일자 < '1990-01-01' /** storage engine access 조건 */
AND 사원번호 > 100000; /** mysql engine filtering 조건 */
```
```
	id	select_type	  table		type	  possible_keys	           key		         ref	  Extra
	1	  SIMPLE	      사원		range	   PRIMARY,I_입사일자	      I_입사일자	     null   Using where; Using index
```
- 아래가 chatGPT의 답변이다.
- where문에 쓰이는 조건은 access조건일 수도, filtering 조건일 수도 있다는 점이다.
- 주로 access조건에는 index가 있는 column이 쓰인다. explain에서 key로 되어있는 column이 곧 access condition으로 쓰인 것이다.
- 무조건 처음 오는 조건이 access condition이라고 optimizer가 해석하지는 않는다. 다만 그렇게 하는 게 읽기 좋은 쿼리문이다.
```
Access Conditions:

Access conditions are used to retrieve specific rows from tables by directly accessing indexes or table .
These conditions are typically based on columns that are indexed or partitioned, allowing the database optimizer to efficiently locate and retrieve relevant rows.
Examples of access conditions include conditions on primary keys, unique keys, or columns with indexes.
Access conditions are often placed first in the WHERE clause to maximize index usage and minimize the number of rows accessed.
Filtering Conditions:

Filtering conditions are used to further refine the result set by applying additional criteria to the rows retrieved through access conditions.
These conditions are applied after accessing the relevant rows and are used to filter out rows that do not meet certain criteria.
Filtering conditions are typically based on columns that are not indexed or are less selective, such as text columns or columns with complex data types.
Examples of filtering conditions include conditions based on non-indexed columns, functions, or expressions.
Filtering conditions are often applied after access conditions and may involve more intensive processing, such as scanning large portions of the table or performing complex calculations.
To determine which conditions belong to access and which belong to filtering, consider the following guidelines:

Access conditions typically involve conditions on indexed or partitioned columns that are used to efficiently locate specific rows in the table.
Filtering conditions typically involve additional criteria applied to the rows retrieved through access conditions to further refine the result set.
Additionally, you can analyze the execution plan generated by the database optimizer to understand how the conditions are being processed. The execution plan provides insights into how the query is being executed, including the access methods used and the order of condition evaluation. This can help identify opportunities for optimization and ensure that access and filtering conditions are being applied effectively.
```

## <span style="color:#802548">_선택률이 높다면 index를 쓰지 말라_</span>
- index가 있는 column이라고 해도, 선택률이 높으면 full scan만도 못하다.
```sql
SELECT *
FROM 사원출입기록
WHERE 출입문 = 'B'
```

- 아래와 같이 index를 없애는 hint를 주자.
```sql
SELECT * /*+ IGNORE_INDEX(I_출입문)*/ 
FROM 사원출입기록
WHERE 출입문 = 'B';
```
- oracle은 아래와 같다.
```sql
SELECT /*+ INDEX(사원출입기록 NO_INDEX(I_출입문)) */ *
FROM 사원출입기록
WHERE 출입문 = 'B';
```

- 다른 예시를 들어보자.
- 우변을 가공해서 사용하는 경우는 index를 불러오는 데 문제가 되진 않는다.
- 다만 입사일자라는 indexed column, 좌변을 함수로 가공하면 그 땐 index를 잃어버린다.
```sql
select 이름,성
FROM 사원
WHERE 입사일자 BETWEEN STR_TO_DATE('1994-01-01', '%Y-%m-%d') AND STR_TO_DATE('2000-12-31', '%Y-%m-%d');
```
- 물론 그래도 함수를 쓰면 overhead가 생기니 안 쓸수 있다면 안 쓰는게 좋다.
```sql
select 이름, 성
FROM 사원
WHERE 입사일자 BETWEEN '1994-01-01' AND '2000-12-31'
```

- 만약 indexed date type column을 썼는데 선택률이 높다면?
- 오히려 좌변에 함수를 써야 한다. full scan으로 바꿔서 성능을 올려야 한다.
```sql
select 이름, 성
FROM 사원
WHERE year(입사일자) BETWEEN '1994' AND '2000';
```

- 참고로 현재 내 optimizer는 알아서 ALL scan 때리게 최적화해주고 있다.



## <span style="color:#802548">_불필요한 서브쿼리 지우기-WHERE_</span>
- 사원번호가 450,000보다 크고 최대연봉이 100,000보다 큰 사원을 조회해보자.
```sql
SELECT 사원.사원번호, 사원.이름, 사원.성
FROM 사원
WHERE 사원번호 > 450000
AND (SELECT MAX(연봉)
      FROM 급여
      WHERE 사원번호 = 사원.사원번호
      ) > 100000;
```

```
	id	select_type	          table		type	    possible_keys	    key		      ref	                    Extra
	1	  PRIMARY	              사원		range	    PRIMARY	          PRIMARY	    null                     Using where
	2	  DEPENDENT SUBQUERY	  급여		ref	      PRIMARY	          PRIMARY	    tuning.사원.사원번호	    null
```

- 실행계획을 보면 상관서브쿼리가 사용되었음을 알 수 있다. 실행계획을 살펴보면 아래와 같다.
```
1. FROM 절의 main table인 사원 테이블에 접근. 
2. 접근 방식은 사원 테이블의 PK이며, range scan을 활용.
3. 다음으로 급여 테이블에 접근
4. 급여 테이블은 PK는 (사원번호, 시작일자)다. 시작일자가 아니라 사원번호로 조회하기 때문에 index를 타게 된다.
```

- 서브쿼리를 join으로 바꿔보자.
- join으로 바꾸는 이유는 안정적인 실행계획을 만들기 위해서다.
```sql
SELECT 사원.사원번호, 사원.이름, 사원.성
FROM 사원, 급여
WHERE 사원.사원번호 > 450000
AND 사원.사원번호 = 급여.사원번호
GROUP BY 사원.사원번호
HAVING MAX(급여.연봉) > 100000;
```


## <span style="color:#802548">_불필요한 서브쿼리 지우기-FROM_</span>
- 불필요한 서브쿼리는 from절에도 자주 나타난다.
- A출입문으로 출입한 사원이 총 몇명인지 구해보자.
```sql
SELECT COUNT(DISTINCT 사원.사원번호) AS 데이터건수
FROM 사원, (SELECT 사원번호
            FROM 사원출입기록 기록
            WHERE 출입문 = 'A'
            ) 기록
WHERE 사원.사원번호 = 기록.사원번호;
```

- 사실 위의 sql문은 view-merging되어 아래와 같이 join으로 수행된다.
```sql
SELECT COUNT(DISTINCT 기록.사원번호) AS 데이터건수
FROM 사원, 사원출입기록 기록
WHERE 사원.사원번호 = 기록.사원번호
AND 출입문 = 'A';
```

- 그걸 아래와 같이 WHERE EXISTS로 바꿔주자.
- DISTINCT는 필요가 없으니 지워준다. 사원번호는 어차피 PK라 DISTINCT가 필요가 없다.
```sql
SELECT COUNT(1) AS 데이터건수
FROM 사원
WHERE EXISTS (SELECT 1
              FROM 사원출입기록 기록
              WHERE 출입문 = 'A'
              AND 기록.사원번호 = 사원.사원번호);
```              

## <span style="color:#802548">_너무 많은 데이터를 가져오지 않게 indexed filtering을 거쳐라_</span>
- 사원번호가 10,001번부터 10,100번까지인 사원들의 평균연봉과 최고연봉, 최저연봉을 구해보자.
```sql
SELECT 사원.사원번호,
      급여.평균연봉,
      급여.최고연봉,
      급여.최저연봉
FROM 사원,
    (
      SELECT 사원번호,
            ROUND(AVG(연봉),0) 평균연봉,
            ROUND(MAX(연봉),0) 최고연봉,
            ROUND(MIN(연봉),0) 최저연봉
      FROM 급여
      GROUP BY 사원번호
      ) 급여
WHERE 사원.사원번호 = 급여.사원번호
AND 사원.사원번호 BETWEEN 10001 AND 10100;
```

- index full scan을 수행하는데, GROUP BY를 동반합니다.
- 조건절이 없는 GROUP BY로 너무 많은 데이터에 접근하지는 않는지 봐야 합니다.
```
	id	select_type	    table		      type	      possible_keys	         key		      ref	                Extra
	1	  PRIMARY	        사원		      range	       PRIMARY	             PRIMARY	    null                 Using where; Using index
	1	  PRIMARY	        <derived2>		ref	        <auto_key0>	           <auto_key0>	tuning.사원.사원번호	null
	2	  DERIVED	        급여		      index	       PRIMARY,I_사용여부	    PRIMARY      null                 null
```

- GROUP BY를 바꿔서 서브쿼리로 만들고, index를 타게 설정합니다.
- 조건절은 선택률이 매우낮을 떄, 즉 PK일 때 성능 향상 폭이 더 커진다.
- 3번을 scan한다고 해도 별로 부담되지도 않는 scan이다.
```sql
SELECT 사원.사원번호,
( SELECT ROUND(AVG(연봉),0)
 FROM 급여 AS 급여1
 WHERE 사원번호 = 사원.사원번호
 ) AS 평균연봉,
 ( SELECT ROUND(MAX(연봉),0)
 FROM 급여 AS 급여2
 WHERE 사원번호 = 사원.사원번호
 ) AS 최고연봉,
 ( SELECT ROUND(MIN(연봉),0)
 FROM 급여 AS 급여3
 WHERE 사원번호 = 사원.사원번호
 ) AS 최저연봉
 FROM 사원
 WHERE 사원.사원번호 BETWEEN 10001 AND 10100;
 ```


```
	id	select_type	            table		type	    possible_keys	                key		        ref	                    Extra
	1	  PRIMARY	                사원		range	    PRIMARY,I_입사일자,I_성별_성	  PRIMARY	      null               	    Using where; Using index
	4	  DEPENDENT SUBQUERY	    급여3		ref	      PRIMARY	                       PRIMARY	     tuning.사원.사원번호	    null
	3	  DEPENDENT SUBQUERY	    급여2		ref	      PRIMARY	                       PRIMARY	     tuning.사원.사원번호	    null
	2	  DEPENDENT SUBQUERY	    급여1		ref	      PRIMARY	                       PRIMARY	     tuning.사원.사원번호	    null
```


## <span style="color:#802548">_join을 깨고 서브쿼리를 써도 driving table은 작게 유지하자_</span>
- 사원번호가 10,001번부터 50,000번 사이에 해당하는 데이터를 사원번호 기준으로 묶어 연봉 합계 기준으로 내림차순한다. 그중 150번째 데이터부터 10건의 데이터만 가져온다.
```sql
SELECT 사원.사원번호, 사원.이름, 사원.성, 사원.입사일자
FROM 사원, 급여
WHERE 사원.사원번호 = 급여.사원번호
AND 사원.사원번호 BETWEEN 10001 AND 50000
GROUP BY 사원.사원번호
ORDER BY SUM(급여.연봉) DESC
LIMIT 150,10;
```
- 실행계획 자체는 큰 문제가 없다.
- 그러나 문제는 join을 할 때 driven table이 너무 크다는 것이다.
- 급여 table이 driven table인데, 2844047로 2백만 건이 넘는다.
- 그럼 304,000 x 2,844,047 만큼 반복이 일어난다.
```
	id	select_type	      table	  type	      possible_keys	                key		      ref	                    	Extra
	1	  SIMPLE	          사원		range	      PRIMARY,I_입사일자,I_성별_성	  PRIMARY		  null                   	  Using where; Using temporary; Using filesort
	1	  SIMPLE	          급여		ref	        PRIMARY	                       PRIMARY		 tuning.사원.사원번호	   
```

- 이럴 떄는 서브쿼리를 써서라도 join될 컬럼의 갯수를 줄여야 한다.
- FROM 테이블에 inline view를 만들었다. 이 inline view의 record는 10개밖에 안된다.
- 10개의 record가 driving table이다. 1개의 record가 사원 테이블에 반복해 접근하며 이 과정이 반복된다.
- 따라서 10 x 304,000 만큼 반복이 일어난다. 엄청나게 줄어든 것이다.
```sql
SELECT 사원.사원번호, 사원.이름, 사원.성, 사원.입사일자
  FROM (SELECT 사원번호
          FROM 급여
          WHERE 사원번호 BETWEEN 10001 AND 50000
          GROUP BY 사원번호
          LIMIT 150,10) 급여,
          사원
  WHERE 사원.사원번호 = 급여.사원번호;
```


## <span style="color:#802548">_Join이 반드시 필요한지 의심하라_</span>
- 사원 테이블에서 성별이 M이고, 사원번호가 300,000을 초과하는 사원번호를 출력해보자.
```sql
SELECT COUNT(사원번호) AS 카운트
FROM (
      SELECT 사원.사원번호, 관리자.부서번호
      FROM (
            SELECT *
            FROM 사원
            WHERE 사원번호 > 300000
      ) 사원
      LEFT JOIN 부서관리자 관리자
      ON 사원.사원번호 = 관리자.사원번호
) 서브쿼리;
```
- type이 range면 어쨌든 index range scan이다. 근데 ref는 null이라면 사원이 driving table임을 의미한다.
- ref는 join을 수행할 때 driven table에 어떤 column으로 해당 테이블에 access하는 지를 알려준다.
- driving table은 당연히 ref가 null일 수 밖에 없다.
- driven table은 type이 ref가 떠야 최적화의 발을 뗀 것이다. ref관련이 아니면 특별한 경우를 제외하면 join 조건절에 문제가 있는 것이다.
- type이 range이며 extra에 Using index가 있는 것은 index only scan이라는 뜻이다. 매우 좋은 index scan이다. 
- 하지만 join자체가 필요없다면 무의미하다.
```
	id	select_type	  table	    type	  possible_keys	    key		        ref	                Extra
	1	  SIMPLE	      사원		  range	  PRIMARY	          PRIMARY				null	               Using where; Using index
	1	  SIMPLE	      관리자		ref	    PRIMARY	          PRIMARY		    tuning.사원.사원번호  Using index
```
- FROM 절에서 JOIN을 하고 있다.
- 이건 뭔가 필요 없는 정보가 들어갔다는 신호일 수 있다.
- 아래와 똑같은 내용이니 바꿔주자.
```sql
SELECT COUNT(사원.사원번호)
    FROM (
          SELECT *
          FROM 사원
          WHERE 사원번호 > 300000
    ) 사원
    LEFT JOIN 부서관리자 관리자
    ON 사원.사원번호 = 관리자.사원번호;
```

- 이제 제대로 tuning을 시작해보자.
- join을 하게 되는 조건이 사원.사원번호 = 관리자.사원번호다.
- 저 조건은 무의미하다. 그러니 아래와 같이 일반 쿼리문으로 바꿔주자.
```sql
SELECT COUNT(사원번호) AS 카운트
FROM 사원
WHERE 성별 = 'M'
AND 사원번호 > 300000;
```
- where 조건이 성별과 PK로 두개다. 이같이 조건이 indexed column이 multiple일 때, optimizer가 skip scan을 실행하는 경우도 있다. 
- 그건 extra를 보고 알 수 있다. using index for skip scan이라 떠있다.
- 한 개의 index로는 커버될 수 없을 때, skip scan을 실행한다. 사원번호와 성별 모두 index 처리된 조건이라 그런 것이다.
```
index skip scan 조건
1.인덱스가 SELECT 리스트의 모든 부분을 커버하는 경우. 즉, 커버링 인덱스가 적용되는 경우

2.SELECT DISTINCT, SELECT ... GROUP BY 또는 단일 투플 SELECT 문인 경우

3.MIN/MAX 함수를 제외한 모든 집계 함수가 DISTINCT를 포함하는 경우

4.COUNT(*)가 사용되어선 안 됨

5.부분 키(subkey)의 카디널리티(고유 값의 개수)가 전체 인덱스의 카디널리티보다 100배 작은 경우
```
```
+----+-------------+-----------+------------+------+-------------------+-----------+---------+-------+------+----------+--------------------------------+
| id | select_type | table     | partitions | type | possible_keys     | key       | key_len | ref   | rows | filtered | Extra                          |
+----+-------------+-----------+------------+------+-------------------+-----------+---------+-------+------+----------+--------------------------------+
|  1 | SIMPLE      | 사원      | NULL       | range| PRIMARY,I_성별_성 | I_성별_성  | NULL    | NULL  | NULL | NULL     | Using where; Using index for skip scan |
+----+-------------+-----------+------------+------+-------------------+-----------+---------+-------+------+----------+--------------------------------+
```
- 그와 비슷하게 index_merge라는 type도 있는데, 여기는 각 indexed column의 조건을 통해 얻어온 결과값들이 겹치지 않을 때 자주 선택된다.
- 흔하게 보기는 어렵다. 보통 결과값들은 겹치기 때문이다.
```
+----+-------------+-----------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
| id | select_type | table     | partitions | type  | possible_keys | key     | key_len | ref   | rows | filtered | Extra                    |
+----+-------------+-----------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
|  1 | SIMPLE      | table1    | NULL       | index_merge   | index1,index2 | index1,index2 | NULL    | NULL |    NULL | Using intersect(index1,index2) |
+----+-------------+-----------+------------+-------+---------------+---------+---------+-------+------+----------+--------------------------+
```

- 다른 예시를 보자.
- 부서사원_매핑 테이블에도 해당 부서번호가 있는 경우에만, 부서 관리자가 소속된 부서번호를 조회해보자.
- 두 테이블이 join을 했다. 따라서 둘 모두 type이 ref일 수는 없다. 
- ref는 다른 테이블의 index에 대한 reference가 이미 있다는 전제하에 가능하기 때문에, 한 테이블은 반드시 index scan이 필요하다.
- 따라서 한 테이블은 무조건 type이 index거나 range여야 한다.
- 참고로 range도 index scan일 수밖에 없다. full scan은 index를 타지 않기 때문이다.
```sql
SELECT DISTINCT 매핑.부서번호
FROM 부서관리자 관리자, 
      부서사원_매핑 매핑
WHERE 관리자.부서번호 = 매핑.부서번호
ORDER BY 매핑.부서번호;
```
- type이 index인데 ref가 null인 것은 index full scan이라는 의미다.
- type이 index인데 ref가 null이면서 extra가 Using index니까 여기서는 index only scan이다. index scan 중에는 가장 효율적이다.
- index full scan은 어쨌든 table data에 접근이 필요하기 때문이다.
```
	id	select_type	    table		type	  possible_keys	     key		      ref 	                  Extra
	1	  SIMPLE	        관리자	index	  I_부서번호	        I_부서번호		null	                  Using index; Using temporary; Using filesort
	1	  SIMPLE	        매핑		ref	    PRIMARY,I_부서번호	I_부서번호		tuning.관리자.부서번호	 Using index
```
```
When MySQL accesses table2 using an index from table1 to perform the join, it often categorizes this access method as "ref" in the execution plan. 
```

- 데이터를 가져오는 것은 세 가지 방식이 있다.
```
table scan. 무조건 full scan이다.
index scan. index range scan이거나, index full scan이거나. index only scan이거나.
index lookup. driven table의 join에서만 쓰는 driving table의 index를 reference 삼아 가져오기 
```

- index scan은 세부적으로 몇 개로 나뉜다.
```
index range scan
index full scan
index only scan(=covering index)
Index Skip Scan(loose index scan)
Index Merge Scan
Index Condition Pushdown
```
```
Table Scan with Indexed Lookup: One table is scanned (using a full table scan or index scan), while the other table uses an index to perform lookups based on values from the scanned table. In this case, the scanned table would have a type of "ALL" or "index" in the execution plan, while the other table could have a type of "ref."

Indexed Lookup with Indexed Lookup: Both tables use indexes to perform lookups based on the join condition. This can happen when both tables have suitable indexes, and MySQL can perform indexed lookups efficiently on both sides of the join. In this case, both tables could have a type of "ref" in the execution plan.

Table Scan with Table Scan: In some cases, MySQL might resort to full table scans for both tables if no suitable indexes exist or if the optimizer determines that scanning the entire table is more efficient than using available indexes. In this scenario, both tables could have a type of "ALL" in the execution plan.
```

- FROM 절에서 부서사원_매핑 table의 데이터를 가져올 때 부서번호 데이터를 미리 중복 제거한다. 이제 가벼워졌습니다.
- 그리고 조건문도 더 가볍게 데이터의 존재여부만 판단합니다. WHERE EXISTS를 씁니다.
```sql
SELECT 매핑.부서번호
FROM ( SELECT DISTINCT 부서번호
        FROM 부서사원_매핑 매핑
        ) 매핑
WHERE EXISTS ( SELECT 1
                FROM 부서관리자 관리자
                WHERE 부서번호 = 매핑.부서번호)
ORDER BY 매핑.부서번호;
```
```
| id | select_type | table      | partitions | type  | possible_keys         | key          | key_len | ref                    | rows | filtered | Extra                                                  |
|----|-------------|------------|------------|-------|-----------------------|--------------|---------|------------------------|------|----------|--------------------------------------------------------|
| 1  | PRIMARY     | 관리자     |            | index | I_부서번호             | I_부서번호     | 12     |                        | 24   | 37.50    | Using index; Using temporary; Using filesort; LooseScan |
| 1  | PRIMARY     | <derived2> |            | ref   | <auto_key0>           | <auto_key0>  | 12      | tuning.관리자.부서번호  | 2    | 100.00   | Using index                                            |
| 2  | DERIVED     | 매핑       |            | range | PRIMARY,I_부서번호     | I_부서번호     | 12     |                        | 9    | 100.00   | Using index for group-by                               |
```



## <span style="color:#802548">_좋은 index를 써야 한다_</span>
- index 없이는 작은 table도 조회해서는 안 된다.
```sql
SELECT *
FROM 사원
WHERE 이름 = 'Gerogi'
AND 성 = 'Wielonsky';
```
- table full scan을 때리고 있다. 1건의 데이터를 가져오려고 table을 full scan하는 건 비효율적이다.
- 이걸 index scan으로 바꿔주자. 성과 이름 모두에 index를 만들어야 하는데, 순서도 중요하다.
- count가 더 많은 column을 선두에 놓는 게 성능에 유리하기 떄문이다. 더 많이 필터링할 수 있어서다.
```
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows  | filtered | Extra         |
|----|-------------|-------|------------|------|---------------|-----|---------|-----|-------|----------|---------------|
| 1  | SIMPLE      | 사원  |            | ALL  |               |     |         |     | 299379| 1.00     | Using where   |

```

- 따라서 아래와 같이 갯수를 구한다.
```sql
SELECT COUNT(DISTINCT(이름)) 이름_개수,
        COUNT(DISTINCT(성)) 성_개수,
        COUNT(1) 전체
  FROM 사원;
```
- 성_개수가 더 많으므로 데이터 범위를 더 축소할 수 있는 건 성이다. 따라서 성을 앞에 두고 index를 만든다.
- DISTINCT()로 보는 이유는 index는 null 및 겹치는 row를 제외하기 위해서다. 제외하는 이유는 High Cardinality Columns을 알기 위해서다.
- 그냥 COUNT(column)으로 하면 null을 제외한 전체 row가 항상 계산돼서 나온다.
```
	이름_개수	성_개수	전체
	1275	1637	300024

  ALTER TABLE 사원 ADD INDEX I_사원_성_이름 (성, 이름);
```

- 조건문이 있다면, 선택률을 보는 게 좋다. 선택률이 낮다면 index를 걸어주자.
- multiple 조건문이 있다면 where 조건문에서 일부 column만 index를 주면 안 된다.
```sql
SELECT * 
FROM 사원
WHERE 이름 = 'Matt'
OR 입사일자 = '1987-03-31'
```
- 입사일자에만 index가 걸려있어서 index scan이 아닌 ALL scan을 타고 있다.
```
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows   | filtered | Extra         |
|----|-------------|-------|------------|------|---------------|-----|---------|-----|--------|----------|---------------|
| 1  | SIMPLE      | 사원  |            | ALL  | I_입사일자    |     |         |     | 299379 | 10.02    | Using where   |

```
- 그럼 이름에 index를 주어도 되는지부터 파악해보자.
- 전체 row는 300024다.
```sql
  SELECT COUNT(1) FROM 사원;
```

- 이름에 따른 row는 233이므로 233/300024하면 선택률이 1%도 안된다. index를 주기 적합하다.
```sql
SELECT COUNT(1) FROM 사원 where 이름 = 'Matt';
```
```
	COUNT(1)
	233
```

- 물론 이름이 Matt만 있는 것은 아니기에, GROUP BY로 count를 해서 다양한 경우의 선택률을 보는 게 좋다.
- 물론 자주 쓰는 것만 거의 쓴다면, 해당 경우만 보면 된다.
- 아래와 같이 count를 재니 전부 300이하다. 선택률이 1%가 채 안되니 index를 주기 적합한 field다.
```
SELECT 이름, COUNT(이름)
FROM 사원
GROUP BY 이름;
```

- 따라서 아래와 같이 index를 준다.
```sql
ALTER TABLE 사원 ADD INDEX I_이름(이름)
```
- 그럼 ALL SCAN이 아니라 INDEX_MERGE가 된다. 조건에 따른 row가 서로 겹치지 않는 것으로 보인다.
```
| id | select_type | table | partitions | type        | possible_keys                   | key                          | key_len | ref | rows | filtered | Extra                                   |
|----|-------------|-------|------------|-------------|---------------------------------|------------------------------|---------|-----|------|----------|-----------------------------------------|
| 1  | SIMPLE      | 사원  |            | index_merge | I_입사일자,idx_사원_이름          | idx_사원_이름,I_입사일자      | 44,3    |     | 344  | 100.00   | Using union(idx_사원_이름,I_입사일자); Using where |
```

- index 구성이 중요한 다른 예시를 살펴보자.
```sql
SELECT 사원번호, 이름, 성
FROM 사원
WHERE 성별 = 'M'
AND 성= 'Baba'
```
```
| id | select_type | table | partitions | type | possible_keys        | key                | key_len | ref         | rows | filtered | Extra   |
|----|-------------|-------|------------|------|----------------------|--------------------|---------|-------------|------|----------|---------|
|  1 | SIMPLE      | 사원  |            | ref  | idx_사원_성별_성    | idx_사원_성별_성 | 51      | const,const |  135 |   100.00 |         |
```

- 그런데 성 열의 데이터는 총 1637건인데, 성별 열은 단 2건이다.
- 데이터가 다양하지 않은 field를 선두 index로 두는 것은 바람직하지 않다.
```sql
SELECT COUNT(distinct 성) 성_개수,
      COUNT(distinct 성별) 성별_개수
  FROM 사원;
```

- 성이 먼저가 아니라 성별이 먼저오게 index를 조정해주자.
- 지금은 데이터가 많지 않아 차이가 크게 없지만, 대용량 데이터가 되면 이런 순서도 성능에 차이를 낸다.
```sql
ALTER TABLE DROP INDEX I_성별_성,
            ADD INDEX I_성별_성(성,성별);
```





## <span style="color:#802548">_update에도 index가 필요하기도 하고, 아니기도 하다._</span>
```sql
 UPDATE 사원출입기록
 SET 출입문 = 'X'
 WHERE 출입문 = 'B'
 ```
- update문은 수정할 데이터에 접근이 먼저 필요하다. 그럼 그 떄 index가 사용된다.
- 동시에 변경하는 범위에도 index가 포함되므로 index가 많으면 update에 느려진다.
```
| id | select_type | table     | partitions | type  | possible_keys | key     | key_len | ref | rows   | filtered | Extra         |
|----|-------------|-----------|------------|-------|---------------|---------|---------|-----|--------|----------|---------------|
| 1  | UPDATE      | 사원출입기록 |          | index | I_출입문       | PRIMARY | 8       |     | 658935 | 100.00   | Using where   |
```

- 과연 필요한 index만 들어있는지 검사해보자.
```sql
show index from 사원출입기록
```
- I_출입문 index가 실제로 사용되지 않는다면 삭제를 고려해봄직하다.
- 아니면 야간 배치 update 시, 해당 index만 잠깐 삭제하고 update를 때릴 수도 있다. update가 끝나면 복원해준다.
```
| Table      | Non_unique | Key_name | Seq_in_index | Column_name | Collation | Cardinality | Sub_part | Packed | Null | Index_type | Comment | Index_comment | Visible | Expression |
|------------|------------|----------|--------------|-------------|-----------|-------------|----------|--------|------|------------|---------|---------------|---------|------------|
| 사원출입기록 | 0          | PRIMARY  | 1            | 순번        | A         | 658935      |          |        | YES  | BTREE      |         |               | YES     |            |
| 사원출입기록 | 0          | PRIMARY  | 2            | 사원번호    | A         | 658935      |          |        | YES  | BTREE      |         |               | YES     |            |
| 사원출입기록 | 1          | I_지역    | 1            | 지역        | A         | 4           |          |        | YES  | BTREE      |         |               | YES     |            |
| 사원출입기록 | 1          | I_시간    | 1            | 입출입시간  | A         | 651510      |          |        | YES  | BTREE      |         |               | YES     |            |
| 사원출입기록 | 1          | I_출입문  | 1            | 출입문      | A         | 3           |          |        | YES  | BTREE      |         |               | YES     |            |
```
- 실행계획은 똑같다. 그러나 index가 줄어들어서 쿼리실행 시간은 13초에서 0.6초로 줄어든다.
```
| id | select_type | table     | partitions | type  | possible_keys | key     | key_len | ref | rows   | filtered | Extra         |
|----|-------------|-----------|------------|-------|---------------|---------|---------|-----|--------|----------|---------------|
| 1  | UPDATE      | 사원출입기록 |          | index | I_출입문       | PRIMARY | 8       |     | 658935 | 100.00   | Using where   |
```

## <span style="color:#802548">_대소문자 문제는 collation이 중요하다_</span>
- 부서테이블의 비고 열 값이 소문자 active일 때 데이터를 조회한다고 해보자.
- 총 4건이 출력된다.
```sql
SELECT 부서명, 비고
FROM 부서
WHERE 비고 = 'active'
  AND ASCII(SUBSTR(비고,1,1)) = 97
  AND ASCII(SUBSTR(비고,2,1)) = 99
```

- 만약 ascii코드 조건을 빼고 조회한다면, 7건이 출력된다.
```sql
SELECT 부서명, 비고
FROM 부서
WHERE 비고 = 'active'
```

- 이러한 차이가 어디서 오는 걸까? 바로 collation에서 온다. 
```sql
SELECT COLUMN_NAME, collation_name
FROM information_schema.COLUMNS
WHERE table_schema = 'tuning'
AND TABLE_NAME = '부서';
```
```
COLUMN_NAME	COLLATION_NAME
부서번호	utf8mb3_general_ci
부서명	utf8mb3_general_ci
비고	utf8mb3_general_ci
```

- utf8mb3_general_ci는 case insensitive하기 때문에 대소문자를 구분하지 않는다.
- 그럼 대소문자를 구분하는 collation으로 바꿔주면 된다.
- UTF8MB4_bin는 대소문자를 구분하며 이모지까지 지원된다. 따라서 엔간하면 저게 좋다.
```sql
ALTER TABLE 부서
CHANGE COLUMN 비고 비고 VARCHAR(40) NULL default null
COLLATE 'UTF8MB4_bin'
``` 

- collation을 바꾸면 아스키 조건문은 삭제해도 된다.
```sql
SELECT 부서명, 비고
FROM 부서
WHERE 비고 = 'active'
```


- 또 다른 예시를 살펴보자.
- 이번에는 화면에서 입력값을 받는다고 해보자.
- 좌변에도 LOWER를 붙인 이유는 이름 열이 utf8-bin이기 때문이다.
- utf8-bin은 대소문자를 구별하며, 이모지는 지원되지 않는다.
```sql
SELECT 이름,성, 성별, 생년월일
FROM 사원
WHERE LOWER(이름) = LOWER(#{name})
AND 입사일자 >= STR_TO_DATE(#{date}, '%Y-%m-%d')
```
- 실제 select 시 아래와 같은 결과를 보여준다.
- 첫글자가 대문자로 되어있다.
```
	이름
	Georgi
	Bezalel
	Parto
	Chirstian
	Kyoichi
	Anneke
	Tzvetan
	Saniya
	Sumant
	Duangkaew
```


- 화면에서 입력값은 대소문자가 어떻게 날라올지 모른다.
- 따라서 모두 소문자로 만들어버린다.
- 그를 위해선 기존 column도 소문자로 만드는 함수를 집어넣었다. 
- 좌변에 함수를 넣었으니 index를 타지 못한다.
- 이를 고치려면, 소문자_이름 column을 하나 만들어주는 게 좋다.
- table의 collation은 utf8-bin이 아닌 utf8-general-ci이기 때문에 해당 collation이 계승된다.
```sql
ALTER TABLE 사원 ADD COLUMN 소문자_이름 VARCHAR(14) NOT NULL AFTER 이름;
UPDATE 사원 SET 소문자_이름 = LOWER(이름);
ALTER TABLE 사원 ADD INDEX I_소문자이름(소문자_이름);
```

- 아래와 같이 바꿔줄 수 있을 거 같다.
- 그럼 좌변에서 함수가 빠지니 index 적용이 가능하다.
```sql
SELECT 이름,성, 성별, 생년월일
FROM 사원
WHERE 소문자_이름 = LOWER(#{name})
AND 입사일자 >= STR_TO_DATE(#{date}, '%Y-%m-%d')
```

- 그런데 만약 collation이 utf8-general-ci면 LOWER함수도 필요가 없다.
- 어차피 대소문자를 구별하지 않기 때문이다.
- index처리도 되었기 때문에 성능도 향상된다.
- Jaewon이든, JAewon이든, jaeWon이든 jaewoN이든 어떤 입력값이라도 상관없다. 대소문자를 구별하지 않기 때문이다.
- 소문자_이름의 collation이 utf8-general-ci라 가능하다. 이름은 utf8-bin이라 안 된다.
```sql
SELECT 이름,성, 성별, 생년월일
FROM 사원
WHERE 소문자_이름 = #{name}
AND 입사일자 >= STR_TO_DATE(#{date}, '%Y-%m-%d')
```

