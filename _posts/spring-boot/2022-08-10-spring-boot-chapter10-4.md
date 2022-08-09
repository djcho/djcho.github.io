---
title : "[SpringBoot][Chapter10.4] 유효성 검사와 예외처리 - 예외 처리"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Exception]
toc: true
toc_sticky : true
published : true
date : 2022-08-10
last_modified_at : 2022-08-10
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





## 예외 처리

애플리케이션을 개발할 때는 불가피하게 많은 오류가 발생하게 된다. 자바에서는 이러한 오류를 `try/catch`, `throw` 구문을 활용해 처리한다. 스프링 부트에서는 더욱 편리하게 예외 처리를 할 수 있는 기능을 제공한다. 이번 절에서는 예외 처리의 기초를 소개하고 스프링 부트에서 적용할 수 있는 예외 처리 방식을 알아보겠다.



### 예외와 에러

프로그래밍에서 예외(exception)란 입력 값의 처리가 불가능하거나 참조된 값이 잘못된 경우 등 애플리케이션이 정상적으로 동작하지 못하는 상황을 의미한다. 예외는 개발자가 직접 처리할 수 있는 것이므로 미리 코드 설계를 통해 처리할 수 있다.

다음으로 에러(error)가 있다. 많은 사람들이 예외와 비슷한 의미로 사용하고 있지만 소프트웨어 공학에서는 엄현히 다르게 사용되는 영어이다. 에러는 주로 자바의 가상머신에서 발생시키는 것으로서 예외와 달리 애플리케이션 코드에서 처리할 수 있는 것이 거의 없다. 대표적인 예로 메모리 부족(OutOfMemory), 스택 오버플로(StackOverFlow)등이 있다. 이러한 에러는 발생 시점에 처리하는 것이 아니라 미리 애플리케이션의 코드를 살펴보면서 문제가 발생하지 않도록 예방해서 원천적으로 차단해야 한다.



### 예외 클래스

자바의 예외 클래스는 아래와 같은 상속 구조를 갖추고 있다.

