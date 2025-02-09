## <span style="color:#802548">_inner join_</span>

- join을 사용하면 아래와 같이 다양한 값을 뽑을 수 있다.
- 아래의 방식은 oracle join이다. 
- 읽을 줄은 알아야겠지만, 쓸 때는 지양해야 할 방법이다.

```sql
SELECT
  e.employee_id,
  e.job_id,
  e.manager_id,
  e.department_id,
  d.location_id,
  l.country_id,
  e.first_name,
  e.last_name,
  e.salary,
  e.commission_pct,
  d.department_name,
  j.job_title,
  l.city,
  l.state_province,
  c.country_name,
  r.region_name
FROM
  employees e,
  departments d,
  jobs j,
  locations l,
  countries c,
  regions r
WHERE e.department_id = d.department_id
  AND d.location_id = l.location_id
  AND l.country_id = c.country_id
  AND c.region_id = r.region_id
  AND j.job_id = e.job_id
```

- ansi로 바꿔주자.

```sql
SELECT
  e.employee_id,
  e.job_id,
  e.manager_id,
  e.department_id,
  d.location_id,
  l.country_id,
  e.first_name,
  e.last_name,
  e.salary,
  e.commission_pct,
  d.department_name,
  j.job_title,
  l.city,
  l.state_province,
  c.country_name,
  r.region_name
FROM
  employees e
INNER JOIN departments d
ON d.department_id = e.department_id
INNER JOIN jobs j
ON j.job_id = e.job_id
INNER JOIN locations l
ON  l.location_id = d.location_id
INNER JOIN countries c
ON c.country_id = l.country_id
INNER JOIN regions r
ON  r.region_id = c.region_id
```

## <span style="color:#802548">_left join_</span>

- left join을 통해서 null값들도 붙여 원하는 값을 만들 수 있다.
- 아래는 실패 기록을 정리한 것이다.

```sql
select 
    count(case when de.department_id is null then 1 else null end) as "부서가 없는 도시수",
    count(distinct case when de.department_id is not null then lo.location_id else null end) as "부서가 있는 도시수"
from departments de  
left join locations  lo
on de.location_id = lo.location_id;  --1. departments로 기준 table을 설정해서 원하는 null 형태를 만들 수 없었다.
```

```sql
select 
    count(case when de.department_id is null then 1 else null end) as "부서가 없는 도시수",
    count(case when de.department_id is not null then de.department_id else null end) as "부서가 있는 도시수"
from locations  lo
left join  departments de 
on de.location_id = lo.location_id;  --2. locations을 기준 table을 설정했지만, 부서가 있는 도시수가 계속 전체 row수가 나왔다.
```

```sql
select 
    count(case when de.department_id is null then 1 else null end) as "부서가 없는 도시수",
    count(distinct case when de.department_id is not null then de.department_id else null end) as "부서가 있는 도시수"
from locations  lo
left join  departments de 
on de.location_id = lo.location_id;  --3. locations을 기준 table을 설정했고, 전체 row수가 나오는 겹치는 row를 줄여주기 위해 distinct를 넣어주었지만 여전히 전체 row가 보인다. department_id가 PK니까 당연히 distinct가 의미가 없다. 
```

```sql
select 
    count(distinct case when de.department_id is not null then lo.location_id else null end) as "부서가 있는 도시수(O)",
    count(case when de.department_id is null then 1 else null end) as "부서가 없는 도시수(X)"
from locations  lo
left join  departments de 
on de.location_id = lo.location_id;  --4. locations을 기준 table을 설정했고, 도시가 겹치는 게 문제이므로 lo.location_id를 distinct로 놓고 계산하면 원하는 결과가 나온다.
```


```sql
select 
    count(distinct case when de.department_id is not null then lo.location_id else null end) as "부서가 있는 도시수(O)",
    count(case when de.department_id is null then 1 else null end) as "부서가 없는 도시수(X)"
from locations  lo
left join  departments de 
on lo.location_id = de.location_id; --5. lo의 location_id가 index로 작동하는 것이므로 해당 column을 왼쪽으로 두어 index로 작동함을 명시적으로 알려준다.
```


- 아래처럼 group by를 활용하는 방법도 있다.
- 일반적으로 group by vs distinct의 경우, group by가 속도가 좋다.
- 단, memory가 많이 들어 temp로 빠져버리면 더 느리다.
- distinct의 경우, memory는 덜 들지만, 더 느리다.
- 따라서 distinct는 group by로 바꿀 수 있다면 바꿔보자.
    - 물론 exists로 바꿀 수 있다면 더더욱 좋긴 하다.
    - 다만 위의 조건은 case when에서 where 조건문 사용은 불가능해서 exists를 활용할 수 없다.
    - 물론 바꿨다면 실행계획을 통해 실제 performance 차이도 확인해줘야 한다.

```sql
select 
	count(case when dp_count > 0 THEN 1 END) AS "부서가 있는 도시(o)",
	count(case when dp_count <= 0 THEN 1 END) AS "부서가 없는 도시(X)"
from (
	select 
		loc.location_Id as loc_id,
		count(dp.department_id) as dp_count
	from departments dp
	left join locations loc
	on dp.location_id = loc.location_id
	group by loc.location_id
);
```


## <span style="color:#802548">_constraint_</span>

- null은 복수개가 있어도 unique key를 위배하지 않는다. null은 값이 아니기 때문이다.
- UK라고 해도 NN이어야 한다면, NN을 명시적으로 부여해야 한다.



## <span style="color:#802548">_table constraint_</span>

