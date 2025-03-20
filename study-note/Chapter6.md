# Chapter.6 다양한 연관관계 매핑

> ### 📋 차례
> [1) 다대일](#1-다대일) <br>
> [2) 일대다](#2-일대다) <br>
> [3) 일대일](#3-일대일) <br>
> [4) 다대다](#4-다대다) 


> ### 📢 모든 연관관계
> 1) **다대일**: 단방향, 양방향
> 2) **일대다**: 단방향, 양방향
> 3) **일대일**: 주테이블 단방향, 양방향
> 4) **일대일**: 대상테이블 단방향, 양방향
> 5) **다대다**: 단방향, 양방향

<br>

## 1) 다대일
### 1️⃣ 다대일 단방향
```java
@Entity
public class Member (

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;

    // 연관관계 매핑
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")       // TEAM_ID와 매핑
    private Team team;
)

public class Team (

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private String id;
    private String name;
)
```
* member ➡︎ team : Member.team필드로 참조 가능 ⭕️
* team ➡︎ member : 참조하는 필드 없음 ❌

👉 회원과 팀은 <ins>**다대일 단방향 연관관계**</ins> <br>
👉 `@JoinColumn(name = "TEAM_ID")`을 이용해 `Member.team`필드를 `TEAM_ID`외래 키와 매핑

<br>

### 2️⃣ 다대일 양방향
```java
// 객체 연관관계
@Entity
public class Member (

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;

    // 연관관계 매핑
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")       // TEAM_ID와 매핑
    private Team team;
    
    public void setTeam(Team team) {
        this.team = team;
        if (!team.getMembers().contains(this)) 
            team.getMembers().add(this);        // 양방향 인한 무한루프 방지
    }
)

public class Team (

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private String id;
    private String name;

    @OneToMany(mappedBy = "team")       // Member와 매핑(Member가 주인)
    private List<Member> members = new ArrayList<>();

    public void setMember(Member member) {
      this.members.add(member);
      if (member.getTeam() != this)
        member.setTeam(this);           // 양방향 인한 무한루프 방지
    }
)
```
* **양방향은 외래키가 있는 쪽(다)이 연관관계의 주인이다.**
  * member ➡︎ team : 외래 키가 있으므로 주인 설정
  * team ➡︎ member : 주인이 아니라 조회만 가능

* **양방향 연관관계는 항상 서로를 참조해야 한다.**

<br>

## 2) 일대다
### 1️⃣ 일대다 단방향
* 보통 자신이 매핑한 테이블의 외래 키를 관리하는데, 이 매핑은 반대쪽 테이블에 있는 외래 키를 관리
```java
@Entity
public class Member (

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;
)

public class Team (

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private String id;
    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")       // Member와 매핑(Team이 주인)
    private List<Member> members = new ArrayList<>();
)
```
* `@JoinColumn`을 명시하지 않으면 조인 테이블 `@JoinTable`전략 사용

<br>

#### 일대다 단방향 매핑의 단점
* 매핑한 객체가 관리하는 외래 키가 다른 테이블에 있음 <br> 👉 다른 테이블에 외래 키가 있으면 연관관계 처리를 위한 `UPDATE`SQL 추가 실행 필요
```java
team1.getMembers().add(member1);
team1.getMembers().add(member2);        // 연관관계 설정

em.persist(member1);    // INSERT member1 (team정보 null)
em.persist(member2);    // INSERT member2 (team정보 null)
em.persist(team1);      // INSERT team1, UPDATE member1.2
```
👉 먼저 member의 team정보가 nullfh insert, 추후 team을 저장하며 업데이트

**<span style="font-size:16px;"> 👉 일대다 단방향 매핑보다는 다대일 양방향 매핑 권장 </span>**

<br>

### 2️⃣ 일대다 양방향
* 일대다 양방향 매핑은 존재하지 않음. 대신 양방향 매핑 사용
* 관계형 데이터베이스 특성상 일대다, 다대일 관계는 항상 다쪽에 외래키가 있다.
* `@OneToMany`, `@ManyToOne` 둘 중에 연관관계의 주인은 항상 다 쪽인 `@ManyToOne`을 사용하는 곳

**<span style="font-size:16px;"> 👉 `@ManyToOne`에는 `mappedBy`속성이 없어 일대다 양방향은 존재하지 않음 </span>**

<br>

#### 다대일 단방향 매핑을 읽기 전용으로 구현
```java
@Entity
public class Member (

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;
    
    @ManyToOne
    @JoinColumn(name = "TEAM_ID", 
        insertable = false, updatable = false)      // 다대일을 읽기 전용으로 
    private Team team;
)

public class Team (

    @Id @GeneratedValue
    @Column(name = "TEAM_ID")
    private String id;
    private String name;

    @OneToMany
    @JoinColumn(name = "TEAM_ID")       // Member와 매핑(Team이 주인)
    private List<Member> members = new ArrayList<>();
)
```
일대다 단방향 매핑 반대편에 다대일 단방향 매핑을 읽기 전용으로 추가해서 일대다 양방향처럼 보이도록 하는 방법


