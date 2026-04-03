---
layout: post
title:  "데이터베이스 4주차 개발블로그"
date:   2026-04-03
categories: Database
---

# SQL 고급 문법 정리 — 내장함수부터 윈도우 함수까지

---

## 1. 내장함수

SQL 함수는 크게 두 가지로 나뉜다. DBMS가 기본 제공하는 **내장 함수**와, 사용자가 직접 만드는 **사용자 정의 함수**. 내장 함수는 상수나 속성 이름을 입력받아 단일 값을 반환하며, SELECT 절이나 WHERE 절, UPDATE 절 등에서 자유롭게 쓸 수 있다.

### 1-1. 숫자 함수

SQL에서는 사칙 연산자(`+`, `-`, `*`, `/`)와 나머지 연산자(`%`)를 그대로 쓸 수 있고, MySQL은 자주 쓰이는 수학 연산을 내장 함수로 제공한다.

| 함수 | 설명 | 예시 |
|---|---|---|
| `ABS(숫자)` | 절댓값 | `ABS(-4.5)` → 4.5 |
| `CEIL(숫자)` | 올림 (크거나 같은 최소 정수) | `CEIL(4.1)` → 5 |
| `FLOOR(숫자)` | 내림 (작거나 같은 최소 정수) | `FLOOR(4.1)` → 4 |
| `ROUND(숫자, m)` | 반올림, m은 자릿수 | `ROUND(5.36, 1)` → 5.4 |
| `LOG(n, 숫자)` | 자연로그 | `LOG(10)` → 2.30259 |
| `POWER(숫자, n)` | n 제곱 | `POWER(2, 3)` → 8 |
| `SQRT(숫자)` | 제곱근 | `SQRT(9.0)` → 3.0 |
| `SIGN(숫자)` | 음수 -1, 0이면 0, 양수 1 | `SIGN(3.45)` → 1 |

숫자 함수에는 직접 상수를 넣을 수도 있고, 컬럼 이름을 넣을 수도 있다. 여러 함수를 중첩해서 쓰는 것도 가능하다.

```sql
-- 고객별 평균 주문 금액을 100원 단위로 반올림
SELECT custid '고객번호', ROUND(SUM(saleprice)/COUNT(*), -2) '평균금액'
FROM   Orders
GROUP BY custid;
```

### 1-2. 문자 함수

문자 함수는 문자값을 반환하는 것과 숫자값을 반환하는 것으로 나뉜다.

**문자값 반환 함수:**

| 함수 | 설명 |
|---|---|
| `CONCAT(s1, s2)` | 두 문자열 연결. `CONCAT('마당', '서점')` → '마당서점' |
| `LOWER(s)` / `UPPER(s)` | 소문자 / 대문자 변환 |
| `LPAD(s, n, c)` / `RPAD(s, n, c)` | 왼쪽/오른쪽을 지정 문자로 채워 n자리로 만듦 |
| `REPLACE(s1, s2, s3)` | s1에서 s2를 s3로 치환 |
| `SUBSTR(s, n, k)` | s의 n번째 위치에서 k개 문자 추출 |
| `TRIM(c FROM s)` | 양쪽에서 지정 문자 제거 (기본값은 공백) |

**숫자값 반환 함수:**

| 함수 | 설명 |
|---|---|
| `ASCII(c)` | 아스키코드 값 반환 |
| `LENGTH(s)` | 바이트 수 반환. 한글은 UTF-8 기준 3바이트 |
| `CHAR_LENGTH(s)` | 문자 수 반환 |

`LENGTH`와 `CHAR_LENGTH`의 차이는 한글 처리에서 확실히 드러난다. `LENGTH("가나다abc")`는 12(한글 9 + 영문 3)를 반환하지만, `CHAR_LENGTH("가나다abc")`는 6을 반환한다.

```sql
-- 도서 제목에서 '야구'를 '농구'로 일괄 변경
SELECT bookid, REPLACE(bookname, '야구', '농구') bookname, publisher, price
FROM   Book;
```

### 1-3. 날짜·시간 함수

날짜 함수에서 핵심은 **format 지정자** 개념이다. `%Y`는 4자리 연도, `%m`은 월, `%d`는 일, `%H`는 24시간제 시, `%i`는 분, `%s`는 초를 의미한다.

