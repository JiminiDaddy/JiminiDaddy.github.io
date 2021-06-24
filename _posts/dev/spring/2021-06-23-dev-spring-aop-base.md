---
layout: post
title: "[Spring] AOP"
subtitle: "AOP란 무엇인가? 어떻게 사용하는가?"
categories: dev
tags: spring
comments: true
---

Spring Framework을 사용하여 개발을 시작할 때 MVC에 대한 이해와 사용을 먼저 시작했다.  
내부적으로 어떻게 돌아가는지는 일단 넘어가고 클라이언트의 요청을 어떻게 Controller가 받아내는지, Controller는 무엇을 해야하는지, Services는 어떻게 구현하면되는지 등등 구현에 목적을 두었던 것 같다.  

이와같은 방법이 꼭 나쁘다고는 생각하지 않는다.  
내가 아직은 역량이 낮은 개발자라서 그럴진 몰라도 처음엔 일단 생각했던대로 구현이 되는것을 눈으로 보는게 좋다고 생각한다.  
일단 의도된대로 프로그래밍이 완성된 뒤에 하나씩 파헤쳐보는게 책이나 문서를 보면서 뭔지도 모르는걸 억지로 이해하려는것보다 낫다는 생각이 든다.  

시중에 나와있는 대부분의 Spring 책들도 구현 위주로 되어있다.  
덕분에 2019년 초 Spring에 대해 거의 대부분을 모른 상태였지만 이후 책 3권정도를 읽고 그대로 구현해보고 살짝 응용도 해보면서 API를 구현해보고 화면도 만들어 연동해보고 데이터베이스도 연동해보면서 하나씩 배워나갈 수 있었다.  
~~덤으로 자신감도 붙고 뭔가 성장해 나가는 기분도 들고...~~  

토비의 스프링을 만나기 전까지는.  

부끄러운 얘기지만 Java/Spring 개발을 시작한지 3년정도 되가는데 아직도 토비의 스프링 1,2권을 다 읽지 못했다.  
아니 정확히 말해서 1권도 채 완독한적이 없다. 2권은 시작도 안했고.  

토비의 스프링이 나올때보다 지금의 스프링이 굉장히 많이 발전해서 별로 도움이 안되지 않을까 싶었는데 이 책은 나온 의도가 아예 다르다.  
1권에서는 스프링 프레임워크에 대해 어떻게 사용하는지에 대해서는 얘기하지 않는다.  
OOP를 위해 어떻게 코드를 작성해야하는지, 스프링이 OOP를 지키기 위해 어떤 노력을 했는지부터 시작해서 그 어렵고 어려운 AOP가 중반? 쯤에 나온다.  
처음 읽었을 때 AOP 나오고 아마 책 덮은 것 같고, 두번째 읽었을 때는 다 이해하지 못했지만 뒤에도 슥슥 넘어가면서 보았는데 이제 그때보단 조금 더 성장했으니 다시 한번 읽을 때가 된 것 같다.  
(요즘 Spring에 대해 좀 더 깊게 이해해야 할 것 같다는 생각이 무지무지 많이 들기 때문이다.)  

서론이 길었는데 오늘은 AOP에 대해 간단히 공부한 내용을 정리해보려고 한다.  

---

## AOP란?  

Asepct Oriented Programming의 약자로 블로그를 보든 책을보든 관점지향 프로그래밍이라고 나온다.  
관점지향이 뭐지? 생각이 들 수 있는데 (나 또한 이 표현이 최선이었을까? 생각이 든다.)  
그냥 난 이렇게 이해했다.  

API를 개발하기위해서는 Controller부터 Service, Domain 등을 각각 구현해야 한다.  
구현하다보면 위 순서대로 잘 흘러가는지 테스트를 포함해 기능 검증을 하게 된다.  
이후 계속 개발 요청이오면 보통 저런 순서로 많이 개발을 진행하게 된다.  

S/W 기획은 계속해서 추가되고 변경되기 마련인데 이로인해 많은 기능을 구현하다보면 어느덧 개발자들이 극혐하는 "중복" 을 마주치는 순간이 온다.  
'아 저거 다른 API에서도 쓰고있는데..', '저긴 약간만 다르지 사실 거의 유사한 기능인데..' 이런 생각이 들 때가 안올 수가 없다.  

