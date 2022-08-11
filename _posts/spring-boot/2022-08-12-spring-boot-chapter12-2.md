---
title : "[SpringBoot][Chapter12.2] 서버 간 통신 - RestTemplate 사용하기"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, RestTemplate]
toc: true
toc_sticky : true
published : true
date : 2022-08-12
last_modified_at : 2022-08-12
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





## RestTemplate 사용하기

이제 `RestTemplate`을 사용해보겠다. 요청을 보낼 서버 용도로 별도의 프로젝트를 하나 생성하고 다른 프로젝트에서 `RestTemplate`을 통해 요청을 보내는 방식으로 실습을 진행할 예정이다.



### 서버 프로젝트 생성하기

먼저 `RestTemplate`의 동작을 확인하기 위해 서버 용도의 프로젝트를 생성하겠다. 실습 환경에서는 한 컴퓨터 안에서 두 개의 프로젝트를 가동시켜야 하기 때문에 톰캣의 포트를 변경해야 한다. 프로젝트에는 `spring-boot-starter-web` 모듈만 의존성으로 추가하며, 이 책에서는 `serverBox`라는 이름으로 프로젝트를 생성했다. 이 프로젝트의 구조는 아래와 같다.

<br>

![image](https://user-images.githubusercontent.com/13410737/184150198-ef20a27b-75c2-490a-a7f8-730c8fdf472c.png){: .align-center}

<br>

```properties
server.port=9090
```

<br>

컨트롤러에서는 GET과 POST 메서드 형식의 요청을 받기 위한 코드를 구성하겠다. 컨트롤러 클래스는 아래와 같다.

<br>

```java
@RestController
@RequestMapping("api/v1/crud-api")
public class CrudController {

    @GetMapping
    public String getName(){
        return "Flature";
    }

    @GetMapping(value = "/{variable}")
    public String getVariable(@PathVariable String variable) {
        return variable;
    }

    @GetMapping("/param")
    public String getNameWithParam(@RequestParam String name){
        return "Hello. " + name + "!";
    }

    @PostMapping
    public ResponseEntity<MemberDto> getMember(
            @RequestBody MemerDto request,
            @RequestParam String name,
            @RequestParam String email,
            @RequestParam String organization)
    {
        System.out.println(request.getName());
        System.out.println(request.getEmail());
        System.out.println(request.getOrganization());

        MemberDto memberDto = new MemberDto();
        memberDto.setName(name);
        memberDto.setEmail(email);
        memberDto.setOrganization(organization);

        return ResponseEntity.status(HttpStatus.OK).body(memberDto);
    }

    @PostMapping(value = "/add-header")
    public ResponseEntity<MemberDto> addHeader(@RequestHeader("my-header") String header,
                                               @RequestBody MemberDto memberDto){

        System.out.println(header);

        return ResponseEntity.status(HttpStatus.OK).body(memberDto);
    }
}
```

<br>

PUT, DELETE 메서드는 GET과 POST 형식과 각 구성 방식이 거의 비슷하기 때문에 생략했다. 위 코드의 5~18번 줄의 코드는 GET 형식의 요청이 들어오는 상황의 케이스를 구현한다. 첫 번째 메서드는 아무 파라미터가 없는 경우, 두 번째는 `PathVariable`을 사용하는 경우, 세 번째는 `RequestParameter`를 사용하는 경우이다.

20~46번 줄에는 POST 형식의 요청을 받기 위한 두 개의 메서드가 구현돼 있다. 첫번째 메서드는 예쩨의 간소화를 위해 요청 파라미터(Request Parameter)와 요청 바디(Request Body)를 함께 받도록 구현했고, 두 번째 메서드는 임의의 HTTP 헤더를 받도록 구현했다.

여기서 사용된 `MemeberDto`객체는 아래와 같다.

<br>

```java
public class MemberDto {

    private String name;
    private String email;
    private String organization;

    public String getName(){
        return name;
    }

    public void setName(String name){
        this.name = name;
    }

    public String getEmail(){
        return email;
    }

    public void setEmail(String email){
        this.email = email;
    }

    public String getOrganization(){
        return organization;
    }

    public void setOrganization(String organization){
        this.organization = organization;
    }

    @Override
    public String toString(){
        return "MemberDto{" +
                "name='" + name + '\'' +
                ", email='" + email + '\'' +
                ", organization='" + organization + '\'' +
                '}';
    }
}
```

<br>

`MemberDto`클래스는 `name`, `email`, `organization`이라는 총 3개의 필드를 가지고 있다.



### RestTemplate구현하기

일반적으로 `RestTemplate`은 별도의 유틸리티 클래스로 생성하거나 서비스 또는 비즈니스 계층에 구현된다. 앞서 생성한 서버 프로젝트에 요청을 날리기 위해 서버의 역할을 수행하면서 다른 서버로 요청을 보내는 클라이언트의 역할도 수행하는 새로운 프로젝트를 생성한다. 간단하게 도식화하면 아래와 같다.

<br>

![image](https://user-images.githubusercontent.com/13410737/184151440-97596a2d-2392-4d8b-b430-86dda2c3c816.png){: .align-center}

<br>

위 그림에서 클라이언트는 서버를 대상으로 요청을 보내고 응답을 받는 역할을 하고, 앞에서 구현한 서버 프로젝트는 서버2가 된다.

지금부터 `RestTemplate`을 포함하는 프로젝트를 생성하겠다. 다음과 같이 설정해서 프로젝트를 생성한다. 스프링 부트 버전은 이전과 같은 2.5.6 버전으로 진행하며, 다음과 같은 내용을 설정한다.

- groupId : com.springboot
- artifactId : rest
- name : rest
- Developer Tools : Lombok, Spring Configuration Processor
- Web : Spring Web

<br>

그리고 클라이언트에서 요청하는 것처럼 실습하기 위해 `SwaggerConfiguration` 클래스 파일과 의존성 추가를 해야 한다. `RestTemplate`은 이미 `spring-boot-starter-web` 모듈에 포함돼 있는 기능이므로 `pom.xml`에 별도로 의존성을 추가할 필요는 없다. 프로젝트의 구조로는 클라이언트로부터 요청을 받는 컨트롤러와, `RestTemplater`을 활용해 다른 서버에 통신 요청을 하는 서비스 계층으로 작성하겠다.

<br>

#### GET 형식의 RestTemplate 작성하기

먼저 GET 형식의 `RestTemplate` 예제를 살펴보겠다.

<br>

```java
@Service
public class RestTemplateService {

    public String getName() {
        URI uri = UriComponentsBuilder
                .fromUriString("http://localhost:9090")
                .path("/api/v1/crud-api")
                .encode()
                .build()
                .toUri();

        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> responseEntity = restTemplate.getForEntity(uri, String.class);

        return responseEntity.getBody();
    }

    public String getNameWithPathVariable() {
        URI uri = UriComponentsBuilder
                .fromUriString("http://localhost:9090")
                .path("/api/v1/crud-api/{name}")
                .encode()
                .build()
                .expand("Flature") // 복수의 값을 넣어야할 경우 , 를 추가하여 구분
                .toUri();

        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> responseEntity = restTemplate.getForEntity(uri, String.class);

        return responseEntity.getBody();
    }

    public String getNameWithParameter() {
        URI uri = UriComponentsBuilder
                .fromUriString("http://localhost:9090")
                .path("/api/v1/crud-api/param")
                .queryParam("name", "Flature")
                .encode()
                .build()
                .toUri();

        RestTemplate restTemplate = new RestTemplate();
        ResponseEntity<String> responseEntity = restTemplate.getForEntity(uri, String.class);

        return responseEntity.getBody();
    }
}
```

<br>

`RestTemplate`을 생성하고 사용하는 방법은 아주 다양하다. 그중 가장 보편적인 방법은 `UriComponentsBuilder`를 사용하는 방법이다. `UriComponentsBuilder`는 스프링 프레임워크에서 제공하는 클래스로서 여러 파라미터를 연결해서 URI 형식으로 만드는 기능을 수행한다.

각 메서드는 정의된 컨트롤러 메서드와 비교해 코드를 확인할 수 있다. 4~16번 줄의 메서드는 `PathVariable`이나 파라미터를 사용하지 않는 호출 방법이다. `UriComponentsBuilder`는 빌더 형식으로 객체를 생성한다. `fromUriString()`메서드에서는 호출부의 URL을 입력하고, 이어서 `path()` 메서드에 세부 경로를 입력한다. `encode()` 메서드는 인코딩 문자셋을 설정할 수 있는데, 인자를 전달하지 않으면 기본적으로 UTF-9로 다음과 같은 코드가 실행된다.

<br>

```java
public final UriComponentsBuilder encode() {
    return encode(StandardCharsets.UTF-8);
}
```

<br>

이후 `builder()`메서드를 통해 빌더 생성을 종료하고 `UriComponents`타입이 리턴된다. 예에에서는 `toUri()` 메서드를 통해 URI 타입으로 리턴받았따. 만약 URI 객체를 사용하지 않고 `String` 타입의 URI를 사용한다면 `toUriString()` 메서드로 대체해서 사용하면 된다.

이렇게 생성된 uri는 `restTemplate`이 외부 API를 요청하는데 사용되며 ,13번 줄의 `getForEntity()`에 파라미터로 전달된다. `getForEntity()`는 URI와 응답받는 타입을 매개변수로 사용한다.

18~31번 줄의 코드에서 눈여겨볼 내용은 `path()` 메서드와 `expand()` 메서드 내에 입력한 세부 URI 중 중괄호({}) 부분을 사용해 개발 단계에서 쉽게 이해할 수 있는 변수명을 입력하고 `expand()` 메서드에서는 순서대로 값을 입력하면 된다. 값을 여러 개 넣어야 하는 경우에는 콤마(,) 로 구분해서 나열한다.

33~46번 줄은 파라미터로 전달하는 예제이다. 예제에서 볼 수 있듯이 `queryParam()` 메서드를 사용해 (키, 값) 형식의 파라미터를 추가할 수 있다.



#### POST 형식의 RestTemplate 작성

POST 형식의 `RestTemplate` 사용법은 아래와 같다.

<br>

```java
public ResponseEntity<MemberDto> postWithParamAndBody() {
    URI uri = UriComponentsBuilder
            .fromUriString("http://localhost:9090")
            .path("/api/v1/crud-api")
            .queryParam("name", "Flature")
            .queryParam("email", "flature@wikibooks.co.kr")
            .queryParam("name", "Wikibooks")
            .encode()
            .build()
            .toUri();

    MemberDto memberDto = new MemberDto();
    memberDto.setName("flature!!");
    memberDto.setEmail("flature@gmail.com");
    memberDto.setOrganization("Around Hub Studio");

    RestTemplate restTemplate = new RestTemplate();
    ResponseEntity<MemberDto> responseEntity = restTemplate.postForEntity(uri, memberDto, MemberDto.class);

    return responseEntity;
}

public ResponseEntity<MemberDto> postWithHeader() {
    URI uri = UriComponentsBuilder
            .fromUriString("http://localhost:9090")
            .path("/api/v1/crud-api")
            .encode()
            .build()
            .toUri();

    MemberDto memberDto = new MemberDto();
    memberDto.setName("flature!!");
    memberDto.setEmail("flature@gmail.com");
    memberDto.setOrganization("Around Hub Studio");

    RequestEntity<MemberDto> requestEntity = RequestEntity
            .post(uri)
            .header("my-header", "Wikibooks API")
            .body(memberDto);

    RestTemplate restTemplate = new RestTemplate();
    ResponseEntity<MemberDto> responseEntity = restTemplate.exchange(requestEntity,
            MemberDto.class);

    return responseEntity;
}
```

<br>

예제에서 1~22번 줄은 POST 형식으로 외부 API에 요청할 때 Body 값과 파라미터 값을 담는 방법 두 가지를 모두 보여준다. 2~10번 줄에서는 파라미터에 값을 추가하는 작업이 수행되며, 12~19번 줄에서는 `RequestBody` 에 값을 담는 작업이 수행된다. `RequestBody`에 값을 담기 위해서는  12~15번 줄과 같이 데이터 객체를 생성한다. `postForEntity()` 메서드를 사용할 경우에는 파라미터로 데이터 객체를 넣으면 된다.

`postForEntity()` 메서드로 서버 프로젝트의 API를 호출하면 서버 프로젝트의 콘솔 로그에는 `RequestBody`값이 출력되고 파라미터 값은 결괏값으로 리턴된다. 앞에서 프로젝트를 생성하면서 설명했지만 이 프로젝트에서 쉽게 API를 호출할 수 있게 Swagger를 설정하겠다. `pom.xml`파일에 Swagger 의존성을 추가한 후 아래와 같이 Swagger 설정 코드를 작성한다.

<br>

```java
@Configuration
@EnableSwagger2
public class SwaggerConfiguration {

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
            .apiInfo(apiInfo())
            .select()
            .apis(RequestHandlerSelectors.basePackage("com.springboot.rest"))
            .paths(PathSelectors.any())
            .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
            .title("Spring Boot Open API Test with Swagger")
            .description("설명 부분")
            .version("1.0.0")
            .build();
    }
}
```

<br>

그러고 나서 앞에서 작성한 서비스 코드를 연결하는 컨트롤러 코드를 아래와 같이 작성한다.

<br>

```java
@RestController
@RequestMapping("/rest-template")
public class RestTemplateController {
    
    private final RestTemplateService restTemplateService;
    
    public RestTemplateController(RestTemplateService restTemplateService){
        this.restTemplateService = restTemplateService;
    }
    
    @GetMapping
    public String getName(){
        return restTemplateService.getName();
    }
    
    @GetMapping("/path-variable")
    public String getNameWithPathVariable(){
        return restTemplateService.getNameWithPathVariable();
    }
    
    @PostMapping
    public ResponseEntity<MemberDto> postDto(){
        return restTemplateService.postWithParamAndBody();
    }
    
    @PostMapping("/header")
    public ResponseEntity<MemberDto> postWithHeader(){
        return restTemplateService.postWithHeader();
    }
}
```

<br>

여기까지 진행했다면 애플리케이션을 실행하고 `postDto()` 메서드에 해당하는 POST API를 호출하면 아래의 결과가 출력된다. 참고로 이번 장에서 진행하는 실습은 앞서 생성한 2개의 프로젝트가 모두 가동돼 있는 상태에서 진행해야 한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/184162003-f11bee81-6bea-4c9a-ba34-3bd188c5498b.png){: .align-center}

<br>

```
flature!!
fature@gmail.com
Around Hub Studio
```

<Br>

위 출력 결과는 서버 프로젝트가 파라미터의 값과 Body 값을 정상적으로 전달받았다는 것을 의미한다.

위에서 작성한 서비스 코드의 24~47번 줄의 메서드는 헤더를 추가하는 예제이다. 대부분의 외부 API는 토큰키를 받아 서비스 접근을 인증하는 방식으로 작동한다. 이때 토큰값을 헤더에 담아 전달하는 방식이 가장 많이 사용된다. 헤더를 설정하기 위해서는 `RequestEntity` 를 정의해서 사용하는 방법이 가장 편한 방법이다. 37~40번 줄은 `RequestEntity`를 생성하고 `post()` 메서드로 URI를 설정한 후 `header()` 메서드에서 헤더의 키 이름과 값을 설정하는 코드이다. 대체로 서버 프로젝트의 API 명세에는 헤더에 필요한 키 값을 요구하면서 키 이름을 함께 제시하기 때문에 그에 맞춰 헤더 값을 설정하면 된다.

마지막으로 43번 줄에는 `RestTemplate`의 `exchange()` 메서드를 사용했다. `exchange()` 메서드는 모든 형식의 HTTP 요청을 생성할 수 있다. `RequestEntity`의 설정에서 `post()` 메서드 대신 다른 형식의 메서드로 정의만 하면 `exchange()` 메서드로 쉽게 사용할 수 있기 때문에 대부분 `exchange()` 메서드를 사용하는 편이다.

지금까지 GET, POST 형식으로 `RestTemplater`을 사용하는 방법을 알아봤다.



### RestTemplate 커스텀 설정

`RestTemplate`은 `HTTPClient`를 추상화하고 있다. `HttpClient`의 종류에 따라 기능에 차이가 다소 있는데, 가장 큰 차이는 커넥션 풀(Connection Poll)이다.

`RestTemplate`은 기본적으로 커넥션 풀을 지원하지 않는다. 이 기능을 지원하지 않으면 매번 호출할 때 마다 포트를 열어 커넥션을 생성하게 되는데, `TIME_WAIT` 상태가 된 소켓을 다시 사용하려고 접근한다면 재사용하지 못하게 된다. 이를 방지하기 위해서는 커넥션 풀 기능을 활성화해서 재사용할 수 있게 하는 것이 좋다. 이 기능을 활성화하는 가장 대표적인 방법은 아파치에서 제공하는 `HttpClient`로 대체해서 사용하는 방식이다.

먼저 아파치의 `HttpClient`를 사용하려면 아래와 같이 의존성을 추가해야 한다.

<br>

```xml
<dependencies>
    .. 생략 ..
    <dependency>
        <groupId>org.apache.httpcomponents</groupId>
        <artifactId>httpclient</artifactId>
    </dependency>
    .. 생략 ..
</dependencies>
```

<br>

의존성을 추가하면 `RestTemplate`의 설정을 더욱 쉽게 추가하고 변경할 수 있다. 아래를 통해 살펴보겠다.

<br>

```java
public RestTemplate restTemplate(){
    HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory();

    HttpClient client = HttpClientBuilder.create()
            .setMaxConnTotal(500)
            .setMaxConnPerRoute(500)
            .build();

    CloseableHttpClient httpClient = HttpClients.custom()
            .setMaxConnTotal(500)
            .setMaxConnPerRoute(500)
            .build();

    factory.setHttpClient(httpClient);
    factory.setConnectTimeout(2000);
    factory.setReadTimeout(5000);

    RestTemplate restTemplate = new RestTemplate(factory);
    
    return restTemplate;
}
```

<br>

`RestTemplate`의 생성자를 보면 다음과 같이 `ClientHttpRequestFactory`를 매개변수로 받는 생성자가 존재한다

<br>

```java
public RestTemplate(ClientHttpRequestFactory requestFactory) {
    this();
    this.setRequestFactory(requestFactory);
}
```

<br>

`ClientHttpRequestFactory`는 함수형 인터페이스(functional interface)로, 대표적인 구현체로서 `SimpleClieentHttpRequestFactory`와 `HttpComponentsClientHttpRequestFactory`가 있다. 별도의 구현체를 설정해서 전달하지 않으면 `HttpAccessor`에 구현돼 있는 내용에 의해 `SimpleClientHttpRequestFactory`를 사용하게 된다.

별도의 `HttpComponentsClientHttpRequestFactory` 객체를 생성해서 `ClientHttpRequestFactory`를 사용하면 15~16번 줄과 같이 `RestTemplate`의 `Timeout` 설정을 할 수 있다.

그리고 `HttpComponentsClientHttpRequestFactory`는 커넥션 풀을 설정하기 위해 `HttpClient`를 `HttpComponentsClientHttpRequestFactory`에 설정할 수 있다. `HttpClient`를 생성하는 방법은 두 가지가 있는데 4~7번 줄의 `HttpClientBuilder.create()` 메서드를 사용하거나 9~12번 줄의 `HttpClients.custom()` 메서드를 사용하는 것이다.

생성한 `HttpClient`는 14번 줄과 같이 `factory`의 `setHttpClient()` 메서드를 통해 인자로 전달해서 설정할 수 있다. 이렇게 설정된 `factory` 객체를 `RestTemplate`을 초기화하는 과정에서 인자로 전달하면 된다.

<br>

> 4번 줄에서는 `HttpClient` 객체를 생성했고 9번 줄에서는 `CloseableHttpClient`를 생성했다. 두 객체는 비슷하면서도 기능 면에서 차이가 있다. 이 두객체의 차이를 비교해보면 커넥션에 대한 기초 지식까지 늘릴 수 있는 기회가 될 것이다.
