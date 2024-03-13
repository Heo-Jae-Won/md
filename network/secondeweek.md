## <span style="color:#802548">_HTTP protocol_</span>
- TCP/IP 통신 위에서 서버 -클라이언트 간 데이터를 교환하는 프로토콜이다.
- 아래와 같은 특징을 지닌다.
  - 비연결지향성: client가 request를 날리면, 연결이 이뤄지고 response를 받으면 연결이 끊어진다. 연결에 드는 컴퓨팅자원이 최소화되어 더 많은 연결(동접)을 감당한다.
  - 무상태성: 이전 request를 기억하지 않기에, 어떤 서버든 request를 이어받을 수 있다. 수평확장(서버 기기 늘리기)을 통한 트래픽 대응에 능하다.
    - 상태 유지가 필요하다면 session(WAS에 저장), cookie(client에 저장) 등을 사용할 수 있다.
    - 실제 cookie는 C:\Users\<username>\AppData\Local\Google\Chrome\User Data\Default\Network\Cookies 와 같은 곳에 저장된다고 한다.


## <span style="color:#802548">_HTTP history_</span>
- HTTP 0.9 
  - GET으로 조회만 했고, HTML만 전송했다.
- HTTP 1.0
  - 프로세스를 처리하기 위해 POST가 도입됐다.
  - 프로세스를 처리하는 데 도움을 주는 부가정보인 header도 도입되었다.
  - header에서 contentType을 지정하면서 HTML 이외 파일도 전송하게 되었다.
  - 데이터가 처리된 결과를 알려주는 status code도 도입되었다.
  - TCP/IP connection을 재활용하기 위한 keep-alive 기술이 도입되었다.
- HTTP 1.1
  - 컨텐츠 협상 헤더가 도입되었다. 
  - 수정, 삭제 등을 알리는 PUT, DELETE가 도입되었다.
  - keep-alive를 진화시킨 persisent connection, 이를 더 진화시킨 pipelining이 등장하였다.
    - 매번 3 way handshake를 하지 않게 되어 지연시간이 줄어들었다.
    - 그러나 먼저 받은 요청이 response가 안오면 그 뒤 요청들이 response가 도착하지 못했다(HOLB).
    - 전체 pipeline이 막혀버리는 건 subsequent request도 막히는 중대한 문제였기 때문에 현대 브라우저들은 이 기능을 지원하지 않기로 한다.
    - IE11이상 버전, 크롬, 사파리, 엣지, 파이어폭스 등은 pipelining을 지원하지 않는다. 대신 HTTP2의 multiplexed stream을 지원한다.
- HTTP2 이전까지 2000년대는 아래와 같다.


```
Restful API
PATCH method가 도입되었다.
entity가 아니라 representation으로 명칭하게 RFC 스펙이 바뀌었다.
```


- HTTP2
  - 페이지 반응속도를 개선하는데 주안점을 두었다.
  - 그 결과 Server push, 중복헤더 제거 및 압축, multiplexed streams, binary protocol 등이 도입되었다.
  - Server push는 client가 필요로 할 듯한 data, 특히 js나 image를 한꺼번에 서버가 판단해 보내버린다.
    - 사실상 사용되지 않는다. 미리 보낼 resource의 기준이 모호해서다.
  - multiplexed stream은 브라우저가 하나의 커넥션에서 한꺼번에 request를 쏟아내고 순서에 상관없이 response를 받게 하는 것이다.
    - 처리시간이 오래걸리는 request를 보내고 이어서 request를 보내도 response가 blocking되지 않고 client에 도달하게되었다.
  - binary protocol은 HTTP 1.1의 패킷구조가 text 기반인데에 반해, 2는 0과 1로 이뤄진 binary라는 점이다.
    - binary가 byte가 덜나가고 압축이 잘되는 편이라 데이터의 크기가 줄어든다. 
    - 물론 browser가 decode해서 개발자도구창에서는 그냥 text로 보인다.



## <span style="color:#802548">_HTTP 데이터 전송 과정_</span>
- HTTP에서 데이터는 아래와 같이 전송된다.
  - browser에서 url을 친다. 즉 request를 웹 서버에 날린다.
  - 이 때 request에는 header와 body가 들어간다. 
    - body는 data, header는 server가 data를 처리하는 데 도움을 주는 메타데이터다.
  - 웹 서버는 해당 request를 보고 설정에 따라 처리한다.
    - 보통 css, js, image은 nginx 같은 웹서버가 반환해준다.
    - 그게 아닌 경우는 WAS로 넘긴다.
    - WAS로 넘겨진 데이터는 보통 DB처리가 필요하다. 
    - DB처리를 거쳐 WAS가 client에 data를 return할 수도 있고, html(jsp)를 return할 수도 있다.

