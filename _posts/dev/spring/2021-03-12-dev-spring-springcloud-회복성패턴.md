---
layout: post
title: Application Recovery
subtitle: Spring-Cloud Histrix
categories: dev
tags: spring
comments: true
---

## 애플리케이션 회복  
<strong>클라이언트 부하 분산</strong>  
- 리본을 사용해서 분산처리  

<strong>회로차단기</strong>  
- 빠른실패, 원만한실패, 원만한회복  

<strong>폴백</strong>  
- 실패할경우 대안은?  

<strong>벌크헤드</strong>  
- 서비스를 분리시켜 문제가 발생한서비스를 제거함으로써, 다른 서비스에 영향없도록  

<hr/>  

### 회로차단기  
maven  

```xml
spring-cloud-starter-netflix-hystrix
```  

bootstrap class

```java
 @EnableCircuitBreaker  
 @SpringBootApplication
 public static void main(String[] args) {
     ...
 }
 ```

<strong>A서비스 -> B서비스로의 호출이 있는경우</strong>  
1. A서비스 -> A-DB로의 호출을 Hystrix 회로차단기에 연결  
2. B서비스 -> B-DB로의 호출을 Hystrix 회로차단기에 연결  
3. A서비스 -> B서비스의 호출을 Hystrix 회로차단기에 연결  
  
<br/>  
<strong>동기식 Hystrix 회로차단기</strong>  
스프링이 제공하는 <strong_blue>@HystrixCommand</strong_blue> 어노테이션을 사용하면 Hystrix가 회로차단기가 관리하는 Java-Class-Method라고 SpringFramework가 인지한다.  
- SpringFramework는 <strong_blue>@HystrixCommand</strong_blue> 애너테이션을 만나면, 이 메서드를 감싸는 Proxy를 동적으로 생성하고, 원격호출을 처리하기위해 확보한 Thread가 있는 Thread-Pool로 해당 메서드에 대한 모든 호출을 관리한다.  
  
 @HystrixCommand를 프로퍼티없이 기본값으로만 사용하면 모든 원격 서비스 호출에, 동일한 스레드 풀을 사용하므로 애플리케이션에 문제가 발생할 수 있다.  
따라서 프로퍼티를 통해 원격 서비스 호출을 자체 스레드 풀로 분리하거나, 스레드 풀을 독립적으로 동작시키는 구성이 필요하다.  

```java
@HystrixCommand(
    commandProperties = {
        // 회로차단기가 타임아웃 되기까지 기다리는 시간
        @HystrixProperty(
            name="execution.isolation.thread.timeoutInMilliseconds", value="12000"
        )
    }
)
public void METHOD() {
    ...
}
```

<strong>폴백전략 (대안을 찾는다)</strong>  
- 자원이 타임아웃되거나, 실패할때의 행동방침 제공하는 메커니즘  
- 대안을 수행하는 폴백메서드 또한 이전과 동일한 장애가 발생할 수 있음을 인지해야한다.   

```java
@HystrixCommand(
    // 실행할 폴백 메서드를 기술한다.
    fallbackMethod = "getLicenseFallback"
)
public List<License> getLicenseByOrg(String organizationId) {
    ...
}
// 폴백 메서드는 이전 메서드와 서식(Parameter, ReturnType)이 완전히 동일해야한다. (대리 역할을 해야하므로)
public List<License> getLicenseFallback(String organizationId) {
    ...
}
```
  
<strong>벌크헤드 패턴 구현</strong>  
- 기본적으로 Hystrix가 공용 스레드풀을 사용하여 각 서비스들의 원격호출이 동일한 스레드풀에서 실행된다.  
- 따라서 특정 서비스의 처리가 느려지면 스레드가 몰리는 현상이 발생하여, 전체 서비스에 영향일 끼칠 수 있다.  
- Hystrix에 벌크헤드 패턴을 적용하면, 각 서비스별로 스레드풀을 생성할 수 있으므로, 문제가 발생하는 서비스와 완벽히 격리될 수 있다.  

