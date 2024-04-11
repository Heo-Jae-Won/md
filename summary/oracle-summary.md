## <span style="color:#802548">_sql 파싱_</span>
- sql 파싱은 optimizer가 SQL을 보고 실행계획을 만드는 행위다.
  - 처음에 SQL이 파싱되면 저수준까지 모두 만들어야 한다.
  - 첫 파싱이 끝나면 SQL 구문을 key로 실행계획 등이 SGA의 library cache에 저장된다.
    - 라이브러리 캐시에 있는 걸 가져오면 소프트 파싱이다.
    - 저수준까지 만들어야 하면 하드 파싱이다.
- 아래 sql문은 모두 서로 다른 sql문으로 라이브러리 캐시에 저장된다.
- 그런데 캐시가 부족하면 바로 버려진다.
```sql
SELECT * FROM emp WHERE empno = 7900;
SELECT * FROM emp WHERE EMPNO = 7900;
SELECT * FROM emp where empno = 7900 ;
SELECT * FROM EMP WHERE empno = 7900;
SELECT * FROM emp WHERE empno = 7900   ;
```

- 그러한 이유로 WAS단에서 sql을 만들 때는 아래와 같이 만들어선 안 된다.
- 실제로 대부분의 Java에서 사용되는 DB library들은 실제로 아래와 같은 쿼리문을 만들지 않는다.
```java
String sql = "SELECT * FROM CUSTOMER WHERE LOGIN_ID= '" + login_id " '";
Statement st = con.createStatement();
```

- 로그인을 위와 같은 식으로 만들었다고 해보자.
  - 그런데 고객이 500만명이다. 내일 10시에 이벤트가 있어 100만명이 몰린다.
  - 라이브러리 캐시에서 이미 버려진 고객들이 많다.
  - 따라서 대부분 고객의 이름마다 하드파싱이 일어난다.
  - CPU 사용량이 치솟아 버리고, 하드파싱끼리 서로 누가 먼저 할지 경합하게 된다.
- 이런 비효율적인 상황을 초래하지 않으려면 아래와 바인드 변수를 써줘야 한다.
  - 보통 JAVA에서는 PreparedStatement와 같이 엮여서 쓰인다. 
  - 대부분의 PreparedStatement는 bind variable로 구현한다.
```java
String sql = "SELECT * FROM CUSTOMER WHERE LOGIN_ID = ?";
PreparedStatement st = con.prepareStatement();
st.setString(1,login_id);
.
.
.
```

- 실제 내부 프로시저는 아래와 같이 형성된다.
- 그럼 jaewon을 넣든, jinsoo를 넣든 아래에 이미 만들어진 프로시저를 재사용하게 된다.
- 첫 고객의 로그인에서만 하드파싱이 되고, 나머지 100만명의 고객은 모두 파싱된 공통 SQL을 재사용한다.
- 그에 따라서 CPU연산이 부족할 일이 없다. 모두가 행복해진다. 트래픽만 관리하면 되는 것이다.
```sql
create procedure LOGIN (login_id in varchar2)
```



## <span style="color:#802548">_디스크 I/O_</span>
- 디스크 I/O는 블록킹이라 그동안 프로세스가 일을 하지 않고 CPU를 반환한다.
- I/O가 일어나면 프로세스가 대기 큐에서 잠을 자버린다.
- I/O Call은 Single Block I/O 기준 평균 10ms, SAN 스토리지는 4-8ms, SSD 스토리지는 1ms다.
- Single Block I/O 기준 아무리 빨라도 1초에 1000블록이 최대다. 
- I/O call을 통해 데이터 블록을 가져온다고 했다.
  - 한 번에 한 블록씩 가져오는 것은 single block I/O라고 부른다. index를 읽을 때 쓴다.
    - 소량을 읽을 때는 효율적이다.
  - 한번에 여러개를 가져오는 것은 multi block I/O라고 부른다. table을 읽을 때 쓴다.
    - 대량으로 읽을 떄는 효율적이다.
    - 그 중에서도 multiple block 단위가 클수록 좋다. 
    - I/O call이 줄어 process가 대기상태에 빠지는 횟수가 줄어들기 때문이다.
      - OS단에서 최대 단위가 보통 1MB이기 떄문에 DB에서도 1MB 이상 설정할 필요가 없다.
      - 아래와 같이 multiblock 단위를 확인할 수 있다.

```sql
show parameter db_file_multiblock_read_count
```

- 그럼 아래와 같이 multiblock 기준이 나온다.
```
NAME                                TYPE          VALUE
db_file_multiblock_read_count      integer       16
```

- 프로세스가 잠드는 disk I/O call을 최대한 줄이려면 block 크기를 늘려야 한다.
- 8KB block 기준 128로 바꾸면 1MB로 최대 크기를 설정할 수 있다.
  - 8Kb x 128이면 딱 1MB다.
  - 물론 8KB가 아니라 16KB 기준으로는 64로 설정하는 게 최대일 것이다.
  - 만약 OS가 2MB라면 8KB 기준 256으로 설정해도 된다.
```sql
alter session set db_file_multiblock_read_count = 128;
```

## <span style="color:#802548">_table scan_</span>
- table scan의 경우에는 테이블 세그먼트 헤더에 저장된 익스텐트 맵을 이용한다.
- 익스텐트 맵을 통해 각 익스텐트의 첫 번쨰 데이터 블록 DBA(데이터블록주소)를 알 수 있다.
- 익스텐트는 연속된 데이터 블록 집합이다. 따라서 첫 번째 데이터 블록 뒤에 연속해서 저장된 블록을 읽으면 된다.
- 각 인전합 익스텐트끼리는 보통 서로 다른 테이블일 확률이 높다. DB가 파일 경합을 줄이기 위해 데이터를 여러 파일에 분산 저장하기 때문이다.
```
SEGMENT     |TABLESPACE_NAME     |EXTENT_ID        |FILE_ID        |BLOCK_ID       |BLOCKS
TABLE        USERS               |0                |1              |1              |4
TABLE        USERS               |1                |1              |9              |4
TABLE        USERS               |2                |2              |1              |4
TABLE        USERS               |3                |2              |5              |4
TABLE        USERS               |4                |2              |13             |4
TABLE        USERS               |5                |3              |1              |4
TABLE        USERS               |6                |4              |9              |4
TABLE        USERS               |7                |5              |5              |4
TABLE        USERS               |8                |5              |17             |4
```

- 데이터 블록이 DBMS가 데이터를 읽고 쓰는 단위다.
- 특정 레코드 하나를 읽는 건 불가능하다. 반드시 블록 단위로 읽게 된다.
- index 또한 마찬가지로 index only scan을 제외하면 데이터 블록을 읽게 된다.
- 아래는 table scan시의 예시다.
```
TABLE 세그먼트
EMP 테이블    Salary 테이블 ........ 

EMP 테이블 내 익스텐트
익스텐트1
데이터 블록1
EMPNO     ENAME     MGR     JOB         SAL     DEPTNO
7369      SMITH     7902    CLERK       800     20
7499      ALLEN     7968    SALESMAN    1600    30
7521      WARD      7698    SALESMAN    800     30
7566      JONES     7839    MANAGER     800     20
7654      MARTIN    7698    SALESMAN    800     30


데이터 블록2
EMPNO     ENAME     MGR     JOB         SAL     DEPTNO
7698      BLAKE     7839    MANAGER     800     30
7782      CLARK     7839    MANAGER     1600    10
7788      SCOTT     7566    ANALYST     800     20
7839      KING              PRESIDENT   800     10
7844      MARTIN    7698    SALESMAN    800     30

데이터 블록3
EMPNO     ENAME     MGR     JOB         SAL     DEPTNO
7876      ADAMS     7788    CLERK       1100    20
7900      JAMES     7698    CLEKR       950     30
7902      FORD      7566    ANAYLST     3000    20
7934      MILLER     7782    CLERK       1300    10

데이터 블록4
```

- 예를 들어 1byte짜리 컬럼인 JOB(varhcar2)을 읽고 싶다고 해보자.
- column 단위로는 당연히 읽을 수 없다. 그런데 문제는 record 단위로도 읽을 수 없다는 점이다.
- 무조건 block 단위로 읽어야 한다. 보통 block은 8KB다(VALUE가 8192).
- 아래를 통해 확인 가능한데 보통 8KB다. 2,4,16,32KB 중 선택가능하다.
```sql
show parameter block_size
```

- index scan을 해도 1byte짜리 column 값을 가져오려고 8KB 블록을 읽어야 하는 건 마찬가지다.
- 어떤 scan을 하든 data block을 읽어야 하는 건 동일하다.
- 다만 table scan과 달리 index scan은 rowid를 통해 다른 데이터 block을 순차로 읽지 않고 곧바로 접근할 수 있을 뿐이다.
- table은 순차적으로 모든 데이터블록을 읽는 점에서 sequential access라고 한다.
- 반면에 index는 직접 필요한 데이터 블록에 곧바로 접근할 수 있기 때문에 random(임의) access라고 한다.



## <span style="color:#802548">_버퍼캐시_</span>
- I/O call을 통해 가져온 data는 매우 값비싸다.
- 따라서 메모리에 저장하는데, 그 공간을 데이터 캐시라고 한다.
  - 서버 프로세스와 데이터파일 사이에 데이터 캐시가 있다.
  - 데이터가 메모리에 있으면 전기적 접근이 가능하여 물리적 접근(disk I/O)에 보통 10,000배 빠르다.
  - 데이터 캐시의 용량은 아래와 같이 확인 가능하다. 요새는 데이터 캐시의 용량을 늘리는 추세다.
```sql
show sga
```
- 캐시에서 블록을 찾으면 프로세스가 I/O call을 할 필요가 없다.
  - process가 멈추지 않기 때문에 속도가 그만큼 빨라진다.
  - 버퍼캐시는 공유메모리이므로, 버퍼캐시에 놓으면 해당 블록을 원래는 disk I/O로 읽어야하는 다른 process도 이득을 본다.
- 그래서 direct path I/O (배치용도)를 제외한 모든 blcok I/O는 메모리 버퍼캐시를 경유한다.
- index range scan이든 table full scan이든 메모리 버퍼캐시가 필요하다는 의미다.
- 버퍼캐시는 공유자원이라 누구나 접근할 수 있다.
- 다만 동시접근이 일어나는 경우, 순차적으로 접근하게끔 해주어야 한다.
  - 이렇게 순차로 접근하게 하는 걸 직렬화라고 한다. 줄세우는 것이다.
  - 줄 세우기 메커니즘은 래치로 구현한다.

## <span style="color:#802548">_table data block과 index leaf block_</span>
- index leaf block은 data block과 완전히 다르다.
- index leaf block은 end까지 entire read가 필요하지 않다.
  - condition을 만족하지 않는 index record의 key를 발견하는 순간 scan을 멈춘다. 
  - index record는 key-value pair다. key - value pair는 index leaf block만 가지고 있다.
  - index root, branch block은 idnex leaf block에 쉽게 접근할 수 있게끔 트리형태로 정보를 제공한다.
  - index record는 db에서 사용자에게 별도로 제공해주지 않는다. 해당 command가 없다.
- index range scan은 실제로 index key scan이다. 
  - root -> branch -> leaf로 내려가면서 key를 scan하고 컨디션을 만족하는 첫 index record를 발견한다.
    - 이렇게 starting point를 결정하는 걸 index의 수직적 탐색이라고 한다.
  - 그럼 index는 모두 정렬되어 있기 때문에 인접한 key-value pair, 즉 index record를 읽어나간다.
  - 그러다 condition을 만족하지 못하는 index record를 발견하면 scan을 멈춘다. 
    - 이렇게 end point를 정하는 과정을 수평적탐색이다.
- 그럼 index range scan을 통해 필요한 key를 모두 찾게 된다. 
  - 해당 key에서 value를 찾고, value에 있는 rowid를 찾는다.
  - rowid를 통해 다시 table의 data block을 읽으면 table record를 가져올 수 있다. 
  - 다만 table data block은 entire read가 수반된다. 이러한 과정을 table access라고 한다.
- 이러한 총 과정을 index range scan이라고 통칭한다.

- 만약 select와 order by, group by, where  조건절을 모두 포함해 전부 index안에 존재한다면, table access가 필요없다. 
  - b-tree는 실제 해당 column에 기반하여 key형태로 보관하고 있기에, 실제 value값에 어느정도 formatting만 해주는 것이다. 
  - 따라서 해당 formatting을 db에서 역으로 실행하면 select에 필요한 값을 알 수 있다. 이를 covered index, index only scan이라 한다.
  - 이 경우 table access가 없기 때문에 엄청나게 빠르다.

## <span style="color:#802548">_index scan_</span>
- index는 큰 테이블에서 소량 데이터를 검색할 때 사용한다.
- index를 이용한 tuning에서 중요한 것은 두 가지가 있다.
  - 인덱스 스캔 효율화 튜닝이다.
    - 정렬을 처음부터 잘해야한다는 의미다.
    - index 선두 컬럼을 무엇으로 설정할 지에 관한 것이다.
    - 또한 starting point와 end point를 어디로 정할지에 관한 것이다.
  - random acces 최소화다.
    - 테이블에 access하는 횟수를 최대한 줄여야 한다.
    - 사실 이게 훨씬 중요하다.
    - index에 어떤 컬럼을 포함시킬 지에 관한 것이다.
- 예를 들어 이름, 시력, 학년-반-번호로 구성된 앨범이 있다고 해보자. 여기선 이름이 선두에 있는 정렬 필드다.
- 홍길동을 찾고자 한다면 이름이 선두 정렬 필드라 쉽게 찾을 수 있다.
```
이름          |시력           |학년-반-번호
강수지        |1.5            |4학년 3반 37번
김철수        |0.5            |3학년 2반 13번
이영희        |1.2            |...
홍길동        |1.0            |2학년 6반 24번
홍길동        |1.5            |5학년 1반 16번
홍길동        |2.0            |1학년 5반 15번
....
```


- 그런데 만약에 이름이 아닌 시력이 선두라면?
- 홍길동을 찾으려면 앨범에서 이름이 '홍'인 것을 찾아가는 게 아니라 전체를 찾아가야 한다.
- 매우 비효율적인 scan이 될 것이다.
```
|시력       이름     |학년-반-번호
|0.5       김철수     |3학년 2반 13번
|1.0       홍길동     |2학년 6반 24번
|1.2       이영희     |...
|1.5       강수지     |4학년 3반 37번
|1.5       홍길동     |5학년 1반 16번
|1.6       김신후     |3학년 1반 24번
|1.7       허재원     |2학년 1반 11번
...
|2.0       홍길동     |1학년 5반 15번
...
```

- index tuning의 두번째는 random access 최소화인데, 즉 table access 횟수를 줄이는 것이다.
- 인덱스 스캔이 비효율적이어도 학생명부를 뒤지면 된다.
- 그러나 학생명부가 없다면 직접 교실을 찾아가며 방문해야 한다.
- random access는 학생명부(index leaf block)가 없어 직접 교실(table data block)을 찾아다니는 상태인 것이다.

- index 탐색에는 아래와 같은 세 종류가 있다.
  - 수직 탐색: starting point를 정하기
  - 수평 탐색: end point를 정하기
- index 탐색이 끝나면 이제 index 필터링을 시작한다.
  - range scan으로 얻은 index record(rowid) 중 어떤 것만 random access를 할 지 정한다
- 그림으로 보면 아래와 같다.
<img src="/image/index-scan-table-access.jpg" />

- 그렇게 가져온 table record 중 table filtering을 시작한다.
  - 실제 resultSet에 담을 걸 정하는 과정이다.

- index를 수직 탐색한다는 의미는 조건을 만족하는 레코드를 찾는다는 의미가 아니다.
- 조건을 만족하는 '첫' 레코드를 찾는다는 의미다. 즉 시작점을 찾는 것이다.
- 아래와 같은 sql이 있다고 해보자.
```sql
create index 고객 on 고객(성별, 고객명);

select from 고객
where 성별 = '남'
and 고객명 = '이재희'
```

- 그럼 index root block -> index branch block -> index leaf block으로 나아가면서 시작점을 찾아간다.
- 남 & 이재희가 발견된 해당 leaf block이 곧 시작점이다.
- 해당 리프 블록에서 scan을 더 수행해서 만족하는 지점이 없으면 scan을 멈춘다. 이렇게 되면 수직 탐색만 존재하는 셈이다.
- 만약 계속 만족하는 조건이 나오면 다음 리프블록도 읽어간다. 이게 수평 탐색이다.
- 수평탐색을 더 자세하게 살펴보자.
- index 수평 탐색은 index range scan의 end point를 찾아가는 과정이다.
- index를 수평탐색하는 이유는 두 가지다.
  - 조건절을 만족하는 모든 index record 찾기
    - index block leaf는 서로 양방향 연결 리스트라 좌 - 우, 우 - 좌로 모두 이동이 가능하다.
    - 따라서 내림차순으로 정렬하면 조건을 만족하는 가장 큰 값을 찾아 우측으로 수직 탐색을 하다 좌측으로 수평탐색을 한다.
    - 따라서 오름차순으로 정렬하면 조건을 만족하는 가장 큰 값을 찾아 좌측으로 수직 탐색을 하다 우측으로 수평탐색을 한다.

- index는 range를 scan하는 것이라는 말을 이제는 이해했을 것이다.
  - index range scan양이 똑같기 때문이다. block I/O call할 양이 똑같다는 의미다. 성능차이가 없다는 의미다.
  - index column이 모두 조건문에 있고, 해당 조건문이 모두 =인 경우에는 인덱스 구성의 순서를 바꾼다고 해도 성능 변화가 없다.
  - 똑같이 index column의 구성 요소면서 모두 등치 조건이라면 where조건문에서의 순서는 index scan에 아무런 영향이 없다. 
  - 따라서 인덱스가 (고객명, 성별)이든 (성별, 고객명)이든 등치조건이라면 일하는 양은 다르지 않다.
- 구체적 수치로 index scan의 효율성을 따지는 방법은 sql trace를 보면 된다.
  - 아래와 같은 trace에서 읽은 블록은 7463개지만, 얻은 record는 10개밖에 안된다.
  - 한 블록당 평균 500개가 담긴가도 하면, 5463 x 500 총 3,731,500개 record를 읽어 10개 record를 얻은 것이다.
  - 엄청나게 비효율적이라고 볼 수 있다.
```
Rows      Row Source Operation
10        TABLES ACCESS BY INDEX ROWID BIG_TABLE(cr=7471 pr=1466 pw=0 time=22137)
10          INDEX RANGE SCAN BIG_TABLE_IDX (cr=7463 pr=1466 pw=0 time=22328)
```

## <span style="color:#802548">_index scan의 종류_</span>
1. index range scan
- 선두 컬럼을 가공하지 않은 상태로 조건절에 써야한다.
- index range scan이라고 무조건 효율적 scan을 의미하지 않는다.
- index scan 범위를 줄이고, table access도 줄여야 한다.
```sql
SELECT * FROM emp where deptno = 20;
```

2. index full scan
- index로 갖고 있는 column이 하나라도 있기 때문에 table full scan이 아닌 index full scan이 가능하다.
- 물론 cardinality가 높아야, 즉 선택률이 낮아야 이렇게 index full scan 활용이 의미가 있다.
```sql
create index emp_ename_sal_idx on emp (ename, sal);

SELECT * FROM emp
WHERE sal > 2000
order by ename;
```

- index full scan이나 index range scan 모두 order by 연산이 생략될 수도 있다.
- index column 순으로 정렬되기 때문이다. 물론 아래와 같은 경우는 order by가 있어야 할 것이다.
  - (ename, sal)로 만들었기 때문이다. (sal, ename)으로 만들었고 index scan이 이뤄지면 order by를 안써도 될 수도 있다.
  - 다만 oracle 12c부터는 order by를 반드시 명시해주는 게 좋다. batch I/O가 발동하면 index 정렬 순서대로 늘 record가 뽑히진 않기 때문이다.
```sql
SELECT /*+ first_rows */
from emp
where sal > 1000
order by ename;
```

3. index unique scan
  - unique index를 = 조건으로 탐색하는 경우, 수직탐색만 한다.
  - 다만 결합 index의 경우, 모든 조건을 전부 =으로 써야 index unique scan이 된다.
  - (주문일자, 고객ID, 상품ID) PK에서 where 조건문에 주문일자와 고객ID만 있다면 그것은 index range scan이 된다.
```sql
create unique index pk_emp on emp(empno);

select empno, ename from emp where empno = 7788;
```

4. index skip scan
  - 인덱스 선두 컬럼이 조건절에 없어도 index를 활용하는 방식 중 하나다.
  - index full scan과 달리 조건에 맞지 않는 경우 index scan을 skip하여 효율적으로 scan한다.
  - cardinality가 극단적으로 높고 낮은 조합의 index에 보통 이용된다.
  - 성별과 PK의 조합이 대표적이다. 선두컬럼은 중복값이 많고, 후행은 중복값이 적을 때 유용하다.
  - 인덱스 선두가 있고 중간 컬럼이 없는 조건절의 경우에도 활용할 수 있다.
  - 해당 scan은 조건절에 맞는 index record를 포함할 거 '같은' index leaf block을 scan하는 방식이다.
  - index skip scan의 간단한 예시를 보여주면 아래와 같다.
```sql
SELECT * FROM 사원 WHERE 연봉 between 2000 and 4000;
```

- 1번, 3번, 6번, 7번, 10번 블록을 읽는다.
- 핵심은 1번 블록과 6번 블록, 10번 블록이다.
- optimizer는 성별이 남과 여만 있다는 사실을 모르기 때문에 남자가 연봉이 800이하여도 1번 블록을 access한다. 
  - optimizer 기준에서는 '남'보다 작은 성별이 존재할 수 있기 때문이다.
- 6번 블록을 access하는 이유는 여성 중 연봉이 2000이상 3000이하인 index record가 저장되기 때문이다.
  - 그 외에도 optimizer 기준에서는 남 과 여만 있는 걸 모르기 때문에 남 과 여 사이의 다른 성별이 6번 블록에 저장되므로 access한다.
