## <span style="color:#802548">_group by 사용_</span>

- group by를 사용하게 되면, group 별로 묶이게 된다.
- group by를 사용하여 avg, count, max, min 등을 출력할 때 편리하다.
- 아래와 같은 count()함수는 group by가 없는데도 작동하는 것을 볼 수 있다.

```sql
select count(*) from employees;
```

- 이 경우에는 group by가 대상이 되는 컬럼이 없이 전체를 세도록 작동한 것이다.
- 따라서 실제 쿼리는 아래처럼 작동한다고 생각하면 되겠다.
- 물론 아래의 구문은 syntax 상으로는 error다.

```sql
select count(*) from employees
group by ();
```

- group이 없을 때, group 함수는 일반 column과 같이 쓰일 수 없다.
- group function은 single row가 나와야 하는데, 일반 column이 성격 상 single row가 아니기 때문이다.
- 실제 일반 column이 지금은 single row라고 해도 error가 난다. 일반 column은 언제든 single row가 아니게 될 수 있다.
- 따라서 성질 상 multiple row이고, 따라서 문법 상으로 multiple row로 보아야 하기 때문에 오류다.

```sql
select 
    release_date as 년도,
    sum(case date_format(release_date, '%w') WHEN 0 THEN 1 ELSE 0 END) AS "일요일",
    sum(case date_format(release_date, '%w') WHEN 1 THEN 1 ELSE 0 END) AS "월요일",
    sum(case date_format(release_date, '%w') WHEN 2 THEN 1 ELSE 0 END) AS "화요일",
    sum(case date_format(release_date, '%w') WHEN 3 THEN 1 ELSE 0 END) AS "수요일",
    sum(case date_format(release_date, '%w') WHEN 4 THEN 1 ELSE 0 END) AS "목요일",
    sum(case date_format(release_date, '%w') WHEN 5 THEN 1 ELSE 0 END) AS "금요일",
    sum(case date_format(release_date, '%w') WHEN 6 THEN 1 ELSE 0 END) AS "토요일"
from box_office
where year(release_date) between 2004 and 2013
-- error
```

- 마찬가지로 general function으로 묶어도 같이 쓰일 수 없다. 
- group function은 single row가 나와야 하는데, general function으로 묶어도 single row가 아니기 때문이다.
- general function은 일반 column의 단순한 가공일 뿐이다. 일반 column과 똑같이 multiple row다.

```sql
select 
    year(release_date) as 년도,
    sum(case date_format(release_date, '%w') WHEN 0 THEN 1 ELSE 0 END) AS "일요일",
    sum(case date_format(release_date, '%w') WHEN 1 THEN 1 ELSE 0 END) AS "월요일",
    sum(case date_format(release_date, '%w') WHEN 2 THEN 1 ELSE 0 END) AS "화요일",
    sum(case date_format(release_date, '%w') WHEN 3 THEN 1 ELSE 0 END) AS "수요일",
    sum(case date_format(release_date, '%w') WHEN 4 THEN 1 ELSE 0 END) AS "목요일",
    sum(case date_format(release_date, '%w') WHEN 5 THEN 1 ELSE 0 END) AS "금요일",
    sum(case date_format(release_date, '%w') WHEN 6 THEN 1 ELSE 0 END) AS "토요일"
from box_office
where year(release_date) between 2004 and 2013;
```

- 위의 구문을 년도별로 보고 싶다면, group으로 묶어서 써야할 필요가 생기는 것이다.

```sql
select 
    year(release_date) as 년도,
    sum(case date_format(release_date, '%w') WHEN 0 THEN 1 ELSE 0 END) AS "일요일",
    sum(case date_format(release_date, '%w') WHEN 1 THEN 1 ELSE 0 END) AS "월요일",
    sum(case date_format(release_date, '%w') WHEN 2 THEN 1 ELSE 0 END) AS "화요일",
    sum(case date_format(release_date, '%w') WHEN 3 THEN 1 ELSE 0 END) AS "수요일",
    sum(case date_format(release_date, '%w') WHEN 4 THEN 1 ELSE 0 END) AS "목요일",
    sum(case date_format(release_date, '%w') WHEN 5 THEN 1 ELSE 0 END) AS "금요일",
    sum(case date_format(release_date, '%w') WHEN 6 THEN 1 ELSE 0 END) AS "토요일"
from box_office
where year(release_date) between 2004 and 2013
group by year(relaes_date);
```


