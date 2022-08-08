---
title : "[SpringBoot][Chapter10.3] 유효성 검사와 예외처리 - 스프링 부트에서의 유효성 검사"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Validator]
toc: true
toc_sticky : true
published : true
date : 2022-08-08
last_modified_at : 2022-08-08
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

![image](https://user-images.githubusercontent.com/13410737/183298172-ee21c36d-b822-4e36-b13b-2ea4d6488d8d.png){: .align-center}

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

위의 값은 위에서 설정한 유효성 검사를 통과할 수 있는 값들이다. 먼저 `age`는 `@Min(value=20)`, `@Max(value=40)`으로 값이 20살 이상, 40살 이하인 데이터만 받겠다는 것을 의미한다. `booleanCheck`는 `@AssertTrue`를 통해 값이 `true`인지 체크한다. `count`에는 `@positive`가 설정돼 있으므로 0이 아닌 양수가 값으로 들어오는지 체크한다. `description`은 `@Size`를 통해 문자열의 길이를 제한했다. `@Email` 어노테이션을 설정한 `email`필드에서는 값에 '`@`' 문자가 있는지 확인한다.(실습에서는 이 정도로만 설정하지만 실무에서는 도메인을 검사하거나 비정상적인 이메일인지 검토하는 추가 설정이 필요할 수 있다.) `name`은 `@NotBlank`로 `null`값이나 "", " " 모두 허용하지 않게 설정해서 값을 의무적으로 받도록 설정했다. `phoneNumber`는 `@Pattern`을 통해 정규식을 설정했다. `regexp` 속성의 값을 `"01(?:0|1|[6-9])[.-]?(\\d{3}|\\d{4})[.-]?(\\d{4})$"`로 설정하면서 휴대전화 형식인지 검증할 수 있다.

위 그림처럼 Swagger에서 애플리케이션을 호출하면 별 문제없이 '200 OK'로 응갑하는 것을 볼 수 있다. 이번에는 앞에서 설정한 규칙에 벗어나는 값으로 변경해 호출해보겠다. `age`를 -1로 설정하고 호출하면 아래와 같이 400 에러가 발생한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183426167-f5578db8-e7b0-492c-bbbd-4c124a46a6e6.png){: .align-center}

<br>

다음과 같이 애플리케이션의 로그에 로그가 출력되어 문제가 발생한 지점을 확인할 수 있다. 그러나 아직 별도의 예외 처리를 하지 않았기 때문에 에러가 발생했을 때 클라이언트는 어디에서 에러가 발생했는지 확인할 수가 없다.

<br>

```
[Field error in object 'validRequestDto' on field 'age': rejected value [-1]; codes [Min.validRequestDto.age,Min.age,Min.int,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validRequestDto.age,age]; arguments []; default message [age],20]; default message [20 이상이어야 합니다]] ]
```

<br>

2개 이상의 유효성 검사를 통과하지 못한 경우에는 다음과 같이 검사를 실패한 개수를 로그에 포함시킨다. 이번에는 `count`의 값도 -1로 설정해서 유효성 검사를 실패하게 했다.

<br>

```
with 2 errors: [Field error in object 'validRequestDto' on field 'count': rejected value [-1]; codes [Positive.validRequestDto.count,Positive.count,Positive.int,Positive]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validRequestDto.count,count]; arguments []; default message [count]]; default message [0보다 커야 합니다]] [Field error in object 'validRequestDto' on field 'age': rejected value [-1]; codes [Min.validRequestDto.age,Min.age,Min.int,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validRequestDto.age,age]; arguments []; default message [age],20]; default message [20 이상이어야 합니다]] ]
```

<br>

출력 결과를 보며 'with 2 errors'라는 표현으로 두 요소에서 에러가 발생했다는 것이 명시돼 있다. 이후 내용에서는 각각 'default massage'로 어느 부분이 잘못됐는지 더욱 쉽게 확인할 수 있다.



