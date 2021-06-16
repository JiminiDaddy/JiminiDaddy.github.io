---
layout: post
title: Java의 접근제한자
subtitle: 접근제한자별 특징
categories: dev
tags: java
comments: true
---

## Java의 접근제한자  

Java, C++과 같이 객체지향을 다루는 언어들은 기본적으로 접근 제한자가 있다.  
물론 다른 언어들에서도 명칭은 달라도 최소한의 구분은 있다고 알고 있다. (예를들어 Python)  

평소 그냥 대수롭지 않게 사용했던 접근제한자에 대해 한번 정리해보려고 한다.  

__Java에서 제공하는 접근제한자는 public, protected, private, default 4가지로 나뉘어져 있다.__  

---

### public  

보통 외부에 인터페이스로 제공할 때 사용한다.  
public으로 선언된 클래스의 객체는 패키지에 제한없이 어떤 곳에 있는 객체에서도 접근이 가능하다.  
클래스와 메서드가 모두 public인경우 외부에서 해당 객체로 메시지 전송이 가능하다.  
(클래스만 public이고 메서드는 다른 접근제한자를 사용한다면 조건에 따라 접근이 가능할수도, 불가능할수도 있다.)  
public 인터페이스를 제공할경우 API 버전이 향상되어도 변경하기 어려우므로 항상 신중하게 설계해야 한다.  

---  

### private  

클래스의 내부 기능을 구현할 때 사용한다.  
외부는 물론이고 같은 패키지에서도 접근이 불가능하고 해당 클래스 내에서만 접근이 가능하다.  
(단, Reflection을 이용하면 외부에서도 접근이 가능하다.)  
대개 외부로 제공할 인터페이스는 public으로 선언하며, 실제 구현할 기능은 private으로 선언한다.  
이런 패턴이 가진 가장 큰 장점으로는 변경에 용이하다고 할 수 있겠다.  
외부에 제공한 인터페이스(API)에 버그가 생겨 패치하거나, 혹은 기능이 확장되는 경우가 있다고 가정할 때  
내부에 구현한 기능까지 public 메서드로 제공한다면 해당 메서드를 사용하고 있는 클라이언트로 인해 함부로 수정하기가 어렵다.  
기능 변경으로 인해 메서드의 반환 타입이나 시그니처가 변경될 수도 있는데 수정하지 못한다는것은 꽤 불편하게 작용될 것이다.  
하지만 내부 기능을 private 제한자로 캡슐화하면 얼마든지 패치하여 배포할 수 있다.  
왜냐하면 외부에서는 public 인터페이스로 전달한 메시지를 통해 원하는 결과값만 가져가면 되기 때문이다.  
  
<strong_red>다시말해 클라이언트는 내부 구현에 대해서는 신경쓰지않고, 원하는 결과값만 받으면 된다.</strong_red>  

---

### protected  

때로는 특정 클래스나 특정 패키지로는 공개하고싶고, 그 외에는 공개하고 싶지 않은 클래스 설계가 필요할 때가 있다.  
메서드를 protected로 선언하면 자신을 상속받은 클래스 및 동일 패키지에 존재하는 클래스에게는 public처럼 제한을 풀어주고,  
그 외의 모든 클래스에게는 private으로 접근을 제한한다.  
참고로 protected는 클래스에는 선언할 수 없고, 메서드에만 선언이 가능하다.  
외부에서 protected 선언된 메서드에 접근하면 아래와 같은 컴파일 오류가 발생한다.  
![Alt](/assets/img/dev/java/java_access_modifier_protected_cannot_call.png)

상속받은 자식 클래스는 부모 클래스에서 public, protected, default로 정의된 메서드를 오버라이딩할 수 있다.  
  
<strong_red>참고로 Overriding은 부모 클래스에서 선언한 접근제한자보다는 범위가 넓은 접근제한자를 사용해야 한다.</strong_red>  
  
예를들어 부모 클래스에서 protected로 선언된 메서드는 자식 클래스에서 protected 혹은 public 제한자만 사용할 수 있다.  

