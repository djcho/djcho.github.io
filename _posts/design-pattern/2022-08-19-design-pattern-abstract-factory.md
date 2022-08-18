---
title : "[C++][DesignPattern] 생성 패턴 - 추상 팩토리"
categories:
- GofDesignPattern
tag :
- [C++, GoF, DesignPattern, AbstractFactory]
toc: true
toc_sticky : true
published : true
date : 2022-08-19
last_modified_at : 2022-08-19
---









에릭 감마, 리처드 헬름, 랄프 존슨, 존 블라시디스 지음, [GoF의 디자인패턴::재사용성을 지닌 객체지향 소프트웨어의 핵심요소] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





## 추상 팩토리

### 의도

상세화된 서브클래스를 정의하지 않고도 서로 관련성이 있거나 독립적인 여러 객체의 군을 생성하기 위한 인터페이스를 제공한다.



### 다른 이름

키트(Kit)



### 활용성

추상 팩토리는 다음의 경우에 사용한다.

- 객체가 생성되거나 구성/표현되는 방식과 무관하게 시스템을 독립적으로 만들고자 할 때
- 여러 제품군 중 하나를 선택해서 시스템을 설정해야 하고 한번 구성한 제품을 다른 것으로 대체할 수 있을 때
- 관련된 제품 객체들이 함께 사용되도록 설계되었고, 이 부분에 대한 제약이 외부에도 지켜지도록 하고 싶을 때
- 제품에 대한 클래스 라이브러리를 제공하고, 구현이 아닌 인터페이스를 노출시키고 싶을 때



### 구조

![image](https://user-images.githubusercontent.com/13410737/185437997-75c1f8e4-c87a-4a19-b918-513aebbb1a15.png){: .align-center}
