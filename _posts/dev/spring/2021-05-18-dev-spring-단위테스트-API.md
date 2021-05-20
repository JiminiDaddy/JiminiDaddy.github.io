---
layout: post
title: SpringBoot API 단위테스트
subtitle: SpringBoot에서 Controller만 따로 단위테스트 하기
categories: dev
tags: spring
comments: true
---

# SpringBoot 테스트 방법  

Spring이 아니어도 단위 테스트는 굉장히 중요한 기능아다.  

<strong_black>내가 구현한 코드가 요구사항을 제대로 반영했는지, 예외케이스가 제대로 떨어지는지, 또다른 버그는 없는지  
개발자가 본인 코드의 무결성을 검증하기 위한 필수적인 작업이라고 생각한다.</strong_black>   

SpringBoot에서도 테스트 모듈이 잘 구현되어있는데 SpringBoot 2.x부터는 기본적으로 Junit5가 탑재된다.  
~~(최근 2.4.x버전 부터는 Junit5의 vintage가 빠진다는 글을 본 것 같은데 시간될 때 좀더 확인해봐야겠다. 보통 난 jupiter를 사용하긴 했지만..)~~  

SpringBoot + JPA를 이용해서 1개의 API를 개발하면 여러 객체들의 의존으로 인해 테스트 코드를 작성하는게 아주 쉽지는 않았다.  
예를들어 Controller - Service - Repository의 구조로 엔티티를 저장하는 기능을 구현한다고 했을 때
1. Repository에 엔티티가 제대로 저장이 되는지를 확인해야 하고  
2. Service가 Repository를 호출한 뒤 예외없이 저장이 되늰지, Controller로 전달할 결과값을 제대로 생성하는지 확인해야 하고  
3. Controller는 Service로 기능을 위임하고 원하는 결과를 얻는지 확인해야 한다.  

<hr>

__처음에 이런 테스트가 어려웠던게 결국 저장 기능의 최종목적지는 Database다.__  
즉 Database에 내가 요청한 데이터가 제대로 저장하는지 확인하려면 Database가 설치되어야 있어야하며, 연결도 되어야 한다.  
단순히 구현한 기능에 대해 테스트만 진행하려고 했는데 배보다 배꼽이 더 큰 격이 된 셈이다.  

물론 Production 코드에서는 모든 설정이 다 이루어져있을 것이므로 실제 서버를 올려서 Postman같은 Tool을 통해 간단한 테스트를 해볼수도 있겠다.  
하지만 기능이 많아지면?  한 가지 기능에 발생할 수 있는 경우의 수가 많다면?  
이 둘만 곱해도 몇십, 몇백개인데 일일이 다 수작업으로 테스트할 것인가?  

난 인내심이 좋은편이라(?) 한 5개까지는 아주 가끔은 뭐 할 수도 있을 것 같다.  
하지만 50개를 테스트 해야 한다면? 난 못할 것 같다. 어떻게든 그것만은 피할 수 있는 다른 방법을 생각해낼 것 같다.  

SpringBoot에서는 테스트를 위해 크게 통합 테스트와 단위 테스트를 제공한다.  

<hr>

### 통합 테스트  
통합 테스트는 말 그대로 구현한 서버의 전체를 테스트하는 기능이다.  
내장 웹 서버가 올라오며 모든 Spring Bean들이 등록되고 DB도 구동되므로 웹 애플리케이션으로서의 기능을 완벽히 재현한다.  
따라서 Controller, Service, Repository 모든 영역에 대한 테스트가 가능하다.  
단점이면 아무래도 웹 서버가 올라오는만큼 구동시간이 발생하여 그만큼 테스트에 필요한 시간이 많아진다.  
또한 각각 모듈에 대한 테스트는 어렵다.  

<hr>

