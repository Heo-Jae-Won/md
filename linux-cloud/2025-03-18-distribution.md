## <span style="color:#802548">_AWS 프리티어 생성_</span>

- 스토리지는 30GB까지 공짜다.

<img src="/image/aws-instance1.png">
<img src="/image/aws-instance2.png">
<img src="/image/aws-instance3.png">


## <span style="color:#802548">_보안 그룹 생성_</span>

- SSH : 원격 접속
- HTTP / HTTPS : 웹 접근 여부
- DB : Mysql 3306 포트 및 외부접속 허용
- 사용자 지정포트 : 9005 → 프로젝트 설정 포트 외부접속


<img src="/image/aws-instance4.png">


## <span style="color:#802548">_원격 접속 생성_</span>

- extension에 Remote쳐서 나오는 Remote - SSH 다운로드
- 다운로드 이후 좌측 side bar에 추가된 Remote explorer 클릭
    - SSH 써져있는 곳에 줄 대면 우측에 OpenSSH Config file 보임. 클릭
    - C:\Users\user\.ssh\config 클릭하면 config 쓰는 화면으로 전환됨

- 반드시 tab을 정확하게 4칸으로 맞춰서 적어줘야 한다.

```sh
# Read more about SSH config files: https://linux.die.net/man/5/ssh_config
Host tanomuzoko (진짜 그냥 이름)
    HostName 16.159.2.11 (여기는 ip)
    User ubuntu (유저는 ubuntu 고정인듯?)
    IdentityFile C:/Users/user/.ssh/tanomuzoko.pem (처음에 keypair 생성하고 나서 이후의 복사해서 가져온 private key)
```

- 아래처럼 여러개의 host를 만드는 것도 가능하다.

```sh
# Read more about SSH config files: https://linux.die.net/man/5/ssh_config
Host tanomuzoko1 (진짜 그냥 이름)
    HostName 16.159.2.11 (여기는 ip)
    User ubuntu (유저는 ubuntu 고정인듯?)
    IdentityFile C:/Users/user/.ssh/tanomuzoko.pem (처음에 keypair 생성하고 나서 이후의 복사해서 가져온 private key)
Host tanomuzoko2
    HostName 16.159.2.11 
    User ubuntu 
    IdentityFile C:/Users/user/.ssh/tanomuzoko.pem 
```

## <span style="color:#802548">_외부 접속 가능한 session key 만들기_</span>

- 키는 window에 가져왔다고 가정했지만, key를 만드는 과정이 이전에 필요한다.
- 처음에는 aws를 통해서 들어가게 딘다. 따라서 처음 들어간 session에서 key를 만드는 것이 시작이다
- 서버에는 공용키를 추가한다. cat의 경로는 잘 조절해서 넣어주자.
- 저 명렁어를 써서 authorized_keys에 공용키가 들어가야 AWS instance에 접속가능하다.

```sh
ssh-keygen -t rsa -b 4096 -f ~/.ssh/my-aws-key
cat ~/my-aws-key.pub >> ~/.ssh/authorized_keys
```

- 우리는 개인키를 window로 가져와야 한다.
- 노출된 내용을 복사해서 window에서 만들어주자.
- 메모장으로 만들어서 쓰고, 모든 파일로 한 다음 ~~.pem으로 저장해주자.
- window의 CRLF라도 상관없고, UNIX의 LF라도 상관은 없었다. 
    - 중요한 것은 ---END~~~---하고 나서 한 줄을 무조건 띄어주고 저장하는 것이다.

```sh
cat my-aws-key
```


- key로 접속한 다음 볼 일 다 봤으면 아래 명령어 실행하여 session 종료시킨다.
- 종료하지 않으면 해당 key는 다른 데서 사용 불가능. session 1개에 key 1개기 때문이다.

```sh
exit
```


## <span style="color:#802548">_EC2 SWAP_</span>

