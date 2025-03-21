# Chapter.7 고급 매핑

> ### 📋 차례
> [1) 상속 관계 매핑](#1-상속-관계-매핑) <br>
> [2) @MappedSuperclass](#2-@MappedSuperclass) <br>
> [3) 복합 키와 식별 관계 매핑](#3-복합-키와-식별-관계-매핑) <br>
> [4) 조인 테이블](#4-조인-테이블) <br>
> [5) 엔티티 하나에 여러 테이블 매핑](#5-엔티티-하나에-여러-테이블-매핑) <br>


<br>

## 1) 상속 관계 매핑
* `관계형 데이터베이스`에는 상속의 개념이 없는 대신 `슈퍼타입 서브타입 관계`모델링 기법과 유사
> ### 💡 `슈퍼타입 서브타입 관계` 구현 방법
> 1. <ins>**조인 전략**</ins>: 각각을 테이블로 만들고 조회할 때 조인
> 2. <ins>**단일 테이블 전략**</ins>: 테이블을 하나만 사용해서 통합
> 3. <ins>**구현 클래스마다 테이블 전략**</ins>: 서브 타입마다 하나의 테이블

<br>

### 1️⃣ 조인 전략
* 엔티티 각각을 모두 테이블로 만들고 <br> 자식 테이블이 부모 테이블의 기본 키를 받아서 **기본 키 + 외래 키**로 사용
* 테이블에는 타입의 개념이 없으므로 타입을 구분하는 칼럼`DTYPE` 사용
```java
// 조인 전략 매핑
@Entity
@Inheritance(strategy = InheritanceType.JOINED)     // 조인 전략
@DiscriminatorColumn(name = "DTYPE")            // DTYPE 칼럼 사용
public abstract class Item (                    // 추상 클래스

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private String id;
    private String name;
    private int price;
)

@Entity
@DiscriminatorValue("B")                          // 구분 칼럼(DTYPE)에 입력할 타입 값
//@PrimaryKeyJoinColumn(name = "BOOK_ID")         // 기본키 칼럼명 수정
public class Book extends Item {
    private String author;
    private String isbn;
}
```

<br>

### 2️⃣ 단일 테이블 전략
* 테이블을 하나만 사요하여 구분 칼럼`DTYPE`으로 자식데이터 구분 가능 <br>**조회시 조인을 사용하지 않음**
* `null`을 허용
```java
// 단일 테이블 전략 (InheritanceType 설정만 다름)
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)     // 단일 테이블 전략
@DiscriminatorColumn(name = "DTYPE")            // DTYPE 칼럼 사용
public abstract class Item (                    // 추상 클래스

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private String id;
    private String name;
    private int price;
)

@Entity
@DiscriminatorValue("B")                          // 구분 칼럼(DTYPE)에 입력할 타입 값(필수)
public class Book extends Item {
    ...
}
```

<br>

### 3️⃣ 구현 클래스마다 테이블 전략
* 자식 엔티티마다 테이블 생성 + 구분 칼럼 없음
* 추천하지 않음
```java
// 구현 클래스마다 테이블 전략
@Entity
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)     // 구현 클래스 테이블 전략
public abstract class Item (                    // 추상 클래스

    @Id @GeneratedValue
    @Column(name = "ITEM_ID")
    private String id;
    private String name;
    private int price;
)

@Entity         // @DiscriminatorValue 생략
public class Book extends Item {
    ...
}
```

<br>

### 💡 상속 관계 매핑 정리
|              | 조인 전략                                                                    | 단일 테이블 전략                                                                    | 구현 클래스마다 테이블 전략                                       |
|--------------|:-------------------------------------------------------------------------|------------------------------------------------------------------------------|-------------------------------------------------------|
| @Inheritance | InheritanceType.JOIN                                                     | InheritanceType.SINGLE_TABLE                                                 | InheritanceType.TABLE_PER_CLASS                       |
| 설명           | 엔티티를 모두 테이블 + 자식 테이블이 부모의 기본 키를 받음                                       | 테이블 하나 사용 + 구분 칼럼`DTYPE`으로 자식데이터 구분                                          | 자식 엔티티마다 테이블 생성 + 구분 칼럼 없음(권장하지 않음)                   |
| DTYPE 유무     | ⭕️                                                                       | ⭕️                                                                           | ❌                                                     |
| null 유무      | ❌                                                                        | ⭕️                                                                           | ❌                                                     |
| 장점           | - 테이블 정규화 <br> - 외래 키 참조 무결성 제약조건(null 값이 없음) <br> - 저장공간 효율             | - 조인이 필요 없음 <br> - 조회 쿼리 단순                                                  | - 서브 타입을 구분해서 처리 <br> - `not null` 제약 조건 사용 가능        |
| 단점           | - 조회할 떄 조인사용 👉 성능 저하 <br> - 조회 쿼리 복잡 <br> - 데이터 저장시 `INSERT` SQL을 2번 수행 | - 자식 엔티티가 매핑한 칼럼은 모두 `null`을 허용 <br> - 단일 테이블에 모두 저장 👉 테이블 커짐 (상황에 따른 성능저하) | - 여러 자식 테이블을 함께 조회할 때 성능 저하 <br> -자식 테이블을 통해 쿼리하기 어려움 |




## 2) @MappedSuperclass
* 테이블과 매핑하지 않고 부모 클래스를 상속 받는 자식에게 매핑 정보만 제공 **(매핑 정보를 상속할 목적)**
```java
@MappedSuperclass   // 추상메서드를 공통속성으로 설정 
public abstract class BaseEntity (      // 직접 생성하지 않으니 추상클래스 사용

    @Id @GeneratedValue
    private String id;
    private String name;
)

//@AttributeOverride({
//        @AttributeOverride(name = "id", column = @Column(name = "MEMBER_ID")),
//        @AttributeOverrice(name = "name", column = @Column(name = "MEMBER_NAME"))
//})            //  매핑정보 정의 가능
@Entity
public class Member extends BaseEntity (
    ...         // BaseEntity의 id, name 상속
)
```

<br>

## 3) 복합 키와 식별 관계 매핑
> ### 💡 식별 관계 VS 비식별 관계
> 1) **식별 관계**: 부모 테이블의 기본키를 받아 자식 테이블의 **기본 키 + 외래 키**
> 2) **비식별 관계**: 부모 테이블의 기본키를 받아 자식 테이블의 **외래 키**로만 사용
>     * **필수적 비식별 관계**: 외래 키 null ❌, 연관관계 필수
>     * **선택적 비식별 관계**: 외래 키 null ⭕️, 연관관계 선택

