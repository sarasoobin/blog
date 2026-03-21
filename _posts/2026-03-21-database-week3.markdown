---
layout: post
title:  "데이터베이스 3주차 개발블로그"
date:   2026-03-21
categories: Database
---

# Chapter 3. SQL 기초 정리


## 2. SQL 개요

SQL은 관계 데이터베이스를 위한 표준 질의어고 비절차적 언어다.

분류는 세 가지
- **DDL** (데이터 정의어) : CREATE, ALTER, DROP
- **DML** (데이터 조작어) : SELECT, INSERT, UPDATE, DELETE
- **DCL** (데이터 제어어) : GRANT, REVOKE

SELECT 문 기본 구조는 이렇다.
```
SELECT : 결과로 추출되는 속성 리스트
FROM   : 어느 테이블이 사용되는지
WHERE  : 질의 조건
```

---

## 3. 데이터 조작어 - 검색

### SELECT 문 기본형

```sql
SELECT [ALL | DISTINCT] 속성이름
FROM 테이블이름
[WHERE 조건]
[GROUP BY 속성이름]
[HAVING 조건]
[ORDER BY 속성이름 [ASC | DESC]]
```

ALL은 중복 허용(기본값), DISTINCT는 중복 제거

**질의 예시**
```sql
-- 모든 도서 이름과 가격
SELECT bookname, price FROM Book;

-- 전체 속성
SELECT * FROM Book;

-- 중복 제거하고 출판사만
SELECT DISTINCT publisher FROM Book;
```

---

### WHERE 조건

조건으로 쓸 수 있는 술어들

- 비교 : =, <>, <, <=, >, >=
- 범위 : BETWEEN
- 집합 : IN, NOT IN
- 패턴 : LIKE
- NULL : IS NULL, IS NOT NULL
- 복합 : AND, OR, NOT


```sql
-- 가격 20000 미만
SELECT * FROM Book WHERE price < 20000;

-- 가격 10000~20000 사이
SELECT * FROM Book WHERE price BETWEEN 10000 AND 20000;

-- 굿스포츠 또는 대한미디어 출판사
SELECT * FROM Book WHERE publisher IN ('굿스포츠', '대한미디어');

-- 축구 포함된 도서
SELECT bookname, publisher FROM Book WHERE bookname LIKE '%축구%';

-- 두 번째 글자가 '구'인 도서
SELECT * FROM Book WHERE bookname LIKE '_구%';

-- 축구 관련이고 가격 20000 이상
SELECT * FROM Book WHERE bookname LIKE '%축구%' AND price >= 20000;
```

---

### ORDER BY

```sql
-- 이름순 오름차순
SELECT * FROM Book ORDER BY bookname;

-- 가격 오름차순, 같으면 이름순
SELECT * FROM Book ORDER BY price, bookname;

-- 가격 내림차순, 같으면 출판사 오름차순
SELECT * FROM Book ORDER BY price DESC, publisher ASC;
```

NULL은 오름차순에서 맨 마지막, 내림차순에서 맨 처음에 나온다.

---

### 집계 함수와 GROUP BY

집계 함수 종류 : SUM, AVG, COUNT, MAX, MIN
- NULL값은 제외하고 계산
- WHERE절에서 사용 불가, SELECT나 HAVING 절에서만 사용 가능

```sql
-- 총판매액
SELECT SUM(saleprice) AS 총매출 FROM Orders;

-- custid 2번 고객 총판매액
SELECT SUM(saleprice) AS 총매출 FROM Orders WHERE custid=2;

-- 총판매액, 평균, 최저가, 최고가
SELECT SUM(saleprice) AS Total,
       AVG(saleprice) AS Average,
       MIN(saleprice) AS Minimum,
       MAX(saleprice) AS Maximum
FROM Orders;

-- 판매건수
SELECT COUNT(*) FROM Orders;
```

COUNT(*) 는 NULL 포함 전체 개수, COUNT(속성)은 NULL 제외

**GROUP BY**
```sql
-- 고객별 주문 총수량과 총판매액
SELECT custid, COUNT(*) AS 도서수량, SUM(saleprice) AS 총액
FROM Orders
GROUP BY custid
ORDER BY custid;
```

**HAVING**
GROUP BY 결과에 조건 걸 때 사용. WHERE보다 뒤에 와야 하고 집계함수를 써야 한다.