- EC2의 RAM이 너무 적기 때문에 swap 해줘야 한다.
- 적은 RAM으로는 DB까지 구축하는 것이 불가능에 가깝기 때문이다.


<img src="/image/aws-instance5.png">


```sh
sudo swapon --show                  #SWAP 확인
sudo fallocate -l 2G /swapfile      #SWAP 파일 생성
sudo chmod 600 /swapfile
sudo mkswap /swapfile

# SWAP은 최대 기존 RAM에 2배만큼만 하는게 성능이 가장 좋음
# 프리티어는 1G라서 2G로 SWAP 함
```

## <span style="color:#802548">_systme update_</span>

```sh
sudo apt update -y    # Ubuntu
```

## <span style="color:#802548">_systme update_</span>

- git을 깔아주자.

```sh
sudo apt install git -y    # Ubuntu
git --version #버전확인
```

- git ssh key 생성 후 공용키를 복사한다.

```sh
cd ~/.ssh
ssh-keygen -t rsa -C github계정 메일(example@github.com)
vi id_rsa.pub
```

- github -> setting -> SSH and GPG keys 탭으로 이동후 new SSH key 버튼을 클릭
- id_rsa.pub의 내용 붙여 넣고 키등록을 마친다.
- clone 중 SSH를 복사한 뒤 AWS instance에서 붙여넣기
- 키 신뢰여부는 yes다.

<img src="/image/aws-instance6.png">


## <span style="color:#802548">_JDK_</span>

```sh
sudo apt install openjdk-17-jdk -y            # Ubuntu
```


## <span style="color:#802548">_MYSQL_</span>

- MYSQL DB 깔고 start시키기

```
sudo apt install mysql-server -y    # Ubuntu
sudo systemctl start mysqld
sudo systemctl enable mysqld
mysql_secure_installation
```

- mysql을 깔고 접속한다.
- 첫 비번은 root다. 
- 만약 mysql로 안되면 앞에 sudo를 붙여서 해보자.

```sh
mysql -u root -p
```

- root는 기본적으로 root에 root 비번인데, 이를 바꾸지 않으면 해킹당할 수 있다.
- 또한 dbeaver 등을 통한 host 원격 접속도 막아야 한다.
- root 권한은 오직 localhost에서만 다룰 수 있게 변경한다. 그게 바로 @localhost다.

```sql
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '새로운비밀번호';
FLUSH PRIVILEGES;
```

- 그 뒤 DB와 user를 만들어주자.
- 실제 사용할 db는 원격접속이 되어야 편하므로 %로 주어 어디서든 접속가능하게한다.
- 그래야 dbeaver에서 db에 붙을 수 있다.

```sql
CREATE DATABASE tanomuzoko;
CREATE USER 'tanomuzoko'@'%' IDENTIFIED BY 'tanomuzoko';
GRANT ALL PRIVILEGES ON tanomuzoko.* TO 'tanomuzoko'@'%';
FLUSH PRIVILEGES;
```

- 만들어졌는지 확인한다.

```sql
SELECT user, host FROM mysql.user;
```

- 그 뒤 MYSQL의 외부 접속을 허용하게 만들어야 한다.
- conf를 열어준다.

```sh
sudo vi /etc/mysql/mysql.conf.d/mysqld.cnf
```

- 열었다면 bind-address 0.0.0.0 으로 저장하자. 
- 그래야 외부접속이 가능하다. dbeaver나 spring boot 접속이 가능하다는 의미다.
- 설정을 바꿨다면 다시 restart시키자.

```sh
sudo systemctl restart mysql
```


- dbeaver에서 이제 외부접속이 가능한 지 test가 필요하다.
- 가끔 안되는 경우가 있다. 다 정확한데 안 되면 그 때는 비밀번호 플러그인의 문제일 확률이 높다.
    - 만약 plugin이 caching_sha2_password라면, 이를 변경해야한다.
    - 당연히 dbeaver에서는 접속이 안되므로 aws instance에서 mysql 직접 들어가서 아래 query를 수행해야 한다.

