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


## <span style="color:#802548">_GroupBy_</span>

- Distinctがある場合、Group Byに変換できるか検討したほうがいい

```sql
select 
	count(case when dp_count > 0 THEN 1 END) AS "部署がある都市",
	count(case when dp_count <= 0 THEN 1 END) AS "部署がない都市"
from (
	select 
		loc.location_Id as loc_id,
		count(dp.department_id) as dp_count
	from departments dp
	left join locations loc
	on loc.location_id　= dp.location_id
	group by loc.location_id
);
```

- この場合、Probedテーブルをまず集約してJoinされる行を圧縮することで性能がよくなる
- とくに、部署は全体行がなん百万件あっても、集約すると数百件に減るので、とても性能に優れている
- 普通は join してからGroup byしたほうがいいが、Where条件がないため、Probedテーブルで結合される行を減らすのが必要

```sql
SELECT 
    COUNT(CASE WHEN sub.dp_count > 0 THEN 1 END) AS "部署がある都市",
    COUNT(CASE WHEN sub.dp_count IS NULL THEN 1 END) AS "部署がない都市" 
FROM locations loc
LEFT JOIN (
    SELECT 
        location_id,
        COUNT(department_id) AS dp_count
    FROM departments
    GROUP BY location_id
) sub 
ON loc.location_id = sub.location_id; 
```


## <span style="color:#802548">_group by + case when_</span>
- 年度ごとの推移を見たいため、年度で集約する

```sql
select 
    year(release_date) as 年度,
    sum(case date_format(release_date, '%w') WHEN 0 THEN 1 ELSE 0 END) AS "日曜日",
    sum(case date_format(release_date, '%w') WHEN 1 THEN 1 ELSE 0 END) AS "月曜日",
    sum(case date_format(release_date, '%w') WHEN 2 THEN 1 ELSE 0 END) AS "火曜日",
    sum(case date_format(release_date, '%w') WHEN 3 THEN 1 ELSE 0 END) AS "水曜日",
    sum(case date_format(release_date, '%w') WHEN 4 THEN 1 ELSE 0 END) AS "木曜日",
    sum(case date_format(release_date, '%w') WHEN 5 THEN 1 ELSE 0 END) AS "金曜日",
    sum(case date_format(release_date, '%w') WHEN 6 THEN 1 ELSE 0 END) AS "土曜日"
from box_office
where year(release_date) between 2004 and 2013
group by year(release_date);  
```

- ColumnにIndexを加工するような行為はIndex利用を不可能にさせるため、以下のように書き換える

```sql
SELECT 
    YEAR(release_date) AS 年度,
    SUM(CASE DATE_FORMAT(release_date, '%w') WHEN 0 THEN 1 ELSE 0 END) AS "日曜日",
    SUM(CASE DATE_FORMAT(release_date, '%w') WHEN 1 THEN 1 ELSE 0 END) AS "月曜日",
    SUM(CASE DATE_FORMAT(release_date, '%w') WHEN 2 THEN 1 ELSE 0 END) AS "火曜日",
    SUM(CASE DATE_FORMAT(release_date, '%w') WHEN 3 THEN 1 ELSE 0 END) AS "水曜日",
    SUM(CASE DATE_FORMAT(release_date, '%w') WHEN 4 THEN 1 ELSE 0 END) AS "木曜日",
    SUM(CASE DATE_FORMAT(release_date, '%w') WHEN 5 THEN 1 ELSE 0 END) AS "金曜日",
    SUM(CASE DATE_FORMAT(release_date, '%w') WHEN 6 THEN 1 ELSE 0 END) AS "土曜日"
FROM box_office
WHERE release_date BETWEEN '2004-01-01' AND '2013-12-31' 
GROUP BY YEAR(release_date);
```

- Group ByにもIndexが適用されるように、関数基盤Indexを作る

