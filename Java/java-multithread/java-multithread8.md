## <span style="color:#802548">_blocking I/O에서 multiThread의 한계- 1_</span>
- 스레드풀을 유지하여 애플리케이션의 전체 기간 동안 같은 스레드를 재사용할 수 있다.
- 새 작업이 있을 때마다 스레드를 만들고 시작하고 종료하는 부하를 줄일 수 있다.
- 스레드 풀의 최적의 크기는 컴퓨터의 코어 수만큼으로 해놓으면 하드웨어 수준에서의 최적화가 동반된다.
- 그러나 blocking I/O에서는 위에서 말한 사항이 적용되지 않는다.
  - HTTP REQUEST가 대표적으로 그런 blocking I/O다.
  - Request를 읽고 Request를 보내는 것은 매우 빠른 CPU 연산이다.
  - 그러나 DB에서 읽어오는 것은 다른 컴퓨터가 해오는 네트워크 작업이며, blocking I/O라 CPU가 그동안 논다.
  - 아니면 js, css, html을 읽어오는 blocking I/O라 CPU가 그동안 논다.
  - 그래서 CPU에는 여유가 있지만, 요청이 끝나지 않아 다음 요청은 queue에 들어가게 된다.
- 스레드를 CPU 코어 수만큼 해놓아도 blocking I/O에서는 기다리는 시간 동안 스레드를 처리하는 CPU가 놀아버리기 때문에 더이상 최적화가 아니게 된다.
- 스레드 중에서 1/4만  blocking I/O를 해서 느려져도 가용가능한 스레드는 빠르게 고갈된다.

<img src="/image/blocking1.jpg" />
<img src="/image/blocking2.jpg" />
<img src="/image/blocking3.jpg" />
<img src="/image/blocking4.jpg" />


- 1000개까지 스레드가 늘어난다. 그러나 스레드를 무한정으로 늘리진 못한다. 
- 태스크를 10_000으로 늘려서 스레드를 10_000개를 할당하려고 하면 memory 관련 error가 난다. 따라서 newFixedThreadPool(1000)으로 고정해준다.
```java
public class IoBoundApplication {
    private static final int NUMBER_OF_TASKS = 1000;

    public static void main(String[] args) {
        System.out.printf("Running %d tasks\n", NUMBER_OF_TASKS);

        long start = System.currentTimeMillis();
        performTasks();
        System.out.printf("Tasks took %dms to complete\n", System.currentTimeMillis() - start);
    }

    private static void performTasks() {
        try (ExecutorService executorService = Executors.newCachedThreadPool() /*newFixedThreadPool(1000) */) { //정해진 thread가 아니라 필요한 thread만큼 계속 생성하고 재사용함. 동접자가 늘어나면 그만큼 많은 스레드가 발급됨. 안쓰이면 나중에 사라짐

            for (int i = 0; i < NUMBER_OF_TASKS; i++) {
                executorService.submit(() ->  blockingIoOperation());
            }
        }
    }

    // Simulates a long blocking IO
    private static void blockingIoOperation() {
        System.out.println("Executing a blocking task from thread: " + Thread.currentThread());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
}
```

## <span style="color:#802548">_blocking I/O에서 multiThread의 한계- 2_</span>
- 또한 blocking I/O를 처리하기 위해  thread를 많이 발급하면 context switching이 일어나게 된다. 너무 많은 context switching으로 인해 오히려 전체 blocking I/O 연산보다 더 느려질 수도 있다.
- blocking call을 1000ms가 아니라 10ms로 만들어 더 자주 blocking I/O(여기선 Thread.sleep)를 호출하면 이러한 문제가 명확하게 드러난다.
  - 원래 10초면 끝나던게 이제 23초가 걸린다.
    - blocking I/O가 발생하면 OS는 해당 스레드의 스케쥴링을 취소하고 진행가능한 thread를 찾아 context switching을 하게 된다. thread를 CPU에 넣었다 뺐다 하는 것이다. 
    - 이렇게 CPU가 시스템을 관리하는 것 때문에 CPU 자원을 소모하는 것을 threashing이라고 하며 피해야 한다.
```java
private static void blockingIoOperation() {
        System.out.println("Executing a blocking task from thread: " + Thread.currentThread());
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
    }
```

- thread per task model은  I/O가 call 되면 해당 thread가 놀아버린다.
- 따라서 다른 사용자의 요청을 처리하려면 thread를 많이 발급해야 한다.
- 또한 그 blocking I/O call이 자주 일어날수록 context swtiching overhead가 커져 반응속도도 느려진다.
  - 이와 같은 성능저하 이외에도 HTTP request에 관해서 다른 문제가 생긴다.
  - 우리 내부에서 통제가능한 DBcall이 아니라, 다른 네트워크에 있는  DB에 요청을 넣게 되면 외부 DB가 느린 경우 사용가능한 thread가 더 빠르게 소모된다. 그러나 우린 대처할 수가 없다. 안정성에 문제가 생겼다.
  - 거기다 그 몇개의 스레드로 인해 우리 앱을 사용하는 모든 사용자가 영향을 받아 느린 앱 서비스를 경험하게 된다.
- 아래같은 코드가 바로 blocking I/O call 코드다. 여기선 server 시작 시 thread도 1개라서 문제다.
```java
private void startHttpServer() throws IOException {
    HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
    HttpContext context = server.createContext("/");
    context.setHandler(this::handleHttpRequest);
    server.start();
}
 
/** Handles an incoming HTTP request from a user
 */
private void handleHttpRequest(HttpExchange httpExchange) {
    try {
        int numberOfProducts = parseRequest(httpExchange);
        URI requestURI = URI.create(String.format("best-online-store/products?number-of-products=%d", numberOfProducts));
 
        HttpResponse<String> response = httpClient.send(
                                                        HttpRequest
                                                            .newBuilder()
                                                            .GET()
                                                            .uri(requestURI)
                                                            .build(),
                                                        HttpResponse.BodyHandlers.ofString()
                                                    );
 
        sendWebpageToUser(httpExchange, response);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```

