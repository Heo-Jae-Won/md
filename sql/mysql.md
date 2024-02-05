- mysql cli에서 한글 깨질 떄 아래와 같이 써준다.
```sh
ALTER DATABASE your_database_name CHARACTER SET utf8 COLLATE utf8_general_ci;

mysql -u your_user -p  --port 3355 --default-character-set=utf8 databaseName
```