```sql
select 
a.SEQUENCE_NO
, a.FUND_CODE
, a.UPDATE_DATE
, a.TYPE
, a.VALID_DATE
, a.INTERNAL_NO
, a.BALANCE
, a.CANCELED
, a.PROCESS_TIME
, a.API_ID
, a.ROW_VERSION
, B.FUND_NAME
, B.CURRENCY_CLASS
, B.CURRENCY_CLASS_NAME
, d.ITEM_NAME
, SUM(e.CHANGE_AMOUNT) as additionalSum
, SUM(f.CHANGE_AMOUNT) as cancelSum
from
    BALANCE_INFO a
LEFT JOIN FUND b
    on b.FUND_CODE = a.FUND_CODE
    and b.START_DATE <= {basisDate}
    and b.END_DATE >= {basisDate}
    and b.DELETED = false
    and b.APPROVED in ('1', '2', '3')
LEFT JOIN CURRENCY_INFO c
    on a.CURRENCY_CLASS = b.CURRENCY_CLASS
    and c.START_DATE <= {basisDate}
    and c.END_DATE <= {basisDate}
    and c.DELETED = false
    and b.APPROVED in ('1', '2', '3')
LEFT JOIN CODE d
    on d.CODE = 'A01'
    and d.ITEM_CODE = a.CANCELED
    and d.DELTEED = false
LEFT JOIN CHANGE_INFO e
    on e.FUND_CODE = a.FUND_CODE
    and e.CHANGE_DATE = a.UPDATE_DATE
    and e.DEPOSIT_AND_WITHDRAWN = '1'
    and b.CANCELD = false
    and b.APPROVED in ('1', '2', '3')
LEFT JOIN CHANGE_INFO f
    on f.FUND_CODE = a.FUND_CODE
    and f.CHANGE_DATE = a.UPDATE_DATE
    and f.DEPOSIT_AND_WITHDRAWN = '2'
    and f.CANCELD = false
    and f.APPROVED in ('1', '2', '3')
WHERE a.REVOKED = false
and a.UPDAET_DATE <= {basisDate}
and VALID_DATE >= {basisDate}
GROUP BY
    a.SEQUENCE_NO
    , a.FUND_CODE
    , a.UPDATE_DATE
    , a.TYPE
    , a.VALID_DATE
    , a.INTERNAL_NO
    , a.BALANCE
    , a.CANCELED
    , a.PROCESS_TIME
    , a.API_ID
    , a.ROW_VERSION
    , B.FUND_NAME
    , B.CURRENCY_CLASS
    , B.CURRENCY_CLASS_NAME
    , d.ITEM_NAME
```


to

- Single-Pass Aggregation: Uses your CASE WHEN pattern to scan CHANGE_INFO exactly once.
- Scope Protection: The CHANGE_DATE <= {basisDate} clause prevents the subquery from reading unnecessary future history records.
- Safe Output: Keeps your COALESCE handling to ensure NULL values show up cleanly as 0.

