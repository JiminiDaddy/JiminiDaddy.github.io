---
layout: post
title: [EffectiveJava3 Item50]
subtitle: 적시에 방어적 복사본을 만들라
categories: dev
tags: java
comments: true
---

# Item50. 적시에 방어적 복사본을 만들라  

<hr>

## 가변 객체를 인자로 받아서 복사할 때 문제점  
EffectiveJava 책 뿐만 아니라 많은 개발자분들이 지향하는 방식으로 불변 객체를 사용하는게 좋다고 알고있고 가급적 그렇게 구현하고 있다.  
그런데 분명 난 불변객체를 만들려고 한건데 의도치않게 객체가 가변될 수도 있다는것을 알게되었다.  
Item50에서도 그러한 문제점을 알려주고있다.  
아래와 같이 Period라는 객체는 public interface method를 제공하지 않는다.  
따라서 외부에서 Period 객체에 접근할 수 없도록 불변 객체를 만드려는 의도가 보인다.  
<strong><u>하지만 아쉽게도 Peroid 객체는 불변이 아니다.</u></strong>

```java
public class Period {
    private Date start;
    private Date end;

    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)  {
            throw new IllegalArgumentException(start + "(시작시간)가 " + end + "(종료시간) 보다 늦을 수 없습니다.");
        }
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
}
```

__분명 외부에서 start를 변경할 수 있는 메서드를 제공하지 않는데 왜 Period는 가변 객체가 되는 것일까?__  


그 이유는 생성자에서 인자를 받는 Date 객체는 가변인데, 인스턴스 변수 start와 end가 Date 객체의 얕은 복사를 하기 때문이다.  
(~~얕은 복사라는 말이 맞나? 잠시 용어좀 찾아봐야겠다~~)  


잠시 내용을 추가하면, 객체를 복사하는 방법에는 얕은 복사와 깉은 복사가 있다.  
(~~이건 C공부할때 배우긴했는데 아마 모든 프로그래밍 언어에서 비슷하지않을까? 언어를 많이는 안다뤄봐서 모르겠지만..~~)  
<strong_blue>
깊은 복사는 인자로 넘어온 객체가 갖고있는 값을 통채로 복사하는 방식이고 얕은 복사는 인자로 넘어온 객체의 주소를 복사한다.  
(주소값만 받아온다)  
</strong_blue>
따라서 깊은 복사를 하면 완전히 똑같은 새로운 객체가 생성된다.  
A라는 객체를 깊은 복사한 B 객체가 있다고 하면, B는 A와 완전히 같은 값을 가진 객체가 되고, B의 값을 수정해도 A에는 영향이 없다.  
(A와 B는 참조하는 메모리 주소가 다르다. 같은 값으로 채워진 메모리가 2군데 존재할 뿐)  

하지만 얕읕 복사를 하면 겉모양만? 똑같은 새로운 객체가 생성된다.  
A라는 객체를 얕은 복사한 B 객체가 있다고 하면, B는 A가 참조하는 있는 객체의 주소를 가리키게 된다.  
A와 B는 같은 메모리 주소를 바라보므로 같은 값이 조회되서 마치 완전히 복사한것처럼 보일 수 있다.  
하지만 같은 메모리를 참조하고 있으므로 A가 객체의 값을 수정하면 B의 값도 수정된다.  

~~갑자기 생각나서 적어보긴했는데 이건 다음번에 다시 좀더 깊은 내용으로 작성해보고 오늘은 Item50에 대해 정리해봐야겠다.~~  


위 예제의 첫번째 문제는 생성자에서 얕은 복사를 하고 있다.  
```java
this.start = start;
```  

인스턴스 변수 start(Date)가 인자로 받아온 start(Date)를 참조하고 있는데, 이 Date는 가변 객체이다.  
따라서 외부에서 Period 객체를 생성한 뒤, 인자로 넘겼던 Date객체를 수정한다면?  
외부에 있는 Date 객체 뿐 아니라, Period의 Date 객체들도 값이 변경된다.  
<strong_purple>왜냐하면 그들은 모두 같은 객체를 참조하고 있기 때문이다.</strong_purple>  
(Date 객체는 단 한번만 생성되었다. 단지 이 객체를 참조하고 있는 참조 객체들이 늘어나고 있을 뿐..)  

그렇다면 어떻게 해야 외부에서 변경해도 Period의 객체는 변경되지 않을까?  
<hr>

## 방어적 복사본을 만들어야 한다.  
방어적 복사본이란 말만 보았을 땐 뭔가 쎄보였다.   
(~~어? 내가 뭔가 또 몰랐던 용어네, 방어적 복사본이 무슨말이지?~~)  

그냥 아래와 같이 객체를 새로 생성하고 값만 복사해서 쓰라는 말이다.  
```java
this.start = new Date(start.getTime());
```  

이렇게 하면 외부의 Date 객체와 Period의 Date 객체는 서로 다른 메모리 공간을 참조하기때문에 변경이 일어나지 않는다.  
그럼 이렇게만 하면 이제 Period 객체는 완전히 불변일까?  

### 아니다!!  

아래 프로퍼티를 보면 Date 객체를 그냥 내보내고 있다.  
```java
public Date start() {
    return start;
}
```  

__이러면 외부에서 Period.start() 메서드를 호출할 경우 Period의 Date객체의 참조가 전달될 것이며, 참조한 외부 객체는 또다시 값을 변경할 수 있는 상태가 된다.__  
따라서 이곳에서도 아래와 같이 방어적 복사본을 생성해서 반환해야 한다.  
```java
public Date start() {
    return new Date(start.getTime());
}
```  

