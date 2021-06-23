---
layout: post
title: [EffectiveJava-3 Item26]
subtitle: Raw Type은 사용하지 말라
categories: dev
tags: java
comments: true
---

## Item26. [Generic] Raw Type은 사용하지 말라  

C++에서 Template을 처음 공부했을때 뭔가 복잡하고 지저분하고 어려웠다.  
~~아마 책 보다가 중간에 덮었던 듯 하다.~~  
그러다 STL을 공부하면서 자연스럽게 '아, 템플릿이라는게 여러 타입을 쓰더라도 코드를 재사용할 수 있도록 만들어주는 구나' 라고 대충? 이해하고 넘어갔었던 것 같다.  
지금은 Java라는 언어를 사용해 개발하고 있는데, Java에서는 C++의 Template과 유사하지만 다른 Generic이라는 것이 존재한다.  

이펙티브 자바 5장에서는 Generic을 제대로 쓰는 방법, 잘못된 구현 방법 등을 가이드해주고 있다.  

__그중 Item26에서는 Raw Type을 사용하지 말라고 강조한다.__  

현재 실무에서 내가 맡고 있는 프로젝트는 굉장히 레거시 코드가 많다.  
(아마 최초 개발된게 2000년대 후반이니까.. 지금이랑은 많이 다른게 어쩌면 일반적일 수도 있겠다.)  

암튼 기존 코드를 보면 대부분 Generic을 Raw Type으로 사용했다.  
이것때문에 난 코드를 볼 때 마다 항상 타입을 찾기 위해 불필요한 검색하는 일이 발생했고,  
새로 작성한 코드에 타입을 지정했다가 컴파일 오류가 발생하는 불상사도 생겼었다.  

2019년에 '이걸 대체 왜 이렇게 한거야! 내가 아직 Java를 시작한지 얼마안되서 너무 잘 모르는건가?'  
라고 생각 많이 했었는데 역시나 이런 패턴은 좋지 않았다.  

오늘은 Item26 Raw Type관련한 내용을 정리해보려고 한다.

---  

## Generic

Class, Interface 선언부에 아래와 같이 타입 파라미터가 정의되어 있는경우를 Java에서는  
각각 Generic Class, Generic Interface라고 말하며, 합쳐서 Generic Type 이라고 말한다.  

```java
// ArrayList의 원소 타입으로 E라는 Generic이 사용되었다.
public class ArrayList<E> extends AbstractList<E> {
    // ... 코드 생략
}

public interface Comparable<T> {
    // ... 코드 생략
}
```

Generic Type은 매개변수화 타입을 정의한다.  
__다시말해 List<String> 이란 String을 원소로 갖는 List를 말하며, 이를 매개변수화 타입이라고 부른다.__  
(Item26 끝부분에 Generic 용어에 대해 나오는데 뭔가 익숙치 않아서 잘 안들어온다.. 하지만 계속 봐봐야지)  

이에반해 Raw Type도 존재한다.  
예를들어 Java코드에서 List numbers;  이런식으로 객체를 선언할 수 있다.  
Generic Type이 지워진 형태로, 어떤 원소도 추가할 수 있고 Generic이 나오기 전 (Java1.5 이전 버전) 코드들과 호환이 된다.  

코드들과 호환이 된다는 말은 컴파일에서 오류가 발생하지 않음을 말한다.  
하지만 컴파일에서 오류가 발생하지 않는다고 런타임에서도 오류가 발생하지 않는다고 보장해주진 않는다.  
__즉 다시말해 런타임 오류가 발생할 수 있음을 말하며, 이는 굉장히 까다로운 버그가 발생할 수 있고 개발자가 신경써야 할 부분이 많음을 의미한다.__  

---  

## Generic Raw Type 문제 1

Generic을 Raw Type으로 사용하면 컴파일 오류는 피할 수 있지만, 아래와 같이 런타임 오류가 발생할 수 있다.  

```java
public class UnsafeRawType {
 public static void main(String[] args) {
  List<String> strings = new ArrayList<>();

  unsafeAdd(strings, Integer.valueOf(100));  // runtime error!
  String item = strings.get(0);
  System.out.println(item);
 }

 // strings가 매개변수로 Raw Type인 List를 전달받았다.
 private static void unsafeAdd(List strings, Object object) {
  strings.add(object);
 }
}
```

