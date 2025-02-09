## <span style="color:#802548">_DB는 테이블 중심_</span>

- DBMS는 instance와 database로 나뉜다.
- 중요한 것은 instance다.
  - instance는 RAM에
  - database는 HDD쪽에 저장된다.
- DDL은 데이터가 들어갈 공간을 정의함. 
  - 테이블을 만들고 테이블을 다룬다는 의미다.
    - 테이블 만들기
    - 테이블 column data type 수정하기

## <span style="color:#802548">_DB의 진화 역사_</span>

- 가장 처음에는 파일형이었다.
  - 개발자마다 파일의 헤더블록이 달라 공유가 안됐다.
  - 그래서 똑같은 내용인데 또 만들어야했다.
- 그런 문제를 해결하는 것이 계층형이다.
  - 그러나 파일간에 1:1로만 연결되는 상하관계라 속도가 느렸다.
- 그래서 네트워크형이 등장했다.
  - 그러나 네트워크형은 이전과 달리 포인터라는 개념을 사용했다. 
  - 따라서 프로그래밍 이해도를 요구해서 어려웠다. 
- 프로그래밍의 포인터 개념을 몰라도 되게 RDB가 등장했다.
  - RDB는 join을 쓸 때 포인터개념 몰라도 돼서 편하다.
- 최근에는 OODB로 전환하려는 흐름이 있었다.
  - 결과적으로는 실패했다.
  - 하지만 일부가 흡수되었다. 
  - 테이블이 객체마냥 늘어나고 줄어들게 바뀌었다.


## <span style="color:#802548">_DB 여러 용어_</span>

```
행 -- row, tuple, record
열 --- column, attribute, variable, feature
테이블 --- table, relation
vector ---> 1차원 배열. column, row 하나씩만. 
dataframe ---> 2차원 배열. table. RDB에서 씀.
matrix -----> 2차원배열인데, table과 달리 모든 data type이 고정
array ----> ??
list ----> NoSql에서 씀. array, matrix, dataframe, vector 모두 넣기 가능
```

## <span style="color:#802548">_DB cmd 접속_</span>

- linux 서버에서 db에 접속하려면 command를 알아야 한다.
- 현재 OS 계정이 OS의 주인이라면, sysdba로 비밀번호 설정없이 바로 접속 가능하다.
- 하지만 user가 OS의 계정이 아니라면 아래 방식으로 접근이 불가능하다.

```sh
sqlplus sysdba

##

sqlplus /nolog
conn /as sysdba #sys가 db user
```

- sys는 최상위기 때문에 모든 객체의 정보를 조회할 수 있다.

```sql
select username, account_status from dba_users
```

- 그 중에 hr도 있는데, oracle을 깔면 생기는 test용 유저다.

- 처음에는 잠겨있으므로 lock을 풀고 expire를 풀어준다.

```sh
alter user hr account unlock;
alter user hr identified by [비밀번호];
```

- sys로 테이블을 건드리는 것은 바람직하지 않다.
- 풀어놓은 hr user로 접속해 학습을 진행한다.

```sh
conn hr/scit # hr이 db user
```

- hr로 접속한 뒤 가지고 있는 객체를 살펴본다.

```sql
select * from tab;
```





  ## <span style="color:#802548">_DB connection과 session_</span>

- connection은 TCP 연결을 의미한다.
- tcp 연결이 되었다면 끝이 아니다.
- db에 접속해서 사용하려면 login이 필요하다.
  - login을 session으로도 표현한다.
  - session은 곧 user라는 의미다.

```
connection - tcp 연결 (물리적 연결)
session - db를 사용하기 위한 login (논리적 연결) 
```

- session의 차이는 commit을 하지 않았을 때 명확하다.
  - A session에서 row insert하고서 commit 안 하면 51개다.
  - 그리고 Bsession에서 조회 시 45개로 조회된다.
- session 간의 data 불일치를 조정하려면 반드시 commit을 해야 한다.
- DDL 명령어들을 사용하게 되면 여태까지의 transaction이 자동으로 commit된다.
  - DDL 명령어 시 별도의 commit 없이 Bsession 조회 시에도 51개로 보인다.

```
CREATE TABLE ...
DROP ...
```