![image](https://user-images.githubusercontent.com/13410737/183451719-b14c55a2-83ed-46bf-8e2f-c6db4c06fc8c.png){: .align-center}

<br>

모든 예외 클래스는 `Throwable` 클래스를 상속받는다. 그리고 가장 익숙하게 볼 수 있는 `Exception` 클래스는 다양한 자식 클래스를 가지고 있다. 이 클래스는 크게 Checked Exception과 Unchecked Exception으로 구분할 수 있다.

|                      | Checked Exception                 | Unchecked Exception                                          |
| -------------------- | --------------------------------- | ------------------------------------------------------------ |
| 처리 여부            | 반드시 예외 처리 필요             | 명시적 처리를 강제하지 않음                                  |
| 확인 시점            | 컴파일 단계                       | 실행 중 단계                                                 |
| 대표적인 예외 클래스 | `IOException`<br />`SQLException` | `RuntimeException`<br />`NullPointerException`<br />`IllegalArgumentException`<br />`IndexOutOfBoundException`<br />`SystemException` |

<br>

Checked Exception은 컴파일 단계에서 확인 가능한 예외 상황이다. 이러한 예외는 IDE에서 캐치해서 반드시 예외 처리를 할 수 있도록 표시헤준다. 반면 Unchecked Exception은 런타임 단계에서 확인되는 예외 상황을 나타낸다. 즉, 문법상 문제는 없지만 프로그램이 동작하는 도중 예기치 않은 상황이 생겨 발생하는 예외를 의미한다.

간단히 분류하자면 `RuntimeException`을 상속받는 `Exception` 클래스는 Unchecked Exception이고 그렇게 않은 `Exception	` 클래스는 Checked Exception이다.



### 예외 처리 방법

예외가 발생했을 때 이를 처리하는 방법은 크게 세 가지가 있다.

- 예외 복구
- 예외 처리 회피
- 예외 전화

<br>

먼저 예외 복구 방법은 예외 상황을 파악해서 문제를 해결하는 방식이다. 대표적인 방법이 `try/catch` 구문이다. `try` 블록에는 예외가 발생할 수 있는 코드를 작성한다. 대체로 외부 라이브러리를 사용하는 경우에는 `try` 블록을 사용하라는 IDE의 알람이 발생하지만 개발자가 직접 작성한 로직은 예외 상황을 예측해서 `try` 블록에 포함시켜야 한다. 그리고 `catch` 블록을 통해 `try` 블록에서 발생하는 예외 상황을 처리하는 내용을 작성한다. 이때 `catch` 블록은 여러 개를 작성할 수 있다. 이 경우 예외 상황이 발생하면 애플리케이션에서는 여러 개의 `catch` 블록을 순차적으로 거치면서 예외 유형과 매칭되는 블록을 찾아 예외 처리 동작을 수행한다.

<br>

```java
int a = 1;
String b = "a";

try {
    System.out.println(a + Integer.parseInt(b));
} catch (NumberFormatException e) {
    b = "2";
    System.out.println(a + Integer.parseInt(b));
}
```

<br>

또 다른 예외 처리 방법 중 하나는 예외 처리를 회피하는 방법이다. 이방법은 예외가 발생한 시점에서 바로 처리하는 것이 아니라 예외가 발생한 메서드를 호출한 곳에서 에러 처리를 할 수 있게 전가하는 방식이다.  이때 `throw` 키워드를 사용해 어떤 예외가 발생했는지 호출부에 내용을 전달할 수 있다.

<br>

```java
int a = 1;
String b = "a";

try {
    System.out.println(a + Integer.parseInt(b));
} catch (NumberFormatException e) {
    throw new NumberFormatException("숫자가 아닙니다.");
}
```

<br>

마지막으로 예외 전환 방법이 있다. 이 방법은 앞의 두 방식을 적절하게 섞은 방식이다. 예외가 발생했을 때 어떤 예외가 발생했느냐에 따라 호출부로 예외 내용을 전달하면서 좀 더 적합한 예외 타입을 전달할 필요가 있다. 또는 애플리케이션에서 예외 처리를 좀 더 단순하게 하기 위해 래핑(wrapping)헤야 하는 경우도 있다. 이런 경우에는 `try/catch` 방식을 사용하면서 `catch`블록에서 `throw` 키워드를 사용해 다른 예외 타입으로 전달하면 된다. 이 방식은 앞으로 나올 커스텀 예외를 만드는 과정에서 사용되는 방법이므로 별도로 예제를 보여주지 않겠다.



### 스프링 부트의 예외 처리 방식

웹 서비스 애플리케이션에서는 외부에서 들어오는 요청에 담신 데이터를 처리하는 경우가 많다. 그 과정에서 예외가 발생하면 예외를 복구해서 정상으로 처리하기보다는 요청을 보낸 클라이언트에 어떤 문제가 발생했는지 상황을 전달하는 경우가 많다. 이번 절에서는 이를 반영해서 예외 상황을 복구하는 방법보다는 스프링 부트에서 사용하는 예외 처리 방법을 중심으로 설명하고 실습하겠다.

예외가 발생했을 떄 클라이언트에 오류 메시지를 전달하려면 각 레이어에서 발생한 예외를 엔드포인트 레벨인 컨트롤러로 전달해야 한다. 이렇게 전달받은 예외를 스프링 부트에서 처리하는 방식으로 크게 두 가지가 있다.

- `@(Rest)ControllerAdvice`와 `@ExceptionHandler`를 통해 모든 컨트롤러의 예외를 처리
- `@ExceptionHandler`를 통해 특정 컨트롤러의 예외를 처리



> **💡 Tip.**
>
> `@ControllerAdvice` 대신 `@RestControllerAdvice`를 사용하면 결괏값을 JSON 형태로 변환할 수 있다.

<br>

먼저 `@RestControllerAdvice`를 활용한 핸들러 클래스를 생성하겠다. 아래와 같이 `CustomExceptionHandler`클래스를 생성한다.

```java
@RestControllerAdvice
public class CustomExceptionHandler {
    
    private final Logger LOGGER = LoggerFactory.getLogger(CustomExceptionHandler.class);
    
    @ExceptionHandler(value = RuntimeException.class)
    public ResponseEntity<Map<String, String>> handlerException(RuntimeException e, HttpServletRequest request){
        HttpHeaders responseHeaders = new HttpHeaders();
        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;
        
        LOGGER.error("Advice 내 HandleException 호출, {}, {}", request.getRequestURI(), e.getMessage());
        
        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", e.getMessage());
        
        return new ResponseEntity<>(map, responseHeaders, httpStatus);
    }
}
```

<br>

예제에서 사용한 `@RestcontrollerAdvice`와 이 예제에서는 사용하지 않는 `@ControllerAdvice`는 스프링에서 제공하는 어노테이션이다. 이 어노테이션은 `@Controller`나 `@RestController` 에 발생하는 예외를 한 곳에서 관리하고 처리할 수 있게 하는 기능을 수행한다. 즉, 다음과 같이 별도 설정을 통해 예외를 관제하는 범위를 지정할 수 있다.

<br>

```java
@RestControllerAdvice(basePackages = "com.springboot.valid_exception")
```

<br>

6번 줄에 지정된 `@ExceptionHandler`는 `@Controller`나 `@RestController`가 적용된 빈에서 발생하는 예외를 잡아 처리하는 메서드를 정의할 때 사용한다. 어떤 예외 클래스를 처리할지는 `value` 속성으로 등록한다. `value` 속성은 배열의 형식으로도 전달받을 수 있어 여러 예외 클래스를 등록할 수도 있다. 위 예에제어슨 `RuntimeException`이 발생하면 처리하도록 코드를 작성했으므로 `RuntimeException`에 포함되는 각종 예외가 발생할 경우를 포착해서 처리하게 된다.

8~18번 줄에서는 클라이언트에게 오류가 발생했다는 것을 알리는 응답 메시지를 구성해서 리턴한다. 컨트롤러의 메서드에 다른 타입의 리턴이 설정돼 있어도 핸들러 메서드에서 별도의 리턴 타입을 지정할 수 있다.

이 예제를 테스트하기 위해 발생시킬 수 있는 컨트롤러를 생성하겠다. 아래와 같이 `ExceptionController`를 생성한다.

<br>

```java
@RestController
@RequestMapping("/exception")
public class ExceptionController {

    @GetMapping
    public void getRuntimeException(){
        throw new RuntimeException("getRuntimeException 메서드 호출");
    }

}
```

<br>

위 예제의 `getRuntimeException()` 메서드를 컨트롤러로 요청이 들어오면 `RuntimeException`을 발생시킨다. 아래와 같이 Swagger 페이지에서 이 메서드를 호출해 보자.

<br>

![image](https://user-images.githubusercontent.com/13410737/183654354-0598b090-33ec-41f1-854e-4fc14ff31eec.png){: .align-center}

<br>

위와 같이 호출하면 400 에러와 함께 아래와 같이 에러 메시지가 Body 값에 담겨 응답이 돌아온다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183654866-9782c827-3b5f-4b69-9b10-8a60cae95346.png){: .align-center}

<br>

핸들러 메서드는 `Map` 객체에 응답할 메시지를 구성하고 `ResponseEntity`에 `HttpHeader`, `HttpStatus`, `Body` 값을 담아 전달한다. 이 핸들러 메서드는 위 그림과 같은 응답을 출력한다.

이처럼 컨트롤러에서 던진 예외는 `@ControllerAdvice`또는 `@RestControllerAdvice`가 선언돼 있는 핸들러 클래스에서 매핑된 예외 타입을 찾아 처리하게 된다. 두 어노테이션은 별도 범위 설정이 없으면 전역 범위에서 예외를 처리하기 때문에 특정 컨트롤러에서만 동작하는 `@ExceptionHandler` 메서드를 생성해서 처리할 수도 있다. `ExceptionController`에 아래와 같이 메서드를 추가로 생성해 보자.

<br>

```java
@RestController
@RequestMapping("/exception")
public class ExceptionController {

    private final Logger LOGGER = LoggerFactory.getLogger(ExceptionController.class);
    
    @GetMapping
    public void getRuntimeException(){
        throw new RuntimeException("getRuntimeException 메서드 호출");
    }

    @ExceptionHandler(value = RuntimeException.class)
    public ResponseEntity<Map<String, String>> handleException(RuntimeException e, HttpServletRequest request) {
        HttpHeaders responseHeaders = new HttpHeaders();
        responseHeaders.setContentType(MediaType.APPLICATION_JSON);
        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;
        
        LOGGER.error("클래스 내 handleException 호출, {}, {}", request.getRequestURI(),
                e.getMessage());
        
        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", e.getMessage());
        
        return new ResponseEntity<>(map, responseHeaders, httpStatus);
    }
}
```

<br>

예제처럼 컨트롤러 클래스 내에 `@ExceptionHandler` 어노테이션을 사용한 메서드를 선언하면 해당 클래스에 국한해서 예외를 처리할 수 있다. 앞서 작성한 핸들러 메서드와 위의 핸들러 메서드에서 각 로그 메시지에 차이를 두고 다시 Swagger 페이지에서 테스트를 수행하면 다음과 같은 내용이 콘솔에 출력되는 것을 볼 수 있다.

<br>

```
com.springboot.valid_exception.controller.ExceptionController 클래스 내 handleException 호출, /exception, getRuntimeException 메서드 호출
```

<br>

출력 결과에서 '클래스 내 handleException 호출' 이라는 메시지를 볼 수 있다. 만약 `@ControllerAdvice`와 컨트롤러 내에 동일한 예외 타입을 처리한다면 좀 더 우선수위가 높은 클래스 내의 핸들러 메서드가 사용되는 것을 볼 수 있다.

우선순위를 비교하는 방법은 총 두 가지가 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183669361-ec09817a-8be3-4c98-b5db-f4b2e3c04fc9.png){: .align-center}

