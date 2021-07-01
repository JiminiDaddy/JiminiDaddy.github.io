---
layout: post
title: "[EffectiveJava3 Item28]"
subtitle: "배열보다는 리스트를 사용하라"
categories: dev
tags: java
comments: true
---

Java로 (다른 언어도 비슷할것이라 생각함) 프로그래밍을 하다보면 많이 사용하는 자료구조들이 있다.  
그 중 대표적인게 배열과 리스트가 빠질 수 없다.  

배열은 자료구조중에서도 가장 사용하기 쉽다.  
그이유는 아마도 Generic이 배열을 지원하지 않기 때문일거라 생각한다.  

---

## 배열과 리스트의 차이  

배열과 리스트(ArrayList)는 유사하지만 일부 차이가 있다.  
<br/>
<strong_red>가장 큰 차이는 배열은 고정된 크기를 갖고 리스트는 가변의 크기를 갖는다.</strong_red>  

~~오늘 작성할 Generic과 관련한 내용은 아니지만..~~
<br/>
<strong_red>두 번째 차이로는 배열은 공변성을 갖고, 리스트는 불공변성을 갖는다.</strong_red>  

__공변성이란 어떤 타입 A와 B가 있고 A가 B의 Super 타입이라면, B[]는 A[]의 하위 타입이 된다.__  
예를들어 Integer[]는 Number[]의 하위 타입이 되고, Number[]는 Object[]의 하위 타입이 된다.  
즉 배열은 타입이 변경되면 내부 원소들도 하위 타입으로 변한다.  

반면 Generic은 불공변성을 갖는다.  
__A가 B의 상위 타입일 때, ```List<A>```와 ```List<B>```는 아무런 관계가 없다.__  
즉 ```List<Integer>```는 ```List<Number>```나 ```List<Object>```의 하위 타입이 아니다.  
내부 원소들은 상위/하위 타입을 갖게될지 모르지만 그것을 담고 있는 Collection(List)들 끼리는 상위/하위 관계를 나타내지 않는다.  

이것은 별거 아닌것처럼 보이지만 사용에 따라 애플리케이션에 예외를 발생시킬 수 있다.  

---

## 배열의 공변성

위에 말한것처럼 배열은 공변성을 갖고있다.  
따라서 다형성을 이용하여 객체의 타입은 상위타입으로 생성하되, 인스턴스(구현체)의 타입은 하위 타입으로 생성이 가능하다.
아래와 같은 코드는 컴파일에 아무런 문제가 없다.  

```java
Object[] objects = new Long[1]; 
```

objects의 타입은 Object이지만, 실제 구현 타입은 Long이다.  
Long은 Object의 하위 타입이므로 위 배열 객체는 정상적으로 생성된다.

하지만 아래와 같은 코드가 추가되면 어떨까?  

```java
Object[] objects = new Long[1]; 
objects[0] = "Long에 String을 쓰면 런타임에 캐스팅 오류난다";
```

objects의 첫 번째 원소가 String 타입의 객체를 참조하도록 작성했다.  
String은 Object의 하위 타입이므로 컴파일에 전혀 문제가 되지 않는다.  

---

## Generic의 불공변성

하지만 Long으로 생성된 객체에 String을 넣고 있어 누가봐도 문제가 될 것처럼 보이는 코드다.  
예상한대로 예외가 발생하는지 테스트 코드를 작성해보았다.  

```java
@Test
@DisplayName("배열은 공변으로 타입캐스팅이 런타임에 오류난다")
void arrayIsCovariant() {
    Object[] objects = new Long[1];
    Assertions.assertThatThrownBy(() -> {
        objects[0] = "Long에 String을 쓰면 런타임에 캐스팅 오류난다";
    }).isInstanceOf(ArrayStoreException.class);
}
```

예외가 발생해야 성공처리되도록 테스트 코드를 작성했는데, 예상대로 위 코드는 성공처리된다.  

### 여기서 문제는 런타임에 예외가 발생한다는 점이다  

