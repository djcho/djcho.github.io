---
title : "[SpringBoot][Chapter10.3] 유효성 검사와 예외처리 - 스프링 부트에서의 유효성 검사"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Validator]
toc: true
toc_sticky : true
published : true
date : 2022-08-07
last_modified_at : 2022-08-07
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





## 스프링 부트에서의 유효성 검사

지금부터 애플리케이션에 유효성 검사 기능을 추가하겠다. 기본 프로젝트 뼈대는 7장에서 사용한 패키지와 클래스 구조를 그대로 가져와 만들겠다.



### 프로젝트 생성

이번 장에서 사용할 새로운 프로젝트를 생성하겠다. 스프링 부트 버전은 이전과 같은 2.5.6 버전으로 지정하고, 아래와 같은 내용들을 설정한다.

- groupId : com.springboot
- artifactId : valid_exception
- name : valid_exception
- Developer Tools : Lombok, Spring Configuration Processor
- Web : Spring Web
- SQL : Spring Data JPA, MariaDB Driver

<br>

그리고 7장에서 다음과 같이 자바 파일을 가져와 기본적인 프로젝트를 생성한다. 그리고 5장에서 다룬 것처럼 `SwaggerConfiguration`클래스 파일을 생성하고 관련 의존성을 `pom.xml`에 추가한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183295576-e0a7789e-8e15-4e1d-a1f5-a14eb10d7e1f.png){: .align-center}

<br>

### 스프링 부트용 유효성 검사 관련 의존성 추가

원래 스프링 부트의 유효성 검사 기능은 spring-boot-starter-web에 포함돼 있었다. 하지만 스프링 부트 2.3 버전 이후로 별도의 라이브러리로 제공하고 있다. 아래와 같이 `pom.xml`파일에 유효성 검사 라이브러리를 의존성으로 추가하면 사용할 수 있다.

<br>

```xml
<dependencies>
    ...생략...
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
    ...생략...
</dependencies>
```



### 스프링 부트의 유효성 검사

유효성 검사는 각 계층으로 데이터가 넘어오는 시점에 해당 데이터에 대한 검사를 실시한다. 스프링 부트 프로젝트에서는 계층 간 데이터 전송에 대체로 DTO 객체를 활용하고 있기 때문에 아래 그림과 같이 유효성 검사를 DTO 객체를 대상으로 수행하는 것이 일반적이다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183296199-b6d294c8-7a61-47ff-9faf-82712412e28c.png){: .align-center}

<br>

이번 장의 실습을 위한 DTO와 컨트롤러를 생성하겠다. 먼저 `ValidRequestDto`라는 이름의 DTO 객체를 아래와 같이 생성한다.

