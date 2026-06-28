## <span style="color:#802548">_集約を最小限にする_</span>
- SUMが一番上のSELECTで行われているため、GROUP BYの対象が増えてしまう
- 同じテーブルなのに、結合条件がちょっとだけ変わることでJOINを2回やっている

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



- CASEーWHENを利用し、JOINを１つに減らす
- SUMを使うテーブルであらかじめ集約を行い、大量のデータを対象に集約しないようにする

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

## <span style="color:#802548">_WhereでSubQueryじゃんく、JOIN_</span>
- Subqueryを使うと、WHERE employee_no > 450000の条件を満たす全ての行を対象にして Subqueryが発動する
- つまり、100,000行だとすると、Indexがちゃんとせってされてない場合、100,000回のTableScanが発生してしまう

```sql
SELECT employee.employee_no, employee.employee_first_name, employee.employee_last_name
FROM employee
WHERE employee_no > 450000
AND (SELECT MAX(salary)
      FROM salary
      WHERE employee_no = employee.employee_no
      ) > 100000;
```


- そのため、Joinを使い、TableScanを1回にする

```sql
SELECT employee.employee_no, employee.employee_first_name, employee.employee_last_name
FROM employee
INNER JOIN salary
on employee.employee_no = salary.employee_no
WHERE employee.employee_no > 450000
GROUP BY employee.employee_no, employee.employee_first_name, employee.employee_last_name
HAVING MAX(salary.salary) > 100000;
```


## <span style="color:#802548">_FromでSubQueryじゃんく、Exists_</span>
- Joinで結合の対象を減らす処理がない
    - Aという人が100回のレコードがあるとき、Joinの対象が増えてしまう

```sql
SELECT COUNT(DISTINCT employee.employee_no) AS count
FROM employee 
INNER JOIN (
    SELECT employee_no
    FROM employee_entry_record
    WHERE entrance_gate = 'A'
) r 
ON employee.employee_no = r.employee_no;
```

- 条件を満たすレコードがあるとき、直ちにEarly Stopを行うようにExistsを使う
- Joinではないため、Bloat現象も起きない
- 行が増えないため、つまりBloatがないため、Distinctが必要とされない

```sql
SELECT COUNT(1) AS count
FROM employee
WHERE EXISTS (SELECT 1
              FROM employee_entry_record r
              WHERE entrance_gate = 'A'
              AND r.employee_no = employee.employee_no);
```              

## <span style="color:#802548">_groupbyをするときは、必ずデータの量を減らす_</span>
- このSQLでは、Salaryテーブル全体を対象にしてソートが行われる
- あれは性能に悪い

```sql
SELECT employee.employee_no,
      salary.average,
      salary.maximum,
      salary.minimum
FROM employee,
    (
      SELECT employee_no,
            ROUND(AVG(salary),0) average,
            ROUND(MAX(salary),0) maximum,
            ROUND(MIN(salary),0) minimum
      FROM salary
      GROUP BY employee_no
      ) salary
WHERE employee.employee_no = salary.employee_no
AND employee.employee_no BETWEEN 10001 AND 10100;
```

- Group BYをしないように、Subqueryに変換する

```sql
SELECT employee.employee_no,
( SELECT ROUND(AVG(salary),0) FROM salary AS salary1 WHERE employee_no = employee.employee_no) AS average,
( SELECT ROUND(MAX(salary),0)FROM salary AS salary2 WHERE employee_no = employee.employee_no) AS maximum,
( SELECT ROUND(MIN(salary),0) FROM salary AS salary3 WHERE employee_no = employee.employee_no) AS minimum
FROM employee
WHERE employee.employee_no BETWEEN 10001 AND 10100;
 ```


- ただし、Subqueryは行あたりにScanが発生してしまうため、Inner Joinに変換する
- データベースのロジカル順序では On句がWhere句より早く読み込まれると知られているが、Optimizer最終的に判断する
- この場合は、Whereをまず読んで１００件にしてからほかの作業を進めたほうが性能に優れるため、Optimizerの大半はそっちを選ぶ

```sql
SELECT e.employee_no,
       ROUND(AVG(s.salary), 0) AS average,
       ROUND(MAX(s.salary), 0) AS maximum,
       ROUND(MIN(s.salary), 0) AS minimum
FROM employee e
INNER JOIN salary s ON e.employee_no = s.employee_no
WHERE e.employee_no BETWEEN 10001 AND 10100
GROUP BY e.employee_no;
```

- ただし、それをOptimizerにまかせるのじゃなく、Sqlとして強制させるために、InlineViewを使う
- そうしたら、可読性も上がるので良い

```sql
SELECT e.employee_no,
       s.average,
       s.maximum,
       s.minimum
FROM employee e
INNER JOIN (
    -- Force the salary table to look up and aggregate ONLY the 100 IDs
    SELECT employee_no,
           ROUND(AVG(salary), 0) AS average,
           ROUND(MAX(salary), 0) AS maximum,
           ROUND(MIN(salary), 0) AS minimum
    FROM salary
    WHERE employee_no BETWEEN 10001 AND 10100
    GROUP BY employee_no
) s ON e.employee_no = s.employee_no;
```


