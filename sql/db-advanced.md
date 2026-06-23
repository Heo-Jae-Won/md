## <span style="color:#802548">_page_</span>
- everything is managed by page
- it is called block either
- so index, rows are stored in page
- page is 4 ~ 16KB block usually
- https://medium.com/databases-in-simple-words/breaking-down-the-anatomy-of-a-database-page-90e751d018fe
- https://shiftasia.com/community/pages-and-blocks-in-sql-databases/


## <span style="color:#802548">_index access_</span>
- index adopts b-tree
    - index itself consists of index root, branch, leaf block
    - root, branch helps find out where is leaf block
    - going deep with root -> branch -> leaf block, found index key that satisfies condition
    - index is sorted so keep reading index record(key-value pair)
- index leaf scan is totally different from data block scan
- index leaf block doesnt read entire row
    - at first, finding starting point, which is vertical scan
    - finally, seeking all index leaf that satisfies where condition from starting ponit to end point, which is finding end point
    - this is horizental scan
    - thoses vertical and horizental scan is about index access condition
- vertical scan
    - finding first records that satisfies condition
    - starting point of page
      - for knowing exact starting point, columns for index should not be manipulated
    - let's take a breif look into good practice
    - this is good example of utilizing vertical scan
    - index range scan can be utilized

    ```sql
    where birthday between '20070101' and '20070131'
    ```

    - this is bad
    - we now dont know where to start, which means inefficient index full scan

    ```sql
    where substr(birthday, 5, 2) = '05'
    where company like '%Japan%'
    where (tel_no = :tel_no OR customer_name = :cust_nm)
    where tel_no in (:tel_no1, :tel_no2)
    ```

    - fortunately, in operator and or operator is optimized
    - now index range scan is utilized

    ```sql
    where customer_name = :cus_nm
    union all
    where tel_no = :tel_no
    ```

    - take a look into complex index
    - let's suppose that index is configured as (team_code, employee_name, age)
    - this is bad, because employee_name is located second in index
    - cuz index is sorted by team_code at first, index range scan is not fullfilled
    - index full scan would be applied

    ```sql
    SELECT employee_code, team_code, age, registration_date, tel_no
    from employee
    where employee_name = 'tarou'
    ```
  

- horizental scan
    - keep lookingup after sepcifying starting point
    - finding every rows that satisfy condition

- likewise, index sorts record
- so, if index is properly set, order by or group by doesnt need sort operation at all
- let's suppose index is consistd of (equipment_no, change_date, change_history_order)
- record is sorted equipment_no, change_date order, so in followed sql, change_history_order must be sorted properyly
- therefore sort by doesnt need
- index block is double-link list, so max is same as min

```sql
SELECT MIN(change_history_order)
FROM change_history
WHERE equipment_no = 'C'
AND change_date = '20180316'
```

- but when u manipulate indexed column, then index is ignored
- problem is not nvl(), but TO_NUMBER()
- change_history_order better to be a int type
- or type conversion better to occur in application code

```sql
SELECT NVL(MAX(TO_NUMBER(change_history_order)), 0)
FROM change_history
WHERE equipment_no = 'C'
AND change_date = '20180316'
```

## <span style="color:#802548">_how vertical/horizental scan work in index access_</span>
- let's suppose followed query

```sql
WHERE C1 = 'B'
```

- index leaf block scan dont jump down to C1 = B leaf block
- it goes down step by step, cuz C1 = B is not a starting point of block in this case
- cuz leaf block is not accumulated only with that condition, leaf block's starting point is not C1 = B

<img src="/image/condition1.jpg" />


- see this complex where condition with (C1, C2) index

```sql
WHERE C1 = 'B'
AND C2 = 3
```

- then as leading column is C1, at first goes down to C1= B
- and then finding C2=3 index record. that process is vertical scan
- index is sorted so just reading all C2=3 index record. that process is horizental scan
- horizental scan stops at the point of encoutering C2 = 4 index record

<img src="/image/condition2.jpg" />


- what about range where ?

```sql
WHERE C1 = 'B'
AND C2 >= 3
```

- end point changes from (C2 = 3) to (C1 = C)
- but still, range where condition helps a lot to reduce range scan

<img src="/image/condition3.jpg" />


- even if reverse Equality signs, no problem
- cuz index is doulbe-linked list

```sql
WHERE C1 = 'B'
AND C2 <= 3
```

- range where condition helps a lot to reduce range scan
- between has same effect cuz it's basically same to range where condition


<img src="/image/condition4.jpg" />


- what about leading column is range where condition ?

```sql
WHERE C1 BETWEEN 'A' and 'C'
AND C2 BETWEEN 2 AND 3
```

- leading column's range where condition helps a lot in reducing index scan
- but followed column C2 does not play a role in reducing index scan
- if index field column is range scan, then after that columns, index access is invalidated

<img src="/image/condition6.jpg" />

- what if leading column does not exist? 
- then index full scan would happen
- bascially 500 rows are included for each blcok
- therefore, 5463 x 500 = 3,731,500 is read and 10 record is acquired
- it's very inefficient way

```
**sql trace
Rows      Row Source Operation
10        TABLES ACCESS BY INDEX ROWID BIG_TABLE(cr=7471 pr=1466 pw=0 time=22137)
10          INDEX RANGE SCAN BIG_TABLE_IDX (cr=7463 pr=1466 pw=0 time=22328)
```

## <span style="color:#802548">_index filter_</span>
- index filter decides which table do u access
- take a look at followed query

```sql
select /*+ index(emp emp_x01) */
from emp
where deptno = 30
and sal >= 2000
```

- index consists of (deptno, job)
- 6 index record is found
- table access 6 times either cuz we dont have a information about sal in index record

<img src="/image/not-added-column-index.jpg" />


- we want to change (deptno, job) to (deptno, sal) but (deptno, job) is used
- therefore, add new index field to exsitng complex index like (deptno, job, sal)
- it doesnt improve index range scan but table access time is decreased from 6 times to 1 time
- random access is minimalized

<img src="/image/column-added-index.jpg" />



- adding a complete new index leads to DML performance get lower and management
- to existing index, adding a new field is recommended on maintanence


- now take a look at real example
- It is like, but range scan is possible because it is a prefix match.
- cuz we can confine starting point

```sql
select rental_management_no, customer_name, service_management_no, service_no, reservation_date
        ,visiting_country_code1, visiting_country_code2, visiting_country_code3, roaming_approval_no, autoroaming_approval_no
from roaming_rental
where service_no like '010%'
and use_yn = 'Y'
```

- but does it mean it has a good performance ?
- block be read from index are 266476, and table access is executed as many times as the index records.
- table access reads cr=266968, excluding index block is 1011 count, 265,957 is read
- as table access block count is 266,476 and block I/O is 265,957, it performs very bad
  - it means data is scattered everywhere cuz it's large dataset
- in this condition, after use_yn checked, left rows count is 1909

<img src ='/image/roaming-not-added-useyn.jpg' />

- to prevent this kind of bad table access, we can include use_yn, (service_no, use_yn)


## <span style="color:#802548">_how to reduce index scan: in-list_</span>
- if leading column is range scan, then all other index column acts as index filter
- to let other columns act as a index access, index configuration change is needed
- but using in operator, we can optimize index range scan
  - it reduces horizental scan

<img src="/image/in-list-vertical-scan.jpg" />

- if index condition is satisfied, in operator actually acts like below
- so it reduces horizental scan

```sql
SELECT FLOOR, PRICE_PER_PYEONG, REGISTRATION_DATE, BUILDING_NUMBER, PROPERTY_TYPE, DAYS_OF_USE_PER_YEAR, AGENCY_CODE
FROM APARTMENT_FOR_SALE
WHERE IS_ONLINE_PROPERTY = 1
AND APARTMENT_MARKET_PRICE_CODE = 'A01011350900056'
AND SIZE_PYEONG = '59'
AND TYPE = 'A'
ORDER BY REGISTRATION_DATE DESC


UNION ALL

SELECT FLOOR, PRICE_PER_PYEONG, REGISTRATION_DATE, BUILDING_NUMBER, PROPERTY_TYPE, DAYS_OF_USE_PER_YEAR, AGENCY_CODE
FROM APARTMENT_FOR_SALE
WHERE IS_ONLINE_PROPERTY = 2
AND APARTMENT_MARKET_PRICE_CODE = 'A01011350900056'
AND SIZE_PYEONG = '59'
AND TYPE = 'A'
ORDER BY REGISTRATION_DATE DESC


UNION ALL 

SELECT FLOOR, PRICE_PER_PYEONG, REGISTRATION_DATE, BUILDING_NUMBER, PROPERTY_TYPE, DAYS_OF_USE_PER_YEAR, AGENCY_CODE
FROM APARTMENT_FOR_SALE
WHERE IS_ONLINE_PROPERTY = 3
AND APARTMENT_MARKET_PRICE_CODE = 'A01011350900056'
AND SIZE_PYEONG = '59'
AND TYPE = 'A'
ORDER BY REGISTRATION_DATE DESC

```

- The problem is that a large number of items in the IN operator can lead to inefficiency
  - especially if leaf block has deep depth, then vertical scan would show bad performance
- in that case, u can use join

```sql
SELECT b.FLOOR, b.PRICE_PER_PYEONG, b.REGISTRATION_DATE, b.BUILDING_NUMBER, b.PROPERTY_TYPE, b.DAYS_OF_USE_PER_YEAR, b.AGENCY_CODE
FROM INTEGRATED_CODE a, APARTMENT_FOR_SALE b
WHERE a.CODE_CLASSIFICATION = 'CD064'
AND a.CODE BETWEEN '1' AND '3'
AND b.IS_ONLINE_PROPERTY = a.CODE
AND b.APARTMENT_MARKET_PRICE_CODE = 'A01011350900056'
AND b.SIZE_PYEONG = '59'
AND b.TYPE = 'A'
ORDER BY b.REGISTRATION_DATE DESC

```

- or u can use index skip scan
- this is better choice when leading column perform range search

```sql
SELECT /*+ INDEX_SS(t MONTHLY_SALES_SUMMARY_BY_CUSTOMER_IDX2) */ COUNT(*)
FROM MONTHLY_SALES_SUMMARY_BY_CUSTOMER t
WHERE SALES_CLASSIFICATION = 'A'
AND SALES_MONTH BETWEEN '201801' AND '201812'
```

- in practice, there is a condition where having the non-leading column serve as an index filter yields better performance
- if index is (customer_no, commodity_type_code), then modifying not leading columns can be a choice

