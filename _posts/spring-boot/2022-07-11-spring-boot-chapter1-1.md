---
title : "[SpringBoot][Chapter1.1] 스프링 부트란? - 스프링 프레임워크"
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



## 스프링 프레임워크

스프링 프레임워크(Spring Framework)는 자바(Java) 기반의 애플리케이션 프레임워크로 엔터프라이즈급 어플리케이션을 개발하기 위한 다양한 기능을 제공한다. 스프링은 목적에 따라 다양한 프로젝트를 제공하는데 그 중 하나가 스프링 부트이다.

스프링 프레임워크는 자바에서 가장 많이 사용하는 프레임 워크이다. 스프링은 자바 언어를 이용해 엔터프라이즈급 개발을 편리하게 만들어주는 '오픈소스 경량급 애플리케이션 프레임워크'로 불리고 있다. 쉽게 말해서 <u>자바로 애플리케이션을 개발하는 데 필요한 기능을 제공하고 쉽게 사용하도록 돕는 도구</u>이다.



> 엔터프라이즈급 개발 이란?
>
> '엔터프라이즈급 개발'은 기업 환경을 대상으로 하는 개발을 뜻한다. 네이버나 카카오톡 같은 대규모 데이터를 처리하는 환경을 엔터프라이즈 환경이라고 부른다. 스프링은 이 환경에 알맞게 설계돼 있어 개발자는 애플리케이션을 개발할 때 많은 요소를 프레임워크에 위임하고 비즈니스 로직을 구현하는데 집중할 수 있다.



- 스프링의 핵심 가치
  - **<u>"애플리케이션 개발에 필요한 기반을 제공해서 개발자가 비즈니스 로직 구현에만 집중할 수 있게끔 하는 것"</u>**



### 제어 역전

일반적인 자바 개발의 경우 객체를 사용하기 위해서는 객체를 선언하고 해당 객체의 의존성을 생성한 후 객체에 제공하는 기능을 사용한다. 즉, 객체를 생성하고 사용하는 일련의 작업을 개발자가 직접 제어하는 구조.

```java
//일반적인 자바 코드에서의 객체 사용법

@RestController
public class NoDIController{
    private MyService service = new MyServiceImpl();
    
    @GetMapping("/no-di/hello")
    public String getHello(){
        return service.getHello();
    }
}
```

하지만 제어 역전(IoC; Inversion of Control)을 특징으로 하는 스프링은 기존 자바 개발 방식과 다르게 동작한다.

IoC를 적용한 환경에서는 사용할 객체를 직접 생성하지 않고 객체의 생명주기 관리를 외부에 위임한다. 여기서 '외부'는 스프링 컨테이너(Spring Container) 또는 IoC컨테이너(IoC Container) 를 의미 한다. **<u>객체의 관리를 컨테이너에 맡겨 제어권이 넘어간 것을 제어 역전이라 부르며, 제어 역전을 통해 의존성 주입(DI; Dependency Injection), 관점 지향 프로그래밍(AOP; Aspect-Oriented Programming)등이 가능해 진다.</u>**



### 의존성 주입(DI)

의존성 주입(DI; Dependency Injection)이란 제어 역전의 방법 중 하나로, 사용할 객체를 직접 생성하지 않고 외부 컨테이너가 생성한 객체를 주입받아 사용하는 방식이다.

스프링에서 의존성을 주입 받는 방법은 세 가지가 있다.

1. **생성자**를 통한 의존성 주입
2. **필드 객체 선언**을 통한 의존성 주입
3. **setter 메서드**를 통한 의존성 주입

스프링에서는 @Autowired라는 어노테이션(annotation)을 통해 의존성을 주입할 수 있다. 

> 스프링 4.3 이후 버전은 생성자를 통해 의존성을 주입할 때 @Autowired 어노테이션을 생략할 수 있지만 가독성을 위해 어노테이션을 명시하는 것이 좋다.

```java
// 생성자를 통한 의존성 주입
@RestController
public class DIController{
    MyService myService;
    @Autowired
    public DIContrroller(MyService myService){
        this.myService = myService;
    }
    
    @GetMapping("/di/hello")
    public String getHello(){
        return myService.getHello();
    }
}
```



```java
//필드 객체 선언을 통한 의존성 주입
@RestController
public class FieldInjectionController{
    @Autowired
    private MyService myService;
}
```



```java
//setter 메서드를 통한 의존성 주입
@RestController
public class SetterInjectionController{
    MyService myService;
    
    @Autowired
    public void setMyService(MyService myService){
        this.myService = myService;
    }
}
```

