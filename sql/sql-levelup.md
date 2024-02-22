## <span style="color:#802548">_실행계획_</span>
- 어떤 순서로 기억장치의 데이터에 접근할 지에 관한 계획
- 실행계획을 읽는 순서를 sql을 통해 알아보자.

```sql
select shop_name
from Shops S INNER JOIN Reservation R
on S.shop_id = R.shop_id;
```

- 위의 sql은 아래와 같은 실행계획을 보인다.
```
Id  Operation                               Name            Rows    Bytes   Cost(%CPU)  Time
0   Select Statement                                        1       48      3   (0)     00:00:01
1    NESTED LOOPS                                           1       48      3   (0)     00:00:01
2       TABLE ACCESS FULL                   RESERVATIONS    1       7       2   (0)     00:00:01
3           TABLE ACCESS BY INDEX ROWID     SHOPS           1       41      1   (0)     00:00:01
4               INDEX UNIQUE SCAN           PK_SHOPS        1               0   (0)     00:00:01
```
- 실행되는 순서는 2- 4- 3 - 1- 0 ID가 높은수록 더 빨리 실행된다.
- 결합 전에 먼저 결합대상이 되는 두 테이블을 select한다.
- 그 중에 먼저 일어나는 것은 2번 ID인 RESERVAIOTNS다.
  - RESERVATION table은 driven table로 full scan된다. 
  - SHOPS TABLE은 driving table로 index scan이 된다.

## <span style="color:#802548">_버퍼_</span>
- DB는 데이터 접근 속도가 좋은 메모리에 정보를 저장하고 싶어함.
- 그러나 데이터를 메모리에 저장하면 비용이 듦.
  - 메모리 자체가 용량이 크지 않은데, 그걸 써야 하기 때문.
- 따라서 자주 접근하는 데이터만 메모리에 올리는 방식을 택함
  - 그럼 디스크(하드디스크 - HDD)에 접근하지 않기 때문에 속도가 압도적으로 빠름.
  - 바로 이런 성능향상 목적의 데이터를 저장하는 메모리= 버퍼, 캐시
  - 종류는 데이터 캐시, 로그 버퍼.
- 데이터 캐시가 바로 메모리에 있는 데이터를 저장하는 버퍼. 흔히 버퍼라 하면 데이터 캐시
  - 버퍼에서 데이터가 없으면 HDD에 접근하고, 급속도로 느려짐. 디스크 I/O기 때문.
- 로그 버퍼는 갱신처리. insert, delete, update, merge를 때리면, 로그 버퍼위에 변경 정보를 보내고, 디스크에서 변경을 수행하기 때문
  - 즉 갱신처리는 비동기로 이뤄짐. 이렇게 비동기로 하는 이유는 역시 성능 때문. 
  - 갱신 SQL -> 메모리의 로그 버퍼 -> commit -> HDD에 갱신
  - 다시말하면 commit 시에는 무조건 디스크 I/O가 일어나게 됨.
  - 보통 로그 버퍼는 용량이 굉장히 작음. DB는 검색을 기본으로 하기 때문에 데이터 캐시에 많이 용량을 배정하는 것.
  - 만약 갱신이 자주 일어난다면, 로그 버퍼의 크기를 늘려주는 등의 행위가 필요함.
- 메모리 영역은 데이터 캐시, 로그 버퍼 이외에 워킹 메모리도 있음.
  - 해당 영역은 정렬, 해시 관련 처리에 사용됨. 
  - order by, max, min, sum 등의 집합 연산, window function, hash join 등에 사용된다.
  - oracle은 PGA, Mysql은 정렬버퍼라고 부른다.
  - 워킹 메모리가 들어온 데이터보다 작은 경우, 디스크 I/O(저장소 I/O)가 일어나 성능이 매우나빠진다.
    - 데이터양이 늘어서 메모리에 들어가지 않으면 어쩔수없이 저장소를 사용하는 것이다.
    - oracle에서는 TEMP TableSpace라고 부른다. 어쨌든 디스크 위에 존재한다.
    - 특히 하나의 sql문만으로는 데이터가 많지 않지만, 여러개의 sql문을 한꺼번에 수행하는 순간 메모리가 부족해져 디스크 I/O가 일어나기도 한다.
    - 즉 동접자가 많은 경우에 갑자기 성능 이슈가 생길수 있다는 의미다.

## <span style="color:#802548">_= null은 왜 안될까?_</span>
- =은 데이터에 적용할 수 있는 연산자다.
- null은 unknown 상태의 데이터다. 당연히 column = null을 적용할 수가 없다.
- 그럼? IS NULL + IS NOT NULL로 써줘야 한다.


## <span style="color:#802548">_옵티마이저_</span>
- 옵티마이저는 DBMS의 핵심이다. 실행계획을 세우기 때문이다.
- 특히 인덱스, 데이터 분산 정도, SQL 수행 통계 정보, 데이터 양 등을 활용한다.
- SQL 수행 정보 같은 통계 정보 갱신은 특히 데이터가 크게 바뀔 떄 매우 중요하다.
```sql
/*oralce*/
EXEC DBMS_STATS.GATHER_TABLE_STATS('schema_name', 'table_name');
EXEC DBMS_STATS.GATHER_SCHEMA_STATS('schema_name');
EXEC DBMS_STATS.GATHER_DATABASE_STATS;

/*mysql*/
ANALYZE TABLE [tableName]
```

## <span style="color:#802548">_객체조작_</span>
- 객체는 table, index, partition, sequence 등을 의미.
- table scan의 종류는 2가지가 있다.
  - TABLE ACCESS FULL은 full scan을 의미한다.
  - TaBLE ACCESS BY INDEX ROWID는 index를 이용한 scan을 의미한다.
  - 보통 INDEX UNIQUE SCAN,  INDEX RANGE SCAN과 같이 쓰인다. unique는 row가 1개고, range는 row가 multiple인 정도의 차이다.
  - row가 1개이려면 filtering 조건이 PK나 UK, FK 혹은 UI여야 할것이다. multiple은 PK나 UK는 아닌 것이다. 당연히 unique scan이 성능이 더 좋다.
  - 다만 INDEX FULL SCAN은 좀 다른데, 이건 주로 hint와 같이 쓰인다. 즉 where조건문이 없을 때 index를 사용하길 원하는 경우, 주로 쓴다.
  - index scan은 logn 복잡도고, full scan은 n 복잡도라 index scan을 성능 상 많이 쓴다.
  - 다만 index scan의 선택률이 5% 미만일 때가 좋고, 20%를 넘으면 index를 안 쓰고 full scan을 때리는 게 더 성능이 좋다.

## <span style="color:#802548">_결합_</span>
- nested loop, sort meger, hash가 있다.
  - nested loop는 한 테이블을 읽으면서 해당 테이블의 레코드마다, 결합 조건에 맞는 레코드를 다른 테이블에서 찾는 형태다.
  - sort merge는 레코드를 on절(결합키)로 정렬하고, 순차적으로 테이블을 결합한다. 정렬 시 워킹메모리를 사용하게 된다.
  - hash는 결합키값을 해시값으로 매핑하는데, 워킹 메모리를 사용한다.



## <span style="color:#802548">_group by_</span>
- group by로 나누면 count, sum, avg, max, min와 같은 집약(aggregation)함수를 쓰기 쉬워진다.

```sql
SELECT sex, count(*)
    from Address
    GROUP BY sex;
```

```
sex | count
남      4
여      5
```

```sql
SELECT address, COUNT(*)
    from Address
    GROUP BY address;
```

```
address | count
서울시      3
인천시      2
부산시      2
속초시      1
서귀포시    1
```

```sql
SELECT count(*)
    from Address
    GROUP BY (); /*GROUP BY ()는 없는 것과 동일. group by할 기준이 없기 떄문. 그냥 일반 select 문*/
```


```
count 
9
```

## <span style="color:#802548">_HAVING_</span>
- groupy by로 나눴다면, where문이 아닌 HAVING으로 filtering을 한다.

```sql
SELECT address, COUNT(*)
    FROM Address
    GROUP BY address
    HAVING COUNT(*) = 1;
```

```
address | count
속초시   |   1
서귀포시 |  1
```

- 아래에서 *를 이해하는 게 어려울 수도 있다.
- 아래 *는 grouping에 쓰인 column의 조합이다.
- 따라서 사원번호, 연봉 조합의 row가 2개인 경우에만 출력하겠다는 의미다.
```sql
SELECT 사원번호, 연봉, COUNT(*) 
FROM 급여
GROUP BY 사원번호, 연봉
HAVING COUNT(*) = 2;
```

- 참고로 2개인 경우에 사원번호만 알고 싶다면 아래와 같이 연봉은 생략가능하다.
```sql
SELECT 사원번호, COUNT(*) 
FROM 급여
GROUP BY 사원번호, 연봉
HAVING COUNT(*) = 2;
```

- 만약 group by에 쓰인 게 3개라면?
- *는 사원번호,연봉,입사일자의 조합일 것이다.
```sql
SELECT 사원번호, COUNT(*) 
FROM 급여
GROUP BY 사원번호, 연봉,입사일자
HAVING COUNT(*) = 2;
```


## <span style="color:#802548">_order by_</span>
- select문은 어떤 순서로 출력하는가? 정확한 규칙이 없다. 
- 따라서 지멋대로 나온다.
- 그게 싫다면 order by로 반드시 순서를 지정해줘야 한다.
- 참고로 order by date desc하면 최신순으로 정렬된다.

## <span style="color:#802548">_매칭_</span>
- table과 table 간의 관계를 filtering 조건으로 걸고 싶다면?
- 그럴 때 상수로 wherer column = "해당상수";로 하면 해당상수가 바뀌면 망한다.
- 따라서 매칭을 실행한다. 구체적 예시를 살펴보자.

- Address2 table에 있는 이름에 한해서 Address에서 이름을 찾게 한다. 이걸 상수로 했다면? 매번 바꿔야하니 끔찍했을 것이다.
```sql
SELECT name
    FROM Address
    WHERE name in (SELECT name FROM Address2)
```

- 아래와 같이 상관 서브쿼리와 같이 쓰이는 경우가 흔하다.
- in보다는 exists가 성능이 더 좋으니까 exists로 바꿀 수 있다면 바꿔주자.
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

## <span style="color:#802548">_CASE WHEN_</span>
- 교환 정의를 위해 사용하며, 성능에 매우 긍정적이다.
- 간단한 예시를 살펴보자.

```sql
SELECT 
        name, address,
    CASE 
         WHEN address = '서울시' THEN '경기'
         WHEN address = '인천시' THEN '경기'
         WHEN address = '부산시' THEN '영남'
         WHEN address = '속초시' THEN '관동'
         WHEN adresss = '서귀포시'THEN '호남'
    ELSE NULL END AS district
FROM Address
```

```
name        |   address     |   district
인성        |   서울시      |       경기
하진        |   서울시      |       경기
하린        |   부산시      |       영남
빛나래      |   인천시      |       경기
```

