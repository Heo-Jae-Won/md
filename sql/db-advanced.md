## <span style="color:#802548">_page_</span>
- everything is managed by page
- it is called block either
- so index, rows are stored in page
- page is 4 ~ 16KB block usually
- https://medium.com/databases-in-simple-words/breaking-down-the-anatomy-of-a-database-page-90e751d018fe
- https://shiftasia.com/community/pages-and-blocks-in-sql-databases/

## <span style="color:#802548">_table space_</span>
- allocated space, used space, and free space
- 

## <span style="color:#802548">_index access_</span>
- index adopts b-tree
    - index itself consists of index root, branch, leaf block
    - root, branch helps find out where is leaf block
    - going deep with root -> branch -> leaf block, found index key that satisfies condition
    - index is sorted so keep reading index record(key-value pair)
- index leaf scan is totally different from data block scan
- index leaf block doesnt read entire row
    - at first, finding starting point, which is vertical lookup
    - finally, seeking all index leaf that satisfies where condition from starting ponit to end point, which is finding end point
    - this is horizental lookup
    - thoses vertical and horizental lookup is about index access condition
- vertical lookup
    - finding first records that satisfies condition
    - starting point of page
      - for knowing exact starting point, columns for index should not be manipulated
    - let's take a breif look into good practice
    - this is good example of utilizing vertical lookup
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
  

- horizental lookup
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

## <span style="color:#802548">_how vertical/horizental scan work in index acess_</span>
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
      base_date, field_code, success_transaction, success_transaction_count, transaction_amount
FROM daily_field_transaction A
WHERE field_type_code = '01'
AND base_date BETWEEN '20080501' AND '20080531'
```

- when leading and medium index column lacks, u can use index skip scan either

```sql
SELECT /*+ INDEX_SS(A daily_field_transaction_PK) */
      base_date, field_code, success_transaction, success_transaction_count, transaction_amount
FROM daily_field_transaction A
AND base_date BETWEEN '20080501' AND '20080531'
```

- when leading column is <>, BETWEEN, LIKE, index skip scan is useful
- let's suppose (base_date + field_type_code) index
- if index range scan is executed, base_date that satisfies BETWEEN condition should be scaned, which means a lot of scans
- if using index skip scan, we can skip index range scan that field_type_code is not 01

```sql
SELECT /*+ INDEX_SS(A daily_field_transaction_PK) */
      base_date, field_code, success_transaction, success_transaction_count, transaction_amount
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

## <span style="color:#802548">_IOT vs HOT_</span>
todo

- heap-organized table
- index-organized table


## <span style="color:#802548">_index lookup_</span>

- index lookup means index access + index filter + table filter
- index lookup condition
    - for index lookup, there is 3 way
        - index access: deciding index range(vertical lookup, horizental lookup) 
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



## <span style="color:#802548">_covered index_</span>

todo
- to let not happen table random access, we can use covered index


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
- problem is that, subquery is divieded by modules
- subquery has 3 types
  - select: scalar subquery
  - from: inline view
  - where: nested subquery
- subquery is divided by unnset and filter

```sql
SELECT c.customer_no, c.customer_name
FROM customer c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
AND EXISTS (
  SELECT /*+ no_unnest */ 'x'
  FROM transactions
  WHERE customer_no = c.customer_no
  AND transaction일시 >= trunc(sysdate, 'mm')
)
```

- nested query 

```Execution plan
------------------------------------------
0     SELECT STATEMENT 
1       FILTER
2         TABLE ACCESS (BY INDEX ROWID) OF 'customer'
3           INDEX (RANGE SCAN) OF 'customer_X01'
4         INDEX (RANGE SCAN) OF 'transaction_X01' 
```

- filter operation is just same as NL join
- filter is NESTED LOOP
  - 다만 exists기 때문에 조건을 만족하는 순간 진행을 멈추고 메인쿼리의 다음 record에 관해 다시 join된다.
  - 그것 말고 NL join과 Filter 자체가 다른 점은, Filter는 캐싱 기능이 있다는 점이다.
  - 또한 필터 서브쿼리는 join순서가 고정이다. 늘 메인쿼리가 driving table이다.

<br />


- 그럼 unnest 방식일 때의 실행계획도 살펴보자.

```sql
SELECT c.customer_no, c.customer_name
FROM customer c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
AND EXIST (
      SELECT /*+ unnest nl_sj */ 'x'
      FROM transaction
      WHERE customer_no = c.customer_no
      AND transaction일시 >= trunc(sysdate, 'mm')
    )
```

```
Execution plan
------------------------------------------
0     SELECT STATEMENT 
1       NESTED LOOPS (SEMI)
2         TABLE ACCESS (BY INDEX ROWID) OF 'customer'
3           INDEX (RANGE SCAN) OF 'customer_X01'
4         INDEX (RANGE SCAN) OF 'transaction_X01' 
```

- EXIST를 쓰게 되면 SEMI 조인으로 작동하며 사실상 NL join과 다르지 않다.
- 그리고 unnest라도 캐싱 기능이 적용되기 때문에 filter operation과 동일하다.
- 다른 점은 바로 driving 집합이 서브쿼리가 될 수 있다는 사실이다.

```sql
SELECT /*+ leading(transaction@subq) use_nl(c) */ c.customer_no, c.customer_name
FROM customer c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
AND EXISTS (
  SELECT /*+ qb_name(subq) unnest */ 'x'
  FROM transaction
  WHERE customer_no = c.customer_no
  AND transaction일시 >= trunc(sysdate, 'mm')
)
```

```
0       SELECT STATEMNT
1         NESTED LOOPS
2           NESTED LOOPS
3             SORT
4               TABLE ACCESS
5                 INDEX
6             INDEX
7         TABLE ACCESS
```

- SORT UNIQUE가 뜬 이유는 서브쿼리를 unnest했기 때문이다.
- unnest하면서 메인쿼리 결과집합이 서브쿼리쪽 집합으로 확장될 수 있기 때문이다.
- 그래서 서브쿼리에 대해 sort unique를 수행하는 것이다.


<br />

