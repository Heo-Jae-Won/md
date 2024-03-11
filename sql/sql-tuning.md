## <span style="color:#802548">_sql 파싱_</span>
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
create procedure LOGIN_JAEWON();
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



## <span style="color:#802548">_디스크 I/O_</span>
- SQL이 느린 이유를 하드파싱에서 찾는 건 현실적이지 않다.
- 대부분의 db connection library는 바인드변수 방식으로 구현했기 때문이다.
- 현실적으로 SQL이 행하는 I/O가 문제기 때문이다.
- 다시말하면 디스크 I/O는 블록킹이라 그동안 프로세스가 잠든다는 게 문제다.
- I/O가 일어나면 CPU가 반환되고 프로세스가 대기 큐에서 잠을 자기 때문이다.
- I/O Call은 Single Block I/O 기준 평균 10ms, SAN 스토리지는 4-8ms, SSD 스토리지는 1ms다.
- Single Block I/O 기준 아무리 빨라도 1초에 1000블록이 최대다. 

## <span style="color:#802548">_테이블 스페이스_</span>
- 테이블 스페이스를 생성해야 데이터르 저장할 수 있다.
- 테이블 스페이스는 여러 개의 세그먼트로 나뉜다.
  - 테이블, LOB, 인덱스, 파티션
- 세그먼트는 익스텐트로 구성된다. 데이터를 저장하는 용량이 익스텐트다.
  - 만약 테이블, 인덱스 등이 해당 익스텐트 용량으로 부족하면 익스텐트를 추가로 할당받는다.
- 익스텐트는 블록으로 구성된다. 블록이 실제 데이터가 저장되는 공간이다.
  - 블록에 저장되는 데이터는 모두 하나의 테이블에 속하게 된다. 이는 익스텐트도 동일하다.
- 각 오브젝트(테이블, 인덱스 등) 익스텐트 끼리는 서로 같은 파일 상에 위치하지 않을 확률이 높다.

## <span style="color:#802548">_테이블 스페이스를 이용한 scan_</span>
- 데이터 블록은 몇 번 데이터 파일의 몇번째 블록인지 나타내는 고유 주소값이 있다. 
  - 이를 Data Block Address(DBA)라고 한다.
- DBA와 로우 번호(블록 내 순번)의 구성이 ROWID다.
  - 이 ROWID를 이용하여 인덱스 range scan을 수행한다.
- 반면 table scan의 경우에는 익스텐트 맵을 이용한다.
- 테이블 세그먼트 헤더를 스캔한다.
```sql
SELECT segment_type, tablespace_name, extent_id, file_id, block_id, blocks
FROM dba_extents
where owner = USER
and segment_name = 'MY_SEGMENT'
order by extent_id;
```

- 익스텐트 맵을 통해 각 익스텐트의 첫 번쨰 블록 DBA를 알 수 있다.
- 익스텐트는 연속된 블록 집합이다. 따라서 첫 번째 블록 뒤에 연속해서 저장된 블록을 읽으면 된다.
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


- 블록이 DBMS가 데이터를 읽고 쓰는 단위다.
- 특정 레코드 하나를 읽는 건 불가능하다. 반드시 블록 단위로 읽게 된다.
- 아래는 table scan시의 예시다.
```
TABLE 세그먼트
EMP 테이블    Salary 테이블 ........ 

EMP 테이블 내 익스텐트
익스텐트1
블록1
EMPNO     ENAME     MGR     JOB         SAL     DEPTNO
7369      SMITH     7902    CLERK       800     20
7499      ALLEN     7968    SALESMAN    1600    30
7521      WARD      7698    SALESMAN    800     30
7566      JONES     7839    MANAGER     800     20
7654      MARTIN    7698    SALESMAN    800     30


블록2
EMPNO     ENAME     MGR     JOB         SAL     DEPTNO
7698      BLAKE     7839    MANAGER     800     30
7782      CLARK     7839    MANAGER     1600    10
7788      SCOTT     7566    ANALYST     800     20
7839      KING              PRESIDENT   800     10
7844      MARTIN    7698    SALESMAN    800     30

블록3
EMPNO     ENAME     MGR     JOB         SAL     DEPTNO
7876      ADAMS     7788    CLERK       1100    20
7900      JAMES     7698    CLEKR       950     30
7902      FORD      7566    ANAYLST     3000    20
7934      MILLER     7782    CLERK       1300    10

블록4
```

- 예를 들어 1byte짜리 컬럼인 JOB(varhcar2)을 읽고 싶다고 해보자.
- 그럼에도 block 단위로 읽어야 한다. 보통 block은 8KB다(VALUE가 8192).
- 아래를 통해 확인 가능한데 보통 8KB다. 2,4,16,32KB 중 선택가능하다.
```sql
show parameter block_size
```

- index scan을 해도 1byte짜리 column 값을 가져오려고 8KB 블록을 읽어야 하는 건 마찬가지다.
- 어떤 scan을 하든 block을 읽어야 하는 건 동일하다.
- 다만 index는 rowid를 통해 직접 block에 접근할 수 있을 뿐이다.

## <span style="color:#802548">_시퀀셜과 액세스 스캔_</span>
- 시퀀셜 액세스는 익스텐트 맵을 따라서 순차적으로 접근한다.
- 반면에 랜덤 액세스는 순서를 따르지 않고 직접 rowid에 따른 개별 block에 접근한다.



## <span style="color:#802548">_데이터 캐시_</span>
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
  - 버퍼캐시는 공유메모리이므로, 해당 블록을 읽어야하는 다른 process도 이득을 본다.


## <span style="color:#802548">_효율적 scan_</span>
- 데이터 캐시에 있는 것만으로 scan을 실행하는 게 가장 효율적이다.
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
- 따라서 SQL을 수행하면서 읽은 모든 블록 I/O는 논리 I/O다.
- 디스크를 갔다 오는 I/O인 물리 I/O는 매우 느리다.
- 더군다나 물리 I/O는 외생변수라서 개발자가 통제가 불가능하다.
  - 자전거를 타는데 바람을 통제할 수 없는 것과 동일하다.
- 따라서 모든 튜닝은 논리 I/O를 줄이는 일이다. 
- 논리 I/O를 줄이는 길은 바로 scan을 줄이는 것이다.

