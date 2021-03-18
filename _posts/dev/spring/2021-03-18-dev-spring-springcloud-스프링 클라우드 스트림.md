---
layout: post
title: Async Event & Caching
subtitle: Spring-Cloud-Stream Kafka & Redis
categories: dev
tags: spring
comments: true
---

## Caching의 목적  
A서비스에서 B서비스로 데이터를 조회하는경우 A서비스는 B서비스에 연결된 Database에 Access해야 하며 이는 결국 비용이다.  
만약 B서비스의 데이터가 한번 저장되면 자주 변경되지 않는 데이터라면?  
한번 가져온 데이터를 어딘가에 Caching 한다면 Database에 Access하는 비용을 감소시킬 수 있으므로 응답시간 향상에 도움이 된다.  
<strong_red>대표적인 Caching으로 Redis(Nosql)을 사용한다.</strong_red>  

## Caching System의 요구사항  
1. <strong>캐싱된 데이터는 같은 서비스 그룹 내 모든 인스턴스에서 일관성이 있어야 한다.</strong>  
   - A서비스가 10개의 인스턴스를 생성했을경우, 모든 인스턴스는 Caching한 결과가 같아야 한다.  
   - 즉, 각 마이크로 서비스의 인스턴스안에 로컬 캐싱을 해선 안된다.  
2. <strong>컨테이너 메모리에 데이터를 캐싱하는 일을 피한다.</strong>  
   - 런타임 컨테이너는 종종 크기 제약이 있고, 다양한 패턴으로 데이터를 액세스한다.  
   - 로컬 캐시는 클러스터 내 다른 서비스들과 동기화를 보장해야하므로 복잡성이 증가한다.  
3. <strong>Database의 레코드가 변경되었을 경우 호출하는 서비스는 호출된 서비스의 상태 변화를 인식해야 한다.</strong>  
   - B서비스의 데이터가 변경되었을경우 A서비스는 캐싱된 B서비스의 특정 데이터를 삭제할 수 있어야 한다.  
  
<hr/>  

## Sync vs Async  
### 동기  
RestAPI를 통해 두 서비스간 통신을 구성한다.  
> <u>대표적으로 RestTemplate이나 Feign을 사용한다.</u>  
  
결제 시스템과 같이 외부 서비스를 이용해 결과를 받아야하는경우는 동기처리한다. (트랜잭션이 유지되어야 한다.)  
단, 실패할경우 예외처리를 반드시하고 롤백시킨다. 또는 회로 차단기를 사용하여 실패시 다음 대안으로 넘어간다.  

### 비동기  
호출하는 서비스는 메시지를 큐에 발행하고, 수신하는 서비스는 중개자로부터 수신한다.  
> <u>대표적으로 Kafka, RabbitMQ를 사용한다.</u>  
  
### 동기방식의 단점  
동기식으로 구현할경우 DB로부터 데이터를 읽어와 메시지를 전달하는 서비스와, 메시지를 수신하여 Cache-server에 저장/조회하는 서비스 간 결합이 강해진다.  
A서비스의 엔드포인트가 변경되면 A서비스로 메시지를 전송할 B서비스의 수정이 불가피하다.  
A서비스의 성능이 저하될경우 A서비스에게 메시지를 보낼 B서비스 또한 응답이 지연되므로 함께 성능이 저하될 수 있다.  
만약 B서비스가 A서비스의 Redis에 직접 통신한다면 B서비스와 Redis간 의존도 추가되는 문제도 발생할 수 있다.  
B서비스의 데이터를 수신하고싶은 C서비스가 추가될경우 B서비스의 코드가 수정되야하는 문제가 발생할 수 있다.  
> (ex. FeignClient 추가)  
  
두 서비스간 직접 통신하기 때문에 발생한 문제로, 마이크로 서비스 환경에서는 이러한 방식은 좋지 않다. ~~(금기한다.)~~  

