---
title : "[SpringBoot][Chapter12.4] 서버 간 통신 - WebClient사용하기"
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





## WebClient 사용하기

이제 환경이 구성됐으므로 `WebClient`를 활용한 코드를 작성해보겠다. 다만 지금까지의 실습은 리액티브 프로그래밍을 기반으로 작성된 애플리케이션이 아니기 때문에 `WebClient`를 온전히 사용하기에는 제약사항이 있다.



### WebClient 구현

`WebClient`를 생성하는 방법은 다음과 같이 크게 두 가지가 있다.

- `create()` 메서드를 이용한 생성
- `builder()` 를 이용한 생성



먼저 앞에서 생성했던 서버 프로젝트의 GET 메서드 컨트롤러에 접근할 수 있는 `WebClient`를 생성해보자. 아래와 같이 코드를 작성한다.

<br>

```java
@Service
public class WebClientService {

    public String getName(){
        WebClient webClient = WebClient.builder()
                .baseUrl("http://localhost:9090")
                .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                .build();

        return webClient.get()
                .uri("/api/v1/crud-api")
                .retrieve()
                .bodyToMono(String.class)
                .block();
    }

    public String getNameWithPathVariable() {
        WebClient webClient = WebClient.create("http://localhost:9090");

        ResponseEntity<String> responseEntity = webClient.get()
                .uri(uriBuilder -> uriBuilder.path("/api/v1/crud-api/{name}")
                        .build("Flature"))
                .retrieve().toEntity(String.class).block();

        return responseEntity.getBody();
    }

    public String getNameWithParameter() {
        WebClient webClient = WebClient.create("http://localhost:9090");

        return webClient.get().uri(uriBuilder -> uriBuilder.path("/api/v1/crud-api")
                        .queryParam("name", "Flature")
                        .build())
                .exchangeToMono(clientResponse -> {
                    if (clientResponse.statusCode().equals(HttpStatus.OK)) {
                        return clientResponse.bodyToMono(String.class);
                    } else {
                        return clientResponse.createException().flatMap(Mono::error);
                    }
                })
                .block();
    }
}
```

<br>

위 코드에서 3개의 메서드가 정의돼 있다. 4~15번 줄의 `getName()` 메서드는 `builder()`를 활용해 `WebClient`를 만들고 다른  두 개의 메서드에서는 `create()`를 활용해 `WebClient`를 생성한다.

`WebClient`는 우선 객체를 생성한 후 요청을 전달하는 방식으로 동작한다. 이를 위해 5~8번 줄을 보면 `builder()`를 통해 `baseUrl()`메서드에서 기본 URL을 설정하고 `defaultHeader()` 메서드로 헤더의 값을 설정했다. 일반적으로 `WebClient` 객체를 이용할 때는 이처럼 `WebClient` 객체를 생성한 후 재사용하는 방식으로 구현하는 것이 좋다. 예제에서 소개된 메서드 외에 `builder()` 를 사용할 경우 확장할 수 있는 메서드는 다음과 같다.

- `defaultHeader()` : `WebClient`의 기본 헤더 설정
- `defauiltCookie()` : `WebClient`의 기본 쿠키 설정
- `defaultUriVariable()` : `WebClient`의 기본 URI 확장갑 설정
- `filter()` : `WebClient` 에서 발생하는 요청에 대한 필터 설정



일단 빌드된 `WebClient`는 변경할 수 없다. 다만 다음과 같이 복사해서 사용할 수는 있다.

<br>

```java
WebClient webClient = WebClient.create("http://localhost:9090");
WebClient clone = webClient.mutate().build();
```

<br>

10~14번 줄에는 실제 요청 코드가 작성돼 있다. `WebClient`는 HTTP 메서드를 `get()`, `put()`, `delete()` 등의 네이밍이 명확한 메서드로 설정할 수 있다. 그리고 URI를 확장하는 방법으로 `uri()` 메서드를 사용할 수 있다.

`retrieve()` 메서드는 요청에 대한 응답을 받았을 때 그 값을 추출하는 방법 중 하나이다. `retrieve()` 메서드는 `bodyToMono()` 메서드를 통해 리턴 타입을 설정해서 문자열 객체를 받아오게 돼있다. 참고로 Mono는 리액티브 스트림에 대한 선행 학습이 필요한 개념이며, Flux 와 비교되는 개넘이다. Flux와 Mono는 리액티브 스트림에서 데이터를 제공하는 발행자 역할을 수행하는 `Publisher`의 구현체이다.

`WebClient`는 기본적으로 논블로킹(Non-Blocking) 방식으로 동작하기 때문에 기존에 사용하던 코드의 구조를 블로킹 구조로 바꿔줄 필요가 있다. 예제에서는 14번 줄에 `block()`이라는 메서드를 추가해서 블로킹 형식으로 동작하게끔 설정했다.

17~26번 줄의 `getNameWithPathVariable()` 메서드는 `PathVariable` 값을 추가해 요청을 보내는 예제이다. 21번 줄처럼 `uri()` 메서드 내부에서 `uriBuilder`를 사용해 `path`를 설정하고 `build()` 메서드를 추가할 값을 넣는 것으로 `pathVariable`을 추가할 수 있다. 좀 더 간략하게 작성하고 싶다면 다음과 같이 작성할 수 있다.

