---
title : "[SpringBoot][Chapter6.8] DB연동 - 리포지토리 인터페이스 설계"
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



# 리포지토리 인터페이스 설계

Spring Data JPA는 `JpaRepository`를 기반으로 거욱 쉽게 데이터베이스를 사용할 수 있는 아키텍처를 제공한다. 스프링 부트로 `JpaRepository`를 상속하는 인터페이스를 생성하면 기존의 다양한 메서드를 손쉽게 활용할 수 있다.



## 리포지토리 인터페이스 생성

여기서 이야기하는 리포지토리(Repository)는 Spring Data JPA가 제공하는 인터페이스이다. 엔티티를 데이터베이스의 테이블과 주고를 생성하는 데 사용했다면 리포지토리는 엔티티가 생성한 데이터베이스에 접근하는 데 사용된다.

리포지토리를 생성하기 위해서는 접근하려는 테이블과 매핑되는 엔티티에 대한 인터페이스를 생성하고, 아래와 같이 `JpaRepository`를 상속받으면 된다.

<br>

```java
public interface ProductRepository extends JpaRepository<Product, Long>{

}
```

<br>

이때 리포지토리 인터페이스는 `data.repository` 패키지를 생성한 후 아래와 같이 해당 패키지에 생성하면 된다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180596173-f821da18-fdb9-474d-99d1-79dd493c6d51.png){: .align-center}

<br>

`ProductRepository`가 `JpaRepository`를 상속받을 때는 대상 엔티티와 기본값 타입을 지정해야 한다. 위 예제와 같이 대상 엔티티를 `Product`로 설정하고 해당 엔티티의 `@Id` 필드 타입인 `Long`을 설정하면 된다.

생성된 리포지토리는 아래와 같이 `JpaRepository`를 상속받으면서 별도의 메서드 구현 없이도 많은 기능을 제공한다.

<br>

```java
@NoRepositoryBean
public interface JpaRepository<T, ID> extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
    List<T> findAll();

    List<T> findAll(Sort sort);

    List<T> findAllById(Iterable<ID> ids);

    <S extends T> List<S> saveAll(Iterable<S> entities);

    void flush();

    <S extends T> S saveAndFlush(S entity);

    <S extends T> List<S> saveAllAndFlush(Iterable<S> entities);

    /** @deprecated */
    @Deprecated
    default void deleteInBatch(Iterable<T> entities) {
        this.deleteAllInBatch(entities);
    }

    void deleteAllInBatch(Iterable<T> entities);

    void deleteAllByIdInBatch(Iterable<ID> ids);

    void deleteAllInBatch();

    /** @deprecated */
    @Deprecated
    T getOne(ID id);

    T getById(ID id);

    <S extends T> List<S> findAll(Example<S> example);

    <S extends T> List<S> findAll(Example<S> example, Sort sort);
}
```

<br>

`JpaRepository`의 상속 구조를 보면 아래와 같다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180596387-ec45e569-6203-4d70-8054-709bcfc5bcd8.png){: .align-center}

<br>

위 그림에서 타 리포지토리를 상속받고 있는 리포지토리 인터페이스에는 위 코드와 같이 각 메서드가 구현돼 있다. 타 리포지토리에 만들어진 메서드는 모두 앞에서 생성한 `ProductRepository`에서도 사용할 수 있다.



## 리포지토리 메서드의 생성 규칙

리포지토리에서는 몇 가지 명명규칙에 따라 커스텀 메서드도 생성할 수 있다. 일반적으로 CRUD(Create, Read, Update, Delete) 에서 따로 생성해서 사용하는 메서드는 대부분 Read 부분에 해당하는 Select 쿼리밖에 없다. 엔티티를 저장하거나 갱신 또는 삭제할 때는 별도의 규칙이 필요하지 않기 때문이다. 다만 리포지토리에서 기본적으로 제공하는 조회 메서드는 기본값으로 단일 조회하거나 전체 엔티티를 조회하는 것만 지원하고 있기 때문에 필요에 따라 다른 조회 메서드가 필요하다.

메서드에 이름을 붙일 때는 첫 단어를 제외한 이후 단어들의 첫 글자를 대문자로 설정해야 JPA에서 정상적으로 인식하고 쿼리를 자동으로 만들어준다. 조회 메서드(find)에 조건을 붙일 수 있는 몇 가지 기능을 소개하면 다음과 같다.

- FindeBy : SQL문의 `where` 절 역할을 수행하는 구문이다. `findBy` 뒤에 에닡티의 필드값을 입력해서 사용한다.
  - 예) `findByName(String name)`
- AND, OR : 조건을 여러 개 설정하기 위해 사용한다.
  - 예) `findByNameAndEmail(String name, String email)`
- Like / NotLike : SQL문의 `like`와 동일한 기능을 수행하며, 특정 문자를 포함하는지 여부를 조건으로 추가한다. 비슷한 키워드로 `Containing`, `Contains`, `isContaing`이 있다.
- StartsWith / StartingWith : 특정 키워드로 시작하는 문자열 조건을 설정한다.
- EndsWith / EndingWith : 특정 키워드로 끝나는 문자열 조건을 설정한다.
- IsNull / IsNotNull : 레코드 값이 Null 이거나 Null이 나닌 값을 검색한다.
- True / False : Boolean 타입의 레코드를 검색할 때 사용한다.
- Before / After : 시간을 기준으로 값을 검색한다.
- LessThan / GreaterThan : 특정 값(숫자)을 기준으로 대소 비교를 할 때 사용한다.
- Between : 두 값(숫자) 사이의 데이터를 조회한다.
- OrdeBy : SQL 문에서 `order by`와 동일한 기능을 수행한다.
  -  예) 가격순으로 이름 조회를 수행한다면 `List<Product> findByNameOrderByPriceAsc(String name);`와 같이 작성한다.
- countBy : SQL 문의 `count`와 동일한 기능을 수행하며, 결괏값의 개수(count)를 추출한다.



자세한 쿼리 메서드는 7장에서 다룬다.