<br>

## 3) 일대일
> ### 📢 일대일 연관관계의 특징
> 1) 일대일 관계는 그 반대도 일대일 관계이다
> 2) 일대일 관계는 주 테이블이나 대상 테이블 둘 중 어느곳이나 외래 키를 가질 수 있다.
> <br>
> * 주 테이블에 외래키
>   * 주 객체 ➡ 대상 객체 참조
>   * 객체 지향 개발자 선호: 외래키를 객체 참조처럼 사용
>   * **주 테이블이 외래 키를 가지고 있으므로 주 테이블만 확인해도 대상 테이블과의 관계를 알 수 있음**
> * 대상 테이블에 외래키
>   * 전통적인 데이터베이스 개발자 선호
>   * **테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 유지가능**

<br>

### 1️⃣ 주 테이블에 외래 키
#### ❶ 일대일 단방향
```java
@Entity
public class Member (

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")       // LOCKER_ID와 매핑
    private Team team;
)

public class Locker (

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private String id;
    private String name;
)
```
👉 일대일 관계이므로 객체 매핑에 `@OneToOne` 사용하고, 데이터베이스에는 `LOCKER_ID` 외래 키에 유니크 제약조건 추가 <br>
<br>

#### ❷ 일대일 양방향
```java
@Entity
public class Member (

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;

    @OneToOne
    @JoinColumn(name = "LOCKER_ID")       // LOCKER_ID와 매핑
    private Team team;
)

public class Locker (

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private String id;
    private String name;

    @OneToOne(mappedBy = "locker")          // MEMBER_ID 매핑 (Member가 주인)
    private Member member;
)
```

<br>

### 2️⃣ 대상 테이블에 외래 키
#### ❶ 일대일 단방향
* 대상 테이블에 외래 키가 있는 단방향 관계는 JPA가 지원하지 않음

<br>

#### ❷ 일대일 양방향
```java
@Entity
public class Member (

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;

    @OneToOne(mappedBy = "member")          // LOCKER_ID와 매핑 (Locker가 주인)
    private Team team;
)

public class Locker (

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private String id;
    private String name;

    @OneToOne
    @JoinColumn(name = "MEMBER_ID")         // MEMBER_ID 매핑
    private Member member;
)
```
주 테이블 외래키 일대일 양방향에서 주인만 바꾼다. <br>

<br>


## 4) 다대다
* 테이블은 2개의 테이블로 다대다 관계를 표현할 수 없다. 👉<ins>**중간에 연결테이블 추가**</ins>
* 객체의 경우 각 컬렉션 필드끼리 참조하는 방법으로 다대다 관계 구현 가능

<br>

### 1️⃣ 다대다 단방향
```java
@Entity
public class Member (

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;
    
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",     // 다대다 단방향
        joinColumns = @JoinColumn(name = "MEMBER_ID",
        inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    private List<Product> products = new ArrayList<>();     // PRODUCT_ID 와 매핑
)

public class Product (

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private String id;
    private String name;
)
```
중간 테이블 없이 `@ManyToMany`와 `@JoinTable`을 사용해서 연결 테이블 매핑
* `@JoinTable.name`: 연결할 중간테이블 지정
* `@JoinTable.joinColumns`: 현재 방향인 조인 칼럼 정보를 지정
* `@JoinTable.inverseJoinColumns`: 반대 방향인 조인 칼럼 정보를 지정

<br>

#### 저장
```java
Member member1 = new Member();
member1.setId("member1");
member1.setUsername("회원1");
member1.getProducts().add(productA);    // 연관관계 설정
em.persist(member1);
```
```sql
INSERT INTO PRODUCT ...
INSERT INTO MEMBER ...
INSERT INTO MEMBER_PRODUCT ...
```

<br>

#### 조회
```java
// 상품 저장 후
Member member = em.find(Member.class, "member1");
List<Product> products = member.getProducts();
```
```sql
SELECT * FROM MEMBER_PRODUCT MP
INNER JOIN  PRODUCT P ON MP.PRODUCT_ID = P.PRODUCT_ID
WHERE MP.MEMBER_ID = ?
```
👉 중간 연결 테이블과 `PRODUCT` 테이블 연결

<br>

### 2️⃣ 다대다 양방향
```java
@Entity
public class Member (

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;
    
    @ManyToMany
    @JoinTable(name = "MEMBER_PRODUCT",     // 다대다 단방향
        joinColumns = @JoinColumn(name = "MEMBER_ID",
        inverseJoinColumns = @JoinColumn(name = "PRODUCT_ID"))
    private List<Product> products = new ArrayList<>();     // PRODUCT_ID 와 매핑
)

public class Product (

    @Id @GeneratedValue
    @Column(name = "LOCKER_ID")
    private String id;
    private String name;
    
    @ManyToMany(mappedBy = "products")      // 역방향 추가
    private List<Member> members;
)
```

