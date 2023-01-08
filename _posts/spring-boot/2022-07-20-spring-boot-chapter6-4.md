---
title : "[SpringBoot][Chapter6.4] DB연동 - 하이버네이트"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Hibernate]
toc: true
toc_sticky : true
published : true
date : 2022-07-20
last_modified_at : 2022-07-20
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



# 하이버네이트

<a href="https://hibernate.org" target="_blank">하이버네이트</a>는 자바의 ORM 프레임워크로, JPA가 정의하는 인터페이스를 구현하고 있는 JPA 구현체 중 하나이다. 이 책은 하이버네이트의 기능을 더욱 편하게 사용하도록 모듈화된 Spring Data JPA를 활용하기 때문에 JPA 자체를 직접 사용할 일은 거의 없다. 그러나 앞으로 사용할 기능의 개념에 대해서는 한번 살펴보겠다.

<br>

## Spring Data JPA

Spring Data JPA는 JPA를 편리하게 사용할 수 있도록 지원하는 스프링 하위 프로젝트 중 하나이다.  Spring Data JPA는 CRUD 처리에 필요한 인터페이스를 제공하며 하이버네이트의 엔티티 매니저(EntityManager)를 직접 다루지 않고 리포지토리를 정의해 사용함으로써 스프링이 적합한 쿼리를 동적으로 생성하는 방식으로 데이터베이스를 조작한다. 이를 통해 하이버네이트에서 자주 사용되는 기능을 더 쉽게 사용할 수 있게 구현한 라이브러이이다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180012513-df18dfce-fdce-46ff-aba8-1cb7fb9e4e79.png){: .align-center}