| 함수 | 설명 |
|---|---|
| `STR_TO_DATE(string, format)` | 문자열 → DATE 변환 |
| `DATE_FORMAT(date, format)` | DATE → 문자열 변환 |
| `ADDDATE(date, INTERVAL n DAY)` | 날짜 더하기/빼기 |
| `DATEDIFF(date1, date2)` | 두 날짜의 차이(일 단위) |
| `SYSDATE()` | 현재 날짜/시간 반환 |

```sql
-- 주문일로부터 10일 후 확정일자 계산
SELECT orderid '주문번호', orderdate '주문일',
       ADDDATE(orderdate, INTERVAL 10 DAY) '확정'
FROM   Orders;
```

```sql
-- 2024년 7월 7일 주문 조회 (문자열을 DATE로 변환하여 비교)
SELECT orderid '주문번호', DATE_FORMAT(orderdate, '%Y-%m-%d') '주문일',
       custid '고객번호', bookid '도서번호'
FROM   Orders
WHERE  orderdate = STR_TO_DATE('20240707', '%Y%m%d');
```

**주의할 점 몇 가지:**

DBMS마다 날짜 함수의 이름과 동작이 다르다. MySQL은 `NOW()`, `CURDATE()`, `DATE_ADD()`를 쓰지만, Oracle은 `SYSDATE()`, `ADD_MONTH()`, `TO_CHAR()` 등을 쓴다. PostgreSQL은 또 `CURRENT_TIMESTAMP`, `INTERVAL` 문법이 다르다. 쿼리를 다른 DBMS로 이식할 때 이 부분을 꼭 신경 써야 한다.

또 하나, `NOW()`와 `SYSDATE()`는 비슷해 보이지만 동작이 다르다. `NOW()`는 쿼리 실행 시작 시점의 시간을 반환해서 같은 쿼리 안에서는 항상 동일한 값이 나오고, `SYSDATE()`는 함수 호출 순간의 시간을 반환해서 쿼리 도중에도 값이 달라질 수 있다.

성능 측면에서도 주의가 필요한데, WHERE나 JOIN, ORDER BY에서 컬럼에 함수를 적용하면 인덱스를 타지 못하고 풀 테이블스캔이 발생할 수 있다. `WHERE DATE(order_date) = '2026-03-28'` 대신 `WHERE order_date >= '2026-03-28 00:00:00' AND order_date < '2026-03-29 00:00:00'`처럼 범위 조건을 쓰는 게 유리하다.

---

## 2. NULL 값 처리

NULL은 '아직 지정되지 않은 값'이다. 0도 아니고, 빈 문자열(`''`)도 아니고, 공백(` `)과도 다른 특별한 값이다.

NULL의 특징을 정리하면 이렇다:

- `=`, `<`, `>` 같은 비교 연산자로 비교할 수 없다. (`IS NULL`, `IS NOT NULL`을 써야 한다)
- NULL과 어떤 숫자를 연산하면 결과도 NULL이다. (`NULL + 100` → `NULL`)
- 집계 함수(`SUM`, `AVG` 등)는 NULL이 포함된 행을 자동으로 제외하고 계산한다.
- 해당되는 행이 하나도 없으면 `SUM`과 `AVG`는 NULL을, `COUNT`는 0을 반환한다.
- `COUNT(*)`는 NULL을 포함한 전체 행 수, `COUNT(컬럼명)`은 NULL을 제외한 행 수를 센다.

```sql
-- Mybook 테이블에서 확인
SELECT SUM(price), AVG(price), COUNT(*), COUNT(price)
FROM   Mybook;
-- 결과: 30000 / 15000.0000 / 3 / 2  (bookid=3의 price가 NULL이라 COUNT(price)는 2)
```

### IFNULL과 COALESCE

NULL을 다른 값으로 대체하고 싶을 때 쓰는 함수다.

```sql
-- 전화번호가 없는 고객은 '연락처없음'으로 표시
SELECT name '이름', IFNULL(phone, '연락처없음') '전화번호'
FROM   Customer;
```

`IFNULL`은 MySQL/MariaDB 전용이고 인자를 2개만 받는다. 반면 `COALESCE`는 ANSI SQL 표준 함수로, 여러 인자를 받아서 첫 번째로 NULL이 아닌 값을 반환한다. 이식성을 생각하면 `COALESCE`를 쓰는 게 낫다.

---

## 3. 행번호 출력

MySQL에서 사용자 변수를 이용하면 결과에 행번호를 매길 수 있다. 변수 이름 앞에 `@`를 붙이고, 값 대입에는 `SET`과 `:=`을 사용한다.

