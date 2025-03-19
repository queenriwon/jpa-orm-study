# Chapter.5 연관관계 매핑 기초

> ### 📋 차례
> [1) 단방향 연관관계](#1-단방향-연관관계) <br>
> [2) 연관관계 사용](#2-연관관계-사용) <br> 
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[i. 저장](#1-저장) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[ii. 조회](#2-조회) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[iii. 수정](#3-수정) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[iv. 연관관계 제거](#4-연관관계-제거) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[v. 삭제](#5-삭제) <br>
> [3) 양방향 연관관계](#3-양방향-연관관계) <br>
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[i. 조회](#1-조회) <br>
> [4) 연관관계 주인](#4-연관관계-주인) <br>
> [5) 양방향 연관관계 저장](#5-양방향-연관관계-저장) <br>
> [6) 양방향 연관관계의 주의점](#6-양방향-연관관계의-주의점)


> ### 📢 연관관계 매핑 키워드
> 1) 방향: **단방향**, **양방향**
> 2) 다중성: **다대일(N:1)**, **일대다(N:1)**, **일대일(1:1)**, **다대다(N:M)**
> 3) 연관관계의 주인: 양방향으로 만들시 연관관계의 주인 설정

<br>

## 1) 단방향 연관관계

_예시) 회원과 팀이 (다대일 관계라면)_
* `객체` 연관관계
  * 회원 객체는 Member.team <ins>**필드**</ins>로 팀 객체와 연관관계를 맺는다.
  * **`단방향 관계`**
    * member ➡︎ team : 회원은 Member.team필드로 팀 인지 ⭕️
    * team ➡︎ member : 팀은 회원 인지 ❌

  ➡︎ `객체`는 `참조(주소)`로 연관관계를 맺으며 연관관계는 `단방향`이다. 
* `태이블` 연관관계
  * 회원 테이블은 TEAM_ID <ins>**외래키**</ins>로 팀 테이블과 연관관계를 맺는다.
  * **`양방향 관계`**
    * member ➡︎ team : 회원 테이블은 TEAM_ID 외래키를 통해 팀 테이블과 조인 ⭕️
    * team ➡︎ member : 팀 테이블은 TEAM_ID 외래키를 통해 회원 테이블과 조인 ⭕️

  ➡︎ `테이블`은 `외래키와 조인`으로 연관관계를 맺으며 연관관계는 `양방향`이다.

<br>
<span style="font-size:14px;"> 👉 참조를 통한 연관관계는 언제나 단뱡항이므로, 객체간 연관관계를 양방향으로 만들고 싶으면 반대쪽에도 필드를 추가해 참조를 해야한다.</span>

**객체 그래프 탐색** : 객체는 참조를 사용해서 연관관계 탐색

```java
// 객체 연관관계
public class Member (
        
    @Id 
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;
    
    // 연관관계 매핑
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;      // 팀의 참조를 보관
)

public class Team (

    @Id
    @Column(name = "TEAM_ID")
    private String id;
    private String name;
)
```

```sql
-- 테이블 연관관계
CREATE TABLE MEMBER (
    MEMBER_ID VARCHAR(255) NOT NULL,
    USERNAME VARCHAR(255),
    TEAM_ID VARCHAR(255),
    PRIMARY KEY (MEMBER_ID)
)

CREATE TABLE MEMBER (
    MEMBER_ID VARCHAR(255) NOT NULL,
    USERNAME VARCHAR(255),
    TEAM_ID VARCHAR(255),
    PRIMARY KEY (MEMBER_ID)
)
```

* `@ManyToOne`: 다대일 매핑 정보. 연관관계 매핑시 사용
* `@JoinColumnn(name = "TEAM_ID")`: 외래키를 매핑할 때 사용. 생략가능

#### @ManyToOne 속성(기본값)
1) `optional`(true): false로 설정하면 연관된 엔티티가 항상 있어야 함
2) `fetch`: fetch 전략 사용
   * `FetchType.EAGER`: `@ManyToOne`에서 기본값
   * `FetchType.LAZY`: `@OneToMany`에서 기본값
3) `cascade`: 영속성 전이 기능 사용

#### @JoinColumn 속성(기본값)(생략가능)
1) `name`(필드명_기본키칼럼명): 매핑할 외래 키 이름
2) `referencedColumnName`(참조하는 테이블의 기본키 칼럼명): 외래키가 참조하는 댇상 테이블의 칼럼명
3) `foreignKey`: 외래키 약조건 직접 지정


<br>

## 2) 연관관계 사용

### 1️⃣ 저장
```java
Team team1 = new Team("team1", "팀1");
em.persist(team1);

Member member1 = new Member("member1", "회원1");
member1.setTeam(team1);             // 연관관계 설정 member1 -> team1
em.persist(member1);                // 저장 -> 쿼리에 식별자로 저장됨
```
<br>

### 2️⃣ 조회
* 객체 그래프 탐색
```java
Member member = em.find(Member.class, "member1");
Team team = member.getTeam();       // 객체 그래프 탐색
```
* 객체지향 쿼리 사용 (JPQL)
```java
String jpql = "select m from Member m join m.team t where " +
    "t.name = :teamName";                   // jpql 문법

List<Member> resultList = em.createQuery(jpql, Member.class)
    .setParameter("teamName", "팀1")         // 파라미터 바인딩
    .getResultList();
```

