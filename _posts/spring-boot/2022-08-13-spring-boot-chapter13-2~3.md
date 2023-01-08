---
title : "[SpringBoot][Chapter13.2~3] 서비스의 인증과 권한 부여 - 스프링 시큐리티"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, SpringSecurity]
toc: true
toc_sticky : true
published : true
date : 2022-08-14
last_modified_at : 2022-08-14
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}







# 스프링 시큐리티

스프링 시큐리티는 애플리케이션의 인증, 인가 등의 보안 기능을 제공하는 스프링 하위 프로젝트 중 하나이다. 보안과 관련된 많은 기능을 제공하기 때문에 스프링 시큐리티를 활용하면 더욱 편리하게 원하는 기능을 설계할 수 있다.



## 스프링 시큐리티의 동작 구조

스프링 시큐리티는 서블릿 필터(Servlet Filter)를 기반으로 동작하며, 아래와 같이 `DispatcherServlet`앞에 필터가 배치돼 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/184499834-f8891aa1-abe0-4eb3-86c8-0b089476c0dc.png){: .align-center}

<br>

위 그림의 필터체인(FilterChain)은 서블릿 컨테이너에서 관리하는 `ApplicationFilterChain`을 의미한다. 클라이언트에서 애플리케이션으로 요청을 보내면 서블릿 컨테이너는 URI를 확인해서 필터와 서블릿을 매핑한다. 스프링 시큐리티는 사용하고자 하는 필터체인을 서블릿 컨테이너의 필터 사이에서 동작시키기 위해 아래와 같이 `DelegatingFilterProxy`를 사용한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/184499856-03d97c30-0f9e-4696-9403-d1715886db90.png){: .align-center}

<br>

`DelegatingFilterProxy`는 서블릿 컨테이너의 생명주기와 스프링 애플리케이션 컨텍스트(Application Context) 사이에서 다리 역할을 수행하는 필터 구현체이다. 표준 서블릿 필터를 구현하고 있으며, 역할을 위임할 필터체인 프록시(`FliterChainProxy`)를 내부에 가지고 있다. 필터체인 프록시는 스프링 부트의 자동 설정에 의해 자동 생성된다.

필터체인 프록시는 스프링 시큐리티에서 제공하는 필터로서 보안 필터체인(`SecurityFilterChain`)을 통해 많은 보안 필터(Security Filter)를 사용할 수 있다. 필터체인 프록시에서 사용할 수 있는 보안 필터체인은 `List` 형식으로 담을 수 있게 설정돼 있어 URI 패턴에 따라 특정 보안필터 체인을 선택해 사용하게 된다.

보안필터 체인에서 사용하는 필터는 여러 종류가 있으며, 각 필터마다 실행되는 순서가 다르다. 공식 문서에서 소개하는 필터의 실행 순서는 다음과 같다.

- `ChannelProcessingFilter`
- `WebAsyncManagerIntegrationFilter`
- `SecurityContextPersitenceFilter`
- `HeaderWriterFilter`
- `CorsFilter`
- `CsrfFilter`
- `LogoutFilter`
- `OAuth2AuthorizationRequestRedirectFilter`
- `Saml2WebSsoAuthenticationRequestFilter`
- `X509AuthenticationFilter`
- `AbstractPreAuthenticatedProcessingFilter`
- `CasAuthenticationFilter`
- `OAuth2LoginAuthenticationFilter`
- `Saml2WebSsoAuthenticationFilter`
- `UsernamePasswordAuthenticationFilter`
- `OpenIdAuthenticationFilter`
- `DefaultLoginPageGeneratingFilter`
- `DefaultLogoutPageGeneratingFilter`
- `ConcurrentSessionFilter`
- `DigestAuthenticationFilter`
- `BearerTokenAuthenticationFilter`
- `BasicAuthenticationFilter`
- `RequestCasheAwareFilter`
- `SecurityContextHolderAwareRequestFilter`
- `JaasApiIntegrationFilter`
- `RememberMeAuthenticationFilter`
- `AnonymousAuthenticationFilter`
- `OAuth2AuthorizationCodeGrantFilter`
- `SessionManagementFilter`
- `ExceptionTranslationFilter`
- `FilterSecurityInterceptor`
- `SwitchUserFilter`

