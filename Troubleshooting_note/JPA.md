1. 초기 세팅
   pom.xml 에 jpa 관련 디펜던시 추가 시 함께 해주어야 하는 것들: jpa, hibernate, H2

```
    <!-- jpa -->
    <dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>

		<!-- JPA 하이버네이트 -->
		<dependency>
			<groupId>org.hibernate</groupId>
			<artifactId>hibernate-entitymanager</artifactId>
			<version>5.3.10.Final</version>
		</dependency>

		<!-- H2 데이터베이스 -->
		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<version>1.4.199</version>
		</dependency>
```

2. dbeaver에서 확인하기: H2 embeded

- new connection 설정

  1. DBeaver > create new connection > Embedded > H2 Embedded
  2. 원하는 로컬 path, username/password 설정
  3. Test connection 확인

- The file is locked: nio:C:/shong91/Database/h2/my-jpa-database.mv.db
  1. 원인:
     Just configure the h2 database in embedded mode, and the file mode only allows one connection.
  2. 해결:
     shut down any additional connection (from spring boot or dbeaver etc.) and restart.
     or, start a separate h2 database for remote connection.
