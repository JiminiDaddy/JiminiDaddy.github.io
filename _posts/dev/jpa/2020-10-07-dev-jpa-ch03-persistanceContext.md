---
layout: post  
title: 영속성 관리
subtitle: 영속성 컨텍스트
categories: dev
tags: jpa
comments: true
---

#### JPA의 기능
엔티티와 테이블을 매핑하는 설계하는 부분
매핑한 엔티티를 실제 사용하는 부분

EntityManager
엔티티를 저장하는 가상의 Database로 엔티티를 저장하고, 수정하고, 삭제하고, 조회하는 등 엔티티와 관련된 모든 일을 처리한다.
<hr>
#### 영속성 컨텍스트 (Persistence Context)
엔티티를 영구 저장하는 환경
엔티티 매니저를 통해 엔티티를 저장하거나 조회하면 엔티티매니저는 영속성 컨텍스트에 엔티티를 보관하고 관리한다.
```java
// Example
entityManager.persist(member);  // entityManager는 member 엔티티를 영속성 컨텍스트에 저장한다.
```
<hr>
#### 엔티티의 생명주기
<strong_purple>비영속 (new/transient)</strong_purple>
- 객체 생성만 한 엔티티 (순수한 객체 상태)
  
<strong_purple>영속 (managed)</strong_purple>
- 영속성 컨텍스트에 저장하여 영속성 컨텍스트가 관리하는 엔티티
- 조회한 엔티티도 영속성 컨텍스트가 관리하는 영속 상태에 해당된다.
  
<strong_purple>준영속 (detached)</strong_purple>
- detach, close, clear등으로 영속성 컨텍스트로부터 관리받지 않게된 엔티티
  
<strong_purple>삭제 (removed)</strong_purple>
- DB와 영속성 컨텍스트로부터 삭제된 엔티티
  
<hr>
#### 영속성 컨텍스트의 특징
식별자 값
  - 영속성 컨텍스트는 엔티티를 식별자 값(@Id로 테이블 PK와 매핑된 값)으로 구분
  - <strong_red>영속 상태인 엔티티는 반드시 식별자 값을 가지고 있어야 한다.</strong_red>
  
Database 저장
  - 트랜잭션이 커밋될 때 영속성 컨텍스트에 새로 저장된 엔티티가 DB에 반영되는데 이것을 flush라고 한다.
  
장점
  - 1차 캐시
    + 영속 상태의 엔티티는 모두 1차 캐시에 저장된다. (${ID}:${EntityInstance}와 같이 Map으로 구성됨)
    + 만약 1차 캐시에 없는 엔티티라면 DB에서 조회한다.
  - 동일성 보장
    + 1차 캐시에서 반복적으로 엔티티를 조회해도 같은 인스턴스를 반환하므로 엔티티의 동일성이 보장된다.
  - 트랜잭션을 지원하는 쓰기 지연
    + 커밋하기 직전까지 내부 쿼리 저장소에 요청된 DML SQL을 꾸준히 모아두고 커밋할때 한번에 DB로 전달한다.
    + <strong_red>커밋될 때 영속성 컨텍스트가 flush되고, 영속성 컨텍스트의 변경사항을 DB에 저장한다.</strong_red>
  - 변경 감지
    + 엔티티를 수정할 때, 별도 update없이 자동으로 변경사항이 영속성 컨텍스트에 반영된다.
    + JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초 조회된 상태를 스냅샷해놓는다.
    + <strong_red>flush 시점에 스냅샷과 엔티티를 비교하여 변경사항이 있으면 update SQL을 작성하는데, 이 때 엔티티 내 모든 필드가 update된다.</strong_red>
    + 모든 필드를 update하는 SQL이 Application이 로딩될 때 미리 작성해놓기때문에 재사용이 가능하다.
  - 지연 로딩
    + 실제 객체 대신 프록시 객체를 로딩하고, 해당 객체를 실제로 사용할 때 영속성 컨텍스트를 통해 데이터를 가져온다.
    + <strong_red>엔티티를 조회할 시점에 연관된 다른 엔티티를 함께 로딩하지 않고, 실제로 연관된 엔티티를 사용할 때 SQL이 호출되므로 불필요한 데이터 로드를 줄일 수 있다.</strong_red> 하지만 즉시 로딩에 비해 DB 접근 횟수가 늘어날 수 있다.
<hr>
#### 플러시
영속성 컨텍스트의 변경 내용을 DB에 반영한다. (영속성 컨텍스트의 변경내용을 DB에 동기화)
Flush 방법
1. EntityManager.flush(); 호출
2. Transaction Commit
3. JPQL Query 실행
<strong>트랜잭션을 통해 DB의 동기화를 최대한 늦추고 커밋시점에 반영하는것이 가능하다.<strong>

준영속/비영속 -> 영속 상태로 변경하려면 병합(merge)을 사용하면 된다.

<strong_blue>병합 전/후 객체는 동일해보이지만 준영속/비영속 상태의 엔티티와 영속 상태의 엔티티는 서로 다른 인스턴스이므로 (동일성 실패) 병합 이후에는 준영속/비영속 상태의 엔티티를 사용할 필요도 없고, 사용해서도 안된다.</strong_blue>

<hr>
> 자바 ORM 표준 JPA 프로그래밍 (김영한님 저) 도서 참조