## <span style="color:#802548">_single vs multi block I/O_</span>
- I/O call을 통해 데이터 블록을 가져온다고 했다.
  - 한 번에 한 블록씩 가져오는 것은 single block I/O라고 부른다. index를 읽을 때 쓴다.
    - 소량을 읽을 때는 효율적이다.
  - 한번에 여러개를 가져오는 것은 multi block I/O라고 부른다. table을 읽을 때 쓴다.
    - 대량으로 읽을 떄는 효율적이다.
    - 그 중에서도 multiple block 단위가 클수록 좋다. 
    - I/O call이 줄어 process가 대기상태에 빠지는 횟수가 줄어들기 때문이다.
      - OS단에서 최대 단위가 보통 1MB이기 떄문에 DB에서도 1MB 이상 설정할 필요가 없다.
```sql
show parameter db_file_multiblock_read_count
```
```
NAME                                TYPE          VALUE
db_file_multiblock_read_count|      integer       16
```

- 8KB block 기준 128로 바꾸면 1MB로 최대 크기를 설정할 수 있다.
  - 8Kb x 128이면 딱 1MB다.
  - 물론 8KB가 아니라 16KB 기준으로는 64로 설정하는 게 최대일 것이다.
  - 만약 OS가 2MB라면 8KB 기준 256으로 설정해도 된다.
```sql
alter session set db_file_multiblock_read_count = 128;
```

- 블록은 익스텐트의 경계를 넘지는 못한다.
- 따라서 128로 설정한다고 해도, 익스텐트를 넘어선 다른 블록까지 읽어오지 않는다.
- 한 익스텐트에 300 block이 있다고 해보자.
- 그럼 세번쨰 I/O call에서 다른 익스텐트것을 포함해 가져오지 않고 44개에서 끝난다.
```
128 128 44
```

- multiblock I/O에서도 single block I/O가 나올 수 있다.
- 그 이유는 버퍼캐시 때문이다.
- multiblock I/O 단위가 4라고 하자.
- 그런데 익스텐트 맵을 통해 확인한 결과 블록 목록 1부터 10까지 있었다.
- 그 중에 버퍼캐시에는 1,6,8만 있다.
```
1번이 캐시버퍼 체인에 있다. 1번 버퍼블록을 캐시에서 바로 읽는다.
2번은 못 찾았다. 디스크 I/O를 보류한다.
3번을 못 찾았다. 디스크 I/O를 보류한다.
4번을 못 찾았다. 디스크 I/O를 보류한다.
5번도 못 찾았다. 4개 채웠으니까 2 - 5번 block을 multiblock I/O 방식으로 디스크 I/O call한다. 2 - 5번 버퍼블록을 캐시에서 읽는다.
6번을 캐피버퍼 체인에서 찾았다. 6번 버퍼블록을 캐시에서 바로 읽는다.
7번은 못 찾았다. 디스크 I/O를 보류한다.
8번은 찾았다. 7번 블록은 singloe block I/O로 디스크 I/O call한다. 7-8번 버퍼블록을 캐시에서 읽는다.
9번은 못찾았다. 디스크 I/O를 보류한다.
10번은 못찾았다. 디스크 I/O를 보류하지 못한다. 마지막 익스텐트 블록이다. multiblock I/O로 디스크 I/O call한다. 9 - 10번 버퍼블록을 캐시에서 읽는다.
```

## <span style="color:#802548">_table full scan vs index range scan_</span>
- index range scan은 사실 index를 이용한 table access다.
- 인덱스에서 일정범위를 scan하여 얻은 rowid로 table record에 access한다.
  - rowid는 table record가 disk 어디에 저장됐는 지 가리키는 위치 정보다.
- index range scan은 random access + singleblock I/O의 조합이다.
  - 캐시에서 블록이 없으면 레코드 하나를 읽으려고 매번 process가 대기상태에 빠진다. 
  - 당연히 비효율적이다. 다만 소량의 데이터를 scan하는데는 적합하다.
  - 그러나 선택률이 높은 경우, index range scan보다 table full scan이 더 낫다.
```
cardinality가 높다면.. hint를 써서라도 table full scan으로 전환하라
```
- table full scan이라고 나쁜 게 아니다. 대용량 처리인 집계용 SQL 혹은 배치는 table full scan이 더 낫다.
- table full scan은 sequential scan + multiblock I/O의 조합이다.
- 한 블록에 속한 모든 레코드를 한번에 읽는다. 캐시에 없으면 한번 I/O call을 통해 인접한 수백개의 block을 한꺼번에 I/O 한다.
- 따라서 process가 대기상태에 빠지는 일이 현저하게 적다.
- 해쉬 조인으로 선택하여 성능을 올린다.

## <span style="color:#802548">_버퍼캐시 탐색 매커니즘_</span>
- direct path I/O를 제외한 모든 blcok I/O는 메모리 버퍼캐시를 경유한다.
- index range scan이든 table full scan이든 메모리 버퍼캐시가 필요하다는 의미다.
```
인덱스 루트 블록 read
인덱스 루트 블록 read해 얻은 주소 정보로 브랜치 블록 read
브랜치 블록 read해 얻은 주소 정보로 리프 블록 read
리프 블록 read해 얻은 주소 정보로 table block read
table block full scan
```


- 버퍼캐시는 공유자원이라 누구나 접근할 수 있다.
- 다만 동시접근이 일어나는 경우, 순차적으로 접근하게끔 해주어야 한다.
  - 이렇게 순차로 접근하게 하는 걸 직렬화라고 한다. 줄세우는 것이다.
  - 줄 세우기 메커니즘은 래치로 구현한다.

## <span style="color:#802548">_index 구조_</span>
- 대부분의 인덱스의 구조는 거꾸로 된 나무처럼 되어있다. 이른바 B tree index다.
- 구조를 그려보면 아래와 같다.
```
                                  Root Node
                                      S

              branch Node                                                 Branch Node
                F     G                                                    Sp    W

Leaf Node       Leaf Node         Leaf Node                 Leaf Node     Leaf Node         Leaf Node     
Al Rabeeah      Flores            Garcia                    Salehi        Spievak           Wah
Beena           Fung              Mishra                    Schluter      Vu                Winter
Doshi                             Johnson                   Scruton                         Wong
```
- index table에는 leaf node를 총망라하면 아래와 같이 저장되어 있을 것이다.
- 그럼 실제 해당 rowid에 해당하는 table의 record를 찾아 access한다.
- 저중에는 LMC가 S이다. 가장 위에 올라가는 Root Node가 바로 LMC다.
```
rowid       lastname
6           Al Rabeeah
16          Beena
13          Doshi
10          Flores
11          Fung
19          Gani
8           Garcia
21          Hu
22          Israr
9           Johnson
18          Ly
2           Mishra
17          Ngo
20          Pham
3           Salehian
1           Schluter
14          Scruton
12          Spievak
4           Vu
5           Wah
15          Winter
7           Wong
```