<br>

```java
ResponseEntity<String> responseEntity1 = webClient.get()
    .uri("/api/v1/crud-api/{name}", "Flature")
    .retrieve()
    .toEntity(String.class)
    .block();
```

<br>

17~26번 줄은 `bodyToMono()` 메서드가 아닌 `toEntity()`를 사용하는 예제이다. `toEntity()` 를 사용하면 `ResponseEntity`타입으로 응답을 전달받을 수 있다.

28~42번 줄의 `getNameWithParameter()` 메서드는 쿼리 파라미터를 함께 전달하는 방법을 제시한다. 쿼리 파라미터를 요청에 담기 위해서는 31번 줄과 같이 `uriBuilder`를 사용하며, `queryParam()` 메서드를 사용해 전달하려는 값을 설정한다. 그리고 예제에서는 `retrieve()` 대신 `exchange()` 메서드를 사용했다. `exchange()` 메서드는 지원 중단됐기 때문에 `exchangeToMono()`또는 `exchangeToFlux()`를 사용해야 한다. `exchange()` 메서드는 응답 결과 코드에 따라 다르게 응답을 설정할 수 있다. 34~40번 줄을 보면 `clientResponse` 결괏값에 따라 `if`문 분기를 만들어 상황에 따라 결괏값을 다르게 전달할 수 있다.

POST 요청을 아래와 같이 작성할 수 있다.

<br>

```java
@Service
public class WebClientService {
    
    public ResponseEntity<MemberDto> postWithParamAndBody() {
        WebClient webClient = WebClient.builder()
                .baseUrl("http://localhost:9090")
                .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                .build();

        MemberDto memberDto = new MemberDto();
        memberDto.setName("flature!!");
        memberDto.setEmail("flature@gmail.com");
        memberDto.setOrganization("Around Hub Studio");

        return webClient.post().uri(uriBuilder -> uriBuilder.path("/api/v1/crud-api")
                        .queryParam("name", "Flature")
                        .queryParam("email", "flature@wikibooks.co.kr")
                        .queryParam("organization", "Wikibooks")
                        .build())
                .bodyValue(memberDto)
                .retrieve()
                .toEntity(MemberDto.class)
                .block();
    }

    public ResponseEntity<MemberDto> postWithHeader() {
        WebClient webClient = WebClient.builder()
                .baseUrl("http://localhost:9090")
                .defaultHeader(HttpHeaders.CONTENT_TYPE, MediaType.APPLICATION_JSON_VALUE)
                .build();

        MemberDto memberDto = new MemberDto();
        memberDto.setName("flature!!");
        memberDto.setEmail("flature@gmail.com");
        memberDto.setOrganization("Around Hub Studio");

        return webClient
                .post()
                .uri(uriBuilder -> uriBuilder.path("/api/v1/crud-api/add-header")
                        .build())
                .bodyValue(memberDto)
                .header("my-header", "Wikibooks API")
                .retrieve()
                .toEntity(MemberDto.class)
                .block();
    }
}
```

<br>

`WebClient`를 생성하고 사용하는 방법은 앞서 본 GET 요청을 만ㅁ드는 방법과 다르지 않다. 다만 POST 방식에서 눈여겨볼 내용은 HTTP 바디 값을 담는 방법과 커스텀 헤더를 구가하는 방벙비다.

15~24번 줄은 `webClient` 에서 `post()` 메서드를 통해 POST 메서드 통신을 정의했고, `uri()`는 `uriBuilder`로 `path`와 `parameter`를 설정했다. 그 후 `bodyValue()` 메서드를 통해 HTTP 바디 값을 설정한다. HTTP 바디에는 일반적으로 데이터 객체(DTO, VO 등)를 파라미터로 전달한다.

26~48번 줄의 `postWithHeader()` 메서드는 POST 요청을 보낼 때 헤더를 추가해서 보내는 예제이다. 전반적인 내용은 동일하며, 42번 줄에 `header()` 메서드를 사용해 헤더에 값을 추가했다. 일반적으로 임의로 추가한 헤더에는 외부 API를 사용하기 위해 인증된 토큰값을 담아 전달한다.

<br>

> **💡 Tip.**
>
> 자세한 `WebClient` 의 사용법은 공식 문서에서 확인할 수 있다.
>
> - <a href="https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client">https://docs.spring.io/spring-framework/docs/current/reference/html/web-reactive.html#webflux-client</a>

<br>

지금까지 웹 통신을 위해 `RestTemplate` 과 `WebClient`를 사용하는 방법을 알아봤다. 실무에서 다른 서버의 리소스에 접근하는 상황은 자주 발생한다. 이러한 경우 대체로 이번 장에서 소개한 통신 모듈을 사용해 기능을 구현하면 해결된다.

이 책에서 소개한 방법을 익혔다면 이후에는 통신하는 횟수나 접근하는 서버의 특성에 맞게 커넥션 풀이나 타임아웃 등의 설정을 최적화하는 작업으로 심화 학습을 진행하면 된다. 또한 서버와의 웹 통신을 최적화하는 방법으로 어떤 방법이 있는지 알아보고, 각 상황에 따른 설정 방법을 공부해 보는 것을 권장한다.

