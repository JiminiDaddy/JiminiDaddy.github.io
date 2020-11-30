---
layout: post
title: 자료구조 1. List
subtitle: (ArrayList vs LinkedList)
categories: dev
tags: java
comments: true
---


### 자료구조 List
Java에서 제공하는 자료구조는 크게 Collection과 Map 인터페이스로 나뉘어진다.
Collection과 Map 인터페이스는 추후 다시 작성하기로하며, 여기에서는 List에 대해서 공부한 내용을 적어본다.

List에는 크게 ArrayList와 LinkedList로 나뉘어진다.

많은 책들과 인터넷 블로그 등을 통해 공부하다보면 두 클래스의 특징, 장단점 등이 잘 설명되어있다.
대표적인 내용으로 아래와 같은 특징들이 있다.
<hr>
#### ArrayList
각 데이터의 인덱스를 가지고 있으므로 검색할 때 빠르다.
중간에 데이터를 추가/삭제할 경우 해당 인덱스의 뒷부분들은 모두 한블럭씩 뒤로 밀리거나 앞으로 당겨지므로 추가/삭제될 인덱스의 뒷부분의 양이 많다면 성능이 크게 저하된다.

<hr>
#### LinkedList
내부적으로 양방향 연결 리스트로 이루어져 있기 때문에 참조하려는 요소의 위치에 따라 순방향으로 순회할지 역방향으로 순회할지 정해진다.
검색할 때 Head or Tail부터 하나씩 차례대로 노드를 찾는 방식이므로 검색에서 성능이 저하된다.
데이터를 추가/삭제 할 경우 마지막 노드가 가리키는 주소값만 변경하면되므로 ArrayList에서 요소들이 밀리거나 당겨지는 현상은 없다.
<strong_red>하지만 중간에 데이터를 추가/삭제할 경우 중간에 있는 노드를 순회하는 과정이 필요하므로 노드의 인덱스에 따라 성능이 좌우된다. </strong_red>

<hr>
이전에는 단순히 검색은 ArrayList가 빠르고 추가/삭제는 LinkedList가 빠르다고만 생각했는데 정말 과연그럴까? 란 생각에
테스트를 진행해보았다.

```java
public class ListExample {
    public static void main(String[] args) {
        List<String> arrayList  = new ArrayList<>();
        List<String> linkedList = new LinkedList<>();
        System.out.println("Add. Size:500000, StartIndex:0");
        add(arrayList, 100000 * 5);
        add(linkedList, 100000 * 5);

        System.out.println("Add. Size:100000, StartIndex:100");
        add(arrayList, 100000, 100);
        add(linkedList, 100000, 100);

        System.out.println("Add. Size:100000, StartIndex:1000");
        add(arrayList, 100000, 1000);
        add(linkedList, 100000, 1000);

        System.out.println("Add. Size:100000, StartIndex:10000");
        add(arrayList, 100000, 10000);
        add(linkedList, 100000, 10000);

        System.out.println("Add. Size:100000, StartIndex:100000");
        add(arrayList, 100000, 100000);
        add(linkedList, 100000, 100000);

        System.out.println("Get. Size:1000, StartIndex:0");
        get(arrayList, 10000, 0);
        get(linkedList, 10000, 0);

        System.out.println("Get. Size:1000, StartIndex:10000");
        get(arrayList, 10000, 10000);
        get(linkedList, 10000, 10000);

        System.out.println("Remove. Size:100000, StartIndex:0");
        remove(arrayList, 100000, 0);
        remove(linkedList, 100000, 0);

        System.out.println("Remove. Size:100000, StartIndex:100");
        remove(arrayList, 100000, 100);
        remove(linkedList, 100000, 100);

        System.out.println("Remove. Size:100000, StartIndex:Size - 100000");
        remove(arrayList, 100000, arrayList.size() - 100000);
        remove(linkedList, 100000, linkedList.size() - 100000);
    }

    private static void add(List<String> list, int loopCount) {
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < loopCount; ++i) {
            String a = String.valueOf(System.currentTimeMillis());
            list.add(a);
        }
        System.out.println("Wasted add time<" + (list.getClass().getSimpleName()) + ">: " + (System.currentTimeMillis() - startTime));
    }

    private static void add(List<String> list, int addCount, int addIndex) {
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < addCount; ++i) {
            String a = String.valueOf(System.currentTimeMillis());
            list.add(addIndex, a);
        }
        System.out.println("Wasted add2 time<" + (list.getClass().getSimpleName()) + ">: " + (System.currentTimeMillis() - startTime));
    }

    private static void get(List<String> list, int loopCount, int startIndex) {
        long startTime = System.currentTimeMillis();
        for (int i = startIndex; i < loopCount; ++i) {
            String item = list.get(i);
        }
        System.out.println("Wasted get time<" + (list.getClass().getSimpleName()) + ">: " + (System.currentTimeMillis() - startTime));
    }

    private static void remove(List<String> list, int removeCount, int removeIndex) {
        long startTime = System.currentTimeMillis();
        for (int i = 0; i < removeCount; ++i) {
           list.remove(removeIndex);
        }
        System.out.println("Wasted remove time<" + (list.getClass().getSimpleName()) + ">: " + (System.currentTimeMillis() - startTime));
    }
}
```
<hr>
결과는 아래와 같이 나왔다.
get의 경우 Loop가 일정 수 이상으로 증가하면 LinkedList의 성능이 너무 떨어져 최대한 작게 테스트했다.

