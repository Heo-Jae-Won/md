- EC2는 컴퓨터를 빌려서 원격으로 접속해 사용하는 서비스다.
  - 즉 EC는 하나의 컴퓨터다.
- 서버를 배포하려면 서버가 필요한데, 그 컴퓨터에 보안, 로깅, 오토스케일링, 로드밸런싱 등의 기능을 AWS에서 제공해준다.
- AWS EC2는 backend에서 주로 쓴다.
- front의 경우에는 vercel, netlify나 AWS S3를 사용해서 주로 배포한다.

<br>

- region이란 지리별로 나뉜 각각의 데이터 센터다.
- EC2가 서울에 있거나, 도쿄에 있거나 등 데이터 센터별로 전부 나눠져있다.
- 네트워크는 지리상에서 가까울수록 빠르기 때문에 주 사용자들의 위치를 고려해 리전을 결정해야 한다.

<br>

- 실제 서버를 실행시키려면 instance를 만들어줘야 한다.
- Application and OS images는 어떤 OS를 가진 컴퓨터를 가져올 지 선택한다.
 - window나 mac은 일반 사용자를 위한 부가 기능이 굉장히 많아 서버로서 택하기엔 지나치게 무겁다
 - 배포용 서버로서 필요한 기능들 위주로 들어가는 ubuntu나 red hat을 사용한다.


 <br>

- instance는 컴퓨터를 빌리는 단위다. instance가 1이면 컴퓨터 1대를 빌리는 것이다.
- instance 유형은 컴퓨터 사양을 의미한다. 사양이 좋을수록 더 빠른 처리가 가능하다.
- 프리티어라고 해도 하루 2000명 정도까지는 돌아간다고 한다. 사양에 문제가 생기면 그 때 바꿔주자.
- key pair는 인증기능이다. 누구나 instance(컴퓨터)에 접근하면 안되기 때문이다.
- key pair는 어떤 서버용인지 도메인을 바로 알아볼 수 있게 써야 한다.
- key pair를 발급받으면 잃어버리면 안 된다. 보통 RSA방식의 공개-개인키를 사용한다.


<br>

- 네트워크 설정에서는 VPC가 있는데, VPC는 AWS 입문에서 중요하지 않다.
- 보안그룹은 필수로 알아야 한다. 
- 보안 그룹이란 AWS 클라우드에서의 네트워크 보안을 의미한다.
- 쉽게말하면, EC2 instance가 집이라면, 보안그룹은 울타리와 대문이다.
- 보안 그룹에 규칙을 지정하면, 인바운드 트래픽과, 아웃바운드 트래픽의 허용할 IP 범위와 port 설정이 가능하다.
- 보통 SSH TCP 22, 위치무관인데, 어떤 IP든 HTTPS를 허용한다.

<br>

- EC2도 컴퓨터라 저장 공간이 필요하다. 그 저장공간을 EBS라고 부른다.
- Elastic block storage를 포괄적인 용어로 Storage, Volume이라고도 부른다.
- 한마디로 EBS는 SSD, HDD라고 보면 된다. 프리티어는 30기가가 최대다.
- launch instance를 눌러서 시작시켜주자. EC2 dashboard에서 running으로 뜨면 시작된 것이다.

<br>

- 이제 들어가서 summary를 보면 public ipV4가 나온다. 그게 현재 EC2 instance의 공인 IP 주소다.
- 상태가 실행중이면 서버가 꺼지지 않았다는 것이다.
- 인스턴스 상태도 조종할 수 있는데, 중지, 시작, 재부팅, 절전, 종료 등이 가능하다.
  - 인스턴스 종료는 컴퓨터를 삭제하는 것이다. terminate다. 따라서 일반적인 종료를 생각하면 안 된다.
