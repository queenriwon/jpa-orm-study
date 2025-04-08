# Chapter.10 객체지향 쿼리 언어

> ### 📋 차례
> [1) 객체지향 쿼리 소개](#1-객체지향-쿼리-소개) <br>
> [2) JPQL](#2-JPQL) <br>
> [3) Criteria](#3-Criterial) <br>
> [4) QueryDSL](#4-QueryDSL) <br>
> [5) 네이티브 SQL](#5-네이티브-SQL) <br>
> [6) 객체지향 쿼리 심화](#6-객체지향-쿼리-심화) <br>

<br>

## 1) 객체지향 쿼리 소개

> #### 엔티티 조회 방법
> - 식별자로 조회 `EntityManager.find()`
> - 객체 그래프 탐색 `a.getB().getC()`
> - SQL에서 걸러서 최대한 조회 할 것

> #### JPQL의 기본 특징
> - 테이블이 아닌 객체를 대상으로 검색하는 <ins>**객체지향 쿼리**</ins>
>   - SQL은 데이터베이스의 테이블을 대상으로 하는 <ins>**데이터 중심 쿼리**</ins>
> - SQL을 추상화해서 특정 데이터베이스 SQL에 의존하지 않음

> ### 💡 객체지향 쿼리
> 1) **JPQL** (Java Persistence Language)
> 2) **Criteria 쿼리**: JPQL을 편하게 작성하도록 도와주는 API. 빌더 클래스 모음
> 3) **Native SQL**: JPA에서 JPQL 대신 직접 SQL을 사용
> 4) **QueryDSL**: Criteria 쿼리처럼 JPQL을 편하게 작성하도록 도와주는 빌더 클래스 모음. 비표준 오픈소스 프레임웤,
> 5) **JDBC 직접 사용, MyBatis 같은 SQL 매퍼 프레임워크 사용**

<br>

<details>
<summary> 객체지향 쿼리 </summary>
<div markdown="1">

### 1️⃣ JPQL 소개
* <ins>**JPQL**</ins>: 엔티티 객체를 조회하는 객체지향 쿼리
* JPQL은 SQL을 추상화해서 특정 데이터베이스에 의존하지 않는다.
* ❶ 엔티티 직접 조회, ❷ 묵시적 조인, ❸ 다형성 지원

<br>

### 2️⃣ Criteria 쿼리 소개
* Criteria 쿼리는 프로그래밍 코드로 JPQL을 작성할 수 있음, 컴파일 시점에 오류 발생
  * JPQL은 문자 쿼리이기 때문에 실행되는 시점에 런타임 오류 발생
* 코드 자동완성 지원
* 동적쿼리 작성 용이
* <ins>메타 모델</ins>: 엔티티 클래스에서 Criteria 전용 클래스 생성
* 복잡하고 사용하기 불편, 코드가 한눈에 들어오지 않는 단점

<br>

### 3️⃣ Native SQL 소개
* JPA에서 SQL을 직접 사용할 수 있음
* 특정 데이터베이스에 의존하는 기능을 사용해야 할때 사용

<br>

### 4️⃣ QueryDSL 소개
* Criteria처럼 JPQL 빌더 역할, 코드 기반
* Criteria에 비해 단순하고 사용하기 쉬움, 코드가 한눈에 들어옴
* 어노테이션 프로세서를 이용해 쿼리 전용 클래스 생성 필요

<br>

### 5️⃣ JDBC 직접 사용, 마이바티스같은 SQL 매퍼 프레임워크 사용
* JDBC 커넥션에 직접 접근하고 싶으면 JPA는 JDBC 커넥션을 획득하는 API를 제공하지 않아 JPA 구현체가 제공하는 방법 사용
* JDBC나 마이바티스를 JPA와 함께 사용하면 영속성 컨텍스트를 적절한 시점에 강제 플러시 ➡︎ 무결성 훼손 가능성 ➡ 영속성 컨텍스트로 수동 동기화

</div>
</details>

