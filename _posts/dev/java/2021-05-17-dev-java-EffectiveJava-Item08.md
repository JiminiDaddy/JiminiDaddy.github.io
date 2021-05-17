---
layout: post
title: [EffectiveJava3 Item08]
subtitle: finalizer와 cleaner 사용을 피하라
categories: dev
tags: java
comments: true
---

# Item8. finalizer와 cleaner 사용을 피하라  

finalizer, cleaner는 개발하면서 사용해 본적이 없는 클래스다.  

Java를 공부한지 얼마 안 되었을때, C++에서는 굉장히 중요한 역할을 하는 소멸자의 부재에 대해 궁금한적이 있었다.  
당연히 Heap에 생성한 인스턴스라던가, 파일이나 소켓과 같이 사용 후 리소스를 반납해야 할텐데 과연 이게 어디서 이뤄지는가?  
GC가 참조하지 않는 객체들을 찾아 메모리는 회수해준다고 하더라도, 스트림을 닫을 방법이 있는가?  

그때 적절한 대답을 찾지 못했던 것 같다.  
단지 내가 할 수 있던건 저런 실수를 하지 말자정도?  

finalizer가 그런 역할을 위해 탄생한 듯 하다.  

<strong_blue>하지만 finalizer는 JAVA9에서 Deprecated되었으며 이펙티브 자바에서 사용하지 말 것을 강조한다.</strong_blue>  

그 이유는 첫 번째로 finalizer가 수행 될 시점이 JAVA명세에 없다고 한다.  
파일 디스크럽터가 엄청 많이 열린 상태에서 finalize를 통해 닫히는것을 기대한다면 이는 잘못된 판단이라고 한다.  
왜냐하면 GC가 언제 일어날지 모르기때문에 (System.gc()를 수행해도 finalize가 바로 호출되지 않음)  
메모리가 최대치를 넘어가거나 유효한 리소스가 최대치를 넘어가면 시스템 오류료 번져나갈 수 있기 때문이다.  

따라서 절대로, 즉시 실행되거나 의도한 타이밍에 실행되어야 할 작업은 finalizer로 실행해서는 안된다.  

__finalizer의 단점을 보완해주기위해 cleaner가 등장하였지만 여전히 예측할 수 없고, 느리다고 한다.__  

이펙티브 자바에서는 finalizer, cleaner는 GC의 효율을 떨어트린다고 되어있는데 왜 떨어지는지에 대해선 찾아봐야 할 것 같다.  
~~(사실 쓰지말라고 하니까 그렇게 찾고 싶진 않다. 그냥 이건 안써야겠다.. 생각이 드는건 왜일까?)~~  

결국 finalizer, cleaner를 사용하지 않고 안전하고 빠르고 깔끔하게 자원을 반납하는 방법이 무엇인가?  
<strong_red>AutoClosable 인터페이스를 구현해주고, 이를 사용하는 클라이언트에서는 close() 메서드를 호출하면 된다.</strong_red>  

참고로 각 인스턴스는 close()로 인해 자신이 닫혀있는지를 알 필요가 있다.  
(2번 이상 중복 호출 될 수도 있으므로)  

close()가 호출되었을 때 인스턴스 변수로 이를 기록하고, 중복 호출 될 경우 예외를 발생시키는게 가장 아름다운 구현인 듯 하다.  

아래는 Deprecated finalize() 메서드의 코드다. 
```java
// Object class ...
    @Deprecated(since="9")
    protected void finalize() throws Throwable { }
```  

학습을 위해 이펙티브 자바에서 제공하는 예제를 작성해보았다.  
```java

public class Room implements AutoCloseable{
    private static final Cleaner cleaner = Cleaner.create();

    private final State state;

    private final Cleaner.Cleanable cleanable;

    public Room(int numJunkPiles) {
        state = new State(numJunkPiles);
        cleanable = cleaner.register(this, state);
    }

    @Override
    public void finalize() throws Throwable {
        super.finalize();
    }

    public boolean isCleaned() {
       return state.isCleaned;
    }

    @Override
    public void close() {
        System.out.println("Room.close");
        cleanable.clean();
    }

    private class State implements Runnable {
        int numJunkPiles;       // garbages counts

        private boolean isCleaned = false;

        State(int numJunkPiles) {
            this.numJunkPiles = numJunkPiles;
        }

        @Override
        public void run() {
            System.out.println("State.run, 방 청소");
            isCleaned = true;
        }
    }
```  