- CASE WHEN을 이용하면 UNION ALL 등을 대체할 수 있다.
- 아래와 같이 sql을 날려보자.
```sql
SELECT item_name, year, price_tax as price
FROM Items
WHERE year <= 2001

UNION ALL

SELECT item_name, year, price_tax_in as price
FROM Items
WHERE year <= 2002
```

- 그럼 실행계획이 아래와 같이 나온다.
```
id      Operation                   Name                Rows
0       SELECT STATEMENT                                13
1       UNION_ALL                   
2       TABLE ACCESS FULL           ITEMS               7
3       TABLE ACCESS FULL           ITEMS               6
```

- 결합이 아니기 때문에 순서대로 읽으면 된다.
- 즉 2번이 먼저고, 3번이 그다음이다.
- index를 사용하지 않는 full scan이 2번 이뤄졌다.
- 하지만 where 조건문에서 CASE WHEN 분기를 하지 않고, select에서 한다면 TABLE ACCESS를 줄일 수 있다.
```sql
SELECT item_name, year,
        CASE WHEN year <= 2001  THEN price_tax_ex
            WHEN year <= 2002   THEN price_tax_in END AS PRICE
FROM Items;
```

- 그럼 아래와 같이 실행계획이 1번으로 줄어든다.
```
id      Operation               Name
0       SELECT STATEMENT
1       TABLE ACCESS FULL       ITEMS
```

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

- 이번에는 PostgreSql이다.
```
HashAggregate (rows=10)             //select
->  HashAggregate (rows = 10)       //select
    ->  append (rows = 10)          //union
        => Seq Scan on population   //table full scan
            Filter: (sex = '1')     //no index filtering
        -> Seq Scan on popuplation  //table full scan
            Filter: (sex = '2')     //no index filtering
```

- 위의 subquery union을 CASE WHEN으로 바꿔주자.
```sql
select 
        prefacture, 
        SUM(CASE WHEN sex ='1' THEN pop ELSE 0 END) AS pop_men, 
        SUM(CASE WHEN sex ='2' THEN pop ELSE 0 END) AS pop_wom
FROM Population
GROUP BY prefecture;
```

- 그럼 table을 한번만 스캔한다.
```
HashAggregate (rows=10)             //select
    => Seq Scan on population       //table full scan
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

- 그럼 아래와 같은 실행계획이 나온다.
```
Id              Operation                   Name
0               SELECT STATEMENT
1                SORT UNIQUE                                    // UNION이라고 쓰면 실제로 모든 RECORD가 UNIQUE한지 확인한 후 합침
2                 UNION-ALL                                     // 합치는 작업은 UNION ALL
3                  FILTER
4                   HASH GROUP BY
5                    TABLE ACCESS FULL      EMPLOYEES
6                  FILTER
7                   HASH GROUP BY
8                    TABLE ACCESS FULL      EMPLOYEES
9                   FILTER
10                   HASH GROUP BY
11                    TABLE ACCESS FULL     EMPLOYEES
```

- 매우 심각하게 부담이 가는 실행 계획이다. 
- table full scan을 3번이나 한다. 줄여놓자.
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


- 그러면 table full scan 한번만 때리게 바뀐다!
```
Id              Operation                   NAME
0               SELECT STATEMENT
1               HASH GROUP BY
2               TABLE ACCESS FULL           EMPOYEES
```

## <span style="color:#802548">_union_</span>
- CASE WHEN을 쓰는게 더 좋지만, union을 써야할 때도 있다.
- 바로 table 자체가 다를 때 결과값을 통합하는 경우다.
- table 자체가 다른 때에는 결과를 아래와 같이 union으로 통합해야 한다.
```sql
SELECT name
FROM user
where id = '4'

UNION ALL

SELECT name
FROM admin
where id = '1'
```

- 또 하나의 경우는, union을 사용할 때 index를 타게 되는 경우다.
- index를 타지 않는 경우를 먼저 살펴보자. where절에서 or를 사용했을 경우, 법칙은 아니지만 대부분의 경우 index를 타지 않는다.
```sql
SELECT key, name, date_1, flg_1, date_2, flg_2, date_3, flg_3
FROM ThreeElements
WHERE (date_1 = '2013-11-01' AND flg_1 = 'T')
OR  (date_2 = '2013-11-01' AND flg_1 = 'T')
OR  (date_2 = '2013-11-01' AND flg_1 = 'T')
```
- or문을 압축하면 아래와 같은 in문이 된다.
```sql
SELECT key, name, date_1, flg_1, date_2, flg_2, date_3, flg_3
FROM ThreeElements
WHERE ('2013-11-01','T') IN ( (date_1,flg_1), (date_2,flg_2), (date_3,flg_3));
```

- 아래도 OR나 IN을 사용할 때와 동일한 문제를 안고 있다. multiple column을 OR로 연결한 것과 같다.
- 하지만 where문에는 CASE WHEN을 쓰지 말자. 비즈니스 로직이 바뀌면 아예 다른 결과를 초래한다.
```sql
SELECT key, name, date_1, flg_1, date_2, flg_2, date_3, flg_3
FROM ThreeElements
WHERE 
    CASE 
         WHEN date_1 = '2013-11-01' THEN flg_1
         WHEN date_2 = '2013-11-01' THEN flg_2
         WHEN date_3 = '2013-11-01' THEN flg_3
    ELSE NULL END = 'T';
```



- 실행 계획은 아래와 같이 나타난다.
```
ID                          Operation           Name
0                           SELECT STATEMENT    
1                           TABLE FULL SCAN     THREEELEMENTS
```

- 실행계획은 간결하지만 full scan이다.

- 이제 union을 활용해 index를 활성화시켜보자.
```sql
CREATE INDEX IDX_1 ON ThreeElements(date_1, flg_1);
CREATE INDEX IDX_1 ON ThreeElements(date_2, flg_2);
CREATE INDEX IDX_1 ON ThreeElements(date_3, flg_3);

SELECT key,name,
    date_1, flg_1,
    date_2, flg_2,
    date_3, flg_3
FROM ThreeElements
WHERE date_1 = '2013-11-01'
AND     flg_1 = 'T'

UNION

SELECT key,name,
    date_1, flg_1,
    date_2, flg_2,
    date_3, flg_3
FROM ThreeElements
WHERE date_2 = '2013-11-01'
AND     flg_2 = 'T'

UNION

SELECT key,name,
    date_1, flg_1,
    date_2, flg_2,
    date_3, flg_3
FROM ThreeElements
WHERE date_3 = '2013-11-01'
AND     flg_3 = 'T'
```

- 실행계획은 아래와 같다.
- table index scan을 3번 수행하는 모습이다.
```
ID              Operation                           Name    
0               SELECT STATEMENT            
1               SORT UNIQUE                 
2               UNION-ALL
3               TABLE ACCESS BY INDEX ROWID         THREEELEMENTS   4   

4               INDEX RANGE SCAN                    IDX_1
5               TABLE ACCESS BY INDEX ROWID         THREEELEMENTS
6               INDEX RANGE SCAN                    IDX_2
7               TABLE ACCESS BY INDEX ROWID         THREEELEMENTS
8               INDEX RANGE SCAN                    IDX_3
```

- 즉 or문과 union의 차이는 index scan 여러번 vs full scan 한번이다.
- 이 경우, table 크기가 크고 index 설정이 잘 되어있을수록 index scan이 유리하다.
- 다만 이 경우는 date_n, flg_n 값 중 1개만이 제대로 된 값이고 나머지는 (null, null)일 때 제대로 성능을 발휘한다.


## <span style="color:#802548">_CASE WHEN과 GROUP BY_</span>
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
        MAX(CASE WHEN data_type ='C' THEN data_61 ELSE NULL END AS data_6),
FROM NonAggTbl
GROUP BY id;
```

- 아래와 같이 잘 정리되어 나온다.
```
id  |   data_1  |   data_2  |   data_3  |   data_4  |   data_5  |   data_6
Jim |   100     |    10     |    167    |   77      |   90      |    457
Beth|   75      |    0      |   183     |           |   4       |   12
Ken |   78      |   5       |   178     |   346     |   85      |   33
```

- 실행 계획은 아래와 같다.
- HASH GROUP BY를 사용하면 HASH 연산을 하게 되는데, 그 경우 워킹 메모리를 사용한다.
- 즉 워킹 메모리가 부족하면 디스크 I/O가 일어나 느려지게 된다.
- 레코드 수가 적을 땐 워킹 메모리가 부족하지 않겠지만, 많을 때는 속도가 문제가 될 수 있다.
- 혹시라도 워킹 메모리 부족에 대비해 지정한 TEMP 영역도 다 써버리면 SQL 구문이 비정상 종료되어 다운된다.
```
Id          |           Operation       |       Name
0           |       SELECT STATEMENT    |       
1           |        HASH GROUP BY      |
2           |         TABLE ACCESS FULL |       NONaGGTBL
```


## <span style="color:#802548">_집약함수와 GROUP BY_</span>
- 범위를 커버하는 것도 집약함수와 GROUP BY로 가능하다.
```
product_id      low_age     high_age        price
제품1           0           50              2000
제품1           51          100             3000
제품2           0           100             4200
제품3           0           20              500
제품3           31          70              800
제품3           71          100             1000
제품4           0           99              8900
```

- 아래와 같이 하면 하단 0부터 상단 100까지의 가격 범위가 모두 있는 제품만 선택할 수 있다.
```sql
SELECT product_id
FROM PriceByAge
GROUP BY product_id
HAVING SUM(high_age - low_age + 1) = 101;
```


- 다른 예시를 들어보자. 
```
room_nbr            start_date              end_date
101                 2008-02-01              2008-02-06
101                 2008-02-06              2008-02-08
101                 2008-02-10              2008-02-13
202                 2008-02-05              2008-02-08
202                 2008-02-08              2008-02-11
202                 2008-02-11              2008-02-12
303                 2008-02-03              2008-02-17
```


- 다른 예시는 아래와 같이 숙박한날이 10일 이상인 방을 선택하는 것이다.
- 여기서도 GROUP BY와 SUM을 사용하면 된다.
```sql
SELECT room_nbr, SUM(end_dae - start_date) AS working_days
FROM HotelRooms
GROUP BY room_nbr
HAVING SUM(end_date - start_date) >= 10;
```


- GROUP BY는 CASE와 같이 쓰일 수도 있다.
- 어린이, 성인, 노인의 수를 그룹별로 나눠 센다고 해보자.

```sql
SELECT CASE WHEN age < 20 THEN '어린이'
            WHEN age BETWEEN 20 ADN 69 THEN '성인'
            WHEN age >= 70 THEN '노인'
            ELSE NULL END AS age_class,
        COUNT(*)
FROM Persons
GROUP BY WHEN age < 20 THEN '어린이'
         WHEN age BETWEEN 20 AND 69 THEN '성인'
         WHEN age >= 70 THEN '노인'
         ELSE NULL END;
```

