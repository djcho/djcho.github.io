---
title : "[SpringBoot][Chapter7.6] 테스트 코드 작성하기 - 테스트 주도 개발(TDD)"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, TDD]
toc: true
toc_sticky : true
published : true
date : 2022-08-01
last_modified_at : 2022-08-01
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## 테스트 주도 개발(TDD)

TDD란 'Test-Driven Development' 의 줄임말로 '테스트 주도 개발'이라는 의미를 가지고 있다. 테스트 주도 개발은 반복 테스트를 이용한 소프트웨어 개발 방법론으로서 테스트 코드를 먼저 작성한 후 테스트를 통과하는 코드를 작성하는 과정을 반복하는 소프트웨어 개발 방식이다. 애자일 방법론 중 하나인 익스트림 프로그램(eXtream Programming)의 Test-First 개념에 기반을 둔, 개발 주기가 짧은 개발 프로세스로 단순한 설계를 중시한다.

<br>

> **💡 Tip. 애자일 소프트웨어 개발 방법론이란?**
>
> 애자일은 신속한 반복 작업을 통해 실제 작동 가능한 소프트웨어를 개발하는 개발 방식이다. 원래 애자일 방법론 자체는 일하는 방법에 대한 관점으로 소프트웨어 개발에만 국한되지는 않는다.
>
> 애자일 소프트웨어 개발 방법론의 핵심은 신속한 개발 프로세스를 통해 수시로 변하는 고객의 요구사항에 대응해서 제공하는 서비스의 가치를 극대화 하는 것이다.



### 테스트 주도 개발의 개발 주기

테스트 주도 개발의 개발 주기는 아래 그림과 같이 표현할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182035284-ec549558-6599-458e-b304-2e8cf7138395.png){: .align-center}

<br>

테스트 주도 개발에서는 위 그림과 같이 총 3개의 간계로 개발주기를 표현한다.



- 실패 테스트 작성(Write a failing test) : 실패하는 경우의 테스트 코드를 먼저 작성한다.
- 테스트를 통과하는 코드 작성(Make a test pass) : 테스트 코드를 성공시키기 위한 실제 코드를 작성한다.
- 리팩토링(Refactor) : 중복 코드를 제거하거나 일반화하는 리팩토링을 수행한다.



일반적인 개발 방법은 설계를 진행한 후 그에 맞게 애플리케이션 코드를 작성하고 마지막으로 테스트를 작성하는 흐름으로 진행된다. 반면 테스트 주고 개발에서는 설계 이후 바로 테스트 코드를 작성하고 애플리케이션 코드를 작성한다는 점에서 차이가 있다.



### 테스트 주도 개발의 효과

테스트 주도 개발에 따라 개발을 진행하면 다음과 같은 이점을 얻을 수 있다.



**디버깅 시간 단축**

- 테스트 코드 기반으로 개발이 진행되기 때문에 문제가 발생했을 때 어디에서 잘못됐는지 확인하기가 쉽다.

**생산성 향상**

- 테스트 코드를 통해 지속적으로 애플리케이션 코드의 불안정성에 대한 피드백을 받기 때문에 리팩토링 횟수가 줄고 생산성이 높아진다.

**재설계 시간 단축**

- 작성돼 있는 테스트 코드를 기반으로 코드를 작성하기 때문에 재설계가 필요할 경우 테스트 코드를 조정하는 것으로 재설ㄹ계 시간을 단축할 수 있다.

**기능 추가과 같은 추가 구현이 용이**

- 테스트 코드를 통해 의도한 기능을 미리 설계하고 코드를 작성하기 때문에 목적에 맞는 코드를 작성하는 데 비교적 용이하다.



이처럼 테스트 주도 개발은 다양한 장점을 가지고 있다. 그러나 아직 테스트 주도 개발 프로세스를 잘 지키며 개발하는 조직은 많지 않다. 기존 개발 방식에 익숙해져 있는 상황에서 모든 프로세스를 바꾸기가 쉽지 않기 때문이다. 테스트 주도 개발이라는 새로운 개발 방법에 적응하는 과정에서 발생하게 될 생산성 저하의 우려로 개발 조직이 쉽게 도전하기 어려운 것이 사실이다.



> 이번 장에서는 테스트의 개념과 이론을 살펴보고 테스트 코드를 작성하는 방법을 알아봤다. 테스트 코드의 중요성이 부각된 것은 비교적 최근의 일이며, 아직 많은 회사에서는 테스트 코드를 작성하는 것을 비용으로 생각하고 간과하기도 한다.
>
> 그러나 대부분의 개발 조직에서 테스트 코드의 중요성에 대해서는 인정하는 분위기이며, 필자 또한 테스트 코드를 잘 작성하는 것은 애플리케이션 개발에 매우 중요한 요소라고 생각한다.
>
> 이번 장에서 다룬 테스트 코드 작성법을 기반으로 테스트 코드를 많이 작성해보는 것을 권장한다. 여러 시나리오에 맞춰 다양한 테스트 코드를 작성하다 보면 다양한 예외 상황에 대처하는 순발력과 대처능력도 길러진다.
>
> JaCoCo를 활용한 테스트 커버리지 관리는 현업에서는 SonarQube  라는 솔루션과 함께 사용되는 경우가 많다. SonarQube는 코드 품질 관리 도구로서 많은 회사에서 채택해서 사용하고 있으므로 기회가 된다면 JaCoCo와 연동하는 것도 좋겠다.
