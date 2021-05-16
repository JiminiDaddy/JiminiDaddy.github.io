---
layout: post
title: [EffectiveJava3 Item06]
subtitle: 불필요한 객체 생성을 피하라
categories: dev
tags: java
comments: true
---

# Item6. 불필요한 객체 생성을 피하라  

이번 장도 Java라는 언어를 공부해봤거나 실무자로 있는 개발자들은 충분히 알고 있는 내용인 듯 하다.  
Java에서 객체를 생성하는 방법은 new 키워드를 통해 클래스를 인스턴스화다.  
__이렇게 생성된 객체는 Heap에 저장되고 그 주소값을 반환함으로써 우리는 해당 객체에 접근 가능한 참조 객체를 사용할 수 있다.__  
즉 아래와 같은 코드가 있다고 할 때  
```java
User user = new User();
```  
new로 생성된 User 클래스의 인스턴스는 Heap에 저장되어 있다.  
우리가 생성한 객체 user는 Heap에 저장된 객체가 아니라, Heap에 저장된 위 객체를 참조할 수 있는 주소를 갖고있는 참조 객체일 뿐이다.  

다만 자바에서 예외적으로 new를 사용하지 않아도 생성할 수 있는 객체들이 있는데 대표적인게 String Class와 기본 타입의 Wrapper Class들이 있다.  
다음 2가지는 같을까?  
```java
String s1 = "Hello";
String s2 = new String("Hello");
```  
테스트를 해보면 알겠지만 둘의 동등성 관계(equals)는 성립하지만 동일성 관계(==)는 실패한다.  
<strong_red>왜나하면 둘은 같은 문자열을 갖고있는 객체를 가리키지만 그 두 객체가 서로 다른 객체이기 때문이다.</strong_red>  

s1과 같이 리터럴 형태로 String을 생성하면 String Pool에 있는 객체를 참조한다.  
따라서 "Hell"를 리터럴로 생성하는 모든 String들은 이미 만들어진 객체를 재사용하므로 메모리 낭비가 없다.  
하지만 s2와 같이 new 키워드를 사용할 경우 사용할 때마다 객체가 생성된다.  
따라서 경우에 따라 사용하는 메모리가 많이 증가하거나, 혹은 GC로 인해 불필요한 작업이 발생할 수 있다.  

그러므로 무조건!!  
<strong_red>String을 사용해서 문자열을 얻고자 할 땐 무조건 리터럴 방식을 사용하도록 한다.</strong_red>

Boolean과 같은 Wrapper Class도 마찬가지다.  
Boxing/Unboxing 과정에서 성능 손실이 발생하기 때문에 new 연산자를 사용하지 않는다.  
Integer, Boolean 등 Wrapper Class에서는 정적 팩터리 방식을 사용해서 객체를 제공해주고 있다.  
```java
// 이렇게 문자열로부터 객체를 생성할 수도 있고, Primitive 값으로부터 객체를 생성할 수도 있다.
Boolean b1 = Boolean.valueOf("TRUE");
```  

Boolean.valueOf는 아래와 같이 구현되어 있다.  
```java
public static final Boolean TRUE = new Boolean(true);

public static final Boolean FALSE = new Boolean(false);

public static Boolean valueOf(String s) {
    return parseBoolean(s) ? TRUE : FALSE;
}
```  
<strong_red>TRUE, FALSE가 정적 필드로 미리 생성되어있고, valueOf가 호출되었을 때 해당 객체의 참조를 반환해준다.</strong_red>  
따라서 Boolean.valueOf를 통해서 생성하면 불필요한 객체 생성이 발생하지 않는다.  
* java9버전부터 이런 Wrapper Class들의 생성자를 통한 객체 생성은 Deprecated 되었다.  

