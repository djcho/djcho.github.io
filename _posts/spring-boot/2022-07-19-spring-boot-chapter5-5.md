---
title : "[SpringBoot][Chapter5.5] API 작성법 - DELETE API만들기"
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



## DELETE API 만들기

DELETE API는 웹 애플리케이션 서버를 거쳐 데이터베이스 등의 저장소에 있는 리소스를 삭제할 때 사용한다. 서버에서는 클라이언트로부터 리소르를 식별할 수 있는 값을 받아 데이터베이스나 캐시에 있는 리소스를 조회하고 삭제하는 역할을 수행한다. 이때 컨트롤러를 통해 값을 받는 단계에서는 간단한 값을 받기 때문에 GET메서드와 같이 URI에 값을 넣어 요청을 받는 형식으로 구현된다.

먼저 아래와 같이 컨트롤러 클래스를 작성한다. 이 컨트롤러에서 작성하는 메서드는 GET 메서드를 작성하는 방법과 동일하므로 간단하게 소개한다.

```java
package com.springboot.api.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/delete-api")
public class DeleteController {
    
}
```



### @PathVariable과 @RequestParam을 활용한 DELETE 메서드 구현

`@PathVariable`을 이용하면 URI에 포함된 값을 받아 로직을 처리할 수 있다.

```java
    // http://localhost:8080/api/v1/delete-api/{String 값}
    @DeleteMapping(value = "/{variable}")
    public String DeleteVariable(@PathVariable String variable){
        return variable;
    }
```



`@DeleteMapping` 어노테이션에 정의한 `value`의 이름과 메서드의 매개변수 이름을 동일하게 설정해야 삭제할 값이 주입된다. 또는 `@RequestParam` 어노테이션을 통해 쿼리스트링 값도 받을 수 있다.



```java
    // http://localhost:8080/api/v1/delete-api/request1?email=value
    @DeleteMapping(value = "/request1")
    public String getRequestParam1(@RequestParam String email){
        return "e-mail : " + email;
```

![image](https://user-images.githubusercontent.com/13410737/179677059-c0294b96-ea7b-4a7e-99ac-a5f08bc801a4.png){: .align-center}
