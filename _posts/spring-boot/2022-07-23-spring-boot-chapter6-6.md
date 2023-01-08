---
title : "[SpringBoot][Chapter6.6] DB연동 - 데이터베이스 연동"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij]
toc: true
toc_sticky : true
published : true
date : 2022-07-23
last_modified_at : 2022-07-23
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



# 데이터베이스 연동

## 프로젝트 생성

5장에서 생성한 방식과 동일하게 프로젝트를 생성한다. 버전은 이전과 같은 2.5.6 버전이며 groupId는 기존과 동일하게 'com.springboot' 로 설정하고 name은 'jpa', 'artifactId'는 'jpa'로 설정한다. 의존성 선택 단계에서는 다음의 라이브러리를 선택한다.

- Developer Tools : Lombok, Srping Configuration Processor
- Web : Spring Web
- SQL : Spring Data JPA, MariaDB Driver



그리고 5.6절에서 진행했던 Swagger 의존성을 `pom.xml`파일에 추가한다. 새로 프로젝트 만들었다면 이전 장에서 사용한 프로젝트에서 다음과 같은 파일들은 가져온다.

**소스코드**

- config/SwaggerConfiguration.java

**리소스**

- logback-spring.xml



`SwaggerConfiguration`파일을 그대로 복사하면 `api()` 메서드 내의 `basePackage` 가 일치하지 않아 정상적으로 동작하지 않기 때문에 아래와 같이 수정한다.

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
                .apis(RequestHandlerSelectors.basePackage("com.springboot.jpa"))
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

Spring Data JPA 의존성을 추가한 후에는 별도의 설정이 필요하다. 애플리케이션이 정상적으로 실행될 수 있게 연동할 데이터베이스의 정보를 `application.properties`에 작성해야 한다. 이 설정 없이는 스프링 부트 애플리케이션이 실행되지 않는다. 아래와 같이 데이터베이스 관련된 내용을 추가한다.

<br>

```properties
spring.datasource.driverClassName=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://localhost:3306/springboot
spring.datasource.username=root
spring.datasource.password=password

spring.jpa.hibernate.ddl-auto=create
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

<br>

1번 줄인 `spring.datasource.driverClassName`에는 연동하려는 데이터베이스의 드라이버를 정의한다. 마리아 DB의 경우 `org.mariadb.jdbc.driver`를 입력하고, `pom.xml`에도 `mariadb-java-client` 의존성이 필요하다.

2번 줄의 `spring.datasource.url` 항목에서 마리아DB의 경로임을 명시하고 경로와 데이터베이스명을 입력한다.

3~4번 줄에는 데이터베이스를 설치할 때 설정한 계정 정보를 기입한다. 사실 프로퍼티 파일에 패스워드가 그대로 들어가는 것은 보안상 취약하기에 일반적으로 패스워드를 암호화해서 사용한다.

6~8번 줄은 하이버네이트를 사용할 때 활성화할 수 있는 선택사항이다. 6번 줄의 `ddl-auto`는 의미 그대로 데이터베이스를 자동으로 조작하는 옵션이다. 여기서 사용할 수 있는 옵션과 설명은 아래와 같다.

<br>

- create : 애플리케이션이 가동되고 SessionFactory가 실행될 때 기존 테이블을 지우고 새로 생성한다.
- create-drop : create 와 동일한 기능을 수행하나 애플리케이션울 종료하는 시점에 테이블을 지운다
-  update : SessionFactory가 실행될 때 객체를 검사해서 변경된 스키마를 갱신한다. 기존에 저장된 데이터는 유지된다.
- validate : update처럼 객체를 검사하지만 스키마는 건드리지 않는다. 검사 과정에서 데이터베이스의 테이블 정보와 객체의 정보가 다르면 에러가 발생한다.
- none : ddl-auto 기능을 사용하지 않는다.

<br>

운영 환경에서는 `create`, `create-droup`, `update` 기능은 사용하지 않는다. 데이터베이스에 축적된 데이터를 지워버릴 수도 있고, 사람의 실수로 객체의 정보가 변경됐을 때 운영 환경의 데이터베이스 정보까지 변경될 수 있기 때문이다. 운영 환경에서는 대체로 `validate`나 `none`을 사용한다. 반면 개발 환경에서는 `create ` 또는 `update`를 사용하는 편이다.

7번 줄의 `show-sql`은 로그에 하이버네이트가 생성한 쿼리문을 출력하는 옵션이다. 아무 설정이 없으면 저장에 용이한 형태로 출력되기 때문에 사람이 보기에는 불편하게 한 줄로 출력된다. 8번 중에서 사용한 `format_sql` 온션으로 사람이 보기 좋게 포매팅할 수 있다.
