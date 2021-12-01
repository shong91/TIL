# docker를 사용하여 mariaDB 설치하기 

## 1. docker 이미지 다운로드 
```
$ docker pull mariadb
```

## 2. 컨테이너 실행 
- -p: 3306 port 로 설정 
- -v: docker volume 을 바인딩함: \<bind-path>:\<file or directory on the host machine> 
- --name: 컨테이너 실행 시 이름을 부여 

```
$ docker container run -d -p 3306:3306 	\
-e MYSQL_ROOT_PASSWORD=1234 		\
-v /Users/Shared/happy/mariadb:/var/lib/mysql 	\
--name mariadb mariadb

```

## 3. 정상 실행 여부 확인 
```
$ docker container ls -a
```

## 4. 접속 확인 
```
$ docker exec -i -t mariadb bash
$ mysql -uroot -p1234
```

## 5. 정상 접속 확인 & 테스트 데이터베이스 생성
```
$ show databases; 
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.001 sec)

$ create database test; 
+--------------------+
| Database           |
+--------------------+
| test               |
| information_schema |
| mysql              |
| performance_schema |
+--------------------+

```

## 6. my.cnf 수정: utf8 설정 

### 6-1. my.cnf 위치 확인

```
$ docker exec -i -t mariadb bash
$ mysql --help | grep my.cnf
/etc/my.cnf /etc/mysql/my.cnf ~/.my.cnf 
                      order of preference, my.cnf, $MYSQL_TCP_PORT,
```

### 6-2. vim 설치 
```
$ apt-get update 
$ apt-get install vim
```

### 6-3. my.cnf 파일 수정 
```
$ vi /etc/mysql/my.cnf
```

```
character-set-client-handshake = FALSE
init_connect="SET collation_connection = utf8mb4_general_ci"
init_connect="SET NAMES utf8"
character-set-server = utf8mb4

[client]
default-character-set = utf8

[mysql]
default-character-set = utf8

[mysqldump]
default-character-set = utf8
```

처음에는 모두 utf8으로 맞춰줬었는데, 이렇게 하고 mariadb를 재실행하니 /**[ERROR] COLLATION 'utf8mb4_general_ci' is not valid for CHARACTER SET 'utf8mb3'** 라는 에러가 발생했다. 

구글링해보니 collation 과 character-set-server 의 설정값이 잘못된 것으로, 해당 값은 utf8mb4, utf8mb4_general_ci 로 바꾸어주니 문제가 해결되었다. 


## 7. 컨테이너 재시작

```
$ docker stop mariadb && docker start mariadb
```

## 8. 로그 확인 
정상 여부와, 오류 발생 시 오류 로그를 확인할 수 있다. 
```
$ docker logs -f --tail=10 mariadb
```


### Reference. 
https://happygrammer.github.io/docker/mariadb/