- 다시 알아보기 쉽게 간단한 예제를 하나 더 들어보자.
- b tree index가 아래와 같이 생겼다고 가정해보자.
```
                              [17]
                            /   |   \
                  [5, 10]   [20]   [25, 30]
                 /  |  \    /  \     /   \
            [1, 3] [7] [12, 15] [22] [27] [35]
```
- 위에서는 key와 data가 같았지만, 실제로는 key와 data가 같지 않을 것이다.
- key는 int, varchar 등 여러 타입이 될 수 있다.
```
| rowid | Key |   Data   |
|-------|-----|----------|
|   1   |  5  |  Record1 |
|   2   | 10  |  Record2 |
|   3   | 17  |  Record3 |
|   4   | 20  |  Record4 |
|   5   | 25  |  Record5 |
|   6   | 30  |  Record6 |
|   7   | 35  |  Record7 |
```

## <span style="color:#802548">_index 튜닝 유형_</span>
- index는 큰 테이블에서 소량 데이터를 검색할 때 사용한다.
- index를 이용한 tuning에서 가장 중요한 것은 인덱스 스캔 효율화 튜닝이다.
  - 정렬을 처음부터 잘해야한다는 의미다.
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
- 사실 인덱스 스캔 효율화 튜닝보다 이게 훨씬 중요하다.
- 인덱스 스캔이 비효율적이어도 학생명부를 뒤지면 되겠지만, random access는 직접 교실을 찾아가며 방문하는 것과 같기 때문이다.


## <span style="color:#802548">_index 탐색_</span>
- index를 수직 탐색한다는 의미는 조건을 만족하는 레코드를 찾는다는 의미가 아니다.
- 조건을 만족하는 '첫' 레코드를 찾는다는 의미다. 즉 시작점을 찾는 것이다.
- 아래와 같은 sql이 있다고 해보자.
```sql
create index 고객 on 고객(성별, 고객명);

select from 고객
where 성별 = '남'
and 고객명 = '이재희'
```

- 그럼 아래와 같이 root -> branch -> leaf로 나아가면서 시작점을 찾아간다.
- 남 & 이재희가 발견된 해당 리프 블록이 곧 시작점이다.
- 해당 리프 블록에서 scan을 더 수행해서 만족하는 지점이 없으면 scan을 멈춘다. 이렇게 되면 수직 탐색만 존재하는 셈이다.
- 만약 계속 만족하는 조건이 나오면 다음 리프블록도 읽어간다. 이게 수평 탐색이다.
```
                          LMC
                        남 & 최
      남 & 송재훈                     남 & 홍병남           
      남 & 이재룡                     여 & 김소희
남 & 강경윤   남 & 이재명        여 & 류하영      여 & 최지우
남 & 강기중   남 & 이재희(발견)  여 & 박미란      여 & 추자현
```
- 말로 나타내면 아래와 같다.
```
1. 우선 루트 블록에서 53이 속한 키 값을 찾는다. 두 번째 레코드가 선택될 것이므로 거기서 가리키는 3번 블록으로 찾아간다. (수직)
2. 3번 블록에서 다시 53이 속한 키 값을 찾는다. 여기서는 첫 번째 레코드가 선택될 것이므로 9번 블록으로 찾아간다. (수직)
3. 찾아간 9번은 리프 블록이므로 거기서 값을 찾거나 못 찾거나 둘 중 하나다. 다행히 세 번째 레코드에서 찾아지므로 함께 저장된 ROWID를 이용해 테이블 블록을 찾아간다. ROWID를 분해해 보면, 오브젝트 번호, 데이터 파일번호, 블록번호, 블록 내 위치 정보를 알 수 있다. (수직)
4. 테이블 블록에서 레코드를 찾아간다. (수직)
5. 인덱스가 Unique 인덱스가 아닌 한, 값이 53인 레코드가 더 있을 수 있다. 따라서 9번 블록에서 레코드 하나를 더 읽어 53인 레코드가 더 있는지 확인한다. (수평)
6. 53인 레코드가 더 이상 나오지 않을 때까지 스캔하면서 테이블 액세스 단계를 반복한다. 만약 9번 블록을 다 읽었는데도 계속 53이 나오면 10번 블록으로 넘어가서 스캔을 계속한다. (수평)
```

- 도식도로 나타내면 아래와 같다.
- LMC, 즉 root block은 branch block에 대한 정보를 갖고 있다.
- branch block은 leaf block에 대한 정보를 갖고 있다.
- leaf block은 rowid를 갖고 있다.
- 그럼 해당 rowid를 가지고 table에 access하여 record 정보를 가져오는 것이다.
```
                    블록번호1번
                    1-50: 2번블록
                    51-100:3번블록
          블록번호2번         블록번호3번
          1-10: 4번블록       51-60:9번블록
          11-20:5번블록       61-70:10번블록
          21-30:6번블록       71-80:11번블록
          31-40:7번블록
          41-50:8번블록
블록번호4번                 블록번호5번             블록번호6번       ....      블록번호10번
이전블록번호:NULL           이전블록번호:4번        이전블록번호:5번             이전블록번호:9번
다음블록번호:5번            다음블록번호:6번        다음블록번호:7번             다음블록번호:NULL
1: ROWID                   11:ROWID               21:ROWID                   ....
2: ROWID                   12:ROWID               22:ROWID
3: ROWID                   ...                    ...
.
10:ROWID
```

- 인덱스 컬럼을 바꾼다고 해도 조건이 모두 =인 경우에는 성능 변화가 없다.
- 시작점이 바뀌지 않기 때문이다. 따라서 인덱스가 (고객명, 성별)이든 (성별, 고객명)이든 똑같다.
- index는 엑셀의 filtering같이 작동하는 것이 아니다.