<br>

위 그림 처럼 `@ControllerAdvice`의 글로벌 예외 처리와 `@Controller` 내의 컨트롤러 예외 처리에 동일한 타입의 예외 처리를 하게 되면 범위가 좁은 컨트롤러의 핸들러 메서드가 우선순위를 갖게 된다.

<br>

>예외를 잘 처리하기 위해서는 어느 상와에 어떤 예외 타입을 사용해야 적절한지 알아야 한다. 다음 URL을 통해 CheckedException과 Unchecked Exception을 구분해서 확인하면 적절한 예외 처리에 도움이 될 것이다.
>
>- <a href="https://docs.oracle.com/en/java/javase/11/docs/api/index.html" target="_blank">https://docs.oracle.com/en/java/javase/11/docs/api/index.html</a>



### 커스텀 예외

애플리케이션을 개발하다 보면 점점 예외로 처리할 영역이 늘어나고, 예외 상황이 다양해지면서 사용하는 예외 타입도 많아진다. 대부분의 상황에서는 자바에서 이미 적절한 상황에 사용할 수 있도록 제공하는 표준 예외(Standard Exception)를 사용하면 해경된다. 사실 애플리케이션의 예외 처리에는 표준 예외만 사용해도 모든 상황들을 처리할 수 있다. 그런데 왜 커스텀 예외(Custom Exception)를 만들어 사용해야할까?

