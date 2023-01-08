---
title : "[SpringBoot][Chapter8.1~2] Spring Data JPA 활용 - JPQL"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, JPQL]
toc: true
toc_sticky : true
published : true
date : 2022-08-01
last_modified_at : 2022-08-01
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



6장에서는 Spring Data JPA의 기본 기능을 살펴봤다. 또한 리포지토리를 활용해 데이터베이스에 접근하고 기본적인 CRUD 기능을 사용해 봤다.

이번 장에서는 Spring Data JPA에서 제공하는 기능들을 더 알아보고 다양한 활용법을 살펴보겠다. 그 과정에서 리포지토리 예제를 작성하고 리포지토리의 활용법을 테스트 코드를 통해 살펴보겠다.

<br>

> **💡 Tip.**
>
> Spring Data JPA의 자세한 내용은 공식 사이트 <a href="https://docs.spring.io/spring-data/jpa/docs/current/reference/html" target="_blank">https://docs.spring.io/spring-data/jpa/docs/current/reference/html</a> 에서 확인할 수 있다.



# 프로젝트 생성

이번 장에서는 내용에 집중하기 위해 새로운 프로젝트를 생성하겠다. 스프링 버전은 이전과 같은 2.5.6 버전으로 진행하며, 다음과 같은 내용을 설정한다.



- groupId : com.springboot
- artifactId : advanced_jpa
- name : advanced_jpa
- Developer Tools : Lombok, Spring Configuration Processor
- Web : Spring Web
- SQL : Spring Data JPA, MariaDB Driver



그리고 jpa를 다뤘던 6장에서 다음과 같이 자바 파일을 가져와 기본적인 프로젝트를 생성한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182074054-5300f05b-9711-4ba5-9d0e-bb2f87582c66.png){: .align-center}

<br>



# JPQL

JPQL은 JPA Query Language의 줄임말로 JPA 에서 사용할 수 있는 쿼리를 의미한다. JPQL의 문법은 SQL과 매우 비슷해서 데이터베이사 쿼리에 익숙한 분들이라면 어렵지 않게 사용할 수 있다. SQL과 차이점은 SQL에서는 테이블이나 칼럼의 이름을 사용하는 것과 달리 JPQL은 아래와 같이 엔티티 객체를 대상으로 수행하는 쿼리이기 때문에 매핑된 엔티티의 이름과 필드의 이름을 사용한다는 것이다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182063658-1af08745-5335-43f0-ae1b-917872ab2bdf.png){: .align-center}