- 참고로 subquery 안에 rownum은 쓰면 안 된다.
- rownum을 쓰게 되면 서브쿼리가 unnest되지 않는다.
- unnest를 하게 되면 optimizer가 좀더 좋은 경로를 선택할 가능성이 높아지기 때문에 하는 것이다.
- 그런데 rownum을 써서 unnest를 막아버렸으니 hint를 줘도 소용이 없는 것이다.
```sql
SELECT 글번호, 제목, 작성자, 등록일시
FROM 게시판 b
WHERE 게시판구분 = '공지'
AND 등록일시 >= trunc(sysdatet -1)
AND EXISTS (
        SELECT /*+ unnest nl_sj */ 'x'
        FROM 수신대상자
        WHERE 글번호 = b.글번호
        AND 수신자 = :memb_no
        AND ROWNUM <=1
)
```

- 필터로 작동하는 서브쿼리의 경우, 서브쿼리 필터링을 먼저 처리하면 성능이 좋아지는 경우가 많다.
- 아래는 서브쿼리 필터링을 맨 마지막에 처리하는 쿼리다.
```sql
SELECT /*+ leading(p) use_nl(t) */ count(distinct p.commodity_no), sum(t.주문금액)
FROM commodity p, 주문 t
WHERE p.commodity_no = t.commodity_no
AND p.등록일시 >= trunc(add_months(sysdate, -3), 'mm')
AND t.order_date >= trunc(sysdate - 7)
AND EXIST (
        SELECT 'x'
        FROM 상품분류
        WHERE 상품분류코드 = p.상품분류코드
        AND 상위분류코드 = 'AK'
)
```


<img src="/image/subquery-join.jpg" />

- Filter Operation이기 때문에 main query가 먼저 실행된다.
- main query는 join문이고, driving table인 commodity table scan부터 시작한다.
  - 우선 commodity table에 full scan으로 데이터를 가져온다. 그게 1000개다.
  - table full scan이므로 where 조건문으로 filtering을 해도 별다른 실행계획 뜨지 않고 table access full 상품으로 나온다.
- 이제 driven table인 주문 table scan을 시작한다.
  - join이 걸린 주문 table은 index scan을 진행한다. 
  - 위에서 얻은 1000개의 join record에 대해 index scan을 한 결과로 60,000 record가 뽑힌다.
  - index range scan이 60,000개인데 table access by index rowid로 접근한 것도 60,000개니까 필터링은 없는 셈이다.
- 그 뒤에는 exists 절에 있는 내용을 실행한다.
  - exists절은 index scan을 해서 가져오는데, 3개밖에 없어 보인다.
  - table access 시에 1개로 줄어든 것을 보면, 상위분류코드는 적어도 index access조건은 아니다.
  - index filter인지, table filter인지까지는 확인은 어렵다. index definition을 봐야 알 수 있다.
- exists를 통해 필터링을 마치게 되면 총 3000개의 record를 얻는다.

```
--------------------------------------------------------------
| ROW  | Operation                                   | Name        
--------------------------------------------------------------
|   0  | SELECT STATEMENT                            |              
|   1  | SORT AGGREGATE(cr=38103)                    |              
| 3000 | FILTER(cr=38103)                            |             
| 60000|   NESTED LOOPS(cr=38097)                    |             
| 1000 |     TABLE ACCESS FULL(cr=95)                | commodity        
| 60000|       TABLE ACCESS BY INDEX ROWID(cr=38002) | 주문      
| 60000|         INDEX RANGE SCAN(cr=2002)           | 주문_PK   
|    1 |       TABLE ACCESS BY INDEX ROWID(cr=6)     | 상품분류
|    3 |         INDEX UNIQUE SCAN(cr=3)             | 상품분류_PK
--------------------------------------------------------------
```


- 블록은 몇개나 읽었을까?
- 총 읽은 data block의 갯수는 38103개(join 38,097개 + exists 6개)다.
  - join 과정에서 총 38,097개 data block을 읽었다.
    - 처음 commodity table full scan을 하며 95개의 cr을 읽는다.
    - 그 다음 주문 table을 index scan하며 index block을 2002개 읽는다. 이건 총 읽은 블록에서는 제외다.
    - index block에서 얻은 rowid를 통해 38002개의 data block을 읽는다.
  - 그리고 마지막에 exists 절에서 index scan을 하면서 3개의 index leaf 블록을 읽는다. 해당 건은 제외다.
    - 읽은 leaf block에서 rowid를 가져와 6개의 data block을 읽었다.
- 38103개 block이면 1개 block 당 500개의 table record로 잡아도 엄청나게 많은 양이다.

<br />

- 서브쿼리 필터링을 먼저 처리한다면 어떨까? 읽은 블록이 확 줄어든다.
- 그러한 기술을 Pushing subquery라고 한다.
- 이 기능은 unnest되지 않은 subquery에서만 작동한다. 
- 따라서 push_subq hint는 no_unnest와 같이 사용해야 한다.
```sql
SELECT /*+ leading(p) use_nl(t) */ count(distinct p.commodity_no), sum(t.주문금액)
FROM commodity p, 주문 t
WHERE p.commodity_no = t.commodity_no
AND p.등록일시 >= trunc(add_months(sysdate, -3), 'mm')
AND t.order_date >= trunc(sysdate - 7)
AND exists (SELECT /*+ NO_UNNEST PUSH_SUBQ*/ 'x'
            FROM 상품분류
            WHERE 상품분류코드 = p.상품분류코드
            AND 상위분류코드 = 'AK'
          )
```

- 서브쿼리 필터링을 먼저처리해서 driving table의 scan을 하고 filtering하는 조건으로 작동한다.
- 실제 아래 실행계획을 보면 FILTER가 사라지고 맨밑의 상품분류 Operation이 중간으로 갔다.
- 블록은 몇개나 읽었을까?
- 총 읽은 data block의 갯수는 1903개(join 1903개)다. exists는 join에 읽은 block 갯수에 포함된다.
  - join 과정에서 총 1903개 data block을 읽었다.
    - 처음 commodity table full scan을 하며 101개 cr을 읽는다.
    - 그 다음 주문 table을 index scan하며 index block을 302개 읽는다. 이건 총 읽은 블록에서는 제외다.
    - index block에서 얻은 rowid를 통해 1802개의 data block을 읽는다.