### 단위 테스트  
단위 테스트는 각각 모듈별로 테스트하는 가능이다.  
Controller, Service, Repository 각각의 테스트를 진행할 수 있다.  
각각 모듈만을 테스트해야하는데 모듈끼리 결합된 부분이 있으므로 이러한 의존을 제거해주는 역할이 필요한데  
Mockito 라는 모듈이 이런 의존을 제거해준다.  
예를들어 API 요청/응답에 대해서만 테스트를 한다면 Controller만 Bean으로 등록하고  
Service는 빈으로 등록하지 않음으로써 Controller와 Service의 의존을 끊는다.  
따라서 Controller만 테스트가 가능해진다.  
__SpringBoot에서 API만을 테스트하려면 @WebMvcTest 애너테이션을 사용하면 된다.__  
Repository에 대한 단위 테스트를 진행하고 싶다면 @DataJpaTest를 사용할 수 있다.  
Service에 대한 테스트가 조금 까다로웠는데 이번 포스팅은 API Test가 주제이므로 Service Test는 다음번에 정리할 예정이다.  

API 테스트를 진행하기 위해 아주아주 간단한 예제를 하나 작성했다.  
이름과 나이를 통해 회원가입을 하고(~~요즘같은 시대에 나이값을 통해 회원가입을한다고??~~) 발급된 Id를 가지고 조회하는 기능이다.  
~~예제니까 뭐 다른 기능 아예없다. 이름이 중복되도 저장되고.. 보안처리 이런거 하나도없다..~~

<hr>

가장 먼저! 빌드 파일을 작성했는데 gradle을 사용했고, 아래와 같이 작성했다.

```
dependencies {
    implementation group: 'org.springframework.boot', name: 'spring-boot-starter-web'
    implementation group: 'org.springframework.boot', name: 'spring-boot-starter-data-jpa'

    compileOnly group: 'org.projectlombok', name: 'lombok'
    annotationProcessor 'org.projectlombok:lombok:1.18.20'

    testImplementation group: 'org.springframework.boot', name: 'spring-boot-starter-test'

    implementation group: 'com.h2database', name: 'h2'
}
```

### Controller Class  
Id를 통한 회원 조회, 이름과 나이를 통한 회원가입 API를 제공한다.  
편의상 로그는 삭제했다.  (Github에는 로그를 추가하여 약간 지저분함)  
조회는 Id를 URI Path로 받도록 구현했고, 가입은 HttpBody에 Json으로 요청온 데이터를 받도록 구현했다.  
```java
@Slf4j
@RestController
public class MemberController {
    private final MemberService memberService;

    public MemberController(MemberService memberService) {
        this.memberService = memberService;
    }

    @GetMapping("/member/{id}")
    public MemberFindResponseDto findMember(@PathVariable(value = "id") Long memberId) {
        return memberService.find(memberId);
    }

    @PostMapping("/member")
    public Long joinMember(@RequestBody MemberJoinRequestDto requesteDto) {
        return memberService.join(requesteDto);
    }
}
```  

### Service Class  
Controller부터 받은 요청을 처리하며, MemberRepository를 사용한다.  
딱히 기능은 없다.  
~~음 근데 find에 Transactional Readonly를 붙여줘야하는데 깜박하고 커밋해버렸네..~~
```java
@Transactional
@Service
public class MemberService {
    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }

    public MemberFindResponseDto find(final Long memberId) {
       Member member = memberRepository.findById(memberId).orElseThrow(
               () -> new IllegalArgumentException("Cannot found member, WrongId:<" +  memberId + ">"));
       return new MemberFindResponseDto(member);
    }

    public Long join(final MemberJoinRequestDto requesteDto) {
        Member member = memberRepository.save(requesteDto.toEntity());
        return member.getId();
    }
}
```  

### Repository Class  
SpringDataJpa를 사용했다.  
필드가 많아지고 검색조건이 많아지면 이렇게 간단하게 안되겠지만 예제니까 정말 깔끔해보인다.  
```java
public interface MemberRepository extends JpaRepository<Member, Long> {
}
```  

