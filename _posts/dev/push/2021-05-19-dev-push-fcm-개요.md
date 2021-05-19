---
layout: post
title: FCM?
subtitle: FCM이 무엇이고 어떻게 사용하는가?
categories: dev
tags: push
comments: true
---

# FCM(Firebase Cloud Messaging) 이란?  
Target에 Push를 보내는 서비스  


<hr>

### 일반적인 데이터 통신과 푸시 서비스의 비교  
일반적인 데이터 통신(TCP나 Websocket과 같은 프로토콜을 사용하여 Client간 메시지 송/수신)의 경우  
양쪽 모두 소켓으로 연결이 된 상태야만 가능하다.  
또한 데이터를 주고받을 Target이 명확히 정해져있으므로 Client끼리의 강한 결합이 발생한다.  
PushService는 Client간에 Broker가 존재하므로 더이상 Client끼리의 결합이 사라진다.  
(모든 Client는 Broker에 의존하게되고 전송한 메시지가 언제 어떤 Client에게 도착했는지에 대해 고려하지 않는다.)  

<hr>

### 푸시 서비스를 이용한 메시지 발행/구독 개요    
Publisher(메시지 발행자)는 Topic으로 메시지를 발행한다.  
즉 메시지 수신 대상이 특정 Client가 아니라 Topic이다.  
발행한 메시지는 해당 Topic을 구독하고 있는 Subscriber(메시지 구독자)에게만 전송된다.  
Subscriber입장에서 메시지를 누가 전송했느냐가 아니라 구독중인 어떤 Topic으로부터 메시지가 수신되었는지가 관심있을 뿐이다.  

<hr>

아래는 FCM을 이용하여 Mobile과 App-Server간 푸시 서비스의 동작 과정은 크게 2가지로 나뉜다.  
1. Token 발급  
2. 메시지 전송  

아래는 푸시 서비스의 동작과정을 Sequence Diagram으로 그려봤다.  
* FCM 푸시 서비스를 이용하려는 주체(App-Server)는 Firebase-console에서 미리 Firebase-API-Key를 발급받아야 한다.  

### FCM PushService를 위한 Key 발급 과정   
<div class="mermaid"> 
sequenceDiagram;
    Mobile ->> FCM-Server: 등록 요청(with SenderId)
    FCM-Server -->> FCM-Server: Register-Token 생성(by SenderId)
    FCM-Server ->> Mobile: Register-Token 전송
    Mobile ->> App-Server: Register-Token 전송
    App-Server -->> App-Server: Register-Token 저장
</div>

### FCM PushService를 이용해 메시지 전달 과정  
<div class="mermaid"> 
sequenceDiagram;
    App-Server ->> FCM-Server: Http Post방식으로 메시지 전송<br/>(with Register-Token, Firebase-API-Key)
    FCM-Server -->> FCM-Server: Register-Token 으로부터 Target 식별<br/>(Platform, Application)
    FCM-Server ->> Mobile: 메시지 전송
</div>  

<hr>

위 SequenceDiagram의 내용을 풀어보면 아래와 같다.  
1. Target(Mobile)은 고유 ID(SenderID)를 이용해 FCM-Server에 등록을 요청한다.  
2. FCM Server는 Target에게 서비스를 제공하기위해 고유 ID를 통해 Register-Token을 발급한다.  
3. Target은 App-Server에게 발급받은 Register-Token를 전달하고, App-Server는 이 Token을 저장하여 관리한다.  
4. App-Server는 저장한 Register-Token과 Firebase로부터 발급받은 API-Key, 메시지를 구성하며,  
   Http Post방식을 사용하여 FCM서버로 메시지를 전송한다.  
5. 메시지를 수신받은 FCM-Server는 Register-Token으로부터 Target의 정보(Platform, Application)을 확인하여 Target으로 메시지를 전송한다.  

<hr>

FCM은 크게 2가지 방식을 지원한다.  
__Down-Stream__  
App-Server to Target의 단방향 통신 지원  
Server에서 Client에게 일방적으로 알림을 제공하는 경우에 사용할 수 있다.  
SDK 또는 Http 프로토콜을 사용해서 메시지를 전송한다.  

__Up-Stream__  
Target to Target의 양방향 통신 지원  
카카오톡과 같은 메신저와 같은 경우 양방향 통신이 되어야 한다.  
XMPP 프로토콜을 사용해서 메시지를 전송한다.
<hr>

### FCM 메시지 구조 예제  
Body의 메시지 구조를 제대로 구현하지 않으면 오류가 발생한다.  
메시지는 notification과 data 2가지로 구분되는데 보통 혼용하여 쓴다고 한다.  
notification의 경우 하위 Key/Value이 미리 정의되어있으며, data경우는 사용자 정의된 Key/Value를 사용한다.  
__Header__  
```
https://fcm.googleapis.com/fcm/send
Content-Type:application/json
Authorization:key=SERVER_KEY
```  

__Body__
```json
{
   "to": "/topics/JavaSampleApproach",
   "notification": {
      "title": "TITLE",
      "body": "BODY"
   },
   "data": {
      "Key-1": "VALUE 1",
      "Key-2": "VALUE 2"
   },
}
```  

