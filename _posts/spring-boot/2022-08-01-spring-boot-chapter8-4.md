---
title : "[SpringBoot][Chapter8.4] Spring Data JPA 활용 - 정렬과 페이징 처리"
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





## 정렬과 페이징 처리

애플리케이션에서 자주 사용되는 정렬과 페이징 처리는 앞서 소개한 쿼리 메서드를 작성하는 방법을 기반으로 수행할 수 있다. 물론 기본 쿼리 메서드인 이름을 통한 정렬과 페이징 처리도 가능하지만 다른 방법들도 많이 쓰인다. 이번 장에서는 기본적으로 정렬과 페이징 처리 방법을 알아보겠다.



### 정렬 처리하기

일반적인 쿼리문에서 정렬을 사용할 때는 `ORDER BY` 구문을 사용한다. 쿼리 메서드도 정렬 기능에 동일한 키워드가 사용된다. 아래와 같이 작성하면 정렬 기능을 사용할 수 있다.
<br>

```java
// Asc : 오름차순, Desc : 내림차순
List<Product> findByNameOrderByNumberAsc(String name);
List<Product> findByNameOrderByNumberDesc(String name);
```

<br>

위와 같이 기본 쿼리 메서드를 작성한 후 `OrderBy` 키워드를 삽입하고 정렬하고자 하는 칼럼과 오름차순/내림차순을 설정하면 정렬이 수행된다. 2번 줄의 쿼리 메서드를 해석하면 '상품정보를 이름으로 겅색한 후 상품 번호로 오름차순 정렬을 수헹'한다는 뜻이다. 오름차순으로 정렬하려면 `Asc` 키워드를, 내림차순으로 정렬하려면 `Desc` 키워드를 사용한다.

2번 줄의 메서드를 호출했을 때 나오는 하이버네이트 로그는 아래와 같다.

<br>

```
Hibernate:
```



<br>