<br>

### 1️⃣ 복합 키: 비식별 관계 매핑
* 식별자 필드가 2개 이상일시 식별자 클래스 + equals, hashCode 구현
* 방법
  * `@IdClass`: 관계형 데이터베이스 방법
  * `@EmbeddedId`: 객체지향 방법

<br>

#### ❶ 복합 키 `@IdClass`
```java
@Entity
@IdClass(ParentId.class)            // ✔️복합키 식별자 클래스 명시
public class Parent {
    @Id 
    @Column(name = "PARENT_ID1")
    private String id1;             // ParentId.id1 와 연결

    @Id
    @Column(name = "PARENT_ID2")
    private String id2;             // ParentId.id2 와 연결
}

// 복합키 클래스
public class ParentId implements Serializable {
    private String id1;             // Parent.id1 와 매핑
    private String id2;             // Parent.id2 와 매핑
    
    // 생성자, equals, hashCode 메서드
}
```
`@IdClass`를 사용할 때 식별자 클래스 조건
* 식별자 클래스 속성명 = 엔티티 식별자 속성명
* `Serializable` 인터페이스 구현
* 기본생성자, `equals`, `hashCode` 구현
* `public`으로 구현

<br>

```java
// 자식 클래스
@Entity
public class Child {
    @Id 
    private String id;
    
    @ManyToOne
    @JoinColumn({           // 외래키 컬럼을 @JoinColumn으로 매핑
            @JoinColumn(name = "PARENT_ID1", referenceColumnName = "PARENT_ID1"),
            @JoinColumn(name = "PARENT_ID2", referenceColumnName = "PARENT_ID2")
    })
    private Parent parent;
}
```

<br>