- not null 조건과 unique 조건은 같이 걸어주는 게 보통이다.
- unique한 조건만으로는 NN을 막을 수 없기 때문이다.

```sql
create table p_test(
	p_id int primary key,
    p_name varchar(20) not null unique,
    age tinyint,
    );
```

- check조건은 아래처럼 부여할 수 있다.

```sql
create table p_test(
	p_id int primary key,
    p_name varchar(20) not null unique,
    age tinyint,
    
    check ( 20 <= age and age <=24)
    );
```

- FK는 아래와 같은 형태로 부여할 수 있다.
- FK도 NN인 것이 좋다. NN이 아닐수는 있지만.. 그럼 테이블 설계가 잘못되었는지 살펴봐야 한다.

```sql
create table c_test(
		c_id int primary key auto_increment,
        c_name varchar(20) not null,
        c_score int not null,
        c_comment varchar(500), --, 없으면 안됨. , 없으면 뒤의 constraint syntax가 안 읽힘.
        
        
        constraint you_pk foreign key (c_name) references p_test(p_name)
    );
```

- 하지만 먼저 table을 만들고 alter로 FK를 나중에 부여하는 방법도 있다.

```sql
alter table my_emp
add constraint y_fk foreign key (wid) references wspace(wid);
```

- FK를 지울 때는 아래와 같이 해준다.

```sql
ALTER TABLE my_emp
DROP FOREIGN KEY y_fk
```

## <span style="color:#802548">_alter DDL_</span>

```sql
alter table my_emp
drop constraint y_fk 
```

```sql
alter table my_emp
add column m_salry smallint
```

```sql
ALTER TABLE my_emp
RENAME COLUMN m_salry TO m_salary;
```

```sql
alter table my_emp
modify column m_salary int
```

```
mysql 숫자형

Type	    Storage     Minimum Value Signed	    Maximum Value Signed	
TINYINT	        1	        -128		                    127	
SMALLINT	    2	        -32768		                    32767	
MEDIUMINT	    3	        -8388608		                8388607	
INT	            4	        -2147483648		                2147483647	
BIGINT	        8	        -2의 63제곱		                 2의 63제곱 -1	
```



## <span style="color:#802548">_entity-relation modeling_</span>

- oracle 용 logical ERD의 경우, 아래와 같은 규칙이 있다.
- FK는 logical이 아니라 물리 설계 때 들어간다.

```
pk는 #
UK는 (#)
optional은 o
NN은 *
```

- FK의 경우, 1 : N 중 N 쪽에 만드는 게 좋음.
- 다시 말해 <가 있는 table에 FK를 만드는 게 좋다는 의미.


- 정규화 등을 여러가지 하게 되면 아래와 같이 나오게 된다.

```
엔티티: 고객, 호텔, 예약, 호텔 룸, 호텔 부대서비스

고객 어트리뷰트
고객PK
성명
주소

예약 어트리뷰트
예약PK
고객PK
체크인타임
체크아웃타임
호텔 PK,
룸 PK


호텔 어트리뷰트
호텔PK
호텔이름
호텔주소


호텔 룸 어트리뷰트
호텔 PK
룸 번호/ 룸번호와 호텔 PK를 묶어서 compound PK로 지정
층번호
침대수
침대 프레임
룸레벨
주차가능 수
가격


호텔 부대서비스 어트리뷰트
호텔PK / FK가 이 table PK. identifying PK
조식
피트니스
스파
수영장
레스토랑
```




## <span style="color:#802548">_mysql dict_</span>

- mysql에는 중요한 dict가 아래와 같이 있다.
- mysql의 dict는 information_schema라고 불린다.

- information_schema 자체가 하나의 schema라서 그냥 select로는 정보를 조회하는 게 불가능하다.
- 아래와 같은 방식으로만 information_schema가 가진 table에 접근이 가능하다.

```sql
use information_schema;
show tables;
```


- 위처럼 조회해서 보이는 table들 중에 제약조건을 다룰 때 중요한 것은 아래와 같다.

```sql
desc information_schema.key_column_usage;
desc information_schema.table_constraints;
desc information_schema.referential_constraints;
```


- FK를 제외한 제약조건을 확인하는 방법은 아래와 같다.
- 두 개를 같이 해주는 게 좋다.

```sql
select * from information_schema.key_column_usage
where constraint_schema = 'youdb';

select * from information_schema.table_constraints
where constraint_schema = 'youdb';
```


- table info나 column info는 다른 table에 존재한다.

```sql
desc information_schema.tables;
desc information_schema.columns;
```


- table에 관련된 정보를 select해오는 sql이다.

```sql
SELECT
    table_name, table_comment
FROM
    information_schema.tables
WHERE
    table_schema = 'youdb' AND table_name = 'box_office';
```


- column에 관련된 정보를 select해오는 sql이다.

```sql
select * 
from 
    information_schema.columns
where 
    table_schema = 'world'
    and table_name = 'box_office';
```




## <span style="color:#802548">_oracle dict_</span>
- oracle의 dict는 dict다.

```sql
desc dict;
```

- 그 중에 constarint가 담긴 table은 아래와 같다.

```sql
desc USER_CONSTRAINTS;
desc USER_CONS_COLUMNS;
```

- constraint 확인하는 방법은 아래와 같다.

```sql
select TABLE_NAME
from dict
where TABLE_NAME like upper('%cons%');
```

- user가 가진 table 확인은 어차피 tab으로 한다.

```sql
select * from tab;
```

- column 확인은 아래와 같이 해준다.

```sql
select COLUMN_NAME from ALL_TAB_COLUMNS where TABLE_NAME='MyTableName';
```