```sql
SET @seq := 0;
SELECT (@seq := @seq + 1) '순번', custid, name, phone
FROM   Customer
WHERE  @seq < 2;
```

다만 단순히 상위 N개 행만 뽑는 거라면 `LIMIT`이 훨씬 간결하다.

```sql
SELECT custid, name, phone
FROM   Customer
LIMIT  2;
```

---

## 4. CASE WHEN

조건에 따라 다른 값을 반환하는 구문이다. 프로그래밍 언어의 if-then-else와 같은 역할을 하며, 집계함수와 함께 쓰거나 출력 컬럼을 가공할 때 유용하다.

```sql
CASE
    WHEN 조건1 THEN 결과1
    WHEN 조건2 THEN 결과2
    ...
    ELSE 기본값
END AS 별칭
```

```sql
-- 책 가격을 고가/중간/저가로 분류
SELECT bookname,
    CASE
        WHEN price > 20000 THEN '고가'
        WHEN price BETWEEN 10000 AND 20000 THEN '중간'
        ELSE '저가'
    END AS price_category
FROM book;
```

집계함수와 결합하면 피벗 테이블처럼 쓸 수도 있다:

```sql
-- 연도별 매출을 열로 펼치기
SELECT
    SUM(CASE WHEN year(orderdate) = 2024 THEN saleprice ELSE 0 END) AS '2024년 매출',
    SUM(CASE WHEN year(orderdate) = 2026 THEN saleprice ELSE 0 END) AS '2026년 매출'
FROM orders;
```

```sql
-- 월별로 매출을 고/중/저로 분류
SELECT
    MONTH(orderdate) AS month,
    SUM(CASE WHEN saleprice >= 20000 THEN saleprice ELSE 0 END) AS high_sales,
    SUM(CASE WHEN saleprice BETWEEN 10000 AND 19999 THEN saleprice ELSE 0 END) AS mid_sales,
    SUM(CASE WHEN saleprice < 10000 THEN saleprice ELSE 0 END) AS low_sales
FROM orders
GROUP BY MONTH(orderdate)
ORDER BY month;
```

---

## 5. 부속질의 (Subquery)

부속질의는 하나의 SQL 문 안에 다른 SQL 문이 중첩된 형태다. 메인 쿼리의 조건에 서브쿼리 결과를 가져다 쓰거나, 다른 테이블의 데이터를 참조해서 가공하는 데 쓰인다.

조인으로도 같은 결과를 낼 수 있는 경우가 많지만, 필터링 대상이 적거나 서브쿼리 결과가 유니크한 값 하나일 때, 또는 복잡한 집계가 포함될 때는 부속질의가 가독성이나 성능 면에서 유리할 수 있다.

부속질의는 위치에 따라 세 가지로 나뉜다:

| 종류 | 위치 | 특징 |
|---|---|---|
| WHERE 부속질의 (중첩질의) | WHERE 절 | 조건 필터링용. 상관/비상관 모두 가능 |
| SELECT 부속질의 (스칼라 부속질의) | SELECT 절 | 단일 행, 단일 열 값을 반환 |
| FROM 부속질의 (인라인 뷰) | FROM 절 | 가상 테이블로 사용 |

### 5-1. WHERE 부속질의

WHERE 절에서 쓰이는 부속질의로, 다양한 연산자와 함께 사용할 수 있다.

**비교 연산자 (`=`, `>`, `<` 등):** 서브쿼리가 반드시 단일 행, 단일 열을 반환해야 한다.

```sql
-- 평균 주문금액 이하인 주문 조회
SELECT orderid, saleprice
FROM   Orders
WHERE  saleprice <= (SELECT AVG(saleprice) FROM Orders);
```

**상관 부속질의:** 서브쿼리가 메인 쿼리의 컬럼을 참조하는 형태. 메인 쿼리의 각 행마다 서브쿼리가 실행된다.

```sql
-- 각 고객의 평균 주문금액보다 큰 주문만 조회
SELECT orderid, custid, saleprice
FROM   Orders od1
WHERE  saleprice > (SELECT AVG(saleprice)
                    FROM   Orders od2
                    WHERE  od1.custid = od2.custid);
```

**IN, NOT IN (집합 연산자):** 서브쿼리 결과 집합에 값이 포함되어 있는지 확인한다.

```sql
-- 대한민국에 거주하는 고객에게 판매한 도서의 총판매액
SELECT SUM(saleprice) 'total'
FROM   Orders
WHERE  custid IN (SELECT custid FROM Customer WHERE address LIKE '%대한민국%');
```