```sql
SELECT user, host, plugin FROM mysql.user WHERE user = 'tanomuzoko'
ALTER USER 'tanomuzoko'@'%' IDENTIFIED WITH mysql_native_password BY 'tanomuzoko';
FLUSH PRIVILEGES;
```

- dbeaver로 접속되었다면 간단한 query를 날려보자.

```sql
show tables;
show databases;
```

## <span style="color:#802548">_Spring boot_</span>

- git으로 clone받은 폴더에 gradle이 존재할 것이다.
- 받은 폴더에 실행권한을 부여해야 실행이 가능하다.
- test가 실패하는데 test가 굳이 불필요하다면 build -x test로 하자.
    - test에는 JVM options들이 들어가지 않을 수 있어 그러한 차이에서 실패하는 경우도 있다.

```sh
chmod +x gradlew         # clone 받은 폴더에서 빌드 권한 부여
./gradlew build          # 빌드 실행
./gradlew build -x test  # test 없이 빌드. 
```

- 그 외 별도로 서버가 필요한 dependecy들은 다운로드가 필요하다.

```sh
sudo apt install redis-server #redis설치
```

- 매번 java -jar로 실행시키는 것은 쉽지 않으므로 service를 등록시킨다.

```sh
sudo vi /etc/systemd/system/springboot.service
```

- 아래 같이 Unit, Service, Install로 나뉜다.
- 우리는 위의 Remote - SSH에서 user를 ubuntu로 정했으므로 ec2-user도 ubuntu로 나올 것이다.

```sh
[Unit]
Description=Spring Boot Application
After=network.target

[Service]
User=ec2-user
WorkingDirectory=/home/ec2-user/tanomuzoko
ExecStart=/usr/bin/java -jar /home/ec2-user/ec2-user/target/tanomuzoko.jar
SuccessExitStatus=143
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
```

- 처음 서버를 start할 떄는 아래처럼 해주면 된다.
- tanomuzoko는 그냥 프로젝트 폴더이름이다. 각자에 맞게 해준다.
- 새로운 서비스를 만들었기 때문에 이를 인식시키기 위해 daemon-reload가 필요하다.

```sh
# systemd 데몬 리로드 (새로운 서비스 인식)
sudo systemctl daemon-reload

# 서비스 시작
sudo systemctl start tanomuzoko

# 부팅 시 자동 실행 등록
sudo systemctl enable tanomuzoko
```


- ExecStart에 JVM option이 필요하면 아래처럼 넣어준다. 무조건 java -jar 뒤에 바로 넣어줘야 한다.
- ~~.jar 하고 뒤에 JVM option을 넣어도 인식이 안된다.

```sh
/usr/bin/java -jar -Dspring.profiles.active=dev -Djasypt.encryptor.password=zzz /home/ubuntu/tanomuzoko/build/libs/tanomuzoko_SNAPSHOT.jar
```

- 해당 부분은 service file을 바꾸는 일이기 때문에 daemon-reload가 필요하다.

```sh
sudo systemctl daemon-reload
sudo systemctl restart tanomuzoko
```

- 서비스가 running 상태인지 확인할 수 있다.

```sh
# 서비스 상태 확인
sudo systemctl status tanomuzoko
```

- 별도의 log를 말지 않았다면 아래의 방식으로 로그를 확인해야 한다.
- tanomuzoko

```sh
# 실시간 서비스 로그 확인
sudo journalctl -u tanomuzoko -f
```

## <span style="color:#802548">_DNS_</span>

- aws의 DNS 설정은 매우 단순하다.
- AWS 콘솔에서 만져주면 된다.
- 우리가 사용한 domain은 가비아이므로 가비아 기준이다.

<img src="/image/aws-instance7.png">



## <span style="color:#802548">_nginx_</span>

