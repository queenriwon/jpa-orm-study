# Chapter.3 영속성 관리

> ### 📋 차례
> [1) 엔티티 매니저 팩토리와 엔티티 매니저](#1-엔티티-매니저-팩토리와-엔티티-매니저)<br>
> [2) 영속성 컨텍스트란?](#2-영속성-컨텍스트란)<br>
> [3) 엔티티와 생명주기](#3-엔티티와-생명주기)<br>
> [4) 영속성 컨텍스트의 특징](#4-영속성-컨텍스트의-특징)<br>
> [5) 플러시](#5-플러시)<br>
> [6) 준영속](#6-준영속)<br>                                 

<br>

## 1) 엔티티 매니저 팩토리와 엔티티 매니저

```java
// [엔티티 매니저 팩토리] - 생성
EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");

// [엔티티 매니저] - 생성
EntityManager em = emf.createEntityManager();
```
* `EntityManagerFactory`: 한 개만 만들어서 애플리케이션 전체에서 공유, 여러 스레드가 동시에 접근해도 안전하므로 서로 다른 스레드 간에 공유 가능
* `EntityManager`: 여러 스레드가 동시에 접근하면 **동시성 문제**가 발생 👉 <span style="color:red;">스레드 간에 절대 공유하면 안된다.</span>
* `EntityManager`는 데이터베이스 연결이 필요한 시점까지 커넥션을 얻지 않는다.(트랜잭션을 시작하면 얻는다.)

<br>

## 2) 영속성 컨텍스트란?
* <span style="color:red;">**영속성 컨텍스트**</span>: 엔티티를 영구 저장하는 환경

#### 영속성 컨텍스트 저장
```java
em.persist(member);
```

<br>

## 3) 엔티티의 생명주기
<img src="https://flaxen-swan-41e.notion.site/image/attachment%3A0d99f17d-213c-4210-9446-a118fb675105%3Aimage.png?table=block&id=1b9b649e-bbbd-80f4-b9a3-dabca2874297&spaceId=765aecbd-0d96-4102-a877-7bb784aaedf1&width=1040&userId=&cache=v2">

> ### 엔티티의 상태
> 1) <span style="color:orange;">**비영속(new/transient)**</span>: 영속성 컨텍스트와 전혀 관계가 없는 상태
> ```java
> Member member = new Member();
> member.setId("member1");
> member.setUsername("회원1");
> ```
> 2) <span style="color:orange;">**영속(menaged)**</span>: 영속성 컨텍스트에 저장된 상태
> ```java
> em.persist(member);
> ```
> 3) <span style="color:orange;">**준영속(detached)**</span>: 영속성 컨텍스트에 저장되었다가 분리된 상태
> ```java
> em.detach(member);
> em.close();       // 영속성 컨텍스트를 닫음
> em.clear();       // 영속성 컨텍스트를 초기화
> ```
> 4) <span style="color:orange;">**삭제(removed)**</span>: 삭제된 상태
> ```java
> em.remove(member);
> ```

<br>

## 4) 영속성 컨텍스트의 특징

* 영속 상태는 식별자 값이 반드시 있어야한다.(id)
* **플러시**: 영속성 컨텍스트에 새로 저장된 엔티티를 데이터베이스에 반영

<br>

### 1️⃣ 1차 캐시
<img src="https://flaxen-swan-41e.notion.site/image/attachment%3Ac3db6cc3-ff4d-4ac7-b688-6d06c4ecad92%3Aimage.png?table=block&id=1b9b649e-bbbd-8026-90f5-f55dd3043b2d&spaceId=765aecbd-0d96-4102-a877-7bb784aaedf1&width=1420&userId=&cache=v2">

* 엔티티를 영속하면 <ins>**1차 캐시에 엔티티를 저장**</ins>한다. 이때 데이터베이스에 엔티티가 저장되지는 않는다.
* 엔티티를 조회하면, <ins>**1차 캐시에 있는 데이터를 조회**</ins>한다.
* 만약 조회했음에도 불구하고 1차 캐시에 해당되는 데이터가 없을 경우 <ins>**DB에서 조회한 후 1차 캐시에 저장**</ins>한다.


<br>

### 2️⃣ 동일성 보장
* JPA는 1차 캐시를 이용하므로 같은 주소값을 가지는 엔티티를 가져올 수 있다. <br> 👉 <ins>영속성 컨텍스트는 성능상 이점과 엔티티의 동일성을 보장</ins>

