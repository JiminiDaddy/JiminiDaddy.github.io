---
layout: post
title: Git Fork한 Repository 동기화
subtitle: fork한 repository를 원본 Repository의 최신 코드에 맞게 업데이트하기
categories: dev
tags: git
comments: true
---

## Forked Repository Update  

Github를 쓰다보면 남들이 만들어놓은 오픈소스를 분석하거나 좋은 정보를 자주 보기위해 fork하는 경우가 많다.  
나 역시 많은 개발자분들이 작성한 코드를 fork할 때도 있고, 코드 외의 정보를 자주 찾는 경우에도 fork한다.  
근데 이게 문제가 생길 수 있는게, fork한 시점이후 계속 업데이트가 되고있다면, 내 저장소는 레거시? 가 되버린다.  
다시말해 원본 저장소는 꾸준히 업데이트 되는데, fork받은 내 저장소는 fork받았을 때의 시점에 멈추게 되버리는것이다.  
그래서 찾아보았다. 어떻게 업데이트 할 수 있는지..  
구글링 한번만 해보면 이미 수많은 분들이 작성해놓은게 보이는데 그때그때 검색하지않고 바로 필요할 때 사용하기위해 기록해본다.  

---  

### 현재 내 원격저장소 정보 확인  

가장 먼저 해야 할 일은 원본 원격저장소를 내 원격저장소에 추가해야 한다.  
추가하기전에 현재 내 원격저장소의 정보를 확인해본다.

```command
git remote -v
```

![Alt](/assets/img/dev/git/git_remote_v.png)

---  

### fork한 원본 원격저장소를 내 Working-Directory에 추가  

위 명령을 통해 현재 내 Working-Directory에 연결된 원격저장소가 origin임을 알 수 있다.  
origin은 내 github에서 관리되고 있는 원격저장소다.(fork받은 저장소)  
지금 내가 하려는건 원본 원격저장소의 최신정보를 가져오는것이므로 원본 원격저장소와 연결할 필요가 있다.  
아래와 같은 명령을 통해 원본 원격저장소를 연결할 수 있다.  
(add 뒤에 이름은 뭘 넣어도 상관없다.)  

```command
git remote add upstream ${원본 원격저장소 URI}
git remote -v
```

![Alt](/assets/img/dev/git/git_remote_add_upstream.png)

---  

### Fetch를 통해 원격저장소의 데이터를 로컬저장소로 가져오기  

fetch란 원격저장소의 데이터만 다운로드하고 커밋 위치는 이동시키지 않는다고 한다.  
~~(아직 git에 대해 깊게모름.. 현재 프로젝트에서는 git을 안쓰니 익히기가 좀 어렵네ㅠㅠ)~~  
pull로 땡겨와도 될거같다고 생각했는데 왜 fetch를 쓸까?  
아무래도 원본저장소의 커밋위치가 더 앞서있기때문에 pull을 사용할경우 내 원격저장소의 커밋위치보다 로컬저장소의 커밋위치가 앞으로 나가서 문제가 발생하기 때문일까?  
일단 이렇게 fetch를 하면 원격저장소에 추가된 데이터(파일)들이 내 로컬저장소에 다운로드된다.  
(마치 내가 작업한것처럼)  

```command
git fetch upstream
```  

![Alt](/assets/img/dev/git/git_fetch_upstream.png)

---  

### 병합할 로컬 브랜치를 선택하고, 원격 저장소의 브랜치를 기준으로 내 로컬 저장소의 브랜치를 병합  

위에서 fetch를 사용했으므로 내 Working-Directory에는 추가/변경된 파일들이 존재하지만 커밋위치는 변경되지 않았다.  
~~사실 아직 이게 정확히 이해가안간다. 뭔가 좀 보고 와야할 것 같다.~~  

어찌됬든.. 직접 병합을 해주어 원본저장소와 내 로컬저장소의 커밋위치를 맞춰줘야한다.  
병합할 로컬 브랜치를 선택한다.  
대개 master인데 내가 fork받은 저장소는 main으로 되어있었고, 별도로 브랜치를 설정해주지 않았기때문에 내 브랜치도 main을 따라갔다.  

```command
git checkout main
git merge upstream/main
```  

![Alt](/assets/img/dev/git/git_merge_upstream.png)

사실 fetch와 merge를 각각 진행하지않고 pull로 진행할 수도 있다.  
(pull = fetch + merge)  
구글링을 좀 해보니 보통 fetch + merge 조합을 많이 쓰는 듯 하다.  
아마 pull로 인해 병합과정에서 conflict 발생할 수 있으므로 그 전에 fetch만 해서 비교해보려는게 아닐까?  
~~이상 git알못의 뇌피셜..~~

---  

### 로컬저장소에 병합된 내용을 원격저장소와 동기화  

어찌됐건 내가 가장하고싶은건 내 원격저장소를 최신버전으로 업데이트 하는 것이다.  
내 로컬저장소는 원본저장소와 병합되어 최신버전을 가리키고 있으므로, 이제 원격저장소에 덮어씌우는 일만 남았다.  
push를 사용해서 로컬저장소의 변경이력을 원격저장소로 업데이트한다.  

```command
git push
```

![Alt](/assets/img/dev/git/git_push.png)

---  

예전에 github 관련해서 심플한 책을 쑥 읽고 지나갔는데, 그래서 별로 남은게없나보다.  
이거 쓰는김에 다른 블로그들을 여럿봤는데 git을 내가 너무 쉽게 생각했나보다.  
생각보다 볼게 많은 듯 하다.  
(오늘도 이렇게 공부할것만 더 생기는구나.. 열심히 일하고있는데 자꾸 일 더 들어오는 기분이다ㅠㅠ)  

--- 

### 참조한 블로그  

<https://json.postype.com/post/210431>  
<https://brownbears.tistory.com/466>  
<https://velog.io/@chy0428/git-fork%ED%95%9C-repository-%EB%8F%99%EA%B8%B0%ED%99%94>  
<https://blog.outsider.ne.kr/866>  