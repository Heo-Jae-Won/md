- url을 만들어 거기에 있는 정보를 가져올 때는 URL instance를 활용한다.
- stream을 열어야 하는데, 그것은 openStream()으로 가능하다.
```java
public class NetWorkEx4 {
    public static void main(String[] args) {
        URL url = null;
        BufferedReader input = null;
        String address = "http://www.codechobo.com/sample.hello.html";
        String line = "";

        try{
            url = new URL(address);
            input = new BufferedReader(new InputStreamReader(url.openStream()));

            while( (line = input.readLine()) != null) {
                sysout(line);
            }
            input.close();
        } catch(Exceptione e) {
            e.printStackTrace();
        }
    }
}
```

- try catch보다는 try with resources로 바꿔주자.
```java
 try( BufferedReader input = new BufferedReader(new InputStreamReader(new URL(address).openStream())) ){
    while( (line = input.readLine()) != null) {
        sysout(line);
    }
}
```

- URL은 인터넷 상의 주소여야 한다.
- 하지만 그걸 출력할 Stream은 반드시 문자열일 필요가 없다.
- 아래와 같이 파일로 만들 수도 있다.
```java
public class NetWorkEx5 {
    public static void main(String[] args) {
        URL url = null;
        InputStream out = null;
        FileOutputStream out = null;
        String address = "http://www.codechobo.com/book/src/javajungsuk3_src.zip";

        int ch = 0;

        try(in = new URL(address).openStream(); out = new FileOutputStream("javajungsuk3_src.zip")) {
            while( (ch = in.read()) != -1) {
                out.write(ch);
            }
        }
    }
}
```

```java
public class TcpIpServer {
    public static void main(String[] args) {
        ServerSocket serverSocket = null;

        try {
            serverSocket = new ServerSocket(7777); //서비스 소켓이 7777 port와 bind된다.
            System.out.println(getTime() + "서버가 준비되었습니다.");
        } catch(IOException e) {
            e.printStackTrace();
        }

        while(true) {
            try {
                System.out.println(getTime() + "연결요청을 기다립니다.");
                //서비스 소켓은 client의 연결요청이 올 때까지 실행을 멈추고 기다린다.
                Socket socket = serverSocket.accept(); //연결 요청이 오면 클라이언트 소켓과 통신할 새로운 소켓을 만든다.
                System.out.println(getTime() + socket.getOutputStream());
                DataOutputStream dos = new DataOutputStream(out);

                OutputStream out = socket.getOutputStream(); //소켓의 outputStream을 얻는다.
                DataOutputStream dos = new DataOutputStream(out);

                dos.writeUTF("[Notice] Test Message1 from Server."); //원격 소켓에 데이터를 보낸다.
                System.out.println(getTime() + "데이터를 전송했습니다.");

                dos.close();
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}

static String getTime() {
    SimpleDateFormat sdf = new SimpleDateFormat("[hh:mm:ss]");

    return sdf.format(new Date());
}
```

