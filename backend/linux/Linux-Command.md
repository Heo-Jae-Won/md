## <span style="color:#802548">_1. 파일의 내용 들춰보기_</span>
```sh
cat [파일명] 
## 전체 파일내용을 출력한다.
```
​
```sh
cat [파일명] | tail -n 20 : 
##하위 20줄의 파일 내용만 출력한다. |는 파이프라고 하는데, 다른 명령어를 연이어 쓸 수 있다. 따라서 cat [파일명]을 한 뒤 그 결과물에 대해 tail -n 20 명령어를 실행한 것이다.
```

```sh
more : 
##파일을 한 페이지씩 맨위에서부터 보여준다. 

                  v : 
                  ##편집모드로 넘어가서 볼 수 있다. 

                  spacebar: 
                  ##다음 페이지로 넘어간다

                 b: 
                 ##이전 페이지로 돌아간

                  f: 
                  ##2페이지를 넘어간다

                  g:  
                  ##페이지의 맨 처음 줄이 아니라  파일의 맨처음 줄로 이동.

                 shift g :
                 ## 페이지의 맨 마지막 줄이 아니라 파일의 맨 마지막 줄로 이동
```

​
```sh
less 
##more와 동일하지만 맨아래에서부터 보여준다.
```

​
```sh
tail -f /var/log/tomcat/logback.log | grep proxy : 
##실시간으로 변하는 로그를 출력하는데, 그중 문자열이 proxy인 문장만 출력한다.
```


​
```sh
ctrl c 
##프로세스 종료. 다만 쉘 스크립트로 가동되는 프로그램은 정지시키지 못하는 것으로 보인다. 
##혹시 tail -f 프로세스를 종료할 때 ctrl c가 아니라 ctrl z를 누른다면 일시정지가 되는데, 이것은 프로세스가 종료된 상태가 아니다.  
##따라서 종료를 원한다면 fg를 눌러 다시 불러오고 ctrl c를 누르면 된다. 
##일시정지 시킨 프로세스는 jobs에 뜨지만, 일시정지가 아니라 background에서나 foreground에서 돌아가고 있는 process는 jobs 명령어에 뜨지 않는다.
```

​


​
## <span style="color:#802548">_2. 파일 생성-복제-이동_</span>

```sh
touch [파일명]
## 파일 생성
```
​
```sh
rm -r [디렉토리명]
##디렉토리 삭제. 
```

​

- 예를 들어 아래와 같이 했다고 해보자. 
```sh
rm -r .git
## 그럼 .git 디렉토리가 삭제된다. 
##참고로 .git 폴더는 숨겨진 디렉토리라서 ls -a로 보아야 보인다.
```

```sh
rm [파일명]
##파일 삭제
```
​
```sh
cp [source파일] [target파일] : 
##source파일의 내용을 복제해 target이라는 이름으로 만듦.
```
​
```sh
cp -r [source디렉토리] [target디렉토리]: 
##source폴더의 내용을 하위까지 복제해 target디렉토리 아래 만든다.
```
​
```sh
cp -r /root/.jenkins/workspace/vue/dist  /var/www/html/vue
## 예를 들어 위와 같이 했다고 해보자. 
## 아래는 vue nginx 배포 예시다. dist폴더 안의 index.html을 참조한다.
## 그럼  ~~~/vue 폴더 아래 dist 폴더가 복제된다.
```


```sh
mv [source파일] [target파일] 
## source파일의 이름이 target파일로 바뀐다.
```

​

- 예를 들어 아래와 같이 했다고 해보자.
```sh
touch a.txt
mv a.txt b.txt
```
 - a.txt가 생성된 뒤 b.txt로 이름이 바뀌었다. 
​
```sh
ㅡ mv [source파일] [target디렉토리]: 
## 해당 파일을 target 폴더로 옮기게 된다. 
```

​

