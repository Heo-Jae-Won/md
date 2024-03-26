## <span style="color:#802548">_TCP 프로토콜_</span>
- 연결성과 신뢰성 있는 데이터 전송 프로토콜이다.
- UDP와 달리 연결을 맺고 끊는 과정이 존재한다.
  - 연결을 맺는 과정은 3 way handshake라고 부른다.
  - 연결을 끊는 과정은 4 way handshake라고 부른다.
- 3 way handshake의 과정은 아래와 같다.
  - client가 server에게 syn(x) 패킷을 보낸다. 
    - syn패킷에 시퀀스 번호를 포함해 보낸다. 
    - SYN 플래그 비트를 1로 설정한 세그먼트를 전송한다.
    - 이 때 client는 SYN_SENT이며, server는 listen 상태다.
  - server는 syn(x)을 받고 client에게 syn(y)/ack(x+1) 패킷을 보낸다. 
    - client의 Sequence Number에 1을 더해 ack 패킷에 넣어 보낸다.
    - YN과 ACK 플래그 비트를 1로 설정한 새그먼트를 보낸다.
    - 이 때 client는 CLOSED이며, server는 SYN_RCV 상태다. 
  - client는 server에게서  ACK(x+1)와 SYN(y) 패킷을 받고, ack 패킷(y+1)을 보낸다.
    - 이 때 client는 ESTABLISED이며, server는 ESTABLISHED가 된다.
- 3 단계를 거치면서 클라이언트와 서버 모두 데이터를 전송하고 받을 준비가 되었다는 것을 보장한다. 
- 이러한 3way handshake connection 과정을 통해 안전한 데이터 전송이 이루어 지게 된다.



<img src="/image/tcp-3way-handshake.png" />


- 4 way handshake의 과정은 아래와 같다.
  - client가 server에게 FIN패킷을 보낸다.
    - 이 때 FIN 패킷에는 ack(x)이 포함되어 있다.
    - 일단 client에서는 server에서 오는 데이터를 기다려야 한다. 
    - server가 보낸 잔여 데이터에 응답하지 않으면 서버가 커넥션을 닫지 못한기 때문이다.
    - 이 때 client는 FIN_WAIT상태다.
  - server는 FIN을 받고, 확인했다는 ACK를 client에게 보낸다. 
    - server는 ack을(x + 1)로 지정하고, ACK 플래그 비트를 1로 설정한 세그먼트를 전송한다.
    - server는 남은 데이터가 있다면 마저 전송을 마친 후에 close를 호출한다.
    - 이 때 server는  ClOSE_WAIT 상태다.
  - server는 연결이 종료에 합의 한다는 의미로 FIN 패킷을 client에게 보낸다.
    - server는 FIN 패킷을 보낸 뒤 client가 ack을 보내줄 때까지 기다린다.
    - 이 떄 server는 LAST_ACK 상태다.
  - client는 FIN을 받고, 확인했다는 ACK를 server에게 보낸다.
    - 아직 server로부터 받지 못한 데이터가 있을 수 있으므로 TIME_WAIT을 통해 기다린다
    - 이 때 client의 상태는 FIN_WAIT 에서 TIME-WAIT 으로 변경된다.
  - server는 ACK를 받은 이후 소켓을 닫는다 (Closed)
  - TIME_WAIT 시간이 끝나면 클라이언트도 닫는다 (Closed)
- ClOSE_WAIT/TIME_WAIT 상태를 일정 시간 유지하는 이유는 뭘까?
   - 아직 서버/클라에서 받지 못한 데이터가 연결이 해제되어 유실되는 경우를 대비해 잉여 패킷을 기다리는 것이다.
   - 이렇게 서로 완전히 닫지 않고 약간 대기하는 것을 half close라고 한다.
     - half close가 되면 송신은 가능하지만, 수신은 불가능하거나 그 반대 상황이 된다.


<img src="/image/tcp-4way-handshake.png" />


- TCP는 신뢰성 있는 프로토콜이라 불리는 이유가 있다.
- flow control과 congestion control을 통해 데이터의 신뢰성을 보장하기 때문이다.
- 먼저 flow control을 알아보자.
  - 송신과 수신 측 데이터 처리 속도의 차이를 해결한다.
  - receiver가 packet을 지나치게 많이 받지 않도록 조절하는 것이다.
  - receiver가 sender에게 현재 자신의 상태를 feedback 한다.
    - stop and wait: 매번 전송한 패킷에 대해 확인 응답을 받아야만 그 다음 패킷을 전송하는 방법
    - Sliding Window: 수신측에서 설정한 윈도우 크기만큼 송신측에서 확인응답없이 세그먼트를 전송하는 방법
    - 여기서 window란 단위 시간 내에 보내는 패킷의 수를 의미한다.
