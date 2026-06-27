## <span style="color:#802548">_sql 파싱과 바인드 변수를 써야하는 이유_</span>
- SQL 실행 -> 파서 ->전처리 -> 옵티마이저 실행 계획 -> 스토리지 엔진에서 데이터 획득 ->결과 전달
  - 파서는 sql syntax 오류를 점검한다.
  - 전처리는 해당 db 오브젝트(table, view, column, index, 권한)이 실제로 있는지 검사한다.
  - 옵티마이저는 실행계획을 수립한다.
  - 스토리지 엔진에서 데이터를 가져온다. join, filter의 작업도 여기서 이뤄진다.
- DB 오브젝트 중 PK는 클러스터형 인덱스다. PK들끼리는 서로 가까워서 index 중에서도 더 빠르다.

- sql 최적화는 파싱 - 최적화 - 로우코드 생성으로 이뤄진다.
- 최적화는 optimizer가 수행한다. 개발자가 hint를 통해 제어할 수도 있다.
- 다만 hint를 쓸거면 촘촘하게 써야하고, 여러 조건을 ,없이 공백으로 구분해 나열해줘야 한다.
- 로우코드 생성까지 끝났다면, 생성한 내부 프로시저(SQL의 data가 아닌 SQL문 자체의 저수준 결과물)를 두고두고 쓴다.
- 그러한 공간을 라이브러리 캐시라고 한다. 라이브러리 캐시에 있는 걸 가져오면 소프트 파싱, 저수준까지 만들어야 하면 하드 파싱이다.
  - 하드 파싱이라 불리는 이유는 optimizer가 실행계획을 짜는 데 엄청 많은 것들을 연산하기 때문이다. 유일하게 CPU 연산이 엄청 들어간다.
  - SQL문을 만들 때 매개변수 없이 만들게 되면 매번 다른 SQL문을 저장하게 되므로 비효율적이다.
  - 아래 sql문은 모두 서로 다른 sql문으로 라이브러리 캐시에 저장된다.

```sql
SELECT * FROM emp WHERE empno = 7900;
SELECT * FROM emp WHERE EMPNO = 7900;
SELECT * FROM emp where empno = 7900 ;
SELECT * FROM EMP WHERE empno = 7900;
SELECT * FROM emp WHERE empno = 7900   ;
```

- 그러한 이유로 WAS단에서 sql을 만들 때는 아래와 같이 만들어선 안 된다.

```java
String sql = "SELECT * FROM CUSTOMER WHERE LOGIN_ID" = ' " + login_id " + "'";
Statement st = con.createStatement();
.
.
.
```


- 아래와 같이 서로 다른 SQL들이 모두 library cache에 저장되어 CPU 부족사태가 일어난다.

```
SELECT * FROM CUSTOMER WHERE LOGIN_ID = 'jaewon';
SELECT * FROM CUSTOMER WHERE LOGIN_ID = 'jinsoo';
SELECT * FROM CUSTOMER WHERE LOGIN_ID = 'jiksn';
SELECT * FROM CUSTOMER WHERE LOGIN_ID = 'cheolsu';
SELECT * FROM CUSTOMER WHERE LOGIN_ID = 'fira';
```

- 실제 내부 프로시저를 보면 서로 모두 이름이 다르게 저장되어 있다.

```sql
create procedure LOGIN_JAEWON();+
create procedure LOGIN_JinSoo();
create procedure LOGIN_JIKSN();
create procedure LOGIN_CHEOLSU();
create procedure LOGIN_FIRA();
```

- 이런 비효율적인 상황을 초래하지 않으려면 아래와 바인드 변수를 써줘야 한다.

```java
String sql = "SELECT * FROM CUSTOMER WHERE LOGIN_ID = ?";
PreparedStatement st = con.prepareStatement();
st.setString(1,login_id);
.
.
.
```

- 바인드변수로 1번의 값이 들어간다는 의미다.

```
SELECT * FROM CUSTOMER WHERE LOGIN_ID = :1;
```

- 실제 내부 프로시저는 아래와 같이 형성된다.
- 그럼 jaewon을 넣든, jinsoo를 넣든 아래에 이미 만들어진 프로시저를 재사용하게 되어 CPU연산이 부족할 일이 없다.