```sql
CREATE INDEX idx_functional_year ON box_office ((YEAR(release_date)));
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


## <span style="color:#802548">_window function- max over_</span>
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

- 履歴データの開始日付と終了日日付もMaxを使えばうまくできる

```sql
SELECT equipment_no,
       statuts_code,
       start_date AS 開始日,
       NVL(
           MAX(start_date) OVER(
               PARTITION BY equipment_no 
               ORDER BY start_date
               ROWS BETWEEN 1 FOLLOWING AND 1 FOLLOWING
           ) - 1,
           TO_DATE('9999-12-31', 'YYYY-MM-DD')
       ) AS 終了日
FROM change_history;
```

- より簡単にかける方法として、Leadがある

```sql
SELECT equipment_no,
       statuts_code,
       start_date AS 開始日,
       NVL(
           LEAD(start_date) OVER(PARTITION BY equipment_no ORDER BY start_date) - 1, 
           TO_DATE('9999-12-31', 'YYYY-MM-DD')
       ) AS 終了日
FROM change_history;
```


- max overのみ使えるときもある
- Lagは過去たった１つの行と比べになるため、歴代との比較はMaxOverしかない

```sql
SELECT change_date, 
       current_temperature,
       MAX(current_temperature) OVER(PARTITION BY equipment_no) as 歴代最高温度,
       MAX(current_temperature) OVER(PARTITION BY equipment_no) - current_temperature as 歴代温度との差
FROM change_history
WHERE equipment_no = 'EQ_001';
```

- それ以外にもMaxOverはIndexがちゃんとかかってない時、性能の防御に優れている
    - max + subqueryではテーブルScanが2倍になる
    - RankOverなら、Diskソートがすごいため、とんでもないほど遅くなる
    - Max overは、テーブルは1つしか読んでないし、ソートも必要でないため、1番良い


- Windo Fucntionは以外にもいろいろな活用し方がある
- 以下のように郵便番号の中で最も近い値を探すQueryを工夫した

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

- テーブル読みが多いため、減らす

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

- しかし、うえのSubqueryもテーブル読みが２回行われている
- そのため、Windoｗ functionを使って1回に減らす
- inline viewもViewMergeがないと、2回読むことになるが、WindowFunctionを使えば、1回になる
- データの量が増えるほど、テーブル読みを減らすのが有利になる


```sql
SELECT pcode,
        district_name,
FROM (
        SELECT pcode,
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
        FROM Address
    ) Foo
WHERE hit_code = min_code;
```


## <span style="color:#802548">_exist -> subquery -> lad_</span>
- 以下のようなテーブルを仮定してみましょう

```
年度|       タイプ|       次数|       大会コード(pk)
2023|         A|          1|        2023年 A大会 1次
2023|         A|          2|        2023年 A大会 2次
2023|         A|          3|        2023年 A大会 3次
2022|         A|          1|        2022年 A大会 1次
2022|         A|          2|        2022年 A大会 2次
1999|         A|          1|        1999年 A大会 1次
1999|         A|          2|        2023年 A大会 2次
1999|         A|          3|        2023年 A大会 3次
2023|         B|          1|        2023年 B大会 1次
2023|         C|          1|        2023年 C大会 1次
```

- ここで以前のシーズンの年度を見たいなら、どのクエリを作成すればいいのか?

- A大会 3次は 2023年で、前は 1999年だったら、以前のシーズンは1999年
- A大会 2次は 2023年で、前は 2022年だったら、以前のシーズンは2022年
- B大会 １次は 2023年で、前は 2022年なかったら、以前のシーズンは なし
- Existsの性能がいいから、こういうふうに作成したかもしれない


```sql
SELECT
    *,
    CASE
        WHEN EXISTS (
            SELECT 1 
            FROM 大会 t2 
            WHERE t2.タイプ = t1.タイプ 
            AND t2.次数 = t1.次数 
            AND t2.年度 < t1.年度
        ) THEN (
            SELECT MAX(年度)
            FROM 大会 t2 
            WHERE t2.タイプ = t1.タイプ 
            AND t2.次数 = t1.次数 
            AND t2.年度 < t1.年度
        )
        ELSE 'なし'
    END AS '以前のシーズン'
