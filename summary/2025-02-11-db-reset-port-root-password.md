## <span style="color:#802548">_local db port 변경_</span>

- 아래 경로로 들어가 파일을 연다.

```sh
cd C:\Program Files\MariaDB 10.x\data\my.ini
```

- port 번호 변경 후 아래 명령어를 관리자 cmd에서 실행한다.

```sh
net stop mysql
net start mysql
```

## <span style="color:#802548">_root 비번 변경_</span>


- 아래 명령어를 수행한다. 

```sql
use mysql;
ALTER USER 'root'@'localhost' IDENTIFIED BY '원하는 password';
```