```sql
SELECT
      a.SEQUENCE_NO
    , a.FUND_CODE
    , a.UPDATE_DATE
    , a.TYPE
    , a.VALID_DATE
    , a.INTERNAL_NO
    , a.BALANCE
    , a.CANCELED
    , a.PROCESS_TIME
    , a.API_ID
    , a.ROW_VERSION
    , b.FUND_NAME
    , b.CURRENCY_CLASS
    , b.CURRENCY_CLASS_NAME
    , d.ITEM_NAME
    , COALESCE(ch.additionalSum, 0) AS additionalSum
    , COALESCE(ch.cancelSum, 0) AS cancelSum
FROM BALANCE_INFO a
LEFT JOIN FUND b
    ON b.FUND_CODE = a.FUND_CODE
   AND b.START_DATE <= {basisDate}
   AND b.END_DATE >= {basisDate}
   AND b.DELETED = false
   AND b.APPROVED IN ('1', '2', '3')
LEFT JOIN CURRENCY_INFO c
    ON c.CURRENCY_CLASS = b.CURRENCY_CLASS
   AND c.START_DATE <= {basisDate}
   AND c.END_DATE >= {basisDate}
   AND c.DELETED = false
   AND c.APPROVED IN ('1', '2', '3')
LEFT JOIN CODE d
    ON d.CODE = 'A01'
   AND d.ITEM_CODE = a.CANCELED
   AND d.DELETED = false
LEFT JOIN (
    SELECT
          FUND_CODE
        , CHANGE_DATE
        , SUM(
            CASE
                WHEN DEPOSIT_AND_WITHDRAWN = '1'
                THEN CHANGE_AMOUNT
                ELSE 0
            END
          ) AS additionalSum
        , SUM(
            CASE
                WHEN DEPOSIT_AND_WITHDRAWN = '2'
                THEN CHANGE_AMOUNT
                ELSE 0
            END
          ) AS cancelSum
    FROM CHANGE_INFO
    WHERE CANCELED = false
      AND APPROVED IN ('1', '2', '3')
      AND CHANGE_DATE <= {basisDate}
    GROUP BY
          FUND_CODE
        , CHANGE_DATE
) ch
    ON ch.FUND_CODE = a.FUND_CODE
   AND ch.CHANGE_DATE = a.UPDATE_DATE
WHERE a.REVOKED = false
  AND a.UPDATE_DATE <= {basisDate}
  AND a.VALID_DATE >= {basisDate};
```

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
| id | select_type         | table | type  | possible_keys | key      | ref                     | Extra       |
|----|---------------------|-------|-------|---------------|----------|-------------------------|-------------|
| 1  | PRIMARY             | 사원  | range | PRIMARY       | PRIMARY  | null                    | Using where |
| 2  | DEPENDENT SUBQUERY  | 급여  | ref   | PRIMARY       | PRIMARY   | tuning.사원.사원번호     | null        |
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
| id | select_type | table      | type  | possible_keys      | key          | ref                    | Extra                       |
|----|-------------|------------|-------|--------------------|--------------|------------------------|-----------------------------|
| 1  | PRIMARY     | 사원       | range | PRIMARY            | PRIMARY      | null                   | Using where; Using index   |
| 1  | PRIMARY     | <derived2> | ref   | <auto_key0>        | <auto_key0>  | tuning.사원.사원번호   |                             |
| 2  | DERIVED     | 급여       | index | PRIMARY,I_사용여부  | PRIMARY      | null                   |                             |
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
| id | select_type         | table  | type  | possible_keys                    | key      | ref                   | Extra                        |
|----|---------------------|--------|-------|----------------------------------|----------|-----------------------|------------------------------|
| 1  | PRIMARY             | 사원   | range | PRIMARY,I_입사일자,I_성별_성      | PRIMARY  | null                  | Using where; Using index    |
| 4  | DEPENDENT SUBQUERY | 급여3  | ref   | PRIMARY                          | PRIMARY  | tuning.사원.사원번호  |                              |
| 3  | DEPENDENT SUBQUERY | 급여2  | ref   | PRIMARY                          | PRIMARY  | tuning.사원.사원번호  |                              |
| 2  | DEPENDENT SUBQUERY | 급여1  | ref   | PRIMARY                          | PRIMARY  | tuning.사원.사원번호  |                              |
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
| id | select_type | table | type  | possible_keys                    | key      | ref                     | Extra                                         |
|----|-------------|-------|-------|----------------------------------|----------|-------------------------|-----------------------------------------------|
| 1  | SIMPLE      | 사원  | range | PRIMARY,I_입사일자,I_성별_성     | PRIMARY  | null                    | Using where; Using temporary; Using filesort |
| 1  | SIMPLE      | 급여  | ref   | PRIMARY                          | PRIMARY  | tuning.사원.사원번호    |                                               |
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
| id | select_type | table   | type  | possible_keys | key      | ref                    | Extra                    |
|----|-------------|---------|-------|---------------|----------|------------------------|--------------------------|
| 1  | SIMPLE      | 사원    | range | PRIMARY       | PRIMARY  | null                   | Using where; Using index |
| 1  | SIMPLE      | 관리자  | ref   | PRIMARY       | PRIMARY  | tuning.사원.사원번호    | Using index              |
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

## <span style="color:#802548">_매칭_</span>
- table과 table 간의 관계를 filtering 조건으로 걸고 싶다면?
- 그럴 때 상수로 wherer column = "해당상수";로 하면 해당상수가 바뀌면 망한다.
- 그러한 이유로 상관서브쿼리를 활용한 매칭을 실행한다. 구체적 예시를 살펴보자.

