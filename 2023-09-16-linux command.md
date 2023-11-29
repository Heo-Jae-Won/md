```shell
ls -lt #최근 갱신된 파일을 상세하게
``` 

```shell
mv file1 file2 #file1 파일을 file2파일로 이름 변경
mv dir1 dir2 #dir1 폴더를 dir2폴더로 이름 변경
mv dir1/file1 dir2/file2 #dir1 폴더의 file1 파일을 dir2폴더로 이동. 이름은 file2로 변경
mv dir1/file1 ./file2 #dir1 폴더의 file1 파일을 현재폴더로 이동. 이름은 file2로 변경
mv -i #덮어쓰기 주의 표시를 고지
```

```shell
cp file1 dir1 dir2 #file1 파일을 dir1폴더에서 dir2폴더로 복사 cp file from to
cp dir1/file1 dir2/file2 #dir1 폴더의 file1 파일을 dir2폴더로 복사. 이름은 file2로 변경
cp dir1/file1 ./file2 #dir1 폴더의 file1 파일을 현재폴더로 복사. 이름은 file2로 변경
cp -r dir1 dir2 #dir1폴더를 dir2폴더에 복사. 이름은 유지
cp -i #덮어쓰기 주의 표시를 고지
```


```shell
rm drink rum #drink와 rum 파일을 삭제
rm -rf dir1 #폴더나 파일을 강제로 삭제
rm -i #삭제하기 전 고지
```

```shell
cd ~/malt #홈디렉토리를 뜻하는 ~를 붙여서 홈디렉토리 아래 어느 폴더로 이동
```

```shell
clear #터미널 내용 지우기
```

```shell
프로그램실행명령어 & #&를 붙이면 background에서 실행하게 된다.
bg #foreground에서 구동되는 프로그램을 백그라운드로 전환한다.
ctrl z # 실행중 프로그램 일시정지. 프로그램을 정지시키고 다른 명령어로 작업을 한다.
fg # 일시정지된 프로그램을 다시 불러온다. 셸이 대기상태가 되면서 입력불가상태가 된다.
ctrl c #프로그램을 강제 종료시킨다.
jobs #백그라운드에서 돌아가는 모든 작업을 표시한다. 프로세스는 리눅스 단위고, 작업은 셸 단위지만 똑같은 프로그램이다.
bg %1 #백그라운드 작업번호 1에 있는 프로그램을 다시 구동시킨다. 일시정지되어있는 것을 백그라운드로 실행시키고자 할 떄 필요하다.
```

```shell
ps #jobs는 해당 shell 터미널에서만 알 수 있고, 해당 리눅스 port를 통틀어 일관된 ID는 processId다.
ps -ef  #현재 shell terminal이 아니라 전체 시스템에서 돌아가는 process를  detail하게 알려준다.
ps -ef | grep "jenkins" #ps -ef인데 대상은 jenkins로 한정한다.
ps -ef |
```

```shell
kill -STOP #정지
kill -KILL #강제종료
kill -TERM #종료
```

- 프로세스: 실행중인 프로그램
- 프로그램을 두 번 눌러 두 개가 실행되면 프로세스가 2개가 되는 것.
- 리눅스는 여러 프로세스를 한꺼번에 돌림. 동시 실행되는 것은 아님. 매우 빠르게 프로세스(task) 간 전환이 일어나는 것.
- 그 전환의 순서를 scheduling이라고 함.


- 병렬처리하면 빠르게 처리되므로 CPU를 여러개 탑재.
- 하지만 병렬처리 때문에 매번 프로세스를 새로 만들면 프로세스 사이에 자원 공유가 불가능
- 따라서 스레드 단위로 자원을 공유하여 처리.

- 늘 동작해야만 프로세스도 존재함. 익러한 것들은 daemon(데몬)이라고 부름

```shell
crontab ~/.crontab #스케줄링 파일을 등록. cron regex에 맞게 작성되어야 함.
```

```shell
uname #OS 정보를 표시
```

```shell
zcat rgb.txt.gz #gz압축풀지 않고 안의 내용물만 출력
xzcat rgb.txt.gz #tz압축풀지 않고 안의 내용물만 출력
tar xzf jenkins.tar.gz #gz압축을 풀기
```

