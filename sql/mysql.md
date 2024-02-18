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
id      |select_type        |table      |partitions     |type       |possible_keys      |key            |key_len        |ref        |rows       |filterd        |extra
1       |SIMPLE             |사원       |NULL           |range      |primary            |primary        |4              |NULL       |20080      |100.00         |Using Where
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
id      |select_type        |table      |partitions     |type       |possible_keys       |key            |key_len        |ref         |rows        |filterd        |extra
1       |PRIMARY             |사원       |NULL           |const      |primary            |primary        |4              |const       |1           |100.00         |null
1       |PRIMARY             |급여       |NULL           |ref        |primary            |primary        |4              |const       |17          |100.00         |null
2       |DEPENDENT SUBQUERY  |사원       |NULL           |null       |null               |null           |null           |NULL        |20080       |100.00         |Select tables optimized away
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
	id	select_type	  table	partitions	type	possible_keys	key	key_len	ref	  rows	filtered	Extra
	1	SIMPLE	사원		ALL		 null			  null  null          null  null  null  299379	100.00	null
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
	id	select_type	table	partitions	  type	    possible_keys	  key	           key_len	ref	                    rows	filtered	    Extra
	1	  SIMPLE	    사원		null        range	    PRIMARY	        PRIMARY	        4	      null                    10	     100.00	    Using where
	1	  SIMPLE	    급여		null        ref	      PRIMARY	        PRIMARY	        4	      tuning.사원.사원번호	    9	       33.33	    Using where
```
- PRIMARY의 첫번쨰 예시다.
```sql
SELECT 사원.사원번호, 사원.이름, 사원.성, 급여.연봉, /** 사원과 급여가 메인쿼리*/
        (SELECT MAX(부서번호) 
            FROM 부서사원_매핑 AS 매핑 WHERE 매핑.사원번호 = 사원.사원번호) AS 카운트
    FROM 사원, 급여
```
```
	id	select_type	table	    partitions	type	possible_keys	key	  key_len	ref	 rows	        filtered	Extra
	1	PRIMARY	사원		ALL			nuil	      null	 null         null   null         299379	    100.00	    null
	1	PRIMARY	급여		ALL	    null        null   null         null   null         2838398	    100.00	    Using join buffer (hash join)
	2	DEPENDENT SUBQUERY	매핑		ref	PRIMARY	PRIMARY	4	tuning.사원.사원번호	1	100.00	 Using index
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
	id	select_type	table	partitions	type	possible_keys	    key	    key_len	ref	    rows	filtered	Extra
	1	  PRIMARY	    사원1		null       const	PRIMARY	        PRIMARY	 4	    const	  1	    100.00	    null
	2	  UNION	      사원2		null       const	PRIMARY	        PRIMARY	 4	    const	  1	    100.00	    null
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
	id	select_type	table	partitions	type	 possible_keys	key	          key_len	  ref	    rows	    filtered	  Extra
	1	  PRIMARY			null  null        null   null           null          null      null  	null        null      No tables used
	3	  SUBQUERY	  급여	null	      ALL	   null           null          null			null    2838398	    100.00	   null
	2	  SUBQUERY	  매핑  null		    index	 null	          I_부서번호	    12       null		 331143	     100.00	    Using index
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
	id	select_type	table	partitions	type	possible_keys	    key	        key_len	ref	        rows	filtered	Extra
	1	PRIMARY	<derived2>		        ALL					                                        184756	100.00	
	1	PRIMARY	사원		            eq_ref	PRIMARY	            PRIMARY	    4	    급여.사원번호	1	100.00	Using index
	2	DERIVED	급여		            range	PRIMARY,I_사용여부	 PRIMARY	4		             184756	100.00	Using where
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
	id	select_type	  table	    partitions	type	possible_keys	key	    key_len	ref	  rows	  filtered	Extra
	1	  PRIMARY	      <derived2>	null	    ALL		null			    null     2	  null  null    100.00	
	2	  DERIVED	      사원1		    null      ref	    I_성별_성	  I_성별_성	1	    const	149689	100.00	
	3	  UNION	        사원2		    null      ref	    I_성별_성	  I_성별_성	1	    const	149689	100.00	
	4	  UNION RESULT	<union2,3>	null	    ALL			null				 null   null    null   null             Using temporary
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
	id	select_type	  table	    partitions	type	possible_keys	key	    key_len	ref	  rows	  filtered	Extra
	1	  PRIMARY	      <derived2>	null	    ALL		null			    null     2	  null  null    100.00	
	2	  DERIVED	      사원1		    null      ref	    I_성별_성	  I_성별_성	1	    const	149689	100.00	
	3	  UNION	        사원2		    null      ref	    I_성별_성	  I_성별_성	1	    const	149689	100.00	
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
id	select_type	          table	      partitions	type	possible_keys	      key	  key_len	ref	            rows	filtered	Extra
1	    PRIMARY	            관리자	    null	      index		I_부서번호	        null  12		  null            24	  100.00	Using index
2	    DEPENDENT SUBQUERY	 사원1		  null        eq_ref	PRIMARY,I_성별_성	PRIMARY	4	tuning.관리자.사원번호	1	  50.00	Using where
3	    DEPENDENT UNION	    사원2	      null	      eq_ref	PRIMARY,I_성별_성	PRIMARY	4	tuning.관리자.사원번호	1	  50.00	Using where
```

- materialized의 예시다.
```sql
SELECT *
  FROM 사원
  WHERE 사원번호 IN (SELECT 사원번호 FROM 급여 WHERE 시작일자 > '2020-01-01');
