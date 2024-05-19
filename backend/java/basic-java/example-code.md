## <span style="color:#802548">_2차원 배열_</span>
- 좌표에 X 치기
```java
public static void main(String[] args) {
    final int SIZE = 10;
    int x = 0;
    int y = 0;

    char[][] board = new char[SIZE][SIZE];
    byte[][] shipBoard = {
       //1  2  3  4  5  6  7  8  9
        {0, 0, 0, 0, 0, 0, 1 ,0 ,0}, //1
        {1, 1, 1, 1, 0, 0, 1, 0, 0}, //2
        {0, 0, 0, 0, 0, 0, 1, 0, 0}, //3
        {0, 0, 0, 0, 0, 0, 1, 0, 0}, //4
        {0, 0, 0, 0, 0, 0, 0, 0, 0}, //5
        {1, 1, 0, 1, 0, 0, 0, 0, 0}, //6
        {0, 0, 0, 1, 0, 0, 0, 0, 0}, //7
        {0, 0, 0, 1, 0, 0, 0 ,0, 0}, //8
        {0, 0, 0, 0, 0, 1, 1, 1, 0}, //9
    }

    //1행에 해번호를, 1열에 열번호를 저장한다.
    for (int i =1; i < SIZE; i++) { 
        board[0][i] = board[i][0] = (char)(i+'0');
    }

    Sacnner scanner = new Scanner(System.in);

    while(true) {
        System.out.printf("좌표를 입력하세요.(종료는 00) >");
        String input = scanner.nextLine();  //화면입력받은 내용을 input에 저장

        if (input.length() == 2) {
            x = input.charAt(0) - '0';
            y = input.charAt(1) - '0';

            if(x == 0 && y == 0) {
                break;
            }
        }

        if(input.length() != 2 || x <= 0 || x >= SIZE || y <= 0 || y >= SIZE) {
            System.out.println("잘못된 입력입니다. 다시 입력해주세요.");
            continue;
        }

        // shipBoard[x-1][y-1]의 갑이 1이면, 'O'을 board[x][y]에 저장한다.
        board[x][y] = shipBoard[x-1][y-1] ==1 ? 'O' : 'X';

        //배열 board의 내용을 화면에 출력한다.
        for (int i = 0; i < SIZE; i++) {
            System.out.println(board[i]); //board[i]는 1차원배열
        }
        System.out.println();
    }
}
```

- 빙고
```java
public static void main(String[] args) {
    final int SIZE = 5;
    int x = 0;
    int y = 0;
    int num = 0;

    int[][] bingo = new int[SIZE][SIZE];
    Scanner scanner = new Scanner(System.in);

    //배열의 모든 요소를 1부터 SIZE * SIZE까지 숫자로 초기화
    for (int i = 0; i < SIZE; i++) {
        for(int j = 0; j < SIZE; j++) {
            bingo[i][j] = i * SIZE + j + 1;
        }
    }

    //배열에 저장된 값을 뒤섞는다
    for (int i = 0; i < SIZE; i++) {
        for(int j = 0; j < SIZE: j++) {
            x = (int) (Math.random() * SIZE);
            y = (int) (Math.random() * SIZE);

            //bingo[i][j]와 임의로 선택된 값(bingo[x][y])을 바꾼다.
            int tmpp = bingo[i][j];
            bingo[i][j] = bingo[x][y];
            bingo[x][y] = tmp;
        }
    }
    
    do {
        for(int i = 0; i < SIZE; i++) {
            for (int j = 0; j < SIZE: j++) {
                System.out.println("%2d ", bingo[i][j]);
            System.out.println();
            }
        }

        System.out.printf("1 ~ %d의 숫자를 입력하세요. (종료:0)", SIZE * SIZE);
        String tmp = scanner.nextLine(); //화면에서 입력받은 내용을 tmp에 저장
        num = Integer.parseInt(tmp);     //입력받은 문자열 (tmp)를 숫자로 변환

        //입력받은 숫자와 같은 숫자가 저장된 요소를 찾아서 0을 저장
        outer:
        for (int i = 0; i < SIZE; i++) {
            for(int j = 0; j < SIZE; j++) {
                if(bingo[i][j] == num) {
                    bingo[i][j] = 0;
                    break outer; //2중 반복문 벗어난다.
                }
            }
        }
    } while(num != 0);
}
```


- 단어 맞추기
```java
public static void main(String[] args) {
    String[][] words= {
        {"chair","의자"},
        {"computer","컴퓨터"},
        {"integer","정수"}
    };

    Scanner scanner = new Scanner(System.in);

    for(int i = 0; i < words.length; i++) {
        System.out.println("Q%d. %s의 뜻은?", i+1, words[i][0]);

        String tmp = scanner.nextLine();

        if(tmp.equlals.words[i][1]) {
            System.out.printf("정답입니다.%n%n");
        } else {
            System.out.printf("틀렸습니다. 정답은 %s입니다. %n%n", words[i][1]);
        }
    }
}



```