- 10번 블록을 access하는 이유는 '여성'보다 큰 성별이 존재할 수 있기 때문이다.
<img src="/image/index-skip-scan.jpg" />

- 중간 컬럼이 비는 경우의 예시를 살펴보자.
- PK는 업종유형코드 + 업종코드 + 기준일자로 구성되어있다고 해보자.
- 아래는 업종코드라는 중간 컬럼이 비어있다. 이럴 때 index skip scan을 쓸 수 있다.
- 중간컬럼이 비기 때문에 index range scan 기준으로는 기준일자는 index filter 조건이 된다.
- 따라서 업종유형코드가 01인 모든 record를 읽어야만 한다.
- 그러나 index skip scan을 하게 되면 기준일자가 아래에 해당할 거 '같은' index leaf block만 골라서 access한다.
```sql
SELECT /*+ INDEX_SS(A 일병업종별거래_PK) */
      기준일자, 업종코드, 체결건수, 체결수량, 거래대금
FROM 일병업종별거래 A
WHERE 업종유형코드 = '01'
AND 기준일자 BETWEEN '20080501' AND '20080531'
```

- 선두와 중간 모두 비어있고 마지막 컬럼만 있어도 쓸 수 있다.
```sql
SELECT /*+ INDEX_SS(A 일병업종별거래_PK) */
      기준일자, 업종코드, 체결건수, 체결수량, 거래대금
FROM 일병업종별거래 A
AND 기준일자 BETWEEN '20080501' AND '20080531'
```

- 선두 컬럼이 부등호, BETWEEN, LIKE같은 범위 검색 조건일 때도 index skip scan이 가능하다.
- 기준일자 + 업종유형코드로 구성된 index가 있다고 해보자.
- 만약 index range scan을 하게 되면 기준일자 BETWEEN조건을 만족하는 모든 index 구간을 scan해야 한다.
- 하지만 index skip scan을 하게 되면 기준일자 BETWEEN 구간을 만족하는 구간 중 업종유형코드가 01인 레코드를 포함할 리프 블록만 골라 scan한다.
```sql
SELECT /*+ INDEX_SS(A 일병업종별거래_PK) */
      기준일자, 업종코드, 체결건수, 체결수량, 거래대금
FROM 일병업종별거래 A
AND 기준일자 BETWEEN '20080501' AND '20080531'
AND 업종유형코드 = '01'
```

1. index fast full scan
  - 논리순서에 따른 scan이 아니라 물리 순서에 따른 scan을 실행한다.
  - 따라서 한꺼번에 많이 읽는 게 중요하며, multiblock I/O 방식으로 진행된다.
  - 다만 논리순서가 아니기 때문에 제대로 정렬되지 않을 가능성이 높다.
- 아래가 논리적 순서로 배치한 블록이다.
- 루트 ->브랜치 1 ->1->2->3->4->5->6->7->8->9->10번 순으로 블록을 읽는다.
```
                      루트
          1번브랜치           2번브랜치
    1번 2번 3번 4번 5번   6번 7번 8번 9번 10번리프
```

- 물리순서에 따르면 아래와 같다.
- 1->2->10->3->9->8->7->4->5->6 순으로 읽는다.
- index full scan은 leaf block만 필요하기 때문에 root와 branch는 읽어도 버린다.
```
1번 익스텐트                |     2번 익스텐트
              루트          |       루트
          1번브랜치         |      2번 브랜치
    1번 2번 10번 3번 9번    |   8번 7번 4번 5번 6번
```

## <span style="color:#802548">_index range scan- LIKE 대신 BETWEEN_</span>
- LIKE보단 BETWEEN이 더 나은 경우가 많다.
- 아래와 같이 두 sql이 있다고 해보자.
  - 인덱스가 (판매월,판매구분)으로 구성되었다.
  - 판매구분은 A와 B가 있고 A가 90%, B가 10%다.
    - BETWEEN 201901 and 201912라면 201912에서 A를 만나게 되면 멈춘다.
    - LIKE는 그게 안된다. 201913이 존재할 수도 있기 때문에 2020이 나올때까지 전부 읽는다. 
    - scan량을 줄이는 데 도움이 안되는 것이다.
```sql
SELECT *
FROM 월별고객별판매집계
WHERE 판매월 LIKE '2019%'
AND 판매구분 = 'B'
```

```sql
SELECT *
FROM 월별고객별판매집계
WHERE 판매월 BETWEEN '201901' and '201912'
AND 판매구분 = 'B'
```


<img src="/image/between-like-difference.jpg" />

- LIKE가 편하다고 남용하면 sql 성능에서 댓가를 치른다.
- 특히 LIKE를 index column에 쓰면 안된다.
- 아래와 같은 쿼리가 있는데, 인덱스가 (회사코드,지역코드,상품명)으로 구성됐다고 해보자.
- 그 경우 아래와 같이 두 개로 나눠서 처리하면 적어도 지역코드가 존재할 떈 range scan량을 줄일 수 있다.
```sql
SELECT * 고객ID, 상품명, 지역코드, ....
FROM 가입상품
WHERE 회사코드 = :com
AND 지역코드 = :reg
AND 상품명 LIKE :prod || '%'
```

```sql
SELECT 고객ID, 상품명, 지역코드, ...
FROM 가입상품
WHERE 회사코드 = :com
AND 상품명 LIKE :prod || '%'
```

<img src="/image/efficient-like.jpg" />

- 그런데 sql을 하나로 쓰려고 통합했다고 해보자.
- 그럼 지역코드를 입력해도 입력하지 않은 경우와 range scan양이 크게 다르지 않게 된다.
- access조건이 filter조건으로 변해버렸기 때문에 그만큼 scan양이 많아진 것이다.
```sql
SELECT 고객ID, 상품명, 지역코드, ...
FROM 가입상품
WHERE 회사코드 = :com
AND 지역코드 LIKE :reg || '%'
AND 상품명 LIKE :prod || '%'
```

<img src="/image/inefficient-like.jpg" />

- BETWEEN으로 처리하는 것도 LIKE처럼 처리하는 효과를 낼 떄도 있다. 조심해야 한다.
- 종목코드를 입력하면 종목1과 종목2가 모두 같은 종목코드로 처리되어 = 조건과 동일해진다.
- 종목코드를 입력하지 않으면 왼쪽은 빈칸6개에서 오른쪽은 ZZZZZZ로 처리되어 모든 걸 검색하는 조건과 동일해진다.
- 즉 BETWEEN을 사실상 LIKE처럼 쓴 셈이다. 
```sql
SELECT 거래일자, 종목코드, 투자자유형코드
FROM 일별종목거래
WHERE 거래일자 BETWEEN :시작일자 AND :종료일자               /**필수조건 */
AND 종목코드 BETWEEN :종목1 AND :종목2                      /** 옵션 조건 */
AND 투자자유형코드 BETWEEN :투자자유형 AND :투자자유형2       /** 옵션 조건 */
AND 주문매체구분코드 BETWEEN :주문매체구분1 AND :주문매체구분2 /** 옵션 조건 */
```


## <span style="color:#802548">_index range scan- BETWEEN vs IN_</span>
- in 조건을 활용하면 index 구성을 바꾸지 않고도 index range scan을 효율적으로 진행할 수 있다.
  - 만약 아래가 인터넷매물  betwenn 1 and 3이었다면, 1부터 3까지 전부 index record를 scan했을 것이다.
  - 그럼 그 아래 조건 아파트시세코드, 평형, 평형타입은 모두 index filter조건으로 기능하게 된다. 비효율적 scan이 일어나게 되는 것이다.
  - 대신 아래와 같이 쓰면 수직적탐색을 3번 거치지만, 수평탐색이 전혀 없게 된다.
  - 이른바 IN-LIST 변환이다.

<img src="/image/in-list-vertical-scan.jpg" />


```sql
SELECT 해당층, 평당가, 입력일, 해당동, 매물구분, 연사용일수, 중개업소코드
FROM 매물아파트매매
WHERE 인터넷매물 in ('1','2','3')
AND 아파트시세코드 = 'A01011350900056'
AND 평형 = '59'
AND 평형타입 = 'A'
ORDER BY 입력일 DESC
```

- 위의 sql은 실제로 아래와 같이 변환할 수 있다.
```sql
SELECT 해당층, 평당가, 입력일, 해당동, 매물구분, 연사용일수, 중개업소코드
FROM 매물아파트매매
WHERE 인터넷매물 = 1
AND 아파트시세코드 = 'A01011350900056'
AND 평형 = '59'
AND 평형타입 = 'A'
ORDER BY 입력일 DESC

UNION ALL

SELECT 해당층, 평당가, 입력일, 해당동, 매물구분, 연사용일수, 중개업소코드
FROM 매물아파트매매
WHERE 인터넷매물 = 2
AND 아파트시세코드 = 'A01011350900056'
AND 평형 = '59'
AND 평형타입 = 'A'
ORDER BY 입력일 DESC

UNION ALL 

SELECT 해당층, 평당가, 입력일, 해당동, 매물구분, 연사용일수, 중개업소코드
FROM 매물아파트매매
WHERE 인터넷매물 = 3
AND 아파트시세코드 = 'A01011350900056'
AND 평형 = '59'
AND 평형타입 = 'A'
ORDER BY 입력일 DESC
```

- 다만 문제는 in에 들어갈 수가 많다면 between 보다 더 비효율적일 수 있다.
  - 수직적 탐색이 많아지는데, 그렇게 되면 branch block을 그만큼 반복 탐색하는 횟수가 늘어난다.
  - 그게 수평 탐색이 많아지는 것보다 더 비효율을 초래할 수도 있다. 특히 depth가 깊다면 말이다.
  - 수평 탐색이 실제로 별로 크지 않았다면 in으로 변환해도 큰 효과를 누리지 못할 수 있다.
  - in으로 바꾼다는 것은 결국 수직탐색을 늘리고, 수평탐색을 줄인다는 의미기 때문이다.
- 실제로 아래와 같은 경우, 왼쪽은 in-list로 바꾸는 게 좋지만, 오른쪽은 그닥 효과가 없을 것이다.

<img src="/image/inefficient-in-list.jpg" />

- 그럴 떄 만약 코드를 table로 따로 관리한다면, join이 괜찮은 대체재가 될 수 있다.
```sql
SELECT b.해당층, b.평당가, b.입력일, b.해당동, b.매물구분, b.연사용일수, b.중개업소코드
FROM 통합코드 a, 매물아파트매매 b
WHERE a.코드구분 = 'CD064'
AND a.코드 BETWEEN '1' and '3'
AND b.인터넷매물 = a.코드
AND b.아파트시세코드 = 'A01011350900056'
AND b.평형 = '59'
AND b.평형타입 = 'A'
ORDER BY b.입력일 DESC
```

- 또는 index skip scan을 사용하는 것도 좋은 선택이다.
- 선두컬럼이 betwenn으로 범위검색을 하는 경우 특히 in-list로 바꾸는 것보다 유용하다.
```sql
SELECT /*+ INDEX_SS(t 월별고객별판매집계_IDX2) */ count(*)
FROM 월별고객별판매집계 t
WHERE 판매구분 = 'A'
AND 판매월 BETWEEN '201801' AND '201812'
```

- in 조건이 = 조건으로(index access) 작동하려면 in-list 방식으로 풀려야한다.
  - union all을 썼을 때와 동일하게 변환될 수 있어야한다는 의미다.
  - 즉 BETWENN으로 풀리면 안된다는 의미다.
- 만약 그렇지 않다면 in 조건은 index filter 조건으로 작동한다.
  - 그러나 in 조건이 index access 조건으로 풀려야만 좋은 것은 아니다. 
  - 때로는 filter 조건으로 가는 게 훨씬 나은 판단일 수도 있다.
- in 조건절을 access 혹은 filter로 바꾸는 걸 자의로 조절하려면 hint를 사용하면 된다.
- 아래 방식은 인덱스 선두컬럼만 access 조건으로 사용한다는 의미다.
```sql
SELECT /*+ num_index_keys(a 고객별가입상품_X1 1) */ *
FROM 고객별가입상품 a
WHERE 고객번호 = :cust_no
AND 상품ID in ('NH00037', 'NH00041','NH00050')
```

- 아니면 상품ID를 가공해버려서 access조건으로 쓰지 못하게 할 수도 있다.
- 그러면 index filter조건으로만 활용이 가능하다.
```sql
SELECT * 
FROM 고객별가입상품
WHERE 고객번호 =:cust_no
AND RTRIM(상품ID) in ('NH00037', 'NH00041','NH00050')

SELECT *
FROM 고객별가입상품
WHERE 고객번호 = :cust_no
AND 상품ID || '' in ('NH00037', 'NH00041','NH00050')
```

- 상품ID를 access 조건으로 활용하려면 아래와 같이 2번째까지 index access 조건으로 사용한다고 하면 된다.
- 그럼 in-list 방식으로 풀려 index access 조건으로 활용할 수 있다.
```sql
SELECT /*+ num_index_keys(a 고객별가입상품_X1 2) */ *
FROM 고객별가입상품 a
WHERE 고객번호 = :cust_no
AND 상품ID in ('NH00037', 'NH00041','NH00050')
```

## <span style="color:#802548">_index를 비효율적으로 타는 경우 1 -OR_</span>
- or 조건의 경우, 아래와 같은 조건이 아니면 쓰지 않는 게 효율적이다.
  - 인덱스 액세스 조건으로 사용 불가
  - 인덱스 필터 조건으로도 사용 불가
  - 테이블 필터 조건으로만 사용 가능
  - 다만 or expansion으로 활용할 수 있다면 상관없다.
    - 예를 들어, (고객ID, 타입, 거래일자)를 index로 묶어보자.
    - 고객ID라는 선두컬럼만 범위조건이 아니면 or expansion이 발동할 수 있다.
    - 아래와 같은 쿼리는 or expansion이 가능하다.
```sql
SELECT *
FROM 거래
WHERE 고객 ID = :cust_id
AND (
  (:dt_type = 'A' AND 거래일자 BETWEEN :dt1 and :dt2)
  or
  (:dt_type = 'B' AND 결제일자 BETWEEN :dt1 and :dt2)
)
```

- 확실하게 하려면 아래와 같이 hint를 주면 된다.
- or expansion은 IN-LIST처럼 UNION ALL로 바뀌는 것을 의미한다.
```sql
SELECT /*+ OR_EXPAND */ * 
FROM 거래
WHERE 고객 ID = :cust_id
AND (
  (:dt_type = 'A' AND 거래일자 BETWEEN :dt1 and :dt2)
  or
  (:dt_type = 'B' AND 결제일자 BETWEEN :dt1 and :dt2)
)
```

- 반면 아래와 같이는 or expansion이 발동하지 않는다.
- 인덱스 선두컬럼이 or조건으로 갔기 때문이다.
- 다시 말해 인덱스 선두컬럼은 OR조건에 쓰이면 안 된다.
```sql
SELECT *
FROM 거래
WHERE (고객ID is null or 고객ID = :cust_id)
AND 거래일자 BETWEEN :dt1 and :dt2
```

## <span style="color:#802548">_index를 비효율적으로 타는 경우 2 -LIKE/BETWEEN_</span>
- LIKE - BETWEEN을 쓸 떄는 아래 같은 조건에 활용하지 않는게 좋다.
```
인덱스 선두 컬럼
NULL 허용 컬럼
숫자형 컬럼
가변 길이 컬럼
```

- index 선두컬럼에 LIKE/BETWEEN을 쓰면 그 아래 조건들은 전부 index filter로 처리된다는 걸 알것이다.
- 이번에는 그런문제보다 더 심각하다. 인덱스 선두컬럼이 입력되지 않는 경우는 range를 모르므로 index full scan이 일어난다.
```sql
SELECT *
FROM 거래
WHERE 고객ID LIKE :cust_id || '%'
AND 거래일자 BETWEEN :dt1 AND :dt2
```


- NULL 허용컬럼에 LIKE를 허용하는 것은 더 문제다.
  - 성능이 문제가 아니라 결과 집합에 오류가 생긴다.
  - LIKE/BETWEEN NULL같은 건 없다. IS NULL로만 NULL 값을 가져올 수 있기 때문이다.
```sql
SELECT *
FROM 거래
WHERE 고객ID LIKE '%'
AND 거래일자 BETWEEN :dt1 AND :dt2
```

- 숫자형 컬럼에도 LIKE는 안된다.
  - LIKE는 문자열 연산자다. 숫자형 컬럼이 묵시적 형변환이 일어나게 된다.
  - 그럼 숫자형 컬럼은 index access가 아닌 index filter조건으로 변환된다.
  - 의도치 않게 index scan양이 늘어날 수 있다는 의미다.
  - 묵시적 형변환으로 인해 (고객ID, 거래일자) index는 아예 무력화된다는 것도 문제다.
```sql
/**고객ID는 int */
SELECT * FROM 거래
WHERE 거래일자 = :trd_dt
AND 고객ID/**실제론 TO_CHAR(고객ID)로 변환됨 */ LIKE :cust_id || '%'
```

- 옵션 조건은 union all을 통해 효율적인 index scan을 할 수도 있다.
  - 단, 이경우 (cust_id, 거래일자)와 (거래일자)라는 index가 모두 존재해야 한다.
  - 그럼 cust_id가 있든 없든 index range scan이 일어난다.
  - 특히 NULL 허용 컬럼이여도 쓸 수 있다는 점에서 매우 좋은 해결방안이다.
```sql
SELECT *
FROM :cust_id is null
AND 거래일자 BETWEEN :dt1 and :dt2

UNION ALL

SELECT *
FROM 거래
WHERE :cus_id is not null
AND 고객ID = :cust_id
AND 거래일자 BETWEEN :dt1 and :dt2
```

- WAS의 경우 옵션조건을 대부분 dynamic sql로 처리한다.
- 좋은 방법이지만, hint를 사용하는 데 있어 신중할 필요가 있다.
- 또한 하드파싱 문제1가 없게 바인드 변수를 잘 사용해야 한다.


## <span style="color:#802548">_index를 비효율적으로 타는 경우 3 -가공된 index column_</span>
- 그러나 가장 중요한 것은 인덱스의 선두 컬럼이 조건절에 쓰이는 경우, 조건절의 가장 처음에 가공하지 않은 채로 나와야 한다는 것이다. 
- 인덱스가 (소속팀, 사원명, 연령) 순으로 구성했다고 해보자.
- 이는 아래와 같은 의미다.
```
데이터를 소속팀 순으로 정렬하고, 소속팀이 같으면 사원명 순으로 정렬하고, 사원명까지 같으면 연령 순으로 정렬한다.
```

- 다시말해 이름이 같더라도 소속팀이 다르면 서로 멀리 떨어지게 된다는 의미다.
- 소속팀 정렬이 이뤄지지 않았다면 말이다. 아래 sql은 index range scan의 역할을 못하게 된다는 의미다.
- 그럼 index full scan을 수행하게 된다.
```sql
SELECT 사원번호, 소속팀, 연령, 입사일자, 전화번호
from 사원
where 사원명 = '홍길동'
```

- index의 두번째 구성요소는 가공하더라도 index range scan을 탈 수 있다.
- 인덱스가 (기준연도, 과세구분코드, 보고회차, 실명확인번호)라고 해보자.
- 그럼 아래는 index range scan을 수행할 수 있다.
- 기준연도라는 최선두 컬럼이 조건절에 첫번째로 가공되지 않은 채 쓰였기 때문이다.
```sql
SELECT * FROM TXA1234
WHERE 기준연도 = :stdr_year
and substr(과세구분코드,1,4) = :txtn_dcd
and 보고회차 = :rpt_tmrd
and 실명확인번호 = :rnm_cnfm_no
```

- 그러나 위의 sql은 성능이 좋기는 어려울 것이다.
- 과세구분코드부터는 조건절로서의 의미를 잃어버렸기 때문이다.
- 그럼 그 뒤 보고회차, 실명확인번호는 모두 index range scan에 아무런 역할을 하지 못한다.
- 아래의 sql도 마찬가지다. index가 (주문일자, 상품번호)라고 했을 때 주문일자만이 range를 좁히는 데 역할을 한다.
- 상품번호는 range를 좁혀 scan량을 줄이는 데 아무런 도움이 되지 못한다.
```sql
select *
from 주문상품
where 주문일자 = :ord_dt
and 상품번호 like '%PING%'
```


## <span style="color:#802548">_index를 비효율적으로 타는 경우 4 -묵시적 형변환_</span>
- 묵시적 형변환도 조심해야 한다.
- 생년월일이 char라면, 아래와 같은 쿼리는 인덱스를 가공하게 된다.
```sql
SELECT * FROM 고객
WHERE 생년월일 = 19821225
```

- 그래서 실제로는 아래와 같은 쿼리를 수행하게 된다.
- 숫자형과 문자형이 같다고 하면, 숫자형이 더 세서 문자가 숫자로 형변환된다.
```sql
SELECT * FROM 고객
WHERE TO_NUMBER(생년월일) = 19821225
```

- 날짜형과 문자형이 만나면 날짜형이 이긴다.
- 따라서 아래와 같은 쿼리는 index 문제는 없다.
```sql
SELECT * FROM 고객
WHERE 가입일자 = '01-JAN-2018'
```

- 다만 날짜 포맷을 정확히 지정하는 편이 훨씬 좋다.
- NLS_DATE_FORMAT 파라미터가 다른 환경에서는 결과집합이 다를 수 있기 때문이다.
- 따라서 아래와 같이 date type으로 바꿔주자.
```sql
SELECT * FROM 고객
WHERE 가입일자 = TO_DATE('01-JAN-2018', 'DD-MON-YYYY')
```

- 숫자형과 문자형이 만나서 이기는 경우는, LIKE 연산자가 쓰였을 때다.
- LIKE는 그 자체로 문자열 비교 연산자이기 때문이다.
```sql
SELECT * FROM 고객
WHERE 고객번호 LIKE '9410%'
```