FROM 大会 t1
ORDER BY 年度 DESC, タイプ, 次数;
```

- ただし、この場合は、むしろExistsの実行のため、テーブルをScanして、その結果として残された行を対象にもう1度SubQueryを実行するようになる

```
1	PRIMARY	            t1		ALL					10	100.00	Using filesort
3	DEPENDENT SUBQUERY	t2		ALL					10	10.00	Using where
2	DEPENDENT SUBQUERY	t2		ALL					10	10.00	Using where
```

- そのため、テーブル読みを減らすため、ここではExistsを使わないようにする

```sql
SELECT
    *,
    (SELECT MAX(年度) 
    FROM your_table t2 
    WHERE t2.タイプ = t1.タイプ 
    AND t2.次数 = t1.次数 
    AND t2.年度 < t1.年度) AS '以前のシーズン'
FROM your_table t1
ORDER BY 年度 DESC, タイプ, 次数;
```


- Subqueryは減ったが、SubQueryは行ごとに結合してしまうため、テーブル読みが行ごとに行われる

```
1	PRIMARY	            t1		ALL					10	100.00	Using filesort
2	DEPENDENT SUBQUERY	t2		ALL					10	10.00	Using where
```

- そのため、以下のようなWindowFunctionを使う
- データを一気に取ってきて1度だけソートしてメモリで直前の値を取ってくるので、早い

```sql
SELECT
    *,
    COALESCE(
        CAST(LAG(年度) OVER (
            PARTITION BY タイプ, 次数 
            ORDER BY 年度 ASC
        ) AS CHAR), 
        'なし'
    ) AS '以前のシーズン'
FROM 大会
ORDER BY 年度 DESC, タイプ, 次数;

```

- つまり、Existsでテーブル読みを減らせない場合、使わないほうがいい
- 減らせる場合、Stop Key作用が働くので、使えるんだったら、性能が良くなる


## <span style="color:#802548">_Rownumを使うときの注意点_</span>

- 以下はだめ
- SelectがOrder Byより速いので、ソートされてない状態の行を取ってしまう

```sql
select 
  rownum, 
  salary 
from employees 
where rownum < = 5
order by 2 desc;
```

- まずソートしてからRownumを使う

```sql
select *
from  
( 
  select * 
  from emp 
  order by sal desc 
) 
 where ROWNUM <= 5;
```

## <span style="color:#802548">_論理的な実行順序がいつも守られることはない_</span>

- 以下のようにSqlがあるとして、実行順序はどうなるのか？
- from -> join -> on -> where -> group by -> having -> select -> distinct -> order by -> limit/offsetだと思うかもしれない
- そういう場合、Whereで Probedテーブルとして結合されるレコードを減らす可能性が減ってしまう
- ただし、Optimizerは PushDownなどを使ってより性能の良いSqlに変換してくれる
- つまり、From -> where -> join になる

```sql
SELECT 
    COUNT(CASE WHEN sub.dp_count > 0 THEN 1 END) AS "部署がある都市",
    COUNT(CASE WHEN sub.dp_count IS NULL THEN 1 END) AS "部署がない都市" 
FROM locations loc
LEFT JOIN (
    SELECT location_id, COUNT(department_id) AS dp_count
    FROM departments
    GROUP BY location_id
) sub ON sub.location_id = loc.location_id
WHERE loc.location_id = 100; 
```

- 実際には以下のように変換される
- Where条件文が Fromに含まれる
- Where条件が定数との比較のため、ResultsetがNULLの可能性はなくなったため、Left　-> inner に変換される
- Subqueryでも同じく定数の条件の On句になるため、そこにも入れて結合の対象を減らす

```sql
SELECT ...
FROM (
    SELECT * FROM locations WHERE location_id = 100  -- predicate pushdown into driving table
) loc
INNER JOIN ( -- LEFT JOIN -> INNER JOIN
    SELECT location_id, COUNT(department_id) AS dp_count
    FROM departments
    WHERE location_id = 100 -- predicate pushdown into probed table
    GROUP BY location_id
) sub 
ON sub.location_id = 100;

```

- ああいう作業がうまく働かないときは、開発者がSubqueryなどを使い、強制させる

```sql
SELECT 
    COUNT(CASE WHEN sub.dp_count > 0 THEN 1 END) AS "部署がある都市",
    COUNT(CASE WHEN sub.dp_count IS NULL THEN 1 END) AS "部署がない都市" 