#### ❷ 복합 키 `@EnbeddedId`
* 저장할 때 식별자 클래스 `parentId`를 직접 생성해서 사용 (`@IdClass`와 다른점)
```java
@Entity
public class Parent {
    @EmbeddedId             // 복합 키를 필드에 선언
    private String id;
}

// 복합키 클래스
@Embeddedable               // 식별자 클래스 선언
public class ParentId implements Serializable {
    @Column(name = "PARENT_ID1")
    private String id1;
    @Column(name = "PARENT_ID2")
    private String id2;
    
    // 생성자, equals, hashCode 메서드
}
```
`@Embeddedable`를 사용할 때 식별자 클래스 조건
* `Serializable` 인터페이스 구현
* 기본생성자, `equals`, `hashCode` 구현
* `public`으로 구현

<br>

> ### 📢 `@IdClass`와 `@EmbeddedId`의 차이 (조회)
> * `@EmbeddedId`가 객체 지향적이고 코드 중복이 적다.
> ```java
> em.createQuery("select p.id1, p.id2 from Parent p");          // @IdClass
> em.createQuery("select p.id.id1, p.id.id2 from Parent p");    // @EmbeddedId
> ```
> 

> ### 📢 복합 키와 `equals()`, `hashCode()`
> 복합 키는 `equals()`, `hashCode()`를 필수로 구현 <br>
> 👉 <ins>인스턴스의 동일성을 보장하기 위해</ins>


<br>

### 2️⃣ 복합 키: 식별 관계 매핑
* 자식 테이블이 부모 테이블의 기본 키를 포함하여 복합 키 구성
* 방법
  * `@IdClass`: 관계형 데이터베이스 방법
  * `@EmbeddedId`: 객체지향 방법

<br>

<details>
<summary> ❶ 복합 키 @IdClass</summary>
<div markdown="1">

```java
// 부모 엔티티 클래스
@Entity
public class Parent {
    @Id
    @Column(name = "PARENT_ID1")
    private String id1;             // ParentId.id1 와 연결
    ...
}

// 자식 엔티티 클래스
@Entity
@IdClass(ChildId.class)             // 자식 복합키 클래스 명시
public class Child {
    @Id 
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    public Parent parent;           // ChildId.parent 와 매핑

    @Id @JoinColumn(name = "CHILD_ID")
    public String childId;          // ChildId.childId 와 매핑
}

// 자식 복합키 클래스
public class ChildId implements Serializable {
    private String parent;          // Child.parent 와 매핑
    private String childId;         // Child.childId 와 매핑
    // 생성자, equals, hashCode 메서드
}

// 손자 엔티티 클래스
@Entity
@IdClass(GrandChildId.class)        // 자식 복합키 클래스 명시
public class GrandChild {
    @Id
    @ManyToOne                      // 기본키와 외래키 매핑
    @JoinColumns({                   // PARENT_ID와 CHILD_ID 복합 키
            @JoinColumn(name = "PARENT_ID"),
            @JoinColumn(name = "CHILD_ID")
    })
    public Child child;             // GrandChildId.child 와 매핑
  
    @Id @JoinColumn(name = "GRANDCHILD_ID")
    public String id;               // GrandChildId.id 와 매핑
}

// 손자 복합키 클래스
public class GrandChildId implements Serializable {
    private ChildId child;          // GrandChild.child 와 매핑
    private String id;              // GrandChild.id 와 매핑
    // 생성자, equals, hashCode 메서드
}
```
</div>
</details>

<details>
<summary> ❷ 복합 키 @EmbeddedId</summary>
<div markdown="1">

```java
// 부모 엔티티 클래스
@Entity
public class Parent {
    @Id
    @Column(name = "PARENT_ID")
    private String id;               // ParentId.id 와 연결
    ...
}

// 자식 엔티티 클래스
@Entity
public class Child {
    @MapsId("parentId")
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    public Parent parent;           // ChildId.parentId 와 매핑

    @EmbeddedId
    public String childId;          // ChildId.childId 와 매핑
}

// 자식 복합키 클래스
@Embeddedable
public class ChildId implements Serializable {
    private String parentId;        // @MapsId("parentId")로 Child.parent 와 매핑
  
    @Column(name = "CHILD_ID")
    private String childId;         // Child.childId 와 매핑
    // 생성자, equals, hashCode 메서드
}

// 손자 엔티티 클래스
@Entity
public class GrandChild {
    @MapsId("childId")
    @ManyToOne
    @JoinColumns({
            @JoinColumn(name = "PARENT_ID"),
            @JoinColumn(name = "CHILD_ID")
    })
    public Child child;             // GrandChildId.childId 와 매핑

    @EmbeddedId   
    public String id;               // GrandChildId.id 와 매핑
}

// 손자 복합키 클래스
public class GrandChildId implements Serializable {
    private ChildId child;          // @MapsId("childId")로 GrandChild.child 와 매핑
  
    @Column(name = "GRANDCHILD_ID")
    private String id;              // GrandChild.id 와 매핑
    // 생성자, equals, hashCode 메서드
}
```
* `@IdClass`와 다른점: `@MapsId`사용
  * `@MapsId`: 외래 키와 매핑한 연관관계를 기본 키에도 매핑
  * `@MapsId`의 속성 값은 `@EmbeddedId`를 사용한 식별자 클래스의 기본 키 필드 지정