- 만약 고객번호가 int 형이면 아래와 같이 변환된다.
```sql
SELECT * FROM 고객
WHERE TO_CHAR(고객번호) LIKE '9410%'
```

- LIKE 패턴과 관련된 중요한 안티 패턴 중 하나가 옵션 조건을 걸 떄 자주 일어난다.
- 계좌번호가 입력되지 않으면 NULL값이 입력되어 모든 계좌번호가 조회된다.
- 즉 계좌번호는 optional 조건이다.
```sql
SELECT * FROM 거래
WHERE 계좌번호 LIKE :acnt_no || '%'
AND 거래일자 BETWEEN :trd_dt1 and :trd_dt2
```

- 위와 같이 사용하게 되면, 계좌번호 컬럼이 숫자형일 때, 자동 형변환이 발생하기 때문에 계좌번호가 아예 index access 조건으로 사용될 수가 없다.
- 그렇다고 (계좌번호, 거래일자)가 아니라 (거래일자, 계좌번호)로 바꾸면 full scan은 아니지만, range scan량이 압도적으로 늘어나게 된다.
  - 거래일자 조회 범위에 속한 거래 데이터를 먼저 모두 읽으면서 계좌번호를 필터링하기 때문이다.
- 저 경우에는 dynamic sql을 WAS에서 사용해주는 게 좋다.


## <span style="color:#802548">_index 구성 보기_</span>
- oracle에서 comment 없이 index 구성만 보는 법은 아래와 같다.
- column_position이 1번이면 leading column, 2번이면 그 뒤에 붙는 컬럼이라고 보면 된다.
```sql
SELECT a.table_name 
     , a.index_name 
     , a.column_name 
     , a.column_position 
  FROM all_ind_columns a 
 WHERE a.table_name = '테이블 이름' 
 ORDER BY a.index_name
        , a.column_position
```


## <span style="color:#802548">_index design_</span>
- index를 설계할 때는 위의 starting point, end point, random access, table filter를 알아야 한다.
- 이를 기준으로 조건절에 쓰이는 column들을 구분하면 아래와 같다.
  - index access
    - index 구성 column이어야 한다.
    - index range scan을 정한다
    - starting point와 end point를 지정한다
    - = 조건으로 계속되면 전부 index access조건이다.
  - index filter
    - index 구성 column이어야 한다.
    - = 조건이 끝나고 난 다음 column부터는 range scan에 영향이 없다.
    - 대신 range scan이 끝나고 나온 index record 중 어떤 record가 random access할 지 결정하는 filter역할을 한다.
    - index filter는 random access를 줄여준다는 점에서 매우 중요한 역할을 한다.
  - table filter
    - index 구성 column이 아니어야 한다.
    - index filter까지해서 random access를 수행한 뒤, 실제 table data block까지 읽어온 resultSet에서 filtering한다.
- 위의 3가지 조건을 생각하면서 어떻게 index를 구성할 지 생각해야 한다.
- 다시말하면 아래와 같다.
- where 조건문에 쓰는 조건은 index access 조건과 index filter 조건, table filter 조건으로 나뉜다.
  - index access는 수직탐색(시작점)과 수평탐색(끝점)을 결정하는 용도의 조건이다.
  - index filter는 table에 실제로 access를 할지 결정하는 용도의 조건이다.
  - table filter는 index가 아예 없는 column의 조건문인 경우다. table record를 얻어온 뒤 filtering하는 조건이다.
  - index로 사용된 컬럼 중 범위처리된 조건까지만이 index access 조건이며, 그 뒤부터는 index filter조건이다. 
- index access 조건으로 index key scan을 할 범위를 정한다.
  - 그 중에서 어떤 index record의 rowid를 가지고 table access할 지는 index filter조건이 정한다. 
    - 그렇게 가져온 table record를 다시 table filter조건으로 걸러서 resultSet을 만드는 것이다.

- 아래와 같은 조건절들이 있다고 해보자.
```sql
WHERE 고객등급 = :v1
AND   고객번호 = :v2
AND   거래일자 >= :v3

WHERE 고객등급 = :v1
AND 고객번호  = :v2
AND 거래일자 >= :v3
AND 거래유형 = :v4

WHERE 고객등급 = :v1
AND 고객번호 = :v2
AND 거래일자 >= :v3
AND 상품번호 = :v5

WHERE 고객등급 = :v1
AND 고객번호 = :v2
AND 거래일자 >= :v3
AND 거래유형 = :v4
AND 상품번호 = :v5
```

- 위에서 고객등급, 고객번호, 거래일자는 필수 조건이다. 거래유형과 상품번호는 옵션조건이다.
  - 고객등급, 고객번호는 어떤 컬럼이 앞에 오든 index range scan에 영향이 없다. 둘 모두 = 조건이기 때문이다.
  - 거래유형과 상품번호도 어떤 컬럼이 앞에 오든 index range scan에 영향이 없다.
    - index scan범위가 고객등급, 고객번호, 거래일자에 의해 결정되기 때문이다. 
- 따라서 고객등급, 고객번호, 거래일자는 index를 만들 떄 반드시 포함해줘야 한다.
- 옵션 조건은 아래와 같이 두 개로 만들어 둘 수 있다.
```
X01: 고객등급, 고객번호, 거래일자, 거래유형
X02: 고객등급, 고객번호, 거래일자, 상품번호
```

- 하지만 위를 보면 사실 거래일자에서 index range scan이 끝난다.
- 따라서 두 index를 나눌 필요가 없다. 거래유형, 상품번호는 index filter조건이기 때문이다.
- 따라서 두 index를 하나로 합쳐준다.
```
X01: 고객등급, 고객번호, 거래일자, 거래유형, 상품번호
```

- 고객번호가 PK라서 아래와 같이 바꿔야 한다고 생각할 수도 있다.
```
X01: 고객번호, 거래일자, 고객등급, 거래유형
X02: 고객번호, 거래일자, 고객등급, 상품번호
```
- 하지만 = 조건으로 걸리는 경우, 어느것이 선두컬럼이든 range scan의 양이 줄어들진 않는다.
- 따라서 어떻게 만들어도 상관없다.
- 오히려 고객번호가 필수인데 고객등급이 범위검색일 떄는 고객등급이 먼저 오는 게 좋다.
  - 그래야 index skip scan을 하는 데 있어 유리하다. 고객등급이 2인 경우, 1인 index record는 전부 skip해야 skip 성능이 극대화되기 때문이다.
  - 선두컬럼에서 skip scan을 해줌으로써 scan 량이 확 줄어들 수 있는 것이다.


## <span style="color:#802548">_index design example 1 -index 마지막에 column 더하기_</span>
- 그런데 현실적으로 운영 index를 새로 짜는 건 쉽지 않은 작업이다.
- 관리의 문제 및 DML 성능 저하의 문제가 있다.
- 그래서 기존 index column의 끝에 새로운 index field를 더하는 형태로 진행된다.
- 아래와 같은 sql이 있다고 해보자.
```sql
select /*+ index(emp emp_x01) */
from emp
where deptno = 30
and sal >= 2000
```

- 현재 deptno + job으로 index가 구성되었다고 해보자.
- 해당 sql을 만족하는 data를 찾기위해서 index는 6개를 모두 scan한다.
- 그리고 table에도 6번 모두 access할 수밖에없다. scan은 6번은 아니지만, 6번의 access가 필요하다.
- sal이 2000 이상인 record가 더 있는 지 알수가 없기 때문이다.



<img src="/image/not-added-column-index.jpg" />


- deptno + job 대신 deptno + sal로 index를 바꾸고 싶지만 아래와 같이 index를 사용하는 sql이 있다.
```sql
SELECT *
FROM emp
where deptno = 30
and job = 'CLERK'
```

- 따라서 차선책으로 deptno + job + 'sal'로 sal을 맨 끝에다가 추가해준다.
- 물론 index range scan량이 줄어들지는 않는다.
- 하지만 index에 sal이 포함되어 정렬되기 때문에, table access양이 6번에서 1번으로 줄어든다.
- random access양이 줄어드는 획기적인 개선이다.


<img src="/image/column-added-index.jpg" />


- 실제 개선사례를 살펴보도록 하자.
- 아래 sql에서 서비스 번호만을 index로 사용하고 있었다.
- like지만 전방일치기 때문에 range scan이 가능하다.
- 시작점을 한정지을 수 있기 때문이다.
```sql
select 렌탈관리번호, 고객명, 서비스관리번호, 서비스번호, 예약접수일시
        ,방문국가코드1, 방문국가코드2, 방문국가코드3, 로밍승인번호, 자동로밍여부
from 로밍렌탈
where 서비스번호 like '010%'
and 사용여부 = 'Y'
```

- 아래는 query 실행결과를 report로 만든것이다.
- index를 scan해서 얻은 건수(rows)는 266476인데, 그 건수만큼 table을 random access했다.
- table access 쪽에 보면 cr=266968인데 index scan시 읽은 block 1011개를 뺴면 265,957개 만큼 읽었다.
- table access가 266,476인데 블록 I/O가 265,957이면 CF 계수가 매우 낮은 것이다. 데이터가 워낙 많아서 서비스 번호를 만족하는 데이터가 뿔뿔이 흩어진 것이다.
- 거기다 사용여부 조건 체크 이후 남은 rows는 겨우 1909개다. 테이블 access 이후 사용여부 = 'Y' 조건을 필터링하면서 대부분 걸러졌다. 


<img src='/image/roaming-not-added-useyn.jpg' />

- 이렇게 비효율적으로 scan과 I/O를 하지 않게 사용여부를 index에 포함시켜보면 획기적으로 성능이 개선된다.

<img src='/image/roamding-added-useyn.jpg' />


## <span style="color:#802548">_index design example 2 -수행빈도 고려하기_</span>
- 인덱스 설계에는 여러 가지 고려 사항이 있다.
  - 인덱스 설계의 가장 큰 조건은 조건절에 항상 사용하거나, 자주 사용하는 컬럼을 선정하는 것이다.
    - 특히 = 조건으로 자주 조회하는 컬럼은 앞으로 가야한다.
  - 그 외에는 수행빈도도 중요하다.
    - 자주 쓰이지 않는 SQL은 조금 비효율적이어도 괜찮다.
    - 하지만 자주 쓰이는 SQL은 효율적으로 수행되게 짜야 한다.
    - 특히 join 시 driving table은 알고리즘 상 비효율 scan이 한번에 그친다.
    - 그러나 driven table은 record마다 일어나기에, 매우 효율적인 index로 구성해야 한다.
    - driven table은 되도록이면 조건절 column을 index에 넣어 최소한 index filter조건으로는 사용되게 하여 table access를 줄여야 한다.

- 이제는 실질적인 index 설계 팁을 살펴보자.
  - 가장 큰 팁은 10개의 sql이라면, 그 중 액세스 경로 1-2개만 최적 인덱스를 설계하는 것이다.
  - 나머지는 조금 비효율적이어도 목표한 성능만 나오게 설계하는 것이다.

- 예를 들어, 취급부서-취급지점,취급자,입력자,대리점설계사,대리점지사와 청약일자, 보험개시일자, 보험종료일자, 데이터 생성일시를 모두 짝지으면 24개나 된다.
- 그것보단 4개의 index로 줄여준다. 그 이유는 24개의 index는 insert, update 같은 DML에서 성능을 떨어뜨리기 때문이다.
```
X01: 청약일자, 취급부서, 취급지점, 취급자, 입력자, 대리점설계사, 대리점지사
X02: 보험개시일자, 취급부서, 취급지점, 취급자, 입력자, 대리점설계사, 대리점지사
X03: 보험종료일자, 취급부서, 취급지점, 취급자, 입력자, 대리점설계사, 대리점지사
X04: 데이터생성일시, 취급부서, 취급지점, 취급자, 대리점설계사, 대리점지사
```

- 위와 같이 설계한 이유는 2가지이다.
  - 일자 조회구간이 길지 않으면 인덱스 스캔 비효율은 성능에 미치는 영향이 적다
    - 실제로 가계약은 주로 3일 내를 조회하며, 보통 전일자 조회다.
  - 인덱스 스캔 효율보다 테이블 액세스가 더 큰 부하요소다.


## <span style="color:#802548">_index design example 3- 중복 index 제거하기_</span>
- 아래와 같은 index의 경우 X03만 남기면 된다. X01, X02를 X03이 포함하기 때문이다.
```
X01: 계약ID, 청약일자
X02: 계약ID, 청약일자, 보험개시일자
X03: 계약ID, 청약일자, 보험개시일자, 보험종료일자
```

- 아래와 같은 경우는 달라보여도 중복이나 다름없을 수도 있다. 계약ID의 카디널리티가 낮을 때 그러하다.(선택율이 높을 때, 중복값이 많을 떄)
- 그럼 X05를 제외하곤 나머지는 전부 없애도 된다.
```
X01: 계약ID, 청약일자
X02: 계약ID, 보험개시일자
X03: 계약ID, 보험종료일자
X04: 계약ID, 데이터생성일시
X05: 계약ID, 청약일자, 보험개시일자, 보험종료일자, 데이터생성일시
```

## <span style="color:#802548">_index design example 4- sort 연산 고려하기_</span>

- 소트연산을 줄이기 위해 조건절에는 없어도 index에 포함시킬 수도 있다.
```sql
SELECT *
FROM 계약
WHERE 취급지점ID = :trt_brch_id
AND 청약일자 BETWEEN :sbcp_dt1 AND :sbcp_dt2
AND 입력일자 >= trunc(sysdate -3)
AND 계약상태코드 in (:ctr_stat_cd1, :ctr_stat_dt2, :ctr_stat_cd3)
ORDER BY 청약일자, 입력자ID
```

- 비록 입력자ID는 조건절에는 없지만, sort 연산을 줄이기 위해 index에 추가해준다.
- 따라서 취급지점ID, 청약일자, 입력자ID로 index를 구성하게 된다.
- 그냥 청약일자, 입력자ID로만 구성하면 조건절이 index를 제대로 타지못해 index full scan이 수행된다.

<br/>

- in절의 경우, order by가 소트 연산을 생략하기 위해서는 조건 column이 index filter로 작동해야 한다.
- index는 (거주지역, 혈액형, 연령)으로 구성됐다.
```sql
SELECT 고객번호, 고객명, 거주지역
FROM 고객
WHERE 거주지역 = '서울'
AND 혈액형 IN ('A','O')
ORDER BY 연령
```

- in절 변환이 in-list 형식으로 풀려선 안된다. 즉 union all로 풀려선 안된다는 의미다.
- 만약 in-list로 풀린다면, index를 바꾸거나 index 중 하나를 
  - index를 거주지역, 혈액형, 연령으로 하면 3개가 모두 index access 조건이 된다.
  - 따라서 index를 거주지역, 연령, 혈액형으로 비틀어준다.

- index를 일단 생성하기로 했다면, 설계 시 중요한 것은 사용빈도다.
  - 선택도는 생성을 할 지에 관한 판단기준이다.
  - 생성을 했다면 사용빈도가 선두컬럼에 관한 판단기준이다.
  - 그 중 = 조건을 앞쪽에 위치시키는 게 좋다.


<img src="/image/index-modeling.jpg"  />

## <span style="color:#802548">_index의 sort효과_</span>
- index는 자체로 정렬이 되어있기 때문에, order by를 써도 optimizer가 sort 연산을 하지 않을 수 있다.
- index가 (장비번호, 변경일자, 변경순번)이라고 해보자.
- 조건절에는 장비번호, 변경일자만 쓰이고 변경순번은 쓰이지 않았지만 별 상관없다. 
- order by에 변경순번이라고 한다면 index는 이미 장비번호, 변경일자 대로 정렬되어 있기에 그 다음 변경일자도 정렬되어 있다.
- 따라서 order by 변경순번을 붙이든 안 붙이든 상관이 없는 것이다. 
- 대신 order by를 index에 없는 column을 하면 sort by 연산이 추가될 것이다.
```sql
select *
from 상태변경이력
where 장비번호 = 'C'
and 변경일자 = '20180316'
```

- index를 가공해선 안된다는 점은 order by에서도 동일하다.
- 물론 조건절에서 가공한 것보단 덜 치명적이다. order by에서는 sort by 연산이 추가되는 것이기 때문이다.
- 별칭과 관련된 어처구니 없는 실수를 살펴보면서 별칭에 주의해주자.
- 주문번호 column을 TO_CHAR로 가공한 게 보인다. 그런데 ORDER BY를 가공한 주문번호로 정렬했다.
- 주의 깊지 못한 별칭으로 인해 sort by 연산이 추가된다.
```sql
SELECT *
FROM (
  SELECT TO_CHAR(A.주문번호, 'FM000000') AS 주문번호, A.업체번호, A.주문금액
  FROM 주문 A
  WHERE A.주문일자 = :dt
  AND A.주문번호 > NVL(:next_ord_no, 0)
  ORDER BY 주문번호 /**여길 A.주문번호 라고 해야... */
)
WHERE ROWNUM <=30
```

- sort by의 효과는 order by에서만 보이는 게 아니다.
- select에서도 볼 수 있다.
- 아래와 같이 (장비번호, 변경일자, 변경순번)으로 구성되었다고 해보자.
- 장비번호와 변경일자 순으로 scan을 하기 때문에 그다음 변경순번은 반드시 정렬되어 있다.
- 따라서 sort by 연산이 필요없다.
```sql
SELECT MIN(변경순번)
FROM 상태변경이력
WHERE 장비번호 = 'C'
AND 변경일자 = '20180316'
```

- 아래와 같이 MAX도 마찬가지다.
- 블록은 양방향연결 리스트기 때문에 MAX든 MIN이든 상관없다.
- sort by연산이 필요없는 것은 MAX도 마찬가지라는 의미다.
```sql
SELECT MAX(변경순번)
FROM 상태변경이력
WHERE 장비번호 = 'C'
AND 변경일자 = '20180316'
```


- 그러나 이는 같은 type일 때를 기준으로 한다.
- 아래와 같이 type을 바꿔버리면 sort by가 필요하다.
- 저기서 NVL은 문제가 없다. 문제는 TO_NUMBER다.
- number로 바꾸는 순간, sort by 연산이 필요해진다.
- 애초에 숫자type으로 설계했어야 했다. 모든걸 char, varchar로 바꾸는 것은 tuning의 가능성을 제한해버린다.
```sql
SELECT NVL(MAX(TO_NUMBER(변경순번)), 0)
FROM 상태변경이력
WHERE 장비번호 = 'C'
AND 변경일자 = '20180316'
```

- 오라클 11g부터는 prefetch와 배치 I/O도 도입됐다.
- prefetch나 배치 I/O나 버퍼블록에서 읽으면 차이가 없다. 디스크 I/O일 때 차이가 난다.
- 테이블 배치 I/O를 하게되면 order by를 명시하지 않으면 index를 통한 암묵적 결과 정렬이 통하지 않는다.
- 아래처럼 맨 끝에 ORDER BY를 넣어줘야 순서가 정렬된 채로 resultSet이 나온다.
- 물론 order by를 추가한다고 sort 연산이 추가되는 것은 아니다.
```sql
SELECT A.등록일시, A.번호, A.제목, B.회원명, A.게시판유형, A.질문유형
FROM (
  SELECT A.*, ROUWNUM NO
  FROM (
        SELECT 등록일시, 번호, 제목, 작성자번호, 게시판유형, 질문유형
        FROM 게시판
        WHERE 게시판유형 = :TYPE
        ORDER BY 등록일시 DESC /**TOP N 알고리즘을 위한 order by */
      ) A
  WHERE ROWNUM <= (:page * 10) /**TOP N 알고리즘을 위한 rownum*/
    ) A, 회원 B
WHERE A.NO >= (:page - 1) * 10 + 1
AND B.회원번호 = A.작성자번호
ORDER BY A.등록일시 DESC  /**11g부터 여기에 order by를 명시해야 정렬 순서 보장 */
```

- prefetch의 실행계획은 아래와 같이 나온다.

<img src="/image/prefetch.jpg" />

- batch IO의 실행계획은 아래와 같이 나온다.

<img src="/image/batch-IO.jpg" />



## <span style="color:#802548">_SORT 연산_</span>
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

- 소트 연산은 메모리 집약적이며, CPU 집약적이다.
- 거기다 PGA가 작으면 disk I/O까지 일어나니 쿼리 성능에 핵심적이다.
- 될 수 있다면 sort를 발생시키지 않는게 좋고, 필요하면 메모리로 끝내야 한다.



<img src="/image/sort-process.jpg" />


## <span style="color:#802548">_SORT 연산 byte 최소화_</span>
- sort area에서 sort를 해야 한다면, byte를 적게 잡아야 한다.
- 그래야 메모리 안에서 sort가 이뤄진다. sort area를 벗어나면 바로 disk I/O가 돼버린다.
- LPAD 등으로 가공하면서 공백이 늘면 그만큼 byte도 늘어난다.
```sql
SELECT LPAD(상품번호, 30) || LPAD(상품명, 30) || LPAD(고객ID, 10)
        ||  LPAD(고객명, 20) || TO_CHAR(주문일시, 'yyyymmdd hh24:mi:ss')
FROM 주문상품
WHERE 주문일시 BETWEEN :start and :end
ORDER BY 상품번호
```

- 차라리 아래처럼 sort 자체는 기본 field로 해야한다.
- 그래야 sort byte가 적어져 sort area에서 sort가 끝난다.
```sql
SELECT LPAD(상품번호, 30) || LPAD(상품명, 30) || LPAD(고객ID, 10)
        ||  LPAD(고객명, 20) || TO_CHAR(주문일시, 'yyyymmdd hh24:mi:ss')
FROM (
  SELECT 상품번호, 상품명, 고객ID, 고객명, 주문일시
  FROM 주문상품
  WHERE 주문일시 BETWEEN :start and :end
  ORDER BY 상품번호
)
```