```sql
SELECT *
FROM purchased_commoditiy_user
WHERE customer_no = :cust_no
AND commodity_no || '' in ('NH00037', 'NH00041','NH00050')
```

## <span style="color:#802548">_index skip scan_</span>
- for reducing index scan, u can use index skip scan either
- index skip scan
  - until now, we suppose good leading index column
  - in practice, there could be no leading index column in where clause
  - in this case, we can use index skip scan, actually it's used in case of columns that one has high cardinality and one has low
- let's suppose that PK consists  of (field_type_code,field_code,base_date)
- followed query lacks field_code
- in this case, we can use index skip scan
    - if u give hint, it would be more clear

```sql
SELECT /*+ INDEX_SS(A daily_field_transaction_PK) */
      base_date, field_code, success_transaction, success_transaction_count, transaction_count
FROM daily_field_transaction A
WHERE field_type_code = '01'
AND base_date BETWEEN '20080501' AND '20080531'
```

- when leading and medium index column lacks, u can use index skip scan either

```sql
SELECT /*+ INDEX_SS(A daily_field_transaction_PK) */
      base_date, field_code, success_transaction, success_transaction_count, transaction_count
FROM daily_field_transaction A
AND base_date BETWEEN '20080501' AND '20080531'
```

- when leading column is <>, BETWEEN, LIKE, index skip scan is useful
- let's suppose (base_date + field_type_code) index
- if index range scan is executed, base_date that satisfies BETWEEN condition should be scaned, which means a lot of scans
- if using index skip scan, we can skip index range scan that field_type_code is not 01

```sql
SELECT /*+ INDEX_SS(A daily_field_transaction_PK) */
      base_date, field_code, success_transaction, success_transaction_count, transaction_count
FROM daily_field_transaction A
AND base_date BETWEEN '20080501' AND '20080531'
AND field_type_code = '01'
```

## <span style="color:#802548">_index access good practice condsidering type priority_</span>

- date vs char wins date
- followed query has no problem
- but better to format it, cuz parameter could be different

```sql
SELECT * FROM custoemr
/*WHERE register_date = '01-JAN-2018'*/
WHERE register_date = TO_DATE('01-JAN-2018', 'DD-MON-YYYY')
```

- Like makes everything string

```sql
SELECT * FROM customer
WHERE customer_no LIKE '9410%'
```

- index is ignored because the column is modified

```sql
SELECT * FROM customer
WHERE TO_CHAR(customer_no) LIKE '9410%'
```

- suppose that if account_no exists, then use acount_no
- and no, then transaciont_date is needed

```sql
SELECT * FROM transaction
WHERE account_no LIKE :acnt_no || '%'
AND transaction_date BETWEEN :trd_dt1 and :trd_dt2
```

- that one is very bad
- cuz if account_no is int, implicit conversion happens index is ignodred when (account_no, trasaction_date) index
- which means transaction_date cannot use index at all
- to avoid this problem, if u changing index to (transaction_date, account_no), then index range scan would increase enormoulsy
- therefore, when u need to use optional like, then let application code decide with dynamic sql


<img src='/image/roamding-added-useyn.jpg' />


## <span style="color:#802548">_another way of index filter: index-organized table(clustered index)_</span>

- instead of adding a new index field column, using index-organized table can be option
- in index-organized table, index leaf block is data block

```sql
create table_index_org_t (a number, b varchar(10) constraint index_org_t_pk primary key (a))
organization index;
```

- in mysql or SQL server, table basically adopts clustered index, IOT
- IOT stores actual data rows in index leaf nodes and sorted
- so it uses sequential access according to index leaf node, not random access
- this is best choice when insert pattern and select pattern is different
- typical example is insert by date, select by PK
  - let's suppose that batch to calculate sales performance is performed by date
  - then this inserted data block is scattered everywhere
    - Day 1 (Jan 1): Sales records for Salesman A, B, and C are written into Data Block #1.
    - Day 2 (Jan 2): Sales records for Salesman A, B, and C are written into Data Block #2.
    - Day 3 (Jan 3): Sales records for Salesman A, B, and C are written into Data Block #3.
  - but select is executed by only employee_no
    - But because the rows are physically scattered based on the insert date, Row 1 is in Block 1, Row 2 is in Block 2, Row 3 is in Block 3...
    - so CF is very bad and table access is enormoulsy increased
- in this case, we can use IOT
  - Physical Sorting on Insert
    - It forces the data into the B-Tree structure sorted by salesman_id
  - Physical Data Alignment
    - let blocks be allocated in a few continuous data blocks (e.g., Block 10, 11, 12, 13)
  - The Resulting Query Benefit
    - cuz it's sorted by employee_no and collected in near data blocks 
    - we can use fast Sequential Access
- but there is trade-off 
    - IOT slower insert
        - database must constantly split and rearrange blocks to keep them sorted by salesman_id

```sql
create table sales_performance (사번 varchar2(5), 일자 varchar2(8), .....) organization index;
create index(employee_no, regist_date) in sales_performance
```



## <span style="color:#802548">_clusterd index vs non-clustered index_</span>
- clustered index physically sorts and stores the rows of the table on disk
  - so at the end of b-tree seek, u can find data row itself
  - it means that, in real clsuterd index is almost table
  - actually, clustered index use sequential I/O like table scan
    - so if u scan big index range with clustered index, it consumes so many byte 2KB with 
  - while non-clustered index find pointer to real actual data
- only 1 clusterd index is allowed and usually that is PK
  - data must be sorted in two different ways simultaneously is impossible, so it's only 1 
- basically, order by, group by column or range scan column is included
  - like (PK, date), (date, UK), (date, status, FK), (date, identtiy)
- automatically, indexes except for clustered-index is non-clustered index

## <span style="color:#802548">_index scan_</span>

- index scan means index access + index filter + table filter
- index scan condition
    - for index scan, there is 3 way
        - index access: deciding index range(vertical scan, horizental scan) 
        - index filter: table access(including column into index field or not)
        - table filter: just filtering rows aftering finishing scaning page, so it has no effect on performance
- so if u decide which index should be leading column and which columns should be included, this can be standard
    - sort order
        - cardinality
        - equal > range > not equal
    - table access minimalization
        - clusetering factor
        - https://engineering-skcc.github.io/oracle%20tuning/Clustering-Reorg/


<img src="/image/index-access-filter.jpg"/>



## <span style="color:#802548">_why u wouuld better not to use like_</span>
- dont use like if possible
- cuz like increases index range scan

```sql
SELECT *
FROM 월별customer별판매집계
WHERE 판매월 LIKE '2019%'
AND 판매구분 = 'B'

SELECT *
FROM 월별customer별판매집계
WHERE 판매월 BETWEEN '201901' and '201912'
AND 판매구분 = 'B'
```

<img src="/image/between-like-difference.jpg" />

- u want to merge sql for reducing manageing risk, so u wrote sql like followed using 'like' operator
- if u overuse like, u pay the price in performance

```sql
SELECT customer_id, commodity_name, 지역코드, ...
FROM 가입상품
WHERE 회사코드 = :com
AND 지역코드 LIKE :reg || '%'
AND commodity_name LIKE :prod || '%'
```

<img src="/image/inefficient-like.jpg" />

- u must separate sql
- u can use dynamic sql in level of application code 

```sql
SELECT * customer_id, commodity_name, 지역코드, ....
FROM 가입상품
WHERE 회사코드 = :com
AND 지역코드 = :reg
AND commodity_name LIKE :prod || '%'

SELECT customer_id, commodity_name, 지역코드, ...
FROM 가입상품
WHERE 회사코드 = :com
AND commodity_name LIKE :prod || '%'
```

<img src="/image/efficient-like.jpg" />


- especially for nullable column, dont use like
- resultset would be corrupted 
- even if the input is empty, query will not be turned into LIKE '%' and return all rows.
- it just return null cuz null is unknown type so where clause would be false

```sql
WHERE column LIKE :input || '%' 
```

- like in number type column is worst choice
  - like is string operator so it converts number column to string column, which leads to modifying columns that cannot utilize index access

```sql
SELECT * FROM transaction
WHERE transaction_date = :trd_dt
AND customer_id/**internally TO_CHAR(customer_id) */ LIKE :cust_id || '%'
```

## <span style="color:#802548">_index design_</span>
- there is something to condisder when doing index design
  - often used column with = in where clause
  - how many times query is executed
  - minimize sort byte when executing group by 
- now look at practical example
- for example, suppose these columns (HANDLING_DEPARTMENT -HANDLING_BRANCH ,HANDLER ,DATA_ENTRY_USER ,AGENCY_PLANNER ,AGENCY_BRANCH, APPLICATION_DATE , INCEPTION_DATE , EXPIRATION_DATE , DATA_CREATION_TIMESTAMP) 
  - if search data is not that long, then index scan inefficiency is not that harmful
    - and in this case, contract search is done for 3 days from now
    - many index means DML performance gets lower

```
X01: APPLICATION_DATE, HANDLING_DEPARTMENT, HANDLING_BRANCH, HANDLER, DATA_ENTRY_USER, DATA_ENTRY_USER, AGENCY_BRANCH
X02: APPLICATION_DATE, HANDLING_DEPARTMENT, HANDLING_BRANCH, HANDLER, DATA_ENTRY_USER, DATA_ENTRY_USER, AGENCY_BRANCH
X03: TERMINATION_DATE, HANDLING_DEPARTMENT, HANDLING_BRANCH, HANDLER, DATA_ENTRY_USER, DATA_ENTRY_USER, AGENCY_BRANCH
X04: DATE_ENTRY_DATE, HANDLING_DEPARTMENT, HANDLING_BRANCH, HANDLER, DATA_ENTRY_USER, AGENCY_BRANCH
```


- even if column does not exist in where clause, we can include that column into complex index for memory sort
- for example, DATA_ENTRY_USER does not exist as a index field, but to reduce sort byte operation, DATA_ENTRY_USER could be added

```sql
SELECT *
FROM Contracts
WHERE handling_branch_id = :trt_brch_id
AND APPLICATION_DATE BETWEEN :sbcp_dt1 AND :sbcp_dt2
AND INPU_DATE >= trunc(sysdate -3)
AND Contract_stauts_code in (:ctr_stat_cd1, :ctr_stat_dt2, :ctr_stat_cd3)
ORDER BY APPLICATION_DATE, DATA_ENTRY_USER
```


- but for that strategy to work, in clause must not be resolved as a index access
- for example, index consists of (RESIDENCE, BLOOD_TYPE, AGE)
    - BLOOD_TYPE has a high cardinality
    - and BLOOD_TYPE in A, O both is not well sorted even if index works