- 아래 security tab을 눌러보면 우리가 이전에 설정했던 security group rule을 살펴볼 수 있다.
- outbound는 정하지 않았다면 all이 default 값이다.
- 모니터링은 꽤나 자주 보게 된다. 컴퓨터의 성능을 보여주는 지표기 때문이다.
  - CPU는 얼마나 쓰고, IO는 얼마나 쓰고, RAM은 얼마나 쓰고 등등을 보아야 할 때가 있다.

  <br>

  - EC2 instance를 실제로 활성화되었다면, 접속(connect)까지 해주어야 한다.
  - 우리는 EC2 instance를 사용할 것이다. 인터넷에 접속 script가 뜨게 된다.
  - EC2 처음 접속 시 그냥 기본만 깔려 있다.

  <br>

  - 이제는 탄력적 IP가 필요하다. EC2 instance로 할당받은 IP는 임시기 때문이다.
  - 매번 실행마다 IP가 바뀌면 배포가 어렵다. 고정 IP를 할당받는데, 그게 바로 탄력적 IP다.
  - 고정인데 왜 탄력적 IP인가에 대해서는 그냥 elastic을 어필하려고 그랬던 게 아니라 그냥 keyword를 밀어서 그런 것 같다.
  - 처음부터 고정 IP를 주지 않은 이유는 IPv4 갯수 고갈 때문이다.
  - 왼쪽 tab에 allocate elastic IPs 클릭한다. ap-northeast-2로 설정되어 있지만, 그대로 쓰면 된다.
  - name edit를 해서 우리가 쉽게 알아볼 수 있는 이름으로 바꿔놓는다.
  - actions를 클릭해서 associate elastic IP address를 클릭한다.
  - 만약 instance가 여러개라면, 원하는 instacne를 연결시켜야 한다. 그래야 해당 instance에 고정 IP가 할당된다.

  <br>

  - 이제 배포를 시작해보자.
  - 먼저 가벼운 express를 배포해보자.
  - node.js를 먼저 깔아주고 기본셋팅을 만져준다.

```sh
sudo su
$ apt-get update && /
apt-get install -y ca-certificates curl gnupg && /
mkdir -p /etc/apt/keyrings && /
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg && /
NODE_MAJOR=20 && /
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list && /
apt-get update && /
apt-get install nodejs -y
```

- 노드가 잘 깔렸는지 확인하게 명령어를 써준다.
- 제대로 깔렸는지 중간중간에 한번 확인해주는 행위가 매우 중요하다.


```sh
node -v
```

- github에서 express 프로젝트를 clone한다.

```sh
git clone ~
cd folder이름
npm i
.env파일에 DATABASE_NAME=my_database
```

- port 80번은 생략이 가능하다.
- 명령어를 사용하여 배포하고, 자기 public IP를 브라우저 검색창에 넣어준다.

```sh
npm i pm2
sudo start pm2 app.js
```

<br>

- 스프링 부트는 JDK 17 버전으로 깔아준다.

```sh
$ sudo apt update && /
sudo apt install openjdk-17-jdk -y
```

- 메시지가 아래처럼 뜨면, 제대로 안 깔린 것이니 다시 apt install을 해준다.

```
command java not found, but can be installed with
apt install ~~
```


- JDK가 제대로 깔렸는지 확인하려면 명령어를 사용한다.

```sh
java -version
```

- 이전에 켰던 express 서버를 꺼줘야 한다.
- 두 개가 포트가 같아서 충돌이 난다.

```sh
lsof -i :80
kill -9 [pid]
```

- 위처럼 해도 pm2가 계속 새로운 port를 부여한다.
- pm2 자체를 죽여야 한다.

```sh
pm2 kill
```

- 껐다면 jar가 있는 곳을 찾아서 jar 파일을 실행시킨다.

```sh
cd build/libs
sudo nohup java -jar ec2-spring-boot-sample-0.0.1-SNAPSHOT.jar &
```

- EC2를 다 썼다고 생각이 들면 instance를 terminate해 준다.
- terminate하고나서, 탄력적 IP도 같이 없애야 한다. release elastic IP다.

<br>

- 여태까지는 IP를 그대로 넣었지만, 사실 대부분은 도메인을 사용한다.
- 따라서 domain을 연결해주고, DNS를 설정해주는 게 필요하다.
- route 53은 DNS를 처리해주는 서비스다.
- 특히 그냥 IP주소에는 HTTPS를 적용할 수 없다. 도메인이 필수로 요구되는 이유다.
- gabia나 후이즈 등에서도 도메인을 구매하고 관리할 수 있다. 자신이 원하는 형태의 도메인을 구매할 수 있으면 된다.