- Address2 table에 있는 이름에 한해서 Address에서 이름을 찾게 한다. 이걸 상수로 했다면? 매번 바꿔야하니 끔찍했을 것이다.
```sql
SELECT name
    FROM Address
    WHERE name in (SELECT name FROM Address2)
```

- 매칭은 위와 같이 상관 서브쿼리와 같이 쓰이는 경우가 흔하다.
- in보다는 exists가 성능이 더 좋은 경우가 많으니 가능하다면 exists를 활용해보자.
```sql
SELECT name
FROM customers c
WHERE EXISTS (
    SELECT 1
    FROM orders o
    WHERE o.customer_id = c.customer_id
    AND o.order_date > '2023-01-01' -- Change this to your desired date
);
```


## <span style="color:#802548">_case-when_</span>


- 마찬가지로 UNION ALL만이 아니라 UNION도 줄일 수 있다.
```sql
SELECT prefecture, SUM(pop_men) AS pop_men, SUM(pop_wom) AS pop_wom
FROM    ( 
        SELECT prefecture, pop AS pop_men, null AS pop_wom
            FROM Population
            WHERE sex = '1'

        UNION

        SELECT prefecture, NULL AS pop_men, pop AS pop_wom
            FROM Population
            WHERE sex = '2'
        ) TMP
GROUP BY prefecture;
```

```sql
select 
        prefacture, 
        SUM(CASE WHEN sex ='1' THEN pop ELSE 0 END) AS pop_men, 
        SUM(CASE WHEN sex ='2' THEN pop ELSE 0 END) AS pop_wom
FROM Population
GROUP BY prefecture;
```

- CASE WHEN을 사용하는 또다른 예제를 살펴보자.
- 요구조건은 아래와 같다.
```
소속팀이 1개라면 해당 직원은 팀의 이름을 그대로 출력한다
소속팀이 2개라면 해당 직원은 2개 직무를 겸무라는 문자열을 출력한다
소속팀이 3개이상이면 해당 직원은 3개이상을 겸무라는 문자열을 출력한다
```

- 절차지향으로 짠다면 아래와 같이 짤지도 모른다.
```sql
SELECT emp_name,
        max(team) AS TEAM /* team이 아니라 max(team)인 이유는 바로 GROUP BY에 쓰인 column이 있기 때문이다. 집약구가 들어가면, 집약구에 있어야만 select에서 온전히 쓸 수 있다. 그게 아니면 MAX와 같이 집약 함수와 같이 써야 한다.*/
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) = 1

UNION

SELECT emp_name,
        '2개를 겸무' AS TEAM
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) = 2

UNION

SELECT emp_name,
        '3개 이상을 겸무' AS TEAM
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) >= 3;
```

- 매우 심각하게 부담이 가는 실행 계획이다. table full scan을 3번이나 한다. 줄여놓자.
- 아래와 같이 바꿔보자.  안타깝게도 이건 불가능하다. 
- emp_name으로 group을 나누지 않으면 relation인 Employees의 모든 row를 count하기 때문이다.
```sql
SELECT case when count(emp_name) = 1 then team
		when count(emp_name) = 2 then '2개를 겸무'
		when count(emp_name) >= 3 then '3개를 겸무' END AS team
FROM Employees  
```


- 아래와 같이 GROUP BY를 써줘야 한다. 그리고 GROUP By에 안 썼는데, select에 쓰려면 MAX와 같은 집약함수를 써야만 한다.
```sql
SELECT 직원이름,   
        CASE 
             WHEN COUNT(*) = 1 THEN MAX(team) /* select로 선택한 field가 한개이기 때문에 COUNT(*)는 emp_name이 몇개인지 세는 형태로 진행된다. 다만 COUNT (emp_name)과 다르게 null도 포함해 센다. COUNT(*)와 COUNT(1)은 동일 */
             WHEN COUNT(*) = 2 THEN '2개를 겸무'
             WHEN COUNT(*) = 3 THEN '3개를 겸무'
        END AS team
FROM Employees
GROUP BY 직원이름;
```

