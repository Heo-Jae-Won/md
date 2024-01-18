## <span style="color:#802548">_1. substr_</span>

- mysql의 substring()과 똑같은 기능의 함수다.
- substr(문자열,시작위치,길이)로 쓴다.
  - 띄어쓰기 된 빈 공간도 인식해서 가져온다.


```sql
substr('매니저',2,1) //니
substr('김밥이좋아',-3,2) //이좋
substr('필통이 없어', 4) // 없어
```
## <span style="color:#802548">_2. rownum_</span>

- mysql에서도 똑같은 함수명으로 사용가능하며,  조회된 순서대로 순번을 가져온다.


```sql
select rownum a.* from emp a 
```

- row_num을 정렬하는 기준을 주려면 아래와 같이 over절 안에 order by절을 사용하면 된다. 


```sql
select row_number() over(order by a.jbob, a.ename) row_num, a.* 
            from emp a 
                order by a.job, a.ename
```
- 만약 특정한 기준마다 조회 순서를 다르게 뽑고 싶다면 over절 안에 partition을 줄 수 있다.


```sql
select row_number() over(partition by a.job order by a.job, a.ename) row_num, a.* 
        from emp a 
            order by a.job, e.ename
```
## <span style="color:#802548">_3. nvl_</span>

​

- mysql의 ifnull()과 같다.
- null일 경우의 null을 반환하지 않고 다른 값을 반환한다.


```sql
select nvl(ename,'이민');
```
## <span style="color:#802548">_4. to_date_</span>

​

- mysql에도 있는 str_to_date()함수와 동일하다. 
- 날짜형으로 변환될 수 있는 문자열의 경우 오른쪽과 같은 형태의 날짜형으로 변환한다.
- dual이라는 table은 그냥 임시테이블이다. 
  - 테이블을 만들지 않고도 간단하게 테스트를 할 수가 있어서 편하다.


```sql
select to_date('2021-12-12','YYYY-MM-DD') from dual 
select to_date('20211212175000','YYYYMMDDHH24MISS') from dual 
```
## <span style="color:#802548">_5. to_char_</span>

​

- mysql의 date_format과 동일하다.
- 날짜형을 문자열로 변환한다. 


```sql
select to_char(sysdate,'YYYYMMDDHH24MISS') from dual
```
## <span style="color:#802548">_5. ||_</span>

​

- 두 개의 다른 return을 하나로 모아주는 역할을 한다.


```sql
select 'I am' as name from dual //I am
select 'I am' || column as name from dual // I am [column value]
```