커스텀 예외를 만들어서 사용하면 네이밍에 개발자의 의도를 담을 수 있기 때문에 이름만으로도 어느 정도 예외 상황을 짐작할 수 있다. 앞에서 언급했듯이 표준 예외에서도 다양한 예외 상황을 처리할 수 있는 클래스를 제공하고 있지만 표준 예외에서 제공하는 클래스는 해당 예외 타입의 이름만으로 이해하기 어려운 경우가 있다. 그래서 표준 예외를 사용할 때는 예외 메시지를 상세히 작성해야 하는 번거로움이 있다.

또한 커스텀 예외를 사용하면 애플리케이션에서 발생하는 예외를 개발자가 직접 관리하기가 수월해진다. 표준 예외를 상속받은 커스텀 예외들을 개발자가 직접 코드로 관리하기 때문에 책임 소재를 애플리케이션 내부로 가져올 수 있게 된다. 이를 통해 동일한 예외 상황이 발생할 경우 한 곳에서 처리하며 특정 상황에 맞는 예외 코드를 적용할 수 있게 된다.

마지막으로 커스텀 예외를 사용하면 예외 상황에 대한 처리도 용이하다. 앞에서 `@ControllerAdvice`와 `@ExceptionHandler`에 대해 알아봤는데, 이러한 어노테이션을 사용해 애플리케이션에서 발생하는 예외 상황들을 한 곳에서 처리할 수 있었다. 예를 들어, `RuntimeException`에 대해 `@ControllerAdvice`의 내부에서 표준 예외 처리를 하는 로직을 작성한 경우 개발자가 의도한 `RuntimeException`에 대해 `@ControllerAdvice`의 내부에서 표준 예외 처리를 하는 로직을 작성한 경우 개발자가 의도한 `RuntimeException`부분이 아닌 의도하지 않은 부분에서 발생하는 에러들이 존재할 수 있다. 표준 예외를 사용하면 이처럼 의도하지 않은 예외 상황도 정해진 예외 처리 코드에서 처리하기 때문에 어디에서 문제가 발생했는지 확인하기가 어렵다. 그러나 커스텀 예외로 관리하면 의도하지 않았던 부분에서 발생한 예외는 개발자가 관리하는 예외 처리 코드가 처리하지 않으므로 개발 과정에서 혼동할 여지가 줄어든다.