```
age_class           |       COUNT(*)
어린이              |           1
성인                |           6
노인                |           2
```
- GROUP BY에 CASE WHEN을 써도 실행계획은 바뀌지 않는다.
- GROUP BY와 집약함수들을 쓸 때 조심해야 하는 건 메모리 용량 초에 따른 저장소 I/O발생이다.
- GROUP BY에 함수를 써도 실행계획은 동일하다. 다만 함수처리에 따른 CPU 연산이 늘어날 뿐이다.
- 아래는 postgreSQL 실행계획이다.
```
HashAggregate
    ->  Seq Scan on persons
```

- GROUP BY에 CASE WHEN과 함수를 사용한 예시를 살펴보자.
- BMI = w / t^2이다. 그걸 sql로 구해보자.

```
이름            키         몸무게
anderson        188        90
Adela           167        55
Bates           158        48
Becky           187        70
Bill            177        120
Chris           175        48
Darwin          160        55
Dawson          182        90
Donald          176        53
```

```
이름            BMI         분류
anderson        25.5        과체중
Adela           19.7        정상
Bates           19.2        정상
Becky           20          정상
Bill            38.3        과체중
Chris           15.7        저체중
Darwin          21.5        정상
Dawson          27.2        과체중
Donald          17.1        저체중
```

- BMI 연산은 weight / POWER(height/100, 2)로 간단하다.
- 그럼 GROUP BY와 select 모두에 구한 BMI에 맞게 CASE를 적어주면 된다.

```sql
SELECT CASE WHEN weight / POWER(height/100, 2) < 18.5 THEN '저체중'
            WHEN weight / POWER(height/100, 2) >= 18.5 THEN '정상'
            WHEN weight / POWER(height/100, 2) < 25     THEN '과체중'
            ELSE NULL END AS bmi,
            COUNT(*)
FROM Persons
GROUP BY  CASE WHEN weight / POWER(height/100, 2) < 18.5 THEN '저체중'
            WHEN weight / POWER(height/100, 2) >= 18.5 THEN '정상'
            WHEN weight / POWER(height/100, 2) < 25     THEN '과체중'
            ELSE NULL END
```

- BMI에 맞게 field명 변경된다. 즉 교환 정의다.
- 그리고 count도 그에 맞게 산출한다.
```
BMI         |   COUNT(*)
저체중      |       2
정상        |       4
과체중      |       3
```


## <span style="color:#802548">_PARTITION BY_</span>
- GROUP BY는 집약을 하지만, PARTITION BY는 집약하지 않는다.
- 다만 정보를 추가 할 뿐이다.

```sql
SELECT name,
        age,
        CASE WHEN age < 20 THEN '어린이'
            WHEN age BETWEEN 20 AND 69 THEN '성인'
            WHEN age >= 70 THEN '노인'
            ELSE NULL END AS age_class,
            RANK() OVER(PARTITION BY CASE WHEN age < 20 THEN '어린이'
                                        WHEN age BETWEEN 20 AND 69 THEN '성인'
                                        WHEN age >= 70 THEN '노인'
                                        ELSE NULL END
                        ORDER BY age) AS age_rank_in_class
FROM Persons
ORDER BY age_class, age_rank_in_class;
```


```
name        |       age     |age_class      |age_rank_in_class
Darwin      |       12      |어린이         |   1
--------------------------------------------------------------
Adela       |       21      |성인           |   1
Dawson      |       25      |성인           |   2
Anderson    |       30      |성인           |   3
Donald      |       30      |성인           |   3
Bill        |       39      |성인           |   5
Becky       |       54      |성인           |   6
--------------------------------------------------------------
Bates       |       87      |노인           |   1
Chris       |       90      |노인           |   2
```

## <span style="color:#802548">_DB는 반복계가 아니다._</span>
- DB에서 반복문을 거의 제공하지 않는데, 그 이유는 DB 자체가 집합의 원리를 구현하려 했기 때문이다.
- 물론 내부적으로는 반복문을 사용한다. 특히 FK 제약에서 CASCADE가 재둎적이다.
- 이들을 갱신할 때, 만약 대량의 데이터를 건든다면 실제로 성능저하가 발생한다. 반복문으로 돌아가기 때문이다.

- 반복문은 짜는 난이도가 매우 낮다. 그러나 성능이 좀 안 좋은 경우가 많다.
- 반복문이 성능을 저하시키는 현상은 자주 찾아볼 수 있다.
  - 반복계가 성능이 좋지 않은 이유는 간단하다. 선형으로 처리 시간이 증가하기 때문이다.
    - 처리 횟수가 늘어날수록 그에 비례해 처리 시간이 늘어나는 구조다.
  - 또 문제인 것은, 전처리와 후처리에서 시간이 오래 걸린다는 점이다.
    - 전처리와 후처리를 먼저 살펴보자.
```
1. SQL 구문을 네트워크로 전송
2. 데이터베이스 연결
3. SQL 구문 파스
4. sql 구문의 실행 계획 생성 또는 평가
-----------------------------전처리 끝----------------------
1. 결과 집합을 네트워크로 전송
-----------------------------후처리 끝---------------------
```
   - 1번과 5번은 WAS와 DB가 같은 본체에 있다면 발생하진 않는다. 
   - 물론 보통 분리되어있긴 하지만, 같은 LAN에 속하기 떄문에 전송 속도는 고속이므로 대부분 오버헤드가 거의 없다.
   - 2번도 연결을 일정 수 확보하는 connection pool에 의해 오버헤드를 감소시키기 떄문에 오버헤드가 크지 않다.
   - 문제는 3번이다. 3번은 느린 경우 1초까지도 걸리므로 오버헤드가 크다.
  - 또 문제인 것은 병렬처리에서 최적화 이득을 보지 못한다는 점이다.
  - 또한 DB의 기술 진보는 대용량 데이터를 다루는 데서 이뤄지는 만큼, 단순 SQL처리는 기술 향상의 혜택을 보기 어렵다.
  - 종합해서 보면, 반복계는 튜닝을 해도 성능이 향상되기 어렵다는 의미다.

- 물론 장점도 있다.
- 아래 쿼리가 바로 반복계의 대표 예시다.
- PL/SQL이라 써놓진 않겠지만 이해하기는 쉽지 않다.
- 그러나 본질적으로는 아래와 같은 쿼리문을 반복 수행하는 것이다.
```sql
SELECT col_a FROM foo WHERE p_key = 1;
```

- 실행계획도 아주 단순하다. 이해하기 쉽다.
- transaction 관리도 매우 쉽다.
- 처리 시간도 매우 정밀하며, 실행 계획도 데이터양에 따라 바뀔 일도 거의 없다.
```
Index Scan using foo_pkey on foo
    Index Cond: (p_key = 1)
```


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

- 식이 복잡하고, 실행계획에서도 손해다.
- 무조건 window function으로 바꿀 수 있다면 그렇게 하자.
```
Id          |           Operation                       |       Name
0           |           SELECT STATEMENT                |       
1           |            SORT AGGREGATE                 |       
2           |             TABLE ACCESS BY INDEX ROWID   |       SALES
3           |               INDEX UNIQUE SCAN           |       PK_SALES
4           |                SORT AGGREGATE             |
5           |                FIRST ROW                  |
6           |                 INDEX RANGE SCAN(MIN, MAX)|       PK_SALES
7           |           TABLE ACCESS FULL               |       SALES
```      

- 아래는 window function을 쓴 예시다.
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

- ROWS BETWEEN 1 PRECEDING AND 1 PRECEDING으로 인해 직전 record를 기준으로 =, +, -가 붙게 된다.
- ROWS BETWEEN 2 PRECEDING AND 2 PRECEDING로 하면 2개 전 record를 기준으로 =, +, -가 붙게 된다.
- 실행계획이 매우 단순해졌다. 하지만 그 외에도 필요없는 INDEX UNIQUE SCAN, INDEX RANGE SCAN이 줄었다.
```
Id          |           Operation           |       Name
0           |           SELECT STATEMENT    |       
1           |           WINDOW SORT         |       
2           |           TABLE ACCESS FULL   |       SALES
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
```
company     |year       |sale               |pre_company        |pre_sale
A           |2002       |50                  |
A           |2003       |52                 |A                  |50
A           |2004       |55                 |A                  |55
B           |2001       |27                 |                   |
B           |2005       |28                 |B                  |27
B           |2009       |30                 |B                  |28
C           |2001       |40                 |                   |
C           |2005       |39                 |C                  |40
C           |2006       |38                 |C                  |39
C           |2010       |35                 |C                  |38
```

- 다른 예시를 들어보자.
- 아래와 같이 사원번호와 년도별 연봉, 직전연봉을 볼 수 있을 것이다.
```sql
SELECT 사원번호,
        시작일자,
        연봉,
        MAX(연봉)
            OVER (PARTITION BY 사원번호
                ORDER BY 시작일자
                ROWS BETWEEN 1 PRECEDING
                        AND  1 PRECEDING) AS 직전연봉
FROM 급여;
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

## <span style="color:#802548">_RECURSIVE_</span>
- 여태까지는 7번의 횟수가 정해져있었지만, n회반복이어야 할 경우는 어떻게 해야 할까?
- db에 저장을 잘하는 게 첫번째다. 아래 테이블을 보자.
```
name            |pcode          |new_pcode(이사갈 곳)
A               |20184          |20143
A               |20143          |20155
A               |20155          |
B               |20455          |21113
B               |21113          |14154
C               |11561          |
```

- 이사한 곳과, 이사갈 곳의 주소 우편번호를 모두 한 record에 담는 것이다.
- 그리고 이전의 pcode를 찾아가면서 계속 우편번호를 탐색한다.
- A씨의 경우, 20184 -> 20143 -> 20155로 이사를 한 셈이다.
- 반면 C씨의 경우, 이사를 하지 않았다.
- 이렇게 포인터 체인 형식으로 이어가는 것을 인접 리스트 모델이라고 부른다.
- 몇번 반복할 지를 모를때는 재귀로 찾아가는 수밖에 없다.
- 아래 sql문을 활용하면 가장 처음의 주소지를 찾아낼 수 있다.
```sql
WITH RECURSIVE Explosion(name, pcode, new_pcode, depth)
AS
(SELECT name,pcode,new_pcode,1
FROM AddressHistory
Where name = 'A'
AND new_code IS NULL

UNION ALL

SELECT Child.name, Child.pcode, Child.new_pcode, depth + 1
FROM Explosion AS Parent, AddressHistory AS Child
WHERE Parent.pcode = Child.new_pcode
AND Parent.name = Child.name)

SELECT name, pcode, new_pcode
FROM Explosion
WHERE depth = (SELECT MAX(depth) FROM Explosion);
```