- 예를 들어 아래와 같이 했다고 해보자. 아래는 war파일 배포의 예시다.
```sh
mv /root/.jenkins/workspace/pipelinedeploy/target/dashboard-1.0.0-BUILD-SNAPSHOT.war
/usr/local/apache-tomcat-8.5.85/webapps
```
- 그럼 해당 경로의 war파일은 target폴더로 이동된다.

```sh
ln -s [source파일] [target디렉토리]: 
## source 파일의 내용을 target에 바로가기로 만든다. 
```

​

- 예를 들어 아래와 같이 했다고 해보자. 아래는 nginx 설정파일 예시다.
```sh
ln -s /etc/nginx/sites-available/dashboard.conf /etc/nginx/sites-enabled
```
- 그럼 아래와 같이 symboilc link(바로가기)가 만들어져있다. 원본(source)가 어딨는지 참조표시가 존재한다.

```sh
chmod -R +x [파일명] :
## 실행권한 부여
```
​

## <span style="color:#802548">_3. 프로세스_</span>
```sh
ㅡ ps -ef | grep [검색어]
## 해당 검색어가 포함된 프로세스가 돌아가고 있다면 출력해준다. jar 파일 배포는 java로 출력했다. 
```

```sh
kill -9 [프로세스아이디] :
## 실행 중인 프로세스를 강제종료한다. 강제종료라서 종료하는 로그가 남지 않는다.
```

​
```sh
ㅡ kill -15 [프로세스아이디] :
## 실행 중인 프로세스를 안전하게 종료한다. 종료하는 로그가 남는다.
```


## <span style="color:#802548">_4. 검색_</span>
```sh
find / -name [파일명] :
## 해당 파일명이 들어있는 path를 보여준다.
find /etc/home -name jenkins 
#/etc/home 아래에 jenkins라는 파일이 존재하는지 확인하여 경로를 알려준다.
```
```sh
ㅡ which [명령어] :
## 해당 명령어가 규정된 파일의 위치를 보여준다. 다만 vim으로 열어봤자 binary라서 볼 수 있는 것이 없다. 
```


​

## <span style="color:#802548">_5. 이동_</span>
```sh
cd /etc/bind/named.conf : 
## dns 설정으로 이동
```
```sh
cd /etc/nginx/sited-avaiable : 
## nginx 설정으로 이동

ㅡ 편집기로 열었을 시 :

                                     end : 
                                     ## 해당 줄 맨 끝으로 이동  

                                     home : 
                                     ## 해당 줄 맨 처음으로 이동 

                                     ctrl 왼 오른 : 
                                     ## 개행(\n)을 기준으로 다음 문자로 이동

                                     shift g : 
                                     ## 문서 맨 끝으로 이동

                                     gg : 
                                     ## 문서 맨 처음으로 이동

                                     /[검색어] : 
                                     ## 해당 키워드를 검색. n을 누르면 검색된 단어의 다음 줄로 이동

                                     i : 
                                     ## 문서를 쓸 수 있는 상태가 된다. 더이상 쓰지 않으려면 esc를 클릭

                                    :wq : 
                                    ## esc상태에서 여태 기록했던 내용이 저장

                                    :q! : 
                                    ## esc상태에서 여태 기록했던 내용을 원복시키고 저장

                                     page down, up : 
                                     ## 페이지를 이동

                                    n% : 
                                    ##n% 만큼 출력한 페이지로 이동

                                    대문자 m : 
                                    ## 페이지의 중간으로 이동
```
​

## <span style="color:#802548">_6. 설치_</span>
```sh
apt-get update : 
## 패키지 최신화. 참고로 ubuntu라서 apt-get이고 red-hat이라면 yum을 쓴다.
```
```sh
apt-get install [package] : 
## 해당 패키지를 apt라는 package manager를 통해 설치
```

​

- 예를 들어 아래와 같이 쓸 수 있다.
```sh
apt-get install vim
apt-get install openjdk-11-jdk
 ```

