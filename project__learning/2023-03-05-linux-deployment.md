## <span style="color:#802548">_배포를 위한 도메인 설정_</span>
- 우선 도메인 설정을 하려면 당연히 먼저 도메인이 있어야 한다.
- 도메인을 구매했다면, linux server에 해당 도메인을 받게해 줄 설정파일이 필요하다. 이를 zone file이라고 한다.
- zone 파일들은 bind9을 통해 구현했다. db파일을 가져오게 한다. zone 파일 안의 string이 domain name이다.

```sh
# /etc/bind/named.conf
zone "dev.jaewon.net" IN {
  type master;
  file "/etc/bind/dev.jaewon.net.db"
};
```

- domain database(db)파일도 같이 /etc/bind 경로 아래 있다.
- @는 domain이다. NS는 name server인데, name server가 도메인서버와 동일하다는 의미다.
- 그리고 그 아래 바로 붙는 A는 도메인의 IP 주소를 나타낸다.
- www는 서브도메인이며 도메인의 IP를 따라간다는 의미다.

```sh
# /etc/bind/dev.jaewon.net
$TTL  3H
@     IN      SOA     root.     ( 2 1D 1H 1W 1H)

@     IN      NS      @
      IN      A       119.148.53.13

www   IN      A       119.148.53.13
```

- 아래는 chatGPT가 짜준 dns db 파일이다. 차이점은 아래와 같다.


- TTL 설정: 첫 번째 파일은 3시간, 두 번째 파일은 7일입니다.
- SOA 레코드의 관리자: 첫 번째 파일은 root., 두 번째 파일은 admin@example.com.으로 설정되어 있습니다.
- 네임 서버: 첫 번째 파일은 도메인 자체를 네임 서버로, 두 번째 파일은 ns1.example.com.을 네임 서버로 지정합니다.
- A 레코드: 첫 번째 파일은 도메인과 www 서브도메인을 119.148.53.13로, 두 번째 파일은 도메인을 192.0.2.1로, 네임 서버를 192.0.2.2로 설정합니다.

```sh
$TTL    604800
@       IN      SOA     ns1.example.com. admin.example.com. (
                              2         ; Serial
                         604800         ; Refresh
                          86400         ; Retry
                        2419200         ; Expire
                         604800 )       ; Negative Cache TTL
;
@       IN      NS      ns1.example.com.
@       IN      A       192.0.2.1
ns1     IN      A       192.0.2.2 
```

- 만들었다면 아래 명령어로 제대로 설정됐는지 확인해보자.

```sh
named-checkconf /etc/bind/named.conf
```
- 제대로 등록됐는지 확인하기 위해 command를 치면 reponse가 온다.
- non-authroitive answer라고 올탠데 별 문제 없다. 개인 플젝용이라서..

```sh
nslookup dev.jaewon.net
```

- response가 안 오고 아래 오류 메시지가 오면 뭔가 잘못됐다는 의미다.
  -  ** server can't find example.com: NXDOMAIN
  - ** server can't find example.com: SERVFAIL
  - ** server can't find example.com: REFUSED
  - ;; connection timed out; no servers could be reached


## <span style="color:#802548">_reverse proxy nginx 설정_</span>
- WAS만이 아니라 WEB SERVER도 만들어주자.
- 보통 nginx를 많이 사용한다. apahce가 돌아가고 있다면 apahce와 nginx가 서비스 port가 겹치기 때문에 stop시킨다.
- 파일은 아래와 같이 만들어준다.

```sh
apt-get nginx install
service apache2 stop
service nginx start
```

```sh
# /etc/nginx/sites-available/dashboard.conf
server {
listen 80;
listen [::]:80;
server_name dev.company.net;


location / {
  proxy_pass http://dev.company.net:[포워딩한 WAS port];
  proxy_set_header X-Real-IP $remote_addr;
  proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  proxy_set_header Host $http_host;
}

}

```

- /로 설정하면 모든 request가 WAS에 도달하므로 별로 좋지 않다.
- 따라서 WAS에 보낼 것과 web server에서 return할 것을 구분한다.

```sh
server {
listen 80;
listen [::]:80;
server_name dev.company.net;


  location / {
    proxy_pass http://dev.company.net:[포워딩한 WAS port];
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
  }

  location /upload {
    root /usr/share/nginx;
  }

  location /resources {
    root  /user/share/nginx;
  }

}
```

- nginx 파일을 만들었다면 심볼릭 링크 파일도 만들어준다.
- 기존의 default.conf symbolic link파일은 지워줘야 한다. default가 먼저 적용되기 때문이다. 

