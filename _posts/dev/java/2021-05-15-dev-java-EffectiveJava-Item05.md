---
layout: post
title: [EffectiveJava3 Item05]
subtitle: 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라
categories: dev
tags: java
comments: true
---

# Item5. 자원을 직접 명시하지 말고 의존 객체 주입을 사용하라  

Spring을 공부하고 사용했기때문에 사실 의존 주입이라는 용어는 굉장히 익숙하고, 실제로도 사용하고 있다.  
이번 장에서는 의존 주입이 필요한점, 싱글턴이나 유틸리티에서 자원을 소유하고 있는게 왜 안좋은지에 대해 가이드해준다.  

예제로 MemberService 클래스와 MemberRepository 인터페이스를 구현한 MemoryMemberRepository를 생성했다.  
```java
public class MemberService {
    private static final MemberRepository memberRepository = new MemoryMemberRepository();

    private static final MemberService INSTANCE = new MemberService();

    private MemberService() {}

    public static MemberService getInstance() {
        return INSTANCE;
    }

    public MemberRepository currentMemberRepository() {
        return memberRepository;
    }
}
```  
위 예제 코드는 MemberService를 싱글턴으로 구현했다.  
이 클래스의 가장 큰 문제는 MemberService 클래스가 로딩될 때, MemberRepository가 고정으로 생성된다는 것이다.  
__만약 MemoryMemberRepository가 아니라 JdbcMemoryRepository로 변경해야 하는 일이 발생한다면?__  

당연히 코드를 수정해야 한다.  

<strong_red>따라서 위 코드는 OCP를 명백히 위반한다.</strong_red>  

그렇다면 MemberRepository를 final이 아닌 그냥 static으로 생성한다면 어떨까?  

위 코드를 살짝 바꾸어봤다.  
```java
public class MemberService {
    private static MemberRepository memberRepository = new MemoryMemberRepository();

    // ... 이하 동일하므로 생략

    public void changeMemberRepository(MemberRepository memberRepository) {
        MemberService.memberRepository = memberRepository;
    }
}
```  
이제 MemberService는 싱글턴도 유지하고 MemberRepository도 런타임중에 얼마든지 변경할 수 있게 되었다.  

만약 MemberService가 단일스레드에서 동작한다면 별 문제 없어보인다.  
하지만 대다수 애플리케이션은 멀티 스레드 환경에서 구동된다.  
__10개의 스레드가 MemberService에 접근하고 로직에 의해 changeMemberRepository가 호출되는 경우가 발생한다면?__  

<strong_red>분명 어떤 서비스에서는 Jdbc가, 어떤 서비스에는 Memory로 동작해야 하는일이 있을텐데 이것이 보장되지 않는다.</strong_red>  
왜냐하면 모든 스레드가 하나만 생성된 MemberService 객체에 접근하기 때문이다.  
또한 동기화 처리가 되지 않으면 이 MemberService를 사용하는 애플리케이션은 어느순간 비정상적으로 종료될 것이다.  

이펙티브 자바에서는 아래와 같이 설명하고 있다.  
> 사용하는 자원에 따라 동작이 달라지는 클래스에는 정적 유틸리티 클래스나 싱글턴 방식이 적합하지 않다.  

__멀티 스레드에서 싱글턴과 같이 공유되는 객체는 반드시 불변이어야 한다.__  

따라서 MembserService는 아래와 같이 싱글턴을 버리고 인스턴스화 해야한다.  
<strong_red>인스턴스화할 때 생성자에 필요한 자원을 넘겨주는 방식을 의존 주입(DI)이라 하는데 굉장히 많이 사용하는 패턴이다.</strong_red>  

```java
public class MemberService {
    private final MemberRepository memberRepository;

    public MemberService(MemberRepository memberRepository) {
       this.memberRepository = memberRepository;
    }

    public MemberRepository currentMemberRepository() {
        return memberRepository;
    }
}
```  
<strong_blue>위와 같이 구현하면 MemberService는 생성될 때 MemberRepository를 외부에서 주입받게 되며, 단 한번만 할당되므로 불변이 보장된다.</strong_blue>  

개발 초기에 DB가 결정되지 않아 MemoryMemberRepository로만 구현하여 사용하다가 DB가 확정이 되면?  
기존에 MemberService로 전달한 MemoryMemberRepository를 JdbcMemberRepository 객체로 변경만 하면 된다.  
Spring의 경우 위와 같이 Bean의 생성을 결정하는 역할은 설정 클래스가 담당하는데 설정 클래스의 코드만 변경하면 된다.  
즉 기존 로직은 변경이 전혀 일어나지 않아 OCP가 지켜진다.  

생성자에 자원을 주입하는 방식만 테스트했는데 이펙티브 자바에서는 생성자에 팩터리를 주입하는 방식도 설명하고 있다.  
시간이 늦어 따로 테스트는 못했는데 내일 시간되면 해봐야겠다.  