- 우선 nginx를 설치한다.

```sh
sudo apt install nginx -y    # Ubuntu
```

- nginx를 설치했다면 이제 conf를 써줘야 한다.

```sh
sudo vi /etc/nginx/nginx.conf
```

- nginx의 conf는 기본적으로 client_max_body_size, default_type을 확인한다.
- default client_max_body_size가 1MB라서 파일 upload를 압축하지 않고 한다면 늘려야 할 수 있다.
- default_type은 application/json이 아니다. HTTP method를 처리하지 않고 파일만 처리한다.
    - 따라서 파일처리에 범용적으로 쓰이는 binary type인 octet-stream을 기본 타입으로 한다.
- SSL 인증을 추가했기 때문에 ssl_protocols에 관한 설정도 들어가야 한다.
- reverse proxy에 관한 설정은 default.conf가 처리중이다.
    - 이를 include로 나타낸다.

```sh
user www-data;
worker_processes auto;
pid /run/nginx.pid;
error_log /var/log/nginx/error.log;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        types_hash_max_size 2048;
        client_max_body_size 100M;
        # server_tokens off;
         # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;

        ### Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;

}



```


- nginx.conf에는 실제 request의 reverse proxy에 관한 내용이 아닌, nginx 자체 설정을 넣었다.
- 이제 다른 conf 파일에 reverse proxy에 관한 내용을 넣어준다.


```sh
sudo vi /etc/nginx/sites-available/default
```

- server_name은 서버의 IP를 주거나, domain을 준다. 
    - IP는 REMOTE - SSH로 들어갈 떄 사용한 그 IP다.
    - domain은 DNS가 등록되었다는 전제하에 가능하다.
- port는 Spring boot가 사용하는 port에 맞게 수정한다.
- tanomuzoko.shop이든 www.tanomuzoko.shop이든 전부 요청을 받아올 수 있게 만들었다.

```sh
server {
    listen 80 tanomuzoko.shop www.tanomuzoko.shop;               # Listens on port 80 for HTTP traffic.
    listen [::]:tanomuzoko.shop www.tanomuzoko.shop;           # Also listens on port 80 for IPv6 addresses.

    server_name 13.111.111.14;               # IP or domain name (tanomuzoko.shop)

    # Serve JS files from /var/www/static for any URL starting with /js/
    location /js/ {
        root /var/www/static;                # JavaScript files are located in /var/www/static/js/
    }

    # Serve CSS files from /var/www/static for any URL starting with /css/
    location /css/ {
        root /var/www/static;                # CSS files are located in /var/www/static/css/
    }

    # Serve images (if any) from /var/www/static for any URL starting with /images/
    location /images/ {
        root /var/www/static;                # Images are located in /var/www/static/images/
    }

    # Redirect all other requests to the Spring Boot application
    location / {
        proxy_pass http://localhost:8080;     # Proxy requests to the Spring Boot app (default port 8080).
        proxy_set_header Host $host;          # Pass the original host header.
        proxy_set_header X-Real-IP $remote_addr; # Pass the real IP of the client.
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; # Pass the forwarded-for header.
        proxy_set_header X-Forwarded-Proto $scheme; # Pass the scheme (HTTP/HTTPS).
    }
}
```


## <span style="color:#802548">_SSL_</span>

- Let's Encrypt로 SSL을 설정해주자.

```sh
sudo apt install certbot python3-certbot-nginx -y    # Ubuntu
sudo certbot --nginx -d your-domain.com
echo "0 0 * * * root certbot renew --quiet" | sudo tee -a /etc/crontab
```


## <span style="color:#802548">_nginx에 SSL 설정_</span>
- SSL을 적용했으니 다시 conf를 써야 한다.

```sh
sudo vi /etc/nginx/sites-available/default
```

- 아래는 SSL이 설정된 이후의 default.conf다.
- 우선 static file에 관한 설정을 넣어준다.
- 해당 설정은 spring을 쓰는 입장에서 큰 의미가 없다. 
- 아래는 front build시 index.html로 말리는 front framework 용이다.

