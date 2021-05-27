---
layout: post
title: JSON & json-simple
subtitle: JSON 소개 및 json-simple을 사용한 Java Example
categories: dev
tags: java
comments: true
---

## JSON이란 ?  

__JavaScript Object Notation__  
데이터를 저장/전송할 때 사용하는 형식  
Javascript에서 객체를 만들 때 사용하는 표현식  

과거에는 ~~(많이)~~ 데이터를 주고 받을 때 XML 포맷을 많이 사용했다고 한다.  
아직도 다양한곳에서 많이 쓰고있기는 한 것 같다.  

스프링 레거시 프로젝트에서도 여전히 설정 파일로는 XML을 사용하고, Maven도 XML, 안드로이드도 예전엔 XML로 리소스를 관리했으며, SAML과 같은 인증 방식도 XML 포맷을 사용한다.  

XML을 보면 잘 갖춰진 포맷으로 보이긴한데 일단 <,> 이 꺽새들이 많으니까 조금 보기 힘들다.  
__그리고 Tag가 <>로 시작해서 </>로 끝나야하므로 불필요하게 데이터 값이 커지는 단점도 있다.__  
일단 설정이 XML로 되어있으면 왠지모를 실망감? 부터 시작하는게 나만의 느낌은 아닐 듯 하다.  

요새는(~~사실 좀 오래되긴했지만~~) XML보다 JSON을 많이 사용하는 것 같다.  
아무래도 2010년대에 Javascript가 비약적으로 발전해서 함께 대세가 된게 아닐까 싶다.  

나는 2017년에 MQTT로 메시지 포맷을 만들때 JSON 포맷을 처음으로 사용했다.  
처음엔 Object, Array 등 표현이 약간 어렵다고 느꼈는데 몇번 사용해보면 사용하기 진짜 쉽다.  
특히 XML을 보는것보다 마음이 편한 느낌인데 아무래도 Key:Value의 구조로 되어있는게 이해하기 좋은 것 같다.  
A는100, B는200, C는300...과 같이 우리가 일상생활에서 사물을 매핑하는것과 동일한 방식을 문자열로 구성하니 읽기 좋은 것 같다.  
~~(지극히 개인적인 생각)~~  

---

## JSON 문법  

Javascript의 객체를 만들 때 사용하는 구조이다보니 모든 데이터의 시작과 끝이 중괄호( { } )로 되어있다.  
Key:Value 구조로 되어있으며 Key는 문자열 타입으로 표현되고 Value는 모든 타입이 가능하다.  
문자열을 표기할 때는 큰따옴표( " " )로 묶어준다.  
배열은 대괄호( [ ] )로 묶어준다.  
데이터는 String, Number, Object, Array, Boolean 및 null로 표현할 수 있다.  

---

## Java에서 JSON 사용법  

Maven Repository에서 찾아보면 Java에서 JSON을 쉽게 사용할 수 있도록 지원해주는 라이브러리가 많이 있다.  
__대표적인게 json-simple, jackson, gson 등이 있다.__  
그 중에서 오늘은 json-simple을 사용한 방법을 소개하고자 한다.  

---

## json-simple  

### 라이브러리 종속성 추가  

[MavenRepository](https://mvnrepository.com/)에서 json-simple을 검색한다.  
maven을 사용할 경우 아래와 같은 종속성을 pom.xml에 추가한다.  
뒤에 설명할 예제는 maven을 사용했고 아래의 종속성을 추가했다.  

```xml
<dependency>
    <groupId>com.googlecode.json-simple</groupId>
    <artifactId>json-simple</artifactId>
    <version>1.1.1</version>
</dependency>
```  

gradle을 사용할 경우는 아래와 같은 종속성을 build.gradle에 추가한다.  

```groovy
implementation group: 'com.googlecode.json-simple', name: 'json-simple', version: '1.1.1'

```

---

### 예제  

첫번째 예제는 JSON 형식으로 작성된 파일을 파싱하여 Java Object로 변환한다.  
JSON파일은 아래와 같이 구성했다.  

```json
{
  "name": "chpark",
  "age": 34,
  "job": "programmer",
  "position": "server",
  "favorite_language": "java",
  "favorite_ide": "intellij",
  "has_skills": {
    "backend": [
      "java", "spring", "netty", "javascript", "node.js"
    ],
    "frontend": [
      "javascript", "react", "typescript"
    ]
  },
  "will_skills": [
    "unity", "flutter", "android", "webflux", "vr"
  ]
}
```  

name, job, position, favorite_language, favorite_ide는 문자열 값이고, age는 숫자, has_skill는 객체, will_skills는 배열로 구성했다.  
has_skills는 객체이므로 또다른 타입의 값을 가질 수 있는데 하위 값으로 배열을 가지고 있다.  

json-simple을 사용하면 아래와 같은 클래스를 사용하여 JSON을 처리할 수 있다.  

```java
// JSON 파싱을 위한 객체
JSONParser jsonParser;
// JSON 객체 
JSONObject jsonObject;
// JSON 배열
JSONArray jsonArray;
```

JSONParser는 파일이나 입력 스트림을 JSON 형식으로 변환해준다.  
__만약 스트림이 JSON 형식을 벗어나면 문법 오류이므로 파싱하지 못하고 예외가 발생한다.__  
예를들어 마지막에 중괄호( } )로 끝나지 않고 다른 문자가 들어간다면 아래와 같은 예외가 발생한다.  