스프링 공식 문서에서 권장하는 의존성 주입 방법은 생성자를 통해 의존성을 주입 받는 방식이다. 다른 방식과는 다르게 생성자를 통해 의존성을 주입받는 방식은 레퍼런스 객체 없이는 객체를 초기화할 수 없게 설계할 수 있기 때문이다.



### 관점 지향 프로그래밍(AOP)

관점 지향 프로그래밍(AOP; Aspect-Oriented Programming)은 스프링의 아주 중요한 특징이다.  간혹 AOP를 OOP의 대체 개념으로 오해하는 사람들이 있지만 AOP는 OOP를 더 잘 사용하도록 돕는 개념으로 보는것이 바람직하다.

AOP는 관점을 기준으로 묶어 개발하는 방식이다. 여기서 관점(aspect)이란 어떤 기능을 구현할 때 그 <u>기능을 '핵심 기능'과 '부가 기능'으로 구분해 각각을 하나의 관점으로 보는것을 의미</u>한다.

'핵심 기능'은 비즈니스 로직을 구현하는 과정에서 비즈니스 로직이 처리하려는 목적 기능을 말한다. 예를 들면, 클라이언트로부터 상품 정보 등록 요청을 받아 데이터베이스에 저장하고, 그 상품 정보를 조회하는 비즈니스 로직을 구현한다면 (1) 상품 정보를 데이터베이스에 저장하고, (2) 저장된 상품 정보 데이터를 보여주는 코드가 '핵심 기능'이다.

그런데 실제 애플리케이션 개발할 때는 핵심 기능에 부가 기능을 추가할 상황이 생긴다. 핵심 기능인 비즈니스 로직 사이에 로깅 처리를 하거나 트랜잭션을 처리하는 코드를 예로 들 수 있다.

일반적인 OOP형식으로 비즈니스 로직을 작성하면 아래처럼 비즈니스 동작 흐름이 발생한다.

![image](https://user-images.githubusercontent.com/13410737/178282764-e2cdde16-0a4a-4886-9991-9157795bc0f9.png)

OOP 방식의 애플리케이션 로직에서는 객체마다 핵심 기능을 수행하기 위한 로직과 함께 부가 기능인 로깅, 트랜잭션 등의 코드를 작성한다. 상품정보 등록 기능과 상품 정보 조회 기능은 엄연히 다른 기능으로, 각자 로직이 구현되어 있지만 유지보수 목적이나 데이터 베이스 접근을 위해 작성된 로깅과 트랜잭션 영역은 상품정보를 등록할 때나 상품정보를 조회할 때 동일한 기능을 수행할 확률이 높다. 즉, <u>핵심 기능을 구현한 두 로직에 동일한 코드가 포한된다는 것을 의미한다.</u>

AOP 관점에서는 아래와 같이 부가 기능은 핵심 기능이 어떤 기능인지에 무관하게 로직이 수행되기 전 또는 후에 수행되기만 하면 된다.

![image](https://user-images.githubusercontent.com/13410737/178283548-2197c712-6bda-46f9-b384-f42e1189118d.png)

이처럼 여러 **비즈니스 로직에서 반복되는 부가 기능을 하나의 공통 로직으로 처리하도록 모듈화해 삽입하는 방식을 AOP 라 한다.** 이러한 AOP를 구현하는 방법은 크게 세 가지가 있다.

1. 컴파일 과정에 삽입하는 방식
2. 바이트코드를 메모리에 로드하는 과정에 삽입하는 방식
3. 프락시 패턴을 이용한 방식

이 가운데 스프링은 디자인 패턴 중 하나인 프락시 패턴을 통해 AOP 기능을 제공하고 있다. 

스프링 AOP의 목적은 OOP와 마찬가지로 모듈화해서 재사용 가능한 구성을 만드는 것이고, 모듈화된 객체를 편하게 적용할 수 있게 함으로써 개발자가 비즈니스 로직을 구현하는 데만 집중할 수 있게 도와 주는 것이다.



### 스프링 프레임워크의 다양한 모듈

스프링 프레임워크는 기능별로 구분된 약 20여 개의 모듈로 구성돼 있다.

![image](https://user-images.githubusercontent.com/13410737/178284622-a41b63f0-0eb7-4ad8-829e-356aad156cbd.png)

스프링 프레임워크 공식 문서에서는 스프링 버전별로 다은 다이어그램을 제시하고 있지만 큰 틀은 유사하다. 그리고 스프링 프레임워크를 사용한다고 해서 모든 모듈을 사용할 필요는 없다. 애플리케이션 개발에 필요한 모듈만 선택해서 사용하게끔 설계돼 있으며, 이를 <u>경량 컨테이너 설계라고 부른다.</u>