- oracle join에서 ansi join으로 바꿔주자.
- with로 묶인 query부터 실행된다. UNION ALL 이전의 식이 처음 발화식이다.
- 거기서 explosion table의 pcode를 구해온다.
- 그리고 UNION ALL 아래부터 재귀의 시작이다.
- 재귀는 Parent.pcode = Child.new_pcode and Parent.name = Child.name 조건절이며, 해당 조건절에 맞는 row가 없을 때까지 진행된다.
```sql
WITH Explosion(name, pcode, new_pcode, depth)
AS
(SELECT name,pcode,new_pcode,1
FROM AddressHistory
Where name = 'A'
AND new_code IS NULL /* 처음 검색 시작 조건 */

UNION ALL

SELECT Child.name, Child.pcode, Child.new_pcode, depth + 1
FROM Explosion AS Parent
Inner JOIN AddressHistory AS Child
on Parent.pcode = Child.new_pcode
AND Parent.name = Child.name)

SELECT name, pcode, new_pcode
FROM Explosion
WHERE depth = (SELECT MAX(depth) FROM Explosion);
```

- 진행을 뜯어보면 다음과 같다.
```
A의 new_pcode가 null인 경우는 pcode가 20155일 떄다. 따라서 pcode가 20155일 떄가 처음 시작이다. 또한 그 때의 record가 Explosion에 쌓인다.
그 record는 아래와 같다. Explosion이 Parent다.
name        |pcode       |new_pcode      |depth
A           |20155       |               |1


이제부터 재귀의 시작이다. Parent.pcode = Child.new_pcode and Parent.name = Child.name을 만족하는 것을 모두 찾게 된다.
Parent.pcode가 20155이므로, Child.new_pcode가 20155여야 한다. name도 똑같다. 그렇게 AddressHistory의 record를 가져온다.
name        |pcode       |new_pcode      |depth
A           |20155       |               |1
A           |20143       |20155          |2

다시 재귀한다.
Parent.pcode가 20143이므로, Child.new_pcode가 20143여야 한다. name도 똑같다. 그렇게 AddressHistory의 record를 가져온다.
name        |pcode       |new_pcode      |depth
A           |20155       |               |1
A           |20143       |20155          |2
A           |20184       |20143          |3

다시 재귀한다.
Parent.pcode가 20184이므로, Child.new_pcode가 20184여야 한다. name도 똑같다. 그렇게 AddressHistory의 record를 가져온다.
그러나 이번에는 찾지 못했다. 따라서 재귀가 끝나게 된다.
name        |pcode       |new_pcode      |depth
A           |20155       |               |1
A           |20143       |20155          |2
A           |20184       |20143          |3

재귀테이블에서 가장 depth가 깊은, 재귀 마지막의 record를 가져온다. 
여기선 가장 과거의 record라고 볼 수 있다. 최초의 주소지다.

SELECT name, pcode, new_pcode
FROM Explosion
WHERE depth = (SELECT MAX(depth) FROM Explosion);
```

- 결과값은 아래와 같다.
```
name        |pcode       |new_pcode
A           |20184       |20143
```

- 실행계획은 아래와 같다.
```
Id          |Operation                          |Name
0           |SELECT STATEMENT                   |
1           | TEMP TABLE TRANSFORMATION         |
2           |  LOAD AS SELECT                   | SYS_TEMP_~~~ //inline view- Explosion
3           |   UNION ALL (RECURSIVE WITH)~~    |
4           |    TABLE ACCESS FULL              |AddressHistory //UNION ALL 위의 SELECT 문
5           |    NESTED LOOPS                   |               //UNION ALL 아래의 SELECT 문. AddressHistory or Explosion
6           |     NESTED LOOPS                  |               //UNION ALL 아래의 SELECT 문. AddressHistory or Explosion
7           |     RECURSIVE WITH PUMP           |               //RECURSIVE 수행
8           |     INDEX RANGE SCAN              |IDX_NEW_PCODE  //UNION ALL 아래의 SELECT문 where문에서 (pcode,name)이 PK이고, new_pcode는 index. 그런데 multiple rows라 INDEX RANGE SCAN.
9           |    TABLE ACCESS BY INDEX ROWID    |AddressHistory //
10          | VIEW                              |
11          |   TABLE ACESS FULL                |SYS_TEMP~~~
12          |   SORT AGGREGATE                  |
13          |    VIEW                           |
14          |     TABLE ACCESS FULL             |SYS_TEMP
```
- 다른 예시를 들어보자.
- 데이터는 아래와 같다.
```
| employee_id | employee_name | manager_id |
|-------------|---------------|------------|
| 1           | Alice         |            |
| 2           | Bob           | 1          |
| 3           | Charlie       | 1          |
| 4           | David         | 2          |
| 5           | Eve           | 2          |
```

- RECURSIVE를 꼭 안 붙여도 된다. 재귀 테이블 이름도 Explosion 고정이 아니다. 원하는대로 줄 수 있다.
```sql
WITH RECURSIVE ReportingStructure AS (
    SELECT 
        employee_id,
        employee_name,
        manager_id,
        1 AS depth
    FROM Employees
    WHERE employee_id = 1 -- Starting point: Alice

    UNION ALL

    SELECT 
        e.employee_id,
        e.employee_name,
        e.manager_id,
        rs.depth + 1
    FROM ReportingStructure rs
    JOIN Employees e ON rs.employee_id = e.manager_id
)

SELECT employee_id, employee_name
FROM ReportingStructure
WHERE depth = (SELECT MAX(depth) FROM ReportingStructure);
```
- 결과값은 아래와 같다.
- depth를 계속 내려가면 말단 직원이 나오게 되는 경우다.
```
| employee_id | employee_name | manager_id |depth|
| 4           | David         | 2          |3    |
| 5           | Eve           | 2          |3    |
```

- 진행을 뜯어보면 다음과 같다.
```
employee_id = 1인 경우는 employee_name이 Alice일 떄다. 그 때의 record가 Explosion에 쌓인다.
그 record는 아래와 같다. Explosion이 Parent다.
| employee_id | employee_name | manager_id |depth|
|-------------|---------------|------------|-----|
| 1           | Alice         |            |1    |



이제부터 재귀의 시작이다. rs.employee_id = e.manager_id을 만족하는 것을 모두 찾게 된다.
rs.employee_id는 1이므로, Child.manager_id가 1이여야 한다. 그러한 Employees의 record를 가져온다.
| employee_id | employee_name | manager_id |depth|
|-------------|---------------|------------|-----|
| 1           | Alice         |            |1    |
| 2           | Bob           | 1          |2    |
| 3           | Charlie       | 1          |2    |

다시 재귀한다.
rs.employee_id는 2이므로, Child.manager_id가 2이여야 한다. 그러한 Employees의 record를 가져온다.
| employee_id | employee_name | manager_id |depth|
|-------------|---------------|------------|-----|
| 1           | Alice         |            |1    |
| 2           | Bob           | 1          |2    |
| 3           | Charlie       | 1          |2    |
| 4           | David         | 2          |3    |
| 5           | Eve           | 2          |3    |

다시 재귀한다.
rs.employee_id는 3이므로, Child.manager_id가 3이여야 한다. 그러한 Employees의 record를 가져온다.
그러나 이번에는 찾지 못했다. 따라서 재귀가 끝나게 된다.
| employee_id | employee_name | manager_id |depth|
|-------------|---------------|------------|-----|
| 1           | Alice         |            |1    |
| 2           | Bob           | 1          |2    |
| 3           | Charlie       | 1          |2    |
| 4           | David         | 2          |3    |
| 5           | Eve           | 2          |3    |

재귀테이블에서 가장 depth가 깊은, 재귀 마지막의 record를 가져온다. 
여기선 가장 직급이 낮은 사원이다.

SELECT employee_id, employee_name
FROM ReportingStructure
WHERE depth = (SELECT MAX(depth) FROM ReportingStructure);
```

## <span style="color:#802548">_중첩 집합 모델_</span>
- 재귀의 다른 방식으로는 좌표와 같은 방식을 사용하는 것이다.
- 다만 이 방식은 언뜻보면 이해할 수 없는 column을 넣는 것이기 때문에 활용이 쉽지 않다.
- 아래와 같이 범위 안에 중첩되게끔 중첩을 나타내는 데이터를 넣어준다.
```
name            |pcode           |lft             |rgt
A               |20184           |12              |15
A               |20143           |9               |18  
A               |20155           |0               |27
B               |20455           |12              |15
B               |21113           |9               |18
C               |11561           |12              |15
```

- 그럼 가장 외부의 원을 찾게 된다면, 그게 가장 오래된 최초의 주소지가 된다.
```sql
SELECT name,pcode
FROM AddressHistory AH1
WHERE name = 'A'
AND NOT EXISTS(
    SELECT *
    FROM    AddressHistory AH2
    WHERE AH.name = 'A'
    AND AH1.lft > AH2.lft
);
```

- 실행계획은 아래와 같다.
```
Id          |Operation                        |Name
0           |SELECT STATEMENT  
1           | NESTED LOOPS ANTI               |                // ANTI가 붙은 이유: NOT EXISTS라서. EXISTS보다 검색속도가 더 빠름. EXISTS는 찾을 때까지 해야하지만, not EXISTS는 일치하지 않는 것 찾는 순간 검색 중단
2           |  TABLE ACCESS BY INDEX ROWID    |AddressHistory  // main query의 select문
3           |   INDEX RANGE SCAN              |PK_NAME_PCODE2  // select field에 PK들을 명시하여 index scan으로 진행. 다만 multiple row라 range scan.
4           |  TABLE ACCESS BY INDEX ROWID    |AddressHistory  // sub query의 select문
5           |   INDEX RANGE SCAN              |UQ_NAME_LFT     // sub query의 where문에서 NOT EXISTS lft 조건 filtering
```
- 만약 가장 바깥의 원을 최신으로 나오게 데이터를 바꾼다면?
```
name            |pcode           |lft             |rgt
A               |20184           |0               |27
A               |20143           |9               |18  
A               |20155           |12              |15
B               |20455           |0               |27
B               |21113           |9               |18
C               |11561           |0               |27
```

- 이 떄는 가장 오래된 최초의 주소지를 찾는 sql을 <로 바꿔주면 된다.
```sql
SELECT name,pcode
FROM AddressHistory AH1
WHERE name = 'A'
AND NOT EXISTS(
    SELECT *
    FROM    AddressHistory AH2
    WHERE AH.name = 'A'
    AND AH1.lft < AH2.lft
);
```


## <span style="color:#802548">_결합_</span>
- CROSS JOIN은 거의 쓸일이 없다. 쓰였다면, 그건 실수일 것이다.
- 아래는 실수로 CROSS JOIN을 건 것이다. 결합조건을 쓰지 않았다.
- ANSI join을 썼다면 이럴 일이 없었을 것이다.
```sql
SELECT *
FROM Employees, Departments
```

- 떄로 3가지 이상 table을 조합할 때도 cross join이 예기치 않게 끼어들 때가 있다.
- 아래와 같이 A-B, A-C 간의 결합조건은 정했지만, B-C는 정하지 않았기 때문이다.
```sql
SELECT A.col_a, B.col_b, C.col_c
    FROM Table_A A
    INNER JOIN TABLE_B B
        ON A.col_a = B.col_b
    INNER JOIN TABLE_C C
        ON A.col_A = C.col_c;
```

