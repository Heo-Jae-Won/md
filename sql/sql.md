



# select문을 최대로 줄인 다중 필드 update
- 한 과목씩 값을 갱신하는 sql이 있다고 해보자.
- sql을 세번씩이나 쓰는 건 좋은 방식이 아니다.
```sql
update ScoreCols
    set score_en = (select score 
                    from ScoreRows SR
                    where SR.student_id = ScoreCols.student_id
                    and subject = '영어'),
        score_nl = (select score
                    from ScoreRows SR
                    where SR.student_id = ScoreCols.student_id
                    and subject = '국어'),
        score_mt = (select score
                    from ScoreRows SR
                    where SR.student_id = ScoreCols.student_id
                    and subject = '수학');
```

- 다중 필드 할당을 아래와 같이 해주면 select query 하나로 3개의 field를 update
- 이건 oracle에서의 구문

```sql
update ScoreCols
    set (score_en, score_nl, score_mt)
        = (select   max(case when subject = '영어'
                        then score
                        else null end) as score_en,
                    max(case when subject = '국어'
                        then score
                        else null end) as score_nl,
                    max(case when subject = '수학'
                        then score
                        else null end) as score_mt
        from scoreRows SR
        where SR.student_id = ScoreCols.student_id);
```

- mysql의 구문은 아래와 같음.


```sql
UPDATE ScoreCols
JOIN (
    SELECT
        student_id,
        MAX(CASE WHEN subject = '영어' THEN score ELSE NULL END) AS score_en,
        MAX(CASE WHEN subject = '국어' THEN score ELSE NULL END) AS score_nl,
        MAX(CASE WHEN subject = '수학' THEN score ELSE NULL END) AS score_mt
    FROM scoreRows
) AS Subquery
ON ScoreCols.student_id = Subquery.student_id
SET
    ScoreCols.score_en = Subquery.score_en,
    ScoreCols.score_nl = Subquery.score_nl,
    ScoreCols.score_mt = Subquery.score_mt;
```


- 하지만 위의 경우들로는 not null의 경우를 대응할 수 가 없다.
- 이제는 not null도 대응할 수 있게 만들어보자.
- 아래와 같이 바꾸면 여전히 select문을 3번 쓴다. table scan이 3번 일어난다.

```sql
update ScoreColsNN
    set score_en = coalesce( (select score
                                from ScoreRows
                                where student_id = ScoreColsNN.student_id
                                and subject = '영어'),0),
        score_nl = coalesce( (select score
                                from ScoreRows
                                where student_id = ScoreColsNN.student_id
                                and subject = '수학'),0),
        score_mt = coalesce( (select score
                                from ScoreRows
                                where student_id = ScoreColsNN.student_id
                                and subject = '수학'), 0)
where exists (select *  /*처음부터 학생이 존재하지 않는 떄의 null 대응 */
                from ScoreRows 
                where student_id = ScoreColsNN.student_id);
```


- table scan을 1번으로 줄여주었던 이전의 query문을 이용해 null 처리를 해주자.
- where문이 가장 먼저 읽히므로, 거기서 학생 field가 존재하는지 판별해준다. 
  - 학생이 없다면 애초에 update 할 필요 자체가 없다.
- 그리고 나서 update를 할 때 이제는 학생은 있지만, 과목 점수가 null인 경우를 다뤄야 한다.
  - 그 때는 과목 점수가 0인 경우를 coalesce함수로 감싸서 0으로 만들어준다.
- coalesce는 ifnull과 같은데, 어느 RDB에서나 쓸 수 있다. 따라서 해당 함수를 쓰도록 하자.
- coalesce(column , default value)와 같은 형식으로, 해당 record가 null이면 default value로 대체된다.


```sql
update ScoreCols
    set (score_en, score_nl, score_mt)
        = (select  coalesce(max(case when subject = '영어'
                        then score
                        else null end),0) as score_en,
                    coalesce(max(case when subject = '국어'
                        then score
                        else null end),0) as score_nl,
                    coalesce(max(case when subject = '수학'
                        then score
                        else null end),0) as score_mt
        from scoreRows SR
        where SR.student_id = ScoreCols.student_id)
where exists (select *
                    from ScoreRows
                    where student_id = ScoreCols.student_id);
```                    

- mysql에서는 아래와 같다.


