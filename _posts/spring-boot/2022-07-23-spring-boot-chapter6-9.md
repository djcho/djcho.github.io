---
title : "[SpringBoot][Chapter6.9] DB연동 - DAO 설계"
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



## DAO 설계

DAO(Data Access Object)는 데이터베이스에 접근하기 위한 로직을 관리하기 위한 객체이다. 비즈니스 로직의 동작 과정에서 데이터를 조작하는 기능은 DAO 객체가 수행한다. 다만 스프링 데이터 JPA에서 DAO의 개념은 리포지토리가 대체하고 있다.

규모가 작은 서비스에서는 DAO를 별도로 설계하지 않고 바로 서비스 레이어에서 데이터베이스에 접근해서 구현하기도 하지만, 이번 장에서는 DAO를 서비스 레이어와 리포지토리의 중간 계층을 구성하는 역할로 사용할 예정이다. 이 책에서는 간단한 데이터베이스 호출만 다루고 있기 때문에 큰 의미는 없지만 실제로 업무에 필요한 비즈니스 로직을 개발하다 보면 데이터를 다루는 중간 계층을 두는 것이 유지보수 측면에서 용이한 경우가 많다. 물론 서비스 레이어에서 리포지토리의 메서드를 호출하고 그 결과에 대해 처리할 수 있지만 비즈니스 로직을 수행하는 과정에서 데이터베이스에 관한 작업을 처리하는 것은 기능을 분리하고 관리하기에 좋은 코드라고 보기 어렵다.

객체지향적인 설계에서는 서비스와 비즈니스 레이러를 분리해서 서비스 레이어에서는 서비스 로직을 수행하고 비즈니스 레이어에서는 비즈니스 로직을 수행해야 한다는 의견도 많다. 그러나 이번 장에서는 이런 관점은 간단하게만 다루고 서비스 객체가 비즈니스 로직까지 포함하는 방향으로 진행하겠다. 도메인(엔티티) 객체를 중심으로 다뤄지는 로직은 비즈니스 로직으로 볼 수 있다.

> **DAO vs. 리포지토리**
>
> DAO와 리포지토리는 역할이 비슷하다. 그렇기 때문에 아직도 DAO와 리포지토리를 비교하거나 어떤 차이가 있는지 논쟁하는 경우가 많다. 실제로 리포지토리는 Spring Data JPA에서 제공하는 기능이기 때문에 기존의 스프링 프레임워크나 스프링 MVC의 사용자는 리포지토리라는 개념을 사용하지 않고 DAO 객체로 데이터베이스에 접근했다. 이런 측면에서 각 컴포넌트의 역할을 고민하는 시간을 가져보면 좋을 것 같다.


