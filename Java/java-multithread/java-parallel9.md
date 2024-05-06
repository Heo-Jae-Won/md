- webClient를 이용하면 아래와 같이도 쓸 수 있다.
- retrieveMovie()는 blocking I/O call이다.
```java
public class MoviesClient {

    private final WebClient webClient;

    public MoviesClient(WebClient webClient) {
        this.webClient = webClient;
    }

    public Movie retrieveMovie(Long movieInfoId) {
        var movieInfo = invokeMovieInfoService(movieInfoId);
        var reviews = invokeReviewsService(movieInfoId);
        return new Movie(movieInfo, reviews);
    }

    private List<Review> invokeReviewsService(Long movieInfoId) {
        String uri = UriComponentsBuilder.fromUriString("/v1/       reviews") 
                .queryParam("movieInfoId", movieInfoId)
                .buildAndExpand()
                .toUriString();

        List<Review> reviews = webClient.get().uri(uri)
                .retrieve()
                .bodyToFlux(Review.class)  //bodyToFlux는 multi value. 따라서 List인 것.
                .collectList()
                .block();

        return reviews;
    }


    public MovieInfo invokeMovieInfoService(Long movieInfoId) {

        var movieInfoUrlPath = "/v1/movie_infos/{movieInfoId}";

        var movieInfo = webClient.get()
                .uri(movieInfoUrlPath, movieInfoId)
                .retrieve()
                .bodyToMono(MovieInfo.class)//single은 bodyToMono
                .block(); //데이터를 가져옴.

        return movieInfo;
    }
}
```



- test case다.
- baseURL을 정하고서 webClient 객체를 만든다.
- 그 뒤에 retrieveMovie나 Review, MovieInfo 등을 한다.
```java
@Disabled
class MoviesClientTest {

    WebClient webClient = WebClient.builder().baseUrl("http://localhost:8080/movies").build();
    MoviesClient moviesClient = new MoviesClient(webClient);


    @BeforeEach
    void setUpMoviesClient() {
        var movieInfoId = 1L;
        moviesClient.retrieveMovie(movieInfoId);
    }

    @Test
    @RepeatedTest(10)
    void retrieveMovie() {
        CommonUtil.startTimer();
        //given
        var movieInfoId = 1L;

        //when

        var movie = moviesClient.retrieveMovie(movieInfoId);


        //then
        assertNotNull(movie);
        assertEquals("Batman Begins", movie.getMovieInfo().getName());
        assertEquals(1, movie.getReviewList().size() );

        CommonUtil.timeTaken();
    }
}
```


- blocking이 아니라 async하게 만들어보자.
- invokeReviewsService와 invokeMovieInfoService는 바뀌지 않는다.
- retrieveMovie()만 CompletableFuture를 return하고 사용하게 바뀐다.
```java
public class MoviesClient {

    private final WebClient webClient;

    public MoviesClient(WebClient webClient) {
        this.webClient = webClient;
    }

    public CompletableFuture<Movie> retrieveMovie_CF(Long movieInfoId) {

        var movieInfo = CompletableFuture.supplyAsync(() -> invokeMovieInfoService(movieInfoId));
        var reviews = CompletableFuture.supplyAsync(() -> invokeReviewsService(movieInfoId));
        return movieInfo.thenCombine(reviews, Movie::new); //review와 movieinfo 두 결과를 합침.
    }

    private List<Review> invokeReviewsService(Long movieInfoId) {
        String uri = UriComponentsBuilder.fromUriString("/v1/       reviews") 
                .queryParam("movieInfoId", movieInfoId)
                .buildAndExpand()
                .toUriString();

        List<Review> reviews = webClient.get().uri(uri)
                .retrieve()
                .bodyToFlux(Review.class)  //bodyToFlux는 multi value. 따라서 List인 것.
                .collectList()
                .block();

        return reviews;
    }


    public MovieInfo invokeMovieInfoService(Long movieInfoId) {

        var movieInfoUrlPath = "/v1/movie_infos/{movieInfoId}";

        var movieInfo = webClient.get()
                .uri(movieInfoUrlPath, movieInfoId)
                .retrieve()
                .bodyToMono(MovieInfo.class)//single은 bodyToMono
                .block(); //데이터를 가져옴.

        return movieInfo;
    }
}
```


- CF를 이용해 만든 webFlux에 대해서도 test case를 만들자.
- test case의 로직은 거의 바뀌지 않는다. 다만 CF를 이용해서 확인하려는 것은 반응속도다. CF가 시간이 적게 걸린다는 것을 알 수 있다.
- 한 번으로는 모르니까 @RepeatedTest로 해서 잰다.
- 처음에는 connection을 맺으니 700ms 정도가 걸리지만 그 이후로는 15ms정도 걸린다.
```java
@Test
@RepeatedTest(10)
void retrieveMovie_CF() {
    //given
    var movieInfoId = 1L;

    //when
    CommonUtil.startTimer();
    var movie = moviesClient.retrieveMovie_CF(movieInfoId).join();
    CommonUtil.timeTaken();
    //then
    assert movie!=null;
    assertEquals("Batman Begins", movie.getMovieInfo().getName());
    assert movie.getReviewList().size() == 1;


}
```