<img src="/image/sliding-window.png" />



- TCP는 원래 flow control 기능만 존재했다.
- 그런데 네트워크가 활발해지면서 혼잡을 통제하기 어려워졌다.
- 그에 따라 congestion control도 차후에 등장하게 되었다.
- 혼잡을 피하기 위해 송신 측에서 보내는 데이터의 전송속도를 강제로 줄이는 방식이다.
  - AIMD
    - 패킷을 보내 문제없이 도착하면 window 크기를 1씩 증가시켜가며 전송
    - 패킷 전송에 실패하거나 일정 시간을 넘으면 패킷의 보내는 속도를 절반으로 줄인다.
    - 네트워크가 혼잡해지고 나서야 대역폭을 줄이는 방식이다.
  - slow start
    - 패킷을 보내 문제없이 도착하면 각 ACK 패킷마다 window size를 1씩 늘려준다
    - AIMD가 선형적으로 전송속도가 증가했던 것과 달리 지수 함수 꼴로 증가한다. 
    - 대신에 혼잡 현상이 발생하면 window size를 1로 떨어뜨리게 된다.
  - fast recovery
    - 혼잡한 상태가 되면 window size를 1로 줄이지 않고 반으로 줄이고 선형증가시킨다.
    - 혼잡 상황을 한번 겪고 나서부터는 순수한 AIMD 방식으로 동작
  - fast retransmit
    - 중복된 순번의 패킷을 3개 받으면 재전송을 하게 된다.
    - 재전송은 곧 혼잡을 의미하므로 window size를 줄이게 된다.

## <span style="color:#802548">_UDP 프로토콜_</span>
- TCP와 달리 연결을 맺는 과정 없이 데이터를 통신하는 프로토콜이다.
  - 비연결, 비신뢰의 프로토콜이다.
  - flow control, congestion control, pakce retransmit, handshake도 없다.
  - 위의 신뢰성을 보장하는 기법이 모두 없어 그만큼 속도가 빠르지만, 패킷 손실도 그만큼 높다.
  - 주로 패킷손실을 감내할 수 있는 영상 스트리밍, 인터넷전화, DNS 등에 쓰인다.
  - UDP에서도 데이터의 무결성을 위한 checksum은 존재한다.
    - 전송된 데이터의 값이 변경되었는지(무결성)를 검사하는 값으로, 수신된 데이터에 오류가 없는지 여부를 확인한다.
    - IP Header 와 UDP header에 있는 일부 정보들을 가져와 IPv4 Pseudo Header 를 만든 후에 계산해 줘야 한다.
    - checksum 값은 선택이며, 0으로 보내면 수신자 측에서 별도로 검증하지 않는다.


<img src="/image/UDP-checksum-calculate.png" />



## <span style="color:#802548">_전송후대기 프로토콜_</span>
- RDT는 reliable data transfer로 신뢰성 있는 데이터의 전송을 위한 프로토콜이다.
- RDT 1.0은 완벽하게 신뢰할 수 있는 채널이라 별도의 조치가 없다. 이론적으로 존재한다.
- RDT 2.0은 비트 오류를 가정한다.
  - ACK, NAK으로 재전송 여부를 결정하는 ARQ 프로토콜이다.
  - 기본적으론 stop and wait 방식이다.
- RDT 2.1은 여기서 sequence number가 붙였다.
  - 중복이면 같은 seq number가 오고, 새로운 거면 +1된 seq number가 온다.
- RDT 2.2는 NAK 대신 ACK만 사용한다.
  - 패킷의 정상 수신 여부를 ACK만으로 판별할 수 있게 되었다.
- RDT 3.0은 비트 오류 및 패킷 손실을 가정한다.
  - 패킷이 손실되었을 때 행위는 RDT 2.2와 동일하다.
    - checksum, seq number, ACK, 재전송 등..
  - 다른 것은 패킷 손실을 검출하는 방법이 추가됐다는 점이다.
    - Timer를 이용한다. 정해진 시간안에 답변이 n번 오지 않으면 손실이라고 보는 식이다.
- RDT 3.0까지는 모두 stop and wait 방식이라 느리다.
- 그래서 ACK을 기다리지 않고 한꺼번에 packet을 보내는 방법이 등장했다.
- 그게 바로 파이프라이닝 프로토콜이다. 훨씬 더 빠르다.



