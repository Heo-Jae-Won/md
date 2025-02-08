## <span style="color:#802548">_movie와 review를 따로 가져오기_</span>

- 처음에는 OneToMany를 쓰지 않고 싶어서 아래와 같이 api를 두 번 보내게끔 구성했다.

```java
@GetMapping("/movie/{movieSeq}")
@ResponseBody
public MovieDTO.MovieResponse fetchMovieOne(@PathVariable("movieSeq") String movieSeq) {

    return movieService.findById(Long.valueOf(movieSeq));
}

@GetMapping("/review/list")
@ResponseBody
public List<ReviewDTO.ReviewResponse> fetchReviewList(@RequestParam(name = "movieSeq") String movieSeq) {

    if ( reviewService.findReviewList(Long.valueOf(movieSeq)).isEmpty() ) {
        return  Collections.emptyList();
    }

    return reviewService.findReviewList(Long.valueOf(movieSeq));
}
```

- load가 되기 전에 callMovie가 발동하면 안된다.
    - 따라서 load eventlistener - call function도 async - await로 묶어준다.
    - api call function은 fetch를 return시켜 활용하기 위해 async-await 형태로 만들었다.
    

```js
window.addEventListener('load', async function () {
    await callMovie();
    await callReview();
    async function callMovie() {
        //맨 마지막 query Parameter만 인식됨.
        const path = window.location.pathname;
        const pathParts = path.split('/');
        const movieSeq = pathParts[pathParts.length - 1];

        try {
            const response = await fetch(`/movie/${movieSeq}`, {
                method: 'get'
            })
            const data = await response.json();

            renderMovieHTML(data);
        } catch (error) {
            console.log(error);
            return;
        }
    }

    async function callReview() {
        const path = window.location.pathname;
        const pathParts = path.split('/');
        const movieSeq = pathParts[pathParts.length - 1];

        try {
            const response = await fetch(`/review/list?movieSeq=${movieSeq}`, {
                method: 'get'
            });

            const data = await response.json();

            renderReviewTML(data);
        } catch (error) {
            console.log(error)
        }
    }
})
```

- 위처럼 eventlistener 같은 callback function에는 async await가 필요없다.
- 말그대로 load가 끝나고 난 이후의 callback function이기 때문이다. await가 무의미하다.

```js
window.addEventListener('load',  function () {
     callMovie();
     callReview();
})
```

- 하지만 위의 상황에서는 async - await를 써야한다.
- 그 이유는 fetch 함수가 2개라서 그렇다. 
- load 이후에 eventListener가 작동하기에 문제는 없지만, movie와 review 중 우선순위의 fetch 순서가 있을 것이다.
    - movie가 보이고 나서 댓글이 보이는 게 자연스러우니 async - await를 걸어줘야 한다.
    - 하지만 이렇게 되면 영화를 call해서 받아온 뒤에 callReview가 발동하므로 너무 느려진다.
    

```js
window.addEventListener('load',  async function () {
     await callMovie();
     await callReview();
})
```

- 사실 movie와 review는 같이 api를 날려도 되는데, dom을 movie를 먼저 그리게 하고 싶은 것이다.

```js

```



- 그러면 network 2번, db 2번의 call이 발생한다.


## <span style="color:#802548">_movie과 review 한꺼번에 가져오기_</span>
- 이를 network 1번, db 1번의 call로 줄여보려고 했다.
- 그래서 movie와 review를 한꺼번에 가져오기로 했다.
    - 페이지 상의 로직이 한꺼번에 가져오는 로직이기 때문이다.
    - 영화 디테일 페이지 안에 댓글도 보여주는 형태다.
    - 그런데 Service에서 어느 entity를 return할 지가 좀 애매했다. 둘 다 역할이 있었기 떄문이다.
- 따라서 DTO return하는 것으로 결정하고 진행했다. 
    - movie + review라서 어느 한 entity에 완전히 속하지는 않는다. 따라서 Entity를 활용할 수는 없다.
    - 하지만 movie가 aggregate root이므로 movie의 DTO에 새로 하나를 만들면 된다.
    - 아래와 같이 만들었다.