- oracle은 로그인 시, 맨 처음 glogin.sql을 읽어서 적용한다.
  - 해당 파일은 건드리지 말자.
- 대신 login 시 특정 환경변수를 적용하고 싶다면, login.sql을 작성하자.
  - glogin.sql 이후에는 login.sql을 읽기에 login.sql에 적어도 된다. 
  - pagesize라던가, linesize라던가... 그런 것들은 session별로 적용되기 때문.
- login.sql말고 다른 이름의 파일은 로그인 시에 읽지 않으므로 환경변수들이 변경되지 않음.

- 해당 user가 어떤 권한을 가졌는 지 확인하려면 아래 sql을 사용한다.

```sql
select * from session_privs; --해당 user가 어떤 권한을 가졌는지 확인
```

- 아래와 같은 naming rule에 따라 객체들을 생성한다.

```
dba_  (dba_users, ... )
user_ (user_tables, user_trigger ... ) 
all_ (전부..)
v$_ (tunging 관련. dynamic performance view)

https://isstory83.tistory.com/138 
```


- 따라서 sql이 아래와 같은 식인 것이다.

```sql
select username, account_status from dba_users;

select tname from user_tables;
```

  ## <span style="color:#802548">_DB schema_</span>

- schema는 각 dbms에서 의미하는 바가 다를 수 있다.
- 즉 oracle에서 말하는 schema는 해당 user가 갖고 있는 schema다.
- 그런 의미에서 oracle에서는 session = schema = user다.
- 하지만 일반적으로 schema는 일개 user가 소유한 object가 아니다.
  - user들이 가진 object의 합, 즉 system이 소유한 object를 schema로 부른다.
  - 차이가 있음을 인지해두면 뭔가 오해가 있다고 생각될 때 질문 때리면 된다.
  - schema가 db를 의미하는지, user를 의미하는지

## <span style="color:#802548">_SQL 파일 만들고 실행하고 열고 저장하기_</span>

- 아래 명령어를 사용하여 sql 파일을 만들 수 있다.

```sh
save 1.sql cre
```

- 만들어 둔 sql 파일을 열 수 있다.

```sh
ed 1.sql
```

- 만들어 둔 sql을 실행하려면 @ + sql파일이름을 사용하면 된다.
- oracle process에 접속한 상태에서만 사용가능하다.

```sh
@1
```

- 다만 sql파일을 실행하려면 sql파일은 가장 밑에 /를 붙여주여야 한다.

```sql
select * from employees;
/
```


- 만약에 sqlplus에 접속한 위치에 1.sql이 없고 상위에 있을 수있다.
- 그럴 때는 상위로 올라가야 한다.

```sh
@..\1 #(/가 아님) 
```

- 내가 방금 막 실행한 쿼리를 다시 실행해보려면?

```sh
/ 
#
run
```

- 내가 방금 막 실행한 쿼리를 실행하지 않고, 쿼리 자체만 보고 싶다면?

```sh
list
#
l (엘)
```


- 내가 방금 막 실행한 쿼리를 원래 sql에 추가해 저장하고 싶을 때가 있다.

```sh
save 1.sql app
```


## <span style="color:#802548">_oracle process 안에서 os process command 쓰기_</span>


- oracle process에서 os command를 실행하면 듣지 않는다.
- 그럴 때, $나 !를 붙여 쓴다.
  - $는 window 
  - !는 linux

- 혹시 app으로 append한 내용을 oracle process에서 보고 싶다면?

```sh
$type ..\1.sql
```

- 마찬가지로 만약 cmd를 많이 써서 깨끗이 지우고 싶다면?

```sh
$cls - window
#
!clear - linux
```

## <span style="color:#802548">_oracle DDL_</span>

- create 정식 command도 있지만, test 시에는 아래처럼 복사해서 야매로 만들면 된다.
- 만약 table의 row는 가져오지 않고 틀만 만들고 싶다면 row가 없어야 한다. 

```sql
create table m_emp
as 
select
	department_id as dno,
	employee_id as eno,
	LAST_NAME  || ', ' || FIRST_NAME as f_name,
	job_id as job,
	salary as sal
from employees
where rownum < 1 -- < 1이어야 한다. 그래야 row 안 가져오기.
```