- 마찬가지 이유로 필요한 필드만 적어야 sort byte를 최소화한다.
```sql
SELECT 계좌번호, 총예수금
FROM 예수금원장
ORDER BY 총예수금 DESC
``` 

- sort area를 최소화하는 기술로 top n sort도 있다.
- top n sort는 top n stopkey보단 비효율적이지만, sort 용량을 크게 줄여주는 데서 의미가 있다.
- top n stopkey를 유도하는 SQL패턴을 그대로 사용하면 된다. 거기서 index를 타지 않으면 top n sort가 된다.
  - top n sort는 딱 가져올 만큼의 공간만 존재하면 된다. 
  - 10개라면 10개를 저장한 공간만 있으면 총데이터 집합이 엄청 커도 10개만 가져오고 멈춘다.
  - 따라서 sort area에 들어가는 건 10개의 byte뿐이다.
  - 따라서 top n sort의 경우 대부분 trace에 pw,pr이 0으로 찍힌다. disk I/O를 안타는 셈이다.
```sql
SELECT * 
FROM (
  SELECT ROWNUM no, a.*
  FROM (
    SELECT 거래일시, 체결건수, 체결수량, 거래대금
    FROM 종목거래
    WHERE 종목코드 = 'KR123456'
    AND 거래일시 >= '20180304'
    ORDER BY 거래일시
  ) a
  WHERE ROWNUM <= (:page * 10)
)
WHERE no >= (:page - 1) * 10 + 1
```

- 절대 아래처럼 바꾸면 안 된다.
- 보기에 간단하지만, 이러면 top n sort가 안 먹힌다.
- 그럼 전체를 긁고서 sort를 하니까 공간이 부족해 disk I/O가 일어날 수 있다.
```sql
SELECT * 
FROM (
  SELECT ROWNUM no, a.*
  FROM (
    SELECT 거래일시, 체결건수, 체결수량, 거래대금
    FROM 종목거래
    WHERE 종목코드 = 'KR123456'
    AND 거래일시 >= '20180304'
    ORDER BY 거래일시
  ) a
)
WHERE no BETWEEN (:page -1) * 10 + 1 and (:page * 10)
```


- 참고로 mysql은 top n sort를 지원해주지 않는다.
- top n sort처럼 처리되지 않고 위의 절대 바꾸며 안되는 패턴처럼 처리된다.
```sql
SELECT 거래일시, 체결건수, 체결수량, 거래대금
FROM 종목거래
WHERE 종목코드 = 'KR123456'
AND 거래일시 >= '2018-03-04'
ORDER BY 거래일시
LIMIT :offset, 10;
```

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
SELECT 장비번호, 변경일자, 변경순번, 상태코드, 메모
FROM (SELECT 장비번호, 변경일자, 변경순번, 상태코드, 메모
            , MAX(변경순번) OVER(PARTITION BY 장비번호) 최종변경순번
            FROM 상태변경이력
            WHERE 변경일자 = :upd_dt)
WHERE 변경순번 = 최종변경순번
```


<img src="/image/max-sort-disk.jpg" />


- rank 같은 window function을 쓰면 훨씬 disk I/O가 줄어든다.
```sql
SELECT 장비번호, 변경일자, 변경순번, 상태코드, 메모
FROM (SELECT 장비번호, 변경일자, 변경순번, 상태코드, 메모
            , RANK(변경순번) OVER(PARTITION BY 장비번호 ORDER BY 변경순번 DESC) 최종변경순번
            FROM 상태변경이력
            WHERE 변경일자 = :upd_dt)
WHERE 변경순번 = 최종변경순번
```

<img src="/image/window-sort-no-disk.jpg" />

## <span style="color:#802548">_SORT execution plan_</span>
- sort operation은 아래와 같다.
  - sort Aggregate
    - 집계함수를 사용한 것이다.
    - 실제 데이터 정렬은 없다.
    - sum, max, min, avg 등이 예시다.
  - sort order by
    - 데이터를 정렬한다.
  - sort group by
    - 데이터를 정렬한다.
    - 그룹 개수가 적으면 수억 row가 있어도 temp table space를 쓰지 않는다.
    - order by가 없으면 대부분 hash group by로 처리된다.
    - sort group by와 다른 점은 해싱알고리즘이라는 점 말고 없다.
    - hash과 sort나 group by는 결과가 정렬됨을 보장하지 않는다.
  - sort unique
    - 서브쿼리를 unnest할 때, 서브쿼리가 1대 다 중 다쪽일 때 
    - 1쪽 집합인데 조인컬럼에 unique index가 없을 때 나타난다
    - 메인쿼리와 조인하기 전에 중복 레코드를 제거해야 하기 때문이다.
    - 아니면 union, minus, intersect, distinct를 써도 나타난다.
    - distinct의 경우 hash unique가 더 많이 쓰인다.
  - sort join
    - sort merge join을 수행할 때 나타난다.
  - window sort
    - 윈도우 함수를 수행할 때 나타난다.


- 참고로 SORT GROUP BY에서도 index를 활용해 SORT를 하지 않을 수 있다.
- 이 경우 region이 선두컬럼이어야 한다.
```sql
SELECT region, avg(age), count(*)
FROM customer
GROUP BY region
```

- 실행계획에 SORT GROUP BY NOSORT라고 되어있다.
<img src="/image/sort-group-by.jpg" />



## <span style="color:#802548">_SORT를 피하는 SQL 1 -EXISTS/UNION ALL_</span>
- union, minus, distinct는 최대한 자제해야 한다.
  - union보다는 union all로 쓸 수 있는지 확인해봐야 한다.
  - distinct보다는 exists를 사용할 수 있는지 확인해봐야 한다.
  - 대량데이터가 아니라면 hash join이 아니라 NL join을 써야 한다.


- union의 예시를 들어보자.
```sql
SELECT 결제번호, 결제수단코드, 주문번호, 결제금액, 결제일자, 주문일자
FROM 결제
WHERE 결제일자 ='20180316'

UNION

SELECT 결제번호, 결제수단코드, 주문번호, 결제금액, 결제일자, 주문일자
FROM 결제
WHERE 주문일자 = '20180316'
```

- 아래와 같이 union all과 <> 연산자를 이용해 sort가 일어나지 않게 바꾼다.
```sql
SELECT 결제번호, 결제수단코드, 주문번호, 결제금액, 결제일자, 주문일자
FROM 결제
WHERE 결제일자 ='20180316'

UNION ALL

SELECT 결제번호, 결제수단코드, 주문번호, 결제금액, 결제일자, 주문일자
FROM 결제
WHERE 결제일자 = '20180316'
AND 결제일자 <> '20180316'
```

- 만약 결제일자가 null 허용 컬럼이면 아래와 같이 바꿔준다.
```sql
SELECT 결제번호, 결제수단코드, 주문번호, 결제금액, 결제일자, 주문일자
FROM 결제
WHERE 결제일자 ='20180316'

UNION ALL