</div>
</details>

<br>

### 3️⃣ 복합 키: 식별 관계를 비식별 관계로 구현
```java
// 부모 엔티티 클래스
@Entity
public class Parent {
    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private Long id;
}

// 자식 엔티티 클래스
@Entity
public class Child {
    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private Long id;
    
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    private Parent parent;
}

// 손자 엔티티 클래스
@Entity
public class GrandChild {
    @Id @GeneratedValue
    @Column(name = "GRANDCHILD_ID")
    private Long id;

    @ManyToOne
    @JoinColumn(name = "CHILD_ID")
    private Child child;
}
```
**<span style="font-size:16px;"> 👉 매핑이 쉽고 코드가 단순 + 복합키 클래스 불필요 </span>**

<br>

### 4️⃣ 일대일 식별관계
* 일대일 식별 관계는 자식 테이블의 기본 키 값으로 부모 테이블의 키 값만 사용 (복합키로 구성하지 않아도 됨)

```java
// 부모 엔티티 클래스
@Entity
public class Board {
    @Id @GeneratedValue
    @Column(name = "BOARD_ID")
    private Long id;
    private String title;
    
    @OneToOne(mapped = "board")
    private BoardDetail boardDetail;
}

// 자식 엔티티 클래스
@Entity
public class BoardDetail {
    @Id
    private Long boardId;
    private String content;
  
    @MapsId
    @OneToOne
    @JoinColumn(name = "BOARD_ID")
    private Board board;
}
```
* 식별자가 단일 칼럼일시 `@MapsId`사용 + 속성은 비워둠

<br>

### 💡 식별 관계 VS 비식별 관계
|    | 식별 관계                                                                                                                                                                        | 비식별 관계                                                           |
|----|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------|
| 장점 | - 기본 키 인덱스를 활용하기 좋음 <br/>   - 특정 상황에 조인 없이 하위 테이블만으로 검색 가능                                                                                                                   | - **대리 키** 사용으로 유지보수성이 높음 <br/> - JPA는대리 키를 생성하기 위한 방법 제공 <br/> `@GeneratedValue` |
| 단점 | - 기본키 자식칼럼이 늘어남 ➡︎ 조인 쿼리, 인덱스 복잡 <br> - 2개 이상의 칼럼 ➡︎ 복합 기본키를 만들어야 하는 경우 많음 <br/> - **자연 키** 조합 주로 사용 ➡︎ 요구사항 변경시 유지보수 문제 <br/> - 테이블이 유연하지 않음 <br/> - JPA에서는 별도의 복합 키 클래스 필요 |                                                                  |
**<span style="font-size:16px;"> 👉 비식별 관계 사용 권장 </span>**

<br>


## 4) 조인 테이블
> ### 💡 연관관계 설계 방법
>   1) **조인 칼럼 사용** (외래 키)
>   2) **조인 테이블 사용** (테이블 사용)

|         |  조인 칼럼 사용  | 조인 테이블 사용  |
|:-------:|:----------:|:----------:|
| 비식별 관계  | 선택적 비식별 관계 | 필수적 비식별 관계 |
| null 허용 |     ⭕️     |     ❌      |
|   조인    |  외부 조인 사용  |  내부 조인 사용  |
**<span style="font-size:16px;"> 👉 조인 테이블 사용 권장 </span>**


