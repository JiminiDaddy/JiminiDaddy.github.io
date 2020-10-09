---
layout: post
title: "H2 DB에서 Table이 Drop되지 않는 문제"
subtitle: H2와 JPA버전간 의존성 문제
categories: dev
tags: trouble-shooting
comments: false
---





Spring-Boot Webservie 샘플 프로젝트를 돌리는 중에 에러가 하나 발생했다.

---

**jdbc.JdbcSQLSyntaxErrorException: Syntax error in SQL statement "CREATE TABLE POSTS ..... "**

![Alt text](/assets/img/dev/trouble-shooting/springboot229버전업후_실행에러.png)

열심히 구글링했는데 딱히 나오지 않다가 인프런에 누군가가 나랑 같은 현상을 겪는 것 같았다.

이런저런 질문답변이 주고받고 진행되다가 김영한님이 답변해주신 내용이 하나 보였다.

H2 DB버전의 최신버전이 1.4.200으로 되어있는데, 해당 버전이 버그가 있어 Hibernate와 충돌이 난다고 한다.

난 pom.xml에 버전은 따로 명시하지않았는데... 설마 하는마음에 확인했더니

1.4.200으로 되있었다.

![Alt text](/assets/img/dev/trouble-shooting/h2버전오류.png)

버전정보만 수정해서 1.4.199(~~현재 lastest stable version)~~ 로 다시 의존성 받고 실행했더니  
다행히 서버도 정상적으로 올라오고 정상적으로 테스트 코드도 수행되었다.

![Alt text](/assets/img/dev/trouble-shooting/h2버전다운.png)

#### H2버전 1.4.200 -> 1.4.199 변경 후 정상동작

![Alt text](/assets/img/dev/trouble-shooting/h2버전변경후정상동작.png)

~~이미지캡쳐해서 메일로 다 보내놨는데 생각해보니 보안때문에 집에서 다운받을수가없네ㅠㅠ 이쁘게 꾸미고싶었는데~~

---

이런 버전 정보하나하나까지 다 일일이 체크하면서 개발하긴 어려울거같은데..  
다른 개발자분들 정말 다 개발할때마다 라이브러리 버전 체크하면서 하시려나?  
구글링해서 안나왔으면 과연 내가 이 문제를 해결했을까? 라는 생각이 든다.

---