### 동기방식의 단점을 보완한 비동기 처리  
위와같은 문제를 해결하기위해 비동기로 메시지를 전송하는 방식을 사용한다.  
메시지 발행(전송)측은 수신 대상의 서비스에 대해 알지못하고 오직 메시지 큐에만 의존한다.  
메시지 구독(수신)측은 전송 대상의 서비스에 대해 알지못하고 오직 메시지 큐에만 의존한다.  
따라서 발행/구독 서비스가 증가하거나 감소하더라도 각 서비스들은 코드를 변경할 필요가 없어진다.  
또한 데이터 조회를 위한 서비스가 장애가 발생하더라도, 캐쉬에 있는 데이터를 사용하면 되므로 서비스가 유지될 수 있다.  
> (비록 지난 데이터일 수도 있지만 없는 데이터보단 운영에서 확실히 낫다.)  
  
### 비동기방식의 단점
애플리케이션이 메시지의 소비 순서를 기반으로 어떻게 동작할지와 메시지가 순서대로 처리되지 않을 때 어떤일이 발생할지 이해해야 한다.  

<hr/>

### Redis 설치  
brew로 설치하는게 아무래도 편하니 그냥 brew로 설치  
```java
// 설치
brew install redis
```

Redis Service 실행  
```java
brew services start redis
```

Redis Path  
```java
- 실행파일 : /usr/local/bin/redis-server
- 설정파일 : /usr/local/etc/redis.conf
```

Redis Server 실행  
```java 
${실행파일경로}/redis-server or command-line 에서 redis-server
// 기본 포트: 6379
```

Sample Application 구현 (for Redis & Kafka) 
  - CommentService에서 댓글이 하나 추가될 때마다 PostService로 Message를 전송하고, PostService는 수신된 메시지를 Redis에 저장함  
  - Client가 게시물 정보를 조회할 때 PostService는 RedisServer로부터 댓글목록을 조회하고, 있으면 캐쉬 데이터를 사용하고 없는경우 CommentService로 Rest를 통해 댓글목록을 가져옴  
  
<hr/>

## 필요한 파일 설정  

### Redis
<strong>pom.xml</strong>  
   - 의존성 라이브러리 추가  

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
   <groupId>redis.clients</groupId>
   <artifactId>jedis</artifactId>
</dependency>
<dependency>
   <groupId>org.apache.commons</groupId>
   <artifactId>commons-pool2</artifactId>
</dependency>
```
<br/>  
<strong>RedisConfig 구현</strong>  
   - Redis서버에 접속할 Connection과 객체 저장에 필요한 Bean 정의  

```java
@Configuration
public class RedisConfig {
   // Redis-server에 실제 Database-Connection을 설정한다.
   @Bean
   public JedisConnectionFactory jedisConnectionFactory() {
      JedisConnectionFactory factory = new JedisConnectionFactory();
      return factory;
   }

   // Reids-server에 작업을 수행하는데 필요한 RedisTemplate 객체 생성
   @Bean
   public RedisTemplate<String, List<CommentMessage>> redisTemplate() {
      RedisTemplate<String, List<CommentMessage>> template = 
         new RedisTemplate<>();
      template.setConnectionFactory(jedisConnectionFactory());
      return template;
   }
}
```
<br/>  
<strong>Redis Repository 구현</strong>  
   - 간단하게 SAVE, FIND 기능을 구현
   - ~~데이터를 저장할 때 마다 Key에 해당하는 List를 가져와서 추가하고 다시 저장하는데 더 좋은 방법이 없을까?~~  

```java
Slf4j
@RequiredArgsConstructor
@Repository
public class CommentRedisRepository {
    private static final String COMMENT_KEY = "COMMENT";
    private static final String KEY_PREFIX = "POST";

    private final RedisTemplate<String, List<CommentMessage>> redisTemplate;
    private HashOperations<String, String, List<CommentMessage>> hashOperations;

    public List<CommentMessage> findAllComments(Long postId) {
        List<CommentMessage> allComments = 
           hashOperations.get(COMMENT_KEY, createKey(postId));
        return Optional.ofNullable(allComments).orElseGet(ArrayList::new);
    }