```sql
create procedure LOGIN (login_id in varchar2)
```


## <span style="color:#802548">_빠르게 가져오기 위한 메모리 영역_</span>
- DB는 데이터 접근 속도가 좋은 메모리에 정보를 저장하고 싶어함.
- 그러나 데이터를 메모리에 저장하면 비용이 듦.
  - 메모리 자체가 용량이 크지 않은데, 그걸 써야 하기 때문.

- 따라서 자주 접근하는 데이터만 메모리에 올리는 방식을 택함
  - 그럼 디스크(하드디스크 - HDD)에 접근하지 않기 때문에 속도가 압도적으로 빠름.
  - 바로 이런 성능향상 목적의 데이터를 저장하는 메모리= 버퍼, 캐시
  - 종류는 데이터 캐시(read), 로그 버퍼(DML), working memory(sort 용)
  - working memory는 data cache를 거쳐 private session memory에서 이뤄지는 게 일반적

- 데이터 캐시가 바로 메모리에 있는 데이터를 저장하는 버퍼. 흔히 버퍼라 하면 데이터 캐시
  - 버퍼에서 데이터가 없으면 HDD에 접근하고, 급속도로 느려짐. 디스크 I/O기 때문.


- 로그 버퍼는 갱신처리. insert, delete, update, merge를 때리면, 로그 버퍼 위에 변경 정보를 보내고, 디스크에서 변경을 수행하기 때문
  - 즉 갱신처리는 비동기로 이뤄짐. 이렇게 비동기로 하는 이유는 역시 성능 때문. 
  - 갱신 SQL -> 메모리의 로그 버퍼 -> commit -> HDD에 갱신
  - 다시말하면 commit 시에는 무조건 디스크 I/O가 일어나게 됨.
  - 보통 로그 버퍼는 용량이 굉장히 작음. DB는 검색을 기본으로 하기 때문에 데이터 캐시에 많이 용량을 배정하는 것.
  - 만약 갱신이 자주 일어난다면, 아래 절차가 가동됨. 

```
백그라운드 프로세스의 긴급 Flush (디스크 쓰기): ex) 용량의 1/3이 차거나, 1MB가 쌓였을 때 디스크의 Redo Log 파일로 내려씁니다.

사용자 트랜잭션 대기

Redo Log 파일 자체의 대기로 인한 DB전체의 대기: ex)  Redo Log 파일 자체가 가득 차는 경우
```

- 이에 대한 해결책은 아래와 같다.

```text
로그 버퍼 크기 확장

디스크 성능 개선: HDD -> SSD

Redo 파일 크기 최적화: 디스크에 있는 Redo Log 파일의 크기 키우기
```

- 실제로 멈추고 있나를 확인하는 방법은 아래와 같다.
- CHAT GPT 등에 물어보는 게 빠르다.
    - 트랜잭션 로그 파일의 크기 및 자동 증가(Growth) 설정 확인
    - 로그 버퍼 상태 카운터 조회
    - 로그 버퍼 대기 통계 조회

- 메모리 영역은 데이터 캐시, 로그 버퍼 이외에 워킹 메모리도 있음.
  - 해당 영역은 정렬, 해시 관련 처리에 사용됨. 
  - order by, max, min, sum 등의 집합 연산, window function, hash join 등에 사용된다.
  - oracle은 PGA, Mysql은 정렬버퍼라고 부른다.
  - 워킹 메모리가 들어온 데이터보다 작은 경우, 디스크 I/O(저장소 I/O)가 일어나 성능이 매우나빠진다.
    - 데이터양이 늘어서 메모리에 들어가지 않으면 어쩔수없이 저장소를 사용하는 것이다.
    - oracle에서는 TEMP TableSpace라고 부른다. 어쨌든 디스크 위에 존재한다.
    - 특히 하나의 sql문만으로는 데이터가 많지 않지만, 여러개의 sql문을 한꺼번에 수행하는 순간 메모리가 부족해져 디스크 I/O가 일어나기도 한다.
    - 즉 동접자가 많은 경우에 갑자기 성능 이슈가 생길수 있다는 의미다.