```java
@Builder
public record MovieReviewDTO(
    MovieDTO.MovieResponse movie,
    List<ReviewDTO.ReviewResponse> reviews
) {
    public static MovieReviewDTO TODTO(MovieDTO.MovieResponse movie, List<ReviewDTO.ReviewResponse> reviews) {
        return MovieReviewDTO.builder()
                                .movie(movie)
                                .reviews(reviews)
                                .build();
    }
}
```

- 그 외 결국 movie + review를 movie controller에 url을 둘지, review에 둘 지 결정해야 한다.
- aggregate root는 movie이므로 movie에 두기로 했다.

```java
@GetMapping("/movie/{movieSeq}/reviews")
@ResponseBody
public MovieReviewDTO fetchMovieWithReviewDTO(@PathVariable("movieSeq") Long movieSeq) {

    return movieService.findMovieWithReviews(movieSeq);
}
```

- Optional로 놓는 이유는 Optional의 map 함수를 활용하기 위함이다.

```java
@Query("SELECT m FROM MovieEntity m LEFT JOIN FETCH m.reviews r WHERE m.movieNum = :movieSeq")
Optional<MovieEntity> findMovieWithReviews(@Param("movieSeq") Long movieSeq);
```

- Optional의 map 함수를 이용해서 MovieReviewDTO를 구성한다.
- 이로써 db는 1번만 call하게 바꿨다.

```java
public MovieReviewDTO findMovieWithReviews(Long movieSeq) {
    return movieRepository.findMovieWithReviews(movieSeq).map(movieEntity -> {
        List<ReviewDTO.ReviewResponse> reviews = movieEntity.getReviews().stream().map(ReviewDTO.ReviewResponse::TODTO).toList();

        return MovieReviewDTO.TODTO(MovieDTO.MovieResponse.TODTO(movieEntity), reviews);
    }).orElseThrow(()->new RuntimeException("no"));
}
```

- front에서도 1번만 call하게 변경한다.
- movie 안에 field가 들어가 있는 형태라서, 아래와 같이 destructuring한다.
- render를 movie따로, reviews 따로 하던것도 하나로 합쳤다.
- insertAdjacentHTML는 무조건 beforeend로 설정한다.

```js
function renderMovieAndReviewsHTML(data) {

    const {
        movie: { movieNum, genre, movieName, movieSummary, movieDate },
        reviews
    } = data;

    document.querySelector("#movie_num").value = movieNum;
    document.querySelector("#genre").textContent = genre;
    document.querySelector("#movie_name").textContent = movieName;
    document.querySelector("#movie_summary").textContent = movieSummary;

    if (reviews.length == 0) {
        document.querySelector("#rating").textContent = "0.0";
    } else {
        document.querySelector("#rating").textContent =
            (reviews.reduce((accumulator, review) => accumulator + review.score, 0) / reviews.length).toFixed(2);
    }

    for (const item of reviews) {
        const { reviewerNickName, reviewText, score, reviewDate } = item;
        const html = `
                <tr>
                    <td>${reviewerNickName}</td>
                    <td>${reviewText}</td>
                    <td>${score}</td>
                    <td>${makeTime(reviewDate)}</td>
                </tr>
        `
        //무조건 beforeend, beforebegin으로 하면 tbody가 id가 없는 것으로 매번 새로 생성됨.
        document.querySelector("#generate_area").insertAdjacentHTML('beforeend', html);
    }

}
```


## <span style="color:#802548">_MovieEntity에서 필요없는 column 제외하여 가져오기_</span>
- 그런데 사실 movieDate는 movie detail에 필요하지 않다.
- JPARepository는 무조건 * select를 하게 되어있다. 
- 이를 JPA Projection을 통해 변경해보자.
- movie 안에서 review가 있는 구조이므로 아래처럼 nested projection을 작성한다.

```java
public interface MovieReviewProjection {
    Long getMovieNum();
    String getMovieName();
    String getGenre();
    String getMovieSummary();
    
    List<ReviewProjection> getReviews();
    
    interface ReviewProjection {
        Long getReviewNum();
        String getReviewerNickName();
        String getReviewText();
        Integer getScore();
    }
}
```

- DTO로 만드는 건 동일하기 때문에 오버로딩을 이용해 똑같은 method명으로 만든다.