<br>

## 2) JPQL
### 1️⃣ 기본 문법과 쿼리 API
* SELECT문
  * 엔티티와 속성은 대소문자를 구분
  * JPQL 키워드는 대소문자를 구분하지 않음
  * 엔티티 이름을 사용
  * 별칭(식별변수) 필수
* `TypeQuery`, `Query`
  * `TypeQuery`: 반환할 타입을 명확하게 지정할 수 있을 때 쿼리 객체
  * `Query`: 반환 타입이 명확하지 않을 때 쿼리 객체
    * `object[]`: 조회 대상이 둘 이상일 때
    * `object`: 조회 대상이 하나만 있을 때
* 결과 조회
  * `query.getResultList()`: 결과 반환
  * `query.getSingleResult()`: 결과가 하나일 때 사용(결과가 없을 때, 1개보다 많으면 오류)

<br>

### 2️⃣ 파라미터 바인딩
* **이름 기준 파라미터 바인딩**과 **위치 기준 파라미터 바인딩** 지원
* 파라미터 바인딩 방식을 사용하지 않으면 SQL 인젝션 공격을 당할 수 있음

<br>

### 3️⃣ 프로젝션
* **엔티티 프로젝션**: 엔티티를 조회, 이렇게 조회한 엔티티는 영속성 컨텍스트에서 관리
* **임베디드 타입 프로젝션**: 임베디드 타입은 엔티티 타입이 아닌 값 타입이기 때문에 엔티티를 기준으로 작성되어야 함 (`o.address`)
  * 영속성 컨텍스트에서 관리가 되지 않음
* **스칼라 타입 프로젝션**
* 여러 값 조회: `Query` + `Object[]` 사용
* **NEW 명령어**: 객체를 사용하여 DB의 데이터 조회
  * 반환받을 클래스를 지정하여 이 클래스의 생성자에 jPQL 조회 결과를 넘겨줄 수 있음
  * 패키지 명을 포함한 전체 클래스 명을 입력해야하며, 순서와 타입이 일치하는 생성자 필요

<br>

### 4️⃣ 페이지 API
```java
setFirstResult(int startPosition);   // 조회 시작 위치
setMaxResults(int maxResult);       // 조회할 데이터 수
```

<br>

### 5️⃣ 집합과 정렬
* **집합 함수**
  * NULL 값은 무시하므로 통계에 잡히지 않음
  * 값이 없을 떄 집합 함수를 사용하면 NULL 값이 됨(COUNT는 0)
  * DISTINCT를 집합 함수 안에 사용해서 중복된 값 제거 후 집합을 구할 수 있음
  * DISTINCT를 COUNT해서 사용할 때 임베디드 값을 사용하지 않음
* **GROUP BY, HAVING**
  * 통계 데이터 쿼리, 실시간으로 사용하기에는 부담
* **ORDER BY**

<br>

### 6️⃣ JPQL 조인
* **내부 조인**
  * `INNER` 생략 가능
  * 연관 필드 사용 하여 조인 `FROM Member m INNER JOIN m.team t`
* **외부 조인**
  * 보통 `OUTER`를 생략하여 `LEFT JOIN`을 사용
* **컬렉션 조인**
  * 컬렉션을 사용하는 곳에 조인하는 것
    * [회원 -> 팀]으로의 조인은 다대일 조인, 단일 값 연관 필드 사용
    * [팀 -> 회원]으로의 조인은 일대다 조인, 컬렉션 값 연관 필드 사용 (외부조인)
* **세타 조인**
  * 조인에 참여하는 두 릴레이션 속성값을 비교하여 조건을 만족하는 튜플만 반환
  * 내부조인만 지원
* **JOIN ON 절**
  * 내부 조인의 ON절은 WHERE 절을 사용할 때와 결과가 같으므로 보통 ON 절은 외부 조인에만 사용

<br>


### 7️⃣ JPQL 조인
성능 최적화를 통해 연관된 엔티티나 컬렉션을 같이 조회 <br>