```sql
-- 가격 8000 이상 도서 구매한 고객 중 2권 이상 구매한 고객
SELECT custid, COUNT(*) AS 도서수량
FROM Orders
WHERE saleprice >= 8000
GROUP BY custid
HAVING COUNT(*) >= 2;
```

SQL 실행 순서 : FROM(1) → WHERE(2) → GROUP BY(3) → HAVING(4) → SELECT(5) → ORDER BY(6)

---

### 조인

여러 테이블을 연결해서 검색하는 것. 조인 속성의 도메인은 같아야 한다.

```sql
-- 고객과 주문 데이터 조인
SELECT *
FROM Customer, Orders
WHERE Customer.custid = Orders.custid;

-- 고객별 정렬
SELECT *
FROM Customer, Orders
WHERE Customer.custid = Orders.custid
ORDER BY Customer.custid;

-- 세 테이블 조인 (고객 이름과 주문 도서 이름)
SELECT Customer.name, Book.bookname
FROM Customer, Orders, Book
WHERE Customer.custid = Orders.custid AND Orders.bookid = Book.bookid;

-- 가격 20000원 도서 주문한 고객 이름과 도서 이름
SELECT Customer.name, Book.bookname
FROM Customer, Orders, Book
WHERE Customer.custid = Orders.custid
  AND Orders.bookid = Book.bookid
  AND Book.price = 20000;
```

테이블 별칭(튜플변수) 사용 가능
```sql
SELECT C.name, B.bookname
FROM Customer C, Orders O, Book B
WHERE C.custid = O.custid AND O.bookid = B.bookid AND B.price = 20000;
```

**외부조인**
조인 조건을 만족하지 않는 튜플도 검색 대상에 포함

```sql
-- 도서 구매하지 않은 고객 포함
SELECT Customer.name, saleprice
FROM Customer LEFT OUTER JOIN Orders
ON Customer.custid = Orders.custid;
```

---

### 부속질의

SELECT 문 안에 또 다른 SELECT 문을 포함하는 것. 괄호로 묶고 ORDER BY 사용 불가. 부속질의 먼저 수행 후 상위 질의 수행.

- 단일 행 부속질의 : 결과가 한 행 → 비교 연산자 사용 가능
- 다중 행 부속질의 : 결과가 여러 행 → IN, EXISTS 등 사용

```sql
-- 가장 비싼 도서 이름
SELECT bookname FROM Book
WHERE price = (SELECT MAX(price) FROM Book);

-- 도서 구매한 적 있는 고객 이름
SELECT name FROM Customer
WHERE custid IN (SELECT custid FROM Orders);

-- 대한미디어 출판 도서 구매한 고객 이름
SELECT name FROM Customer
WHERE custid IN (SELECT custid FROM Orders
                 WHERE bookid IN (SELECT bookid FROM Book
                                  WHERE publisher = '대한미디어'));
```

**상관 부속질의** : 상위 쿼리와 하위 쿼리가 서로 의존적. 행 단위로 반복 실행됨.

```sql
-- 출판사별 평균가격보다 비싼 도서
SELECT b1.bookname
FROM Book b1
WHERE b1.price > (SELECT AVG(b2.price)
                  FROM Book b2
                  WHERE b2.publisher = b1.publisher);
```

**집합 연산**
```sql
-- UNION (합집합, 중복 제거)
SELECT name FROM Customer WHERE address LIKE '대한민국%'
UNION
SELECT name FROM Customer WHERE custid IN (SELECT custid FROM Orders);

-- UNION ALL (중복 포함)
SELECT name FROM Customer WHERE address LIKE '대한민국%'
UNION ALL
SELECT name FROM Customer WHERE custid IN (SELECT custid FROM Orders);
```

MySQL에는 MINUS, INTERSECT 없어서 NOT IN, IN으로 대체한다.

**EXISTS / NOT EXISTS**
```sql
-- 주문이 있는 고객 이름과 주소
SELECT name, address FROM Customer cs
WHERE EXISTS (SELECT * FROM Orders od
              WHERE cs.custid = od.custid);
```

---

## 4. 데이터 정의어

### CREATE TABLE

```sql
CREATE TABLE 테이블이름 (
    속성이름 데이터타입
    [NULL | NOT NULL | UNIQUE | DEFAULT 값 | CHECK 조건]
    [PRIMARY KEY 속성이름]
    [FOREIGN KEY 속성이름 REFERENCES 테이블이름(속성이름)]
    [ON DELETE {CASCADE | SET NULL}]
);
```

