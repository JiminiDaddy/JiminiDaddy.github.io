---
layout: post
title: "[Spring AOP] Proxy"
subtitle: "Spring AOP가 Proxy를 생성하는 2가지 방식"
categories: dev
tags: spring
comments: true
---

## Spring AOP의 프록시 생성 방식

Spring AOP는 Proxy 객체를 생성하여 Target(핵심 기능)에 Aspect(공통 기능)를 삽입한다.  
여기에서 Proxy 객체를 생성하는 방식이 크게 2가지가 있다.  

첫 번째는 Java에서 지원하는 JDK Dynamic Proxy, 두 번째는 CGLib 방식이 있다.  

---

## JDK Dynamic Proxy

Java 1.3에서 생긴 기능이고 Interface 기반으로 Proxy 객체를 생성한다.  
만약 인터페이스가 아닌 클래스에 AOP를 사용하려고 하면 CGLib방식으로 동작하게된다.  
<br/>
<strong_red>이 방식은 InvocationHandler 인터페이스를 구현하게되는데, 런타임에 Target에 대해 Reflection을 사용한다.</strong_red>  
  
따라서 다소 성능이 떨어지는 단점이 있다.  
그리고 Target이 반드시 인터페이스를 구현해야 하는 단점이 있다.  
과거 SpringMVC의 경우 Service Bean만 하더라도 인터페이스를 만들고, 인터페이스를 구현한 ServiceImpl을 사용하는 패턴이 많았다.  

인터페이스를 분리시키는 방식은 OOP중 ISP를 지키는 좋은 규칙이지만 사실 Spring에서는 이 방식이 그렇게 무조건적일 필욘 없다고 생각한다.  
ISP의 강점은 다형성과 관계가 깊은데, 코드 변경없이 런타임에 다른 구현체로 바꿔도 타입이 일치하면 정상 동작하게끔을 보장해준다.  
하지만 Spring Bean이 런타임 중 다른 구현체로 바뀔 일이 극히 드문 특징이 있다.  
> 하지만 토비의 스프링에서 인터페이스를 써야한다고 강조했던 것 같다. (그러니까 쓰는게 더 좋긴한거겠지..?)  

---

## CGLib PRoxy

SpringBoot는 굳이 불필요한 인터페이스를 사용하지 않고 Service Bean들도 그냥 클래스로 구현하여 사용한다.  
그래서인지 SpringBoot에서 AOP Proxy 객체는 기본적으로 CGLib Proxy를 통해 생성된다.  
(설정을 통해 JDK-Dynamic Proxy로 변경될 수 있다.)  

이 방식은 Refrection을 사용하지 않고, 상속을 통해 Proxy 객체를 생성한다.  
<br/>
<strong_red>런타임에 SpringContainer가 올라가면 Bean으로 등록할 클래스들을 탐색한 뒤 등록하는 과정이 있는데, 이 때 Proxy 객체가 함께 생성되어 Target대신 Bean으로 등록된다.</strong_red>  
  
Proxy 객체는 Target(원본 대상객체)을 상속받아 구현된다.  
누군가 Target의 메서드를 호출하려고하면 Bean으로 등록된 Proxy 객체가 호출되고, Proxy는 내부에 감싸고있는 Target을 호출하게된다.  

즉, Target의 메서드를 Proxy 객체가 Overriding한 구조가 된다.  

Reflection이 일어나지 않아 JDK-Dynamic Proxy보다 성능이 낫고, 인터페이스도 필요없는 장점이 있지만 단점이 하나 존재한다.  

상속을 사용해야하므로, Target 클래스는 반드시 final로 정의할 수 없다.  
(final class는 상속 불가능하다)  
그리고 기본 생성자를 통해 상속받으므로 private이 아닌 접근제한자의 기본 생성자가 필요하다.
> 이 부분은 현재(Spring4.0이상)는 개선되었고, 생성자에 파라미터가 있어도 가능하다. 단, 생성자의 파라미터는 Spring Bean만 가능하다.  

만약 private 생성자를 정의해놓았다면 상속에 실패하여 런타임 오류가 발생한다.  

아래 이미지는 private 생성자를 추가했을 경우 발생하는 런타임 에러다.  
![Alt](/assets/img/dev/spring/dev-spring-aop-cglib-fail-log1.png)

![Alt](/assets/img/dev/spring/dev-spring-aop-cglib-fail-log2.png)

