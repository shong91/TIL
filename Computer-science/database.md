# CS 지식 모음 - DB

> ### Q: RDBMS와 NoSQL의 차이는?

| RDBMS                                  | NoSQL                                         |
| -------------------------------------- | --------------------------------------------- |
| 관계형DB - 스키마에 맞추어 데이터 관리 | 비관계형DB- key-value 형태로 데이터 관리      |
| 데이터의 정합성 보장                   | 자유로운 데이터 관리 가능                     |
| 시스템이 커질수록 관리 어려움          | 중복 데이터 추가가 가능하여, 관리 포인트 상승 |
| (복잡도 향상, 수평적 확장 어려움)      |                                               |

> ### Q: 인덱스 란?

> ### A: 인덱스 = 데이터베이스 테이블의 검색 속도 향상을 위한 자료구조.

- 인덱스 미적용 시 Full scan 수행되어 조회 처리 속도가 떨어지는 쿼리를, 인덱스(=색인) 을 적용하여 조회 성능을 개선할 수 있다.
- B+Tree 인덱스를 주로 사용: Tree의 리프노드들을 LinkedList로 연결하여 순차 검색을 용이하게 함.

> ### 힌트 란?

> ### A: 힌트 = SQL 튜닝을 위한 지시구문

옵티마이저가 최적의 계획으로 SQL 문을 처리하지 못하는 경우, 개발자가 HINT 절을 통해 직접 최적의 실행 계획을 제공한다.

> ### Q: 트랜잭션 이란?

> ### A: 데이터베이스 작업의 단위. 트랜잭션 단위로 커밋/롤백된다. ACID 원칙을 준수하여야 한다.

- 원자성(Atomicity): 트랜잭션에 포함된 작업은 전부 수행되거나 전부 수행되지 않아야 한다.
- 일관성(Consistency): 트랜잭션을 수행하기 전이나 후나 데이터베이스는 항상 일관된 상태를 유지해야 한다.
- 고립성(Isolation): 수행 중인 트랜잭션에 다른 트랜잭션이 끼어들어 변경중인 데이터 값을 훼손하지 않아야한다.
- 지속성(Durability): 수행을 성공적으로 완료한 트랜잭션은 변경한 데이터를 영구히 저장해야 한다.

> ### Q: 트랜잭션 격리수준(Isolation Level) 에 대하여 설명하시오.

> ### A: 특정 트랜잭션이 다른 트랜잭션에 변경한 데이터를 볼 수 있도록 허용할지 결정하는 격리 수준

- READ UNCOMMITTED
  - commit/rollback 여부와 상관없이 조회 가능
  - Dirty Read 발생
  - ACID 가 깨질 수 있기 때문에 비권장됨
- READ COMMITTED
  - 트랜잭션이 commit 되어야만 다른 트랜잭션에서 조회 가능 (ORACLE default)
  - Non-repeatable read 발생
- REPEATABLE READ
  - 트랜잭션이 시작되기 전에 커밋된 내용에 대해서만 조회 가능 (MySQL default)
  - Phantom read 발생
- SERIALIZABLE
  - 트랜잭션 단위 read consistency 적용 (가장 완벽한 read consistency)

#### 참고

https://joont92.github.io/db/트랜잭션-격리-수준-isolation-level/

> ### Q: DB Lock 의 종류

> ### A: 여러 개의 트랜잭션들이 하나의 데이터로 동시에 접근하려고 할 때, 이를 제어해주는 도구

- 공유락(LS, Shared Lock): 트랜잭션이 읽기를 할 때 사용하는 락, 데이터를 읽을 수 있지만 쓸 수 없음
- 배타락(LX, Exclusive Lock): 트랜잭션이 읽고 쓰기를 할 때 사용하는 락, 데이터를 읽고 쓸 수 있음

> ### Q: 데드락 Deadlock 이란 ?

> ### A: 트랜잭션들이 동시 실행될 때, 서로가 서로에 대한 락을 소유한 상태로 대기상태에 빠져 더이상 진행하지 못하는 상황.

#### 참고

https://www.letmecompile.com/mysql-innodb-lock-deadlock/
{
"memberId": 1,
"itemList": [
{
"id": 7,
"orderQuantity": 2
}
]
}

| API URI                  | HttpMethod | API 명         | 요청               | 응답            |
| ------------------------ | ---------- | -------------- | ------------------ | --------------- |
| /api/v1/items            | GET        | 메뉴 목록 조회 | -                  | `List<ItemDto>` |
| /api/v1/point/{memberId} | PUT        | 포인트 충전    | {"points": 10000}  | `pointId`       |
| /api/v1/order            | POST       | 커피 주문/결제 | {                  | `orderId`       |
|                          |            |                | "memberId": 1,     |                 |
|                          |            |                | "itemList": [{     |                 |
|                          |            |                | "id": 7,           |                 |
|                          |            |                | "orderQuantity": 2 |                 |
|                          |            |                | }]                 |                 |
| /api/v1/items/top3       | GET        | 인기 메뉴 조회 | -                  | `List<ItemDto>` |
