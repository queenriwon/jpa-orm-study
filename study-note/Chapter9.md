# Chapter.9 값 타입

> ### 📋 차례
> [1) 기본값 타입](#1-기본값-타입) <br>
> [2) 임베디드 타입](#2-임베디드-타입) <br>
> [3) 값 타입과 불변 객체](#3-값-타입과-불변-객체) <br>
> [4) 값 타입의 비교](#4-값-타입의-비교) <br>
> [5) 값 타입 컬렉션](#5-값-타입-컬렉션) <br>

> ### 👉 값 타입 종류
> 1) 기본값 타입 (자바 기본타입, 래퍼 클래스, String)
> 2) 임베디드 타입 (복합 값 타입)
> 3) 컬렉션 값 타입

<br>

## 1) 값 타입
```java
private String name;
private int age;
```
* 값 타입은 식별자 값도 없고 생명주기가 엔티티에 의존.
* 엔티티 인스턴스를 제거하면 해당 값 타입도 사라짐 👉 공유 불가

## 2) 임베디드 타입
* 값 타입을 지정 가능
```java
@Embedded Period workPeriod;
@Embedded Address homeAddress;

// Period 임베디드 타입
@Embeddalbe
public class Period {
    @Temporal(TemporalType.DATE) java.util.Date startDate;
    @Temporal(TemporalType.DATE) java.util.Date endDate;
    
    // 값 타입 정의 메서드
}

// Address 임베디드 타입
@Embeddalbe
public class Address {
    private String city;
    private String street;
    private String zipcode;

  // 값 타입 정의 메서드
}
```
* `@Embedded`: 값 타입을 정의하는 곳에 표시
* `@Embedable`: 값 타입을 사용하는 곳에 표시
* 임베디드 타입은 기본 생성자가 필수
* 임베디드 타입은 하위 임베디드 타입 또는 연관관계의 엔티티를 포함할 수 있다.

임베디드 타입을 포함한 모든 기본 값 타입은 엔티티의 생명주기에 의존 <br>
👉 엔티티와 임베디드의 관계는 <ins>**컴포지션 관계**</ins> <br>
👉 하이버네이트는 임베디드 타입을 <ins>**컴포넌트**</ins>라고 한다.

<br>

## 3) 값 타입과 불변 객체
* 임베디드의 공유 참조는 값은 인스턴스를 상요하기 때문에 인스턴스 수정시 모든 속성이 수정되는 부작용 발생 <br> 👉 인스턴스 복사해서 사용해야 됨 `.clone()`
  * 임베디드 타입은 자바의 기본 타입이 아닌 객체 타입이기 때문
* 객체의 공유 참조를 피할 수는 없다. 대신, 객체 값을 수정하지 못하게 <ins>**불변 객체**</ins>로 설계
  * 생성자로만 값을 설정하고 수정자를 만들지 않으면 된다.

<br>

## 4) 값 타입의 비교
* 값 타입의 동일성을 비교하려면 `equals()`및 `hashcode()`메서드 재정의 필요

<br>

## 5) 값 타입 컬렉션
```java
@ElementCollection
@CollectionTable(name = "FAVORITE_FOODS",
    joinColumns = @JoinColumns(name = "MEMBER_ID"))
@Column(name = "FOOD_NAME")
private Set<String> favoriteFoods = new HashSet<String>();

@ElementCollection
@CollectionTable(name = "ADDRESS",
    joinColumns = @JoinColumns(name = "MEMBER_ID"))
private List<Address> addressHistory = new ArrayList<Address>();
```
1) <ins>**기본 값 타입 컬렉션**</ins>
   * 기본 타입을 컬렉션으로 가짐
   * 관계형 테이블은 컬럼안에 컬렉션을 포함할 별도의 테이블을 추가하고 매핑 `@CollectionTable`
2) <ins>**임베디드 타입 컬렉션**</ins>
   * 임베디드 타입을 컬렉션으로 가짐

<br>

