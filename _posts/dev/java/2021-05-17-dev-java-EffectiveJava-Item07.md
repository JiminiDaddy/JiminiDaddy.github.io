---
layout: post
title: [EffectiveJava3 Item07]
subtitle: 다 쓴 객체 참조를 해제하라
categories: dev
tags: java
comments: true
---

# Item7. 다 쓴 객체 참조를 해제하라  

나는 2013년부터 2017년까지 C++로 개발을 하다가 우연히 맡게된 Java Project를 처음 진행하면서 Java언어를 쓰기 시작했다.  
문법도 약간 차이가 있지만 당시 가장 의심스러우면서? 신기했던부분은 GC였다.  
<strong_black>C/C++에서는 개발자가 동적 할당한 메모리는 반드시 해제해야 한다.</strong_black>  
(malloc - free 또는 new - delete)  
버퍼로 생성한 객체에 대해서는 delete[]; 와 같이 해제해주어야 제대로 메모리가 해제됬었는데 개발하고나서 메모리 해제부분이 누락되었는지 체크하는 일도 참 귀찮긴 했었다.  
또한 항상 null로 초기화해주는 작업도 여간 귀찮은 일이 아니었다.  

__C++11이후 스마트포인터가 이 부분을 해결해주긴 했으나 잘못 사용하면 여전히 메모리 누수가 발생되긴 했다.__  
~~QT에서도 GC와 같은 기능이 있어 new로 할당한 부분을 알아서 처리해주긴했다. 하지만 2년차의 난 그 부분을 제대로 이해하지 못했고 강제로 메모리 해지했었던 것 같다.~~  

Java에서는 참조가 끊긴 객체들은 GC대상이 되며 개발자가 메모리를 해제하지않아도 GC에 의해 알아서 회수된다.  

그럼 Java에서는 다 쓴 객체에 대해 null로 초기화하는건 불필요한 일인가?  

Java를 사용한지 몇달 되지 않았을 때 한 선배 개발자로부터 코드 가이드를 받은적이 있었다.  
~~(왜냐하면 당시 난 Java 신입 개발자 같은 5년된 개발자였으니까..ㅠㅠ)~~  

C++에서 했던것과 유사하게 생성자에서 객체를 null 초기화하고, 사용끝났을 때 null로 다시 초기화해주는 코드를 작성한 적이 있었다.  
선배 분께서는 이것을 지적해주셨고, Java에서는 null초기화같은건 안하는게 좋다고 말씀해주셨다.  
그땐 그냥 '아~ 그런가보다' 하고 넘어가고 초기화 부분에 대해서만 살짝 찾아봤는데, Java는 객체가 생성되면 기본으로 0 혹은 null 초기화가 되었다.  
__예를들어 C++에서는 배열을 생성하면 항상 memset 혹은 {0,}; 과 같이 초기화 작업을 해줘야한다.__  
(Modern C++에서는 어찌되는지 모르겠다. 저런 부분이 자동으로 되고있는지?)  

어찌됐든 가이드 받은대로 그 이후론 생성할 땐 null처리를 거의 안했던 것 같다.  
작업이 끝난 객체에 대해선 null처리 한것도 있고 안한것도 있는데 아마 초반에 많이 고민했던 것 같다.  
~~(정말 안하는게 맞는지? 난 해주는게 맞는거같은데..)~~  

하지만 이펙티브 자바 Item7에서는 사용이 끝난 객체 대해 null로 초기화 해주는 작업을 강조하고 있다.  
뭐 항상 그래야 된다는건 아니다.  

__null로 값을 초기화한다는건, 사용하는 쪽(메서드를 호출하는 등)에서 항상 null 참조에 대한 리스크를 갖게 된다는 문제가 있다.__  
__또한 null safe를 보장한다는 메서드를 구현할경우 문서에 확실하게 기록해두어야 하는 등 부가적인 작업이 항상 필요하다.__  

kotlin이나 최근 java에서도 null로부터 구원해주려는? 모습들은 아마 null이 그만큼 프로그래밍에 골칫덩어리로 잡혀있기 때문인 듯 하다.  

나도 코드를 작성하다보면 정말 하기 싫은게 null처리다.  
```java
if (A != null) {
    // ...
}
```  
깔끔하게 코드를 작성하고 싶어도 저런 코드가 한줄 두줄 생기다보면 개발자로써 눈살 찌푸리게 된다.  

<hr>

### Stack  