- row를 채우는 방식도 야매로 가져오면 된다.

```sql
insert into m_emp
select
	department_id as dno,
	employee_id as eno,
	'%' || LAST_NAME  || ', ' || FIRST_NAME as f_name,
	job_id as job,
	salary as sal
from employees
where department_id = 50;
```

- 만약 table의 row도 가져오고 싶다면 rownum 조건을 변경한다.

```sql
create table m_emp
as 
select
	department_id as dno,
	employee_id as eno,
	LAST_NAME  || ', ' || FIRST_NAME as f_name,
	job_id as job,
	salary as sal
from employees
where rownum < 10 -- 1개 이상으로 설정한다.
```



- oracle에서는 table 복구를 지원하는 command가 있다. 
- 테이블 지워버리면 BIN$4cCOiqa1Qweia1n5GHIzDg==$0 와 같은 형태로 남기는데, 이걸 복구하는 것이다.

```sql
flashback table [테이블명] to before drop
```


- 완전히 복구 안되게끔 쓰레기통에서마저 지워버리는 것은

```sql
purge table [table]
```
- 처음부터 쓰레기통에도 안 넣고 복구불가하게 삭제하는 것은

```sql
drop table [table] purge
```




## <span style="color:#802548">_oracle select cmd에서 보기 편하게 조절하는 option_</span>

- gui와 다르게 cmd는 전부 알아서 조절해야 한다.
  - linesize
  - pagesize
  - column
    - 숫자는 9999 형태 
    - 문자는 a35 형태 (35줄까지)

- sqlplus 명령어라서 ;를 꼭 쓰지 않아도 된다.

```sql
col username format a20
col account_status format a35 
set linesize 200 
set pagesize 30
```

## <span style="color:#802548">_SQL- select parsing 순서_</span>

- select가 들어오게 되면 아래 순서로 DB가 읽어들인다.
  - from 절
  - where 절
  - rownum이 있다면 순서부여
  - select 절
  - order by 절
- 만약 from 절이 복수 개의 subquery라면?
  - 제일 깊이 있는 subquery부터 읽는다.

- 이러한 순서의 차이로 인해 where 절에서는 alias를 쓸 수 없다.
- where 절은 column 취급되는 거만 쓸 수 있다.
- 하지만 order by는 select 이후에 읽힌다.
- 따라서 order by에선 alias를 쓸 수 있다.

```sql
select 
	department_id, 
	first_name, job_id, 
	salary * 12*1300 as yearly_salary, 
	commission_pct as comm 
from 
	employees 
where 
	commission_pct is not null 
order by 
	5 asc, yearly_salary desc
-- select 문에서 쓴 5번째 column이 위의 5인데, 쓰지 말것. 가독성 ㅎㅌㅊ
-- where clause는 alias 못씀. 아래는 안 됨.

where
	year_salary > 150000 
```

## <span style="color:#802548">_SQL- select의 distinct_</span>


- distinct는 행단위다. 
- 그래서 job_id, last_name을 distinct로 적용한다면?
  - 두개의 column을 합쳐서 unique를 만족하는 row를 찾는다.
  - 따라서 job_id와 last_name 모두 distinct 한 것을 찾는 게 아니게 된다.
  - 2개 이상의 행을 distinct로 쓰면 주의해야 한다.
- 뚀한 아래와 같이 두 개를 모두 ()로 묶는 것은?
- 불가능하다.

```sql
select distinct(JOB_ID) from employees where DEPARTMENT_ID=50 OR DEPARTMENT_ID=100;

select distinct(JOB_ID,LAST_NAME) from employees where DEPARTMENT_ID in (50,100);
-- error
```

- distinct는 두 개 이상의 column을 넣을 떄는 괄호로 묶지 않고 사용해야 한다.
- 가독성이 크게 떨어지므로 크게 추천되지 않는다.

```sql
select distinct JOB_ID, LAST_NAME from employees where DEPARTMENT_ID in (50,100);
```

## <span style="color:#802548">_where like operator_</span>

