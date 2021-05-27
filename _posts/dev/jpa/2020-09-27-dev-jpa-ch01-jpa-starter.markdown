---
layout: post
title: JPA
subtitle: What is JPA?
categories: dev
tags: jpa
comments: true
---

## JPA ?

Java Persistence API의 약자로, Java로 구성된 ORM(Object Relational Mapping) 기술 표준으로 객체와 관계형 데이터베이스를 매핑한다.  
개발자는 응용단에서 개발하듯이 객체를 다루면, JPA가 이를 해석해 알맞은 SQL을 작성하여 DB에 반영한다.  
JPA는 응용서비스와 데이터베이스의 중간자 역할을 수행해준다. 
(~~엄연히 말하면 AppService와 JDBC Driver사이에 존재한다.~~)  

![Alt text](/assets/img/dev/jpa/JPA_Define2.jpg)

JPA의 구현체 중 대표적인 예로 Hibernate가 있다. (Spring Data JPA를 사용해도 내부적으로 Hibernate를 사용하게 된다.)  

![Alt text](/assets/img/dev/jpa/JPA_Define.jpg)

### JPA, SpringDataJPA, Hibernate 개념 구분  

- JPA는 기술 명세로써, 인터페이스 표준을 말한다.  
- Spring Data JPA는 JPA를 좀 더 추상화하여 사용하기 쉽게 만든 모듈이다.  
  ~Repository라는 이름으로 인터페이스를 제공한다. (ex. CrudRepository, JPARepository, ...)  
- Hibernate는 JPA인터페이스를 구현한 실체이다.  
- 예를들어, SpringDataJPA를 사용하면 ~~~Repository를 통해 findByName(); 등을 사용하여 DB에 접근할 수 있고  
  Hibernate를 사용한다면 JPA 인터페이스를 구현하였으므로 EntityManager(Session)를 통해 persist(), find(); 메서드로 DB에 접근할 수 있다.

### JPA의 사용 목적

- 개발자(응응 프로그래머)는 객체지향 언어를 사용하고(ex. JAVA) 객체지향적인 설계를 통해 객체지향 프로그래밍을 한다.  
- 데이터베이스(관계형)는 집합적인 사고를 요한다. 특정 주제에 관계가 있는 데이터들이 모여 하나의 큰 데이터를 표현한다.  
- 데이터베이스(RDBMS)를 통해 데이터를 관리하려면 SQL을 사용한다.  
- 객체를 사용하면 1줄로 표현할 수 있는것을 SQL로 표현하면 목적에 따라 각각 다르게 표현된다. ~~즉 요구사항이 분기될 수록 SQL문이 늘어나게되며, 이것은 유지보수 비용이 증가함을 나타낸다.~~  
- JPA를 사용하면 개발자는 직접 SQL을 다루거나 JDBC API를 다루지 않아도 된다. JPA 인터페이스만 사용하면 되고, 실제 DB로 전달되는 내용은 JPA 구현체가 담당한다. (ex. Hibernate)  
- 객체지향 세계에서는 추상화, 상속, 참조, 다형성과 같은 개념으로 구성되어있지만 데이터베이스는 데이터를 중심으로 구성되므로 이런 개념이 없다.  
- <strong_red>JPA는 개발자가 개발은 객체지향으로, 데이터는 관계형 데이터베이스로 구성하는 두 이념 사이에서 소모하는 시간과 코드를 줄일 수 있다.</strong_red>  
- JPA는 내부에 캐쉬를 사용하므로, 매번 DB에 접근하여 SQL을 날리는게 아니라 캐쉬에 값이 없는경우에만 DB에 요청한다.  
- 서비스별로 각기다른 데이터베이스를 사용하는경우, 서비스 단에서 각각 데이터베이스 기술에 종속되지 않는다.  
  이것은 JPA가 서비스와 데이터베이스 간 추상회된 인터페이스를 제공하기 때문에 가능하다. 어플리케이션은 JPA에게 데이터베이스와 객체 간 매핑 방식만 전달하면 된다.  

---

### 연관관계

객체는 참조를 사용해서 다른 객체와 연관관계를 가지고, 참조를 통해 연관된 객체를 조회할 수 있다.  
ex)

```java
Member member = members.findById("member_id_0001");
Team team = member.getTeam();
```

테이블은 외래키(Foreign key)를 사용해서 다른 테이블과 연관관계를 가지고, Join을 이용해 연관돤 테이블을 조회할 수 있다.  
ex)

```sql
SELECT M.MEMBER_NAME, T.TEAM_NAME FROM MEMBER M, TEAM T WHERE M.TEAM_ID = T.TEAM_ID;
```

객체가 참조를 통해 연관된 다른 객체를 조회할 때, 참조가 있는 방향으로만 조회가 가능하다. (단방향)  
Member를 통해 해당 Member의 Team이 무엇인지는 조회가 가능하지만, Team을 통해 Member를 조회하는 것은 불가능하다.  

- 다대다 관계를 맺으면 Team에서도 Member를 찾을 수 있겠지만, 다대다 관계로 인해 복잡해지는 문제도 발생할 수 있다.  

JPA는 객체의 참조를 외래키로 변환해서 DB에 전달할 수 있으므로, 객체와 DB간 패러다임 불일치 문제를 해결 할 수 있다.  

__객체의 참조를 통해 연관된 객체를 탐색하는 과정을 객체 그래프 탐색이라 한다.__

SQL은 정적이다. SELECT 구문 실행 시점에서 조회하는 대상이 결정되므로, 객체 그래프 탐색 범위가 고정된다.  

<strong_blue>JPA는 연관된 객체를 즉시 조회할 수도 있고, 사용하는 시점(참조하는 시점)에 조회하여 SELECT 구문을 실행할 수 있다.</strong_blue>  
전자를 즉시 로딩, 후자를 지연 로딩이라 한다.  

---

### 동일성과 동등성

#### 동일성  

- \==를 통한 비교  
- 객체 인스턴스의 주소 값을 비교하므로, 두 객체가 같은 객체일 때만 같다고 판단한다.  

#### 동등성  

- equals를 통한 비교  
- 객체 내부의 고유값을 비교하므로, 서로 다른 두 객체라도 고유값이 같으면 같다고 판단한다.  

<strong_black>동등성 비교는 성공했지만, 동일성 비교가 실패하는 예</strong_black>

```java
String memberId = "ID0001";
Member member1 = memberDao.getMember(memberId);
Member member2 = memberDao.getMember(memberId);
// member1 != member2, member1.equals(member2) == true
// DB로부터 각각 조회한 결과는 같지만 두 객체는 서로 다른 인스턴스이므로 둘의 동일성은 보장되지 못한다.
```

<strong_black>동등성 및 동일성 비교가 모두 성공하는 예</strong_black>

```java
Member member1 = members.get(0);
Member member2 = members.get(0);
// member1 == member2, member1.equals(member2) == true
// 컬렉션 내의 같은 원소(인스턴스)를 참조하므로, 두 객체는 완전히 동일하다.
```

#### JPA의 동일성

JPA는 같은 트랜잭션일 때, 조회되는 객체의 동일성을 보장한다.

```java
String memberId = "ID0001";
Member member1 = memberRepository.find(Member.class, memberId);
Member member2 = memberRepository.find(Member.class, memberId);
// member1 == member2, member1.equals(member2) == true
```

---

> 자바 ORM 표준 JPA 프로그래밍 (김영한님 저) 도서 참조