__왜냐하면 항상 같은 흐름대로 개발을 진행했기 때문이다.__  
항상 위에서부터 Controller -> Service -> Domain 로직을 생각하고 있고, 거기에 맞춰 개발을 진행했다.  
근데 이렇게 만든 기능이 엄청나게 많아졌고, 옆에서 보니까 자잘한 기능들이 군데군데 껴있다는걸 느끼게 된 것이다.  
AOP는 이런 거라고 생각한다.  
<br/>
<strong_red>위에서 흘러가는 곧은 방향대로 보는게 아니라 옆에서 제 3자 입장에서 보는 것.</strong_red>

마치 장기를 두는 선수의 눈으로 보는게 아니라 옆에 앉은 훈수꾼의 눈으로 게임을 바라보는 것.  

그래서 결론은 이렇게 많은 기능들에 중복된 기능들을 어떻게 분리해낼것인가? 이것이 AOP의 기본 목표가 아닐까 싶다.  
동일한게 2개 사용되는 기능은 분명 3개, 4개가 될 수있기 때문에 어떻게든 분리시켜야 한다.  

하지만 단순히 중복된다고 분리시키는게 AOP의 목표는 아니다.  
<strong_purple>AOP는 엄연히 말해 중복된 기능을 분리시키는게 아니라 도메인의 핵심 기능과 공통 기능을 분리시키는 것이다.</strong_purple>  

---

## AOP의 예  

예를들어 한 커머스 시스템이 있고, 제품 Q/A 게시판에 고객이 등록하면 Q/A 목록에 추가되고, 담당자에게 메일을 전송하는 기능을 구현했다고 하자.  
이후 리뷰기능이 개발되었고 이번엔 구매한 제품에 대해 리뷰를 쓰면 고객은 포인트를 적립하고 리뷰목록에는 이미지와 별점, 리뷰내용이 담겨있는 게시물이 저장된다. 그리고 담당자에게 메일이 전송되어 새로 등록된 리뷰에 대해 빨리 답글을 달도록 기능이 구현되었다.  
이후 시스템이 더 커졌고 결제완료된 주문은 최대한 빨리 발송 처리 및 배송업체로 넘기기위해 담당자에게 메일을 전송하는 기능을 구현했다.  

처음 기능을 구현할 땐 문제가 없었지만, 시스템 확장에 따라 계속 기능을 구현하다보니 "메일 전송" 이라는 기능이 여기저기 붙어 있다.  
메일 전송 기능이 없다고해서 Q/A, 리뷰, 발송 기능에 문제가 되는일은 없다.  
단지 좀 더 빨리 처리하여 고객께 만족을 주기 위해 부가적으로 구현한 기능일 뿐이다.  

이와같이 핵심 기능(Q/A, 리뷰, 발송)과 부가 기능(메일 전송)을 분리하여 비즈니스 로직에 핵심 기능만 구현하려는 방식이 AOP다.  

---

## AOP의 구성요소  

AOP를 구현하기 위해서는 필요한 구성요소들이 있다.  
용어가 익숙치 않고 사실 글로만 봐서는 잘 와닿지 않는다.  
하지만 한번 정리해보고 직접 구현해보면 "아~ 이거구나." 라는 느낌이 조금씩 들게 된다.  

### Aspect  

여러 도메인에 중복되어 있는 공통 기능을 모듈화한 공통 기능 모듈

### Target  

핵심 기능이 구현되어 있는 객체를 말한다.  
Aspect가 적용되는 곳으로써, 클래스나 메서드가 될 수 있다.

### Advice

언제 부가 기능(공통적으로 관심있어 중복된 기능)을, 핵심 로직에 적용할 것인가? 에 대해 정의한다.  
예를들어 A 메서드를 호출한 후에(언제), 메일을 전송한다(공통 기능) 라는 기능을 적용하겠다고 정의한다.  

### Joinpoint

Advice를 적용할 수 있는 지점을 말한다.  
메서드가 호출되거나, 필드 값 변경 등이 해당된다.  
__단 Spring-AOP는 Proxy를 이용해서 구현되기 때문에 메서드 호출에서만 JoinPoint가 지원된다.__  

### Pointcut

