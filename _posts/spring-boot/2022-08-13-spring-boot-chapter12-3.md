---
title : "[SpringBoot][Chapter12.3] 서버 간 통신 - WebClient란?"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, WebClient]
toc: true
toc_sticky : true
published : true
date : 2022-08-13
last_modified_at : 2022-08-13
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





# WebClient 란?

일반적으로 실제 운영환경에 적용되는 애플리케이션은 정식 버전으로 출시된 스프링 부트의 버전보다 낮은 경우가 많다. 그렇기 때문에 `RestTemplate`을 많이 사용하고 있다. 하지만 최신 버전에서는 `RestTemplate`이 지원 중단되어 `WebClient`를 사용할 것을 권고하고 있다. 이러한 흐름에 맞춰 현재 빈번히 가용되고 있는 `RestTemplate`과 앞으로 많이 사용될 `WebClient`를 모두 알고 있는 것이 좋다.

Spring WebFlux는 HTTP 요청을 수행하는 클라이언트로 `WebClient`를 제공한다. `WebClient`는 리액터(Reactor) 기반으로 동작하는 API이다. 리액터 기반이므로 스레드와 동시성 문제를 벗어나 비동기 형식으로 사용할 수 있다. `WebClient`의 특징을 먼저 살펴보겠다.

- 논블로킹(Non-Blocking) I/O를 지원한다.
- 리액티브 스트림(Reactive Streams)의 백프레셔(Back Pressure)를 지원한다.
- 적은 하드웨어 리소스로 동시성을 지원한다.
- 함수형  API를 지원한다.
- 동기, 비동기 상호작용을 지원한다.
- 스트리밍을 지원한다.

최근 프로그래밍 추세에 맞춰 스프링에도 리액티브 프로그래밍(Reactive Programming)이 도입되면서 여러 동시적 기능이 제공되고 있다. 다만 이 책에서는 리액티브 프로그래밍을 자세히 다루지 않으며, `WebClient` 를 사용할 수 있는 환경을 구성하고 사용하는 방법에 대해서만 다룰 예정이다.



## WebClient 구성

`WebClient`를 사용하려면 WebFlux 모듈에 대한 의존성을 추가해야 한다. 아래와 같이 `pom.xml`파일에 의종성을 추가한다.

<br>

```xml
<dependencies>
    ..생략..
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-webflux</artifactId>
    </dependency>
    ..생략..
</dependencies>
```

<br>

WebFlux는 클라이언트와 서버 간 리액티브 애플리케이션 개발을 지원하기 위해 스프링 프레임워크 5에서 새롭게 추가된 모듈이다. `pom.xml`에 위와 같이 WebFlux를 추가하면 `WebClient`를 사용할 수 있는 환경이 만들어진다.