아래와 같이 이펙티브 자바에 있는 스택 예제를 구현했다.  
```java
public class MyStack<T> {
    private static final int DEFULT_STACK_SIZE = 16;
    private Object[] elements;
    private int size = 0;

    public MyStack() {
        elements = new Object[DEFULT_STACK_SIZE];
    }

    public void push(T item) {
        ensureCapacity();
        elements[size++] = item;
    }

    public Object pop() {
        if (size == 0) {
            throw new EmptyStackException();
        }
        Object item = elements[--size];
        // pop호출 후 비활성화된 영역은 null처리함으로써 GC에게 알려주어야 한다. 
        // elements[size] = null;
        return item;
    }

    public int size() {
        return size;
    }

    private void ensureCapacity() {
        if (elements.length == size) {
            elements = Arrays.copyOf(elements, 2 * size + 1);
        }
    }
}
```  

테스트 코드도 작성했다.  (주석이 해지된 상태로 테스트함)  
```java
public class MyStackTest {
    @Test
    @DisplayName("스택의 push, pop기능이 잘 동작하는지 확인")
    void checkStackPushAndPop() {
        MyStack<Integer> stack = new MyStack();
        stack.push(100);
        stack.push(200);
        stack.push(300);
        Assertions.assertThat(stack.size()).isEqualTo(3);
        Assertions.assertThat(stack.pop()).isEqualTo(300);
        Assertions.assertThat(stack.pop()).isEqualTo(200);
        Assertions.assertThat(stack.pop()).isEqualTo(100);
        Assertions.assertThatThrownBy(stack::pop).isInstanceOf(EmptyStackException.class);
    }
}
```

__pop() 메서드에서 주석 친 부분이 메모리 누수의 원인이 될 수 있는 부분이다.__  
위 MyStack은 Object[]을 이용하여 스택을 구현했다.  
push()가 호출되면 size가 하나씩 늘어나다가 size가 capacity를 넘어서면 더 큰 메모리를 할당 받는다.  
pop()메서드가 호출되면 size를 하나씩 감소시켜 size를 넘어가는 영역에 대해 접근하지 못하도록 구현되어있다.  
문제는 pop() 메서드의 의도를 GC가 알 길이 없다는 것에 있다.  
pop() 메서드가 size를 감소시켰다고해서 elements가 가변으로 size가 줄어들진 않는다.  
따라서 size뒤에 있는 영역에 참조하고 있는 객체가 존재한다면, 실제로 이 영역은 사용되지 않더라도 메모리가 회수되지 않는다.  
<strong_red>왜냐하면 GC 입장에서는 여전히 elements가 해당 객체를 참조하고 있기 때문이다.</strong_red>  

따라서 위의 주석을 제거해서 더이상 참조되지 않는 영역에 대해서는 null처리가 필요하다.  

<strong_purple>위 MyStack이 메모리 누수가 발생할 수 있는 위험이 이유는, 메모리(elements)를 MyStack이 직접 관리하고 있기 때문이다.</strong_purple>  
(elements는 배열로 구현되어있고 Stack의 영역을 element에 size를 이용해서 MyStack이 직접 제어한다.)  

<hr>

### Cache  

이펙티브 자바에서는 메모리 누수를 일으키는 주범 중 하나가 Cache라고 가이드해준다.  
Cache에 객체의 참조를 넣은 후 따로 관리해주지 않으면 계속해서 참조 객체가 GC 대상에서 제외되기 때문이다.  
Cache에서 유효기간을 설정해줌으로써 참조를 끊을 수 있지만, 유효기간을 얼마로 잡을것인지에 대한 정의가 상황에 따라 어려울 수 있을 듯 하다.  

<hr>

### Listener or Callback  

비동기 작업이 필요할 때 Listener 혹은 Callback을 사용하게 된다.  
어떤 작업을 실행하고, 실행이 끝난 다음 작업이 필요하다면 Callback을 통해 전달받은 후 처리하면 되기 때문이다.  
문제는 필요한 콜백을 모두 등록하고, 필요없는 콜백에 대해 제거하지 않는 경우에 발생한다.  

이펙티브 자바에서는 WeakHashMap을 사용해서 약한 참조로 저장하라고 되어있는데, 난 WeakHashMap을 사용해보지 않았다.  
~~(뭔가 부끄러운..)~~  

WeakHashMap에 대해 코드를 작성해보고 따로 정리해봐야겠다.  


이번 아이템의 가르침은 더이상 참조가 필요없는 객체에 대해 GC 대상이 될 수 있도록 반드시 처리해야한다.  
(null처리가 될 수도 있고, scope를 작게 만들어 자동으로 참조 해지할 수도 있다.)