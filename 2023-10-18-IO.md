## <span style="color:#802548">_입출력 stream_</span>
- 입출력이란 컴퓨터 내부 또는 외부의 장치과 프로그램 간에 데이터를 주고받는 것이다.
- 키보드 입력, System.out.println 등이 예시다.
- 입출력을 자바에서 수행하려면, 두 대상을 연결하고 데이터를 전송할 stream이 필요한데, 단방향이다.
- 따라서 입출력을 동시에 수행하려면 입력 stream과 출력 stream 2개가 필요하다.
- stream은 먼저 보낸 데이터를 먼저 받는 FIFO이며, 중간에 건너뛸 수 없다.
- 스트림은 바이트 기반과 문자 기반이 있다. 보조스트림도 있지만, 이는 바이트 기반 혹은 문자 기반 스트림을 생성하고 나서 추가로 보완을 위해 넣는 스트림이다.

```java
FileInputStream fis = new FileInputStream("test.txt");
BufferedInputStream bis = new BufferedInputStream(fis);
bis.read();
```

- 위의 코드를 보면 bis라는 보조스트림이 입력을 수행하는 것처럼 보이지만, 실제로는 fis가 입력을 한다.
- bis는 그저 buffer를 제공하는 보완 스트림, 보조스트림의 역할을 할 뿐이다.
- 버퍼가 있고 없고는 차이가 극심하기 때문에 대부분 버퍼 관련 보조스트림을 같이 생성한다.

- 위에서는 바이트 스트림을 알아보았다.
- 자바에서는 문자 스트림도 존재한다.
- C언어와 달리 char형이 2byte이기 때문이다.
- InputStream은 Reader로, OutputStream은 Writer로 바뀐다.

## <span style="color:#802548">_바이트 stream의 조상_</span>
- 조상으로 InputStream, OutputStream을 갖는다.
- inputStream의 flush()는 buffer가 있을 떄만 유의미하다.
- outputStream의 flush()는 아무 작동도 없다.
- 그러나 Input, OutputStream을 직접 바로 쓰지는 않는다.

- 구현체로 쓰는데, 그 중에서 가장 기본은 ByteArrayInputStream, ByteArrayOutputStream이다. 잘 쓰이지는 않는다.

```java
byte[] inSrc = {0,1,2,3,4,5,6,6,7,8,9};
byte[] outSrc = null;

ByteArrayInputStream input = null;
ByteArrayOutputStream output = null;

input = new ByteArrayInputStream(inSrc);
output = new ByteArrayOutputStream();

int data = 0;

while((data = input.read()) != -1){
    output.write(data);
}

outSrc = output.toByteArray();
```

- 위의 예시는 한번에 1byte만 읽기 때문에 작업효율이 떨어진다.
- 이를 배열로 처리해주자.

```java
byte[] inSrc = {0,1,2,3,4,5,6,6,7,8,9};
byte[] outSrc = null;
byte[] temp = new byte[10];

ByteArrayInputStream input = null;
ByteArrayOutputStream output = null;

input = new ByteArrayInputStream(inSrc);
output = new ByteArrayOutputStream();

input.read(temp,0,temp.length);
output.write(temp,5,5);

outSrc = output.toByteArray();

System.out.println("Input source: " + Arrays.toString(inSrc));
System.out.println("temp source: " + Arrays.toString(temp));
System.out.println("Output source: " + Arrays.toString(outSrc));
```

- 위의 예시는 read와 write를 할 때 오류를 잡지 않았다.
- try ~ catch를 추가해주자.
- 또한 배열의 크기를 적절하게 주자. 10 ->4로 바꾸었다.

```java
byte[] insSrc = {0,1,2,3,4,5,6,7,8,9};
byte[] outSrc = null;
byte[] temp = new Byte[4];

ByteArrayInputStream input = null;
ByteOutputStream output = null;

input = new ByteArrayInputStream(inSrc);
output = new ByteArrayOutputStream();

System.out.println("Input Source: " + Arrays.toString(inSrc));

try{
    while(input.available()>0){
        input.read(temp);
        output.write(temp);
        //System.out.println("temp: " + Arrays.toString(temp));

        outSrc = output.toByteArray();
        printArrays(temp, outSrc);
    }
}catch(IOException e){}

static void printArrays(byte[] temp, byte[] outSrc){
    System.out.println("temp: " + Arrays.toString(temp));
    System.out.println("Output Source: " + Arrays.toString(outSrc));
}
// Input Source: [0,1,2,3,4,5,6,7,8,9]
// temp: [0,1,2,3]
// Output Source: [0,1,2,3]
// temp: [4,5,6,7]
// Output Source: [0,1,2,3,4,5,6,7]
// temp: [8,9,6,7]
// Output Source: [0,1,2,3,4,5,6,7,8,9,6,7]
```

- temp 배열을 보면 8,9로 끝나는 게 아니라 기존 내용을 덮어쓰고 있다. 성능을 위해서다.
- 하지만 그러한 이유로 저장된 모든 내용을 출력해버리면 원하는 결과를 얻어오지 못한다. 
- 따라서 값을 읽어온 만큼만 출력하게 변경해준다.