main함수에서 생성된 strings는 String Type의 원소를 갖는 List이다.  
unsafeAdd 메서드는 Raw Type의 List를 매개변수로 받은 뒤, 2번째 매개변수 Object Type의 객체를 Raw Type의 List에 추가한다.  
Raw Type은 어떤 타입의 원소도 추가할 수 있으므로 unsafeAdd 메서드는 정상적으로 컴파일이 진행된다.  
unsafeAdd 메서드 호출 이후 main함수에서는 strings List의 첫 번째 원소를 꺼내려 한다.  
하지만 unsafeAdd로 넘긴 객체는 String Type이 아닌 Integer Type이였기 때문에, strings의 첫 번째 원소는 Integer Type이다.  
결국 런타임에서 Class-Casting 오류가 발생하게 된다.  
![Alt](/assets/img/dev/effective-java/item26-unsafe-add-raw-type-runtime-error.png)

__S/W 세상에서 가장 좋은 오류는 컴파일 오류다.__  
운영중에 갑자기 발생한 오류로 인해 애플리케이션이 뻗는다는 소식은 그 어떤 개발자에게도 두려울 것이다.  

만약 운영중에 저런 코드로 인해 오류가 발생했다면, strings에 추가된 원소들이 어디에서 추가되었는지 코드를 뒤져야 할 것이다.  

사실 위와같이 코드를 작성하면 IDE에서는 분명히 경고를 내려준다.  
하지만 많은 개발자들이 오류는 해결하지만 경고는 조금 무시하는 듯 하다.  
~~(이렇게 썼지만 사실 나도 한때 경고메시지 정도는 무시하거나 Annotation으로 없애기도 했었다.)~~  

Raw Type의 Generic을 사용할 경우 아래와 같은 경고 메시지가 발생한다.  
![Alt](/assets/img/dev/effective-java/item26-unsafe-add-raw-type.png)  

Raw Type List에 Type이 확인되지 않은 원소가 추가된다는 경고 메시지다.  
이 메시지를 무시했기때문에 그 결과 위의 런타임 오류가 발생했으니 이제 타입을 지정하여 컴파일 오류를 내뱉게끔 코드를 아래와 같이 수정해준다.  
메서드의 첫 번재 파라미터를 ```List<Object>``` 혹은 ```List<String>```와 같이 명시적으로 타입을 지정해주면 되는데, 이 때 타입을 뭘로 정하느냐에 따라 컴파일 오류 위치는 변경된다.  
만약 ```List<Object>```로 받는다면 메서드 내에서 원소를 추가할 때는 문제가 발생하지 않을 것이다.  
하지만 메서드 인자로 넘길 때 (unsafeAdd 메서드를 호출할 때) ```List<String>```이 ```List<Object>```로 Casting될 수 없으므로 컴파일 오류가 발생한다.  
만약 ```List<String>```으로 받는다면 unsafeAdd 메서드를 호출할 때는 ```List<String>``` 과 ```List<String>``` 타입이 동일하므로 오류가 발생하지 않을 것이다.  

하지만 ```List<String>``` 타입에 Object를 추가하려고 시도하므로 메서드 내에서 오류가 발생하게된다.  
__이는 한정적 제한과 관련된 부분인데, 지정된 타입보다 상위 타입은 원소로 추가하는게 불가능하다.__  

그리고 지정된 타입보다 하위 타입으로 캐스팅하여 가져오려는 것 또한 불가능하다.  
~~타입 한정에 대해서는 별도로 Generic 한정/비한정에 대해 정리할 예정이다.~~  

어찌됬든 아래와 같이 타입을 지정하여 컴파일 오류를 일으키도록 구현해보았다.  
![Alt](/assets/img/dev/effective-java/item26-safe-add-compile-error.png)

간단한 코드지만 Raw Type의 Generic은 런타임 오류를 유발한다는 것을 알게 되었고, 항상 타입을 명시적으로 정의해야함을 배웠다.  
그렇다면 ```List```와 ```List<Object>```의 차이는 무엇일까?  
둘 다 모든 타입을 받을 수 있는 List가 아닌가?  

이펙티브 자바에서는 아래와 같이 차이점을 정의해주었다.  
Raw type인 List는 모든 Type의 List를 받을 수 있다는 의미이고,  
매개변수와 티입 ```List<Object>```는 모든 Type의 원소를 받을 수 있는 List를 의미한다.  

즉 다시 말해 List는 모든 타입의 List를 받을 수 있으므로, 인자로 ```List<String>```, ```List<Object>```, ```List<Integer>```와 같이 Type이 지정된 List를 넘길 때, 호환되므로 컴파일 오류가 발생하지 않는다.  

