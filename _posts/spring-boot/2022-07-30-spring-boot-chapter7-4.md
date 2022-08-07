---
title : "[SpringBoot][Chapter7.4] 테스트 코드 작성하기 - JUnit 테스트 코드 작성"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, JUnit]
toc: true
toc_sticky : true
published : true
date : 2022-07-30
last_modified_at : 2022-07-30
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## JUnit을 활용한 테스트 코드 작성

<a href="https://junit.org/junit5/" target="_blank">JUnit</a>은 자바 언어에서 사용되는 대표적인 테스트 프레임워크로서 단위 테스트를 위한 도구를 제공한다. 또한 단위 테스트뿐만 아니라 통합 테스트를 할 수 있는 기능도 제공한다. JUnit의 가장 큰 특징은 어노테이션 기반의 테스트 방식을 지원한다는 것이다. 즉, JUnit을 사용하면 몇 개의 어노테이션만으로 간편하게 테스트 코드를 작성할 수 있따. 또한 JUnit을 활용하면 단정문(assert)을 통해 테스트 케이스의 기댓값이 정상적으로 도출됐는지 검토할 수 있따는 장점이 있다.

참고로 이 책에서 사용할 JUnit 5 버전은 스프링 부트 2.2 버전부터 사용 가능하며, 이 책에서 사용 중인 스프링 부트는 2.5.6버전이다.



### JUnit의 세부 모듈

JUnit 5는 크게 Platform, Jupiter, Vintage의 세 모듈로 구성된다.



**JUnit Platform**

- JUnit Platform은 JVM에서 테스트를 시작하기 위한 뼈대 역할을 한다. 테스트를 발견하고 테스트 계획을 생성하는 테스트 엔진(TestEngine)의 인터페이스는 가지고 있다. 테스트 엔진은 테스트를 발견하고 테스트를 수행하며, 그 결과를 보고하는 역할을 수행한다. 또한 각종 IDE와의 연동을 보조하는 역할을 수행한다.(IDE 콘솔 출력 등) Platform에는 TestEngine API, Console Launcher, JUnit 4 Based Runner 등이 포함돼 있다.

**JUnit Jupiter**

- 테스트 엔진 API의 구현체를 포함하고 있으며, JUnit 5에서 제공하는 Jupiter 기반의 테스트를 실행하기 위한 테스트엔진을 가지고 있다. 테스트의 실제 구현체는 별도 모듈의 역할을 수행하는데, 그중 하나가 Jupiter Engine 이다. Jupiter Engine은 Jupiter API를 활용해서 작성한 테스트 코드를 발견하고 실행하는 역할을 수행한다.

**JUnit Vintage**

- JUnit 3, 4에 대한 테스트 엔진 API를 구현하고 있다. 기존에 작성된 JUnit 3,4 버전의 테스트 코드를 실행할 때 사용되며 Vintage Engine을 포함하고 있다.



이처럼 JUnit은 하나의 Platform 모듈을 기반으로 Jupiter와 Vintage 모듈이 구현체의 역할을 수행한다. JUnit의 구조를 간단하게 그림으로 표현하면 다음와 같다.

![image](https://user-images.githubusercontent.com/13410737/181782013-f7038122-a911-4d5b-8cd3-f9407ef86b51.png){: .align-center}

> **💡 Tip.**
>
> JUnit 5에 대한 자세한 내용은 아래의 사용자 가이드를 참고한다.
>
> - <a href="https://junit.org/junit5/docs/current/user-guide/" target="_blank">https://junit.org/junit5/docs/current/user-guide/</a>



### 스프링 부트 프로젝트 생성

이번 장에서 사용할 프로젝트를 설정한다. groupId는 'com.springboot'로 설정하고 name은 'test', artifactId는 'test'로 설정한다. 그리고 다음과 같이 의존성을 추가한다.

- Developer Tools : Lombok, Spring Configuration Processor
- Web : Spring Web
- SQL : Spring Data JPA, MariaDB Driver

그리고 6장에서 만들었던 프로젝트의 코드 일부를 다음과 같이 가져온다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181783936-03f305d6-1c4d-4131-80cc-e1ef55fbff95.png){: .align-center}

<br>

6장에서는 프로젝트의 구조 설계를 설명하기 위해 DAO 레이어를 추가했지만 이 책에서 다루는 예제들은 간단한 코드로 구성돼 있기 때문에 그 특징을 살리기에는 부족하다. 그렇기 때문에 이번장 부터는 DAO 레이어는 제외하고 서비스 레이어에서 바로 리포지토리를 사용하는 구조로 진행하겠다. 이에 따라 `ProductServiceImpl` 클래스를 다음과 같이 수정한다.

<br>