```java
byte[] inSrc = {0,1,2,3,4,5,6,7,8,9};
byte[] outSrc = null;
byte[] temp = new byte[4];

ByteArrayInputStream input = null;
ByteArrayOutputStream output = null;

input = new ByteArrayInputStream(inSrc);
output = new ByteArrayOutputStream();

System.out.println("Input Source: " + Arrays.toString(inSrc));

try{
    while(input.available()>0){
        int len = input.read(temp);
        output.write(temp, 0, len);
        //System.out.println("temp: " + Arrays.toString(temp));

        outSrc = output.toByteArray();
        printArrays(temp, outSrc);
    }
}catch(IOException e){}
// Input Source: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
// temp: [0, 1, 2, 3]
// Output Source: [0, 1, 2, 3]
// temp: [4, 5, 6, 7]
// Output Source: [0, 1, 2, 3, 4, 5, 6, 7]
// temp: [8, 9, 6, 7]
// Output Source: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```


## <span style="color:#802548">_file IO Stream_</span>
- 파일 IO는 정말 많이 쓰이는 IO Stream이다.


```java
FileInputStream fis = new FileInputStream(args[0]);
int data = 0;
while((data=fis.read()) != -1){
    char c = (char)data;
    System.out.print(c);
}
```

```java
class FileCopy{
    public static main(String[] args){
        try{
            FileInputStream fis = new FileInputStream(args[0]);
            FileOutputStream fos = new FileOutputStream(args[1]);

            int data = 0 ;
            while(( data = fis.read()) != -1){
                fos.write(data);
            }
            fis.close();
            fos.close();
        }catch(IOException e){
            e.printStackTrace();
        }
    }
}
```

- 그럼 아래와 같이 내용이 복제된다.
```shell
java FileCopy.java FileCopy.bak
type FileCopy.bak
# class FileCopy{
#     public static main(String[] args){
#         try{
#             FileInputStream fis = new FileInputStream(args[0]);
#             FileOutputStream fos = new FileOutputStream(args[1]);

#             int data = 0 ;
#             while(( data = fis.read()) != -1){
#                 fos.write(data);
#             }
#             fis.close();
#             fos.close();
#         }catch(IOException e){
#             e.printStackTrace();
#         }
#     }
# }
```

- 물론 위는 text이므로 FileIOStream보다는 FileWriter와 FileReader가 낫다

## <span style="color:#802548">_byte 기반 보조Stream_</span>

- BufferedIOStream은 스트림의 입출력 효율을 높이려고 버퍼를 사용한다
- 거의 대부분에서 사용된다. 한 바이트씩보다는 버퍼(바이트배열)을 이용해 여러 바이트를 출력하는 게 효율이 좋기 떄문이다
- 보통 파일의 경우에는 8K를 사용한다
```java
class BufferedOutputStreamEx1{
    public static void main(String[] args){
        try{
            FileOutputStream fos = new FileOutputStream("123.txt");
            BufferedOutputStream bos = new BufferedOutputStream(fos, 5); //버퍼의 크기는 5
            for(int i = '1'; i <= '9'; i++){
                bos.write(i);
            }
            
            //fos.close(); 이거로 하면 안됨. 버퍼를 닫아줘야 함. 버퍼에 닫힌 것까지 다 내보내야 하기 때문
            bos.close();
        }catch(IOException e){
            e.printStackTrace();
        }
    }
}
```

- DataIOStream은 데이터를 읽을 때 byte단위가 아니라 Java의 8가지 기본 자료형의 단위로 읽고 쓸 수 있다.
- 16진수로 저장된다. 따라서 file을 만들어도 일반적인 문서 편집기로는 읽을 수 없다.


```java
class DataOutputStreamEx1{
    public static void main(String[] args){
        FileOutputStream fos = null;
        DataOutputStream dos = null;

        try{
            fos = new FileOutputStream("sample.dat");
            dos = new DataOutputStream(fos);
            dos.writeInt(10);
            dos.writeFloat(20.0f);
            dos.writeBoolean(true);

            dos.close();
        }catch(IOException e){
            e.printStackTrace();
        }
    }
}
```

```java
class DataOutputStreamEx1{
    public static void main(String[] args){
        FileOutputStream fos = null;
        DataOutputStream dos = null;

        try{
            fos = new FileOutputStream("sample.dat");
            dos = new DataOutputStream(fos);
            dos.writeInt(10);
            dos.writeFloat(20.0f);
            dos.writeBoolean(true);

            System.out.println(dos.readInt());      //10
            System.out.println(dos.readFloat());    //20.0
            System.out.println(dos.readBoolean());  //true


            dos.close();
        }catch(IOException e){
            e.printStackTrace();
        }
    }
}
```

- 사실 아래와 같이 예외에 대비하여 finally에서 close를 하는 것이 더 적절한 예외처리다.

