# QueryDSL

- SQL, JPQL을 코드로 작성할 수 있도록 도와주는 빌더 API

## 특징

### SQL, JPQL의 문제점

"자바와 문자열의 한계"

- Type-check 불가능
- 컴파일 시점에 오류 확인 불가. 해당 로직(쿼리) 실행 시점에 오류를 발견할 수 있음

### QueryDSL 장점

- 문자열 X -> 코드로 작성 (JPQL 과 거의 유사하게 작성 가능)
- 컴파일 시점에 문법 오류를 발견
- 동적 쿼리 사용 시 유용

```
// JPQL
select m from Member m where m.age > 18

JPAFactoryQuery query = new JPAQueryFactory(em);

// Member 엔티티를 기반으로, QMember 라는 QueryDSL 전용 객체 생성
QMember m = QMember.member;

// QueryDSL 코드로 작성​
List<Member> list =
    query.selectFrom(m)
         .where(m.age.gt(18))
         .orderBy(m.name.desc())
         .fetch();

```

## QueryDSL 으로 동적 쿼리 작성하기

- BooleanBuilder에 조건을 넣고 쿼리를 실행시킨다. (조건이 있으면 넣고, 없으면 안넣고)

```
String name = "member";
int age = 9;
​
QMember m = QMember.member;
​
BooleanBuilder builder = new BooleanBuilder();
if(name != null) {
    builder.and(m.name.contains(name));
}
if(age != 0) {
    builder.and(m.age.gt(age));
}
​
List<Member> list =
    query.selectFrom(m)
         .where(builder)
         .fetch();

```

객체 지향적인 관점에셔, where 조건에 들어가는 제약조건이 중복 사용될 경우 method 로 extract 하여 재사용할 수 있다.

```
return query.selectFrom(coupon)
            .where(
                 coupon.type.eq(typeParam),
                 isServiceable()
       )
       .fetch();
​
private BooleanExpression isServiceable() {
    return coupon.status.wq("LIVE")
        .and(marketing.viewCount.lt(markting.maxCount));
}
```

## Reference.

1. https://ict-nroo.tistory.com/117