```sh
ln -s /etc/nginx/sites-available/dashboard.conf /etc/nginx/sites-enabled
cd /etc/nginx/sites-enabled && rm default
```

- 이제 배포를 하려면 legacy의 경우는 war 파일을 말아야 한다. boot는 jar파일로 말아야 한다.
- 말았던 jar,war를 서버에 파일질라로 전송하여 보낸다.
- tomcat 경로에 올려두고 start 시키면 된다.
- 다만 background에서 start시켜야만 하기에 &를 붙인다.
- 또한 터미널의 세션이 종료되어도 계속 실행되어야 하므로 nohup도 붙인다.

```sh
java -jar ~~~.jar nohup &
```

- 다만 리소스를 올려도 파일에 권한이 없으면 404 오류가 뜬다.
- 권한이 없는 경우에는 권한을 추가해줘야 한다.
- 또는 갑사 개발서버에서 특정 확장자를 거부하는 경우도 있다.
  - 신한카드에서 폰트에서 otf, 바이너리에서 bin파일을 거부하는 경우가 있었다.
  - 이 경우 ttf나 json으로 바꿔서 진행하자.


## <span style="color:#802548">_legacy 배포시 UTF-8 깨짐 문제 pom.xml에 maven옵션을 주어 해결_</span>

- 그게 아니면 build 자체에서 utf-8을 주어야 하는 경우도 있다. 
- jenkins에서 build할 때 기본 default 인코딩이 주어져있지 않은 경우 ANSI로 설정하게 되는데 messageconverter를 거치지 않는 Exception의 경우, default charset이 맞지 않아 한글이 깨지는 경우가 생길 수 있다. 
- legacy에서는 아래 설정이 필요했는데 boot에서는 아래 설정없이도 자동으로 utf-8 build가 이뤄지는 것으로 보인다. 이유는 잘 모르겠다. 
```xml
<project.build.sourceEncoding>utf-8</project.build.sourceEncoding>
```

## <span style="color:#802548">_jenkins github 연동_</span>
- 대부분의 회사 repo는 private이다. jenkins에서 private repo는 ssh 배포로 진행된다. 따라서 개인키 공개키를 설정해야 한다.

```sh
ssh-keygen -t rsa -b 4096 -C "email"
```

- 만들었으면 .pub이란 공개키를 github에 넣어준다.
- github setting의 SSH and GPG keys에 넣는다.
- private key는 jenkins의 Manage Credentials에 넣는다.
- 그 뒤 linux 서버에서 jenkins를 known hosts에 등록해줘야 한다.
- url 같은 게 노출되지 않게끔 jenkins에 환경변수로 만드는 것도 좋다.

```sh
git ls-remote -h -- git@~~~~.git HEAD
```


- 아래는 spring legacy용 배포 스크립트다.
- legacy 때와 달리 알아서 죽여주는 script가 없으므로 대충 만들었다.

```sh
# shutdown.sh
PID=`ps -ef|grep java | awk 'NR==2 {print $2}'`

echo "Running PID: {$PID}"

if [ $PID ]
     then
     kill -9 $PID
fi
```

- 실제로는 boot 서버를 가동할 때 processID를 적어놓은 파일을 만들어준다.
- 아래는 application.yaml에 적을 경로다.

```sh
spring:
  pid:
    file: /home/ubuntu/test-app/test-app.pid # PID 파일 생성 경로 지정
```

- 실제로 저 경로에 processId를 가져오게 하는 역할은 ApplicationPidFileWriter가 담당한다.

```java
public static void main(String[] args) {
    SpringApplication app = new SpringApplication(Application.class);
    app.addListeners(new ApplicationPidFileWriter()); // ApplicationPidFileWriter 설정
    app.run(args);
}
```

- processId를 적는 것을 만들었다면, 이제 인식하고 tomcat을 종료시킨 뒤, 해당 파일을 삭제하는 sh를 만들자.
- 

```sh
#!/bin/bash
# shutdown.sh

# PID 파일 경로
PID_FILE=/home/ubuntu/test-app/test-app.pid

if [ -f "$PID_FILE" ]; then
    PID=$(cat "$PID_FILE")
    echo "Killing process ID $PID"
    kill -9 $PID
    echo "Process $PID has been terminated."
    rm -f "$PID_FILE"
else
    echo "PID file not found: $PID_FILE"
fi
```

- 아래는 pipeline 합본이다.