**ALL, SOME/ANY (한정 연산자):** 비교 연산자와 함께 써서 서브쿼리 결과 전체(ALL) 또는 일부(ANY)와 비교한다.

```sql
-- 3번 고객이 주문한 최고 금액보다 더 비싼 주문 조회
SELECT orderid, saleprice
FROM   Orders
WHERE  saleprice > ALL (SELECT saleprice FROM Orders WHERE custid = '3');
```

**EXISTS, NOT EXISTS (존재 연산자):** 서브쿼리의 결과가 존재하는지만 확인한다. 실제 값을 비교하는 게 아니라 행의 존재 여부만 본다.

```sql
-- 대한민국에 거주하는 고객에게 판매한 도서의 총판매액 (EXISTS 사용)
SELECT SUM(saleprice) 'total'
FROM   Orders od
WHERE  EXISTS (SELECT *
               FROM   Customer cs
               WHERE  address LIKE '%대한민국%' AND cs.custid = od.custid);
```

### 5-2. 스칼라 부속질의 (SELECT 부속질의)

SELECT 절에서 사용되는 부속질의다. 반드시 단일 행, 단일 열의 스칼라 값을 반환해야 한다. 다중 행이나 다중 열을 반환하면 에러가 나고, 결과가 없으면 NULL을 반환한다.

```sql
-- 고객별 판매액 (고객 이름을 스칼라 부속질의로 가져오기)
SELECT (SELECT name FROM Customer cs WHERE cs.custid = od.custid) 'name',
       SUM(saleprice) 'total'
FROM   Orders od
GROUP BY od.custid;
```

스칼라 부속질의는 SELECT뿐 아니라 UPDATE에서도 활용 가능하다:

```sql
-- Orders 테이블에 도서 이름을 일괄 입력
ALTER TABLE Orders ADD bname VARCHAR(40);
UPDATE Orders
SET    bname = (SELECT bookname FROM Book WHERE Book.bookid = Orders.bookid);
```

### 5-3. 인라인 뷰 (FROM 부속질의)

FROM 절에서 사용되는 부속질의로, 기존 테이블로부터 일시적으로 만든 가상 테이블(뷰) 역할을 한다. 필요한 데이터를 먼저 걸러놓고 조인하면 처리 성능을 높일 수 있다.

```sql
-- 고객번호 2 이하인 고객의 판매액
SELECT cs.name, SUM(od.saleprice) 'total'
FROM   (SELECT custid, name
        FROM   Customer
        WHERE  custid <= 2) cs,
       Orders od
WHERE  cs.custid = od.custid
GROUP BY cs.name;
```

---

## 6. CTE (Common Table Expression)

CTE는 `WITH` 절을 사용해서 복잡한 쿼리의 중간 결과를 임시 테이블로 정의하는 문법이다. 인라인 뷰와 비슷한 역할을 하지만, 가독성이 훨씬 좋고 같은 결과 집합을 여러 번 참조할 수 있다는 장점이 있다. 쿼리 실행 시에만 존재하며 데이터베이스에 영구 저장되지 않는다.

```sql
-- 인라인 뷰 방식
SELECT sub.custid, sub.name, sub.bookname, sub.saleprice
FROM (
    SELECT o.custid, c.name, b.bookname, o.saleprice
    FROM   orders o
    JOIN   customer c ON o.custid = c.custid
    JOIN   book b ON o.bookid = b.bookid
) sub
WHERE sub.saleprice > 10000;

-- 같은 쿼리를 CTE로 변환
WITH order_details AS (
    SELECT o.custid, c.name, b.bookname, o.saleprice
    FROM   orders o
    JOIN   customer c ON o.custid = c.custid
    JOIN   book b ON o.bookid = b.bookid
)
SELECT custid, name, bookname, saleprice
FROM   order_details
WHERE  saleprice > 10000;
```

CTE의 진짜 강점은 한 번 정의한 임시 테이블을 여러 맥락에서 재사용할 수 있다는 점이다:

```sql
WITH sales_summary AS (
    SELECT od.custid, cs.name,
           SUM(od.saleprice) AS total_sales
    FROM   orders od
    JOIN   customer cs ON od.custid = cs.custid
    GROUP BY od.custid, cs.name
)
-- 1) 매출 내림차순 정렬
SELECT custid, name, total_sales
FROM   sales_summary
ORDER BY total_sales DESC;

-- 2) 평균 매출보다 높은 고객만 조회
SELECT custid, name, total_sales
FROM   sales_summary
WHERE  total_sales > (SELECT AVG(total_sales) FROM sales_summary)
ORDER BY total_sales DESC;
```