- 위는 전부 그냥 Moview를 return했다.
- 하지만 List(Movie)를 return하는 경우는 어떻게 할까?
- sequential하게는 stream으로 열고 CF를 이용안한 get method를 가져온다.
- parallel하게는 parallelStream으로 열고 CF를 이용한 get method를 가져온다.
```java
public class MoviesClient {

    private final WebClient webClient;

    public MoviesClient(WebClient webClient) {
        this.webClient = webClient;
    }
    public List<Movie> retrieveMovieList(List<Long> movieInfoIds) {

        return movieInfoIds
                .stream()
                .map(this::retrieveMovie)
                .collect(Collectors.toList());

    }

    public List<Movie> retrieveMovieList_CF(List<Long> movieInfoIds) {

        var movieFutures = movieInfoIds
                .stream()
                .parallel()
                .map(this::retrieveMovie_CF)
                .collect(Collectors.toList());


        return movieFutures
                .stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList());

    }
}
```


- test case는 아래와 같다.
- blocking call이 있기 때문에 CF가 더 빠르다.
```java
@Test
@RepeatedTest(10)
void retrieveMovieList() {
    //given
    var movieInfoIds = List.of(1L,2L, 3L , 4L, 5L, 6L, 7L);

    //when
    CommonUtil.startTimer();
    var movies = moviesClient.retrieveMovieList(movieInfoIds);
    System.out.println("movies : " + movies);
    CommonUtil.timeTaken();

    //then
    assert movies.size() == 7;
}


@Test
void retrieveMovieList_CF() {
    //given
    var movieInfoIds = List.of(1L,2L, 3L , 4L, 5L, 6L, 7L);

    //when
    CommonUtil.startTimer();
    var movies = moviesClient.retrieveMovieList_CF(movieInfoIds);

    System.out.println("movies : " + movies);
    CommonUtil.timeTaken();
    //then
    assert movies.size() == 7;

}
```


- 여기서 더 성능이 좋아질 방법이 있다. 
- service에서 각 future마다 join하는 것을 한꺼번에 진행하는 것이다.
- join method가 여전히 쓰이고는 있지만, CompletableFuture.allOf로 인해 thenApply()가 모든 CompletableFuture가 전부 완료되어야 호출되기 때문이다.
```java
public class MoviesClient {

    private final WebClient webClient;

    public MoviesClient(WebClient webClient) {
        this.webClient = webClient;
    }

    public List<Movie> retrieveMovieList(List<Long> movieInfoIds) {

        return movieInfoIds
                .stream()
                .map(this::retrieveMovie)
                .collect(Collectors.toList());

    }

    public List<Movie> retrieveMovieList_CF_allOf(List<Long> movieInfoIds) {

        var movieFutures = movieInfoIds
                .stream()
                .parallel()
                .map(this::retrieveMovie_CF)
                .collect(Collectors.toList());

        var cfAllof = CompletableFuture.allOf(moviewFutures.toArray(new CompletableFuture[movieFutures.size()]))

       return cfAllOf.thenApply(v -> v.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList()))
    }
}
```

- db call은 내부 DB 부르는 거라 빠른데, rest api call은 외부 api(공공api 등)라서 더 느린것으로 가정한다.
- soap call은 더 느린 것으로 가정한다.
- allOf와 달리 anyOf는 CF 중에 하나만 되도 담아서 return시킨다.
- 따라서 dbcall의 list만 담기고 restapi call이나 soap api call에서 받아온 값들은 쓰레기 값이 된다.
```java
public String anyOf() {
    startTimer();


    //db call
    CompletableFuture<String> db = CompletableFuture.supplyAsync(() -> {
        delay(1000);
        log("response from db");
        return "Hello World";
    });

    //restapi call
    CompletableFuture<String> restApi = CompletableFuture.supplyAsync(() -> {
        delay(2000);
        log("response from restApi");
        return "Hello World";
    });


    //soap call
    CompletableFuture<String> soapApi = CompletableFuture.supplyAsync(() -> {
        delay(3000);
        log("response from soapApi");
        return "Hello World";
    });

    List<CompletableFuture<String>> cfList = List.of(db, restApi, soapApi);
    CompletableFuture<Object> cfAnyOf = CompletableFuture.anyOf(cfList.toArray(new CompletableFuture[cfList.size()]));

    String result =  (String) cfAnyOf.thenApply(v -> {
        if (v instanceof String) {
            return v;
        }
        return null;
    }).join();

    timeTaken();
    return result;
}
```