```shell
ln -s states america #바로가기 만들기. 심볼릭링크 형성
```

```shell
find /etc/home -name jenkins #/etc/home 아래에 jenkins라는 파일이 존재하는지 확인하여 경로를 알려준다.
```

- 표준 입력: 키보드로 입력
- 표준 출력: 터미널에 출력. 파일 디스크립터 번호는 1번
- 표준 에러 출력: 터미널에 출력. 파일 디스크립터 번호는 2번


```shell
ps > ps.txt # ps 명령어의 결과를 ps.txt라는 파일로 전환. 만약에 ps.txt가 존재한다면 덮어쓰고 없다면 새로 생성
ps >> ps.txt # ps 명령어의 결과를 ps.txt라는 파일로 전환. ps.txt가 존재한다면 그 끝에 append 없다면 새로 생성
ls 2> errfile.txt #표준 에러 출력을 errfile.txt 파일에 출력합니다. 에러가 없다면 errfile에는 내용이 없다.
ls 2>&1 > normalfile.txt #표준 출력을 normalfile.txt에 저장하고, 그 뒤에 오류도 표준 출력에 포함하여 터미널에 나타낸다. 즉 일단 표준 출력만 file에 저장된다.
ls > allfile.txt 2>&1 #표준 에러출력과 표준 출력을 모두 file에 저장합니다.
ls >& file #표준 에러출력과 표준 출력을 모두 file에 저장합니다.
ls 2> /dev/null #에러를 출력하지 않습니다.
```

- 보통 redirect를 하면 화면에는 출력 결과가 나타나지 않는다.
- 따라서 화면과 파일 양쪽에서 보고 싶다면 다른 명령어가 필요하다

```shell
ps | tee pslog #화면과 파일 양쪽에 ps 명령어 결과를 출력한다. 단 기존 파일에 덮어쓰기 된다.
ps | tee -a pslog #기존 파일 끝에 추가된다.
```

- test할 것.
cat stdout.log | grep -n "debugging word"
sed '-300,300!d' stdout.log | less
/debugging word에다가 f누르면서 찾아보기

```shell
!cat #cat으로 실행한 명령어 중 가장 최근 것 실행
```

- shell 변수는 변수를 설정한 셸에서만 유효함
- 환경변수는 해당 셸에서 실행한 프로세스에도 유효함.


```shell
declare #유효한 쉘 변수 표시
printenv #유효한 환경 변수 표시
VARNAME=value123 # 변수명 = 값/ 쉘 변수 설정 띄어쓰기 없이 다 붙여써야함.
unset VARNAME #쉘 변수 해제
export name=value #환경변수 설정

```

- test할 것. 쉘 변수는 쉘 끄면 해제되는 걸까? 해제된다.
declare
LOGPATH = /userdir/ncop/~~~~~~~해서 stdout.log 전까지
cd $LOGPATH

```shell
source ~/.bashrc #셸 재시작 없이 설정파일 변경 반영

```

```shell
ping [ip or domain] #접속포트가 열렸는지 확인. ping이 안된다고 늘 접속 문제는 아님 ICMP 패킷 응답하지 않는 서버도 존재.
dig [id or domain] #DNS 등록이 되어있는 지 확인. 되어있다면 answer section에 DNS 정보가 뜬다.
ifconfig #네트워크 ip주소 확인
```

```shell
ssh rrr@111.11.231.11 -p 1110 #해당 서버에 해당 계정으로 접속한다.
```

```shell
wget https://www.example.com/index.html #웹 상의 파일 다운로드
curl -0 https://www.example.com/index.html #웹 상의 파일 다운로드
```

- ftp나 lftp로 파일을 전송할 수 있다.
- 그러나 암호화가 안되므로 sftp나 scp로 보내는 게 좋다.

```shell
sftp id@[ip or domain] #파일을 전송하기 위해 해당 서버 접속
mget file #내 서버로 파일을 가져온다
mput file #해당 서버로 파일을 보낸다.
scp root@111.223.44.11:~/file #해당 서버로 파일을 암호화하여 전송
```
 





