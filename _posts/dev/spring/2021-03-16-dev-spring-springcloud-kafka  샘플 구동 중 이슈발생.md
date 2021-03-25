---
layout: post
title: Kafka 개발환경 구축중 이슈 (버전)
subtitle: SpringBoot-SpringCloud간 버전 의존성
categories: dev
tags: spring
comments: true
---

## KAFKA Message 이슈  
## 사전작업  
1. 카프카 설치  
    - brew install kafka, zookeeper 했으나 구동안되서 패스
    - apache 공식에서 다시 다운로드 
    - /usr/local/kafka/kafka_2.12...
  
2. 카프카 실행  
    - property enable  
        vi config/server.properties  
        listeners=PLAINTEXT://localhost:9092  
    - zookeeper  
        bin/zookeeper-server-start.sh config/zookeeper.properties  
    - kafka  
        bin/kafka-server-start.sh config/server.properties  
      
3. 카프카 기본 제공되는 스크립트를 통해 테스트 진행
```java
// create topic  
bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test  
// topic list
bin/kafka-topics.sh --list --bootstrap-server localhost:9092  
// publish
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test  
```  

## 문제의 시작  
SpringBoot 2.2.X로 프로젝트를 구성했으나 이상하게 Kafka에 토픽 생성이안됨  
https://spring.io/projects/spring-kafka 에서 spring-kafkad와 spring-boot 버전을 확인했으나, 이상없는듯함  
내 프로젝트에서 kafka버전 확인했더니 3.x대로 설치되어 있음 -> springboot 2.2.x이상에서는 되야하는데, 일단안되서 버전 낮추기로 변경  

Springboot 2.0.3으로 내림  
따라서 Springcloud 버전도 하나 내림  
내렸더니 spring-kafka버전이 2.0대로 내려감  
동작함  
Maven Project -> Dependencies 확인  
![Alt Text](/assets/img/dev/spring/dev-spring-boot-cloud-kafka-version-dependency.png)


PostService와 CommentService를 구동하려 했더니 아래와 같이 오류 발생  
```
KafkaProducerMessageHandler. at attemp was made to call a method that does not exist.  
KafkaProducerMessageHandler.determineSendTimeout  
```
![Alt Text](/assets/img/dev/spring/dev-spring-kafka-version-issue.png)  


왜 생산자에서 문제가 생길까? 구글링함  
[참고 링크](https://github.com/spring-cloud/spring-cloud-stream/issues/2047)  


API-Gateway에서 아래와 같은 에러 메시지 발생  
```
com.netflix.client.ClientException: Load balancer does not have available server for client
```


게이트웨이에서 아래와 같이 설정추가함  
```java
// application.yml에서 Gateway를 Eureka-Agent에 등록  
eureka:
    client:
        fetchRegistry: true  
```

게이트웨이 동작함!  
CommentService에서 PostService로 Kafka를 통해 메시지 전송 완료!  
  
2.2.X -> 2.0.3으로 버전 내렸더니 뭔가 문법적인 부분에서도 자꾸 IDE에서 오류 찾던데..  
그런부분은 나중에 확인해야봐야할거같음  
  
~~아마도 모듈간 의존성 문제로 예상되긴함.. SpringBoot 2.1.x 이하에서는 Junit4가 Default인것처럼..~~   

