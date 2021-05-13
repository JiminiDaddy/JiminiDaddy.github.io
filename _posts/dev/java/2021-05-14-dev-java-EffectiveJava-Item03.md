---
layout: post
title: [EffectiveJava3 Item03]
subtitle: private 생성자나 열거 타입으로 싱글턴임을 보증하라
categories: dev
tags: java
comments: true
---

# Item3. private 생성자나 열거 타입으로 싱글턴임을 보증하라  

## 싱글턴 정의  
싱글턴이란?  
애플리케이션 안에서 오직 하나의 인스턴스만 존재하는 클래스  
전역으로 사용이 필요할 때 만들기도 하지만 많은 이유로 인해 안티패턴 취급을 받고 있다.  
~~싱글턴에 대해선 나중에 디자인 패턴 정리하면서 다시 쓰는것으로..~~  

<hr>

## 클래스를 싱글턴으로 만드는 방법  
Effective Java에서는 크게 3가지로 싱글턴을 만드는 방법을 소개한다.  

### 첫번째로는 필드로 싱글턴을 제공하는 방법  
말 그대로다. 싱글턴 객체를 public static final 키워드를 통해 필드로 제공한다.  
단, 생성자는 private 제한을 걸어두어 외부에서 new 키워드를 통해 인스턴스화할 수 없게 구현한다.  
```java
public class SingletonField {
    public static SingletonField INSTANCE = new SingletonField();

    private SingletonField() {
        System.out.println("SingletonField.SingletonField");
    }
}
```  
이렇게 구현하면 외부에서는 SingletonField 객체를 생성할 수 없고 오직 클래스 멤버변수인 INSTANCE를 통해서만 접근이 가능하다.  
SingletonField.INSTANCE는 정적 객체이므로 클래스가 로딩될 때 단 한번만 인스턴스화 되고, 단 하나만 존재하는것이 보장된다.  

### 두번째로는 정적 팩터리 메서드 방식을 통해 제공하는 방법  
필드는 private으로 캡슐화하고 public static 메서드를 구현하여 싱글턴 객체를 반환하는 방식이다.  
위와 동일하게 생성자는 private으로 구현한다.  
```java
public class SingletonFactory {
    private static final SingletonFactory INSTANCE = new SingletonFactory();

    private SingletonFactory() {}

    public static SingletonFactory getInstance() {
        return INSTANCE;
    }
}
```  
외부에서는 SingletonFactory.getInstance() 를 호출하여 싱글턴 객체를 사용할 수 있게된다.  

얼핏보면 두 방식은 완벽한 싱글턴 객체를 제공하는 것을 보이지만 구멍이 있다.  

자바에는 리플렉션이라는것이 존재한다.  
리플렉션을 사용하면 런타임중에 특정 클래스에 접근하여 객체를 생성하고, 객체의 메서드, 필드에 접근할 수 있다.  
뿐만 아니라 private 접근제한자에 접근할수도 있다.  
따라서 아무리 캡슐화를 완벽하게 하더라도 리플렉션을 공격을 피하긴 어렵다.  

__위의 두가지 싱글턴 생성방법은 리플렉션 공격에 의해 private 생성자가 호출될 수 있는데, 이렇게 되면 싱글턴 객체가 아닌 새로운 객체가 인스턴스화된다.__  

실제로 테스트해보면 아래와 같이 생성자가 2번 호출되고, 서로 다른 인스턴스임을 확인할 수 있다.  
아래는 테스트 코드다.  
```java
class SingletonTest {
    @Test
    @DisplayName("리플렉션 공격을 피할 수 있는지 확인")
    void checkCreateSingletonByReflectionAttack() throws IllegalAccessException, InvocationTargetException, InstantiationException {
        SingletonField singletonField1 = SingletonField.INSTANCE;

        Class c1 = singletonField1.getClass();
        Constructor[] constructors = c1.getDeclaredConstructors();
        for (Constructor constructor : constructors) {
            constructor.setAccessible(true);
        }
        // 리플렉션을 통해 인스턴스화 시도
        SingletonField newInstance = (SingletonField) constructors[0].newInstance();

        assertThat(singletonField1).isNotSameAs(newInstance);
    }
}
```  
실행 결과
![Alt](/assets/img/dev/effective-java/item03-singleton-fail.png)  