```

```
	id	select_type	  table	        partitions	type	  possible_keys	        key	                  key_len	ref	          rows	filtered	Extra
	1	  SIMPLE	      사원		       null       ALL	    PRIMARY				         null                 null    null          299379	100.00	
	1	  SIMPLE	      <subquery2>		null        eq_ref	<auto_distinct_key>	  <auto_distinct_key>	  4	      사원.사원번호	  1	    100.00	
	2	  MATERIALIZED	급여		       null       index	  PRIMARY	              I_사용여부	4		        null   null           2838398	33.33	Using where; Using index
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
	id	select_type	table	partitions	type	  possible_keys	          key	    key_len	  ref	        rows	filtered	Extra
	1	    SIMPLE	  매핑	null	      range	  PRIMARY,I_부서번호	     PRIMARY	4		     null         12	100.00	Using where; Using index
	1	    SIMPLE	  부서  null		    eq_ref	PRIMARY	                PRIMARY	  12	    매핑.부서번호	1	100.00	
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
	id	select_type	table	partitions	type	possible_keys	key	    key_len	ref	        rows	filtered	Extra
	1	  SIMPLE	    사원	  null	    range	PRIMARY	      PRIMARY	  4		  null        10	100.00	      Using where; Using index
	1	  SIMPLE	    직급		null      ref	  PRIMARY	      PRIMARY	  4	    사원.사원번호	1	100.00	       Using index
```

- ref의 다른 예시다.
- index column과 비교 연산자로 filtering할 때 쓰인다.
```sql
SELECT * 
FROM 사원
WHERE 입사일자 = '1985-11-21'
```
```
	id	select_type	table	partitions	type	possible_keys	key	      key_len	ref	  rows	filtered	Extra
	1	    SIMPLE	  사원	null        ref	  I_입사일자	   I_입사일자	3	      const	119	100.00	
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
	id	select_type	table	      partitions	  type	      possible_keys	  key	      key_len	ref	rows	    filtered	Extra
	1	    SIMPLE	사원출입기록	  null        ref_or_null	I_출입문	        I_출입문	4	      const	329468	100.00	Using index condition