로그를 보면 "BeanCreationException"이 발생했고, 그 이유는 "No visible constructors" 라는 것을 확인할 수 있다.  
해석해보면 __Proxy Bean을 만드려고 시도했지만, 생성자가 보이지 않아 실패했다.__  

---

## 예제 분석  

그럼 이제 지난번 예제를 다시 한번 짚어보려한다.  
분명히 Target인 MiniCalculator는 Calculator 인터페이스를 구현했다.  
그리고 Aspect도 제대로 구현했고, Pointcut도 문제 없었기에 AOP는 잘 동작했다.  
하지만 내가 의도한건 JDK-Dynamic Proxy인데도 불구하고 CGLib방식을 사용했다.  

### 시도 1

Main Class에서 @EnableAspectJAutoProxy를 명시적으로 선언했고, proxyTargetClass = false를 강제로 설정했다.  
![Alt](/assets/img/dev/spring/dev-spring-aop-EnableAspectJAutoProxy-proxyTargetClass-false.png)

proxyTargetClass의 기본값이 false이기 때문에 사실 이 방법은 별 의미 없을 것이라 생각하긴 했다.  
역시나 결과는 CGLib를 사용했다.  

그리고 좀 더 찾아봤는데 SpringBoot의 자동 설정에 의해 @EnableAspectJAutoProxy는 자동으로 불러진다고 한다.  
Main Class에서 굳이 추가 안해줘도 AOP가 동작한다.  

### 시도 2

시도 1이 실패하고 다른 방법이 없을까 공식 문서도 들어가보고 블로그들도 많이 찾아보았다가 내 맘을 딱 알아주는 블로그를 하나 찾았다.  

[SpringBoot AOP에서 JDK Dynamic Proxy를 이용하는 법](https://multifrontgarden.tistory.com/m/282?category=799626)

설정 파일로 구성하는것을 생각못한 것은 아니었다.  
수동설정이 자동설정보다 우선인것도 알고 있었는데 따로 수동설정을 안해줬으니까 기본값인 자동설정으로 되어있을거라고 생각했다.  
하지만 SpringBoot 내부에서 이 값을 기본적으로 설정해주고 있나보다.  

SpringBoot는 application.properties 또는 application.yml 파일을 사용해서 환경을 수동으로 설정할 수 있도록 지원해준다.  

spring.aop.auto와 spring.aop.proxy-target-class 이 두 가지 값을 설정할 수 있다.

__auto 속성은 자동 설정으로 AOP를 활성화 시킬지 여부를 결정해준다.__  
기본값은 true인데 만약 false로 지정해주면 Application에서 Annotation으로 활성화하든 해야 한다.  

__문제는 두번째인 proxy-target-class 속성이다.__  
이 값은 기본값은 true로 되어있어 CGLib를 사용한다.  
암만 Application에서 false로 지정해주어도 프로퍼티 파일에 의한 수동설정이 우선이기 때문에 CGLib가 사용된 것이었다.  

이제 아래와 같이 JDK-Dynamic Proxy를 사용할 수 있도록 설정해주었다.  
![Alt](/assets/img/dev/spring/dev-spring-aop-properties-file-aop-setting.png)

그리고 코드를 실행하면 JDK-Dynamic Proxy를 사용한 로그가 출력된다.  
(Proxy객체 이릠에 CGLib가 붙으면 CGLib방식, Proxy가 붙으면 JDK-Dynamic Proxy방식이다.)  
![Alt](/assets/img/dev/spring/dev-spring-aop-jdk-dynamic-proxy-result.png)

참고로 CGLib를 사용하면 아래와 같은 결과가 출력된다.  
![Alt](/assets/img/dev/spring/dev-spring-aop-cglib-result.png)

---

Spring-AOP가 무엇인지, 어떻게 동작하는지, Proxy 방식이 무엇인지에 대해 공부해볼 수 있었다.  
~~다만 글 쓰는데 시간이 너무 걸리는것 같다..~~  

Pointcut을 잘 쓰려면 정규식에 좀 익숙해져야 할 것 같다는 생각이 든다.  
간단한 예제에 사용한 정규식도 책에 있는거 보고 약간만 응용한 정도인데 사실 다 지우고 쓰려면 잘 안된다.  
~~한 100번정도 써보면 익숙해지겠지?~~  