- in this case, u need to change index to (RESIDENCE, AGE, BLOOD_TYPE)
- then, order by opeartion is skipped and index filter works cuz BLOOD_TYPE is in index field

```sql
SELECT customer_no, customer_name, RESIDENCE
FROM customer
WHERE RESIDENCE = '서울'
AND BLOOD_TYPE IN ('A','O')
ORDER BY AGE
```

- if u decide to generate index, important thing to consider is how often that is used
- let's suppose these kind of queryies

```sql
WHERE customer_rating = :v1
AND   customer_no = :v2
AND   transaction_date >= :v3

WHERE customer_rating = :v1
AND customer_no  = :v2
AND transaction_date >= :v3
AND transaction_type = :v4

WHERE customer_rating = :v1
AND customer_no = :v2
AND transaction_date >= :v3
AND commodity_no = :v5

WHERE customer_rating = :v1
AND customer_no = :v2
AND transaction_date >= :v3
AND transaction_type = :v4
AND commodity_no = :v5
```

- customer_rating, customer_no, transaction_date is neccessary condition
  - customer_rating, customer_no are both  = condition
    - so customer_rating, customer_no doesnt care which column is leading
    - transaction_date is range scan so shoul be last field
  - transaction_type and commodity_no is optional condition
    - transaction_type and commodity_no is not index access condition
    - so transaction_type and commodity_no doesnt care which column is leading
- only (customer_rating, customer_no, transaction_date, transaction_type, commodity_no) index would be needed

<img src="/image/index-modeling.jpg"  />

- in case of followed, index is enough X03.
- X03 is included X01, X02

```
X01: contract_id, APPLICATION_DATE
X02: contract_id, APPLICATION_DATE, INSURANCE_START_DATE
X03: contract_id, APPLICATION_DATE, INSURANCE_START_DATE, INSURANCE_END_DATE
```

- in followed case, below seems different but it's almost duplicated especially when contract_id's cardinality is low(like PK)
- therefore, except for X05 left things could be removed

```
X01: contract_id, APPLICATION_DATE
X02: contract_id, INSURANCE_START_DATE
X03: contract_id, INSURANCE_END_DATE
X04: contract_id, DATA_ENTRY_DATE
X05: contract_id, APPLICATION_DATE, INSURANCE_START_DATE, INSURANCE_END_DATE, DATA_ENTRY_DATE
```

## <span style="color:#802548">_join_</span>
- for web, a small amount of data is read, so still NL join using index is better
- for batch, a big amount of data is read and updated, so Hash join using table scan is better
    - but to read big amount of data, it takes long time
    - so u need to use partition and paralleism
    - partition using index would become slower compare to using table full scan
        - partition's index is stored in each partition, so gets more long time
    - but partition divided by year using index and scan with year has high performance

- NL join
- MERGE join
- HASH join

## <span style="color:#802548">_NL join_</span>
- NL join is nested loop
- netsed loop is followed
  - at first, driving table is selected and read
  - for each row be read during scanning driving tables, probed table scan is executed
  - probed table scan is looped so u should use index
- but we dont know which table is driving table
- look at this query

```sql
SELECT e.employee_name, c.customer_name, c.phone_number
FROM employee e
inner join customer c
on e.employed_date >= '19960101'
AND c.management_employee_no = e.employee_no
```

- if driving table is employee like below execution plan, then employed_date condition is executed only once
- for found rows by employed_date, loop scan to probed table customer is executed corresponding to c.management_employee_no = e.employee_no condition 
- it means that, probed table must be applied index efficient scan
- if management_employee_no has no index, then loop scan would be very slow

```
|ID       |Operation                      |Name      |Rows     |Bytes     |Cost
------------------------------------------------------------------
0         |SELECT STATMEENT               |         |5         |58        |
1         | NESTED LOOPS                  |         |5         |58
2         |  TABLE ACCESS BY INDEX ROWID  |employee     |3          |20
3         |   INDEX RANGE SCAN            |employee_X1  |5
4         |  TALBE ACCESS BY INDEX ROWID  |customer     |5          |76
5         |   INDEX RANGE SCAN            |customer_X1  |8
```

- real process is like below
- find 1 record from driving table - > record finding c.management_employee_no = e.employee_no

<img src="/image/NL.jpg" />


- now let's see real exeuction plan by NL join
- look at ID number
  - scan does not be executed by sequential order
  - rather, it's order is 3 -> 2-> 1 -> 5 -> 4 
  - at first finding driving table rows and table access to driving(outer) table
    - Since 2,780 records were retrieved from the Employee index, the number of random accesses to the Employee table is also 2,780
    - However, only 3 rows were ultimately returned. 
  - and then nested loops, which means NL join
  - then finding probed tables rows and table access to probed(inner) table
This means that a large number of rows were filtered out after accessing driving table 
  - better to include table filter condition in the index to reduce the number of random accesses

```
|ID       |Operation                      |Name      |Rows     |Bytes     |Cost
------------------------------------------------------------------
0         |SELECT STATMEENT               |         |5         |58        |
1         | NESTED LOOPS                  |         |5         |58
2         |  TABLE ACCESS BY INDEX ROWID  |employee     |3          |20
3         |   INDEX RANGE SCAN            |employee_X1  |2780
4         |  TALBE ACCESS BY INDEX ROWID  |customer     |5          |76
5         |   INDEX RANGE SCAN            |customer_X1  |8
```

- if index column is included ?
- followed shows more detailed information
  - cr = buffer cache page
  - pr = disk page
  - pw = disk write
- index record and table access records are 102 and 105, so there is no table filtering
- it menas that index scan is not that inefficient
- but problem is that, with 2780 rows, resultset count is only 5
- it's better to change driving table if it can reduce inner loop count(2780)

```
|ROW      |Operation                      
------------------------------------------------------------------
5         | NESTED LOOPS(cr=112 pr=34 pw=0 time==122 us)                  
2780      |  TABLE ACCESS BY INDEX ROWID OF employee(cr=105 pr=32 pw=0 time=118 us)
2780      |   INDEX RANGE SCAN OF employee_X1(cr=102 pr=31 pw=0 time=16)            
5         |  TALBE ACCESS BY INDEX ROWID OF customer(cr=7 pr=2 pw=0 time=4 us)
8         |   INDEX RANGE SCAN OF customer_X1(cr=5 pr=1 pw=0 time=0 us)          
```

- when a large number of index records scan happens, prefetch and batch I/O happens
- developers cannot trigger but knowing basic can help, cuz it changes order
- so if there is possibility of prefetch and batch I/O is triggered, then u should write explicitly order by
- Prefetch and Batch I/O transform slow Random I/O into fast Sequential or Parallel I/O. 
  - Read Row 1 → Wait for Disk → Process Row 1 → Read Row 2 → Wait for Disk → Process Row 2
    - then CPU idle time is increased
  - so let trigger async I/O for CPU not to have idle time is prefetch 
    - CPU processing Row 1, background threads are already fetching Rows 2, 3, and 4 from the disk
- but still prefetch is single page I/O
- in other hand, batch I/O is based on multiple page I/O
  - just full table scan read blocks sequentially
  - but batch I/O, which means vector I/O uses an index first
    - "Out of 10,000 blocks in this table, you only need Blocks 5, 22, 23, 80, and 900."
    - those specific block numbers, sorts them (5 -> 22 -> 23 -> 80 -> 900)
    - and requests only those specific pages 
  - it Drastically Reduces OS Kernel Context switching
  - more hit rate of data pages like an array cache hit at the CPU level

- prefetch's execution plan is like followed

<img src="/image/prefetch.jpg" />

- batch I/O's execution plan is like followed

<img src="/image/batch-IO.jpg" />

- but the problem is that, Batch I/O harms order
- because betch I/O rearrange order by physical order

```text
Logical Index Order (Chronological):
[ Block 900 ] ──► [ Block 5 ] ──► [ Block 22 ]

           ▼ BATCH I/O RE-SORTS BY DISK GEOMETRY ▼

Physical Disk Order (Scrambled Chronology):
[ Block 5 ] ──► [ Block 22 ] ──► [ Block 900 ]
```

- but NL join always prceeds with prefetch/batch I/O together, u must use order by

```text
  [ STEP 1: INDEX RANGE SCAN ] 
               │
               ▼
  Index Prefetch loops ahead here to load upcoming INDEX PAGES.
               │
               ▼
  [ STEP 2: TABLE ACCESS BY INDEX ROWID ]
               │
               ├─► Data Page Prefetch loads upcoming DATA PAGES in index order.
               │
               └─► Batch I/O gathers row pointers, sorts them, 
                   and blasts a multi-block read for physical DATA PAGES.
```

- then why for small dataset, prefetch/Batch I/O is not recommended ?
  - The Cost of Memory Allocation and Sorting in batch I/O (Must collect a group of addresses, pause, and sort them )
  - Double Context-Switching Overhead in batch I/O (Vector I/O is expensive)
  - wasted memory buffer space in prefetch

## <span style="color:#802548">_hash join_</span>
- hash join has 2 steps
  - Build phase: The smaller table is read (as the build input) and a hash table (hash map) is created.
    - The hash table stores both the join key and all columns used in the SQL query.
    - If only `employee_no` were stored, additional table access would be required, which would introduce latch acquisition overhead.
    - This would effectively negate the advantages of a hash join, which is why only the join key is not sufficient.
    - Typically, the table with lower cardinality after applying filter conditions is chosen as the build input.  
  - Probe phase: The larger table (probe input) is read and joined by probing the hash table.
- Let’s look at the SQL example below.


```sql
SELECT /*+ ordered use_hash(c) */
        e.employee_no, e.employee_name, e.employed_date
        c.customer_no, c.customer_name, c.phone_number, c.final_order_amount
FROM employee e, customer c
WHERE c.management_employee_no = e.employee_no
AND e.employed_date >= '19960101'
AND e.department_code = 'Z123'
AND c.final_order_amount >=20000
```

- in build step, join column is used as a hash table key
- creating hash map is executed on small table, cuz it's efficient
  - big hash table -> memory usage up
  - big hash table -> a lot of probed table access
  - big has table -> in building, CPU cost up
- small table doesnt mean row count is small
- when filtered result set size is small, that is small table

```sql
SELECT employee_no, employee_name, employed_date
FROM employee
WHERE employed_date >= '19960101'
AND department_code ='Z123'
```

<img src="/image/hash-join-build-input.jpg" />


- scan through creaeted hash map

```sql
SELECT customer_no, customer_name, phone_number, final_order_amount, management_employee_no
FROM customer
WHERE final_order_amount >= 20000
```