Advice가 실제로 적용되는 Joinpoint를 말한다.  
정규표현식이나 AspectJ 문법을 사용해서 표현할 수 있다.  

### Weaving

Pointcut에 의해 결정된 Target의 Joinpoint에 Advice를 삽입하는 과정을 말한다.  
Target(핵심 기능)의 코드를 변경하지 않으면서 Advice(필요한 공통 기능)를 추가할 수 있는 AOP의 핵심 처리과정이다.  

---

## AOP 적용 방법  

핵심 기능에 공통 기능을 삽입하는 방법으로, 크게 3가지 시점에 적용할 수 있다.  

1. 컴파일 시점  
   Java 파일을 Class파일로 컴파일할 때, Advice를 추가한 조작된 바이트 코드를 생성한다.  
   클래스를 로딩하거나 런타임에서 성능 저하가 발생하지 않지만 별도의 컴파일 과정이 필요하다.  
   컴파일을 위해 별도의 AOP 도구(AspectJ)가 필요하다.  

2. 클래스 로딩 시점  
   컴파일은 순수한 Java코드로 진행하고, 클래스가 로딩될 때 바이트 코드에 Advice를 삽입하는 방식이다.  
   클래스가 로딩될 때 바이트 코드 조작이 일어나므로 로딩 시점에서 부하가 발생한다.  

3. 런타임  
   런타임에 프록시 객체를 생성하여 프록시에 Advice를 삽입하는 방식이다.  
   Spring은 런타임에 SpringContainer가 Bean들을 탐색해서 생성하고 등록하는데, 이 때 Proxy-Bean도 함께 진행한다.  
   요청된 핵심 기능의 메서드가 호출되기 직전에 Proxy-Bean의 메서드가 호출되어 Advice를 실행한다.  
   이후 Proxy-Bean이 핵심 기능을 실행하기 위해 Target을 호출한다.  
   Proxy-Bean이 생성되고 SpringContainer에 등록되야하므로 최초 런타임에 부하가 발생할 수 있다.  

<strong_red>Spring AOP는 3번 런타임 방식을 사용한다.</strong_red>  

---

## Proxy 방식을 사용하는 Spring AOP

런타임에 프록시 객체를 통해 공통 기능을 실행하고, Target을 호출해 핵심 기능을 실행한다.  
핵심 기능은 Target에 존재하며, Proxy는 절대로 핵심 기능을 구현하지 않는다.  

Spring-AOP의 동작 과정은 아래와 같이 간단하게 표현할 수 있다.  

<div class="mermaid">
sequenceDiagram;
    Client ->> AOP-Proxy : Q/A 게시물 작성
    AOP-Proxy ->> Target : 게시물 작성 (핵심 기능)
    Target ->> AOP-Proxy : 핵심 기능 처리 결과
    AOP-Proxy ->> Aspect : 메일 전송 (공통 기능)
    AOP-Proxy ->> Client : 게시물 작성 결과
</div>

Spring-AOP는 Proxy를 통해 위와 같이 흘러가며 핵심 기능과 부가 기능이 실행된다.  
참고로 Proxy를 생성하는 방식으로도 크게 JDK-Dynamic-Proxy와 CGLib-Proxy 2가지 방식이 존재한다.  

~~이 2가지에 대한 비교와 내용만 해도 양이 많으므로 다음 포스팅에서 진행할 예정이다.~~  

---

## Spring-AOP 예제  

메일 전송은 관련된 라이브러리도 써야하고 예제로써 적당하지 않을 수 있어 엄청 간단한 코드로 예제를 작성하려 한다.  
핵심로직의 수행 시간 측정!  

예제 내용은 간단하다.  

1. Calculator Interface를 구현한 클래스 2개를 Bean으로 등록한다.  
   하나는 AOP를 사용하고 하나는 AOP를 사용하지 않는다.  
2. Aspect Class를 하나 구현한다.  
   소요시간을 측정하기 위한 클래스로, Advice와 Pointcut을 지정한다.  
3. Main Class에서 Calculator Bean들을 생성하고 실행한다.  

먼저 Calculator Interface와 구현체 코드다.  