### Domain Class  
가장 중요한 멤버 엔티티 클래스다.  
Hibernate를 사용하게 되므로 기본 생성자를 추가했다.  (사실 public안하고 protected로 하면 더 좋긴함)  
```java
@NoArgsConstructor
@Getter
@Table(name = "members")
@Entity
public class Member {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "name", nullable = false, length = 128)
    private String name;

    @Column(name = "age", nullable = true)
    private Integer age;

    @Builder
    public Member(String name, Integer age) {
        this.name = name;
        this.age = age;
    }
}
```  

### Dto Class  
클라이언트와 주고받을 데이터로 엔티티를 사용하면 안되므로 별도의 Dto 클래스를 작성했다.  
* 엔티티를 직렬화/역직렬화에 사용하면 성능/보안 이슈 및 코드 유지보수에 어려움이 발생할 수 있다.  

```java
// 멤버 조회 
@Getter
public class MemberFindResponseDto {
    private Long id;

    private String name;

    private Integer age;

    public MemberFindResponseDto(Member entity) {
        this.id = entity.getId();
        this.name = entity.getName();
        this.age = entity.getAge();
    }
}

// 멤버 가입
@Getter
public class MemberJoinRequestDto {
    private String name;

    private Integer age;

    public MemberJoinRequestDto(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    public Member toEntity() {
       return Member.builder().name(name).age(age).build();
    }
}
```  

API(/member)가 제대로 동작하는지 아래와 같이 단위 테스트 코드를 작성했다.  
```java
@WebMvcTest
public class MemberControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private MemberService memberService;

    @Test
    @DisplayName("회원가입 확인")
    void joinMember() throws Exception {
        MemberJoinRequestDto requesteDto = new MemberJoinRequestDto("chpark", 34);

        // when
        // any : 어떤 타입으로 입력이 들어오든 넘어가기위해 설정
        when(memberService.join(any())).thenReturn(1000L);

        mockMvc.perform(post("/member")
                .contentType(MediaType.APPLICATION_JSON)
                .content(new ObjectMapper().writeValueAsString(requesteDto)))
                .andExpect(status().isOk())
                .andExpect(content().string("1000"))
                .andDo(print());
    }

    @Test
    @DisplayName("회원조회 확인")
    void findMember() throws Exception {
        MemberFindResponseDto responseDto = new MemberFindResponseDto(new MemberJoinRequestDto("chpark", 34).toEntity());

        when(memberService.find(1L)).thenReturn(responseDto);

        MvcResult result = mockMvc.perform(get("/member/1"))
                .andExpect(status().isOk())
                .andDo(print()).andReturn();
        DocumentContext documentContext = JsonPath.parse(result.getResponse().getContentAsString());
        Assertions.assertThat((String)documentContext.read("name")).isEqualTo("chpark");
        Assertions.assertThat((int)documentContext.read("age")).isEqualTo(34);
    }
}
```  

오늘 작성할 포스팅의 목적인 테스트 코드다!  
<br>
<strong_purple>@WebMvcTest 애너테이션을 사용하면 Controller만 떼어내서 테스트가 가능하다.</strong_purple>  
@Controller, @ControllerAdvice, @JsonComponent, @Converter 등등 Web과 관련된 Bean들만 등록된다.  
따라서 @Service, @Repository와 같이 Web과 직접 관련이 없는 Bean들은 사용할 수 없다.  

하지만 Controller에서 Service를 호출할텐데 이 경우 어떻게 해야 하는가?  
Service가 Bean으로 등록되지 않으니까 수동으로라도 생성해줘야 되지 않을까?  
만약 테스트 코드에서 MemberService memberService = new MemberService(); 와 같이 생성한다면  
코드 내에서 컴파일 오류는 피할지도 모르겠다.  
__하지만 Controller가 의존하고 객체는 이 객체가 아니지않은가?__  
결국 의존이 끊긴 Service에 접근하여 NPE가 발생할 것이다.  
<hr>

