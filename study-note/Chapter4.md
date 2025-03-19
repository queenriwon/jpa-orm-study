# Chapter.4 엔티티 매핑

> ### 📋 차례
> [1) @Entity](#1-entity) <br>
> [2) @Table](#2-table) <br>
> [3) 다양한 매핑 사용](#3-다양한-매핑-사용) <br>
> [4) 데이터베이스 스키마 자동 생성](#4-데이터베이스-스키마-자동-생성) <br>
> [5) DDL 생성 기능](#5-ddl-생성-기능) <br>
> [6) 기본 키 매핑](#6-기본-키-매핑) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[i. 기본키 직접 할당](#1-기본키-직접-할당) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[ii. IDENTITY 자동 생성 전략](#2-IDENTITY-자동-생성-전략) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[iii. SEQUENCE 자동 생성 전략](#3-SEQUENCE-자동-생성-전략) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[iv. TABLE 자동 생성 전략](#4-TABLE-자동-생성-전략) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[v. AUTO 자동 생성 전략](#5-AUTO-자동-생성-전략) <br>
> [7) 필드와 칼럼 매핑: 레퍼런스](#7-필드와-칼럼-매핑-레퍼런스)


<br>

## 1) @Entity

#### @Entity 속성(기본값)
1) `name`(클래스 이름): 엔티티 이름 지정. 이름이 같은 엔티티 클래스와 충돌이 발생하지 않도록 주의

> ### 🚨 @Entity 적용 시 주의사항
> * <ins>기본 생성자는 필수</ins> (생성자가 없을시 기본 생성자 자동 생성)
> * final 클래스, enum, interface, inner 클래스에는 사용 불가
> * 저장할 필드에 final 사용 불가

<br>

## 2) @Table

#### @Table 속성(기본값)
1) `name`(엔티티 이름): 매핑할 테이블 이름
2) `catalog`: catalog 기능이 있는 데이터베이스에서 catalog를 매핑
3) `schema`: schema 기능이 있는 데이터베이스에서 schema를 매핑
4) `uniqueConstraints`: 유니크 제약조건을 만든다.(2개 이상의 복합 유니크 제약조건도 가능)

<br>

## 3) 다양한 매핑 사용

```java
@Enumerated(EnumType.STRING)
private RoleType roleType;      // 1️⃣ 자바의 enum 타입 사용

@Temporal(TemporalType.TIMESTAMP)
private Date createDate;        // 2️⃣ 자바의 날짜 타입과 매핑

@Lob
private String description;     // 3️⃣ CLOB, BLOB 매핑
```

<br>

## 4) 데이터베이스 스키마 자동 생성

JPA는 데이터베이스 스키마를 자동으로 생성하는 기능을 지원한다.
```yaml
# yaml
jpa:
  hibernate:
    ddl-auto: create
```
```properties
# properties
jpa.hibernate,ddl-auto: create
```

* 자동으로 생성되는 DDL은 지정한 데이터베이스 방언에 따라 달라짐
* 스키마 자동생성 기능을 사용하면 애플리케이션 실행 시점에 데이터베이스 테이블이 자동으로 생성된다.

> ### 💡 jpa.hibernate,ddl-auto 속성
> 1) `create`: 기존 테이블을 삭제하고 새로 생성한다. (DROP + CREATE)
> 2) `create-drop`: create 속성에 추가로 애플리케이션을 종료할 때 테이블을 삭제한다. (DROP + CREATE + DROP)
> 3) `update`: 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 변경 사항만 추가한다.
> 4) `validate`: 데이터베이스 테이블과 엔티티 매핑정보를 비교해서 차이가 있으면 경호를 남기고, 애플리케이션을 실행하지 않는다. DDL을 수정하지 않는다.
> 5) `none`: 자동 생성기능 사용하지 않음


> ### 🚨 HBM2DDL 주의사항
> * 개발 초기 단계: `create` 또는 `update`
> * 자동화된 테스트 개발환경과 CI 서버: `create` 또는 `create-drop`
> * 테스트 서버: `update` 또는 `validate`
> * 스테이징과 운영 서버: `validate` 또는 `none`