## <span style="color:#802548">_파이프라인 프로토콜_</span>
- 기존의 RDT 3.0으로는 stop and wait 방식이라 고속으로 네트워크를 활용할 수가 없다.
- RTT(Round Trip Time, 하나의 패킷이 전송되고 ACK가 돌아올 때까지의 시간)동안 다른 packet을 보낼 수 없기 때문이다.
- 그래서 한번에 많이 보내는 프로토콜인 파이프라인 프로토콜이 개발되었다.
  - Go-Back-N
    - 만약 윈도우 크기가 3이면 ACK가 돌아올 때까지 3개까지의 메시지를 동시에 보낼 수 있는 것이다.
    - ack11은 11번 packet에만 한정된 ack이 아니라 11번 패킷까지 모두 잘받았다는 의미다.
    - 대신 어느 패킷까지 전송되었는지 알 수 없다. 누적이기 때문이다.
    - 그런 이유로 유실되어 timeout이 발생할 시 재전송할 패킷이 불분명하다. 따라서 모두 재전송해주게 된다.
    - 재전송하기 위해서 보냈던 packet은 모두 송신자가 버퍼에 담고 있어야 한다. 수신자는 버퍼가 불필요하다.
    - go-back-n의 문제는 정상 전송된 패킷이 버려지고 재전송된다는 점이다.
  - Selective repeat
    - go-back-n의 문제를 개선했다. 패킷이 유실되어 재전송할 때 선별적으로 패킷을 재전송한다.
    - go-back-n과 달리 패킷을 받을 때마다 각각 ACK를 전송시켜준다.
    - ACK 7을 받게 되면 여태까지의 패킷을 모두 받았다는 게 아니라, 7번 패킷만 받았다는 의미가 된다.
    - 패킷이 순서대로 정렬되어야 하므로 순서의 패킷을 임시로 저장해 줄 필요가 있다. 여기서는 송신자와 수신자 모두 버퍼를 사용한다.
    - 2번 패킷을 받지 못하고 3, 4번 패킷을 받았을 때 3,4번 패킷을 수신자 버퍼에 저장해주게 된다.
    - 2번 패킷을 다시 보내면 송신자는 2번 패킷과 더불어 버퍼의 내용(3, 4, 5번 패킷)도 함께 저장해준다.
    - 0, 1, 2, 3, 4, 5번 패킷이 순서대로 정렬되게 된다.
    - 시퀀스 번호를 전달하기 위해서는 세그먼트 헤더에 정보를 넣어야 하는데, 더 큰 수를 지원하려면 더 많은 비트가 필요해져서 헤더 또한 길이가 길어지게 된다.
    - 대략 window size가 N이라면 시퀀스 번호는 N*2 정도가 필요하다.
  - TCP는 두개의 방식이 섞였다.
    - buffer의 경우 sender와 receiver 양측 모두에 존재한다.
    - ack은 cumulative ACK 방식을 사용한다
    - Timer는 각 packet마다 별도로 만들지 않고 os별 공용 timer 설정을 사용한다.
    - Timeout이 되거나 3-duplicated ACK를 받았을 때 fast retransmission이 일어난다.


## <span style="color:#802548">_출처_</span>
- https://velog.io/@nnnyeong/Network-TCP-3-way-4-way-Handshake - tcp 3/4 way handshake
- https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=sdug12051205&logNo=221053748674 - tcp half close
- https://gyoogle.dev/blog/computer-science/network/%ED%9D%90%EB%A6%84%EC%A0%9C%EC%96%B4%20&%20%ED%98%BC%EC%9E%A1%EC%A0%9C%EC%96%B4.html - tcp 흐름제어/혼잡제어
- https://www.cloudflare.com/ko-kr/learning/ddos/glossary/user-datagram-protocol-udp/#:~:text=%EC%82%AC%EC%9A%A9%EC%9E%90%20%EB%8D%B0%EC%9D%B4%ED%84%B0%EA%B7%B8%EB%9E%A8%20%ED%94%84%EB%A1%9C%ED%86%A0%EC%BD%9C - udp
- https://limjunho.github.io/2021/06/05/UDP-cksum.html - udp checksum
- https://dev-nicitis.tistory.com/28 - 파이프라인 프로토콜
- https://dev-nicitis.tistory.com/29 - timeout 설정
- https://huammmm1.tistory.com/168 - tcp의 파이프라인 프로토콜