    public void saveComment(CommentMessage commentMessage) {
        String key = createKey(commentMessage.getPostId());
        List<CommentMessage> allComments = 
            findAllComments(commentMessage.getPostId());
        allComments.add(commentMessage);
        hashOperations.put(COMMENT_KEY, key, allComments);
    }

    public void saveAllComments(Long postId, List<CommentMessage> allComments) {
        String key = createKey(postId);
        hashOperations.put(COMMENT_KEY, key, allComments);
    }

    @PostConstruct
    private void init() {
        hashOperations = redisTemplate.opsForHash();
    }

    private String createKey(Long postId) {
       // postId에 해당하는 Comments를 얻기위해 postId에 의존한 Key를 생성한다.
       return KEY_PREFIX + ":" + postId;
    }
```

### Kafka  
<strong>pom.xml</strong>  
   - 의존성 라이브러리 설치  

```xml
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-stream</artifactId>
</dependency>
<dependency>
   <groupId>org.springframework.cloud</groupId>
   <artifactId>spring-cloud-starter-stream-kafka</artifactId>
</dependency>
```
  
<br/>  
<strong>CommentClient (Redis로부터 캐싱된 데이터 조회)</strong>  
   - Client가 Rest로 게시물을 조회하면 PostService는 댓글목록을 가져오기 위해 RedisServer에 먼저 캐싱된 데이터가 있는지 확인하고 없는경우 CommentService로 데이터를 요청  

```java
@Slf4j
@RequiredArgsConstructor
@Component
public class CommentClient {
    private final CommentRedisRepository commentRedisRepository;
    private final RestTemplate restTemplate;
    private static final String COMMENTS_SERVICE_URL = "http://localhost:8080/api/v1/comments";

    public List<CommentMessage> findAllComments(Long postId) {
        log.debug("findAllComments, postId:<{}>", postId);
        // TODO null -> Optional로 변환
        List<CommentMessage> comments = findAllCommentsRedisCache(postId);
        if (comments != null) {
            log.debug("Find allComments, by cached");
            return comments;
        }
        log.debug("Cannot find comment by redis-cache. So try to find from Comment-service");
        // TODO RestTemplate -> OpenFeign 전환, 코드가 많이 지저분함
        ResponseEntity<List> restExchange = restTemplate.exchange(
                COMMENTS_SERVICE_URL + "/{postId}",
                HttpMethod.GET,
                null,
                List.class,
                postId);

        comments = (List<CommentMessage>) restExchange.getBody();
        if (comments == null) {
            log.info("Comments is empty");
            return new ArrayList<>();
        }

        saveComment(postId, comments);
        return commentRedisRepository.findAllComments(postId);
    }

    private List<CommentMessage> findAllCommentsRedisCache(Long postId) {
       return commentRedisRepository.findAllComments(postId);
    }

    private void saveComment(Long postId, List<CommentMessage> commentMessages) {
       commentRedisRepository.saveAllComments(postId, commentMessages);
    }
}
```

<hr/>

### Test  
게시물 등록 및 결과
처음 게시물을 등록할땐 댓글이 없으므로 null이 조회됨  
![Alt Text](/assets/img/dev/spring/dev-spring-save-post.png)  
  
<br/>  
댓글 등록 및 결과  
![Alt Text](/assets/img/dev/spring/dev-spring-save-comment.png)  
  
<br/>  
CommentService to PostService 메시지 발행 (kafka)  
![Alt Text](/assets/img/dev/spring/dev-spring-kafka-producer.png)  
  
<br/>  
PostService에서 메시지 수신  
![Alt Text](/assets/img/dev/spring/dev-spring-kafka-consumer.png)  
  
<br/>  
게시물 조회 및 결과  
![Alt Text](/assets/img/dev/spring/dev-spring-find-post.png)  
  
<br/>  
CommentClient에서 RedisServer로부터 캐쉬된 데이터 접근  
![Alt Text](/assets/img/dev/spring/dev-spring-redis-find_all_comments.png){: width="100%" height="40"}  
  
