---
layout: post
title: Entity Mapping
subtitle: 엔티티 매핑
categories: dev
tags: jpa
comments: true
---


#### JPA에서 가장 중요한 일?
<u>Entity와 Table을 정확해 매핑하는 일</u>
JPA에서는 Entity와 Table 매핑을 위해 다양한 Annotation을 제공한다.
- 객체와 테이블 매핑
  + @Entity
  + @Table
- 기본 키 매핑
  + @Id
- 필드와 컬럼 매핑
  + @Column
- 연관관계 매핑
  + @ManyToOne
  + JoinColumn
<hr>

#### @Entity
테이블과 매핑할 클래스에 필수로 붙여줘야 JPA에서 관리된다.
name 속성으로 엔티티의 이름을 지정할 수 있으며, 기본 값은 클래스명이다.
어노테이션을 적용시 아래와 같이 주의사항이 존재한다.
1. 기본 생성자는 필수로 구현되어 있어야한다. (접근 제한자는 public or protected가 되야 한다.)
<u>JPA에서 엔티티 객체를 생성할 때, 기본 생성자를 이용한다. 따라서 파라미터가 있는 생성자를 구현할 경우 기본 생성자도 함께 구현한다.</u>
2. final, inner 클래스 및 enum, interface에는 사용할 수 없다.
3. 저장할 필드는 final로 사용할 수 없다.
<hr>

#### @Table
엔티티와 매핑할 테이블을 지정한다.
name속성으로 테이블 명을 지정할 수 있는데, 기본값은 엔티티 이름이다.
단, 테이블명이 데이터베이스 예약어와 충돌 되는 경우가 있으므로(ex. Order) 보통 "T_", "TB_" 와 같은 Prefix를 추가하거나 Suffix에 "S를 붙인다. 
<hr>

#### 기본 키 매핑
직접 할당 : 기본 키(Primary Key)를 어플리케이션에서 할당한다. 따라서 @Id값이 설정되지 않은 채 영속화를 시도한다면 예외가 발생한다.
기본 키로 사용할 필드에 @Id를 추가해준다.
자동 생성 : 기본 키를 데이터베이스와 같은 타 시스템에서 할당한다. 
기본 키로 사용할 필드에 @Id와 @GeneratedValue를 추가하며, 아래와 같은 생성 타입이 나뉘어진다.
  - IDENTITY : 데이터베이스에 위임한다. (DB에 엔티티가 저장된 후 키가 생성된다.)
  - SEQUENCE : 데이터베이스의 시퀀스에서 식별자 값을 획득한 후, 영속성 컨텍스트에 저장한다.
  - TABLE : 데이터베이스 시퀀스용 테이블에서 식별자 값을 획득한 후, 영속성 컨텍스트에 저장한다.
  - AUTO : 선택한 데이터베이스에 따라 위 3가지중 하나를 자동으로 설정한다. (ORACLE = SEQUENCE, MYSQL = IDENTITY)

IDENTITY
데이터베이스에 위임하는 전략이며, 기본 키 필드를 비워두고 엔티티를 저장하면, 데이터베이스에 값이 저장되고 나서 기본 키가 할당된다.
(MySQL의 AUTO_INCREMENT와 유사한 방식)
JPA는 엔티티를 저장할때 + 기본 키값을 얻기 위해 반드시 데이터베이스에 조회해야하므로 최소 2번의 조회가 이루어진다.
~~JDBC3에 추가된 Statement.getGeneratedKeys()를 사용하면 데이터 저장 + 기본 키 획득을 1번의 조회로 가능하다.~~
데이터베이스에 저장해야 기본 키를 할당받을 수 있으므로 엔티티를 영속화할때 JPA는 데이터베이스로 Insert 쿼리를 즉시 날린다.
<string_red>따라서 이 방식은 트랜잭션 쓰기지연 방식을 사용할 수 없다.</string_red>

SEQUENCE
유일한 값을 순서대로 생성하는 데이터베이스의 특별한 오브젝트 SEQUENCE를 사용한다. 
따라서 SEQUENCE를 지원하는 데이터베이스만 사용할 수 있으며 Oracle, PostgreSQL, DB2, H2에서 사용 가능하다.
(Example. CREATE SEQUENCE MY_SEQ START WITH 1 INCREMENT BY 1; 과 같은 DDL문으로 생성 가능하다.)

TABLE
시퀀스용 테이블을 임의로 생성하고, 해당 테이블의 컬럼으로부터 값을 읽어오는 방식이므로 데이터베이스의 영향을 받지 않는다. (모든 DB에서 사용 가능하다.)
별도의 테이블을 사용하는 것 외에는 SEQUENCE방식과 내부 동작은 동일하다.
테이블이 값이 없을경우 JPA가 Insert 쿼리를 날리므로 예외가 발생하지 않는다.
<hr>

#### 필드와 컬럼 매핑
@Column
해당 필드를 DB의 컬럼으로 매핑한다.
null허용여부, unique, length등을 설정할 수 있다.
기본값으로는 nullable=true이므로 자바의 기본타입을 사용한다면 주의해야한다. (자바의 기본타입은 반드시 not null이다.)
<string_red>왜냐하면 자바의 기본타입 필드에 @Column 기본속성을 사용한다면 DB의 컬럼은 null이 될 수 있는데 엔티티의 필드는 null이 될 수 없어 오류가 발생할 수 있다.</string_red>
따라서 자바의 기본타입을 컬럼으로 매핑할 때는 nullable=false로 설정해야 한다.

@Enumerated
Enum 필드를 DB의 컬럼으로 매핑하되, 옵션의 기본값은 ORDINAL이다. 
EnumType.STRING : Enum의 이름을 DB에 저장
EnumType.ORDINAL : Enum의 순서를 DB에 저장

@Temporal
날짜 타입의 필드를 DB의 컬럼으로 매핑한다. 아래 셋중 하나의 옵션을 필수로 설정해야 한다.
TemporalType.DATE : DB의 DATE 타입과 매핑
TemporalType.TIME : DB의 TIME 타입과 매핑
TemporalType.TIMESTAMP : DB의 TIMESTAMP 타입과 매핑

@Lob
해당 필드를 DB의 CLOB, BLOB 타입으로 매핑한다.
필드 타입이 문자인 경우 (String, char[])는 CLOB, 그 외에는 BLOB으로 매핑된다.

@Transient
해당 필드를 DB에 매핑하지 않는다. DB에 저장되지 않으므로 당연히 조회할수도 없다.

@Access
JPA가 엔티티에 접근하는 방식을 설정한다.
AccessType.FIELD : JPA가 엔티티 필드에 직접 접근하며, private 필드도 접근 가능하다.
AccessType.PROPERTY : JPA가 엔티티의 Getter 메서드를 통해 접근한다.
AccessType을 설정하지 않으면 @Id의 위치를 기준으로 접근방식이 설정된다.
(@Id가 필드에 설정되어있다면 FILED방식, @Id가 Getter에 설정되어 있다면 PROPERTY방식)


<hr>
> 자바 ORM 표준 JPA 프로그래밍 (김영한님 저) 도서 참조