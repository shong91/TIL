---
layout: post
title: [Oracle] 오라클 기본키(PK) 2개 이상 지정하기
tags: [Oracle]
author: shong91
excerpt_separator: 
---


DB를 관리하다 보면 기본키를 2개 이상 지정하여야 하는 경우가 있다. <br>
테이블 생성 시 기본키를 지정할 때, 아래와 같이 생성한다면 기본키 에러가 발생한다. 

```
CREATE TABLE TEST(
    CODE VARCHAR2(30) PRIMARY KEY, 
    SEQ NUMBER PRIMARY KEY
);
```

**ORA-02260: table can have only one primary key**

기본키는 복수가 되는데 왜? 라는 의문을 가질수 있지만 <br>

**'기본키를 구성하는 컬럼이 복수일 수는 있어도'** <br>

**'기본키가 복수일 수는 없다'** 라고 생각하면 이해가 쉬울 것 같다. <br>
<br>
그럼 기본키를 구성하는 컬럼을 복수로 하기위해선 이와 같이 하나의 기본키 명에 해당 컬럼들을 포함시키는 방식으로 제약조건을 구성하여야 한다. 

```
CREATE TABLE TEST(
    CODE VARCHAR2(30), 
    SEQ NUMBER,
    
    CONSTRAINT TEST_PK(기본키이름) PRIMARY KEY(CODE, SEQ)
);
```

**Table TEST이(가) 생성되었습니다.** <br>

이 경우 두 개 컬럼이 하나의 기본키로 작동하기 때문에 컬럼1 / 2는 각각 중복될 수 있으나, 컬럼 1 & 2 의 값이 중복되는 경우에는 기본키의 UNIQUE 오류를 내게 된다. 

```
INSERT INTO TEST (CODE, SEQ) VALUES ('A', '001');
INSERT INTO TEST (CODE, SEQ) VALUES ('A', '002');
INSERT INTO TEST (CODE, SEQ) VALUES ('A', '003');
INSERT INTO TEST (CODE, SEQ) VALUES ('B', '001');
INSERT INTO TEST (CODE, SEQ) VALUES ('B', '002');

INSERT INTO TEST (CODE, SEQ) VALUES ('A', '001'); -- Unique constraint violated
```

#### References:
[https://multifrontgarden.tistory.com/31](https://multifrontgarden.tistory.com/31)