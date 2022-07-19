---
title : "[SpringBoot][Chapter5.1] API 작성법- 프로젝트 설정"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij]
toc: true
toc_sticky : true
published : true
date : 2022-07-19
last_modified_at : 2022-07-19
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## 프로젝트 설정

이번 장부터 본격적인 애플리케이션 개발에 필요한 내용을 소개한다. 각 HTTP 메서드에 해당하는 API를 개발해보고 그 과정에서 필요한 내용들을 살펴본다. 아직 데이터베이스를 설치하지 않아 정확한 기능 구현은 어렵지만 외부의 요청을 받아 응답하는 기능을 구현해서 컨트롤러가 어떻게 구성되는지 알아보겠다.

먼저 실습할 프로젝트를 생성한다. 4장에서 소개한 방법과 동일하게 생성하면 된다. 다만 이번에는 groupId 는 'com.springboot' 로 설정하고 name 과 artifactId는 'api'로 설정한다.


