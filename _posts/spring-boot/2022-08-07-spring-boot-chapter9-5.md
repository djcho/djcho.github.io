---
title : "[SpringBoot][Chapter9.5] 연관관계 매핑 - 다대다 매핑"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, JPA]
toc: true
toc_sticky : true
published : true
date : 2022-08-07
last_modified_at : 2022-08-07
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## 다대다 매핑

다대다(M:N) 연관관계는 실무에서 거의 사용되지 않는 구성이다. 다대다 연관관계를 상품과 생산업체의 예로 들자면 한 종류의 상품이 여러 생산업체를 통해 생산될 수 있고, 생산업체 한 곳이 여러 상품을 생산할 수도 있다.

![image](https://user-images.githubusercontent.com/13410737/183258315-1ad034a6-9a71-46bc-8b9b-6fef105d373e.png){: .align-center}

다대다 연관관계에서는 각 엔티티에서 서로를 리스트로 가지는 구조가 만들어진다. 이런 경우에는 교차 엔티티라고 부르는 중간 테이블을 생성해서 다대다 관계를 일대다 또는 다대일 관계로 해소한다.



### 다대다 단방향 매핑