Map 인터페이스에는 keySet() 이라는 메서드를 제공해준다.  
keySet 메서드는 Map에 담긴 key들을 set으로 반환해주는 기능이 있다.  
그럼 같은 Map에서 keySet을 여러번 호출하면 객체가 계속해서 생성될까?  
아래와 같이 테스트 코드를 작성해보았다.  
```java
public class KeySetTest {
    @Test
    @DisplayName("Map.keySet() 의 반환값은 항상 같은 인스턴스인지 확인")
    void testKeySetIsSameInstance() {
        Map<String,Integer> map = new HashMap<>();
        map.putIfAbsent("A", 1);
        map.putIfAbsent("B", 2);
        map.putIfAbsent("C", 3);
        map.putIfAbsent("D", 4);

        Set<String> set1 = map.keySet();
        Set<String> set2 = map.keySet();
        Assertions.assertThat(set1).isSameAs(set2);
        set1.remove("A");
        Assertions.assertThat(set1.size()).isEqualTo(set2.size());
    }
}
```  

결과는 set1, set2 두 객체는 동일성이 보장되었다.  
(isSameAs가 동일성비교, isEqualsTo가 동등성비교)  

ketSet 내부를 확인해보았다.  
```java
public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}
```  

keySet의 참조값을 반환해주고있다. 당연히 keySet()을 호출한 모든 객체는 같은 객체(keySet)를 참조하므로 동일성이 보장된 것이다.  

이펙티브 자바에서는 잘못된 객체 생성의 예로 AutoBoxing도 설명해주고 있다.  
long과 같이 기본형 타입의 연산결과를 Long과 같은 Boxing된 객체로 받을 때 불필요한 객체 생성이 발생할 수 있다는 점이다.  
아래와 같은 테스트 코드를 작성해보았다.  
```java
public class AutoBoxingTest {
    @Test
    @DisplayName("오토박싱, 언박싱이 반복해서 일어날 때 성능이 떨어지는지 확인")
    void checkAutoBoxingPerfomance() {
        Long sum = 0L;
        long startTime = System.currentTimeMillis();
        long result1 = sumAllIntegerByBoxing(sum);
        System.out.println("wasted<Boxing>: " + (System.currentTimeMillis() - startTime));

        long sum2 = 0;
        startTime = System.currentTimeMillis();
        long result2 = sumAllIntegerByUnboxing(sum2);
        System.out.println("wasted<Unboxing>: " + (System.currentTimeMillis() - startTime));

        Assertions.assertThat(result1).isEqualTo(result2);
    }

    private long sumAllIntegerByBoxing(Long sum) {
        for (long i = 0; i < Integer.MAX_VALUE; ++i) {
            // 이 과정에서 오토박싱된 Long 객체가 불필요하게 생성된다.
            sum += i;
        }
        return sum;
    }

     private long sumAllIntegerByUnboxing(long sum) {
        for (long i = 0; i < Integer.MAX_VALUE; ++i) {
            sum += i;
        }
        return sum;
    }
```  

길어보이지만 별거없고 Long += long의 연산과 long += long의 연산이 매우 많이 반복되었을 때 소요된 시간을 비교한 테스트다.  
이펙티브 자바에서는 약 6.5배 차이가 났다고 하는데 내 PC에는 거의 10배 가까이 차이가 났다.  
~~(내 맥북 프로가 좀 늙긴했지..)~~  

Boxing된 객체를 사용할 경우, sum += i; 이 과정에서 Boxing 과정이 발생하여 객체가 생성된다.  
루프가 Integer.MAX_VALUE(2^31 - 1)만큼 발생하므로 성능 손실이 어마어마 할거라는건 예제니까 그냥 넘어갈 수 있는거다.  

Item6에서 말해주고자 하는점은 <strong_blue>굳이 생성하지 않아도 될때에는 객체를 생성하지말아라</strong_blue> 이다.  
Item50에서는 가급적 방어적 복사본을 사용해 객체를 생성하라는 내용과는 약간 상반되긴 하다.  

물론 당연히 Item50의 내용이 훨씬 더 중요하다.  
왜냐하면 불필요한 객체 생성은 메모리나 성능에서만 이슈가 될 수 있지만, 잘못된 객체의 사용은 시스템 오류가 될 수 있기 때문이다.  

여튼 프로그래밍을 할 때 항상 객체 생성에 대한 고민은 많이 할 필요가 있다고 본다.  
~~재미있다.~~  
