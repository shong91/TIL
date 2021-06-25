# What is JPA (Java Persistence API) ?

### 1. 정의

자바 ORM 기술에 대한 API 표준 명세. (ORM을 사용하기 위한 인터페이스의 모음)

JPA는 API의 규격이며, Hibernate, OpenJPA 등이 JPA를 구현한 구현체(ORM 프레임워크)이다.

> cf) ORM? (Object-Relational Mapping; 객체와 관계형 데이터베이스 매핑 )

- 객체가 테이블이 되도록 매핑시켜주는 프레임워크.
- 프로그램의 복잡도를 줄이고, 자바 객체와 쿼리를 분리할 수 있으며, 트랜잭션 처리나 기타 데이터베이스 관련 작업들을 편리하게 처리할 수 있는 방법.

### 2. 왜 JPA를 사용할까?

DB중심 설계의 단점을 보완하고, 효율적인 개발 방법론에 대한 고민.
=> 단순히 객체(DTO, VO) 를 데이터 전달 목적으로만 사용하는 것이 아닌, 객체지향의 장점을 살리고 객체와 테이블을 매핑시켜주는 ORM이 주목받기 시작.

**장점**

- 생산성, 유지보수 용이
  - SQL을 직접 작성하는 것이 아닌, 객체를 사용하여 동작 -> 재사용성 up
- DBMS에 대한 종속성 감소
  - DB 컬럼 변경에 따른 테이블/쿼리 수정 작업 감소
  - 각 객체에 대한 코드 별도 작성 -> Object에만 집중하면 되는 방식. 코드의 가독성 up
- 실시간 처리용 쿼리에 최적화

**단점**

- 업무 비즈니스가 복잡할 경우 JPA로 처리하기 어려움 (통계처리 등)
- 대용량 데이터 기반 환경에서 튜닝 어려움

JPA를 사용하는 것이 무조건적으로 좋은 것은 아니며, 업무환경 및 장단점을 고려하여 Mybatis/JPA 사용 여부를 결정하여야 함.

# 초기 세팅

### 1. pom.xml 에 jpa 관련 디펜던시 추가: jpa, hibernate, H2.

인메모리 DB(H2) 를 사용하거나, MariaDB, MySQL등 DBMS를 연결할 수도 있다.

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

### 2. application.properties 설정 - mariaDB

```
# DBMS
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://localhost:3306/${스키마명}
spring.datasource.username=${username}
spring.datasource.password=${password}

#JPA
# Dialect
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MariaDBDialect

#하이버네이트가 실행하는 모든 SQL문을 콘솔로 출력해 준다.
spring.jpa.properties.hibernate.show_sql=true

#콘솔에 출력되는 JPA 실행 쿼리를 가독성있게 표현한다.
spring.jpa.properties.hibernate.format_sql=true

#디버깅이 용이하도록 SQL문 이외에 추가적인 정보를 출력해 준다.
spring.jpa.properties.hibernate.use_sql_comments=true

spring.jpa.generate-ddl=true
spring.jpa.hibernate.ddl-auto=none
spring.jpa.show-sql=true

```

> cf) H2 embeded 시 new connection 설정

1. DBeaver > create new connection > Embedded > H2 Embedded
2. 원하는 로컬 path, username/password 설정
3. Test connection 확인

- The file is locked: nio:C:/shong91/Database/h2/my-jpa-database.mv.db
  1. 원인:
     Just configure the h2 database in embedded mode, and the file mode only allows one connection.
  2. 해결:
     shut down any additional connection (from spring boot or dbeaver etc.) and restart.
     or, start a separate h2 database for remote connection.

### 3. 도메인 생성하기

- @Entity:

  - DB 테이블과 1:1로 매칭되는 객체단위. 인스턴스 하나가 테이블에서 하나의 레코드 값을 의미한다.
  - application.properties에서 설정한 hibernate.ddl-auto 설정에 따라(create 혹은 update로 되어있을 경우) 프로젝트가 시작될 때 EntityManager가 자동으로 DDL을 수행해 테이블을 생성해준다.
  - name속성을 지정할 경우 DB에 생성될 테이블명을 정할 수 있다.

- @Id: PK설정
- @GeneratedValue(strategy = GenerationType.IDENTITY):
  - PK를 자동으로 생성
  - GenerationType을 IDENTITY: auto increment로 설정.