- 1903개 block이면 아까 38103개에 비하면 거의 1/14개로 줄어들은 것이다.
```
--------------------------------------------------------------
| ROW  | Operation                                          | Name        
--------------------------------------------------------------
|   0 | SELECT STATEMENT                                    |              
|   1 |   SORT AGGREGATE(cr=1903)                           |              
| 3000|     NESTED LOOPS(cr=1903)                           |             
| 150 |       TABLE ACCESS FULL commodity(cr=101)                 | commodity      
| 1   |         TABLE ACCESS BY INDEX ROWID 상품분류(cr=6)   | 상품분류
| 3   |           INDEX UNIQUE SCAN 상품분류_PK(cr=3)        | 상품분류_PK  
| 3000|       TABLE ACCESS BY INDEX ROWID 주문(cr=1802)      | 주문      
| 3000|         INDEX RANGE SCAN 주문_PK(cr=302)             | 주문_PK   
--------------------------------------------------------------
```

## <span style="color:#802548">_inline view_</span>
- inline view is 쓸 때도 위와 같이 조건문이 아쉬울 때가 있다.
  - 분명히 어차피 전월 이후 가입한 customer을 필터링할 것이다.
  - 그런데도 inline view안에 해당 조건문이 없어 inline view는 그냥 scan을 진행한다.

```sql
SELECT c.customer_no, c.customer_name, t.평균transaction, t.최소transaction, t.최대거리
FROM customer c,
        (SELECT customer_no, avg(transaction금액) 평균transaction,
                min(transaction금액) 최소transaction, max(transaction금액) 최대transaction
        FROM transaction
        WHERE transaction일시 >= trunc(sysdate, 'mm') /**당월 발생한 transaction */
        group by customer_no) t
WHERE c.가입일시 >= trunc(add_months(sysdate, -1) ,'mm') /**전월 이후 가입 customer */
AND t.customer_no = c.customer_no
```

- execution is like below
- if no merge inline view, 진행되고, WHERE 조건문은 맨마지막에 filter로 진행된다.


<img src="/image/inline-view-no-merge.jpg" />


- 그럴 떄 merge hint를 사용해준다.
```sql
SELECT c.customer_no, c.customer_name, t.평균transaction, t.최소transaction, t.최대transaction
FROM customer c,
      (SELECT /*+ merge */ customer_no, avg(transaction금액) 평균transaction,
        min(transaction금액) 최소transaction, max(transaction금액) 최대transaction
        FROM transaction
        WHERE transaction일시 >= trunc(sysdate, 'mm')  /**당월 발생한 transaction */
        GROUP BY customer_no) t
WHERE c.가입일시 >= trunc(add_months(sysdate , -1), 'mm') /**전월 이후 가입 customer */
AND t.customer_no = c.customer_no
```

- 그럼 실행계획이 아래와 같이 바뀐다.
- view가 사라진 걸 볼 수 있다. 
- merge hint가 적용되어 WHERE 조건문의 필터링 조건이 inline view에 적용된다.
- 더 효율적으로 view를 scan해오는 것이다.
<img src="/image/inline-view-merge-hint.jpg" />


- inline view보다는 일반 table을 통한 join 형태로 변환해주는 게 대개 더 좋다.
  - 다만 위의 방식을 쓰면 group by를 마지막에 쓰기 때문에, 부분범위처리가 불가능하다.
  - join에 성공한 resultSet을 가지고 group by를 진행해야 하기 때문이다. 
    - 또한 driving table의 resultSet(여기선 전월 이후 가입한 customer)이 클 때는 join record에 따른 상대 table access가 많아진다.
    - 마찬가지로 driven table의 resultSet(여기선 당월 transaction)가 크다면 table access 이후 range scan양이 많아진다.
    - 따라서 이러한 경우 hash join으로 table scan을 진행하는 게 좋다.
- 만약 driving table만 데이터가 너무 많을 때는 inline view를 활용하는 것도 방법이다.


<br />

- 그 외 방법으로는 join 조건 pushdown이 있다.
- merge랑 비슷하게 inline view로 조건절이 들어간다.
  - pushdown을 이용하면 join record 건건이 group by를 수행하기에, 중간에 멈출 수 있다.
  - 따라서 group by가 강제되지 않아 부분범위처리가 가능하다는 점이다.
  - 대개 merge보다는 이 방법이 더 낫다고 볼 수 있다.
    - hint는 push_pred이며 no_merge와 반드시 같이 써야 한다.
```sql
SELECT c.customer_no, c.customer_name, t.평균transaction, t.최소transaction, t.최대transaction
FROM customer c,
      (SELECT /*+ no_merge push_pred */ customer_no, avg(transaction금액) 평균transaction,
        min(transaction금액) 최소transaction, max(transaction금액) 최대transaction
        FROM transaction
        WHERE transaction일시 >= trunc(sysdate, 'mm')  /**당월 발생한 transaction */
        GROUP BY customer_no) t
WHERE c.가입일시 >= trunc(add_months(sysdate , -1), 'mm') /**전월 이후 가입 customer. 이게 inline view로 들어간다. */
AND t.customer_no = c.customer_no
```

- view pushed predicate라는 실행계획이 추가된걸 볼 수 있다.
- 그럼 잘 작동한 것이다.
<img src="/image/pushdown-predicate.jpg" />


## <span style="color:#802548">_scalar subquery_</span>
- 스칼라 서브쿼리를 쓰지 않고 프로시저를 사용하면 record마다 호출된다.
- 당연히 dept table을 record마다 scan한다. 더 큰 문제는 프로시저 호출 시 컨텍스트 스위칭이 일어난다는 점이다.

```sql
CREATE OR REPLACE function GET_DNAME(p_deptno number) return varchar2
is
  l_dname dept.dname%TYPE;
  begin
    select dname into l_dname from dept where deptno = p_deptno;
    return l_dname;
  exception
    when others then
      return null;
  end;


SELECT empno, ename, sal, hiredate,
        GET_DNAME(e.deptno) as dname
FROM emp e
WHERE sal >= 2000
```

- 아래같이 스칼라 서브쿼리를 쓸 때도, record마다 dept table을 scan하는 건 같다.
- 하지만 적어도 context swithcing은 일어나지 않는다.

```sql
SELECT empno, ename, sal, hiredate,
      (SELECT d.danme from dept d where d.deptno = e.deptno) as danme
FROM emp e
WHERE sal >= 2000
```

- 실제 스칼라 서브쿼리는 아래와 같이 NL join과 같이 이뤄진다고 보면 된다.

```sql
SELECT /*+ ordered use_nl(d) */ e.empno, e.ename, e.sal, e.hidredate, d.dname
FROM emp e, dept d
WHERE d.deptno(+) = e.deptno /** +붙었으니 outer join */
and e.sal >= 2000
```

