---
title : "[SpringBoot][Chapter4.1] 개발하기 - 프로젝트 생성"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij]
toc: true
toc_sticky : true
published : true
date : 2022-07-18
last_modified_at : 2022-07-18
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



# 프로젝트 생성

스프링 부트 프로젝트를 쉽게 만드는 방법으로 1) '인텔리제이 IDEA 에서 프로젝트를 생성하는 방법'과 2) 'Spring Initializr를 이용해 생성하는 방법'이 있다.

<br>

## 인텔리제이 IDEA에서 프로젝트 생성하기

인텔리제이 IDEA 얼티밋 버전은 커뮤니티 버전보다 많은 기능을 지원한다. 그 중 하나가 바로 내장된 Spring Initializr 이다. 이 기능을 이용하면 외부 프로젝트를 생성할 필요 없이 인텔리제이 IDEA 에서 곧바로 스프링 프로젝트를 생성할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179540768-ee7d561c-a77f-42c6-a248-f91c1e499891.png){: .align-center}

<br>

인텔리제이 IDEA는 다양한 형식의 프로젝트를 지원한다. 'Spring Initializr' 는 원래 스프링 공식 사이트에서 제공하는 스프링 부트 프로젝트 생성 기능인데, 인텔리제이 IDEA에도 내장돼 있다. 아래와 같이 'Spring Initializr'를 선택하면 설정이 필요한 항목들이 나온다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179542672-87c91cc8-ed2b-4892-9664-2ce0922f33e2.png){: .align-center}

<br>

각 항목에 대해 다음과 같이 설정한다.

- Name : 프로젝트의 이름을 설정한다.
- Location : 프로젝트를 생성할 위치를 설정한다.
- Language : JVM 상에서 동작하는 언어를 선택한다. 'Java'를 선택한다.
- Type : 빌드 툴을 선택한다. 여기서는 Maven 으로 선택했는데 각자 익숙한 것을 선택해도 좋다.
- Group : 이 프로젝트를 정의하는 고유한 식별자 정보인 그룹을 설정한다. 
- Artifact : 세부 프로젝트를 식별하는 정보를 기입한다.
- Package name : Group과 Aritifact를 설정하면 자동으로 입력된다.
- Project SDK : 11 버전으로 설정한다.
- Java : 11 버전으로 설정한다.
- Packaging : 애플리케이션을 쉽게 배포하고 동작하게 할 파일들의 패키징 옵션이다. 'Jar'를 선택한다.

<br>

> Jar와 War 모두 자바 언어의 툴에서 사용하는 아카이브 파일이다. 애플리케이션의 배포와 동작을 위해 사용되는데, 두 형식의 차이점과 특징을 알아두면 도움이 된다.

다음 단계로 프로젝트에서 사용할 의존성을 추가한다. 의존성은 초기에 추가할 수도 있고 개발을 진행 중에 추가할 수도 있다. 일단 아래와 같이 기본적인 의존성을 선택하고 넘어간다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179543583-45bca178-e42b-4c66-827b-ecfae317d58d.png){: .align-center}

<br>

>💡 Tip.
>
>만약 이 단계에서 스프링 부트 버전으로 2.5.6 버전이 선택되지 않는다면 아무 버전이나 선택해서 프로젝트 생성을 완료한 후에 아래와 같이 po.xml 파일에서 버전을 변경한다.
>
>```xml
><parent>
><groupId>org.springframework.boot</groupId>
><artifactId>spring-boot-starter-parent</artifactId>
><version>2.5.6</version> <!-- 버전 변경 -->
><relativePath/><!-- lookup parent from repository -->
></parent>
>```

<br>

이 단계까지 마치면 인텔리제이 IDEA 우측 하단에 상태 표시줄이 나타나고, 그 사이에 인텔리제이 IDEA가 메이븐(Maven)을 통해 프로젝트를 초기화한다. 그리고 프로젝트에 필요한 의존성을 가져오고 필요한 색인 작업을 한다. 모든 작업이 완료되면 상태 표시중이 사라지면서 아래와 같은 화면을 볼 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179545536-95ff43e9-aa8c-4e6f-b901-09b1c30d9432.png){: .align-center}

<br>

## 스프링 공식 사이트에서 프로젝트 생성하기

<br>

만약 사용 중인 인텔리제이 IDEA가 커뮤니티 버전이라면 지금 소개하는 방법으로 프로젝트를 생성하는 것이 좋다. 스프링 공식 사이트에는 스프링 부트 프로젝트를 자동으로 만들어주는 서비스가 있다.

- https://start.spring.io

위 경로로 접속하면 아래와 같은 화면이 나온다. 이 페이지에서 각 항목을 선택하고 [Generate]버전을 누르면 설정이 적용된 프로젝트 파일을 내려 받을 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179546208-c5fc4a22-6049-4b7c-bfd2-da9c547a27ef.png){: .align-center}

<br>

우선 각 항목에 대해 다음과 같이 설정한다.

- Product : Maven Project
- Language : Java
- Spring Boot : 2.7.1
- Project Metadata
  - Group : com.springboot
  - Artifact : hello
  - Name : hello
  - Description(자유롭게 서술 가능) : Demo project Spring Boot
  - Package Name(자동완성) : com.springboot.hello
  - Packaging: Jar
  - Java : 11

<br>

그리고 Dependencies 항목을 채우기 위해 [ADD DEPENDENCIES...]버튼을 클릭한다. 그럼 아래와 같이 의존성 추가 화면이 나온다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179547040-b624162b-3985-4e42-9fd1-bb4bbd22a5d5.png){: .align-center}

<br>

맨 위 검색창을 활용하거나 마우스 스크롤을 내려 아래와 같이 항목을 추가한다.

- Lombok
- Spring Configuration Processor
- Spring Web

<br>

![image](https://user-images.githubusercontent.com/13410737/179547306-cc6f7a80-e884-46f6-9ad6-bd1eb3e5effa.png){: .align-center}

<br>

모든 옵션을 선택하고 의존성까지 추가했다면 스프링 프로젝트를 생성할 준비가 끝났다. spring initializr 사이트 하단의 [GENERATE] 버튼을 클릭해 프로젝트를 내려 받는다. 내려받은 압축 파일을 프로젝트를 진행할 경로로 옮기고 압축을 푼 후 인텔리제이 IDEA를 실행하고 아래와 같이 프로젝트를 선택해 열어본다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179548037-b8344290-f6b3-4f82-8f14-5383ac22555e.png){: .align-center}

<br>

외부에서 내려받은 프로젝트를 인텔리제이 IDEA에서 열면 경고 문구가 나타나는 경우가 있다. 아래와 같은 경고 문구가 나온다면 [Trust Project] 버튼을 클릭한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179548138-f3104da7-a431-419b-a094-e015f9bbec61.png){: .align-center}

<br>

프로젝트를 처음 열면 몇 가지 초기화 작업이 진행된다. 그리고 아래와 같은 화면이 나오고 인텔리제이 IDEA 우측 하단에 표시된 진행 사항이 모두 완료되면 정상적으로 프로젝트를 이용할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179548394-9b83b2f9-6094-471f-95b3-39f3af7dc6c8.png){: .align-center}