<br>

## 5) DDL 생성 기능
```java
@Entity(name = "Member")
@Table(name = "MEMBER", uniqueConstraints = {           // 2️⃣ 유니크 제약조건
        @UniqueConstraint(
                name = "NAME_AGE_UNIQUE",
                columnNames = {"NAME", "AGE"}
        )
})
public class Member {
    @Id 
    @Column(name = "ID")
    private String id;
    
    @Column(name = "NAME", nullable = false, length = 10)   // 1️⃣ null 불가
    private String username;
}
```
👉 이런 기능은 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.

<br>

## 6) 기본 키 매핑

> #### JPA가 제공하는 기본키 생성 전략
> * **직접 할당** : 기본 키를 애플리케이션에서 직접 할당
> * **자동 생성** : 대리키 사용방식
>   * `IDENTITY`: 기본 키 생성을 데이터베이스에 위임
>   * `SEQUENCE`: 데이터베이스 시퀀스를 사용해서 기본키를 할당
>   * `TABLE` : 키 생성 테이블을 사용

- 오라클 데이터베이스는 시퀀스를 제공하지만 MySQL은 시퀀스를 제공하지 않는다 대신 기본키를 자동으로 채우는 `AUTO_INCREMENT` 제공
- `TABLE` 전략은 키 생성용 테이블을 하나 만들어두고 시퀀스처럼 사용 👉 모든 데이터베이스에서 사용
- `@id`: **기본키 직접 할당**
- `@GeneratedValue`: **자동 생성 전략**

<br>

### 1️⃣ 기본키 직접 할당
```java
@Id 
@Column(name = "id")
private String id;
```
* `@Id` 적용가능 자바 타입: 자바 기본형, 자바 래퍼형, String, Date, BigDecimal, BigInteger

```java
Board board = new Board();
board.setId("id1");     // 기본키 직접 할당
em.persist(board);      // 엔티티를 저장하기 전에 기본키를 직접 할당
```

<br>

### 2️⃣ `IDENTITY` 자동 생성 전략
* `IDENTITY`는 기본키 생성을 데이터베이스에 위임하는 전략이다. (MySQL, PostgreSQL, SQL Server, DB2)
* `AUTO_INCREMENT`는 데이터베이스에 값을 저장할 때 ID 칼럼을 비우면 데이터베이스가 순서대로 값을 채운다.
* 데이터베이스에 값을 저장하고 나서야 기본키 값을 구할 수 있을 때 사용

```java
@Entity
public class Board {
    @Id 
    @GeneratedValue(strategy = GenercationType.IDENTITY)
    private Long id;
}
```
<br>

> ### 💡 `IDENTITY` 전략과 최적화
> 엔티티에 식별자 값을 할당하려면 JPA는 추가로 데이터베이스를 조회해야한다.<br>
> 그러나, `Statement.getGeneratedKeys()` 사용하면 데이터 저장 + 기본키 값 조회 <ins>(한번만 통신)</ins>

<br>

> ### 🚨 `IDENTITY` 전략과 영속상태
> `IDENTITY` 식별자 생성 전략은 엔티티를 데이터베이스에 저장해야 식별자를 구할 수 있다. <br>
> ➡︎ em.persist() 호출시 INSERT SQL이 전달 <br>
> ➡︎ <ins>**쓰기 지연이 발생하지 않음**</ins>
 
<br>

### 3️⃣ `SEQUENCE` 자동 생성 전략
* 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트 (오라클, PostgreSQL, DB2, H2)
```sql
CREATE SEQUENCE BOARD_SEQ START WITH 1 INCREMENT BY 1;  -- 시퀀스 생성
```
```java
@Entity
@SequenveGenerator(
        name = "BOARD_SEQ_GENERATOR",
        sequenceName = "BOARD_SEQ",              // 매핑할 데이터베이스 시퀀스 이름
        initialValue = 1, allocationSize = 1
)
public class Board {
    @Id 
    @GeneratedValue(
            strategy = GenerationType.SEQUENCE,
            generator = "BOARD_SEQ_GENERATOR"   // 시퀀스 생성기 매핑
    )       
    private String id;
    
    @Column(name = "NAME", nullable = false, length = 10)   // 1️⃣ null 불가
    private String username;
}
```

