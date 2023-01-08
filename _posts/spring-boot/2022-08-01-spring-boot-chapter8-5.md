---
title : "[SpringBoot][Chapter8.5] Spring Data JPA 활용 - @Query 사용하기"
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





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



# @Query 어노테이션 사용하기

데이터베이스에서 값을 가져올 때는 앞 절에서 소개한 것처럼 메서드의 이름만으로 쿼리 메서드를 생성할 수도 있고 이번 절에서 살펴볼 `@Query` 어노테이션을 사용해 직업 JPQL을 작성할 수도 있다.

JPQL을 사용하면 JPA 구현체에서 자동으로 쿼리 문장을 해석하고 실행하게 된다. 만약 데이터베이스를 다른 데이터베이스로 변경할 일이 없다면 직접 해당 데이터베이스에 특화된 SQL을 작성할 수 있으며, 주로 튜닝된 쿼리를 사용하고자할 때 직접 SQL을 작성한다. 이 책에서는 JPQL을 직접 다루는 방법을 알아볼 텐데, 먼저 기본적인 JPQL을 사용해 상품정보를 조회하는 메서드를 리포지토리에 추가한다.

<br>

```java
@Query("SELECT p FROM Product AS p WHERE p.name = ?1")
List<Product> findByName(String name);
```

<br>

1번 줄과 같이 `@Query` 어노테이션을 사용해 JPQL 형식의 쿼리문을 작성한다. `FROM` 뒤에서 엔티티 타입을 지정하고 별칭을 생성한다.(`AS`는 생략 가능하다). `WHERE`문에서는 SQL과 마찬가지로 조선을 지정한다. 조건문에서 사용한 `'?1'`은 파라미터를 전달받기 위한 인자에 해당한다. 1은 첫 번째 파라미터를 의미한다. 하지만 이 같은 방식을 사용할 경우 파라미터의 순서가 바뀌면 오류가 발생할 가능성이 있어 `@Param` 어노테이션을 사용하는 것이 좋다.

<br>

```java
@Query("SELECT p FROM Product p WHERE p.name = :name")
List<Product> findByNameParam(@Param("name") String name);
```

<br>

보다시피 파라미터를 바인딩하는 방식으로 메서드를 구현하면 코드의 가독성이 높아지고 유지보수가 수월해진다. 앞에서 살펴본 두 예제는 하이버네이트에서 다음과 같이 동일한 쿼리를 생성해서 실행한다.

<br>

```
Hibernate: 
    select
        product0_.number as number1_0_,
        product0_.created_at as created_2_0_,
        product0_.name as name3_0_,
        product0_.price as price4_0_,
        product0_.stock as stock5_0_,
        product0_.updated_at as updated_6_0_ 
    from
        product product0_ 
    where
        product0_.name=?
```

<br>

그리고 `@Query`를 사용하면 엔티티 타입이 아니라 원하는 칼럼의 값만 추출할 수 있다.

<br>

```java
@Query("SELECT p.name, p.price, p.stock FROM Product p WHERE p.name = :name")
List<Object[]> findByNameParam2(@Param("name") String name);
```

<br>

이처럼 `SELECT`에 가져오고자 하는 칼럼을 지정하면 된다. 이때 메서드에서는 `Object` 배열의 리스트 형태로 리턴 타입을 지정해야 한다. 위 메서드를 호출했을 때 생성되는 쿼리는 다음과 같다.

<br>

```
Hibernate: 
    select
        product0_.name as col_0_0_,
        product0_.price as col_1_0_,
        product0_.stock as col_2_0_ 
    from
        product product0_ 
    where
        product0_.name=?
```

