---
layout: post
title: JPA Proxy
subtitle: JPA의 Proxy와 Fetch 전략
categories: dev
tags: jpa
comments: true
---

## JPA Proxy와 즉시로딩, 지연로딩

JPA를 공부하며 'JPA가 어렵구나' 라는 느낌이 몇번 들게되는 시점이 있었는데 그 중 하나가 Proxy였다.  
(영속성 컨텍스트, 복잡한 연관관계, Proxy, OSIV 등 몇 번 책을 다시 찾게되는 부분들이 있는 것 같다)  

JPA에서 제공하는 강력한 기능 중 하나가 Proxy를 통한 지연 로딩이 아닌가 싶다.  
그전에 Fetch 전략에 조금 얘기하면, JPA는 즉시로딩(Eager)과 지연로딩(Lazy)라는 두 가지 전략을 지원한다.  

JPA는 객체를 통해 객체에 연관된 다른 객체들에 대해 탐색하고, 또 다시 연관된 다른 객체들에 대해 탐색하는 객체 그래프 탐색 과정을 거치게된다.  
객체(Entity)는 데이터베이스에 저장된 데이터를 조회해야하는데 SQL을 통해 데이터베이스에 질의하는 건 질의 시점에 객체들간의 관계가 확정되버리는 문제가 있다.  

즉 탐색이 필요할 때마다 데이터베이스에 SQL을 전송해야한다는 말인데, 데이터베이스에 자주 접근한다는건 그만큼 성능에 이슈가 발생할 수 있는 문제가 된다.  

JPA는 이러한 문제를 해결하고자 프록시라는 기술을 사용하는데 이 프록시를 주로 지연로딩을 통해 사용된다.  

---

### 즉시로딩  

매번 필요할 때 마다 SQL을 데이터베이스로 전송하는 방식이다.  
연관관계를 맺은 객체들은 JOIN으로 묶이게되고 이렇게 묶인 하나의 큰 SQL을 데이터베이스로 전송해 필요한 모든 정보를 가져온다.  
데이터베이스에 접근하는 횟수가 1회이므로 데이터베이스로의 접근성은 줄였지만 조회할 데이터의 크기가 방대하다면 이 또한 성능 이슈가 발생할 수 있다.  
또한 굳이 연관관계가 맺어진 객체에 대해서는 조회할 필요가 없는 경우에도 항상 한번에 조회하므로, 불필요한 정보가 대다수일 경우가 많다.  
예를들어 인사시스템에서 한 사원의 이름과 사번을 가져오려 했지만 사원 객체가 부서, 역할, 소속 등 다른 객체들과 연관되어 있는경우 부서, 역할, 소속 테이블과 JOIN으로 묶여져 데이터를 조회하게 된다.  

이런 경우가 많다면 즉시로딩보다는 지연로딩을 사용하는게 성능에 유리할 수 있다.  

---  

### 지연로딩  

연관관계로 묶인 객체들에 대해 JOIN연산을 수행하지 않고, 조회하려는 객체에 대해서만 SQL을 작성하여 데이터베이스를 조회한다.  
위 예제의 경우 사원 테이블에만 접근하여 사원 객체를 생성할 뿐 부서, 역할, 소속 등 다른 테이블에는 접근하지 않는다.  
이게 어떻게 가능할까?  
분명 코드에는 연관관계를 맺고있는데 도대체 어떻게 SQL문을 작성한거지?  
JOIN없이도 그럼 연관관계된 객체들에 대해 알아서 가져온다고?  
JOIN없이 현재 탐색하려는 객체에 대해서만 조회할 수 있는 이유는 연관관계된 객체들을 프록시 객체로 사용하기 때문에 가능하다.  
연관관계로 맺어진 객체들을 null로 두면 당연히 접근할 때 오류가 발생하겠지만, 이 객체들은 프록시객체로 생성되어 null을 피할 수 있다.  

---  

### Proxy  

지연로딩을 사용할 때, JPA는 연관관계로 맺어진 객체들에 대해서 개발자가 작성한 Entity 클래스를 그대로 사용하지 않는다.  
Entity 클래스를 상속받은 EntityProxy 클래스를 만들고 EntityProxy 객체를 생성하게 된다.  
(EntityProxy는 Entity를 상속받았으므로 Entity의 모든 기능을 사용할 수 있다.)  
이렇게 생성된 프록시 객체는 내부에 target이라는 이름으로 실제 엔티티의 참조를 가지고 있게 된다.  