> ### 🚨 `SEQUENCE` 전략과 영속상태
> `SEQUENCE`은 em.persist() 호출시 테이터베이스 시퀀스를 사용해서 식별자 조회 <br>
> ➡︎ 식별자를 엔티티에 할당한 후 엔티티를 영속성 컨텍스트에 저장 <br>
> ➡︎ 트랜잭션을 커밋해서 플러시가 일어나면 엔티티를 데이터베이스에 저장


<br>

#### @SequenceGenerator 속성(기본값)
1) `name`(필수): 식별자 생성기 이름
2) `sequenceName`(hibernate_sequence): 데이터베이스에 등록되어 있는 시퀀스 이름
3) `initialValue`(1): 시퀀스 DDL을 생성할 때 처음 시작하는 수를 지정
4) `allocationSize`(50): 시퀀스 한번 호출에 증가하는 수(성능 최적화)
5) `catalog`, `schema`: 데이터베이스 catalog, schema 이름

```sql
create sequence [sequenceName]
start with [initialValue] increment by [allocationSize]
```

<br>

> ### 💡 `SEQUENCE` 전략과 최적화
> 시퀀스를 통해 식별자를 조회하는 추가 작업이 필요하다. <ins>(2번 통신)</ins> <br>
> 그러나, `@SequenceGenerator.allocaionSize` 사용하면 한번에 시퀀스 값을 증가시키고 나서 메모리에 시퀀스 값을 증가 시킨다. <br>
> 시퀀스 값을 선점하므로, 기본키 값이 충돌하지 않는다.

<br>

### 4️⃣ `SEQUENCE` 자동 생성 전략
* 키 생성 전용 테이블을 만들고 데이터베이스 시퀀스를 흉내 (모든 데이터베이스)
* `SEQUENCE` 전략과 내부 동작이 같다.
```sql
create table MY_SEQUENCES (
    sequence_name varchar(255) not null,
    next_val bigint,
    primary key ( sequence_name )
)
```
```java
@Entity
@TableGenerator(
        name = "BOARD_SEQ_GENERATOR",
        table = "MY_SEQUENCES",              // 매핑할 데이터베이스 테이블 이름
        pkColumnValue = "BOARD_SEQ", allocationSize = 1
)
public class Board {
    @Id 
    @GeneratedValue(
            strategy = GenerationType.TABLE,
            generator = "BOARD_SEQ_GENERATOR"   // 시퀀스 생성기 매핑
    )       
    private String id;
    
    @Column(name = "NAME", nullable = false, length = 10)   // 1️⃣ null 불가
    private String username;
}
```

#### @TableGenerator 속성(기본값)
1) `name`(필수): 식별자 생성기 이름
2) `table`(hibernate_sequence): 키 생성 테이블명
3) `pkColumnName`(sequence_name): 시퀀스 컬럼명
4) `valueColumnName`(next_val): 시퀀스 값 컬럼명
3) `initialValue`(0): 초기 값, 마지막으로 생성된 값이 기준
4) `allocationSize`(50): 시퀀스 한번 호출에 증가하는 수(성능 최적화)
5) `catalog`, `schema`: 데이터베이스 catalog, schema 이름
6) `uniqueConstraints(DDL)`: 유니크 제약 조건
   
<br>

> ### 💡 `Table` 전략과 최적화
> 값을 조회하면서 Select 쿼리를 사용하고, 값을 증가시키기 위히 update쿼리를 사용한다. <ins>(`SEQUENCE`보다 한번 더 통신)</ins><br>
> 그러나, `@TableGenerator.allocaionSize` 를 사용하면 `SEQUENCE`최적화 방식처럼 사용할 수 있다.

<br>

### 5️⃣ `AUTO` 자동 생성 전략
* 선택한 데이터베이스 방언에 따라 `IDENTITY`, `SEQUENCE`, `TABLE` 선택
* 데이터베이스를 변경해도 코드를 수정할 필요가 없다. (개발 초기시 주로 사용)
```java
@Entity
public class Board {
@Id
@GeneratedValue(strategy = GenercationType.AUTO)
private Long id;
}
```
<br>