- index 수평 탐색은 실제 데이터를 찾아가는 과정이다.
- index를 수평탐색하는 이유는 조건절을 만족하는 모든 데이터를 찾는 게 첫번째다.
- rowid를 얻는 것은 두 번째다.
- index block leaf는 서로 양방향 연결 리스트라 좌 - 우, 우 - 좌로 모두 이동이 가능하다.
- 따라서 내림차순으로 정렬하면 조건을 만족하는 가장 큰 값을 찾아 우측으로 수직 탐색을 하다 좌측으로 수평탐색을 한다.
- 따라서 오름차순으로 정렬하면 조건을 만족하는 가장 큰 값을 찾아 좌측으로 수직 탐색을 하다 우측으로 수평탐색을 한다.


## <span style="color:#802548">_index 비효율_</span>
- index는 기본적으로 특별한 경우를 제외하면 range scan을 한다는 사실을 알게 되었다.
- 그런데 range scan을 효율적으로 진행하려면 시작점을 알고 끝점을 알아야 한다.
- 그 중에서도 시작점이 중요하다. 아래와 같이 하면 시작점과 끝점이 분명하다.
```sql
where 성년월일 between '20070101' and '20070131'
```
- 그러나 시작점을 모르는 경우, index를 full scan때릴 수 밖에 없다.
- 포함하는 조건,인덱스 컬럼을 가공하는 조건 등은 인덱스의 시작점을 알 수 없게 만든다.
- 년도와 상관없이 5월에 태어난 사람을 찾아보라고 하면, index scan을 어디서 시작해야할 지 당연히 정보가 없다.
- index full scan을 때릴 수 밖에 없다는 의미다. index column은 반드시 원형 그대로 놔둬야 한다.
```sql
where substr(생년월일, 5, 2) = '05'
```
- 마찬가지로 index column이 원래 null인데 이런식으로 가공하면 index에 담겨있는 정보가 의미가 없으니 시작점도 찾을 수가 없다.
```sql
where nvl(주문수량, 0) < 100
```
- 포함하는 조건은 위에서 말했듯, 시작점을 당연히 알수가 없다.
```sql
where 업체명 like '%대한%'
```
- or조건도 그렇다. 전화번호가 XX거나 고객명이 홍길동인 조건은 시작지점을 지정할 수가 없다.
```sql
where (전화번호 = :tel_no OR 고객명 = :cust_nm)
```
- or 조건의 다른 표현은 in도 마찬가지다. 
```sql
where 전화번호 in (:tel_no1, :tel_no2)
```

- or 조건은 아래와 같이 optimizer에서 변환해서 index range scan을 하기도 한다.
  - 이를 or expansion이라고 한다.
```sql
where 고객명 = :cus_nm
union all
where 전화번호 = :tel_no
```

- in 조건도 동일하다. 이러한 최적화를 in-list iterator 방식이라고 한다.
- in-list안의 갯수만큼 index range scan을 반복한다.
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

- like와 비슷하게 decode 또한 문제가 있다.
- oracle 구현인 decode는 아래와 같이 사용한다.
- a=b라면 c를 반환하고 아니면 d를 반환한다.
- 그런데 반환된 데이터의 타입은 c가 결정한다.
- 그런데 c가 null이면 varchar2로 취급하게 된다. 따라서 어처구니 없는 일이 일어난다.
- 회장을 제외하고 가장 높은 연봉을 조회한다고 해보자.
```sql
SELECT ROUND(AVG(sal)) avg_sal
      ,MIN(sal) min_sal
      ,MAX(sal) max_sal
      ,MAX(decode(job, 'PRESIDENT', NULL, sal)) max_sal2
FROM emp;
```

- 그럼 문제가 발생한다.
- c에 null이 들어갔다. varchar2 type이 된것이다. 그럼 sal도 varchar다.
- sal이 int형이었다고 해도, 문자로 형변환된다. 그럼 3000과 950 중에 숫자라면 3000이 컸을 것이다.
- 그런데 문자형이 되었기에 3000보다 950이 더 크다. 9가 3보다 크기 때문이다.
- 따라서 max연봉이 3000이 아니라 950이 찍혀버리는 불상사가 일어난다.
- 이런 경우, 형변환을 적절하게 해주어야 한다.
- 늘 적절하게 TO_CHAR, TO_DATE, TO_NUMBER 등으로 묵시적 형변환을 해제해야 한다.
- 형변환 함수는 별로 손해보지도 않는다. 중요한건 I/O를 줄이는 것이다.
```sql
SELECT ROUND(AVG(sal)) avg_sal
      ,MIN(sal) min_sal
      ,MAX(sal) max_sal
      ,MAX(decode(job, 'PRESIDENT', TO_NUMBER(NULL), sal)) max_sal2 /**TO_NUBMER(NULL) 대신 0도 가능하다. */
FROM emp;
```

## <span style="color:#802548">_index는 sort by를 쓴 효과를 낸다_</span>
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

- 물론 sort by를 생략한다고 해서 아래와 같이 복잡하게 쓴다면 성능이 좋지 않을 것이다.
- 해당 장비구분코드에 맞는 장비번호와 최종변경일자, 순번을 알아오기 위한 코드인데, 꽤나 복잡하다.
```sql
SELECT 장비번호, 장비명, 상태코드
      ,(SELECT MAX(변경일자)
        FROM 상태변경이력
        WHERE 장비번호 = P.장비번호) 최종변경일자
      ,(SELECT MAX(변경순번)
        FROM 상태변경이력
        WHERE 장비번호 = P.장비번호
        AND 변경일자 = (SELECT MAX(변경일자)
                        FROM 상태변경이력
                        WHERE 장비번호 = P.장비번호)) 최종변경순번
FROM 장비 P
WHERE 장비구분코드 = 'A001';
```

- 위의 쿼리를 아래와 같이 간단하게 쓸 수도 있다.
- 그러나 MAX에 변경일자와 변경순번을 모두 쑤셔넣으면 기존에 index로 되어있던 sort by 연산이 박살난다.
- 따라서 sort by연산이 추가된다. index column을 가공한 결과는 나쁘다.
```sql
SELECT 장비번호, 장비명, 상태코드
      ,SUBSTR(최종이력, 1, 8) 최종변경일자
      ,SUBSTR(최종이력, 9) 최종변경순번
FROM (
  SELECT 장비번호, 장비명, 상태코드
        ,(SELECT MAX(변경일자 || 변경순번)
          FROM 상태변경이력
          WHERE 장비번호 = P.장비번호) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```