FROM (
    SELECT location_id 
    FROM locations 
    WHERE location_id = 'toyko' 
) loc
INNER JOIN (
    SELECT 
        location_id,
        COUNT(department_id) AS dp_count
    FROM departments
    GROUP BY location_id
) sub 
ON sub.location_id = loc.location_id;
```

- それでもOptimizerが賢すぎて最適化を図ろうとしてViewを Mergeすることになってしまったら、防ぐ措置を導入する
- これは Mysql

```sql
SELECT 
    COUNT(CASE WHEN sub.dp_count > 0 THEN 1 END) AS "部署がある都市",
    COUNT(CASE WHEN sub.dp_count IS NULL THEN 1 END) AS "部署がない都市" 
FROM (
    SELECT location_id 
    FROM locations 
    WHERE location_id = 'tokyo'
    LIMIT 18446744073709551615  -- View Merge防ぐ
) loc
INNER JOIN (
    SELECT location_id, COUNT(department_id) AS dp_count
    FROM departments
    GROUP BY location_id
) sub 
ON sub.location_id = loc.location_id;
```

- これは Oracle

```sql
SELECT 
    COUNT(CASE WHEN sub.dp_count > 0 THEN 1 END) AS "部署がある都市",
    COUNT(CASE WHEN sub.dp_count IS NULL THEN 1 END) AS "部署がない都市" 
FROM (
    SELECT location_id 
    FROM locations 
    WHERE location_id = 'tokyo'
    AND ROWNUM > 0           -- View Merge防ぐ
) loc
INNER JOIN (
    SELECT location_id, COUNT(department_id) AS dp_count
    FROM departments
    GROUP BY location_id
) sub 
ON sub.location_id = loc.location_id;
```

- これは SqlServer

```sql
SELECT 
    COUNT(CASE WHEN sub.dp_count > 0 THEN 1 END) AS "部署がある都市",
    COUNT(CASE WHEN sub.dp_count IS NULL THEN 1 END) AS "部署がない都市" 
FROM (
    SELECT TOP 100 PERCENT location_id  -- View Merge防ぐ
    FROM locations 
    WHERE location_id = 'tokyo'
) loc
INNER JOIN (
    SELECT location_id, COUNT(department_id) AS dp_count
    FROM departments
    GROUP BY location_id
) sub 
ON sub.location_id = loc.location_id;
```

- これはデータベースごとに対処方法が違くなるため、それを全部覚えるのが嫌な人は以下の臨時テーブルを検討したほうがいい

```sql
CREATE TEMPORARY TABLE temp_loc AS 
SELECT location_id FROM locations WHERE location_id = 'tokyo';

SELECT ... 
FROM temp_loc loc 
INNER JOIN (...) sub ON sub.location_id = loc.location_id;
```

## <span style="color:#802548">_For文でのSql呼び出しの反復を回避する_</span>
- SQLをFor文で何回も繰り返して呼び出すのは性能にとても悪い

```sql
String[] param={"M","B","P","F"};
String[] outputKey={"My","Benefit","Payment","Finance"};
HashMap<String,Object> inputParamMap = new HashMap<>();
HashMap<String,Object> elementMap = new HashMap<>();
HashMap<String,Object> decodeXssMap = new HashMap<>();
HashMap<String,Object> decodeXssList = new HashMap<>();
HashMap<String,Object> outputMap = new HashMap<>();

List<HashMap<String,Object>> subPopupList = new ArrayList<>();
String type = "";