<br>

### 3️⃣ 트랜잭션을 지원하는 쓰기 지연
<img src="https://flaxen-swan-41e.notion.site/image/attachment%3A7f7e0f87-571f-4e3d-9066-4bda78829f69%3Aimage.png?table=block&id=1b9b649e-bbbd-80bd-8a52-e7e98b0e13f8&spaceId=765aecbd-0d96-4102-a877-7bb784aaedf1&width=1420&userId=&cache=v2">

* 엔티티 매니저는 트랜잭션을 커밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않고 <ins>**쓰기 지연 SQL 저장소**</ins>에 SQL을 쌓아둔다.
* 커밋할 때 모아둔 쿼리를 데이터베이스에 보낸다. 👉 <span style="color:red;">**쓰기 지연**</span>
* 트랜잭션을 커밋하면 엔티티 매니저는 영속성 컨텍스트를 플러시

* 데이터를 저장하면 등록 쿼리를 데이터베이스에 보내지 않고 메모리에 모아둔다. <br> 그리고 트랜잭션을 커밋할 때 모아둔 등록쿼리를 데이터베이스에 보낸 후에 커밋한다.

<br>

### 4️⃣ 변경 감지
* 엔티티의 변경사항을 데이터 베이스에서 자동으로 반영 👉 <span style="color:red;">**변경 감지**</span>
* JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해서 저장<ins>**(스냅샷)**</ins>
<img src="https://flaxen-swan-41e.notion.site/image/attachment%3A237b5be4-4b7f-4a51-967f-17e5dfae12bf%3Aimage.png?table=block&id=1b9b649e-bbbd-805c-a060-e448e5a9196a&spaceId=765aecbd-0d96-4102-a877-7bb784aaedf1&width=1420&userId=&cache=v2">
  1) 트랜잭션을 커밋하면 엔티티 매니저 내부에서 먼저 flush() 호출
  2) 엔티티와 스탭샷을 비교해서 변경된 엔티티를 찾는다.
  3) 변경된 엔티티가 있으면 <ins>**수정 쿼리 생성해서 쓰기 지연 SQL 저장소**</ins>에 보낸다.
  4) 쓰기 지연 저장소의 SQL을 데이터베이스에 보낸다.
  5) 데이터베이스 트랜잭션을 커밋한다.
* 변경감지는 영속상태의 엔티티만 적용된다.
* 수정시 모든 필드를 업데이트 업데이트한다.<ins>**(수정쿼리가 항상 같고 바인딩 값만 조절 할 수 있도록 재사용)**</ins>

> #### @DynamicInsert
> 수정된 데이터만 사용해서 동적으로 UPDATE SQL를 생성한다.(30개 이상 컬럼)


<br>

## 5) 플러시
* 플러시는 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영 <span style="color:red;">(1차 캐시를 지우는 것이 아님!!)</span>
  1) 엔티티를 스냅샷과 비교해서 수정된 엔티티를 찾는다.
  2) 수정된 엔티티는 수정 쿼리를 만들어 쓰기 지연 SQL 저장소에 등록한다.
  3) 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송한다.

> #### 플러시 방법
> 1) em.flush()를 직접 호출
> 2) 트랜잭션 커밋시 플러시가 자동 호출
> 3) JPQL 쿼리 실행시 플러시가 자동 호출

> #### 플러시 모드
> 1) FlushModeType.AUTO: 커밋이나 쿼리를 실행할 때 플러시
> 2) FlushModeType.COMMIT: 커밋할 때만 플러시


<br>

## 6) 준영속
준영속 방법에는 3가지 방법이 있다.
1) em.detach(entity): 특정 엔티티만 준영속 상태로 전환한다.
2) em.clear(): 영속성 컨텍스트를 완전히 초기화한다.
3) em.close(): 영속성 컨텍스트를 종료한다.

* 1차 캐시부터 쓰기지연 SQL 저장소까지 해당 엔티티를 관리하기 위한 모든 정보가 삭제된다.<br><ins>(준영속 상태, 영속성 컨텍스트에서 분리된 상태)</ins>
* 준영속 상태란, 비영속 상태에 가까우며, 식별자 값을 가지고 있다. 또한 지연로딩을 할 수 없다.
* 준영속 상태의 엔티티를 다시 영속 상태로 변경하려면 병합을 사용하면 된다. `merge()`