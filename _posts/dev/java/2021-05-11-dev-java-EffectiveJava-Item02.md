---
layout: post
title: [EffectiveJava3 Item2]
subtitle: 생성자에 매개변수가 많다면 빌더를 고려하라
categories: dev
tags: java
comments: true
---

# Item2. 생성자에 매개변수가 많다면 빌더를 고려하라  

## 생성자 패턴이나 정적 팩터리 메서드를 통해 객체를 생성할 때 발생할 수 있는 문제점  
객체를 생성할 때 필수적인 매개변수와 선택적인 매개변수로 나뉘어져있고, 매개변수의 개수가 많다면 생성자 타입과 정적 팩터리 타입 모두 매개변수에 의존하게 된다.  
예를들어 생성자는 아래와 같이 점층적 생성자 패턴방식을 사용하게 된다.  
```java
class NutiritionFacts {
    private int servingSize;
    private int servings;
    private int fat;
    private int calrory;
    public NutiritionFacts(int servingSize, int servings) {
        NutiritionFacts(servingSize, servings, 0);
    }
    public NutiritionFacts(int servingSize, int servings, int fat) {
        NutiritionFacts(servingSize, servings, fat, 0);
    }

    public NutiritionFacts(int servingSize, int servings, int fat, int calrory) {
        // ...................
    }

    public static void main(String[] args) {
        NutiritionFacts facts = new NutiritionFacts(10, 10);
        // NutiritionFacts facts = new NutiritionFacts(10, 10, 100);
        // NutiritionFacts facts = new NutiritionFacts(10, 10, 100, 200);
        // ...
        // 도대체 어떤 생성자를 써야하지? 그냥 최소값으로 쓰고 알아서 초기값 받으면되나? 
    }
}
```  
매개변수의 갯수가 많아질수록 불필요한 생성자를 계속해서 정의해야하는 문제도 발생하고  
개발자는 항상 매개변수의 순서에 대해 고려해야하며, 객체를 생성할 때 늘 문서를 확인해야하는 단점이 발생한다.  
즉 유지보수에 어려운 코드가 된다.  


## 자바빈즈 패턴을 통해 위 문제 해결
이러한 문제를 해결하기위해 자바빈즈 패턴이 있는데 이 경우도 역시 단점이 발생한다.  
자바빈즈 패턴은 점층적 생성자를 사용하지 않고, Setter 메서드를 제공하여 클라이언트가 필요한 설정값을 메서드를 통해 호출한다.  
생성자의 코드가 간단해지므로 인스턴스화는 쉬워졌지만 이 역시 단점이 존재한다.  
```java
class NutiritionFacts {
    private int servingSize;
    private int servings;
    private int fat;
    private int calrory;
    public NutiritionFacts(int servingSize, int servings) {
        this.servingSize = servingSize;
        this.servings = servings;
    }
    public void setFat(int fat) {
        this.fat = fat;
    }
    public void setCalrory(int calrory) {
        this.calrory = calrory;
    }
    // ...
    public static void main(String[] args) {
        NutiritionFacts facts = new NutiritionFacts(10, 1);
        facts.setFat(100);
        facts.setCalrory(200);
        // ... 필요한 값이 많을 수록 계속해서 Setter 메서드를 호출해야 한다.
    }
}
```

필요한 인자값이 많을 경우 객체 하나를 만들기 위해 여러개의 Setter 메서드를 생성해야 한다.  
가장 큰 단점은 객체를 생성한 이후에 Setter 메서드를 호출하므로, 필요한 Setter가 모두 호출되기 전까지는 객체는 완전한 상태가 아니다.  
객체를 생성하고 필요한 Setter 메서드를 호출하더라도 코드 어디에선가 누군가 잘못된 Setter를 호출하면 객체는 변하게 된다.  
즉 객체의 잘못된 값 설정으로 인해 런타임에서 프로그램이 오류를 범할 수 있는 위험이 항상 존재한다.  


## 빌더 패턴을 통해 위 문제 해결  
이러한 문제를 해결하기 위해 빌더 패턴을 고려할 수 있다.  
빌더 패턴은 필수 매개변수만으로 생성자를 호출한 뒤 빌더 객체를 참조하고, 이후 필요한 매개변수들은 빌더가 제공하는 메서드로부터 설정한다.  
빌더는 보통 정적 멤버 클래스로 구현하고 마지막에 build 메서드를 제공하여 생성하려던 인스턴스의 클래스 타입을 반환한다.  
클래스는 public 및 Setter 메서드를 제공하지 않으므로 이렇게 생성된 객체는 불변을 보장할 수 있게된다.  
따라서 자바빈즈 패턴으로 생성한 객체에 비해 훨씬 안전하게 사용할 수 있다.  
```java
class NutiritionFacts {
    private int servingSize;
    private int servings;
    private int fat;
    private int calrory;
    public NutiritionFacts(Builder builder) {
        this.servingSize = builder.servingSize;
        this.servings = builder.servings;
        this.fat = builder.fat;
        this.calrory = builder.calrory;
    }

    public static class Builder {
        private int servingSize;
        private int servings;
        private int fat;
        private int calrory;
        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }
        public Builder fat(int fat) {
            this.fat = fat;
            return this;
        }
        public Builder calrory(int calrory) {
            this.calrory = calrory;
            return this;
        }
        public NutiritionFacts build() {
            return new NutiritionFacts(this);
        }
    }

    public static void main(String[] args) {
        NutiritionFacts facts = new NutiritionFacts.Builder(1, 2).fat(100).calrory(200).build
    }
}
```


## 빌더 패턴의 단점  
객체를 생성하기위해 빌더 클래스를 항상 구현해야하므로 매개변수가 적을 땐 오히려 코드가 더 복잡해보일 수 있다.  
또한 항상 빌더 객체를 생성해야하므로 이것은 비용이 추가됨을 의미한다.  


빌더패턴도 단점은 존재하지만 불변 객체를 사용해서 얻을 수 있는 이점이 훨씬 많으므로 가급적 빌더 패턴은 사용하면 좋을 것 같다.  
lombok에서 @Builder를 사용하면 빌더 코드를 구현하지 않고도 빌더 패턴을 이용해 불변 객체를 생성할 수 있다.  
