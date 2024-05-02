- 중요한 건 latency(지연시간)과 throughput(처리량)이다.
- web에서는 그게 중요하다.
- task를 subtask로 줄이고 싶을 떄는, 현재 돌아가는 process의 수와 CPU 코어의 수를 알아야 한다.
- task를 나누는 데에도 컴퓨터 자원이 필요하고, 나눠진 task를 종합하는 데도 컴퓨터 자원이 필요하다.
- 따라서 늘 task를 나누는 게 좋은 결과를 초래하진 않는다.
- 다만 기존 작업이 무거울수록 나누는 게 좋고, 별것도 아닌것이라면 나누지 않는 게 좋다.


 - 아래와 같이 이미지 파일을 나눠서 각자 스레드가 처리하게 병렬처리하면 속도가 향상된다.
```java

public static void recolorMultithreaded(BufferedImage originalImage, BufferedImage resultImage, int numberOfThreads) {
    List<Thread> threads = new ArrayList<>();
    int width = originalImage.getWidth();
    int height = originalImage.getHeight() / numberOfThreads;

    for(int i = 0; i < numberOfThreads ; i++) {
        final int threadMultiplier = i;

        Thread thread = new Thread(() -> {
            int xOrigin = 0 ;
            int yOrigin = height * threadMultiplier;

            recolorImage(originalImage, resultImage, xOrigin, yOrigin, width, height);
        });

        threads.add(thread);
    }

    for(Thread thread : threads) {
        thread.start();
    }

    for(Thread thread : threads) {
        try {
            thread.join();
        } catch (InterruptedException e) {
        }
    }
}
```


- 아래는 CPU연산 로직이다. blocking I/O가 아닌 business logic이라고 보면 된다.
```java
public static void recolorSingleThreaded(BufferedImage originalImage, BufferedImage resultImage) {
    recolorImage(originalImage, resultImage, 0, 0, originalImage.getWidth(), originalImage.getHeight());
}

//이미지 전체 pixel을 순회하며 색상을 바꿈.
public static void recolorImage(BufferedImage originalImage, BufferedImage resultImage, int leftCorner, int topCorner,
                                int width, int height) {
    for(int x = leftCorner ; x < leftCorner + width && x < originalImage.getWidth() ; x++) {
        for(int y = topCorner ; y < topCorner + height && y < originalImage.getHeight() ; y++) {
            recolorPixel(originalImage, resultImage, x , y);
        }
    }
}

public static void recolorPixel(BufferedImage originalImage, BufferedImage resultImage, int x, int y) {
    int rgb = originalImage.getRGB(x, y);

    int red = getRed(rgb);
    int green = getGreen(rgb);
    int blue = getBlue(rgb);

    int newRed;
    int newGreen;
    int newBlue;

    if(isShadeOfGray(red, green, blue)) {
        newRed = Math.min(255, red + 10);   //보라는 red가 좀 더 강하니 더 추가
        newGreen = Math.max(0, green - 80); //보라는 green색깔이 필요 없으니 확 빼버림
        newBlue = Math.max(0, blue - 20);   //보라는 blue도 들어가지만 많이 안 필요하니 줄임
    } else {
        newRed = red;
        newGreen = green;
        newBlue = blue;
    }
    int newRGB = createRGBFromColors(newRed, newGreen, newBlue);
    setRGB(resultImage, x, y, newRGB);
}

public static void setRGB(BufferedImage image, int x, int y, int rgb) {
    image.getRaster().setDataElements(x, y, image.getColorModel().getDataElements(rgb, null));
}

public static boolean isShadeOfGray(int red, int green, int blue) { //픽셀의 특정 색상 값을 취하고 넣을 회색을 결정
    return Math.abs(red - green) < 30 && Math.abs(red - blue) < 30 && Math.abs( green - blue) < 30;
}

public static int createRGBFromColors(int red, int green, int blue) {
    int rgb = 0;
        //<<는 쉬프트 연산자
        // |=는 비트 연산자
    rgb |= blue;
    rgb |= green << 8;
    rgb |= red << 16;

    rgb |= 0xFF000000; //16진수 255

    return rgb;
}

public static int getRed(int rgb) {
    return (rgb & 0x00FF0000) >> 16; //색깔을 bit 연산자로 처리하는 과정
}

public static int getGreen(int rgb) {
    return (rgb & 0x0000FF00) >> 8; //색깔을 bit 연산자로 처리하는 과정
}

public static int getBlue(int rgb) {
    return rgb & 0x000000FF; //색깔을 bit 연산자로 처리하는 과정
}
```

