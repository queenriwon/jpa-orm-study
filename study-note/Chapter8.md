# Chapter.8 프록시와 연관관계 관리

> ### 📋 차례
> [1) 프록시](#1-프록시) <br>
> [2) 즉시 로딩과 지연 로딩](#2-즉시-로딩과-지연-로딩) <br>
> [3) 지연 로딩 활용](#3-지연-로딩-활용) <br>
> [4) 영속성 전이: CASCADE](#4-영속성-전이:-CASCADE) <br>
> [5) 고아 객체](#5-고아-객체) <br>
> [5) 영속성 전이 + 고아 객체, 생명주기](#6-영속성-전이-+-고아-객체,-생명주기) <br>

<br>

## 1) 프록시
* <ins>**지연 로딩**</ins>: 엔티티가 실제 사용될 때까지 데이터베이스 조회를 지연하는 방법 <br> 실제 사용하는 시점에 데이터베이스에서 엔티티에 필요한 데이터를 조회
* <ins>**프록시 객체**</ins>: 지연로딩을 사용할 때, 실제 엔티티 객체 대신 데이터베이스 조회를 지연하는 **가짜 객체**
    * 프록시 객체는 객체에 대한 <ins>참조</ins> 보관
    * 프록시 객체 메서드 호출시 **실제 객체 메서드 호출**

```java
// 1️⃣ 즉시로딩: 엔티티 조회
Member member = em.find(Member.class, "member1");

// 2️⃣ 지연로딩: 프록시 객체 반환
Member memeber = em.getReference(Member.class, "member1");
```
<br>

#### 프록시 객체 초기화
```java
Member member = em.getReference(Member.class, "id1");   // 프록시 객체 반환
member.getName();       // 진짜 객체 반환
```
1) 프록시 객체에 `member.getName()`을 호출해서 실제 데이터 조회
2) **초기화:** 프록시 객체는 실제 엔티티가 생성되어 있지 않으면 영속성 컨텍스트에 실제 엔티티 생성 요청
3) 영속성 컨텍스트는 데이터베이스를 조회(실제 엔티티 객체 생성)
4) 프록시 객체는 생성된 실제 엔티티 객체의 참조를 `Member target` 멤버변수에 저장
5) 프록시 객체는 실제 엔티티 객체의 `getName()`을 호출하여 결과 반환

<br>

#### 프록시 특징
- 사용시 한번만 초기화
- 프록시 객체가 초기화되면 프록시 객체를 통해 실제 엔티티 접근 *(프록시 객체가 바뀌는 것이 아님)*
- 프록시 객체는 원본 엔티티를 **상속**받은 객체
- 영속성 컨텍스트에 찾는 엔티티가 있으면 실제 엔티티 반환 👉 무조건 프록시 객체를 반환하는 것은 아님
- **준영속 상태의 프록시를 초기화 불가**

<br>

#### 프록시와 식별자
- 프로퍼티 접근 `@Access(AccessType.PROPERTY)`: 식별자 값을 조회해도 프록시를 초기화 하지 않음
- 필드 접근 `@Access(AccessType.FIELD)`: 식별자 값을 조회하면 프록시 객체 초기화

<span style="font-size:14px;"> 👉 필드방식 선호 (연관관계 설정시 식별자 값만 사용, 프록시를 사용하면 DB접근을 줄일 수 있음)</span>


<br>

## 2) 즉시 로딩과 지연 로딩
* <ins>**즉시 로딩**</ins>: 엔티티를 조회할 때 연관된 엔티티도 함께 조회
  * `@ManyToOne(fetch = FetchType.EAGER)`
  * 대부분의 JPA 구현체는 즉시로딩 최적화로 외부 조인사용
    * 외부 조인: `@JoinColumn(nullable = true)` 사용(기본값)
    * 내부 조인: `@JoinColumn(nullable = false)` 사용
  * 자주 함께 사용되는 엔티티는 즉시 로딩으로 사용
* <ins>**지연 로딩**</ins>: 연관된 엔티티를 실제 사용할 때 조회
  * `@ManyToOne(fetch = FetchType.LAZY)`
  * 실제 데이터가 필요할 때만 데이터베이스 조회(프록시 객체 초기화)
  * 가끔 엔티티가 함께 사용될 경우 지연 로딩으로 사용


<br>

