---
layout: post
title: [Oracle] 다중 LIKE 연산자 - 정규표현식 (REGEXP_LIKE)
tags: [Oracle]
author: hhhongso
excerpt_separator: 
---


오라클에서 WHERE 조건에 2개 이상의 값을 넣는 경우가 있다.

이 경우 OR 연산자를 써서 여러 조건을 추가할 수도 있지만, 

```
SELECT * FROM EMPLOYEES WHERE LAST_NAME = 'King' OR LAST_NAME = 'Austin'; 
```

OR 연산의 반복을 피하기 위해 **IN 연산자** 를 사용한다. 

```
SELECT * FROM EMPLOYEES WHERE LAST_NAME IN ('King', 'Austin'); 
```

![img_01](../z.images/oracle-regexp_like_01.png)

조건이 명확할 경우(EX) 이름이 '홍길동', '김영희' 인 데이터 찾기)에는 위의 방법으로 사용이 가능하지만, 특정 문자가 들어간 모든 데이터를 찾고자 할 경우(EX) 이름에 '홍'이 들어간 데이터 찾기)에는 이야기가 달라진다. 

우선, 기본적으로 **LIKE 연산자**를 활용하여야 한다. 

```
SELECT * FROM EMPLOYEES WHERE LAST_NAME LIKE '%ins%'; 
```

![img_02](../z.images/oracle-regexp_like_02.png)

LIKE 연산자 역시 OR 를 반복하여 사용할 수 있다. 하지만, LIKE 연산자는 IN 연산자와 함께 사용할 수 없어, 다음과 같이 사용할 경우 구문 오류를 뱉어낸다. 

이를 해결하기 위해 다중 LIKE 연산에는 **REGEXP_LIKE(컬럼명, 정규식)**을 사용한다.

**파이프( | )** 를 이용해 단어를 구분하는데, 일반적으로 단어만 나열할 경우 LIKE '%ins%' 와 같은 효과를 얻을 수 있다.

```
SELECT * FROM EMPLOYEES WHERE REGEXP_LIKE (LAST_NAME, 'ins|Mar');
```

![img_03](../z.images/oracle-regexp_like_03.png)

정규표현식을 사용하여 더 효율적인 검색도 가능하다. REGEXP_LIKE 외에도 정규표현식을 사용하는 연산자는 아래와 같다. 

**REGEXP\_LIKE**

**REGEXP\_INSTR**

**REGEXP\_SUBSTR**

**REGEXP\_REPLACE**

**REGEXP\_COUNT** : 특정 문자의 개수를 세는 함수


#### 정규표현식 정리

- '^pattern' : Pattern으로 시작하는 line 출력  

```
SELECT * FROM EMPLOYEES WHERE REGEXP_LIKE (LAST_NAME, '^A|^B') order by LAST_NAME asc;
SELECT * FROM EMPLOYEES WHERE REGEXP_LIKE (LAST_NAME, '^a|^b', 'i') order by LAST_NAME asc; -- 대소문자 구분 없애기 위해 'i' 추가
```

![img_04](../z.images/oracle-regexp_like_04.png)

- 'pattern$' : Pattern으로 끝나는 line 출력

```
SELECT * FROM EMPLOYEES WHERE REGEXP_LIKE (LAST_NAME, 'a$|s$');
```

![img_05](../z.images/oracle-regexp_like_05.png)

- 'p....n' : p로 시작하여 n으로 끝나는 line 

```
SELECT * FROM EMPLOYEES WHERE REGEXP_LIKE (LAST_NAME, 'M...s'); -- . = 1 character
```

![img_06](../z.images/oracle-regexp_like_06.png)
  
- '\[a-z\]\*' : 모든 이라는 뜻. 글자수가 0일 수도 있음

- '\[Pp\]attern' : Pattern에 해당하는 한 문자

- '\[^a-m\]attern' : Pattern에 해당하지 않는 한 문자


#### References: 
[https://muyu.tistory.com/entry/Oracle-정규-표현식-정리-regexp](https://muyu.tistory.com/entry/Oracle-정규-표현식-정리-regexp)