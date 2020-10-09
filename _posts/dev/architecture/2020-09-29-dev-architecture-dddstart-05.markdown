---
layout: post
title: "[DDD-START] Ch05. Repository"
subtitle: "Repository의 조회기능(JPA중심)"
categories: dev
tags: architecture
comments: false
---





Ch4는 읽었는데 잘 와닿지 않았다. 아직 JPA도 공부를 안한 상태라 그런지 내가 습득하기는 좀 어려운 거 같다.

Ch5도 여차 비슷했는데, 사실 부끄러지만 책 내용을 읽으면서 Spring Data JPA쓰면 이런거 다 구현안해도 되지않나

왜 안쓰고 이렇게 직접 다 구현해야하나 이런 생각을 종종하면서 읽고 있었는데

이 챕터 뒷부분에 저자님이 이런이런거 Spring Data JPA를 사용하면 일일이 직접 구현 안해도 된다고 써주셨다.

알고 쓰는거랑 모르고 쓰는거랑은 아예 다른거니까.. 일단 한권을 훑어서라도 읽고 싶어서 이해안가는 부분은 추후 다시 봐야겠다.

JPA를알고 접근하면 좀더 이해하기 좋을거같기도하고....

---

### Spec

#### 검색을 위한 스펙

검색 조건이 다양해지면 각 조회 별로 findByXX 메서드를 정의하기 어렵다.

~~(검색 조건이 몇개가 될지알고?)~~

SQL을 보다보면 수십개의 AND, OR절로 연결된 부분이 종종 있던데 이런건 어떻게 해야되나?  
이 경우 Specification 인터페이스를 정의하고, 필요한 Spec을 직접 구현하여 연결 조건을 하나의 메서드로 처리할 수 있다.

```java
// 스펙 인터페이스
public interface Specification <T> {
    public boolean isSatisfiedBy(T agg);
}
// 스펙 구현체
public class OrdererSpec implements Specification<Order> {
    private String ordererId;
    public OrdererSpec(String ordererId) { this.ordererId = ordererId; }
    public boolean isSatisfiedBy(Order agg) {
        // Order 애그리거트 객체로부터 주문자Id를 참조할 수 있고, 두 값을 비교해서 일치할경우 true
        return agg.getOrdererId().getMemberId().getId().equals(ordererId);1
    }
}
// 스펙을 사용하는 리포지터리
public class MemoryOrderRepository implements OrderRepository {
    public List<Order> findAll(Specfication spec) {
        // 먼저 모든 주문 데이터를 조회한 뒤
        List<Order> allOrders = findAll();1
        // 스펙의 조건을 충족시키는~~(여기서는 주문자의 Id가 일치하는)~~ 주문만 필터링해서 반환한다. 
        return allOrders.stream().filter(order -> spec.isSatisfiedBy(order)).collect(toList());
    }
}
```

---

#### JPA를 위한 스펙

위 방식의 단점이 하나 있는데, 전체 루프를 2회 순회한다.

-   전체 주문건 조회시 전체 순회 1회
-   스펙에 조건을 충족하는 주문 필터링시 전체 순회 1회

따라서 위 방식처럼 구현하지 않고 실제 SQL에서 사용한는거처럼 where절을 구현해 필요없는 데이터는 걸러야 성능을 잡을 수 있다.

-   CriteriaBuilder TODO
-   Predicate TODO

아래와 같이 스펙 생성을 위한 팩토리를 생성할 수 있다.

```java
// Spec Factory Class
public class Spec {
    public static <T> Spectification<T> and(Spectification<T> ... specs) {
        // And연산 스펙 구현체
        return new AndSpecfication<>(specs);
    }
    public static <T> Spectification<T> or(Spectification<T> ... specs) {
        // Or연산 스펙 구현체
        return new OrSpecfication<>(specs);
    }
}
```

---

#### @SubSelect

Hibernate JPA 확장 기능  
Class 앞에 @Subselect("~~QUERY문~~")를 선언하여 Queryr결과를 Entity로 매핑할 수 있다.  
단, 읽기전용으로만 사용된다. @Subselect에 의해 매핑된 Entity객체를 수정하면 JPA는 Update를 실행하게 되는데  
해당 Entity는 테이블이 아니므로 Update할 대상이 없다. 따라서 에러가 발생하게 된다.  
@Immutable은 이 문제 예방을 위해 해당 Entity를 불변으로 만들어준다. (~~Entity가 수정되어도 무시하고 DB 반영X)

```java
// @Subselect에 기술한 쿼리의 결과가 OrderSummary Entity에 매핑된다.
@Entity
@Immutable
@Subselect("select .......")
@Synchronize
public class OrderSummary {
    @Id
    private String number;
    private String ordererId;
    ......
}
```



<hr>

> DDD-START (최범균님 저) 도서 참조

