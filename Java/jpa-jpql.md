# JPQL (Java Persistence Query Language)

엔티티를 조회하는 객체지향 쿼리.

특정 데이터베이스에 의존하지 않고 엔티티 객체를 조회할 수 있다.

## 기본 문법

```
SELECT m FROM Member m WHERE m.username = :username
```

1. 대소문자 구분

엔티티와 속성은 대소문자를 구분한다.

예를 들어, Member, username 과 같은 엔티티 속성은 대소문자를 구분하여야 하며,

SELECT, FROM, WHERE 같은 JPQL 키워드는 대소문자를 구분하지 않아도 된다.

2. 엔티티 이름

JPQL에서 사용한 Member는 클래스 명이 아니라 엔티티 명이다.

엔티티명은 @Entity(name="abc")로 지정할 수 있으며, 지정하지 않을 시 클래스명을 기본값으로 사용한다.

3. Alias 필수

JPQL은 별칭을 필수로 사용하여야 한다. (AS 생략 가능)

## 쿼리 객체 (TypeQuery, Query)

JPQL 실행을 위해 쿼리 객체를 생성하여야 한다.

반환 타입을 명확히 지정할 수 있을 경우 `TypeQuery` 객체를, 명확히 지정이 불가할 경우 `Query` 객체를 사용한다.

```
// TypeQuery
List<Member> resultList = em.createQuery("select m from Member m", Member.class).getResultList();

// Query
List resultList = em.createQuery("select m.username, m.age from Member m").getResultList();
```

## 파라미터 바인딩

1. 이름 기준 파라미터

`:params` 로 표기. (권장)

```
List<Member> resultList = em.createQuery("select m from Member m where m.username = :username", Member.class)
                          .setParameter("username", param)
                          .getResultList();

```

2. 위치 기준 파라미터

`?index` 로 표기. index 값은 1부터 시작한다.

```
List<Member> resultList = em.createQuery("select m from Member m where m.username = ?1", Member.class)
                          .setParameter(1, param)
                          .getResultList();

```

## 프로젝션

SELECT 절에 조회할 대상을 지정하는 것을 프로젝션이라 한다.

1. 엔티티 프로젝션

원하는 엔티티 객체를 바로 조회할 수 있다.

조회한 엔티티는 영속성 컨텍스트에서 관리된다.

```
List<Member> memberList = em.createQuery("SELECT m FROM Member m", Member.class) .getResultList();
```

2. 임베디드 타입 프로젝션

임베디드 타입은, 엔티티 타입이 아닌 값 타입이다.

조회한 임베디드 타입은 영속성 컨텍스트에서 관리되지 않는다.

```
List<Address> addressList = em.createQuery("SELECT o.address FROM Order o", Address.class) .getResultList();
```

3. 스칼라 타입 프로젝션

스칼라 타입은 숫자, 문자, 날짜와 같은 기본 데이터 타입으로, 통계 쿼리에 주로 이용된다.

```
List<String> usernameList = em.createQuery("select username from Member m", String.class) .getResultList();
```

4. new 명령어

엔티티 전체가 아닌 꼭 필요한 데이터만 선택하여 조회하고자 할 때, new 명령어로 객체로 변환하여 사용한다.

```
public class UserDTO{

    private String username;
    private int age;

    public UserDTO(String username, int age){
        this.username = username;
        this.age = age;
    }
}

List<UserDTO> resultList  =
    em.createQuery("select new jpabook.jpql.UserDTO(m.username, m.age) from Member m", UserDTO.class)
    .getResultList();
```

## 페이징 API

- setFirstResult (int startPosition) : 조회 시작 위치 (0부터 시작한다.)
- setMaxResults (int maxResult) : 조회할 데이터 수

```
 return em.createQuery("select o from Order o join o.member m "
        + "where o.status = :status "
        + "and m.name like :name", Order.class)
        .setParameter("status", orderSearch.getOrderStatus())
        .setParameter("name", orderSearch.getMemberName())
        .setFirstResult(100) // 페이징 시 변수 설정
        .setMaxResults(1000)
        .getResultList();

```

## Join

1. INNER JOIN

연관 필드 (다른 엔티티와 연관관계를 가지기 위해 사용하는 필드) 를 통해 join

```
List<Member> memberList = em.createQuery("select m from Member m inner join m.team t where t.name = :teamName", Member.class)
                          .setParameter("teamName", teamName)
                          .getResultList();
```

2. OUTER JOIN

```
SELECT m FROM Member LEFT JOIN m.team t
```

3. THETA JOIN

서로 관계없는 엔티티도 where 절을 사용하여 세타 조인할 수 있다.

```
SELECT count(m) FROM Member m, Team t
where m.username = t.name
```

4. FETCH JOIN

JPQL 성능 최적화를 위해 제공하는 기능.

연관된 엔티티나 컬렉션을 한 번에 같이 조회하는 기능으로, n+1 문제를 해결하는 데 주로 사용된다.

```
List<Member> memberList = em.createQuery("select m from Member m join fetch m.team", Member.class)
                          .getResultList();
```

## Reference.

1. https://leveloper.tistory.com/103
