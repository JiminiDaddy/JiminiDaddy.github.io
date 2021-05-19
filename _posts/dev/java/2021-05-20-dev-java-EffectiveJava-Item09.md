---
layout: post
title: [EffectiveJava3 Item09]
subtitie: try-finally보다는 try-with-resources를 구하라
categories: dev
tags: java
comments: true
---

# Item9. try-finally보다는 try-with-resources를 구하라  

재작년인가 이펙티브 자바 책을 처음 읽었을 때 이부분은 실제로 코딩을 해보면서 공부했던 부분이라 기억이 잘 남는다.  
물론 이번 장의 내용이 Item08과 겹치는 부분이 있기도 하고 큰 내용이 없어 사실 크게 정리할 게 없는 것 같다.  

JDK6 까지는 자바의 예외 처리는 아래와 같이 try - catch - finally 구문을 사용했다.  
```java

public class TryFinally {
    public static String firstLineOfFile(String path) throws IOException {
        BufferedReader br = new BufferedReader(new FileReader(path)) ;
        try {
            return br.readLine();
        } finally {
            br.close();
        }
    }

    public static void copy(String src, String dest) throws IOException {
        final int BUFFER_SIZE = 1024;
        InputStream in = new FileInputStream(src);
        try {
            OutputStream out = new FileOutputStream(dest);
            try {
                byte[] buf = new byte[BUFFER_SIZE];
                int n;
                while ((n = in.read(buf)) >= 0) {
                   out.write(buf, 0, n);
                }
            } catch (IOException e) {
                e.printStackTrace();
            } finally {
                out.close();
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            in.close();
        }
    }
}
```  
위 1번째 예제는 단순히 FileReader를 통해 읽은 파일을 Stream으로 변환하여 1줄 읽고 종료하는 기능을 구현했다.  

<strong_black>만약 readLine과 close 모두에서 예외가 발생한다면?</strong_black>  

개발자는 아마 readLine의 실패 원인을 먼저 분석하고 싶어질 것이다.  
(readLine이 먼저 실패했고, 이후에 close가 실패했으므로 첫번째 예외가 발생한 원인이 당연히 궁금하지 않을까?)  
하지만 위 경우 readLine의 예외는 스택 트레이스에서 가려진다.  
왜냐하면 finally 구문에서 마지막으로 예외가 발생했기때문에 이전 예외가 덮어지는 것이다.  
이런 경우 디버깅이 매우 어려워질 수 있다.  

위 2번째 예제는 FileInputStream으로부터 문자열을 읽은 뒤, FileOutputStream으로 내보내는 기능을 구현했는데  
InputStream과 OutputStream에서 각각 예외가 발생할 수 있으므로 이중 try-fainally 구문이 생겨났다.  
대충 눈으로 보기에도 굉장히 읽기 불편하다.  

<strong_red>Java7에선 이러한 문제점을 해결하기위해 try-with-resources가 등장했다.</strong_red>  
try-with-resources는 try 구문이 끝나면 try 구문에서 생성한 자원을 자동으로 반환해준다.  
에를들어 BufferedReader의 close를 개발자가 코드로 구현하는게 아니라 자동으로 try절에서 수행해준다.  

하지만 프로그래밍은 마법이 아니다. 자동으로 close가 호출되기 위해서는 한 가지 사전 작업이 필요하다.  

<strong_red>try절에서 사용될 자원은 반드시 AutoClosable 인터페이스를 구현해야 한다.</strong_red>  

위 try-finally 예제를 try-with-resources로 구현하면 아래와 같이 깔끔해진다.  
```java

public class TryWithResources {
    public static String firstLineOfFile(String path, String defaultValue) {
        try (BufferedReader br = new BufferedReader(new FileReader(path))) {
            return br.readLine();
        } catch (IOException e) {
            e.printStackTrace();
            return defaultValue;
        }
    }

    public static void copy(String src, String dest) {
        final int BUFSIZE = 1024;
        try(InputStream in = new FileInputStream(src); 
            OutputStream out = new FileOutputStream(dest)) {
            int n;
            byte[] buf = new byte[BUFSIZE];
            while ((n = in.read(buf)) >= 0) {
                out.write(buf, 0, n);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
```  

try-with-resources에서 readLine, close 모두 예외가 발생하면 close는 내부에 숨겨져있으므로 예외가 드러나지 않는다.  
따라서 개발자는 readLine에서 발생한 예외를 확인할 수 있게 된다.  

__다른것보다 코드가 훨씬 깔끔해져 실수할 확률이 줄어들 것 같다.__  
실무 코드에서 굉장히 지저분한게 많은데 프로젝트에서 사용하는 자바 버전을 올릴 수 있게되면  
이런 부분들 리팩터링을 진행해야겠다.  