```java
@Service
public class ProductServiceImpl implements ProductService {

    private final Logger LOGGER = LoggerFactory.getLogger(ProductServiceImpl.class);
    private final ProductRepository productRepository;

    @Autowired
    public ProductServiceImpl(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Override
    public ProductResponseDto getProduct(Long number) {
        LOGGER.info("[getProduct] input number : {}", number);
        Product product = productRepository.findById(number).get();

        LOGGER.info("[getProduct] product number : {}, name : {}", product.getNumber(),
                product.getName());
        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(product.getNumber());
        productResponseDto.setName(product.getName());
        productResponseDto.setPrice(product.getPrice());
        productResponseDto.setStock(product.getStock());

        return productResponseDto;
    }

    @Override
    public ProductResponseDto saveProduct(ProductDto productDto) {
        LOGGER.info("[saveProduct] productDTO : {}", productDto.toString());
        Product product = new Product();
        product.setName(productDto.getName());
        product.setPrice(productDto.getPrice());
        product.setStock(productDto.getStock());
        product.setCreatedAt(LocalDateTime.now());
        product.setUpdatedAt(LocalDateTime.now());

        Product savedProduct = productRepository.save(product);
        LOGGER.info("[saveProduct] savedProduct : {}", savedProduct);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(savedProduct.getNumber());
        productResponseDto.setName(savedProduct.getName());
        productResponseDto.setPrice(savedProduct.getPrice());
        productResponseDto.setStock(savedProduct.getStock());

        return productResponseDto;
    }

    @Override
    public ProductResponseDto changeProductName(Long number, String name) throws Exception {
        Product foundProduct = productRepository.findById(number).get();
        foundProduct.setName(name);
        Product changedProduct = productRepository.save(foundProduct);

        ProductResponseDto productResponseDto = new ProductResponseDto();
        productResponseDto.setNumber(changedProduct.getNumber());
        productResponseDto.setName(changedProduct.getName());
        productResponseDto.setPrice(changedProduct.getPrice());
        productResponseDto.setStock(changedProduct.getStock());

        return productResponseDto;
    }

    @Override
    public void deleteProduct(Long number) throws Exception {
        productRepository.deleteById(number);
    }
}
```

<br>

그리고 `Product`  엔티티 클래스를 다음과 같이 수정한다.

<br>

```java
package com.springboot.test.data.entity;

import lombok.*;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Builder
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@EqualsAndHashCode
@ToString(exclude = "name")
@Table(name = "product")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;
}
```

<br>

### 스프링 부트의 테스트 설정

스프링 부트는 테스트 환경을 쉽게 설정할 수 있게 `spring-boot-starter-test` 프로젝트를 지원한다. 이 프로젝트를 사용하려면 아래와 같이 `pom.xml` 파일에 관련 의존성을 추가해야 한다.

<br>

```xml
<dependencies>
    ... 생략 ...
    <dependency>
    	<groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

<br>

예제에서 추가한 라이브러리는 아래와 같은 의존성을 가지고 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181788926-4917f86c-f0a7-40c1-850e-dcde3aefc8b5.png){: .align-center}

<br>

스프링 부트에서 제공하는 `spring-boot-starter-test` 라이브러리는 JUnit, Mockito, assertJ 등의 다양한 테스트 도구를 제공한다. 또한 자동 설정을 지원하므로 편리하게 쓸 수 있다. `spring-boot-starter-test`라이브러리에서 제공하는 대표적인 라이브러리는 다음과 같다.

- JUnit 5 : 자바 애플리케이션의 단위 테스트를 지원한다.
- Spring Test & Spring Boot Test : 스프링 부트 애플리케이션에 대한 유틸리티와 통합 테스트를 지원한다.
- AssertJ : 다양한 단정문(assert)을 지원하는 라이브러리이다.
- Hamcrest : Matcher를 지원하는 라이브러리이다.
- Mockito : 자바 Mock 객체를 지원하는 프레임워크이다.
- JSONassert : JSON용 단정문 라이브러리이다.
- JsonPath : JSON용 XPath를 지원한다.



### JUnit의 생명주기

JUnit의 동작 방식을 확인하기 위해 생명주기를 알아보겠다. 생명주기와 관련되어 테스트 순서에 관여하게 되는 대표적인 어노테이션은 다음과 같다.

- `@Test `: 테스트 코드를 포함한 메서드를 정의한다.
- `@BeforeAll` : 테스트를 시작하기 전에 호출되는 메서드를 정의한다.
- `@BeforeEach` : 각 테스트 메거드가 실행되기 전에 동작하는 메서드를 정의한다.
- `@AfterAll` : 테스트를 종요하면서 호출되는 메서드를 정의한다.
- `@AfterEach` : 각 테스트 메서드가 종료되면서 호출되는 메서드를 정의한다.

이러한 어노테이션의 동작을 아애보기 위해 아래와 같은 코드를 작성해 보겠다. `com.springboot.test` 패키지 하위에 `TestLifeCycle.java` 파일을 생성한다.

<br>

```java
import org.junit.jupiter.api.*;

public class TestLifeCycle {

    @BeforeAll
    static void beforeAll() {
        System.out.println("## BeforeAll Annotation 호출 ##");
        System.out.println();
    }

    @AfterAll
    static void afterAll() {
        System.out.println("## AfterAll Annotation 호출 ##");
        System.out.println();
    }

    @BeforeEach
    void beforeEach() {
        System.out.println("## BeforeEach Annotation 호출 ##");
        System.out.println();
    }

    @AfterEach
    void afterEach() {
        System.out.println("## AfterEach Annotation 호출 ##");
        System.out.println();
    }

    @Test
    void test1() {
        System.out.println("## test1 시작 ##");
        System.out.println();
    }

    @Test
    void test2() {
        System.out.println("## test2 시작 ##");
        System.out.println();
    }

    @Test
    void test3() {
        System.out.println("## test3 시작 ##");
        System.out.println();
    }
    
}
```

<br>

예제를 실행하면 다음과 같은 콘솔 로그가 출력된다.

<br>

```
## BeforeAll Annotation 호출 ##

## BeforeEach Annotation 호출 ##

## test1 시작 ##

## AfterEach Annotation 호출 ##