```java
@Builder
public static record MovieResponse(Long movieNum, String genre, String movieName, String movieSummary, LocalDateTime movieDate) {
    public static MovieResponse TODTO(MovieEntity movieEntity) {
        return MovieResponse.builder()
                            .movieNum(movieEntity.getMovieNum())
                            .genre(movieEntity.getGenre())
                            .movieDate(movieEntity.getMovieDate())
                            .movieName(movieEntity.getMovieName())
                            .movieSummary(movieEntity.getMovieSummary())
                            .build();
    }

    public static MovieResponse TODTO(MovieReviewProjection movieReviewProjection) {
        return MovieResponse.builder()
                            .genre(movieReviewProjection.getGenre())
                            .movieName(movieReviewProjection.getMovieName())
                            .movieSummary(movieReviewProjection.getMovieSummary())
                            .movieNum(movieReviewProjection.getMovieNum())
                            .build();

    }
    

}
////////////////////////////////////////////////
@Builder
    public static record ReviewResponse(Long reviewNum, String reviewerNickName, String reviewText, Integer score, LocalDateTime reviewDate) {
        public static ReviewResponse TODTO(ReviewEntity reviewEntity) {
            return ReviewResponse.builder()
                                    .reviewNum(reviewEntity.getReviewNum())
                                    .reviewerNickName(reviewEntity.getReviewerNickName())
                                    .reviewText(reviewEntity.getReviewText())
                                    .reviewerNickName(reviewEntity.getReviewerNickName())
                                    .score(reviewEntity.getScore())
                                    .reviewDate(reviewEntity.getReviewDate())
                                    .build();
        }

        public static ReviewResponse TODTO(MovieReviewProjection.ReviewProjection reviewProjection) {
            return ReviewResponse.builder()
                                    .reviewNum(reviewProjection.getReviewNum())
                                    .reviewerNickName(reviewProjection.getReviewerNickName())
                                    .reviewText(reviewProjection.getReviewText())
                                    .reviewerNickName(reviewProjection.getReviewerNickName())
                                    .score(reviewProjection.getScore())
                                    .reviewDate(reviewProjection.getReviewDate())
                                    .build();
        }
    }
```

- 그런데 repository에서 entity로 가져오면 의미가 없다. entity로는 projection을 써도 all fetch가 진행된다.
- 따라서 select field를 전부 명시한다.
- 

```java
// @Query("SELECT m FROM MovieEntity m LEFT JOIN FETCH m.reviews r WHERE m.movieNum = :movieSeq")
// Optional<MovieReviewProjection> findMovieWithReviewsProjections(@Param("movieSeq") Long movieSeq);

@Query("SELECT m.movieNum AS movieNum, \n" + //
        "           m.movieName AS movieName, \n" + //
        "           m.genre AS genre, \n" + //
        "           m.movieSummary AS movieSummary, \n" + //
        "           r.reviewNum AS reviews_reviewNum, \n" + //
        "           r.reviewerNickName AS reviews_reviewerNickName, \n" + //
        "           r.reviewText AS reviews_reviewText, \n" + //
        "           r.score AS reviews_score,\n" + //
        "           r.reviewDate AS reviews_reviewDate\n" + //
        "    FROM MovieEntity m \n" + //
        "    LEFT JOIN m.reviews r \n" + //
        "    WHERE m.movieNum = :movieSeq")
Optional<MovieReviewProjection> findMovieWithReviewsProjections(@Param("movieSeq") Long movieSeq);
////////////////////////////////////// jdk 15version은 아래처럼.
@Query("""
    SELECT m.movieNum AS movieNum, 
           m.movieName AS movieName, 
           m.genre AS genre, 
           m.movieSummary AS movieSummary, 
           r.reviewNum AS reviews_reviewNum, 
           r.reviewerNickName AS reviews_reviewerNickName, 
           r.reviewText AS reviews_reviewText, 
           r.score AS reviews_score,
           r.reviewDate AS reviews_reviewDate
    FROM MovieEntity m 
    LEFT JOIN m.reviews r 
    WHERE m.movieNum = :movieSeq
""")
Optional<MovieReviewProjection> findMovieWithReviewsProjections(@Param("movieSeq") Long movieSeq);
```

- findMovieWithReviews()를 만들듯 똑같이 만들어준다.