먄약 컴파일 오류가 발생했다면 개발자가 컴파일 시점에 코드를 수정했겠지만 위 코드는 컴파일 시점에선 모르고 넘어갈 수 있는 큰 문제가 있다.  

Generic은 불공변성이므로 위와 같은 코드는 컴파일 시점에 오류가 발생한다.  
아래와 같이 테스트 코드를 작성해보았다.  

```java
@Test
@DisplayName("제네릭은 불공변으로 타입캐스팅이 컴파일에 오류난다")
void genericIsIncovariant() {
    List<Object> objects = new ArrayList<Long>();

    List<Long> objects2 = new ArrayList<>();
    objects2.add("Long에 String을 쓰면 컴파일에 오류난다");
}
```

objects는 Object 타입의 원소를 담고 있는 리스트인데 Long 타입의 ArrayList로 구현했다.  
```List<Object>```는 ```List<Long>```의 상위 타입이 아니므로 (아무런 관계가 없음) 컴파일 시점에 오류가 발생한다.  

object2도 비슷하다.  
선언할 때는 구현체의 타입을 생략함으로써 문제가 발생하지 않는다.  
하지만 Long 타입의 원소를 담는 리스트에 String 타입의 원소를 추가하려했으므로 이는 타입 캐스팅 실패로 인해 컴파일 오류가 발생한다.  

즉 Generic을 사용하면 이처럼 개발자의 잘못된 의도나 실수를 컴파일 시점에 방지할 수 있게된다.  

__Generic은 컴파일 시점에 타입 캐스팅을 마치게 되고, 런타임 시점에는 타입이 소거된 Raw Type으로 남게된다.__  
(이 점이 C++의 Template과 가장 큰 차이가 아닐까 싶다.)  

즉 Generic은 타입을 컴파일 시점에 검사하고 런타임 시점에는 뭐가 오든 신경 안쓴다는 얘기가 된다.  
(Generic을 이렇게 만든 이유는 Java1.5에 나온 Generic이 이전 버전과의 하위 호환을 위함이라 한다.)

---

## 배열과 Generic의 혼합 사용 시 발생할 수 있는 문제점

공변성을 갖는 배열과 불공변성을 갖는 Generic을 혼합하여 사용할 경우, 런타임에 예외가 발생할 수 있다.  
아래 코드는 런타임에 ClassCastException이 발생한다.  

```java
@Test
@DisplayName("제네릭 배열 생성은 허용되지 않는다")
void genericArrayIsDeny() {
    // 1. List<String> 타입의 배열 생성
    List<String>[] stringList = new ArrayList[1]; // type unchecked warning
    // 2. Object 타입의 배열 생성하고 (1)에서 생성한 List<String> 타입의 배열을 참조
    Object[] objects = stringList;
    // 3. List<Integer> 타입의 컬렉션 생성
    List<Integer> intList = new ArrayList<>();
    // 4. (3)에서 생성한 List<Integer> 타입의 컬렉션에 원소 추가 
    intList.add(100);
    // 5. (2)에서 생성한 Object 타입의 배열의 첫 번째 원소가 (3)에서 생성한 컬렉션을 참조
    objects[0] = intList;
    // 6. (1)에서 생성한 List<String> 타입 배열의 첫 번째 원소로부터 첫 번째 String 원소를 참조
    Assertions.assertThatThrownBy(() -> {
        String str = stringList[0].get(0); // Integer로 꺼낸 원소를 String으로 Casting하므로 런타임 오류!
    }).isInstanceOf(ClassCastException.class);
}
```

### 위 코드는 6번 과정에서 런타임 예외가 발생한다

코드를 한줄 씩 해석보면 아래와 같다.  

1. 정상적으로 배열이 생성된다.

2. ```List<String>```은 Object의 하위 타입이므로 공변성으로 인해 정상적으로 ```Object[]```가 생성된다.

3. 정상적으로 컬렉션(List)가 생성된다.

4. 정상적으로 원소가 추가된다.

5. ```List<Integer>```는 Object의 하위 타입이므로 공변성으로 인해 ```Object[]```의 첫 번째 원소로 참조가 가능하다.