- null 값이 있다면 null값을 포함하여 산출하게 된다.
- 다만 where 절이 먼저 적용되므로 where 조건에서 null 값들은 쓸려 나간다.
    - is not null을 안걸어도 날짜 > 2015 만 해도 null은 전부 걸러지기 때문이다.
- 보통 null 값이 사라진 상태에서 group filtering이 적용되게 된다.

- 아래가 기본 구문이다.

```sql
select
    release_date,
    count(*) from box_office
group by release_date
--OK
```

- group by에 쓰인 column이 반드시 select에 쓰일 필요는 없다.
- 혹시라도 column 이름은 예약어로 만들지 말자. 아래처럼 예약어인 name으로 만들었다간 오류가 난다.

```sql
select
    release_date,
    count(*) from box_office
group by release_date, name
--error
```

- 아래처럼 movie_같이 붙여주자.

```sql
select
    release_date,
    count(*) from box_office
group by release_date, movie_name
--OK
```

- select에 쓰인 일반 column은 반드시 group by절에 있어야 한다.
- select에는 distributor가 있지만 group by에 distributor가 없기 때문에 에러가 난다.

```sql
select
    release_date,
    distributor,
    count(*) from box_office
group by release_date, movie_name
--error
```

- 일반 column을 반드시 써야 한다면 그룹함수로 반드시 만들어서 써야한다.
- max 말고 min, avg로 해도 된다. 문자열이면 grouping 함수는 뭘써도 다 무난하다.
- 다만 숫자의 경우는 불가능하다. max, min, avg 등을 사용하면 원래 의도를 해치기 떄문이다.
- 문자열 변환 후 max를 씌워서 가져오자.

```sql
select
    release_date,
    max(convert(years, char)),
    count(*) from box_office
group by release_date, movie_name
--OK
```

- group function이 아니면 어떤 함수로 묶어도 group by에 없는 column을 활용할 수 없다.
- general function인 concat으로는 일반 column과 성격이 동일하기 때문에 group by에 명시되어야 한다.

```sql
select
    release_date,
    max(convert(years, char)),
    count(*) from box_office,
    concat(years, 'r')
group by release_date, movie_name
-- error
```


## <span style="color:#802548">_group by + case when_</span>

- group by와 case when을 묶으면 강력한 무기가 된다.
- 2004년과 2013년 사이에서 어느 요일에 영화가 많이 개봉했는지 보고자 한다. 
- 모든 row에 대해서 case를 나눠서 일월화수목금토 중 어디에 해당하는 지 찾아낸다.

```sql
select 
    case date_format(release_date, '%w') WHEN 0 THEN 1 ELSE 0 END AS "일요일",
    case date_format(release_date, '%w') WHEN 1 THEN 1 ELSE 0 END AS "월요일",
    case date_format(release_date, '%w') WHEN 2 THEN 1 ELSE 0 END AS "화요일",
    case date_format(release_date, '%w') WHEN 3 THEN 1 ELSE 0 END AS "수요일",
    case date_format(release_date, '%w') WHEN 4 THEN 1 ELSE 0 END AS "목요일",
    case date_format(release_date, '%w') WHEN 5 THEN 1 ELSE 0 END AS "금요일",
    case date_format(release_date, '%w') WHEN 6 THEN 1 ELSE 0 END AS "토요일"
from box_office
where year(release_date) between 2004 and 2013;
```
![alt text](/image/case-when1.png)

- sum을 하여 일월화수목금토에 개봉한 영화가 single row로 count 되어 single row로 출력한다.
- 현재는 group 이 없기 때문에 single row다. group 별로 single row가 출력된다는 사실을 잊지 말자.
- 그냥 무조건적으로 single row를 뽑는 것이 아니다.
    - 만약 만든 그룹이 15개가 된다면, 1개씩 해서 15개의 row가 뽑히게 된다.