```java
@HystrixCommand(
    fallbackMethod = "~~~~",
    // 고유한Key로 해당 원격서비스 호출을 위한, 새로운 스레드 풀을 생성한다.
    threadPoolKey = "스레드풀_고유Key",
    threadPoolProperties = {
        // 스레드 풀 사이즈
        @HystrixProperty(name = "coreSize", value="30"),
        // 스레드 풀이 모두 작업중일때, 대기할 큐의 사이즈
        @HystrixProperty(name="maxQueueSize", value="10"),
    },
)
```
  
maxQueueSize가 -1 이면 SynchronousQueue가 사용되며, maxQueueSize가 1 이상이면 LinkedBlockQueue를 사용한다.  
 
이상적인 스레드풀 사이즈(by Netflix)  
- (서비스가 정상이라는 가정하에 최고점에서 초당 요청수 * 99 백분위 수 지연 시간(초)) + 오버헤드를 대비한 소량의 추가 스레드  

<hr/>  

### Hystrix 세부 설정
- Hystrix가 회로차단기의 차단 시점을 결정하는 과정  
  
1. 문제가 발생했을 때(지연 발생?) 최소 요청수가 실패했는지?  
   - 실패수가 임계치 아래면 정상으로 간주 (회로차단기 동작 X)  
  
2. 에러 임계치에 도달했는지?  
   - 실패수가 임계치를 넘어가면 회로차단기 동작하고, 넘어가지 않았다면 동작 X  
  
3. 원격 서비스 호출에 여전히 문제가 발생하는지?  
   - Hystrix가 주기적인 호출(모니터링)하는데 이 때 성공으로 결정되면 다시 회로차단기를 초기화시켜 호출을 허용한다.  
   - 실패로 결정되면 차단된 상태를 유지하고, 일정 시간 뒤에 재호출한다. (성공될때까지 반복)  

```java
@HystrixCommand(
    commandPoolProperties = {
        // Hystrix가 호출 차단을 고려하기까지 대기 시간으로, 이 시간동안 연속 호출한 횟수를 제어한다.
        @HystrixProperty(name="circuitBreaker.requestVolumeThreshold", value="10"),
        // 회로차단기를 차단한 후, 설정한10초만큼 호출한 후, 타임아웃/예외/Http500 에러 등으로 실패해야 하는 호출 비율
        @HystrixProperty(name="circuitBreaker.errorThresholdPercentage", value="75"),
        // Hystrix가 서비스의 회복 상태를 확인하기까지 대기할 시간
        @HystrixProperty(name="circuitBreaker.sleepWindowInMilliseconds", value="7000"),
        // 서비스 호출 문제를 모니터링할 시간
        @HystrixProperty(name="metrics.rollingStats.timeInMilliseconds", value="15000"),
        // timeInMilliseconds의 시간동안 통계를 수집할 횟수 (15초동안  5개의 버킷에 통계 데이터를 수집해야하므로, 하나당 3초 사용)
        @HystrixProperty(name="metrics.rollingStats.numBuckets", value="5")}
    }
)
```
  
* 확인할 통계 간격이 작고(3초) 버킷 수(5개)가 많을수록, 서비스에서 CPU, Memory 사용량이 증가한다.  

<hr/>  

### Thread-Context & Hystrix  
<u>Hystrixs의 격리 전략으로는 THREAD방식과 SEMAPHORE방식이 있다.</u>  
- 대부분은 THREAD방식 사용  
  . <strong_red>부모스레드(호출한 서비스)와 자식스레드(Hystrix 실행 스레드)가 완전히 분리됨</strong_red>  
  . 부모 스레드에 영향없이 자식 스레드의 실행을 중단시킬 수 있음  
  . 무거움  

