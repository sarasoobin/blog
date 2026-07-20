---
layout: post
title:  "데이터베이스 5주차 개발블로그"
date:   2026-04-11
categories: Database
---

# 데이터베이스 이론 정리 (SQL 고급 + DB 프로그래밍)

이번 주 수업 내용이 꽤 많았는데 한번에 정리해보려고 한다.
뷰, 인덱스, 저장 프로그램(프로시저/트리거/함수), 거기다 JDBC에 SOLID·DAO·DTO까지.

---

## 뷰 (View)

뷰는 한마디로 **가상 테이블**이다.
실제 데이터를 저장하는 게 아니라, SELECT 쿼리를 저장해두고 테이블처럼 쓰는 것.

```sql
CREATE VIEW vw_Book AS
SELECT * FROM Book
WHERE bookname LIKE '%축구%';

SELECT * FROM vw_Book;  -- 일반 테이블처럼 조회
```

수정할 때는 `CREATE OR REPLACE VIEW`, 삭제는 `DROP VIEW`.

왜 쓰냐면 세 가지 이유가 있다.

- **편리성** : 복잡한 조인 쿼리를 뷰로 만들어두면 매번 다시 쓸 필요가 없다
- **보안성** : 주민번호 같은 민감한 컬럼을 뺀 뷰만 특정 사용자한테 주면 된다
- **독립성** : 기본 테이블 구조가 바뀌어도 뷰를 통한 인터페이스는 유지된다

뷰에 INSERT/UPDATE도 되긴 하는데, PK가 없거나 집계함수·GROUP BY가 들어간 뷰는 변경이 안 된다. 실무에서는 그냥 조회 전용으로만 쓰는 게 대부분이라고 한다.

---

## 인덱스 (Index)

인덱스는 **데이터를 빠르게 찾기 위한 자료구조**다. 책의 색인이랑 같은 개념.

### B-tree 기반이 공통 원리

대부분의 DBMS는 **B-tree(Balanced Tree) 기반 자료구조를 기본 인덱스로 채택**하고 있다. 단, 각 DBMS마다 고유의 자료구조와 기법도 따로 보유하고 있어서 완전히 동일하진 않다.

실제로 쓰이는 건 B+Tree인데, 리프 노드끼리 연결 리스트로 이어져 있어서 범위 검색이 빠르다. 100만 건 데이터도 3~4번 I/O면 찾을 수 있다고 한다.

### MySQL 인덱스 두종류

**클러스터 인덱스**는 PK 만들 때 자동으로 생성되고, 데이터 자체가 인덱스 순서대로 물리적으로 저장된다. 테이블당 하나만 가능.

**보조 인덱스**는 PK 이외의 컬럼에 만드는 인덱스. 리프 노드에 실제 데이터가 아니라 PK 값을 저장해두고, 거기서 다시 클러스터 인덱스로 실제 데이터를 찾아가는 2단계 구조다.

```sql
CREATE INDEX ix_Book ON Book(bookname);

SHOW INDEX FROM Book;          -- 인덱스 확인
EXPLAIN SELECT * FROM Book WHERE publisher = '굿스포츠';  -- 실행계획 확인
ANALYZE TABLE Book;            -- 인덱스 재구성
DROP INDEX ix_Book ON Book;    -- 삭제
```

인덱스는 많다고 좋은 게 아니다. WHERE나 JOIN에 자주 쓰는 컬럼에만 만들고, 테이블당 4~5개 정도가 권장이다. 컬럼에 함수를 씌우면 인덱스가 무용지물이 되니까 WHERE 절에서는 컬럼을 가공하지 않는 게 좋다.

---

## 저장 프로그램

### 저장 프로시저

여러 SQL문을 묶어서 DBMS에 저장해두고 `CALL`로 실행하는 것.
반복되는 업무 로직을 캡슐화할 때 쓴다.

현업에서는 쇼핑몰 주문 처리처럼 재고 차감 + 결제 기록 + 배송 업데이트를 한 번에 처리해야 할 때 프로시저로 묶어두고 `CALL processOrder(주문번호)` 이런 식으로 쓴다고 한다.

```sql
DELIMITER //
CREATE PROCEDURE getAge(
    IN p_id INT,
    OUT p_name VARCHAR(50),
    OUT p_age INT
)
BEGIN
    SELECT 이름, 나이 INTO p_name, p_age
    FROM person WHERE id = p_id;
END //
DELIMITER ;

CALL getAge(2, @name, @age);
SELECT @name, @age;
```

복잡한 비즈니스 로직을 DB에 다 몰아넣으면 나중에 유지보수가 힘들어지니까, 단순하고 반복적인 업무 위주로 사용하는 게 좋다고 한다.

---

### 트리거

INSERT, UPDATE, DELETE가 발생할 때 **자동으로 실행**되는 프로그램.
직접 호출하는 게 아니라 이벤트가 생기면 DBMS가 알아서 실행한다.

현업에서는 은행 잔액 음수 방지, 중요 데이터 변경 이력 자동 로깅, 재고 자동 차감 같은 데 쓰인다.

```sql
DELIMITER //
CREATE TRIGGER checkBalance
BEFORE UPDATE ON Accounts
FOR EACH ROW
BEGIN
    IF NEW.balance < 0 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = '잔액 부족으로 인출 불가!';
    END IF;
END //
DELIMITER ;
```