## BeforeEach Annotation 호출 ##

## test2 시작 ##

## AfterEach Annotation 호출 ##

## AfterAll Annotation 호출 ##
```

<br>

`@BeforAll`과 `@AfterAll` 어노테이션이 지정된 메서드는 전체 테스트 동작 에서 처음과 마지막에만 각각 수행된다. `@BeforEach`와 `@AfterEach` 어노테이션이 지정된 메서드는 각 테스트가 실행될 때 `@Test` 어노테이션이 지정된 테스트 메서드를 기준으로 실행되는 것을 볼 수 있다. 또한 `test3()` 에는 `@Disabled` 어노테이션을 지정했는데, 이 어노테이션이 지정된 테스트는 실행되지 않는 것을 볼 수 있다. 



### 스프링 부트에서의 테스트

이번 장의 앞부분에서 설명한 것처럼 테스트 방식은 매우 다양하다. 전체적인 비즈니스 로직이 정상적으로 동작하는지 테스트하고 싶다면 통합 테스트를 하고, 각 모듈을 테스트하고 싶다면 단위 테스트를 해야 한다. 특히 스프링 부트를 사용하는 애플리케이션에서는 스프링 부트가 자동 지원하는 기능들을 사용하고 있기 때문에 일부 모듈에서만 단위 테스트를 수행하기 어려운 경우도 있다. 그래서 이번 장에서는 레이어별로 사용하기 적합한 방식의 테스트 가이드를 소개할 예정이다. 다만 목적에 따라 여기서 소개하는 내용이 적합할 수도 있고 적합하지 않을 수 있으니 다양한 테스트를 구성해보길 권장한다.



### 컨트롤러 객체의 테스트

컨트롤러는 클라이언트로부터 요청을 받아 요청에 걸맞은 서비스 컴포넌트로 요청을 전달하고 그 결괏값을 가공해서 클라이언트에게 응갑하는 역할을 수행한다. 즉, 애플리케이션을 구성하는 여러 레이어 중 가장 웹에 가까이 있는 모듈이라고 볼 수 있다. 이번 절에서는 프로젝트에 생성한 `ProductController`를 대상으로 `getProduct()`와 `createProduct()` 메서드에 대한 테스트 코드를 작성하겠다. 현재 `ProductController`의 `getProduct()` 메서드는 아래와 같다.

```java
@RestController
@RequestMapping("/product")
public class ProductController {

    private final ProductService productService;

    @Autowired
    public ProductController(ProductService productService){
        this.productService = productService;
    }

    @GetMapping()
    public ResponseEntity<ProductResponseDto> getProduct(Long number){
        ProductResponseDto productResponseDto = productService.getProduct(number);

        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }
    
    ... 생략 ...
}
```

<br>

`ProductController`는 `ProductService`의 객체를 의존성 주입받는다. 앞으로 나올 다른 클래스에서도 최소 1개 이상의 객체를 의존성 주입받는 코드가 등잘할 예정이다. 테스트하는 입장에서 `ProductController`만 테스트하고 싶다면 `ProductService`는 외부 요인에 해당한다. 독립적인 테스트 코드를 작성하기 위해서는 Mock 객체를 활용해야 한다. 이러한 내용을 포함해서 위 예제의 `ProductController`의 `getProduct()` 메서드를 테스트하고 싶다면 다음과 같이 테스트 코드를 작성할 수 있다. 테스트 클래스는 `test/java/com.springboot.test` 패키지에 `controller` 패키지를 생성하고 `ProductControllerTest.java` 파일로 생성한다.

![image](https://user-images.githubusercontent.com/13410737/181793734-d18eb3e8-038c-4dee-a2c6-cfadfea41aca.png){: .align-center}

<br>

```java
package com.springboot.test.controller;