- sum, count, avg, max, min가 표준 집약 함수다.
- 이들을 이용해서 union으로는 합칠 수 없는 값을 합쳐서 나타낼 수 있다.
- union은 field의 갯수가 다르면 합칠수가 없다.
```sql
SELECT id, data_1, data_2
FROM NonAggTbl
WHERE id = 'Jim'
AND data_type = 'A';

UNION

SELECT id, data_3, data_4, data_5
FROM NonAggTbl
WHERE id = 'Jim'
AND data_type = 'B'

UNION

SELECT id, data_6
FROM NonAggTbl
WHERE id = 'Jim'
AND data_type ='C'
```


- 따라서 CASE WHEN을 사용해서 합친다.
```sql
SELECT id,
        CASE WHEN data_type ='A' THEN data_1 ELSE NULL END AS data_1,
        CASE WHEN data_type ='A' THEN data_2 ELSE NULL END AS data_2,
        CASE WHEN data_type ='B' THEN data_3 ELSE NULL END AS data_3,
        CASE WHEN data_type ='B' THEN data_4 ELSE NULL END AS data_4,
        CASE WHEN data_type ='B' THEN data_5 ELSE NULL END AS data_5,
        CASE WHEN data_type ='C' THEN data_6 ELSE NULL END AS data_6,
FROM NonAggTbl
GROUP BY id;
```

- 하지만 오류가 난다. GROUP BY에 data_1을 적어놓지 않았기 때문이다.
- 그렇기에 MAX로 감싸준다.
```sql
SELECT id,
        MAX(CASE WHEN data_type ='A' THEN data_1 ELSE NULL END AS data_1),
        MAX(CASE WHEN data_type ='A' THEN data_2 ELSE NULL END AS data_2),
        MAX(CASE WHEN data_type ='B' THEN data_3 ELSE NULL END AS data_3),
        MAX(CASE WHEN data_type ='B' THEN data_4 ELSE NULL END AS data_4),
        MAX(CASE WHEN data_type ='B' THEN data_5 ELSE NULL END AS data_5),
        MAX(CASE WHEN data_type ='C' THEN data_6 ELSE NULL END AS data_6),
FROM NonAggTbl
GROUP BY id;
```


## <span style="color:#802548">_window function_</span>
- 이제 반복을 표현하는 다른 방법을 살펴보자.
- 그럼 위의 sql보다 좀 더 복잡한 포장계는 어떻게 구성될까?
- 흔한 방법 중에 서브쿼리 혹은 case-when + window function이 있다.
- 직전회사명과 직전매상을 검색한다고 해보자. 서브쿼리로는 아래와 같다.

```sql
SELECT compnay,
        year,
        sale,
        CASE SIGN(sale -(SELECT sale /** 직전 연도의 매상 선택 */
                            FROM Sales SL2
                            WHERE SL1.company = SL2.company
                            AND SL2.year = /**직전 연도 선택 */
                            (SELECT MAX(year)
                                FROM Sales SL3
                            WHERE SL1.company = SL3.company
                            AND SL1.year > SL3.year )))
        WHEN 0 THEN = '='
        WHEN 1 THEN = '+'
        WHEN -1 THEN = '-'
        ELSE NULL END AS var
FROM Sales;
```

- 아래는 window function을 쓴 예시다.
- 왜 MAX를 사용해야 할까? 사실 sql의 resultSet에 영향을 미치는 것은 없다.
- 그저 MAX 같은 집약, 랭킹 등의 함수없이 OVER와 같은 winodw function을 사용하면 오류가 나기 때문이다.
  - 그러한 함수로는 count, max, min, avg, sum, rank, row_number, lag, lead 등이 있다.
```sql
SELECT company,
        year,
        sale,
        CASE SIGN(sale - MAX(sale)
                            OVER (PARTITION BY company
                                    ORDER BY year
                                    ROWS BETWEEN 1 PRECEDING
                                            AND 1 PRECEDING))
            WHEN 0 THEN '='
            WHEN 1 THEN '+'
            WHEN -1 THEN '-'
            ELSE NULL END AS var
FROM Sales;
```

- 위의 함수는 lag를 사용하면 아래와 같이 간단하게 바뀐다.
```sql
SELECT company,
        year,
        sale,
        CASE SIGN(sale - lag(sale)
                            OVER (PARTITION BY company
                                    ORDER BY year
                                    ))
            WHEN 0 THEN '='
            WHEN 1 THEN '+'
            WHEN -1 THEN '-'
            ELSE NULL END AS var
FROM Sales;
```