```

- range의 에시다.
```sql
SELECT *
FROM 사원
WHERE 사원번호 BETWEEN 10001 AND 100000;
```

```
	id	select_type	table	partitions	type	possible_keys	key	      key_len	ref	rows	filtered	Extra
	1	    SIMPLE	  사원	null        range	PRIMARY	      PRIMARY	    4		      149689	100.00	Using where
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
- 위의 sql은 사원번호와 직급명만 사용되었다. 사원번호는 4byte고, 직급명은 (50 + 1) x3이므로 159로 key_len과 동일하다.
- 또한 PK를 사용한 조회도 아니었으므로 type이 index로 나온다. 효율적인 ref가 아니라 index full scan을 할수밖에 없었다.
```
	id	select_type	table	partitions	type	possible_keys	key	        key_len	ref	  rows	filtered	Extra
	1	      SIMPLE	직급	null	      index	PRIMARY       PRIMARY	    159		  null  442426	10.00	  Using where; Using index
```


- 만약 직급명에 index를 만들면 아래와 같이 실행계획이 바뀐다.
- type이 index가 아니라 ref가 된다. index를 효율적으로 scan할 수 있게 된 것이다.
```
	id	select_type	table	partitions	type	possible_keys	          key	          key_len	  ref	rows	filtered	Extra
	1	  SIMPLE	    직급	null        ref	  PRIMARY,idx_직급_직급명	idx_직급_직급명	152	      const	24	100.00	  Using index
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
	id	select_type	table	partitions	type	possible_keys	key	    key_len	ref	        rows	      filtered	Extra
	1	    SIMPLE	  사원	null	      range	PRIMARY	      PRIMARY	 4		  null         100	        100.00	Using where; Using index
	1	    SIMPLE	  직급	null  	    ref	PRIMARY	        PRIMARY	 4	    사원.사원번호	1	            100.00	Using index
```

- const인 경우를 보자.
- 특정한 문자명을 상수로 제공했기 때문이다.
```sql
SELECT 사원번호
FROM 직급
WHERE 직급명 = 'Manager'
```
```
	id	select_type	table	partitions	type	possible_keys	key	      key_len	ref	  rows	filtered	Extra
	1	    SIMPLE	  사원	 null     	ref	  I_입사일자	  I_입사일자	3	      const	119	  100.00	
```

- WHERE 조건문을 range로 바꾸면 ref는 null이 된다.
```sql
select 사원번호
from 급여
WHERE 사원번호 >=10010;
```

```
	id	select_type	table	partitions	type	possible_keys	      key	      key_len	ref	    rows	filtered	Extra
	1	  SIMPLE	    급여	null	      range	PRIMARY,I_사용여부	I_사용여부	8		    null    946038	100.00	Using where; Using index for skip scan
```

- range가 아니라 명확한 조건을 =으로 넣으면 다시 ref가 나온다.
```sql
select 사원번호
from 급여
WHERE 사원번호 =10010;
```

```
	id	select_type	table	partitions	type	possible_keys	    key	    key_len	ref	rows	filtered	Extra
	1	  SIMPLE	    급여	null        ref	  PRIMARY,I_사용여부	PRIMARY	4	    const	6	    100.00	Using index
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
	id	select_type	table	partitions	type	possible_keys	             key	      key_len	    ref	rows	filtered	Extra
	1	  SIMPLE	    직급	null	      ref	   PRIMARY,idx_직급_직급명	  PRIMARY	    4	         const	1	  100.00	Using index
```

- 급여 table은 PK가 (사원번호, 시작일자)로 되어있다.
- 시작일자가 없어도 Using index로 충분하다. covering index가 된다는 의미다.
```sql
select 사원번호
from 급여;
```

```
	id	select_type	table	partitions	type	possible_keys	  key	      key_len	ref	    rows	filtered	Extra
	1	    SIMPLE	  급여	null      	index	I_사용여부        null	   4		   null    2838398	100.00	Using index
```


## <span style="color:#802548">_sql 성능 튜닝_</span>
### <span style="color:#802548">_where에 함수X_</span>
- 사원번호가 1100으로 시작하면서 사원번호가 5자리인 사람을 출력해봅시다.