<img src="/image/webserver-config.png" />

- 위는 아주 간단한 설정파일 구성이다. port forwarding이 된 상태다.
- url에 upload가 붙어있는 경우에 Web Server가 처리하는 경로기반 라우팅이다.

## <span style="color:#802548">_HTTP 데이터 구성_</span>
- 메시지(data)는 첫째줄을 빼면 header와 body로 이뤄진다.
- header는 HTTP Message 번역에 필요한 메타 데이터의 집합이다.
  - 기본적으로 request와 response가 있다.
  - request header는 요청에 담긴 맥락을 제공한다.
  - 그 중에 caching가 관련이 있는 header도 있다.
  - image,js,css 등 caching 만료 여부 판단을 서버에 요청한다.
    - if-none-matchs가 그렇다. 캐싱을 사용하라고 하면 304를 client에 return한다.
- 자세한 헤더는 아래를 참조하자.

<img src="/image/request-header.png" />

- response header는 반응에 담긴 자세한 맥락을 제공한다. 
  - if-none-match에 대응되는 게 Etag다.

- header에 의한 caching을 간단히 설명하면 아래와 같다.
- 기본적으로 웹 browser 자체 내부에서 caching 관련 header가 들어가는 편이다.
```
클라이언트가 resource 요청을 보낸다. 
서버는 Etag response header와 resource를 return한다.
클라이언트가 같은 resource 요청을 또 보낸다. 이 때 If-None-Match request header를 같이 보낸다.
서버가 resource 요청에 온 If-None-Match header를 까서 현재 서버 내 resource의 Etag와 비교한다.
바뀐게 없다면 304 status code만, 바꼈다면 200 status code와 함께 resource를 return한다.
```

- 참고로 general header는 RFC spec에서 사라졌지만, chrome에선 여전히 보인다.
- 첫째줄에는 request의 경우 HTTP method와 버전이 명시된다.
- response의 경우 http version과 status code가 명시된다.


## <span style="color:#802548">_HTTP STATUS CODE_</span> 
- success: 200번대
  - 200: 일반적 성공
  - 201: POST(생성) 성공
  - 204: PUT, PATCH(수정) 성공
- redirect: 300번대
  - 304: caching이 유효함
  - 307: 임시 url redirect
  - 308: 영구 url redirect
- client fail: 400번대
  - 400: server가 요구하는 request양식에 맞지 않는 data 형식(쿼리파라미터가 없다거나..)
  - 401: 인증 없음
  - 403: 권한 없음
  - 404: not found
  - 405: 허용되지 않는 HTTP method
  - 415: Server와 Client 간 content-type 불일치
- server fail: 500번대
  - 500: WAS 내 로직 오류
  - 502: 웹서버 설정 오류


## <span style="color:#802548">_HTTP METHOD_</span> 
- GET
  - 조회용도.
  - resource 변화 X
  - 멱등성(매 호출마다 resource 변화) O
  - 캐싱 O
- POST
  - 생성용도
  - resource 변화 O
  - 멱등성 X
  - 캐싱 X
- PUT
  - 전체수정
  - resource 변화 O
  - 멱등성 여부는 구현법에 따라..
  - 캐싱 X
- PATCH
  - 일부수정
  - resource 변화 O
  - 멱등성 O
  - 캐싱 X
- DELETE
  - 삭제
  - resource 변화 X
  - 멱등성 O
  - 캐싱 X
- HEAD
  - 조회용도. hedaer만 return
  - 보통 캐싱 용도
- OPTIONS
  - CORS preflight용도



## <span style="color:#802548">_HTTPS_</span> 
- HTTPS는 HTTP통신에서 데이터를 전송할 때 보안을 강화하기 위해 데이터를 암호화해 보내는 프로토콜이다.
- HTTPS가 암호화에 사용하는 프로토콜은 SSL/TLS다.
- SSL/TLS는 SSL handshake를 하는데, 목적은 아래와 같다.
  - 비대칭키를 이용해 TCP 패킷들을 암호화    (패킷의 감청 방지), 
  - 패킷에 전자서명을 첨부하며              (패킷의 변조 방지), 
  - 서버 측에서 공개키에 대한 인증서를 제공  (서버의 신원 인증)
