---
layout: post
title: JPA (Persist vs Merge)
subtitle: EntityManager의 persist와 merge method의 반환값 차이
categories: dev
tags: jpa
comments: true
---


### EntityManager의 persist와 merge method의 반환값 차이

일반적으로 Spring-Data-Jpa 프로젝트를 통해 Repository를 구현할 때 아래와 같이 JpaRepository를 상속받아 구현한다.  

```java
// MemberRepository.java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

JpaRepository는 엔티티를 영속화할 수 있도록 save(); 메서드를 제공한다.
* JpaRepository는 CrudRepository를 상속받고 있다. 

```java
 // SimpleJpaRepository.java (CrudRepository implementation)
 ...
 /*
  * (non-Javadoc)
  * @see org.springframework.data.repository.CrudRepository#save(java.lang.Object)
  */
 @Transactional
 public <S extends T> S save(S entity) {

  if (entityInformation.isNew(entity)) {
   em.persist(entity);
   return entity;
  } else {
   return em.merge(entity);
  }
 }
...
```

위와같이 save메서드는 파라미터로 들어온 엔티티가 새로운 엔티티인지 검사하고 새로운 엔티티면 persist를, 기존의 엔티티면 merge를 수행한다.  
새로운 엔티티인지 확인하는 isNew 메서드는 아래와 같이 구현되어 있다.  

```java
// EntityInformation.java
/*
 * (non-Javadoc)
 * @see org.springframework.data.repository.core.EntityInformation#isNew(java.lang.Object)
 */
public boolean isNew(T entity) {

	ID id = getId(entity);
	Class<ID> idType = getIdType();

	if (!idType.isPrimitive()) {
		return id == null;
	}

	if (id instanceof Number) {
		return ((Number) id).longValue() == 0L;
	}

	throw new IllegalArgumentException(String.format("Unsupported primitive id type %s!", idType));
}
```

더이상 깊게는 확인하지 않았지만 엔티티 식별자를 검사하여 값이 없으면 새로운 엔티티로 체크하고 값이 있으면 기존의 객체로 판단하는 것 같다.  

* <strong><u>값이 없다 == 비영속 엔티티</u></strong>  
* <strong><u>값이 있다 == 준영속 엔티티</u></strong>  

문제는 saved메서드가 엔티티의 상태에 따라 persist와 merge 두 메서드의 결과값을 각각 반환하는데 이 때 호출하는쪽에서 saved 메서드를 어떻게 사용하느냐에 따라 애매해지는 경우가 발생할 수 있다.  

테스트를 위해 다음과 같이 아주 간단한 엔티티를 구현했다.  

```java
@Getter
@Setter
@Entity
@Table(name = "MEMBERS")
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "MEMBER_ID")
    private Long id;
    private String name;
    private int Age;
}

멤버 엔티티는 Id를 자동으로 생성한다. (DATABASE로 기본키 생성 위임)  
그 외의 멤버는 테스트를 위해 아무 이유없이 생성했다.   
~~(테스트에서는 id만 필요하긴한데 id만 있으면 너무 허전하니까..)~~  

그리고 아래와 같이 테스트 코드를 작성했다.  

```java
/**
 * Created by Choen-hee Park
 * User : chpark
 * Date : 20/10/2020
 * Time : 6:48 AM
 */
@RunWith(SpringRunner.class)
@DataJpaTest
public class MemberTest {
    @Autowired
    private MemberRepository memberRepository;

    @PersistenceContext
    private EntityManager entityManager;

    @Test
    public void jpa_save_테스트(){
        Member member = new Member();
        member.setName("Chpark");
        member.setAge(34);
        // 비영속 -> 영속
        Member savedMember = memberRepository.save(member);

        // 영속화 후 반환값과 파라미터값 비교
        Assert.assertEquals(member, savedMember);
        Assert.assertTrue(entityManager.contains(member));

        Member newMember = new Member();
        newMember.setId(member.getId());
        newMember.setAge(20);
        newMember.setName("tester");
        // 준영속 -> 영속
        Member modifiedMember = memberRepository.save(newMember);

        // ID 비교
        Assert.assertEquals(savedMember.getId(), modifiedMember.getId());

        // 영속화 후 반환값과 파라미터값 비교
        Assert.assertNotEquals(newMember, modifiedMember);
        Assert.assertFalse(entityManager.contains(newMember));
    }
```

위 테스트과정은 아래와 같이 진행된다.  

1. member 엔티티를 생성해 name,age를 설정한 뒤 영속화한다.  

   <strong_blue>member는 이전에 영속성 컨텍스트에서 관리된적 없는 비영속 엔티티이므로 영속성 컨텍스트는 member를 1차캐시에 저장하여 영속화한다.</strong_blue>  
2. member 엔티티와 save()메서드에서 반환된 savedMember 엔티티의 동일성 비교를 진행한다.  
3. 새로운 newMember 엔티티를 생성하고 member에 저장된 식별자를 이용한다.  
   newMember는 member와 식별자 값이 같지만 서로 다른 인스턴스이므로  
   (참조하는 메모리 주소가 다르다.)  
   영속성 컨텍스트는 newMember를 준영속으로 인지한다.  
4. newMember 엔티티와 saved()메서드에서 반환된 modifiedMember 엔티티의 동일성 비교를 진행한다.  

<u>EntityManager의 persist()가 반환될 때, 반환된 엔티티와 넘어온 파라미터의 엔티티는 같은 인스턴스이다.</u>  

따라서 위의 2번 동일성비교는 TRUE가 된다.  

하지만 EntityManager의 merge()가 반환될 때는 반환된 엔티티와 넘어온 파라미터의 엔티티는 다른 인스턴스이다.  

<strong_red>* merge는 반환할 때 새로운 영속상태의 엔티티를 생성해서 반환한다.</strong_red>
따라서 위의 4번 동일성비교는 FALSE가 된다.  

이전에 내가 작성했던 테스트 코드들을 확인했는데 경우에 따라 save() 메서드의 반환값을 사용한적도 있었고 넘겨준 파라미터를 사용한 적도 있음을 확인했다.  
물론 각 테스트 코드는 일회성이고 병합을 굳이 테스트한적은 없어서 문제가 되지 않았던 것 같다.  

<strong_red>persist, merge 둘다 반환되는 값 은 영속상태의 엔티티이므로 가급적 Jpa의 save메서드를 호출 후에는 파라미터로 넘겨준 엔티티를 사용하지말고 반환된 영속 상태의 엔티티를 사용해야겠다.</strong_red>  