- 이럴 땐 hint를 활용해서 풀어야한다.
- 인덱스를 역순으로 써서 MAX와 동일한 효과를 낸다.
- 또한 첫번째 레코드에서 멈춰서 더 scan하지 않게 ROWNUM <=1 조건을 달아준다.
```sql
SELECT 장비번호, 장비명
      ,SUBSTR(최종이력, 1, 8) 최종변경일자
      ,SUBSTR(최종이력,9) 최종변경순번
FROM (
  SELECT 장비번호, 장비명, (SELECT /*+ INDEX_DESC(X 상태변경이력_PK) */
    변경일자 || 변경순번
    FROM 상태변경이력 X
    WHERE 장비번호 = P.장비번호
    AND ROWNUM <= 1) 최종이력
  FROM 장비 P
  WHERE 장비구분코드 = 'A001'
)
```

## <span style="color:#802548">_index scan의 종류_</span>
- index range scan
  - 선두 컬럼을 가공하지 않은 상태로 조건절에 써야한다.
  - index range scan이라고 무조건 효율적 scan을 의미하지 않는다.
  - index scan 범위를 줄이고, table access도 줄여야 한다.
```sql
SELECT * FROM emp where deptno = 20;
```

- index full scan
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
- 떄로 선택률이 높은데도 index를 쓰는 경우가 있다. 처음 일부만 보여주려고 의도할 때가 그러하다. 부분범위처리다.
- 만약 record를 계속 보려고 스크롤을 내리면 table full scan보다 효율이 훨씬 떨어지게 된다.
- WAS에서는 자주 쓰이지는 않는다. WAS에서는 paging을 명시로 해야하기 때문이다.
```sql
SELECT /*+ first_rows */
from emp
where sal > 1000
order by ename;
```

- index unique scan
  - unique index를 = 조건으로 탐색하는 경우, 수직탐색만 한다.
  - 다만 결합 index의 경우, 모든 조건을 전부 =으로 써야 index unique scan이 된다.
  - (주문일자, 고객ID, 상품ID) PK에서 where 조건문에 주문일자와 고객ID만 있다면 그것은 index range scan이 된다.
```sql
create unique index pk_emp on emp(empno);

select empno, ename from emp where empno = 7788;
```

- index skip scan
  - 인덱스 선두 컬럼이 조건절에 없어도 index를 활용하는 방식 중 하나다.
  - index full scan과 달리 조건에 맞지 않는 경우 index scan을 skip하여 효율적으로 scan한다.
  - cardinality가 극단적으로 높고 낮은 조합의 index에 보통 이용된다.
  - 성별과 PK의 조합이 대표적이다.
  - 인덱스 선두가 있고 중간 컬럼이 없는 경우에도 활용할 수 있다.
```sql
SELECT * FROM 사원 WHERE 성별 = '남' and 연봉 between 2000 and 4000;
```

- PK는 업종유형코드 + 업종코드 + 기준일자로 구성되어있다고 해보자.
- 아래는 업종코드라는 중간 컬럼이 비어있다. 이럴 때 index skip scan을 쓸 수 있다.
- hint를 주면 더 확실하다.
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

- index fast full scan
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


## <span style="color:#802548">_table access 최소화_</span>
- index로 scan한다고 해서 table access가 없는 것은 아니다.
- index rowid라는 논리적 주소로 table access를 매우 빠르게 할 뿐이다.
- index rowid는 7번 데이터파일 123번 블록 10번째 레코드와 같이 논리주소를 담는다. pointer와 같은 직접 연결된 물리주소가 아니다.
- redis같은 in memory db는 진짜 pointer로 작동하기 때문에 매우 빠르다.
- 그러나 oracle은 불가능한데, 데이터 캐싱 과정에서 있었다 사라졌다하는 과정이 반복되기 때문이다.
- oracle에서 index rowid를 이용하는 과정을 아래와 같다.
```
index의 수직-수평 탐색을 실행한다
리프블록에서 rowid를 읽는다
읽은 rowid를 분해해서 dba 정보를 얻는다
dba를 해시 함수에 입력해서 해시 체인을 찾고(해싱 알고리즘) 버퍼헤더를 찾는다.
거기서 얻은 논리주소로 버퍼 블록을 찾아간다
못찾았으면 디스크에서 읽어온다
```

- 일반 RDBMS의 rowid는 우편주소다. 우체부 아저씨가 직접 일일이 찾아나서야 한다. 전화번호처럼 직통이 아니다.
- rowid는 pointer에 비해 매우 고비용임을 알 수 있다.

- table access를 최소화하는 데는 clusetering factor도 중요하다.
- 특정 컬럼 기준으로 데이터가 서로 모여있어야 index의 효율이 좋다는 의미다.
- 다음 인덱스 레코드도 같은 테이블 블록인 경우, 래치 획득과 해시 체인 스캔 과정을 생략하기 때문이다.
- 다시 말해 논리 I/O도 줄어들고, disk I/O도 줄어든다는 의미다.
- 물론 특정 컬럼, 고객번호 기준으로 CF계수가 높다면, 다른 컬럼, 진료일자 기준으로는 CF계수가 낮을 것이다.
https://engineering-skcc.github.io/oracle%20tuning/Clustering-Reorg/



## <span style="color:#802548">_웹 vs 배치_</span>
- 웹은 보통 소량 데이터를 읽는다.
- 이 경우 index를 이용해 소트 연산을 생략하는 NL방식을 사용한다.
- 그 편이 대량 데이터에도 매우 빠르게 연산할 수 있기 때문이다.
- 반면에 데이터를 읽고 '갱신'해야 하는 배치 프로그램은 full scan과 해시조인을 사용한다.
- 전체를 스캔해서 처리해야하기 때문이다.
- 그러나 초대용량을 table full scan하는 것은 시간이 굉장히 오래걸린다.
- 그래서 배치의 경우, 파티션을 활용하고, 병렬처리를 활용해야 한다.
- 파티션 된 테이블에 index를 사용하는 것은 table full scan을 할 때보다 더 느려질 가능성이 높다.
- partition의 index는 각 partition에서만 작동하는데, partition이 여러 개에 걸쳐 data를 받아오는 경우 성능이 오히려 더 안좋아지는 것이다.
- 하지만 년단위로 분해한 partition에서 년단위 조회 혹은 해당 년도 내 월 조회 등을 하는 경우는 partition + index 조합이 극강의 위력을 발휘한다.