## <span style="color:#802548">_Joinをするときは、必ずデータの量を減らす_</span>
- 同じく、Joinをする際にもデータの量を減らすことが大事
- 以下のようなSQLの場合、employeeのすべてのレコードを対象にしてOn句とマッチするsalaryテーブルのレコードを探す
- もしemployeeが大量のデータを持ってるテーブルだったら、大変になる

```sql
SELECT employee.employee_no, employee.employee_first_name, employee.employee_last_name, employee.entry_date
FROM employee
INNER JOIN salary
ON employee.employee_no = salary.employee_no
WHERE employee.employee_no BETWEEN 10001 AND 50000
GROUP BY employee.employee_no
ORDER BY SUM(salary.salary) DESC
LIMIT 150, 10;
```


- こういうとき、InlineViewにしてDrivingテーブルのデータの量を減らす

```sql
SELECT employee.employee_no, employee.employee_first_name, employee.employee_last_name, employee.entry_date
FROM (
    SELECT employee_no
        FROM salary
        WHERE employee_no BETWEEN 10001 AND 50000
        GROUP BY employee_no
        LIMIT 150,10
    ) salary
INNER JOIN employee
ON employee.employee_no = salary.employee_no;
```


## <span style="color:#802548">_case-when_</span>
- 以下のような要件があったと仮定してみましょう

```
1 team: Output employee_first_name
2 teams: Output "Dual assignment"
3+ teams: Output "Multiple assignments"
```

- Unionを使うのが一番実装しやすい
- 一番先のQueryでMax（Team）になったのは、GROUPBYを使っているためだ
- ただし、定数のStringには結果がいつでも同一に担保されるため、Maxが要らない
- Columnは１つのレコードじゃなく、いろんなチームがある可能性があるため、Maxが要る
- つまり、１：Nの場合があるからColumnにはMaxが必要だ

```sql
SELECT emp_name, max(team) AS TEAM 
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) = 1

UNION

SELECT emp_name,
        'Dual assignment' AS TEAM
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) = 2

UNION

SELECT emp_name,
        'Multiple assignments' AS TEAM
FROM Employees
GROUP BY emp_name
HAVING COUNT(*) >= 3;
```

- ただし、Unionはテーブル読みが増えるため、Case-Whenを使って性能を改善する
- COUNT(*)は COUNT(1)と同一で、COUNT (emp_name)と違ってNullも含めて数える

```sql
SELECT emp_name, 
        CASE 
             WHEN COUNT(*) = 1 THEN MAX(team) 
             WHEN COUNT(*) = 2 THEN 'Dual assignment'
             WHEN COUNT(*) = 3 THEN 'Multiple assignments'
        END AS team
FROM Employees
GROUP BY emp_name;
```

- Case-Whenはいろんな値を合わせて見せるときにも使える
- 以下は実行ができないSQLだ

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


- CASEーWHENを使えば、実行できるようになる

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
- 会社で売り上げが発生した直前年度とその売り上げを探してと言われた
- それでこういうSubqueryを作成した

```sql
SELECT compnay,
        year,
        sale,
        CASE SIGN(sale -(
                            SELECT sale /** 直前年度の売上を選ぶ */
                            FROM Sales SL2
                            WHERE SL1.company = SL2.company
                            AND SL2.year = /**直前年度を選ぶ */
                            (
                                SELECT MAX(year)
                                FROM Sales SL3
                                WHERE SL1.company = SL3.company
                                AND SL1.year > SL3.year 
                            )
                        )
                    )
        WHEN 0 THEN = '='
        WHEN 1 THEN = '+'
        WHEN -1 THEN = '-'
        ELSE NULL END AS var
FROM Sales;
```

- 上のSubqueryは3重複になるため、数知らずに内部でTable/Index Scanが行われてしまう
- それを回避してたった1つのScanで済ませられるのがこのWindow Functionだ
- しかも (Company, yaer) のIndexがある場合、ソートすら要らなくなる
- ただ、Max Over関数は、中間年度が欠けている可能性に備えて RANGE BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING条件を使う
- ROWS BETWEEN 1 PRECEDING AND 1 PRECEDINGでは年度に値がない場合に対応できない

```sql
SELECT company,
        year,
        sale,
        CASE SIGN(sale - MAX(sale)
                            OVER (
                                    PARTITION BY company
                                    ORDER BY year
                                    RANGE BETWEEN UNBOUNDED PRECEDING AND 1 PRECEDING
                                )
                )
            WHEN 0 THEN '='
            WHEN 1 THEN '+'
            WHEN -1 THEN '-'
            ELSE NULL END AS var
FROM Sales;
```

- 上のMaxOver（）よりはLagという簡単なWindow Functionを使うと良い
- 最適化がより優れている

```sql
SELECT company,
        year,
        sale,
        CASE SIGN(sale - lag(sale)　OVER (PARTITION BY company　ORDER BY year))
            WHEN 0 THEN '='
            WHEN 1 THEN '+'
            WHEN -1 THEN '-'
            ELSE NULL END AS var
FROM Sales;
```

- 要件が拡張されてしまい、直前の会社も取得することになった
- そういう場合、Order Byの基準が変わる
- CompanyをGroupにしてまとめても直前の会社はわからないためだ
- Companyごとにでなく、むしろ全体を見てyearでソートして探す