> ## 💡기본키 매핑 정리
> * **직접 할당** : em.persist()를 호출하기 전에 애플리케이션에서 직접 식별자 값을 할당해야한다. 만약 식별자 값이 없으면 예외가 발생한다.
> * `IDENTITY`: 데이터베이스에 엔티티를 저자해서 식별자 값을 획득후 영속성 컨텍스트에 저장
> * `SEQUENCE`: 데이터베이스 시퀀스에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장
> * `TABLE` : 데이터베이스 시퀀스 생성용 테이블에서 식별자 값을 획득한 후 영속성 컨텍스트에 저장

> ### 📢 권장하는 식별자 선택 전략
> * 데이터베이스 기본키 조건: 1️⃣ null값 불가 2️⃣ 유일 3️⃣ 불변
> * 테이블 기본키 선택전략 1️⃣ **자연 키**(비즈니스에 의미), 2️⃣ **대리 키(대체 키)**(비즈니스에 의미가 없음)<ins>(대리키 사용 권장)</ins>
> * 자연키의 후보가 되는 것은 **유니크 인덱스** 설정 후 사용 권장


<br>

## 7) 필드와 컬럼 매핑: 레퍼런스

### 1️⃣ `@Column`
#### @Column 속성(기본값)
1) `name`(객체 필드 이름): 매핑할 테이블의 컬럼 이름
2) `insertable`(true): 엔티티 저장시 필드 저장 (false는 읽기 전용)
2) `updateable`(true): 엔티티 수정시 필드 수정 (false는 읽기 전용)
3) `table`(현재 클래스 테이블): 지정한 필드를 다른 테이블에도 매핑가능
4) `nullable`(true): null값 허용 여부
5) `unique`: 한 칼럼에 간단한 유니크 제약조건을 걸 때 사용
6) `length`(255): 문자길이 제약조건, String에만 사용
7) `precision`(19),`scale`(2): BigDecimal에 사용, presicion(자릿수), scale(소수의 자릿수)

> ### 📢 `@Column` 생략
> * int 타입으로 필드 생성시 not null 설정
> * Integer 타입으로 필드 생성시 null 가능 설정
> * @Column + int 타입으로 필드 생성시 null 가능 설정

<br>

### 2️⃣ `@Enumerated`
#### @Enumerated 속성(기본값)
1) `value`(EnumType.ORDINAL)
   * `EnumType.ORDINAL`: enum 순서를 데이터베이스에 저장 (저장되는 크기가 작으나 저장된 enum 순서 변경 불가)
   * `EnumType.STRING`: enum 이름을 데이터베이스에 저장 (확장성이 좋으며, 데이터 크기가 크다)

<br>

### 3️⃣ `@Temporal`
#### @Temporal 속성(기본값)
1) `value`(필수 지정)
    * `TemporalType.DATE`: 날짜, 데이터베이스 date타입과 매핑
    * `TemporalType.TIME`: 시간, 데이터베이스 time타입과 매핑
    * `TemporalType.TIMESTAMP`: 날짜와 시간, 데이터베이스 timestamp타입과 매핑

> * `datetime`: MySQL
> * `timestamp`: H2, 오라클, PostgreSQL


<br>

### 4️⃣ `@Lob`
#### @Lob 속성 정리
* `CLOB`: String, char[], java.sql.CLOB
* `BLOB`: byte[], java.sql,BLOB

<br>

### 5️⃣ `@Transient`
객체에 따로 보관하고 싶은 필드가 있을 때 사용한다. 데이터베이스에 저장 및 조회하지 않는다.

<br>

### 6️⃣ `@Access`
엔티티가 데이터에 접근하는 방식을 지정한다.
* `AccessType.FIELD`**필드 접근**: 필드에 `@Id`를 사용해 직접 접근한다.
* `AccessType.PROPERTY`**프로퍼티 접근**: 접근자에 `@Id`를 사용해 접근한다.