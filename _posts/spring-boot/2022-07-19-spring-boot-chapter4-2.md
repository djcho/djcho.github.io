---
title : "[SpringBoot][Chapter4.2] 개발하기 - pom.xml 살펴보기"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Maven]
toc: true
toc_sticky : true
published : true
date : 2022-07-19
last_modified_at : 2022-07-19
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



# pom.xml(Project Object Model) 살펴보기

pom.xml 파일은 메이븐의 기능을 사용하기 위해 작성하는 파일이다. 이 파일에는 프로젝트, 의존성 라이브러리, 빌드 등의 정보 및 해당 프로젝트를 관리하는 데 필요한 내용이 기술돼 있다. 이 파일을 살펴보기에 앞서 메이븐이라는 빌드 도구에 대해 먼저 살펴보겠다.

<br>

## 빌드 관리 도구

빌드 관리 도구는 JVM이나 WAS가 프로젝트를 인식하고 실행할 수 있게 우리가 작성한 소스코드와 프로젝트에 사용된 파일들(.xml, .jar. .properties)을 빌드하는 도구이다. 개발 규모가 커질수록 관리할 라이브러리가 많아지고 라이브러리 간 버전 호환성을 체크해야 하는 어려움이 발생하는데, 빌드 관리 도구를 이용하면 이 같은 문제를 해결할 수 있다.

<br>

## 메이븐

아파치 메이븐은 자바 기반의 프로젝트를 빌드하고 관리하는데 사용하는 도구이다. 초창기 자바 프로젝트의 대표적 관리 도구였던 Ant를 대체하기 위해 개발됐다. 메이븐의 가장 큰 특징은 pom.xml 파일에 필요한 라이브러리를 추가하면 해당 라이브러리에 필요한 라이브러리까지 함께 내려받아 관리한다는 점이다. 메이븐의 대표 기능을 다음과 같다.

- 프로젝트 관리 : 프로젝트 버전과 아티팩트를 관리한다.
- 빌드 및 패키징 : 의존성을 관리하고 설정된 패키지 형식으로 빌드를 수행한다.
- 테스트 : 빌드를 수행하기 전에 단위 테스트를 통해 작성된 애플리케이션 코드의 정상 동작 여부를 확인한다.
- 배포 : 빌드가 완료된 패키지를 원격 저장소에 배포한다.

<br>

### 메이븐 생명주기

메이븐의 기능은 생명주기 순서에 따라 관리되고 동작한다. 인텔리제이 IDEA에서 생성한 프로젝트의 경우 인텔리제이 IDEA에서 우측에 있는 'Maven' 탭을 클릭하면 아래와 같이 메이븐의 생명주기를 확인할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179550510-a1bf2f50-cdde-4bd6-843a-12b832841e01.png){: .align-center}

<br>

메이븐의 생명주기는 크게 기본 생명주기(Default Lifecycle), 클린 생명주기(Clean Lifecycle), 사이트 생명주기(Site Lifecycle)의 3가지로 구분한다. 각 생명주기에는 아래와 같은 단계(phase)가 존재하며, 특정 단계를 수행하기 위해서는 이전 단계를 마쳐야 한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179552311-51e1ffd0-fbd1-4287-9a34-a4f940b70a8d.png){: .align-center}

<br>

각 단계는 메이븐에서 제공하는 플러그인이 설정된 목표(goal)를 수행하는 방식으로 동작한다. 또한 그림에서 표현되지 않은 세부 단계들이 존재한다. 메이븐의 생명주기 단계를 순차적으로 실행되며, 각 생명주기 단계의 역할은 다음과 같다.

<br>

#### 클린 생명주기

- clean : 이전 빌드가 생성한 모든 파일을 제거한다.

#### 기본 생명주기

- validate : 프로젝트를 빌드하는데 필요한 모든 정보를 사용할 수 있는지 검토한다.
- compile : 프로젝트의 소스코드를 컴파일한다.
- test : 단위 테스트 프레임워크를 사용해 테스트를 실행한다.
- package : 컴파일한 코드를 가져와서 JAR 등의 형식으로 패키징을 수행한다.
- verify : 패키지가 유효하며 일정 기준을 충족하는지 확인한다.
- install : 프로젝트를 사용하는데 필요한 패키지를 로컬 저장소에 설치한다.
- deploy : 프로젝트를 통합 또는 릴리즈 환경에서 다른 곳에 공유하기 위해 원격 저장소에 패키지를 복사한다.

##### 사이트 생명주기

- site : 메이븐의 설정 파일 정보를 기반으로 프로젝트의 문서 사이트를 생성한다.
- site-deploy : 생성된 사이트 문서를 웹 서버에 배포한다.

<br>

메이븐에 관한 더 자세한 내용은 [공식 사이트]에서 확인할 수 있다.

> 여전히 메이븐도 많이 사용하지만 최근에는 [그레이들(Gradle)]이라는 빌드 도구로 전환되는 추세이다. 일례로 안드로이드에서는 그레이들을 표준 빌드 도구로 채택했다. 이 빌드 도구에 대해 알아보면 개발 환경을 구축하는데 도움이 될 것이다.

[공식 사이트]: http://maven.apache.org/
[그레이들(Gradle)]: https://gradle.org/