#### 저장
```java
// 편의 메서드로 연관관계 설정
public void addProduct(Product product) {
  ...
  products.add(product);
  product.getMembers().add(this);
}
```

<br>

#### 조회
```java
Product product = em.find(Product.class, "ProductA");
List<Member> members = product.getMembers();
```

### 3️⃣ 연결 엔티티 사용 - 복합키 사용
* 연결 테이블에 추가 칼럼이 필요할시 연결 엔티티를 사용한다. <br> 👉 `@ManyToMany` 사용 불가 <br> 👉 일대다, 다대일 관계로 풀기

```java
@Entity
public class Member (

    @Id
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;
    
    // 역방향
    @OneToMany(mappedBy = "member")     // MemberProduct 연결 (MemberProduct이 주인)
    private List<MemberProduct> memberProducts;
)

public class Product (

    @Id
    @Column(name = "LOCKER_ID")
    private String id;
    private String name;
)
```

```java
// 회원상품 중간 테이블
@Entity
@IdClass(MemberProductId.class)
public class MemberProduct {
    
    @Id 
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;      // Member와 MemberProductId.member 연결

    @Id
    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;      // Product와 MemberProductId.product 연결
}

// 회원상품 식별자
public class MemberProductId implements Serializable {
    
    private String member;      // MemberProduct.member와 연결
    private String product;     // MemberProduct.product와 연결
  
    // hashCode 와 equals
}
```
* 기본키와 외래키를 한번에 매핑
* `MEMBER_ID`와 `PRODUCT_ID`로 이루어진 복합 기본키 사용 👉 <ins>별도의 식별자 클래스 사용</ins>
* 식별자 클래스르 지정하기 위해 `@IdClass`사용

> ### 💡 복합키를 위한 식별자 클래스
> 1) 복합키는 별도의 식별자 클래스로 만들어야 한다.
> 2) `Serializable`을 구현해야 한다.
> 3) `equals`와 `hashCode`를 구현해야한다.
> 4) 기본 생성자가 있어야 한다.
> 5) 식별자 클래스는 public 이어야 한다.
> 6) `@IdClass`를 사용하는 방법 외에 `@EmbeddedId`를 사용할 수 있다.


> #### 💡 식별 관계
> * <ins>**식별 관계**</ins>: 부모 테이블의 기본 키를 받아서 자신의 기본키 + 외래키로 사용하는 것

<br>

#### 저장
```java
// 회원 및 상품 저장
MemberProduct memberProduct = new MemberProduct();
memberProduct.setMember(member1);       // 주문 회원 - 연관관계 설정
memberProduct.setProduct(productA);     // 주문 상품 - 연관관계 설정
memberProduct.setOrderAmount(2);        // 추가 컬럼 저장

em.persist(memberProduct);
```
<br>

#### 조회
```java
// 회원 및 상품 저장
MemberProductId memberProductId = new MemberProductId();
memberProductId.setMember("member1");
memberProductId.setProduct("productA");

MemberProduct memberProduct = em.find(MemberProduct.class, memberProductId);

Member member = memberProduct.getMember();
Product product = memberProduct.getProduct();
```

<br>

### 4️⃣ 연결 엔티티 사용 - 기본키 사용
* 데이터베이스에서 자동으로 생성해주는 대리키를 Long값으로 사용

```java
// 회원상품 중간 테이블(주문)
@Entity
public class Order {

    @Id @GeneratedValue
    @Column(name = "ORDER_ID")
    private Long id;            // 기본키 생성 전략
    
    @Id 
    @ManyToOne
    @JoinColumn(name = "MEMBER_ID")
    private Member member;      // Member와 Order.member 연결

    @Id
    @ManyToOne
    @JoinColumn(name = "PRODUCT_ID")
    private Product product;      // Product와 Order.product 연결
}

@Entity
public class Member (

    @Id @Column(name = "MEMBER_ID")
    private String id;
    private String username;

    // 역방향
    @OneToMany(mappedBy = "member")     // MemberProduct 연결 (MemberProduct이 주인)
    private List<Order> orders = new ArrayList<>();
)

public class Product (

    @Id @Column(name = "PRODUCT_ID")
    private String id;
    private String name;
)
```

<br>

#### 저장
```java
// 회원 및 상품 저장
Order order = new Order();
order.setMember(member1);       // 주문 회원 - 연관관계 설정
order.setProduct(productA);     // 주문 상품 - 연관관계 설정
order.setOrderAmount(2);        // 추가 컬럼 저장

em.persist(order);
```
<br>

#### 조회
```java
// 회원 및 상품 저장
Long orderId = 1L;
Order order = em.find(Order.class, orderId);

Member member = order.getMember();
Product product = order.getProduct();
```

> 💡 다대다 연관관계 정리
> * <ins>**식별 관계**</ins>: 받아온 식별자를 기본키 + 외래키로 사용한다.
> * <ins>**비식별 관계**</ins>: 받아온 식별자는 외래키로만 사용하고 새로운 식별자 추가