지연 로딩을 한 가지 예를 들어 설명해보겠다.  
사원 객체를 생성하고 사원 정보를 읽을 땐 연관관계에 있는 부서, 역할, 소속 등 객체들은 SQL을 사용해서 데이터를 조회한게 아니므로 아무값도 가지고 있지 않는다.  
다만 각 프록시 객체들이 인스턴스화되어 NPE를 피할 수 있는 상태일 뿐이다. (아무런 데이터 없는 껍데기 객체상태)  

이런 상태에서 프록시 객체로 조회 요청이 호출된다면 (엔티티의 데이터에 접근하는 경우) 프록시 객체는 영속성 컨텍스트에 실제 엔티티를 생성해달라고 요청하게 된다.
(이 과정을 프록시 초기화 과정이라고 한다.)  
만약 실제 엔티티가 이미 영속성 컨텍스트에 존재한다면 프록시를 사용할 이유가 없어지므로 엔티티를 반환할 것이다.  
하지만 엔티티가 존재하지 않는다면 데이터베이스를 조회하여 엔티티를 생성한다.  
그 뒤 프록시 객체는 실제 엔티티의 메서드를 호출하여 결과를 반환하게 된다.  

즉 지연로딩을 사용하면 데이터가 실제로 필요한 시점에 데이터베이스로 질의하여 엔티티 객체를 구성하게 된다.  
프록시 객체는 내부에서 자신이 상속받은 실제 엔티티 객체에 대한 포인터를 갖고 있으므로, 실제 엔티티의 메서드를 호출하고 값을 반환함으로써 객체 그래프 탐색이 가능하게 된다.  

주의할 점은, 프록시의 초기화는 영속성 컨텍스트로 요청함으로써 시작된다.  
영속성 컨텍스트에 접근하려면 반드시 영속 상태여야 하므로, 프록시 객체가 비영속/준영속 상태이면 예외가 발생한다.  

---

위에서 프록시 객체는 실제 엔티티를 상속받았다고 했다.  
JPA는 상속받기위해 기본 생성자를 사용한다.  
따라서 엔티티 클래스에는 기본 생성자를 명시적으로 정의해야한다. (물론 아무 생성자도 없으면 기본 생성자가 자동으로 정의된다.)  
또한 상속받을 수 있도록 접근제한자가 public이나 protected로 선언되어야 한다.  
만약, 기본 생성자가 없다면 아래와 같은 예외가 발생한다.  
![Alt](/assets/img/dev/jpa/jpa_error_no_default_constructor.png)  

테스트를 위해 Member와 Team이라는 간단한 클래스를 구현했다.  

#### Member Class

```java
@NoArgsConstructor
@Getter
@Entity
public class Member {
 @Id
 @GeneratedValue(strategy = GenerationType.AUTO)
 private Long id;

 private String name;

 public Member(String name, Team team) {
  this.name = name;
  this.team = team;
 }

 // fetch 전략은 일반 객체(1:1)의경우 EAGER을, 컬렉션인 경우(1:N) Lazy를 기본값으로 사용한다.
 @ManyToOne(cascade = CascadeType.ALL)
 private Team team;
}
```

#### Team Class  

```java
@NoArgsConstructor
@Getter
@Entity
public class Team {
 @Id
 @GeneratedValue(strategy = GenerationType.AUTO)
 private Long id;

 private String name;

 public Team(String name) {
  this.name = name;
 }
}
```

---  

실제로는 Member와 Team은 N:1로 컬렉션을 사용해서 구성해야겠지만 간단하게 테스트를 위해 1:1로 구성했다.  
테스트 코드는 아래와 같이 작성했다.  
보통 Spring-Data-JPA를 사용하지만 JPA에 대해 직접 다루는 연습?을 해야할 것 같아 EntityManager를 직접 생성하여 사용했다.  

```java
@DataJpaTest
class MemberTest {
 @Autowired
 private EntityManagerFactory entityManagerFactory;

 @BeforeEach
 void setUp() {
  EntityManager entityManager = entityManagerFactory.createEntityManager();

  Team team = new Team("토트넘");
  Member member = new Member("손흥민", team);

  entityManager.persist(member);
  EntityTransaction transaction = entityManager.getTransaction();
  transaction.begin();
  entityManager.flush();
  transaction.commit();
  entityManager.close();
 }

 @Test
 @DisplayName("회원과 팀 정보 출력")
 void printUserAndTeam() {
  EntityManager entityManager = entityManagerFactory.createEntityManager();
  Member member = entityManager.find(Member.class, 1L);
  Team team = member.getTeam();
  System.out.println(member.getClass().getName());
  System.out.println(team.getClass().getName());
  Assertions.assertThat(member).isNotNull();
 }
```