import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.verify;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(ProductController.class)
public class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    ProductServiceImpl productService;

    @Test
    @DisplayName("MockMvc를 통한 Product 데이터 가져오기 테스트")
    void getProductTest() throws Exception {

        given(productService.getProduct(123L)).willReturn(
                new ProductResponseDto(123L, "pen", 5000, 2000));

        String productId = "123";

        mockMvc.perform(
                get("/product?number=" + productId))
                .andExpect(status().isOk())
                .andExpect(jsonPath(
                        "$.number").exists())
                .andExpect(jsonPath("$.name").exist())
                .andExpect(jsonPath("$.price").exists())
                .andExpect(jsonPath("$.stock").exists())
                .andDo(print());

        verify(productService).getProduct(123L);
    }
}
```

<br>

예제에서 사용된 어노테이션은 다음과 같다.



**`@WebMvcTest(테스트 대상 클래스.class)`**

- 웹에서 사용되는 요청과 응답에 대한 테스트를 수행할 수 있다. 대상 클래스만 로드해 테스트를 수행하며, 만약 대상 클래스를 추가하지 않으면 `@Controller`, `@RestController`, `@ContollerAdvice` 등의 컨트롤러 관련 빈 객체가 모두 로드된다. `@SpringBootTest`보다 가볍게 테스트하기 위해 사용된다.

**`@MockBean`**

- `@MockBean`은 실제 빈 객체가 아닌 Mock(가짜) 객체를 생성해서 주입하는 역할을 수행한다. `@MockBean`이 선언된 객체는 실제 객체가 아니기 때문에 실제 행위를 수행하지 않는다. 그렇기 때문에 해당 객체는 개발자가 Mockito의 `given()` 메서드를 통해 동작을 정의해야 한다.

**`@Test`**

- 테스트 코드가 포함돼 있다고 선언하는 어노테이션이며, JUnit Jupiter 에서는 이 어노테이션을 감지해서 테스트 계획이 포함시킨다.

**`@DisplayName`**

- 테스트 메서드의 이름이 복작해서 가독성이 떨어질 경우 이 어노테이션을 통해 테스트에 대한 표현을 정의할 수 있다.



일반적으로 `@WebMvcTest` 어노테이션을 사용한 테스트는 슬라이스(Slice) 테스트라고 부른다. 슬라이스 테스트는 단위 테스트와 통합 테스트의 중간 개념으로 이해하면 되는데, 레이어드 아키텍처를 기죽으로 각 레이어별로 나누어 테스트를 진행한다는 의미이다. 단위 테스트를 수행하기 위해서는 모든 외부 요ㅕ인을 차단하고 테스트를 진행해야 하지만 컨트롤러는 개념상 웹과 맞닿은 레이어로서 외부 요인을 차단하고 테스트하면 의미가 없기 때문에 슬라이스 테스트를 진행하는 경우가 많다.

예제 코드의 25~26번 줄에서는 `@MockBean` 어노테이션을 통해 `ProductController`가 의존성을 가지고 있던 `ProductService` 객체에 Mock 객체를 주입했다. Mock 객체에슨 테스트 과정에서 맡을 동작을 정의해야 한다. 34~35번 줄과 같이 Mockito에서 제공하는 `given()` 메서드를 통해 이 객체에서 어떤 메서드가 호출되고 어떤 파라미터를 주입받는지 가정한 후 `willReturn()` 메서드를 통해 어떤 결과를 리턴할 것인지 정의하는 구조로 코드를 작성한다. 메서드 이름에서 알 수 있듯이 이 부분의 코드가 앞에서 설명한 Given에 해당한다.

23번 줄에서 사용된 `MockMvc`는 컨트롤러의 API를 테스트하기 위해 사용된 객체이다. 정확하게는 서블릿 컨테이너의 구동 없이 가상의 MVC 환경에서 모의 HTTP 서블릿을 요청하는 유틸리티 클래스이다. 

`Perform()` 메서드를 이용하면 서버로 URL 요청을 보내는 것처럼 통신 테스트 코드를 작성해서 컨트롤러를 테스트할 수 있다. `perform()`메서드는 `MockMvcRequestBuilders`에서 제공하는 HTTP 메서드로 URL을 정의해서 사용한다. `MockMvcRequestBuilders`는 GET, POST, PUT, DELETE에 매핑되는 메서드를 제공한다. 이 메서드는 `MockHttpServletRequestBuilder` 객체를 리턴하며, HTTP 요청 정보를 설정할 수 있게 된다.

그리고 `perform()` 메서드의 결괏값으로 `ResultActions` 객체가 이런되는데 ,예제의 39~44번 줄과 같이 `andExpect()` 메서드를 사용해 결괏값 검증을 수행할 수도 있다. `andExpect()` 메서드에서는 `ResultMatcher`를 활용하는데, 이를 위해 `MockMvcResultMatchers` 클래스에 정의돼 있는 메서드를 활용해 생성할 수 있다.

요청과 응답의 전체 내용을 확인하려면 45번 줄과 같이 `andDo()` 메서드를 사용한다. `MockMvc`의 코드는 모두 합쳐져 있어 구분하기는 애매하지만 전체적인 'When-Then'의 구조를 갖추고 있음을 왁인할 수 있다.

마지막으로 `verify()` 메서드는 지정된 메서드가 실행됐는지 검증하는 역할이다. 일반적으로 `given()`에 정의된 동작과 대응한다.

<br>

> **💡 Tip.**
>
> 슬라이스 테스트를 위해 사용할 수 있는 대표적인 어노테이션은 다음과 같다. 필요한 경우에 선택적으로 사용하면 도움이 될것이다.
>
> - `@DataJdbcTest`
> - `@DataJpaTest`
> - `@DataMongoTest`
> - `@DataRedisTest`
> - `@JdbcTest`
> - `@JooqTest`
> - `@JsonTest`
> - `@RestClientTest`
> - `@WebFluxTest`
> - `@WebMvcTest`
> - `@WebServiceClientTest`

<br>

다음으로 `createProduct()` 메서드의 테스트 코드를 작성하겠다. `ProductController`에 작성돼 있는 `createProduct()` 메서드는 다음과 같이 구현돼 있다.

<br>

```java
@RestController
@RequestMapping("/product")
public class ProductController {

    private final ProductService productService;

    @Autowired
    public ProductController(ProductService productService){
        this.productService = productService;
    }

    @PostMapping()
    public ResponseEntity<ProductResponseDto> createProduct(@RequestBody ProductDto productDto){
        ProductResponseDto productResponseDto = productService.saveProduct(productDto);

        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }
}
```

<br>

예제의 `createProduct()` 메서드는 `@RequestBody`로 값을 받고 있다. 이에 따른 테스트 코드는 아래와 같이 작성할 수 있다.

<br>

```java
package com.springboot.test.controller;

import com.google.gson.Gson;
import com.springboot.test.data.dto.ProductDto;
import com.springboot.test.data.dto.ProductResponseDto;
import com.springboot.test.service.impl.ProductServiceImpl;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;

