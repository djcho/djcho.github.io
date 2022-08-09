---
title : "[SpringBoot][Chapter11.1] 액추에이터 - 프로젝트 생성"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Actuator]
toc: true
toc_sticky : true
published : true
date : 2022-08-10
last_modified_at : 2022-08-10
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





애플리케이션을 개발하는 단계를 지나 운영 단계에 접어들면 애플리케이션이 정상적으로 동작하는지 모니터링하는 환경을 구축하는 것이 매우 중요해진다. 스프링 부트 액추에이터는 HTTP 엔드포인트나 JMX를 활용해 애플리케이션을 모니터링하고 관리할 수 있는 기능을 제공한다. 이번 장에서는 액추에이터의 환경을 설정하고 활용하는 방법을 다룰 예정이다.

<br>

> **💡 Tip. JMX란?**
>
> JMX(Java Management Extensions)는 실행 중인 애플리케이션의 상태를 모니터링하고 설정을 변경할 수 있게 해주는 APP이다. JMX를 통해 리소스 관리를 하려면 MBeans(Managed Beans)를 생성해야 한다.



## 프로젝트 생성 및 액추에이터 종속성 추가

이번 장에서 사용할 새로운 프로젝트를 생성하겠다. 스프링 부트 버전은 이전과 같은 2.5.6 버전으로 진행하며, 다음과 같은 내용을 설정한다.

- groupId : com.springboot
- artifactId : actuator
- name : actuator
- Developer Tools : Spring Configuration Processor
- Web : Spring Web

<br>

그리고 이전 장에서 사용한 `SwaggerConfiguration` 클래스를 가져오고 그에 따른 의존성을 추가한다.

액추에이터 기능을 사용하려면 애플리케이션에 spring-boot-starter-actuator 모듈의 종속성을 추가해야 한다. 아래와 같이 `pom.xml`파일에 추가하면 된다.

<br>

```xml
<dependencies>
    ... 생략 ...
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ... 생략 ...
</dependencies>
```

