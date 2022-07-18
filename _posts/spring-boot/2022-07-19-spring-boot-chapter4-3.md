---
title : "[SpringBoot][Chapter4.3] 개발하기 - Hello World 출력하기"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij]
toc: true
toc_sticky : true
published : true
date : 2022-07-19
last_modified_at : 2022-07-19
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## Hello World 출력하기

본격적인 애플리케이션 개발에 앞서 'Hello World!'를 출력하는 애플리케이션을 만들어 보며 스프링 부트에 입문해 보자.



### 컨트롤러 작성하기

먼저 앞에서 생성한 프로젝트에 패키지를 생성해야 한다. 이 예제에서는 레이어드 아키텍처에 맞춰 도메인 구분 없이 패키지를 구성하겠다. `com.springboot.hello` 패키지에 마우스 오른쪽 버튼을 클릭한 수 [New] → [Package]를 차례로 선택해 `'controller'` 라는 이름의 하위 패키지를 생성한다. 그리고 나서 `'controller'` 패키지에 마우스 오른쪽 버튼을 클릭한 후 [New] → [Java Class]를 클릭하고  `HelloController`라는 이름의 컨트롤러를 생성한다.

![image](https://user-images.githubusercontent.com/13410737/179554847-098dd70e-7a2d-4777-a186-e49552f01248.png){: .align-center}

컨트롤러에 포함된 로직에서는 애플리케이션의 사용자 또는 클라이언트가 입력한 값에 대한 응답을 수행한다. 특별한 경우를 제외한 모든 요청은 컨트롤러를 통해 진행돼야 한다. 이번 예제는 컨트롤러 내부에서 모든 로직을 처리했지만 데이터를 다루거나 별도의 로직을 처리해야 하는 경우네는 서비스 또는 데이터 액세스 레이어까지 요청을 전달하는 경우가 일반적이다. `HelloController` 클래스에는 아래와 같은 코드를 작성한다.

```java
package com.springboot.hello.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {
    @RequestMapping("/hello")
    public String hello(){
        return "Hello World";
    }
}
```



### 애플리케이션 실행하기

애플리케이션을 실행하는 방법은 다른 자바 프로젝트와 같다. 아래와 같이 인텔리제이 IDEA 우측 상단부에 위치한 실행 버튼을 누르면 애플리케이션이 실행된다.

![image](https://user-images.githubusercontent.com/13410737/179557981-ba50361b-9141-4a77-923a-0c7935f4e652.png){: .align-center}

애플리에키션이 정상적으로 실행되면 IDE 하단의 콘솔(Console) 캡에서 아래와 같이 실행 로그가 출력된다.

![image](https://user-images.githubusercontent.com/13410737/179558256-6956d594-125d-45bc-ae3f-a29921ee7b69.png){: .align-center}

8080번 포트를 통해 웹 서버가 열린 것을 로그의 세번째 줄에서 확인할 수 있다.

------

o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)

------

> 💡 Tip.
>
> 스프링 부트에서는 기본적으로 8080번 포트를 통해 웹 애플리케이션이 실행된다. 필요에 따라 포트를 변경해야 한다면 아래와 같이 src/main/resources/application.properties 파일에서 변경할 수 있다.
>
> ![image](https://user-images.githubusercontent.com/13410737/179559007-31f2495d-3049-4deb-80f1-2278f3844dea.png){: .align-center}
>
> 포트를 변경하고 다시 애플리케이션을 실행하면 로그에서 변경된 포트 번호로 출력되는 것을 확인할 수 있다.



### 웹 브라우저를 통한 동작 테스트

웹 브라우저로 스프링 부트가 설정한 URL에 접속하면 간단하게 실행 결과를 확인할 수 있다. 아래와 같이 웹 브라우저의 주소창에 `'http://localhost:8080/hello'` 를 입력하면 Hello World가 출력되는 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/13410737/179559637-f37e42ba-f377-4f47-a866-ddefef0be9cb.png){: .align-center}



### Talend API Tester를 통한 동작 테스트

웹 브라우저를 통한 동작 테스트는 간편하지만 상세한 응답을 확인할 수 없다는 단점이 있다. 구글 크롬의 확장 프로그램인 Talend API Tester를 사용하면 이 같은 문제를 해결할 수 있다. 크롬 브라우저의 주소창에 `'https://chrome.google.com/webstore/category/extensions?hl=ko'`에서  'Talend API Tester - Free Edition'을 찾아  설치한다.

![image](https://user-images.githubusercontent.com/13410737/179560505-808fab4b-6424-4e47-b86f-3987fcf1b767.png){: .align-center}

Talend API Tester 는 HTTP 통신을 테스트하는 프로그램이다. GET, POST, PUT, DELETE 등의 다양한 HTTP 메서들를 설정하고 쿼리(query) 와 파라미터(parameter)를 담아 요청을 보낼 수 있다. 크롬 브라우저에서 Talend API Tester를 실행하면 아래와 같은 화면이 표시된다.

![image](https://user-images.githubusercontent.com/13410737/179560913-eb448aae-348c-4ebf-ba74-a8e6f5fd4a77.png){: .align-center}

이 화면에서 HTTP 요청을 보내려는 경로와 메서드를 설정하고 [Send] 버튼을 클릭해 요청을 보내면 같은 화면의 아래 부분에 위치한 'Response' 화면에 결괏값이 출력된다. 한 가지 주의해야 할 점은 URL 입력란에 'https'가 기본값으로 설정돼 있는데 이를 http로 변경해야 한다는 점이다.

![image](https://user-images.githubusercontent.com/13410737/179561267-6f88fc7f-dea7-40ca-93ed-88c752c38421.png){: .align-center}

위와 같이 설정한 후 테스트를 진행하면 아래와 같은 응답 화면이 출력된다. 앞서 웹 브라우저에서 확인했을 때와 마찬가지로 'Hello World'가 정상적으로 출력되는 모습을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/13410737/179562559-0720131c-4eca-4270-aa79-106dd9b42225.png){: .align-center}

Talend API Tester의 장점은 HTTP 헤더를 볼 수 있다는 점이다. REST 통신에서는 Body 값뿐만 아니라 헤더에도 값을 추가해서 요청에 필요한 데이터를 담아 보내는 경우가 많다. 이와 관련된 자세한 사항은 시큐리티(Spring Security)를 다루는 13장에서 자세히 다룬다.
