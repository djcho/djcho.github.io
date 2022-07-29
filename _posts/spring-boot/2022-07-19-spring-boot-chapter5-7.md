---
title : "[SpringBoot][Chapter5.7] API 작성법 - 로깅 라이브러리 - Logback"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Logback]
toc: true
toc_sticky : true
published : true
date : 2022-07-19
last_modified_at : 2022-07-19
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## 로깅 라이브러리 - Logback

로깅(logging)이란 애플리케이션이 동작하는 동안 시스템의 상태나 동작 정보를 시간순으로 기록하는 것을 의미한다.

로깅은 개발 영역 중 '비기능 요구사항'에 속한다. 즉, 사용자나 고객에게 필요한 기능은 아니라는 의미다. 하지만 로깅은 디버깅하거나 개발 이후 발생한 문제를 해결할 때 원인을 분석하는 데 꼭 필요한 요소이다.

자바 진영에서 가장 많이 사용되는 로깅 프레임워크는 <a href="https://logback.qos.ch/" target="_blank">Logback</a>이다. Logback이란 log4j 이후에 출시된 로깅 프레임워크로서 <a href="https://www.slf4j.org/" target="_blank">slf4j</a>를 기반으로 구현됐으며, 과거에 사용되던 log4j에 비해 월등한 성능을 자랑한다. 또한 스프링 부트의 spring-boot-starter-web 라이브러리 내부에 내장돼 있어 별도의 의존성을 추가하지 않아도 사용할 수 있다.

Logback의 특징은 다음과 같다.

- 크게 5개의 로그레벨(TRACE, DEBUG, INFO ,WARN, ERROR)을 설정할 수 있다.
  - ERROR : 로직 수행 중에 시스템에 심각한 문제가 발생해서 애플리케이션의 작동이 불가능한 경우를 의미한다.
  - WARN : 시스템 에러의 원인이 될 수 있는 경고 레벨을 의미한다.
  - INFO : 애플리케이션의 상태 변경과 같은 정보 전달을 위해 사용된다.
  - DEBUG : 애플리케이션의 디버깅을 위한 메시지를 표시하는 레벨을 의미한다.
  - TRACE : DEBUG 레벨보다 더 상세한 메시지를 표현하기 위한 레벨을 의미한다.
- 실제 운영 환경과 개발 환경에서 각각 다른 출력 레벨을 설정해서 로그를 확인할 수 있다.
- Logback의 설정 파일을 일정 시간마다 스캔해서 애플리케이션을 재기동하지 않아도 설정을 변경할 수 있다.
- 별도의 프로그램 지원 없이도 자체적으로 로그 파일을 압축할 수 있다.
- 저장된 로그 파일에 대한 보관 기간 등을 설정해서 관리할 수 있다.

<br>

> **💡 Tip**
>
> 각 로그 레벨에 대해서는 아파치 Log4j의 레벨과 관련된 내용을 보면 도움이 된다.
>
> - <a href="https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/Level.html" target="_blank">https://logging.apache.org/log4j/1.2/apidocs/org/apache/log4j/Level.html</a>

<br>

### Logback 설정

이제 Logback을 사용하기 위한 설정 파일을 만들어보자. 일반적으로 클래스패스(classpath)에 있는 설정 파일을 자동으로 참조하므로 Logback 설정 파일은 리소스 폴더 안에 생성한다. 파일명의 경우 일반적인 자바 또는 스프링 프로젝트에서는 `logback.xml`이라는 이름으로 참조하지만 스프링 부트에서는 `logback-spring.xml` 파일을 참조한다. 따라서 아래와 같이 `logback-spring.xml` 파일을 추가한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179761396-7e806bd5-c715-4efa-9b0b-69867960282a.png){: .align-center}

<br>

이 설정 파일은 XML 형식을 띠고 있다. 파일을 생성한 후 내부에 아래와 같이 몇 가지 정해진 규칙을 따라 내용을 추가하면 된다.

<br>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <property name="LOG_PATH" value="./logs"/>

    <!-- Appenders -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] [%thread] %logger %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="INFO_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <file>${LOG_PATH}/info.log</file>
        <append>true</append>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_PATH}/info_${type}.%d{yyyy-MM-dd}.gz</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] [%thread] %logger %msg%n</pattern>
        </encoder>
    </appender>

    <!-- TRACE > DEBUG > INFO > WARN > ERROR > OFF -->
    <!-- Root Logger -->
    <root level="INFO">
        <appender-ref ref="console"/>
        <appender-ref ref="INFO_LOG"/>
    </root>