<img src="/image/hash-join-probe-input.jpg" />

- hash join is latch-free, cuz hash table is created on private process memory
- hash join only reads small one, which means leads to high possibility of not converting to disk I/O
- hash join is used even though NL join is optimized, it is still slow
  - but for hash join, join is over, hash table is destroyed
  - it means every time that query is executed, CPU and memory is used and there is no caching 
    - therefore, this is for just batch process



## <span style="color:#802548">_sort merge join_</span>
- sort merge is used when we cannot use hash or NL join
- Oracle uses Private Process Memory for memory sort, not Shared Memory
- in this case, latch is not needed for sorting, which means high performance
- merge join's flow is like below
  - sort
  - merge

```sql
SELECT /*+ ordered use_merge(c) */
      e.employee_no, e.employee_name, e.employed_date
      , c.customer_no, c.customer_name, c.phone_number, c.final_order_amount
FROM employee e, customer c
WHERE c.management_employee_no = e.employee_no
AND e.employed_date >= '19960101'
AND e.department_code = 'Z123'
AND c.final_order_amount >= 20000
```

- above sql is resolved into below
- records by select query is stored in private process memory, which does not require latch
  - select itself needs latch
- and sorted by order by column

```sql
SELECT employee_no, employee_name, employed_date
FROM employee
WHERE employed_date >= '19960101'
AND department_code = 'Z123'
ORDER BY employee_no
```


- for counterpart table, same thing is executed

```sql
SELECT customer_no, customer_name, phone_number, final_order_amount, management_employee_no
FROM customer c
WHERE final_order_amount >= 20000
ORDER BY management_employee_no
```

- now merge
- this is internal logic

```sql
begin
  for outer in (select * from PGA_SORTED_employee)
  loop
    for inner in (SELECT * FROM PGA_SORTED_customer
                  WHERE management_employee_no = outer.employee_no)
    loop
      dbms_output.put_line(....);
    end loop;
  end loop;
end;
```

- this is fast cuz sorted data itself play a role as a index with sequential scan
  - it's easy to find starting poin cuz it has order
  - if join fails, it just stops and go to other records
  - no latch requires so very fast
  - no random access by each records
- but it's not that often used
  - sort-merge join must sort both tables, while hash doesnt
  - hash join can use latch-free 
  - therefore, in some rare cases sort-merge join is used
    - not mapped as a hash like inequality
    - data has index on order by column
    - memory shortage


## <span style="color:#802548">_DB parallellism_</span>
- in Db, there is parallellism system
  - it's about multi-proceesing one query using multiple CPU
- it's used during Full scan  large index range scan
- or GROUP BY / JOIN (hash join) /ORDER BY

```text
Single thread:
  100m row sequential handle

Parallel:
  Thread1: 0~25m
  Thread2: 25m~50m
  Thread3: 50m ~ 75m
  Thread4: 75m ~ 100m
```
- hash join usually use parallellism
  - it's full table scan
  - it's CPU heavy
- but the problem is that, db parallelism doesnt have so much impact when data skew happens
  - Thread1: 90%
  - Thread2: 5%
  - Thread3: 5%

- Range partitioning (주의 필요)
  - id 1~1000 → thread1
  - 1001~2000 → thread2
- 데이터 분포 문제 (가장 흔함)
  - status = 'ACTIVE' 90%
  - status = 'INACTIVE' 10%
- join key skew
- user_id = 1 (hot key)
  - 9-4. 실무에서 skew 줄이는 방법
  - better indexing
- selectivity 개선

2. composite key hashing
hash(user_id + tenant_id)

3. partitioning (물리 분산)
range / hash partition

4. query rewrite
filter 먼저 줄이고 join

5. avoid hot keys

이게 가장 중요

5. skew + parallel + hash join = 최악 조합

이게 실무에서 제일 중요하다.

상황
user_id skew 있음
join key skew 있음
결과
Thread1: hot key 80%
Thread2: 10%
Thread3: 10%
문제
한 thread만 계속 busy
나머지는 idle
memory grant는 전체 기준

👉 “parallel인데 single thread보다 느림”

parallel 좋은 경우
- full scan
- large aggregation
- big hash join

parallel 나쁜 경우
- OLTP (small queries)
- high skew data
- nested loop 중심

OLTP 셋팅
MAXDOP = 1~4
Cost threshold = 30~100
RCSI = ON (많이)

1. 배치 분리가 주는 진짜 이점
OLTP DB
- 짧은 SELECT / UPDATE
- latency 중요

Batch
- 대량 scan / update / join

분리하면:

CPU 경쟁 제거
lock 경쟁 감소
IO 경쟁 감소
parallelism 영향 제거

👉 OLTP 안정성 크게 올라감

(2) MAXDOP / parallel 정책 분리 가능

OLTP → MAXDOP 1~4
Batch → MAXDOP 8~16


(3) 스케줄링 가능

배치 = 새벽
OLTP = 낮


--------

(1) 데이터 동기화 문제

어떻게 옮길까?

replication
CDC
ETL
dual write

👉 항상 복잡도 증가


(2) 데이터 지연 (lag)

OLTP에서 변경
→ Batch DB 반영까지 delay

👉 실시간성이 깨짐

(3) 운영 비용 증가
DB 2개 운영
backup 2개
monitoring 2개
장애 대응 2배


(4) consistency 문제

OLTP 기준 데이터 vs Batch 기준 데이터 다름


3. 그래서 실무에서는 이렇게 나뉜다
A. 중소 / 일반 서비스 (가장 많음)

👉 DB 1개

대신:

RCSI ON
chunk batch
MAXDOP 조절
index tuning

B. 중간 규모

👉 read replica 분리

OLTP write → primary
read / batch → replica


C. 대형 시스템

👉 완전 분리

OLTP DB
Batch / Analytics DB
DW


4. 핵심 판단 기준
배치 분리하는 게 좋은 경우
- 배치가 OLTP 성능을 실제로 죽이고 있음
- CPU / IO contention 심함
- batch window 길어짐
- scale이 커짐 (수백만~수천만 row)

굳이 분리 안 하는 경우
- 아직 데이터 작음
- 운영 복잡도 줄이는 게 중요
- real-time consistency 필요


## <span style="color:#802548">_nested subquery_</span>
- recent optimizer proceeds query transformation 
- problem is that, subquery is divided by modules
- subquery has 3 types
  - select: scalar subquery --> dont use this one
  - from: inline view --> this is for limiting count of driving table
  - where: nested subquery
- subquery is divided by unnset and filter
- unnset resolves subquery into join

```sql
SELECT c.customer_no, c.customer_name
FROM customer c
WHERE c.entry_date >= trunc(add_months(sysdate, -1), 'mm')
AND EXISTS (
  SELECT /*+ no_unnest */ 'x'
  FROM transactions
  WHERE customer_no = c.customer_no
  AND transaction_date >= trunc(sysdate, 'mm')
)
```

- look at this query

```sql
SELECT c.customer_no, c.customer_name
FROM customer c
WHERE c.entry_date >= trunc(add_months(sysdate, -1), 'mm')
AND EXIST (
      SELECT /*+ unnest nl_sj */ 'x'
      FROM transaction
      WHERE customer_no = c.customer_no
      AND transaction_date >= trunc(sysdate, 'mm')
    )
```

- look at nested Loops(SEMI)
- it means subquery is resolved into join

```
Execution plan
------------------------------------------
0     SELECT STATEMENT 
1       NESTED LOOPS (SEMI)
2         TABLE ACCESS (BY INDEX ROWID) OF 'customer'
3           INDEX (RANGE SCAN) OF 'customer_X01'
4         INDEX (RANGE SCAN) OF 'transaction_X01' 
```

- if u use EXIST, it almost perform semi join, which is equal to NL join

```sql
SELECT /*+ leading(transaction@subq) use_nl(c) */ c.customer_no, c.customer_name
FROM customer c
WHERE c.entry_date >= trunc(add_months(sysdate, -1), 'mm')
AND EXISTS (
  SELECT /*+ qb_name(subq) unnest */ 'x'
  FROM transaction
  WHERE customer_no = c.customer_no
  AND transaction_date >= trunc(sysdate, 'mm')
)
```

- unnest is usually better than filter
- with unnest, optimizer can perform better optimization
  - but when using group by, distinct, window function in subqeruy -> unnest fails so resolved into filter
  - using NOT in and that column is nullable -> unnest fails
    - use not exists or adding where column is not null
  - using calculation like where (select age from) + 10
  - using limiting count like ROWNUM / LIMIT keyword

- it is needed to avoid below query
- even if using hint, unnest dont happen

```sql
SELECT post_no, title, writer, data_entry_date
FROM board b
WHERE board_type = 'notice'
AND data_entry_date >= trunc(sysdatet -1)
AND EXISTS (
        SELECT /*+ unnest nl_sj */ 'x'
        FROM receive_target
        WHERE post_no = b.post_no
        AND receiver = :memb_no
        AND ROWNUM <=1
)
```

<img src="/image/subquery-join.jpg" />


- but sometimes filter is better, especially needs to reduce amount of rows in driving table
- look at this query

```sql
SELECT /*+ leading(p) use_nl(t) */ count(distinct p.commodity_no), sum(t.order_amount)
FROM commodity p
INNER JOIN order t
on t.commodity_no = p.commodity_no
AND p.data_entry_date >= trunc(add_months(sysdate, -3), 'mm')
AND t.order_date >= trunc(sysdate - 7)
AND exists (SELECT  'x'
            FROM commoditiy_type
            WHERE commoditiy_type_code = p.commoditiy_type_code
            AND parent_code = 'AK'
          )
```

- it's execution plan shows unnest(NESTED LOOPS) 
- FILTER is table filter predicate
- cuz p is driving table, probed table is order
- so, on p.commodity_no = t.commodity_no, index is on when t.commdity_no looks up rows

```
--------------------------------------------------------------
| ROW  | Operation                                   | Name        
--------------------------------------------------------------
|   0  | SELECT STATEMENT                            |              
|   1  | SORT AGGREGATE(cr=38103)                    |              
| 3000 | FILTER(cr=38103)                            |             
| 60000|   NESTED LOOPS(cr=38097)                    |             
| 1000 |     TABLE ACCESS FULL(cr=95)                | commodity        
| 60000|       TABLE ACCESS BY INDEX ROWID(cr=38002) | order      
| 60000|         INDEX RANGE SCAN(cr=2002)           | order_PK   
|    1 |       TABLE ACCESS BY INDEX ROWID(cr=6)     | commoditiy_type
|    3 |         INDEX UNIQUE SCAN(cr=3)             | commoditiy_type_PK
--------------------------------------------------------------
```