- like는 유사한 문자를 검색하는 operator다.
- %가 앞에 붙으면 앞쪽을 무시하는 것이다.
- 이러면 뒤에 IT라는 문자가 있는 row를 검색하게 된다.
- 앞의 문자가 어떻게 되든 상관쓰지 않는 것이다.

```sql
select 
	* 
from 
	departments 
where 
   department_name like '%IT'
```

- 반면에 %가 뒤에 붙으면 뒷쪽을 무시하는 것이다.
- 이러면 앞에 IT라는 두 글자가 있는 row를 검색하게 된다.
- 뒤의 문자가 어떻게 되든 상관쓰지 않는다.


```sql
select 
	* 
from 
	departments 
where 
   department_name like 'IT%'
```

- n번째 문자를 검색하고 싶다면 _를 넣어서 like로 검색할 수 있다.
  - _가 4번 반복되면, 앞의 4번째 글자를 무시한다.
  - 5번째 글자는 j이다.
  - 뒤 글자는 어떻게 되든 신경쓰지 않는다.

```sql
select 
	name,
    population,
    countrycode
from city ci 
where ci.name like '___j%' or ci.name like '___w%' 
and population >= 500000; 
-- _를 한 숫자만큼 앞의 문자열들을 무시함. index가 제대로 scan이 안되니 성능은 구림.
```

## <span style="color:#802548">_where clause special char escape_</span>

- 특수문자를 where 절에서 쓸 때는? escape가 필요하다.
  - escape 를 써서 예약어로서 기능하지 않게끔 조절할 수 있다.
- escape가 없이는 아래 query가 돌아가지 않는다.

```sql
select * 
from 
	m_emp
where f_name like '_%'
```

- _로 시작하는 것을 찾고 싶다면 아래와 같이 실행시킴.
- like 연산자에서 \기호 다음은 예약어가 아니다라고 해석됨. 

```sql
select * 
from 
	m_emp
where f_name like '\_%' escape '\'
```

- 마찬가지로 %를 포함한 것(시작하는 거 아님)도 아래와 같이 쓴다.

```sql
select * 
from 
	m_emp
where f_name like '%\%%' escape '\';
```

## <span style="color:#802548">_SQL- where clause null_</span>

- where clause에서 null을 다룰 때는 위처럼 =를 쓰지 않는다.
- is와 is not을 쓴다.

```sql
select 
	* 
from 
	departments 
where 
	MANAGER_ID is null 
```


## <span style="color:#802548">_SQL- select alias escape_</span>

- select alias에 특수기호를 써야 할 때는 위처럼 복잡한 escape가 필요하지 않다.

- ''는 문자열 전용이고, 여기서는 ""를 써줘서 감싸면 된다.

```sql
select
      salary / 20 / 8 as "시급(원)"
from employees;
```



## <span style="color:#802548">_SQL- rownum top n stopkey algorithm_</span>

- rownum은 select로 가져올 때 접근가능하다.
- 아래처럼 하면 rownum대로 나오지 않음.

```sql
select 
  rownum, 
  salary 
from employees 
where rownum < = 5
order by 2 desc;
```

- 읽는 순서가 from ---> where ---> select ---> order by
- 따라서 from을 읽고 select 를 읽으면 rownum으로 순위를 부여하게 된다.
  - rownum은 순서를 정렬하는 개념이 아니다. 
  - 정렬하게 되면 file I/O를 동반할 수 있는 작업이 수반된다.
  - rownum은 sorting 전에 query가 끝나면 부여되는 가상의 번호다.
- rownum을 부여하고 select를 해서 가져오게 된다. 
  - 그 과정에서 rownum이 먼저 적용되고, 5개만 가져오게 된다.
  - 그 5개의 row를 order by로 정렬하게 되는 것이다.
- 의도처럼 5명의 가장 많은 돈을 받는 사람을 뽑아오지 않는다.
- 그러한 의도라면 아래처럼 subquery를 활용해 table을 따로 만들어줘야 한다.

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

 - pagination을 쓰려면 아래와 같이 쓸수도 있다.
 - 하지만 그렇게 하면 stopkey 효과를 받을 수 없다.
 - order by 할 때 가져온 전체를 order by 한다는 의미다.


 ```sql
select 
    *
    from
        (
            select rownum as rank, name, sal 
            from (
                    select first_name as name, salary as sal from employees
                )
        )
where rank between 6 and 17;
```