- 스칼라 서브쿼리는 강력한 캐싱 기능을 제공한다.
- 캐싱 효과를 사용하기 위해 프로시저를 스칼라 서브쿼리로 덮어버리기도 한다.
- 캐싱효과 덕분에 호출횟수를 최소화할 수 있기 때문이다.

```sql
SELECT empno, ename, sal, hiredate,
        (SELECT GET_DNAME(e.deptno) FROM dual) dname
FROM emp e
WHERE sal >= 2000
```

- join에 쓰이는 데이터를 캐시에서 찾으면 join 성능이 매우 좋아진다.
  - 첫 시도는 느리지만, 그 다음부터는 매우 빨라진다는 의미다.
  - 하지만 PGA에 캐시를 할당하는 것이라, 캐시에 담을 join access에 쓰이는 record가 적어야 한다.
    - 예를 들어, transaction구분코드가 20개라면 캐싱에 충분하고, 캐싱이 제대로 작동할 것이다.
    - 만약 customer이 100만명인데, customer_id로 join access를 쓴다면 100만개를 캐시에 저장해야 한다.
    - 하지만 캐시는 작아서 그만큼 저장하지 못하기 때문에 캐시를 탐색해봐야 없을 것이다.
    - 이러면 캐시 탐색 비용만 늘어나서 메모리/CPU 사용률만 높아진다.

```sql
SELECT transaction번호, customer_no, 영업조직ID, transaction구분코드,
        (SELECT customer_name FROM customer WHERE customer_no = t.customer_no) customer_name
FROM transaction t
WHERE transaction_date >= to_char(add_months(sysdate, -3), 'yyyymmdd')
```

- 캐싱 효과가 부작용을 일으키는 경우가 하나 더 있다.
- 메인쿼리 집합이 작은 경우다. 스칼라 서브쿼리는 쿼리 단위로 캐싱이 이뤄진다.
  - 메인쿼리 집합이 커야만 효과가 크다. 그래야 많이 재사용되기 때문이다.
  - 재사용되지도 않는데 귀한 캐시 영역에 넣었다가 바로 버리면 비효율적인 것이다.
  - 메인쿼리로 가져오는 record가 1개밖에 없다면 더더욱 그럴 것이다.
  - 아래 sql을 보면 customer은 보통 계좌를 1개만 갖고 있기 마련이다. 
    - 그런데 1개에 관해 사용자정의 함수를 호출해 scalar subquery로 덮으면 오히려 성능이 떨어진다.


```sql
SELECT account_no, 계좌명, customer_no, 개설일자, 계좌종류구분코드, 은행계설여부, 은행연계여부
      ,(SELECT brch_nm(management_지점코드) from dual) management_지점명
      ,(SELECT brch_nm(개설지점코드) from dual) 개설지점명
FROM 계좌
WHERE customer_no =:customer_no
```

- 아래는 스칼라 서브쿼리를 사용할 때의 실행계획이다.

```sql
SELECT c.customer_no, c.customer_name
      ,(SELECT ROUND(avg(transaction금액), 2) 평균transaction금액
        FROM transaction
        WHERE transaction일시 >= trunc(sysdate, 'mm')
        AND customer_no = c.customer_no)
FROM customer c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
```

- 실행계획에서 후행 step이 먼저 발동하는 쿼리다.
- 따라서 customer이 먼저 full scan이 된다. 즉 메인쿼리가 먼저 발동한다.
- 그 다음 transaction table이 index scan된다. scalar subquery는 나중에 실행되는 것이다.

```
EXECUTION PLAN
----------------------------------------------
0       SELECT STATEMNT 
1   0     SORT (AGGREGATE)
2   1       TABLE ACCESS (BY INDEX ROWID BATCHED) OF 'transaction'
3   2         INDEX (RANGE SCAN) OF 'transaction_X02' (INDEX)
4   0     TABLE ACCESS (FULL) OF 'customer' (TABLE)
5   4       INDEX (RANGE SCAN) OF 'customer_X01' (INDEX)
```


- scalar subquery로 두 개 이상의 값을 얻는 건 불가능하다.
- 아래와 같은 sql은 불가능하다는 의미다.

```sql
SELECT c.customer_no, c.customer_name
      ,(SELECT avg(transaction금액), min(transaction금액), max(transaction금액)
        FROM transaction
        WHERE transaction일시 >= trunc(sysdate, 'mm')
        AND customer_no = c.customer_no)
FROM customer c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
```

- 그렇다고 아래처럼 바꾸면 transaction 테이블에서 같은 테이블을 반복해 읽게 된다.

```sql
SELECT c.customer_no, c.customer_name
        ,(SELECT AVG(transaction금액) FROM transaction
          WHERE transaction일시 >= trunc(sysdate, 'mm') AND transaction번호 = c.customer_no)
        ,(SELECT MIN(transaction금액) FROM transaction
          WHERE transaction일시 >= trunc(sysdate, 'mm') AND transaction번호 = c.customer_no)
        ,(SELECT max(transaction금액) FROM transaction
          WHERE transaction일시 >= trunc(sysdate, 'mm') AND customer_no = c.customer_no)
FROM customer c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
```

- 따라서 사실 아래와 같은 방식이 많이 사용됐다.

```sql
SELECT customer_no, customer_name
      , to_number(substr(transaction금액, 1, 10))  평균transaction금액
      , to_number(substr(transaction금액, 11, 10)) 최소transaction금액
      , to_number(substr(transaction금액, 21))     최대transaction금액
FROM (
  SELECT c.customer_no, c.customer_name
        , (SELECT LPAD(AVG(transaction금액), 10) || LPAD(MIN(transaction금액), 10) || MAX(transaction금액)
            FROM transaction
            WHERE transaction일시 >= trunc(sysdate, 'mm')
            AND customer_no = c.customer_no) transaction금액
  FROM customer c
  WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
)
```

- 11g부터는 pushdown이 도입돼 위와 같이 쓰지 않아도 된다.
- 아래와 같이 inline view를 쓰는 게 가능해져 더 직관적으로 바뀌었다.

```sql 
SELECT c.customer_no, c.customer_name, t.평균transaction, t.최소transaction, t.최대transaction
FROM customer c
    , (SELECT /*+ no_merge push_pred */
              customer_no, avg(transaction금액) 평균transaction
              , min(transaction금액) 최소transaction, max(transaction금액) 최대transaction
      FROM transaction
      WHERE transaction일시 >= trunc(sysdate, 'mm')
      GROUP BY customer_no) t
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
AND t.customer_no(+) = c.customer_no
```