`

## <span style="color:#802548">_새롭게 index 만들기보단 인덱스컬럼 추가_</span>
- 기존 index column을 제끼고 아예 새로운 index를 만드는 건 흔하지 않다.
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


<img src ="/image/roaming-not-added-useyn.jpg" />

- 이렇게 비효율적으로 scan과 I/O를 하지 않게 사용여부를 index에 포함시켜보면 획기적으로 성능이 개선된다.

<img src='/image/roamding-added-useyn.jpg' />


## <span style="color:#802548">_인덱스 구조 테이블_</span>
- 인덱스를 이용한 access는 고비용 구조라고 했다.
- 그럼 랜덤 엑세스가 아예 발생하지 않게끔 하려면 어떤 방법이 있을까?
- 가장먼저 떠오르는 건 query에 필요한 모든 column에 index를 넣어주는 방식이다.
- 이를 커버링 인덱스, 커버드 인덱스라고 한다.
- 하지만 index-organized table로 만드는 방법도 있다. rowid가 있을 자리에 테이블 데이터가 있는 형식이다.
- 여기선 index leaf block이 곧 data block이다.

```sql
create table_index_org_t (a number, b varchar(10) constraint index_org_t_pk primary key (a))
organization index;
```

- 위와 같은 IOT는 index기 때문에 정렬상태를 유지하며 데이터를 입력한다.
- 따라서 랜덤 엑세스가 아닌 시퀀셜 액세스로 접근하며, 범위 검색에 유리하다.
- 특히 데이터 insert는 일자 기준인데, select는 일자 기준이 아닌 경우, 입력 패턴과 조회 패턴이 다른 경우 성능이 좋다.
- 예를 들어, 일자별로 영업실적을 집계하는 배치 테이블이 있다고 해보자.
- 그럼 배치 테이블에는 일자별로 insert가 되며, 영업사원 별로 하루마다 한 블록씩 365 block이 만들어진다.
- 하지만 일자 기준으로 insert가 이뤄져 CF계수가 매우 안 좋으므로 row수에 준하는 block I/O가 발생한다.
- 이럴 떄 IOT를 구성하여 CF를 올려 random access를 줄인다.
```sql
create table 영업실적 (사번 varchar2(5), 일자 varchar2(8), .....) organization index;
```


## <span style="color:#802548">_부분범위 처리 활용_</span>
- 전체 결과집합을 모두 한꺼번에 전송하지 않고, fetch call이 있을 떄마다 나눠 전송하는 것을 의미한다.
- array size는 WAS단에서 설정하는 편이며, 자바의 arraylist는 기본 size가 10이다. 
  - 대량 파일을 받을거면 최대한 size를 늘리는 게 좋다.
  - 그래야 fetch call을 줄이고, process가 잠에 빠지는 일이 줄어든다.
  - 반면에 앞부분만 좀 보여주고 만다면 size를 작게 설정하는 게 좋다.
  - 데이터를 받아오는 데 네트워크 및 메모리를 소모했는데, 쓰지 않는 비효율을 줄일 수 있기 때문이다.
- 다만 문제는 order by가 있는 경우, sort 연산이 들어가면 전체를 다 scan해야 하기 때문에 부분범위처리가 무용지물이 된다.
- 만약 선두컬럼에 인덱스가 있으면 sort by연산을 하지 않으므로 부분범위 처리는 가능하다.
```sql
SELECT name from big_table order by created /**선두컬럼이라 부분범위처리 가능 */
SELECT name from big_table order by name /**선두컬럼이 아니라 전체범위처리로만 가능 */
```
- fetch call이 100인데, array size가 20이라면 처음에는 연속 5번의 fetch call이 발생한다.
- 그 이후부터는 20개씩 단위로 fetch call이 발생한다.
- 정렬된 상태를 유지하는 index를 이용하면, 정렬 작업을 생략하고 앞쪽 일부 데이터를 빠르게 보여줄 수 있다.
- 인덱스 선두컬럼이 (게시판구분코드, 등록일시)가 아니면 sort by 연산이 생략 불가능하다.
- 선두컬럼을 (게시판구분코드, 등록일시)로 한다면 sort by 연산이 생략 가능하기 때문에 random access가 없어 빠르다.
```sql
SELECT 게시글ID, 제목, 작성자, 등록일시
FROM 게시판
WHERE 게시판구분코드 = 'A'
order by 등록일시 desc
```

- 그러나 현재 oracle 12c 버전이상부터는 index의 정렬을 믿고 필요한 order by를 안 쓰면 안 된다.
- batch I/O가 추가된 결과, index를 이용한 데이터 정렬 순서가 매번 달라질 수 있기 때문이다.
- batch I/O는 single block I/O가 기본이지만, 읽을 블록을 모아놨다가 한꺼번에 읽는다.
  - 즉 multiblock I/O처럼 작동해 I/O call을 줄이는 방식이다.
- 아래 sql은 index 정렬을 믿고 order by를 쓰지 않은 모습이다.
```sql
SELECT /*+ INDEX(H 상태변경이력_PK) */ 장비번호, 변경일시, 상태코드
FROM 상태변경이력 H
WHERE 장비번호 = :eqp_no
AND ROWNUM <= 10;
```
- 부분범위처리 효과를 얻기위해 rownum까지도 생략하는 경우가 있다.
```sql
SELECT /*+ INDEX(H 상태변경이력_PK) */ 장비번호, 변경일시, 상태코드
FROM 상태변경이력 H
WHERE 장비번호 = :eqp_no
```
- 그러나 order by가 없으면 순서가 정렬되어 보이지 않는다.
- 인덱스 자체는 정렬되어 있으나, batch I/O가 실행되면 resultSet은 정렬되지 않을 수 있다.
- 따라서 순서대로 data를 담아야한다면, 이젠 order by를 반드시 써야하는 것이다.

## <span style="color:#802548">_인덱스 탐색_</span>
- index를 탐색하는 절차에 대해 알아보자.
- 아래와 같은 sql이 있다.

```sql
WHERE C1 = 'B'
```
- 그럼 C1=B인 브랜치, 리프블록으로 내려가야 할 거 같지만 그렇지 않다.
- 그 직전인 A=3부터 내려간다. 그 이유는 C1=B가 '시작점'이 아니기 때문이다.
- A=3이 시작점이 될 수 있기 때문에 A=3인 브랜치 혹은 리프 블록으로 내려간다.
<img src="/image/condition1.jpg" />


- 아래와 같은 중첩조건문의 sql이 있다.
```sql
WHERE C1 = 'B'
NAD C2 = 3
```
- 그럼 C1=B인 브랜치, 리프블록 직전인 A=3 블록에 내려가고, 그 중에서도 C2=3인 블록을 찾는 거까지가 수직탐색이다.
- 그렇게 시작점을 발견했다면, index라서 정렬이 되어있는 상태일 것이다. 그럼 C2 = 3을 넘어 C2 = 4를 찾으면 거기서 탐색이 끝난다.
<img src="/image/condition2.jpg" />


- 아래와 같은 sql을 살펴보자.
- 이제는 부등호가 생겼다.
```sql
WHERE C1 = 'B'
AND C2 >= 3
```
- C1 = B 설명은 생략한다.
- C2 >= 3으로 인해 시작점이 변경되었다. 
- C2 = 3부터 C1 = C를 만날때까지로 끝점도 변경되었다. 
- 부등호가 스캔 시작점을 결정하는 데 중요한 역할을 했다.

<img src="/image/condition3.jpg" />


- 아래 sql을 보자.
- 부등호의 방향이 반대로 바뀌었다.
```sql
WHERE C1 = 'B'
AND C2 <= 3
```

- C2 <= 3 조건절은 스캔을 시작하는 데는 역할을 하지 못한다.
- 하지만 스캔을 멈추는 데는 중요한 역할을 한다.
- 이 경우에도 부등호는 스캔을 줄이는 데 역할을 했다.


<img src="/image/condition4.jpg" />

- 아래 sql을 보자.
- 이제는 선두컬럼은 =이지만, 두번쨰 조건부터는 범위가 아예 지정이다.
```sql
WHERE C1 = 'B'
AND C2 BETWEEN 2 AND 3
```

- C1과 C2모두 스캔 시작과 끝 지점을 결정하는 데 중요한 역할을 했다.
- 쌍부등호 or BETWEEN도 스캔량을 줄이는 데 역할을 했다.


- 아래 sql을 보자.
- 선두컬럼도 범위로 지정되었다.
```sql
WHERE C1 BETWEEN 'A' and 'C'
AND C2 BETWEEN 2 AND 3
```

<img src="/image/condition6.jpg" />

- C1은 스캔 시작점과 끝지점을 지정하여 스캔량을 줄이는 데 역할을 했다.
- 하지만 C2는 전혀 역할을 하지 못했다. 특히 C1이 B일 떄 더 그렇다.
- 선두컬럼을 범위로 놓으면 그 뒤 컬럼부터는 스캔량을 줄이는 데 도움이 되지 않음을 알 수 있다.




- 지금까지는 index가 계속 붙어있는 것을 가정했다.
- 하지만 index 선행 컬럼이 존재하지 않는 경우도 있다. 
- 이 경우에는 결과값은 같을지라도, 훨씬 더 많은 index leaf block scan이 필요할 가능성이 높다.
- 성능검으로 시작하는 record를 읽을 때 필요한 건 3 block이지만,
- 성능으로 시작하다가 중간이 비고 4번째가 선으로 끝나는 걸 읽을 때 필요한 block은 6 block이다.

<img src="/image/index-omitted-inefficietn.jpg" />

- 구체적 수치로 index scan의 효율성을 따지는 방법은 sql trace를 보면 된다.
- 아래와 같은 trace에서 읽은 블록은 7463개지만, 얻은 record는 10개밖에 안된다.
- 한 블록당 평균 500개가 담긴가도 하면, 5463 x 500 총 3,731,500개 record를 읽어 10개 record를 얻은 것이다.
- 엄청나게 비효율적이라고 볼 수 있다.
```
Rows      Row Source Operation
10        TABLES ACCESS BY INDEX ROWID BIG_TABLE(cr=7471 pr=1466 pw=0 time=22137)
10          INDEX RANGE SCAN BIG_TABLE_IDX (cr=7463 pr=1466 pw=0 time=22328)
```

## <span style="color:#802548">_인덱스 액세스조건과 필터조건_</span>


- index access 조건과 filter 조건은 서로 다른 개념이다.
- access 조건은 index scan 범위를 결정하고, filter 조건은 table access 여부를 결정한다.
- index filter 이외에 table filter는 access 이후에 결과를 가져온 뒤 filtering하는 조건이다.

<img src="/image/index-access-filter.jpg"/>

- index를 이용한 테이블 액세스 비용은 table filter를 뺀 index access, index filter의 합이다.
  - index root와 branch block에서 읽는 block을 plus
  - index leaf block에서 읽는 block을 plus
  - table access에서 읽는 block수를 plus

- table과 달리 index에는 같은 값을 갖는 key-value pair가 서로 군집해 있다.
- index를 (C1,C2,C3,C4)로 지정했다고 해보자.
- 그 상태에서 조건절 1을 보자. 모두 =으로 걸려있다. key-value pair들이 연속해서 모여있게 된다.
- index leaf block은 data block과 다르게 entire read가 아니다. 따라서 조건절을 만족하지 않는 element를 만나면 read를 멈춘다.
- index leaf block에서 확보한 rowid를 일일이 건별로 data block에 access하면 불필요한 access가 많아진다.
  - 어차피 같은 data block에 속하는 rowid인데도 두 번 access를 해야할 수도 있기 때문이다.
  - 따라서 현대 db는 index leaf block에서 얻은 rowid를 통합해 확보하고 필요한 table access를 줄이는 방식으로 불필요한 access를 줄일 가능성이 높다.
  - 그 다음 rowid를 통해 실제 table segment의 data block을 entire read하게 된다.
    - data block과 index leaf block은 서로 다른 개념이라고

- 조건절 2는 맨 마지막 컬럼만 범위검색 조건이다.
- 그래도 5~10번 key-value pair가 모여있는 모습을 볼 수 있다.
  - 범위검색 조건인 BETWEEN, LIKE(전방일치)를 쓸 때도 똑같다.

- 마지막이 아닌 중간 컬럼이 범위 검색 조건일 때는 좀 다르다.
  - C1분터 C3까지 조건을 만족하는 key - value pair는 2~12번으로 서로 모여있다. 
  - C4까지 만족하는 key-value pair는 서로 흩어진다. 2,3,5,6,7,11번으로 말이다.

- 첫번째 나오는 범위조건까지는 key-value pair(index record)가 모여있지만, 그 뒤부터는 어떤 조건이든 다 흩어지게 된다.
- 따라서 처음 나오는 범위조건까지가 index access 조건이고, 그 뒤부터는 index filter 조건이다.
- 만약 조건절이 index가 걸리지 않은 column이면 table filter조건이 된다.
- 실행계획 상으로는 그렇지 않을수도 있지만, 대개 무시할만한 수준이기 때문에 위처럼 외우면 된다.


<img src ="/image/index-leaf-block-clustring.jpg" />


- index는 모두 = 조건(등치조건)으로 걸리는 게 가장 좋다.
  - leaf block을 읽으면서 얻은 key-value pair가 index filter조건에 걸러지지 않는다.
  - 따라서 모두 table access로 이어지기 때문에 scan이 완전 효율을 달성한다.
  - 맨 마지막 column이 등치가 아닌 것은 상관없다.
- 반면에 선행 컬럼이 없거나, 선행에서 범위검색으로 가버리면 비효율이 발생한다.
  - leaf block을 읽으면서 얻은 key-valu pair인 index record가 index filter조건에 걸러진다.



## <span style="color:#802548">_인덱스 액세스조건과 필터조건_</span>

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
```sql
/**아래는 비효율적 index range scan이며 only scan이다. */
create index idx_covering(name,age);
select name
from employee
where age = 10;

