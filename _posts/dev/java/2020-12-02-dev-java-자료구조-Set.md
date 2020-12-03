---
layout: post
title: 자료구조 2. Set
subtitle: HashSet vs TreeSet
categories: dev
tags: java
comments: true
---

## 자료구조 Set
Collection Interface 중 하나로써, 아래와 같은 특징을 갖는다.  
  

<u>데이터의 중복을 불허한다.</u>
* List는 중복된 객체가 저장될 수 있다.  

<u>인덱스가 제공되지 않는다.</u>
* List는 인덱스를 통해 저장된 객체의 순서를 보장한다.  
* Set은 인덱스가 제공되지 않으므로 get();을 통해 특정 위치의 요소에 접근할 수 없고, iterator();를 통해서만 접근이 가능하다.  


HashSet, TreeSet, LinkedHashSet 등의 구현체들이 있다.  
<hr/>  

### HashSet  
객체를 저장할 때, Hash를 사용하므로 검색속도가 빠르다.  
내부적으로 HashMap을 생성하여 사용한다.  
<strong_red>데이터를 꺼낼 때 순서가 보장되지 않는다.</strong_red>  

HashSet의 내부 구성  
HashMap을 그대로 이용하므로 HashMap의 성능과 동일하다.  
```java
private transient HashMap<E,Object> map;

public HashSet() {
    map = new HashMap<>();
}
...
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
...
public boolean contains(Object o) {
    return map.containsKey(o);
}
...
```
<hr/>  

### TreeSet  
객체를 저장할 때 Tree구조로 정렬을 진행하면서 저장한다.  
내부적으로 TreeMap을 생성하여 사용한다.  
<strong_red>데이터를 꺼낼 때 기본적으로 오름차순 정렬의 순서로 꺼내진다.</strong_red>  

TreeSet의 내부 구성  
TreeMap을 그대로 이용하므로 TreeMap과 성능은 동일하다.  
```java
private transient NavigableMap<E,Object> m;

public TreeSet() {
    this(new TreeMap<E,Object>());
}
...
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
...
public boolean contains(Object o) {
    return m.containsKey(o);
}
```
<hr/>  

### LinkedHashSet  
HashSet을 상속하였으므로 기본적으로 HashSet의 특성을 지닌다.  
<strong_red>데이터를 꺼낼 때 저장한 순서대로 꺼내진다.</strong_red>  

<hr/>  

간단한 성능 테스트를 위해 예제를 작성해보았다.  
```java
public class SetExample {
    private static long[] testTime = new long[2];
    public static void main(String[] args) {
        Set<Long> hashSet = new HashSet<>();
        Set<Long> treeSet = new TreeSet<>();
        Set<Long> linkedHashSet = new LinkedHashSet<>();

        System.out.println("Add. Size: 1,000,000");
        add(hashSet, 10000 * 100);
        add(treeSet, 10000 * 100);
        add(linkedHashSet, 10000 * 100);

        System.out.println("Add. Size: 10,000,000");
        add(hashSet, 10000 * 1000);
        add(treeSet, 10000 * 1000);
        add(linkedHashSet, 10000 * 1000);

        System.out.println("Search. Size: 11,000,000");
        search(hashSet);
        search(treeSet);
        search(linkedHashSet);
    }

    private static void add(Set set, int count) {
        long start = System.currentTimeMillis();
        for (int i = 0; i < count / 2 - 1; ++i) {
            set.add(System.nanoTime());
        }
        testTime[0] = System.nanoTime();
        set.add(testTime[0]);
        for (int i = count / 2; i < count; ++i) {
            set.add(System.nanoTime());
        }
        long end = System.currentTimeMillis();
        testTime[1] = end;
        System.out.println("[Add] <" + set.getClass().getSimpleName() + "> wasted: " + (end - start));
    }

    private static void search(Set set) {
        int c = -1;
        while (++c < testTime.length) {
            long start = System.currentTimeMillis();
            set.contains(testTime[c]);
            long end = System.currentTimeMillis();
            System.out.println("[Get] <" + set.getClass().getSimpleName() + "> c:" + c + ", wasted: " + (end - start));
        }
    }
```
<hr/>  

테스트 결과는 아래와 같다.  
```java
Add. Size: 1,000,000
[Add] <HashSet> wasted: 274
[Add] <TreeSet> wasted: 1760
[Add] <LinkedHashSet> wasted: 194
Add. Size: 10,000,000
[Add] <HashSet> wasted: 6602
[Add] <TreeSet> wasted: 9640
[Add] <LinkedHashSet> wasted: 9385
Search. Size: 11,000,000
[Get] <HashSet> c:0, wasted: 11
[Get] <HashSet> c:1, wasted: 12
[Get] <TreeSet> c:0, wasted: 13
[Get] <TreeSet> c:1, wasted: 13
[Get] <LinkedHashSet> c:0, wasted: 11
[Get] <LinkedHashSet> c:1, wasted: 11
```

<hr/>

### 1. 데이터 저장  
HashSet과 LinkedHashSet은 Hash를 이용해 저장하므로 저장 속도가 매우 빨랐다.  
반면 TreeSet의 경우 정렬이 들어가므로 속도는 약간 느렸다.  
  
### 2. 데이터 검색  
HashSet, LinkedHashSet, TreeSet 모두 객체의 존재여부 검색속도는 매우 빨랐다.  
하지만 iterator();를 통해 데이터를 찾는 작업은 일일이 Node를 하나씩 확인해야하므로 LinkedList와 같이 성능애 매우 낮았다.