이제 Period는 완전히 불변 객체가 되었다.  
물론 다른 private 타입이 아닌 메서드를 추가로 제공할경우는 가변이 될 수 있으므로 메서드를 제공할 때 항상 불변을 유지하도록 유의해야 한다.  


그런데 이와 같이 방어적 복사본을 할경우 얕은 복사에 비해 비용이 많이 들게 된다.  
<strong_red>당연하게도 메모리 주소만 복사한다면 메모리 주소 크기(포인터 크기) 만큼만 복사하면 되지만, 객체를 새로 생성하면 객체의 크기를 전체 복사해야 한다.</strong_red>  
따라서 모든 객체에 대해 방어적 복사본을 수행하면 성능의 이슈가 생길 수 있으므로 트레이드오프를 잘 고려해야 한다.  

예를들어, 외부 모듈이 아닌 같은 패키지 내에서 혹은 신뢰할 수 있는 파트에 대해서는 룰을 정해 인자로 넘긴 객체에 대해 수정하지 않기로 하거나 수정이 필요한 경우는 외부에서(사용하는 곳에서) 깊은 복사를 수행하도록 하는것이 좋을 것 같다.  

<hr>

### 방어적 복사본을 사용하지 않고 clone() 메서드를 사용할 순 없나?  

나도 처음에 Item50 제목을 보고 뭔말이지? clone 쓰라는건가? 생각이 들긴했다.  
하지만 clone() 메서드는 한 가지 문제가 있다.  
clone() 메서드는 Object 클래스의 protected 접근 제한이 걸린 메서드다.  
따라서 Object를 상속받는(모든 클래스에 해당) 클래스는 이 clone() 메서드를 public 타입으로 Overriding 해야 외부에서 쓸 수 있다.  

<strong_red>여기서 중요한 포인트가 상속을 하고, clone() 메서드를 Overriding 한다는 것이다.</strong_red>  

이 말은 clone() 메서드를 구현한 클래스는 final로 정의하지 않는이상 계속해서 하위 클래스가 생길 수 있다는 말이고,  
Period의 생성자에 매개변수로 넘어온 Date 객체는 실제로는 Date가 아닐 수 있다는 말이다.  
만약 내가 Date 클래스를 상속받은 MyDate라는 클래스를 정의하고, clone() 메서드를 Overriding 했다면?  
그리고 MyDate 객체에서 날짜/시간을 이상한 값으로 설정해버리는 메서드를 제공해주거나 내부에서 그렇게 설정해버린다면?  
Period의 start는 Date 객체일거라 안심했겠지만, 실제 인스턴스가 MyDate와 같이 엉뚱한 인스턴스일 가능성을 배제할 수 없다.  

<strong_red>따라서 가변 객체를 매개변수로 전달받을 땐 clone() 메서드를 통해 복사해서는 안되며, 위와 같이 방어적 복사본을 생성해야 한다.</strong_red>  


프로퍼티, Getter 메서드에서는 clone()을 사용해도 된다.  
왜냐하면 Period 객체의 생성자에서 방어적 복사를 통해 완전히 새로운 객체를 생성했으므로, 이 객체는 외부와 메모리를 공유하지 않는다.  

<strong_deepblue>따라서 Getter에서 clone()을 통해 Date 복사본을 넘겨도, 외부에서는 이 객체는 Date 클래스의 인스턴스라는것이 보장된다.</strong_deepblue>  


아래와 같이 예제를 구성해봤는데 clone()을 통해 객체를 전달해도 정상적으로 불변이 유지됨을 확인했다.  
```java
// PetClinic에 인자로 넘길 Class
public class Pet implements Cloneable {
    private String name;

    Pet(String name) {
        this.name = name;
    }

    public void changename(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    @Override
    public Object clone() throws CloneNotSupportedException {
        return super.clone();
    }
}

// Pet을 상속받은 Dog - clone() 검증을 위해 정의함
public class Dog extends Pet implements Cloneable{
    public Dog(String name) {
        super(name);
    }

    @Override
    public Object clone() throws CloneNotSupportedException {
        return this;
    }
}

// 외부에 clone()을 통해 객체를 전달해도 불변이 유지되는지? 
public class PetClinic {
    private Pet pet;

    public PetClinic(Pet pet) {
        // Pet Class가 상속 가능하다면? (final class가 아닌경우)
        // 상속받은 Class에서 clone을 재구현하고 있다면?
        // clone()을 통해 shallow copy를 한다면 PetClinic의 멤버는 Pet 타입인데 실제로 cloning된 객체는 Pet의 하위 타입이 될 수 있다.
        this.pet = new Pet(pet.getName());
    }

    public Pet customer() throws CloneNotSupportedException {
        // 생성자에서 방어적 복사본을 통해 pet 객체가 Pet 타입이라고 확신할 수 있기 때문에 clone()을 반환해도 불변을 유지할 수 있다.
        return (Pet) pet.clone();
    }
}

// 테스트 코드
class PetClinicTest {
    @Test
    @DisplayName("PetClinic의 customer가 제대로 조회되는지 확인")
    void validPetclinicCustomer() throws CloneNotSupportedException {
        Pet dog = new Dog("dog");

        String originalPetName = dog.getName();

        PetClinic petClinic = new PetClinic(dog);
        Pet customer = petClinic.customer();
        customer.changename("new dog");

        String customerPetName = petClinic.customer().getName();

        Assertions.assertThat(customerPetName).isEqualTo(originalPetName);
    }
}
```

실행하면 아래와 같이 테스트코드가 정상적으로 수행된다.  
![Alt](/assets/img/dev/effective-java/item50-petclinic-success.png)