```sql
SELECT company,
       year,
       sale,
       LAG(company) OVER (ORDER BY year, company) AS pre_company,
       LAG(sale)    OVER (ORDER BY year, company) AS pre_sale
FROM Sales;
```


todo
전체 탑 100
limit order by

그룹별 탑 100
rank over()


maxover
- 모든 변경 이력을 화면에 한 줄도 빠짐없이 다 보여주면서, 동시에 "가장 최신 데이터와 지금 행의 값이 얼마나 차이 나는지" 비교 수치를 같이 보여주어야 할 때입니다.
대량 데이터 환경에서 인덱스가 없을 때의 "최후의 보루"앞서 MAX() + 서브쿼리가 1등(가장 효율적)이라고 말씀드렸던 결정적인 전제 조건은 "복합 인덱스가 예쁘게 잘 걸려있을 때"였습니다.만약 운영 환경에서 인덱스를 전혀 만들 수 없는 통계성 테이블이거나 데이터가 수백만 건인데 인덱스가 없다면 어떻게 될까요?MAX() + 서브쿼리: 메인 쿼리 한 번, 서브 쿼리 한 번 테이블을 총 두 번 풀 스캔(Full Scan)해야 하므로 디스크 I/O가 2배로 듭니다.RANK() OVER(): 테이블은 한 번만 읽지만, 무식한 컴퓨터식 뇌 때문에 전체 정렬을 하다가 디스크 Temp가 터집니다.MAX() OVER(): 테이블을 딱 한 번만 읽으면서(Single Scan), 정렬도 안 하고 해시(Hash) 메모장으로만 풀기 때문에 인덱스가 없는 대량 데이터 환경에서는 성능 방어력 1위가 됩니다.

```sql
SELECT change_date, 
       current_temperature,
       -- 전체 데이터를 유지하면서 역대 최고 온도를 행마다 붙여줌
       MAX(current_temperature) OVER(PARTITION BY equipment_no) as 역대최고온도,
       -- 최고 온도와 현재 온도의 차이 계산
       MAX(current_temperature) OVER(PARTITION BY equipment_no) - current_temperature as 최고온도와의차이
FROM change_history
WHERE equipment_no = 'EQ_001';
```

- "이력 데이터의 시작일과 종료일(유효기간) 채우기" 예시 쿼리입니다.

```sql
SELECT equipment_no,
       상태코드,
       start_date AS 시작일,
       -- 내 바로 다음 행(1 FOLLOWING)부터 그다음 행(1 FOLLOWING)까지 중의 최대값
       -- 즉, 바로 다음 1개 행의 날짜를 집계하여 가져옴
       NVL(
           MAX(start_date) OVER(
               PARTITION BY equipment_no 
               ORDER BY start_date
               ROWS BETWEEN 1 FOLLOWING AND 1 FOLLOWING
           ) - 1,
           TO_DATE('9999-12-31', 'YYYY-MM-DD')
       ) AS 종료일
FROM change_history;
```

```sql
SELECT equipment_no,
       상태코드,
       start_date AS 시작일,
       -- 바로 다음 행의 시작일을 가져와서 1일을 뺌 (오라클 기준 -1)
       -- 만약 다음 행이 없다면(현재 진행 중인 최신 상태) '9999-12-31'로 채움
       NVL(
           LEAD(start_date) OVER(PARTITION BY equipment_no ORDER BY start_date) - 1, 
           TO_DATE('9999-12-31', 'YYYY-MM-DD')
       ) AS 종료일
FROM change_history;
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
- 이건 employee_last_name능에 매우 좋지 않다. 특히 데이터가 커질수록 말이다.
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
- 다만 아래 sql은 복잡해서 이해하기 어렵고, employee_last_name능도 나쁘다.

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

- 위의 쿼리를 절차 지향으로 바꾸면 employee_last_name능이 개선된다.
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

- 테이블에 존재하는 시퀀스 구하기는 집합 지향이나, 절차 지향이나 모두 employee_last_name능이 비슷하다.
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

- 물론 sort by를 생략한다고 해서 아래와 같이 복잡하게 쓴다면 employee_last_name능이 좋지 않을 것이다.
- 해당 장비구분코드에 맞는 equipment_no와 최종change_date, 순번을 알아오기 위한 코드인데, 꽤나 복잡하다.

```sql
SELECT equipment_no, 장비명, 상태코드
      ,(SELECT MAX(change_date)
        FROM change_history
        WHERE equipment_no = P.equipment_no) 최종change_date
      ,(SELECT MAX(change_history_order)
        FROM change_history
        WHERE equipment_no = P.equipment_no
        AND change_date = (SELECT MAX(change_date)
                        FROM change_history
                        WHERE equipment_no = P.equipment_no)) 최종change_history_order
FROM 장비 P
WHERE 장비구분코드 = 'A001';
```

- 위의 쿼리를 아래와 같이 간단하게 쓸 수도 있다.
- 그러나 MAX에 change_date와 change_history_order을 모두 쑤셔넣으면 기존에 index로 되어있던 sort by 연산이 박살난다.
- 따라서 sort by연산이 추가된다. index column을 가공한 결과는 나쁘다.
```sql
SELECT equipment_no, 장비명, 상태코드
      ,SUBSTR(최종이력, 1, 8) 최종change_date
      ,SUBSTR(최종이력, 9) 최종change_history_order
