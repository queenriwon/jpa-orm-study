# Chapter.2 JPA 시작 

> ### 📋 차례
> [1) 객체 매핑 시작](#1-객체-매핑-시작)<br>
> [2) persistence.xml 설정](#2-persistencexml-설정)<br>
> [3) 애플리케이션 개발](#3-애플리케이션-개발)

<br>

## 1) 객체 매핑 시작

```java
@Entity
@Table(name = "MEMBER")
public class Member {
    
    @Id
    @Column(name = "ID")
    private String id;
    
    @Column(name = "NAME")
    private String username;
    
    private Integer age;
}
```

| Member   | @Table(name = "MEMBER") | MEMBER  |
|----------|:-----------------------:|---------|
| id       |           @id           | ID (PK) |
| username |  @Column(name = "NAME") | NAME    |
| age      |                         | AGE     |

<br>

* @Entity: 클래스를 테이블과 매핑 (엔티티 클래스)
* @Table: 엔티티 클래스와 매핑할 테이블 정보
* @Id: 엔티티 클래스의 필드를 테이블의 기본 키에 매핑 (식별자 필드)
* @Column: 필드를 칼럼에 매핑
* 매핑 정보가 없는 필드: 매핑 어노테이션을 생략하여 필드명으로 컬럼명 매핑


<br>

## 2) persistence.xml 설정

* 영속성 유닛: 연결할 베이스당 하나의 영속성 유닛 등록
> * JPA 표중 속성
>   * javax.persistence.jdbc.driver: JDBC 드라이버
>   * javax.persistence.jdbc.user: 데이터베이스 접속 아이디
>   * javax.persistence.jdbc.password: 데이터베이스 접속 비밀번호
>   * javax.persistence.jdbc.url: 데이터베이스 접속 URL
> * 하이버네이트 속성
>   * hibernate.dialect: 데이터베이스 방언 설정
>   * hibernate.show_sql: 하이버네이트가 실행한 SQL을 출력
>   * hibernate.format_sql: 하이버네이트가 실행한 SQL을 출력할 때 보기 쉽게 출력
>   * hibernate.use_sql_comments: 쿼리를 출력할 때 주석도 함께 출력
>   * hibernate.id.new_generator_mappings: JPA 표준에 맞춘 새로운 키 생성 전략 사용


<br>

## 3) 애플리케이션 개발

```java
// [엔티티 매니저 팩토리] - 생성
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");

// [엔티티 매니저] - 생성
EntityManager em = emf.createEntityManager();

// [트랜잭션] - 획득
EntityTransaction tx = em.getTransaction();

try {
    tx.begin();     // [트랜잭션] - 시작
    logic(em);      // 비즈니스 로직
    tx.commit();    // [트랜잭션] - 커밋
} catch (Exception e) {
    tx.rollback();  // [트랜잭션] - 롤백
} finally {
    em.close();     // [엔티티 매니저] - 종료
}

emf.close();        // [엔티티 매니저 팩토리] - 종료
```
<br>

* 엔티티 매니저 설정
  1) `앤티티 매니저 팩토리` 생성
     * JPA를 동작시키기 위한 기반 도구를 만들고 JPA 구현체에 따라 데이터베이스 커넥션 풀 생성 👉 <ins>생성 비용이 크다</ins>
     * **엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한번만 생성하고 공유해서 사용해야 함**
  2) `엔티티 매니저` 생성
     * 엔티티 매니저를 사용해서 엔티티를 **데이터베이스에 등록/수정/삭제/조회**
     * 데이터베이스 커넥션과 밀접한 관계가 있으므로 <span style="color:red;">스레드간 공유하거나 재사용 불가</span>
  3) 종료
     * 사용이 끝난 `앤티티 매니저 팩토리`와 `앤티티 매니저`는 종료해야 한다.

<br>

* 트랜잭션 관리
  1) 트랜잭션 커밋: 비즈니스 로직 정상 동작
  2) 트랜잭션 롤백: 예외 발생시 동작

<br>

* 비즈니스 로직
  1) 등록
     * 회원 엔티티를 생성하고 저장
    ```java
  String id = "id1";
  Member member = new Member();
  member.setId(id);
  member.setUsername("지환");
  member.setAge(2);
  
  em.persist(member);       // 1️⃣ 등록
    ```
    ```sql
  INSERT INTO MEMBER (ID, NAME, AGE) VALUES ('id1', '지한', 2)    -- 등록
    ```
  2) 수정
    * 더티체킹을 통해 데이터 베이스 값 수정
    ```java
  member.setAge(20);        // 2️⃣ 수정
    ```
    ```sql
  UPDATE MEMBER
    SET AGE=20, NAME='지한'
  WHERE ID='id1'
    ```
  3) 삭제
    ```java
  em.remove(member);
  ```
    ```sql
  DELETE FROM MEMBER WHERE ID = 'id1'
    ```
  4) 한건 조회
    ```java
  Member findMember = em.find(Member.class, id);
  ```
    ```sql
  SELECT * FROM MEMBER WHERE ID='id1'
    ```
  
<br>

> ### JPQL
> * JPQL은 엔티티 객체를 대상으로 쿼리한다. 클래스와 필드를 대상으로 쿼리문을 작성한다.
> * SQL은 데이터베이스 테이블을 대상으로 쿼리한다.
> 
> 👉 <ins>JPQL은 데이터베이스 테이블을 전혀 알지 못한다.</ins>