- 이러한 경우, disk I/O가 결국 일어나 버리는데, disk I/O가 일어났을 때는 load balancing으로 최대한 빠르게 처리한다
- load balancing 방법은 DB사마다 다르다. 
- mysql은 load balancing보다는 disk I/O가 안 일어나게 하는 데 집중한다.

```text
Oracle -> a Temp Group
SQL Server -> tempdb into multiple data files of equal size 
PostgreSQL -> RAID 0 / NVMe disk stripping
MySQL -> tmp_table_size and max_heap_table_size
```

- temp group만 잠깐 소개하자면, 이는 사람이 물리 disk를 나누고, 이를 바탕으로 load balancing을 하는 형태다
    - /disk1/oradata/temp01.dbf
    - /disk2/oradata/temp02.dbf
    - /disk3/oradata/temp03.dbf
    
```sql
CREATE TEMPORARY TABLESPACE temp01 
TEMPFILE '/disk1/oradata/temp01.dbf' SIZE 10G -- 👈 Physical Disk 1
TABLESPACE GROUP t_group;

CREATE TEMPORARY TABLESPACE temp02 
TEMPFILE '/disk2/oradata/temp02.dbf' SIZE 10G -- 👈 Physical Disk 2
TABLESPACE GROUP t_group;

CREATE TEMPORARY TABLESPACE temp03 
TEMPFILE '/disk3/oradata/temp03.dbf' SIZE 10G -- 👈 Physical Disk 3
TABLESPACE GROUP t_group;
```

- file system에 관한 간략한 설명이다

```text
  /disk1      /oradata      /temp01.dbf
  ──────      ────────      ───────────
    ①            ②               ③
Mount Point   Directory      File Name
 (The Disk)    (Folder)     (The Data)
```

## <span style="color:#802548">_group by_</span>
- group by를 쓰게 되면 group by를 위한 sorting 작업이 필요하므로 temp table을 만들 필요성이 높아진다.
- 다만 index가 있는 경우, index 자체가 sorting을 의미하기에 temp table을 만들지 않는다
- table scan을 할 때만 group by가 temp table에서(현대는 주로 hash)sorting 작업을 수행한다.

- 반대로 말하면, index가 있어도 index를 못타게 하는 것은 매우 위험하다는 의미다
- 아래의 경우처럼 group by column에 함수를 씌우는 경우 index를 못 타게 된다.

```sql
SELECT IFNULL(성별, 'NO DATA') AS 성별, COUNT(1) 건수
FROM 사원
GROUP BY IFNULL(성별, 'NO DATA')
```

```
| id | select_type | table | type  | possible_keys | key       | ref  | Extra                            |
|----|-------------|-------|-------|---------------|-----------|------|----------------------------------|
| 1  | SIMPLE      | 사원  | index | I_성별_성      | I_성별_성  | null | Using index; Using temporary;    |
```


- 성별 column은 not null 컬럼입니다.
- IFNULL을 쓰면 해당 처리를 위한 temp table이 생성됩니다. 그 결과 Using temporary가 추가되었습니다.

```sql
SELECT 성별, COUNT(1) 건수
FROM 사원
GROUP BY 성별
```

```
| id | select_type | table | type  | possible_keys | key       | ref  | Extra       |
|----|-------------|-------|-------|---------------|-----------|------|-------------|
| 1  | SIMPLE      | 사원  | index | I_성별_성      | I_성별_성  | null | Using index |
```

- group by에는 엔간하면 column 그 자체를 넣는 것으로 합니다.
- 이것만으로 index를 탈 확률을 높여서 임시 공간(Temp Space) 유발 확률을 크게 낮춰줍니다.


```sql
SELECT IFNULL(성별, 'NO DATA') AS 성별, COUNT(1) 건수
FROM 사원
GROUP BY 성별; -- 함수를 씌우지 않고 원본 컬럼으로 그룹핑
```

- group by의 본론으로 들어가자.
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

- count는 해당 column의 갯수를 세므로, 이를 group by로 묶어줘야만 한다

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

- count의 대상이 되는 group by가 없는 경우에는 전체가 대상이다