* **엔티티 페치 조인** 
  * 연관된 엔티티를 함께 조회
  * 별칭을 사용할 수 없음 `FROM Member m JOIN FETCH m.team`
* **컬렉션 페치 조인**
  * 연관된 컬렉션을 함께 조회
  * 일대다 조인은 결과가 증가(일대일, 다대일 조인은 결과가 증가하지 않음) ➡︎ DISTINCT 사용
  * 페치 조인을 사용하지 않으면, JPQL은 결과를 반환할 때 연관관계까지 고려하지 않고 엔티티만 조회한다.

> ### 📢 페치 조인의 특징과 한계
> - 페치 조인을 사용하면 준영속 상태에도 객체 그래프 탐색 가능
> - 글로벌 로딩 전략: 엔티티에 직접사용하는 로딩 전략 `@OneToMany(fetch = FetchType.LAZY)`
> - 글로벌 로딩 전략을 즉시로딩으로 설정하면 성능에 악영향
> 
> 👉 <ins>**로딩 전략은 될 수 있으면 지연로딩 사용 + 최적화가 필요할 시 페치 조인 적용**</ins>
> 
> - 페치 조인 대상에는 별칭을 줄 수 없음
> - 둘 이상의 컬렉션을 페치할 수 없음
> - 컬렉션을 페치 조인하면 페이징 API를 사용할 수 없음
> 
> 👉 <ins>**페치 조인은 객체 그래프를 유지할 때 사용하면 효과적**</ins>

<br>

### 8️⃣ 경로 표현식
* 상태 필드: 단순히 값을 저장하기 위한 필드
  * 경로: 경로 탐색의 끝, 더는 탐색할 수 없음
* 연관 필드: 연관관계를 위한 필드, 임베디드 타입 포함
  * <ins>**단일 값 연관 필드**</ins>: `@ManyToOne`, `@OneToOne`, 대상이 엔티티
    * 경로: 묵시적으로 **내부 조인**이 일어남, 계속 탐색 가능
    * <ins>**묵시적 조인**</ins>: 단일 값 연관 필드로 경로 탐색시 SQL에서 내부 조인 발생 `SELECT m.team FROM Member m`
    * <ins>**명시적 조인**</ins>: `JOIN`을 명식적으로 작성 `SELECT m FROM Member m JOIN m.team t`
  * <ins>**컬렉션 값 연관 필드**</ins>: `@OneToMany`, `@ManyToMany`, 대상이 컬렉션
    * 경로: 묵시적으로 **내부 조인**이 일어남, 더는 탐색할 수 없음
    * 조인을 통해 별칭을 얻으면 별칭으로 탐색 가능

> ### 📢 경로 탐색을 사용한 묵시적 조인시 주의사항
> - 경로 탐색을 사용하면 묵시적 조인이 발생해서 SQL에서 내부 조인이 발생할 수 있음
> - 항상 내부 조인
> - 컬렉션은 경로 탐색의 끝이며 컬렉션에서 경로 탐색을 하려면 명시적으로 조인해서 별칭을 얻어야 한다.
> - 경로 탐색은 주로 SELECT, WHERE절에서 사용하지만 묵시적 조인으로 인해 SQL의 FROM 절에 영향
> 
> 👉 <ins>**분석에 용이하도록 명시적 조인을 사용하자**</ins>

<br>

### 9️⃣ 서브 쿼리
서브쿼리는 WHERE, HAVING 절에서만 사용할 수 있고, SELECT, FROM에서는 사용 불가
* `[NOT] EXISTS (subquery)`
* `{ALL | ANY | SOME} (subquery)`
* `[NOT] IN (subquery)`

<br>