데이터 타입 : INTEGER/INT, NUMERIC(m,d), CHAR(n), VARCHAR(n), DATE

```sql
-- 기본키 지정
CREATE TABLE NewBook (
    bookid    INTEGER PRIMARY KEY,
    bookname  VARCHAR(20),
    publisher VARCHAR(20),
    price     INTEGER);

-- 복합 기본키
CREATE TABLE NewBook (
    bookname  VARCHAR(20),
    publisher VARCHAR(20),
    price     INTEGER,
    PRIMARY KEY (bookname, publisher));

-- 제약조건 추가
CREATE TABLE NewBook (
    bookname  VARCHAR(20) NOT NULL,
    publisher VARCHAR(20) UNIQUE,
    price     INTEGER DEFAULT 10000 CHECK(price >= 1000),
    PRIMARY KEY (bookname, publisher));

-- 외래키 지정
CREATE TABLE NewOrders (
    orderid   INTEGER,
    custid    INTEGER NOT NULL,
    bookid    INTEGER NOT NULL,
    saleprice INTEGER,
    orderdate DATE,
    PRIMARY KEY(orderid),
    FOREIGN KEY(custid) REFERENCES NewCustomer(custid) ON DELETE CASCADE);
```

ON DELETE 옵션
- CASCADE : 관련 투플 함께 삭제
- SET NULL : 외래키 값을 NULL로 변경
- RESTRICT : 부모행이 참조되고 있으면 삭제/수정 불가 (기본값)
- NO ACTION : RESTRICT와 동일한 SQL 표준 키워드

### ALTER TABLE

```sql
-- 속성 추가
ALTER TABLE NewBook ADD isbn VARCHAR(13);

-- 타입 변경
ALTER TABLE NewBook MODIFY isbn INTEGER;

-- 속성 삭제
ALTER TABLE NewBook DROP COLUMN isbn;

-- NOT NULL 제약 추가
ALTER TABLE NewBook MODIFY bookname VARCHAR(20) NOT NULL;

-- 기본키 추가
ALTER TABLE NewBook ADD PRIMARY KEY(bookid);
```

### DROP TABLE

```sql
DROP TABLE NewBook;
```

참조하는 테이블이 있으면 삭제 안 됨. 참조하는 테이블부터 먼저 삭제해야 한다.

```sql
DROP TABLE NewOrders;
DROP TABLE NewCustomer;
```

---

## 5. 데이터 조작어 - 삽입/수정/삭제

### INSERT

```sql
INSERT INTO 테이블이름[(속성리스트)]
VALUES (값리스트);
```

```sql
-- 속성 명시
INSERT INTO Book(bookid, bookname, publisher, price)
VALUES (11, '스포츠 의학', '한솔의학서적', 90000);

-- 속성 생략 (테이블 정의 순서대로)
INSERT INTO Book VALUES (12, '스포츠 의학', '한솔의학서적', 90000);

-- 순서 변경
INSERT INTO Book(bookid, bookname, price, publisher)
VALUES (13, '스포츠 의학', 90000, '한솔의학서적');

-- 가격 미정 (NULL로 저장됨)
INSERT INTO Book(bookid, bookname, publisher)
VALUES (14, '스포츠 의학', '한솔의학서적');

-- SELECT 결과 삽입
INSERT INTO Book(bookid, bookname, price, publisher)
SELECT bookid, bookname, price, publisher FROM Imported_book;
```

### UPDATE

```sql
UPDATE 테이블이름
SET 속성이름=값[, 속성이름=값, ...]
[WHERE 조건];
```

WHERE 생략하면 모든 튜플이 수정됨.

```sql
-- custid 5번 고객 주소 변경
SET SQL_SAFE_UPDATES=0;
UPDATE Customer SET address = '대한민국 부산' WHERE custid=5;

-- 다른 테이블 값 참조해서 수정
UPDATE Book
SET publisher = (SELECT publisher FROM imported_book WHERE bookid = 21)
WHERE bookid = 14;
```

### DELETE

```sql
DELETE FROM 테이블이름
[WHERE 조건];
```

WHERE 생략하면 모든 투플 삭제, 빈 테이블이 됨.

```sql
DELETE FROM Book WHERE bookid = 11;
```
