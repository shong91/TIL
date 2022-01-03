# 집합연산자

여러 데이터를 결합할 수 있는 집합연산자. (`union`, `intersect`, `except`)

## 1. 교집합 `intersect`

A intesect B

### _Oracle_

오라클 또는 SQL 서버 2008을 사용할 경우에는 intersect 연산자를 사용할 수 있다.

```
SELECT
	c.first_name
	, c.last_name
FROM customer c
WHERE c.first_name LIKE 'J%'
  AND c.last_name LIKE 'D%'
INTERSECT
SELECT
	a.first_name
	, a.last_name
FROM actor a
WHERE a.first_name LIKE 'J%'
  AND a.last_name LIKE 'D%' ;
```

### _mySQL_

MySQL 버전 8.0은 INTERSECT 연산자를 지원하지 않는다. (ANSI SQL 사양에서 지원)

INNER JOIN 을 사용하여 동일한 결과를 가져올 수 있다.

```
SELECT
	c.first_name
	, c.last_name
FROM customer c
	INNER JOIN actor a
	ON c.first_name = a.first_name
	AND c.last_name = a.last_name ;
```

## 2. 차집합 `except`

A except B

### _Oracle_

오라클 데이터베이스를 사용할 경우, ANSI와 호환되지 않는 minus 연산자를 사용해야 한다.

단, MINUS 집합 연산은 항상 DISTINCT하게 중복 레코드를 제거하고 리턴하기 때문에 SELECT의 최종 결과에 DISTINCT를 붙혀 줘야 다른 DBMS의 MINUS와 동일한 결과를 얻을 수 있다.

```
SELECT
	c.first_name
	, c.last_name
FROM customer c
WHERE c.first_name LIKE 'J%'
  AND c.last_name LIKE 'D%'
MINUS
SELECT
	a.first_name
	, a.last_name
FROM actor a
WHERE a.first_name LIKE 'J%'
  AND a.last_name LIKE 'D%' ;
```

### _mySQL_

MySQL 버전 8.0은 EXCEPT 연산자를 지원하지 않으며, OUTER JOIN(LEFT OUTER JOIN / RIGHT OUTER JOIN) 을 통해 같은 결과를 가져올 수 있다.

```
SELECT
	a.first_name
	, a.last_name
FROM customer c
	RIGHT JOIN actor a
	ON c.last_name = a.last_name
	AND c.first_name = a.first_name
WHERE c.customer_id IS NULL
  AND a.first_name LIKE 'J%'
  AND a.last_name LIKE 'D%' ;
```

## 3. 대칭차집합 `union`, `union all`

(A ∪ B) - (A ∩ B) : (A Union B) except (A intersect B)
(A - B) + (B - A): (A except B) union (B except A)

`UNION ALL` 은 중복을 포함, `UNION` 은 중복을 제거하고 데이터셋을 가져온다.

### _Oracle / MySQL_

```
SELECT
	c.first_name
	, c.last_name
FROM customer c
WHERE c.first_name LIKE 'J%'
  AND c.last_name LIKE 'D%'
UNION ALL
SELECT
	a.first_name
	, a.last_name
FROM actor a
WHERE a.first_name LIKE 'J%'
  AND a.last_name LIKE 'D%' ;
```

### Reference.

1. https://velog.io/@golmori/working-with-sets
2. http://intomysql.blogspot.com/2011/01/mysql-minus-intersect.html