- numberOfThreads를 2로 늘리면 속도가 더 빨라지고, 스레드 6개로 늘리면 더 줄어든다. 다만 스레드 갯수에 비례하게 작업시간이 줄어드는 건 아니다.
- 스레드 수는 물리 CPU 코어를 넘어가면 성능 개선이 줄어들고, 가상코어를 넘어서면 아예 더 느려진다.
- 낮은 해상도의 이미지를 병렬 처리하면 오히려 더 느려질 수도 있다. 
- 병렬 처리 멀티 스레딩에 들어가는 자원도 있기 때문이다. 
```java
public class Main {
    public static final String SOURCE_FILE = "./resources/many-flowers.jpg";
    public static final String DESTINATION_FILE = "./out/many-flowers.jpg";

    public static void main(String[] args) throws IOException {

        BufferedImage originalImage = ImageIO.read(new File(SOURCE_FILE)); //imageIO는 픽셀, 컬러스페이스, 디멘션 등 이미지 데이터를 표현하는 데이터를 넣을 수 있다.
        BufferedImage resultImage = new BufferedImage(originalImage.getWidth(), originalImage.getHeight(), BufferedImage.TYPE_INT_RGB);
        
        long startTime = System.currentTimeMillis();
        //recolorSingleThreaded(originalImage, resultImage);
        int numberOfThreads = 1; 
        recolorMultithreaded(originalImage, resultImage, numberOfThreads);
        long endTime = System.currentTimeMillis();

        long duration = endTime - startTime;

        File outputFile = new File(DESTINATION_FILE);
        ImageIO.write(resultImage, "jpg", outputFile);

        System.out.println(String.valueOf(duration)); //single thread와 multi thread를 비교하기 위해 시간 설정
    }
}
```

- 별개 Thread를 만들어서 sub-task로 나누는 것은 효율이 좋지 않다.
- 방금 했던 이미지와 병렬 처리 작업이 그 예시다.
```
작업을 여러개로 나눈다
스레드를 생성한다.
생성한 스레드에 나눈 작업을 할당한다.
스케줄링한다
결과를 하나로 결합한다
```
- 이 과정이 처리량에 있어 불필요한 작업이라 오버헤드다.
- 각 작업을 별개 스레드에 스케줄링하는 하는게 좋은 선택이다. 이것을 Thread pool로 달성할 수 있다.
  - 스레드를 생성하고 미래 작업을 위해 다시 스레드를 사용한다.
  - 매번 스레드를 생성할 필요가 없다.
  - 스레드가 생성되면 풀에 쌓이고, 작업이 대기열을 통해 스레드 별로 분배된다.
  - 모든 스레드가 바쁘면 대기열에 머무르며 스레드가 이용가능할 때까지 기다린다.
- 이를 통해 낮은 오버헤드와 효율적인 대기열을 만들 수 있다.
- Java에서는 Executor class가 그러한 예시다.
- 예시는 아래와 같다.
- 단어 검색 로직은 아래와 같다.
```java
private static class WordCountHandler implements HttpHandler { 
        private String text;

        public WordCountHandler(String text) {
            this.text = text;
        }

        @Override
        public void handle(HttpExchange httpExchange) throws IOException {
            String query = httpExchange.getRequestURI().getQuery(); // queryString 얻어옴
            String[] keyValue = query.split("=");
            String action = keyValue[0];    //queryString의 key
            String word = keyValue[1];      //queryString의 value
            if (!action.equals("word")) {
                httpExchange.sendResponseHeaders(400, 0); //오류날시 400 return
                return;
            }

            long count = countWord(word);

            byte[] response = Long.toString(count).getBytes(); //직렬화하여 전송
            httpExchange.sendResponseHeaders(200, response.length);
            OutputStream outputStream = httpExchange.getResponseBody();
            outputStream.write(response);
            outputStream.close();
        }

        private long countWord(String word) {
            long count = 0;
            int index = 0;
            while (index >= 0) {
                index = text.indexOf(word, index);

                if (index >= 0) {// index가 양수면 해당 단어가 존재함을 의미
                    count++;
                    index++;
                }
            }
            return count;
        }
    }
```


- multithreading의 핵심은 바로 이 method다. 
- ThreadPool을 만들어 관리하기 위해 Executor class를 활용한다.
```java
public static void startServer(String text) throws IOException {
        HttpServer server = HttpServer.create(new InetSocketAddress(/*port번호*/8000), /*백로그 사이즈. 모든 요청이 대기열 없이 들어가야 하므로 0*/0);
        server.createContext("/search", new WordCountHandler(text));
        Executor executor = Executors.newFixedThreadPool(NUMBER_OF_THREADS);
        server.setExecutor(executor);
        server.start();
    }
```

- 실제구성은 아래와 같이 될 것이다.
```java
public class ThroughputHttpServer {
    private static final String INPUT_FILE = "./resources/war_and_peace.txt";
    private static final int NUMBER_OF_THREADS = 8;

    public static void main(String[] args) throws IOException {
        String text = new String(Files.readAllBytes(Paths.get(INPUT_FILE))); // text 파일 읽어오기
        startServer(text);
    }
}
```