- 해당 문제를 완화하려고 thread를 많이 발급할 수도 있다.
- 요청이 오는 만큼 동적으로 thread를 발급하게 Executor를 사용하여 설정했다.
- 그러나 여전히 thread 고갈 문제는 피해갈 수 없다.
```java
private void startHttpServer() throws IOException {
    HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
    HttpContext context = server.createContext("/");
    context.setHandler(this::handleHttpRequest);
 
    server.setExecutor(Executors.newCachedThreadPool()); // update
 
    server.start();
}
 
/** Handles an incoming HTTP request from a user
 */
private void handleHttpRequest(HttpExchange httpExchange) {
    try {
        int numberOfProducts = parseRequest(httpExchange);
        URI requestURI = URI.create(String.format("best-online-store/products?number-of-products=%d", numberOfProducts));
 
        HttpResponse<String> response = httpClient.send(HttpRequest.newBuilder().GET().uri(requestURI).build(),
                                                        HttpResponse.BodyHandlers.ofString()
                                                    );
 
        sendWebpageToUser(httpExchange, response);
    } catch (Exception e) {
        throw new RuntimeException(e);
    }
}
```


## <span style="color:#802548">_non-blocking I/O_</span>
- 다행히도 blocking I/O가 아닌 Non-blocking I/O를 사용하면 문제가 해결된다.
  - non-blocking I/O는 result가 준비되면 실행되는 callback을 사용한다.
  - 의사코드로는 아래와 같다.
```java
//blocking I/O model
public void handleRequest(HttpExchange exchange) { 
    Request request = parseUserRequest(exchange);
    Data data = readFromDatabase(request);
    sendPageToUser(data, exchange);
}

// non-blocking I/O model
public void handleRequest(HttpExchange exchange) {
    Request requ est = parseUserRequest(exchange);
    readFromDatabaseAsync(request, (data) -> {
        sendPageToUser(data, exchange);
    })
}
```

- 처음 요청이 도착하면 스레드는 요청을 읽고 파싱하는 데 CPU연산을 수행한다.
  - non-blocking I/O 연산으로 database call을 호출하며, 스레드를 차단한 뒤 다음 스레드를 찾지 않고, 콜백함수를 넘기고 반환하여 다음요청을 처리한다. CPU 코어가 놀지 않게 된다.
  - DB가 연산을 끝내고 값을 보내면 OS는 첫번째 non-blocking I/O결과가 준비되었음을 인지하여 스레드에게 콜백을 실행시킨다.
  - 스레드는 콜백을 실행하여 사용자에게 결과를 보낸다.
- 요청을 처리하는 스레드가 차단되지 않았으므로 코어보다 많은 스레드를 만들 필요가 없다. 해당 스레드만으로 request 처리가 충분해져 context-swithcing도 최소화된다.
  - blocking I/O에서는 사용자의 요청에 대응하기 위해서 어쩔수없이 스레드를 더 생성해야 했지만, 이제는 그럴 필요가 없다. CPU 물리 코어가 늘어나야만 병렬로 더 많은 요청을 처리할 수 있다. 이미 OS수준에서 최적화가 끝났기 때문이다.
  - non-blocking I/O를 사용하면 외부 DB가 느림에 따른 영향을 받지 않게 된다. callback으로 해당 response가 올 떄까지 해당 request만 대기하면 된다. 모든 사용자가 대기하지 않는다.

<img src="/image/nonblocking1.jpg"/>
<img src="/image/nonblocking2.jpg"/>
<img src="/image/nonblocking3.jpg"/>
<img src="/image/nonblocking4.jpg"/>



- httpClient.sendAsync라서 async하게 보낸다인데, 핵심은 non-blocking I/O라는 점이다.
```java
/** Starts an HTTP Server listening on port 8080.
    Delegates the handling of the requests to the handleHttpRequest method
 **/
private void startHttpServer() throws IOException {
    HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);
    HttpContext context = server.createContext("/");
    context.setHandler(this::handleHttpRequest);
 
    // UPDATE 
    ExecutorService executorService = Executors.newFixedThreadPool(8);
    this.httpClient = HttpClient.newBuilder().executor(executorService).build();
    server.setExecutor(executorService);
 
    server.start();
}
 
/** UPDATED - Handles an incoming HTTP request from a user
*/
private void handleHttpRequest(HttpExchange httpExchange) {
    int numberOfProducts = parseRequest(httpExchange);
    URI requestURI = URI.create(String.format("best-online-store/products?number-of-products=%d", numberOfProducts));
 
    CompletableFuture<HttpResponse<String>> responseFuture =  httpClient.sendAsync(
                                                                                    HttpRequest.newBuilder()
                                                                                        .GET()
                                                                                        .uri(requestURI)
                                                                                        .build(),
                                                                                    HttpResponse.BodyHandlers.ofString()
                                                                                );
 
    responseFuture.thenAccept( response -> {
        sendWebpageToUser(httpExchange, response);
    });
}
```


- 그러나 non-blocking I/O에도 이슈가 있다.
  - 코드 가독성이 가장 문제다. js의 콜백지옥이 떠오를 것이다.
    -  디버그도 어렵고 읽기도 어렵다.
   - java에서 제공하는 API도 사용법이 지극히 복잡하기 때문에 webFlux 같은 모듈을 사용해야만 한다. 직접 최적화하기가 불가능에 가깝다.
- 두 개의 장점만 통합한 게 바로 가상 I/O다.


<img src="/image/virtualthread.jpg" />