보안 필터체인은 `WebSecurityCOnfigurerAdapter` 클래스를 상속 받아 설정할 수 있다. 앞에서 이야기한 것처럼 필터체인 프록시는 여러 보안 필터체인을 가질 수 있는데, 여러 보안 필터체인을 만드기 위해서는 `WebSecurityConfigurerAdapter` 클래스를 상속받은 클래스를 여러 개 생성하면 된다. 이때 `WebSecurityConfigurerAdapter` 클래스에는 `@Order` 어노테이션을 통해 우선순위가 지정돼 있는데, 2개 이상의 클래스를 생성했을 때 똑같은 성정으로 우선순위가 100이 성정돼 있으면 예외가 발생하기 때문에 상속받은 클래스에서 `@Order` 어노테이션을 지정해 순서를 정하는 것이 중요하다.

별도의 설정이 없다면 스프링 시큐리티에서는 아래와 같이 `SecurityFilterChain`에서 사용하는 필터 중 `UsernamePasswordAuthenticationFilter`를 통해 인증을 처리한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/184500692-0e549c8c-d5ee-466e-babb-997fb9fcc819.png){: .align-center}

<br>

위 그림의 인증 수행 과정을 설명하면 다음과 같다.

1. 클라이언트로부터 요청을 받으면 서블릿 필터에서 `SecurityFilterChain`으로 작업이 위임되고 그중 `UsernamePasswordAuthenticationFilter`(위 그림에서 `AuthenticationFitler`에 해당)에서 인증을 처리한다.
2. `AuthenticationFilter`는 요청 객체(`HttpServletRequest`)에서 `username`과 `password`를 추출해서 토큰을 생성한다.
3. 그리고 나서 `AuthenticationManager`에게 토큰을 전달한다. `AuthenticationManager`는 인터페이스이며, 일반적으로 사용되는 구현체는 `ProviderManager`이다.
4. `ProviderManager`는 인증을 위해 `AuthenticationProvider`로 토큰을 전달한다.
5. `AuthenticationProvider`는 토큰의 정보를 `UserDetailsService`에 전달한다.
6. `UserDetailsService`는 전달받은 정보를 통해 데이터베이스에서 일치하는 사용자를 찾아 `UserDetails`객체를 생성한다.
7. 생성된 `UserDetails` 객체는 `AuthenticationProvider`로 전달되며, 해당 `Provider`에서 인증을 수행하고 성공하게 되면 `ProviderManager`로 권한을 담은 토큰을 전달한다.
8. `ProviderManager`는 검증된 토큰을 `AuthenticationFilter`로 전달한다.
9. `AuthenticationFilter`는 검증된 토큰을 `SecurityContextHolder`에 있는 `SeucirytContext` 에 저장한다.

<br>

이 과정에서 사용된 `UsernamePasswordAuthenticationFilter`는 접근 권한을 확인하고 인증이 실패할 경우 로그인 폼이라는 화면을 보내는 역할을 수행한다. 이 책에서 실습 중인 프로젝트는 화면이 없는 RESTful 애플리케이션이기 때문에 다른 필터에서 인증 및 인가 처리를 수행해야 한다. 이 책에서는 JWT 토큰을 사용해 인증을 수행할 예정이라 JWT와 관련된 필터를 생성하고 `UsernamePasswordAuthenticationFilter` 앞에 배치해서 먼저 인증을 수행할 수 있게 설정하겠다.



> **💡 Tip.**
>
> 스프링 시큐리티에 대한 자세한 내용은 공식 문서를 참고하기 바란다.
>
> - <a href="https://docs.spring.io/spring-security/site/docs/5.5.3/reference/html5/">https://docs.spring.io/spring-security/site/docs/5.5.3/reference/html5/</a>