```sql
SELECT count(*)
    from Address
    GROUP BY (); /*GROUP BY ()는 없는 것과 동일. group by할 기준이 없기 떄문. 그냥 일반 select 문*/
```


```
count 
9
```

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


## <span style="color:#802548">_where 조건문은 access와 filter로 나뉜다_</span>
- 입사일자와 사원번호를 기준으로 filtering을 해 조회해보자.
- 아래와 같이 두개를 쓸 수 있다.

```sql
SELECT 사원번호
FROM 사원
WHERE 입사일자 LIKE '1989%'
AND 사원번호 > 100000;
```


```sql
SELECT 사원번호
FROM 사원
where 사원번호 > 100000;
and 입사일자 LIKE '1989%'
```

- 두 개의 조건 중 어떤 걸 앞으로 놓는게 좋을까?
- 뭘 앞에 두든 뒤에 두든 의미가 없다
- optimizer가 통계, index 등을 파악하여 access 조건을 알아서 선택하기 때문이다.
- 그 외 조건들은 filtering 조건으로 작동한다.

```sql
SELECT 사원번호
FROM 사원
WHERE '1989-01-01' <= 입사일자 AND 입사일자 < '1990-01-01' /** storage engine access 조건 */
AND 사원번호 > 100000; /** mysql engine filtering 조건 */
```

```
| id | select_type | table | type  | possible_keys        | key         | ref   | Extra                    |
|----|-------------|-------|-------|----------------------|-------------|-------|--------------------------|
| 1  | SIMPLE      | 사원  | range | PRIMARY,I_입사일자   | I_입사일자  | null  | Using where; Using index |
```

- where문에 쓰이는 조건은 access조건일 수도, filtering 조건일 수도 있다는 점이다.
- 주로 access조건에는 index가 있는 column이 쓰인다. explain에서 key로 되어있는 column이 곧 access condition으로 쓰인 것이다.
- 무조건 처음 오는 조건이 access condition이라고 optimizer가 해석하지는 않는다. 다만 그렇게 하는 게 읽기 좋은 쿼리문이다.


## <span style="color:#802548">_index column를 가공하지 않는다_</span>
- group by에서 index를 타지 못하게 되는 경우로서 column을 가공한 경우를 말했다.
- 사원번호가 1100으로 시작하면서 사원번호가 5자리인 사람을 출력해봅시다.

```sql
SELECT * 
FROM 사원
WHERE SUBSTRING(사원번호,1,4) = 1100
AND LENGTH(사원번호) = 5
```

```
| id | select_type | table | type | possible_keys | key | ref | Extra       |
|----|-------------|-------|------|---------------|-----|-----|-------------|
| 1  | SIMPLE      | 사원  | ALL  |               |     |     | Using where |
```

- WHERE절에 함수를 쓰지 않습니다. 시작점을 알수가 없기 때문이다.
- 함수를 쓰면 index를 타지 않습니다. 그 결과 type이 range에서 ALL이 되었습니다.
- 아래와 같이 바꿔줍니다.

```sql
SELECT *
FROM 사원
WHERE 사원번호 BETWEEN 11000 AND 11009
```

```
| id | select_type | table | type  | possible_keys | key     | ref  | Extra       |
|----|-------------|-------|-------|---------------|---------|------|-------------|
| 1  | SIMPLE      | 사원  | range | PRIMARY       | PRIMARY | null | Using where |
```

## <span style="color:#802548">_PK가 포함된 row는 DISTINCT X_</span>
- 부서 관리자의 사원번호와 이름, 성, 부서번호 데이터를 중복 제거하여 조회해봅시다.

```sql
SELECT DISTINCT 사원.사원번호, 사원.이름, 사원.성, 부서관리자.부서번호
FROM 사원
JOIN 부서관리자
ON 사원.사원번호 = 부서관리자.사원번호;
```

- 사원번호는 PK입니다. 따라서 row들은 중복이 있을 수가 없습니다.
- 그럼에도 DISTINCT를 사용하는 바람에 정렬- 삭제 작업을 위한 temp table이 추가됐습니다.
- 실행계획에서 Using temporary가 추가되었습니다.