### 1️⃣0️⃣ 조건식
* 타입 표현 : 문자, 숫자, 날짜, Boolean, Enum, 엔티티 타입
* 연산자 우선 순위: 경로 탐색 연산 -> 수학 연산 -> 비교 연산 -> 논리 연산
* 논리연산과 비교식: AND, OR, NOT, = < > <>
* Between, IN, Like, NULL
* 컬렉션 식
  * `{컬렉션 값 연관 경로} IS [NOT] EMPTY`
  * `{엔티티나 값} (NOT) MEMBER [OF] {컬렉션 값 연관 경로}`
* 스칼라 식: 수학 식(사칙연산), 문자함수, 수학함수, 날짜함수
* CASE 식: 기본 CASE, 심플 CASE, COALESCE, NULLIF

<br>

### 1️⃣1️⃣ 다형성 쿼리
* 단일 테이블 전략 `InheritanceType.SINGLE_TABLE` -> `SELECT * FROM ITEM`
* 조인 전략 `InheritanceType.JOINED` -> 조인 사용
* TYPE: 엔티티의 상속 구조에서 조회 대상을 특정 자식으로 한정할 때 사용
* TREAT: 상속 구조에서 부모 타입을 특정 자식 타입으로 다룰 때 사용

<br>

### 1️⃣2️⃣ 사용자 정의 함수 호출
* 하이버네이트 구현체를 사용하면 방언 클래스`H2Dialect`를 상속해서 구현하고 사용할 데이터베이스 함수를 미리 등록

<br>

### 1️⃣3️⃣ 기타 정리
* 기타 정리
  * enum은 = 비교 연산자만 지원
  * 임베디드 타입은 비교를 지원하지 않는다.
* EMPTY STRING
  * 데이터베이스에 따라 ''을 null로 사용할 수 있음
* NULL 정의
  * 조건을 만족하는 데이터가 하나도 없으면 NULL
  * NULL은 알 수 없는 값이다. 따라서 NULL과의 모든 수학적 계산 결과는 NULL
  * NULL==NULL은 알 수 없는 값이다.
  * NULL is NULL 은 참이다.

<br>

### 1️⃣4️⃣ 엔티티 직접 사용
* 기본 키 값
  * 객체 인스턴스는 참조값으로 식별하고, 테이블 로우는 기본키 값으로 식별
  * member 와 member.id가 같음
* 외래 키 값
  * 외래 키 값을 가지고 있으면 묵시적 조인이 일어나지 않음
  * m.team 과 m.team.id가 같음

<br>

### 1️⃣5️⃣ Named 쿼리: 정적 쿼리
* 동적 쿼리: JPQL을 문자로 완성해서 직접 넘김. 런타임에 특정 조건에 따라 JPQL을 동적으로 구성 가능
* 정적 쿼리: 미리 정의한 쿼리에 이름을 부여(Named 쿼리) 한번 정의하면 변경 불가


* Named 쿼리: 애플리케이션 로딩 시점에 JPQL 문법을 체크하고 미리 파싱 -> 오류 확인, 결과 재사용, 성능이점, 조회 최적화
  * 어노테이션 정의 `@NamedQuery`
  * XML에 정의(우선권)

<br>

## 4) QueryDSL

### 1️⃣ 시작
- 쿼리용 클래스 생성(Q로 시작하는 쿼리 타입들 생성)
- 별칭 사용 
```java
// (1) 별칭 사용 - 직접 지정
JPAQuery query = new JPAQueny(em);
QMember qmember = new QMember("m");

// (2) 기본 인스턴스 사용
public class QMember extends EntityPathBase<Member> {
    public static final QMember member = new QMember("member1");
    ...
}
QMember qMember = QMember.member;
```
<br>

### 2️⃣ 검색 조건
```java
List<Item> list = query.from(item)
        .where(item.name.eq("좋은상품").and(item.price.gt(20000)))
        .list(item);
```
- and나 or 사용 가능

<br>

### 3️⃣ 결과 조회
- `uniqueResult()`: 조회 결과가 한 건일 때 사용, 조회 결과가 없으면 null을 반환, 결과가 하나 이상이면 예외 발생
- `singleResult()`: 결과가 하나 이상이면, 처음 데이터를 반환
- `list()`: 결과가 하나 이상일 때 사용. 결과가 없으면 빈 컬렉션 반환