```sql
SELECT * 
FROM 사원
WHERE SUBSTRING(사원번호,1,4) = 1100
AND LENGTH(사원번호) = 5
```
```
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE	사원		ALL					299379	100.00	Using where
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
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE	사원		range	PRIMARY	PRIMARY	4		10	100.00	Using where
```

### <span style="color:#802548">_IFNULL이 필요한지 고민_</span>
- 성별이 null이면 No Data라고 출력해봅시다.
```sql
SELECT IFNULL(성별, 'NO DATA') AS 성별, COUNT(1) 건수
FROM 사원
GROUP BY IFNULL(성별, 'NO DATA')
```
```
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE	사원		index	I_성별_성	I_성별_성	51		299379	100.00	Using index; 
```


- 성별 column은 not null 컬럼입니다.
- IFNULL을 쓰면 해당 처리를 위한 temp table이 생성됩니다. 그 결과 Using temporary가 추가되었습니다.
```sql
SELECT 성별, COUNT(1) 건수
FROM 사원
GROUP BY 성별
```
```
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE	사원		index	I_성별_성	I_성별_성	51		299379	100.00	Using index
```

### <span style="color:#802548">_묵시적형변환을 없애라_</span>
- 유효한 급여 정보만 조회해봅시다.
```sql
SELECT COUNT(1)
FROM 급여
WHERE 사용여부 = 1
```
```
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE	급여		index	I_사용여부	I_사용여부	4		2838398	10.00	Using where; Using index
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
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE	급여		ref	I_사용여부	I_사용여부	4	const	82824	100.00	Using index
```

### <span style="color:#802548">_PK가 포함된 row는 DISTINCT X_</span>
- 부서 관리자의 사원번호와 이름, 성, 부서번호 데이터를 중복 제거하여 조회해봅시다.
```sql
SELECT DISTINCT 사원.사원번호, 사원.이름, 사원.성, 부서관리자.부서번호
FROM 사원
JOIN 부서관리자
ON (사원.사원번호 = 부서관리자.사원번호)
```
```
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE	부서관리자		index	PRIMARY	I_부서번호	12		24	100.00	Using index; Using temporary
	1	SIMPLE	사원		eq_ref	PRIMARY	PRIMARY	4	tuning.부서관리자.사원번호	1	100.00	
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


### <span style="color:#802548">_겹치지 않는다면 UNION ALL_</span>
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
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	PRIMARY	사원		ref	I_성별_성	I_성별_성	51	const,const	135	100.00	Using index
	2	UNION	사원		ref	I_성별_성	I_성별_성	51	const,const	91	100.00	Using index
	3	UNION RESULT	<union1,2>		ALL							Using temporary
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
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	PRIMARY	사원		ref	I_성별_성	I_성별_성	51	const,const	135	100.00	Using index
	2	UNION	사원		ref	I_성별_성	I_성별_성	51	const,const	91	100.00	Using index
```

### <span style="color:#802548">_index 순서대로 GROUP BY해라_</span>
- 성과 성별로 묶어서 조회해보자.
```sql
SELECT 성, 성별, COUNT(1) AS 카운트
FROM 사원
GROUP BY 성, 성별
```

```
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE	사원		index	I_성별_성	I_성별_성	51		299379	100.00	Using index; Using temporary
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
	id	select_type	table	partitions	type	possible_keys	key	key_len	ref	rows	filtered	Extra
	1	SIMPLE	사원		index	I_성별_성	I_성별_성	51		299379	100.00	Using index
```

### <span style="color:#802548">_많이 filter할 수 있는 조건을 앞으로_</span>
- 입사일자와 사원번호를 기준으로 filtering을 해 조회해보자.
```sql
SELECT 사원번호
FROM 사원
WHERE 입사일자 LIKE '1989%'
AND 사원번호 > 100000;
```

