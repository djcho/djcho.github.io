---
title : "[SpringBoot][Chapter1.2] 스프링 프레임워크 vs. 스프링 부트"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Framework, Java]
toc: true
toc_sticky : true
published : true
date : 2022-07-11
last_modified_at : 2022-07-11
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## 스프링 프레임워크 .vs 스프링 부트

스프링 프레임워크는 기존 개발 방식의 문제와 한계를 극복하기 위해 다양한 기능을 제공한다. 하지만 기능이 많은 만큼 설정이 복잡한 편이다.

```xml
<!--스프링의 하이버네이트 관련 설정 파일-->

<bean id="dataSource" class="com.mchange.V2.c3p0.ComboPooledDataSource" destroy-method="close">
    <property name="driverClass" value="${db.driver}"/>
    <property name="jdbcUrl" value="${db.url}"/>
    <property name="user" value="${db.username}"/>
    <property name="password" value="${db.password}"/>
</bean>

<jdbc:initialize-database data-source="dataSource">
    <jdbc:script location="classpath:config/schema.sql"/>
    <jdbc:script location="classpath:config/data.sql"/>
</jdbc:initialize-database>

<bean class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean" id="entityManagerFactory">
    <property name="persistenceUnitName" value="hsql_pu"/>
    <property name="dataSource" ref="dataSource"/>
</bean>

<bean id="transactionManager" class="org.springframework.orm.jpa.JpaTransactionManager">
    <property name="entityManagerFactory" ref="entityManagerFactory"/>
    <property name="dataSource" ref="dataSource"/>
</bean>

<tx:annotation-driven transaction-manager="transactionManager"/>
```

위처럼 필요한 모듈들을 추가하다 보면 설정이 복잡해지는 문제를 해결하기 위해 등장한 것이 스프링 부트(Spring Boot)이다.

> **Spring Boot makes it easy to create stand-alone, production-grade Spring based Applications that you can "just run"**
>
> **(스프링 부트를 이용하면 단독으로 실행 가능한 사용 수준의 스프링 기반 애플리케이션을 손쉽게 만들 수 있습니다.)**



즉, 별도의 복잡한 설정을 하지 않아도 스프링 부트를 사용하면 개발이 쉬워진다는 뜻이다.



### 의존성 관리

스프링 프레임워크에서는 개발에 필요한 각 모듈의 의존성을 직접 설정해야 했다. 또 호환되는 버전을 명시해야 정상 동작한다. 애플리케이션에서 사용하는 스프링 프레임워크나 라이브러리의 버전을 올리는 상황에서는 연관된 다른 라이브러리의 버전까지 고려해야 한다. 

하지만 스프링 부트에서는 이 같은 불편함을 해소하기 위해 'spring-boot-starter'라는 의존성을 제공한다. spring-boot-starter의 의존성은 여러 종류가 있고, 각 라이브러리의 기능과 관련해서 자주 사용되고 서로 호환되는 버전의 모듈 조합을 제공한다.

많이 사용되는 spring-boot-starter 라이브러리

- spring-boot-starter-web : 스프링 MVC 를 사용하는 RESTful 애플리케이션을 만들기 위한 의존성이다. 기본으로 내장 톰캣(Tomcat)이 포함돼 있어 jar 형식으로 실행 가능하다.
- spring-boot-starter-test : JUnit Jupiter, Mockito 등의 테스트용 라이브러리를 포함한다.
- spring-boot-starter-jdbc : HikariCP 커넥션 풀을 활용한 JDBC 기능을 제공한다.
- spring-boot-starter-security : 스프링 시큐리티(인증, 권한 인가 등) 기능을 제공한다.
- spring-boot-starter-data-jpa : 하이버네이트를 활용한 JPA 기능을 제공한다.
- spring-boot-starter-cache : 스프링 프레임워크의 캐시 기능을 지원한다.



'spring-boot-starter'의 여러 라이브러리를 함께 사용할 때는 의존성이 겹칠 수 있다. 이로 인해 버전 충돌이 발생할 수 있는데 ,의존성 조합 충돌 문제가 없도록 'spring-boot-starter-parent'가 검증된 조합을 제공한다.

{: .notice--warning}

<br>

### 자동 설정

스프링 부트는 스프링 프레임워크의 기능을 사용하기 위한 자동 설정(Auto Configuration)을 지원한다. 자동 설정은 애플리케이션에 추가된 라이브러리를 실행하는 데 필요한 환경 설정을 알아서 찾아준다. 즉, 애플리케이션을 개발하는 데 필요한 의존성을 추가하면 프레임워크가 이를 자동으로 관리해준다. 

```java
//스프링 부트의 메인 애플리케이션 코드

@SpringBootApplication
public class SpringBootApplication{
    public static void main(String[] args){
        SpringApplication.run(SpringBootApplication.class, args);
    }
}
```

@SpringBootApplication 어노테이션은 여러 어노테이션을 합쳐 놓은 인터페이스지만 기능 위주로 보면 크게 다음 세 개의 어노테이션을 합쳐놓은 구성이다.

- @SpringBootConfiguration
- @EnableAutoConfiguration
- @ComponentScan

스프링 부트 애플리케이션이 실행되면 @ComponentScan 어노테이션이 @Component 시리즈 어노테이션이 붙은 클래스를 발견해 빈(bean)으로 등록한다. 이후 @EnableAutoConfiguration 어노테이션을 통해 'spring-boot-autoconfigure' 패키지 안에 spring.factories 파일을 추가해 다양한 자동 설정이 일부 조건을 거쳐 적용 된다.



### 내장 WAS

스프링 부트의 각 웹 애플리케이션에는 내장 WAS(Web Application Server)가 존재한다.  스프링 부트의 자동 설정 기능은 톰캣에도 적용되므로 특별한 설정 없이도 톰캣을 실행할 수 있으며 필요에 따라서는 톰캣이 아닌 다른 웹서버(Jetty, Undertow 등)으로 대체할 수도 있다.



### 모니터링

개발이 끝나고 서비스를 운영하는 시기에는 해당 시스템이 사용하는 스레드 ,메모리, 세션 등의 주요 요소들을 모니터링 해야 한다. 스프링 부트에는 스프링 부트 액추에이터(Spring Boot Actuator)라는 자체 모니터링 도구가 있다.