### 1️⃣ 값 타입 컬렉션 저장
```java
Member member = new Member();

// 1) 임베디드 값 타입
member.serHomeAddress(new Address("통영", "몽돌해수욕장", "660-123"));

// 2) 기본값 타입 컬렉션
member.getFavoriteFoods().add("짬뽕");
member.getFavoriteFoods().add("짜장");
member.getFavoriteFoods().add("탕수육");

// 3) 임베디드값 타입 컬렉션
member.getAddressHistory().add(new Address("서울", "강남", "123-123"));
member.getAddressHistory().add(new Address("서울", "강북", "000-000"));

em.persist(member);
```
1) 임베디드 값 타입은 테이블을 저장하는 INSERT SQL에 포함
2) 기본 값 타입 컬렉션은 각각의 INSERT SQL 실행
3) 임베디드 값 타입 컬렉션은 각각의 INSERT SQL 실행

<br>

### 2️⃣ 값 타입 컬렉션 조회
```java
Member member = em.find(Member.class, 1L);

// 1) 임베디드 값 타입
Address homeAddress = member.getHomeAddress();

// 2) 기본값 타입 컬렉션
Set<String> favoriteFoods = member.getFavoriteFoods();
for (String favoriteFood : favoriteFoods) {
    System.out.println(favoriteFood);
}

// 3) 임베디드값 타입 컬렉션
List<Address> addressHistory = member.getAddressHistory();
addressHistory.get(0);
```
1) 임베디드 값 타입은 테이블을 조회할 때 함께 조회
2) 기본 값 타입 컬렉션은 지연로딩으로 실제 컬렉션을 사용할 때 SELECT SQL 호출
3) 임베디드 값 타입 컬렉션은 지연로딩으로 실제 컬렉션을 사용할 때 SELECT SQL 호출

<br>

### 3️⃣ 값 타입 컬렉션 수정
```java
Member member = em.find(Member.class, 1L);

// 1) 임베디드 값 타입
member.setHomeAddress(new Address("새로운 도시", "신도시1", "123456"));

// 2) 기본값 타입 컬렉션
Set<String> favoriteFoods = member.getFavoriteFoods();
favoriteFoods.remove("탕수육");
favoriteFoods.add("치킨");

// 3) 임베디드값 타입 컬렉션
List<Address> addressHistory = member.getAddressHistory();
addressHistory.remove(new Address("서울", "기존 주소", "123-123"));
addressHistory.remove(new Address("새로운도시", "새로운주소", "123-456"));
```
1) 임베디드 값 타입은 엔티티를 수정
2) 기본 값 타입 컬렉션은 기존값을 제거하고 새로운 값 추가
3) 임베디드 값 타입 컬렉션은 불변해야하기 때문에 기존값을 제거하고 새로운 값 추가

<br>

> ### 📢 값 타입 컬렉션의 제약사항
> * 단점1) 값타입은 식별자라는 개볌이 없고 값을 변경하면 데이터베이스에 저장된 원본데이터를 찾기 어렵다.
>   * 값 타입 컬렉션에 변경사항이 발생하면, 값 타입 컬렉션이 매핑된 테이블의 연관된 모든 데이터를 삭제, 현재 값 타입 컬렉션 객체에 있는 모든 값을 데이터 베이스에 다시 저장
> * 단점 2) 데이터베이스 기본 키 제약 조건으로 인해 칼럼에 null을 입력할 수 없고, 같은 값을 중복해서 자장할 수 없다.
>  * <span style="font-size:14px">👉 새로운 엔티티를 만들어 일대다 관계 + 영속성 전이 + 고아객체 제거</span>

<br>

### 💡 엔티티 타입 vs 값 타입 비교
|      | 엔티티 타입 |    값 타입     |
|:----:|:------:|:-----------:|
| 식별자  |   ⭕️   |      ❌      |
| 생명주기 |   ⭕    |   엔티티에 의존   |
|  공유  |   ⭕️   | 공유하지 않는게 안전 |