- 그럼 optimizer에 따라 cross join이 수행되기도 한다.
- MERGE JOIN CARTESIAN이 바로 cross join이다.
```
Id     |Operation                    |Name
0       |SELECT STATEMENT            |
1       | HASH JOIN                  |
2       |  MERGE JOIN CARTESIAN      |
3       |   TABLE ACCESS FULL        |TABLE_B
4       |   BUFFER SORT              |              //disk 대신 memoery buffer에서 가져옴. 성능 향상. 디스크 I/O 안거치기 때문.
5       |    TABLE ACCESS FULL       |TABLE_C
6       |   TABLE ACCESS FULL        |TABLE_A   
```
- mysql은 아래와 같다. join의 실행계획을 보아 세 테이블이 모두 type이 ALL인데,  이러면 cross join을 의미한다.
```
Id | select_type | table | type        | possible_keys | key  | key_len | ref  | rows  | Extra
-----------------------------------------------------------------------------------------
1  | SIMPLE      | A     | ALL         | NULL          | NULL | NULL    | NULL | 1000  | 
1  | SIMPLE      | B     | ALL         | NULL          | NULL | NULL    | NULL | 10000 | 
1  | SIMPLE      | C     | ALL         | NULL          | NULL | NULL    | NULL | 100000| Using join buffer
```
- table 끼리 결합이 양이 많지않으면 상관없지만, 큰 테이블끼리 cross join을 하면 굉장히 오래 걸린다.
- 그럴 때는 B와 C 간의 무의미한 조건절을 넣어서 실행계획을 변경할 수 있다.
- 다만 결과에 영향이 없을 때만 가능한 기법이다.
```sql
SELECT A.col_a, B.col_b, C.col_c
    FROM Table_A A
    INNER JOIN TABLE_B B
        ON A.col_a = B.col_b
    INNER JOIN TABLE_C C
        ON A.col_A = C.col_c
    AND C.col_c = B.col_b; /**B와 C의 무의미한 결합 조건 추가 */
```

- 그럼 cross join이 사라진다.
```
Id      |Operation                    |Name
0       |SELECT STATEMENT            |
1       | NESTED LOOPS               |
2       | NESTED LOOPS               |
3       |  TABLE ACCESS FULL         |TABLE_A
4       |  TABLE ACCESS FULL         |TABLE_B
6       | TABLE ACCESS FULL          |TABLE_C  
```


- INNER JOIN은 자주 쓰인다.
- INNER라고 불리는 이유는 resultSet이 CROSS JOIN의 일부이기 때문이다.
- 물론 INNER JOIN은 CROSS JOIN을 해놓고 결과를 축소하는 바보짓을 하지 않는다.
- 모든 RDBMS는 INNER JOIN 시 CROSS JOIN이 아니라 결합 대상을 최대한 축소하여 작동한다.
- 우선 데이터가 아래와 같이 구성되었다고 해보자.
```
Employees
emp_id          |emp_name           |dept_id
001             |하린               |10
002             |한미루             |11
003             |사라               |11
004             |중민               |12
005             |웅식               |12
006             |주아               |12

//

Department
dept_id         |dept_name          
10              |총무
11              |인사
12              |개발
13              |영업
```
- CROSS JOIN을 다 해놓고 그 결과값을 축소하는 것은 매우 비효율적이기 때문이다.
- INNER JOIN은 ANSI JOIN으로 아래와 같이 쓴다.
```sql
SELECT E.emp_id, E.emp_name, E.dept_id, D.dept_name
FROM Employees E
INNER JOIN Departments D
on E.dept_id = D.dept_id
```

```
emp_id          |emp_name           |dept_id            |dept_name
001             |하린               |10                 |총무
002             |한미루             |11                 |인사
003             |사라               |11                 |인사
004             |중민               |12                 |개발
005             |웅식               |12                 |개발
006             |주아               |12                 |개발
```

- 서브쿼리를 쓰면, 아래와 같은 상관서브쿼리로 join을 대체할 수 있다.
- 다만 select에서 서브쿼리를 쓰는 것은 join보다 비용이 꽤나 높은 일이다.
- record 수만큼 상관 서브쿼리를 실행해야 하기 때문이다.
- 즉 table을 여러 번 scan해서 성능이 떨어진다.
```sql
SELECT E.emp_id, E.emp_name, E.dept_id,
    (SELECT D.dept_name
        FROM Departments D
        WHERE E.dept_id = D.dept_id) AS dept_name
FROM Employees E;
```

- LEFT JOIN은 INNER JOIN과는 다르다.
- dept_id가 없어도 강제로 끌고와서 Record로 만든다.
- oracle에서는 NULL이 아니라 그냥 빈칸으로 뜬다.
```
emp_id          |emp_name           |dept_id            |dept_name
001             |하린               |10                 |총무
002             |한미루             |11                 |인사
003             |사라               |11                 |인사
004             |중민               |12                 |개발
005             |웅식               |12                 |개발
006             |주아               |12                 |개발
NULL            |NULL               |13                 |영업
```

## <span style="color:#802548">_결합의 알고리즘_</span>
- 결합 알고리즘으로는 3가지가 있다.
  - Nest Loops
  - Hash
  - Sort merge
- 데이터 크기, 결합 키의 분산에 따라 방식이 선택되는데, 보통 NL이 채택된다.
- 그 다음으로 Hash고, 그 다음으로는 Sort merge다.

- 우선 NL부터 다뤄보자.
- SQL에서는 결합은 기본적으로 두 테이블의 결합이다.
- driving table(outer table)에서 driven table(inner table) 하나하나 record를 scan해서 결합 조건에 맞으면 return한다.
- 결합 조건에 맞을 때까지 driven table scan이 필요하다는 의미다. 맞는 record를 찾으면 해당 driving table의 record에 대해서는 반복이 끝난다.
- 그럼 driving table의 다음 record에 대해서 또 같은 방식으로 진행된다.
- 이같이 driving table의 모든 record에 대해서 대응되는 driven table의 record를 찾을 때까지 반복한다.
- 따라서 NL은 record 수가 많을 수록 느려진다.
```
접근 레코드수: A table Record X B table Record
```
- driving table을 뭘로 정하든 어차피 R(A) x R(B)면 실행시간이 같을 거라 생각할 수 있으나 그렇지 않다.
- 그러나 driving table(outer table)의 결합 키 필드에 인덱스가 있으면, driving table을 작게 하는 게 좋다.
- 그 이유는 index를 통해 table access를 건너뛰면 driven table의 record에 대한 scan행위를 줄일 수 있기 때문이다.
- 최적의 경우는 driving table의 index와 driven table의 index가 1:1 대응하는 경우다. unique index가 그런 경우다.
- 이 경우 drivent table이 크면 클수록 index 사용에 따른 반복 생략 효과가 커진다.
```
최적의경우 -> 접근 레코드수: A table Record X 2
해당 레코드를 drivent table의 index만으로 찾을 수 있는 경우
```

- 아까 다뤘던 inner join 쿼리를 다시 살펴보자.
- INDEX UNIQUE SCAN은 해당 WHERE 혹은 ON절에서 비교하는 결과가 유일할 때 뜬다.
- 즉, PK 혹은 UK로만 비교했을 때 INDEX UNIQUE SCAN이 뜨게 된다.
- 그 외에 driving table의 결합키에 대응되는 driven table의 record가 여러개인 경우, INDEX가 1:1 대응이 아니라 내부 반복이 필요하다. 따라서 INDEX RANGE SCAN이 된다.
```sql
SELECT E.emp_id, E.emp_name, E.dept_id, D.dept_name
FROM Employees E INNER JOIN Departments D
ON E.dept_id = D.dept_id;
```

```
ID      |Operation                      |Name
0       |SELECT STATEMENT               |
1       | NESTED LOOPS                  |
2       |  TABLE ACCESS FULL            |EMPLOYEES
3       |   TABLE ACCESS BY INDEX ROWID |DEPARTMENTS
4       |    INDEX UNIQUE SCAN          |PK_DEP     
```

- INDEX RANGE SCAN이 INDEX UNIQUE SCAN보다 안 좋은 이유는, 더 반복을 많이 하기 때문이다.
- INDEX UNIQUE SCAN은 하나의 RECORD만 걸리기 때문에 늘 최적의 경우의 수가 된다.
  - 즉, driving table Record X 2가 반복의 횟수다.
- 반면에 INDEX RANGE SCAN은 하나가 아니라 multiple row가 걸리게 된다.
  - 즉, driving table Record X2 < 반복의 횟수 <  R(A) X R(B) Record가 된다.
- 따라서 결합key로 쓸 column의 cardinality가 굉장히 중요하다.
- 데이터가 고르게 분포되어 있을수록 성능에 좋은 영향을 미친다.
## <span style="color:#802548">_driving table의 역설_</span>
- driving table을 작게 만드는 게 통상적으로 좋다.
- 하지만, on절로 driven table을 조회하려 할때, 많은 record가 걸리게 된다면?
  - 그 떄는 역설적으로 driving table을 큰 테이블로 선택하는 게 나을 수 있다.
  - 따라서 선택률에 따라 driving table을 큰 테이블로 선택하는 게 나을 수도 있다.

- 두 번째 해결법은 Hash다.
- 해시결합은 작은 테이블을 스캔하고, 결합 키에 해시 함수를 적용해 해시값으로 반환한다.
- 이어서 큰 테이블을 스캔하고, 결합 키가 해시값에 존재하는 지 확인하여 결합한다.
- 작은 테이블에서 해시 테이블을 만드는 이유는 해시 테이블은 워킹 메모리에 저장되므로 크면 디스크 I/O로 넘어가버리기 떄문이다.
- 물론 Hash가 사용될 정도면 큰 테이블과 작은 테이블의 크기 차이가 심하지 않다.
```
Departments
dept_id
10
20
30
40
------------------작은 table 스캔-------------------
temp Hash table
dept_id|        hash_dept
10     |        1age35
20      |       2014ss
30      |       5ihj14
40      |       51515fgs
-----------------해시테이블 생성----------------
Employees
dept_id
10
10
20
20
--------------큰 table과 매칭-------------
```
- HASH JOIN은 해시 테이블을 따로 만들기 때문에 메모리 소모가 NL에 비해 심하다.
- 하지만 drivent table에 index가 없을 때, 선택율이 너무 높을 때, driving table 또한 작지 않을 때 사용하면 좋다.
- 웹의 경우, 실시간처리인 OLTP방식이라 Hash를 사용하기에는 메모리가 부족할 가능성이 높습니다. 그 경우 디스크 I/O가 되어버려 느려진다.
- 따라서 대용량을 다룰 야간 배치에 hash join을 자주 사용하게 된다.
```
SELECT /*+ HASH_JOIN(t1 t2) */ *
FROM table1 t1
JOIN table2 t2 ON t1.column = t2.column;
```
- 위와 같이 hint를 통해 hash join을 강제할 수도 있다. 
- 다만 hash join은 동치결합에서만 사용이 가능하다는 단점이 존재한다.
- 또한 hash 결합은 보통 full scan이기 때문에 오래걸린다.
``` 
Id|                 |Operation                   |Name
0 |                 |SELECT STATEMENT            |
1 |                 | HASH JOIN                  |
2 |                 |  TABLE ACCESS FULL         |DEPARTMENTS
3 |                 |  TABLE ACCESS FULL         |EMPLOYEES
```   

