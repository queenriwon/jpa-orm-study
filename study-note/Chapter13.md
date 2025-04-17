# Chapter.13 웹 애플리케이션과 영속성 관리

> ### 📋 차례
> [1) 트랜잭션 범위의 영속성 컨텍스트](#1-트랜잭션-범위의-영속성-컨텍스트) <br>
> [2) 준영속 상태와 지연 로딩](#2-준영속-상태와-지연-로딩) <br>
> [3) OSIV](#3-OSIV) <br>
> [4) 너무 엄격한 계층](#4-너무-엄격한-계층) <br>

<br>

## 1) 트랜잭션 범위의 영속성 컨텍스트

* <ins>**트랜잭션 범위의 영속성 컨텍스트 전략**</ins>: 트랜잭션을 시작할 떄 영속성 컨텍스트를 생성하고 끝날떄 컨텍스트를 종료

<image src="https://flaxen-swan-41e.notion.site/image/attachment%3Af30d498c-1e07-48f3-850a-43105da35dd4%3Aimage.png?table=block&id=1d8b649e-bbbd-807e-bd91-ce91a41ab7f0&spaceId=765aecbd-0d96-4102-a877-7bb784aaedf1&width=1420&userId=&cache=v2">
트랜잭션 범위와 영속성 컨텍스트 생존 범위가 같음

* 트랜잭션 범위의 영속성 컨텍스트 전략의 예시
  * `@Transactional`(스프링 트랜잭션 AOP): 대상 메서드를 호출하기 직전에 트랜잭션 시작, 정상 종료되면 트랜잭션 커밋하며 종료.(메서드 범위 = 트랜잭션 범위)
    * 정상 작동: 트랜잭션 커밋(시작) ➡︎ 영속성 컨텍스트 플러시 ➡︎ 데이터베이스 트랜잭션 커밋(DB반영)
    * 예외 발생: 트랜잭션 롤백(플러시 호출 ❌)

<image src="https://flaxen-swan-41e.notion.site/image/https%3A%2F%2Fultrakain.gitbooks.io%2Fjpa%2Fcontent%2Fchapter3%2Fimages%2FJPA_13_2.png?table=block&id=1d8b649e-bbbd-8007-a615-c286610ef7e8&spaceId=765aecbd-0d96-4102-a877-7bb784aaedf1&width=2000&userId=&cache=v2">

* 트랜잭션 범위의 영속성 컨텍스트 전략
  1) 트랜잭션이 같으면 같은 영속성 컨텍스트 사용
  
<image src="https://flaxen-swan-41e.notion.site/image/attachment%3A7035e659-b8b7-44ec-858d-1c583d67fda1%3Aimage.png?table=block&id=1d8b649e-bbbd-8023-874a-cd3293158cc6&spaceId=765aecbd-0d96-4102-a877-7bb784aaedf1&width=1420&userId=&cache=v2">
  
2) 트랜잭션이 다르면 다른 영속성 컨텍스트를 사용
     * 같은 엔티티 메니저를 사용하낟고 해도 트랜잭션에 따라 영속성 컨텍스트가 다르다.
     * 스프링 컨테이너는 스레드마다 각각 다른 트랜잭션 할당 (멀티스레드 상황에서 안전)

<image src="https://flaxen-swan-41e.notion.site/image/attachment%3A99636052-611a-41c0-8776-3e780f7b170d%3Aimage.png?table=block&id=1d8b649e-bbbd-8055-a9aa-f20a5f6c09d2&spaceId=765aecbd-0d96-4102-a877-7bb784aaedf1&width=1420&userId=&cache=v2">
  

<br>

## 2) 준영속 상태와 지연 로딩
* 조회한 엔티티가 서비스, 레포지토리 계층에서는 영속성 컨텍스트에서 _영속 상태_ / 프리젠테이션 계층(컨트롤러, 뷰)에서는 _준영속 상태(지연 로딩 및 변경 감지 불가)_

> ### 📢 준영속 상태의 지연로딩 문제 해결 방법
>  * 뷰가 필요한 엔티티를 미리 로딩
>    * 글로벌 페치 전략 수정
>    * JPQL 페치 조인
>    * 강제로 초기화
>  * OSIV를 사용해서 엔티티를 항상 영속 상태로 유지

