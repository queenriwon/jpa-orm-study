# Chapter.1 JPA 소개 

> ### 📋 차례
> [1) SQL을 직접 다룰 때의 문제점](#1-sql을-직접-다룰-때의-문제점)<br>
> [2) 패러다임 불일치](#2-패러다임-불일치)<br>
> [3) JPA란?](#3-jpa란)<br>

<br>

## 1) SQL을 직접 다룰 때의 문제점

- <span style="color:red;">데이터베이스는 객체 구조와는 다른 구조를 가지므로</span> 데이터베이스에 직접 저장 및 조회가 불가능 <br>👉 변환과정을 거쳐야함 **(번거로움)**
- 테이블의 필드가 추가될 때마다 관련 쿼리 코드를 모두 수정해야함 <br>👉 객체지향적으로 설계하면 필드가 늘어나더라도 추가 코드수정 ❌

> ### 🚨 SQL를 직접 다룰때의 문제점
> 1) 진정한 의미의 계층 분할이 어렵다.
> 2) 엔티티를 신뢰할 수 없다.
> 3) SQL에 의존적인 개발을 피하기 어렵다.

👉 JPA을 사용하면 코드의 중복을 줄일 수 있고 어플리케이션을 SQL이 아닌 <span style="color:red;">객체 중심으로 개발</span>할 수 있어 **생산성** 과 **유지보수성** 이 좋고, 테스트를 작성하기도 편해짐

<br>

## 2) 패러다임 불일치

`객체`와 `관계형 데이터베이스`는 <ins>**지향하는 목적이 서로 다르므로 둘의 기능과 표현 방법도 다르다.**</ins>  ➡︎ <span style="color:#F05750;">**패러다임 불일치 문제**</span>

<br>

### 1️⃣ 상속 <br>
객체는 상속이라는 기능을 가지고 있지만 테이블은 상속이라는 기능이 없다...

상위 객체를 저장하려면 이 객체를 분해해서 하위 객체를 만들어야한다.**(코드의 양 증가)**

#### JPA의 해결

```java
jpa.persist(album);     // 1️⃣ Create

String albumId = "id100";
Album album = jpa.find(Album.class, albumId);   // 2️⃣ Read
```

#### 위 Java 코드가 SQL로는 어떤 뜻일까?
```sql
INSERT INTO ITEM ...
INSERT INTO ALBUM ...   -- 1️⃣ Create

SELECT I.*, A.*
    FROM ITEM I
    JOIN ALBUM A ON I.ITEM_ID = A.ITEM_ID       -- 2️⃣ Read
```

<br>

### 2️⃣ 연관관계 <br>
`객체`는 **참조**를 사용해서 다른 객체와 연관관계를 가지고 <span style="color:red;">참조에 접근해서 연관된 객체를 조회</span>한다.

반면, `테이블`은 **외래 키**를 사용해서 다른 테이블과 연관관계를 가지고 <span style="color:red;">조인을 사용해서 연관된 테이블을 조회</span>한다.

* `관계형 데이터베이스`는 조인이라는 기능이 있으므로, 외래키의 값을 그대로 보관해도 된다. 하지만 `객체`는 연관된 객체의 참조를 보관해야 연관된 객체를 찾을 수 있다.
<ins>**(객체는 참조 방향을 가진다.)**</ins>
* 개발자가 중간에서 **외래 키**와 **참조**의 변환 역할을 해야한다. <br> 👉 <ins>JPA가 변환을 처리해준다.</ins>

#### JPA의 해결

```java
// 0️⃣ Entity
class Member {
    String id;
    Team team;      // 참조로 연관관계를 맺는다. Team 클래스에서 id는 PK
    String username;
}

// 1️⃣ Create
member.setTeam(team);   // 회원과 팀 연관관계 설정
jpa.persist(member);    // 회원과 연관관계를 함께 저장

// 2️⃣ Read
Member member = jpa.find(Mamber.class, memberId);
Team team = member.getTeam();
```

<br>

### 3️⃣ 객체 그래프 탐색 <br>

* 객체는 마음껏 객체 그래플르 탐색할 수 있어야한다.
* SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다. <ins>(🚨엔티티가 SQL에 논리적으로 종속)</ins>

#### JPA의 해결
<span style="color:red;">**지연 로딩**</span>: 실제 객체를 사용하는 시점까지 데이터베이스 조회를 미룸 <br> 👉 연관된 객체를 신뢰하고 마음껏 조회
```java
member.getOrder().detOrderItem()...;    // 자유로운 객체 그래프 탐색
```
```java
// 처음 조회 시점에 SELECT MEMBER
Member member = jpa.find(Member.class, memberId);

Order order = member.getOrder();
order.getOrderDate();       // Order를 사용하는 시점에 SELECT ORDER
```

<br>

### 4️⃣ 비교 <br>
* **동일성 비교 (==)** : 객체 인스턴스의 주소 값을 비교
* **동등성 비교 (equals())** : 객체 내부의 값을 비교
* 객체 지향에서 동일성을 비교할 때는 False(주소값을 비교함) <br> 👉<ins>JPA는 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장</ins>

<br>

## 3) JPA란 무엇인가?
* <span style="color:red;">**JPA**</span>(Java Persistence API): 자바 진영의 ORM 기술 표준
* <span style="color:red;">**ORM**</span>(Object-Relational Mapping): 객체와 관계형 데이터베이스를 매핑


* <ins>**JPA 저장:**</ins> 1️⃣ Entity 분석, 2️⃣ INSERT SQL 생성, 3️⃣ JDBC API 사용, 4️⃣ 패러다임 불일치 해결을 하여 DB에 데이터를 반영한다.
* <ins>**JPA 조회:**</ins> 1️⃣ SELECT SQL 생성, 2️⃣ JDBC API 사용, 3️⃣ ResultSet 매핑, 4️⃣ 패러다임 불일치 해결을 하여 DB의 데이터를 조회한다.
* 매핑방법을 JPA가 해결해주면서 개발자는 데이중심인 관계형 데이터베이스를 사용해도 객체지향 애플리캐이션 개발에 집중할 수 있다.
* <span style="color:red;">**하이버네이트**</span>: 패러다임 불일치 문제를 해결해주는 성숙한 ORM 프레임워크


> ### 💡 JPA를 사용해야 하는 이유
> 1) <span style="color:orange;">**생산성**</span>: 반복적인 코드와 CRUD 쿼리를 개발자가 직접 작성하지 않아도 됨
> 2) <span style="color:orange;">**유지보수**</span>: 엔티티의 수정해도 수정할 쿼리 코드가 줄어듬 👉 유지보수해야하는 코드가 줄어든다.
> 3) <span style="color:orange;">**패러다임 불일치 해결**</span>: 상속, 연관관계, 객체 그래프 탐색, 비교하기와 같은 패러다임 불일치 해결
> 4) <span style="color:orange;">**성능**</span>: 1차 캐시 사용으로, 조회한 객체를 재사용한다.
> 5) <span style="color:orange;">**데이터 접근 수상화와 벤더 독립성**</span>: 특정 데이터 베이스 기술에 종속되지 않도록 한다.

<br>
<br>

> #### 마이바티스와 차이점
> 마이바티스나 스프링 JdbcTemplate을 SQL 매퍼라고 한다. <br>👉 개발자가 SQL을 직접 작성 <br>👉SQL에 의존하는 개발