자동 실행이라서 편리하긴 한데, 디버깅이 어렵고 "어디서 이게 실행되는 거지?" 하는 상황이 생기기 쉽다. 트랜잭션 롤백 시 같이 취소된다는 것도 알아두자.

---

### 사용자 정의 함수

직접 만들어서 SELECT 문 안에서 바로 호출할 수 있는 함수다.

생년월일로 나이 계산하거나, 이름 마스킹처리, 환율 변환 같이 자주 쓰이는 계산 로직을 함수로 만들어두면 편하다.

```sql
DELIMITER $$
CREATE FUNCTION fnGetAge(birthdate DATE)
RETURNS INT
DETERMINISTIC
BEGIN
    RETURN TIMESTAMPDIFF(YEAR, birthdate, CURDATE());
END$$
DELIMITER ;

SELECT 이름, fnGetAge(생년월일) AS 나이 FROM Members;
```

프로시저랑 다른 점은 CALL이 아니라 SELECT 안에서 바로 쓸 수 있다는 거고, 단일 값을 반환한다는 것. WHERE에서 컬럼에 함수 씌우면 인덱스 무효화되니까 그 점은 주의해야 한다.

---

## JDBC 기본

JDBC는 Java에서 DB에 접속해서 SQL을 실행하는 표준 API다. `java.sql` 패키지를 쓴다.

핵심 클래스는 몇 개 안 된다. `DriverManager`로 연결 만들고, `PreparedStatement`로 SQL 실행하고, `ResultSet`으로 결과 받아오는 게 전부다.

`Statement`도 있는데 실무에서는 `PreparedStatement`를 훨씬 더 많이 쓴다. SQL Injection 방지되고, `?`로 파라미터를 깔끔하게 쓸 수 있어서.

```java
String url = "jdbc:mysql://localhost:3306/madangdb";

// try-with-resources 쓰면 close() 자동 호출됨
try (Connection con = DriverManager.getConnection(url, "root", "1234");
     PreparedStatement ps = con.prepareStatement(
         "SELECT * FROM Book WHERE publisher = ?")) {

    ps.setString(1, "굿스포츠");
    ResultSet rs = ps.executeQuery();

    while (rs.next()) {
        System.out.println(rs.getString("bookname"));
    }

} catch (SQLException e) {
    e.printStackTrace();
}
```

INSERT는 `executeUpdate()`, SELECT는 `executeQuery()`. 자원 반납은 `try-with-resources`가 요즘 표준이다.

---

## DAO / DTO

JDBC 코드를 그냥 막 짜면 SQL이 여기저기 흩어지고 나중에 수정할 때 힘들어진다. 그래서 역할을 나누는 패턴을 쓴다.

**DAO(Data Access Object)** 는 DB 접근 로직만 담당하는 클래스다. SQL 실행 코드를 여기에만 모아두고, 비즈니스 로직이랑 완전히 분리한다.

**DTO(Data Transfer Object)** 는 계층 간에 데이터를 넘길 때 쓰는 순수 데이터 클래스다. 로직 없이 필드 + getter/setter만 있다.

```java
// DTO
public class BookDTO {
    private int bookId;
    private String bookName;
    private int price;
    // getter / setter ...
}

// DAO
public class BookDAO {
    public List<BookDTO> findAll() throws SQLException {
        List<BookDTO> list = new ArrayList<>();
        try (Connection con = DriverManager.getConnection(url, "root", "1234");
             PreparedStatement ps = con.prepareStatement("SELECT * FROM Book");
             ResultSet rs = ps.executeQuery()) {
            while (rs.next()) {
                BookDTO book = new BookDTO();
                book.setBookId(rs.getInt("bookid"));
                book.setBookName(rs.getString("bookname"));
                list.add(book);
            }
        }
        return list;
    }
}
```

데이터 흐름은 `DB → DAO → Service → Controller → View` 이 순서다.

---

## SOLID

객체지향 설계 5원칙이다. DAO/Service 같은 구조를 설계할 때 이게 근거가 된다.

**S - 단일 책임 원칙** : 클래스는 책임을 하나만 가져야 한다. BookDAO는 DB 접근만, BookService는 비즈니스 로직만.

**O - 개방/폐쇄 원칙** : 확장엔 열려 있고 수정엔 닫혀 있어야 한다. 인터페이스 기반으로 DAO를 만들면 구현체(MySQL→Oracle)를 바꿔도 기존 코드를 건드릴 필요가 없다.

**L - 리스코프 치환 원칙** : 하위 클래스는 상위 클래스를 대체할 수 있어야 한다. IBookDAO 인터페이스의 구현체끼리 자유롭게 교체 가능해야 한다는 것.

**I - 인터페이스 분리 원칙** : 사용하지 않는 메소드에 의존하지 않아야 한다. 읽기 전용 DAO랑 쓰기 DAO를 분리하는 것처럼.

**D - 의존성 역전 원칙** : 구현체가 아닌 인터페이스에 의존해야 한다. Service가 `BookDAOImpl`이 아니라 `IBookDAO`에 의존하도록. Spring의 DI(@Autowired)가 이 원칙을 지원하는 구조다.

---