```
| id | select_type | table       | type  | possible_keys | key       | ref                      | Extra                            |
|----|-------------|-------------|-------|---------------|-----------|-----------------------   |----------------------------------|
| 1  | SIMPLE      | 부서관리자 | index | PRIMARY         | I_부서번호 | null                     | Using index; Using temporary    |
| 1  | SIMPLE      | 사원        | eq_ref| PRIMARY       | PRIMARY   | tuning.부서관리자.사원번호 |                                  |
```

- 아래와 같이 distinct를 빼면 using temporary가 사라집니다.
- 필요없는 distinct를 호출해 메모리를 많이 쓸 필요가 없습니다.

```sql
SELECT 사원.사원번호, 사원.이름, 사원.성, 부서관리자.부서번호
FROM 사원
JOIN 부서관리자
ON 사원.사원번호 = 부서관리자.사원번호
```

```
| id | select_type | table       | partitions | type  | possible_keys | key        | key_len | ref                        | rows | filtered | Extra         |
|----|-------------|-------------|------------|-------|---------------|------------|---------|----------------------------|------|----------|---------------|
| 1  | SIMPLE      | 부서관리자   |            | index | PRIMARY       | I_부서번호 | 12      |                            | 24   | 100.00   | Using index   |
| 1  | SIMPLE      | 사원        |            | eq_ref| PRIMARY       | PRIMARY    | 4       | tuning.부서관리자.사원번호 | 1    | 100.00   |               |
```



## <span style="color:#802548">_좋은 index를 써야 한다_</span>
- batch가 아닌 web환경의 query는 index를 잘 써야한다.ㄴ

```sql
SELECT *
FROM 사원
WHERE 이름 = 'Gerogi'
AND 성 = 'Wielonsky';
```
- table full scan을 때리고 있다. 1건의 데이터를 가져오려고 table을 full scan하는 건 비효율적이다.
- 이걸 index scan으로 바꿔주자. 성과 이름 모두에 index를 만들어야 하는데, 순서도 중요하다.
- count가 더 많은 column을 선두에 놓는 게 성능에 유리하기 떄문이다. 더 많이 필터링할 수 있어서다.

```
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows  | filtered | Extra         |
|----|-------------|-------|------------|------|---------------|-----|---------|-----|-------|----------|---------------|
| 1  | SIMPLE      | 사원  |            | ALL  |               |     |         |     | 299379| 1.00     | Using where   |
```

- 따라서 아래와 같이 갯수를 구한다.

```sql
SELECT COUNT(DISTINCT(이름)) 이름_개수,
        COUNT(DISTINCT(성)) 성_개수,
        COUNT(1) 전체
  FROM 사원;
```
- 성_개수가 더 많으므로 데이터 범위를 더 축소할 수 있는 건 성이다. 따라서 성을 앞에 두고 index를 만든다.
- DISTINCT()로 보는 이유는 index는 null 및 겹치는 row를 제외하기 위해서다. 제외하는 이유는 High Cardinality Columns을 알기 위해서다.
- 그냥 COUNT(column)으로 하면 null을 제외한 전체 row가 항상 계산돼서 나온다.

```
	이름_개수	성_개수	전체
	1275	1637	300024

  ALTER TABLE 사원 ADD INDEX I_사원_성_이름 (성, 이름);
```

- 조건문이 있다면, 선택률을 보는 게 좋다. 선택률이 낮다면 index를 걸어주자.
- multiple 조건문이 있다면 where 조건문에서 일부 column만 index를 주면 안 된다.

```sql
SELECT * 
FROM 사원
WHERE 이름 = 'Matt'
OR 입사일자 = '1987-03-31'
```

- 입사일자에만 index가 걸려있어서 index scan이 아닌 ALL scan을 타고 있다.

```
| id | select_type | table | partitions | type | possible_keys | key | key_len | ref | rows   | filtered | Extra         |
|----|-------------|-------|------------|------|---------------|-----|---------|-----|--------|----------|---------------|
| 1  | SIMPLE      | 사원  |            | ALL  | I_입사일자    |     |         |     | 299379 | 10.02    | Using where   |
```

