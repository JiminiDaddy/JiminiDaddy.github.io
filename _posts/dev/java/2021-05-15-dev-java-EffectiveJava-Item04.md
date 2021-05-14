---
layout: post
title: [EffectiveJava3 Item04]
subtitle: 인스턴스화를 막으려거든 private 생성자를 사용하라
catogires: dev
tags: java
comments: true
---

# Item4. 인스턴스화를 막으려거든 private 생성자를 사용하라  

개발을 하다보면 굳이 객체로 생성안하고 static method로만 제공해줄 클래스가 종종 필요할 때가 있다.  
자주쓰이지만 가벼운 연산을 해주는 메서드를 제공한다던가 순수 함수만으로 구성된 클래스는 굳이 인스턴스화가 필요없다고 생각한다.  

클래스에 생성자를 정의하지 않으면 기본적으로 JVM은 public 제한자로 기본 생성자를 정의한다.  
public 이므로 외부에서 얼마든지 인스턴스화가 가능하다.  

<strong_red>따라서 인스턴스화를 막으려면 명시적으로 private 생성자를 정의해야 한다.</strong_red>  

private 생성자 대신에 추상 클래스로 만들경우 이를 상속받는 하위 클래스가 얼마든지 구현될 수 있으므로 인스턴스화가 충분히 가능하다.  
더군다나 abstract class라는 키워드를 본 개발자는 이 클래스를 보는 순간 상속받아야겠다는 생각이 본능적으로 들 것이다.  

private 생성자로 만들어도 리플렉션 공격으로는 생성자가 호출되어 인스턴스화가 가능하다.  
<strong_black>이 경우 생성자에 예외를 던져 인스턴스화를 막는 방법도 가능하다.</strong_black>  

아래의 예제는 인스턴스화를 막기 위한 Utility 클래스를 구현했다.  
```java
public class Utility {
    private Utility() {
        throw new AssertionError("Can not instances");
    }
}
```  

이제 테스트를 작성하여 정말 인스턴스화가 안되는지 확인해보자.  
```java
public class UtilityTest {
    @Test
    @DisplayName("Utility가 인스턴스화 불가능한지 테스트")
    void checkUtilityIsCannotInstances() throws IllegalAccessException, InvocationTargetException, InstantiationException {
        // private 생성자만 정의되어있으므로 아래와 같이 생성자호출은 컴파일 에러가 발생한다.
        //Utility utility = new Utility();

        Class c = Utility.class;
        for (Constructor constructor : c.getDeclaredConstructors()) {
            constructor.setAccessible(true);
            // 리플렉션을 통해 newInstance()를 수행할 때, 생성자에서 예외를 던지고, InvocationTargetException이 발생한다.
            Assertions.assertThatThrownBy(() -> {
                Utility utility = (Utility) constructor.newInstance();
            }).isInstanceOf(InvocationTargetException.class);
        }
    }
}
```  
주석처리된 Utility utility = new Utility(); 는 컴파일 에러가 발생하므로 테스트할 필요가 없었다.  
리플렉션 공격을 막을 수 있는지 테스트를 실행했고, 예상대로 리플렉션으로도 인스턴스화는 불가능할 수 있었다.  
![Alt](/assets/img/dev/effective-java/item04-test-success.png)  

평소에 리플렉션에 대해서는 깊이 고민한적이 없었는데 이펙티브 자바 책을 보면서 리플렉션에 대해 많이 고민할 수 있게 되었다.  
