
# select문을 최대로 줄인 다중 필드 update
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

# 한 테이블의 record를 다른 table로 옮기기

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


# 실행계획
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

# 버퍼
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

# = null은 왜 안될까?
- =은 데이터에 적용할 수 있는 연산자다.
- null은 unknown 상태의 데이터다. 당연히 column = null을 적용할 수가 없다.
- 그럼? IS NULL + IS NOT NULL로 써줘야 한다.


# 옵티마이저
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

# 객체 조작
- 객체는 table, index, partition, sequence 등을 의미.
- table scan의 종류는 2가지가 있다.
  - TABLE ACCESS FULL은 full scan을 의미한다.
  - TaBLE ACCESS BY INDEX ROWID는 index를 이용한 scan을 의미한다.
  - 보통 INDEX UNIQUE SCAN과 같이 쓰인다.
  - index scan은 logn 복잡도고, full scan은 n 복잡도라 index scan을 성능 상 많이 쓴다.
  - 다만 index scan의 선택률이 5% 미만일 때가 좋고, 20%를 넘으면 index를 안 쓰고 full scan을 때리는 게 더 성능이 좋다.

# 결합
- nested loop, sort meger, hash가 있다.
  - nested loop는 한 테이블을 읽으면서 해당 테이블의 레코드마다, 결합 조건에 맞는 레코드를 다른 테이블에서 찾는 형태다.
  - sort merge는 레코드를 on절(결합키)로 정렬하고, 순차적으로 테이블을 결합한다. 정렬 시 워킹메모리를 사용하게 된다.
  - hash는 결합키값을 해시값으로 매핑하는데, 워킹 메모리를 사용한다.



# group by
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

# HAVING
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

# ORDER BY 구
- select문은 어떤 순서로 출력하는가? 정확한 규칙이 없다. 
- 따라서 지멋대로 나온다.
- 그게 싫다면 order by로 반드시 순서를 지정해줘야 한다.
- 참고로 order by date desc하면 최신순으로 정렬된다.

# 매칭
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

# CASE WHEN
- 교환 정의를 위해 사용하며, 성능에 매우 긍정적이다.
- 간단한 예시를 살펴보자.

```sql
SELECT name, address,
    CASE WHEN address = '서울시' THEN '경기'
    CASE WHEN address = '인천시' THEN '경기'
    CASE WHEN address = '부산시' THEN '영남'
    CASE WHEN address = '속초시' THEN '관동'
    CASE WHEN adresss = '서귀포시'THEN '호남'
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
            WHEN year <= 2002   THEN    price_tax_in END AS PRICE
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
            WHERE sex = '2') TMP
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
select prefacture, SUM(CASE WHEN sex ='1' THEN pop ELSE 0 END) AS pop_men, SUM(CASE WHEN sex ='2' THEN pop ELSE 0 END) AS pop_wom
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
SELECT emp_name,   
        CASE WHEN COUNT(*) = 1 THEN MAX(team) /* select로 선택한 field가 한개이기 때문에 COUNT(*)는 emp_name이 몇개인지 세는 형태로 진행된다. 다만 COUNT (emp_name)과 다르게 null도 포함해 센다. COUNT(*)와 COUNT(1)은 동일 */
             WHEN COUNT(*) = 2 THEN '2개를 겸무'
             WHEN COUNT(*) = 3 THEN '3개를 겸무'
            END AS team
FROM Employees
GROUP BY emp_name;
```


- 그러면 table full scan 한번만 때리게 바뀐다!
```
Id              Operation                   NAME
0               SELECT STATEMENT
1               HASH GROUP BY
2               TABLE ACCESS FULL           EMPOYEES
```

# union
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
- index를 타지 않는 경우를 먼저 살펴보자. where절에서 or를 사용했을 경우, index를 타지 않는다.
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

- 아래도 동일한 결과다.
- 하지만 where문에는 CASE WHEN을 쓰지 말자. 비즈니스 로직이 바뀌면 아예 다른 결과를 초래한다.
```sql
SELECT key, name, date_1, flg_1, date_2, flg_2, date_3, flg_3
FROM ThreeElements
WHERE 
    CASE WHEN date_1 = '2013-11-01' THEN flg_1
    CASE WHEN date_2 = '2013-11-01' THEN flg_2
    CASE WHEN date_3 = '2013-11-01' THEN flg_3;
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
3               TABLE ACCESS BY INDEX ROWID         THREEELEMENTS
4               INDEX RANGE SCAN                    IDX_1
5               TABLE ACCESS BY INDEX ROWID         THREEELEMENTS
6               INDEX RANGE SCAN                    IDX_2
7               TABLE ACCESS BY INDEX ROWID         THREEELEMENTS
8               INDEX RANGE SCAN                    IDX_3
```

- 즉 or문과 union의 차이는 index scan 여러번 vs full scan 한번이다.
- 이 경우, table 크기가 크고 index 설정이 잘 되어있을수록 index scan이 유리하다.
- 다만 이 경우는 date_n, flg_n 값 중 1개만이 제대로 된 값이고 나머지는 (null, null)일 때 제대로 성능을 발휘한다.


# CASE WHEN과 GROUP BY
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
        CASE WHEN data_type ='A' THEN data_1 ELSE NULL END AS data_2,
        CASE WHEN data_type ='B' THEN data_1 ELSE NULL END AS data_3,
        CASE WHEN data_type ='B' THEN data_1 ELSE NULL END AS data_4,
        CASE WHEN data_type ='B' THEN data_1 ELSE NULL END AS data_5,
        CASE WHEN data_type ='C' THEN data_1 ELSE NULL END AS data_6,
FROM NonAggTbl
GROUP BY id;
```

- 하지만 오류가 난다. GROUP BY에 data_1을 적어놓지 않았기 때문이다.
- 그렇기에 MAX로 감싸준다.
```sql
SELECT id,
        MAX(CASE WHEN data_type ='A' THEN data_1 ELSE NULL END AS data_1),
        MAX(CASE WHEN data_type ='A' THEN data_1 ELSE NULL END AS data_2),
        MAX(CASE WHEN data_type ='B' THEN data_1 ELSE NULL END AS data_3),
        MAX(CASE WHEN data_type ='B' THEN data_1 ELSE NULL END AS data_4),
        MAX(CASE WHEN data_type ='B' THEN data_1 ELSE NULL END AS data_5),
        MAX(CASE WHEN data_type ='C' THEN data_1 ELSE NULL END AS data_6),
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


# 집약함수와 GROUP BY
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


# PARTITION BY
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