```sql
select 
    sum(case date_format(release_date, '%w') WHEN 0 THEN 1 ELSE 0 END) AS "일요일",
    sum(case date_format(release_date, '%w') WHEN 1 THEN 1 ELSE 0 END) AS "월요일",
    sum(case date_format(release_date, '%w') WHEN 2 THEN 1 ELSE 0 END) AS "화요일",
    sum(case date_format(release_date, '%w') WHEN 3 THEN 1 ELSE 0 END) AS "수요일",
    sum(case date_format(release_date, '%w') WHEN 4 THEN 1 ELSE 0 END) AS "목요일",
    sum(case date_format(release_date, '%w') WHEN 5 THEN 1 ELSE 0 END) AS "금요일",
    sum(case date_format(release_date, '%w') WHEN 6 THEN 1 ELSE 0 END) AS "토요일"
from box_office
where year(release_date) between 2004 and 2013;
```
![alt text](/image/case-when2.png)

- 년도별 추이를 보고 싶기 때문에, 년도 조건을 추가해준다. 년도라는 그룹을 만들어주자.
- 그럼 2004부터 2013까지이므로 10개의 group이 생기고, 10개의 row가 보여지게 된다.

```sql
select 
    year(release_date) as 년도,
    sum(case date_format(release_date, '%w') WHEN 0 THEN 1 ELSE 0 END) AS "일요일",
    sum(case date_format(release_date, '%w') WHEN 1 THEN 1 ELSE 0 END) AS "월요일",
    sum(case date_format(release_date, '%w') WHEN 2 THEN 1 ELSE 0 END) AS "화요일",
    sum(case date_format(release_date, '%w') WHEN 3 THEN 1 ELSE 0 END) AS "수요일",
    sum(case date_format(release_date, '%w') WHEN 4 THEN 1 ELSE 0 END) AS "목요일",
    sum(case date_format(release_date, '%w') WHEN 5 THEN 1 ELSE 0 END) AS "금요일",
    sum(case date_format(release_date, '%w') WHEN 6 THEN 1 ELSE 0 END) AS "토요일"
from box_office
where year(release_date) between 2004 and 2013
group by year(release_date);  
```

![alt text](/image/case-when3.png)

- group by를 썼어도 함수를 적용하지 않은 일반 column이 끼어있으면 에러가 난다.

```sql
select 
    year(release_date) as 년도,
    -- format(count(*),0) as 개봉영화수,
    release_date,
    sum(case date_format(release_date, '%w') WHEN 0 THEN 1 ELSE 0 END) AS "일요일",
    sum(case date_format(release_date, '%w') WHEN 1 THEN 1 ELSE 0 END) AS "월요일",
    sum(case date_format(release_date, '%w') WHEN 2 THEN 1 ELSE 0 END) AS "화요일",
    sum(case date_format(release_date, '%w') WHEN 3 THEN 1 ELSE 0 END) AS "수요일",
    sum(case date_format(release_date, '%w') WHEN 4 THEN 1 ELSE 0 END) AS "목요일",
    sum(case date_format(release_date, '%w') WHEN 5 THEN 1 ELSE 0 END) AS "금요일",
    sum(case date_format(release_date, '%w') WHEN 6 THEN 1 ELSE 0 END) AS "토요일"
from box_office
where year(release_date) between 2004 and 2013
group by year(release_date); 
```

![alt text](/image/case-when4.png)


## <span style="color:#802548">_group by + case when + subquery_</span>

- 매출액이 2억 미만인 건들을 count는 해도 매출액에는 포함시키지 않는 조건 같은 것들도 있을 수 있다.
- 이럴 때 1억을 100000000 으로 쓰기보다는 pow(10,8)으로 쓰는 것이 훨씬 가독성이 좋다.
- 아래와 같이 formatting 함수를 이용해 단위를 억으로 바꾸고, 사람이 보기 쉽게 만든다.