</configuration>
```

<br>

#### Appender 영역

Appender 영역은 로그의 형태를 설정하고 어떤 방법으로 출력할지를 설정하는 곳이다. Appender자체는 하나의 인터페이스를 의미하며, 하위에 여러 구현체가 존재한다. Appender의 상속 구조는 아래와 같다.



![image](https://user-images.githubusercontent.com/13410737/179763142-16f0423b-d4a6-4a8b-973a-f1c6e8a872b6.png){: .align-center}

<br>

Logback의 설정 파일을 이용하면 위 그림에 등장하는 각 구현체를 등록해서 로그를 원하는 형식으로 출력할 수 있다. Appender의 대표적인 구현체는 다음과 같다.

- `ConsoleAppender` : 콘솔에 로그를 출력
- `FileAppender` : 파일에 로그를 저장
- `RollingFileAppender` : 여러 개의 파일을 순회하면서 로그를 저장
- `SMTPAppender` : 메일로 로그를 전송
- `DBAppender` : 데이터베이스에 로그를 저장

<br>

로그를 어떤 방식으로 저장할지 지정하는 방법을 살펴보자. 아래 코드를 보면 `appender`요소의 `class`속성에 각 구현체를 정의하는 것을 볼 수 있다. 그리고 하단에 `filter` 요소로 각 Appender가 어떤 레벨로 로그를 기록하는지 지정한다.

<br>

```xml
 <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] [%thread] %logger %msg%n</pattern>
        </encoder>
        ... 중략 ...
    <appender name="INFO_LOG" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        ... 중략 ...
        <encoder>
            <pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] [%thread] %logger %msg%n</pattern>
        </encoder>
    </appender>
```

<br>

다음으로 `encoder` 요소를 통해 로그의 표현 형식을 패턴(pattern)으로 정의한다. 사용 가능한 패턴은 몇 가지 정해져 있으며, 대표적인 패턴은 다음과 같다.

<br>

| 패턴            | 의미                                               |
| --------------- | -------------------------------------------------- |
| %Logger{length} | 로거의 이름                                        |
| %-5level        | 로그 레벨. -5는 출력 고정폭의 값                   |
| %msg(%message)  | 로그 메시지                                        |
| %d              | 로그 기록 시간                                     |
| %p              | 로깅 레벨                                          |
| %F              | 로깅이 발생한 애플리케이션 파일명                  |
| %M              | 로깅이 발생한 메서드 이름                          |
| %I              | 로깅이 발생한 호출지의 정보                        |
| %thread         | 현재 스레드명                                      |
| %t              | 로깅이 발생한 스레드명                             |
| %c              | 로깅이 발생한 카테고리                             |
| %C              | 로깅이 발생한 클래스명                             |
| %m              | 로그 메시지                                        |
| %n              | 줄바꿈                                             |
| %r              | 애플리케이션 실행 후 로깅이 발생한 시점까지의 시간 |
| %L              | 로깅이 발생한 호출 지점의 라인수                   |

<br>

위와 같은 패턴을 활용해 다음과 같은 패턴을 만들 수 있다.

<br>

```xml
<pattern>[%d{yyyy-MM-dd HH:mm:ss.SSS}] [%-5level] [%thread] %logger %msg%n</pattern>
```

<br>

#### Root 영역

설정 파일에 정의된 Appender를 활용하려면 Root 영역에서 Appender를 참조해서 로깅 레벨을 설정한다. 만약 특정 패키지에 대해 다른 로깅 레벨을 설정하고 싶다면 root 대신 logger를 사용해 아래와 같이 지정할 수 있다.

<br>

```xml
    <root level="INFO">
        <appender-ref ref="console"/>
        <appender-ref ref="INFO_LOG"/>
    </root>
```

<br>

또는

<br>

```xml
    <logger name="com.springboot.api.controller" level="DEBUG" additivity="false">
        <appender-ref ref="console"/>
        <appender-ref ref="INFO_LOG"/>
    </logger>