```sql
UPDATE ScoreCols
JOIN (
    SELECT
        student_id,
        coalesce(MAX(CASE WHEN subject = '영어' THEN score ELSE NULL END),0) AS score_en,
        coalesce(MAX(CASE WHEN subject = '국어' THEN score ELSE NULL END),0) AS score_nl,
        coalesce(MAX(CASE WHEN subject = '수학' THEN score ELSE NULL END),0) AS score_mt
    FROM scoreRows
) AS Subquery
ON ScoreCols.student_id = Subquery.student_id
SET
    ScoreCols.score_en = Subquery.score_en,
    ScoreCols.score_nl = Subquery.score_nl,
    ScoreCols.score_mt = Subquery.score_mt
where exists (select *
                    from ScoreRows
                    where student_id = ScoreColsNN.student_id); 
```

- 굳이 update를 하지 않아도 된다.
- merge into를 활용하면 된다.
- merge into를 활용하면 향후 코드를 변경할 때 수정 실수도 줄일 수 있다.

```sql
merge into ScoreColsNN
    using (select student_id,
                coalesce(max(case when subject = '영어'
                                    then score
                                    else null end),0) as score_en,
                coalesce(max(case when subject = '국어'
                                    then score
                                    else null end),0) as score_nl,
                coalesce(max(case when subject = '수학'
                                    then score
                                    else null end),0) as score_mt
                from ScoreRows
                group by student_id) SR
    on (ScoreColsNN.student_id = SR.student_id)
when matched then
    update set ScoreColsNN.score_en = SR.score_en,
                ScoreColsNN.score_nl = SR.score_nl,
                ScoreColsNN.score_mt = SR.score_mt;
```


- 이번에는 위와 다르게 record -> field update를 살펴보자.
- 이때는 비교적 단순하다.
- 테이블 접근이 1번이고, 그 또한 기본키 index를 쓴다. join도 없다.

```sql
update ScoreRows
    set score = (select case ScoreRows.subject
                        when '영어' then score_en
                        when '국어' then score_nl
                        when '수학' then score_mt
                        else null
                    end
                from ScoreCols
                where student_id = ScoreRows.student_id);
```

# 한 테이블의 record를 다른 table로 옮기기

- 원 테이블을 두고 다른 테이블에 특정 field를 추가하여 data를 관리하는 경우가 있다.
- 주가가 대표적이다. 
- 브랜드, 거래일, 종가를 기준으로 원 테이블을 관리한다.
- 그리고 그 관리되는 테이블에 가격 상승하락 field를 넣어 추가 테이블을 만든다.(이거 농좆에서는 scrapping이라고 하는듯)

- 그럼 아래와 같이 상관 subquery를 사용할 수 있다.
- 하지만 상관 subquery는 테이블 접근이 많아진다.
  - index range scan(MIN/MAX)- PK index only scan
  - TABLE ACCESS by index ROWID - PK index를 이용한 table scan 
  - table access full - 풀 스캔
- 윈도우 함수로 바꿔주자.

```sql
insert into stock2
select brand,sale_date, price,
    case sign(price - (select price 
                        from Stocks S1
                        where brand = Stocks.brand
                        and sale_date = (select MAX(sale_date)
                                            from Stocks S2
                                            where brand = Stocks.brand
                                            and sale_date < Stocks.sale_date)))
        when -1 then '아래 화살표'
        when 0 then '오른쪽 화살표'
        when 1 then '위 화살표'
        else null
    end
from Stocks;
```

- 윈도우 함수로 바꾸면 아래와 같이 된다.
- 그럼 테이블 접근이 1회로 줄어든다.
    - table access full - 풀 스캔
```sql
insert into Stock2
select brand, sale_date, price,
    case sign(price - MAX(price) over (partition by brand 
                                        order by sale_date
                                    rows between 1 PRECEDING
                                            and 1 PRECEDING))
        when -1 then '아래 화살표'
        when 0 then '오른쪽 화살표'
        when 1 then '위 화살표'
        else null
    end
from Stocks S2;
```

- update로도 가능하다.
  - 하지만 insert select가 update보다 더 빠른 편이다.
  - mysql은 자기참조가 안 되는 DB라서 update로 불가능하다.

- 단점은 복제 테이블이 필요하기 때문에 저장소 용량이 2배가 된다는 점이다.
- 하지만 저장소 늘리는 건 싸기 때문에 성능을 늘리는 게 더 이득이다.
- 