6. object[0]은 (5)번 과정으로 인해 ```List<String>```에서 ```List<Integer>```로 참조하는 객체가 변경되었다.  
   따라서 (6)번에서 ```stringList[0].get(0)```에는 (4)번 과정에서 추가한 ```Integer``` 타입의 100이란 값이 들어있다.  
   하지만 String 타입으로 받으므로 ```String```과 ```Integer``` 간의 타입 캐스팅 문제로 인해 런타임 예외가 발생한다.  

---

위 예제는 String, Integer와 같은 단순한 타입이라 코드만 읽어도 이상한 점이 보일 수 있다.  
하지만 개발자가 구현한 클래스라면 꼼꼼히 보지않는 이상 실수로 위와같은 일이 벌어질 수 있다.  

### 위 코드의 문제는 배열의 공변성으로 인해 발생하였다

```List<String>```과 ```List<Integer>```는 전혀 관계가 없는데 Object의 하위 타입이라는 이유로 같은 배열의 원소로 참조 되었기 때문이다.  
__만약 런타임에서 위와같은 문제가 발생하지 않으려면 컴파일 타임에 오류를 내뱉도록 유도해야한다.__  
가장 좋은 방법은 배열 대신 컬렉션을 사용하여 컴파일 타임에 타입을 확정짓는것이다.

배열을 컬렉션으로 변경하면 코드가 약간 길어지고 성능도 약간 떨어질 수 있다.  
하지만 약간의 희생을 감수하고 런타임에서 타입 안전을 보장받을 수 있다면 운영적인 측에서 훨씬 유리하다고 생각된다.  
~~(요즘은 하드웨어 기술이 워낙 뛰어나므로 아주 적은 성능을 감수하는게 훨씬 이득이라고 개인적으로 생각한다.)~~  

---

위 코드처럼 배열과 Generic을 함께 쓴 자체가 사실 문제가 된다.  
배열은 컴파일 시점에는 공변성으로 인해 타입이 확정되지 않고, 런타임에 타입이 확정되어 실체화된다.  
예를들어 아래의 코드가 있다.  

```java
Object[] objects = new String[10];
```

위 코드는 String이 Object의 하위 타입이기때문에 공변성으로 인해 정상적으로 컴파일된다.  
그리고 런타임에 objects는 인스턴스 타입으로 실체화되어 Object[] 타입에서 String[] 타입으로 변경된다.  

Generic의 경우는 다르다.  
Generic은 컴파일 타임에 불공변성으로 인해 타입이 맞지 않으면 생성에 실패하여 컴파일 오류가 발생한다.  
하지만 컴파일이 정상적으로 되었다면 런타임에는 타입이 소거되어 Generic이 어떤 타입인지 전혀 알 수 없다.  
아래 코드를 보자.  

```java
List<String> strings = new ArrayList<String>();
List<Integer> numbers = new ArrayList<Integer>();
```

위 코드는 타입이 일치하여 정상적으로 컴파일된다.  
그리고 런타임에는 아래와 같은 코드로 변경된다.  

```java
List strings = new ArrayList();
List numbers = new ArrayList();
```

strings와 numbers가 구별이 되는가?  
런타임에 strings에 추가한 원소를 꺼내어 numbers에 넣는 일이 발생해도 전혀 알 수 없다.  
(물론 컴파일 시점에 오류가 발생시키므로 실제로는 문제가 발생하지 않을 것이다.)  

---

## 결론  

배열은 공변성을 갖고, 런타임에 실체화된다.  
Generic은 불공변성을 갖고, 런타임에 타입이 소거되어 런타임에는 타입을 알 수 없다.  
즉, 배열은 런타임에 타입이 확정되는데 Generic은 런타임에 타입을 알 수 없다.  
Generic을 배열로 만들면 런타임에 배열이 어떤 타입을 갖게되는지 알 수 없게되고 따라서 타입안정성이 보장되지 않는다.  
<br/>
<strong_red>배열보단 Generic을 이용한 컬렉션을 사용하자.</strong_red>  

그러면 타입을 컴파일 시점에 확정지으므로 런타임에 타입안정성이 보장된다.