테스트가 성공한 이유는 isNotSameAs를 호출했기 때문이다. (서로 다른 인스턴스면 성공)

<strong_red>리플렉션 공격을 막기위해서는 private 생성자에서 이미 INSTANCE가 생성된 경우에 또 호출이 될 경우, 예외를 던지는 방법이 있다.</strong_red>  

```java
public class SingletonField {
    public static SingletonField INSTANCE;

    static {
        try {
            INSTANCE = new SingletonField();
        } catch (AccessDeniedException e) {
            e.printStackTrace();
        }
    }

    private SingletonField() throws AccessDeniedException {
        if (INSTANCE != null) {
            // 리플렉션 공격을 시도했을 땐 이미 싱글턴이 인스턴스화 되어있으므로 예외가 발생한다. 
            throw new AccessDeniedException("This is a singleton-object");
        }
        System.out.println("SingletonField.SingletonField");
    }
}
```  
코드가 조금 지저분하긴한데 이렇게하면 리플렉션을 통해 newInstance를 호출할 때 예외가 발생한다.  
따라서 완전한 싱글턴임이 보장된다.  
아래는 위에 있는 테스트코드를 약간 수정하여 정말 예외가 발생하는지 테스트한 코드다.  
```java
class SingletonTest {
    @Test
    @DisplayName("리플렉션 공격을 피할 수 있는지 확인")
    void checkCreateSingletonByReflectionAttack() {
        SingletonField singletonField1 = SingletonField.INSTANCE;
        Class c1 = singletonField1.getClass();
        Constructor[] constructors = c1.getDeclaredConstructors();
        for (Constructor constructor : constructors) {
            constructor.setAccessible(true);
        }

        assertThatThrownBy(() -> {
            // 리플렉션으로 Private 생성자에 접근하면 예외가 발생한다.
            SingletonField newInstance = (SingletonField) constructors[0].newInstance();
        }).isInstanceOf(Exception.class);
    }
}
```  
newInstance 호출하는 지점에서 예외가 발생하기때문에 테스트는 아래와 같이 성공하며 생성자는 1회만 호출되었다.  
![Alt](/assets/img/dev/effective-java/item03-singleton-success.png)  

여기까지 구현하면 리플렉션 공격이 들어와도 싱글턴임이 보장된다.  

하지만 한가지 더 수정해야 할 부분이 있다.  

<strong_red>역직렬화(Binary to Object)가 수행될 때 readResolve 메서드가 호출되는데, 이 메서드를 구현하지 않으면 싱글턴 객체가 반환되지 않고 새로운 인스턴스가 반환된다.</strong_red>  

SingletonFactory 클래스에 Serializable 인터페이스를 구현했다.  
```java
public class SingletonFactory implements Serializable {
    private static final SingletonFactory INSTANCE = new SingletonFactory();

    private SingletonFactory() {
        System.out.println("SingletonFactory.SingletonFactory");
    }

    public static SingletonFactory getInstance() {
        return INSTANCE;
    }

    // for Deserialization
    private Object readResolve() {
        System.out.println("SingletonFactory.readResolve");
        return INSTANCE;
    }

    // for Serialization
    private Object writeReplace() {
        System.out.println("SingletonFactory.writeReplace");
        return INSTANCE;
    }
}
```  