```java
class DataInputStreamEx2{
    public static void main(String[] args){
        int sum = 0;
        int score = 0;

        FileInputStream fis = null;
        DataInputStream dis = null;

        try{
            fis = new FileInputStream("score.data");
            dis = new DataInputStream(fis);

            while(true){
                score = dis.readInt();
                System.out.println(score);
                sum += score;
            }
        }catch(EOFException e){
            System.out.println("점수의 총합은 " + sum + "입니다.");
        }catch(IOException ee){
            ie.printStackTrace();
        }finally{
            try{
                if(dis !=null){
                    dis.close();
                }
            }catch(IOException ie){
                ie.printStackTrace();
            }
        }
    }
}
```

- 문제는 위의 식은 너무 지저분하다. 따라서 try ~ with ~ resources를 사용하여 간단하게 바꾸자

```java
class DataInputStreamEx3{
    public static void main(String[] args){
        int sum = 0;
        int score = 0;

        try(FileInputStream fis = new FileInputStream("score.dat");
            DataInputStream dis = new DataInputStream(fis)){

            while(true){
                score = dis.readInt();
                System.out.println(score);
                sum += score;
            }
        }catch(EOFException e){
            System.out.println("점수의 총합은 " + sum + "입니다.");
        }catch(IOException e){
            ie.printStackTrace();
        }
    }
}
```

- SequenceInputStream은 입력스트림을 연결해서 하나로 만들어준다.
- 자주 쓰이지는 않는다
```java
FileInputStream file1 = new FileInputStream("file.001");
FileInputStream file2 = new FileInputStream("file.002");
SequenceInputStream in = new SequenceInputStream(file1, file2);
```



## <span style="color:#802548">_문자 기반 Stream_</span>

- FileReader와 FileWriter는 파일에서 텍스트데이터를 읽어온다
- FileInputStream으로 읽으면 한글이 깨지지만, FileReader로 읽으면 깨지지 않는다

```java
FileReader fr = new FileReader(fileName);

int data = 0;
while((data = fr.read()) != -1){
    System.out.print((char)data);
}
fr.close();
```

- PipedReader와 PipedWriter는 Thread 간에 데이터를 주고받을 때 사용된다
- 입력고 ㅏ출력 스트림이 하나의 스트림으로 연결된다. 어느 한 쪽 쓰레드에서 닫아버리면 반대 쪽도 자동으로 닫힌다
```java
public class PipedReaderWriter{
    public static void main(String[] args){
        InputThread inTrhead = new InputThread("InputThread");
        OutputThread outThread = new OutputThread("OutputThread");

        inThread.connect(outThread.getOutput());//PipedReader와 PipedWriter를 연결

        inThread.start();
        outThread.start();
    }
}

class InputThread extends Thread{
    PipedReader input = new PipedReader();
    StringWriter sw = new StringWriter();

    InputThread(String naem){
        super(name);
    }

    public void run(){
        try{
            int data = 0;

            while((data = input.read()) != -1){
                sw.write(data);
            }

            System.out.println(getName() + " received : " + sw.toString());
        }catch(IOException e){}
    }

    public PipedReader getInput(){
        return input;
    }

    public void connect(PipedWriter output){
        try{
            input.connect(output);
        }catch(IOException e){}
    }
}

class OutputThread extends Thread{
    PipedWriter output = new PipedWriter();

    OutputThread(String name){
        super(name);
    }

    public void run(){
        try{
            String msg = "Hello";
            System.out.println(getName() + " sent : " + msg);
            output.write(msg);
            output.close();
        }catch(IOException e)
    }

    public PipedWriter getOutput(){
        return output;
    }

    public void connect(PipedReader input){
        try{
            output.connect(input);
        }catch(IOException e){}
    }
}
//OutputThraed sent : Hello
//InputThread received : Hello
```

- StringReader와 StringWriter는 메모리가 입출력대상인 stream이다.

```java
class StringReaderWriterEx{
    public static void main(String[] args){
        String inputData = "ABCD";
        StringReader input = new StringReader(inputData);
        StringWriter output = new StringWriter();

        int data = 0;

        try{
            whil((data = input.read()) != -1){
                output.write(dat);
            }
        }catch(IOException e){}

        System.out.println("Input Data :" + inputData); // input data: ABCD
        System.out.println("output Data :" + output.toString()); // output data: ABCD
        //System.out.println("output Data :" + output.getBuffer().toString());
    }
}
```

## <span style="color:#802548">_문자기반의 보조 Stream_</span>
- BufferedIOStream과 똑같이 BufferedReader와 BufferedWriter가 있다

```java
try{
    FileReader fr = new FileReader("BufferedReaderEx1.java");
    BufferedReader br = new BufferedReader(fr);

    String line = "";
    for(int i =1; (line = br.readLine()) != null; i++){
        // ";"를 포함한 라인을 출력한다.
        if(line.indexOf(";") != -1){
            System.out.println(i + ":" + line);
        }
    }
}
```

- InputStreamReader와 Writer는 따로 쓸 일이 없어보인다.


## <span style="color:#802548">_파일_</span>

- file instance는 파일일 수도 있고, 폴더일 수도 있다
- file instance를 생성했다고 해서 실제 파일이나 폴더가 생성된 것은 아니다.
- 새로운 파일을 생성하려면 File class의 createNewFile()을 호출해야 한다.