---
title : "[SpringBoot][Chapter6.3] DB연동 - JPA"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, JPA]
toc: true
toc_sticky : true
published : true
date : 2022-07-20
last_modified_at : 2022-07-20
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## JPA

JPA(Java Persistence API)는 자바 진영의 ORM 기술 표준으로 채택된 인터페이스의 모음이다. ORM이 큰 개념이라면 JPA는 더 구체화된 스펙을 포함한다. 즉, JPA 또한 실제로 동작하는 것이 아니고 어떻게 동작해야 하는지 메커니즘을 정리한 표준 명세로 생각하면 된다. 아래에서 JPA의 역할이 ORM이라고 보면 무난하다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180009351-be43ef36-7f88-4d70-ac17-e86a94e72b0b.png){: .align-center}

<br>

JPA의 메커니즘을 보면 내부적으로 JDBC를 사용한다. 개발자가 직접 JDBC를 구현하면 SQL에 의존하게 되는 문제 등이 있어 개발 효율성이 떨어지는데, JPAC는 이 같은 문제점을 보완해서 개발자 대신 적절한 SQL을 생성하고 데이터베이스를 조작해서 객체를 자동으로 매핑하는 역할을 수행한다.

JPA 기반의 구현체는 대표적으로 세 가지가 있다. 아래와 같이 하이버네이트(Hibernate), 이클립스 링크(EclipseLink), 데이터 뉴클리어스(DataNucleus)이며, 그중 가장 많이 사용하는는 구현체는 하이버네이트이다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180010492-727b810a-2775-46a9-af00-14a45e633285.png){: .align-center}

