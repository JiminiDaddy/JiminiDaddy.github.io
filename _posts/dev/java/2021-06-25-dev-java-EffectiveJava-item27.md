---
layout: post
title: "[EffectiveJava3 Item27]"
subtitle: "비검사 경고를 제거하라"
categories: dev
tags: java
comments: true
---

## Item27. 비검사 경고를 제거하라

이번장은 양도 작고 큰 이펙트를 주진 않는다.  
하지만 개발중에 그냥 넘어갔던 부분들에 대해 지적해주고 적합한 해결책을 제시해준다.  

비검사 경고란 프로그래밍 후 컴파일 했을 때 발생하는 타입체크 경고 메시지다.  

Java는 Generic이 존재하기 때문에 런타임에 다른 타입이 들어올 수 있다.  
컴파일러는 이러한 문제가 발생할 수 있음을 인지하고 컴파일 결과에 경고 메시지를 남긴다.  

예를들어 아래 코드는 비검사 경고 메시지가 발생한다.  

```java
Set<String> stringSet = new HashSet();
```

HashSet에 Generic Raw Type이 사용된건데 아래와 같이 다이아몬드 연산자만 추가해주면 경고는 사리진다.  

```java
Set<String> stringSet = new HashSet<>();
```

별 것 아니지만 컴파일러가 알려준 비검사 경고를 모두 없앤다면 애플리케이션은 런타임에 타입 안정성이 확보되는 장점이 생긴다.  
다시말해 런타임에서 타입 체크로 인한 오류(ClassCastExxception)가 절대로 발생하지 않는다는 말이다.  

구현한 코드에 따라서 분명히 타입 안전하지만 제거하기 힘든 경우도 있을 수 있다.  
그럴때 경고 메시지를 지워질 수 있도록 아래와 같은 Annotation을 지원해준다.  

```java
@SuppressWarnings("unchecked")
```

클래스, 메서드, 지역변수 등등 많은 곳에서 위 Annotation을 선언해주면 컴파일러가 비검사 경고 메시지를 생성하지 않는다.  
하지만 경고 메시지를 숨기는 것일 뿐 타입 안정을 보장해주진 않는다.  
따라서 클래스나 메서드 단위로 비검사 경고 메시지를 숨길 경우 필요한 타입 체크를 놓칦 수 있으니 확실한 부분은 지역변수에 적용하고, 절대로 클래스 단위로는 적용하지않는다.  
가급적 메서드도 아주 짧은 메서드만 적용하는게 좋다.  

이펙티브 자바에서 ArrayList의 toArray 메서드를 예시로 한 코드가 있다.  

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size)
    // Make a new array of a's runtime type, but my contents:
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

위 코드는 런타임에서 타입 안정이 보장되지만 T[]로 형변환하여 반환하는 코드에서 비검사 경고 메시지가 발생한다.  
elementData가 Object[] 인데 copy 후 반환할 버퍼가 T[] 이기 때문이다.  
하지만 Arrays.copyOf가 T 타입으로 반환하기때문에, 입력 a의 Type은 toArray 메서드의 반환 타입과 동일하다.  
따라서 런타임에서 ClassCastException이 발생할 일은 없다.  

이런 경우는 굳이 필요없는 비검사 경고 메시지를 없애는게 좋을 수 있다.  
그럼 아래와 같은 코드로 작성하면 될 것이라 생각할 것이다.  

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size)
    // Make a new array of a's runtime type, but my contents:
        @SuppressWarnings("unchecked")
        return (T[]) Arrays.copyOf(elementData, size, a.getClass());
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

하지만 저 코드는 컴파일 오류가 발생한다.  
왜냐하면 @SuppressWarnings는 선언부에서만 추가가 가능하며 return문에는 사용할 수 없기 때문이다.  
따라서 불필요하게 느껴질 수 있지만 copy 결과를 참조할 객체를 하나 지역변수로 생성한 후 지역변수를 반환해야 한다.  

```java
public <T> T[] toArray(T[] a) {
    if (a.length < size) {
        @SuppressWarnings("unchecked") {
        T[] result =  (T[]) Arrays.copyOf(elementData, size, a.getClass());
        return result;
    }
    System.arraycopy(elementData, 0, a, 0, size);
    if (a.length > size)
        a[size] = null;
    return a;
}
```

toArray 메서드에 @SuppressWarnings("unchecked")를 추가할 수도 있겠지만 지역변수 result에 추가함으로써 필요한 부분만 명확하게 비검사 오류를 피하도록 구현하였다.  

의도적으로 Annotation을 추가하여 비검사 경고를 숨겼으므로, 다른 개발자들에게 내용을 공유하기위해서 이런 경우는 가급적 주석을 남기는게 좋다.  

---

그런데 사실 위 예제에서 비검사 경고를 숨기는게 적절한 방법인지는 날 잘 모르겠다.  
왜냐하면 개발자가 의도적으로 런타임에 오류를 발생시킬 수 있기 때문이다.  
__elements가 Object[]이므로 elements에 추가한 원소의 타입과 toArray의 매개변수 T 타입이 다르면 오류가 발생한다.__  

아래와 같은 악의적인 코드(혹은 실수할 수 있는)는 런타임 예외가 발생한다.  

```java
public static void main(String[] args) {
    Integer[] numbers = new Integer[2];
    numbers[0] = 1000;
    numbers[1] = 2000;

    List<String> list = new ArrayList<>();
    list.add("100");
    list.add("200");
    list.add("300");
    Integer[] result = list.toArray(numbers);
    for (Integer a : result) {
        System.out.println("result: " + a);
    }
}
```

list는 String 타입의 원소를 받을 수 있으므로 3개의 String을 추가하였다.  
<br/>
<strong_red>그런데 toArray를 Integer[] 타입으로 받으려고 시도했기 때문에 toArray 메서드 내부에 작성된 System.arraycopy에서 아래와 같은 예외가 발생하게된다.</strong_red>  

![Alt](/assets/img/dev/effective-java/item27-unchecked-result-error-ArrayStoreException.png)

물론 위의 코드는 약간 억지이긴 하지만 타입 경고 메시지가 안뜬다고 무조건 타입을 보장받을 수 있는가? 에 대해 의문이 들어서 작성해보았다.  

copy하기 전에 아래와 같이 elements(원소들을 저장한 버퍼)와 toArray의 매개변수 타입을 비교하는 로직이 추가되야하지 않을까?  

```java
private <T> T[] toArray(T[] buffer) {
    if (buffer.length < names.length) {
        String orgBufferType = names.getClass().getTypeName();
        String newBufferType = buffer.getClass().getTypeName();
        // 원본과 복사본의 타입이 불일치하는경우 null을 반환했다.
        if (!orgBufferType.equals(newBufferType)) {
            return null;
        }
        @SuppressWarnings("unchecked")
        T[] result = (T[]) Arrays.copyOf(names, names.length, buffer.getClass());
        return result;
    }
    System.arraycopy(names, 0, buffer, 0, names.length);
    if (buffer.length > names.length) {
        buffer[names.length] = null;
    }
    return buffer;
}
```

간단한 예제를 위해 Class.getTypeName을 통해 타입 비교를 하였다. (타입 비교가 목적)  
위와 같이 하면 런타임에서 비검사 경고가 제거되고, 런타임에서 타입 캐스팅 관련 예외가 발생하지 않는다.  

물론 위 예제는 null을 반환하므로 좋은 코드는 아니다. 만약 실무레벨로 코드로 구현한다면 null safe한 값으로 변경해주어야 할 것이다.  