```java
public MovieReviewDTO findMovieWithReviewsProjection(Long movieSeq) {
    return movieRepository.findMovieWithReviewsProjections(movieSeq).map(movieProjection -> {
        List<ReviewDTO.ReviewResponse> reviews = movieProjection.getReviews().stream().map(ReviewDTO.ReviewResponse::TODTO).toList();
        MovieDTO.MovieResponse movie = MovieDTO.MovieResponse.TODTO(movieProjection);

        return MovieReviewDTO.TODTO(movie, reviews);
    }).orElseThrow(()->new RuntimeException("no such movie + review"));
}
```

- 원하는대로 query는 분명 movie_date column을 안가져온다.
- 하지만 아래와 같은 오류가 나버린다.

```
org.hibernate.NonUniqueResultException: Query did not return a unique result: 11 results were returned
```

- 이유는 projection은 List를 다룰수 없기 때문이다.
- left join으로 multiple 갯수가 나오는데, MovieReviewProjection은 1개의 객체인데, collection 객체도 없다.
- projection을 쓰기 전에는 JPA가 collection을 보고 mapping을 자동으로 해줬는데, 여기서는 그게 안된다.

```java
public interface MovieReviewProjection {
    Long getMovieNum();
    String getMovieName();
    String getGenre();
    String getMovieSummary();
    
    List<ReviewProjection> getReviews();
    
    interface ReviewProjection {
        Long getReviewNum();
        String getReviewerNickName();
        String getReviewText();
        Integer getScore();
        LocalDateTime getReviewDate();
    }
}
```

- 따라서 repository에서 Optional이 아닌 List로 type을 바꿔준다.
- 여기서 SELECT DISTINCT로 바꿔도 오류는 동일하게 난다.

```java
@Query("""
        SELECT DISTINCT  m.movieNum AS movieNum, 
               m.movieName AS movieName, 
               m.genre AS genre, 
               m.movieSummary AS movieSummary, 
               r.reviewNum AS reviews_reviewNum, 
               r.reviewerNickName AS reviews_reviewerNickName, 
               r.reviewText AS reviews_reviewText, 
               r.score AS reviews_score,
               r.reviewDate AS reviews_reviewDate
        FROM MovieEntity m 
        LEFT JOIN m.reviews r 
        WHERE m.movieNum = :movieSeq
    """)
    List<MovieReviewProjection> findMovieWithReviewsProjections(@Param("movieSeq") Long movieSeq);
```

- List로 하면 review가 null인 오류가 난다.
- mapping이 안되는 것으로 보인다. 1 대 다 는 지원해주지 않는것 같다.
- 공식문서에는 제대로 설명이 없어서 정확힌 모르겠지만, 뭘 해도 null이다..

- 그래서 아래처럼 사용해보았다.

```java
public interface MovieReviewProjection {
    Long getMovieNum();
    String getMovieName();
    String getGenre();
    String getMovieSummary();
    Long getReviewNum();
    String getReviewerNickName();
    String getReviewText();
    Integer getScore();
    LocalDateTime getReviewDate();
}
```

- 위처럼 바꿨으면 parsing 방법도 달라져야 한다. 전부 따로 모아주자.
- 영화는 1개만 있으면 되므로 1개만 가져온다.
- 댓글은 전부 List로 모아준다.

```java
public MovieReviewDTO findMovieWithReviewsProjection(Long movieSeq) {
    List<MovieReviewProjection> projections = movieRepository.findMovieWithReviewsProjection(movieSeq);

    if (projections.isEmpty()) {
        throw new RuntimeException("No such movie + review");
    }

    MovieReviewProjection firstRow = projections.get(0);
    MovieDTO.MovieResponse movie = MovieDTO.MovieResponse.TODTO(firstRow);

    List<ReviewDTO.ReviewResponse> reviews = projections.stream().map(ReviewDTO.ReviewResponse::TODTO).toList();

    return MovieReviewDTO.TODTO(movie, reviews);
}
```

- 그러나 여전히 null 오류가 난다.
- 아무래도 1 : n entity @OneToMany를 참고하기 때문에 mapping이 안되는 것이 그대로 적용되고 있는 듯하다.
- 따라서 JPQL 식을 바꿔주자.
- LEFT JOIN의 대상은 movieEntity의 @OneToMany로 참고되고 있는 reviews field가 아니다.
- 별개의 ReviewEntity를 끌고와서 join시킨다.