<br>

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ValidRequestDto {

    @NotBlank
    String name;

    @Email
    String email;

    @Pattern(regexp = "01(?:0|1|[6-9])[.-]?(\\d{3}|\\d{4})[.-]?(\\d{4})$")
    String phoneNumber;

    @Min(value = 20) @Max(value = 40)
    int age;

    @Size(min = 0, max = 40)
    String description;

    @Positive
    int count;

    @AssertTrue
    boolean booleanCheck;
    
}
```

<br>

예제를 보면 각 필드에 어노테이션이 선언된 것을 볼 수 있다. 각 어노테이션은 유효성 검사를 위한 조건을 설정하는 데 사용된다. 대표적인 어노테이션은 다음과 같다.

<br>

#### 문자열 검증

- `@Null` : null 값만 허용한다.
- `@NotNull` : null을 허용하지 않는다. "", " "는 허용한다.
- `@NotEmpty` : null, ""을 허용하지 않는다. " "는 허용한다.
- `@NotBlank` : null, "", " "을 허용하지 않는다.

#### 최댓값 / 최솟값 검증

- `BigDecimal`, `BigInteger`, `int`, `long` 등의 타입을 지원한다.
- `@DemicalMax(value = "$numberString")` : `$numberString` 보다 작은 값을 허용한다.
- `@DemicalMin(value = "$numberString")` : `$numberString` 보다 큰 값을 허용한다.
- `@Min(value = $number)` : `$number` 이상의 값을 허용한다.
- `@Max(value = $number)` : `$number` 이하의 값을 허용한다.

#### 값의 범위 검증

- `BigDecimal`, `BigInteger`, `int`, `long` 등의 타입을 지원한다.
- `@Positive` : 양수를 허용한다.
- `@PositiveOrZero` : 0을 포함한 양수를 허용한다.
- `@Nagative` : 음수를 허용한다.
- `@NagetiveOrZero` : 0을 포함한 음수를 허용한다.

#### 시간에 대한 검증

- `Date`, `LocalDate`, `LocalDateTime`등의 타입을 지원한다.
- `@Future` : 현재보다 미래의 날짜를 허용한다.
- `@FutureOrPresent` : 현재를 포함한 미래의 날짜를 허용한다.
- `@Past` : 현재보다 과거의 날짜를 허용한다.
- `@PastOrPresent` : 현재를 포함한 과거의 날짜를 허용한다.

#### 이메일 검증

- `@Email` : 이메일 형식을 검사한다. ""는 허용한다.

#### 자릿수 범위 검증

- `BigDecimal`, `BigInteger`, `int`, `long` 등의 타입을 지원한다.
- `@Digits(integer = $number1, fraction = $number2)` : `$number1`의 정수 자릿수와 `$number2`의 소수 자릿수를 허용한다.

#### Boolean 검증

- `@AssertTrue` : `true`인지 체크한다. `null`값은 체크하지 않는다.
- `@AssertFalse` : `false`인지 체크한다. `null`값은 체크하지 않는다.

#### 문자열 길이 검증

- `@Size(min = $number1, max = $number2)` : `$number1` 이상 `$number2` 이하의 범위를 허용한다.

#### 정규식 검증

- `@Pattern(regexp = "$expression")` : 정규식을 검사한다 정규식은 자바의 `java.util.regex.Pattern` 패키지의 컨벤션을 따른다.

<br>

유효성 검사에 사용하는 어노테이션은 인텔리제이 IDEA에서도 확인할 수 있다. 화면 우측의 [Bean Validation] 탭을 클릭하면 아래와 같은 목록을 확인할 수 있다.

<br>

![image-20220808000017341](/Users/n21808003_djcho/Library/Application Support/typora-user-images/image-20220808000017341.png){: .align-center}

<br>

인텔리제이 IDEA에서는 자동으로 우측에 [Bean Validation] 탭을 추가하는 기능이 있는데, 만약 화면에서 이를 확인할 수 없다면 메뉴에서 [New] → [Tool Windows] → [Bean Validation]을 차례로 선택해 탭을 추가할 수 있다.



> **💡 Tip.**
>
> 실습에서 사용한 스프링 부트 2.5.6 버전은 Hibernate Validater 6.2.0.Final 버전을 사용하고 있다. 이 버전은 Jakarta Bean Validation 2.0의 스펙을 따르고 있으며, 관련 내용은 다음 URL에서 확인할 수 있다.
>
> - <a href="https://beanvalidation.org/2.0-jsr380/spec/" target="_blank">https://beanvalidation.org/2.0-jsr380/spec/</a>

<br>

다음으로 앞에서 생성한 DTO를 사용하는 컨트롤러 객체를 생성하겠다. 아래와 같이 `ValidationController`를 생성한다.

<br>

```java
@RestController
@RequestMapping("/validation")
public class ValidationController {

    private final Logger LOGGER = LoggerFactory.getLogger(ValidationController.class);

    @PostMapping("/valid")
    public ResponseEntity<String> checkValidationByValid(
            @Valid @RequestBody ValidRequestDto validRequestDto) {
        LOGGER.info(validRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validRequestDto.toString());
    }
}
```

<br>

예제에서 `checkValidationByValid()` 메서드는 `ValidRequestDto` 객체를 `RequestBody` 값으로 받고 있다. 이 경우 9번 줄과 같이 `@Valid` 어노테이션을 지정해야 DTO 객체에 대해 유효성 검사를 수행한다.

동작을 확인하기 위해 애플리케이션을 실행해 Swagger 페이지에 접속한다. 위의 `checkValidationByValid()` 메서드를 호출하기 위해 각 값을 아래와 같이 입력한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183297930-658b52a7-c789-4342-909d-96f9ab85911d.png){: .align-center}

```json
{
  "age": 30,
  "booleanCheck": true,
  "count": 1,
  "description": "Validation 실습 데이터입니다.",
  "email": "flature@wikibooks.co.kr",
  "name": "Flature",
  "phoneNumber": "010-1234-5678"
}
```

<br>