<strong_red>이렇게 의존해야하지만 등록되지 않은 Bean을 사용하기위해 Mock 이란 기능에 제공된다.</strong_red>  
이름에서 "가짜" 라는 느낌이 오는것처럼 가짜로 객체를 만들어서 "진짜"처럼 행동하게 만들어준다.  
먼저 Mock 기능을 사용하기위해 Mockito 객체를 주입받는다.  

```java
@Autowired
private MockMvc mockMvc;
```  

그리고 Service를 @MockBean을 사용해서 가짜 Bean으로 등록해준다.  

```java
@MockBean
private MemberService memberService;
```  

이렇게 테스트 코드를 실행하면 MemberController는 MemberService에 의존하기위해 위의 MockBean을 사용하게 된다.  
그런데 MemberController의 join 메서드를 보면 MemberJoinRequestDto 객체를 전달받는다.  
그리고 생성된 회원Id를 반환하는데, 저 가짜 memberService가 무슨 값을 던져줄까?  

<strong_black>테스트해보니까 그냥 0으로 반환한다.  일반 객체라면 null이 반환될 것 으로 예상된다.</strong_black>  

그렇다면 항상 회원Id는 0이 반환될텐데, 이것만보고 '아~ HttpStatus=200 이면 정상이네. 성공' 이라고 말할 수 있을까?  

나는 처음 가입한 순서대로 Id를 발급할거라 처음 가입한사람의 Id가 1이 맞는지 보고 싶은데 이건 어떻게 테스트해야하지?  

이걸로 좀 삽질 많이했다.  
<hr>

```java
MemberJoinRequestDto requesteDto = new MemberJoinRequestDto("chpark", 34);
// when
when(memberService.join(requestDto)).thenReturn(1000L);
```  

처음엔 그냥 이렇게 하면 잘 되겠거니 생각으로 테스트를 진행했는데 테스트는 실패한다.  
membserService.join 결과가 0으로 반환되었기 때문이다.  

"어? 분명 1000으로 반환하게끔 했는데 왜 안되지?"  

몇 분 삽질하며 생각했는데, 어찌보면 당연히 테스트는 실패해야된다.  
왜냐하면 테스트코드에서 생성한 requestDto가 ObjectMapper에 의해 직렬화되고,  
다시 서버에서는 이를 역직렬화하여 requestDto를 생성할 것이다.  
__이 직렬화/역직렬화 하기 전의 이 두 객체가 같은 인스턴스인가?__  

아닐것이다.  
값은 같겠지만 둘은 엄연히 다른 공간에 생성된 다른 인스턴스다.  
따라서 실패했을거라 생각한다. 
~~(실제 코드를 본게 아니라 위험한 가설이긴하다. 이건 다시 찾아서 확실하게 짚고 넘어가야겠다.)~~

그럼 테스트는 아예 불가능한것인가?  
여기서 조금 시간을 많이 소모했다. 아는만큼 보일텐데 아는게 별로없어서..  

열심히 찾고 또 찾다보니 Mockito에서 제공해주는 __any()__ 라는 메서드가 있네?  

설명 대강봤더니 뭐 해석되는대로 아무값?이나 받는 것 같다.  

따라서 위의 when절을 아래와 같이 수정했다.  

```java
when(memberService.join(any())).thenReturn(1000L);
```  

![Alt](/asserts/img/dev/spring/../../../../../../../assets/img/dev/spring/dev-spring-membercontroller-test-success.png)  

테스트 코드 돌린결과 정상적으로 Id가 반환되었다!  
예제 코드 작성한건 20분도 안걸렸는데 테스트하느라 거의 2시간 쓴것 같다.  
~~그리고 지금 1시간째 이렇게 또 글을 쓰고있다...~~  

다음번엔 Service, Repository에 대한 테스트코드에 대해 작성해봐야겠다.  