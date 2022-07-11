---
title : "[SpringBoot][Chapter2.2]스프링 부트의 동작 방식"
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



## 스프링 부트의 동작 방식

스프링 부트에서 spring-boot-starter-web 모듈을 사용하면 기본적으로 톰캣(Tomcat)을 사용하는 스프링 MVC 구조를 기반으로 동작한다.

아래는 일반적인 웹 요청이 들어왔을 때의 스프링 부트의 동작 구조이다.

![image](https://user-images.githubusercontent.com/13410737/178304203-73309bc8-8e24-4c90-81e6-dd645acd09ee.png){: .align-center}

서블릿은 클라이언트의 요청을 처리하고 결과를 반환하는 자바 웹 프로그래밍 기술이다. 일반적으로 서블릿은 서블릿 컨테이너(Servlet Container)에서 관리한다.

서블릿 컨테이너는 서블릿 인스턴스(Servlet Instance)를 생성하고 관리하는 역할을 수행하는 주체로서 톰캣은 WAS의 역할과 서블릿 컨테이너의 역할을 수행하는 대표적인 컨테이너이다.

- 서블릿 컨테이너의 특징
  - 서블릿 객체를 생성, 초기화, 호출, 종료하는 생명주기를 관리한다.
  - 서블릿 객체는 싱글톤 패턴으로 관리된다.
  - 멀티 스레딩을 지원한다.

스프링에서는 DispatcherServlet이 서블릿의 역할을 수행한다. 일반적으로 스프링은 톰캣을 임베드(embed)해 사용한다. 그렇게 때문에 서블릿 컨테이너와 DispatcherServlet 은 자동 설정된 web.xml 의 설정 값을 공유한다.

- DispatcherServlet 의 동작

  (1) DispatcherServlet 으로 요청(HttpServletRequest)이 들어오면 DispatcherServlet은 핸들러 매핑(Handler Mapping)을 통해 요청 URL에 매핑된 핸들러를 탐색한다. 여기서 핸들러는 컨트롤러(Controller)를 의미한다.

  (2) 핸들러 어댑터(HandlerAdapter)로 컨트롤러를 호출한다.

  (3) 핸들러 어댑터에 컨트롤러의 응답이 돌아오면 ModelAndView로 응답을 가공해 반환한다.

  (4) 뷰 형식으로 리턴하는 컨트롤러를 사용할 때는 뷰 리졸버(View Resolver)를 통해 뷰(View)를 받아 리턴한다.

핸들러 매핑은 요청 정보를 기준으로 어떤 컨트롤러를 사용할지 선정하는 인터페이스이다. 핸들러 매핑 인터페이스는 여러 구현체를 가지며, 대표적인 구현체 클래스는 다음과 같다.

- `BeanNameUrlHandlerMapping`

  - 빈 이름을 URL로 사용하는 매핑 전략이다.

  - 빈을 정의할 때 슬래시('/')가 들어가면 매핑 대상이 된다.

  - 예) @Bean("/hello")


- `ControllerClassNameHandlerMapping`

  - URL과 일치하는 클래스 이름을 갖는 빈을 컨트롤러로 사용하는 전략이다.

  - 이름 중 Controller를 제외하고 앞 부분에 작성된 suffix를 소문자로 매핑한다.


- `SimpleUrlHandlerMapping`
  - URL 패턴에 매핑된 컨트롤러를 사용하는 전략이다.


- `DefaultAnnotationHandlerMapping`
  - 어노테이션으로 URL과 컨트롤러를 매핑하는 방법이다.




뷰 리졸버는 뷰의 렌더링 역할을 담당하는 뷰 객체를 반환한다.

![image](https://user-images.githubusercontent.com/13410737/178307234-2b870321-4503-4f93-a197-7d1ebaa80fee.png){: .align-center}

뷰가 없는 REST 형식의 @ResponseBody를 사용한다면 뷰 리졸버를 호출하지 않고 MessageConverter 를 거쳐 JSON 형식으로 변환해서 응답한다.

![image](https://user-images.githubusercontent.com/13410737/178307708-c258722d-4700-42b6-a23a-a287dc315373.png){: .align-center}

MessageConverter는 요청과 응답에 대한 Body 값을 변환하는 역할을 수행한다. 스프링 부트의 자동 설정 내역을 보면 httpMessageConverter 인터페이스를 사용하고 있다. 아래 예제는 spring.factories 에 정의된 HttpMessageConvertersAutoConfiguration 클래스이다.

```java
//HttpMessageConverterAutoConfiguration 에서 확인할 수 있는 HttpMessageConvert
@Configuration(proxyBeanMethods = false)
@ConditionalOnClass(HttpMessageConverter.class)
@Conditional(NotReactiveWebApplicationCondition.class)
@AutoConfigureAfter({ GsonAutoConfiguration.class, JacksonAutoConfiguration.class,
                    JsonbAutoConfiguration.class})
@Import({ JacksonHttpMessageConvertersConfiguration.class, GsonHttpMessageConvertersConfiguration.class,
        JsonbHttpMessageConvertersConfiguration.class })
public class HttpMessageConvertersAutoConfiguration{
    static final String PREFERRED_MAPPER_PROPERTY = "spring.mvc.converts.preferred-json-mapper";
    
    @Bean
    @ConditionalOnMissingBean
    public HttpMessageConverters messageConverters(ObjectProvider<HttpMessageConverter<?>> converters){
        return new HttpMessageConvertes(converters.orderedStream().collect(Collectors.toList()));
    }
    
    ...생략...
}
```

2번째 줄의 httpMessageConverter 인터페이스를 빈으로 등록하는 것을 볼 수 있다. 해당 인터페이스를 기반으로 하는 구현체 클래스는 다양하며, Content-Type을 참고해서 Converter 를 선정한다. **<u>스프링 부트에서는 자동 설정되기 때문에 별도 설정이 필요하지 않다.</u>**

> 스프링 부트는 자동 설정을 지원하지 때문에 애플리케이션을 편리하게 개발할 수 있다. 하지만 심도 있는 개발을 위해서는 스프링의 동작 원리를 파악해야 한다. 스프링 모듈만으로 개발을 진행해보면 동작 원리를 파악하는데 큰 도움이 된다.
