---
layout: post
title: Service-Gateway
subtitle: Spring Netfilx-Zuul
categories: dev
tags: spring
comments: true
---

## 서비스 게이트웨이  
횡단 관심사들을 추상화하고 독립적인 위치에서 애플리케이션의 모든 마이크로서비스 호울에 대한 필터와 라우터 역할 수행  
<br/>
<strong>서비스 클라이언트가 서비스(마이크로 서비스)를 직접 호출하지 않고 단일한 정책 시행지점 역할을 하는 서비스 게이트웨이로 모든 호출을 경유시켜 목적지로 라우팅한다.</strong>  
- 핵심 관심사: 비지니스 로직과 같은 제품의 주요 기능  
- 횡단 관심사: 보안, 로깅, 추적과 같이 전체 애플리케이션에 영향을 미치는 관심사  

예제 목표  
- 하나의 URL 뒤에 모든 서비스를 배치하고, 서비스 디스커버리를 이용해 모든 호출을 실제 서비스 인스턴스로 매핑  
- 서비스 게이트웨이를 경유하는 모든 서비스 호출에 상관관계ID를 삽입  
- 호출할때 생성된 상관관계ID를 HTTP응답에 삽입하고 클라이언트로 회신  
- 대중이 사용중인 것과 다른 조직 서비스 인스턴스 엔드포인트로 라우팅하는 동적 라우팅 메커니즘 구현  
  
서비스 게이트웨이는 서비스 클라이언트와 호출될 서비스(마이크로서비스) 사이의 중개 역할 수행  
서비스 클라이언트는 서비스 게이트웨이가 관리하는 하나의 URL을 호출하여 통신  
서비스 게이트웨이는 애플리케이션 안의 모든 마이크로서비스 호출로 유입되는 모든 트래픽에 대해 게이트키퍼 역할 수행  
서비스 게이트웨이는 서비스 호출에 대한 중앙 집중식 정책 시행 지점 역할을 수행  

<strong_deepblue>각 서비스에서 서비스의 횡단 관심사를 구현할필요없고 서비스 게이트웨이에서만 구현하면 됨</strong_deepblue>  

<hr/>  

### 서비스 게이트웨이의 횡단 관심사  
1. 정적 라우팅  
   - 단일 서비스 URL과 API경로로 모든 서비스를 호출하게 한다. 개발자는 모든 서비스에 대해 하나의 서비스 엔드포인트만 알면 된다.  
2. 동적 라우팅  
   - 요청 데이터를 기반으로 서비스 호출자 대상에 따라 라우팅 수행이 가능하다. (베타 프로그램에 참여하는 고객들을 다른 버전의 API로 라우팅할 수 있다.)  
3. 인증과 인가  
   - 모든 서비스 호출은 서비스 게이트웨이로 라우팅되므로, 서비스 호출자가 자신을 인증하고 서비스를 호출할 권한이 있는지 확인할 수 있다.  
4. 측정 지표 수집과 로깅  
   - 모든 서비스 호출에 따른 로깅이 가능하므로 호출횟수, 응답시간과 같은 많은 기본 측정 지표를 서비스 게이트웨이 한곳에서 수집할 수 있다.  

<hr/>  

### Netflix-Zuul  
스프링 클라우드 애너테이션으로 설정하고, 쉽게 서비스 게이트웨이를 구출할 수 있는 오픈소스 프로젝트 
기능  
- 애플리케이션의 모든 서비스 경로를 단일 URL로 매핑  
- 게이트웨이로 유입되는 요청을 검사하고 대응할 수 있는 필터 작성  

MAVEN  

```xml
<dependencies>
   <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
   </dependency>
   <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
   </dependency>
</dependencies>
```
  
ANNOTATION  
@EnableZuulProxy  
* @EnableZuulServer: 자체 라우팅 서버를 만들고, 내장된 Zuul 기능을 사용하지 않을 때 선택 (리버스 프록시 필터 로드 X)  

<hr/>  

### Zuul  
Zuul Proxy-server는 기본적으로 스프링 제품과 동작되도록 설계됨  
<u>Zuul은 본래 Reverse-proxy다.</u>  
(자원에 접근하려는 클라이언트와 자원 사이에 위치한 중개서버로, 클라이언트의 요청을 받은 후 클라이언트를 대신해 원격 자원을 호출한다.)  
<strong_deepblue>자동으로 Eureka를 사용해 ServerID로 서비스를 찾은 후, Netflix-ribbon으로 Zuul 내부에서 요청에 대한 클라이언트 측 부하 분산을 수행한다.</strong_deepblue>  
Zuul은 모든 경로 앞에 /api와 같은 prefix를 서비스 경로에 쉽게 추가할 수 있다.  
Spring-cloud-Cconfig-server를 사용해 Zuul-server를 재시작하지 않고 경로 매핑을 동적으로 로드할 수 있다.  

<hr/>  

### Zuul-Mapping-Machanizm  
1. 서비스 디스커버리를 이용한 자동 경로 매핑  
   - ServerID를 기반으로 요청을 자동 라우팅한다.  
2. 서비스 디스커버리를 이용한 수동 경로 매핑  
   - application.yml에 경로를 명시적으로 정의해서 모든 경로를 매핑한다.  
3. 정적 URL을 이용한 수동 경로 매핑  
   - 유레카로 관리하지 않은 서비스를 라우팅하는데 사용한다.(고정 URL에 직접 라우팅)  

Zuul은 Hystrix와 ribbon을 사용해 오래 수행되는 서비스 호출이 서비스 게이트웨이의 성능에 영향을 미치지 않도록 한다.  
(기본적으로 Zuul은 1초 이상 걸리는 호출을 종료하고 HTTP500 에러를 반환한다. -> Hystrix의 기본동작)  

Timeout 설정  
1. Zuul로 실행중인 모든 서비스에 대해 Hystrix의 timeout 설정  
2. 특정 서비스에 대해 Hystrix의 timeout 설정  
3. Netflix-ribbon의 timeout 설정  

<hr/>  

### Zuul-Filter  
1. 사전 필터 (Zuul에서 목표 대상에 대한 실제 요청이 발생되기 전 호출)  
   - 클라이언트의 인증, 인가  
2. 경로 필터 (대상 서비스가 호출되기 전에 호출을 가로채는데 사용)  
   - 동일 서비스의 두 버전을 라우팅할 수 있는 경로 단위 필터를 사용해 특정 비율의 호출을 새 버전의 서비스로 라우팅할 수 있다.  
3. 사후 필터 (대상 서비스를 호출하고, 응답을 클라이언트로 전송한 후 호출)  
   - 대상 서비스의 응답을 로깅하거나 에러처리, 민감한 정보에 대한 응답 감시  



<hr>

> 스프링 마이크로서비스 코딩공작소 (존 카넬) 도서 참조