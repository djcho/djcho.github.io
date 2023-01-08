---
title : "[C++][DesignPattern] 생성 패턴 - 빌더"
categories:
- GofDesignPattern
tag :
- [C++, GoF, DesignPattern, Builder]
toc: true
toc_sticky : true
published : true
date : 2022-10-11
last_modified_at : 2022-10-11
---



에릭 감마, 리처드 헬름, 랄프 존슨, 존 블라시디스 지음, [GoF의 디자인패턴::재사용성을 지닌 객체지향 소프트웨어의 핵심요소] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





# 빌더

## 의도

복잡한 객체를 생성하는 방법과 표현하는 방법을 별도로 분리하여 서로 다른 표현이라도 동일한 절차로 생성할 수 있도록 제공한다.



## 활용성

빌더 패턴은 다음의 경우에 사용한다.