for(int i = 0; i < param.length; i++){
    subPopupList = Dao.selectList(inputParamMap);
}
```

- こういう場合、M、B、P、FというグループをSQLで作ったほうがいい
- 定数なので、Union Allをしてもテーブル読みにならないため、早い
- Joinをしても結合される行が少ないため、負担がほぼない
- Joinの種類をINNERにするかLEFTにするかは画面側の相手と相談して決める


```sql
WITH TargetParams AS (
    SELECT 'M' AS LOC, 'My' AS KEY_NAME UNION ALL
    SELECT 'B' AS LOC, 'Benefit' AS KEY_NAME UNION ALL
    SELECT 'P' AS LOC, 'Payment' AS KEY_NAME UNION ALL
    SELECT 'F' AS LOC, 'Finance' AS KEY_NAME
),
RankedPopups AS (
    SELECT 
        tp.KEY_NAME,
        pm.BLTN_LOC,
        pm.EXPS_YN,
        pm.EXPS_OS_TP,
        pm.PUP_NO_USE_TP,
        -- Calculate UI Type directly inside the database
        CASE WHEN pm.PUP_TP = 'B' THEN 'bottom' ELSE 'layer' END AS CALCULATED_TYPE,
        -- Rank elements within each code partition group to enforce boundaries
        ROW_NUMBER() OVER (
            PARTITION BY pm.BLTN_LOC 
            ORDER BY pm.REG_DT DESC -- Assuming you want the 5 newest records
        ) as row_num
    FROM POPUP_MASTER pm
    -- Inner join screens out anything that isn't M, B, P, or F instantly
    INNER JOIN TargetParams tp 
    ON pm.BLTN_LOC = tp.LOC
)
SELECT 
    KEY_NAME,
    BLTN_LOC,
    EXPS_YN,
    EXPS_OS_TP,
    PUP_NO_USE_TP,
    CALCULATED_TYPE
FROM RankedPopups
-- Discard any records exceeding the threshold restriction ceiling
WHERE row_num <= 5;
```



## <span style="color:#802548">_파일을 하나만 등록하고 여러곳에서 사용하는 query_</span>
- 등록해 둔 배너 이미지, 내용을 3개의 다른 곳에서 사용할 수 있게 조회하는 쿼리
```sql
select RN as BNNR_DS,
        BNNR_ST_DTM,
        BNNR_ED_DTM
        , CASE WHEN RN = 1
            THEN '로그인 배너'
            WHEN RN = 2 
            THEN '메인 배너'
            WHEN RN = 3
            THEN '전체 메뉴 배너'
            END AS BNNR_DS_NM,
        BNNM,
        CASE WHEN RN = 1
            THEN  BNNR1_IMG_URL
            WHEN RN = 2
            THEN BNNR2_IMG_URL
            WHEN RN = 3
            THEN BNNR3_IMG_URL
            END as BNNR_IMG_URL,
        CASE WHEN RN = 1
            THEN BNNR_CNTN1
            WHEN RN = 2
            THEN BNNR_CNTN2
            WHEN RN = 3
            THEN BNNR_CNTN3
            END as BNNR_CNTN,
        LK_URL, TYPE, APP_DSC
        FROM BNNR_INF
        inner join(select rownum RN from dual connect by rownum <= 3)
        on to_char(sysdate, 'YYYYMMDDHH24MISS') BETWEEN BNNR_ST_DTM AND BNNR_ED_DTM
```



## <span style="color:#802548">_특정 값 먼저노출되게 정렬하는 query_</span>
- 자기 회사 우선하여 가져오기(여기선 project1)
```sql
SELECT UP_C_ID,
        COMN_C_ID,
        COMN_CNM,
        RMK_CNTN,
        RMK_CNTN2,
FROM (
    SELECT UP_C_ID,
            COMN_C_ID,
            COMN_CNM,
            RMK_CNTN1,
            RMK_CNTN2,
            CASE COMN_C_ID
            WHEN 'B1_011' /*모 회사*/
            THEN RMK_CNTN2 || '01'
            WHEN 'B1_012'
            THEN RMK_CNTN2 || '02'
            WHEN 'B2_012'
            THEN RMK_CNTN2 || '03'
            ELSE RMK_CNTN2 || '99'
            END AS ORDER_NO
    FROM O_COMMON_CODE
    WHERE UP_C_ID = 13
    AND COMN_C_UYN = 'Y'
) A
ORDER BY ORDER_NO, COMN_CNM, COMN_C_ID, RMK_CNTN2
```

- 위에는 굳이 subquery를 썼는데, 사실 subquery를 쓸 필요가 별로 없다.
- 아래와 같이 그냥 select 문에서 CASE WHEN을 써도 된다.
```sql
SELECT UP_C_ID,
       COMN_C_ID,
       COMN_CNM,
       RMK_CNTN1,
       RMK_CNTN2,
       CASE COMN_C_ID
           WHEN 'B1_011' THEN RMK_CNTN2 || '01'
           WHEN 'B1_012' THEN RMK_CNTN2 || '02'
           WHEN 'B2_012' THEN RMK_CNTN2 || '03'
           ELSE RMK_CNTN2 || '99'
       END AS ORDER_NO