- Hash 마저 느리면 Sort merge를 사용하기도 한다.
- 우선 대상 테이블을 모두 정렬시킨다. 따라서 Hash보다 메모리 소모가 높다. 
  - 워킹 메모리가 부족하면 디스크 I/O가 일어난다.
- 거의 사용할 일이 없다.

- 소규모 - 소규모인 경우에는 알고리즘에 따른 성능차가 없다.
- 소규모 - 대규모인 경우에는 소규모 테이블을 driving table로 사용하는 NL을 써야한다.
  - join table(inner table)의 결합 field에 index도 만들어줘야 한다.
  - join table(drivent table)에 결합 레코드가 많다면 구동 테이블과 바꾸거나, Hash join을 쓰자
- 대규모 - 대규모는 Hash join을 써야한다.
- 물론 Mysql은 NL밖에 없어서 선택지가 없다.
- 결합은 사실 성능 변동을 유발하는 주된 원인이다. 자제하는 게 제일 좋다.
  - 대안으로 window 함수를 쓸 수 있다면 사용해보자.
- Exists와 Not Exists는 특수한 결합이 사용된다.
```sql
SELECT dept_id, dept_name
FROM    Departments D
WHERE EXISTS (SELECT *
                FROM Employees E
                WHERE E.dept_id = D.dept_id);
```

- NL에 SEMI가 붙어있다.
  - 해당 알고리즘은 결과에 driving table의 데이터만 포함된다.
  - 1개의 record는 1개의 결과만 생성한다.
  - 조건에 맞는 table을 발견하는 즉시 탐색을 중단한다. 따라서 일반 join보다 성능이 좋다.
```
Id|                 |Operation                   |Name
0 |                 |SELECT STATEMENT            |
1 |                 | NESTED LOOPS SEMI          |
2 |                 |  TABLE ACCESS FULL         |DEPARTMENTS
3 |                 |   INDEX RANGE SCAN         |IDX_DEPT_ID
```

- NL에 ANTI가 붙어있다.
  - 해당 알고리즘은 결과에 driving table의 데이터는 제외된다.
  - 1개의 record는 1개의 결과만 생성한다.
  - 조건에 맞는 table을 발견하는 즉시 탐색을 중단한다. 따라서 일반 join보다 성능이 좋다.
```sql
SELECT dept_id, dept_name
FROM Departments D
WHERE NOT EXISTS (SELECT *
                    FROM Employees E
                    WHERE E.dept_id = D.dept_id);
```                    

```
Id|                 |Operation                   |Name
0 |                 |SELECT STATEMENT            |
1 |                 | NESTED LOOPS ANTI          |
2 |                 |  TABLE ACCESS FULL         |DEPARTMENTS
3 |                 |   INDEX RANGE SCAN         |IDX_DEPT_ID
```

- EXISTS와 IN은 결과가 동일한 경우 실행계획이 동일하다.
- 그러나 NOT EXISTS와 NOT IN은 NOT EXISTS가 성능이 더 좋다.
- EXISTS와 INNER JOIN은 INDEX UNIQUE SCAN이라면 실행시간은 동일하다.
- 그런데 INNER JOIN이면 읽기가 쉬우니 이 경우엔 EXISTS가 아닌 JOIN을 쓰는게 좋다.
- 반면에 index 처리가 잘 되어있지 않다면 EXISTS를 쓰는 게 성능이 더 좋다.


## <span style="color:#802548">_subquery_</span>
- subquery는 단점이 많다.
  - subquery는 실제 데이터를 저장하지 않는다. 따라서 서브쿼리에 접근할 때마다 SELECT문이 실행된다.
  - 연산 결과를 저장하기 위해 메모리 혹은 디스크를 사용해야 하는데, 디스크를 사용하게 되면 속도가 느려진다.
  - 제약 혹은 인덱스가 적용되지 않기 때문에 복잡한 로직 혹은 데이터가 큰 resultset에 쥐약이다.
- 그럼에도 자주 쓰이는 이유는, 간편하고 직관적이기 때문이다.

- 고객의 구입 명세 정보를 기록하는 테이블이 있다. 순번 필드는 구입 시기가 오래될수록 작다.
- 이 때 고객별 최소 순번 레코드를 구한다고 해보자.
- 즉, 고객들이 구매했던 가장 오래된 구입 이력을 찾는 것이다.
- 데이터를 아래와 같다.
```
cust_id(PK)         |seq(PK)            |price
A                   |1              |500
A                   |2              |1000
A                   |3              |700
B                   |5              |100
B                   |6              |5000
B                   |7              |300
B                   |9              |200
B                   |12             |1000
C                   |10             |600
C                   |20             |100
C                   |45             |200
C                   |70             |50
D                   |3              |2000
```

- 그냥 서브쿼리를 쓴다면 아래와 같이 inline view로 결합을 하게 된다.
- 해당 서브쿼리가 좋지 않은 이유는, SELECT를 구매 table에서 진행했지만, 서브쿼리를 만나서 한번 더 구매 table을 SCAN해야하기 때문이다.
- 똑같은 TABLE을 두 번 scan하게 되는 것이다.
```sql
SELECT R1.cust_id, R1.seq, R1.price
    FROM Receipts R1
        INNER JOIN
            (SELECT cust_id, MIN(seq) AS min_seq
                FROM Receipts
                GROUP BY cust_id) R2
        ON R1.cust_id = R2.cust_id
    AND R1.seq = R2.min_seq;
```
```
1	PRIMARY	<derived2>		ALL					5	100.00	Using where
1	PRIMARY	R1		eq_ref	PRIMARY	PRIMARY	606	R2.cust_id,R2.min_seq	1	100.00	Using index
2	DERIVED	구매		range	PRIMARY	PRIMARY	602		5	100.00	Using index for group-by
```
- 상관서브쿼리를 쓴다면 아래와 같이 쓸 수 있다.
- 해당 서브쿼리가 좋지 않은 이유는, SELECT를 구매 table에서 진행했지만, 서브쿼리를 만나서 한번 더 구매 table을 SCAN해야하기 때문이다.
- 똑같은 TABLE을 두 번 scan하게 되는 것이다.
- 또한 상관서브쿼리는 결합과 마찬가지로 데이터양에 따라 실행계획에 변동성이 높다.
```sql
select cust_id, seq, price
	FROM 구매 R1
    WHERE seq = (SELECT MIN(seq)
					FROM 구매 R2
                    WHERE R1.cust_id = R2.cust_id);
```     
```
1	PRIMARY	R1		ALL					16	100.00	Using where
2	DEPENDENT SUBQUERY	R2		ref	PRIMARY	PRIMARY	602	practice.R1.cust_id	4	100.00	Using index
```
- 따라서 상관서브쿼리가 아닌, window function을 쓰는 서브쿼리로 바꿔준다.
- 실행계획도 안정적으로 바꾸고, table scan을 1회로 줄일 수 있다.
```sql
SELECT cust_id, seq, price
FROM (SELECT cust_id, seq,price,
		ROW_NUMBER() 
			OVER(PARTITION BY cust_id
					ORDER BY seq) AS row_seq
		FROM 구매 ) WORK
WHERE WORK.row_seq = 1;
```
```
1	PRIMARY	<derived2>		ref	<auto_key0>	<auto_key0>	8	const	1	100.00	 //이건 table scan이 아니다. DERIVED를 이용했기 때문이다.
2	DERIVED	구매		ALL					16	100.00	Using filesort
```

- 나도 나름 답을 찾아보려고 했다. 나는 group by와 집약함수를 이용하려 했다.
- 그런데 안타깝게도 group by절에서는 집약함수 + window function over() 조합을 쓸 수가 없었다.
- min을 그냥 min()으로 쓰면 집약함수라서 group별로 센다.
- 하지만 min over()를 쓰면 window 함수라서 group별이 아니라 전체 record를 대상으로 order by한다.
- 따라서 group by와 window function은 서로 상충되는 관계에 있다. group by는 window function과 같이 쓰면 안되는 것이었다.
```sql
SELECT cust_id,
min(seq) over(order by seq desc)
from 구매
group by cust_id
```

- 그래서 아래와 같이 바꿔봤다.
- 그러나 내가 원하는 결과는 나오지 않았다.
- B가 5가 떠야했는데 자꾸 12가 떴다.
```sql
SELECT cust_id, 
min(seq)
from 구매
group by cust_id;
```
```
A	1
B	12
C	10
D	3
```

- 원인을 찾아보니 seq를 내가 varchar로 정했기 때문이었다.
- 문자열은 첫문자열부터 비교한다고 한다. 12와 5를 비교하면 1과 5를 비교하는 셈이 되는 것이었다. 따라서 12가 5보다 더 작은 것이었다.
- int로 바꾸고 다시 쿼리를 실행해보니 잘 된다.
```sql
SELECT cust_id, 
min(seq)
from 구매
group by cust_id;
```
```
A	1
B	5
C	10
D	3
```
- 실행계획은 아래와 같이 단순하다.
```
1	SIMPLE	구매		range	PRIMARY	PRIMARY	602		5	100.00	Using index for group-by
```
- min을 사용하고 group by를 쓴 게 실행계획은 가장 간단하다. group by로 index scan을 한번 한다.
- 하지만 window functiond을 사용한 예제는 group by없이 table full scan을 한번 한다. 
- 보통 크기가 클 수록 index scan이 중요하다. group by를 하더라도 말이다.
- 따라서 첫 번째가 좋다고 한다. 다만 디스크 I/O가 벌어지면 무조건 window function으로 바꿔야 할 것으로 보인다.
```sql
SELECT cust_id, 
min(seq)
from 구매
group by cust_id;
```


- 서브쿼리의 단점을 보여주는 다른 예시를 살펴보자.
- seq의 최솟값과 최댓값 사이의 가격 차이를 나타내보자.
- 과거에 비해 얼마나 돈을 많이 쓰는지, 적게 쓰는지 보기 위한 것이다.
```sql
SELECT TMP_MIN.cust_id,
        TMP_MIN.price - TMP_MAX.price AS diff
FROM (SELECT R1.cust_id, R1.seq, R1.price
        FROM 구매 R1
            INNER JOIN
                (SELECT cust_id, MIN(seq) AS min_seq
                    FROM 구매
                    GROUP BY cust_id) R2
            ON R1.cust_id = R2.cust_id
            AND R1.seq    = R2.min_seq) TMP_MIN
        INNER JOIN
            (SELECT R3.cust_id, R3.seq, R3.price
                FROM 구매 R3
                    INNER JOIN
                        (SELECT cust_id, MAX(seq) AS min_seq
                            FROM 구매
                            GROUP BY cust_id) R4
                    ON R3.cust_id = R4.cust_id
                    AND R3.seq    = R4.min_seq) TMP_MAX
        ON TMP_MIN.cust_id = TMP_MAX.cust_id;
```