- pushdown을 사용하면 실행계획은 아래와 같이 VIEW PUSHED PREDICATE가 뜨게 된다.


<img src="/image/view-pushdown.jpg" />


- scalar subquery is not fittable parellellism
  - 대량 데이터를 처리하는 병렬쿼리는 해시 조인이 효과적이다.
- 또한 scalar subquery는 NL join이 이뤄지는 것이다. 
  - 따라서 캐싱 효과가 작으면 random access의 부담이 있다.
- 12c부터는 scalar subquery가 효과가 없다면 unnest 해서 merge할 수도 있다.
- /*+ unnest merge */를 통해 목적을 달성한다.
- 만약 unnest merge를 해서 문제가 생겼다면, no_unnest hint를 주면 된다.
- _optimizer_unnest_scalar_sq를 false로 놓는 것은 장기적으론 좋지 않은 선택이다.

<img src="/image/scalar-subquery-unnest-merge.jpg" />

## <span style="color:#802548">_sort 연산_</span>
- oracle은 가공된 데이터 집합이 필요할 때, PGA와 Temp 테이블스페이스를 활용한다.
  - sort merge join
  - hash join
  - group by
  - order by
- sort는 PGA의 Sort Area에서 먼저 이뤄지고, 그거로도 부족하면 Temp 테이블스페이스를 활용한다.
- Sort Area에서만 끝나면 메모리 소트(Internal sort), 디스크까지 활용하면 디스크 소트(External sort)라고 한다.
- sort가 이뤄지는 방식은 아래 이미지와 같다.

```
소트할 대상을 SGA 버퍼캐시를 통해 읽는다.
sort Area에서 정렬을 해본다.
안되면 Temp 테이블스페이스에 임시 세그먼트를 만들어 저장한다.
temp에 저장해 둔 집합을 sort run이라고 부른다.
클라이언트에 전달할 때는 PGA에 있는 걸 먼저 준다.
그 다음 temp에 있는 것을 PGA에 merge해서 PGA에서 client로 전달한다.
```

<img src="/image/sort-process.jpg" />

- 여기서 값 조정이 가능하다.
todo

- 소트 연산은 메모리 집약적이며, CPU 집약적이다.
- 거기다 PGA가 작으면 disk I/O까지 일어나니 쿼리 성능에 핵심적이다.
- 될 수 있다면 sort를 발생시키지 않는게 좋고, 필요하면 메모리로 끝내야 한다.


## <span style="color:#802548">_소트가 발생하지 않게 SQL 작성_</span>
- union, minus, distinct는 최대한 자제해야 한다.
  - union보다는 union all로 쓸 수 있는지 확인해봐야 한다.
  - distinct보다는 exists를 사용할 수 있는지 확인해봐야 한다.
  - 대량데이터가 아니라면 hash join이 아니라 NL join을 써야 한다.

- distinct를 제거하는 쿼리 튜닝도 보자.

```sql
SELECT DISTINCT p.commodity_no, p.commodity_name, p.상품가격
FROM commodity p, Contract c
WHERE p.상품_type코드 = :pclscd
AND c.commodity_no = p.commodity_no
AND c.계약일자 BETWEEN :dt1 and :dt2
AND c.계약구분코드 = :ctpcd
```


- 위의 쿼리 대신 exists를 활용해보자.
- exists를 이용하면 데이터를 모두 읽지 않아도 된다.
- distinct를 사용하지 않았으니 부분범위 처리도 가능하다.

```sql
SELECT p.commodity_no, p.commodity_name, p.상품가격
FROM commodity p
WHERE p.상품_type코드 = :pclscd
AND EXISTS (
          SELECT 'x' FROM Contract c
          WHERE c.commodity_no = p.commodity_no
          AND c.계약일자 BETWEEN :dt1 and :dt2
          AND c.계약구분코드 = :ctpcd
        )
```

- minus를 튜닝해보자.

```sql
SELECT ST.상황접수번호, ST.관제일련번호, ST.상황코드
FROM 관제진행상황 ST
WHERE 상황코드 = '0001'
AND 관제일시 BETWEEN :V_TIMEEFROM || '000000' AND :V_TIMET0 || '235959'

MINUS

SELECT ST.상황접수번호, ST.관제일련번호 ,ST.상황코드, ST.관제일시
FROM 관제진행상황 ST, 구조활동 RPT
WHERE 상황코드 = '0001'
AND 관제일시 BETWEEN :V_TIMEEFROM || '000000' AND :V_TIMET0 || '235959'
AND RPT.출동센터ID = :V_CNTR_ID
AND ST.상황접수번호 = PRT.상황접수번호
ORDER BY 상황접수번호, 관제일시
```


- 아래와 같이 바꿀 수 있다.
- not exists를 이용하면 데이터를 모두 읽지 않아도 된다.
- distinct를 사용하지 않았으니 부분범위 처리도 가능하다.

```sql
SELECT ST.상황접수번호, ST.관제일련번호 ,ST.상황코드, ST.관제일시
FROM 관제진행상황 ST, 구조활동 RPT
WHERE 상황코드 = '0001'
AND 관제일시 BETWEEN :V_TIMEEFROM || '000000' AND :V_TIMET0 || '235959'
AND NOT EXISTS (
              SELECT 'X' FROM 구조활동
              WHERE 출동센터ID = :V_CNTR_ID
              AND 상황접수번호 = ST.상황접수번호
            ) 
ORDER BY ST.상황접수번호, ST.관제일시
```


- join에 관해서도 이야기해보자.
- Contract table의 index가 (지점ID, 계약일시)기 때문에 order by가 생략될 수 있다.
- 그러나 hash join이 되면서 order by가 일어났다.

```sql
SELECT c.계약번호, c.상품코드, p.commodity_name, p.commodity_type_code
FROM Contract c, commodity p
WHERE c.지점ID = :brch_id
AND p.상품코드 = c.상품코드
ORDER BY c.계얄일시 DESC
```


- 이를 NL join으로 바꿔준다.

```sql
SELECT /*+ leading(c) use_nl(p) */ c.계약번호, c.상품코드, p.commodity_name, p.commodity_type_code
FROM Contract c, commodity p
WHERE c.지점ID = :brch_id
AND p.상품코드 = c.상품코드
ORDER BY c.계얄일시 DESC
```