- SSL handshake는 요약하면 아래와 같이 진행된다.
  - 클라이언트가 요청을 날리면 서버는 인증서를 제공하여 신원을 보증한다.
  - 그 뒤엔 비대칭키 교환 방식을 통해서 서버와 클라이언트가 대칭키를 교환한다.
  - connection 이후 data 통신에서는 서버측의 연산량을 절감하기 위해서 대칭키를 사용해 암/복호화를 진행한다.
- 길게 말하자면 아래와 같다.
```
클라이언트 요청: 클라이언트가 서버에 접속을 요청하고 SSL/TLS 핸드셰이크를 시작합니다.
서버 응답: 서버는 클라이언트에게 자신의 인증서를 제공하고, 서버의 공개 키와 함께 대칭 키를 생성합니다.
클라이언트 키 교환: 서버는 공개 키로 대칭 키를 암호화하여 클라이언트에게 전달합니다. 클라이언트는 이를 안전하게 수신하여 복호화합니다.
대칭 키 사용: 이제 클라이언트와 서버는 공유된 대칭 키를 사용하여 통신을 암호화하고 복호화합니다.
```

- 더 길게는 아래와 같다.
- ClientKeyExchange에 써있는 비밀키는 개인키(비대칭 키 교환의 비밀키)가 아니라 대칭키(대칭 키 교환의 비밀키)다.
<img src="/image/ssh-handshake.png" />

- SSL은 TCP와 application 사이의 독립 계층이며 암호화 역할을 한다.
- application -> SSL -> TCP/ TCP -> SSL -> application와 같은 형태다.
- TLS은 SSL의 개선된 버전이다.
  - SSL보다 handshake 단계가 짧아 속도가 빠르다.
  - 메시지 인증코드 알고리즘은 더 안전한 HMAC을 쓴다.
  - 암호화 알고리즘이 보안성능에서 더 뛰어나다.
- SSL의 암호화 알고리즘은 공개키-개인키(비대칭키)기반이다.
  - 보내는 측은 받는 측의 공개키를 구해 데이터를 그 공개키로 encrypt해 전달한다.
  - 받는 측은 자신의 개인키로 해당 데이터를 복호화하여 사용한다.
- SSL은 handshake 중에 인증서를 주고받는데, 인증서를 받는 과정에서 전자서명 과정을 거친다.
  - 원본 데이터를 공개된 방법으로 해싱하여 특정 길이의 문자열로 만든다. 
  - 그 해시값을 송신자의 개인키로 암호화한 전자서명을 생성하여 원본 데이터에 첨부하고, 수신자에게 발송한다.
  - 이후에 수신자는 받은 전자서명을 송신자의 공개키로 복호화하고 이를 받은 데이터를 해싱한 해시값과 비교한다. 
  - 이때 두 해시값이 동일한지 비교하여 데이터의 송신자와, 데이터가 변조되지 않았음을 검증할 수 있다.
    - 그러나 전자서명만으로는 데이터 생성자, 즉 공개키 소유자의 실제(오프라인) 신원을 증빙할 수 없다.
    - 공개키/개인키 키 쌍의 소유자의 신원을 보증하기 위해서 인증서(Certificate)를 이용한다.
    - 사용자들은 암호화나 전자서명에 이용할 키 쌍을 정부기관이나 은행 등 공인된 기관에서 개인정보를 인증하며 발급 받을 수 있다.
    - 발급 받는 사람은 기관으로부터 생성된 개인키와 그에 대한 인증서를 받게 된다.


- 아래는 전자서명의 예시다.    
<img src="/image/electronic-signature.png" />