```java
@Query("""
                SELECT m.movieNum AS movieNum,
                       m.movieName AS movieName,
                       m.genre AS genre,
                       m.movieSummary AS movieSummary,
                       r.reviewNum AS reviewNum,
                       r.reviewerNickName AS reviewerNickName,
                       r.reviewText AS reviewText,
                       r.score AS score,
                       r.reviewDate AS reviewDate
                FROM MovieEntity m
                LEFT JOIN ReviewEntity r ON m.movieNum = r.movieEntity.movieNum
                WHERE m.movieNum = :movieSeq
            """)
    List<MovieReviewProjection> findMovieWithReviewsProjection(@Param("movieSeq") Long movieSeq);
```


- 위와같이 진행하게 되면 null 오류 없이 전부 제대로 원하는 컬럼만 딱딱 parsing되어 나온다.
- 정리하면 아래와 같다. interface로 쓰는 projection은 사실상 DTO나 다름없다.
- 쿼리문도 객체 받아오는 것뿐이지 사실상 nativeQuery랑 생긴게 똑같게 된다.
- 네트워크 상의 불필요한 column들을 피하려 하다보면 사실상 거의 mybatis처럼 쓰게 된다.



## <span style="color:#802548">_댓글 등록 후 새로 화면 받아오는 로직 변경_</span>
- 기존의 댓글 등록 후 새로운 댓글 노출은 아래 형태로 다시 reload였다.

```js
 fetch('/review/create', {
    method: 'post',
    body: params
}).then(async (response) => {
    location.href = `/viewMoovie/${movieSeq}`
}).catch((error) => {
    console.log(error);
})
```

- 이걸 순수하게 다시 그리는 것으로 완전하게 바꾸자.
- 그를 위해선 완료 후 다시 api를 조회하는 대신, 댓글 부분의 DOM을 매번 지워주면 된다.
- 이전 댓글들 DOM은 js로 전부 지워진다.
- 아까 insertAdjacentHTML에서 beforeend를 하지 않았다면 제대로 지워지지 않게 될 것이다.
    - beforebegin을 하게 되면 내가 지정한 id 영역의 tbody가 아니라 완전히 새로운 tbody가 생성되어버린다.

```js

function renderMovieAndReviewsHTML(data) {
    const children = Array.from(document.querySelector("#generate_area").childNodes);  // Get all child nodes (including text nodes)
    children.forEach(child => {
        document.querySelector("#generate_area").removeChild(child);  // Remove each child node
    });
    .
    .
    .
    //무조건 beforeend, beforebegin으로 하면 tbody가 id가 없는 것으로 매번 새로 생성됨.
    document.querySelector("#generate_area").insertAdjacentHTML('beforeend', html);
}
    
```

- 그럼 이제 댓글을 달고 나서 하는 callback을 새로 api를 받아오는 것으로 변경한다.
- 여기선 async, await를 안붙였다. then으로 callback으로 받게 명시되어있기 때문에 그러하다.

```js
 fetch('/review/create', {
    method: 'post',
    body: params
}).then( (response) => {
     callMovieWithReviews();
}).catch((error) => {
    console.log(error);
})
```

## <span style="color:#802548">_left join과 left join fetch_</span>

- left join fetch는 oneToMany나 ManyToMany로 엮인 collection을 early fetching하는 용도다.
- 다시말하면 MovieEntity가 값이 채워질 떄 바로 그 field인 reviews(ReviewEntity)도 값이 채워짐을 의미한다.

```java
@Query("SELECT m FROM MovieEntity m LEFT JOIN FETCH m.reviews r WHERE m.movieNum = :movieSeq")
Optional<MovieEntity> findMovieWithReviews(@Param("movieSeq") Long movieSeq);
```

- 이는 query문의 차이로 드러나게 된다. 한번에 load되려면 전부 한번에 가지고 오게 sql을 한번만 보내게 된다.