- 상관 서브쿼리는 window 함수의 PARTITION BY와 ORDER BY와 같은 기능을 한다.
- 그러나 성능이 문제다. 실행계획이 변동한다는 것도 리스크다.
```sql
SELECT company,
        year,
        sale,
        (SELECT company
        FROM Sales S2
        WHERE S1.company = S2.company
        AND year = (SELECT MAX(year)
                            FROM Sales S3
                            WHERE S1.company = S3.company
                            AND S1.year > S3.year)) AS pre_company,
        (SELECT sale
                FROM Sales S2
        WHERE S1.company = S2.company
        AND year = (SELECT MAX(year)
                            FROM Sales S3
                            WHERE S1.company = S3.company
                            AND S1.year > S3.year)) AS pre_sale
FROM Sales S1;
```

- window function을 사용하면 아래와 같이 단순하게 바꿀 수 있다.
- 왜 MAX를 사용해야 할까? MAX같은 집약함수없이 OVER와 같은 winodw function을 사용하면 오류가 나기 때문이다.
  - 그러한 함수로는 count, max, min, avg, sum, rank, row_number, lag, lead 등이 있다.
```sql
SELECT company,
        year,
        sale,
        MAX(company)
            OVER (PARTITION BY company
                ORDER BY year
                ROWS BETWEEN 1 PRECEDING
                        AND  1 PRECEDING) AS pre_company,
        MAX(sale)
            OVER (PARTITION BY company
                    ORDER BY year
                    ROWS BETWEEN 1 PRECEDING
                            AND 1 PRECEDING) AS pre_sale
FROM Sales;
```


- 포장계 sql의 다른 예시를 살펴보자.
- 우편번호가 가장 유사한 것을 얻어오는 SQL을 반복문으로 짜본다고 해보자.
- 그럼 5번을 반복해야 할 것이다.
```sql
SELECT pcode
FROM Address
WHERE pcode = '20178';

UNION ALL

SELECT pcode
FROM Address
WHERE pcode like '2017%';

UNION ALL

SELECT pcode
FROM Address
WHERE pcode like '201%';

UNION ALL

SELECT pcode
FROM Address
WHERE pcode like '20%';
 
UNION ALL

SELECT pcode
FROM Address
WHERE pcode = '2%';
```

- select를 무려 7번이나 실행한다.
- 이건 성능에 매우 좋지 않다. 특히 데이터가 커질수록 말이다.
- 포장계로 바꿔주자.

```sql
SELECT pcode,
        district_name,
FROM Address
WHERE CASE WHEN pcode = '20178' TEHN 0
            WHEN  pcode like '2017%' THEN 1
            WHEN  pcode like '201%' THEN 2
            WHEN  pcode like '20%' THEN 3
            WHEN  pcode like '2%' THEN 4
            ELSE NULL END = 
            (SELECT MIN(CASE WHEN pcode = '20178' TEHN 0
                            WHEN  pcode like '2017%' THEN 1
                            WHEN  pcode like '201%' THEN 2
                            WHEN  pcode like '20%' THEN 3
                            WHEN  pcode like '2%' THEN 4
                            ELSE NULL END)
            FROM    Address);
```

- table full scan을 두 번 떄리고 있다.
```
Id          |Operation          |Name
0           |SELECT SSTATEMENT  
1           |TABLE ACCESS FULL  |Address
2           |SORT AGGREGATE     |
3           |TABLE ACCESS FULL  |Address
```

- SELECT 절에 index가 새겨진 field만 담았다면, 아래와 같은 실행계획이 나온다.
- 이른바  Oracle에서INDEX FAST FULL SCAN이다.
- 다만 이 접근은 select가 가져올 field가 모두 index가 있을 때만 사용가능하다.
```
Id          |Operation              |Name
0           |SELECT SSTATEMENT  
1           |TABLE ACCESS FULL      |Address
2           |SORT AGGREGATE         |
3           |INDEX FAST FULL SCAN   |PK_PCODE
```



