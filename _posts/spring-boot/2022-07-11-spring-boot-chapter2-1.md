---
title : "[SpringBoot][Chapter2.1] 기초 지식 - 서버 간 통신"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java]
toc: true
toc_sticky : true
published : true
date : 2022-07-11
last_modified_at : 2022-07-11
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



# 서버 간 통신

어떤 포털 사이트를 하나의 서비스 단위로 개발한다고 가정해 보자. 블로그, 카페 메일 등의 기능들을 하나의 애플리케이션으로 통합했다고 가정한다면 서버를 업데이트하거나 애플리케이션을 유지 보수할 때마다 '사이트 작업 중'이라는 팻말을 걸고 작업해야 할 것이다. 그만큼 개발에 보수적인 입장을 취할 수 밖에 없고, 서비스 자체의 규모도 커지기 때문에 서비스를 구동하는데 걸리는 시간도 길어진다.

이 같은 문제를 해결하기 위해 고안된 것이 마이크로서비스 아키텍쳐(MSA; Microservice Architecture)이다. 마이크로서비스 아키텍처는 단어 그대로 서비스 규모를 작게 나누어 구성한 아키텍처이다. 앞에서 예로 든 포털 사이트에 마이크로서비스 아키텍처를 적용한다면 애플리케이션 하나에 여러 기능을 넣어 개발하지 않고 블로그 프로젝트, 카페 프로젝트, 메일 프로젝트 등 애플리케이션을 기능 별로 나눠서 개발하게 된다.

단일 서비스로 구성된 포털 사이트는 내부 메서드 호출 등을 통해 원하는 자원을 가져와 사용할 수 있지만 서비스 기능 별로 구분해서 독립적인 애플리케이션을 개발하게 되면 각 서비스 간에 통신해야 하는 경우가 발생한다. 이런 상황에서의 통신을 '서버 간 통신' 이라고 한다.

서버 간 통신은 한 서버가 다른 서버에 통신을 요청하는 것을 의미하며, 한 대는 서버, 다른 한 대는 클라이언트가 되는 구조이다. 몇 가지 프로토콜에 의해 다양한 통신 방식을 적용할 수 있지만 가장 많이 사용되는 방식은 HTTP/HTTPS 방식이다.