```
Unexpected token END OF FILE
```  

위 예제를 파싱하기 위해서는 먼저 최상단 객체를 JSONObject로 받은 후, 각 하위 값들을 Key를 사용하여 JSONObject로 변환해야 한다.  

파일을 JSONObject로 파싱하는 클래스와, JSON을 Java Object로 변환하는 클래스를 아래와 같이 구현해보았다.  
```java
class JSONFileReader {
    static JSONObject readFile(String filePath) {
        JSONObject object = new JSONObject();
        InputStream inputStream = JSONFileReader.class.getClassLoader().getResourceAsStream(filePath);
        try (InputStreamReader inputStreamReader = new InputStreamReader(inputStream)) {
            JSONParser jsonParser = new JSONParser();
            object = (JSONObject) jsonParser.parse(inputStreamReader);

        } catch (IOException | ParseException e) {
            e.printStackTrace();
        }
        return object;
    }
}
```

```java
public class JSONSample {
    public static void main(String[] args) throws IOException {
        JSONSample jsonSample = new JSONSample();
        // JSON to JavaObject
        jsonSample.jsonFileToJavaObject("json-sample.json");

        // JavaObject to JSON
        jsonSample.javaObjectToJson();
    }

    private void jsonFileToJavaObject(String jsonFile) {
        JSONObject object = JSONFileReader.readFile(jsonFile);
        StringBuilder sb = new StringBuilder();

        String name = (String) object.get("name");
        sb.append("name:<").append(name).append(">\n");

        int age = ((Long)object.get("age")).intValue();
        sb.append("age:<").append(age).append(">\n");

        String job = (String) object.get("job");
        sb.append("job:<").append(job).append(">\n");

        String position = (String) object.get("position");
        sb.append("position:<").append(position).append(">\n");

        JSONObject hasSkill = (JSONObject) object.get("has_skills");
        sb.append("hasSkill:\n\t");
        for (Object skill : hasSkill.keySet()) {
            sb.append("skillType:\n\t\t<");
            JSONArray skillContents = (JSONArray) hasSkill.get(skill);
            for (Object contents : skillContents) {
                sb.append(contents).append(", ");
            }
            sb.replace(sb.lastIndexOf(", "), sb.length(), ">\n");
        }
        sb.append("\n");

        JSONArray willSkils = (JSONArray) object.get("will_skills");
        sb.append("willSkill:<");
        for (Object skill : willSkils) {
           sb.append(skill).append(", ");
        }
        sb.replace(sb.lastIndexOf(", "), sb.length(), ">\n");
        System.out.println(sb.toString());
    }
} 
```  

위 예제의 구성을 설명하면 아래와 같다.  

1. JSONFileReader.readFile 함수를 통해 json파일을 JavaObject로 변환한다.  

2. 파싱된 최상위 object를 name, age, job, position 등등 json파일의 Key값을 통해 파싱한다.  
   JSONObject로부터 파싱된 값은 기본적으로 Object 타입이다. 따라서 문자열, 숫자값은 형변환을 해주어야 한다.  
   age는 숫자 값인데 Integer로 바로 파싱하면 에러가 발생했다.  
   구글링해서 좀 찾다가 Long 타입으로 형 변환 후 Integer로 다시 변환시켜주었는데, 왜 그런지는 다시 찾아봐야 할 것 같다.  
   ~~JSON Number Type이 Long Type인가?~~  