```java
Add. Size:500000, StartIndex:0
Wasted add time<ArrayList>: 75
Wasted add time<LinkedList>: 99

Add. Size:100000, StartIndex:100
Wasted add2 time<ArrayList>: 9401
Wasted add2 time<LinkedList>: 28

Add. Size:100000, StartIndex:1000
Wasted add2 time<ArrayList>: 10872
Wasted add2 time<LinkedList>: 253

Add. Size:100000, StartIndex:10000
Wasted add2 time<ArrayList>: 13568
Wasted add2 time<LinkedList>: 7523

Add. Size:100000, StartIndex:100000
Wasted add2 time<ArrayList>: 13551
Wasted add2 time<LinkedList>: 68694

Get. Size:10000, StartIndex:0
Wasted get time<ArrayList>: 1
Wasted get time<LinkedList>: 371

Get. Size:10000, StartIndex:10000
Wasted get time<ArrayList>: 0
Wasted get time<LinkedList>: 0

Remove. Size:100000, StartIndex:0
Wasted remove time<ArrayList>: 15272
Wasted remove time<LinkedList>: 3

Remove. Size:100000, StartIndex:100
Wasted remove time<ArrayList>: 13378
Wasted remove time<LinkedList>: 15

Remove. Size:100000, StartIndex:Size - 100000
Wasted remove time<ArrayList>: 742
Wasted remove time<LinkedList>: 57556
```

<hr>
테스트 결과를 요약하면 다음과 같다.

<u>데이터를 처음부터 순차적으로 추가할 경우, ArrayList와 LinkedList의 성능 차이는 거의 없었다.</u>
50만 ~ 500만 선에서 테스트를 진행했는데 대부분 ArrayList가 약간 빠르게 나왔지만 LinkedList가 빠른 경우도 간혹 있었다.

<u>데이터를 중간에 추가할경우, 중간의 위치에 따라 LinkedList의 성능이 많이 달라졌다.</u>
index가 0에 가까울수록 LinkedList가 ArrayList보다 월등히 빨랐지만 index가 커질수록 성능도 급격히 감소했다.
위 결과와 같이 index를 100에서 1000으로 변경했을때 LinkedList의 성능은 약 10배 저하되었다.

<u>데이터를 처음부터 순차적으로 삭제할 경우, LinkedList가 ArrayList에 비해 월등히 빨랐다.</u>
이 역시 index=0이므로 노드를 찾는 시간이 없기때문에 LinkedList는 빨랐고, ArrayList는 index 뒷 부분들을 모두 당겨와야하므로
성능이 저하되었다고 생각된다.

<u>데이터를 거의 끝부분에 삭제할경우는 ArrayList가 빨랐다.</u>
LinkedList가 노드를 찾는데 오래걸리는데 비해 ArrayList는 당겨올 데이터들의 수가 줄었기 때문에 나온 결과라고 예상된다.

<hr>
LinkedList의 get 메서드 구현부는 아래와 같다.
List의 사이즈 대비 index의 위치를 비교해 순방향/역뱡향 탐색을 결정하게 된다.
사이즈만큼 찾아가므로 시간복잡도는 O(N)이 될 것이다.

```java
public E get(int index) {
        checkElementIndex(index);
        return node(index).item;
    }
...
 Node<E> node(int index) {
        // assert isElementIndex(index);
        if (index < (size >> 1)) {
            Node<E> x = first;
            for (int i = 0; i < index; i++)
                x = x.next;
            return x;
        } else {
            Node<E> x = last;
            for (int i = size - 1; i > index; i--)
                x = x.prev;
            return x;
        }
    }    
```
<hr>
다음은 ArrayList의 get 메서드 구현부의 코드다.
index로 요소를 바로 찾아가므로 시간복잡도가 O(1)로 되어 LinkedList보다 빠를 것이라는게 확인되었다.
```java
 public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

E elementData(int index) {
        return (E) elementData[index];
    }    
```

<hr>
JAVA 1.6이후에는 ArrayDeque이 추가되었고, 이것은 LinkedList의 성능이 개선되었다고 나와있다.
아직 그 부분까지는 테스트 하진 않았지만 추후 해봐야겠다.
<u>참고로 ArrayDeque은 null을 추가할 수 없다. </u>
예전에 BFS문제를 풀 때 ArrayDeque을 사용해본적이 있는데 계속 오류가 나서 애먹은적이 있었는데 원인이 null이었다...