하지만 ```List<Object>``` 타입에 인자로 ```List<String>```이나 ```List<Integer>```를 전달하면 타입이 일치하지 않아 컴파일 오류가 발생한다.  

이렇게 되는 이유는 ```List<T>```는 List의 하위 타입이지만, ```List<T>```와 ```List<F>```는 원소의 타입이 다른 서로 다른 List일 뿐이다.  
__즉 원소의 관계가 상위/하위 관계가 있더라도 컨테이너 입장에서는 전혀 관계가 없다는 의미이다.__  

Raw Type을 사용할 경우 아래와 같은 경우에서도 문제가 발생할 수 있다.  

---  

## Generic Raw Type 문제 2  

메서드 파리미터로 2개의 Raw Type의 Collection을 받고 다른 Collection의 원소를 추가하려는 경우, 오류가 발생한다.  

```java
public class UnboundedWildCardType {
    public static void main(String[] args) {
        Set<Integer> numbers = new HashSet<>();
        for (int i = 1; i <= 10; ++i) {
            if (i % 2 == 0)  {
                numbers.add(i);  // 2, 4, 6, 8, 10
            }
        }

        Set<String> numbers3 = new HashSet<>();
        for (int i = 1; i <= 10; ++i) {
            if (i % 4 == 0) {
                numbers3.add(String.valueOf(i)); // "4", "8"
            }
        }

        System.out.println("number3: " + getDuplicatedNumberCount(numbers, numbers3));
        // number3의 제네릭 타입은 String인데 getDuplicatedNumberCount에서 Integer 타입이 원소에 추가되었으므로
        // Runtime 중 ClassCastException이 발생한다.
        for (String number : numbers3) {
            System.out.println("item: " + number);
        }
    }

    private static int getDuplicatedNumberCount(Set numbers, Set numbers2) {
        int count = 0;
        for  (Object o1 : numbers) {
            if (numbers2.contains(o1)) {
                ++count;
            } else {
                numbers2.add(o1);
            }
        }
        return count;
    }
}
```

getDuplicatedNumberCount 메서드의 첫 번째 파라미터는 Integer Type의 Set이며, 두 번째 파라미터는 String Type의 Set이다.  
이들을 Raw Type으로 받으므로 컴파일 오류는 발생하지 않았다.  
하지만 분명 else절에서 String Type의 Set에 Integer 원소를 추가하고 있다.  
결국 메서드 호출 이후 String Type의 Set numbers3를 순회하다가 Type Casting 오류가 발생하고 만다.  
메서드 내에서 for문도 Object 타입의 원소로 순회하므로 실제 원소가 언제 추가되었는지 스택을 따라가 추적해야하는 번거로움이 따라오게된다.  

이러한 경우도 Raw Type의 Generic을 사용하여 발생한 문제이므로, 반드시 타입을 명시적으로 지정해주어야 한다.  
아래는 타입을 명시적으로 지정하여 컴파일 오류를 유도한 코드이다.  
Generic Type으로 와일드 카드를 사용하여, 어떤 Type도 허용하며 타입 불변성이 유지되도록 구현하였다.  

<strong_red>단 와일드 카드로 전달받은 Collection은 null 이외에 어떤 타입의 원소도 추가하는것이 불가능하다.</strong_red>  

```java
 // ... 메서드 외 코드 생략
private static int getDuplicatedNumberCountWithWildcard(Set<?> numbers, Set<?> numbers2) {
    int count = 0;
    for  (Object o1 : numbers) {
        if (numbers2.contains(o1) {
            ++count;
        } else {
            numbers2.add(o1); // type error
        }
    }
    return count;
}
```

위와 같이 비한정적 와일드 카드를 타입으로 사용하여 런타임 오류를 미연에 방지할 수 있다.  

---

Item26의 내용은 어려운 내용은 아니었는데 막상 정리해보고 코드도 작성해보니까 2시간이나 걸려버렸다.  
~~(지금시간 01시 25분.. 으악 내 수면시간)~~  

이펙티브 자바 Item26 에서 예외적으로 Raw Type을 쓰는 경우도 가이드 해주고 있는데, 이 부분은 크게 중요한 것 같지 않아 정리는 따로 안하려고 한다.  

난 가급적 객체 선언부에는 타입을 명시적으로 지정해주는게 좋다고 생각하는데 좀 더 경험많은 개발자 분들은 어떻게 생각하시는지 궁금하다.  
아직 가야 할 길이 먼데 최대한 빨리 좋은 코드를 작성할 수 있는 개발자가 되고 싶다.  