SELECT 결제번호, 결제수단코드, 주문번호, 결제금액, 결제일자, 주문일자
FROM 결제
WHERE 결제일자 = '20180316'
AND (결제일자 <> '20180316' or 결제일자 is null) /** LNNVL(결제일자 ='20180316') */
```

- distinct를 제거하는 쿼리 튜닝도 보자.
```sql
SELECT DISTINCT p.상품번호, p.상품명, p.상품가격
FROM 상품 p, 계약 c
WHERE p.상품유형코드 = :pclscd
AND c.상품번호 = p.상품번호
AND c.계약일자 BETWEEN :dt1 and :dt2
AND c.계약구분코드 = :ctpcd
```


- 위의 쿼리 대신 exists를 활용해보자.
- exists를 이용하면 데이터를 모두 읽지 않아도 된다.
- distinct를 사용하지 않았으니 부분범위 처리도 가능하다.
```sql
SELECT p.상품번호, p.상품명, p.상품가격
FROM 상품 p
WHERE p.상품유형코드 = :pclscd
AND EXISTS (
          SELECT 'x' FROM 계약 c
          WHERE c.상품번호 = p.상품번호
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
- 계약 table의 index가 (지점ID, 계약일시)기 때문에 order by가 생략될 수 있다.
- 그러나 hash join이 되면서 SORT ORDER BY 연산이 일어났다.
```sql
SELECT c.계약번호, c.상품코드, p.상품명, p.상품구분코드
FROM 계약 c, 상품 p
WHERE c.지점ID = :brch_id
AND p.상품코드 = c.상품코드
ORDER BY c.계얄일시 DESC
```


- 이를 NL join으로 바꿔준다.
```sql
SELECT /*+ leading(c) use_nl(p) */ c.계약번호, c.상품코드, p.상품명, p.상품구분코드
FROM 계약 c, 상품 p
WHERE c.지점ID = :brch_id
AND p.상품코드 = c.상품코드
ORDER BY c.계얄일시 DESC
```


<img src="/image/join-sort-NL-hash.jpg" />


## <span style="color:#802548">_SORT를 피하는 SQL 2 -TOP N쿼리_</span>
- top n 쿼리는 아래 두개로 나뉜다.
  - top n stopkey
    - top n stopkey는 정렬이 필요 없다.
    - 거기다가 부분범위 처리만 하기에 전체 scan도 하지 않는다.
    - 조건으론 index 구성 컬럼이 where 혹은 order by에 있기만 하면 된다.
    - 다만 없거나, order by에 index 비구성 컬럼이 들어가면 top n sort가 된다.
  - top n sort
    - top n sort는 정렬은 필요하다.
    - 따라서 전체 scan은 필수적이다.
    - 다만 전체 index record를 정렬하지 않고 rownum의 갯수만큼만 정렬하면 된다.

```sql
SELECT * FROM (
  SELECT 거래일시, 체결건수, 체결수량, 거래대금
  FROM 종목거래
  WHERE 종목코드 = 'KR123456'
  AND 거래일시 >= '20180304'
  ORDER BY 거래일시
)
WHERE ROWNUM <= 10
```


- 실행계획은 아래와 같이 나타난다.
- Sort Order by는 사라지고 Count Stopkey가 생겨났다.
- COUNT STOPKEY만 있고 SORT ORDER BY STOPKEY가 없으면 top n stopkey로 가장 좋은 경우다.
- COUNT STOPKEY가 있고 SORT ORDER BY STOPKEY가 있으면 top n sort로 그래도 괜찮은 경우다.
- COUNT가 있고 SORT ORDER BY가 있으면 top n query 어느것도 이용하지 못한 경우다. 
  - sort를 하게 되는데, 필요한만큼만 덜어서 PGA를 채우지 못하기 때문에 disk I/O가 일어날 확률이 높아진다.
- COUNT가 있고 SORT ORDER BY가 없는 경우도 top n query 어느것도 이용하지 못한 경우다.
  - 그래도 sort는 하지 않기 때문에 disk I/O가 일어날 확률은 낮다. COUNT + SORT ORDER BY보단 성능이 낫다.


<img src="/image/top-n-stopkey.jpg" />



- 실제 웹의 페이징처리를 top n query를 적용하면 보통 아래와 같다.
- top n 쿼리므로 ROWNUM으로 지정한 건수만큼 결과 레코드를 얻으면 거기서 멈춘다.
- 뒤쪽 페이지로 이동할수록 읽는 데이터량도 많아지지만, 보통 앞쪽만 확인하니 문제가 거의 없다.
```sql
SELECT *
FROM (
  SELECT ROWNUM no, a.*
  FROM (
    /** SQL Body */

      ) a
  WHERE ROWNUM <= (:page * 10)
  )
WHERE no >= (:page - 1)*10 + 1
```

- 위와 같은 sql은 아래와 같은 원칙을 구현한 것이다.
  - 부분범위 처리가 가능하도록 SQL을 작성한다.
  - 작성한 SQL문을 SQL BODY쪽에 집어넣는다.
  - ORDER BY도 필수적으로 들어가야 한다.
- 이게 뜻하는 바는 아래와 같다.
  - 인덱스 사용 가능하도록 조건절을 구사한다.
  - 조인은 NL조인 위주로 처리한다.
  - ORDER BY절이 있어도 소트 연산을 생략할 수 있게 인덱스를 구성한다.
- 해당 원칙을 지켜 만든 SQL을 만든다면 아래와 같은 형태일 것이다.
```sql
SELECT *
FROM (
  SELECT ROWNUM no, a.*
  FROM (
    /** SQL Body */
      SELECT 거래일시, 체결건수, 체결수량, 거래대금
      FROM 종목거래
      WHERE 종목코드 = 'KR123456'
      AND 거래일시 >= '20180304'
      ORDER BY 거래일시
      ) a
  WHERE ROWNUM <= (:page * 10)
  )
WHERE no >= (:page - 1)*10 + 1
```


<img src="/image/rownum-top-n-count-stopkey.jpg" />



- 페이징 처리의 안티 패턴은 rownum을 지워버리는 것이다.
- rownum을 쓸데없다고 생각하고 지우는 순간, top n query 알고리즘은 멈춘다.
- 더 문제는 top n sort 마저도 작동하지 않아 pr, pw를 해야할 수도 있다는 사실이다.
```sql
SELECT *
FROM (
  SELECT ROWNUM no, a.*
  FROM (
    SELECT 거래일시, 체결건수, 체결수량, 거래대금
    FROM 종목거래
    WHERE 종목코드 = 'KR123456'
    AND 거래일시 >= '20180304'
    ORDER BY 거래일시
  ) a
)
WHERE no BETWEEN (:page -1) * 10 +1 and (:page * 10)
```

- rownum을 지우면 아래와 같이 실행계획이 바뀐다.
- count 옆에 stopkey가 없다. 소트연산은 생략되어도 top n query가 작동하진 않는다.
<img src="/image/rownum-top-n-count-no-stopkey.jpg" />


- stopkey를 못 하면 아래와 같이 되어버린다.
- 전체범위를 전부 훑어야 되는 것이다.
<img src="/image/sort-omit-no-stopkey.jpg" />

- 또 다른 안티 패턴은 아래와 같이 rownum의 <= 조건에서 rownum을 별칭으로 사용하는 것이다.
- 그럼 top n query 알고리즘은 멈춘다. 따라서 전체범위를 훑게 된다.
```sql
SELECT *
FROM (
  SELECT ROWNUM no, a.*
  FROM (
    /** SQL Body */
      SELECT 거래일시, 체결건수, 체결수량, 거래대금
      FROM 종목거래
      WHERE 종목코드 = 'KR123456'
      AND 거래일시 >= '20180304'
      ORDER BY 거래일시
      ) a
  WHERE no <= (:page * 10)
  )
WHERE no >= (:page - 1)*10 + 1
```

- 또 다른 안티 패턴은 아래와 같이 body SQL에서 형변환을 해버리는 것이다.
- 그럼 가공된 컬럼이 되어, 정렬이 필요해지고 top n query 알고리즘은 멈춘다. 따라서 전체범위를 훑게 된다.
```sql
SELECT *
FROM (
  SELECT ROWNUM no, a.*
  FROM (
    /** SQL Body */
      SELECT 거래일시, 체결건수, 체결수량, TO_NUMBER(거래대금)
      FROM 종목거래
      WHERE 종목코드 = 'KR123456'
      AND 거래일시 >= '20180304'
      ORDER BY 거래일시
      ) a
  WHERE no <= (:page * 10)
  )
WHERE no >= (:page - 1)*10 + 1
```

- 그럴 땐 아래와 *를 사용하지 않고 column들을 모두 전부 쓰는 수밖에 없다.
```sql
SELECT *
FROM (
  SELECT ROWNUM no
        ,거래일시
        ,체결건수
        ,체결수량
        ,TO_NUMBER(거래대금)
  FROM (
    /** SQL Body */
      SELECT 거래일시, 체결건수, 체결수량, 거래대금
      FROM 종목거래
      WHERE 종목코드 = 'KR123456'
      AND 거래일시 >= '20180304'
      ORDER BY 거래일시
      ) a
  WHERE ROWNUM <= (:page * 10)
  )
WHERE no >= (:page - 1)*10 + 1
```

- 거래PK는 (거래일자, 계좌번호, 거래순번)로 이뤄졌다.
- 거래_X01은 (계좌번호, 거래순번, 결제구분코드)로 이뤄졌다.
  - 그 상황에서 아래 SQL을 실행하면 TOP N sort으로 타다. 즉 sort by 연산을 생략하지 못한다.
  - 부분범위 처리는 불가능하다는 의미다. 전체를 다 읽어야 하니 오래걸린다.
```sql
SELECT *
FROM (
  SELECT 계좌번호, 거래순번, 주문금액, 주문수량, 결제구분코드
  FROM 거래
  WHERE 거래일자 = :ord_dt
  ORDER BY 계좌번호, 거래순번, 결제구분코드
)
WHERE ROWNUM <=50
```

- 위의 SQL의 실행계획은 아래와 같다.
  - 어차피 거래_X01 index는 쓸수가 없다. where 조건문에 거래일자가 들어갔기 때문이다.
  - 그럼 거래PK를 쓰는데, 하필 결제구분코드가 더 들어갔다.
  - 그래서 sort order by 연산이 진행된 것이다.
  - 반대로 말하면 결제구분코드만 없으면 된다. 실제로 필요하지 않다면 없애자.
    - index가 (거래일자, 계좌번호, 거래순번)이고 거래일자가 = 조건이다.
    - 그럼 거래일자 데이터를 계좌번호, 거래순번 순으로 정렬하면 중복 레코드가 없다.
    - 다시 말하면 결제구분코드는 없어도 된다는 의미다. 없애면 부분범위 처리가 가능해져 매우 빨라진다.


<img src="/image/top-n-useless-column-delete.jpg" />

- top n은 max, min 연산을 대체하는 데도 사용할 수 있다.
  - 특히 index 중 하나가 빠져도 작동해서 유연하다.
  - (DEPTNO, SAL)로 구성해도 TOP N stopkey로 작동한다.
  - 아래에서 MGR이 index 구성으로 없어도 되기에 좀 더 유연하게 index를 구성할 수 있다.
  - 다시말해 선두컬럼만 where 조건문에 있고, order by에 들어가는 건 index 구성 순서를 따른 column이어야 한다.
```sql
SELECT *
FROM (
  SELECT SAL
  FROM EMP
  WHERE DEPTNO = 30
  AND MGR = 7698
  ORDER BY SAL DESC
    )
WHERE ROWNUM <= 1;
```

- 실행계획은 아래와 같이 나타난다.
<img src="/image/max-topn-execution-plan.jpg" />


- MGR 컬럼이 index에 없으니 DEPTNO column만이 index access조건이다.
- 따라서 원래는 DEPTNO = 30을 만족하는 모든 index record를 읽어야 한다. 
- 그러나 top n stopkey의 경우, 조건을 만족하는 모든 index 레코드를 읽지 않는다. 
  - DEPTNO = 30을 만족하는 가장 오른쪽부터 역순으로 스캔하며 table을 access한다.
  - 그 중 MGR = 7698 조건을 만족하는 table record를 하나 찾는 순간 바로 멈춘다.
<img src="/image/max-topn.jpg" />

- min으로 하고 싶다면 DESC가 아니라 AESC로 주면 된다.
- 다른 예시도 들어보자.
- 아래와 같은 쿼리가 있다고 해보자.
- index 구성이 (customer_no, login_date)기 때문에 order by에 login_date가 빠져도 정렬이 가능하다.
```sql
SELECT 
from login
where 
and rownum <= 1
```

- 다만 위와 같은 sql은 top n stopkey 알고리즘이 작동하지 않는다.
- 아래와 같이 만들어줘야 top n stopkey 알고리즘이 작동하게 된다.
- 만약 저기서 order by에 index가 아닌 column을 추가하게 되면 top n sort로 변경된다.
```sql
SELECT * FROM (
  SELECT auto_login_yn, login_date, auto_login_token, customer_grade, customer_name
  FROM 종목거래
  WHERE customer_no = #{customer_no}
  ORDER BY login_date
)
WHERE ROWNUM <= 1
```


- top n sort를 응용하면 더보기도 만들어줄 수 있다.
- index 구성이 모자라서 top n stopkey로 작동은 못해도 top n sort로는 작동가능하다.
```sql
SELECT CASE WHEN COUNT(1) = 11  THEN 1 ELSE 0 END AS HAS_MORE
FROM (
  SELECT board_sqno
  FROM board
  WHERE board_clf = #{board_clf}
  AND board_display = 'Y'
  <if test = "latestNo != null" >
    AND board_sqno < #{latestNo}
    AND board_display_fix_yn <> 'Y'
  </if>
  ORDER BY board_display_fix_yn DESC
          , board_display_order ASC
          , board_sqno DESC
    )
WHERE rownum <= 11
```


## <span style="color:#802548">_SORT를 피하는 SQL 3 -first row_</span>
- 위에서는 MAX, MIN을 TOP N 쿼리로 구해왔다. 
- 그런데 만약 MAX, MIN으로 쓴다면 그 땐 SORT ORDER BY 연산이 일어난다.
  - 전체 데이터를 정렬하진 않지만, 전체 데이터를 읽으면서 값을 비교한다.
- 아래와 같이 sort aggregate라는 연산이 실행계획에 보인다.


<img src="/image/max-sort-aggregate.jpg" />



- index가 정렬되어 있다면, 전체 데이터를 읽지 않아도 된다.
  - index를 맨 왼쪽으로 내려가면 첫번쨰 읽는 게 최소값이다.
  - index를 맨 오른쪽으로 내려가면 첫번째 읽는 게 최대값이다.
  - index leaf block은 양방향리스트니 왼쪽이나 오른쪽이나 가는 방향을 선택할 수 있다.

<img src="/image/max-min-index-sort-aggregate.jpg" />


- 단 위처럼 전체 데이터를 읽지 않으려면 조건이 있다.
  - 조건절 컬럼이 index에 있어야 한다.
  - max/min 컬럼이 index에 있어야 한다.
- 아래의 sql을 보자.
- deptno, mgr, sal이 모두 index에 포함되어 있으면 전체 데이터를 읽지 않아도 된다.
- 아래는 (deptno, mgr, sal)로 index를 구성한 경우다.
  - 이 경우 deptno와 mgr이 index access조건이다.
  - 안타깝게도 TOP N 처럼 MGR이 index에 없어도 되진 않다.
  - MAX 함수를 사용하면 index 효과를 톡톡히 보기 위해 MGR도 반드시 추가되어야 한다.
```sql
SELECT MAX(SAL) FROM EMP WHERE DEPTNO = 30 AND MGR = 7698;
```

- 만약 알고리즘이 제대로 작동했다면 아래와 같이 FIRST ROW라는 실행계획이 추가된다.
- 매우 효율적인 min,max 연산이 이뤄졌다는 의미다.


<img src="/image/first-row-stopkey.jpg" />

- 그림으로 이해하면 아래와 같다.
- 조건절이 모두 index access일 때의 상황이다.
<img src="/image/max-index-range-both-access-firstrow-stop.jpg" />


- index를 조금 바꿔서 (deptno, sal, mgr)로 구성해보자.
  - 이 경우 deptno이 index access조건이다.
  - mgr은 index filter조건이다.
  - 그러나 First row key연산인 점은 변하지 않는다.
    - deptno = 30인 index record 범위에서 가장 오른쪽으로 내려가 가장 큰 SAL 값을 읽는다.
    - 거기서부터 스캔을 시작해 MGR = 7698을 만족하는 레코드를 찾으면 멈춘다.


- 그림으로 이해하면 아래와 같다.
- 조건절 선두컬럼은 index access고, 후행은 index filter인 상황이다.
<img src="/image/max-index-range-one-access-one-filter-firstrow-stop.jpg" />



- index를 조금 바꿔서 (sal, deptno, mgr)로 구성해보자.
  - 이 경우 deptno와 mgr 모두 선두컬럼이 아니다.
  - 따라서 index range scan이 아니라 index full scan이 일어난다.
  - deptno와 mgr 모두 index filter 조건이된다.
  - 그러나 First row key연산인 점은 변하지 않는다.
    - index record 범위에서 가장 오른쪽에서 스캔을 시작한다.
    - DEPTNO = 30이면서 MGR = 7698인 조건을 찾으면 멈춘다.


- 그림으로 이해하면 아래와 같다.
- 조건절 모두가 index filter인 상황이다.
<img src="/image/max-index-full-both-filter-firstrow-stop.jpg" />




- index를 조금 바꿔서 (deptno, sal)로 구성해보자.
  - 이경우 조건절에 쓰인 mgr 컬럼이 없다.
  - DEPTNO = 30이면서 MAX(SAL)인 값은 오른쪽에서부터 시작해 쉽게 찾는다.
  - 그러나 mgr 컬럼이 index에 없어 MGR = 7698 조건은 테이블에서 필터링해야한다.
  - 따라서 First row key연산이 작동하지 않는다.



- 그림으로 이해하면 아래와 같다.
- index column에 조건절이 없어 first row가 작동하지 않았다.
<img src="/image/max-table-scan-not-first-row.jpg" />

## <span style="color:#802548">_SORT 알고리즘의 꽃 -이력조회_</span>
- 이력조회에 first row stopkey나 top n stopkey 알고리즘을 이용하는 게 필요하다.
- 아래와 같이 장비 table이 있다고 해보자.
  - 장비번호 PK
  - 장비명
  - 장비구분코드
  - 상태코드
  - 최종변경일자
- 상태변경 이력 table은 아래와 같다.
  - 장비번호 PK1
  - 변경일자 PK2
  - 변경순번 PK3
  - 상태코드
  - 메모
- 가장 단순하게 이력데이터를 조회해보자.
- 스칼라 서브쿼리에서 장비번호와 변경일자가 쓰이는데, 선두컬럼이기에 first row stopkey 알고리즘이 발동한다.
```sql
SELECT 장비번호, 장비명, 상태코드,
    (SELECT MAX(변경일자)
    FROM 상태변경이력
    WHERE 장비번호 = P.장비번호) 최종변경일자
FROM 장비 P
WHERE 장비구분코드 = 'A001'
```

- 위의 기능에서 최종변경순번도 가져오게 추가되었다.
- inline view에서 최종변경일자와 최종변경순번을 겹쳐 가져오려 한다.
- 그런 시도는 좋지만, 그때문에 index 컬럼이 가공돼 index access조건으로 타지 못하게 됐다.
```sql
SELECT 장비번호, 장비명, 상태코드
      , SUBSTR(최종이력, 1, 8) 최종변경일자
      , TO_NUMBER(SUBSTR(최종이력, 9,4)) 최종변경순번
FROM (
  SELECT 장비번호, 장비명, 상태코드
  , (SELECT MAX(H.변경일자 || LPAD(H.변경순번, 4))
        FROM 상태변경이력 H
        WHERE 장비번호 = P.장비번호) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
    )
```

- 따라서 아래와 같은 쿼리로 바꿔주는 게 낫다.
- 스칼라 서브쿼리가 많아서 비효율적일 거 같지만 index access로 작동하는 게 더 빠르다.
```sql
SELECT 장비번호, 장비명, 상태코드
      ,(SELECT MAX(H.변경일자)
        FROM 상태변경이력 H
        WHERE 장비번호 = P.장비번호) 최종변경일자
      ,(SELECT MAX(H.변경순번)
        FROM 상태변경이력 H
        WHERE 장비번호 = P.장비번호
        AND 변경일자 = (SELECT MAX(H.변경일자)
                        FROM 상태변경이력 H
                        WHERE 장비번호 = P.장비번호)) 최종변경순번
FROM 장비 P
WHERE 장비구분코드 = 'A001'
```

- 그런데 최종변경순번에다가 또 다른 field도 가져와야 한다면?
- 최종상태코드를 가져오려면 아래와 같이 길어진다.
- 다음은 더 길어질 것이다.
```sql
SELECT 장비번호, 장비명, 상태코드
      ,(SELECT MAX(H.변경일자)
        FROM 상태변경이력 H
        WHERE 장비번호 = P.장비번호) 최종변경일자
      ,(SELECT MAX(H1.변경순번)
        FROM 상태변경이력 H1
        WHERE 장비번호 = P.장비번호
        AND 변경일자 = (SELECT MAX(H2.변경일자)
                        FROM 상태변경이력 H2
                        WHERE 장비번호 = P.장비번호)) 최종변경순번
      ,(SELECT H1.상태코드
        FROM 상태변경이력 H1
        WHERE 장비번호 = P.장비번호
        AND 변경일자 = (SELECT MAX(H2.변경일자)
                        FROM 상태변경이력 H2
                        WHERE 장비번호 = P.장비번호)
        AND 변경순번 = (SELECT MAX(H3.변경순번)
                        FROM 상태변경이력 H3
                        WHERE 장비번호 = P.장비번호
                        AND 변경일자 = (SELECT MAX(H4.변경일자)
                                        FROM 상태변경이력 H4
                                        WHERE 장비번호 = P.장비번호))) 최종상태코드
FROM 장비 P
WHERE 장비구분코드 = 'A001'
```                                        


- 단순한 쿼리를 위해 전통적으로는 아래와 같이 썼다.
- INDEX_DESC라 인덱스를 역순으로 읽는것이고, 첫 레코드에서 바로 멈춘다.
- 다만 아래와 같은 방법은 index 구성이 완벽해야 한다.
```sql
SELECT 장비번호, 장비명
      , SUBSTR(최종이력, 1, 8) 최종변경일자
      , TO_NUMBER(SUBSTR(최종이력, 9, 4)) 최종변경순번
      , SUBSTR(최종이력, 13) 최종상태코드
FROM (
  SELECT 장비번호, 장비명
        , (SELECT /*+ INDEX_DESC(X 상태변경이력_PK) */
                  변경일자 || LPAD(변경순번, 4) || 상태코드
            FROM 상태변경이력 X
            WHERE 장비번호 = P.장비번호 
            AND ROWNUM <=1) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```

- 11g에서는 아래와 같이 쓰면 된다.
- WHERE 장비번호 =P.장비번호가 서브쿼리로 들어가있다.
- 그러나 실제로는 inline view로 들어가서 조건절이 작동한다.
- prdicated pushing이다.
```sql
SELECT 장비번호, 장비명
      , SUBSTR(최종이력, 1, 8) 최종변경일자
      , TO_NUMBER(SUBSTR(최종이력, 9, 4)) 최종변경순번
      , SUBSTR(최종이력, 13) 최종상태코드
FROM (
  SELECT 장비번호, 장비명
        , (SELECT 변경일자 || LPAD(변경순번, 4) || 상태코드
            FROM (SELECT 장비번호, 변경일자, 변경순번, 상태코드
                  FROM 상태변경이력 
                  ORDER BY 변경일자 DESC, 변경순번 DESC) /**11g에서 가능한 방식*/
            WHERE 장비번호 = P.장비번호 
            AND ROWNUM <=1 ) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```

- 12c에서는 아래와 같이 쓰면 된다.
```sql
SELECT 장비번호, 장비명
      , SUBSTR(최종이력, 1, 8) 최종변경일자
      , TO_NUMBER(SUBSTR(최종이력, 9, 4)) 최종변경순번
      , SUBSTR(최종이력, 13) 최종상태코드
FROM (
  SELECT 장비번호, 장비명
        , (SELECT 변경일자 || LPAD(변경순번, 4) || 상태코드
            FROM (SELECT 장비번호, 변경일자, 변경순번, 상태코드
                  FROM 상태변경이력 
                  ORDER BY 변경일자 DESC, 변경순번 DESC
                  WHERE 장비번호 = P.장비번호)     /**12c에서 가능한 방식*/
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
SELECT 장비번호, 장비명
      , SUBSTR(최종이력, 1, 8) 최종변경일자
      , TO_NUBMER(SUBSTR(최종이력, 9,4)) 최종변경순번
      , SUBSTR(최종이력, 13) 최종상태코드
FROM (
  SELECT 장비번호, 장비명
        , (SELECT 변경일자 || LPAD(변경순번, 4) || 상태코드
        FROM (SELECT 변경일자, 변경순번, 상태코드
                    , ROW_NUMBER() OVER (ORDER BY 변경일자 DESC, 변경순번 DESC) NO
              FROM 상태변경이력
              WHERE 장비번호 = P.장비번호)
        WHERE NO = 1) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```

- 12c의 row limit기능을 써도 사실 위와 같은 window 함수를 쓰는 것이다.
- 따라서 stopkey 알고리즘이 발동하지 않는다.
```sql
SELECT 장비번호, 장비명
      , SUBSTR(최종이력, 1, 8) 최종변경일자
      , TO_NUBMER(SUBSTR(최종이력, 9,4)) 최종변경순번
      , SUBSTR(최종이력, 13) 최종상태코드
FROM (
  SELECT 장비번호, 장비명
        , (SELECT 변경일자 || LPAD(변경순번, 4) || 상태코드
  FROM 상태변경이력
  WHERE 장비번호 = P.장비번호
  ORDER BY 변경일자 DESC, 변경순번 DESC
  FETCH FIRST 1 ROWS ONLY) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
);
```


- 페이징 처리에 활용할 때 윈도우 함수를 쓰기도 한다.
- 하지만 쓰지 않는 경우도 자주 발생한다. 따라서 index/index_desc hint를 자주 써야 한다.
- sort 생략 가능한 index가 없으면 top n stopkey가 아니라 top n sort가 작동해버린다.
```sql
SELECT 변경일자, 변경순번, 상태코드
FROM (
  SELECT 변경일자, 변경순번, 상태코드
        , ROW_NUBMER() OVER(ORDER BY 변경일자, 변경순번) NO
  FROM 상태변경이력
  WHERE 장비번호 = :eqp_no)
WHERE NO BETWEEN 1 AND 10;
```

- 장비번호가 특정번호일 때는 index 조회가 효과적이다.
- 하지만 전체에 관해 조회할 때는 index를 쓰면 random access에 따른 손해가 크다.
- 이럴 땐 window function을 쓰는 게 더 나은 선택이다.
```sql
SELECT P.장비번호, P.장비명
      , H.변경일자 AS 최종변경일자
      , H.변경순번 AS 최종변경순번
      , H.상태코드 AS 최종상태코드
FROM 장비 P
      , (SELECT 장비번호, 변경일자, 변경순번, 상태코드
              , ROW_NUMBER() OVER(PARTITION BY 장비번호
                                  ORDER BY 변경일자 DESC, 변경순번 DESC) RNUM
        FROM 상태변경이력) H
WHERE H.장비번호 = P.장비번호
AND H.RNUM = 1;
```


<img src="/image/window-function.jpg" />


## <span style="color:#802548">_join_</span>
- join을 하는 이유는 원하는 데이터가 다른 table에 있어서다.
- join은 아래와 같이 4개의 방식이 있다.
  - NL
    - WAS는 대부분 이걸 채택한다.
  - sort merge
    - 거의 안쓰인다.
  - hash
    - 대용량 table batch 시 사용된다.
  - subquery
    - inline view 형태로 진행되는데, 피할 수 있으면 피해야한다.
    - 다만 inline view로 해야 join하는 table의 양이 줄어든다면, 그렇게 해야 한다.


## <span style="color:#802548">_NL 조인_</span>
- NL은 중첩 루프문과 같은 수행 구조를 지닌다.
- 일반적으로 outer와 inner 모두 index를 사용하는 편이다.
  - 다만 outer는 size가 크지 않으면 index를 하지 않기도 한다. 어차피 1번 scan이기 때문이다.
  - 하지만 inner는 여러 번 반복 scan되기 때문에 반드시 index를 사용하는 게 좋다.
```sql
SELECT e.사원명, c.고객명, c.전화번호
FROM 사원 e, 고객 c
WHERE e.입사일자 >= '19960101'
AND c.관리사원번호 = e.사원번호
```

- 위와 같은 경우, outer인 사원의 입사일자는 1번 조회다.
- 그렇게 scan해온(index이든 table이든) record를 inner인 고객의 관리사원번호 조건에 맞게 다시 record를 scan한다.
- 따라서 inner는 outer의 갯수만큼 반복 scan을 하기에 반드시 효율적이어야만 한다.

<br />

- 더 자세한 예시를 들어보자.
- 아래와 같은 예시가 있다.
```sql
SELECT /*+ ordered use_nl(c) index(e) index(c) */ 
  e.사원번호, e.사원명, e.입사일자, c.고객번호, c.고객명, c.전화번호, c.최종주문금액
FROM 사원 e, 고객 c
WHERE c.관리사원번호 = e.사원번호
AND e.입사일자 >= '19960101'
AND e.부서코드 = 'Z123'
AND c.최종주문금액 >= 20000

--
사원_PK: 사원번호
사원_X1: 입사일자
고객_PK: 고객번호
고객_X1: 관리사원번호
고객_X2: 최종주문금액
```

```
|ID       |Operation                      |Name      |Rows     |Bytes     |Cost
------------------------------------------------------------------
0         |SELECT STATMEENT               |         |5         |58        |
1         | NESTED LOOPS                  |         |5         |58
2         |  TABLE ACCESS BY INDEX ROWID  |사원     |3          |20
3         |   INDEX RANGE SCAN            |사원_X1  |5
4         |  TALBE ACCESS BY INDEX ROWID  |고객     |5          |76
5         |   INDEX RANGE SCAN            |고객_X1  |8
```

- 그럼 아래와 같은 과정을 거쳐 NL join이 일어난다.
```
1. 조건절 2번을 만족하는 index record를 찾기 위해 사원_X1 index scan
2. 사원_X1 index에서 읽은 ROWID로 사원 table access해 테이블 filter 조건으로 조건절 3번 사용
3. 조인 조건인 관리사원번호를 만족하는 index record를 찾기 위해 고객_X1 index를 range scan
4. 고객_X1 index에서 읽은 ROWID로 고객 테이블을 access해 테이블 filter 조건으로 조건절 4번 사용
```

- 그런데 1번부터 4번은 단계별로 완료한 뒤 진행이 아니라, 1번에서 찾아진 index record마다 반복되는 형태다.
- 아래 이미지를 보면 이해하기 쉬울 것이다.
- 사원 테이블에서 얻은 record가 3개인데, 사원의 사원번호가 10, 12, 15라고 해보자.
  - 그럼 c.관리사원번호가 10인 경우에 최종주문금액이 20,000이 넘는 고객의 record를 가져온다.
  - c.관리사원번호가 12인 경우에 최종주문금액이 20,000이 넘는 고객의 record를 가져온다.
  - c.관리사원번호가 15인 경우에 최종주문금액이 20,000이 넘는 고객의 record를 가져온다.
- 이렇게 outer에서 가져온 각 record에 맞는 고객의 record를 모두 종합하면 그게 NL join의 결과물이다.

<img src="/image/NL.jpg" />


- 위의 그림에서 튜닝포인트는 4가지다.
  - 사원 index를 읽은 후 사원 table에 access하는 횟수
    - 많은 random access가 발생했다면, 부서코드를 사원_X1 index에 추가하는 게 좋다.
  - 사원 table에서 조건을 만족하는 filtering된 record와 결합하는 고객_X1 index와 결합하는 join횟수
    - 부서코드 Z=123을 만족하는 건수가 3개였기에, 해당 갯수만큼 조인시도를 했다.
    - 조인을 위해 고객_X1 index를 탐색하는 range scan 횟수가 많을 수록 성능이 안 좋아진다.
  - 고객_X1 index를 읽고 고객 테이블 access하는 횟수
    - 많은 random access가 발생했다면, 최종주문금액 컬럼을 추가하는 방안을 고려해야 한다.
  - 맨 처음 access하는 사원_X1 index에서 얻은 range scan의 양
    - 처음에 range가 넓으면 1-3번이 모두 늘어날수밖에 없다.

<br />

- 이제 그럼 실제 튜닝을 해보자.
- 만약 위의 sql이 아래와 같은 row를 뱉었다면?
- 사원에서 index로 가져온 record가 2780개이므로, random access도 똑같다.
- 그런데 가져온 row의 갯수가 3개밖에 없다. 너무 많이 필터링된 것이다.
  - 이 경우 table filter 조건을 index에 포함해 random access를 줄이는 걸 고민해야 한다.
  - 그래서 부서코드를 추가했다.
```
|ID       |Operation                      |Name      |Rows     |Bytes     |Cost
------------------------------------------------------------------
0         |SELECT STATMEENT               |         |5         |58        |
1         | NESTED LOOPS                  |         |5         |58
2         |  TABLE ACCESS BY INDEX ROWID  |사원     |3          |20
3         |   INDEX RANGE SCAN            |사원_X1  |2780
4         |  TALBE ACCESS BY INDEX ROWID  |고객     |5          |76
5         |   INDEX RANGE SCAN            |고객_X1  |8
```

- 그랬더니 아래와 같이 바뀌었다.
- trace가 바뀐건 오라클 9iR2버전부터다.
  - cr은 논리 block 요청(버퍼에서 읽은 block)
  - pr은 디스크에서 읽은 block
  - pw는 디스크에다가 쓴 블록 수
- 사원_X1 index에서 읽은 index record가 table filtering 없이 그대로 읽은 셈이니 range scan의 비효율은 없었다.
  - 물론 이게 실제로 유효한 random access였는 지는 면밀히 살펴봐야 할 것이다.
- 그러나 2780개를 가지고 가서 고객 table과 join한 결과는 겨우 5 row다. 매우 비효율적인 join인 셈이다.
  - 이럴 때는 join 순서를 바꾸는 것을 고려해야 한다.
```
|ROW      |Operation                      
------------------------------------------------------------------
5         | NESTED LOOPS(cr=112 pr=34 pw=0 time==122 us)                  
2780      |  TABLE ACCESS BY INDEX ROWID OF 사원(cr=105 pr=32 pw=0 time=118 us)
2780      |   INDEX RANGE SCAN OF 사원_X1(cr=102 pr=31 pw=0 time=16)            
5         |  TALBE ACCESS BY INDEX ROWID OF 고객(cr=7 pr=2 pw=0 time=4 us)
8         |   INDEX RANGE SCAN OF 고객_X1(cr=5 pr=1 pw=0 time=0 us)          
```



## <span style="color:#802548">_NL 조인에서 조건절 잘 쓰기_</span>
- 아래 sql을 보자.
- 해당 sql을 보고 index를 어떻게 만들어야할까 생각했는가? 
- 만약 index를 PRA_HST_STC_N1 table에 (SALE_ORG_ID, STRD_GRP_ID, STRD_ID, STC_DT)로 만들려 했다면 실수한 것이다.
- 일단 전부 a를 왼쪽에 배치한거부터 매우 잘못되었다. 이러면 가시성이 매우 떨어질 수 밖에 없다.
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
AND b.STRD_GRP_ID = a.STRD_ID
ORDER BY a.STC_DT desc
``` 

## <span style="color:#802548">_sort merge 조인_</span>
- 해쉬 조인, NL 조인을 못 쓸 떄 쓰는 찌꺼기 조인이다.
  - 각 오라클 서버 프로세스에 할당된 메모리 영역을 PGA라고 한다.
  - 만약 PGA 공간이 작다면 Temp 테이블스페이스(디스크)를 이용한다.
  - PGA는 SGA와 다르게 공유되지 않는 독립적인 메모리 공간이라 래치가 불필요하다.
- 따라서 같은 양의 데이터를 읽어도 SGA 버퍼캐시에서 읽는 것보다 훨씬 빠르다.

<img src="/image/sga-pga.jpg" />

- 머지 조인은 아래와 같은 순서로 이뤄진다.
  - 소트
    - 양쪽 집합을 조인 컬럼 기준으로 정렬한다.
  - 머지
    - 정렬한 양쪽 집합을 서로 머지한다.
- 예시를 살펴보자.
```sql
SELECT /*+ ordered use_merge(c) */
      e.사원번호, e.사원명, e.입사일자
      , c.고객번호, c.고객명, c.전화번호, c.최종주문금액
FROM 사원 e, 고객 c
WHERE c.관리사원번호 = e.사원번호
AND e.입사일자 >= '19960101'
AND e.부서코드 = 'Z123'
AND c.최종주문금액 >= 20000
```

- 위의 sql은 풀면 아래와 같다.
- 쿼리로 가져온 PGA 영역에 할당된 Sort Area에 저장한다.
- 상대 table과 join되는 column인 사원번호로 정렬된다.
```sql
SELECT 사원번호, 사원명, 입사일자
FROM 사원
WHERE 입사일자 >= '19960101'
AND 부서코드 = 'Z123'
ORDER BY 사원번호
```


- 상대방 table에서도 결과집합을 가져온다.
- 마찬가지로 Sort Area에 저장한다.
- 상대 table과 join되는 column인 관리사원번호로 정렬된다.
```sql
SELECT 고객번호, 고객명, 전화번호, 최종주문금액, 관리사원번호
FROM 고객 c
WHERE 최종주문금액 >= 20000
ORDER BY 관리사원번호
```

- PGA에 저장한 사원 데이터를 스캔하면서 PGA에 저장한 고객 데이터와 join한다.
- 사실 join자체는 NL join의 process와 다른 것이 없다.
```sql
begin
  for outer in (select * from PGA_SORTED_사원)
  loop
    for inner in (SELECT * FROM PGA_SORTED_고객
                  WHERE 관리사원번호 = outer.사원번호)
    loop
      dbms_output.put_line(....);
    end loop;
  end loop;
end;
```

- 위와 같이 sort merge join은 사원 데이터를 기준으로 매번 고객 데이터를 full scan하진 않는다.
  - 고객 데이터가 정렬돼있으므로 join record가 시작되는 지점을 쉽게 찾을 수 있다.
  - 조인에 실패하는 record를 만나면 정렬돼있으므로 바로 멈추면 된다.
  - sort area에 저장한 데이터 자체가 index이므로 join컬럼에 index가 없어도 사용 가능하다.
- sort merge가 index scan을 하지 않아도 빠른 이유는 PGA에 저장되어 래치 획득 과정이 없기 때문이다.
- 거기다가 건건이 전부 순회하지도 않는다. 데이터를 가져올 때 index scan을 하게 되면?
  -  random access에 따른 부하는 피할 수 없다.
- 현재는 자주 쓰이지 않는데, hash join이 성능이 더 좋기 때문이다. 아래 같은 상황에서만 한정적으로 쓰인다.
  - 조인 조건식이 =가 아닌 대량 데이터 조인
  - 조인 조건식이 없는 크로스 조인

## <span style="color:#802548">_해시 조인_</span>
- 해시 조인은 sort meger join과 달리 sort by에 따른 부하가 없다.
- 또한 sort merge join과 달리 disk I/O에 대한 염려도 거의 없다.
  - 양쪽을 모두 PGA에 담지 않고 한쪽만 bulid input으로 PGA 메모리에 담기 때문이다.
- 해시 조인도 두 단계로 진행된다.
  - build: 작은 쪽 table을 읽어(build input) 해시 테이블(해시맵)을 생성한다.
    - 해시 테이블에는 join key와 sql에 사용된 컬럼이 모두 저장된다.
    - 사원번호만 저장하면 table access를 다시 해야하기 때문에 래치 획득 과정이 필요해진다.
    - 해쉬 조인의 장점을 날려먹는 것이기 때문에 join key만 저장되지 않는 것이다.
    - 보통 테이블 조건절 컬럼에 대한 카디널리티가 작은 테이블을 선택한다.
  - probe: 큰 쪽 테이블(probe input)을 읽어 해시 테이블을 탐색하면서 조인한다.
- 아래 sql 예시를 살펴보자.
```sql
SELECT /*+ ordered use_hash(c) */
        e.사원번호, e.사원명, e.입사일자
        c.고객번호, c.고객명, c.전화번호, c.최종주문금액
FROM 사원 e, 고객 c
WHERE c.관리사원번호 = e.사원번호
AND e.입사일자 >= '19960101'
AND e.부서코드 = 'Z123'
AND c.최종주문금액 >=20000
```

- 첫 단계인 build에;서 join컬럼인 사원번호를 해시 테이블 키값으로 사용한다.
```sql
SELECT 사원번호, 사원명, 입사일자
FROM 사원
WHERE 입사일자 >= '19960101'
AND 부서코드 ='Z123'
```


<img src="/image/hash-join-build-input.jpg" />


- 아래 조건에 해당하는 고객에티러를 읽어 앞서 생성한 해시 테이블을 탐색한다.
```sql
SELECT 고객번호, 고객명, 전화번호, 최종주문금액, 관리사원번호
FROM 고객
WHERE 최종주문금액 >= 20000
```


<img src="/image/hash-join-probe-input.jpg" />


- hash join이 빠른 이유는 sort merge join과 동일하다.
  - 해시 테이블을 PGA 영역에 생성하기 떄문이다.
  - 래치 획득 과정이 없다는 의미다. 
  - 거기다 disk I/O도 없으니 sort merge join보다 당연히 빠르다.



## <span style="color:#802548">_subquery_</span>
- 서브쿼리는 3가지가 있다.
  - nested subquery
    - where절
  - inline view
    - from 절
  - scalar subquery
    - select 절

## <span style="color:#802548">_nested subquery -unnest_</span>
- 최근의 optimizer는 쿼리변환부터 진행한다.
- 문제는 서브쿼리를 모듈별로 나눠 각각 최적화가 진행된다는 점이다.
- 따라서 효율적이지 못한 sql로 변환될 수도 있다. 그래서 서브쿼리를 unnest하는 것이다.
- nested subquery가 unnest 방식으로 처리되어야 optimizer가 더 효율적인 최적화 경로를 찾을 가능성이 높아진다.
  - unnest하는 순간 join으로 풀리는데, optimizer는 join 관련해서 최적화가 잘 이뤄졌기 때문이다.


- 그럼 unnest 방식일 때의 실행계획을 살펴보자.
```sql
SELECT c.고객번호, c.고객명
FROM 고객 c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
AND EXIST (
      SELECT /*+ unnest nl_sj */ 'x'
      FROM 거래
      WHERE 고객번호 = c.고객번호
      AND 거래일시 >= trunc(sysdate, 'mm')
    )
```

```
Execution plan
------------------------------------------
0     SELECT STATEMENT 
1       NESTED LOOPS (SEMI)
2         TABLE ACCESS (BY INDEX ROWID) OF '고객'
3           INDEX (RANGE SCAN) OF '고객_X01'
4         INDEX (RANGE SCAN) OF '거래_X01' 
```

- EXIST를 쓰게 되면 SEMI 조인으로 작동하며 사실상 NL join과 다르지 않다.
- 그리고 unnest라도 캐싱 기능이 적용되기 때문에 filter operation과 동일하다.
- 다른 점은 바로 driving 집합이 서브쿼리가 될 수 있다는 사실이다.
```sql
SELECT /*+ leading(거래@subq) use_nl(c) */ c.고객번호, c.고객명
FROM 고객 c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
AND EXISTS (
  SELECT /*+ qb_name(subq) unnest */ 'x'
  FROM 거래
  WHERE 고객번호 = c.고객번호
  AND 거래일시 >= trunc(sysdate, 'mm')
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

- 위의 실행계획에서 SORT UNIQUE가 뜬 이유는 서브쿼리를 unnest했기 때문이다.
- unnest하면서 서브쿼리쪽 집합에서도 메인쿼리 집합과 똑같은 고객번호가 생길 수 있다.
- 그래서 서브쿼리에 대해 sort unique를 수행하는 것이다.


<br />

- 참고로 unnest를 하려고 하는 경우, subquery 안에 rownum은 쓰면 안 된다.
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

## <span style="color:#802548">_nested subquery -filter_</span>

- unnest만 있는 건 아니고, filter방식도 존재한다.
  - 필터 오퍼레이션은 기본적으로 NL조인과 처리 루틴이 같다.
  - NL join과 Filter 자체가 다른 점은, Filter는 캐싱 기능이 있다는 점이다.
  - 또한 필터 서브쿼리는 join순서가 고정이다. 늘 메인쿼리가 driving table이다.
- 필터로 작동하는 서브쿼리의 경우, 서브쿼리 필터링을 먼저 처리하면 성능이 좋아지는 경우가 많다.
  - 이같이 서브쿼리의 필터링을 먼저 처리하는 기술을 subquery pushing이라고 한다.
- 먼저 서브쿼리 필터링을 맨 마지막에 처리하는 쿼리를 보자.
```sql
SELECT /*+ leading(p) use_nl(t) */ count(distinct p.상품번호), sum(t.주문금액)
FROM 상품 p, 주문 t
WHERE p.상품번호 = t.상품번호
AND p.등록일시 >= trunc(add_months(sysdate, -3), 'mm')
AND t.주문일시 >= trunc(sysdate - 7)
AND EXIST (
        SELECT 'x'
        FROM 상품분류
        WHERE 상품분류코드 = p.상품분류코드
        AND 상위분류코드 = 'AK'
)
```


<img src="/image/subquery-join.jpg" />

- Filter Operation이기 때문에 main query가 먼저 실행된다.
- main query는 join문이고, driving table인 상품 table scan부터 시작한다.
  - 우선 상품 table에 full scan으로 데이터를 가져온다. 그게 1000개다.
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
| 1000 |     TABLE ACCESS FULL(cr=95)                | 상품        
| 60000|       TABLE ACCESS BY INDEX ROWID(cr=38002) | 주문      
| 60000|         INDEX RANGE SCAN(cr=2002)           | 주문_PK   
|    1 |       TABLE ACCESS BY INDEX ROWID(cr=6)     | 상품분류
|    3 |         INDEX UNIQUE SCAN(cr=3)             | 상품분류_PK
--------------------------------------------------------------
```


- 블록은 몇개나 읽었을까?
- 총 읽은 data block의 갯수는 38103개(join 38,097개 + exists 6개)다.
  - join 과정에서 총 38,097개 data block을 읽었다.
    - 처음 상품 table full scan을 하며 95개의 cr을 읽는다.
    - 그 다음 join access를 한 뒤 주문 table을 index scan하며, index block을 2002개 읽는다.
    - 이 과정을 index lookup이라고 하는데, index lookup은 loop 과정에서 읽은 cr 집계에서는 제외된다.
    - index block에서 얻은 rowid를 통해 38002개의 data block을 읽는다.
  - 그리고 마지막에 exists 절에서 index scan을 하면서 3개의 index leaf 블록을 읽는다.
    - data block만 cr 집계에 포함되며, index leaf는 제외된다. 그러나 집계만 안 나온 거고 실제 index scan도 읽는 것에 포함된다. 
    - 읽은 leaf block에서 rowid를 가져와 6개의 data block을 읽었다.
- 38103개 block이면 1개 block 당 500개의 table record로 잡아도 엄청나게 많은 양이다.

<br />

- 서브쿼리 필터링을 먼저 처리한다면 어떨까? 읽은 블록이 확 줄어든다.
- 그러한 기술을 Pushing subquery라고 한다.
- 이 기능은 unnest되지 않은 subquery에서만 작동한다. 
- 따라서 push_subq hint는 no_unnest와 같이 사용해야 한다.
```sql
SELECT /*+ leading(p) use_nl(t) */ count(distinct p.상품번호), sum(t.주문금액)
FROM 상품 p, 주문 t
WHERE p.상품번호 = t.상품번호
AND p.등록일시 >= trunc(add_months(sysdate, -3), 'mm')
AND t.주문일시 >= trunc(sysdate - 7)
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
    - 처음 상품 table full scan을 하며 101개 cr을 읽는다.
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
| 150 |       TABLE ACCESS FULL 상품(cr=101)                 | 상품      
| 1   |         TABLE ACCESS BY INDEX ROWID 상품분류(cr=6)   | 상품분류
| 3   |           INDEX UNIQUE SCAN 상품분류_PK(cr=3)        | 상품분류_PK  
| 3000|       TABLE ACCESS BY INDEX ROWID 주문(cr=1802)      | 주문      
| 3000|         INDEX RANGE SCAN 주문_PK(cr=302)             | 주문_PK   
--------------------------------------------------------------
```

## <span style="color:#802548">_inline view -merge와 pushdown_</span>
- inline view에 쓸 때도 위와 같이 조건문이 아쉬울 때가 있다.
  - 분명히 어차피 전월 이후 가입한 고객을 필터링할 것이다.
  - 그런데도 inline view안에 해당 조건문이 없어 inline view는 그냥 scan을 진행한다.
```sql
SELECT c.고객번호, c.고객명, t.평균거래, t.최소거래, t.최대거리
FROM 고객 c,
        (SELECT 고객번호, avg(거래금액) 평균거래,
                min(거래금액) 최소거래, max(거래금액) 최대거래
        FROM 거래
        WHERE 거래일시 >= trunc(sysdate, 'mm') /**당월 발생한 거래 */
        group by 고객번호) t
WHERE c.가입일시 >= trunc(add_months(sysdate, -1) ,'mm') /**전월 이후 가입 고객 */
AND t.고객번호 = c.고객번호
```

- 실행계획은 아래와 같다.
- no merge라서 inline view로 진행되고, WHERE 조건문은 맨마지막에 filter로 진행된다.
- 사실 그 전에 filter로 먼저 작동했다면 많은 record를 애초에 읽을 필요가 없었을 것이다.

<img src="/image/inline-view-no-merge.jpg" />


- 그럴 떄 merge hint를 사용해준다.
```sql
SELECT c.고객번호, c.고객명, t.평균거래, t.최소거래, t.최대거래
FROM 고객 c,
      (SELECT /*+ merge */ 고객번호, avg(거래금액) 평균거래,
        min(거래금액) 최소거래, max(거래금액) 최대거래
        FROM 거래
        WHERE 거래일시 >= trunc(sysdate, 'mm')  /**당월 발생한 거래 */
        GROUP BY 고객번호) t
WHERE c.가입일시 >= trunc(add_months(sysdate , -1), 'mm') /**전월 이후 가입 고객 */
AND t.고객번호 = c.고객번호
```

- 그럼 실행계획이 아래와 같이 바뀐다.
  - view가 사라진 걸 볼 수 있다. 
  - merge hint가 적용되어 WHERE 조건문의 필터링 조건이 inline view에 적용된다.
  - 다만 위의 방식을 쓰면 group by를 마지막에 쓰기 때문에, 부분범위처리가 불가능하다.
  - join에 성공한 resultSet을 가지고 group by를 진행해야 하기 때문이다. 
  - 또한 driving table의 resultSet(여기선 전월 이후 가입한 고객)이 클 때는?
    - join record에 따른 상대 table access가 많아진다.
  - 마찬가지로 driven table의 resultSet(여기선 당월 거래)가 클 때는?
    - table access 이후 range scan양이 많아진다.
    - 따라서 이러한 경우 hash join으로 table scan을 진행하는 게 좋다.


<img src="/image/inline-view-merge-hint.jpg" />


- 기본적으론 inline view보다는 일반 table을 통한 join 형태로 변환해주는 게 대개 더 좋다.
- 그러나 driving table만 데이터가 너무 많을 때는 inline view를 활용하는 것도 방법이다.
<br />

- 그 외 방법으로는 join 조건 pushdown이 있다.
- merge랑 비슷하게 inline view로 조건절이 들어간다.
  - pushdown을 이용하면 join record 건건이 group by를 수행하기에, 중간에 멈출 수 있다.
  - 따라서 GROUP BY구가 있지만 부분범위처리가 가능하다.
  - hint는 no_merge push_pred다.
```sql
SELECT c.고객번호, c.고객명, t.평균거래, t.최소거래, t.최대거래
FROM 고객 c,
      (SELECT /*+ no_merge push_pred */ 고객번호, avg(거래금액) 평균거래,
        min(거래금액) 최소거래, max(거래금액) 최대거래
        FROM 거래
        WHERE 거래일시 >= trunc(sysdate, 'mm')  /**당월 발생한 거래 */
        GROUP BY 고객번호) t
WHERE c.가입일시 >= trunc(add_months(sysdate , -1), 'mm') /**전월 이후 가입 고객. 이게 inline view로 들어간다. */
AND t.고객번호 = c.고객번호
```

- view pushed predicate라는 실행계획이 추가된걸 볼 수 있다.
- 그럼 잘 작동한 것이다.
<img src="/image/pushdown-predicate.jpg" />


## <span style="color:#802548">_scalar subquery_</span>
- 스칼라 서브쿼리를 쓰지 않고 함수를 사용하면 record마다 호출된다.
- 당연히 dept table을 record마다 scan한다. 
- 더 큰 문제는 함수 호출 시 컨텍스트 스위칭이 일어난다는 점이다.
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
- 캐싱 효과를 사용하기 위해 함수를 스칼라 서브쿼리로 덮어버리기도 한다.
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
    - 예를 들어, 거래구분코드가 20개라면 캐싱에 충분하고, 캐싱이 제대로 작동할 것이다.
    - 만약 고객이 100만명인데, 고객ID로 join access를 쓴다면 100만개를 캐시에 저장해야 한다.
    - 하지만 캐시는 작아서 그만큼 저장하지 못하기 때문에 캐시를 탐색해봐야 없을 것이다.
    - 이러면 캐시 탐색 비용만 늘어나서 메모리/CPU 사용률만 높아진다.
```sql
SELECT 거래번호, 고객번호, 영업조직ID, 거래구분코드,
        (SELECT 고객명 FROM 고객 WHERE 고객번호 = t.고객번호) 고객명
FROM 거래 t
WHERE 거래일자 >= to_char(add_months(sysdate, -3), 'yyyymmdd')
```

- 캐싱 효과가 부작용을 일으키는 경우가 하나 더 있다.
- 메인쿼리 집합이 작은 경우다. 스칼라 서브쿼리는 쿼리 단위로 캐싱이 이뤄진다.
  - 메인쿼리 집합이 커야만 효과가 크다. 그래야 많이 재사용되기 때문이다.
  - 재사용되지도 않는데 귀한 캐시 영역에 넣었다가 바로 버리면 비효율적인 것이다.
  - 메인쿼리로 가져오는 record가 1개밖에 없다면 더더욱 그럴 것이다.
  - 그런데 1개에 관해 사용자정의 함수를 호출해 scalar subquery로 덮으면 오히려 성능이 떨어진다.
```sql
SELECT 계좌번호, 계좌명, 고객번호, 개설일자, 계좌종류구분코드, 은행계설여부, 은행연계여부
      ,(SELECT brch_nm(관리지점코드) from dual) 관리지점명
      ,(SELECT brch_nm(개설지점코드) from dual) 개설지점명
FROM 계좌
WHERE 고객번호 =:고객번호
```

- 아래는 스칼라 서브쿼리를 사용할 때의 실행계획이다.
```sql
SELECT c.고객번호, c.고객명
      ,(SELECT ROUND(avg(거래금액), 2) 평균거래금액
        FROM 거래
        WHERE 거래일시 >= trunc(sysdate, 'mm')
        AND 고객번호 = c.고객번호)
FROM 고객 c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
```

- 실행계획에서는 후행 step이 먼저 발동하는 쿼리다.
- 따라서 고객이 먼저 full scan이 된다. 즉 메인쿼리가 먼저 발동한다.
- 그 다음 거래 table이 index scan된다. scalar subquery는 나중에 실행되는 것이다.
```
EXECUTION PLAN
----------------------------------------------
0       SELECT STATEMNT 
1   0     SORT (AGGREGATE)
2   1       TABLE ACCESS (BY INDEX ROWID BATCHED) OF '거래'
3   2         INDEX (RANGE SCAN) OF '거래_X02' (INDEX)
4   0     TABLE ACCESS (FULL) OF '고객' (TABLE)
5   4       INDEX (RANGE SCAN) OF '고객_X01' (INDEX)
```


- scalar subquery로 두 개 이상의 값을 얻는 건 불가능하다.
- 아래와 같은 sql은 불가능하다는 의미다.
```sql
SELECT c.고객번호, c.고객명
      ,(SELECT avg(거래금액), min(거래금액), max(거래금액)
        FROM 거래
        WHERE 거래일시 >= trunc(sysdate, 'mm')
        AND 고객번호 = c.고객번호)
FROM 고객 c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
```

- 그렇다고 아래처럼 바꾸면 거래 테이블에서 같은 테이블을 반복해 읽게 된다.
```sql
SELECT c.고객번호, c.고객명
        ,(SELECT AVG(거래금액) FROM 거래
          WHERE 거래일시 >= trunc(sysdate, 'mm') AND 거래번호 = c.고객번호)
        ,(SELECT MIN(거래금액) FROM 거래
          WHERE 거래일시 >= trunc(sysdate, 'mm') AND 거래번호 = c.고객번호)
        ,(SELECT max(거래금액) FROM 거래
          WHERE 거래일시 >= trunc(sysdate, 'mm') AND 고객번호 = c.고객번호)
FROM 고객 c
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
```

- 따라서 사실 아래와 같은 방식이 많이 사용됐다.
```sql
SELECT 고객번호, 고객명
      , to_number(substr(거래금액, 1, 10))  평균거래금액
      , to_number(substr(거래금액, 11, 10)) 최소거래금액
      , to_number(substr(거래금액, 21))     최대거래금액
FROM (
  SELECT c.고객번호, c.고객명
        , (SELECT LPAD(AVG(거래금액), 10) || LPAD(MIN(거래금액), 10) || MAX(거래금액)
            FROM 거래
            WHERE 거래일시 >= trunc(sysdate, 'mm')
            AND 고객번호 = c.고객번호) 거래금액
  FROM 고객 c
  WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
)
```

- 11g부터는 pushdown이 도입돼 위와 같이 쓰지 않아도 된다.
- 아래와 같이 inline view를 쓰는 게 가능해져 더 직관적으로 바뀌었다.
```sql 
SELECT c.고객번호, c.고객명, t.평균거래, t.최소거래, t.최대거래
FROM 고객 c
    , (SELECT /*+ no_merge push_pred */
              고객번호, avg(거래금액) 평균거래
              , min(거래금액) 최소거래, max(거래금액) 최대거래
      FROM 거래
      WHERE 거래일시 >= trunc(sysdate, 'mm')
      GROUP BY 고객번호) t
WHERE c.가입일시 >= trunc(add_months(sysdate, -1), 'mm')
AND t.고객번호(+) = c.고객번호
```

- 위처럼 pushdown을 사용하면 실행계획은 아래와 같이 VIEW PUSHED PREDICATE가 뜨게 된다.


<img src="/image/view-pushdown.jpg" />


- scalar subquery는 병렬쿼리에선 엔간하면 쓰면 안된다.
  - 대량 데이터를 처리하는 병렬쿼리는 해시 조인이 효과적이다.
- 또한 scalar subquery는 NL join이 이뤄지는 것이다. 
  - 따라서 캐싱 효과가 작으면 random access의 부담이 있다.
- 12c부터는 scalar subquery가 효과가 없다면 unnest 해서 merge할 수도 있다.
  - /*+ unnest merge */를 통해 목적을 달성한다.
  - 만약 unnest merge를 해서 문제가 생겼다면, no_unnest hint를 주면 된다.
  - _optimizer_unnest_scalar_sq를 false로 놓는 것은 장기적으론 좋지 않은 선택이다.

<img src="/image/scalar-subquery-unnest-merge.jpg" />


## <span style="color:#802548">_DML -복구를 위한 redo로그_</span>
- DML을 수행할 때마다 Redo 로그를 생성한다.
- Redo 로그는 복구를 위해 기록된다.
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


## <span style="color:#802548">_DML -read consistency를 위한 undo로그_</span>
- DML을 수행할 때마다 undo 로그도 생성된다.
- undo로그의 목적은 롤백과 read consistency를 위한 목적이다.
- read consistency는 oracle이 DML을 안전하게 수행하기 위한 방식이다.
- 오라클은 MVCC모델을 쓰는데, select문은 항상 consistent모드로 읽는다.
- 반면에 DML을 consistent 모드로 record를 찾고, cuirrent 모드로 DML을 수행한다.
  - consistent모드로 DML문이 시작된 지점에 존재했던 데이터 블록을 찾는다.
  - consistent모드에서 얻은 rowid를 가지고 current모드로 원본 블록을 찾아서 갱신한다는 의미다.
- current모드는 디스크에서 캐시로 적재된 원본 블록을 현재상태 그대로 읽는 것이다.
- consistent 모드는 트랜잭션에 의해 변경된 블록을 만나면 쿼리가 시작된 지점으로 되돌려 읽는 방법이다.

<img src="/image/mvcc-consistent-select.jpg" />


## <span style="color:#802548">_DML -무결성 보호를 위한 Lock_</span>
- 오라클은 공유 리소스와 사용자 데이터를 보호할 목적으로 lock을 사용한다.
  - DML lock
  - DDL Lock
  - latch
  - buffer lock
  - library cache lock
  - pin
- 가장 중요한 것은 DML Lock이다. 
- DML Lock은 동시 트랜잭션에 대한 데이터의 무결성을 보호하는 핵심 기법이다.
- DML Lock은 두 가지로 나뉜다.
  - Table Lock
    - TM Lock이라고도 부른다.
    - table Lock은 모드가 여러가지인데, 선행 트랜잭션이 특정 모드가 되면 후행 트랜잭션은 그에 호환되어야 한다.
    - Table Lock은 후행 트랜잭션은 절대 접근못하게 막는 그런 개념이 아니다. 다음에 올 수 있는 모드를 제시하는 푯말에 가깝다.
    - 예를 들어 insert, update, delete, merge를 하려고 row lock을 설정한다면 table에 먼저 RX 모드 table lock을 설정해야 한다.
    - 호환이 안 되는 모드는 기다리던지(wait 3), 포기하던지 해야 한다.(nowait)
  - Row Lock
    - 두 개의 동시 transaction이 같은 row를 변경하는 것을 방지한다.
    - row lock은 모두 배타적 모드를 사용한다.
    - update와 delete를 할 때는 어떤 경우에도 row lock이 걸린다.
    - insert의 경우 unique index일 때만 row lock이 걸린다.
    - select의 경우 row lock이 없다. MVCC 모델이라 consistency 모드로 읽기 떄문이다.
    - mysql은 select에도 공유 lock을 사용하기에 DML과 SELECT도 서로 경합한다.
- lock에 따른 장애가 일어나기도 하는 데 아래 두가지 상황이 있다.
  - blocking
    - 선행 transaction이 설정한 lock 때문에 후행 transcation이 작업을 진행하지 못하고 멈춰 있는 상태다.
  - deadlock
    - DB Deadlock은 상대방이 소유하고 있는 Lock을 요청해서 작업의 처리를 진행하지 못하는 상태다.  
    - 이를 먼저 인지한 transaction이 롤백을 진행한다. 롤백이 되면 이제 blocking 상태로 전환된다.
    - blocking 상태에서 커밋을 할지, 롤백을 할 지 결정해야 대기를 지속하지 않는다.


<img src="/image/table-lock.jpg" />

- lock은 트랜잭션 격리수준, 레벨과 관련이 높다. 둘 모두 높아질수록 DML 성능이 나빠진다.
- 트랜잭션 격리성 수준은 4가지로 나뉜다. oracle은 read uncommited는 아예 지원하지 않는다.
- 격리성 수준을 올릴수록 lock을 오랫동안 유지한다.
  - read uncommitted
    - 트랜잭션에서 처리 중인, 아직 커밋되지 않은 데이터를 다른 트랜잭션이 읽는 것을 허용
    - Dirty Read, Non-Repeatable Read, Phantom Read 현상 발생
  - read commited
    - Dirty Read 방지 : 트랜잭션이 커밋되어 확정된 데이터만 읽는 것을 허용
    - DB2, SQL Server, Sybase의 경우 select를 shared Lock으로 구현. 하나의 레코드를 읽을 때 Lock을 설정하고 해당 레코드를 빠져 나가는 순간 Lock 해제
    - Oracle은 Lock을 사용하지 않고 쿼리시작 시점의 Undo 데이터를 제공하는 방식으로 구현
  - repeatable read
    - 선행 트랜잭션이 읽은 데이터는 트랜잭션이 종료될 때가지 후행 트랜잭션이 갱신하거나 삭제하는 것을 불허함으로써 같은 데이터를 두 번 쿼리했을 때 일관성 있는 결과를 리턴한다.
    - Phantom Read 현상은 여전히 발생함.( InnoDB는 발생하지 않음)
    - MySQL 의 InnoDB 에서 기본으로 사용되는 격리수준.
    - Oracle의 select for update가 여기에 해당.
  - serializable
    - 선형 트랜잭션이 읽은 데이터를 후행 트랜잭션이 갱신/삭제/삽입 모두 금지
    - 거의 쓰이지 않는다. 성능이 너무 안 좋아진다.
- lock 레벨은 4가지로 나뉜다. oracle은 lock 에스컬레이션은 사용하지 않는다.
  - row
  - block
  - extent
  - table
- lock수준을 떨어뜨리면 데이터 품질이 나빠진다. 
- lock 수준을 높이면 동시에 들어오는 데이터 처리가 원활하지 않다.
- 동시에 실행되는 트랜잭션 수를 최대화(고성능)하면서 DML 시 데이터 무결성을 유지(고품질)하는 게 동시성 제어다.


## <span style="color:#802548">_DML -LOCK을 푸는 열쇠 commit_</span>
- Lock에 걸린 경우 commit을 해야 lock이 풀리게 되므로 commit은 매우 중요한 역할을 한다.
- commit 모드는 아래와 같이 지정할 수 있다.


```sql
commit write immediate wait     -- LGWR이 로그버퍼를 파일에 기록했다는 완료 메시지를 받을 떄까지 대기
commit write immediate nowait;  -- LGWR을 기다리지 않고 다음 transaction 수행
commit write batch wait;        -- commit 명령을 받을 때마다 LGWR이 로그 버퍼를 파일에 기록
commit write batch nowait;      -- session 내부에 transaction data를 일정량 buffering했다가 일괄 처리
```


- commit은 아래와 같은 과정을 거쳐서 만들어진다.
  - DML 실행 -> redo로그버퍼에 변경사항 기록
  - 버퍼블록에서 데이터를 변경. 버퍼캐시에 없으면 데이터파일 읽기(disk I/O)
  - commit
  - LGWR process가 redo로그버퍼 내용 로그파일에 일괄 저장
  - DBWR(database writer) 프로세스가 변경된 버퍼블록을 데이터파일에 일괄 저장.

<img src="/image/dml-process.jpg" />

- 버퍼캐시가 휘발성이어서 redo 로그를 남긴다고 했다.
- 그런데 redo 로그도 휘발성 로그버퍼에 기록하면 영속성이 보장될까?
- 영속성이 보장된다. redo로그만 디스크에 기록되어있다면 영속성이 보장된다.
- commit 시점에 LGWR(log writer) 프로세스가 꺠어나기 때문이다.
  - 서버 프로세스가 commit record를 로그버퍼에 기록한다.
  - 서버 프로세스가 LGWR 프로세스에 신호를 보내고 wait 큐에서 sleep상태로 전환한다.
  - LGWR 프로세스가 로그 버퍼를 디스크에 기록하는 작업을 마친다.
  - LGWR 프로세스가 wait 큐에 대기중인 서버 프로세스에 완료 메시지를 전송한다.
  - 신호를 받은 서버 프로세스는 runnable 큐로 옮겨진 후 CPU를 할당받아 다음 작업을 이어간다.
- 위에서 보았듯 LGWR 프로세스가 redo로그를 기록하는 작업은 disk I/O기 때문에 생각보다는 느리다.
  - 트랜잭션을 너무 잘게 짜르면 disk I/O가 많아져 성능이 나빠진다.
  - 트랜잭션을 너무 길게 유지하면 undo로그 공간이 부족해져 시스템 장애가 올 수 있다.
- Spring에서는 @Transactional로 트랜잭션을 만드는데, 3개의 db조작 method를 모아서 하나의 method에 @Transactional을 달아준다.
  - 그럼 그 3개 중 하나만 잘못돼도 모두 rollback되는데, 이러한 작용을 할 때 transaction이 너무 긴 건 아닌지 고려해야 할 때도 있다.

## <span style="color:#802548">_DML -constraint 해제로 성능 향상_</span>
- 인덱스가 있으면 DML 성능이 더 떨어진다.
  - 테이블 data block에 추가하는 것에 더해 index leaf block에도 추가해야 하기 때문이다.
  - 그런데 index leaf block을 찾기 위해 수직적 탐색이 있어야 한다.
  - 거기다 인덱스는 정렬된 집합이라서 정렬순서를 맞춰줘야 해서 관련 operation이 모두 이뤄진다.
    - index가 PK만 있다면 100만건을 넣을 때 4초지만, index를 2개 더 넣으면 40초다.
    - index 중에서도 PK, FK제약이 DML 성능에 미치는 영향이 크다.
    - 사용자 정의 제약은 그정도로 큰 편은 아니다.

<img src="/image/index-dml.jpg" />


- 설명했듯 index와 무결성 제약은 DML 성능을 떨어뜨린다.
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

## <span style="color:#802548">_DML -array를 통한 batch insert로 성능 향상_</span>
- db call은 3가지 종류가 있다.
  - parse call
    - SQL 파싱과 최적화. 소프트파싱이면 최적화는 생략
  - execute call
    - SQL 실행. DML은 여기까지 끝
  - fetch call
    - resultSet을 네트워크로 전송하여 사용자가 볼 수 있게 함
- 떄로는 인입 경로에 따라 2개로 나누기도 한다.
  - user call
    - 네트워크를 경유해 DB외부에서 들어오는 call
    - 대부분 WAS에서 넘어오는 call이다.
  - recursive call
    - DB 내부에서 발생하는 call.
    - 주로 parse call에서 발생하는 데이터 딕셔너리조회
    - PL/SQL로 만들어진 함수/프로시저/트리거에 내장된 SQL 실행
- db call이 많을수록 당연히 성능이 느리다.
- 그 중에 user call은 특히나 성능에 미치는 영향이 크다.
  - WAS와 같은 LNA안에 있으면 낫지만, 다른 LAN이라면 특히 더 크다.
  - PL/SQL로 recursive call을 하면 29초인것도 Java로 user call하면 210초 걸린다.
  - 그래서 하나의 SQL에서 모든 business를 다 처리하는 게 중요하다.
- 그게 불가능하다면 array processing을 활용하는 게 좋다.
```java
public void  execute() throws Exception {
  int arraySize = 10000;
  long[] no     = new long [arraySize];
  long[] empno     = new long [arraySize];
  long[] ename  = new long [arraySize];
  long[] job  = new long [arraySize];
  long[] mgr    = new long [arraySize];
  long[] hiredate    = new long [arraySize];
  long[] sal    = new long [arraySize];
  long[] comm    = new long [arraySize];
  long[] deptno   = new long [arraySize];

  String SQLStmt = "select no, empno, ename, job, mgr" 
                  + ", to_char(hiredate, 'yyyymmdd hh24miss'), sal, comm, deptno "
                  + "from source";

  PreparedStatement st = con.prepareStatement(SQLStmt);
  st.setFetchSize(arraySize);

  ResultSet rs = st.executeQuery();

  int i = 0;
  while(rs.next()) {
    no [i] = rs.getLong(1); //i = 0이돼도 rs.next로 인해 어차피 다음 record를 받게 됨.
    empno [i] = rs.getLong(2);
    ename [i] = rs.getString(3);
    job[i] = rs.getString(4);
    mgr[i] = rs.getInt(5);
    hiredate[i] = rs.getString(6);
    sal[i] = rs.getLong(7);
    comm[i] = rs.getLong(8);
    deptno[i] = rs.getInt(9);
    
    i = i + 1
    if(i == arraySize) {
      insertTarget(i,no,empno,ename,job,mgr,hiredate,sal,comm,deptno);
      i = 0;
    }
  }

  //10,000개를 다 못채운 경우도 insert
  if(i > 0) {
    insertTarget(i,no,empno,ename,job,mgr,hiredate,sal,comm,deptno);
  }

  rs.close();
  st.close();
}
```

- insert 함수는 아래와 같다.
```java
public void insertTarget(int length
                        ,long[] p_no
                        ,long[] p_empno
                        ,String[] p_ename
                        ,String[] p_job
                        ,int [] p_mgr
                        ,String[] p_hiredate
                        ,long[] p_sal
                        ,long[] p_comm
                        ,int[] p_deptno) throws Exception {
  String SQLStmt = "insert into target "
                  + "(no, empno, ename, job, mgr, hiredate, sal, comm, deptno) "
                  + "values (?, ?, ?, ?, ?, to_date(?, 'yyyymmdd hh24miss'), ?, ?, ?)"                          

  PreparedStatement st = con.prepareStatement(SQLStmt);

  for (int i = 0; i <length; i++) {
    st.setLong (1, p_no [i]);
    st.setLong (2, p_empno [i]);
    st.setString (3, p_ename [i]);
    st.setString (4, p_job [i]);
    st.setInt (5, p_mgr [i]);
    st.setString (6, p_hiredate [i]);
    st.setLong (7, p_sal [i]);
    st.setLong (8, p_comm [i]);
    st.setInt (9, p_deptno [i]);
    st.addBatch();
  };
  
  st.executeBatch();
  st.close();
}
```

## <span style="color:#802548">_비효율적 update -scalar subquery_</span>
- 전통적인 update문으로는 다른 table과 join이 필요한 update를 할 때 select를 여러 번 하는 비효율이 발생한다.
```sql
UPDATE 고객 c
SET     최종거래일시 = (SELECT MAX(거래일시) FROM 거랙
                        WHERE 고객번호 = c.고객번호
                        AND 거래일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
        , 최근거래횟수 = (SELECT COUNT(*) FROM 거래
                          WHERE 고객번호 = c.고객번호
                          AND 거래일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
        , 최근거래금액 = (SELECT SUM(거래금액) FROM 거래
                          WHERE 고객번호 = c.고객번호
                          AND 거래일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
WHERE exists (SELECT 'x' FROM 거래
              WHERE 고객번호 = c.고객번호
              AND 거래일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))                        
```

- 약 4번의 SELECT를 하는 걸 2번의 SELECT로 고칠 수도 있다.
- 다만 여기도 총고객수가 엄청나게 많다면 버벅거릴 수 있다.
```sql
UPDATE 고객 c
set (최종거래일시, 최근거래횟수, 최근거래금액) =
    (SELECT MAX(거래일시), count(*), sum(거래금액)
      FROM 거래
      WHERE 고객번호 = c.고객번호
      AND 거래일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
WHERE EXISTS (SELECT 'x' FROM 거래
              WHERE 고객번호 = c.고객번호
              AND 거래일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
```

- 그래서 총고객 수가 많다면 아래와 같이 바꿔줄 수도 있다.
- exists 서브쿼리를 hash semi join으로 바꿨다.
```sql
UPDATE 고객 c
set (최종거래일시, 최근거래횟수, 최근거래금액) =
    (SELECT MAX(거래일시), count(*), sum(거래금액)
      FROM 거래
      WHERE 고객번호 = c.고객번호
      AND 거래일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
WHERE EXISTS (SELECT /*+ unnest hash_sj */'x' FROM 거래
              WHERE 고객번호 = c.고객번호
              AND 거래일시 >= TRUNC(ADD_MONTHS(sysdate, -1)))
```


## <span style="color:#802548">_효율적 update -join view_</span>
- 수정 가능 조인 뷰를 활용하면 참조 테이블과 두 번 조인하는 비효율마저도 없앨 수 있다.
- 단, 수정 가능 조인 뷰의 경우, 1 : M 집합 중 M 집합만 DML이 허용된다.
- 또한 1쪽 집합에 반드시 unique index 설정이 있어야만 한다. 
- 즉 키보존 테이블이어야 한다는 의미다.
  - 키보존 테이블이란view에 rowid를 제공할 수 있는 unique index를 가진 column이다.
- 다만 아래 쿼리는 12c 이상 버전부터만 실행된다.
```sql
UPDATE
  (SELECT /*+ ORDERED USE_HASH(c) no_merge(t) */
    c.최종거래일시, c.최근거래횟수, c.최근거래금액
    , t.거래일시, t.거래횟수, t.거래금액
  FROM (SELECT 고객번호
              , MAX(거래일시) 거래일시
              , COUNT(*) 거래횟수
              , SUM(거래금액) 거래금액
        FROM 거래
        WHERE 거래일시 >= TRUNC(ADD_MONTHS(sysdate, -1))
        GROUP BY 고객번호
        ) t
  )
  SET 최종거래일시 = 거래이릿
    , 최근거래횟수 = 거래횟수
    , 최근거래금액 = 거래금액
```

- 11g에서는 위의 update를 사용할 수가 없고, merge into문으로 바꿔줘야 한다.
- 12c 이상 버전부터는 unique index가 없어도 inline view에 group by를 쓰면 에러가 나지 않는다.
- GROUP BY를 한 집합과 조인한 테이블은 key가 보존된다는 점을 오라클에서 인정한 것이다. 

## <span style="color:#802548">_효율적 update -merge_</span>
- 전통적 방법은 update를 할 때 table을 여러번 scan해야 하는 문제가 있었다.
- 아래와 같이 만들면 최소 2번은 scan해야 한다.
```sql
UPDATE dept d
SET d.avg_sal = (
    SELECT ROUND(AVG(e.sal), 2) 
    FROM emp e 
    WHERE e.deptno = d.deptno
)
WHERE EXISTS (
    SELECT 1 
    FROM emp e2 
    WHERE e2.deptno = d.deptno
);
```
- 그래서 위에서 소개한 join view가 등장했다. 
- join view를 통해 scan을 여러번 하는 비효율은 해결했다.
```sql
UPDATE
    (SELECT 
            d.DEPTno, d.avg_sal as d_avg_sal, e.avg_sal as e_avg_sal
      FROM (SELECT
            deptno, round(avg(sal), 2) avg_sal
            FROM emp 
            GROUP BY deptno) e
            , dept d
      WHERE d.deptno = e.deptno)
set d_avg_sal = e_avg_sal;
```

- 그러나 복잡했다. 이를 대체하기 위해 merge statement가 등장했다.
- 이를 더 간단하게 대체하면 아래와 같이 바뀐다.
```sql
merge into dept d
using (SELECT deptno, round(avg(sal), 2) avg_sal FROM emp GROUP BY deptno) e
ON (d.deptno = e.deptno)
WHEN MATCHED THEN UPDATE SET d.avg_sal = e.avg_sal;
```

- 다만 데이터 검증을 merge 안에 포함해서 쓰면 더 복잡해진다.
- ROWID를 사용했다고 해도 ROWID는 물리 포인터가 아니기 떄문에 성능과 무관하다.
- 또한 검증용 SELECT에서 1번 scan하고, MERGE 할 때 left outer join을 위해 1번 scan하니 2번 scan하게 된다.
```sql
MERGE INTO EMP T2
USING (SELECT T.ROWID AS RID, S.ENAME
        FROM EMP T, EMP_SRC S
        WHERE T.EMPNO = S.EMPNO
        AND T.ENAME <> S.ENAME) S
ON (T2.ROWID = S.RID)
WHERE MATCHED THEN UPDATE SET T2.ENMAE = S.ENMAE;
```

- 그럴땐 1번 SCAN하는 join view update를 하는 게 낫다.
- 그게 읽기도 더 간단하다.
```sql
UPDATE (
  SELECT S.ENAME AS S_ENAME, T.ENAME AS T_ENAME
  FROM EMP T, EMP_SRC S
  WHERE T.EMPNO = S.EMPNO
  AND T.ENAME <> S.ENAME
)
SET T_ENAME = S_ENAME;
```

- merge문의 기본 구성은 아래와 같다.
```sql
MERGE INTO CUSTOMER t using customer_delta s on (t.cust_id = s.cust_id)
WHEN MATCHED THEN update
  SET t.cust_nm = s.cust_nm, t.email = s.email, ..
WHEN NOT MATCHED THEN insert
  (cust_id, cust_nm, email, ...) values
  (s.cust_id, s.cust_nm, s.email, s.tel_no, s.region, s.addr....)
```

- 두 개 중 하나만 선택해서 할 수도 있다.
```sql
MERGE INTO CUSTOMER t using customer_delta s on (t.cust_id = s.cust_id)
WHEN MATCHED THEN update
  SET t.cust_nm = s.cust_nm, t.email = s.email, ..
```
```sql
MERGE INTO CUSTOMER t using customer_delta s on (t.cust_id = s.cust_id)
WHEN NOT MATCHED THEN insert
  (cust_id, cust_nm, email, ...) values
  (s.cust_id, s.cust_nm, s.email, s.tel_no, s.region, s.addr....)
```

- on 외에도 다른 조건절을 where로 걸 수도 있다.
- 이미 저장된 데이터를 조건에 따라 지울 수도 있다.
```sql
MERGE INTO CUSTOMER t using customer_delta s on (t.cust_id = s.cust_id)
WHEN MATCHED THEN 
update SET t.cust_nm = s.cust_nm, t.email = s.email, ..
DELETE WHERE t.withdraw_dt is not null
WHEN NOT MATCHED THEN insert
  (cust_id, cust_nm, email, ...) values
  (s.cust_id, s.cust_nm, s.email, s.tel_no, s.region, s.addr....)
```



## <span style="color:#802548">_효율적 INSERT -배치용 direct path I/O_</span>
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


- direct path I/O를 활용한다면 병렬 DML을 같이 활용하면 효과가 훨씬 좋아진다.
- 다만 exclusive TM Lock이 걸리므로, transaction이 빈번한 주간에는 사용하면 안 된다.
- append hint를 활용하면 direct path I/O를 이용해 INSERT할 수 있다.
- 하지만 update, delete의 경우는 append hint가 무의미하다.
```sql
ALTER session enable parallel dml;

INSERT /*+ parallel(c 4)  */ into 고객 c
SELECT /*+ parallel(o 4) */ * FROM 외부가입고객 o;
```



## <span style="color:#802548">_table partition -range partition_</span>
- 파티셔닝은 테이블 또는 index data를 특정 컬럼(파티션 키) 값에 따라 별도 segment에 나눠 저장하는 것을 의미한다.
- 파티션이 좋은 이유는 아래와 같다.
  - 관리의 측면에서 파티션 단위로 백업, 추가 ,삭제 등이 일어나면 가용성이 높다.
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


## <span style="color:#802548">_table partition -hash partition_</span>
- hash partition 은 PK처럼 변별력이 좋고 데이터 분포가 고른 컬럼을 파티션 기준으로 선정해야 한다.
- 반드시 = 조건으로 풀려야 한다.


## <span style="color:#802548">_table partition -list partition_</span>
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
  - 오라클이 자동 관리하며, table partition 구성을 변경해도 index 재생성 안해도 됨
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

CREATE INNDEX 주문_X04 on 주문 (고객ID, 배송일자);
```

- 인덱스 파티션과 관련해 중요한 제약은 아래와 같다.
- unique index(주로 PK)를 파티셔닝하려면, 테이블 파티션 키가 모두 index 구성 컬럼이어야한다는 점이다.
- 만약 PK index 키가 주문번호인데, 파티션 키는 주문일자라면?
  - 주문번호가 123456인 주문 레코드를 입력하려면, 중복값이 있는지 확인하려고 인덱스 파티션을 모두 탐색한다.
  - 주문번호가 123456인 레코드는 어떤 파티션에든 입력될 수 있기 때문이다.
  - 따라서 PK 인덱스 키를 (주문일자, 주문번호)로 해줘야 한다.
  - 게다가 레코드를 입력하고 커밋하기 전까지, 다른 트랜잭션이 같은 주문번호로 다른 파티션에 입력하는 현상까지 막아야 한다.
  - 그러면 추가적인 lock 메커니즘까지 필요해서 느려진다.

## <span style="color:#802548">_partition exchange-data exchange_</span>
- 테이블이 파티셔닝되어있고, 인덱스도 다행히 로컬 파티션이라면, 임시 테이블을 만들어 원본 파티션과 바꿔치기하기도 한다.
- 만약 상태코드가 추가되면서 기존 상태코드도 바꾸기로 결정했다고 해보자.
- 그럼 기존의 상태코드들을 모두 바꿔야 할 것이다. 그런데 그냥  update하기에는 index 구조 등 시간이 오래걸린다.
- 따라서 만약 파티션 테이블이라면 미리 임시 파티션 테이블을 바꾼 상태코드로 복제해서 만든다. 
- 프로세스는 아래와 같이 진행하면 된다.
```
1. 임시 테이블을 생성한다. 가능하면 nologging 모드
2. 거래데이터를 읽어 임시 테이블에 insert하면서 상태코드 값을 수정한다.
3. 임시 테이블에 원본 테이블과 같은 구조로 index를 insert한다. 가능하면 nologging 모드
4. 2014년 12월 파티션과 임시 테이블을 exchange한다.
5. 임시 테이블을 drop한다.
6. 만약 nologging모드였다면, 파티션을 logging 모드로 전환한다.
```


- sql로는 아래와 같다.
```sql
1. CREATE TABLE 거래_t nologging as SELECT * FROM 거래 WHERE 1 = 2;

2. INSERT /*+ append */ into 거래_t
SELECT 고객번호, 거래일자, 거래순번, (CASE WHEN 상태코드 <> 'ZZZ' THEN 'ZZZ' ELSE 상태코드 END) 상태코드
FROM 거래
WHERE 거래일자 < '20150101'

1. CREATE UNIQUE INDEX 거래_t_pk on 거래_t (고객번호, 거래일자, 거래순번) nologging;
CREATE INDEX 거래_t_x1 on 거래_t(거래일자, 고객번호) nologging;
CREATE INDEX 거래_t_x2 on 거래_t(상태코드, 거래일자) nologging;

1. ALTER TABLE 거래
EXCHANGE PARTITION p201412 WITH TABLE 거래_t
INCLUDING INDEXES WITHOUT validation;

1. DROP TABLE 거래_t;

2. ALTER TABLE 거래 MODIFY partition p201412 logging;
ALTER TABLE 거래_pk MODIFY partition p201412 logging;
ALTER TABLE 거래_x1 MODIFY partition p201412 logging;
ALTER TABLE 거래_x2 MODIFY partition p201412 logging;
```


## <span style="color:#802548">_partition truncate- data delete_</span>
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
ALTER TABLE 거래 DROP PARTITION p201412;
```

- 만약 조건절이 복잡하다면?
- 임시 테이블(거래_t)를 생성하고, 남길 데이터만 복제한다.
- 프로세스는 아래와 같다.
```
1. 임시 테이블을 생성하고, 남길 데이터만 복제한다.
2. 삭제 대상 테이블 파티션을 truncate한다.
3. 임시 테이블에 복제해 둔 데이터를 원본 테이블에 입력한다.
4. 임시 테이블을 drop한다.
```

- sql로는 아래와 같다.
```sql
1. CREATE TABLE 거래_t
AS 
SELECT *
FROM 거래
WHERE 거래일자 < '20150101'
AND 상태코드 = 'ZZZ'

2. ALTER TABLE 거래 TRUNCATE PARTITION p201412;

3. INSERT INTO 거래
SELECT * FROM 거래_t;

4. DROP TABLE 거래_t;
```


- 서비스 중단 없이 파티션을 drop 또는 truncate하려면 아래 조건을 모두 만족해야 한다.
  - 파티션키와 커팅 기준 컬럼일 일치
    - 파티션 키와 조건절 컬럼이 모두 신청일자
  - 파티션 단위와 커팅 주기가 일치
    - 월단위 파티션을 월 주기로 조건절 걸음
  - 인덱스가 모두 로컬 파티션 인덱스
    - 파티션 키가 PK 구성에 포함되어야 함.


## <span style="color:#802548">_partition -data insert_</span>
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


## <span style="color:#802548">_동시성 제어_</span>
- 비관적 동시성 제어와 낙관적 동시성 제어로 나뉜다.
  - 비관적 동시성 제어
    - 사용자들이 같은 데이터를 동시에 수정할 것으로 가정한다.
    - 한 사용자가 데이터를 읽는 시점에 lock을 걸고 조회/갱신처리가 완료될때까지 유지
    - 첫 사용자가 transaction을 완료할 때까지 해당 data 수정이 불가능
    - 주로 select for update로 구현한다. 특히 무한대기에 이르지 않게 for update wait 3를 준다.
    - 3초 뒤에도 update가 이뤄지지 않으면 oracle이 exception을 던지고, Spring에서도 exception을 인식한다.
    - 대신 Spring에서 exception을 인식하는 예외처리를 반드시 사용자가 알 수 있게 해줘야 한다.
    - select는 DML과 서로 양립가능하기 때문에 select for update가 걸려도 다른 사용자가 select하는 데는 문제가 없다.
    - 다만 실제 JPA나 mybatis에서는 method 단위로 tranasction이 묶이기에 실제로 사용하기는 어렵다.
    - 특히 webpage의 경우, fetch를 위한 select와 수정을 위한 udpate가 다른 tranasction이다.
    - 이럴 땐 낙관적 동시성 제어를 사용하는 게 더 수월하다.
  - 낙관적 동시성 제어
    - 사용자들이 같은 데이터를 동시에 수정하지 않을 것으로 가정한다.
      - 한 사용자가 데이터를 읽는 시점에 lock을 설정하지 않는다.
      - 대신 수정 시점에 앞서 읽은 데이터가 다른 사용자에 의해 변경되었는지 검사 필요
      - 이전의 값을 받아서 그대로 같이 update를 쳐주는데 count가 0이면 다른 사용자에 의해 변경됨을 알린다.
```sql
SELECT 적립포인트, 방문횟수, 최근방문일시, 구매실적 into :a, :b, :c, :d
FROM 고객
WHERE 고객번호 = :cust_num;

UPDATE 고객
SET 적립포인트 = :적립포인트
AND 적립포인트 = :a
AND 방문횟수 = :b
AND 최근방문일시 = :c
AND 구매실적 = :d

if sql%rowcount = 0 THEN
  alert('다른 사용자에 의해 변경되었습니다.');
end if;
```


- 만약 최종변경 일시를 관리한다면 아래와 같이 최종변경일시만 검색해주면 된다.
```sql
SELECT 적립포인트, 방문횟수, 최근방문일시, 구매실적 into :a, :b, :c, :d
FROM 고객
WHERE 고객번호 = :cust_num;

UPDATE 고객 SET 적립포인트 = :적립포인트, 변경일시 = SYSDATE
WHERE 고객번호 = :cust_num
AND 변경일시 = :mod_dt;

if sql%rowcount = 0 THEN
  alert('다른 사용자에 의해 변경되었습니다.');
end if;
```

- 아니면 update 전에 select for update를 할 수도 있다.
- 대신 비관적 lock과 달리 무조건 nowait 옵션을 써야 한다.
- 대신 Spring에선 하나의 transaction으로 묶이게 @Transactional을 달아줘야 된다.
```sql
--method1
SELECT 고객번호
FROM 적립
WHERE 고객번호 = :cust_num
AND 변경일시 = :mod_dt
FOR UPDATE NOWAIT;

if(rowCount > 0) {
  method2();
}
--method2
update 고객번호
SET 적립포인트 = :적립포인트
WHERE 고객번호 = :cust_num
```

- 특히 상품조회의 경우, 가격은 반드시 비관적이든 낙관적이든 lock을 구현해야만 한다.
- 아래와 같이 주문을 구현해서는 안 된다. 상품가격을 상품 공급업체가 결정하기 때문이다.
```sql
INSERT INTO 주문(상품코드, 고객ID, 주문일시, 상점번호, ...)
values(#{상품코드}, #{고객ID}, #{주문일시}, #{상점번호}, ...)
```

- 따라서 아래와 같이 table에 있는 값을 select해오는 형태로 구현되어야 한다.
- 주문을 하려고 주문 페이지에 들어오고 주문을 누르려는 직전에 상품 공급업체가 가격을 바꿀수도 있다.
- 그래서 웹페이지에서 본 가격을 지닌 상품이 있는지 판단하기 위한 조건절에도 넣어줘야 한다.
```sql
INSERT INTO 주문
SELECT :상품코드, :고객ID, :주문일시, :상점번호, ...
FROM 상품
WHERE 상품코드 = :상품코드
AND 가격 = :가격 --주문을 시작한 시점 가격

if sql%rowcount = 0 THEN
  alert('상품가격이 변경되었습니다.');
end if;
```

- 낙관적 동시성 제어의 예시를 하나 더 살펴보자.
- 관리자가 메인팝업을 관리하는데, 복수의 관리자가 있어 웹페이지에서 같은 메인팝업 row를 update하려고 한다.
- 이 경우, 선행 트랜잭션이 바꾼 값을 확인할 수 있는 변경일자를 활용하면 낙관적 동시성 제어가 쉽다.
- changed_date는 처음에 원본을 select해서 얻은 변경일자기 때문에, 만약 누군가 바꿔놨다면 해당 일자에 row가 존재하지 않는다.
- 따라서 update는 0 row가 되고, 그에 따라 0 row udpate인 경우에는 화면에 이미 수정된 내용임을 알려준다.
```sql
update 
        mainpopup
set main_display_yn = #{main_display_yn}
.
.
.
where board_position = #{board_position}
and changed_date = #{changed_date} -- 낙관적 동시성 제어를 위해 추가한 조건절
```

```java
if(rowCount == 0) {
  alert('이미 수정된 내용입니다. 새로고침한 뒤 시도해주세요');
}
```

- 비관적 동시성 제어는 성능을 떨어뜨리기 때문에 select for update는 자주 쓰이지 않지만, 데이터 무결성을 위해 필요하면 꼭 써야한다.
- 부담된다면 처음엔 낙관적 동시성 제어를 하고, 데이터가 변경된게 발견된다면 롤백하고 다시 시도할 때 비관적 동시성 제어를 사용하는 방식이다.
- 혹시 select를 join으로 가져온다면, for update를 하게 되면 양쪽 table에 모두 lock이 걸린다.
- 그럴 때 한쪽 table에만 lock을 걸고 싶다면 아래와 같이 해줄 수 있다. 주문 table에만 lock이 걸린다.
```sql
SELECT b.주문수량
FROM 계좌마스터 a, 주문 b
WHERE a.고객번호 = :cust_no
AND b.계좌번호 = a.계좌번호
AND b.주문일자 = :ord_dt
FOR UPDATE OF b.주문수량
```

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
  - 시퀀스 오브젝트는 오라클 내부에서 관리하는 채번 테이블이다.
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
  

## <span style="color:#802548">_빠른 INSERT에 따른 INDEX LEAF BLOCK 경합_</span>
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


## <span style="color:#802548">_Oracle hint 사용법_</span>

## <span style="color:#802548">_execution plan 읽는법_</span>
- oracle에서 실행계획을 보는 방법은 아래와 같다.
```sql
EXPLAIN PLAN FOR [query문]

SELECT * FROM TABLE(DBMS_XPLAN.DISPLAY)
```


- 실행계획을 이해하기 위해 위에서 썼던 예제를 하나 가져왔다.
```sql
SELECT /*+ leading(p) use_nl(t) */ count(distinct p.상품번호), sum(t.주문금액)
FROM 상품 p, 주문 t
WHERE p.상품번호 = t.상품번호
AND p.등록일시 >= trunc(add_months(sysdate, -3), 'mm')
AND t.주문일시 >= trunc(sysdate - 7)
AND EXIST (
        SELECT 'x'
        FROM 상품분류
        WHERE 상품분류코드 = p.상품분류코드
        AND 상위분류코드 = 'AK'
)
```


<img src="/image/subquery-join.jpg" />

- Filter Operation이기 때문에 main query가 먼저 실행된다.
- main query는 join문이고, driving table인 상품 table scan부터 시작한다.
  - 우선 상품 table에 full scan으로 데이터를 가져온다. 그게 1000개다.
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
|   1  | SORT AGGREGATE(cr=38103)                    |              8번
| 3000 | FILTER(cr=38103)                            |              7번
| 60000|   NESTED LOOPS(cr=38097)                    |              2번  
| 1000 |     TABLE ACCESS FULL(cr=95)                | 상품         1번   
| 60000|       TABLE ACCESS BY INDEX ROWID(cr=38002) | 주문         4번
| 60000|         INDEX RANGE SCAN(cr=2002)           | 주문_PK      3번
|    1 |       TABLE ACCESS BY INDEX ROWID(cr=6)     | 상품분류      6번
|    3 |         INDEX UNIQUE SCAN(cr=3)             | 상품분류_PK   5번
--------------------------------------------------------------
```


- 블록은 몇개나 읽었을까?
- 총 읽은 data block의 갯수는 38103개(join 38,097개 + exists 6개)다.
  - join 과정에서 총 38,097개 data block을 읽었다.
    - 처음 상품 table full scan을 하며 95개의 cr을 읽는다.
    - 그 다음 join access를 한 뒤 주문 table을 index scan하며, index block을 2002개 읽는다.
    - 이 과정을 index lookup이라고 하는데, index lookup은 loop 과정에서 읽은 cr 집계에서는 제외된다.
    - index block에서 얻은 rowid를 통해 38002개의 data block을 읽는다.
  - 그리고 마지막에 exists 절에서 index scan을 하면서 3개의 index leaf 블록을 읽는다.
    - data block만 cr 집계에 포함되며, index leaf는 제외된다. 그러나 집계만 안 나온 거고 실제 index scan도 읽는 것에 포함된다. 
    - 읽은 leaf block에서 rowid를 가져와 6개의 data block을 읽었다.
- 38103개 block이면 1개 block 당 500개의 table record로 잡아도 엄청나게 많은 양이다.

- 아래와 같이 되어 있으면 index range scan을 먼저하고, 그 뒤에 random access가 이뤄지는 것이다.
```
|    1 |       TABLE ACCESS BY INDEX ROWID(cr=6)     | 상품분류      6번
|    3 |         INDEX UNIQUE SCAN(cr=3)             | 상품분류_PK   5번
```

- 아래와 같이 NL join이면 driving table부터 쿼리가 실행된다.
- 즉 상품 table full scan이 첫 시작이다.
- 그리고 NL이 일어나 join access를 한다.
- join access에 해당하는 record를 찾기 위해 drivent table에서 index scan이 실행된다.
```
NESTED LOOPS(cr=38097)                    |               
  TABLE ACCESS FULL(cr=95)                | 상품            
    TABLE ACCESS BY INDEX ROWID(cr=38002) | 주문         
      INDEX RANGE SCAN(cr=2002)           | 주문_PK      
```

- 위에서 보는 cr은 consistent read라 하여 버퍼캐시를 읽는 행위다.
- 즉 메모리에서 data block을 가져오는 행위였다.
- 하지만 disk에서 가져오는 경우도 있다. 아래는 top n stopkey와 top n sort 모두 실패한 사례다.
```sql
SELECT *
FROM (
  SELECT ROWNUM no, a.*
  FROM (
    SELECT 거래일시, 체결건수, 체결수량, 거래대금
    FROM 종목거래
    WHERE 종목코드 = 'KR123456'
    AND 거래일시 >= '20180304'
    ORDER BY 거래일시
      ) a 
    )
WHERE no BETWEEN (:page - 1) * 10 + 1 and (:page * 10)
```

- pr과 pw는 각각 디스크에서 읽고 쓰는 행위다.
- disk I/O가 발생했으므로 굉장히 느려졌다는 의미다.
```
STATEMENT
  VIEW (cr=690 pr=698 pw=698)
    COUNT (cr=690 pr=698 pw=698)
      VIEW (cr=690 pr=698 pw=698)
       SORT ORDER BY (cr=690 pr=698 pw=698)
        TABLE ACCESS FULL 종목거래(cr=690 pr=0 pw=0)
```


## <span style="color:#802548">_execution plan 지시어_</span>



## <span style="color:#802548">_trace파일 만드는법_</span>
- 이를 측정하기 위한 도구로 버퍼캐시 히트율(BCHR)이 있다. 
  - 아래와 같은 sql로 확인해볼 수 있다.
```sql
alter session set sql_trace = true;
select * from emp where empno = 7900;
alter session set sql_trace = false;

select value from v$dialog_info where name = 'Diag Trace'; /** /oracle/diag/rdbms/ora11g/trace에 트레이스 파일이 생성된다. */
select value from v$diag_info where name = 'Default Trace File'; /** trace file 경로가 나온다. */

tkprof/**파일 경로로 이동하여 해당 db가 깔린 리눅스 서버에서 유틸리티 사용 */
tkprof [파일이름] report.prf sys=no
```

- 위와 같은 과정을 거쳐 보기 쉬운 trace파일을 만들었다.
- 그럼 여기서 Query와 Current를 합친게 논리 I/O다. 여기선 1351677 + 12367이다.
- 물리 I/O는 disk다. 601456이다. 물리 I/O가 엄청 많은 것이니 폐급이다.
- 논리 I/O에는 물리 I/O가 이미 포함되어있다. 버퍼캐시에 적재하고서 디스크를 읽기 때문이다.
```
Call    |Count      |Cpu Time         |Elapsed Time         |Disk       |Query        |Current        |Rows
Parse   |1          |   0.000         |     0.001           |0          |0            |0              |0
Execute |1          |   0.100         |     0.006           |0          |0            |0              |0    
Fetch   |2          |  138.600        |   1746.630          |601458     |1351677      |12367          |1
-----------------------------------------------------------------------------------------------------------
Total   |4          |  138.690        |   1746.637          |601458     |1351677      |12367          |1
```