>**💡 Tip. 정규식(Regular Express)란?**
>
>특정한 규칙을 가진 문자열 집합을 표현하기 위해 쓰이는 형식이다. 전화번호, 주민등록번호, 이메일과 같이 특정 형식의 값을 검증해야 할 때가 있다. 이러한 값은 정규식을 통해 쉽게 검증할 수 있다.
>
>정규식은 다음과 같은 요소를 사용한다.
>
>- `^` : 문자열의 시작
>- `$` : 문자열의 종료
>- `.` : 임의의 한 문자
>- `*` : 앞 문자가 없거나 무한정 많음
>- `+` : 앞 문자가 하나 이상
>- `?` : 앞 문자가 업거나 하나 존재
>- `[, ]` : 문자의 집합이나 범위를 나타내며, 두 문자 사이는 ~ 기호로 범위를 표현
>- `{, }` : 횟수 또는 범위를 의미
>- `(, )` : 괄호 안의 문자를 하나의 문자로 인식
>- `|` : 패턴 안에서 OR 연산을 수행
>- `\` : 정규식에서 역슬래시는 확장문자로 취급하고, 역슬래시 다음에 특수문자가 오면 문자로 인식
>- `\b` : 단어의 경계
>- `\B` : 단어가 아닌 것에 대한 경계
>- `\A` : 입력의 시작 부분
>- `\G` : 이전 매치의 끝
>- `\Z` : 종결자가 있는 경우 입력의 끝
>- `\z` : 입력의 끝
>- `\s` : 공백 문자
>- `\S` : 공백 문자가 아닌 나머지 문자(`^\s`와 동일)
>- `\w` : 알파벳이나 숫자
>- `\W` : 알파벳이나 숫자가 아닌 문자(`^\w`와 동일)
>- `\d` : 숫자 [0-9]와 동일하게 취급
>- `\D` : 숫자를 제외한 모든 문자(`^0-9`와 동일)
>
>정규식은 익숙하지 않은 문자의 조합으로 구성되기 때문에 다음과 같은 사이트에서 직접 정규식을 만들어보며 연습하는 것을 권장한다.
>
>- <a href="https://regexr.com/" target="_blank">https://regexr.com/</a>
>- <a href="https://regex101.com/" target="_blank">https://regex101.com/</a>



### @Validated 활용

앞의 예제에서는 유효성 검사를 수행하기 위해 `@Valid` 어노테이션을 선언했다. `@Valid` 어노테이션은 자바에서 지원하는 어노테이션이며, 스프링도 `@Validated`라는 별도의 어노테이션으로 유효성 검사를 지원한다. `@Valid`는 `@Validated` 어노테이션의 기능을 포함하고 있기 때문에 `@Validated`로 변경할 수 있다. 또한 `@Validated`는 유효성 검사를 그룹으로 묶어 대상을 특정할 수 있는 기능이 있다.

검증 그룹은 별다른 내용이 없는 마커 인터페이스를 생성해서 사용한다 실습을 위해 그림과 같이 `data` 패키지 내에 `group` 패키지를 생성하고 `ValidationGroup1`과 `ValidationGroup2`라는 인터페이스를 생성한다.

<br>

```java
public interface ValidationGroup1 {
    
}
```

```java
public interface ValidationGroup2 {
    
}
```

<br>

검증 그룹 설정은 DTO 객체에서 한다. 아래와 같이 새로운 DTO 객체를 생성한다.

<br>

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ValidatedRequestDto {

    @NotBlank
    private String name;

    @Email
    private String email;

    @Pattern(regexp = "01(?:0|1|[6-9])[.-]?(\\d{3}|\\d{4})[.-]?(\\d{4})$")
    private String phoneNumber;

    @Min(value = 20, groups = ValidationGroup1.class) 
    @Max(value = 40, groups = ValidationGroup1.class)
    int age;
    
    @Size(min = 0, max = 40)
    private String description;
    
    @Positive(groups = ValidationGroup2.class)
    private int count;
    
    @AssertTrue
    private boolean booleanCheck;
    
}
```

<br>

이 전 코드와 비쇼했을 때 수정된 부분은 17~18번 줄과 24번 줄이다. 17~18번 줄에서 `@Min`, `@Max` 어노테이션이 `groups` 속성을 사용해 `ValidationGroup1` 그룹을 설정하고 24번줄에서 `ValidationGroup2`를 성정했다. 이 설정을 통해 어느 그룹에 맞춰 유효성 검사를 실시할 것인지 지정한 것이다.

실제로 그룹을 어떻게 설정해서 유효성 검사를 실시할지 결정하는 것은 `@Validated` 어노테이션에서 한다. 유효성 검사 그룹을 설정하기 위해 컨트롤러 클래스에 아래와 같은 메서드를 추가한다.