<br>

>커스텀 예외의 효과에 대해서는 개발자들의 의견이 분분하다. 우선 커스텀 예외를 만들어 사용해보는 것을 시작으로 어떤 방식이 효과적인지 직접 고민하고 자신만의 논리를 구축하길 바란다.



### 커스텀 예외 클래스 생성하기

이제 커스텀 예외를 생성하고 활용하는 방법을 살펴보겠다. 커스텀 예외는 만드는 목적에 따라 생성하는 방법이 다르다. 이 책에서는 스프링 환경에서 사용할 수 있는 `@ControllerAcvice`와 `@ExceptionHandler`의 무분별한 예외 처리를 방지하기 위한 커스텀 예외를 생성하는 과정을 실흡해 보겠다. 우선 앞에서 한번 살펴본 그림을 다시 보자.

<br>

![image](https://user-images.githubusercontent.com/13410737/183451719-b14c55a2-83ed-46bf-8e2f-c6db4c06fc8c.png){: .align-center}

<br>

커스텀 예외는 예외가 발생하는 상황에 해당하는 상위 예외 클래스를 상속받는다. 그래서 커스텀 예외는 상위 예외 클래스보다 좀 더 구체적인 이름을 사용하기도 한다. 그러나 여기서는 커스텀 예외의 네이밍보다는 클래스의 구조적인 설계를 통한 예외 클래스 생성 방법을 알아보겠다.

먼저 `Exception`클래스의 커스텀 예외를 만들어보겠다. 예외 클래스의 상속 구조는 보면 `Exception`클래스는 `Throwable` 클래스를 상속받는다. 아래 실습에서는 그중 필수적으로 사용되는 `message` 변수를 이용해 `Exception` 클래스의 커스텀 예외를 만들겠다. 먼저 `Exception` 클래스는 아래와 같다.

<br>

```java
public class Exception extends Throwable {
    static final long serialVersionUID = -3387516993124229948L;

    public Exception() {
        super();
    }

    public Exception(String message) {
        super(message);
    }

    public Exception(String message, Throwable cause) {
        super(message, cause);
    }

    public Exception(Throwable cause) {
        super(cause);
    }
    
    protected Exception(String message, Throwable cause,
                        boolean enableSuppression,
                        boolean writableStackTrace) {
        super(message, cause, enableSuppression, writableStackTrace);
    }
}
```

<br>

위의 8~10번 줄에 있는 생성자는 `String` 타입의 메시지 문자열을 받고 있다. 이 생성자는 `Throwable` 클래스의 생성자를 호출한다.

<br>

```java
public class Throwable implements Serializable {

    private static final long serialVersionUID = -3042686055658047285L;

    private transient Object backtrace;

    private String detailMessage;
    
    ...	생략 ...
    
    public Throwable() {
        fillInStackTrace();
    }
    
    public Throwable(String message) {
        fillInStackTrace();
        detailMessage = message;
    }
    
    public String getMessage() {
        return detailMessage;
    }
    
    public String getLocalizedMessage() {
        return getMessage();
    }
    
    ...	생략 ...
        
}
```

<br>

위에서 살펴볼 수 있듯이 `Exception` 클래스는 부모 클래스인 `Throwable` 클래스의 15~18번 줄의 생성자를 호출하게 되며, `message` 변수의 값을 `detailMessage` 변수로 전달받는다. 커스텀 예외를 생성하는 경우에도 이 `message` 변수를 사용하게 된다.

그리고 `HttpStatus`를 커스텀 예외 클래스에 포함시키면 핸들러 안에서 선언해서 사용하는 것이 아닌 예외 클래스만 전달받으면 그 안에 내용이 포함돼 있는 구조로 설계할 수 있다. 참고로 `HttpStatus`는 열거형(`Enum`)이다. 열거형은 서로 관련 있는 상수를 모든 심볼릭한 명칭의 집합이다. 쉽게 생각해서 클래스 타입의 상수로 볼 수 있다. 아래에서 `HttpStatus`의 주요 코드 일부를 살펴보겠다.

<br>

```java
public enum HttpStatus {
    
    // --- 4xx Client Error ---
    BAD_REQUEST(400, HttpStatus.Series.CLIENT_ERROR, "Bad Request"),
    UNAUTHORIZED(401, HttpStatus.Series.CLIENT_ERROR, "Unauthorized"),
    PAYMENT_REQUIRED(402, HttpStatus.Series.CLIENT_ERROR, "Payment Required"),
    FORBIDDEN(403, HttpStatus.Series.CLIENT_ERROR, "Forbidden"),
    NOT_FOUND(404, HttpStatus.Series.CLIENT_ERROR, "Not Found"),
    METHOD_NOT_ALLOWED(405, HttpStatus.Series.CLIENT_ERROR, "Method Not Allowed"),
    
    private HttpStatus(int value, HttpStatus.Series series, String reasonPhrase) {
        this.value = value;
        this.series = series;
        this.reasonPhrase = reasonPhrase;
    }

    public int value() {
        return this.value;
    }

    public HttpStatus.Series series() {
        return this.series;
    }
}
```

<br>

`HttpStatus`는 3~9번 줄과 같이 `value`, `series`, `reasonPhrase` 변수로 구성된 객체를 제공한다. 흔히 볼 수 있는 `Http`응답 코드와 메시지이다. 위 예제에서는 4xx 코드만 나왔지만 1xx, 2xx, 3xx, 4xx, 5xx에 대해서도 코드 모음이 구성돼 있다. 각 값들은 17~27번 줄에 작성돼 있는 메서드를 통해 값들을 가져와 사용한다.

최정적으로 이번에 만들어볼 커스텀 예외 클래스를 생성하는 데 필요한 내용은 다음과 같이 정리할 수 있다.

- 에러 타입(error type) : `HttpStatus`의 `reasonPharse`
- 에러 코드(error code) : `HttpStatus`의 `value`
- 메시지(message) : 상황별 상세 메시지

<br>

위와 같은 구성으로 커스텀 예외 클래스를 생성하겠다. 추가로 애플리케이션에서 가지고 있는 도메인 레벨을 메시지에 표현하기 위해 `ExceptionClass` 열거형 타입을 생성하겠다. 이를 도식화하면 아래 그림과 같은 커스텀 예외 클래스 구조가된다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183679146-2d4f3088-1c43-44f0-ba6d-a94e52154103.png){: .align-center}