```
A	-200
B	-900
C	550
D	0
```

```
1	PRIMARY	<derived3>		ALL					5	100.00	Using where
1	PRIMARY	R1		eq_ref	PRIMARY	PRIMARY	606	R2.cust_id,R2.min_seq	1	100.00	
1	PRIMARY	<derived5>		ref	<auto_key0>	<auto_key0>	602	R2.cust_id	2	100.00	Using where; Using index
1	PRIMARY	R3		eq_ref	PRIMARY	PRIMARY	606	R2.cust_id,R4.min_seq	1	100.00	
5	DERIVED	구매		range	PRIMARY	PRIMARY	602		5	100.00	Using index for group-by
3	DERIVED	구매		range	PRIMARY	PRIMARY	602		5	100.00	Using index for group-by
```

- 실행계획이 매우 길다. 딱봐도 불안정한 실행계획이다.
- 테이블 접근과 결합을 최대한 줄여보자. 
- 집약함수와 CASE WHEN을 사용한다.
```sql
SELECT cust_id,
        SUM(CASE WHEN min_seq = 1 THEN price ELSE 0 END)
        - SUM(CASE WHEN max_seq = 1 THEN price ELSE 0 END) AS diff
FROM (SELECT cust_id, price,
            ROW_NUMBER() OVER(PARTITION BY cust_id
                                ORDER BY seq) AS min_seq,
            ROW_NUMBER() OVER(PARTITION BY cust_id
                                ORDER BY seq DESC) AS max_seq
        FROM 구매 ) WORK
WHERE WORK.min_seq = 1
    OR WORK.max_seq = 1
GROUP BY cust_id;
```

```
1	PRIMARY	<derived2>		ALL					13	19.00	Using where; Using temporary            //scan이 아님. primary는 outer query라는 뜻. DERIVED가 수행하는 full scan이 저장된 결과가 memory 혹은 TEMP 영역(디스크)에 보관되는데, 거기서 정보를 가져옴. 
                                                                                                    //당연히 TEMP에서 가져오면 디스크 I/O라서 속도가 확 갑자기 느려짐
2	DERIVED	구매		ALL					13	100.00	Using filesort                              //여기서 full scan이 일어남. DERIVED는 sub query라는 뜻.
```

## <span style="color:#802548">_subquery를 써야할 떄_</span>
- subquery를 써야할 때는 결합 대상 record를 줄여야할 때다.
- driving과 driven table 모두 결합 record를 줄이는 게 좋다는 의미다.
- 아래와 같은 테이블이 있다.
```
회사
cd_cd(회사코드)         |district(지역)
001                     |A
002                     |B
003                     |C
004                     |D
```
```
종업원
cd_cd              |shop_id          |emp_nbr            |main_flg
001                |1                |300                |Y
001                |2                |400                |N
001                |3                |250                |Y
002                |1                |100                |Y
002                |2                |20                 |N
003                |1                |400                |Y
003                |2                |500                |Y
003                |3                |300                |N
003                |4                |200                |Y
004                |1                |999                |Y
```
- 주요사업소의 종업원 합계를 회사별로 보고 싶다고 해보자.
```
cd_cd           |district           |sum_emp
001             |A                  |550
002             |B                  |100
003             |C                  |1100
004             |D                  |999
```

```
cd_cd              |shop_id          |emp_nbr            |main_flg
001                |1                |300                |Y
001                |3                |250                |Y
002                |1                |100                |Y
003                |1                |400                |Y
003                |2                |500                |Y
003                |4                |200                |Y
004                |1                |999                |Y
```

- 그럼 아래와 같이 쿼리문을 짜게 된다.
- 결합을 하고, 집약을 한다.
- 결합 비용이 비싼 대신, 집약 비용이 싸다.
- 메모리가 충분할 때는 성능이 떨어진다.
- 결합해야 할 record가 많기 때문이다. 회사 테이블 레코드 4개, 사업소 테이블 레코드 10개다.
```sql
SELECT A.cd_cd, district, sum(emp_nbr)
FROM 회사 A
INNER JOIN 종업원 B
ON A.cd_cd = B.cd_cd
WHERE main_flg = 'Y'
GROUP BY A.cd_cd
```



- 그럼 이번에는 집약을 먼저 해보자.
- 집약을 먼저하고, 결합을 한다.
- 결합 비용이 싼 대신, 집약 비용이 비싸다.
- 메모리가 부족해 Disk I/O로 넘어가면 성능이 떨어진다.
- 메모리가 충분하다면 성능이 좋다.
- 결합해야 할 record가 적기 때문이다. 회사 테이블 레코드 4개, 사업소 테이블 레코드 4개다.
```sql
SELECT A.cd_cd, A.district, sum_emp
FROM 회사 A
INNER JOIN
    (SELECT cd_cd,
            SUM(emp_nbr) as sum_emp
        FROM 종업원
        WHERE main_flg = 'Y'
        GROUP BY cd_cd) CSUM
ON A.cd_cd = CSUM.cd_cd;
```

- 결과는 똑같다.
- 그런데 실행계획은 다르다.
- 서브쿼리에서 먼저 결합한 쪽이 우수할 가능성이 높다.
- 그 이유는 결합할 record가 4개로 줄었기 때문이다.
- 물론 집약 비용이 두번째가 더 높다. subquery 안에서의 group by는 서브쿼리가 아닐때의 group by에 비해 추가 단계를 거친다.
- 서브쿼리의 데이터를 메모리나 디스크 등에 저장한다던가 하는 형태로 말이다.
- 때로는 partition이나, index를 타기위해, 결합이 record 수를 더 줄일 때 view merging이 일어난다.
- 때로 view merging(subquery는 inline view다)이 일어나면 subquery를 써도 결합처럼 실행계획이 작동한다.
- 아래와 같이 sql을 짰어도,
```sql
SELECT A.cd_cd, A.district, sum_emp
FROM 회사 A
INNER JOIN
    (SELECT cd_cd,
            SUM(emp_nbr) as sum_emp
        FROM 종업원
        WHERE main_flg = 'Y'
        GROUP BY cd_cd) CSUM
ON A.cd_cd = CSUM.cd_cd;
```

- 실제 실행계획은 아래처럼 서브쿼리가 풀린 채로 작동하게 된다.
- record가 늘어나는 대신 index의 효과를 받을 수 있을 것이다.
- 무엇이 나은지는 실제 실행을 해봐야 안다.
```sql
SELECT A.cd_cd, district, sum(emp_nbr)
FROM 회사 A
INNER JOIN 종업원 B
ON A.cd_cd = B.cd_cd
WHERE main_flg = 'Y'
GROUP BY A.cd_cd
```


## <span style="color:#802548">_record에 순번 붙이기_</span>
- 기본키가 한개의 필드일 경우, record에 순서를 붙이는 것은 간단하다.
- student_id가 PK라고 해보자.
```sql
SELECT student_id, ROW_NUMBER() OVER(ORDER BY student_id) AS seq
FROM Weights;
```

- mysql에서는 ROW_NUMBER()가 없다.
- 아래와 같이 서브쿼리를 사용할 수밖에 없다.
- 다만, window function이 성능 상유리하다.
- index only scan에다가 scan이 1회밖에 없기 때문이다.
```sql
SELECT student_id,
    (SELECT COUNT(*)
        FROM Weights W2
            WHERE W2.student_id <= W1.student_id) AS seq
    FROM Weights W1;
```

```
student_id          |seq
A100                |1
A101                |2
A124                |3
B343                |4
B346                |5
C345                |6
C563                |7
```

- 기본키가 여러개의 필드 결합으로 이뤄진 경우, orderby에 둘 모두 적고, select에도 둘 모두 적는다.
```sql
SELECT class, student_id,
    ROW_NUMBER() OVER(ORDER BY class, student_id) AS seq
FROM Weights2;
```

- 그럼 이때 subquery는 어떻게 될까?
```sql
SELECT class, student_id,
    (SELECT COUNT(*)
        FROM Weights W2
            WHERE (W2.class, W2.student_id) <= (W1.class, W1.student_id) ) AS seq
    FROM Weights W1;
```

```
class               |student_id         |seq
A                   |100                |1
A                   |101                |2
A                   |124                |3
B                   |343                |4
B                   |346                |5
C                   |345                |6
C                   |563                |7
```

- 그룹마다 순번을 붙이는 경우는 어떻게 될까?
```sql
SELECT class, student_id,
    ROW_NUMBER() OVER(PARTITION BY class ORDER BY student_id) AS seq
FROM Weigths2;
```

- subquery로는 아래와 같다.
- W2.class = W1.class로 조건이 바뀌었다.
```sql
SELECT class, student_id,
    (SELECT COUNT(*)
        FROM Weights2 W2
        WHERE W2.class = W1.class
        AND W2.student_id <= W1.student_id) AS seq
    FROM Weights2 W1;
```

```
class               |student_id         |seq
A                   |100                |1
A                   |101                |2
A                   |124                |3
B                   |343                |1
B                   |346                |2
C                   |345                |1
C                   |563                |2
```

- 순번을 갱신할 떄는 어떻게 해야할까?
```sql
UPDATE Weigths3
    SET seq = (SELECT seq
                FROM (SELECT class, student_id,
                                ROW_NUMBER()
                                    OVER(PARTITION BY class
                                    ORDER BY student_id) AS seq
                        FROM Weigths3) SeqTbl
    WHERE Weigths3.class = SeqTbl.class
    AND Weights3.student_id = SeqTbl.student_id);
```


- subquery는 아래와 같이 간단하다.
- 하지만 Mysql에서는 안된다. update에서는 자기 자신 table을 subquery로 참조할 수 없다.
```sql
UPDATE Weigths3
    SET seq = (SELECT COUNT(*)
                FROM Weigths3 W2
                WHERE W2.class = Weights3.class
                AND W2.student_id <= Weights3.student_id);
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

## <span style="color:#802548">_sequence는 그만..._</span>
- 시퀀스의 문제는, 성능문제가 있다.
- 시퀀스는 유일성, 연속성, 순서성을 만족해야 해서 다음 시퀀스를 만들 때 lock이 필요하다.
- 동시에 여러 사람이 시퀀스를 만들경우, 락 충돌로 성능 저하가 발생한다.
```
sequence 객체에 배타 락을 적용
NEXT VALUE를 검색
CURRENT VALUE를 1만큼 증가
시퀀스 객체에 배타 락을 해제
```

- 또한 순번처럼 비슷한 데이터를 연속으로 INSERT할 때, 물리적으로 같은 영역에 저장된다.
- 저장소의 특정 물리 블록에만 I/O가 발생하여 성능이 악화된다.
- 특히 sequence를 이용한 대량의 batch insert 시 이 문제가 심각하다.
- 사실 auto_increment 혹은 idenetiy field도 마찬가지다.


## <span style="color:#802548">_빈 값을 효율적으로 채우기_</span>
```
keycol          |seq            |val
A               |1
A               |
A               |
A               |
B               |   
B
B
B
B
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

