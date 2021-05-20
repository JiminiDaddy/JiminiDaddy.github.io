---
layout: post
title: SpringBoot Service/Repository 단위 테스트
subtitle: SpringBoot에서 Service, Repository만 따로 단위 테스트 하기
categories: dev
tags: spring
comments: true
---

# SpringBoot Service와 Repository의 단위 테스트 방법    

지난 포스팅에서 API(Controller)만 따로 단위 테스트하는 방법을 정리했다.  
[SpringBoot API 단위테스트](https://jiminidaddy.github.io/dev/2021/05/18/dev-spring-%EB%8B%A8%EC%9C%84%ED%85%8C%EC%8A%A4%ED%8A%B8-API/)  

이번엔 Repository와 Service에 대한 단위 테스트를 진행해보려고 한다.  

API의 단위 테스트를 위해 Controller와 Service의 결합을 제거하여 테스트를 진행했다면  
Service와 Repository는 어떻게 다른 영역과의 결합을 제거할 수 있을까?  

<hr>

## Repository Test  
Repository는 엔티티를 영속화하기위해 사용된다.  
엔티티의 영속화 요구는 서비스에서 발생한다.  
표현 계층은 Client와 맞닿은 영역이므로 Client의 요청/응답을 처리하며, 필요한 기능을 서비스 계층으로 위임하게 된다.  
서비스 계층은 요구사항을 처리하는 영역으로, 도메인을 통해 비즈니스 로직을 수행한다.  
(주문을 하거나, 주문 취소를 하거나, 결제를 하거나, 회원 가입을 한다던가, 상품 등록을 한다던가 등등...)  
비즈니스 로직을 수행하고 난 도메인을 영속화해야하는데 이 기능을 저장소 영역으로 위임한다.  
__따라서 Repository의 기능만 테스트를 하려면 Service와의 결합을 끊어야 한다.__  

SpringBoot 테스트는 __@DataJpaTest__ Annottation을 제공하는데, 이것을 통해 Repository의 단위 테스트가 가능하다.  

<strong_blue>@DataJpaTest을 사용할경우 아래와 같은 기능이 수행된다.</strong_blue>  
- JPA 관련된 설정만 로드한다.  (WebMVC와 관련된 Bean이나 기능은 로드되지 않는다)  
- JPA를 사용해서 생성/조회/수정/삭제 기능의 테스트가 가능하다.  
- @Transactional을 기본적으로 내장하고 있으므로, 매 테스트 코드가 종료되면 자동으로 DB가 롤백된다.  
- 기본적으로 내장 DB를 사용하는데, 설정을 통해 실제 DB로 테스트도 가능하다.  (권장하지 않는다)  
- @Entity가 선언된 클래스를 스캔하여 저장소를 구성한다.  

<hr>

회원 가입과 회원 조회에 대한 테스트를 진행했다.  
테스트할 MemberRepository는 Bean으로 등록되므로 @Autowiried를 통해 의존성을 주입받았다.  
Repository 외에 다른 Bean은 필요없으므로 별다른 설정할 게 없다.  

```java
@DataJpaTest
public class MemberTest {
    @Autowired
    private MemberRepository memberRepository;

    @Test
    @DisplayName("멤버가 DB에 저장이 잘 되는지 확인")
    void saveMember() {
        // given
        Member member = new MemberJoinRequestDto("chpark", 34).toEntity();
        // when
        Member savedMember = memberRepository.save(member);
        // then
        Assertions.assertThat(member).isSameAs(savedMember);
        Assertions.assertThat(member.getName()).isEqualTo(savedMember.getName());
        Assertions.assertThat(savedMember.getId()).isNotNull();
        Assertions.assertThat(memberRepository.count()).isEqualTo(1);
    }

    @Test
    @DisplayName("저장된 멤버가 제대로 조회되는지 확인")
    void findMember() {
        // given
        Member savedMember = memberRepository.save(new MemberJoinRequestDto("chpark", 34).toEntity());
        Member savedMember2 = memberRepository.save(new MemberJoinRequestDto("tester", 20).toEntity());
        // when
        Member findMember = memberRepository.findById(savedMember.getId())
                .orElseThrow(() -> new IllegalArgumentException("Wrong MemberId:<" + savedMember.getId() + ">"));
        Member findMember2 = memberRepository.findById(savedMember2.getId())
                .orElseThrow(() -> new IllegalArgumentException("Wrong MemberId:<" + savedMember2.getId() + ">"));
        // then
        Assertions.assertThat(memberRepository.count()).isEqualTo(2);
        Assertions.assertThat(findMember.getName()).isEqualTo("chpark");
        Assertions.assertThat(findMember.getAge()).isEqualTo(34);
        Assertions.assertThat(findMember2.getName()).isEqualTo("tester");
        Assertions.assertThat(findMember2.getAge()).isEqualTo(20);
    }
```  
<hr>

## Service Test  
API, Repository, Service Test 중 개인적으로 Service Test가 제일 어려웠었다.  
물론 내가 테스트코드에 그만큼 숙련되지 않은게 가장 큰 문제였고, JUnit에서 제공되는 기능들도 제대로 다 파악하지 못했기 때문이다.  
기본적인 것들은 테스트코드를 작성해보면서 사용했는데, JUnit5기준으로 제공하는 기능들을 한번 싹 정리해봐야겠다.  
~~(매번 필요할때마다 구글링하는것도 힘드니 그냥 내 블로그에 직접 작성해봐야겠다!)~~  

Service는 위로는 Controller, 아래로는 Domain에 의존하고 있다.  
따라서 결합을 두 군데나 끊어야 한다. (이것때문에 어려웠던 것 같다.)  

<strong_red>먼저 Controller와의 연결을 끊어야한다.</strong_red>  
Controller는 Web모듈이므로 Service Test를 진행하려면 Web에 대한 의존성을 받으면 안된다.
따라서 @WebMvcTest, @SpringBootTest와 같은 테스트를 사용하면 Service만을 테스트하기가 어려워진다.  

<strong_red>두 번째로 Repository와의 연결을 끊어야 한다.</strong_red>  
Domain을 통해 비즈니스 로직은 수행해야하지만, 실제로 DB에 저장할 건 아니기 때문에 이 부분을 제거할 방법이 필요하다.  
SpringBoot 테스트는 특정 객체를 가짜로 대체할 Mocking을 제공하고 있고, 아래와 같은 Annotation을 제공한다.  
@Mock, @MockBean, @Spy, @SpyBean  

@Mock으로 선언한 객체는 의존하고 있는 실제 객체 대신에 @Mock으로 선언한 객체로 바꿔치기된다.  
__따라서 Service 내에 의존하고 있는 Repository를 @Mock으로 선언하면 Repository Bean에 의존하지 않고 테스트가 가능해진다.__  
그리고 Service 클래스를 @InjectMocks로 선언함으로써, @Mock으로 선언된 가짜 객체들을 의존한 Service 객체가 생성된다.  

회원가입과 회원조회 기능에 대해 Service 테스트 코드를 아래와 같이 작성해보았다.  
 ```java
@ExtendWith(MockitoExtension.class)
public class MemberServiceTest {
    @Mock
    private MemberRepository memberRepository;

    @InjectMocks
    private MemberService memberService;

    @Test
    @DisplayName("join기능이 제대로 동작하는지 확인")
    void join() {
        // given
        MemberJoinRequestDto requestDto = new MemberJoinRequestDto("chpark", 34);
        when(memberRepository.save(any())).thenReturn(requestDto.toEntity());
        MemberJoinRequestDto requestDto2 = new MemberJoinRequestDto("tester", 20);
        when(memberRepository.save(any())).thenReturn(requestDto2.toEntity());

        // when
        memberService.join(requestDto);
        memberService.join(requestDto);

        // then
        // Id 생성 전략을 Identity를 사용하므로, 실제 DBd에 저장되야만 Id가 생성된다. 따라서 테스트에서 Id를 검증할 수 없다.
        // 만약 Id를 검증하려면 Repository를 Mock이 아니라 실제 Bean으로 사용해야 가능할 듯 싶다.
    }

    @Test
    @DisplayName("find기능이 제대로 동작하는지 확인")
    void find() {
        // given
        MemberJoinRequestDto requestDto = new MemberJoinRequestDto("chpark", 34);
        when(memberRepository.save(any())).thenReturn(requestDto.toEntity());
        memberService.join(requestDto);
        when(memberRepository.findById(1L)).thenReturn(Optional.of(requestDto.toEntity()));

        // when
        MemberFindResponseDto responseDto = memberService.find(1L);

        // then
        Assertions.assertThat(responseDto).isNotNull();
        Assertions.assertThat(responseDto.getName()).isEqualTo("chpark");
        Assertions.assertThat(responseDto.getAge()).isEqualTo(34);
    }
```  

Service Test는 비즈니스 로직 처리가 제대로 되는지만 검증하면 되므로 Spring과 연관될 이유가 없다.  
따라서 SpringContaianer가 로드되지 않도록 SpringExtension.class를 사용하지 않고,  
@ExtendWith(MockitoExtension.class) 을 추가하여 단위 테스트를 작성했다.  
두 Annotation 사용에 차이는 아래에 간단히 기술했다.  

### @ExtendWith(SpringExtension.class)  
SpringContainer를 로드하므로 Test 객체에 @Autowired를 통해 Bean 의존성을 주입시킬 수 있다.  
또한 Bean을 Mocking하기위한 @MockBean 기능을 사용할 수 있다.  
테스트를 위해 Spring이 필요하다면 위 코드가 필요하다.  

### @ExtendWith(MockitoExtension.class)  
SpringContainer를 로드하지않고 테스트를 위한 기능만 제공한다.  
@Mock, @Spy 기능을 사용할 수 있다.  
테스트에 Spring이 필요없이 순수한 단위 테스트만 필요하다면 위 코드를 추가하면 된다.  

Controller, Service, Repository 및 HelloWorld에 대한 테스트 코드 전체를 실행했는데 모두 통과했다.  
![Alt](/assets/img/dev/spring/dev-spring-unittest-success.png)  
~~초록색 불로 가득차면 뭔가 마음이 편하다..~~  