<br>

### 1️⃣ 글로벌 페치 전략
연관된 엔티티를 미리 로딩하여 가져간다.
👉 준영속 상태가 되어도 로딩 가능

```java
@ManyToOne(fetch = FetchType.EAGER)     // 즉시 로딩 전략
```

#### 🚨 글로벌 페치 전략 즉시 로딩 사용 시 단점
1) 사용하지 않는 엔티티를 로딩한다.
2) N+1 문제가 발생한다.
   * JPA가 JPQL을 분석해서 SQL을 생성할 때는 글로벌 페치 전략을 사용하지 않고 자체 JPQL을 사용 👉 즉시 로딩 및 지연 로딩 구분하지 않음
   * <ins>**N+1**</ins>: 처음 조회한 데이터 수만큼 다시 SQL을 사용해서 조회
   * JPQL 페치조인으로 해결 가능

<br>

### 2️⃣ JPQL 페치 조인
호출한 시점에 함께 로딩할 엔티티를 선택할 수 있다. <br>
페치 조인을 사용하면 SQL JOIN을 사용해서 페치 조인 대상까지 함께 조회 👉 N+1 문제가 발생하지 않음

```sql
-- JPQL
select o
from Order o
join fetch o.member

-- SQL
select o.*, m.*
from Order o
join Member m on o.MEMBER_ID=m.MEMBER_ID
```

#### 🚨 JPQL 페치 조인 사용 시 단점
1) 화면에 맞춘 리포지토리 메서드가 증가하여 프리젠테이션 계층이 데이터 접근 계층을 침범
   * 각각 필요한 메서드만 호출하거나, 한 메서드에 페치 조인 사용으로 해결

<br>

### 3️⃣ 강제로 초기화
영속성 컨텍스트가 살아있을 때, 프리젠테이션 계층이 필요한 엔티티를 강제로 초기화해서 반환 <br>
지연로딩 ✚ 서비스 레이어에서 프록시 객체를 강제로 초기화

```java
class OrderService{
    @Transactional
    public Order findOrder(id) {
        Order order = orderRepository.findOrder(id);
        order.getMember().getName();    // 프록시 객체를 강제로 초기화
        return order;
    }
}
```

#### 🚨 강제로 초기화 시 단점
1) 뷰가 필요한 엔티티마다 서비스 계층의 로직을 변경하므로 프리젠테이션 계층이 서비스 계층 침범
   * 해결방법: `FACADE` 계층에서 프록시 초기화를 담당 (논리적인 의존성 분리)
   * `FACADE` 계층의 역할과 특징
     1) 프리젠테이션 계층과 도메인 모델 계층간의 논리적 의존성 분리
     2) 프리젠테이션 계층에서 필요한 프록시 객체 초기화
     3) 서비스 계층을 호출해서 비즈니스 로직을 실행
     4) 리포지토리를 직접 호출해서 뷰가 요구하는 엔티티를 찾는다.


> ### 🚨 준영속 상태와 지연로딩의 문제점
> 1) 초기화 되었는지 아닌지 헷갈릴 수 있음
> 2) 애플리케이션 로직과 뷰가 물리적으로 나누어져는 있지만 논리적으로 의존
> 
> 👉 엔티티가 프리젠테이션 계층에서 준영속 상태이기 때문에 발생

<br>

## 3) OSIV
* <ins>**OSIV**</ins>(Open Session In View): 영속성 컨텍스트를 뷰까지 열어둠.

### 1️⃣ 요청 당 트랜잭션 방식의 OSIV
<ins>**요청 당 트랜잭션**</ins>: 요청이 들어오자마자 트랙잭션 시작, 요청이 끝날 때 트랜잭션 끝남
* 영속성 컨텍스트가 끝까지 살아 있으므로 조회한 엔티티도 영속 상태 유지
* 뷰에서도 지연로딩 가능: 엔티티를 미리 초기화하지 않아도 됨, FACADE 계층 없이도 뷰에 독립적인 서비스 계층 유지 가능

<image src="https://user-images.githubusercontent.com/2491418/84342949-d3586b80-abe1-11ea-99ff-e67647db8354.png">