```sh
wget [도메인] : 
## 해당 도메인주소를 통해 파일을 설치. 보통 package인 jar 파일을 설치했다.
```

​

- 예를 들어 아래와 같이 쓸 수 있다. cfr은 decompiler다. 
```sh
wget http://www.benf.org/other/cfr/cfr-0.152.jar
# 웹 상의 파일 다운로드
curl -0 https://www.example.com/index.html 
# 웹 상의 파일 다운로드
```

```sh
tar -xvzf apache-tomcat-8.5.85.tar.gz : 
# tar 압축을 푼다. tar 파일을 wget으로 받게되면 해당 명령어를 사용해 압축을 해제한다.
```

## <span style="color:#802548">_7. 네트워크_</span>
```sh
ping -c 5 [domain or IP] : 
## 해당 domain이 dns서버에 등록되어 있는지 확인한다.
```
- 만약 없다면 아래와 같이 name resolution fail오류가 뜬다. 
- ping이 안된다고 늘 접속 문제는 아니다. 보안을 이유로 ICMP 패킷에 응답하지 않는 서버도 존재한다.

​
```sh
dig [domain] : 
## ping과 마찬가지로 해당 domain이 있는지 dns서버에서 확인한다.
```
- 예를 들어 아래와 같은 명령어를 입력했다고 해보자.
```sh
dig google.com
```
- 그럼 아래와 같이 길다란 정보가 뜬다. 
- 거기서 옵션으로 +noall +answer로 주면 아래와 같이 짧게 나타낼 수 있다. 

```sh
netstat  : 
## 현재 사용중인 포트가 있는지 확인한다
```
​

-  예를들어 이와 같은 방식을 사용할 수 있다. 옵션을 붙이지 않으면 너무 많은 정보가 나온다. 
```sh
netstat -tnlp
```
- 아래와 같이 줄여서 정보가 나온다.
- 참고로 telnet은 보안 상 문제가 있어 쓰지 않는 것을 권장한다고 한다. 

```sh
ifconfig 
# 네트워크 ip주소 확인
```
​


## <span style="color:#802548">_8. 명령어 생성_</span>
- 사람마다 다르겠지만 나는 명령어를 .bashrc에 만들었다. 
```sh
vim ~/.bashrc
```
- 명령어 생성

```sh
source ~/.bashrc
```
- 아래와 같이 명령어를 alias로 규정한다.
- 그리고나서 한번 source로 reload해준다. 그럼 명령어를 사용할 수 있다.
- 참고로 alias에 관해 설명하자면 nohup은 리눅스 서버가 꺼지지 않는이상 계속 프로그램이 돌아간다. 
- 맨 끝에 &를 붙이면 프로세스가 백그라운드에서 돌아간다. 

​
```sh
1> /dev/null
## 위 옵션을 붙이게 되면 출력이 모두 버려진다. 숫자가 없을 경우 default는 1이다. 1이 표준 출력이다. 나는 숫자를 쓰지 않았다. 저렇게 하면 표준 출력 오류는 버려지지않고 그대로 뜬다.
```

​
```sh
2> /dev/null로 붙이게 되면
## 하면 표준 출력 오류가 모두 버려진다. 저렇게 하면 표준 출력은 그대로 뜬다. 이건 사실상 쓸 일이 없다.
```

​
```sh
> /dev/null 2>&1
## 하면 표준 출력과 표준 오류 모두 버려진다. 오류까지 버릴 필요는 없어보인다. 그래서 나는 > /dev/null로 썼다. 
```

​

- 또 참고로 설명하자면 --는 두 개이상 글자 옵션이며 -는 각자 모두 한 글자씩의 옵션이다.
-  --help는 help가 하나의 옵션이고
- ls -aGlr은 -a, -G, -l, -r 4개가 모인 옵션이다.

​