- 그럼 이름에 index를 주어도 되는지부터 파악해보자.
- 전체 row는 300024다.

```sql
SELECT COUNT(1) FROM 사원;
```

- 이름에 따른 row는 233이므로 233/300024하면 선택률이 1%도 안된다. index를 주기 적합하다.

```sql
SELECT COUNT(1) FROM 사원 where 이름 = 'Matt';
```

```
COUNT(1)
233
```

- 물론 이름이 Matt만 있는 것은 아니기에, GROUP BY로 count를 해서 다양한 경우의 선택률을 보는 게 좋다.
- 물론 자주 쓰는 것만 거의 쓴다면, 해당 경우만 보면 된다.
- 아래와 같이 count를 재니 전부 300이하다. 선택률이 1%가 채 안되니 index를 주기 적합한 field다.

```sql
SELECT 이름, COUNT(이름)
FROM 사원
GROUP BY 이름;
```

- 따라서 아래와 같이 index를 준다.

```sql
ALTER TABLE 사원 ADD INDEX I_이름(이름)
```

- 그럼 ALL SCAN이 아니라 INDEX_MERGE가 된다. 조건에 따른 row가 서로 겹치지 않는 것으로 보인다.

```
| id | select_type | table | partitions | type        | possible_keys                   | key                          | key_len | ref | rows | filtered | Extra                                   |
|----|-------------|-------|------------|-------------|---------------------------------|------------------------------|---------|-----|------|----------|-----------------------------------------|
| 1  | SIMPLE      | 사원  |            | index_merge | I_입사일자,idx_사원_이름          | idx_사원_이름,I_입사일자      | 44,3    |     | 344  | 100.00   | Using union(idx_사원_이름,I_입사일자); Using where |
```

- index 구성이 중요한 다른 예시를 살펴보자.
- 아래 sql을 위한 index를 고려하고 있다.

```sql
SELECT 사원번호, 이름, 성
FROM 사원
WHERE 성별 = 'M'
AND 성 = 'Baba'
```

- 그런데 성 열의 데이터는 총 1637건인데, 성별 열은 단 2건이다.
- 데이터가 다양하지 않은 field를 선두 index로 두는 것은 바람직하지 않다.
- 선택률이 쏠릴 확률이 매우 높기 때문이다.

```sql
SELECT COUNT(distinct 성) 성_개수,
      COUNT(distinct 성별) 성별_개수
FROM 사원;
```

- 성별이 먼저가 아니라 성이 먼저오게 index를 조정해주자.
- 지금은 데이터가 많지 않아 차이가 크게 없지만, 대용량 데이터가 되면 이런 순서도 성능에 차이를 낸다.

```sql
ALTER TABLE DROP INDEX I_성별_성(성별, 성),
            ADD INDEX I_성별_성(성,성별);
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



## <span style="color:#802548">_JOIN_</span>
- join에서 on은 어떻게 짝을 맞출까에 관한 행위다.
    - left join에서는 왼쪽 테이블 기준으로 짝이 없어도 null을 넣어버린다
    - inner join에서는 양쪽 테이블 기준으로 짝이 안 맞으면 해당 행은 제외한다
- where에선 최종적으로 해당 조건에 안 맞는 모든 행을 filter한다

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
- 이 경우 driven table이 크면 클수록 index 사용에 따른 반복 생략 효과가 커진다.

```
최적의경우 -> 접근 레코드수: A table Record X 2
해당 레코드를 driven table의 index만으로 찾을 수 있는 경우
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

- driving table을 작게 만드는 게 통상적으로 좋다.
- 하지만, on절로 driven table을 조회하려 할때, join record가 많다면?
  - 그 떄는 역설적으로 driving table을 큰 테이블로 선택하는 게 나을 수 있다.
  - 따라서 선택률에 따라 driving table을 큰 테이블로 선택하는 게 나을 수도 있다.


## <span style="color:#802548">_subquery_</span>
- subquery는 단점이 많다.
  - subquery는 실제 데이터를 저장하지 않는다. 따라서 서브쿼리에 접근할 때마다 SELECT문이 실행된다.
  - 연산 결과를 저장하기 위해 메모리 혹은 디스크를 사용해야 하는데, 디스크를 사용하게 되면 속도가 느려진다.
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
- 아래와 같이는 사용해도 된다.