<hr>

SpringBoot로 푸시 서비스를 이용하기위한 간단한 AppServer 샘플을 구현해보려고 자료좀 찾아보았는데
아래 포스팅이 잘 설명되어있어 따라하면 간단하게 메시지를 푸시하는 앱 서버를 구현할 수 있다.  
[FCM Sample - SpringBoot](https://grokonez.com/spring-framework/spring-boot/firebase-cloud-messaging-server-spring-to-push-notification-example-spring-boot)  

앱 서버에서 푸시 서비스를 이용하기위해서는 몇 가지 사전 작업이 필요하다.  
첫번째로 프로젝트가 생성되어 있어야한다.  
아래 링크를 통해 Firebase 프로젝트 페이지로 이동한다.  
[Firebase Project](https://console.firebase.google.com/)  

프로젝트를 추가한다.  
![Alt](/assets/img/dev/push/dev-pns-fcm-create-project-step1.png)  
  

프로젝트 이름을 설정하고, 이후 "계속" 버튼을 누르고 몇 단계 거치면 프로젝트가 생성된다.  
![Alt](/assets/img/dev/push/dev-pns-fcm-create-project-step2.png)  

앱 서버에서 FCM 서버로 메시지를 전송할 때 2가지 정보가 필요한데, 하나는 API-Key이며 하나는 Server Url이다.  
API-Key는 아래와 같이 프로젝트 설정화면에서 "클라우드 메시징" 메뉴를 선택하면 확인할 수 있다.  
![Alt](/assets/img/dev/push/dev-pns-fcm-create-project-step3.png)  

ServerUrl은 "https://fcm.googleapis.com/fcm/send" 로 설정하면 된다.  

<hr>

앱 서버에서 FCM 서버로 메시지를 전송하는 부분의 코드는 아래와 같다.  

```java
@Service
public class FCMService {
    private static final Logger logger = LoggerFactory.getLogger(FCMService.class);

    private static final String FIREBASE_SERVER_KEY = "서버 Key";
    private static final String FIREBASE_API_URL = "https://fcm.googleapis.com/fcm/send";

    public String send(FCMRequestDto requestDto) {
        RestTemplate restTemplate = new RestTemplate();
        ArrayList<ClientHttpRequestInterceptor> interceptors = new ArrayList<>();
        interceptors.add(new HeaderRequestInterceptor("Authorization", "key=" + FIREBASE_SERVER_KEY));
        interceptors.add(new HeaderRequestInterceptor("Content-Type", "application/json"));
        restTemplate.setInterceptors(interceptors);

        String firebaseResponse = restTemplate.postForObject(FIREBASE_API_URL, requestDto.toString(), String.class);
        return firebaseResponse;
    }
}
```

전송할 메시지는 아래와 같은 구조로 notification과 data를 모두 구현했다.  
```java
    @Override
    public String toString() {
        JsonObject jsonObject = new JsonObject();
        jsonObject.addProperty("to", "/topics/test");
        jsonObject.addProperty("priority", "high");

        JsonObject notification = new JsonObject();
        notification.addProperty("title", title);
        notification.addProperty("body", body);

        JsonObject data = new JsonObject();
        data.addProperty("key1", "val 1");
        data.addProperty("key2", "val 2");

        jsonObject.add("notification", notification);
        jsonObject.add("data", data);

        return jsonObject.toString();
    }
```

내가 코드를 구현한 목적은 FCM을 사용해서 푸시 메시지가 전송되는지를 눈으로 확인해보고 싶었다.  
__내 예제 코드는 MVC로 구현했는데 사실 이러한 푸시 메시징은 비동기로 구현하는게 맞다.__  
클라이언트의 요청을 앱 서버가받고, 앱 서버가 FCM 서버로 메시지를 전송하는 구조로 구현을 했는데  
클라이언트의 입장에서는 굳이 앱 서버가 FCM 서버로 메시지를 전달하는것까지 기다려야할까?  
<strong_red>(클라이언트의 요청과 앱서버가 FCM서버로 메시지 전송하는게 하나의 트랙잭션으로 구성될 필요가 없다.)</strong_red>  
~~조만간 코틀린 공부를 시작할 예정인데 코틀린이 손에 조금 익으면 코틀린 + 웹플럭스를 이용해서 웹플럭스 공부를 해봐야겠다.~~  

<hr>

과거에 MQTT 프로토콜을 사용한 Push Server 프로젝트를 수행한 적이 있었는데, 이번에 FCM을 알아보면서 푸시 서비스 아키텍쳐는 거의 유사하다는것을 느꼈다.  
Message Broker를 사용함으로써 Client간의 결합을 없앤 구조는 정말 잘 만든 것 같다.  
MQTT에는 QOS, Will과 같은 처리가 잘 되어있어 푸시 서비스를 구현하기 좋았다고 느꼈는데 FCM에서는 이런 부분들을 어떻게 구현하고 있는지 확인해봐야겠다.  
~~(개발 공부라는건 끝이 없는 것 같다. 하나를 배우면 10개가 새로 생겨나는 느낌.. )~~  