- let it filter first with hint(NO_UNNEST PUSH_SUBQ)

```sql
SELECT /*+ leading(p) use_nl(t) */ count(distinct p.commodity_no), sum(t.order_amount)
FROM commodity p
INNER JOIN order t
on t.commodity_no = p.commodity_no
AND p.data_entry_date >= trunc(add_months(sysdate, -3), 'mm')
AND t.order_date >= trunc(sysdate - 7)
AND exists (SELECT /*+ NO_UNNEST PUSH_SUBQ*/ 'x'
            FROM commoditiy_type
            WHERE commoditiy_type_code = p.commoditiy_type_code
            AND parent_code = 'AK'
          )
```

- then performance becomes very high

```
--------------------------------------------------------------
| ROW  | Operation                                          | Name        
--------------------------------------------------------------
|   0 | SELECT STATEMENT                                    |              
|   1 |   SORT AGGREGATE(cr=1903)                           |              
| 3000|     NESTED LOOPS(cr=1903)                           |             
| 150 |       TABLE ACCESS FULL commodity(cr=101)                 | commodity      
| 1   |         TABLE ACCESS BY INDEX ROWID commoditiy_type(cr=6)   | commoditiy_type
| 3   |           INDEX UNIQUE SCAN commoditiy_type_PK(cr=3)        | commoditiy_type_PK  
| 3000|       TABLE ACCESS BY INDEX ROWID order(cr=1802)      | order      
| 3000|         INDEX RANGE SCAN order_PK(cr=302)             | order_PK   
--------------------------------------------------------------
```

## <span style="color:#802548">_join_</span>

- if u dont like hint, u can change it to JOIN
  - with joining first with commoditiy_type, we can reduce driving table row
  - unlike exists, join has no early-termination
  - so join can inflate rows but in when join coilumn is PK, dont need to worry
  - cuz PK is 1 : 1, so no worry about inflating rows
- and with inner join, optimizer can choose various options
  - if early-termination has more benefits, then optimizer would choose semi-join like exists
  - if driving table change has more benefits, then optimizer would choose change
  - it only happens when join keyword is used, not when subquery
- in inner join, on clause and where clause is equivalent to optimizer
  - writing conditions on each is just for sql's readability
  - but in left join, logical order is very important so on and where leads to different resultset

```sql
SELECT count(distinct p.commodity_no), sum(t.order_amount)
FROM commodity p
INNER JOIN commoditiy_type c
   ON c.commoditiy_type_code = p.commoditiy_type_code
  AND c.parent_code = 'AK'
INNER JOIN order t
   ON t.commodity_no = p.commodity_no
WHERE p.data_entry_date >= trunc(add_months(sysdate, -3), 'mm')
  AND t.order_date >= trunc(sysdate - 7);
```

- be careful when u use left join 
- left join becomes dangerous when
  - using probed table condition in where clause (t.order_date >=)
  - when order_date is nullable, where condition filter null then LEFT join is now same as inner join

  ```sql
  FROM commodity p
  LEFT JOIN order t ON t.commodity_no = p.commodity_no
  WHERE t.order_date >= sysdate - 7
  ```

  - using driving table condition on clause (p.data_entry_date >=)
  - left join doesnt filter rows, it fills data with null

    ```sql
    FROM commodity p
    LEFT JOIN order t ON t.commodity_no = p.commodity_no
    AND p.data_entry_date >= ...
    ```

## <span style="color:#802548">_inline view_</span>
- before left join, if possible, we'd better to reduce joined row
- and better to use inline view in that case

```sql
SELECT count(distinct p.commodity_no), sum(t.order_amount)
FROM (
    -- reduce data at first step
    SELECT commodity_no, commoditiy_type_code
    FROM commodity
    WHERE data_entry_date >= trunc(add_months(sysdate, -3), 'mm')
      AND exists (SELECT 'x'
                  FROM commoditiy_type
                  WHERE commoditiy_type_code = commodity.commoditiy_type_code
                    AND parent_code = 'AK')
) p
-- left join with small dataset
LEFT JOIN order t
  ON t.commodity_no = p.commodity_no
 AND t.order_date >= trunc(sysdate - 7);
```

- but in some case, we want to merge where condition to inline view
- cuz to do that, we can even reduce more unneccesary rows
- in terms of total sql, we should filter customer whether he is new or not
- but inline view does not know it so just scan

```sql
SELECT c.customer_no, c.customer_name, t.average_transaction, t.minimum_transaction, t.maximum_distance
FROM customer c
inner join 
        (SELECT customer_no, avg(transaction_count) average_transaction,
                min(transaction_count) minimum_transaction, max(transaction_count) maximum_transaction
        FROM transaction
        WHERE transaction_date >= trunc(sysdate, 'mm') /**today transaction */
        group by customer_no) t
on t.customer_no = c.customer_no
WHERE c.entry_date >= trunc(add_months(sysdate, -1) ,'mm') /**before 1 month customer entry */
```

- in this case, we can use merge or pushdown hint
  - merge performs merge of driving and probed into inner join , so group by operation at the end 
    - if driving table is big and probed table is small, bulk group by is better
  - pushdown keeps strucutre and just merge that condition, so group by operation before join for each records
    - if driving table's specific row have possibility of inflating, then group by for each records is better cuz it reduces join load

```sql
SELECT c.customer_no, c.customer_name, t.average_transaction, t.minimum_transaction, t.maximum_transaction
FROM customer c,
      (SELECT /*+ merge or + no_merge push_pred */ customer_no, avg(transaction_count) average_transaction,
        min(transaction_count) minimum_transaction, max(transaction_count) maximum_transaction
        FROM transaction
        WHERE transaction_date >= trunc(sysdate, 'mm')  /**today transaction */
        GROUP BY customer_no) t
WHERE c.entry_date >= trunc(add_months(sysdate , -1), 'mm') /**before 1 month customer entry */
AND t.customer_no = c.customer_no
```

- now execution plan would be like below

<img src="/image/inline-view-merge-hint.jpg" />
<img src="/image/pushdown-predicate.jpg" />

- but above hint is only in Oracle
- in other db, scalar subquery was used
- but scalar subquery has serious problem that it repeats duplicated read
- below query scan is 4 scan
  - customer 
  - trnasction for avg
  - transaction for min
  - trnasction for max

```sql
SELECT c.customer_no, c.customer_name
        ,(SELECT AVG(transaction_count) FROM transaction
          WHERE transaction_date >= trunc(sysdate, 'mm') AND transaction_no = c.customer_no)
        ,(SELECT MIN(transaction_count) FROM transaction
          WHERE transaction_date >= trunc(sysdate, 'mm') AND transaction_no = c.customer_no)
        ,(SELECT max(transaction_count) FROM transaction
          WHERE transaction_date >= trunc(sysdate, 'mm') AND customer_no = c.customer_no)
FROM customer c
WHERE c.entry_date >= trunc(add_months(sysdate, -1), 'mm')
```

- so, LPAD with substr is better choice
  - customer 
  - transaction for avg, max, min

```sql
SELECT customer_no, customer_name
      , to_number(substr(transaction_count, 1, 10))  average_transaction_count
      , to_number(substr(transaction_count, 11, 10)) minimum_transaction_count
      , to_number(substr(transaction_count, 21))     maximum_transaction_count
FROM (
  SELECT c.customer_no, c.customer_name
        , (SELECT LPAD(AVG(transaction_count), 10) || LPAD(MIN(transaction_count), 10) || MAX(transaction_count)
            FROM transaction
            WHERE transaction_date >= trunc(sysdate, 'mm')
            AND customer_no = c.customer_no) transaction_count
  FROM customer c
  WHERE c.entry_date >= trunc(add_months(sysdate, -1), 'mm')
)
```

- nowadays, we can use LATERAL keyword
- it does not need LPAD or TO_NUMBER
- for both, push down is activated so no difference on that aspect
- and if amount digit over 10, LPAD logic would break
- so it's better to use LEFT JOIN LATERAL

```sql
SELECT c.customer_no, c.customer_name, 
       t.average_transaction, t.minimum_transaction, t.maximum_transaction
FROM customer c
LEFT JOIN LATERAL (
    SELECT AVG(transaction_count) average_transaction,
           MIN(transaction_count) minimum_transaction, 
           MAX(transaction_count) maximum_transaction
    FROM transaction
    WHERE transaction_date >= TRUNC(SYSDATE, 'mm')
      AND customer_no = c.customer_no -- 밖에서 필터링된 신규 고객번호가 이 안으로 쏙 들어옵니다 (Push down)
) t ON 1=1
WHERE c.entry_date >= TRUNC(ADD_MONTHS(SYSDATE, -1), 'mm');
```

- but that doesnt prevent loop
- it means that when many customer is joined, random access during join would increase, which leads to performance disaster
- in that case, simple left join with group by is better choice
  - and this is quite popular cuz it does not rely on hint and applicapable on all db
  - but if new customer actively perform transaction, then joined row would increase
- so when joined driving table final records is very big, use simple left join
- if driving table is small, then use LEFT JOIN LATERAL

```sql
SELECT c.customer_no, c.customer_name, 
       AVG(t.transaction_count) average_transaction,
       MIN(t.transaction_count) minimum_transaction, 
       MAX(t.transaction_count) maximum_transaction
FROM customer c
left JOIN transaction t 
   ON t.customer_no = c.customer_no
  AND t.transaction_date >= trunc(sysdate, 'mm') 
WHERE c.entry_date >= trunc(add_months(sysdate, -1), 'mm') 
GROUP BY c.customer_no, c.customer_name;
```



## <span style="color:#802548">_sort_</span>
- if db needs sorted or modified columns, they use sort area
- at first, mermoy if memory shortage happens, then disk
  - sort merge join
  - hash join
  - group by
  - order by

<img src="/image/sort-process.jpg" />

## <span style="color:#802548">_keyword that must avoid in sorting_</span>

- dont use union, minus, distinct as possible as u can
  - union -> union all
  - distinct -> exists
  - minus -> not exists
  - hash join -> NL join except for batch

- before removing distinct, look at this query

```sql
SELECT DISTINCT p.commodity_no, p.commodity_name, p.commodity_price
FROM commodity p, Contract c
WHERE p.commodity_type_code = :pclscd
AND c.commodity_no = p.commodity_no
AND c.contract_date BETWEEN :dt1 and :dt2
AND c.contract_classification _code = :ctpcd
```


- use sxists can benefit from 
  - partial range scan
  - stop at condition that do not satisfy