```sql
SELECT *
    FROM SomeTable
    WHERE index_column > 100;
```

- 그러나 아래와 같이는 사용하면 안 된다.

```sql
SELECT *
    FROM SomeTable
    WHERE index_column * 1.1 > 100;
```

- 연산이 필요하다면 우변에다가 해줘야 한다.

```sql
SELECT *
    FROM SomeTable
    WHERE index_column  > 100 * 1.1;
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


- join에서 index를 나타내는 것은 무조건 왼쪽에 쓰자.
- 아래 sql을 보자.
- 해당 sql을 보고 index를 PRA_HST_STC_N1 table에 (SALE_ORG_ID, STRD_GRP_ID, STRD_ID, STC_DT)로 만들려 했다면 잘못 이해한 것이다.
- 일단 전부 a를 왼쪽에 배치한거부터 매우 잘못되었다.

```sql
SELECT * 
FROM PRA_HST_STC a, ODM_TRMS b
WHERE a.SALE_ORG_ID =:sale_org_id
AND a.STRD_GRP_ID = b.STRD_GRP_ID
AND a.STRD_ID = b.STRD_ID
ORDER BY a.STC_DT desc
```

- index가 걸리는 join 조건절임을 나타내기 위해 b를 왼쪽으로 옮겨줘야 한다.
- 그럼 index를 어떻게 만들어야할 지 확실하게 보인다. 
  - PRA_HST_STC table에 (SALE_ORG_ID, STC_DT)로 index를 만들어줘야 한다.
  - ODM_TRMS table에 (STRD_GRD_ID,STRD_ID)로 index를 만들어줘야 한다. 

```sql
SELECT *
FROM PRA_HST_STC a, ODM_TRMS b
WHERE a.SALE_ORG_ID = :sale_org_id
AND b.STRD_GRP_ID = a.STRD_GRP_ID
AND b.STRD_ID = a.STRD_ID
ORDER BY a.STC_DT desc
``` 


1. 일반 쿼리에서 두 메모리의 명확한 역할 분담만약 인덱스를 타는 일반적인 SELECT * FROM customer WHERE customer_no = 100; 쿼리가 실행된다면 다음과 같이 일하게 됩니다.버퍼 캐시(SGA)의 역할 (택배 허브):오라클은 100번 고객이 저장된 디스크 블록을 읽어서 버퍼 캐시(SGA)에 올려놓습니다.이곳은 단순히 "디스크에서 읽어온 데이터 블록을 메모리에 캐싱해두는 보관소"일 뿐입니다.세션 메모리(PGA)의 역할 (내 책상):버퍼 캐시에 올라온 블록에서 100번 고객의 실제 데이터(이름, 주소 등)를 쏙 뽑아와서 내 세션 메모리(PGA)로 가져옵니다.내 세션 메모리에서 데이터를 최종 가공(예: 문자열 합치기, 날짜 포맷 바꾸기)한 뒤, 로그인한 사용자 화면(Java, Web 등)으로 최종 전달합니다.즉, 버퍼 캐시(SGA)는 데이터를 가져오는 통로이자 보관소이고, 세션 메모리(PGA)는 그 데이터를 받아서 실제 연산을 처리하는 작업 공간입니다.

2. 일반 쿼리가 세션 메모리(PGA)를 필수적으로 쓰는 순간들특히 다음과 같은 연산이 포함된 일반 쿼리들은 세션 메모리(PGA)의 크기가 성능을 좌우합니다.ORDER BY (정렬): 버퍼 캐시에서 읽어온 데이터들을 내 세션 메모리(PGA)에 한 줄로 쭉 세워놓고 정렬 순서를 바꿉니다.GROUP BY / DISTINCT (집계/중복제거): 중복을 체크하기 위해 세션 메모리에 임시 해시 맵을 만들어 둡니다.세션 정보 보관: 내가 로그인한 세션의 ID, 언어 설정, 권한 정보 등 나만의 고유한 정보들이 전부 세션 메모리에 상주합니다.