```sql
select
        me1_0.movie_num,
        me1_0.genre,
        me1_0.movie_date,
        me1_0.movie_name,
        me1_0.movie_summary,
        r1_0.movie_num,
        r1_0.review_num,
        r1_0.review_date,
        r1_0.review_text,
        r1_0.reviewer_nickname,
        r1_0.score 
    from
        movie me1_0 
    left join
        review r1_0 
            on me1_0.movie_num=r1_0.movie_num 
    where
        me1_0.movie_num=?
```

- 반면에 left join fetch를 주지 않으면, Eager로 걸어놓든 어떻든 lazy loading되어 나중에 필요할 때 load하기 때문에, 2번의 query가 발생해버린다.
- 물론 entity의 값은 모두 차있는 상태다.

```java
@Query("SELECT m FROM MovieEntity m LEFT JOIN m.reviews r WHERE m.movieNum = :movieSeq")
Optional<MovieEntity> findMovieWithReviews(@Param("movieSeq") Long movieSeq);
```

- dataset이 지나치게 많으면 fetch를 붙이지 않고 pagination을 해서 가져오는 게 요구될 수 있다.
- fetch는 한꺼번에 모든 resultset을 가져오기 때문에 대용량을 가져올 때는 적합하지 않다.

```sql
select
    me1_0.movie_num,
    me1_0.genre,
    me1_0.movie_date,
    me1_0.movie_name,
    me1_0.movie_summary 
from
    movie me1_0 
left join
    review r1_0 
        on me1_0.movie_num=r1_0.movie_num 
where
    me1_0.movie_num=?
Hibernate: 
select
    r1_0.movie_num,
    r1_0.review_num,
    r1_0.review_date,
    r1_0.review_text,
    r1_0.reviewer_nickname,
    r1_0.score 
from
    review r1_0 
where
    r1_0.movie_num=?
```

- 또한 projection에서는 fetch 기능을 쓸수가 없다. 
- collection을 다루지 않기 때문이다.

```java
@Query("""
    SELECT m.movieNum AS movieNum,
            m.movieName AS movieName,
            m.genre AS genre,
            m.movieSummary AS movieSummary,
            r.reviewNum AS reviewNum,
            r.reviewerNickName AS reviewerNickName,
            r.reviewText AS reviewText,
            r.score AS score,
            r.reviewDate AS reviewDate
    FROM MovieEntity m
    LEFT JOIN ReviewEntity r ON m.movieNum = r.movieEntity.movieNum
    WHERE m.movieNum = :movieSeq
""")
List<MovieReviewProjection> findMovieWithReviewsProjection(@Param("movieSeq") Long movieSeq);
```


## <span style="color:#802548">_HTML parsing 빠르게 하는 script 배치_</span>
- 원래 형태는 아래와 같다.
- 이렇게하면 HTML을 읽을 때 script가 들어와서 방해받게 된다.

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script src="/js/util.js"></script>
    <script>
        window.addEventListener('load',  function () {

             callMovieWithReviews();

            document.querySelector("#regist").addEventListener('click', function () {
                const movieSeq = document.querySelector("#movie_num").value;
                const reviewerNickName = document.querySelector("#reviewerNickName").value;
                const reviewText = document.querySelector("#reviewText").value;
                const score = document.querySelector("#score").value;

                if (reviewerNickName.trim().length == 0) {
                    alert("닉네임을 입력하세요.");
                    return;
                }

                if (reviewText.length < 10) {
                    alert("리뷰 내용이 너무 짧습니다.");
                    return;
                }

                //parseInt쓰면 int화 돼서 1.1로 입력되도 1로 처리되어 제대로 안됨.
                if (!isValidNumber(score)) {
                    alert("숫자는 0부터 10사이의 정수만 가능합니다.")
                    return;
                }

                const params = new URLSearchParams();
                params.append("movieSeq", movieSeq)
                params.append("reviewerNickName", reviewerNickName)
                params.append("reviewText", reviewText)
                params.append("score", score)


                fetch('/review/create', {
                    method: 'post',
                    body: params
                }).then( (response) => {
                     callMovieWithReviews();
                }).catch((error) => {
                    console.log(error);
                })
            })

        })

        async function callMovieWithReviews() {
            //맨 마지막 query Parameter만 인식됨.
            const path = window.location.pathname;
            const pathParts = path.split('/');
            const movieSeq = pathParts[pathParts.length - 1];

            try {
                const response = await fetch(`/movie/${movieSeq}/reviews`, {
                    method: 'get'
                })
                const data = await response.json();

                renderMovieAndReviewsHTML(data);
            } catch (error) {
                console.log(error);
                return;
            }
        }


        function renderMovieAndReviewsHTML(data) {
            const children = Array.from(document.querySelector("#generate_area").childNodes);  // Get all child nodes (including text nodes)
            children.forEach(child => {
                document.querySelector("#generate_area").removeChild(child);  // Remove each child node
            });

            const {
                movie: { movieNum, genre, movieName, movieSummary, movieDate },
                reviews
            } = data;

            document.querySelector("#movie_num").value = movieNum;
            document.querySelector("#genre").textContent = genre;
            document.querySelector("#movie_name").textContent = movieName;
            document.querySelector("#movie_summary").textContent = movieSummary;

            if (reviews.length == 0) {
                document.querySelector("#rating").textContent = "0.0";
            } else {
                document.querySelector("#rating").textContent =
                    (reviews.reduce((accumulator, review) => accumulator + review.score, 0) / reviews.length).toFixed(2);
            }

            for (const item of reviews) {
                const { reviewerNickName, reviewText, score, reviewDate } = item;
                const html = `
                        <tr>
                            <td>${reviewerNickName}</td>
                            <td>${reviewText}</td>
                            <td>${score}</td>
                            <td>${makeTime(reviewDate)}</td>
                        </tr>
                `
                //무조건 beforeend, beforebegin으로 하면 tbody가 id가 없는 것으로 매번 새로 생성됨.
                document.querySelector("#generate_area").insertAdjacentHTML('beforeend', html);
            }

        }
    </script>
