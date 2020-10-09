---
layout: post
title:  EntityManager
subtitle: What is EntityManager?
categories: dev
tags: jpa
comments: true
---

#### EntityManager 설정

EntityManager의 생성과정

![Alt text](/assets/img/dev/jpa/EntityManager.png)

1.  JPA 설정파일(ex. Hibernate의 persistence.xml)을 읽은 후 EntityManagerFactory를 생성한다.
2.  EntityManagerFactory로부터 EntityManager를 생성한다.
    
    ```
    // persistence-unit : 데이터베이스당 등록되는 영속성 유닛의 고유 이름값 (persistence.xml에 기술됨)
    EntityManagerFactory factory = Persistence.createEntityManagerFactory("persistence-unit");
    EntityManager entityManager = factory.createEntityManager();
    ```
    

#### EntityManagerFactory

EntityManagerFactory를 생성하는 작업은 비용이 매우 크다.  
왜냐하면 EntityManagerFactory를 생성할 때, 설정 정보를 토대로 JPA구동에 필요한 객체들을 생성하고 JPA 구현체에 따라서 DB 커넥션 풀도 생성하기 때문이다.  
<strong_purple>따라서 EntityManagerFactory는 어플리케이션이 구동하고 1번만 생성하도록 하며, 모든 스레드에서 공유할 수 있도록한다.</strong_purple>

~~(Singleton?)~~


어플리케이션을 종료할 때 EntityManagerFactory를 종료한다.

```
factory.close();
```

#### EntityManager

EntityManager는 JPA의 대부분 기능을 제공한다.  
예를들어 CRUD(Create,Read,Update,Delete) 기능을 사용할 수 있다.  
EntityManager는 내부에서 DataSource(DB-Connection)을 유지하면서 DB에 접근한다.  
<strong_purple>EntityManager는 DB-Connection과 밀접하게 관련되어 있으므로 스레드간 공유하지않고, 재사용도 하지 않는다.</strong_purple>

~~(사용하면 GC 대상으로 만든다.)~~


사용이 끝났을 때(ex.Transaction) EntityManager를 종료한다.

```
entityManager.close();
```

**JPA를 사용할 땐 항상 트랜잭션 내에서 작업을 수행한다.**

```
EntityManagerFactory factory = Persistence.createEntityManagerFactory("persistence-unit");
EntityManager entityManager = factory.createEntityManager();
EntityTransaction transaction = entityManager.getTransaction();
try {
    transaction.begin();    // 트랜잭션 시작
    // ~~~~~ BusinessLogic ~~~~~
    transaction.commit();    // 트랜잭션 커밋
} catch (Exception e) {
    transaction.rollback();    // 트랜잭션 롤백
}
```

#### JPQL

JPA는 엔티티 객체를 대상으로 검색하고, DB는 테이블을 대상으로 검색한다.  
검색 쿼리의 경우 하나의 엔티티를 통해 검색하는게 아니라 DB로부터 필요한 데이터를 모두 읽어들인다음 엔티티 객체로 변환하는 등 대규모 작업이 펼쳐지므로 JPA만으로는 해결하기 어렵다.  
SQL을 추상화한 JPQL을 통해 이를 해결할 수 있다. (SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 등 SQL 기능 사용 가능)

-   JPQL : Entity 객체를 대상으로 쿼리한다. 엔티티 클래스와 필드를 대상으로 쿼리한다.
-   SQL : Database 테이블을 대상으로 쿼리한다.  
    즉, JPQL을 사용한다고 하더라도 JPQL에서 from절에 있는 대상은 엔티티 객체이지 테이블이 아니다.  
    JPQL은 절대로 데이터베이스의 테이블에 대해 알지 못한다.

**JPQL이 Database로 접근하는 과정**

![Alt text](/assets/img/dev/jpa/jpql_jpa_sql.png)

<hr>
> 자바 ORM 표준 JPA 프로그래밍 (김영한님 저) 도서 참조