/**아래는 효율적 index range scan이며 only scan이다. */
create index idx_covering(age,name);
select name
from employee
where age = 10;
```

- where 조건문에 쓰는 조건은 index access 조건과 index filter 조건, table filter 조건으로 나뉜다.
  - index access는 수직탐색(시작점)과 수평탐색(끝점)을 결정하는 용도의 조건이다.
  - index filter는 table에 실제로 access를 할지 결정하는 용도의 조건이다.
  - table filter는 index가 아예 없는 column의 조건문인 경우다. table record를 얻어온 뒤 filtering하는 조건이다.
  - index로 사용된 컬럼 중 범위처리된 조건까지만이 index access 조건이며, 그 뒤부터는 index filter조건이다. 
- index access 조건으로 index key scan을 할 범위를 정한다.
  - 그 중에서 어떤 index record의 rowid를 가지고 table access할 지는 index filter조건이 정한다. 
    - 그렇게 가져온 table record를 다시 table filter조건으로 걸러서 resultSet을 만드는 것이다.


## <span style="color:#802548">_BETWEEN을 IN-LIST로_</span>
- 운영 시스템에서 인덱스 구성을 바꾸기는 사실 쉽지 않다.
- 그렇기 때문에 between이 써있는 데도 그냥 내두는 경우가 흔하다.
- 하지만 in 조건을 활용하면 index 구성을 바꾸지 않고도 index range scan을 효율적으로 진행할 수 있다.
  - 만약 아래가 인터넷매물  betwenn 1 and 3이었다면, 1부터 3까지 전부 index record를 scan했을 것이다.
  - 그럼 그 아래 조건 아파트시세코드, 평형, 평형타입은 모두 index filter조건으로 기능하게 된다. 비효율적 scan이 일어나게 되는 것이다.
  - 대신 아래와 같이 쓰면 수직적탐색을 3번 거치지만, 수평탐색이 전혀 없게 된다.

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
  - 가령 상품ID가 PK고 상품구분코드가 GX, KR인 경우를 찾는 경우라면?
  - 상품구분코드가 in-list로 풀려서 access 조건이 되면 index scan 범위가 매우 넓어지게 되어버린다.
  - 그 경우보다는 range scan 범위를 줄이고, filter를 하는 게 더 편리할 것이다.
  - 오히려 in-list로 변환하는 게 더 나쁘다는 의미다.

- 아래와 같은 간단한 sql 예시를 보면 확실하게 이해가 갈 것이다.
- 아래는 in 조건절이 있다.
```sql
SELECT * FROM 상품
WHERE 상품ID = :prod_id
AND 상품구분코드 in ('GX','KR')
```

- optimizer는 위의 in을 in-list가 아닌 filter조건(BETWEEN)으로 풀어냈다.
- 그 이유가 뭘까? 상품ID가 PK라서 unique index이기 때문이다.
  - index scan 시 오히려 상품구분코드를 access 조건으로 취급하지 않는게 scan 양이 줄어든다.
  - in-list로 푸는 게 도움이 되지 않는다는 의미다. 마찬가지로 index 구조를 바꾸는 것도 도움이 안 된다.
  - 상품구분코드가 선두컬럼이 되면 index scan양이 unique index인 PK가 선두컬럼일 때보다 훨씬 많아진다.
```
상품_PK: 상품ID
상품_X01: 상품ID + 상품구분코드