<br>

```java
@RestController
@RequestMapping("/validation")
public class ValidationController {

    private final Logger LOGGER = LoggerFactory.getLogger(ValidationController.class);
    
    @PostMapping("/validated")
    public ResponseEntity<String> checkValidation(
            @Validated @RequestBody ValidatedRequestDto validatedRequestDto){
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }
    
    @PostMapping("/validated/group1")
    public ResponseEntity<String> checkValidation1(
            @Validated(ValidationGroup1.class) @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }

    @PostMapping("/validated/group2")
    public ResponseEntity<String> checkValidation2(
            @Validated(ValidationGroup2.class) @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }

    @PostMapping("/validated/all-group")
    public ResponseEntity<String> checkValidation3(
            @Validated({ValidationGroup1.class,
            ValidationGroup2.class}) @RequestBody ValidatedRequestDto validatedRequestDto) {
        LOGGER.info(validatedRequestDto.toString());
        return ResponseEntity.status(HttpStatus.OK).body(validatedRequestDto.toString());
    }
}
```

<br>

예제의 9번 줄에서는 `@Validated` 어노테이션에 속성을 지정하지 않고 16번 줄에서는 `ValidationGroup1`, 23번 줄에서는 `ValidationGroup2`를 그룹으로 지정했다. 마지막으로 30~31번 줄에서는 두 그룹을 모두 지정했다.

다시 Swagger 페이지에서 각 메서드를 호출하겠다. 먼저 7~12번 줄의 `checkValidation()` 메서드를 호출하겠다. 호출할 때 전달하는 데이터는 다음과 같다.

<br>

```
{
  "age": -1,
  "booleanCheck": true,
  "count": -1,
  "description": "Validation 실습 데이터입니다.",
  "email": "flature@wikibooks.co.kr",
  "name": "Flature",
  "phoneNumber": "010-1234-5678"
}
```

<br>

위 데이터는 `age`와 `count` 변수에 대한 유효성 검사를 통과하지 못하는 데이터이다. 하지만 첫 번째 메서드를 호출했을 경우 정상적으로 통과하는 것을 볼 수 있다. `@Validated` 어노테이션에 특정 그룹을 지정하지 않는 경우에는 `group` 속성을 설정하지 낳은 필드에 대해서만 유효성 검사를 실시하게 된다.

데이터는 그래도 이용하면서 14~19번 줄의 `checkValidation1()` 메서드를 호출하면 다음과 같은 로그를 확인할 수 있다.

<br>

```
[Field error in object 'validatedRequestDto' on field 'age': rejected value [-1]; codes [Min.validatedRequestDto.age,Min.age,Min.int,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.age,age]; arguments []; default message [age],20]; default message [20 이상이어야 합니다]] ]
```

<br>

검사 오류가 발생할 수 있는 두 변수 중에서 `ValidationGroup1`을 그룹으로 설정한 `age`에 대한 에러가 발생하는 것을 볼 수 있다. 마찬가지로 21~26번 줄의 `checkValidation2()` 메서드를 호출하면 `age`에서는 오류가 발생하지 않고 `count`에 대한 오류만 발생하게 된다.

그리고 28~35번 줄의 마지막 메서드를 호출하면 다음과 같이 검사 오류가 로그가 출력된다.

<br>

```
com.springboot.valid_exception.controller.ValidationController.checkValidation3(com.springboot.valid_exception.data.dto.ValidatedRequestDto) with 2 errors: [Field error in object 'validatedRequestDto' on field 'count': rejected value [-1]; codes [Positive.validatedRequestDto.count,Positive.count,Positive.int,Positive]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.count,count]; arguments []; default message [count]]; default message [0보다 커야 합니다]] [Field error in object 'validatedRequestDto' on field 'age': rejected value [-1]; codes [Min.validatedRequestDto.age,Min.age,Min.int,Min]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.age,age]; arguments []; default message [age],20]; default message [20 이상이어야 합니다]] ]
```

<br>

한 번 더 메서드를 호출하겠다. 이번에는 호출 데이터를 다음과 같이 변경한다.

```json
{
  "age": 30,
  "booleanCheck": false,
  "count": 30,
  "description": "Validation 실습 데이터입니다.",
  "email": "flature@wikibooks.co.kr",
  "name": "Flature",
  "phoneNumber": "010-1234-5678"
}
```