```java
public interface Calculator {
    int sum(int start, int end);
}

// AOP를 사용할 Calculator Bean
@Component
public class MiniCalculator implements Calculator {
    public int sum(int start, int end) {
        int result = 0;
        for (int i = start; i <= end; ++i) {
            try {
                result += i;
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        return result;
    }
}

// AOP를 사용하지 않을 Calculator Bean
@Component
public class NoAspectCalculator implements Calculator{
    @Override
    public int sum(int start, int end) {
        int result = 0;
        for (int i = start; i <= end; ++i) {
            try {
                result += i;
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        return result;
    }
}
```

예제니까 메서드는 대충 작성했다.  
그냥 start, end 값을 받아서 1ms씩 sleep주고 합을 반환한다.  

다음은 오늘의 주인공 Aspect Class 코드다.  

```java
@Component
@Aspect
public class ExecuteTimeAspect {
    // MiniCalculator만 AOP가 적용되도록 Pointcut 지정
    @Pointcut("execution(public * com.chpark.aop.domain.MiniCalculator.*(..))")
    private void publicTarget() {
    }

    // 소요시간 측정
    @Around("publicTarget()")
    public Object measure(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long finish = System.currentTimeMillis();
        Signature signature = joinPoint.getSignature();
        System.out.printf("%s.%s(%s)실행 시간: %d ms\n",
            joinPoint.getTarget().getClass().getSimpleName(),
            signature.getName(),
            Arrays.toString(joinPoint.getArgs()),
            (finish - start)
        );
        return result;
    }
}
```

MiniCalculator 클래스에 한해서만 포인트컷을 지정하였다.  
이제 MiniCalculator의 메서드가 호출되려고 하면 MiniCalculator-Proxy(가칭)의 메서드가 먼저 호출되고,  
핵심 기능을 실행하기 전에 공통 기능을 먼저 실행하게 된다.  

이제 마지막 MainClass다.  

```java
@EnableAspectJAutoProxy
@SpringBootApplication
public class MainApplication {
    public static void main(String[] args) {
        SpringApplication application = new SpringApplication(MainApplication.class);
        application.setWebApplicationType(WebApplicationType.NONE);
        ApplicationContext context = application.run(args);

        System.out.println("\nUse Spring-AOP");
        Calculator calculator = context.getBean(MiniCalculator.class);
        int result = calculator.sum(1, 1000);
        System.out.println("result: " + result);
        System.out.println("bean: " + calculator.getClass().getName());

        System.out.println("\nNot Use Spring-AOP");
        Calculator noAspectCalculator = context.getBean(NoAspectCalculator.class);
        result = noAspectCalculator.sum(1, 1000);
        System.out.println("result: " + result);
        System.out.println("bean: " + noAspectCalculator.getClass().getName());
    }
}
```

@EnableAspectJAutoProxy를 선언해서 Aspect Bean을 Springd

MiniCalculator와 NoAspectCalculator가 모두 Calculator Interface를 구현하였으므로 Bean type을 Calculator로 지정하면 중복 오류가 발생한다.  
따라서 각 구현체로 Bean을 가져오도록 구현하였다.  
코드의 의도는 MiniCalculator의 실행결과는 소요시간이 측정되야하며, Bean이름도 프록시 객체의 이름이 출력되어야 한다.
반면 NoAspectCalculator는 실행 결과값만 출력하고, Bean이름은 자기 자신이 출력되어야 한다.  

예상한대로 결과가 아래와 같이 잘 나왔다.  
![Alt](/assets/img/dev/spring/dev-spring-aop-cglib-result.png)

---

여기까지 AOP의 간단한 배경지식과 예제를 구성해보았다.  
알듯말듯 AOP는 어려운 것 같다.  
대표적인 AOP로 Transaction을 사용할텐데 항상 Annotation만 쓰면 되었지 내부 동작에 대해 고민을 많이 안해보았기 때문이겠지?  

다음번엔 JDK-Dynamic-Proxy와 CGLib-Proxy의 차이와 구동원리에 대해 작성해보려고 한다.  

사실 위 예제를 실행하면서 JDK-Dynamic-Proxy가 실행되어야 하는게 아닌가? 생각이 들었다.  
SpringBoot가 기본적으로 CGLib를 사용한다고는 알지만 Interface를 구현했는데?  

왜 CGLib가 선택되었는지 분석결과도 함께 작성해보아야겠다.  