- mysql에서는 아래와 같다.


```sql
UPDATE ScoreCols
JOIN (
    SELECT
        student_id,
        coalesce(MAX(CASE WHEN subject = '영어' THEN score ELSE NULL END),0) AS score_en,
        coalesce(MAX(CASE WHEN subject = '국어' THEN score ELSE NULL END),0) AS score_nl,
        coalesce(MAX(CASE WHEN subject = '수학' THEN score ELSE NULL END),0) AS score_mt
    FROM scoreRows
) AS Subquery
ON ScoreCols.student_id = Subquery.student_id
SET
    ScoreCols.score_en = Subquery.score_en,
    ScoreCols.score_nl = Subquery.score_nl,
    ScoreCols.score_mt = Subquery.score_mt
where exists (select *
                    from ScoreRows
                    where student_id = ScoreColsNN.student_id); 
```

- 굳이 update를 하지 않아도 된다.
- merge into를 활용하면 된다.
- merge into를 활용하면 향후 코드를 변경할 때 수정 실수도 줄일 수 있다.

```sql
merge into ScoreColsNN
    using (select student_id,
                coalesce(max(case when subject = '영어'
                                    then score
                                    else null end),0) as score_en,
                coalesce(max(case when subject = '국어'
                                    then score
                                    else null end),0) as score_nl,
                coalesce(max(case when subject = '수학'
                                    then score
                                    else null end),0) as score_mt
                from ScoreRows
                group by student_id) SR
    on (ScoreColsNN.student_id = SR.student_id)
when matched then
    update set ScoreColsNN.score_en = SR.score_en,
                ScoreColsNN.score_nl = SR.score_nl,
                ScoreColsNN.score_mt = SR.score_mt;
```


- 이번에는 위와 다르게 record -> field update를 살펴보자.
- 이때는 비교적 단순하다.
- 테이블 접근이 1번이고, 그 또한 기본키 index를 쓴다. join도 없다.

```sql
update ScoreRows
    set score = (select case ScoreRows.subject
                        when '영어' then score_en
                        when '국어' then score_nl
                        when '수학' then score_mt
                        else null
                    end
                from ScoreCols
                where student_id = ScoreRows.student_id);
```


## <span style="color:#802548">_한 테이블의 record를 다른 table로 옮기기_</span>

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

- update로도 가능하다.
  - 하지만 insert select가 update보다 더 빠른 편이다.
  - mysql은 자기참조가 안 되는 DB라서 update로 불가능하다.

- 단점은 복제 테이블이 필요하기 때문에 저장소 용량이 2배가 된다는 점이다.
- 하지만 저장소 늘리는 건 싸기 때문에 성능을 늘리는 게 더 이득이다.


## <span style="color:#802548">_데이터 모델링의 중요성_</span>
- 3일이 넘은 순간에 배송이 지연된다고 판단한다면, flag column이 없다면 구해오기가 매우 불편하다.
- 아래와 같이 긴 쿼리문을 작성해서 매번 가져와야 한다.
- 실행계획도 당연히 나쁠 것이다.
```sql
SEELCT O.order_id,
        O.order_name,
        ORC.delivery_date - O.order_date AS diff_days
    FROM Orders O
        INNER JOIN OrderReceipts ORC
            ON O.order_id = ORC.order_id
WHERE ORC.delivery_date - O.order_date >= 3;
```

```
order_id            |order_name         |diff_days
10000               |윤인성              |3
10000               |윤인성             |4
10001               |연하진             |3
10003               |한빛미디어         |5
1003                |한빛미디어         |5
```

- 하지만 지연 플래그를 넣어주면 select자체는 매우 간단해진다.
- 보통 지연이 흔하지 않기 때문에 선택률이 낮을 것이다.
  - index를 걸어주면 검색 성능이 훨씬 좋아질 것이다.
```sql
SELECT * 
    FROM Orders
WHERE delay_flag = 'Y';
```

- 다만 문제는 저 flag들을 모두 갱신해줘야 한다는 점이다.
- 언제 갱신되는 지가 중요하다. 실시간으로 갱싱해야하는지, 아니면 시차를 두는지가 말이다.
- 야간 배치 작업으로 갱신한다면 시차가 꽤나 커질 것이다.

- flag가 아닌 field를 추가하는 경우도 있다.
- count를 세는 field가 대표적이다. 이 경우, 확장성은 부족하지만 그 상황에 한정해 굉장히 편리하다. 
- 예를 들어, count를 세는 sql이 있다고 해보자.
- select문을 아래와 같이 join을 통해 가져와야 한다.
```sql
SELECT O.oredr_id,
        MAX(O.order_name) AS order_name,
        COUNT(*) AS item_count
    FROM Orders O
        INNER JOIN OrderReceipts ORC
                ON O.order_id = ORC.order_id
    GROUP BY O.order_id;
```

- 혹은 window function을 사용할 것이다.
```sql
SELECT O.order_id,
        O.order_name,
        O.oredr_date,
        COUNT(*) OVER (PARTITION BY O.oredr_id) AS item_count
    FROM Orders O
        INNER JOIN OrderReceipts ORC
            ON O.order_id = ORC.order_id;
```
```
order_id            |order_name         |item_count
10000               |윤인성              |3
10000               |윤인성             |3
10001               |연하진             |3
10003               |한빛미디어         |1
10003                |한빛미디어         |2
```

- 만약 처음 Order를 할 때 item_count를 넣었다면 복잡한 select문이 필요없었을 것이다.
- 그러나 문제는 flag 떄와 같다. 만약 주문이 향후에 바뀐다면? 그 때는 주문이 바뀌는 순간 바로 update를 쳐줘야 한다.
- 실시간성이 강력하게 요구된다. 그럼 insert와 동시에 update를 쳐야하니 성능이 떨어질 수밖에 없다.


## <span style="color:#802548">_index_</span>
- index의 종류는 3가지다.
  - 거의 대부분 B-tree다. 균형잡혀 있기 때문이다. 균형 잡혀 있고 데이터가 정렬상태를 유지하여 검색 성능이 뛰어나다.
  - 비트맵 인덱스는 갱신이 잘 일어나지 않는 환경에서만 사용한다.
  - 해시 인덱스는 거의 쓰이지 않는다.
- index는 PK field에 자동으로 걸린다.
  - 유일하기 때문에, 선택률이 매우 낮다. 카디널리티가 높은 것이다.
  - 따라서 PK가 검색 조건으로 가장 좋다.


- index를 활용할 수 없는 경우도 있다.
- 저 조건에 따른 선택률이 20%라면 너무 높아서 index 검색이 오히려 성능이 나쁘다.
```sql
SELECT order_id, receive_date
    FROM Orders
    WHERE process_flg = '5';
```

- 마찬가지 이유로 매개변수에 따라 선택률이 달라지면 index가 큰 의미가 없다.
- 선택률이 어떤 날짜에는 50%가 걸리고, 어떤 날짜에는 1%가 걸리는 경우, 극단적인 성능이 차이가 난다.
```sql
SELECT oredr_id 
    FROM Orders
    WHERE receive_date BETWEEN :start_date AND :end_date;
```

- LIKE 연산자를 사용할 때도 전방일치가 아니면 index가 작동하지 않는다.
```sql
SELECT order_id
    FROM Orders
    WHERE shop_name LIKE '%대공원%';
```
```sql
SELECT order_id
    FROM Orders
    WHERE shop_name LIKE '%대공원';
```
- LIKE 연산자가 작동하는 경우는 전방일치 뿐이다.
```sql
SELECT order_id
    FROM Orders
    WHERE shop_name LIKE '대공원%';
```

- index field로 연산하는 경우도 index가 풀린다.
```sql
SELECT *
    FROM SomeTable
    WHERE index_column > 100;
```

- IS NULL을 사용해도 index가 풀린다.
```sql
SELECT *
    FROM SomeTable
    WHERE index_column IS NULL
```

- 함수를 사용해도 index가 풀린다.
```sql
SELECT * 
    FROM SomeTable
    WHERE LENGTH(index_column) = 10;
```

- 부정형을 사용해도 index가 풀린다.
```sql
SELECT *
    FROM SomeTable
    WHERE index_column <> 100;
```

- index를 성능이 안나오는 경우에는 UI 설계를 통해 검색 성능 저하를 해결해야 한다.
- 점포 ID만으로 검색하지 않고, 주문일도 함께 입력하게 바꿔서 선택률을 낮춰 index 성능을 높인다.
- 기간 검색을 1개월 단위로만 하여 partition을 설정해준다.
- 특정 쿼리를 위해 필요한 데이터를 긁어 모은 table을 만든다. 농좆의 스크래핑이다. 데이터 마트라고 한다.

## <span style="color:#802548">_데이터 마트_</span>
- 보통 데이터 마트는 테이블 크기를 줄이는 게 목적이다. 그래야 I/O가 줄어 성능이 좋아지기 때문이다.
- 실제로는 해쉬와 정렬 등을 사용하면, 검색도 쓰기가 일어나기 때문에 I/O 성능이 중요하다.
- 따라서 SELECT * 혹은 선택률이 높아지면 절대 안된다. 데이터 마트를 만드는 이유가 없는 셈이다.


## <span style="color:#802548">_커버링 인덱스_</span>
- covering index는 table이 아닌 index만을 스캔 대상으로 삼기 때문에 성능이 매우 뛰어나다.
- 원래 같았다면 아래와 같은 sql은 성능 발휘가 어렵다.
```sql
SELECT oredr_id, receive_date
    FROM Orders;
```

- 하지만 select 되는 field에 index를 넣어주면 index로 검색하는 효과를 얻을 수 있다.
- 테이블이 아닌 index만을 scan 대상으로 하면, INDEX FAST FULL SCAN이라는 execution plan이 뜬다.
- 
```sql
CREATE INDEX CoveringIndex ON Orders (order_id, receive_date);
```


- 그럼 위에서 보았던 index 효과를 얻지 못하는 검색절도 다시 index 효과를 받을 수 있다.
```sql
CREATE INDEX CoveringIndex ON Orders (process_flg, order_id, receive_date);
SELECT oredr_id, receive_date
    FROM Orders
    WHERE process_flg = '5'
```

```sql
CREATE INDEX CoveringIndex ON Orders (shop_name, order_id, receive_date);
SELECT order_id, receive_date
    FROM Orders
    WHERE shop_name LIKE '%대공원%';
```

- 다만 index는 테이블의 update 부하가 커진다.
- 또한 covering index의 경우, SQL문에 field 변동이 일어나는 순간 무용지물이 되어버린다.
- 따라서 covering index만 따로 관리하는 유지관리 정책이 필요하다.