## <span style="color:#802548">_9. 컴파일_</span>
​
```sh
java -jar cfr-0.152.jar UserRestController.class > UserRestController.java  : 
## decompile하기
```
```sh
javac -cp C:\Users\user\Downloads\apache-tomcat-9.0.71\lib\servlet-api.jar Nana.java  : 
## compile하기
```

​
```sh
jar -cvf myclass.jar BoardDAO.class BoardDTO.class  : 
## class 한 데 합쳐 jar파일로 변환하기. 주로 maven dependency가 이 형태로 만들어진다.
```
```sh
jar -xvf myclass.jar : 
## jar 압축풀어 class 파일로 나누기
```

## <span style="color:#802548">_10. 압축관련_</span>
```sh
zcat rgb.txt.gz 
#gz압축풀지 않고 안의 내용물만 출력
xzcat rgb.txt.gz 
#tz압축풀지 않고 안의 내용물만 출력
tar xzf jenkins.tar.gz 
#gz압축을 풀기
```

## <span style="color:#802548">_11. 표준입출력_</span>
```sh
ps > ps.txt 
# ps 명령어의 결과를 ps.txt라는 파일로 전환. 만약에 ps.txt가 존재한다면 덮어쓰고 없다면 새로 생성
ps >> ps.txt 
# ps 명령어의 결과를 ps.txt라는 파일로 전환. ps.txt가 존재한다면 그 끝에 append 없다면 새로 생성
ls 2> errfile.txt 
#표준 에러 출력을 errfile.txt 파일에 출력합니다. 에러가 없다면 errfile에는 내용이 없다.
ls 2>&1 > normalfile.txt 
# 표준 출력을 normalfile.txt에 저장하고, 그 뒤에 오류도 표준 출력에 포함하여 터미널에 나타낸다. 즉 일단 표준 출력만 file에 저장된다.
ls > allfile.txt 2>&1 #표준 에러출력과 표준 출력을 모두 file에 저장합니다.
ls >& file #표준 에러출력과 표준 출력을 모두 file에 저장합니다.
ls 2> /dev/null #에러를 출력하지 않습니다.
```

## <span style="color:#802548">_12. 변수확인_</span>
```sh
declare 
# 유효한 쉘 변수 표시
printenv 
# 유효한 환경 변수 표시
VARNAME=value123 
# 변수명 = 값/ 쉘 변수 설정 띄어쓰기 없이 다 붙여써야함.
unset VARNAME 
# 쉘 변수 해제
export name=value 
# 환경변수 설정
source ~/.bashrc 
# 셸 재시작 없이 설정파일 변경 반영
```

## <span style="color:#802548">_13. 서버 접속_</span>
```sh
ssh rrr@111.11.231.11 -p 1110 
# 해당 서버에 해당 계정으로 접속한다.
sftp id@[ip or domain]  
# 파일을 전송하기 위해 해당 서버 접속
mget file 
# 내 서버로 파일을 가져온다. sftp 접속 이후 process에서 사용 가능한 command
mput file 
# 해당 서버로 파일을 보낸다. sftp 접속 이후 process에서 사용 가능한 command
scp root@111.223.44.11:~/file 
# 해당 서버로 파일을 암호화하여 전송. sftp와 별개
```


## <span style="color:#802548">_14. 로그보기_</span>
```sh
alias 
# 로그 관련 tail 명령어가 분명히 있을 것이다. 거기서 경로를 알아낸다.

cd 경로
# 해당 경로로 이동한다. .log파일 이전까지만 이동

cat ~~.log | grep 'keyword' -n
# 검색하려는 키워드를 보여주고, 줄수를 보여준다.

vi ~~~.log
# 해당 로그파일을 수정모드로 연다. 실제 수정은 하지 않을 것이다.


:[해당줄수] 
# 보통 내가 실행한 api 시간을 보고 그에 맞는 줄수를 검색한다. 

pageup/pagedown
# 거기서부터 페이지를 올리고 내리면서 계속 로그를 본다.
```