FROM (
  SELECT equipment_no, 장비명, 상태코드
        ,(SELECT MAX(change_date || change_history_order)
          FROM change_history
          WHERE equipment_no = P.equipment_no) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```


- 이럴 땐 hint를 활용해서 풀어야한다.
- 인덱스를 역순으로 써서 MAX와 동일한 효과를 낸다.
- 또한 첫번째 레코드에서 멈춰서 더 scan하지 않게 ROWNUM <=1 조건을 달아준다.
```sql
SELECT equipment_no, 장비명
      ,SUBSTR(최종이력, 1, 8) 최종change_date
      ,SUBSTR(최종이력,9) 최종change_history_order
FROM (
  SELECT equipment_no, 장비명, (SELECT /*+ INDEX_DESC(X change_history_PK) */
    change_date || change_history_order
    FROM change_history X
    WHERE equipment_no = P.equipment_no
    AND ROWNUM <= 1) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```

## <span style="color:#802548">_이력조회_</span>
- 이력조회에 first row stopkey나 top n stopkey 알고리즘을 이용하는 게 필요하다.
- 아래와 같이 장비 table이 있다고 해보자.
  - equipment_no PK
  - 장비명
  - 장비구분코드
  - 상태코드
  - 최종change_date
- 상태변경 이력 table은 아래와 같다.
  - equipment_no PK1
  - change_date PK2
  - change_history_order PK3
  - 상태코드
  - 메모
- 가장 단순하게 이력데이터를 조회해보자.
- 스칼라 서브쿼리에서 equipment_no와 change_date가 쓰이는데, 선두컬럼이기에 first row stopkey 알고리즘이 발동한다.
```sql
SELECT equipment_no, 장비명, 상태코드,
    (SELECT MAX(change_date)
    FROM change_history
    WHERE equipment_no = P.equipment_no) 최종change_date
FROM 장비 P
WHERE 장비구분코드 = 'A001'
```

- 위의 기능에서 최종change_history_order도 가져오게 추가되었다.
- inline view에서 최종change_date와 최종change_history_order을 겹쳐 가져오려 한다.
- 그런 시도는 좋지만, 그때문에 index 컬럼이 가공돼 index access조건으로 타지 못하게 됐다.
```sql
SELECT equipment_no, 장비명, 상태코드
      , SUBSTR(최종이력, 1, 8) 최종change_date
      , TO_NUMBER(SUBSTR(최종이력, 9,4)) 최종change_history_order
FROM (
  SELECT equipment_no, 장비명, 상태코드
  , (SELECT MAX(H.change_date || LPAD(H.change_history_order, 4))
        FROM change_history H
        WHERE equipment_no = P.equipment_no) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
    )
```

- 따라서 아래와 같은 쿼리로 바꿔주는 게 낫다.
- 스칼라 서브쿼리가 많아서 비효율적일 거 같지만 index access로 작동하는 게 더 빠르다.
```sql
SELECT equipment_no, 장비명, 상태코드
      ,(SELECT MAX(H.change_date)
        FROM change_history H
        WHERE equipment_no = P.equipment_no) 최종change_date
      ,(SELECT MAX(H.change_history_order)
        FROM change_history H
        WHERE equipment_no = P.equipment_no
        AND change_date = (SELECT MAX(H.change_date)
                        FROM change_history H
                        WHERE equipment_no = P.equipment_no)) 최종change_history_order
FROM 장비 P
WHERE 장비구분코드 = 'A001'
```

- 그런데 최종change_history_order에다가 또 다른 field도 가져와야 한다면?
- 최종상태코드를 가져오려면 아래와 같이 길어진다.
- 다음은 더 길어질 것이다.
```sql
SELECT equipment_no, 장비명, 상태코드
      ,(SELECT MAX(H.change_date)
        FROM change_history H
        WHERE equipment_no = P.equipment_no) 최종change_date
      ,(SELECT MAX(H1.change_history_order)
        FROM change_history H1
        WHERE equipment_no = P.equipment_no
        AND change_date = (SELECT MAX(H2.change_date)
                        FROM change_history H2
                        WHERE equipment_no = P.equipment_no)) 최종change_history_order
      ,(SELECT H1.상태코드
        FROM change_history H1
        WHERE equipment_no = P.equipment_no
        AND change_date = (SELECT MAX(H2.change_date)
                        FROM change_history H2
                        WHERE equipment_no = P.equipment_no)
        AND change_history_order = (SELECT MAX(H3.change_history_order)
                        FROM change_history H3
                        WHERE equipment_no = P.equipment_no
                        AND change_date = (SELECT MAX(H4.change_date)
                                        FROM change_history H4
                                        WHERE equipment_no = P.equipment_no))) 최종상태코드
FROM 장비 P
WHERE 장비구분코드 = 'A001'
```                                        


- 단순한 쿼리를 위해 전통적으로는 아래와 같이 썼다.
- INDEX_DESC라 인덱스를 역순으로 읽는것이고, 첫 레코드에서 바로 멈춘다.
- 다만 아래와 같은 방법은 index 구employee_last_name이 완벽해야 한다.
```sql
SELECT equipment_no, 장비명
      , SUBSTR(최종이력, 1, 8) 최종change_date
      , TO_NUMBER(SUBSTR(최종이력, 9, 4)) 최종change_history_order
      , SUBSTR(최종이력, 13) 최종상태코드
FROM (
  SELECT equipment_no, 장비명
        , (SELECT /*+ INDEX_DESC(X change_history_PK) */
                  change_date || LPAD(change_history_order, 4) || 상태코드
            FROM change_history X
            WHERE equipment_no = P.equipment_no 
            AND ROWNUM <=1) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```

- 11g에서는 아래와 같이 쓰면 된다.
- WHERE equipment_no =P.equipment_no가 서브쿼리로 들어가있다.
- 그러나 실제로는 inline view로 들어가서 조건절이 작동한다.
- prdicated pushing이다.
```sql
SELECT equipment_no, 장비명
      , SUBSTR(최종이력, 1, 8) 최종change_date
      , TO_NUMBER(SUBSTR(최종이력, 9, 4)) 최종change_history_order
      , SUBSTR(최종이력, 13) 최종상태코드
FROM (
  SELECT equipment_no, 장비명
        , (SELECT change_date || LPAD(change_history_order, 4) || 상태코드
            FROM (SELECT equipment_no, change_date, change_history_order, 상태코드
                  FROM change_history 
                  ORDER BY change_date DESC, change_history_order DESC) /**11g에서 가능한 방식*/
            WHERE equipment_no = P.equipment_no 
            AND ROWNUM <=1 ) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```

- 12c에서는 아래와 같이 쓰면 된다.
```sql
SELECT equipment_no, 장비명
      , SUBSTR(최종이력, 1, 8) 최종change_date
      , TO_NUMBER(SUBSTR(최종이력, 9, 4)) 최종change_history_order
      , SUBSTR(최종이력, 13) 최종상태코드
FROM (
  SELECT equipment_no, 장비명
        , (SELECT change_date || LPAD(change_history_order, 4) || 상태코드
            FROM (SELECT equipment_no, change_date, change_history_order, 상태코드
                  FROM change_history 
                  ORDER BY change_date DESC, change_history_order DESC
                  WHERE equipment_no = P.equipment_no)     /**12c에서 가능한 방식*/
            AND ROWNUM <=1 ) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```

- 이력 조회를 하는 서브쿼리에 윈도우 함수를 사용할 수 있다.
- 하지만 top n stopkey가 작동하지 않는다.
- 따라서 index로 sort를 생략하는 경우는 절대 사용하면 안 된다.
- 아래는 쿼리를 모르는 사람들이 짜는 anti 패턴이다.

```sql
SELECT equipment_no, 장비명
      , SUBSTR(최종이력, 1, 8) 최종change_date
      , TO_NUBMER(SUBSTR(최종이력, 9,4)) 최종change_history_order
      , SUBSTR(최종이력, 13) 최종상태코드
FROM (
  SELECT equipment_no, 장비명
        , (SELECT change_date || LPAD(change_history_order, 4) || 상태코드
        FROM (SELECT change_date, change_history_order, 상태코드
                    , ROW_NUMBER() OVER (ORDER BY change_date DESC, change_history_order DESC) NO
              FROM change_history
              WHERE equipment_no = P.equipment_no)
        WHERE NO = 1) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```

- 12c의 row limit기능을 써도 사실 위와 같은 window 함수를 쓰는 것이다.
- 따라서 stopkey 알고리즘이 발동하지 않는다.

```sql
SELECT equipment_no, 장비명
      , SUBSTR(최종이력, 1, 8) 최종change_date
      , TO_NUBMER(SUBSTR(최종이력, 9,4)) 최종change_history_order
      , SUBSTR(최종이력, 13) 최종상태코드
FROM (
  SELECT equipment_no, 장비명
        , (SELECT change_date || LPAD(change_history_order, 4) || 상태코드
  FROM change_history
  WHERE equipment_no = P.equipment_no
  ORDER BY change_date DESC, change_history_order DESC
  FETCH FIRST 1 ROWS ONLY) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
);
```


- 페이징 처리에 활용할 때 윈도우 함수를 쓰기도 한다.
- 하지만 쓰지 않는 경우도 자주 발생한다. 따라서 index/index_desc hint를 자주 써야 한다.
- sort 생략 가능한 index가 없으면 top n stopkey가 아니라 top n sort가 작동해버린다.
```sql
SELECT change_date, change_history_order, 상태코드
FROM (
  SELECT change_date, change_history_order, 상태코드
        , ROW_NUBMER() OVER(ORDER BY change_date, change_history_order) NO
  FROM change_history
  WHERE equipment_no = :eqp_no)
WHERE NO BETWEEN 1 AND 10;
```

- equipment_no가 특정번호일 때는 index 조회가 효과적이다.
- 하지만 전체에 관해 조회할 때는 index를 쓰면 random access에 따른 손해가 크다.
- 이럴 땐 window function을 쓰는 게 더 나은 선택이다.
```sql
SELECT P.equipment_no, P.장비명
      , H.change_date AS 최종change_date
      , H.change_history_order AS 최종change_history_order
      , H.상태코드 AS 최종상태코드
FROM 장비 P
      , (SELECT equipment_no, change_date, change_history_order, 상태코드
              , ROW_NUMBER() OVER(PARTITION BY equipment_no
                                  ORDER BY change_date DESC, change_history_order DESC) RNUM
        FROM change_history) H
WHERE H.equipment_no = P.equipment_no
AND H.RNUM = 1;
```


<img src="/image/window-function.jpg" />




SELECT DISTINCT c.고객ID, c.고객명 
FROM 고객 c JOIN 주문 o ON c.고객ID = o.고객ID; 

join하면 주문 수만큼 data가 늘어남.
1. 고객 테이블 (c)고객ID: 1 / 고객명: 김철수고객ID: 2 / 고객명: 이영희
2. 주문 테이블 (o)주문ID: 101 / 고객ID: 1 (김철수가 첫 번째 주문)주문ID: 102 / 고객ID: 1 (김철수가 두 번째 주문)주문ID: 103 / 고객ID: 1 (김철수가 세 번째 주문)
(아 고객ID는 1개인데 조인결과 3개로 늘어남)

이러한 중복의 가능employee_last_name을 없애려면 distinct가 필요한데, order by 연산추가됨
따라서 exists를 사용

SELECT c.고객ID, c.고객명 
FROM 고객 c 
WHERE EXISTS (SELECT 1 FROM 주문 o WHERE o.고객ID = c.고객ID);

JOIN은 조건에 맞는 모든 주문 데이터를 다 찾아서 결합하므로 느림. (끝까지 찾고 정렬)
    - 1단계 (다 찾기): 컴퓨터가 고객 한 명을 잡고, 주문 테이블에서 그 고객의 주문을 전부(100건이든 10,000건이든) 다 찾아내서 일단 거대한 합체 테이블을 만듭니다. (데이터 뻥튀기)
    - 2단계 (정렬 후 제거): 뻥튀기된 데이터에서 중복을 없애려고(DISTINCT) 전체 데이터를 컴퓨터 메모리에 넣고 정렬(Sort)을 합니다. 이 정렬 작업이 컴퓨터에게는 엄청나게 큰 부담입니다.
EXISTS는 해당 고객이 주문을 단 한 건이라도 했는지 확인하는 즉시 탐색을 종료(Semi-Join)합니다.
    - 1단계 (확인 즉시 패스): 컴퓨터가 고객 한 명을 잡고, 주문 테이블을 위에서부터 스캔합니다.
    - 2단계 (조기 종료): 그러다 그 고객의 주문이 딱 1건이라도 발견되는 순간, 뒤에 주문이 1만 건이 더 남아있어도 조사를 즉시 중단(Early Exit)하고 다음 고객으로 넘어갑니다.
늘어난 데이터를 정렬하는 비용이 통째로 사라지기 때문에 EXISTS가 훨씬 빠릅니다.



내부 동작 원리 (왜 빠를까?)DB 엔진은 orders 테이블의 인덱스를 타고 최신 주문 딱 10건만 먼저 뽑아냅니다.이제 화면에 내보낼 준비가 된 10개의 행에 대해서만 SELECT 절의 스칼라 서브쿼리가 실행됩니다.customers 테이블이 아무리 수천만 건에 달하는 대용량 테이블이더라도, 고작 10번만 인덱스로 콕콕 찔러서(Unique Index Scan) employee_first_name을 가져오고 끝납니다.전체 쿼리는 거의 0.001초 만에 종료됩니다


```sql
SELECT o.order_id,
       o.order_date,
       o.amount,
       -- 딱 10번만 실행되는 스칼라 서브쿼리 (매우 가볍고 빠름)
       (SELECT c.name FROM customers c WHERE c.id = o.customer_id) AS customer_name
FROM orders o
WHERE o.order_date >= '2026-06-01'
ORDER BY o.order_id DESC
LIMIT 10; -- ★ 중요: 결과 행 수를 10건으로 엄격하게 제한
```


 내부 동작 원리 (왜 느려질 수 있을까?)옵티마이저가 완벽하다면 이 쿼리도 10건만 먼저 자르고 조인하겠지만, 테이블 통계 정보가 조금만 꼬여도 옵티마이저는 다음과 같은 최악의 판단을 내릴 수 있습니다 [MySQL].대량 조인 먼저 수행: LIMIT 10 조건이 쿼리 맨 마지막에 있기 때문에, 옵티마이저는 6월 1일 이후 발생한 주문 수십만 건과 대용량 customers 테이블을 해시 조인(Hash Join)이나 대규모 루프 조인으로 통째로 먼저 합쳐버립니다.정렬 및 커팅: 두 테이블이 거대하게 합쳐진 가상 테이블을 가지고 ORDER BY로 정렬한 뒤, 그제야 맨 위의 10건만 남기고 나머지 수십만 건의 조인 결과를 버립니다.결과: 고작 10건 보여주려고 뒤에서 수십만 건의 조인 연산과 메모리(혹은 디스크 I/O)를 낭비하느라 쿼리가 몇 초 이상 버벅거리게 됩니다.


```sql
SELECT o.order_id,
       o.order_date,
       o.amount,
       c.name AS customer_name
FROM orders o
LEFT JOIN customers c ON c.id = o.customer_id -- ⚠️ 여기서 문제가 생길 수 있음
WHERE o.order_date >= '2026-06-01'
ORDER BY o.order_id DESC
LIMIT 10;
```

FROM절 서브쿼리(인라인 뷰)를 활용한 페이징 조인 튜닝 방식입니다.
완벽한 데이터 압축: DB는 우선 가장 안쪽에 있는 서브쿼리(①번 블록)를 실행합니다. 인덱스를 타고 최신 주문 딱 10건만 가상 테이블(메모리)에 올립니다. 이 시점에서 데이터양은 100만 건에서 10건으로 압축됩니다.최소한의 조인 연산: 밖으로 나와서 customers 테이블과 조인을 시도할 때, 조인 대상이 고작 10건밖에 안 되기 때문에 customers 테이블이 1억 건이든 10억 건이든 상관없이 딱 10번만 인덱스로 콕 찔러서 employee_first_name을 가져옵니다.스칼라 서브쿼리와의 차이점 (유지보수의 절대적 우위): 만약 고객의 employee_first_name(name)뿐만 아니라 고객의 등급(grade), 연락처(phone) 등 여러 개의 컬럼을 한 번에 화면에 뿌려야 한다면, 스칼라 서브쿼리는 SELECT 절에 서브쿼리를 3개나 똑같이 적어야 해서 employee_last_name능이 저하됩니다. 하지만 이 방식은 c.grade, c.phone을 SELECT 절에 그냥 적어주기만 하면 단 한 번의 조인으로 모두 가져올 수 있습니다.

```sql
SELECT o.order_id,
       o.order_date,
       o.amount,
       c.name AS customer_name
FROM (
    -- ① [핵심] 주문 테이블에서 최신 데이터 딱 10건만 먼저 완벽하게 잘라냄
    SELECT order_id, order_date, amount, customer_id
    FROM orders
    WHERE order_date >= '2026-06-01'
    ORDER BY order_id DESC
    LIMIT 10
) o
-- ② 딱 10건으로 줄어든 가벼운 결과셋을 가지고 고객 테이블과 조인함
LEFT JOIN customers c ON c.id = o.customer_id;
```


employee_last_name능과 확장employee_last_name 면에서 "스칼라 서브쿼리는 아예 안 쓰고, 말씀하신 FROM 절 서브쿼리(인라인 뷰) 기반의 페이징 조인을 표준(Default)으로 삼는 것이 맞다"는 판단이 실무적으로 100% 옳습니다. 현대 대규모 시스템을 구축하는 많은 아키텍트와 시니어 개발자들이 실제로 그렇게 가이드를 주고 있습니다.



- 상관서브쿼리를 쓴다면 아래와 같이 쓸 수 있다.
- 해당 서브쿼리가 좋지 않은 이유는, SELECT를 구매 table에서 진행했지만, 서브쿼리를 만나서 한번 더 구매 table을 SCAN해야하기 때문이다.
- 똑같은 TABLE을 두 번 scan하게 되는 것이다.
- 또한 상관서브쿼리는 결합과 마찬가지로 데이터양에 따라 실행계획에 변동employee_last_name이 높다.

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


## <span style="color:#802548">_대회 차수별 직전연도를 알려주세요!_</span>
- 만약 아래와 같은 구조의 테이블이 있다고 해보자.
```
연도|       종류|       차수|       대회코드(pk)
2023|         A|          1|        2023년 A대회 1차
2023|         A|          2|        2023년 A대회 2차
2023|         A|          3|        2023년 A대회 3차
2022|         A|          1|        2022년 A대회 1차
2022|         A|          2|        2022년 A대회 2차
1999|         A|          1|        1999년 A대회 1차
1999|         A|          2|        2023년 A대회 2차
1999|         A|          3|        2023년 A대회 3차
2023|         B|          1|        2023년 B대회 1차
2023|         C|          1|        2023년 C대회 1차
```

- DDL로 우선 만들어주자.
```
CREATE TABLE `대회` (
  `대회코드` varchar(200) NOT NULL,
  `차수` varchar(45) DEFAULT NULL,
  `종류` varchar(45) DEFAULT NULL,
  `연도` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`대회코드`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3
```

- 여기서 전시즌의 연도를 보고싶다면 어떻게 쿼리문을 짜야할까?
- A대회 3차가 2023년에 일어났고, 그전에는 1999년에 있었다면 1999년이 전시즌이다.
- A대회 2차가 2023년에 일어났고, 그전에는 2022년에 있었다면 2022년이 전시즌이다.
- B대회 1차가 2023년에 일어났고, 그전에는 없었다면 전시즌은 '없음'이다.
```
연도|       종류|       차수|            대회코드(pk)|          전시즌
2023|         A|          1|        2023년 A대회 1차|           2022
2023|         A|          2|        2023년 A대회 2차|           2022
2023|         A|          3|        2023년 A대회 3차|           1999   
2022|         A|          1|        2022년 A대회 1차|           1999
2022|         A|          2|        2022년 A대회 2차|           1999
1999|         A|          1|        1999년 A대회 1차|           없음
1999|         A|          2|        2023년 A대회 2차|           없음
1999|         A|          3|        2023년 A대회 3차|           없음
2023|         B|          1|        2023년 B대회 1차|           없음
2023|         C|          1|        2023년 C대회 1차|           없음
```

- 아래와 같이 짤 수 있다. null로 가져오고 싶다면 ELSE만 NULL로 써주면 된다.
```sql
SELECT
    *,
    CASE
        WHEN EXISTS (
            SELECT 1 
            FROM 대회 t2 
            WHERE t2.종류 = t1.종류 
            AND t2.차수 = t1.차수 
            AND t2.연도 < t1.연도
        ) THEN (
            SELECT MAX(연도)
            FROM 대회 t2 
            WHERE t2.종류 = t1.종류 
            AND t2.차수 = t1.차수 
            AND t2.연도 < t1.연도
        )
        ELSE '없음'
    END AS '전시즌'
FROM 대회 t1
ORDER BY 연도 DESC, 종류, 차수;
```

- subquery로 바꾸면 아래와 같다. CASE WHEN을 쓰는 습관을 들이는 게 더 좋긴 하지만, 이 경우에는 아니다.
- subquery가 더 직관적이면서 실행계획도 더 낫다.
```sql
SELECT
    *,
    (SELECT MAX(연도) 
    FROM your_table t2 
    WHERE t2.종류 = t1.종류 
    AND t2.차수 = t1.차수 
    AND t2.연도 < t1.연도) AS '전시즌'
FROM your_table t1
ORDER BY 연도 DESC, 종류, 차수;
```

- CASE WHEN일 때의 실행계획이다. 
- 상관 서브쿼리가 2개나 있다.
```
1	PRIMARY	            t1		ALL					10	100.00	Using filesort
3	DEPENDENT SUBQUERY	t2		ALL					10	10.00	Using where
2	DEPENDENT SUBQUERY	t2		ALL					10	10.00	Using where
```

- 반면에 명시적으로 서브쿼리를 사용하는 경우다.
- 상관 서브쿼리가 오히려 1개 줄었다. 성능이 더 좋아진 것이다.
```
1	PRIMARY	            t1		ALL					10	100.00	Using filesort
2	DEPENDENT SUBQUERY	t2		ALL					10	10.00	Using where
```


## <span style="color:#802548">_해당 디바이스 아이디의 고객번호를 찾아주세요!_</span>
- 서브쿼리를 아래와 같이 쓸 수 있다.

```sql
select cusno, grdc
from entry
where cusno = (
    select cusno
    from device
    where devcId = #{devcId}
)
```

- where exists로 검색해야하는 양이 적다면, where exists가 효율성은 더 좋다.

```sql
select cusno, grdc
from entry en
where exists (
    select 1
    from device de
    where devcId = #{devcId}
    and en.cusno = de.cusno
)
```

- 만약 cusno가 entry와 device가 공유하는 컬럼이라면, join으로 가져올 수 있다.
- 3개 중에는 join이 가장 효율적일 수도 있다.
```sql
select en.cusno, en.grdc
from entry en
inner join device de
on en.cusno = de.cusno
where de.devcId = #{devcId}
```

## <span style="color:#802548">_해당 디바이스아이디의 고객번호의 비밀번호 오류를 초기화할게요!_</span>

```sql
UPDATE entry
SET stl_pw_acm_err_nt = 0,
    stl_pw_dd1_err_nt = 0
WHERE cusno = (
    SELECT cusno
    FROM device
    WHERE deviceid = #(devcid)
    AND devcuyn = 'y'
);
```

- where exists로 바꾸면 아래와 같이 된다.
- where exists를 쓰는 이유는 optimizer가 제대로 작동하지 않을 때 더 나은 방식이기 때문이다.
```sql
UPDATE entry
SET stl_pw_acm_err_nt = 0,
    stl_pw_dd1_err_nt = 0
WHERE EXISTS (
    SELECT 1
    FROM device
    WHERE device.deviceid = #(devcid)
    AND device.devcuyn = 'y'
    AND device.cusno = entry.cusno
);
```

- devcuyn이 y인 deviceId는 한개 밖에 없다면, 맨 밑의 cusno 조건도 필요없다.
- 하지만 저 조건을 넣어서 안 타던 index를 탈 수 있다면 넣는게 좋다.
```sql
UPDATE entry
SET stl_pw_acm_err_nt = 0,
    stl_pw_dd1_err_nt = 0
WHERE EXISTS (
    SELECT 1
    FROM device
    WHERE device.deviceid = #(devcid)
    AND device.devcuyn = 'y'
);
```

- 참고로 IN문도 WHERE EXIST로 바꿀 수 있다.
```sql
SELECT name
    FROM Address
    WHERE name in (SELECT name FROM Address2)
```
```sql
SELECT name
FROM Address AS A
WHERE EXISTS (
    SELECT 1
    FROM Address2 AS A2
    WHERE A.name = A2.name
);
```