```sql
SELECT p.commodity_no, p.commodity_name, p.commodity_price
FROM commodity p
WHERE p.commodity_type_code = :pclscd
AND EXISTS (
          SELECT 'x' FROM Contract c
          WHERE c.commodity_no = p.commodity_no
          AND c.contract_date BETWEEN :dt1 and :dt2
          AND c.contract_classification _code = :ctpcd
        )
```

- before removing minus, look at this query

```sql
SELECT ST.case_no, ST.serinal_no, ST.case_code
FROM observation_progress ST
WHERE case_code = '0001'
AND observation_date BETWEEN :V_TIMEEFROM || '000000' AND :V_TIMET0 || '235959'

MINUS

SELECT ST.case_no, ST.serinal_no ,ST.case_code, ST.observation_date
FROM observation_progress ST, rescue RPT
WHERE case_code = '0001'
AND observation_date BETWEEN :V_TIMEEFROM || '000000' AND :V_TIMET0 || '235959'
AND RPT.출동센터ID = :V_CNTR_ID
AND ST.case_no = PRT.case_no
ORDER BY case_no, observation_date
```


- like exists, use not sxists can benefit from 
  - partial range scan
  - stop at condition that do not satisfy

```sql
SELECT ST.case_no, ST.serinal_no ,ST.case_code, ST.observation_date
FROM observation_progress ST, rescue RPT
WHERE case_code = '0001'
AND observation_date BETWEEN :V_TIMEEFROM || '000000' AND :V_TIMET0 || '235959'
AND NOT EXISTS (
              SELECT 'X' FROM rescue
              WHERE 출동센터ID = :V_CNTR_ID
              AND case_no = ST.case_no
            ) 
ORDER BY ST.case_no, ST.observation_date
```


## <span style="color:#802548">_minimizing sortByte_</span>
- if u need sort, minimize sortByte
- otherwise, memory sort could be disk I/O
- increasing space like LPAD is not good idea, cuz byte increase

```sql
SELECT LPAD(commodity_no, 30) || LPAD(commodity_name, 30) || LPAD(customer_id, 10)
        ||  LPAD(customer_name, 20) || TO_CHAR(order_date, 'yyyymmdd hh24:mi:ss')
FROM order_item
WHERE order_date BETWEEN :start and :end
ORDER BY commodity_no
```

- sort is better to be done with basic column

```sql
SELECT LPAD(commodity_no, 30) || LPAD(commodity_name, 30) || LPAD(customer_id, 10)
        ||  LPAD(customer_name, 20) || TO_CHAR(order_date, 'yyyymmdd hh24:mi:ss')
FROM (
  SELECT commodity_no, commodity_name, customer_id, customer_name, order_date
  FROM order_item
  WHERE order_date BETWEEN :start and :end
  ORDER BY commodity_no
)
```

- but without hint, optimizer can merge view, which is called simple view merging
- therefore, we need to use hint

```sql
SELECT LPAD(commodity_no, 30) || LPAD(commodity_name, 30) || LPAD(customer_id, 10)
        ||  LPAD(customer_name, 20) || TO_CHAR(order_date, 'yyyymmdd hh24:mi:ss')
FROM (
  -- The NO_MERGE hint prevents the optimizer from flattening this block
  SELECT /*+ NO_MERGE */ 
         commodity_no, commodity_name, customer_id, customer_name, order_date
  FROM order_item
  WHERE order_date BETWEEN :start AND :end
  ORDER BY commodity_no
);
```

- more advanced way is sorting with only minimized column like rowid
- then sort byte drastically decrease

```sql
SELECT LPAD(order_item_table.commodity_no, 30) 
       || LPAD(order_item_table.commodity_name, 30) 
       || LPAD(order_item_table.customer_id, 10)
       || LPAD(order_item_table.customer_name, 20) 
       || TO_CHAR(order_item_table.order_date, 'yyyymmdd hh24:mi:ss')
FROM (
  -- 1. This inner subquery finds and sorts ONLY the row IDs
  SELECT rowid AS sorted_row_id
  FROM order_item
  WHERE order_date BETWEEN :start AND :end
  ORDER BY commodity_no
) AS sorted_keys
-- 2. This joins the sorted IDs back to the original table using a clear alias
JOIN order_item AS order_item_table 
  ON order_item_table.rowid = sorted_keys.sorted_row_id;
```

- it menas that selected column should not be *
- only writing needed column minimize sort byte

```sql
SELECT account_no, total_deposit
FROM deposit_ledger
ORDER BY total_deposit DESC
``` 

- instead of using max, using window function is good practice
- it reduces sort load, using top n algorithm
- for verifying, let's reduce memory space

```sql
alter session set workarea_size_policy = manual;
alter session set sort_area_size = 524288;
```

- max triggers a lot of disk I/O

```sql
SELECT equipment_no, change_date, change_history_order, 상태코드, 메모
FROM (SELECT equipment_no, change_date, change_history_order, 상태코드, 메모
            , MAX(change_history_order) OVER(PARTITION BY equipment_no) 최종change_history_order
            FROM change_history
            WHERE change_date = :upd_dt)
WHERE change_history_order = 최종change_history_order
```


- rank doesnt

```sql
SELECT equipment_no, change_date, change_history_order, 상태코드, 메모
FROM (SELECT equipment_no, change_date, change_history_order, 상태코드, 메모
            , RANK(change_history_order) OVER(PARTITION BY equipment_no ORDER BY change_history_order DESC) 최종change_history_order
            FROM change_history
            WHERE change_date = :upd_dt)
WHERE change_history_order = 최종change_history_order
```


## <span style="color:#802548">_redo/undo log and commit_</span>
- when performing DML, db writes redo log
  - database recovery
  - cache recovery
- at first, write it in memory and commit it with async I/O to disk
- we can adjust redo log file size and storage period
- to reflect modified configuration of redo log, db server restart is needed

- when performing DML, undo log is created either
  - transaction rollback
  - read consistency(MVCC)
- we can adjust undo log file size and storage period
- to reflect modified configuration of undo log, db server restart is needed
  - if transaction is too long, undo log space woudld experience shortage, which means bottleneck
  - but if too small, cuz disk I/O too often occurs, which means bottlenekc

- commit follow procedure below
  - DML execution -> write change to redo log
  - change data in memory
  - commit
  - LGWR process write redo log content on log file
  - DBWR process reflect data change in memory to disk


<img src="/image/dml-process.jpg" />


## <span style="color:#802548">_batch strategy_</span>


## <span style="color:#802548">_insert batch: drop constraint - reactivating constraint_</span>
- for batch program, index and data integrity constraint degrade DML performance
- let's test without index and data integrity, how much it would become fast

```sql
CREATE TABLE source
as 
SELECT b.no, a.*
FROM (SELECT * FROM emp WHERE rownum <= 10) a
    ,(SELECT ROWNUM AS no FROM dual CONNECT BY LEVEL <= 1000000) b;

CREATE TABLE target
AS
SELECT * FROM source WHERE 1 = 2;

ALTER TABLE target ADD CONSTRAINT
target_pk PRIMARY KEY(no, empno);

CREATE INDEX target_x1 on target(ename);
```

- with PK and INDEX, it takes 1m 19s

```sql
SET TIMING ON;

INSERT /*+ append */ into target
SELECT * FROM source;

commit;
```

- drop PK and index constraints

```sql
TRUNCATE TABLE target;
ALTER TABLE target MODIFY CONSTRAINT target_pk DISABLED DROP index; 
ALTER INDEX target_x1 UNUSABLE;                                     

ALTER SESSION SET skip_unusable_indexes = true;                     
```

- it takes 5s

```sql
SET TIMING ON;

INSERT /*+ append */ into target
SELECT * FROM source;

commit;
```

- and then recreate PK and index
- recreation takes 8s
- therefore, for batch, dropping constraints - inserting - reactivating constraint can be a good choice 

```sql
ALTER TABLE target MODIFY CONSTRAINT target_pk enable NOVALIDATE; 
ALTER INDEX target_x1 rebuild;                                    -
```



## <span style="color:#802548">_update batch: multiple set - hash join hint - updatable join view - merge into_</span>
- traditional update sql requires scalar subquery, which leads several table scan

```sql
UPDATE customer c
SET     final_tranascation_date = (SELECT MAX(transaction_date) FROM transaction
                        WHERE customer_no = c.customer_no
                        AND transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1)))
        , current_transaction_count = (SELECT COUNT(*) FROM transaction
                          WHERE customer_no = c.customer_no
                          AND transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1)))
        , current_transaction_count = (SELECT SUM(transaction_count) FROM transaction
                          WHERE customer_no = c.customer_no
                          AND transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1)))
WHERE exists (SELECT 'x' FROM transaction
              WHERE customer_no = c.customer_no
              AND transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1)))                        
```

- we can reduce scan amount using multiple set
- but this query would have problem either when there are many customer's rows

```sql
UPDATE customer c
set (final_tranascation_date, current_transaction_count, current_transaction_count) =
    (SELECT MAX(transaction_date), count(*), sum(transaction_count)
      FROM transaction
      WHERE customer_no = c.customer_no
      AND transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1)))
WHERE EXISTS (SELECT 'x' FROM transaction
              WHERE customer_no = c.customer_no
              AND transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1)))
```

- in that case, we can force hash join with hint
- so we can find out update target very fast
- but still for calculating set columns, we need to call subquery by that amount of row
- for example, we find out 100,000 customer update target very quick
- but for 100,000 customer, we must call subquery which means 100,000 scans

```sql
UPDATE customer c
set (final_tranascation_date, current_transaction_count, current_transaction_count) =
    (SELECT MAX(transaction_date), count(*), sum(transaction_count)
      FROM transaction
      WHERE customer_no = c.customer_no
      AND transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1)))
WHERE EXISTS (SELECT /*+ unnest hash_sj */'x' FROM transaction
              WHERE customer_no = c.customer_no
              AND transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1)))
```

- if using updatable join view, subquery is not needed
- 100,000 index random scan  -> 1 table hash join table full /index range scan according to index
- however, for updatable join view, inline view table must be unique to customer
  - in basically, customer : transaction is 1 : M
  - in updatable join view, we cannot update 1 table records
    - after joining, customer table would inflate rows
    - and oracle doesnt know which transaction record is standard
    - in original, we cannot update customer
  - but using group by like below query, if we can force to ensure that transaction is unique to customer, then 1 : 1 relation
    - at that time, we can use updatable join view to update customer
    - so group by keyword is used for transaction table
    - of course, transaction's joined column with customer should be unique
    - for this case, selected coulmn is standard and aggregate column, so no problem with using group by

