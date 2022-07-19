---
title : "[SpringBoot][Chapter5.4] API 작성법 - PUT API만들기"
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



## PUT API 만들기

PUT API는 웹 애플리케이션 서버를 통해 데이터베이스 같은 저장소에 존재하는 리소스 값을 업데이트하는 데 사용한다. POST API와 비교하면 요청을 받아 실제 데이터베이스에 반영하는 과정(서비스 로직)에서 차이가 있지만 컨트롤러 클래스를 구현하는 방법은 POST API와 거의 동일하다. 리소스를 서버에 전달하기 위해 HTTP Body를 활용해야 하기 때문이다.
먼저 아래와 같이 `PutController `라는 컨트롤러 클래스를 작성한다.



```java
package com.springboot.api.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("api/v1/put-api")
public class PutController {
    
}
```



### @RequestBody 를 활용한 PUT 메서드 구현

PUT API는 POST 메서드와 마찬가지로 값을 HTTP Body에 담아 전달한다. 서버에서는 이 값을 받기 위해 아래와 같이 `@RequestBody`를 사용한다.

```java
    // http://localhost:8080/api/v1/put-api/member
    @PutMapping(value = "/member")
    public String putMember(@RequestBody Map<String, Object> putData){
        StringBuilder sb = new StringBuilder();

        putData.entrySet().forEach(map -> {
            sb.append(map.getKey() + " : " + map.getValue() + "\n");
        });

        return sb.toString();
    }
```



서버에 어떤 값이 들어올지 모르는 경우에는 Map 객체를 활용해 값을 받을 수 있다. 대부분의 경우 API를 개발한 쪽에서 작성한 명세(specification)를 웹 사이트를 통해 클라이언트나 사용자에게 올바른 사용법을 안내한다. 만약 서버에 들어오는 요청에 담겨 있는 값이 정해져 있는 경우에는 아래와 같이 DTO 객체를 활용해 구현한다.

```java
    // http://localhost:8080/api/v1/put-api/member1
    @PutMapping(value = "member1")
    public String putMemberDto1(@RequestBody MemberDto memberDto){
        return memberDto.toString();
    }
    
    // http://localhost:8080/api/v1/put-api/member2
    @PutMapping(value = "member2")
    public MemberDto putMemberDto2(@RequestBody MemberDto memberDto){
        return memberDto;
    }
```



위의 예쩨는 2개의 메서드로 구성돼 있다. 첫 번째 메서드인 `putMemberDto1`은 리턴 값이 String 타입이고, 두 번째 메서드인 `putMemberDto2`는 DTO객체 타입이다. 이전 예제엔서는 `String`타입만 소개했는데, 여기서는 DTO 객체를 반환할 때는 어떤 차이가 있는지 보겠다.

먼저 스프링 부트 프로젝트를 가동시킨 후 첫 번째 메서드를 테스트하기 위해 Talend API Tester 에 아래와 같이 작성하고 요청을 보낸다.

![image](https://user-images.githubusercontent.com/13410737/179672634-4c054b73-4b18-4e8c-8e3d-0ec095bb3422.png){: .align-center}



이 요청을 보내면 `String` 타입으로 값을 전달받게 되며, 4번째 줄에 작성한대로 리턴 값으로 설정한 DTO 객체의 `toString `메서드 결괏값이 출력된다.

![image](https://user-images.githubusercontent.com/13410737/179672864-f5ea93f6-6f81-4e5e-bed9-e862edc01d42.png){: .align-center}



출력 결과를 보면 보낸 요청 내용이 그래도 결괏값으로 전달된 것을 볼 수 있다. `toString` 메서드로 인해 나름의 형식을 갖춰져 전달됐지만 HEADERS 항목의 `content-type` 을 보면 `'text/plain'`으로서 결괏값으로 일반 문자열이 전달됐음을 확인할 수 있다.

이번에는 동일한 값을 담고 URL만 변경해서 두 번째 메서드에 대한 테스트를 진행해보자. 아래와 같이 Talend API Tester를 설정한다.

![image](https://user-images.githubusercontent.com/13410737/179673283-bdc9db23-d92d-4842-ae44-d8d16c0b227d.png){: .align-center}



우측의 BODY 값은 Talend API가 정렬해 출력한 결괏값으로서 실제로 형식만 유지한 채 전달된다. 또한 HEADERS 영역의 `Content-type` 항목도  `member1`메서드로 전달받은 값은 `'test/plain'`이었던 반면 지금은 `'application/json'` 형식으로 전달된 것을 확인할 수 있다. 앞 장에서 컨트롤러를 작성할 때 언급한 것처럼 `@ResponseBody` 어노테이션은 자동으로 값을 JSON과 같은 형식으로 변환해서 전달하는 역할을 수행한다.



### @ResponseEntity를 활용한 PUT 메서드 구현

스프링 프레임워크에는 `HttpEntity`라는 클래스가 있다. `HttpEntity`는 다음과 같이 헤더(Header)와 Body로 구성된 HTTP 요청과 응답을 구성하는 역할을 수행한다.

```java
// HttpEntity 클래스 일부 발췌
public class HttpEntity<T> {
    public static final HttpEntity<?> EMPTY = new HttpEntity();
    private final HttpHeaders headers;
    @Nullable
    private final T body;
    
    ...
}
```



`RequestEntity`와 `ResponseEntity`는 `HttpEntity`를 상속 받아 구현한 클래스이다. 그중 `ResponseEntity`는 서버에 들어온 요청에 대해 응답 데이터를 구성해서 전달할 수 있게 한다. 다음과 같이 `ResponseEntity`는 `HttpEntity`로부터 `HttpHeader`와 Body를 가지고 자체적으로 `HttpStatus`를 구현한다.



```java
public class ResponseEntity<T> extends HttpEntity<T> {
    private final Object status;

    ..생략..
}
```



이 클래스를 활용하면 응답 코드 변경은 물론 Header와 Body를 더욱 쉽게 구성할 수 있다. 이 클래스는 PUT 메서드를 구현하는 이번 절에서 소개하고 있지만 다른 메서드에서도 모두 사용할 수 있는 클래스이다.

다음은 기존 예제 메서드의 리턴 타입에 `ResponseEntity`를 적용한 예이다.

```java
    // http://localhost:8080/api/v1/put-api/member3
    @PutMapping(value = "member3")
    public ResponseEntity<MemberDto> postMemberDto3(@RequestBody MemberDto memberDto){
        return ResponseEntity
                .status(HttpStatus.ACCEPTED)
                .body(memberDto);
    }
```



예제에서는 메서드의 리턴 타입을 ResponseEntity로 설정하고 리턴 값을 만든다. `status`에 넣을 수 있는 값은 다양한다, 예제에서 사용한 HttpStatus.ACCEPTED`는 응답 코드 202를 가지고 있다. 즉, 이 메서드를 대상으로 요청을 수행하면 아래와 값이 응답 코드가 202로 변경된다.

![image](https://user-images.githubusercontent.com/13410737/179675259-92559ce5-441e-4c00-86e7-f8df9f19ab84.png){: .align-center}





