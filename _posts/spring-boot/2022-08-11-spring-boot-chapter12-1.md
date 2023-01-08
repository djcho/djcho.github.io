---
title : "[SpringBoot][Chapter12.1] 서버 간 통신 - RestTemplate 이란?"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, RestTemplate]
toc: true
toc_sticky : true
published : true
date : 2022-08-11
last_modified_at : 2022-08-11
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





 최근에 개발되는 서비스들은 마이크로서비스 아키텍처(MSA)를 주로 채택하고 있다. MSA는 말 그대로 애플리케이션이 가지고 있는 기능(서비스)이 하나의 비즈니스 범위만 가지는 형태이다. 각 애플리케이션은 자신이 가진 기능을 API로 외부에 노출하고, 다른 서버가 그러한 API를 호출해서 사용할 수 있게 구성되므로 각 서버가 다른 서버의 클라이언트가 되는 경우도 많다. 이번 장에서는 이러한 트랜드에 맞춰 다른 서버로 웹 요청을 보내고 응답을 받을 수 있게 도와주는 `RestTemplate`과 `WebClient`에 대해 살펴보겠다.



# RestTemplate 이란?

`RestTemplate`은 스프링에서 HTTP 통신 기능을 손쉽게 사용하도록 설계된 템플릿이다. HTTP서버와 통신을 단순화한 이 템플릿을 이용하면 RESTful 원칙을 따르는 서비스를 편리하게 만들 수 있다. `RestTemplate` 은 기본적으로 동기 방식으로 처리되며, 비동기 방식으로 사용하고 싶을 경우 `AsyncRestTemplate`을 사용하면 된다. 다만 `RestTemplate`은 현업에서는 많이 쓰이나 지원 중단(deprecated)된 상태라서 향후 빈번하게 쓰이게 될 `WebClient`방식도 함께 알아둘 것을 권장한다.

`RestTemplate`은 다음과 같은 특징을 가지고 있다.

- HTTP 프로토콜의 메서드에 맞는 여러 메서드를 제공한다.
- RESTful 형식을 갖춘 템플릿이다.
- HTTP 요청 후 JSON, XML, 문자열 등의 다양한 형식으로 응답을 받을 수 있다.
- 블로킹(blocking) I/O 기반의 동기 방식을 사용한다.
- 다른 API를 호출할 때 HTTP 헤더에 다양한 값을 설정할 수 있다.



## RestTemplate의 동작 원리

`RestTemplate`의 동작을 도식화하면 아래와 같다.

<br>

![image](https://user-images.githubusercontent.com/13410737/184143753-55f050ca-988e-4ad5-a18b-71bd78c3f553.png){: .align-center}

<br>

여기서 애플리케이션은 우리가 직접 작성하는 애플리케이션 코드 구현부를 의미한다. 애플리케이션에서는 `RestTemplate`을 선언하고 URI와 HTTP 메서드, Body 등을 설정한다.

그리고 외부 API로 요청을 보내게 되면 `RestTemplate`에서 `HttpMessageConverter`를 통해 `RequestEntity`를 요청 메시지로 변환한다.

`RestTemplate`에서는 변환된 요청 메시지를 `ClientHttpRequestFactory`를 통해 `ClientHttpRequest`로 가져온 후 외부 API로 요청을 보낸다.

외부에서 요청에 대한 응답을 받으면 `RestTemplate`은 `ResponseErrorHandler`로 오류를 확인하고, 오류가 있다면 `ClientHttpResponse`에서 응답 데이터를 처리한다.

받은 응답 데이터가 정상적이라면 다시 한번 `HttpMessageConverter`를 거쳐 자바 객체로 변환해서 애플리케이션으로 반환한다.



## RestTemplate의 대표적인 메서드

`RestTemplate`에서는 더욱 편리하게 외부 API로 요청을 보낼 수 있도록 다음과 같은 다양한 메서드를 제공한다.

<br>

| 메서드            | HTTP 형태 | 설명                                                         |
| ----------------- | --------- | ------------------------------------------------------------ |
| `getForObject`    | GET       | GET 형식으로 요청한 결과를 객체로 반환                       |
| `getForEntity`    | GET       | GET 형식으로 요청한 결과를 `ResponseEntity`결과로 반환       |
| `postForLocation` | POST      | POST 형식으로 요청한 결과를 헤더에 저장된 URI로 반환         |
| `postForObject`   | POST      | POST 형식으로 요청한 결과를 객체로 반환                      |
| `posetForEntity`  | POST      | POST 형식으로 요청한 결과를 `ResponseEntity` 형식으로 반환   |
| `delete`          | DELETE    | DELETE 형식으로 요청                                         |
| `put`             | PUT       | PUT 형식으로 요청                                            |
| `patchForObject`  | PATCH     | PATCH 형식으로 요청한 결과를 객체로 반환                     |
| `optionsForAllow` | OPTION    | HTTP 헤더를 임의로 추가할 수 있고, 어떤 메서드 형식에서도 사용할 수 있음 |
| `exchange`        | any       | HTTP 헤더를 임의로 추가할 수 있고, 어떤 메서드 형식에서도 사용할 수 있음 |
| `execute`         | any       | 요청과 응답에 대한 콜백을 수정                               |