- window function을 이용하면 table full scan을 한번으로 줄일 수 있다.
```sql
SELECT pcode,
        district_name,
FROM (SELECT pcode,
            district_name,
            CASE WHEN pcode = '20178' THEN 0
                WHEN  pcode like '2017%' THEN 1
                WHEN  pcode like '201%' THEN 2
                WHEN  pcode like '20%' THEN 3
                WHEN  pcode like '2%' THEN 4
                ELSE NULL END AS hit_code,
            MIN(CASE WHEN pcode = '20178' THEN 0
                    WHEN  pcode like '2017%' THEN 1
                    WHEN  pcode like '201%' THEN 2
                    WHEN  pcode like '20%' THEN 3
                    WHEN  pcode like '2%' THEN 4
                    ELSE NULL END)
            OVER(ORDER BY  CASE WHEN pcode = '20178'    THEN 0
                                WHEN  pcode like '2017%' THEN 1
                                WHEN  pcode like '201%' THEN 2
                                WHEN  pcode like '20%' THEN 3
                                WHEN  pcode like '2%' THEN 4
                                ELSE NULL END) AS min_code
    FROM Address) Foo
WHERE hit_code = min_code;
```

- table full scan이 1회 줄어들었다. 다만 동시에 정렬이 추가로 사용됐다.
- 하지만 table 크기가 크다면 table full scan을 줄이는 게 훨씬 나은 선택이다.
- SELECT가 2개여도 table scan이 1회인 이유는 window function을 보고 optimizer가 최적화했기 때문이다.
```
Id          |Operation              |Name
0           |SELECT STATEMENT  
1           |VIEW                   |
2           |WINDOW SORT            |
3           |INDEX FAST FULL SCAN   |Address
```


## <span style="color:#802548">_record에 순번 붙이기 심화_</span>
- 중앙 값을 구하려고 한다면, 모집합을 상위와 하위로 분할한다.
- 다만 아래 sql은 복잡해서 이해하기 어렵고, 성능도 나쁘다.

```sql
SELECT AVG(weight)
    FROM (SELECT W1.weight
            FROM Weigths W1, Weigths W2
                GROUP BY W1.weigth
                HAVING SUM(CASE WHEN W2.weight >= W1.weigth THEN 1 ELSE 0 END) >= COUNT(*) /2 /**하위 집합의 조건 */
                AND HAVING SUM(CASE WHEN W2.weight <= W1.weigth THEN 1 ELSE 0 END) >= COUNT(*) /2) TMP;/**상위 집합의 조건 */
```



- 이럴 떄는 절차지향을 쓰게 된다.
- 양쪽 끝에서 계속 나아가서 만나는 지점이 중앙값이기에, 그걸 sql로 구현하는 것이다.
- 홀수라면 hi IN (lo)면 되겠지만, 짝수는 균형이 안맞아서 lo+1, lo-1도 붙여준다.
- 홀수는 50, 짝수는 49 +51 /2 로 중앙값이 계산되는 식이다.
- 이렇게 아래와 같이 짜면 Weigths 테이블 접근이 1회로 감소하고, 결합이 사용되지 않는다. 
- 대신 정렬이 2회 늘어나지만, Weights가 충분히 크다면 scan을 덜하는 게 더 이득이다.

```sql
SELECT AVG(weight) AS median
    FROM (SELECT weight,
    ROW_NUMBER() OVER (ORDER BY weight ASC, student_id ASC) AS hi,
    ROW_NUMBER() OVER (ORDER BY weight DESC, student_id DESC) AS lo
    FROM Weights) TMP
    WHERE hi IN (lo, lo+1, lo-1);
```

- 여기서 결합을 1번 더 줄일 수 있다.
- record의 갯수를 세서 count한다.
- record마다 순번을 매기고 2를 곱한뒤 위의 count와 뺀다.
- 그렇게 구한 diff 중 0~2에 해당하는 값이 바로 중앙 값이다.
- 짝수인 경우 0과 2, 홀수인 경우 1이므로 어느 경우에도 구하려면 평균을 내준다.
```sql
SELECT AVG(weigth)
    FROM (SELECT weight,
                    2 * ROW_NUMBER() OVER(ORDER BY weight)
                        - COUNT(*) OVER() AS diff
                FROM Weights) TMP
WHERE diff BETWEEN 0 AND 2;
```