위 코드를 실행하면 정상적으로 테스트가 통과된다.  
하지만 정상적인것을 보려고 시도한게 아니라 '~~이것을 안하면 오류가 발생한다.' 라는 것을 테스트하는게 목적이었기 때문에 문제가 될 만한 점을 하나씩 수정해보면서 테스트를 진행했다.  

---  

<strong_red>flush는 트랜잭션 내에서 수행되어야 한다.</strong_red>  

위 코드에서는 @BeforeEach를 통해 테스트함수가 실행되기전에 샘플 Member, Team 객체를 생성하여 영속성 컨텍스트에 저장한다.  
그런데 persist할 때까지는 문제가 발생하지 않지만 flush를 transaction 밖에서 실행하면 오류가 발생한다.  
예를들어 위 setUp 메서드를 아래와 같이 구성하면 안된다.  

```java
@BeforeEach
 void setUp() {
  EntityManager entityManager = entityManagerFactory.createEntityManager();

  Team team = new Team("토트넘");
  Member member = new Member("손흥민", team);

  entityManager.persist(member);
  entityManager.flush();
  entityManager.close();
 }
```

어찌보면 당연한 결과이긴 하다.  
transaction은 데이터베이스와의 동기화를 위해 처리할 수 있는 일련의 작업이자 단위이고  
flush는 영속성 컨텍스트의 SQL 쓰기지연 저장소에 있는 SQL들을 데이터베이스로 전송하는 과정인데, 트랜잭션 밖에서 트랜잭션 작업을 수행하려하다니?  
Spring-Data-JPA와 같은 추상화된 모듈을 사용하면 이러한 과정들이 내부에 숨겨져있어 실제 동작을 모르고 넘어갈 수 있는데, 이렇게 직접 실행해서 눈으로 보니 더 확실히 알 수 있는 것 같다.  

---  

<strong_red>즉시 로딩을 사용하면 Left Outer Join을 기본적으로 사용하며, 연관관계를 맺은 객체는 실제 엔티티가 된다.</strong_red>  

엔티티(Member, Team)를 즉시 로딩으로 생성하면 아래와 같은 결과가 출력된다.  
![Alt](/assets/img/dev/jpa/jpa_find_member_and_team_with_eager.png)
즉시 로딩이므로 Member와 Team이 JOIN되었는데 Left Outer Join을 사용했다.  
이 값은 기본값으로 JPA가 연관관계된 객체를 참조하는 식별자가 nullable될 수 있기때문에 사용된다고 한다.  
만약 Inner Join으로 변경하려면 @JoinColumn 옵션에 nullable을 설정해주던가 @ManyToOne에 optional=false로 설정한다.  

```java
// 기본 설정 (Left Outer Join)
@ManyToOne(cascade = CascadeType.ALL)
private Team team;

// Inner Join 사용 1
@ManyToOne(cascade = CascadeType.ALL, optional=false)
private Team team;

// Inner Join 사용 2
@ManyToOne(cascade = CascadeType.ALL)
@JoinColumn(name = "team_id", nullable = false)
private Team team;
```

---  

<strong_red>지연로딩을 사용하면 JOIN을 사용하지 않으며, 프록시 객체를 참조한다.</strong_red>  

지연로딩을 사용한 결과는 아래와 같다.  
![Alt](/assets/img/dev/jpa/jpa_find_member_and_team_with_lazy.png)
조회한 SQL에서 Team에 관한 어떠한 내용도 없다.  
그리고 Team 객체의 클래스 이름을 출력해보면 Team이 아니라 Team$HibernateProxy.... 로 조회된다.  
즉 직접 작성한 Team 클래스의 객체가 아니라 Team을 상속받은 Proxy객체라는 의미이다.  

---  

## 영속성 전이  