```sql
UPDATE
  (SELECT /*+ ORDERED USE_HASH(c) no_merge(t) */
          c.final_tranascation_date
        , c.current_transaction_count
        , c.current_transaction_count
        , t.transaction_date
        , t.transaction_count
        , t.transaction_count
   FROM (SELECT customer_no
              , MAX(transaction_date) transaction_date
              , COUNT(*) transaction_count
              , SUM(transaction_count) transaction_count
         FROM transaction
         WHERE transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1))
         GROUP BY customer_no
        ) t
   INNER JOIN customer c 
   ON c.customer_no = t.customer_no
  )
SET final_tranascation_date   = transaction_date
  , current_transaction_count  = transaction_count
  , current_transaction_count = transaction_count;
```

- but better to use merge into, cuz it does not have any kind of 1 : M problem

```sql
MERGE /*+ LEADING(t) USE_HASH(c) */ INTO customer c
USING (
    SELECT customer_no
         , MAX(transaction_date) AS max_date
         , COUNT(*) AS t_count
         , SUM(transaction_count) AS t_amt
    FROM transaction
    WHERE transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1))
    GROUP BY customer_no
) t
ON (c.customer_no = t.customer_no)
WHEN MATCHED THEN
UPDATE SET 
    c.final_tranascation_date   = t.max_date,
    c.current_transaction_count  = t.t_count,
    c.current_transaction_count = t.t_amt;
```

## <span style="color:#802548">_select batch: direct path I/O for_</span>
- but above query must go throuh buffer cache
- but for batch process, it's almost temporary data
- so, better to get data directly to own session memory
- this is direct path I/O

```sql
SELECT /*+ full(t) parallel(t 4)  */ * FROM big_table t;
```

- we can decide extent of parallel
- 4 parallel upgrade performance by 10 times more, not 4 time
  - no buffer cache store process
  - no undo logging
- but it has some drawbacks
  - X lock, which means u need to do it in night
  - waste table space even if table has space

- for insert, use append can upgrade performance
  - append skips finding a new space that takes much time, they insert data into end of table
  - it does not take time

```sql
INSERT /*+ APPEND */ INTO target_table
SELECT /*+ FULL(source) PARALLEL(source 4) */ * FROM source_table;
```

- for update, delete use hint FULL(table) PARALLEL(table extent) is enough

```sql
UPDATE
  (SELECT /*+ ORDERED USE_HASH(c) no_merge(t) */
          c.final_tranascation_date
        , c.current_transaction_count
        , c.current_transaction_amount
        , t.transaction_date
        , t.transaction횟수
        , t.transaction_amount
   FROM (SELECT /*+ FULL(transaction) PARALLEL(transaction 4) */
                customer_no
              , MAX(transaction_date) transaction_date
              , COUNT(*) transaction횟수
              , SUM(transaction_amount) transaction_amount
         FROM transaction
         WHERE transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1))
         GROUP BY customer_no
        ) t
   INNER JOIN customer c 
   ON c.customer_no = t.customer_no
  )
SET final_tranascation_date   = transaction_date
  , current_transaction_count  = transaction횟수
  , current_transaction_amount = transaction_amount;
```

- for merge, 

```sql
MERGE /*+ LEADING(t) USE_HASH(c) */ INTO customer c
USING (
    SELECT /*+ FULL(transaction) PARALLEL(transaction 4) */ 
           customer_no
         , MAX(transaction_date) AS max_date
         , COUNT(*) AS t_count
         , SUM(transaction_amount) AS t_amt 
    FROM transaction
    WHERE transaction_date >= TRUNC(ADD_MONTHS(sysdate, -1))
    GROUP BY customer_no
) t
ON (c.customer_no = t.customer_no)
WHEN MATCHED THEN
UPDATE SET 
    c.final_tranascation_date    = t.max_date,
    c.current_transaction_count  = t.t_count,
    c.current_transaction_amount = t.t_amt;

```



## <span style="color:#802548">_direct path I/O를 이용한 데이터 삽입_</span>
- 비파티션의 경우에는 index를 unusable 하는 방식을 사용한다.
- 프로세스는 아래와 같다.

```
테이블을 nologging 모드로 전환
2. index를 unusable 상태로 전환
3. direct path I/O로 데이터 입력
4. no logging 모드로 index 재생성
5. nologging 모드를 logging모드로 전환
```


- sql로는 아래와 같다.

```sql
ALTER TABLE taregt_t nologging;

ALTER INDEX target_t_x01 unusable;

INSERT /*+ append   */ INTO target_t
SELECT /*+ FULL(source_t) PARALLEL(source_t 4) */ * FROM source_t;

ALTER INDEX target_t_x01 REBUILD nologging;

ALTER TABLE target_t logging;
ALTER INDEX target_t_x01 logging;
```


- partition의 경우, 보통 엔간하면 index를 unusable로 전환하지 않고 insert를 하지만, 조건이 만족된다면 unusable을 한다.
  - table이 partitioned다.
  - index가 local partition이다.
- 프로세스는 아래와 같다.
- 아까전과 크게 다른 게 거의 없다. 그저 작업이 파티션 단위로 이뤄지는 점만 다르다.

```sql
ALTER TABLE taregt_t MODIFY partition p_201712 nologging;

ALTER INDEX target_t_x01 MODIFY partition p_201712 unusable;

INSERT /*+ append   */ INTO target_t
SELECT /*+ FULL(source_t) PARALLEL(source_t 4) */ * FROM source_t WHERE dt BETWEEN '20171201' AND '20171231';

ALTER INDEX target_t_x01 REBUILD partition p_201712 nologging;

ALTER TABLE target_t MODIFY PARTITION p_201712 logging;
ALTER INDEX target_t_x01 MODIFY partition p_201712 logging;
```


## <span style="color:#802548">_range partition_</span>
- 파티셔닝은 테이블 또는 index data를 특정 컬럼(파티션 키) 값에 따라 별도 segment에 나눠 저장하는 것을 의미한다.
- 파티션이 좋은 이유는 아래와 같다.
  - management_의 측면에서 파티션 단위로 백업, 추가 ,삭제 등이 일어나면 가용성이 높다.
  - 성능의 측면에서 파티션 단위로 조회하고 경합하기 때문에 전체 table을 모두 조회하지 않는다.
- 테이블 파티션의 종류는 3가지가 있다. 인덱스 파티션하고는 다르다.
  - range
  - 해쉬
  - 리스트
- range는 주로 날짜 컬럼을 기준으로 파티셔닝한다.
```sql
CREATE TABLE order (order번호 number , order일자 varchar2(8), ...)
partition by range(order일자) (
  partition P2017_Q1 values less than ('20170401')
  partition P2017_Q2 values less than ('20170701')
  partition P2017_Q3 values less than ('20171001')
  partition P2017_Q4 values less than ('20180101')
)
```

- 파티션을 만들면 아래와 같은 이미지로 형성된다고 보면 된다.
- 파티션이 있으면 원하는 파티션만 골라 읽을 수 있어 full scan의 성능이 크게 향상된다.
<img src="/image/partition.jpg" />

- 실제 파티션의 성능 향상 원리가 파티션 pruning에 있다.
- SQL 하드파싱이나 실행 시점에 조건절을 분석한다.
- 읽지 않아도 되는 파티션은 액세스 대상에서 제외해버린다.
- 예를 들어보자.
  - 1200만건의 25%를 index를 써서 조회하자니 full scan보다 느리다.
  - 그렇다고 full scan하면 사이즈가 너무 커서 부담스럽다.
  - 그럴 때 100만건 단위로 파티션을 나누면 일부 파티션만 읽으면 되기에 성능이 좋아진다.
  - 특히 병렬처리와 같이 이뤄진다면 더더욱 성능이 좋다.

<img src="/image/partition-fullscan.jpg" />


## <span style="color:#802548">_hash partition_</span>
- hash partition 은 PK처럼 변별력이 좋고 데이터 분포가 고른 컬럼을 파티션 기준으로 선정해야 한다.
- 반드시 = 조건으로 풀려야 한다.


## <span style="color:#802548">_list partition_</span>
- 사용자가 정의한 그룹핑 기준에 따라 데이터를 분할한다.
```sql
CREATE TABLE 인터넷매물 (물건코드 varchar2(5), 지역분류 varchar2(4), ...)
PARTITION BY list(지역분류) (
  partition P_지역1 values ('서울')
, partition P_지역2 values ('경기', '인천'
, partition P_지역3 values ('부산', '대구', '대전', '광주')
, partition P_기타 values (DEFAULT) 
  )
);
```

- 이 경우에는 될 수 있으면 값이 최대한 고르게 나오게끔 data를 유도하는 게 좋다.
- 비즈니스 도메인용으로 쓰이는 테이블 파티션 종류다.
  



## <span style="color:#802548">_index partition_</span>
- 인덱스 파티션은 3가지로 나뉜다.
- local partitioned index
  - table partition과 index partition이 같은 경우다.
  - index 만들면서 뒤에 LOCAL을 붙여주면 끝이다.
  - 오라클이 자동 management_하며, table partition 구성을 변경해도 index 재생성 안해도 됨
  - 변경작업이 매우 빠르게 끝나므로 피크시간대만 피하면 된다.

<img src="/image/local-partitioned-index.jpg" />


- global partitioned index
  - table partition과 index partition이 다른 경우다.
  - index 만들면서 뒤에 GLOBAL을 붙여주면 끝이다.
  - 비파티션 테이블에도 사용가능하다.
  - 테이블 구성이 바뀌는 순간 index를 재생성해줘야 한다.

<img src="/image/global-partitioned-index.jpg" />


- non-partitioned index
  - 일반 create index 문이다.
  - 테이블 파티션 구성을 바꾸는 순간 인덱스를 재생성해야한다.


<img src="/image/global-non-partitioned-index.jpg" />


```sql
CREATE INDEX order_X01 on order (order일자, order_amount) LOCAL

CREATE INDEX order_X03 on order (order_amount, order일자) GLOBAL
PARTITION BY RANGE(order_amount) (
  PARTITION P_01 values less than (100000)
  PARTITION P_MX values less than (MAXVALUE)
)

CREATE INNDEX order_X04 on order (customer_id, 배송일자);
```