여기서 한가지 잘 몰랐던게 default vs protected 였다.  
default는 같은 패키지내에서만 접근이 가능하고, protected는 상속받은 클래스에서만 접근이 가능하다고 알고 있었다.  
좀 더 찾아보니 __protected로 선언된 메서드는 상속받지 않아도 동일 패키지 내에서는 접근이 가능하다는것을 알게되었다.__  

이 두 제한자 중 누가 더 넓은 범위인지 헷갈렸는데 이번 기회에 default보단 protected가 제한이 약하다는 것을 알게 되었다.  
  
<strong_red>따라서 private < default < protected < public이 되겠다.</strong_red>  
  
테스트해본 결과 protected가 좀 더 넓다고 판단할 수 있었다.  
왜냐하면 부모 클래스의 protected 메서드를 자식 클래스가 오버라이딩할 때, protected는 가능하지만 default는 불가능했기 때문이다.  
아래와 같이 부모 클래스에서 protected로 선언한 메서드가 있다.  

```java
public class User {
 private int age;

 protected void setAge(int age) {
  this.age = age;
 }
}
```

그리고 User 클래스를 상속받은 Employee 클래스를 구현했다.  

```java
public class Employee extends User {
 @Override
 protected void setAge(int age) {
  super.setAge(age + 10);
 }
}
```

User클래스의 setAge 메서드는 protected로 선언되었으므로 Employee에서 protected로 오버라이딩이 가능했다.  
하지만 Employee 클래스의 setAge 메서드를 default로 변경하면 컴파일 오류가 발생한다.  
![Alt](/assets/img/dev/java/java_access_modifier_protected_to_default_cannot_overriding.png)

오류 내용을 해석해보면 setAge 메서드가 'protected'보다 더 약한 제한인 'package-private'을 사용했다는 뜻이 된다.  

* 참고로 package-private은 IntelliJ에서 정의한 명칭으로, 'default'와 동일하다.  

---  

### default  

아래와 같이 클래스나 메서드에 public, private, protected와 같은 접근제한자가 없는 기본 형태를 default라고 한다.  

```java
class User {
    User() {
    }
}
```

위 User클래스는 class 앞에 접근제한자가 없으므로 default 제한자로 설정된다.  
default 접근제한자의 특징은 같은 패키지 내에서만 접근이 가능하다.  
즉 위와같이 User 클래스가 정의될 경우 아래와 같은 특징들이 생긴다.

1. User 객체는 User 클래스가 위치한 패키지 외의 다른 패키지에서 접근이 불가능하다.  
외부에서 User 클래스로 접근을 시도하면 아래와 같이 컴파일 오류가 발생한다.  
![Alt](/assets/img/dev/java/java_access_modifier_default_error.png)

2. 다른 패키지에서 User Class를 상속받는 자식 클래스를 구현할 수 없다.  
![Alt](/assets/img/dev/java/java_access_modifier_default_cannot_extends.png)

외부로는 제공하지 않고 애플리케이션의 특정 로직에서만 사용할 목적으로 개발한다면 default를 사용하기에 좋을 듯 싶다.  

---

접근제한자에 대해서는 총 이 정도로 정의할 수 있겠다.  

public, private이야 워낙 많이 사용하니까 딱히 적을게 없었는데 default와 protected는 알고 있다고 생각했지만  
또 100%는 몰랐던 내용이길래 신선해서 작성해보았다.  

전에 인터뷰때 protected vs default에 대한 질문이 있었는데 인터페이스의 default method에 대한 얘기를 해버린 멍청한 기억이 있다.  
사실 그때 머리가 좀 하얗게 된 상태라서 default 접근 제한자에 대한 생각이 잘 안났다.  
~~(IntelliJ를 주로 사용하다보니 default보다 pakcage-private이 더 친숙해서 그런가..)~~  

예전에 인터뷰 끝나고나서 공부해야지 하고 키워드 적어놓은 것들이 많은데 언제 다 공부할까.......?