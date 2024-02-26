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