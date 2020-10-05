---
layout: post
title: "Maven DependencyManagement 사용시 주의사항"
subtitle: "대소문자 차이로 인해 의존성설치 안되던 현상"
categories: dev
tags: trouble-shooting
comments: false
---



Spring Cloud 모듈을 사용해보고 싶어서 Spring Cloud & MSA 키워드를 통해 구글링을 해봤다.  


너무나 많은 분들이 이미 많이 개발하고 계시고 블로그도 잘 정리해놓은 것을 보고 예제로 한번 구현해봤다.  


물론 복붙은 안했지만 다만 있는그대로만 진행해보면 조금 재미없으니까 일단 돌아가는거 한번보고 Maven Multi Module로 변경하면서 진행해보기로 했다.  

분명히 각각 프로젝트로 구성했을 때 잘되는거 확인했는데 멀티 모듈로 변경하니까 갑자기 의존성 라이브러리를 받지 못하는 현상이 발생했다.



### 왜지?

그전에랑 차이점이라고는...

프로젝트 설정 관련된 (parent, properties, dependencymanagement) 부분은 상위 프로젝트의 Pom.xml에서 작성했고

각 모듈에서는 모듈에서 필요한 의존성만 추가했는데 이게 잘못된것일까?

아니면 사용한 버전이 뭔가 잘못된게 있는가?

버전 의존성 문제라면 이미 그전에 독립 프로젝트로 구성했을때도 문제가 됬어야 할거같은데?

30분넘게 삽질을 하다가 상위 프로젝트에 작성한 pom.xml을 다시한번 확인해보았다.

~(이게 추가된 이후로 이상해진거니까..)~


그전에는 안보였는데 한가지 발견된게 있는데 아래와같이 **POM이 대문자로 있었다.**

### `<type>POM</type>`

혹시나 해서 아래와 같이 소문자로 바꿨더니... 그동안 실패했던 모듈 의존성이 갑자기 다 연결되서 다운받아지는게 아닌가?

![Alt text](/assets/img/dev/trouble-shooting/maven_depedencyManagement.png)

  


사실 이 부분은 더 깊이 파보진 않았다.  
대/소문자 구분하는게 아주 깊은 지식이 필요하다고 생각되지않았고 일단 빨리 동작되는게 보고싶었기때문에.


혹시나 Maven에서 DependencyManagement를 사용하고자 하는분들이 이런 실수로 고생안하시길 바라며..!

~~얼른 예제 만들어봐야겠다.~~