```sh
server {
    listen 80 default_server;                # Listens on port 80 for HTTP traffic.
    listen [::]:80 default_server;           # Also listens on port 80 for IPv6 addresses.
    
    root /var/www/html;                      # The root directory where your website's files are located.
    index index.html index.htm index.nginx-debian.html;  # Default index files Nginx will serve (in order).
    
    server_name _;                           # This is a catch-all for any domain name (it matches any request).
    
    location / {                             # This block handles requests for the root URL and directories.
        try_files $uri $uri/ =404;           # First, Nginx will try to serve the request as a file. 
                                             # If it's not found, it will try as a directory. 
                                             # If neither exists, it returns a 404 error.
    }
}
```

- nginx가 WAS로 보낼 떄는 HTTPS가 불필요하다.
- 그냥 HTTP로 보내면 된다. 

```sh
server {
        listen [::]:443 ssl ipv6only=on; # managed by Certbot
        listen 443 ssl; # managed by Certbot
        server_name tanomuzoko.shop www.tanomuzoko.shop; # managed by Certbot

        
        ssl_certificate /etc/letsencrypt/live/tanomuzoko.shop/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/tanomuzoko.shop/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        # Serve JS files from /var/www/static for any URL starting with /js/
        location /js/ {
            root /var/www/static;                # JavaScript files are located in /var/www/static/js/
            expires 30d;
        }

        # Serve CSS files from /var/www/static for any URL starting with /css/
        location /css/ {
            root /var/www/static;                # CSS files are located in /var/www/static/css/
            expires 30d;
        }

        # Serve images (if any) from /var/www/static for any URL starting with /images/
        location /images/ {
            root /var/www/static;                # Images are located in /var/www/static/images/
            expires 30d;
        }

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.;
                proxy_pass http://127.0.0.1:9005;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}
```

- 이제는 http로 들어온 경우는 전부 https로 redirect 시키는 설정도 추가해준다.

```sh
server {
        listen 80 ;
        listen [::]:80 ;
        server_name tanomuzoko.shop www.tanomuzoko.shop;
        return 301 https://$host$request_uri;
}
```

- 최종적으로 아래와 같은 설정이 된다.

```sh
server {
        listen 80 ;
        listen [::]:80 ;
        server_name tanomuzoko.shop www.tanomuzoko.shop;
        return 301 https://$host$request_uri;
}

server {
        listen [::]:443 ssl ipv6only=on; # managed by Certbot
        listen 443 ssl; # managed by Certbot
        server_name tanomuzoko.shop www.tanomuzoko.shop; # managed by Certbot

        
        ssl_certificate /etc/letsencrypt/live/tanomuzoko.shop/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/tanomuzoko.shop/privkey.pem; # managed by Certbot
        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
        ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

        # Serve JS files from /var/www/static for any URL starting with /js/
        location /js/ {
            root /var/www/static;                # JavaScript files are located in /var/www/static/js/
            expires 30d;
        }

        # Serve CSS files from /var/www/static for any URL starting with /css/
        location /css/ {
            root /var/www/static;                # CSS files are located in /var/www/static/css/
            expires 30d;
        }

        # Serve images (if any) from /var/www/static for any URL starting with /images/
        location /images/ {
            root /var/www/static;                # Images are located in /var/www/static/images/
            expires 30d;
        }

        location / {
                # First attempt to serve request as file, then
                # as directory, then fall back to displaying a 404.;
                proxy_pass http://127.0.0.1:9005;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
}

```

- syntax가 맞는지 확인한다.

- 맞다면 설정파일 적용을 위해 nginx를 재시작시킨다.
- 시작한 적이 없으면 start다.

```sh
sudo nginx -t  # 설정 점검
sudo systemctl restart nginx  # nginx 재시작
```