- 아래처럼 rownum을 활용해주면 order by 시에도 n개만 sort하게 된다.

```sql
select *
from
(
  select a.*, rownum rnum
  from
    (
      select 
            id, 
            data
      from t
      order by id
    ) a
  where rownum <= 150
)
where rnum >= 148;
```

- 위의 query를 적용해 바꿔보자.
- 다만 아래의 경우, ORDER BY에 쓰인 column이 중복된 column이 있으면 안 된다.
- 중복된 column이 많으면 order by가 제대로 이뤄지지 않는다.

```sql
select 
    *
from
    (
      select a.*, rownum as rnum
      from (
            select 
                first_name as name, 
                salary as sal 
            from employees
            order by salary desc
        ) a
      where rownum <= 17
    )
where rnum >= 6;
```

- ORDER BY로 쓸 column이 중복된 column이 많다면? 
- order by 조건에 rowid도 섞어서 써주자.

```sql
select 
    *
from
    (
      select a.*, rownum as rnum
      from (
            select 
                first_name as name, 
                salary as sal 
            from employees
            order by salary, rowid desc
        ) a
      where rownum <= 17
    )
where rnum >= 6;
```

- oracle 11g부터는 prefetch가 도입됐기에 맨마지막에도 orderby도 써줘야 한다.
- 이걸 쓴다고 또 정렬하지는 않는다. 그러니 아래처럼 쓰면 된다.

```sql
select 
    *
from
    (
      select a.*, rownum as rnum
      from (
            select 
                first_name as name, 
                salary as sal 
            from employees
            order by salary, rowid desc
        ) a
      where rownum <= 17
    )
where rnum >= 6;
order by salary, rowid desc; 
```

- 참고할 좋은 link

```
https://blogs.oracle.com/connect/post/on-rownum-and-limiting-results
```





## <span style="color:#802548">_sql developer shortcut_</span>
- block 주석
  - alt shift c



## <span style="color:#802548">_data dictionary_</span>
- parser가 parse하는 규칙은 data file 속의 data dictionary에 있다.

```sql
desc dict;  ------------데이터 dictionary의 형태를 check
```

- dictionary를 보려면?

```sql
select count(*) from dict;

select * from dict;
```

## <span style="color:#802548">_user 권한_</span>

- sys와 hr이라는 user는 서로 디스크에 할당된 공간 자체가 아예 분리되어있다.
- 따라서 각각의 user가 따로 동일한 이름의 tt라는 테이블을 만들수 있음.
  - 각각으로 보관된다.

```sql
select * from tt;
sys --> jw
hr --> scit
```

- 그런데 권한에 따라 볼 수 없는 table도 존재한다.
- sys에서는 hr user의 tt table을 조회해도 조회가 된다.

```sql
select * from hr.tt (hr schema의 scit) 
--scit
```

- 그러나 hr에서는 조회되지 않는다.
- 권한이 없기 때문. sys가 만든 것은 오직 sys만이 볼 수 있다.
- 반면에 sys는 모든 user의 table을 볼 수 있다.

```sql
select * from sys.tt
--error. table does not exist
```

## <span style="color:#802548">_oracle data formatting- 숫자에 콧마 부여_</span>
- to_char로 문자형 변환 시 형식을 줄 수 있다.
- 세자리수마다 콧마가 찍히게 된다.

```sql
 to_char(salary / 20 / 8, '$999,999') as hr$,
```

- L을 준 것은 그 나라 화폐 통화 단위로 적용됨을 의미한다.

```sql
to_char(salary / 20 / 8 * 1300, 'L999,999') as hrw,
```

- 자국이 아닌 다른 나라를 적용하고 싶다면?
  - L로 해놓고 3번째 옵션을 주면 된다.

```sql
to_char(salary / 20 / 8 * 1300, 'L999,999', 'NLS_CURRENCY=￥') as hrw,
```

## <span style="color:#802548">_mysql data formatting- 숫자에 콧마 부여_</span>
- to_char로 문자형 변환 시 형식을 줄 수 있다.
- 세자리수마다 콧마가 찍히게 된다.

```sql
format(sale_amt,0) from box_office;
```