```java
public class TcpIpClient {
    public static void main(String[] args) {
        try {
            String serverIp = "127.0.0.1";

            System.out.println("서버에 연결중입니다. 서버IP :" + serverIp);
            Socket socket = new Socket(serverIp, 7777); //serverIp:7777 소켓을 생성해 연결을 요청한다.

            InputStream in = socket.getInputStream(); //소켓의 inputStream을 얻는다.
            DataInputStream dis = new DataInputStream(in);

            System.out.println("서버로부터 받은 메시지:" + dis.readUTF());
            System.out.println("연결을 종료합니다.");

            dis.close(); //스트림을 닫는다.
            socket.close(); //socket을 닫는다.
        } catch (ConnectException e) {
            e.printStackTrace();
        } catch (IOException e ) {
            e.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

```java
public class TcpIpServer2 {
    public static void main(string[] args) {
        ServerSocket serverSocket = null;

        try(serverSocket = new ServerSocket(7777)) {
            System.out.println(getTime() + "서버가 준비되었습니다.");
        }

        while(true) {
            try {
                System.out.println(getTime() + "연결요청을 기다립니다."); //서버 소켓
                Socket socket = serverSocket.accept();
                System.out.println(getTime() + socket.getInetAddress() + "로부터 연결요청이 들어왔습니다.");

                System.out.println("getPort(): " + socket.getPort()); //상대편 소켓의 port(client). client의 port는 랜덤으로 결정되어 server의 port와 겹칠수도 있다.
                System.out.println("getLocalPort(): " + socket.getLocalPort()); //내 소켓의 port(server)

                OutputStream out = socket.getOutputStream(); //소켓의 outputStream을 얻는다.
                DataOutputStream dos = new DataOutputStream(out);

                dos.writeUTF("[Notice] Test Message1 from Server.");
                System.out.println(getTime() + "데이터를 전송했습니다.");

                dos.close(); //스트림과 소켓을 닫아준다.
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    static String getTime() {
        SimpleDateFormat sdf = new SimpleDateFormat("[hh:mm:ss]");
        
        return sdf.format(new Date());
    }
}
```

- catch문으로 접속요청이 없다면 서버를 종료할 수 있다.
```java
public class TcpIpServer3 {
    public static void main(String[] args) {
        ServerSocket serverSocket = null;
        try(serverSocket = new ServerSocket(7777)){
            System.out.println(getTime() + "서버가 준비되었습니다."); //서버소켓을 생성해 7777 port와 결합
        }

        while(true) {
             try {
                System.out.println(getTime() + "연결요청을 기다립니다."); //서버 소켓
                Socket socket = serverSocket.accept();
                System.out.println(getTime() + socket.getInetAddress() + "로부터 연결요청이 들어왔습니다.");

                System.out.println("getPort(): " + socket.getPort()); //상대편 소켓의 port(client). client의 port는 랜덤으로 결정되어 server의 port와 겹칠수도 있다.
                System.out.println("getLocalPort(): " + socket.getLocalPort()); //내 소켓의 port(server)

                OutputStream out = socket.getOutputStream(); //소켓의 outputStream을 얻는다.
                DataOutputStream dos = new DataOutputStream(out);

                dos.writeUTF("[Notice] Test Message1 from Server.");
                System.out.println(getTime() + "데이터를 전송했습니다.");

                dos.close(); //스트림과 소켓을 닫아준다.
            } catch (IOException e) {
                e.printStackTrace();
            } catch(SocketTimeoutException e) {
                System.out.println("지정된 시간동안 접속 요청이 없어서 서버를 종료합니다.");
                System.exit(0);
            }
        }
    }
}
```

```java
public class TcpIpServer4 implements Runnable {
    ServerSocket serverSocket;
    Thread[] threadArr;

    public static main(String[] args) {
        TcpIpServer4 server = new TcpIpServer4(5); //5개의 thread를 생성하는 서버소켓을 만든다.
        server.start();
    }

    public TcpIpServer4(int num) {
        try {
            serverSocket = new ServerSocket(7777); //서버소켓을 생성해 7777번 포트와 bind한다.
            System.out.println(getTime() + "서버가 준비되었습니다.");

            threadArr = new Thread[num];
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    @Override
    public void start() {
        for(int i = 0; i <threadArr.length; i++) {
            threadArr[i] = new Thread(this);
            threadArr[i].start();
        }
    }

    @Override
    public void run() {
        while(true) {
            try {
                System.out.println(getTime() + "가 연결요청을 기다립니다.");
                Socket socket = serverSocket.accept();
                System.out.println(getTime() + socket.getInetAddress() + "로부터 연결요청이 들어왔습니다.");

                OutputStream out = socket.getOutputStream(); //소켓의 출력스트림을 얻는다.
                DataOutputStream dos = new DataOutputStream(out);

                dos.writeUTF("[Notice]")
            }
        }
    }

    static String getTime() {
        String name = Thread.currentThread().getName();
        SimpleDateFormat sdf = new SimpleDateFormat("[hh:mm:ss]");

        return sdf.format(new Date()) + name; 
    }
}
```

```java
public class TcpIpServer5 {
    public static void main(String[] args) {
        ServerSocket serverSocket = null;
        Socket socket = null;

        try {
            serverSocket = new ServerSocket(7777);
            System.out.println("서버가 준비되었습니다.");

            socket = serverSocket.accept();

            Sender sender = new Sender(socket);
            Receiver receiver = new Receiver(socket);

            sender.start();
            receiver.start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

class Sender extends Thread {
    Socket socket;
    DataOutputStream out;
    String name;

    Sender(Socket socket) {
        this.socket = socket;
        try {
            out = new DataOutputStream(socket.getOutputStream());
            name = "[" + socket.getInetAddress() + ":" + socket.getPort() + "]";
        } catch (Exception e) {}
    }
    
    @Override
    public void run() {
        Scanner scanner = new Scanner(System.in);
        while( out != null) {
            try {
                out.writeUTF(name + scanner.nextLine());
            } catch (IOException e) {}
        }
    }
}

class Receiver extends Thread {
    Socket socket;
    DataInputStream in;

    Receiver(Socket socket) {
        this.socket = socket;
        try {
            in = new DataInputStream(socket.getInputStream());
        } catch (IOException e) {}
    }
    
    @Override
    public void run() {
        while(in != null) {
            try {
                System.out.println(in.readUTF());
            } catch (IOException e) {}
        }
    }
}


```

```java
public class TcpIpMultichatServer {
    HashMap clients;

    TcpIpMultichatServer() {
        clients = new HashMap();
        Collections.synchronizedMap(clients);
    }

    public void start() {
        ServerSocket serverSocket = null;
        Socket socket = null;

        try {
            serverSocket = new ServerSocket(7777);
            System.out.println("서버가 시작되었습니다.");

            while(true) {
                socket = serverSocket.accept();
                System.out.println("[" + socket.getInetAddres() + ":" + socket.getPort() + "]" +"에서 접속하였습니다.");
                ServerReceiver thread = new ServerReceiver(socket);
                thread.start();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    void sendToAll(String msg) {
        Iterator it = clients.keySet().iterator();

        while(it.hasNext()) {
            try {
                DataOutputStream out = (DataOutputStream) clients.get(it.next());
                out.writeUTF(msg);
            } catch (IOException e) {}
        }
    }

    public static void main(String[] args) {
        new TcpIpMultichatServer().start();
    }

    class ServerReceiver extends Thread {
        Socket socket;
        DataInputStream in;
        DataOutputStream out;

        ServerReceiver(Socket socket) {
            this.socket = socket;
            try {
                in = new DataInputStream(socket.getInputStream());
                out = new DataOutputStream(socket.getOutputStream());
            } catch (IOException e) {}
        }

        @Override
        public void run() {
            String name = "";

            try {
                name = in.readUTF();
                sendToAll("#" + name +"님이 들어오셨습니다.");

                clients.put(name,out);
                System.out.println("현재 서버 접속자 수는 " +  clients.size() + "입니다.");

                while(in != null) {
                    sendToAll(in.readUTF());
                }
            } catch (IOException e) {

            } finally {
                sendToAll("#" + name + "님이 나가셨습닌다.");
                clients.remove(name);
                System.out.println("[" + socket.getInetAddress() + ":" + socket.getPort() + "]" + "에서 접속을 종료하였습니다.");
                System.out.println("현재 서버접속자 수는 " + clients.size() + "입니다.");
            }
        }
    }
}
```

```java
public class TcpIpMultichatClient {
    public static void main(String[] args) {
        if(args.length != 1) {
            System.out.println("USAGE: java TcpIpMultichatClient 대화명");
            System.exit(0);
        }

        try {
            String serverIp = "127.0.0.1";
            Socket socket = new Socket(serverIp, 7777);
            System.out.println("서버에 연결되었습니다.");
            Thread Sender = new Thread(new ClientSender(socket, args[0]));
            Thread receiver = new Thread(new ClientSender(socket));

            sender.start();
            receiver.start();
        } catch (ConnectionException ce) {
            ce.printStackTrace();
        } catch(Exception e) {}
    }

    static class ClientSender extends Thread {
        Socket socket;
        DataOutputStream out;
        String name;

        ClientSender(Socket socket, String name) {
            this.socket = socket;
            try {
                out = new DataOutputSTream(socket.getOutputStream());
                this.name = nmae;
            } catch(Exception e) {}
        }

        @Override
        public void run() {
            Scanner scanner = new Scanner(System.in);
            try {
                if(out!=null) {
                    out.writeUTF(name);
                }

                while(out != null) {
                    out.writeUTF("[" + name + "]" + scanner.nextLine());
                }
            } catch (IOException e) {}
        }
    }

    static class ClientReceiver extends Thread {
        SOcket socket;
        DataInputStream in;

        ClientReceiver(Socket socket) {
            this.socket = socket;
            try {
                in = new DataInputStream(socket.getInputStream());
            } catch (IOException e) {}
        }

        @Override
        public void run() {
            while(in != null) {
                try {
                    System.out.println(in.readUTF());
                } catch (IOException e) {}
            }
        }
    }
}
```

- UDP
- UDP는 연결지향 프로토콜이 아니다. 따라서 ServerSocket이 필요 없다.
- UDP는 DatagramSocket class를 사용하며, 데이터는 DatagramPacket에 담아 전송한다.
```java
public class UdpClient {
    public void start() throws IOException, UnknownHostException {
        DatagramSocket datagramSocket = new DatagramSocket();
        InetAddress serverAddress = InetAddress.getByName("127.0.0.1");

        byte[] msg = new byte[100];

        DatagramPacket outPacket = new DatagramPacket(msg, 1, serverAddress, 7777);
        DatagramPacket inPacket = new DatagramPacket(msg, msg.length);

        datagramSocket.send(outPacket);
        datagramSocket.receive(inPacket);

        System.out.println("current server time :" + new String(inPacket.getData()));

        datagramSocket.close();
    }

    public static void main(String[] args) {
        try {
            new UdpClient().start();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}

public class UdpServer {
    public void start() throws IOException {
        DatagramSocket socket = new DatagramSocket(7777);
        DatagramPacket inPacket;
        DatagramPacket outPacket;

        byte[] inMsg = new byte[10];
        byte[] outMsg;

        while(true) {
            inPacket = new DatagramPacket(inMsg, inMsg.length);

            socket.receive(inPacket);

            InetAddress address = inPacket.getAddress();
            int port = inPacket.getPort();

            SimpleDateFormat sdf = new SimpleDateFormat("[hh:mm:ss]");
            String time = sdf.format(new Date());
            outMsg = time.getBytes();

            outPacket = new DatagramPacket(outMsg, outMsg.length, address, port);
            socket.send(outPacket);
        }
    }

    public static void main(String[] args) {
        try { 
            new UdpServer().start();
        } catch( IOException e) {
            e.printStackTrace();
        }
    }
}
```