</head>

<body>
    <input type="hidden" id="movie_num">
    <h2><span id="movie_name"></span>에 대한 리뷰 정보입니다.</h2>
    장르: <span id="genre"></span> <br>
    영화설명: <br>
    <div id="movie_summary"></div>
    관객평점: <span id="rating"></span>
    <br>

    <table>
        <thead>
            <tr>
                <td>닉네임</td>
                <td>리뷰</td>
                <td>점수</td>
                <td>등록일</td>
            </tr>
        </thead>
        <tbody id="generate_area">
        </tbody>
    </table>


    내 평점은요... <br>
    닉네임: <input type="text" name="reviewerNickName" id="reviewerNickName"><br>
    리뷰:
    <textarea name="reviewText" id="reviewText"></textarea><br>
    내 평점: <input type="text" name="score" id="score">

    <button type="button" id="regist">등록하기</button>
</body>

</html>
```


- html에서 head에 외부 script src를 가져오는 경우는, HTML의 parsing을 막지 않기 위해 defer를 꼭 사용하자.
- inline script는 defer가 애초에 적용되지 않으니 defer가 필요없다.
    - 대신 이것도 head에 있으면 HTML parsing을 느리게 하는 것은 매한가지다.
    - 따라서 body 바로 위에 써주는 게 좋다.
    - 또한 src를 적을 때는 web상에서 붙는 것을 전제로 생각해야 한다.
        - 다시말하면, /src가 아니라 web상에서는 /src/main/resources/static 까지의 경로는 기본으로 붙는다.
        - 따라서 /js/util.js로 쓰면된다.

```html
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <script defer src="/js/util.js"></script>
</head>