EXECUTION PLAN
---------------------------------------------------------------
0         SELECT STATEMENT 
1   0       TABLE ACCESS (BY INDEX ROWID) OF '상품' (TABLE)
2   1         INDEX (RANGE SCAN) OF '상품_X01' (INDEX)
--------------------------------------------------------------
PREDICATE INFORMATION
--------------------------------------------------------------
2- access('상품ID'=:prod_id)
2- filter('상품구분코드'='GX' OR '상품구분코드' = 'KR')
```


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

- 다시말하지만 핵심은 index scan양을 줄이는 것이다.
  - in-list방식으로 풀어서 index access로 만드는 게 index scan량을 줄이는지?
  - 아니면 BETWEEN으로 풀어서 index filter로 만드는 게 index scan량을 줄이는 지 판별해야 한다.


## <span style="color:#802548">_BETWEEN과 LIKE_</span>
- LIKE보단 BETWEEN이 더 낫다.
- 아래와 같이 두 sql이 있다고 해보자.
  - 인덱스가 (판매월,판매구분)으로 구성되었다.
  - 판매구분은 A와 B가 있고 A가 90%, B가 10%다.
    - BETWEEN은 바로 201901이면서 B인 지점으로 starting point를 정할 수 있다.
    - LIKE는 그게 안된다. 201900이 존재할 수도 있기 때문에 무조건 처음부터 읽는다. 
    - 따라서 판매구분이 B인 경우의 조건절이 access조건으로서 무력화된다.
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