---

## 7. 윈도우 함수 (Window Functions)

윈도우 함수는 SQL에서 데이터 분석을 위한 "행 간 비교와 누적 계산 도구"다. GROUP BY와 결정적으로 다른 점이 있는데, GROUP BY는 그룹별로 행을 합쳐서 결과 행 수가 줄어드는 반면, 윈도우 함수는 원본 행을 그대로 유지하면서 그룹별 계산 결과를 각 행에 붙여준다.

복잡한 조인 없이도 행 간 계산, 순위 매기기, 누적합계 등을 처리할 수 있어서 실무에서 굉장히 많이 쓰인다.

### 기본 문법

```sql
SELECT 함수명() OVER (PARTITION BY 컬럼명 ORDER BY 컬럼명)
FROM   테이블명;
```

- **PARTITION BY**: 계산을 수행할 그룹을 나눈다. (GROUP BY의 그룹 개념과 유사)
- **ORDER BY**: 그 그룹 안에서 계산할 순서를 정한다.

### 주요 윈도우 함수 유형

- **순위 함수**: `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()` 등
- **집계 함수**: `SUM()`, `AVG()`, `COUNT()`, `MAX()`, `MIN()` 등 (OVER 절과 함께 쓰면 윈도우 함수로 동작)
- **분석/값 함수**: `LEAD()`, `LAG()`, `FIRST_VALUE()`, `LAST_VALUE()` 등

### 실행 순서

윈도우 함수가 포함된 쿼리의 실행 순서는 다음과 같다:

**JOIN → GROUP BY → 집계 → 윈도우 함수 → SELECT → ORDER BY**

이 순서를 모르면 윈도우 함수가 어떤 데이터를 대상으로 계산하는지 헷갈릴 수 있으니 꼭 기억해두자.

### 예제: 출판사별 매출 순위

```sql
SELECT b.publisher, b.bookname,
       SUM(o.saleprice) AS total_sales,
       RANK() OVER (PARTITION BY b.publisher ORDER BY SUM(o.saleprice) DESC)
           AS rank_in_publisher
FROM   orders o
JOIN   book b ON o.bookid = b.bookid
GROUP BY b.publisher, b.bookname
ORDER BY b.publisher, rank_in_publisher;
```

이 쿼리는 출판사별로 도서 매출 순위를 매긴다. `PARTITION BY b.publisher`로 출판사 단위 그룹을 나누고, `ORDER BY SUM(o.saleprice) DESC`로 매출 높은 순서대로 순위를 붙인다. GROUP BY와 비슷하지만 원본 행이 줄어들지 않는다는 점이 핵심이다.

### 같은 결과, 네 가지 방법 비교

고객별 총 매출을 집계하고 순위를 매기는 작업을 GROUP BY, 인라인 뷰, CTE, 윈도우 함수 네 가지 방법으로 비교해보면 윈도우 함수의 장점이 확실히 드러난다.

**1) GROUP BY 기본 집계:**
```sql
SELECT cs.custid, cs.name, SUM(od.saleprice) AS total_sales
FROM   orders od
JOIN   customer cs ON od.custid = cs.custid
GROUP BY cs.custid, cs.name
ORDER BY total_sales DESC;
```

**2) 인라인 뷰 (서브쿼리로 순위 계산):**
```sql
SELECT sub.custid, sub.name, sub.total_sales
FROM (
    SELECT cs.custid, cs.name, SUM(od.saleprice) AS total_sales
    FROM   orders od
    JOIN   customer cs ON od.custid = cs.custid
    GROUP BY cs.custid, cs.name
) sub
ORDER BY sub.total_sales DESC;
```

**3) CTE로 가독성 향상:**
```sql
WITH sales_summary AS (
    SELECT cs.custid, cs.name, SUM(od.saleprice) AS total_sales
    FROM   orders od
    JOIN   customer cs ON od.custid = cs.custid
    GROUP BY cs.custid, cs.name
)
SELECT custid, name, total_sales
FROM   sales_summary
ORDER BY total_sales DESC;
```