<img src="/image/join-sort-NL-hash.jpg" />

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
SELECT account_no, 총예수금
FROM 예수금원장
ORDER BY 총예수금 DESC
``` 

todo

- max 대신 window function을 쓰는 것도 좋은 습관이다.
- 그러면 top n sort 알고리즘이 발동해 sort 부하가 적다.
  - 그만큼 disk I/O가 덜 생겨서 빨라지게 된다는 의미다.


- 실험을 위해 아래같이 sort_area 메모리공간을 줄여보자.

```sql
alter session set workarea_size_policy = manual;
alter session set sort_area_size = 524288;
```

- 아래는 max를 썼다. 
- 그럼 disk I/O가 많다.

```sql
SELECT equipment_no, change_date, change_history_order, 상태코드, 메모
FROM (SELECT equipment_no, change_date, change_history_order, 상태코드, 메모
            , MAX(change_history_order) OVER(PARTITION BY equipment_no) 최종change_history_order
            FROM change_history
            WHERE change_date = :upd_dt)
WHERE change_history_order = 최종change_history_order
```


<img src="/image/max-sort-disk.jpg" />


- rank 같은 window function을 쓰면 훨씬 disk I/O가 줄어든다.

```sql
SELECT equipment_no, change_date, change_history_order, 상태코드, 메모
FROM (SELECT equipment_no, change_date, change_history_order, 상태코드, 메모
            , RANK(change_history_order) OVER(PARTITION BY equipment_no ORDER BY change_history_order DESC) 최종change_history_order
            FROM change_history
            WHERE change_date = :upd_dt)
WHERE change_history_order = 최종change_history_order
```

<img src="/image/window-sort-no-disk.jpg" />

## <span style="color:#802548">_redo로그_</span>
- DML을 수행할 때마다 Redo 로그를 생성한다.
- Redo 로그는 아래와 같은 목적이다.
  - 물리적으로 디스크가 깨지면 복구(database recovery)
  - 정전 등으로 버퍼캐시에 못들어가고 날라갔을 때 복구(cache recovery)
  - 로그 파일에 먼저 기록하고, 동기화는 나중에 배치로 반영(fast commit)
    - 변경된 메모리 버퍼블록과 데이터파일 블록 간 동기화는 async로 수행
    - async라고 해도 몇 밀리초에서 수십 밀리초 정도 수준
    - 참고로 insert에는 redo 로그를 안남기는 것도 가능하다.
- 아래와 같이 하면 총 redo 로그 파일이 크기를 볼 수 있다.

```sql
SELECT 
    ROUND(SUM(bytes) / (1024 * 1024), 2) AS "Size_MB"
FROM 
    v$log;
```


- 아래와 같이 하면 각 redo 로그 파일이 크기를 볼 수 있다.
```sql
SELECT 
    member AS "Redo_Log_File",
    bytes / (1024 * 1024) AS "Size_MB"
FROM 
    v$logfile;
```

- 아래와 같이 하면  현재 Redo 로그의 보관 설정을 확인할 수 있다.
```sql
SELECT destination, TO_CHAR(next_time, 'YYYY-MM-DD HH24:MI:SS') AS next_archive_time
FROM v$archive_dest
WHERE destination IN ('LOG_ARCHIVE_DEST_1', 'LOG_ARCHIVE_DEST_2'); -- 원하는 로그 보관 설정을 확인합니다.
```


- 새롭게 들어오는 DML은 아래와 같이 추가된 redo 로그에 기록되게 된다.
- 다만 새로운 redo 로그를 반영하려면 DB서버를 다시 재시작시켜야 한다.
```sql
ALTER SYSTEM SET LOG_ARCHIVE_DEST_1 = 'LOCATION=/archivelog DESTINATION=USE_DB_RECOVERY_FILE_DEST REOPEN=60'; -- 60분 유지 설정
ALTER DATABASE ADD LOGFILE GROUP 4 ('/arch1/log4a.rdo', '/arch2/log4b.rdo') SIZE 50M; -- redo 파일 해당 크기로 추가
```


## <span style="color:#802548">_undo로그_</span>
- DML을 수행할 때마다 undo 로그(롤백)도 생성된다.
  - redo가 트랜잭션을 재현하는 데 필요한 정보였다.
  - undo로그의 목적은 아래와 같다.
    - 변경된 블록을 이전 상태로 되돌리는 데 필요하다(transaction rollback)
    - transaction recovery
    - read consistency
  - undo 로그는 가장 오래된 곳부터 갈아끼워지므로 과거 트랜잭션은 사라지게 된다.
  - 실제 undo로그의 용량은 undo 테이블스페이스 크기와 undo로그가 유지되는 시간에 의해 결정된다.
  - undo 로그는 당연하게도 db management_자만이 변경가능하다.

```sql
ALTER SYSTEM SET UNDO_RETENTION = 3600; -- undo로그 유지시간은 3600초(1시간)
ALTER DATABASE DATAFILE '/path/to/undo/datafile.dbf' RESIZE 100M; -- undo 테이블스페이스 크기 조정.
SELECT tablespace_name, sum(bytes)/1024/1024 AS "Size_MB" -- 실제로 조정됐는지 확인
FROM dba_data_files
WHERE tablespace_name = 'UNDOTBS1';
```

## <span style="color:#802548">_commit_</span>
- 특히 Lock에 걸린 경우 commit을 해야 lock이 풀리게 되므로 commit은 매우 중요한 역할을 한다.
- commit은 아래와 같은 과정을 거쳐서 만들어진다.
  - DML 실행 -> redo로그버퍼에 변경사항 기록
  - 버퍼블록에서 데이터를 변경. 버퍼캐시에 없으면 데이터파일 읽기(disk I/O)
  - commit
  - LGWR process가 redo로그버퍼 내용 로그파일에 일괄 저장
  - DBWR 프로세스가 변경된 버퍼블록을 데이터파일에 일괄 저장.


<img src="/image/dml-process.jpg" />

- 버퍼캐시가 휘발성이어서 redo 로그를 남긴다.
- 그런데 redo 로그도 휘발성 로그버퍼에 기록하면 영속성이 보장될까?
- 그렇다. redo로그만 디스크에 기록되어있다면 영속성이 보장된다.
- commit 시점에 LGWR 프로세스가 꺠어나기 때문이다.
  - 서버 프로세스가 commit record를 로브버퍼에 기록한다.
  - 서버 프로세스가 LGWR 프로세스에 신호를 보내고 wait 큐에서 sleep상태로 전환한다.
  - LGWR 프로세스가 로그 버퍼를 디스크에 기록하는 작업을 마친다.
  - LGWR 프로세스가 wait 큐에 대기중인 서버 프로세스에 완료 메시지를 전송한다.
  - 신호를 받은 서버 프로세스는 runnable 큐로 옮겨진 후 CPU를 할당받아 다음 작업을 이어간다.
- 위에서 보았듯 LGWR 프로세스가 redo로그를 기록하는 작업은 disk I/O기 때문에 생각보다는 느리다.
  - 트랜잭션을 너무 잘게 짜르면 disk I/O가 많아져 성능이 나빠진다.
  - 트랜잭션을 너무 길게 유지하면 undo로그 공간이 부족해져 시스템 장애가 올 수 있다.
- Spring에서는 @Transactional로 트랜잭션을 만드는데, 3개의 db조작 method를 모아서 하나의 method에 @Transactional을 달아준다.
- 그럼 그 3개 중 하나만 잘못돼도 모두 rollback되는데, 이러한 작용을 할 때 transaction이 너무 긴 건 아닌지 고려해야 할 때도 있다.



## <span style="color:#802548">_PK제약 해제 후 배치_</span>
- index와 무결성 제약은 DML 성능을 떨어뜨린다.
- 그래서 배치 프로그램에서는 해당 기능을 해제하는 경우도 많다.
- 테스트 데이터를 넣은 테이블을 만들어보자.

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

- source table에 천만건의 데이터가 생겼다. 
- 이제 이를 target table에 넣어보자.
- PK와 일반 INDEX 총 2개로 천만개를 넣으면 1분 19초가 걸린다.

```sql
SET TIMING ON;