Room은 State를 갖고있으며, Room이 종료되면 State를 호출해 방에 있는 쓰레기를 수거하는 코드다.  
AutoClosable 인터페이스를 구현하였으므로 외부로부터 정상적으로 Room의 close()가 호출되었다면 방이 청소될것이고,  
호출되지 않는다면 방은 여전히 쓰레기로 가득차있을 것이다.  

아래와 같이 테스트 코드를 작성했다.  
```java
public class RoomTest {
    // 좋은 예. 가급적 자원을 사용하는 클라이언트는 아래와 같이 사용하는것이 좋다.
    @Test
    @DisplayName("try-with-ressource사용 후 close가 제대로 되는지 확인")
    void checkCloseWithTryCatchResources() throws Exception {
        try (Room room = new Room(100)) {
            System.out.println("RoomTest.checkCloseWithCatchResources");
        }
    }

    // 나쁜 예. 객체 참조가 끊겨도 cleaner가 동작한다고 보장할 수 없다.
    @Test
    @DisplayName("try-resource없이 close가 제대로 되는지 확인")
    void checkCloseNotTryCatchResources() throws Exception {
        Room room = new Room(100);
        System.out.println("RoomTest.checkCloseNotTryCatchResources");
    }
}
```  

첫번째 테스트는 Item9에서 나올 try-with-resources를 사용해서 구현했다.  
<strong_red>try-with-resources는 AutoClosable을 구현한 객체에 한해 종료될 때 close()를 호출해준다.</strong_red>  
따라서 첫번째 테스트는 "State.run, 방 청소" 라는 메시지가 출력된다.  
![Alt](/assets/img/dev/effective-java/item08-test-close-success.png)  

두번째 테스트는 room을 생성한 뒤 별도의 close를 호출하지 않았다.  
에상과 동일하게 방 청소는 되지 않았다.  
close()가 호출되지 않았고, 자원은 정상적으로 반납되지 않는다.  
![Alt](/assets/img/dev/effective-java/item08-test-close-fail.png)  


AutoClosable이 자원 정리를 담당한다면 finalizer, cleaner 이것들의 역할은 도대체 무엇일까?  

<strong_purple>첫 번째로는 "안전망" 역할이라고 한다.</strong_purple>  

위의 AutoClosable 인터페이스의 구현은, 반드시 close() 메서드를 호출해주어야 정상적으로 동작한다.  
모든것이 완벽하지만 클라이언트가 이를 누락한다면?  
결국 아무 소용없어지게된다.  

finalizer, cleaner는 이들에 대한 최후의 방법? 정도로 쓰면 될 것 같다.  
언제 호출될지는 모르지만 아예 호출 안되는것보다야 낫지 않은가?  

<strong_purple>두 번째로는 Native Peer와 연결된 객체에서 활용할 수 있다고 한다.</strong_purple>  
Native Peer에 대해 이펙티브 자바에서는 아래와 같이 설명해주고 있다.  

> Native Peer란 일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체를 말한다.  

GC는 Java의 많은 메모리 영역중에 Heap 메모리에 저장된 객체를 대상으로 진행한다.  
하지만 Native 객체는 Native 영역에 있으므로 GC가 회수할 수 없다.  
따라서 이런 영역들은 자원 반납을 위해 cleaner, finalizer가 사용될 수 있다.  
<strong_red>물론 Native Peer에서 심각한 자원을 갖고 있지않고, 성능 저하가 크게 일어나지 않는다는 조건에서 사용되어야 할 것이다.</strong_red>  