```sql
select 
	a.배급사,
    a.개봉수,
    concat(format(round(a.매출)/pow(10,8),0),'억') as 매출,
  --  format(truncate(a.매출, -6), 0) as 매출, 억 단위로 환산하지 않고 숫자 전체를 보여줌. 콧마만 추가
    a.Q1,
    a.Q2,
    a.Q3,
    a.Q4
from (
	select 
		distributor as "배급사",
		sum(case when years = 2016 then 1 else 0 end) as "개봉수",
		sum(case when sale_amt > 200000000 then sale_amt else 0 end) as "매출",
        sum(case when month(release_date) between 1 and 3 then 1 else 0 end) as "Q1",
		sum(case when month(release_date) between 4 and 6 then 1 else 0 end) as "Q2",
		sum(case when month(release_date) between 7 and 9 then 1 else 0 end) as "Q3",
		sum(case when month(release_date) between 10 and 12 then 1 else 0 end) as "Q4"
	from box_office
    where year(release_date) = 2016
	group by distributor
    order by 매출 desc
) a
where pow(10,10) <= 매출 and 매출 <= pow(10,11) * 1.5
```

- order by sum(sale_amt) desc --> 여기다 order by 하면 order by가 제대로 되지 않는다.
- concat으로 문자열로 만들어버렸는데, 별칭이 매출로 똑같으니 실수를 범한 것이다. 
- a.매출로 order by를 해주면 원하는 방식으로 작동함을 볼 수 있다.
- 별칭과 관련된 실수를 늘 조심하자. 문자열된매출로 별칭을 주면 실수를 덜 하게 될 것이다.
- 사실 formatting은 application 단에서 하는 게 더 편하다.

```sql
select 
	a.배급사,
    a.개봉수,
    concat(format(round(a.매출)/pow(10,8),0),'억') as 매출, -- 문자열된매출
  --  format(truncate(a.매출, -6), 0) as 매출, 억 단위로 환산하지 않고 숫자 전체를 보여줌. 콧마만 추가
    a.Q1,
    a.Q2,
    a.Q3,
    a.Q4
from (
	select 
		distributor as "배급사",
		sum(case when years = 2016 then 1 else 0 end) as "개봉수",
		sum(case when sale_amt > 200000000 then sale_amt else 0 end) as "매출",
        sum(case when month(release_date) between 1 and 3 then 1 else 0 end) as "Q1",
		sum(case when month(release_date) between 4 and 6 then 1 else 0 end) as "Q2",
		sum(case when month(release_date) between 7 and 9 then 1 else 0 end) as "Q3",
		sum(case when month(release_date) between 10 and 12 then 1 else 0 end) as "Q4"
	from box_office
    where year(release_date) = 2016
	group by distributor
) a
where pow(10,10) <= 매출 and 매출 <= pow(10,11) * 1.5
order by a.매출 desc
```


- 매출액이 2억 미만인 건들을 count도 제외하고 매출액도 제외하는 조건이라면 좀 더 query가 간단해진다.
- quarter 함수를 사용하면 분기를 뽑아내기 좀 더 간편해진다.

```sql
select distributor as '배급사',
	count(*) as '총개봉수-2016',
    concat(format(sum(sale_amt) / pow(10,8), 0), '억') as '매출-2016', -- pow(10, 8) -> 10의 8승 (1억)
    Sum(Case quarter(release_date) When 1 then 1 else 0 end) as 'Q1',
    Sum(Case quarter(release_date) When 2 then 1 else 0 end) as 'Q2',
    Sum(Case quarter(release_date) When 3 then 1 else 0 end) as 'Q3',
    Sum(Case quarter(release_date) When 4 then 1 else 0 end) as 'Q4',
from box_office
where year(release_date) = 2016 
and sale_amt >= 2 * pow(10,8)
group by distributor
having sum(sale_amt) between 100 * pow(10,8) and 1500 * pow(10,8)
order by sum(sale_amt) desc; -- order by 절에 group by 함수 써서 정렬 가능.
-- month 함수 대신에 quarter로 쓰면 된다. 그럼 분기로 뽑기 쉽다.
```



## <span style="color:#802548">_group by + with rollup_</span>
- group by를 사용한 경우, 총계를 가져오기 위해 with rollup을 사용할 수 있다.

```sql
select 
countrycode,
count(*) as "도시수"
from city
group by CountryCode
with rollup -- 도시수 총계를 보여줌. countrycode는 당연히 null임. 이게 없으면 총계를 보여주는 것이 불가능.
order by 도시수 desc; #1
```