- 이번에는 비어있는 숫자를 맞춰보자.
```
Numbers
num
1
3
4
7
8
9
12
```
- 다음 숫자와 1을 비교했을 떄, -1을 해서 1이 안 나오면 비어있다고 보면 된다.
- 3에서 1을 빼도 1이 나오지 않으므로 단절이 있다. 비어있는 숫자인 것이다.
- 7에서 1을 빼도 4가 나오지 않으므로 단절이 있다. 비어있는 숫자인 것이다.
- 다만 이렇게 하면 NL이기 때문에 실행계획이 불안정하다. table scan도 2번일어난다.
```sql
SELECT (N1.num + 1) AS gap_start,
        '~',
        (MIN(N2.num) - 1) AS gap_end
    FROM Numbers N1 INNER JOIN Numbers N2
    ON N2.num > N1.num
GROUP BY N1.num
HAVING(N1.num + 1) < MIN(N2.num);
```

- 위의 쿼리를 절차 지향으로 바꾸면 성능이 개선된다.
- 결합을 사용하지 않고, 테이블 스캔도 한번만 발생한다.
```sql
SELECT num + 1 AS gap_start,
    '~',
    (num + diff - 1) AS gap_end
    FROM (SELECT num,
                MAX(num)
                OVER(ORDER BY num
                    ROWS BETWEEN 1 FOLLOWING
                    AND 1 FOLLOWING) - num AS diff
            FROM Numbers) TMP
WHERE diff <> 1;
```

- 위의 쿼리는 아래와 동일하다.
```sql
SELECT num + 1 AS gap_start,
    '~',
    (num + diff - 1) AS gap_end
    FROM (SELECT num,
                MAX(num)
                OVER(ORDER BY num
                    ROWS BETWEEN 1 FOLLOWING
                    AND 1 FOLLOWING) - num 
            FROM Numbers) TMP(num,diff)
WHERE diff <> 1;
```

- 테이블에 존재하는 시퀀스 구하기는 집합 지향이나, 절차 지향이나 모두 성능이 비슷하다.
- 그러나 절차 지향으로 쓰면 쿼리가 너무 길어지므로 집합 지향으로 쓰자.
```sql
SELECT MIN(num) AS low,
        '~',
        MAX(num) AS high
    FROM (SELECT N1.num,
                COUNT(N2.num) - N1.num
            FROM Numbers N1
            INNER JOIN Numbers N2
            ON N2.num <= N1.num
            GROUP BY N1.num) N(num,gp)
    GROUP BY gp;
```

## <span style="color:#802548">_select문을 최대로 줄인 다중 필드 update_</span>
- 한 과목씩 값을 갱신하는 sql이 있다고 해보자.
- sql을 세번씩이나 쓰는 건 좋은 방식이 아니다.
```sql
update ScoreCols
    set score_en = (select score 
                    from ScoreRows SR
                    where SR.student_id = ScoreCols.student_id
                    and subject = '영어'),
        score_nl = (select score
                    from ScoreRows SR
                    where SR.student_id = ScoreCols.student_id
                    and subject = '국어'),
        score_mt = (select score
                    from ScoreRows SR
                    where SR.student_id = ScoreCols.student_id
                    and subject = '수학');
```

- 다중 필드 할당을 아래와 같이 해주면 select query 하나로 3개의 field를 update
- 이건 oracle에서의 구문

```sql
update ScoreCols
    set (score_en, score_nl, score_mt)
        = (select   max(case when subject = '영어'
                        then score
                        else null end) as score_en,
                    max(case when subject = '국어'
                        then score
                        else null end) as score_nl,
                    max(case when subject = '수학'
                        then score
                        else null end) as score_mt
        from scoreRows SR
        where SR.student_id = ScoreCols.student_id);
```

- mysql의 구문은 아래와 같음.


```sql
UPDATE ScoreCols
JOIN (
    SELECT
        student_id,
        MAX(CASE WHEN subject = '영어' THEN score ELSE NULL END) AS score_en,
        MAX(CASE WHEN subject = '국어' THEN score ELSE NULL END) AS score_nl,
        MAX(CASE WHEN subject = '수학' THEN score ELSE NULL END) AS score_mt
    FROM scoreRows
) AS Subquery
ON ScoreCols.student_id = Subquery.student_id
SET
    ScoreCols.score_en = Subquery.score_en,
    ScoreCols.score_nl = Subquery.score_nl,
    ScoreCols.score_mt = Subquery.score_mt;
```