- 우리는 무료로 진행하니 무료로 제공되는 내도메인한국 사이트에서 하나 가져오자.
- 도메인을 만들었다면 해당 사이트에서 IP연결에 내가 연결한 탄력 IP를 적어두자.
- 참고로 자주 끊기기 때문에 운영서버에서는 무료 도메인을 쓰지 않는게 좋다.

- route53 서비스로 도메인을 구매하고 사용해도 된다.
- 구매하면 hosted zone에 내가 구매한 도메인이 있다.
- 등록된 도메인으로 보면 바로 뜨지는 않을 수 있다. DNS를 관리하는 쪽에서도 승인이 나야 한다.
- 승인이 나서 정상인증되면 그 때서부터 도메인이 등록된다.
- 도메인이 등록되면 이제 EC2 instance와 연결해준다.
- record를 생성해준다. 우리는 A유형을 사용하면 된다. 
- value에는 우리의 공인 IP를 적어주면 된다.
- 레코드 이름을 앞에 설정하면 url 앞에 prefix가 붙게되는 효과가 있다.
  - api.jeowon.co.kr이면 api.jaewon.co.kr이 도메인이 된다.
  - 도메인을 jaewon.co.kr을 구매하면 prefix로는 무엇이 들어오든 상관없다.
  - 따라서 도메인 1개만 구매해도 api.jaewon.co.kr과 jaewon.co.kr을 모두 쓸 수 있다.


<br>

- ELB로 HTTPS를 연결한다.
- ELB는 Elastic Load Balancer라는 의미다.
- 서버가 두 대 이상인 경우는 필수로 ELB가 들어간다.
- load balancing을 위해 EC2 instance에 보내는게 아니라, ELB에 보내게 한다.
- ELB도 하나의 컴퓨터인데, 이 컴퓨터에 보내면, EC2 instance로 load balancing해준다.
- 사용자 콘솔의 leftside tab에 보면 load balancer가 있다.
- load balancer도 region을 따진다. 서울로 설정해주자.
- HTTP와 HTTPS의 특성을 활용하기에 application level의 load balancer를 만든다.
- VPC와 private IP는 나중에 활용한다. 현재 입문자 수준에선 필요없는 선택지다.

- mappings는 전부 다 체크해준다.

<br>

- 이제 security group을 만들어준다.
- rule은 inbound의 경우 HTTP와 HTTPS를 모두 받아들인다로 설정한다.
- outbound는 그대로 놔둔다. 이제 create하고, ELB의 security group에 넣어준다.

<br>


- 그 다음 리스너 및 라우팅을 만든다.
- target group을 생성해준다.
- IPv4, HTTP1로 설정해서 넣어준다.
- ELB에는 load balacning 기능이 있는데, 만약 load balancing할 instance가 죽었다면?
- 그러한 상황을 처리하기 위해 health check가 있다. health check용 url에서 200번대 응답이 없다면?
- 없으면 해당 instance로는 request를 배분하지 않는다. 다만 health check용 url을 backend에서 구현해야 한다.

<br>

- 다음을 누르면 대상 등록 페이지가 나온다. 어떤 port로 traffic을 보낼지 설정해준다. 보통 80이다.
- include as pending below 버튼을 눌러주면 해당 ELB에 포함되지 않은 instance가 나온다.
- 추가하고 싶은 EC2 instance를 넣어준다.
- 이제 target group은 전부 만들었다.
- 다시 돌아가, listener and routing에서 target group을 선택한다.


<br>

- 만들고 나서 load balancer를 보면, provisioning으로 뜬다.
- load balancer로서 추가된 ELB의 경우, IP가 없고, 도메인 주소를 가지고 접속할 수 있다.
- 도메인 주소는 DNS이름이다. 앞으로는 해당 ELB의 DNS를 front에서 받게끔 설정해야 한다.
- 물론 너무 기니까 ELB의 domain도 
- ELB에 넣게 되면 이제는 EC2 instance에 넣었던 DNS record는 필요 없으니 삭제한다.
- 똑같이 api.~~~로 레코드 이름을 만들고, A유형을 고른뒤, 별칭을 Application/Classic Load로 설정한다.
- region은 아시아 태평양(서울)로 설정하고, 아까 DNS 주소를 적어넣는다.

- CNAME 값을 