SingletonFactory의 직렬화,역직렬화를 수행할 SingletonSerialization 클래스를 구현했다.  
```java
public class SingletonSerialization {
    // Object to ByteArray
    public byte[] serialize(Object instance) {
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        try (byteArrayOutputStream; ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream)){
           objectOutputStream.writeObject(instance);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return byteArrayOutputStream.toByteArray();
    }

    // ByteArray to Object
    public Object deSerialize(byte[] seriailizedData) {
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(seriailizedData);
        try (byteArrayInputStream; ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream)) {
           return objectInputStream.readObject() ;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }
}
```  

마지막으로 이들을 테스트할 테스트 코드를 구현했다.  
```java
class SingletonSerializationTest {
    @Test
    @DisplayName("직렬화해도 싱글톤이 유지되는지 확인")
    void checkSingletonWhenSerialization() {
        SingletonSerialization singletonSerialization = new SingletonSerialization();
        SingletonFactory factory = SingletonFactory.getInstance();

        byte[] serializedData = singletonSerialization.serialize(factory);
        SingletonFactory deserializedData = (SingletonFactory)singletonSerialization.deSerialize(serializedData);

        Assertions.assertThat(factory).isExactlyInstanceOf(SingletonFactory.class);
        Assertions.assertThat(factory).isSameAs(deserializedData);
    }
}
```  

__싱글턴으로 생성한 객체 singletonSerialization와 이를 직렬화,역직렬화한 결과 deserializedData와 비교한 결과 둘은 같은 인스턴스임이 확인되었다.__  

테스트 결과를 따로 캡쳐하진 않았지만 만약 SingletonFactory에서 readResolve를 구현하지 않는다면 테스트는 실패한다.  
아래는 구현한 뒤 정상적으로 테스트가 통과한 결과다.  
![Alt](/assets/img/dev/effective-java/item03-singleton-serialization-success.png)  

이제 이들은 완벽한 싱글턴 객체를 보장한다.  

하지만 코드가 많이 지저분해지는 단점이 존재하게 된다.  
싱글턴이 유지되면서 코드를 간결하게 작성할 순 없을까?  

### 세번째 싱글턴 구현 방법으로 Enum 타입으로 구현하는 방법이 있다.  
```java
public enum SingletonEnum {
    INSTANCE,
}
```  
저렇게 구현하면 끝난다.  
Enum으로 구현한 객체는 단 하나만 생성이 보장되며, 코드도 간결하고 Serializable 인터페이스 구현없이 직렬화가 가능하다.  
또한 리플렉션 공격도 피할 수 있다.  
그 이유는 리플렉션을 사용할 Constructor 클래스의 newInstance 메서드는 아래와 같이 구현되어 있기 때문이다.  
```java
    public T newInstance(Object ... initargs)
        throws InstantiationException, IllegalAccessException,
               IllegalArgumentException, InvocationTargetException
    {
        if (!override) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, clazz, modifiers);
        }
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
        ConstructorAccessor ca = constructorAccessor;   // read volatile
        if (ca == null) {
            ca = acquireConstructorAccessor();
        }
        @SuppressWarnings("unchecked")
        T inst = (T) ca.newInstance(initargs);
        return inst;
    }
```  
<strong_blue>throw new IllegalArgumentException("Cannot reflectively create enum objects");</strong_blue>  
ENUM Type인경우 여기서 예외가 발생하기때문에 리플렉션 공격이 불가하다.  

__단, Enum Type은 상속이 불가능하기때문에 다른 클래스를 상속받을 수 없다.__  
(인터페이스 구현은 가능하다.)  

싱글턴에 대해서는 이미 예전부터 알고있었고, Linux Application 개발할 때 종종 썼던 패턴이었다.  
대강 알고있던터라 사실 이번장은 가볍게만 보고 넘어가려했는데 책 2장보는데 생각보다 깊은 내용들이 있어 놀랐다.  
코드 작성하고 이렇게 문서화하는데 2시간이 넘는시간이 걸렸지만 대충 훑지않고 직접 만져보길 잘한것 같다.  
~~올해안에 이펙티브 자바 책 한권을 전부다 정리할 수 있을지.. 하고싶긴한데 시간이..~~  