<br>

위 데이터는 `age`와 `count`는 검사를 통과하고 `booleanCheck`변수에서 검사를 실패하는 데이터이다. 한편 `checkValidation3()` 메서드를 호출하면 정상적으로 응답이 오는 것을 볼 수 있다. 정리하면 다음과 같다.

- `@Validated` 어노테이션에 특정 그룹을 설정하지 않은 경우에는 `groups`가 설정되지 않은 필드에 대해 유효성 검사를 수행
- `@Validated` 어노테이션에 특정 그룹을 설정하는 경우에는 지정된 그룹으로 설정된 필드에 대해서만 유효성 검사를 수행

이처럼 그룹을 지정해서 유효성 검사를 실시하는 경우에는 어떤 상황에 사용할지를 적절하게 설계해야 의도대로 유효성 검사를 실시할 수 있다. 만략 이를 제대로 설계하지 않으면 비효율적이거나 생산적이지 못한 패턴을 의미하는 안티 패턴이 발생하게 된다.



### 커스텀 Validation 추가

실무에서는 유효성 검사를 실시할 때 자바 또는 스프링의 유효성 검사 어노테이션에서 제공하지 않는 기능을 써야할 때도 있다. 이 경우 `ConstraintValidator`와 커스턴 어노테이션을 조합해서 별도의 유효성 검사 어노테이션을 생성할 수 있다. 동일한 정규식을 계속 쓰는 `@Pattern` 어노테이션의 경우가 가장 흔한 사례이다.

이번에는 전화번호 형식이 일치하는지 확인하는 간단한 유효성 검사 어노테이션을 생성해 보겠다. 먼저 `ConstraintValidator` 인터페이스를 구현하는 클래스를 생성해야 한다. 아래와 같이 `TelephoneValidator`클래스를 생성한다.

<br>

```java
public class TelephoneValidator implements ConstraintValidator<Telephone,String> {

    @Override
    public boolean isValid(String value, ConstraintValidatorContext constraintValidatorContext) {
        if(value == null){
            return false;
        }
        return value.matches("01(?:0|1|[6-9])[.-]?(\\d{3}|\\d{4})[.-]?(\\d{4})$");
    }
}
```

<br>

예제에서 1번 줄에서는 `TelephoneValidator` 클래스를 `ConstraintValidator` 인터페이스의 구현체로 정의한다. 인터페이스를 선언할 때는 어떤 어노테이션 인터페이스인지 타입을 지정해야 한다.(어노테이션 인터페이스에 대해서는 바로 이어서 소개하겠다.) `ConstraintValidator` 인터페이스는 `invalid()`메서드를 정의하고 있다. 이 메서드를 구현하려면 5~7번 줄처럼 직접 유효성 검사 로직을 작성해야 한다. 5~7번 줄과 같이 `null`에 대한 허용 여부 로직을 추가하고, 8번 줄에서는 지정한 정규식과 비료해서 알맞은 형식을 띠고 있는지 검사하게 된다. 이 로직에서 `false`가 리턴되면 `MethodArgumentNotValidException`예외가 발생한다.

그럼 `ConstraintValidator` 인터페이스에서 정의한 `Telephone` 인터페이스를 살펴보겠다. `Telephone` 인터페이스는 아래와 같이 작성할 수 있다.