## <span style="color:#802548">_nginx에서 static처리와 build 연동_</span>

- 하지만 nginx에서 바라보게 해도, pull을 받았다고 저절로 nginx의 위치에 static file들이 들어가지 않는다.
- 수동으로 일일이 옮겨줘야 한다.
- 핵심은 아래 내용이다. 저걸 실행해야 pull로 받아온 static file들이 복사된다.

```sh
rsync -av --delete src/main/resources/static/ /var/www/static/
```

- 다 치기 귀찮다면, 수고로움을 덜기 위해 아래와 같이 bash script를 만들어준다.
```sh
vim deploy.sh
chmod +x deploy.sh
./deploy.sh
```

- 아래와 같은 내용이 들어가면 된다.

```sh
#!/bin/bash

# Navigate to your project directory
cd /home/user/my-spring-boot-app

# Pull the latest code
git pull origin main

# Build the project (if needed)
sudo ./gradlew build -x test

# Copy static files to Nginx's directory
sudo rsync -av --delete src/main/resources/static/ /var/www/static/

# Restart your Spring Boot app (if needed)
sudo systemctl restart my-spring-app

echo "Deployment completed!"
```

- 만약 무언가 JVM option이 추가된 경우, sudo vi /etc/systemd/system/springboot.service로 service를 바꾸게 된다.
- 그럴 때는 service가 바뀐 setting을 인지시키기 위해서 아래 명령어가 추가로 요구된다.

```sh
sudo systemctl daemon-reload
```


## <span style="color:#802548">_CI_CD 설정_</span>

<details>
<summary>content</summary>
<div markdown="1">

- main branch에서 push와 PR이 진행될 때 trigger된다.
- AWS_HOST와 AWS_USER, SSH_PRIVATE_KEY는 모두 AWS에 접속하기 위해 필요한 정보다.
- 해당 정보를 그냥 쓰면 안 된다. public이라서 정보가 다 노출된다. 
- repositry setting - > Security tab의 Secrets and variables - > actions -> repository secrets에 추가
- SCP로 jar를 만들어서 하는 방법도 있지만, git 연동을 AWS에서 완성했다면, 그럴 필요가 없다.
- 직접 안에서 다 수행하면 된다.
    - AWS로 접속할 임식 private key 파일을 만든다. 
    - git action을 실행하는 local의 ssh known host에 AWS_HOST를 등록한다.
    - SSH접속으로 AWS에 들어간다.
    - git pull을 받는다.
    - bulld를 진행한다.
    - spring boot를 stop한다.
    - spring boot를 restart시킨다.

```sh
name: tanomuzoko-build

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      contents: read

    steps:
    - name: SSH into AWS Instance, Build and Restart Service
      env:
        AWS_HOST: ${{ secrets.AWS_HOST }}
        AWS_USER: ${{ secrets.AWS_USER }}
        SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
      run: |
        echo "$SSH_PRIVATE_KEY" > private_key.pem
        chmod 600 private_key.pem
        mkdir -p ~/.ssh
        chmod 700 ~/.ssh
        # Add the AWS EC2 instance's public key to known_hosts securely
        ssh-keyscan -H $AWS_HOST >> ~/.ssh/known_hosts
        # SSH into the EC2 instance and run the build process directly
        ssh -i private_key.pem $AWS_USER@$AWS_HOST << 'EOF'
          # Navigate to the project directory
          cd ~/tanomuzoko/
          # Pull the latest code if needed (e.g., from GitHub)
          git pull origin main
          # Build the project using Gradle (assuming Gradle is installed or using the wrapper)
          ./gradlew build -x test
          # Stop the Spring Boot service
          sudo systemctl stop tanomuzoko.service
          # Restart the service (no need to copy the JAR, it will already be built in place)
          sudo systemctl restart tanomuzoko.service
        EOF
        
        # Clean up the private key
        rm private_key.pem
```

</div>
</details>