import static org.mockito.BDDMockito.given;
import static org.mockito.Mockito.verify;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.result.MockMvcResultHandlers.print;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@WebMvcTest(ProductController.class)
public class ProductControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    ProductServiceImpl productService;

    @Test
    @DisplayName("Product 데이터 생성 테스트")
    void createProductTest() throws Exception {
        //Mock 객체에서 특정 메서드가 실행되는 경우 실제 Return을 줄 수 없기 때문에 아래와 같이 가정 사항을 만들어 줌
        given(productService.saveProduct(new ProductDto("pen", 5000, 2000)))
                .willReturn(new ProductResponseDto(12315L, "pen", 5000, 2000));

        ProductDto productDto = Product.builder()
                .name("pen")
                .price(5000)
                .stock(2000)
                .build();

        Gson gson = new Gson();
        String content = gson.toJson(productDto);

        mockMvc.perform(
                post("/product")
                        .content(content)
                        .contentType(MediaType.APPLICATION_JSON))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.number").exists())
                .andExpect(jsonPath("$.name").exists())
                .andExpect(jsonPath("$.price").exists())
                .andExpect(jsonPath("$.stock").exists())
                .andDo(print());

        verify(productService).saveProduct(new ProductDto("pen", 5000, 2000));
    }
    
}
```

<br>

여기서 사용된 `ProductDto`는 아래와 같다.

<br>

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ProductDto {

    private String name;
    
    private int price;

    private int stock;

}

```

<br>

예제 코드를 실행하려면 `pom.xml` 파일에 <a href="https://github.com/google/gson" target="_blank">Gson</a>에 대한 의존성을 추가해야 한다. Gson은 구글에서 개발한 JSON 파싱 라이브러리로서 자바 객체를 JSON 문자열로 변환하거나 JSON 문자열을 자바 객체로 변환하는 역할을 한다. `ObjectMapper`를 사용해도 무관하지만 여기서는 현업에서 많이 사용하는 라이브러리를 소개하기 위해 사용했다. Gson의 의존성은 아래와 같이 추가한다.

<br>

```xml
<dependencies>
    
    .. 내용 생략 ..
    
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
    </dependency>
    
    .. 내용 생략 ..
    
</dependencies>
```

<br>

`createProduct()`를 테스트하는 코드는 `getProduct()`를 테스트하는 코드와 거의 비슷하게 구성돼 있다. 35~36번 줄에서 `given()` 메서드를 통해 `ProductService`의 `saveProduct()`메서드의 동작 규칙을 설정하고, 38~42번 줄에서는 테스트에 필요한 객체를 생성한다. 실제 테스트를 수행하는 코드는 47~56번 줄로, 리소스 생성 기능을 테스트하기 때문에 `post` 메서드를 통해 URL을 구성한다. 그리고 `@RequestBody`의 값을 넘겨주기 위해 `content()` 메서드에 DTO의 값을 담아 테스트를 진행한다. 마지막으로 POST 요청을 통해 도출된 결괏값에 대한 각 항목이 존재하는지 `jsonPath().exists()`를 통해 검증한다. 검증한 결과, 대응하는 값이 없다면 오류가 발생하게 된다.

<br>

> **💡 Tip.**
>
> Mockito에 대한 자세한 내용은 공식 사이트에서 확인할 수 있다.
>
> - <a href="https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html" target="_blank">https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html</a>



### 서비스 객체의 테스트

이번에는 서비스 레이어에 해당하는 `ProductService` 객체를 테스트하겠다. 앞에서도 언급했듯이 실습 예제에서는 DAO의 역할이 명확하게 드러나지 않기 때문에 DAO 객체는 생략한다.

먼저 `getProduct()` 메서드에 대해 테스트 코드를 작성하겠다. 아무 의존성을 주입받지 않은 상태에서 단위 테스트를 작성하면 아래와 같이 작성할 수 있따. 단위 테스트를 수행할 클래스는 `test/java/com.springboot.test` 내에 `service/impl` 패키지를 생성하고 `ProductServiceTest.java` 파일을 생성한다.