<br>

### 4️⃣ 페이징과 정렬
```java
QItem item = QItem.item;

query.from(item)
    .where(item.price.gt(20000))
    .orderBy(item.price.desc(), item.stockQuantity.asc())
    .offset(10).limit(20)
    .list(item);
```
- 정렬: asc(), desc()
- 페이징: offset, limit를 조합해서 사용

<br>

```java
QueryModifiers queryModifiers = new QueryModifiers(20L, 10L);   // limit, offset
List<Item> list = query.from(item)
        .restrict(queryModifiers)
        .list(item);
```
- `restrict()` 메서드에 `QueryModifiers`를 파라미터로 사용

<br>

```java
SearchResult<Item> result = query.from(item)
        .where(item.price.gt(10000))
        .offset(10).limit(20)
        .listResults(item);

long total = result.getTotal();     // 검색된 전체 데이터 수
long limit = result.getLimit();
long offset = result.getOffset();
List<Item> results = result.getResults();       // 조회된 데이터
```
- 전체 데이터 조회를 위한 쿼리를 한번 더 실행

<br>

### 5️⃣ 그룹
```java
.groupBy(item.price)
.having(item.price.gt(1000))
.list(item);
```

<br>

### 6️⃣ 조인
```java
query.from(order)
    .join(order.member, member)
    .leftJoin(order.orderItems, orderItem)
    .list(order);

query.from(order)
    .leftJoin(order.orderItems, orderItem)
    .on(orderItem.count.gt(2))      // on 사용
    .list(order);

qeury.from(order)
    .innerJoin(order.member, member).fetch()        // 페치조인 사용
    .leftJoin(order.orderItems, orderItem).fetch()
    .list(order);

query.from(order, member)       // 세타조인
    .where(order.member.eq(member))
    .list(order);
```
- `innerJoin(join)`, `leftJoin`, `rightJoin`, `fullJoin`, 페치조인 사용 가능
- `join(조인 대상, 별칭으로 사용할 쿼리 타입)`

<br>

### 7️⃣ 서브 쿼리
```java
query.from(item)
    .where(item.price.eq(
          new JPASubQuery().from(itemSub).unique(itemSub.price.max())
    ))
    .list(item);
```

<br>

### 8️⃣ 프로젝션과 결과 반환
```java
// (1) 프로젝션 대상이 하나
List<String> result = query.from(item).list(item.name);

// (2) 여러 칼럼 반환과 튜플
List<Tuple> result = query.from(item).list(item.name, item.price);

// (3) 빈 생성 - 프로퍼티 접근 (setter)
List<ItemDTO> result = query.from(item).list(
        Projections.bean(ItemDTO.class, item.name.as("username"), item.price));

// (4) 빈 생성 - 필드 직접 접근 (fields)
List<ItemDTO> result = query.from(item).list(
        Projections.fields(ItemDTO.class, item.name.as("username"), item.price));

// (5) 빈 생성 - 생성자 직접 접근 (constructor)
List<ItemDTO> result = query.from(item).list(
        Projections.constructor(ItemDTO.class, item.name, item.price));
```

<br>

### 9️⃣ 수정, 삭제 배치 쿼리
```java
// (1) 수정 쿼리 (JPAUpdateClause 사용)
JPAUpdateClause updateClause = new JPAUpdateClause(em, item);
long count = updateClause.where(item.name.eq("..."))
        .set(item.price, item.price.add(100))
        .execute();

// (2) 삭제 쿼리 (JPADeleteClause)
JPADeleteClause deleteClause = new JPADeleteClause(em, item);
long count = deleteClause.where(item.name.eq("..."))
        .execute();
```

<br>

### 1️⃣0️⃣ 동적 쿼리
```java
BooleanBuilder builder = new BooleanBuilder();
if (param.getPrice != null) {
    builder.and(item.price.gt(param.getPrice()));
}
List<Item> result = query.from(item)
        .where(builder)
        .list(item);
```