INSERT /*+ append */ into target
SELECT * FROM source;

commit;
```

- PK제약과 index를 해제하고 넣어보자.

```sql
TRUNCATE TABLE target;
ALTER TABLE target MODIFY CONSTRAINT target_pk DISABLED DROP index; --PK 제약 해제
ALTER INDEX target_x1 UNUSABLE;                                     --일반 INDEX 해제

ALTER SESSION SET skip_unusable_indexes = true;                     --INDEX unusable 데이터 입력가능하게 설정. 기본이 true
```

- 다시 insert를 해보자.
- 그럼 5초만에 끝난다.

```sql
SET TIMING ON;

INSERT /*+ append */ into target
SELECT * FROM source;

commit;
```

- 그리고 다시 PK와 일반 index제약을 생성해주자.
- 재활성화에 8초가 걸린다.
- 총 13초밖에 안걸린다. 제약이 있는 상태에서 insert는 1분 19초가 걸렸다.

```sql
ALTER TABLE target MODIFY CONSTRAINT target_pk enable NOVALIDATE; -- PK제약 재활성화
ALTER INDEX target_x1 rebuild;                                    -- 일반 index 제약 활성화
```

- NOVALIDATE 옵션을 써서 더 빠른 것도 있다.
- 데이터 무결성 확신이 없다면 아래 query를 통해 확인해보자.

```sql
SELECT no, empno, count(*)
FROM source
GROUP BY no, empno
HAVING COUNT(*) > 1;
```


## <span style="color:#802548">_수정가능 조인뷰_</span>
- 전통적인 update문으로는 다른 table과 join이 필요한 update를 할 때 select를 여러 번 하는 비효율이 발생한다.

```sql
UPDATE customer c
SET     최종transaction일시 = (SELECT MAX(transaction일시) FROM 거랙
                        WHERE customer_no = c.customer_no
                        AND transaction일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
        , 최근transaction횟수 = (SELECT COUNT(*) FROM transaction
                          WHERE customer_no = c.customer_no
                          AND transaction일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
        , 최근transaction금액 = (SELECT SUM(transaction금액) FROM transaction
                          WHERE customer_no = c.customer_no
                          AND transaction일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
WHERE exists (SELECT 'x' FROM transaction
              WHERE customer_no = c.customer_no
              AND transaction일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))                        
```

- 약 4번의 SELECT를 하는 걸 2번의 SELECT로 고칠 수도 있다.
- 다만 여기도 총customer수가 엄청나게 많다면 버벅거릴 수 있다.

```sql
UPDATE customer c
set (최종transaction일시, 최근transaction횟수, 최근transaction금액) =
    (SELECT MAX(transaction일시), count(*), sum(transaction금액)
      FROM transaction
      WHERE customer_no = c.customer_no
      AND transaction일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
WHERE EXISTS (SELECT 'x' FROM transaction
              WHERE customer_no = c.customer_no
              AND transaction일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
```

- 그래서 총customer 수가 많다면 아래와 같이 바꿔줄 수도 있다.
- exists 서브쿼리를 hash semi join으로 바꿨다.

```sql
UPDATE customer c
set (최종transaction일시, 최근transaction횟수, 최근transaction금액) =
    (SELECT MAX(transaction일시), count(*), sum(transaction금액)
      FROM transaction
      WHERE customer_no = c.customer_no
      AND transaction일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
WHERE EXISTS (SELECT /*+ unnest hash_sj */'x' FROM transaction
              WHERE customer_no = c.customer_no
              AND transaction일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
```

- 수정 가능 조인 뷰를 활용하면 참조 테이블과 두 번 조인하는 비효율마저도 없앨 수 있다.
- 단, 수정 가능 조인 뷰의 경우, 1 : M 집합 중 M 집합만 DML이 허용된다.
- 또한 1쪽 집합에 반드시 unique index 설정이 있어야만 한다. 
- 즉 키보존 테이블이어야 한다는 의미다.
  - 키보존 테이블이란view에 rowid를 제공할 수 있는 unique index를 가진 column이다.
- 다만 아래 쿼리는 12c 이상 버전부터만 실행된다.

```sql
UPDATE
  (SELECT /*+ ORDERED USE_HASH(c) no_merge(t) */
    c.최종transaction일시, c.최근transaction횟수, c.최근transaction금액
    , t.transaction일시, t.transaction횟수, t.transaction금액
  FROM (SELECT customer_no
              , MAX(transaction일시) transaction일시
              , COUNT(*) transaction횟수
              , SUM(transaction금액) transaction금액
        FROM transaction
        WHERE transaction일시 >= TRUNC(ADD_MONTHS(sysdate, -1))
        GROUP BY customer_no
        ) t
  )
  SET 최종transaction일시 = transaction이릿
    , 최근transaction횟수 = transaction횟수
    , 최근transaction금액 = transaction금액
```

- 11g에서는 위의 update를 사용할 수가 없고, merge into문으로 바꿔줘야 한다.
- 12c 이상 버전부터는 unique index가 없어도 inline view에 group by를 쓰면 에러가 나지 않는다.
- GROUP BY를 한 집합과 조인한 테이블은 key가 보존된다는 점을 오라클에서 인정한 것이다. 



## <span style="color:#802548">_direct path I/O_</span>
- direct path는 버퍼캐시를 경유하지 않고 직접 data block을 읽고 쓰는 I/O다.
- 특히 대량 데이터를 읽을 때 많이 사용된다. 대용량 처리 프로그램이 읽은 data를 버퍼캐시에 넣는 건 재활용성이 나쁘기 때문이다.
- 아래와 같은 경우에 direct path I/O가 수행된다.
  - 병렬힌트로 select/insert
  - direct 옵션 지정하고 SQL Loader로 데이터 적재
  - insert ... select문에 append hint 사용
- 쿼리에 병렬 hint를 제공하면 병렬도만큼 병렬 프로세스가 떠서 동시에 작업을 한다.
```sql
SELECT /*+ full(t) parallel(t 4)  */ * FROM big_table t;
```

- 위처럼 병렬도를 4로 지정하면 성능이 4배가 아니라 수십 배 빨라진다.
- order by, group by, hash join, sort merge join의 경우 힌트로 지정한 병렬도보다 두 배 많은 프로세스가 사용된다.
- direct path I/O가 빠른 이유는 아래와 같다.
  - freelist를 참조하지 않고 HWM 바깥 영역에 데이터를 순차적으로 입력한다.
  - 블록을 버퍼캐시에서 탐색하지 않는다.
  - 버퍼캐시에 적재하지 않고, 데이터파일에 직접 기록한다.
  - read consistency, transaction rollback, transaction recovery를 위한 undo 로깅이 없다.
  - redo 로깅을 안 할수도 있다.
    - 한 마디로 많은 절차가 생략되기 때문에 빠른 것이다.
- 성능은 빨라지지만 주의점이 있다.
  - exclusive lock이 걸린다. 따라서 transaction이 일어나는 주간에는 쓰면 안 된다.
  - table에 여유 공간이 있어도 table 바깥 영역에 할당되어 디스크를 낭비할 수도 있다.
    - 특히 레인지 파티션 테이블이면 이게 문제가 될 수 있다. 
    - 과거 데이터를 delete해도 의미가 없고, 반드시 파티션을 drop해야만 디스크 공간을 반환하기 때문이다.


<img src="/image/hwm-direct-path.jpg" />


- insert의 경우 append를 활용하면 direct path I/O가 된다.
- 하지만 update, delete의 경우는 append hint가 무의미하다.
- 병렬 DML을 활용해야만 한다.
- 다만 exclusive TM Lock이 걸리므로, transaction이 빈번한 주간에는 사용하면 안 된다.
```sql
ALTER session enable parallel dml;

INSERT /*+ parallel(c 4)  */ into customer c
SELECT /*+ parallel(o 4) */ * FROM 외부가입customer o;
```

## <span style="color:#802548">_direct path I/O를 이용한 데이터 삽입_</span>
- 비파티션의 경우에는 index를 unusable 하는 방식을 사용한다.
- 프로세스는 아래와 같다.
```
1. 테이블을 nologging 모드로 전환
2. index를 unusable 상태로 전환
3. direct path I/O로 데이터 입력
4. no logging 모드로 index 재생성
5. nologging 모드를 logging모드로 전환
```


- sql로는 아래와 같다.
```sql
1. ALTER TABLE taregt_t nologging;

2. ALTER INDEX target_t_x01 unusable;

3. INSERT /*+ append   */ INTO target_t
SELECT * FROM source_t;

4. ALTER INDEX target_t_x01 REBUILD nologging;

5. ALTER TABLE target_t logging;
ALTER INDEX target_t_x01 logging;
```


- 보통 엔간하면 index를 unusable로 전환하지 않고 insert를 하지만, 조건이 만족된다면 unusable을 한다.
  - table이 partitioned다.
  - index가 local partition이다.
- 프로세스는 아래와 같다.
- 아까전과 크게 다른 게 거의 없다. 그저 작업이 파티션 단위로 이뤄지는 점만 다르다.
```sql
1. ALTER TABLE taregt_t MODIFY partition p_201712 nologging;

2. ALTER INDEX target_t_x01 MODIFY partition p_201712 unusable;

3. INSERT /*+ append   */ INTO target_t
SELECT * FROM source_t WHERE dt BETWEEN '20171201' AND '20171231';

4. ALTER INDEX target_t_x01 REBUILD partition p_201712 nologging;

5. ALTER TABLE target_t MODIFY PARTITION p_201712 logging;
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
CREATE TABLE 주문 (주문번호 number , 주문일자 varchar2(8), ...)
partition by range(주문일자) (
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
CREATE INDEX 주문_X01 on 주문 (주문일자, 주문금액) LOCAL

CREATE INDEX 주문_X03 on 주문 (주문금액, 주문일자) GLOBAL
PARTITION BY RANGE(주문금액) (
  PARTITION P_01 values less than (100000)
  PARTITION P_MX values less than (MAXVALUE)
)

CREATE INNDEX 주문_X04 on 주문 (customer_id, 배송일자);
```

- 인덱스 파티션과 관련해 중요한 제약은 아래와 같다.
- unique index(주로 PK)를 파티셔닝하려면, 테이블 파티션 키가 모두 index 구성 컬럼이어야한다는 점이다.
- 만약 PK index 키가 주문번호인데, 파티션 키는 주문일자라면?
  - 주문번호가 123456인 주문 레코드를 입력하려면, 중복값이 있는지 확인하려고 인덱스 파티션을 모두 탐색한다.
  - 주문번호가 123456인 레코드는 어떤 파티션에든 입력될 수 있기 때문이다.
  - 따라서 PK 인덱스 키를 (주문일자, 주문번호)로 해줘야 한다.
  - 게다가 레코드를 입력하고 커밋하기 전까지, 다른 트랜잭션이 같은 주문번호로 다른 파티션에 입력하는 현상까지 막아야 한다.
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