- 비동기 I/O에서는 SEMAPHORE 사용  
  . 타임아웃이 발생하면 부모 스레드를 중단시키므로, 톰캣과 같은 동기식 컨테이너 환경에서는 예외처리가 불가능함  
  . 대용량처리  
  . 가벼움  


  <strong>THREAD방식을 기본적으로 사용하므로, 부모 스레드의 컨텍스트가 자식 스레드(Hystrix과 관리하는 스레드)로 전파되지 않는다.</strong>  
  - 부모 스레드에서 ThreadLocal로 설정된 값은 자식 스레드에서 사용할 수 없다.  
  - ThreadLocal로 생성한 변수/객체는 각 스레드에서 독립적으로 저장되는데 부모->자식으로 전파가 안되므로 부모 스레드에서 생성한 변수/객체를 자식 스레드에서 사용할 수 없다.  
  - 예를들어 Service Class의 특정 메서드를 @HystrixCommand를 통해 Hystrix가 관리하도록 설정하면 Filter에서 ThreadLocal로 생성한 변수는 Controller까지는 부모 스레드영역이므로 값이 유지되지만,   Hystrix가 관리하는 Service의 메서드를 호출할경우 자식 스레드의 영역이므로 값이 유지되지 않는다.   (값이 없다. 설정/할당한적 없으므로)
  - 부모스레드에서 자식스레드로 전파하기위해서 HystrixConcurrencyStrategy(병행성 전략)을 사용한다.  
  
<hr/>  

### HystrixConcurrencyStrategy  
Hystrix를 사용해서 병행성전략을 사용자 정의하고, 이를통해 부모 스레드의 컨텍스트를 자식 스레드로 전파할 수 있다.  

<strong>필요작업<strong>  
<strong_deepblue>1. HystrixConcurrencyStrategy 클래스의 사용자 정의 (상속)</strong_deepblue>  
   -  스프링 클라우드는 이미 스프링 보안정보를 전달하기위해 HistrixConcurrencyStrategy을 체인으로 연결하는 기능을 제공하므로, 사용자정의한 ConcurrencyStrategy를 HistrixConcurrencyStratege에 플러그인으로 사용하게 한다.  
<strong_deepblue>2. HistryxCommand에 UserContext를 주입하도록 자바 Callable 클래스 정의 (구현)</strong_deepblue>  
   - @HystrixCommand가 메서드를 보호하기전에 Callbale.call(); 메서드가 호출되므로, 이때 부모 스레드로부터 전달받은 UserContext를 자식 스레드에 설정한다.  
   - 이후 Callable.call(); 이 호출되어 Hystrix가 보호하는 메서드가 호출되면, 자식 스레드는 부모 스레드로부터 전달받은 UserContext를 공유하게 된다. (ThreadLocal.set 메서드 호출)  
   - ThreadLocal은 현재 실행중인 스레드의 값을 설정하므로, 현재 실행중인 자식스레드의 userContext값이 설정된다.  
<strong_deepblue>3. HystrixConcurrencyStrategy을 사용자 정의하기위해 스프링 클라우드 구성하기</strong_deepblue>  
   - 스프링 클라우드와 Hystrix에서 HystrixConcurrencyStrategy, Callbale 클래스를 Hoocking하도록 연결해야 한다.  
     (ThreadLocalConfiguration class(설정 클래스)를 구현한다.)  
   - DI를 통해 HystrixConcurrencyStrategy 객체를 주입받는다.  
   - @PostConstruct를 사용하여, 초기화시점에 모든 Hystrix Component를 가져와 Hystrix Plugin을 재설정한다.   
   - HystrixPlugins에 생성한 HistryxConcurrencyStrategy(ThreadLocalAwareStrategy)를 등록한다.  
   - HystrixPlugins이 사용하는 모든 HystrixComponent를 재등록한다.  


<hr>

> 스프링 마이크로서비스 코딩공작소 (존 카넬) 도서 참조