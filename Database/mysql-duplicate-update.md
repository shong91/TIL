---
layout: post
title: [MySQL] INSERT 시 중복 키 관리하는 방법 (INSERT IGNORE, REPLACE INTO, ON DUPLICATE UPDATE)
tags: [MySQL]
author: shong91
excerpt_separator: 
---

# MySQL 에서 중복 레코드를 관리하는 방법

> **전제**
> PRIMARY KEY / UNIQUE INDEX 로 중복 키를 관리한다.

### 1. INSERT IGNORE ...

- 중복 키 에러 발생 시, 신규 입력되는 레코드를 무시한다. (INSERT 되지 않음)
- 최초 insert record 가 남아있음
- 최초 insert record 의 auto_increment 값은 변하지 않음

```
mysql> INSERT IGNORE INTO person VALUES (NULL, 'James', 'Seoul');
Query OK, 1 row affected (0.00 sec)

mysql> INSERT IGNORE INTO person VALUES (NULL, 'James', 'Seongnam');
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM person;
+----+---------+---------+
| id | name    | address |
+----+---------+---------+
|  1 | James   | Seoul   |
+----+---------+---------+
```

### 2. REPLACE INTO ...

- 중복 키 에러 발생 시, 신규 입력되는 레코드로 대체된다.
- 최초 insert record 가 삭제되고 신규 record 가 insert 됨
- auto_increment 값 변경

```
mysql> REPLACE INTO person VALUES (NULL, 'James', 'Seoul');
Query OK, 1 row affected (0.00 sec)

mysql> REPLACE INTO person VALUES (NULL, 'James', 'Seongnam');
Query OK, 2 rows affected (0.00 sec)


mysql> SELECT * FROM person;
+----+---------+---------+
| id | name    | address |
+----+---------+---------+
|  1 | James   | Seongnam|
+----+---------+---------+
```

### 3. INSERT INTO ... ON DUPLICATE UPDATE

- 중복 키 에러 발생 시, 사용자가 UPDATE될 값을 지정할 수 있다.
- Oracle 의 MERGE INTO 문과 동일 기능.

```
mysql> INSERT INTO person VALUES (NULL, 'James', 'Seoul')
           ON DUPLICATE KEY UPDATE address = VALUES(address);
Query OK, 1 row affected (0.00 sec)

mysql> INSERT INTO person VALUES (NULL, 'James', 'Incheon')
           ON DUPLICATE KEY UPDATE address = VALUES(address);
Query OK, 2 rows affected, 1 warning (0.01 sec)

mysql> SELECT * FROM person;
+----+---------+---------+
| id | name    | address |
+----+---------+---------+
|  1 | James   | Incheon |
+----+---------+---------+
```

### References:

1. http://jason-heo.github.io/mysql/2014/03/05/manage-dup-key2.html