위의 예제코드에서 설명이 빠진 부분이 하나 있었는데 CASCADING 이란 기능이다.  
CASCADING은 영속성 전이라는 뜻으로, 대표 엔티티만 영속화 하더라도 연관관계된 다른 객체들도 함께 전이되는 기능이다.  
예를들어 사원 객체가 부서, 역할, 소속 객체와 연관관계가 맺어져있다고 가정할 때 사원 객체만 저장해도 나머지 객체들도 함께 저장할 수 있다.  
보통 부모 엔티티에서 CASCADING을 설정하면, 자식 엔티티로 전이된다.  
여기서 부모 엔티티란 1:1관계에서는 상관없지만 1:N 연관관계의 경우 1이 부모 엔티티에 해당한다.  
즉, 사원 N명이 부서 1개에 포함될 수 있다면, 부서 엔티티가 부모 엔티티가 된다.  
CASCADING 은 아래와 같이 필드의 연관관계 옵션 하나만 넣어주면 된다.  

```java
// ... Team Entity
@OneToMany(fetch = FetchType.LAZY, cascade = CascadeType.ALL, mappedBy = "team")
private List<Member> members;

// ... Member Entity
@ManyToOne(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
```

위 코드는 Team과 Member가 1:N으로 연관관계가 매핑되었다고 가정하고, Team 엔티티와 Member 엔티티의 연관관계를 설정한 값이다.  
Team 엔티티에 Cascade.ALL을 설정하여 영속성 전이가 설정됨을 확인할 수 있다.  
참고로 Cascading은 연관된 엔티티도 함께 영속화 시킬 뿐, 연관관계 자체에 영향을 주진 않는다.  

---  

### 영속성 전이에 사용될 수 있는 옵션  

1. CascadeType.ALL
2. CascadeType.PERSIST
3. CascadeType.MERGE
4. CascadeType.REMOVE
5. CascadeType.REFRESH
6. CascadeType.DETACH  

PERSIST와 REMOVE의 발생 시점은 flush시점이다.  (나머진 즉시 발생)  

---  

## 고아 객체  

부모 엔티티와 연결이 끊어진 자식 객체를 말한다.  
위 예제의 경우, Team 객체는 members 필드를 통해 Member 엔티티들을 자식으로 관리하는데,  
만약 members중 하나 혹은 전체를 제거하면 연관관계도 끊을 수 있다.  

고아 객체 기능의 설정은 아래와 같이 필드에 옵션을 넣어주면 된다.  

```java
@OneToMany(orphanRemoval = true)
private List<Member> members;
```

예를들어 members에 5개의 Member 객체가 포함되어 있는경우 아래와 같은 코드가 실행된다고 가정하겠다.  

```java
EntityManager entityManager = ...;
Team team = entityManager.find(Team.class, teamId);
team.getMembers().clear();
```

위 코드에서 Team 객체는 연관관계를 맺고 있는 members 필드의 모든 원소를 비웠다.  
만약 고아 객체 제거 옵션을 설정했다면 원소만 비워질 뿐 아니라 엔티티 객체가 영속성 컨텍스트에서 제거된다.  
즉, 고아 객체 제거란 참조가 제거된 엔티티는 다른 엔티티에서도 참조가 되지 않는다고 가정하고 영속성 컨텍스트에서 제거하는 기능이다.  

여기서 중요한게 "다른 엔티티에서도 참조가 되지 않는다고 가정" 이 될 수 있겠다.  
왜냐하면 다른곳에서도 참조하고 있는 엔티티라면, 영속성 컨텍스트에서 삭제되면 안되기 때문이다.  
따라서 고아 객체 기능은 1:1, 1:N에서만 사용할 수 있다.  

### 만약 CascadeType.ALL과 orphanRemoval=true 기능을 함께 사용한다면?  

자식 엔티티를 저장하려는 경우, 자식 엔티티는 부모 엔티티의 컬렉션에 추가만하고, 부모 엔티티만 영속성 컨텍스트에 저장하면 된다.  
(Cascading)  
자섹 엔티티를 제거하려는 경우, 부모 엔티티에서 자식 엔티티의 참조만 끊으면 자동으로 자식 엔티티가 삭제된다.  
(orphanRemoval)  

즉 위 2가지 옵션을 함께 사용한다면, 부모 엔티티가 자식 엔티티의 생명주기(생성부터 종료까지)에 관여할 수 있으므로, 객체를 관리하게 용이해진다.  
DDD(Domain Driven Design) 책에서도 자식 애그리거트가 수정될 때 루트 애그리거트에도 반영될 수 있어야 한다라는 내용이 있었는데,  
위 옵션들을 활성화해서 구현이 가능할 것이라는 생각이 든다.  

> 자바 ORM 표준 JPA 프로그래밍 (김영한님 저) 도서 참조  