### 1️⃣ 일대일 조인 테이블
* 2개의 유니크 제약 조건
```java
// 부모 클래스
@Entity
public class Parent {

    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private String id;
    private String name;

    @OneToOne
    @JoinTable(name = "PARENT_CHILD",           // name : 매핑할 조인 이름
            joinColumns = @JoinColumn(name = "PARENT_ID"),          // joinColumns : 현재 엔티티를 참조하는 외래키 
            inverseJoinColumns = @JoinColumn(name = "CHILD_ID")     // inverseJoinColumns: 반대방향 엔티티를 참조하는 외래키
    )
    private Child child;
}

// 자식 클래스
@Entity
public class Child {

    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private String id;
    private String name;
    
//    @OneToOne(mappedBy = "child")           // 양방향 구현시 사용
//    private Parent parent;
}
```

<br>

### 2️⃣ 일대다 조인 테이블
* 다와 관련된 컬럼에 유니크 제약조건
```java
// 단방향 일대다
// 부모 클래스
@Entity
public class Parent {

    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private String id;
    private String name;

    @OneToMany
    @JoinTable(name = "PARENT_CHILD",           // name : 매핑할 조인 이름
            joinColumns = @JoinColumn(name = "PARENT_ID"),          // joinColumns : 현재 엔티티를 참조하는 외래키 
            inverseJoinColumns = @JoinColumn(name = "CHILD_ID")     // inverseJoinColumns: 반대방향 엔티티를 참조하는 외래키
    )
    private List<Child> child = new ArrayList<>();
}

// 자식 클래스
@Entity
public class Child {

    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private String id;
    private String name;
}
```

<br>

### 3️⃣ 다대일 조인 테이블
```java
// 양방향 다대일
// 부모 클래스
@Entity
public class Parent {

    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private String id;
    private String name;

    @OneToMany(mappedBy = "parent")
    private List<Child> child = new ArrayList<>();
}

// 자식 클래스
@Entity
public class Child {

    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private String id;
    private String name;

    @ManyToOne(optional = false)
    @JoinTable(name = "PARENT_CHILD",           // name : 매핑할 조인 이름
            joinColumns = @JoinColumn(name = "CHILD_ID"),           // joinColumns : 현재 엔티티를 참조하는 외래키 
            inverseJoinColumns = @JoinColumn(name = "PARENT_ID")    // inverseJoinColumns: 반대방향 엔티티를 참조하는 외래키
    )
    private Parent parent;
}
```

<br>

### 4️⃣ 다대다 조인 테이블
```java
// 양방향 다대일
// 부모 클래스
@Entity
public class Parent {

    @Id @GeneratedValue
    @Column(name = "PARENT_ID")
    private String id;
    private String name;

    @ManyToMany
    @JoinTable(name = "PARENT_CHILD",           // name : 매핑할 조인 이름
            joinColumns = @JoinColumn(name = "PARENT_ID"),          // joinColumns : 현재 엔티티를 참조하는 외래키 
            inverseJoinColumns = @JoinColumn(name = "CHILD_ID")     // inverseJoinColumns: 반대방향 엔티티를 참조하는 외래키
    )
    private List<Child> child = new ArrayList<>();
}

// 자식 클래스
@Entity
public class Child {

    @Id @GeneratedValue
    @Column(name = "CHILD_ID")
    private String id;
    private String name;
}
```

> #### 📢 `@JoinTable` 전략의 한계
> 조인 테이블에 칼럼이 추가하면 `@JoinTable` 사용 불가 <br> 👉 중간 테이블 엔티티 직접 구현 필요

<br>

## 5) 엔티티 하나에 여러 테이블 매핑
* 한 개의 엔티티로 한 번에 두 개의 테이블 생성
```java
@Entity
@Table(name = "BOARD")          // 첫번째 테이블 이름
@SecondaryTable(name = "BOARD_DETAIL",      // 두번째 테이블 이름
        pkJoinColumns = @PrimaryKeyJoinColumn(name = "BOARD_DETAIL_ID")     // 두번째 테이블 기본키
)
public class Member {

    @Id @GeneratedValue
    @Column(name = "MEMBER_ID")
    private String id;          // 첫번째 테이블 기본키
    
    private String title;       // 첫번째 테이블 칼럼
    
    @Column(table = "BOARD_DETAIL")
    private String content;          // 두번째 테이블 칼럼
}
```