<br>

커스텀 예외 클래스를 생성하기에 앞서 도메인 레벨 표현을 위한 열거형을 아래와 같이 생성하겠다.

<br>

```java
public class Constants {
    
    public enum ExceptionClass {
        PRODUCT("Product");
        
        private String exceptionClass;
        
        ExceptionClass(String exceptionClass) {
            this.exceptionClass = exceptionClass;
        }
        
        public String getExceptionClass() {
            return exceptionClass;
        }
        
        @Override
        public String toString() {
            return getExceptionClass() + " Exception. ";
        }
    }
}
```

<br>

예제에서는 `Constants`라는 클래스를 생성한 후 `ExceptionClass`를 내부에 생성했다. 열거형을 별도로 생성해도 무관하지만 상수 개념으로 사용하기 때문에 앞으로의 확장성을 위해 `Constants`라는 상수들을 통합 관리하는 클래스를 생성하고 내부에 `ExceptionClass`를 선언했다.

`ExceptionClass`라는 열거형은 커스텀 예외 클래스에서 메시지 내부에 어떤 도메인에서 문제가 발생했는지 보여주는 데 사용된다. 지금까지 만든 애플리케이션은 상품이라는 도메인에 대해서만 실습 코드를 작성해왔기 때문에 5번 줄과 같이 `PRODUCT`라는 상수만 선언했다.