## <span style="color:#802548">_DNS_</span> 
- DNS는 도메인 주소를 IP 주소로 변환하는 프로토콜이다.
- dns의 과정은 아래와 같다.
  - 클라이언트는 domain을 받아 DNS서버에 질의한다(DNS query).
  - 보통 클라이언트의 DNS 설정에 지정된 DNS 서버로 전송된다.
    - 가정집은 공유기나 DHCP 서버가 자동으로 DNS 서버 정보를 제공한다.
    - 도메인 컨트롤러로 설정된 서버를 DNS 서버로 구성할 수 있습니다.
  - 지정된 DNS서버는 로컬 DNS 캐시를 확인해 있다면 해당 IP를 return한다.
  - 만약 없다면 DNS서버가 domain에 해당하는 IP 주소를 반환한다.
  - 해당 DNS서버가 못찾았다면 다른 DNS서버에게 물어서 알려주거나, 다른 DNS IP를 알려준다.
  - 어쨌든 최종적으로 DNS서버는 도메인에 맞는 IP 주소를 반환하고, local DNS 캐시에 저장한다.
  - 만약에 root까지 가서도 없다면, 오류 안내 페이지가 뜬다.
  - IP를 찾으면 클라이언트는 해당 IP에 해당하는 host에 데이터를 전달한다.

<img src="/image/dns-probe-fail.png" />

  

- dns routing은 아래와 같이 이뤄진다.
- dns resolver는 dns 서버다.
<img src="/image/dns-routing.png" />


- DNS query는 재귀방식, 반복 방식이 있다.
  - 재귀 방식은 하나의 네임서버가 응답을 담당한다.
    - 네임서버가 다른 네임서버에 계속 물어봐서 클라이언트에게 준다.
  - 반복 방식은 여러개의 네임서버가 응답을 담당한다.
    - 한 네임서버가 모르면 다른 네임서버에게 물어보지 않고, 다른 네임서버의 IP를 알려준다.


## <span style="color:#802548">_DNS RECORD_</span> 
- 주요 레코드는 아래와 같다.
  - A:      도메인을 IPv4로 변환
  - AAAA:   도메인을 Ipv6로 변환
  - CNAME:  도메인 주소 별칭. www를 붙인 도메인이 많이 들어간다.
  - SOA:    도메인 영역의 권한. 필수레코드. 관리자의 이메일 주소, 도메인이 마지막으로 업데이트된 시간, 새로 고침 사이에 서버가 대기해야 하는 시간(REFRESH)
  - NS:     네임서버 설정

- DNS Record는 domain 속성과 레코드를 정의하는 zone 파일에 사용된다.

<img src="/image/dns-zone.png" />

- 단 A에 써야하는 IP주소는 외부에서 인식되는, 접근 가능한 공인 IP를 써야한다.
- @는 root 도메인을 의미한다. root domain은 named.conf에 설정해줄 수 있다.

<img src="/image/dns-named-conf.png" />





## <span style="color:#802548">_출처_</span>
- https://stackoverflow.com/questions/58498116/why-is-it-said-that-http2-is-a-binary-protocol - http2가 binary protocol인 이유
- https://velog.io/@baekrang256/Problems-of-HTTP2-Server-Push - 왜 server push는 사용되지 않는가
- https://velog.io/@cjh8746/HTTP-Keep-Alive-%EC%99%80-pipelining-%EA%B7%B8%EB%A6%AC%EA%B3%A0-Multiplexed-Streams%EC%9D%84-%EA%B3%B5%EB%B6%80%ED%95%98%EB%8B%A4%EA%B0%80-%EC%95%8C%EA%B2%8C%EB%90%9C-%EB%B2%84%EC%A0%84%EC%97%B4-HTTP0.9-2.0-%EC%A0%95%EB%A6%AC
- https://stackoverflow.com/questions/36517829/what-does-multiplexing-mean-in-http-2 - http2의 multiplexing
- https://m.blog.naver.com/xcripts/70122755291 - SSL/TLS handshake
- https://epthffh.tistory.com/entry/%EC%95%94%ED%98%B8%ED%99%94-%EC%A0%84%EC%9E%90%EC%84%9C%EB%AA%85-%EC%9D%B8%EC%A6%9D%EC%84%9C%EC%99%80-SSL - SSL의 암호화 알고리즘
- https://aboutssl.org/what-is-digital-signature-how-does-it-work/ - 전자서명 알고리즘
- https://aws.amazon.com/ko/route53/what-is-dns/ - dns routing
- https://silver-liq9118.tistory.com/m/entry/DNS-DNS-%EA%B5%AC%EC%A1%B0-DNS-%EC%A7%88%EC%9D%98-%EA%B3%BC%EC%A0%95 - 한국의 dns routing
- https://tech.ktcloud.com/66 - linux dns 파일 셋팅