![image](https://user-images.githubusercontent.com/13410737/181803305-0a7e5dd5-f1dd-4e9b-affb-98436a42576a.png){: .align-center}

<br>

```java
package com.springboot.test.service.impl;

import com.springboot.test.data.dto.ProductResponseDto;
import com.springboot.test.data.entity.Product;
import com.springboot.test.data.repository.ProductRepository;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import java.util.Optional;

import static org.mockito.Mockito.verify;

public class ProductServiceTest {

    private ProductRepository productRepository = Mockito.mock(ProductRepository.class);
    private ProductServiceImpl productService;

    @BeforeEach
    public void setUpTest(){
        productService = new ProductServiceImpl(productRepository);
    }

    @Test
    void getProductTest(){
        Product givenProduct = new Product();
        givenProduct.setNumber(123L);
        givenProduct.setName("펜");
        givenProduct.setPrice(1000);
        givenProduct.setStock(1234);

        Mockito.when(productRepository.findById(123L))
                .thenReturn(Optional.of(givenProduct));

        ProductResponseDto productResponseDto = productService.getProduct(123L);
        Assertions.assertEquals(productResponseDto.getNumber(), givenProduct.getNumber());
        Assertions.assertEquals(productResponseDto.getName(), givenProduct.getName());
        Assertions.assertEquals(productResponseDto.getPrice(), givenProduct.getPrice());
        Assertions.assertEquals(productResponseDto.getStock(), givenProduct.getStock());

        verify(productRepository).findById(123L);
    }
    
}
```

<br>

단위 테스트를 위해서는 외부 요인을 모두 배제하도록 코드를 작성해야 한다. 이번 예제에서는 `@SpringBootTest`, `@WebMvcTest` 등의 `@...Test` 어노테이션이 선언돼 있지 않다.

이후 17번 줄을 보면 Mockito의 `mock()` 메서드를 통해 Mock 객체로 `ProductRepository`를 주입 받았다. 이 객체를 기반으로 20~23번 줄과 같이 각 테스트 전에 `ProductService` 객체를 초기화해서 사용한다.

25~44번 줄은 본격적인 테스트 코드이다. 테스트 코드를 Given-When-Then 패턴을 기반으로 작성됐다. Given 구문에 해당하는 27~34번 줄에서는 테스트에 사용될 `Product` 엔티티 객체를 생성하고 `ProductRepository`의 동작에 대한 결괏값 리턴을 설정한다.

그리고 36번 줄에서 테스트하고자 하는 `ProductService`의 `getProduct()` 메서드를 호출해서 동작을 테스트한다.

테스트에서 리턴 받은 `ProductResponseDto` 객체에 대해서 38~41번 줄과 같이 Assertion을 통해 값을 검증함으로써 테스트의 목적을 달성하는지 확인한다. 마지막으로는 검증 보완을 위해 42번 줄과 같이 `verify()` 메서드로 부가 검증을 시도한다.

이어서 `saveProduct()` 메서드에 대한 테스트 코드를 작성하겠다.

<br>

```java
package com.springboot.test.service.impl;

import com.springboot.test.data.dto.ProductDto;
import com.springboot.test.data.dto.ProductResponseDto;
import com.springboot.test.data.entity.Product;
import com.springboot.test.data.repository.ProductRepository;
import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.Mockito;

import static org.mockito.AdditionalAnswers.returnsFirstArg;
import static org.mockito.ArgumentMatchers.any;
import static org.mockito.Mockito.verify;

public class ProductServiceTest {

    private ProductRepository productRepository = Mockito.mock(ProductRepository.class);
    private ProductServiceImpl productService;

    @BeforeEach
    public void setUpTest(){
        productService = new ProductServiceImpl(productRepository);
    }

    @Test
    void saveProductTest(){
        Mockito.when(productRepository.save(any(Product.class)))
                .then(returnsFirstArg());

        ProductResponseDto productResponseDto = productService.saveProduct(
                new ProductDto("펜", 1000, 1234));

        Assertions.assertEquals(productResponseDto.getName(), "펜");
        Assertions.assertEquals(productResponseDto.getPrice(), 1000);
        Assertions.assertEquals(productResponseDto.getStock(), 1234);

        verify(productRepository).save(any());
    }
}
```

<br>

예제에서 살펴볼 내용은 28번 줄의 `any()`이다. `any()`는 Mockito 의 `ArgumentMatchers`에서 제공하는 메서드로서 Mock 객체의 동작을 정의하거나 검증하는 단계에서 조건으로 특정 매개변수의 전달을 설정하지 않고 메서드의 실행만을 확인하거나 좀 더 큰 범위의 클래스 객체를 매개변수로 전달받는 등의 상황에 사용한다. 28번 줄에서 `any(Product.class)`로 동작을 설정했는데, 일반적으로 `given()`의로 정의된 Mock 객체의 메서드 동작 감지는 매개변수의 비교를 통해 이뤄지거나 레퍼런스 변수의 비교는 주솟값으로 이뤄지기 때문에 `any()`를 사용해 클래스만 정의하는 경우도 있다.

지금까지 소개한 테스트는 Mock 객체를 활용한 방식이었다. 큰 차이는 없지만 Mock 객체를 직접 생성하지 않고 `@MockBean` 어노테이션을 사용해 스프링 컨테이너에 Mock 객체를 주입 받는 방식을 소개하겠다.

<br>

```java
@ExtendWith(SpringExtension.class)
@Import({ProductServiceImpl.class})
class ProductServiceTest2 {
    
    @MockBean
    ProductRepository productRepository;
    
    @Autowired
    ProductService productService;
    
    ... 테스트 코드 생략 ...
}
```

<br>

동작은 설정하는 `ProductRepository`에 대한 초기화 작업을 어떻게 진행하는지는 비교하기 위한 코드이다. 이전 예제에서는 Mockito를 통해 리포지토리를 Mock 객체로 대체하는 작업을 수행하고 서비스 객체를 직접 초기화했다.

반면 이번 예제에서는 스프링에서 제공하는 테스트 어노테이션을 통해 Mock 객체를 생성하고 의존성 주입을 받고 있다. 둘의 차이라면 스프링의 기능에 의존하느냐 의존하지 않느냐의 차이 뿐이다. 두 예제 모두 Mock 객체를 활용한 테스트 방식인 것은 동일하나 `@MockBean`을 사용하는 방식은 스프링에 Mock객체를 등록해 주입받는 형식이며 `Mockito.mock()`을 사용하는 방식은 스프링 빈에 등록하지 않고 직접 객체를 초기화해서 사용하는 방식이다. 둘 다 테스트 속도에는 큰 차이는 없지만 아무래도 스프링을 사용하지 않는 Mock 객체를 직접 생성하는 방식이 더 빠르게 동작한다.

위 예제에서는 스프링에서 객체를 주입받기 위해 `@ExtendWith(SpringExtension.class)` 를 사용해 JUnit 5의 테스트에서 스프링 테스트 컨텍스트를 사용하도록 설정한다. 그리고 8번 줄에서 `@Autowired` 어노테이션으로 주입받는 `ProductService`를 주입받기 위해 직접 2번 줄에서 클래스를 `@Import` 어노테이션을 통해 사용한다.

<br>

> **💡 Tip.**
>
> SpringExtension 클래스는 JUnit 5의 Jupiter 테스트에 스프링 테스트 컨텍스트  프레임워크(Spring TestContextFramework)를 통합하는 역할을 수행한다. 자세한 내용은 다음 URL을 참고한다.
>
> - <a href="https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/junit/jupiter/SpringExtension.html" target="_blank">https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/test/context/junit/jupiter/SpringExtension.html</a>

<br>



### 리포지토리 객체의 테스트

리포지토리는 개발자가 구현하는 레이어 중에서 가장 데이터베이스와 가깝다. 그리고 `JpaRepository`를 상속받아 기본적인 쿼리 메서드를 사용할 수 있다. 그렇게 때문에 리포지토리 테스트는 특히 구현하는 목적에 대해 고민하고 작성해야 한다.

리포지토리 객체의 테스트 코드를 작성할 때 고려할 내용을 몇 가지 이야기하겠다. 먼저 `findById()`, `save()` 같은 기본 메서드에 대한 테스트는 큰 의미가 없다. 리포지토리의 기본 메서드는 테스트 검증을 마치고 제공된 것이기 때문이다.

데이터베이스의 연동 여부는 테스트 시 고려해 볼 사항이다. 굳이 따지자면 데이터베이스는 외부 요인에 속한다. 만약 단위 테스트를 고려한다면 데이터베이스를 제외할 수 있다. 혹은 테스트용으로 다른 데이터베이스를 사용하는 경우도 있다. 왜냐하면 데이터베이스를 사용한 테스트는 테스트 과정에서 데이터베이스에 테스트 데이터가 적재되기 때문이다. 그렇기 때문에 데이터베이스를 연동한 테스트는 테스트 데이터를 제거하는 코드까지 포함해서 작성하는 것이 좋다. 다만 이처럼 테스트 데이터의 적재를 신경 써야 하는 테스트 환경이라면 잘못된 테스트 코드가 실행되면서 발생할 수 있는 사이드 이펙트를 고려해서 데이터베이스 연동 없이 테스트하는 편이 좋을 수도 있다.

이 책에서는 마리아DB를 사용한다. 여기서는 데이터베이스를 제외한 테스트 상황을 가정해서 테스트 데이터베이스로 <a href="https://www.h2database.com" target="_blank">H2 DB</a> 를 사용하는 방법을 간략하게 소개하고 기본 테스트 환경에서는 마리아DB를 그래도 사용할 예정이다. 그리고 실습을 위한 코드이므로 `JpaRepository`에서 제공하는 기본 메서드로 예제를 진행하겠다.

먼저 H2 DB를 사용한 테스트 코드를 작성하겠다. 별도의 설정이 없다면 테스트 환경에서는 임베디드 데이터베이스를 사용한다. H2 DB를 사용하려면 `pom.xml` 파일에 아래와 같이 의존성을 추가해야 한다.

<br>

```xml
<dependencies>
    ...생략...
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <scope>test</scope>
    </dependency>
    ...생략...
</dependencies>
```

<br>

데이터베이스에 값을 저장하는 테스트 코드는 아래와 같이 작성한다. 테스트 코드를 작성하기 위해 `test/com.springboot.test` 내에 `data/repository` 패키지를 생성한 후 `ProductRepositoryTestByH2.java` 파일을 생성한다.

<br>

```java
@DataJpaTest
public class ProductRepositoryTestByH2 {

    @Autowired
    private ProductRepository productRepository;

    @Test
    void saveTest() {
        // given
        Product product = new Product();
        product.setName("펜");
        product.setPrice(1000);
        product.setStock(1000);

        // when
        Product savedProduct = productRepository.save(product);

        // then
        assertEquals(product.getName(), savedProduct.getName());
        assertEquals(product.getPrice(), savedProduct.getPrice());
        assertEquals(product.getStock(), savedProduct.getStock());
    }
}
```

<br>

위 예제의 1번 줄에서는 `@DataJpaTest` 어노테이션을 사용하고 있다. `@DataJpaTest`는 다음과 같은 기능을 제공한다.

- JPA와 관련된 설정만 로드해서 테스트를 진행한다.
- 기본적으로 `@Transactional` 어노테이션을 포함하고 있어 테스트 코드가 종료되면 자동으로 데이터베이스의 롤백이 진행된다.
- 기본값으로 임베디드 데이터베이스를 사용한다. 다른 데이터베이스를 사용하라면 별도의 설정을 거쳐 사용 가능하다.



`@DataJpaTest` 어노테이션을 선언했기 때문에 4~5번 줄에서는 리포지토리를 정상적으로 주입받을 수 있다.

7~22번 줄은 Given-When-Then 패턴으로 작성된 테스트 코드이다. Given 구문에서는 테스트에서 사용할 `Product` 엔티티를 만들고, When 구문에서 생성된 엔티티를 기반으로 `save()` 메서드를 호출해서 테스트를 진행한다. 이후 정상적인 테스트가 이뤄졌는지 체크하기 위해 `save()` 메서드의 리턴 객체와 Given에서 생성한 엔티티 객체의 값이 일치하는지 `assertEquals()` 메서드를 통해 검증한다.

데이터 조회에 대한 테스트는 아래와 같다.

<br>

```java
@DataJpaTest
public class ProductRepositoryTestByH2 {

    @Autowired
    private ProductRepository productRepository;

    @Test
    void selectTest() {
        // given
        Product product = new Product();
        product.setName("펜");
        product.setPrice(1000);
        product.setStock(1000);

        Product savedProduct = productRepository.saveAndFlush(product);

        // when
        Product foundProduct = productRepository.findById(savedProduct.getNumber()).get();

        // then
        assertEquals(product.getName(), foundProduct.getName());
        assertEquals(product.getPrice(), foundProduct.getPrice());
        assertEquals(product.getStock(), foundProduct.getStock());
    }
}
```

<br>

데이터베이스 조회 테스트를 위해서는 먼저 데이터베이스에 테스트 데이터를 추가해야 한다. 따라서 15번 줄과 같이 Given 절에서 객체를 데이터베이스에 저장하는 작업을 수행한다. 그후 18번 줄의 조회 메서드를 호출해서 테스트를 진행하고 이후 코드에서 데이터를 비교하며 검증을 수행한다.

위의 두 예제 테스트를 실행하면  H2 DB에서 실행되는 것을 확인할 수 있다. 기존에 가용하고 있던 마리아DB에서 테스트하기 위해서는 별도의 설정이 필요하다. 이번 실습을 위해 같은 패키지 경로에 `ProductRepositoryTest.java` 파일을 생성하고 아래와 같이 코드를 작성한다.

<br>

```java
@DataJpaTest
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
public class ProductRepositoryTest {

    @Autowired
    private ProductRepository productRepository;

    @Test
    void save() {
        Product product = new Product();
        product.setName("펜");
        product.setPrice(1000);
        product.setStock(1000);

        Product savedProduct = productRepository.save(product);

        assertEquals(product.getName(), savedProduct.getName());
        assertEquals(product.getPrice(), savedProduct.getPrice());
        assertEquals(product.getStock(), savedProduct.getStock());
    }

}
```

<br>

위 예제의 2번 줄에 있는 `replace` 요소는 `@AutoConfigureTestDatabase` 어노테이션의 값을 조정하는 작업을 수행한다. `replace` 속성의 기본값은 `Replcae.ANY` 이며, 이 경우 임베디드 메모리 데이터베이스를 사용한다. 이 속성값을 `Replace.NONE` 으로 변경하면 애플리케이션에서 실제로 사용하는 데이터베이스로 테스트가 가능하다.

그리고 지금까지 다룬 `@DataJpaTest`를 사용하지 않고 `@SpringBootTest` 어노테이션으로도 테스트할 수 있다. 이번 실습을 위해 같은 패키지 경로에 `ProductRepositoryTest2.java` 파일을 생성한다. `@SpringBootTest` 어노테이션을 사용한 CRUD 테스트는 아래와 같다.

<br>

```java
@SpringBootTest
public class ProductRepositoryTest2
{
    @Autowired
    ProductRepository productRepository;

    @Test
    public void basicCRUDTest() {
        /* create */
        // given
        Product givenProduct = Product.builder()
                .name("노트")
                .price(1000)
                .stock(500)
                .build();

        // when
        Product savedProduct = productRepository.save(givenProduct);

        // then
        Assertions.assertThat(savedProduct.getNumber())
                .isEqualTo(givenProduct.getNumber());
        Assertions.assertThat(savedProduct.getName())
                .isEqualTo(givenProduct.getName());
        Assertions.assertThat(savedProduct.getPrice())
                .isEqualTo(givenProduct.getPrice());
        Assertions.assertThat(savedProduct.getStock())
                .isEqualTo(givenProduct.getStock());

        /* read */
        // when
        Product selectProduct = productRepository.findById(savedProduct.getNumber())
                .orElseThrow(RuntimeException::new);

        // then
        Assertions.assertThat(selectProduct.getNumber())
                .isEqualTo(givenProduct.getNumber());
        Assertions.assertThat(selectProduct.getName())
                .isEqualTo(givenProduct.getName());
        Assertions.assertThat(selectProduct.getPrice())
                .isEqualTo(givenProduct.getPrice());
        Assertions.assertThat(selectProduct.getStock())
                .isEqualTo(givenProduct.getStock());

        /* update */
        // when
        Product foundProduct = productRepository.findById(selectProduct.getNumber())
                .orElseThrow(RuntimeException::new);

        foundProduct.setName("장난감");

        Product updatedProduct = productRepository.save(foundProduct);

        // then
        assertEquals(updatedProduct.getName(), "장난감");

        /* delete */
        // when
        productRepository.delete(updatedProduct);

        // then
        assertFalse(productRepository.findById(selectProduct.getNumber()).isPresent());
    }

}
```

<br>

이 예제에서는 CRUD의 모든 기능을 한 테스트 코드에 작성했다. 기본 메서드를 테스트하기 때문에 Given 구문을 한 번만 사용해 전체 테스트에 활용했다. `@SpringBootTest`어노테이션을 활용하면 스프링의 모든 설정을 가져오고 빈 객체도 전체를 스캔하기 때문에 의존성 주입에 대해 고민할 필요 없이 테스트가 가능하다. 다만 테스트의 속도가 느리므로 다른 방법으로 테스트할 수 있다면 대안을 고려해보는 것이 좋다.