열거형을 생성했으면 아래와 같이 커스텀 예외 클래스를 생성한다.

<br>

```java
public class CustomException extends Exception {
    
    private Constants.ExceptionClass exceptionClass;
    private HttpStatus httpStatus;
    
    public CustomException(Constants.ExceptionClass exceptionClass, HttpStatus httpStatus, String message) {
        super(exceptionClass.toString() + message);
        this.exceptionClass = exceptionClass;
        this.httpStatus = httpStatus;
    }
    
    public Constants.ExceptionClass getExceptionClass() {
        return exceptionClass;
    }
    
    public int getHttpStatusCode() {
        return httpStatus.value();
    }
    
    public String getHttpStatusType() {
        return httpStatus.getReasonPhrase();
    }
    
    public HttpStatus getHttpStatus() {
        return httpStatus;
    }
}
```

<br>

위의 커스텀 예외 클래스는 앞에서 만든 `ExceptionClass`와 `HttpStatus`를 필드로 가진다. 두 객체를 기반으로 예외 내용을 정의하며, 6~10번 줄과 같이 클래스를 초기화한다.

그럼 커스텀 예외를 활용해 보겠다. 먼저 `ExceptionHandler` 클래스에 `CustomException`에 대한 예외 처리 코드를 아래와 같이 추가한다.

<br>

```java
@ExceptionHandler(value = CustomException.class)
public ResponseEntity<Map<String, String>> handleException(CustomException e, 
                                                           HttpServletRequest request) {
    HttpHeaders responseHeaders = new HttpHeaders();
    LOGGER.error("Advice 내 handleException 호출, {}, {}", request.getRequestURI(), e.getMessage());

    Map<String, String> map = new HashMap<>();
    map.put("error type", e.getHttpStatusType());
    map.put("code", Integer.toString(e.getHttpStatusCode()));
    map.put("message", e.getMessage());

    return new ResponseEntity<>(map, responseHeaders, e.getHttpStatus());
}
```

<br>

위와 같이 처리하면 기존에 작성했던 핸들러 메서드와 달리 예외 발생 시점에 `HttpStatus`를 정의해서 전달하기 때문에 클라이언트 요청에 따라 유동적인 응답 코드를 설정할 수 있다는 장점이 있다.

지금까지 커스텀 예외 클래스를 생성하고 예외 처리를 수행하는 방법을 살펴봤다. 앞에서 만든 커스텀 예외에 대해 Swagger로 테스트하기 위해 아래와 같이 컨트롤러 메서드를 생성한다.

<br>

```java
@GetMapping("/custom")
public void getCustomException() throws CustomException{
    throw new CustomException(Constants.ExceptionClass.PRODUCT, HttpStatus.BAD_REQUEST, "getCustomException 메서드 호출");
}
```

<br>

이처럼 `CustomException`을 `throw` 키워드로 던지면 커스텀 예외가 발생한다. 3번 줄에서 괄호 내에 생성자를 정의한 것처럼 `ExceptionClass`에서 도메인을 비롯해 `HttpStatus`를 통해 어떤 응답 코드를 사용할지와 세부 메시지를 전달한다. 예제에서는 세부 메지시를 간단한 문자열로 표현했지만 예외가 발생하는 상황에서 특정 값을 전달하는 구성이라면 상세한 메시지를 작성해서 전달하거나 커스텀 예외 클래스를 적절한 타입으로 변경하는 것도 좋다.

이제 애플리케이션을 재실행하고 Swagger를 통해 위 메서드를 호출하겠다. Swagger를 통해 해당 메서드를 호출하면 아래와 같이 응답 내용이 출력되는 것을 볼 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183687335-7bc071f8-4ba5-49af-8116-6e6abe9c79c1.png){: .align-center}

<br>

Response Body를 통해 예외 발생 지점에서 설정한 값이 정상적으로 담겨 클라이언트로 응답한 것을 볼 수 있다.

