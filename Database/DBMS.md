## Redo / Undo

redo/undo 는 orcale DB 가 판단하여 기능을 수행함.

redo 를 먼저 돌리고 -> 그 데이터 상태에서 -> 끝난 트랜잭션은 디스크로 내리고, 안끝난(장애) 상태는 undo 로 rollback 함.

- REDO

  - Data 복구를 위한 것! (트랜잭션 진행 중에 장애가 발생했을떄, DB를 재기동할 때)
  - ex) 출금 -> 입금 시, redo 는 출금 상태까지 복구를 해줌

- UNDO

  - 원본 이미지를 저장하여, 데이터 변경 시 원본으로 되돌릴 수 있도록 함 (트랜잭션 진행 중에 장애가 발생했을떄)
  - ex) 출금 -> 입금 시, undo 는 트랜잭션 이전 상태로 rollback

## 트랜잭션

트랜잭션: "더이상 분해 불가한 Data 변경의 논리적 최소 작업 단위"

- 동시성 : 동일한 자원을 여러 명이 사용하고자 할 때
  (vs 병렬처리: 각각 다른 자원을 동시에 수행할 수 있도록 할 때)
- 일관성 : 트랜잭션이 종료도니 후에 변경 상태가 반영

### isolation level

1. read uncommitted

- commit/rollback 여부와 상관없이 데이터를 읽을 수 있는 레벨
- ACID 가 깨질 수 있기 때문에 비권장됨

트랜잭션 isolation Level 에 따라 비일관성이 발생하는 3가지 현상
; 낮은 단계의 isolation level에서 발생

- dirty ready (Read uncommitted level)
- non-repeatable read (Read committed level)
- phantom read (Repeatable read level)

2. Read committed

- default

3. Serializable

- 트랜잭션 단위 read consistency 적용 (가장 완벽한 read consistency)

4. Read-only (가장 강한 isolation)

## Optimizer

optimizer: SQL 실행계획을 결정하는 알고리즘. 현재는 CBO 를 사용. optimizer mode 에 따라 실행.

대부분의 SQL문 optimizer 가 잘 최적화 하지만, 일부 경우 제대로 실행하지 못할 때에는 `HINT` 를 사용.

(A, B 라는 테이블이 있다고 가정)

실행계획 상, A 테이블이 먼저 읽힌다고 한다면, A 테이블에 대한 실행계획이 모두 끝나야 다음 실행계획으로 넘어 갈 수 있다. (선행계획으로 다시 돌아가는 경우는 없음. )

## INDEX

Oracle 의 B-tree index

leaf 블럭은 {key:RowID} 를 가짐

full scan 과 index scan 의 손익 분기점 !
Data 양보다 Clustering Factor 가 더 큰 영향을 미침.

### Clustering Factor ?

특정 index의 키 값 정렬 순서대로 table data 가 물리적으로 군집된 정도를 나타낸 수치.

## JOIN
