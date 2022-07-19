---
title : "[SpringBoot][Chapter5.3] API 작성법 - POST API만들기"
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



## POST API 만들기

POST API는 웹 애플리케이션을 통해 데이터베이스 등의 저장소에 리소스를 저장할 때 사용되는 API이다. 앞에서 살펴본 GET API에서는 URL의 경로나 파라미터에 변수를 넣어 요청을 보냈지만 POST API에서는 저장하고자 하는 리소스나 값을 HTTP 바디(body)에 담아 서버에 전달한다. 그래서 URI가 GET API에 비해 간단하다.

POST API를 만들기 위해 우선 아래와 같이 `PostController`라는 이름의 컨트롤러 클래스를 생성하고 `@RequestMapping` 어노테이션을 이용해 공통 URL을 설정한다.

```java
package com.springboot.api.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/post-api")
public class PostController {
    
}
```



### @RequestMapping으로 구현하기

POST API에서 `@RequestMapping`을 사용하는 방법은 GET API 와 크게 다르지 않다. 요청 처리 메서드를 정의할 때 아래처럼 `method` 요소를 `RequestMethod.POST`로 설정하는 부분을 제외하면 GET API와 동일하다.

```java
 @RequestMapping(value = "/domain", method = RequestMethod.POST)
    public String postExample(){
        return "Hello Post API";
    }
```



### @RequestBody를 활용한 POST 메서드 구현

위 에서는 별도의 리소스를 받지 않고 단지 POST 요청만 받는 메서드를 구현했다. 일반적으로 POST 형식의 요청은 클라이언트가 서버에 리소스를 저장하는 데 사용한다. 그러므로 클라이언트의 요청 트래픽에 값이 포함돼 있다. 즉, POST 요청에서는 리소스를 담기 위해 HTTP Body에 값을 넣어 전송한다. 다음과 같이 Talend API Tester 에서 Body 영역에 값을 입력할 수 있다.

![image](https://user-images.githubusercontent.com/13410737/179670964-ed6dfd89-9501-450d-9b86-17cba1301f38.png){: .align-center}

Body 영역에 작성되는 값은 일정한 형태를 취한다. 일반적으로 JSON(JavaScript Object Notation) 형식으로 전송되며, 이 책에서도 가장 대중적으로 사용되는 JSON 형식으로 값을 주고받을 계정이다. 이렇게 서버에 들어온 요청은 아래와 같이 처리할 수 있다.

```java
    //http://localhost:8080/api/v1/post-api/member
    @PostMapping(value = "/member")
    public String postMember(@RequestBody Map<String, Object> postData){
        StringBuilder sb = new StringBuilder();

        postData.entrySet().forEach(map -> {
            sb.append(map.getKey() + " : " + map.getValue() + "\n");
        });
        return sb.toString();
    }
```

2번 줄을 보면 `@RequestMapping` 대신 `@PostMapping`을 사용했다. 이 어노테이션을 사용하며 `method` 요소를 정의하지 않아도 된다. 그리고 3번 줄에서 `@RequestBody`라는 어노테이션을 사용했는데 ,`@RequestBody`는 HTTP의 Body 내용을 해당 어노테이션이 지정된 객체에 매핑하는 역할을 한다.

> **💡 Tip. JSON이란?**
>
> JSON은 'JavaScript Object Notation'의 줄임말로 자바스크립트의 객체 문법을 따르는 문자 기반의 데이터 포맷이다. 현재는 자바스크립트 외에도 다수의 프로그래밍 환경에서 사용한다. 대체로 네트워크를 통해 데이터를 전달할 때 사용하며, 문자열 형태로 작성되기 때문에 파싱하기도 쉽다는 장점이 있다.



`Map` 객체는 요청을 통해 어떤 값이 들어오게 될지 특정하기 어려울 때 주로 사용한다. 요청 메시지에 들어갈 값이 정해져 있다면 아래와 같이 DTO 객체를 매개변수로 삼아 작성할 수 있다. 

```java
    // http://localhost:8080/api/v1/post-api/member2
    @PostMapping(value = "/member2")
    public String postMemberDto(@RequestBody MemberDto memberDto){
        return memberDto.toString();
    }
```

위와 같이 작성하면 `MemberDto`의 멤버 변수를 요청 메시지의 키와 매핑해 값을 가져온다.

![image](https://user-images.githubusercontent.com/13410737/179670868-ecfc71f2-df0d-4629-b666-8751414342fa.png){: .align-center}