<body>
    <input type="hidden" id="movie_num">
    <h2><span id="movie_name"></span>에 대한 리뷰 정보입니다.</h2>
    장르: <span id="genre"></span> <br>
    영화설명: <br>
    <div id="movie_summary"></div>
    관객평점: <span id="rating"></span>
    <br>

    <table>
        <thead>
            <tr>
                <td>닉네임</td>
                <td>리뷰</td>
                <td>점수</td>
                <td>등록일</td>
            </tr>
        </thead>
        <tbody id="generate_area">
        </tbody>
    </table>


    내 평점은요... <br>
    닉네임: <input type="text" name="reviewerNickName" id="reviewerNickName"><br>
    리뷰:
    <textarea name="reviewText" id="reviewText"></textarea><br>
    내 평점: <input type="text" name="score" id="score">

    <button type="button" id="regist">등록하기</button>

    <script>
        window.addEventListener('load',  function () {

             callMovieWithReviews();

            document.querySelector("#regist").addEventListener('click', function () {
                const movieSeq = document.querySelector("#movie_num").value;
                const reviewerNickName = document.querySelector("#reviewerNickName").value;
                const reviewText = document.querySelector("#reviewText").value;
                const score = document.querySelector("#score").value;

                if (reviewerNickName.trim().length == 0) {
                    alert("닉네임을 입력하세요.");
                    return;
                }

                if (reviewText.length < 10) {
                    alert("리뷰 내용이 너무 짧습니다.");
                    return;
                }

                //parseInt쓰면 int화 돼서 1.1로 입력되도 1로 처리되어 제대로 안됨.
                if (!isValidNumber(score)) {
                    alert("숫자는 0부터 10사이의 정수만 가능합니다.")
                    return;
                }

                const params = new URLSearchParams();
                params.append("movieSeq", movieSeq)
                params.append("reviewerNickName", reviewerNickName)
                params.append("reviewText", reviewText)
                params.append("score", score)


                fetch('/review/create', {
                    method: 'post',
                    body: params
                }).then( (response) => {
                     callMovieWithReviews();
                }).catch((error) => {
                    console.log(error);
                })
            })

        })

        async function callMovieWithReviews() {
            //맨 마지막 query Parameter만 인식됨.
            const path = window.location.pathname;
            const pathParts = path.split('/');
            const movieSeq = pathParts[pathParts.length - 1];

            try {
                const response = await fetch(`/movie/${movieSeq}/reviews`, {
                    method: 'get'
                })
                const data = await response.json();

                renderMovieAndReviewsHTML(data);
            } catch (error) {
                console.log(error);
                return;
            }
        }


        function renderMovieAndReviewsHTML(data) {
            const children = Array.from(document.querySelector("#generate_area").childNodes);  // Get all child nodes (including text nodes)
            children.forEach(child => {
                document.querySelector("#generate_area").removeChild(child);  // Remove each child node
            });

            const {
                movie: { movieNum, genre, movieName, movieSummary, movieDate },
                reviews
            } = data;

            document.querySelector("#movie_num").value = movieNum;
            document.querySelector("#genre").textContent = genre;
            document.querySelector("#movie_name").textContent = movieName;
            document.querySelector("#movie_summary").textContent = movieSummary;

            if (reviews.length == 0) {
                document.querySelector("#rating").textContent = "0.0";
            } else {
                document.querySelector("#rating").textContent =
                    (reviews.reduce((accumulator, review) => accumulator + review.score, 0) / reviews.length).toFixed(2);
            }

            for (const item of reviews) {
                const { reviewerNickName, reviewText, score, reviewDate } = item;
                const html = `
                        <tr>
                            <td>${reviewerNickName}</td>
                            <td>${reviewText}</td>
                            <td>${score}</td>
                            <td>${makeTime(reviewDate)}</td>
                        </tr>
                `
                //무조건 beforeend, beforebegin으로 하면 tbody가 id가 없는 것으로 매번 새로 생성됨.
                document.querySelector("#generate_area").insertAdjacentHTML('beforeend', html);
            }

        }
    </script>
</body>

</html>
```


## <span style="color:#802548">_review와 movie 재고려_</span>
- 하지만 위의 고친 방식은 나름대로 문제가 있었다.
- 기존에는 댓글을 달면 댓글만 조회하는 sql을 실행했다.

```sql
select * from review where movie_seq = '?';
```

- 하지만 이제는 영화는 바뀌지 않는데도 매번 left join으로 영화를 가져와야 한다.
    - 바뀌지 않는 영화를 가져오려고 left join을 해야하는 것도 우습다.
    - 더 큰 문제는 left join으로 인한 성능 저하다.
- 결국 페이지 로딩에서 비효율적인 대신, 댓글활동이 활발할수록 이득인 첫번쨰
- 첫 init 페이지 로딩에서 효율적인 대신, 댓글활동이 활발할수록 손해인 두번째 옵션이었던 셈이다.
    - 댓글을 적는 사용자는 적다.
    - 여러 댓글을 연달아서 작성하는 사용자는 더더욱 적다. 
- 결론적으로 첫번째보단 두번째가 더 컴퓨팅 자원을 아낄 수 있다고 생각이 든다.