- 하지만 위의 경우들로는 not null의 경우를 대응할 수 가 없다.
- 이제는 not null도 대응할 수 있게 만들어보자.
- 아래와 같이 바꾸면 여전히 select문을 3번 쓴다. table scan이 3번 일어난다.

```sql
update ScoreColsNN
    set score_en = coalesce( (select score
                                from ScoreRows
                                where student_id = ScoreColsNN.student_id
                                and subject = '영어'),0),
        score_nl = coalesce( (select score
                                from ScoreRows
                                where student_id = ScoreColsNN.student_id
                                and subject = '수학'),0),
        score_mt = coalesce( (select score
                                from ScoreRows
                                where student_id = ScoreColsNN.student_id
                                and subject = '수학'), 0)
where exists (select *  /*처음부터 학생이 존재하지 않는 떄의 null 대응 */
                from ScoreRows 
                where student_id = ScoreColsNN.student_id);
```


- table scan을 1번으로 줄여주었던 이전의 query문을 이용해 null 처리를 해주자.
- where문이 가장 먼저 읽히므로, 거기서 학생 field가 존재하는지 판별해준다. 
  - 학생이 없다면 애초에 update 할 필요 자체가 없다.
- 그리고 나서 update를 할 때 이제는 학생은 있지만, 과목 점수가 null인 경우를 다뤄야 한다.
  - 그 때는 과목 점수가 0인 경우를 coalesce함수로 감싸서 0으로 만들어준다.
- coalesce는 ifnull과 같은데, 어느 RDB에서나 쓸 수 있다. 따라서 해당 함수를 쓰도록 하자.
- coalesce(column , default value)와 같은 형식으로, 해당 record가 null이면 default value로 대체된다.


```sql
update ScoreCols
    set (score_en, score_nl, score_mt)
        = (select  coalesce(max(case when subject = '영어'
                        then score
                        else null end),0) as score_en,
                    coalesce(max(case when subject = '국어'
                        then score
                        else null end),0) as score_nl,
                    coalesce(max(case when subject = '수학'
                        then score
                        else null end),0) as score_mt
        from scoreRows SR
        where SR.student_id = ScoreCols.student_id)
where exists (select *
                    from ScoreRows
                    where student_id = ScoreCols.student_id);
```    


- 원 테이블을 두고 다른 테이블에 특정 field를 추가하여 data를 관리하는 경우가 있다.
- 주가가 대표적이다. 
- 브랜드, 거래일, 종가를 기준으로 원 테이블을 관리한다.
- 그리고 그 관리되는 테이블에 가격 상승하락 field를 넣어 추가 테이블을 만든다.(이거 농좆에서는 scrapping이라고 하는듯)

- 그럼 아래와 같이 상관 subquery를 사용할 수 있다.
- 하지만 상관 subquery는 테이블 접근이 많아진다.
  - index range scan(MIN/MAX)- PK index only scan
  - TABLE ACCESS by index ROWID - PK index를 이용한 table scan 
  - table access full - 풀 스캔
- 윈도우 함수로 바꿔주자.

```sql
insert into stock2
select brand,sale_date, price,
    case sign(price - (select price 
                        from Stocks S1
                        where brand = Stocks.brand
                        and sale_date = (select MAX(sale_date)
                                            from Stocks S2
                                            where brand = Stocks.brand
                                            and sale_date < Stocks.sale_date)))
        when -1 then '아래 화살표'
        when 0 then '오른쪽 화살표'
        when 1 then '위 화살표'
        else null
    end
from Stocks;
```

- 윈도우 함수로 바꾸면 아래와 같이 된다.
- 그럼 테이블 접근이 1회로 줄어든다.
    - table access full - 풀 스캔
    
```sql
insert into Stock2
select brand, sale_date, price,
    case sign(price - MAX(price) over (partition by brand 
                                        order by sale_date
                                    rows between 1 PRECEDING
                                            and 1 PRECEDING))
        when -1 then '아래 화살표'
        when 0 then '오른쪽 화살표'
        when 1 then '위 화살표'
        else null
    end
from Stocks S2;
```