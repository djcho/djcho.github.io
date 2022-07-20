---
title : "[SpringBoot][Chapter5.6] API 작성법 - REST API의 문서화 Swagger"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Swagger]
toc: true
toc_sticky : true
published : true
date : 2022-07-19
last_modified_at : 2022-07-19
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## REST API 명세를 문서화하는 방법 - Swagger

API를 개발하면 명세를 관리해야 한다. 명세란에 해당 API가 어떤 로직을 수행하는지 설명하고 이 로직을 수행하기 위해 어떤 값을 요청하며, 이에 따른 응답값으로는 무엇을 받을 수 있는지를 정리한 자료이다.

API는 개발 과정에서 계속 변경되므로 작성한 명세 문서도 주기적인 업데이트가 필요하다. 또한 명세 작업은 번거롭고 시간 또한 오래 걸린다. 이 같은 문제를 해결하기 위해 등장한 것이 바로 [Swagger]라는 오픈소스 프로젝트이다.

Swagger를 사용하기 위해서는 먼저 pom.xml 파일에 의존성을 추가해야 한다.

[Swagger]: https://swagger.io

<br>

```xml
    <dependencies>
        ... 생략 ...
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
    </dependencies>
```

<br>

종속성을 추가했다면 다음과 같이 Swagger와 관련된 설정 코드를 작성한다. 이 클래스는 설정(Configuration)에 관한 클래스로 `com.springboot.api` 하단에 `config` 라는 패키지를 생성한 후에 그 안에 생성하는 것이 좋다.

<br>

```java
@Configuration
@EnableSwagger2
public class SwaggerConfiguration {
    
    @Bean
    public Docket api(){
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.springboot.api"))
                .paths(PathSelectors.any())
                .build();
    }
    
    private ApiInfo apiInfo(){
        return new ApiInfoBuilder()
                .title("Spring Boot Open API Test with Swagger")
                .description("설명 부분")
                .version("1.0.0")
                .build();
    }
}
```

<br>

위 내용을 구현하면 Swagger 사용을 위한 기본적인 설정이 완료된다. Swagger에서 스캔할 패키지 범위를 `RequestHandlerSelectors.basePackage()`를 사용해 설정한다. 현재 프로젝트의 루트 패키지는 `com.springboot.api`로 설정돼 있어 그대로 작성했다. 그럼 하위 패키지와 클래스를 모두 스캔해서 문서를 생성한다.

인텔리제이 IDEA에서 애플리케이션을 실행한 후 웹 브라우저를 통해 <a herf="http://localhost:8080/swagger-ui.html" target="_blank"> http://localhost:8080/swagger-ui.html</a>로 접속하면 아래와 같이 Swagger 페이지가 출력된다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179682619-d353d8c5-6960-4684-9595-87c54f530386.png){: .align-center}

<br>

Swagger를 더 잘 활용하기 위해 이번 장에서 작성한 API 중 @RequestParam을 활용한 GET 메서드에 대한 명세의 세부 내용을 설정해 보겠다. `GetController`에 작성한 메서드를 아래와 같이 수정해 보자.

<br>

```java
    @ApiOperation(value = "GET 메서드 예제", notes = "@RequestParam을 활용한 GET Method")
    @GetMapping(value = "/request1")
    public String getRequestParam1(
            @ApiParam(value = "이름", required = true) @RequestParam String name,
            @ApiParam(value = "이메일", required = true) @RequestParam String email,
            @ApiParam(value = "회사", required = true) @RequestParam String organization) {
        return name + " " + email + " " + organization;
    }
```

<br>

`@ApiOperation` 과 `@ApiParam` 어노테이션은 Swagger가 사용하는 대표 어노테이션이다. 간략한 설명은 다음과 같다.

- `@ApiOperation` : 대상 API의 설명을 작성하기 위한 어노테이션이다.
- `@ApiParam` : 매개변수에 대한 설명 및 설정을 위한 어노테이션이다. 메서드의 매개변수 뿐 아니라 DTO 객체를 매개변수로 사용할 경우 DTO 클래스 내의 매개변수에도 정의할 수 있다.

위와 같이 설정한 후 API의 명세를 Swagger 페이지에서 살펴보면 아래와 같다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179756233-60403f63-32cc-454f-a5fc-5067969e50fc.png){: .align-center}

<br>

`@ApiOperation` 에 작성한 내용은 그림 상단에 표기되고 `@ApiParam`에 정의한 내용은 아래 'Parameters' 영역의 'Description' 항목에 추가됐다. 위 그림처럼 Swagger 는 해당 API가 무엇인지 설명하고 어떤 값이 필요한지를 한눈에 보여준다.

Swagger에서는 API 명세 관리뿐 아니라 직접 통신도 시도할 수 있다. 위 화면에서 [Try it out] 버튼을 클릭하면 아래와 같은 화면이 표시된다.

<br>![image](https://user-images.githubusercontent.com/13410737/179756657-62e7f81f-3dae-4ee1-bebd-99af9e90b7a7.png){: .align-center}

<br>



각 항목의 값을 기입하고 [Excute] 버튼을 누르면 아래와 같이 자동으로 완성된 요청 URL을 확인할 수 있고, 그에 대한 결괏값도 받아볼 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179756937-1d2c5f4d-b104-4ec7-b6e8-cfd73f05a055.png){: .align-center}

<br>



이렇게 해서 Swagger 사용법을 익혔으므로 이후 내용에서는 Talend API Tester가 아닌 Swagger 페이지를 통해 테스트를 진행을 하겠다.