- 통화로 바꿔주려면 format이 아닌 concat으로 합쳐줘야 한다.
- oracle처럼 to_char가 모든 걸 담당하지 않는다.

```sql
CONCAT('$', format(sale_amt,0) );
```


## <span style="color:#802548">_oracle data formatting- 현재 시간과 row datetime 격차를 years로 나타내기_</span>

- 개월 수도 구해서 내림해버리면서 years를 붙여주면 formatting된다.

```sql
trunc(months_between(sysdate, hire_date) / 12) || ' years' as num_yy 
```

```sql
select *
from (
    select 
        rownum as RANK, 
        id as 사번, 
        fullname as 성명,
        job as 직무, 
        yy$ as 연봉,
        hr$ as 시급$,
        hrw as "시급(원)"  
    from (
        select 
            employee_id as id,
            last_name || ', ' || first_name as fullname,
            job_id as job,
            to_char(salary * 12, '$999,999') as yy$,
            to_char(salary, '$999,999') as mm$,
            to_char(salary/20, '$999,999') as dd$, --법정 공휴일 제외한 날짜가 20일
            to_char(salary / 20 / 8, '$999,999') as hr$,
            to_char(salary / 20 / 8 * 1300, 'L999,999'  /* , 'NLS_CURRENCY=￥' */) as hrw,
            hire_date,
            trunc(months_between(sysdate, hire_date) / 12) || ' years' as num_yy 
        from employees
        order by hr$ desc 
    )
)
where rank  between 15 and 25;
```

## <span style="color:#802548">_mysql data formatting- 현재 시간과 row datetime 격차를 years로 나타내기_</span>

- months_between 함수는 TIMESTAMPDIFF함수로 대체한다.
- truncate 함수는 얼만큼 버릴지 parameter로 정해줘야 한다. 여기선 0이다.

```sql
SELECT CONCAT(
TRUNCATE(
	TIMESTAMPDIFF(MONTH, release_date, now()) / 12, 
    0
),' years'
) as 경과시간
from box_office;
```

- 달로 나눠서 구하는 게 귀찮다면, TIMESTAMPDIFF 단위를 year로 바꾸면된다.
- 그럼 TRUCNATE 함수를 쓸 필요가 없다.

```sql
SELECT 
	movie_name,
    CONCAT(
      TIMESTAMPDIFF(YEAR, release_date, now()), 
      ' years'
    ) as 경과시간
from box_office;
```


## <span style="color:#802548">_oracle data formatting- full name에서 성과 이름 빼내기_</span>

- 이름에서 성과 이름을 자를 때 쓰는 query다.
- instr이 index다. db는 index가 0이 아닌 1부터 시작이다.
  - 따라서 + 1, -1을 해주는 것이다.

```sql
select
    fullname as FULL_N,
    SUBSTR(fullname, instr(fullname,' ') + 1) as FAMILY_N,
    SUBSTR(fullname, 1, instr(fullname,' ') - 1) as GIVEN_N,
from 
    my_emp;
```

## <span style="color:#802548">_mysql data formatting- full name에서 성과 이름 빼내기_</span>

- 이름에서 성과 이름을 자를 때 쓰는 query다.
- instr이 index다. db는 index가 0이 아닌 1부터 시작이다.
  - 따라서 + 1, -1을 해주는 것이다.

```sql
select
    movie_name as FULL_N,
    SUBSTRING(movie_name, instr(movie_name,' ') + 1) as FAMILY_N,
    SUBSTRING(movie_name, 1, instr(movie_name,' ') - 1) as GIVEN_N
from 
    box_office;
```

## <span style="color:#802548">_oracle data formatting- 숫자 억 단위로 문자 변환_</span>

- 634,353,666,251 원이라면, 이를 억의 자리까지만 보여주는 기술이다.

```sql
to_char(round(a.매출)/pow(10,8), '999,999') || '억' as hrw,
```


## <span style="color:#802548">_mysql data formatting- 숫자 억 단위로 문자 변환_</span>

- 634,353,666,251 원이라면, 이를 억의 자리까지만 보여주는 기술이다.

```sql
 concat(format(round(a.매출)/pow(10,8),0),'억') as 매출,
```
