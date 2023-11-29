## <span style="color:#802548">_1. generics_</span>



## <span style="color:#802548">_2. enum_</span>



## <span style="color:#802548">_3. IOStream_</span>
- OutputStream의 예시는 아래와 같다.
- 거의 대부분 BufferedStream을 불러서 사용한다. 그게 효율적이기 때문이다.
- try ~ catch 혹은 try ~ with ~ resources 혹은 throws Exception이 필수다.

```java
void transferFile(){
    byte[] decodedBytes = Base64.getUrlDecoder().decode(insertDto.getFile().substring(insertDto.getFile().lastIndexOf(",") +1)
                                            .replace('+', '-').replace('/', '_'));
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 4; i++) { // Consider first 4 bytes as file signature
        sb.append(String.format("%02X", decodedBytes[i]));
    }

    try{
        File file = new File("/upload/" + fileName + fileExtension);
        FileOutputStream fileOutputStream = new FileOutputStream(file);
        BufferedOutputStream bos = new BufferedOutputStream();
        bos.write(decodedBytes);

        bos.close();
    }catch(IOException e){
        throw e;
    }
}

void transferFile(){
    byte[] decodedBytes = Base64.getUrlDecoder().decode(insertDto.getFile().substring(insertDto.getFile().lastIndexOf(",") +1)
                                            .replace('+', '-').replace('/', '_'));
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 4; i++) { // Consider first 4 bytes as file signature
        sb.append(String.format("%02X", decodedBytes[i]));
    }

        File file = new File("/upload/" + fileName + fileExtension);
    try(FileOutputStream fileOutputStream = new FileOutputStream(file);
        BufferedOutputStream bos = new BufferedOutputStream();){
      
        bos.write(decodedBytes);
    }catch(IOException e){
        throw e;
    }
}

void transferFile() throws IOException{
    byte[] decodedBytes = Base64.getUrlDecoder().decode(insertDto.getFile().substring(insertDto.getFile().lastIndexOf(",") +1)
                                            .replace('+', '-').replace('/', '_'));
    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 4; i++) { // Consider first 4 bytes as file signature
        sb.append(String.format("%02X", decodedBytes[i]));
    }
    File file = new File("/upload/" + fileName + fileExtension);
    FileOutputStream fileOutputStream = new FileOutputStream(file);
    BufferedOutputStream bos = new BufferedOutputStream();
    bos.write(decodedBytes);

    bos.close();
}
```

- input을 output으로 새로운 파일을 만드는 예제다.

```java
try{
    FileInputStream fis = new FileInputStream("123.txt");
    BufferedInputStream bis = new BufferedInputStream();
    FileOutputStream fos = new FileOutputStream("456.txt");
    BufferedOutputStream bos = new BufferedOutputStream();
    int data = 0;
    while((data = bis.read()) != -1){
        bos.write(data); //123.txt의 내용이 그대로 456.txt라는 새 파일에 들어간다.
    }

    bis.close();
}catch(IOException e){
    throw e;
}
```




## <span style="color:#802548">_4. Stream_</span>