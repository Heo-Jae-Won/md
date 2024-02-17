## <span style="color:#802548">_대회 차수별 직전연도를 알려주세요!_</span>
- 만약 아래와 같은 구조의 테이블이 있다고 해보자.
```
연도|       종류|       차수|       대회코드(pk)
2023|         A|          1|        2023년 A대회 1차
2023|         A|          2|        2023년 A대회 2차
2023|         A|          3|        2023년 A대회 3차
2022|         A|          1|        2022년 A대회 1차
2022|         A|          2|        2022년 A대회 2차
1999|         A|          1|        1999년 A대회 1차
1999|         A|          2|        2023년 A대회 2차
1999|         A|          3|        2023년 A대회 3차
2023|         B|          1|        2023년 B대회 1차
2023|         C|          1|        2023년 C대회 1차
```

- DDL로 우선 만들어주자.
```
CREATE TABLE `대회` (
  `대회코드` varchar(200) NOT NULL,
  `차수` varchar(45) DEFAULT NULL,
  `종류` varchar(45) DEFAULT NULL,
  `연도` varchar(45) DEFAULT NULL,
  PRIMARY KEY (`대회코드`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb3
```

- 여기서 전시즌의 연도를 보고싶다면 어떻게 쿼리문을 짜야할까?
- A대회 3차가 2023년에 일어났고, 그전에는 1999년에 있었다면 1999년이 전시즌이다.
- A대회 2차가 2023년에 일어났고, 그전에는 2022년에 있었다면 2022년이 전시즌이다.
- B대회 1차가 2023년에 일어났고, 그전에는 없었다면 전시즌은 '없음'이다.
```
연도|       종류|       차수|            대회코드(pk)|          전시즌
2023|         A|          1|        2023년 A대회 1차|           2022
2023|         A|          2|        2023년 A대회 2차|           2022
2023|         A|          3|        2023년 A대회 3차|           1999   
2022|         A|          1|        2022년 A대회 1차|           1999
2022|         A|          2|        2022년 A대회 2차|           1999
1999|         A|          1|        1999년 A대회 1차|           없음
1999|         A|          2|        2023년 A대회 2차|           없음
1999|         A|          3|        2023년 A대회 3차|           없음
2023|         B|          1|        2023년 B대회 1차|           없음
2023|         C|          1|        2023년 C대회 1차|           없음
```

- 아래와 같이 짤 수 있다. null로 가져오고 싶다면 ELSE만 NULL로 써주면 된다.
```sql
SELECT
    *,
    CASE
        WHEN EXISTS (
            SELECT 1 
            FROM 대회 t2 
            WHERE t2.종류 = t1.종류 
            AND t2.차수 = t1.차수 
            AND t2.연도 < t1.연도
        ) THEN (
            SELECT MAX(연도)
            FROM 대회 t2 
            WHERE t2.종류 = t1.종류 
            AND t2.차수 = t1.차수 
            AND t2.연도 < t1.연도
        )
        ELSE '없음'
    END AS '전시즌'
FROM 대회 t1
ORDER BY 연도 DESC, 종류, 차수;
```

- subquery로 바꾸면 아래와 같다. CASE WHEN을 쓰는 습관을 들이는 게 더 좋긴 하지만, 이 경우에는 아니다.
- subquery가 더 직관적이면서 실행계획도 더 낫다.
```sql
SELECT
    *,
    (SELECT MAX(연도) 
    FROM your_table t2 
    WHERE t2.종류 = t1.종류 
    AND t2.차수 = t1.차수 
    AND t2.연도 < t1.연도) AS '전시즌'
FROM your_table t1
ORDER BY 연도 DESC, 종류, 차수;
```

- CASE WHEN일 때의 실행계획이다. 
- 상관 서브쿼리가 2개나 있다.
```
1	PRIMARY	            t1		ALL					10	100.00	Using filesort
3	DEPENDENT SUBQUERY	t2		ALL					10	10.00	Using where
2	DEPENDENT SUBQUERY	t2		ALL					10	10.00	Using where
```

- 반면에 명시적으로 서브쿼리를 사용하는 경우다.
- 상관 서브쿼리가 오히려 1개 줄었다. 성능이 더 좋아진 것이다.
```
1	PRIMARY	            t1		ALL					10	100.00	Using filesort
2	DEPENDENT SUBQUERY	t2		ALL					10	10.00	Using where
```