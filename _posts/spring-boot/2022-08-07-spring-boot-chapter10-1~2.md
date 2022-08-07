---
title : "[SpringBoot][Chapter10.1~2] 유효성 검사와 예외처리 - 개요"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Validator]
toc: true
toc_sticky : true
published : true
date : 2022-08-07
last_modified_at : 2022-08-07
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





애플리케이션의 비즈니스 로직이 올바르게 동작하려면 데이터를 사전 검증하는 작업이 필요하다. 이것을 유효성 검사 또는 데이터 검증이라 부른다. 유효성 검사의 예로는 여러 계층에서 들어오는 데이터에 대해 의도한 형식대로 값이 들어오는지 체크하는 과정이 있다. 이 같은 유효성 검사(validation)는 프로그래밍에서 매우 중요한 부분이며, 자바에서 가장 신경 써야 하는 것 중 하나로 `NullPointException`예외가 있다.



## 일반적인 애플리케이션 유효성 검사의 문제점

일반적으로 사용되는 데이터 검증 로직에는 몇 가지 문제점이 있다. 계층별로 진행하는 유효성 검사는 검증 로직이 각 클래스별로 분산돼 있어 관리하기가 어렵다. 그리고 검증 로직에 의외로 중복이 많아 여러 곳에 유사한 기능의 코드가 존재할 수 있다. 마지막으로 검증해야할 값이 많다면 검증하는 코드가 길어진다. 이러한 문제로 코드가 복잡해지고 가독성이 떨어진다.

이 같은 문제를 해결하기 위해 자바 진영에서는 2009년 부터 Bean Validation이라는 데이터 유효성 검사 프레임워크를 제공한다. Bean Validation은 어노테이션을 통해 다양한 데이터를 검증하는 기능을 제공한다. Bean Validation을 사용한다는 것은 유효성 검사를 위한 로직을 DTO 같은 도메인 모델과 묶어서 각 계층에서 사용하면서 검증 자체를 도메인 모델에 얹는 방식으로 수행한다는 의미이다.

또한 Bean Validation은 어노테이션을 사용한 검증 방식이기 때문에 코드의 간결함도 유지할 수 있다.



## Hibernate Validator

<a href="https://hibernate.org/validator/" target="_blank">Hibernate Validator</a>는 Bean Validation 명세의 구현체이다. 스프링 부트에서는 Hibernate Validator를 유효성 검사 표준으로 채택해서 사용하고 있다. Hibernate Validator는 JSR-303 명세의 구현체로서 도메인 모델에서 어노테이션을 통한 필드값 검증을 가능하게 도와준다.
