---
title: "[Spring] Fetch JOIN이란?"
excerpt: "JPA의 Fetch JOIN이란? fetch join의 예시는? spring data jpa에서 fetch join의 활용은? N+1 문제란? 컬렉션 Fetch의 경우는? 다중 Fetch join의 경우는? fetch join 최적화 전략은?"
date: 2024-05-20
last_modified_at: 2024-05-20
categories:
  - java
tags:
  - spring-study
---

## 1. Question

`JPA`의 `Fetch join`이란?

## 2. Answer

`Fetch join`은 `JPA(Java Persistence API)`에서 사용되는 기법으로, 연관된 엔티티들을 함께 로드하여 `N+1` 셀렉트 문제를 해결하는 데 매우 효과적이다. 이 기술은 엔티티와 그 연관된 엔티티들을 한 번의 쿼리로 함께 불러오기 때문에 성능을 크게 향상시킬 수 있다.

### A. Fetch join의 기본 개념

`Fetch join`은 `JPQL` 쿼리 내에서 `JOIN FETCH` 구문을 사용하여 구현된다. 이 구문을 사용하면 `JPA`가 `SQL` 쿼리를 생성할 때 연관된 엔티티들을 같이 불러오도록 지시한다. 이렇게 함으로써, 엔티티에 접근할 때 추가적인 쿼리를 실행하지 않아도 되므로 성능이 향상된다.

### B. Fecth join 예시

예를 들어, `User` 엔티티와 `Order` 엔티티가 있고, 각 사용자는 여러 개의 주문을 가질 수 있다고 가정해본다. 이 경우 사용자와 그의 주문들을 함께 가져오고 싶을 때 `fetch join`을 사용할 수 있다.

```java
SELECT u FROM User u JOIN FETCH u.orders
```

이 쿼리는 모든 사용자(`User`)와 그들의 모든 주문(`Order`)을 단일 쿼리로 함께 가져온다. 따라서 각 사용자에 대해 개별적으로 주문을 로드하기 위한 추가 쿼리가 필요 없게 되어, 데이터베이스에 대한 요청 횟수가 현저히 감소하고 전체적인 성능이 향상된다.

### C. Spring Data JPA에서의 활용

`Spring Data JPA`에서는 `@EntityGraph` annotation을 사용하여 `fetch join`을 수행할 수 있다. 이 annotation을 통해 `JPQL`을 작성하지 않고도 엔티티의 특정 속성을 `Eager Loading` 방식으로 가져올 수 있다.

```java
public interface UserRepository extends JpaRepository<User, Long> {
  @EntityGraph(attributePaths = {"orders"})
  List<User> findAllWithOrders();
}
```

이 메서드를 사용하면 `User` 객체들을 조회할 때 자동으로 `orders` 컬렉션을 `Eager Loading`으로 가져온다. 이렇게 하면 별도의 쿼리 없이 연관된 주문들을 한 번에 로드할 수 있어, 성능 최적화에 도움이 된다.

## 3. Detail

### A. N+1 문제

`N+1` 문제는 데이터베이스 쿼리 최적화에서 흔히 발생하는 문제로, 하나의 쿼리로 여러 개의 주요 엔티티를 가져온 후, 각 엔티티에 연관된 자식 엔티티를 가져오기 위해 추가적으로 N번의 쿼리를 수행하는 상황을 말한다. 여기서 N은 주요 엔티티의 수를 의미한다.

예를 들어, 데이터베이스에 10명의 사용자가 있고, 각 사용자에게 여러 개의 주문이 있을 때, 사용자 목록을 가져오는 하나의 쿼리를 실행한 후 각 사용자의 주문을 가져오기 위해 추가로 10개의 쿼리를 더 실행하게 되면, 총 11개의 쿼리가 실행된다. 이 경우 효율적이지못하며 성능 저하의 주요 원인이 된다.

### B. 컬렉션 Fetch

한 엔티티가 여러 개의 다른 엔티티를 컬렉션으로 가지고 있을 때, 이 컬렉션을 `fetch join`으로 가져오는 경우 결과 집합에 중복된 데이터가 발생할 수 있다. 이는 주로 `OneToMany` 또는 `ManyToMany` 연관 관계에서 발생한다.

예시: 사용자(User)가 여러 주문(Order)을 가지고 있는 경우를 가정해 본다. 각 사용자가 평균 10개의 주문을 가지고 있다면, `fetch join`을 사용하여 사용자와 주문을 함께 조회할 경우, 사용자 정보가 주문의 수만큼 중복되어 결과 집합에 나타난다.

결과: 이러한 중복은 메모리 사용량을 증가시키고, 데이터를 처리하는 시간도 늘리므로 전체적인 성능에 부정적인 영향을 줄 수 있다. 또한, 애플리케이션 단에서 추가적인 로직을 구현하여 중복을 제거해야 할 수도 있다.

### C. 다중 Fetch join

여러 연관 관계를 동시에 `fetch join`하는 경우, `SQL` 쿼리의 복잡도가 증가하고 결과 집합의 크기가 커질 수 있다. 이는 특히 `ManyToOne` 또는 `OneToMany` 연관 관계가 여러 개 있는 복잡한 도메인 모델에서 문제가 될 수 있다.

예시: 사용자(User)가 여러 주문(Order)을 가지고 있고, 각 주문이 여러 상품(Product)을 포함하고 있다고 가정해 본다. 사용자, 주문, 상품을 한 번에 `fetch join` 하려고 할 때, `SQL` 쿼리는 매우 복잡해지고, 각 연관 관계의 데이터가 조합될 때 결과 집합의 크기가 기하급수적으로 증가할 수 있다.

결과: 이런 상황은 데이터베이스 서버에 큰 부하를 주고, 네트워크를 통해 전송되는 데이터량도 증가시킨다. 이는 응답 시간을 길게 하고 전체 시스템의 성능을 저하시킬 수 있다.

### D. 최적화 전략

* **적절한 조인 전략 선택**: 모든 상황에서 `fetch join`을 사용할 필요는 없다. 필요한 데이터만을 선택적으로 `fetch join`하고, 나머지는 지연 로딩(`lazy loading`)을 사용한다.

* **하위 수준의 fetch 제한**: 가능한 한 하위 수준의 엔티티(fetch의 깊이)를 제한하여 너무 많은 데이터가 한 번에 로드되는 것을 방지한다.

* **배치 크기 설정**: 컬렉션 fetch 시, `@BatchSize` annotation을 사용하여 한 번에 로드되는 엔티티의 수를 조정할 수 있다. 이를 통해 네트워크 비용과 데이터베이스 부하를 줄일 수 있다.

## 4. Reference

None