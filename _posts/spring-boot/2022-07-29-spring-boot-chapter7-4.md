---
title : "[SpringBoot][Chapter7.2] 테스트 코드 작성하기 - JUnit 테스트 코드 작성"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij]
toc: true
toc_sticky : true
published : true
date : 2022-07-29
last_modified_at : 2022-07-29
---





## JUnit을 활용한 테스트 코드 작성

<a href="https://junit.org/junit5/" target="_blank">JUnit</a>은 자바 언어에서 사용되는 대표적인 테스트 프레임워크로서 단위 테스트를 위한 도구를 제공한다. 또한 단위 테스트뿐만 아니라 통합 테스트를 할 수 있는 기능도 제공한다. JUnit의 가장 큰 특징은 어노테이션 기반의 테스트 방식을 지원한다는 것이다. 즉, JUnit을 사용하면 몇 개의 어노테이션만으로 간편하게 테스트 코드를 작성할 수 있따. 또한 JUnit을 활용하면 단정문(assert)을 통해 테스트 케이스의 기댓값이 정상적으로 도출됐는지 검토할 수 있따는 장점이 있다.

참고로 이 책에서 사용할 JUnit 5 버전은 스프링 부트 2.2 버전부터 사용 가능하며, 이 책에서 사용 중인 스프링 부트는 2.5.6버전이다.



### JUnit의 세부 모듈

JUnit