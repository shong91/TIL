# JPA 개념 정리

## what is JPA(Java Persistence API) ?

- 자바 ORM 기술에 대한 API 표준 명세. (ORM을 사용하기 위한 인터페이스의 모음)
- JPA는 API의 규격일 뿐. (라이브러리나 프레임워크가 아님) Hibernate, OpenJPA 등이 JPA를 구현한 구현체(ORM 프레임워크)이다.

## what is ORM (Object-Relational Mapping) ?

- 객체가 DB 테이블이 되도록 매핑시켜주는 프레임워크.
- 객체 간의 관계를 바탕으로 SQL 을 자동으로 생성
- 프로그램의 복잡도를 줄이고, 자바 객체와 쿼리를 분리할 수 있으며, 트랜잭션 처리나 기타 데이터베이스 관련 작업들을 편리하게 처리할 수 있는 방법.

| SQL Mapper                  | ORM                                       |
| --------------------------- | ----------------------------------------- |
| 자바 클래스와 sql을 매핑    | 자바 클래스와 DB 테이블을 매핑            |
| SQL을 명시하여 직접 DB 조작 | 객체간의 관계를 바탕으로 SQL 자동 생성    |
| 필드를 매핑시키는 것이 목적 | RDB의 관계를 Object 에 반영하는 것이 목적 |
| myBatis, jdbcTemplate       | JPA, Hibernate                            |

## JPA 동작 과정

![img_01](https://velog.velcdn.com/images%2Fadam2%2Fpost%2Fcde32cd8-b9c0-49c4-bf99-b58c0b0c2e18%2FUntitled%203.png)

- JPA 는 애플리케이션과 JDBC 사이에서 동작하여
- 개발자가 JPA 를 사용하면, JPA 내부에서 JDBC API 를 사용하여 SQL 을 호출, DB 와 통신한다.

## why JPA ?

DB중심 설계의 단점을 보완하고, 효율적인 개발 방법론에 대한 고민.
=> 단순히 객체(DTO, VO) 를 데이터 전달 목적으로만 사용하는 것이 아닌, 객체지향의 장점을 살리고 객체와 테이블을 매핑시켜주는 ORM이 주목받기 시작.

### 장점

- 객체 중심적 개발 가능
  - SQL 코드의 반복, 객체지향과 관계지향 데이터베이스의 패러다임 불일치 해소
  - DBMS에 대한 종속성 감소: DB 컬럼 변경에 따른 테이블/쿼리 수정 작업 감소
- 생산성, 유지보수 용이
  - SQL을 직접 작성하는 것이 아닌, 객체를 사용하여 동작 -> 재사용성 up
- 실시간 처리용 쿼리에 최적화

### 단점

- 업무 비즈니스가 복잡할 경우 JPA로 처리하기 어려움 (통계처리 등)
- 대용량 데이터 기반 환경에서 튜닝 어려움

JPA를 사용하는 것이 무조건적으로 좋은 것은 아니며, 업무환경 및 장단점을 고려하여 Mybatis/JPA 사용 여부를 결정하여야 함.

## Reference.

1. https://velog.io/@adam2/JPA는-도데체-뭘까-orm-영속성-hibernate-spring-data-jpa