- 단 rollup 기능은 having 절이 있을 때 무력화 된다.
- 따라서 having 절이 있는 경우, 서브쿼리로 풀어야 한다.
    - mysql의 inline view subquery는 반드시 별칭을 주어야 한다.
    - subquery로 푼 뒤, 무의미한 sum 함수와 무의미한 정렬을 반복해주면 된다.
- 즉슨 성능이 지극히 떨어지기 때문에 총계를 java 같은 application 단에서 구하는 게 낫다.

```sql
select 
	a.name,
   sum(도시수) as 도시갯수
from (
	select 
		co.name as name,
        count(ci.name) as 도시수 
	from country co
	inner join city ci
	on co.code = ci.CountryCode
    group by co.name
    having 60 <= count(*) and count(*) <=150
    order by count(*) desc
) a
group by a.name with rollup
order by 2 desc; #4
--having 때문에 막힌것. 그러면 서브쿼리로 넣어서 rollup 쓰면 된다.
```


```sql
select
count(*) as "총국가수",
count(indepyear) as "독립년도가있는국가",
count(*) - count(indepyear) as "독립년도가없는국가"
from country; #3
```

- '%Y%m%d%H%i%s'로는 안되고, '%Y-%m-%d'로 formatting하면 잘 나온다.
- 이유는 잘 모르겠다. formatting이 잘못된 것일수도..

```sql
select 
	years as 년도,
    count(*) as 년도별영화수,
    case when(month(release_date) < 7) then count(*) END as 상반기,
	case when(month(release_date) >= 7) then count(*) END as 하반기
from box_office
where STR_TO_DATE('2004-01-01 00:00:00','%Y-%m-%d') <= release_date and release_date <= STR_TO_DATE('2013-12-31 23:59:59', '%Y-%m-%d')
group by years; #4
```


## <span style="color:#802548">_mysql table info select_</span>
- table info를 select해오는 sql이다.

```sql
SELECT
    table_name, table_comment
FROM
    information_schema.tables
WHERE
    table_schema = 'youdb' AND table_name = 'box_office';
    
```


## <span style="color:#802548">_where clause operator priority_</span>
- where 조건문에서 or와 and를 사용할 때는 and가 우선순위임을 기억해야 한다.
- 그렇지 않으면 의도하지 않은 결과가 나온다.
- 원래 의도한 것은 2015년 이후의 한국, 미국의 값을 가져오는 것이었다.
- 괄호 없이 or와 and를 썼다.

```sql
select 
year(release_date),
count(*),
sum(case when countries like '%한국%' then 1 else 0 end) as 한국회수,
sum(case when countries like '%미국%' then 1 else 0 end) as 미국회수
from box_office
where countries like '%한국%' or 
countries like '%미국%' and 
audience_num >=pow(10,6) and
year(release_date) >= 2015
group by year(release_date)
order by year(release_date) asc; 
```

- 위의 sql은 아래와 같이 작동하게 된다.
- and가 늘 우선순위에 있기 때문이다. or문은 and문이 끝나고 or문이 붙게 되는 형태다.

```sql
select 
year(release_date),
count(*),
sum(case when countries like '%한국%' then 1 else 0 end) as 한국회수,
sum(case when countries like '%미국%' then 1 else 0 end) as 미국회수
from box_office
where countries like '%한국%' or 
(
    countries like '%미국%' and 
    audience_num >=pow(10,6) and
    year(release_date) >= 2015
)
group by year(release_date)
order by year(release_date) asc; 
```

![alt text](/image/and-or-miss.png)


- 전혀 의도한 바가 아니다.
- 그럴 때는 의도한 바대로 해석되게 하기 위해 괄호를 적절하게 사용해준다.

```sql
select 
year(release_date),
count(*),
sum(case when countries like '%한국%' then 1 else 0 end) as 한국회수,
sum(case when countries like '%미국%' then 1 else 0 end) as 미국회수
from box_office
where (
        countries like '%한국%' or 
        countries like '%미국%' 
    )
and audience_num >=pow(10,6) 
and year(release_date) >= 2015
group by year(release_date)
order by year(release_date) asc; 
```

![alt text](/image/and-or-parenthesis.png)