## 3) 지연 로딩 활용
#### 프록시와 컬렉션 래퍼
```java
Member member = em.find(Member.class, "member1");
List<Order> orders = member.getOrders();
System.out.println("orders = " + orders.getClass().getName());
// 결과: orders = org.hibernate.collection.internal.PersistentBag;
```
- **컬렉션 래퍼**: 하이버네이트는 엔티티를 영속 상태로 만들때 추적 목적으로 원본 컬렉션을 하이버네이트 내장 컬렉션으로 변경**(지연 로딩)**

<br>

> ### 💡 기본 페치 전략
> - `@ManyToOne`, `@OneToOne`: 즉시 로딩 `FetchType.EAGER`
> - `@OneToMany`, `@ManyToMany`: 지연 로딩 `FetchType.LAZY` <br>
> 
> 연관된 엔티티가 하나면 **즉시로딩**, 컬렉션이면 **지연로딩** 사용 <br>
> <span style="font-size:14px;"> 👉 추천하는 방법은 모든 연관관계에 지연로딩 사용 </span>

> ### 🚨 컬렉션에 `FetchType.EAGER` 사용
> - 2개이상의 컬렉션을 즉시로딩 하는 것은 권장하지 않음 (N*M)
> - 컬렉션 즉시로딩은 항상 외부조인 사용
> - `@ManyToOne`, `@OneToOne`
>   - `(optional = false)`: 내부 조인
>   - `(optional = true)`: 외부 조인
> - `@OneToMany`, `@ManyToMany`
>   - `(optional = false)`: 외부 조인
>   - `(optional = true)`: 외부 조인

<br>

## 4) 영속성 전이: CASCADE
특정 엔티티를 영속상태로 만들 때 다른 엔티티도 함께 영속 상태로 만들 경우 👉 <ins>**영속성 전이**</ins> `CASCADE`

### 1️⃣ 영속성 전이: 저장
```java
// 설정
@OneToMany(mappedBy = "parent", cascade = CascadeType.PERSIST)
private List<Child> children = new ArrayList<Child>();

// 저장 코드
Parent parent = new Parent();
child1.setParent(parent);
child2.setParent(parent);
parent.getChildren().add(child1);
parent.getChildren().add(child2);

em.persist(parent);         // 부모 엔티티와 연관된 자식 엔티티 저장
```
부모와 자식엔티티를 한번에 영속화 가능

<br>


### 2️⃣ 영속성 전이: 삭제
```java
// 설정
@OneToMany(mappedBy = "parent", cascade = CascadeType.REMOVE)
private List<Child> children = new ArrayList<Child>();

// 삭제 코드
em.remove(parent);
```
부모 엔티니만 삭제하면 연관된 자식 엔티티도 함께 삭제 <br>
영속성을 설정하지 않으면 외래 키 제약조건으로 인해, 데이터베이스에서 외래 키 무결성 예외 발생

<br>

### 3️⃣ `CASCADE` 종류
```java
public enum CascadeType {
    ALL,        // 모두 적용
    PERSIST,    // 영속
    MERGE,      // 병합
    REMOVE,     // 삭제
    REFRESH,    // 리프레시
    DETACH      // detach
}
```
- 여러 속성 같이 사용 가능
- `CascadeType.PERSIST`, `CascadeType.REMOVE`는 `em.persist()`, `em.remove()`를 실행할 때 바로 전이가 발생하는 게 아닌 플러시를 호출할 때 전이

<br>

## 5) 고아 객체
* <ins>**고아 객체**</ins>: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
```java
// 설정
@OneToMany(mappedBy = "parent", orphanRemoval = true)
private List<Child> children = new ArrayList<Child>();

// 삭제 코드
parent.getChildren().remove(0);
```
* 컬렉션에서 엔티티를 제거하면 데이터베이스의 데이터도 삭제된다.
* 참조가 제거된 엔티티를 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제
* 참조하는 곳이 하나일 때 사용
* 부모를 제거하면 자식도 제거됨(`CascadeType.REMOVE`)

<br>

## 6) 영속성 전이 + 고아 객체, 생명주기
* `CascadeType.ALL` + `orphanRemoval = true` 👉 부모를 통해서 자식의 생명주기를 관리할 수 있다.
  * `CascadeType.ALL`: 자식을 저장하려면 부모에 등록만 하면됨
  * `orphanRemoval = true`: 자식을 삭제하려면 부모에서 제거하면 됨