<br>

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = TelephoneValidator.class)
public @interface Telephone {
    String message() default " 전화번호 형식이 일치하지 않습니다.";
    Class[] groups() default {};
    Class[] payload() default {};
}
```

<br>

먼저 1번 줄에 선언돼 있는 `@Target` 어노테이션은 이 어노테이션을 어디서 선언할 수 있는지 정의하는데 사용된다. 예제에서는 필드에 선언할 수 있게 설정돼 있다. 그 외에 사용할 수 있는 `ElementType`은 다음과 같다.

- `ElementType.PACKAGE`
- `ElementType.TYPE`
- `ElementType.CONSTRUCTOR`
- `ElementType.FIELD`
- `ElementType.METHOD`
- `ElementType.ANNOTATION_TYPE`
- `ElementType.LOCAL_VARIABLE`
- `ElementType.PARAMETER`
- `ElementType.TYPE_PARAMETER`
- `ElementType.TYPE_USE`

<br>

2번 줄에서 사용된 `@Retention` 어노테이션은 이 어노테이션이 실제로 적용되고 유지되는 범위를 의미한다. `@Retention`의 적용 범위는 `RetentionPolicy`를 통해 지정하며, 지정 가능한 항목은 다음과 같다.

- `RetentionPolicy.RUNTIME` : 컴파일 이후에도 JVM에 의해 계쏙 참조한다. 리플렉션이나 로깅에 많이 사용되는 정책이다.
- `RetentionPolicy.CLASS` : 컴파일러가 클래스를 참조할 때까지 유지한다.
- `RetentionPolicy.SOURCE` : 컴파일 전까지만 유지된다. 컴파일 이후에는 사라진다.

<br>

그리고 3번 줄에서는 `@Constraint`어노테이션을 활요해 앞에서 소개한 `TelephoneValidator`와 매핑하는 작업을 수행한다. 5~7번 줄과 같이 `@Telephone` 인터페이스 내부에는 `message()`, `groups()`, `payload()` 요소를 정의해야 한다.(이와 관련된 내용은 `Constrainthelper`에서 살펴보겠다.) 각 항목은 다음과 같은 의미를 가지고 있다.

- `message()` : 유효성 검사가 실패할 경우 반환되는 메시지를 의미 한다.
- `groups()` : 유효성 검사를 사용하는 그룹으로 설정한다.
- `payload()` : 사용자가 추가 정보를 위해 전달하는 값이다.

<br>

위에서는 별도의 `groups()`와 `payload()` 요소는 정의하지 않고 기본 메시지에 대해서만 요소를 설정했다.

이렇게 어노테이션 인터페이스를 설정하고 나면 아래와 같이 인텔리제이 IDEA의 [Bean Validation] 탭에 앞에서 생성한 `@Telephone`이 추가된 것을 볼 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183445584-9e43dcd3-ae18-4cc6-ada2-b0b50db2d4b5.png){: .align-center}

<br>

이제 직접 생성한 새로운 유효성 검사 어노테이션을 적용해 보겠다. 아래와 같이 `ValidatedRequestDto` 클래스에서 `phoneNumber` 변수의 어노테이션을 변경한다.

<br>

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
@Builder
public class ValidatedRequestDto {

    @NotBlank
    private String name;

    @Email
    private String email;

    @Telephone
    private String phoneNumber;

    @Min(value = 20, groups = ValidationGroup1.class)
    @Max(value = 40, groups = ValidationGroup1.class)
    int age;

    @Size(min = 0, max = 40)
    private String description;

    @Positive(groups = ValidationGroup2.class)
    private int count;

    @AssertTrue
    private boolean booleanCheck;

}
```

<br>

`@Pattern` 어노테이션을 `@Telephone` 어노테이션으로 변경한 코드이다. 다시 애플리케이션을 실행하고 Swagger 페이지에서 테스트를 진행하겠다. 테스트 데이터는 다음과 같다.

<br>

```json
{
  "age": 30,
  "booleanCheck": true,
  "count": 30,
  "description": "Validation 실습 데이터입니다.",
  "email": "flature@wikibooks.co.kr",
  "name": "Flature",
  "phoneNumber": "12345678"
}
```

<br>

이 내용으로 메서드를 호출하면 다음과 같이 유효성 검사에서 형식 오류를 감지하는 것을 볼 수 있다. 아랭[서는 별도의 그룹을 지정하지 않았기 때문에 `checkValidation()` 메서드를 호출했을 때 오류가 발생한다.

<br>

```
[org.springframework.web.bind.MethodArgumentNotValidException: Validation failed for argument [0] in public org.springframework.http.ResponseEntity<java.lang.String> com.springboot.valid_exception.controller.ValidationController.checkValidation(com.springboot.valid_exception.data.dto.ValidatedRequestDto): [Field error in object 'validatedRequestDto' on field 'phoneNumber': rejected value [12345678]; codes [Telephone.validatedRequestDto.phoneNumber,Telephone.phoneNumber,Telephone.java.lang.String,Telephone]; arguments [org.springframework.context.support.DefaultMessageSourceResolvable: codes [validatedRequestDto.phoneNumber,phoneNumber]; arguments []; default message [phoneNumber]]; default message [ 전화번호 형식이 일치하지 않습니다.]] ]
```