```

<br>

`logger` 요소의 `name` 속성에는 패키지 단위로 로깅이 적용될 범위를 지정하고 `level`속성으로 로그 레벨을 지정한다. `additivity` 속성은 앞에서 지정한 패키지 범위에 하위 패키지를 포함할지 여부를 결정한다. 기본값은 `true`이며, 이 경우 하위 패키지를 모두 포함한다.

<br>

> **💡 Tip**
>
> Logback에 관한 더 자세한 내용은 공식 사이트의 매뉴얼을 참고한다.
>
> - <a href="https://https://logback.qos.ch/manual/introduction.html" target="_blank">https://https://logback.qos.ch/manual/introduction.html</a>

<br>

#### Logback 적용하기

이제 실습 중인 프로젝트에 Logback를 적용해 보겠다. Logback은 출력할 메시지를 Appender에게 전달할 Logger 객체를 각 클래스에 정의해서 사용한다. 이전에 작성했던 `GetController`에 `Logger`를 적용해보겠다.  아래와 같이 `GetController`의 `LOGGER` 전역 변수로 `Logger`객체를 정의한다.

<br>

```java
@RestController
@RequestMapping("/api/v1/get-api")
public class GetController {
    
    private final Logger LOGGER = LoggerFactory.getLogger(GetController.class);

    ...(후략)
}
```

<br>

 `Logger`는 `LoggerFactory`를 통해 객체를 생성한다. 이 때 클래스의 이름을 함께 지정해서 클래스의 정보를 `Logger`에서 가져가게 한다.

이어서 아래와 같이 로그를 출력하는 코드를 삽입해보자. 

<br>

```java
    //http://localhost:8080/api/v1/get-api/hello
    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String getHello(){
        LOGGER.info("getHello 메서드가 호출되었습니다.");
        return "Hello World";
    }
    //http://localhost:8080/api/v1/get-api/name
    @GetMapping(value = "/name")
    public String getName(){
        LOGGER.info("getName 메서드가 호출되었습니다.");
        return "Flature";
    }
```

<br>

`GetController`의 코드를 수정하면 `info` 레벨에서 로그가 출력된다. 실제로 로그가 출력되는지 확인하기 위해 Swagger 페이지에 접속해서 테스트를 진행해보겠다. 아래와 같이 `Logger`를 삽입ㅇ한 메서드를 선택한 후 [Execute] 버튼을 눌러 테스트를 실행한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179768119-1eda7dcd-6661-4f5d-b743-47b03b26df84.png){: .align-center}

<br>

로그를 통해 컨트롤러에 들어오는 값을 확인하고 싶다면 아래와 같이 코드를 작성한다.

<br>

```java
    //http://localhost:8080/api/v1/get-api/variable1/{String 값}
    @GetMapping(value = "variable1/{variable}")
    public String getVariable1(@PathVariable String variable){
        LOGGER.info("@PathVariable을 통해 들어온 값 : {}", variable);
        return variable;
    }
```

<br>

위 어럼 변수를 지정해 변수로 들어오는 값을 로깅할 수도 있다. 변수의 값이 들어갈 부분을 중괄호 ({})로 지정하면 호매팅을 통해 로그 메시지가 구성된다.

<br>

> 자바에서 문자열을 합치는 방법은 여러 가지 있다. 다음과 같은 방법을 알아보고 연산 속도를 비교해보면 더 좋은 코드를 작성하는 데 도움이 된다.
>
> 1. +연산자
> 2. String 클래스의 concat() 메서드 활용
> 3. String 클래스의 append() 메서드 활용
> 4. String 클래스의 format() 메서드 활용

<br>

Swagger를 통해 'Wikibooks' 라는 단어를 입력하고 호출하면 아래와 같은 출력 결과를 확인할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179769078-2d65ba6b-b300-4528-b017-2a6fea7ba1ba.png){: .align-center}

<br>

> **로그로 무엇을 기록해야 할까?**
>
> 이번 장에서는 로깅을 설정하고 출력하는 방법을 다뤄봤다. 이제 로그에 어떤 내용을 포함해야 할지 고민해 보자. 애플리케이션마다 출력해야 할 로그가 다르기 때문에 한번쯤 고민해볼 만한 주제이다.