<br>

### 3️⃣ 수정
```java
Team team2 = new Team("team2", "팀2");
em.persist(team2);

Member member = em.find(Member.class, "member1");       // 조회
member.setTeam(team2);      // 수정
```
➡︎ **<ins>변경감지</ins>**: 불러온 엔티티의 값만 변경하면 트랜잭션을 커밋할때 플러시가 일어나며 변경 사항 데이터베이스에 자동 반영

<br>

### 4️⃣ 연관관계 제거
```java
Member member1 = em.find(Member.class, "member1");
member1.setTeam(null);      // 연관관계 제거
```

### 5️⃣ 삭제
연관된 엔티티를 삭제하려면 기존에 있던 연관관계를 먼저 제거하고 삭제해야 한다. (외래키 제약조건)
```java
member1.setTeam(null);      // 연관관계 제거
em.remove(team);            // 팀 삭제
```

<br>

## 3) 양방향 연관관계

_예시) 회원과 팀이 (다대일 관계라면)_
* `객체` 양방향 연관관계
    * 회원과 팀은 다대일 관계, 팀과 회원은 일대다 관계 (컬렉션 사용)
    * member ➡︎ team : Member.team
    * team ➡︎ member : Team.member
* `태이블` 양방향 연관관계
    * 데이터베이스 테이블은 외래키 하나로 양방향으로 조회 가능

```java
// 객체 연관관계
public class Member (
        
    @Id 
    @Column(name = "MEMBER_ID")
    private String id;
    private String username;
    
    // 연관관계 매핑
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;      // 팀의 참조를 보관
)

public class Team (

    @Id
    @Column(name = "TEAM_ID")
    private String id;
    private String name;
    
    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<Member>(); // 멤버의 참조(양방향)
)
```

### 1️⃣ 조회
회원 컬렉션으로 객체 그래프 탐색
```java
Team team = em.find(Team.class, "team1");
List<Member> members = team.getMembers();   // 객체 그래프 탐색
```

<br>

## 4) 연관관계의 주인

* `객체` 양방향 연관관계
  * member ➡︎ team : `단방향`
  * team ➡︎ member : `단방향`
* `태이블` 양방향 연관관계
  * member ↔️ team : `양방향`

* 테이블은 외래 키 하나로 연관관계를 관리하는 한편, 객체는 엔티티를 단방향으로 매핑하면 참조를 하나만 사용하게 되며, 참조로 외래키를 관리한다.
* 따라서, 객체의 연관관계를 관리하는 포인트가 2곳이다. <ins>참조는 둘인데, 외래 키는 하나</ins>
* **연관관계의 주인** : 두 객체 연관관계 중 하나를 저해서 테이블의 외래 키 관리 <ins>(외래 키 관리자)</ins>
  * 연관관계의 주인⭕️: 데이터 베이스 연관관계를 매핑 + 외래키 관리(등록, 수정, 삭제)
  * 연관관계의 주인❌(mappedBy): 읽기만 가능
* **연관관계의 주인을 정할 때는 테이블의 관점으로 외래 키가 있는 곳에 설정**

> ### 📢 `@ManyToOne`의 `mappedBy` 속성
> 데이터베이스 테이블의 다대일, 일대다 관계에서는 항상 **다 쪽이 외래 키**를 가진다.
> 다 쪽인 `@ManyToOne`은 항상 연관관계의 주인이므로 `mappedBy`를 설정할 수 없다. <br>
> ➡︎ `@ManyToOne`에는 `mappedBy` 속성이 없다.

<br>

## 5) 양방향 연관관계 저장
* 양방향 연관관계는 주인이 외래 키를 관리한다.<br> 👉 주인이 아닌 방향은 값을 설정하지 않아도 데이터베이스에 외래 키 값이 자동 관리

<br>

## 6) 양방향 연관관계의 주의점
* 연관관계의 주인이 외래 키의 값을 변경할 수 있기 때문에, 연관관계의 주인이 되는 쪽에 데이터를 저장 및 수정해야 한다.
* 객체 고나점에서는 양쪽 방향 모두 값을 입력해주는 것이 가장 안전하다.
```java
member1.setTeam(team);              // 연관관계 설정: member1 -> team1
team1.getMembers().add(member1);    // 연관관계 설정: team1 -> member1
em.persist();
```
* <ins>관계 편의 메서드</ins>: 한번에 양방향 관계를 설정하는 메서드 작성
```java
public void setTeam(Team team) {
    this.team = team;              // 연관관계 설정: member -> team
    team.getMembers().add(this);   // 연관관계 설정: team -> member
}
```
* 연관관계를 양방향으로 연결했다면, 연관관계를 삭제할 때도 양방향으로 삭제해야한다.
```java
public void setTeam(Team team) {
    // 기존 연관관계 객체와 관계 제거(team -> member 연관관계는 제거할 필요 없음)
    if (this.team != null) {
        this.team.getMembers().remove(this);
    }
    this.team = team;              // 연관관계 설정: member -> team
    team.getMembers().add(this);   // 연관관계 설정: team -> member
}
```