```sh
pipeline {
    agent any
    
    options {
      skipDefaultCheckout(true)
    }
    
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven"
        jdk "java-1.11.0-openjdk-amd64"
    }
    
    stages {
        
         stage('checkout') {
            steps {

                    git branch:'master', url: '${boot_git_url}', credentialsId: '자기 거'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn -Dmaven.test.failure.ignore=true clean install"

                // To run Maven on a Windows agent, use
                // bat "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }

        stage('Shutdown') {
            steps {
                sh 'echo "Tomcat shutdown"'
                
                script {
                    try {
                        sh "/usr/local/boot/shutdown.sh"
                    } catch(error) {
                        sh 'echo tomcat stop Fail!!'
                        print(error)
                        env.cloneResult=false
                        currentBuild.result='FAILURE'
                    }
                }
            }
        }
        
        stage('deploy'){
            steps{
                sh "/usr/local/boot/startup.sh"
            }
        }
    
    }

}
```


- front와 back의 origin이 다른 경우에는 둘 다 빌드가 필요하다.
- 아래는 front 서버를 띄우는 jenkins pipeline script다.

```sh
server{
        root /var/www/[내가만든디렉토리]/dist;

        server_name dev.jaewon.net;

        index index.html;

        location /robots.txt{
                return 200 "User-agent:*\nDisallow: /";
        }

        location / {
                try_files $uri $uri/ /index.html;
        }
}

```

## <span style="color:#802548">_jenkins gitlab 연동_</span>

- gitlab의 경우, 신원을 인증하는 ssh 용도와 별개로, repo를 읽을 권한을 허락하는 token이 따로 있다.
- 따라서 두개 모두 등록이 필요하다. gitlab을 이용하려면 우선 jenkins에서 gitlab plugin을 설치한다.
- gitlab url은 https://gitlab.com으로 지정한다.
- credentials는 gitlab에서 따로 만들어야 한다. scope를 api로 한다.

- jenkins에서 boot를 배포한다면, jar를 실행시킬 때 젠킨스가 죽어버린다.
- 이를 방지하기 위해 JENKINS_NODE_COOKIE=dontKillMe 옵션을 사용해야 한다.


```sh
pipeline {
    agent any
    
    options {
      skipDefaultCheckout(true)
    }
    
    tools {
        // Install the Maven version configured as "M3" and add it to the path.
        maven "maven"
        jdk "java-1.11.0-openjdk-amd64"
    }
    
    stages {
        
         stage('checkout') {
              steps {
                script {
                    try {
                        git branch: 'dev', 
                            credentialsId: '${credential}',
                            url: '${srt_gitlab_backend_url}'
                        env.cloneResult=true
                        
                    } catch (error) {
                        print(error)
                        env.cloneResult=false
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
        
        stage('Build') {
            when {
                expression {
                    return env.cloneResult ==~ /(?i)(Y|YES|T|TRUE|ON|RUN)/
                }                
            }
            steps{
                script{
                    try {
                        sh ' mvn -Dmaven.test.failure.ignore=true clean install'
                        env.mavenBuildResult=true
                    } catch (error) {
                        print(error)
                        echo 'Remove Deploy Files'
                        env.mavenBuildResult=false
                        currentBuild.result = 'FAILURE'
                    }
                }
            }
        }
      
      stage('Shutdown') {
           when {
                expression {
                    return env.mavenBuildResult ==~ /(?i)(Y|YES|T|TRUE|ON|RUN)/
                }
            }
            steps {
                sh 'echo "Tomcat shutdown"'
                
                script {
                    try {
                        sh "/usr/local/boot/shutdown.sh"
                         env.shutdownResult=true
                    } catch(error) {
                        sh 'echo tomcat stop Fail!!'
                        print(error)
                        env.shutdownResult=false
                        currentBuild.result='FAILURE'
                    }
                }
            }
        }
     
     
     
        stage('deploy'){
            when {
                expression {
                    return env.shutdownResult ==~ /(?i)(Y|YES|T|TRUE|ON|RUN)/
                }
            }
            steps{
                sh 'echo Tomcat start'
                
                script {
                    try {
                        sh "JENKINS_NODE_COOKIE=dontKillMe && /usr/local/boot/srt-reservation.sh"
                        env.startReulst=true
                    } catch(error) {
                        sh 'echo tomcat start Fail!!'
                        print(error)
                        env.startReulst=false
                        currentBuild.result='FAILURE'
                    }
                }
            }
        }
    
    }

}
```



​