3. has_skills의 값은 Object고 그 Object는 다시 Array 타입으로 구성되어 있다.  
   따라서 has_skills는 Object로 파싱하고, 파싱된 값을 배열로 변환하여 탐색하면서 하위 값들을 파싱한다.  

4. will_skills는 Array로 파싱하여 탐색하면서 하위 값들을 파싱한다.  

5. 결과값 출력을 위해 StringBuilder를 사용했고 출력한 결과는 아래와 같다.  

```json
{
    name:<chpark>
    age:<34>
    job:<programmer>
    position:<server>
    hasSkill:
        skillType:
            <java, spring, netty, javascript, node.js>
        skillType:
            <javascript, react, typescript>
    willSkill:
        <unity, flutter, android, webflux, vr>
}
```

favorite값은 생략했는데 ~~(어차피 예제에 String형태가 대부분이라..)~~ 나머지 결과는 제대로 파싱됨을 확인할 수 있다.  

---

<strong_red>그렇다면 이번엔 반대로 JavaObject를 JSON으로 변환할 수 있을까?</strong_red>  

예제 코드는 아래와 같다.  

```java
public class JSONSample {
// ... import 및 main함수 생략
private void javaObjectToJson() throws IOException {
        // Java Object 생성
        Profile profile = new Profile("tester", 20);
        profile.addJob("programmer");
        profile.addPosition("app");
        profile.addSkill("mobile", "android");
        profile.addSkill("mobile", "ios");
        profile.addSkill("mobile", "flutter");

        profile.addSkill("language", "java");
        profile.addSkill("language", "kotlin");
        profile.addSkill("language", "swift");
        profile.addSkill("language", "dart");

        profile.addWillSkill("python");
        profile.addWillSkill("django");

        profile.addFavoriteLanguage("java");
        profile.addFavoriteIde("vscode");

        // profile Object를 JSON으로 변환
        JSONObject profileObject = new JSONObject();

        profileObject.put("name", profile.getName());
        profileObject.put("age", profile.getAge());
        profileObject.put("job", profile.getJob());
        profileObject.put("position", profile.getPosition());

        profileObject.put("favorite_language", profile.getFavoriteLanguage());
        profileObject.put("favorite_ide", profile.getFavoriteIde());

        JSONObject object = new JSONObject(profile.getHasSkills());
        profileObject.put("has_skills", object);

        profileObject.put("will_skills", profile.getWillSkills());

        String result = profileObject.toJSONString();
        System.out.println(result);

        FileWriter  fileWriter = new FileWriter(
            "json-with-springboot/src/main/resources/json-sample-output.json");
        fileWriter.write(result);
        fileWriter.close();
    }
}
```  

Profile 객체를 하나 생성하여 값들을 임의로 설정했다.  
JSON의 최상위 객체로 profileObject을 생성했다.  
나머지 값들도 생성했고 JSONObject의 put 메서드를 사용해서 profileObject의 하위 값으로 추가했다.  
결과값은 이번엔 파일로 생성했는데, 위의 JSON 파싱에서 사용한 json파일과 동일한 경로에 저장하도록 구현했다.  
결과는 아래와 같이 파일로 작성된다. Pretty 포맷을 사용하지 않았으므로 1줄로 저장되는데 살짝 개행처리 하면 아래와 같다.  
파싱결과는 예상했던대로 나온다.  

```json
{
    "will_skills":[
        "python",
        "django"
    ],
    "favorite_language":"java",
    "name":"tester",
    "has_skills":{
        "mobile":[
            "android","ios","flutter"
        ],
        "language":[
            "java","kotlin","swift","dart"
        ]
    },
    "position":"app",
    "job":"programmer",
    "age":20,
    "favorite_ide":"vscode"
}
```

json-simple은 위와 같이 JSONParser, JSONObject, JSONArray 3개만 사용해도  
왠만한 JSON과 JavaObject의 변환을 쉽게 할 수 있도록 도와준다.  

다음번엔 jackson 라이브러리에 대해 소개와 예제애 대해 포스팅 해봐야겠다.  
사실 Java/Spring을 사용해서 웹 애플리케이션을 개발할 때 json-simple보다는 jackson을 사용한다.  
SpringBoot에서 JSON 처리를 위해 사용하는 MessageConverter가 기본적으로 jackson을 사용하기 때문일 것이라 생각된다.  
(물론 converter를 재구현해서 다른 라이브러리를 사용하는것이 가능하다)  