<br>

### 1️⃣1️⃣ 메서드 위임
- 검색조건 직접 정의 가능(정적 메서드)
```java
public class ItemExpression {
    @QuaryDelegate(Item.class)
    public static BooleanExpression isExpensive(QItem item, Integer price) {
        return item.price.gt(price);
    }
}

query.from(item).where(item.isExpensive(30000)).list(item);
```

<br>

## 5) 네이티브 SQL
> ### 📢 특정 데이터베이스에 종석적인 기능을 지원하는 방법
> - 특정 데이터베이스만 사용하는 함수
>   - JPQL에서 네이티브 SQL 함수를 호출할 수 있다.
>   - 하이버네이트는 데이터베이스 방언에 각 데이터베이스에 종속적인 함수들을 정의
> - 특정 데이터베이스만 지원하는 SQL 쿼리 힌트
>   - 하이버네이트를 포함한 몇몇 JPA 구현체들이 지원
> - 인라인 뷰, UNION, INTERSECT
>   - 하이버네이트는 지원하지 않지만 일부 JPA 구현체들이 지원
> - 스토어 프로시저
>   - JPQL에서 스토어드 프로시저를 호출할 수 있다.
> - 특정 데이터베이스만 지원하는 문법

- 네이티브 SQL을 사용하면 엔티티를 조회할 수 있고 <ins>**JPA가 지원하는 영속성 컨텍스트의 기능을 그대로 사용할 수 있다.**</ins>
- 네이티브 SQL로 SQL을 직접 사용할 뿐 나머지는 JPQL과 같음. **조회한 엔티티가 영속성 컨텍스트에서 관리**
- 파라미터 바인딩
  - JPA: 위치기반 파라미터만 가능
  - 하이버네이트: 위치기반 및 이름기반 파라미터 바인딩 가능

<br>

### 1️⃣ 조회 및 결과 바인딩
```java
// (1) 결과 타입 정의 (엔티티)
public Query createNativeQuery(String sqlString, Class resultClass);

// (2) 결과 타입을 정의할 수 없을 때
public Query createNativeQuery(String sqlString);

// (3) 결과 매핑 사용
public Query createNativeQuery(String sqlString, String resultSetMapping);
```
- 조회한 값들은 object[]에 담아 반환
- 스칼라 값들은 영속성 컨텍스트에서 관리하지 않음 (엔티티는 관리)
- 엔티티와 스칼라 값을 함께 조회할 경우 `@SqlResulSettMapping`정의 (이름으로 여러 엔티티와 칼럼 매핑 가능)

```java
Query nativeQuery = em.createNativeQuery(sql, "memberWithOrderCount");

// (1) ColumnResult 사용
@Entity
@SqlResultSetMapping(name = "memberWithOrderCount",
    entities = {@EntityResult(entityClass = Member.class)},
    columns = {@ColumnResult(name = "ORDER_COUNT")}
)
public class Member {...}

// (2) FieldResult 사용
@Entity
@SqlResultSetMapping(name = "OrderResults",
        entities = {
            @EntityResult(entityClass = com.acme.Order.class,
                fields = {
                    @FieldResult(name = "id", column = "order_id"),
                    @FieldResult(name = "quantity", column = "order_quantity"),
                    @FieldResult(name = "item", column = "order_item"),
                })},
        columns = {@ColumnResult(name = "item_name")}
)
public class Member {...}
```

<br>

### 2️⃣ Named 네이티브 SQL
```java
TypeQuery<Member> nativeQuery = em.createNamedQuery("Member.memberSQL", Member.class)
        .setParameter(1, 20);

@Entity
@NamedNativeQuery(
        name = "Member.memberSQL",
        query = ... ,
        resultClass = Member.class
)
public class Member {...}
```
- 결과 매핑과 함께 사용 가능

<br>