**4) 윈도우 함수로 집계와 순위를 동시에:**
```sql
SELECT cs.custid, cs.name, SUM(od.saleprice) AS total_sales,
       RANK() OVER (ORDER BY SUM(od.saleprice) DESC) AS sales_rank
FROM   orders od
JOIN   customer cs ON od.custid = cs.custid
GROUP BY cs.custid, cs.name;
```

윈도우 함수를 쓰면 별도의 서브쿼리나 CTE 없이 한 번의 SELECT에서 집계와 순위를 동시에 처리할 수 있다. 쿼리가 짧아지는 건 덤이고, 실행 계획상으로도 효율적인 경우가 많다.

---

## 8. 프로그래머스 SQL 문제 풀이

### 8-1. 중고거래 게시물 상태 분류 (CASE WHEN)

```sql
SELECT
    BOARD_ID,
    WRITER_ID,
    TITLE,
    PRICE,
    CASE
        WHEN STATUS = 'SALE' THEN '판매중'
        WHEN STATUS = 'RESERVED' THEN '예약중'
        WHEN STATUS = 'DONE' THEN '거래완료'
    END AS STATUS
FROM USED_GOODS_BOARD
WHERE DATE_FORMAT(CREATED_DATE, '%Y-%m-%d') = '2022-10-05'
ORDER BY BOARD_ID DESC;
```

- `CASE WHEN`으로 STATUS 코드를 한글 명칭으로 변환
- `DATE_FORMAT`으로 작성일자를 포맷 변환하여 특정 날짜 필터링

### 8-2. 게시글과 댓글 데이터 결합 (JOIN & 필터링)

```sql
SELECT
    B.TITLE,
    B.BOARD_ID,
    R.REPLY_ID,
    R.WRITER_ID,
    R.CONTENTS,
    DATE_FORMAT(R.CREATED_DATE, '%Y-%m-%d') AS CREATED_DATE
FROM USED_GOODS_BOARD B
JOIN USED_GOODS_REPLY R ON B.BOARD_ID = R.BOARD_ID
WHERE B.CREATED_DATE BETWEEN '2022-10-01' AND '2022-10-31'
ORDER BY R.CREATED_DATE ASC, B.TITLE ASC;
```

- 게시글 테이블(B)과 댓글 테이블(R)을 `JOIN`으로 연결
- `BETWEEN`으로 2022년 10월 한 달 동안 작성된 게시글만 필터링
- 정렬 기준: 1순위 댓글 작성일 오름차순, 2순위 게시글 제목 오름차순

### 8-3. 카테고리별 도서 및 저자 정보 (JOIN)

```sql
SELECT
    B.BOOK_ID,
    A.AUTHOR_NAME,
    DATE_FORMAT(B.PUBLISHED_DATE, '%Y-%m-%d') AS PUBLISHED_DATE
FROM BOOK B
JOIN AUTHOR A ON B.AUTHOR_ID = A.AUTHOR_ID
WHERE B.CATEGORY = '경제'
ORDER BY B.PUBLISHED_DATE ASC;
```

- 도서 테이블과 저자 테이블을 저자 ID 기준으로 조인
- 카테고리가 '경제'인 도서만 추출 후 출판일 오름차순 정렬

### 8-4. 카테고리별 상품 개수 구하기 (문자열 자르기 & GROUP BY)

```sql
SELECT
    LEFT(PRODUCT_CODE, 2) AS CATEGORY,
    COUNT(*) AS PRODUCTS
FROM PRODUCT
GROUP BY CATEGORY
ORDER BY CATEGORY ASC;
```

- `LEFT(PRODUCT_CODE, 2)`로 상품코드 앞 2자리를 카테고리 코드로 추출
- `GROUP BY`로 카테고리별 묶고 `COUNT(*)`로 개수 집계

### 8-5. 식품 출고 여부 판별 (날짜 비교 & NULL 처리)

```sql
SELECT
    ORDER_ID,
    PRODUCT_ID,
    DATE_FORMAT(OUT_DATE, '%Y-%m-%d') AS OUT_DATE,
    CASE
        WHEN OUT_DATE <= '2022-05-01' THEN '출고완료'
        WHEN OUT_DATE > '2022-05-01' THEN '출고대기'
        WHEN OUT_DATE IS NULL THEN '출고미정'
    END AS 출고여부
FROM FOOD_ORDER
ORDER BY ORDER_ID ASC;
```

- `CASE WHEN`으로 출고일 기준 상태를 분기 처리
- `OUT_DATE IS NULL`인 경우를 별도로 처리하여 '출고미정' 표시