FROM O_COMMON_CODE
WHERE UP_C_ID = 13
  AND COMN_C_UYN = 'Y'
ORDER BY ORDER_NO, COMN_CNM, COMN_C_ID, RMK_CNTN2;
```



## <span style="color:#802548">_서로 다른 내용 합치는 query refactoring_</span>
- 아래 서브쿼리는 서브쿼리가 아주 깊다.
- 안내
```sql
SELECT
    DECODE(new_ds, 1, '안내', 2, '혜택') AS new_nm,
    sub.*
FROM
    (
        SELECT
            '1' AS new_ds,
            bbrd_sqno AS new_id,
            CASE
                WHEN (TO_CHAR(SYSDATE, 'yyyymmdd') - TO_CHAR(TO_DATE(rg_dtm, 'yyyymmddhh24miss'), 'yyyymmdd')) <= 7 THEN 'y'
                ELSE 'n'
            END AS new_yn
        FROM
            (
                SELECT
                    rg_dtm,
                    bbrd_sqno
                FROM
                    board
                WHERE
                    bltn_yn = 'y'
                    AND TO_CHAR(SYSDATE, 'yyyymmddhh24miss') <= ed_dtm
                ORDER BY
                    rg_dtm DESC NULLS LAST
            )
        WHERE
            ROWNUM <= 1
        UNION ALL
        SELECT
            '2' AS new_ds,
            bbrd_sqno AS new_id,
            CASE
                WHEN (TO_CHAR(SYSDATE, 'yyyymmdd') - TO_CHAR(TO_DATE(req_dtm, 'yyyymmddhh24miss'), 'yyyymmdd')) <= 7 THEN 'y'
                ELSE 'n'
            END AS new_yn
        FROM
            (
                SELECT
                    req_dtm
                FROM
                    push
                WHERE
                    cusno = '10'
                    AND del_yn = 'n'
                ORDER BY
                    req_dtm DESC NULLS LAST
            )
        WHERE
            ROWNUM <= 1
    ) sub;
```

- 위의 서브쿼리를 아래와 같이 window function을 이용해 단순하게 만들자.
- 그리고 WITH까지 사용하면 매우 깔끔해진다.
```java
WITH board_cte AS (
    SELECT
        bbrd_sqno AS new_id,
        TO_DATE(rg_dtm, 'yyyymmddhh24miss') AS rg_date,
        ROW_NUMBER() OVER (ORDER BY rg_date DESC NULLS LAST) AS rn_board
    FROM
        board
    WHERE
        bltn_yn = 'y'
        AND TO_CHAR(SYSDATE, 'yyyymmddhh24miss') <= ed_dtm
),
push_cte AS (
    SELECT
        bbrd_sqno AS new_id,
        TO_DATE(req_dtm, 'yyyymmddhh24miss') AS req_date,
        ROW_NUMBER() OVER (ORDER BY req_date DESC NULLS LAST) AS rn_push
    FROM
        push
    WHERE
        cusno = '10'
        AND del_yn = 'n'
)
SELECT
    DECODE(new_ds, 1, '안내', 2, '혜택') AS new_nm,
    sub.*
FROM
    (
        SELECT '1' AS new_ds, new_id, CASE WHEN rg_date >= SYSDATE - 7 THEN 'y' ELSE 'n' END AS new_yn
        FROM board_cte WHERE rn_board = 1

        UNION ALL

        SELECT '2' AS new_ds, new_id, CASE WHEN req_date >= SYSDATE - 7 THEN 'y' ELSE 'n' END AS new_yn
        FROM push_cte WHERE rn_push = 1
    ) sub;
```