> ### 📢 네이티브 SQL 특징과 단점
> - 네이티브 SQL을 사용하면 엔티티를 조회할 수 있고 <ins>**JPA가 지원하는 영속성 컨텍스트의 기능을 그대로 사용할 수 있다.**</ins>
> - 단점: 관리하기 쉽지 않고, 자주 사용하면 특정 데이터베이스에 종속적인 쿼리 증가 -> 이식성 감소
> 👉 <ins>**표준 JPQL 사용**</ins>

<br>

### 3️⃣ 스토어드 프로시저
```sql
DELIMITER //
          CREATE PROCEDURE proc_multiply (INOUT inParam INT, INOUT outParam INT)
          BEGIN
            SET outParam = inParam * 2;
END //
```
```java
StoredProcedureQuery spq = em.createStoredProcedureQuery("proc_multiply");
spq.registStoredProcedureParameter(1, Integer.class, ParameterMode.IN);
spq.registStoredProcedureParameter(2, Integer.class, ParameterMode.OUT);

spq.setParameter(1, 100);       // 순서 기반 파라미터
// spq.setParameter("inParam", 100);        // 이름 기반 파라미터
spq.execute();
```
- Named 스토어드 프로시저도 함께 사용 가능

<br>

## 6) 객체지향 쿼리 심화
### 1️⃣ 벌크 연산
- 벌크 연산: 한번에 수정하거나 삭제하는 연산
- 하이버네이트는 INSERT 벌크 연산도 지원
```java
// (1) UPDATE 벌크 연산 (executeUpdate)
int resultCount = em.createQeury(sqlString)
                .setParameter("stockAmount", 10)
                .executeUpdate();

// (2) DELETE 벌크 연산 (executeUpdate)
int resultCount = em.createQuery(splString)
                .setParameter("price", 100)
                .executeUpdate();
```
> ### 🚨 벌크 연산 주의점
> **영속성 컨텍스트와 2차 캐시를 무시하고 데이터베이스에 직접 실행** <br>
> 벌크 연산이 영속성 컨텍스트를 무시하고 데이터베이스에 직접 쿼리 <br>
> ➡️ 벌크 연산 후 그 데이터를 조회할 때 수정 전 데이터가 출력
>
> #### 해결방법
> 1) `em.refresh()`: 데이터베이스에서 다시 조회
> 2) 벌크 연산 먼저 실행 (권장)
> 3) 벌크 연산 수행 후 영속성 컨텍스트 초기화

<br>

### 2️⃣ 영속성 컨텍스트와 JPQL
- JPQL로 엔티티를 조회 시 엔티티만 영속성 컨텍스트에서 관리
- JPQL로 조회한 엔티티는 영속상태
- 영속성 컨텍스트에 이미 존재하는 엔티티가 있으면 기존 엔티티 반환 
- `find()` vs `JPQL`
  - `find()`: 엔티티를 영속성 컨텍스트에서 먼저 찾고 없으면 데이터베이스에서 찾는다.(성능 이점, 1차 캐시)
  - `JPQL`: 항상 데이터베이스에 SQL을 실행해서 결과 조회, 같은 엔티티가 영속성 컨텍스트에 있으면 영속성 컨텍스트에 있는 기존 엔티티 반환(기존 엔티티 버림)
👉 <ins>**영속성 컨텍스트는 영속 상테인 엔티티의 동일성 보장**</ins>

<br>

### 3️⃣ JPQL과 플러시 모드
```java
em.setFlustMode(FlushModeType.AUTO);    // 커밋 또는 쿼리 실행 시 플러시(기본값)
em.setFlustMode(FlushModeType.COMMIT);  // 커밋 시에만 플러시
```
- `FlushModeType.COMMIT`: 트랜잭션을 커밋할 따만 플러시하고 쿼리를 실행할 때는 플러시 하지 않는다.
  - 플러시가 너무 빈번하게 발생하면 플러시 횟수를 줄여 성능 최적화 가능
  - JDBC를 사용할 때도 플러시 모드 사용 고려(직접 플러시 실행)