#### 🚨 요청 당 트랜잭션 사용 시 단점
1) 프리젠테이션 계층이 엔티티를 변경할 수 있음
  * 변경감지 기능이 작동하여 변경된 엔티티를 데이터베이스에 반영
  * 해결방법
    1) 엔티티를 읽기 전용 인터페이스로 제공
    2) 엔티티 레핑: 읽기 전용 메서드가 있는 클래스 제공
    3) DTO만 반환

<br>

### 2️⃣ 스프링 OSIV: 비즈니스 계층 트랜잭션
비즈니스 계층에서 트랜잭션을 사용하는 OSIV
<image src="https://flaxen-swan-41e.notion.site/image/attachment%3A5c5a3b0f-20da-407f-aec5-cc82620bbe33%3Aimage.png?table=block&id=1d8b649e-bbbd-800b-bcc5-f0a13825c805&spaceId=765aecbd-0d96-4102-a877-7bb784aaedf1&width=1180&userId=&cache=v2">

1) 클라이언트 요청이 들어오면 서블릿 필터나, 스프링 인터셉터에서 영속성 컨텍스트를 생성 (트랜잭션 시작 ❌)
2) 서비스 계층에서 `@Transactional`로 트랜잭션을 시작할 때 영속성 컨텍스트를 찾아와서 트랜잭션 시작
3) 서비스 계층이 끝나면 트랜잭션을 커밋하고 영속성 컨텍스트를 찾아 플러시 (영속성 컨텍스트는 종료 ❌)
4) 컨트롤러와 뷰까지 영속성 컨텍스트가 유지되므로 조회한 엔티티는 영속 상태를 유지
5) 서블릿 필터나, 스프링 인터셉터로 요청이 돌아오면 영속성 컨텍스트를 종료

> #### 📢 스프링 프레임워크가 제공하는 OSIV 라이브러리
> * 하이버네이트 OSIV 서블릿 필터: `org.springframework.orm.hibernate4.support.OpenSessionInViewFilter`
> * 하이버네이트 OSIV 스프링 인터셉터: `org.springframework.orm.hibernate4.support.OpenSessionInViewInterceptor`
> * JPA OSIV 서블릿 필터: `org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter`
> * JPA OSIV 스프링 인터셉터: `org.springframework.orm.jpa.support.OpenEntityManagerInViewInterceptor`

#### 트랜잭션 없이 읽기
엔티티를 변경하지 않고 단순히 조회하면 할 때는 트랜잭션이 없어도 됨

> ### 💬 비즈니스 계층 트랜잭션 OSIV 정리
> 1) 영속성 컨텍스트를 프리젠테이션 계층까지 유지
> 2) 프레젠테이션 계층에는 트랜잭션이 없으므로 엔티티 수정 가능
> 3) 프리젠테이션 계층에는 트랜잭션이 없지만 트랜잭션 없이 읽기를 사용하여 지연로딩 가능

> ### 🚨 비즈니스 계층 트랜잭션 OSIV 주의사항
> * 프리젠테이션 계층에서 엔티티를 수정한 직후에 트랜잭션을 시작하는 서비스 계층을 호출하면 문제 발생
> * 같은 영속성 컨텍스트를 여러 트랜잭션이 공유할 수 있어 문제 발생
> 
> * 👉 해결방법: 트랜잭션이 있는 비즈니스 로직을 모두 호출하고 엔티티 변경

> ### 📢 OSIV vs FACADE vs DTO
> FACADE와 DTO에 비해 OSIV는 코드 길이를 생략할 수 있음 <br>
> 그러나 복잡한 화면을 구성할 때는 DTO로 조회하는 것이 효과적

> ### 📢 원격 상황에서의 OSIV
> OSIV는 같은 JVM을 벗어난 원격 상황에서는 사용할 수 없다. <br>
> 외부 API는 DTO로 변환해서 노출하는 것이 안전하다.

<br>

## 4) 너무 엄격한 계층
OSIV를 사용하면 영속성 컨텍스트가 프리젠테이션 계층까지 살아있으므로 미리 초기화할 필요가 없다.<br>
단순한 엔티티 조회는 컨트롤러에서 리포지토리를 직접 호출해도 문제 없다. <br>
👉 OSIV를 사용하면 유연하고 실용적인 관점에서 접근할 수 있음