- 인덱스 파티션과 관련해 중요한 제약은 아래와 같다.
- unique index(주로 PK)를 파티셔닝하려면, 테이블 파티션 키가 모두 index 구성 컬럼이어야한다는 점이다.
- 만약 PK index 키가 order번호인데, 파티션 키는 order일자라면?
  - order번호가 123456인 order 레코드를 입력하려면, 중복값이 있는지 확인하려고 인덱스 파티션을 모두 탐색한다.
  - order번호가 123456인 레코드는 어떤 파티션에든 입력될 수 있기 때문이다.
  - 따라서 PK 인덱스 키를 (order일자, order번호)로 해줘야 한다.
  - 게다가 레코드를 입력하고 커밋하기 전까지, 다른 트랜잭션이 같은 order번호로 다른 파티션에 입력하는 현상까지 막아야 한다.
  - 그러면 추가적인 lock 메커니즘까지 필요해서 느려진다.

## <span style="color:#802548">_파티션 exchange를 이용한 데이터 변경_</span>
- 테이블이 파티셔닝되어있고, 인덱스도 다행히 로컬 파티션이라면, 임시 테이블을 만들어 원본 파티션과 바꿔치기하기도 한다.
- 만약 상태코드가 추가되면서 기존 상태코드도 바꾸기로 결정했다고 해보자.
- 그럼 기존의 상태코드들을 모두 바꿔야 할 것이다. 그런데 그냥  update하기에는 index 구조 등 시간이 오래걸린다.
- 따라서 만약 파티션 테이블이라면 미리 임시 파티션 테이블을 바꾼 상태코드로 복제해서 만든다. 
- 프로세스는 아래와 같이 진행하면 된다.
```
1. 임시 테이블을 생성한다. 가능하면 nologging 모드
2. transaction데이터를 읽어 임시 테이블에 insert하면서 상태코드 값을 수정한다.
3. 임시 테이블에 원본 테이블과 같은 구조로 index를 insert한다. 가능하면 nologging 모드
4. 2014년 12월 파티션과 임시 테이블을 exchange한다.
5. 임시 테이블을 drop한다.
6. 만약 nologging모드였다면, 파티션을 logging 모드로 전환한다.
```


- sql로는 아래와 같다.
```sql
1. CREATE TABLE transaction_t nologging as SELECT * FROM transaction WHERE 1 = 2;

2. INSERT /*+ append */ into transaction_t
SELECT customer_no, transaction_date, transaction순번, (CASE WHEN 상태코드 <> 'ZZZ' THEN 'ZZZ' ELSE 상태코드 END) 상태코드
FROM transaction
WHERE transaction_date < '20150101'

1. CREATE UNIQUE INDEX transaction_t_pk on transaction_t (customer_no, transaction_date, transaction순번) nologging;
CREATE INDEX transaction_t_x1 on transaction_t(transaction_date, customer_no) nologging;
CREATE INDEX transaction_t_x2 on transaction_t(상태코드, transaction_date) nologging;

1. ALTER TABLE transaction
EEXCHANGE PARTITION p201412 WITH TABLE transaction_t
INCLUDING INDEXES WITHOUT validation;

1. DROP TABLE transaction_t;

2. ALTER TABLE transaction MODIFY partition p201412 logging;
ALTER TABLE transaction_pk MODIFY partition p201412 logging;
ALTER TABLE transaction_x1 MODIFY partition p201412 logging;
ALTER TABLE transaction_x2 MODIFY partition p201412 logging;
```


## <span style="color:#802548">_파티션 truncate를 이용한 데이터 삭제_</span>
- 그냥 delete를 하면 굉장히 느리다.
- 아래와 같은 기나긴 추가과정을 거쳐야 한다.
```
1. 테이블 레코드 삭제
2. 테이블 레코드 삭제에 대한 undo logging
3. 테이블 레코드 삭제에 대한 redo logging
4. 인덱스 레코드 삭제
5. 인덱스 레코드 삭제에 대한 undo logging
6. 인덱스 레코드 삭제에 대한 redo logging
7. undo에 대한 redo logging(2번과 5번)
```



- 따라서 partition drop이나 truncate를 하면 시간이 크게 단축된다.
- 정말 간단한 경우는 아래와 같이 진행한다.
```sql
ALTER TABLE transaction DROP PARTITION p201412;
```

- 만약 조건절이 복잡하다면?
- 임시 테이블(transaction_t)를 생성하고, 남길 데이터만 복제한다.
- 프로세스는 아래와 같다.
```
1. 임시 테이블을 생성하고, 남길 데이터만 복제한다.
2. 삭제 대상 테이블 파티션을 truncate한다.
3. 임시 테이블에 복제해 둔 데이터를 원본 테이블에 입력한다.
4. 임시 테이블을 drop한다.
```

- sql로는 아래와 같다.
```sql
1. CREATE TABLE transaction_t
AS 
SELECT *
FROM transaction
WHERE transaction_date < '20150101'
AND 상태코드 = 'ZZZ'

2. ALTER TABLE transaction TRUNCATE PARTITION p201412;

3. INSERT INTO transaction
SELECT * FROM transaction_t;

4. DROP TABLE transaction_t;
```


- 서비스 중단 없이 파티션을 drop 또는 truncate하려면 아래 조건을 모두 만족해야 한다.
  - 파티션키와 커팅 기준 컬럼일 일치
    - 파티션 키와 조건절 컬럼이 모두 신청일자
  - 파티션 단위와 커팅 주기가 일치
    - 월단위 파티션을 월 주기로 조건절 걸음
  - 인덱스가 모두 로컬 파티션 인덱스
    - 파티션 키가 PK 구성에 포함되어야 함.





## <span style="color:#802548">_채번방식에 따른 INSERT 성능 비교_</span>
- 신규 데이터를 입력하려면 PK 중복을 방지하기 위한 채번이 선행되어야 한다.
- PK를 채번하는 데는 4가지 방법이 있다.
  - IDENETITY
  - 채번 테이블
  - 시퀀스 오브젝트
  - MAX + 1 조회
- IDENTITY는 12c 버전부터만 사용 가능하다.
- 채번 테이블의 경우를 살펴보자.
  - 결번이 없다.
  - 복합 컬럼 PK를 만들 수 있다.
  - 성능이 안 좋다.
    - 채번 레코드를 변경하기 위한 row lock 경합
    - 동시 INSERT가 심하면 채번 table data block 자체도 경합
    - 그래서 잘 안 쓰인다.
- 시퀀스 오브젝트를 살펴보자.
  - IDENTITY 이전에 가장 널리 쓰이는 방식이다.
  - 시퀀스 오브젝트는 오라클 내부에서 management_하는 채번 테이블이다.
  - 단점으로는 채번 이후 롤백, instance 재가동, 삭제에 따라 결번이 생긴다는 점이다.
  - 시퀀스는 비록 lock이 걸리지만 성능이 빠르다.
  - 로우 캐시 lock
    - 딕셔너리 정보(테이블, index, tablespace, data file, segment, extent, user, constraint, sequence, DB link)는 disk가 아닌 SGA에서 가져온다.
    - SGA이기 때문에 공유되는 영역이라 접근 직렬화가 필요하다. 동시에 nextval이 호출되면 row cache에서 시퀀스 레코드를 변경한다.
    - 시퀀스 레코드를 변경할 때 시퀀스 채번에 의한 로우 캐시 lock 경합을 줄이려면 해당 cache를 1000까지 늘리면 된다. 기본은 20이다.
  - 시퀀스 캐시 lock
    - 시퀀스 레코드를 변경할 때가 아니라 시퀀스 값을 얻어올 때도 lock이 필요하다.
  - SV lock
    - RAC 환경(이중화 같은 느낌)에선 sequence를 쓰려면 ORDER 옵션이 필요하다.
    - ORDER 옵션을 사용하면 SV Lock으로 시퀀스 캐시에 대한 액세스를 직렬화한다.
  - 
- MAX + 1 조회를 살펴보자.
  - 해당 방법은 시퀀스나 채번 테이블같이 별도 테이블이 필요 없다.
  - PK가 복합컬럼일 때도 사용가능하다.
  - 단점으론 다중 tranasction이 많아지면 성능이 급격히 나빠진다.


## <span style="color:#802548">_sequence보다 나은 채번 방식_</span>
- 시퀀스보다 좋은 솔루션은 PK를 만들 때, 입력일시를 같이 붙이는 방식이다.
- PK 구분속성에다가 입력일시를 같이 붙여주면 lock 이슈를 거의 해소할 수 있다.
- PK를 (구분속성, 입력일시)와 같이 구성하는 것이다. 구분속성이 앞에 와야 한다.
  - 그럼 특히 파티션 단위로 데이터를 삭제할 때 굉장히 유용하다.
  - 특히 데이터가 빠르게 대량으로 쌓이는 환경이면 데이터 삭제하기가 쉬워야 한다.
  - 삭제할 공간을 바로 시스템에 반납시켜야 DB를 scale-up하는 데 드는 비용이 적다.
    - 그 떄 입력일시를 PK에 포함시키면 파티션키(입력일시)가 PK에 포함된다.
    - 서비스 중단 없이 매우 빠르게 partition drop/truncate를 수행할수 있다는 의미다.
  

## <span style="color:#802548">_INSERT가 빨라도 문제_</span>
- INSERT가 너무 빨라도 INDEX 경합이 생긴다. 특히 채번 과정이 없으면 INDEX BLOCK 경합이 나타난다.
- INSERT 하는 row는 달라도, 같은 INDEX LEAF BLOCK을 갱신하려니 프로세스 간 버퍼 LOCK 경합이 발생한다.
- 특히 순차적으로 값이 증가하는 일련번호, 입력일시 등의 단일컬럼 인덱스는 right growing이라서 이런 현상이 심하다.
- 이러한 경우 RAC 환경에서는 특히나 더 심각한 성능 저하를 일으킨다. 여러 instance가 동시에 current block을 주고받으면 값을 입력해서다.
- 해결방법으론 아래와 같다.
  - 복합 컬럼에서는 거의 일어나지 않으니 복합 컬럼으로 만드는 것도 좋은 선택이다. 
  - 인덱스를 해쉬 파티셔닝한다.
  - 12c의 경우, global 시퀀스와 session sequence를 만든다.
```sql
CREATE sequence g_seq global;
CREATE sequence s_seq session;
```

- 글로벌 시퀀스는 커넥션 풀 프로세스가 DB에 접속하는 순간 호출된다.
- 세션 시퀀스는 INSERT를 수행할 때 호출한다.
- 따라서 아래와 같이 PK를 만들면 서로 다른 index leaf block에 값을 입력해 경합이 없다.
```sql
INSERT INTO t(id, c1,c2) values
(to_char(g_seq.currval, 'fm0000') || to_char(s_seq.nextval,'fm0000'), 'A', 'B')
```