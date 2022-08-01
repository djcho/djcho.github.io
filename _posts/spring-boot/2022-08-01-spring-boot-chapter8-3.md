---
title : "[SpringBoot][Chapter8.3] Spring Data JPA 활용 - 쿼리 메서드 살펴보기"
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





## 쿼리 메서드 살펴보기

리포지토리는 `JpaRepository`를 상속받는 것만으로도 다양한 CRUD 메서드를 제공한다. 하지만 이러한 기본 메서드들은 식별자 기반으로 생성되기 때문에 결국 별도의 메서드를 정의해서 사용하는 경우가 많다. 이때 간단한 쿼리문을 작성하기 위해 사용되는 것이 쿼리 메서드이다.



### 쿼리 메서드의 생성

쿼리 메서드는 크게 동작을 결정하는 주제(Subject)와 서술어(Predicate)로 구분한다. 'find ... By', 'exists ... By'와 같은 키워드로 쿼리의 주제를 정하며 'By'는 서술어의 시작을 나타내는 구분자 역할을 한다. 서술어 부분은 검색 및 정렬 조건을 지정하는 영역이다. 기본적으로 엔티티의 속성으로 정의할 수 있고, AND나 OR를 사용해 조건을 확장하는 것도 가능하다.



> **리포지토리의 쿼리 메서드 생성 예**
>
> // (리턴타입) + {주제 + 서술어(속성)} 구조의 메서드
>
> `List<Person> findByLastnameAndEmail(String lastName, String email);`



서술어에 들어가는 엔티티의 속성 식(Expression)은 위의 예시와 같이 엔티티에서 관리하고 있는 속서(필드)만 참조할 수 있다.



### 쿼리 메서드의 주제 키워드

쿼리 메서드의 주제 부분에 사용할 수 있는 주요 키워드는 다음과 같다.

- `find...By`
- `read...By`
- `get...By`
- `query...By`
- `search...By`
- `stream...By`

보다시피 조회하는 기능을 수행하는 키워드이다. '...' 으로 표시한 영역에는 도메인(엔티티)을 표현할 수 있다. 그러나 리포지토리에서 이미 도메인을 설정한 후에 메서드를 사용하기 때문에 중복으로 판단해 생략하기도 한다. 리턴 타입으로는 `Collection` 이나 `Stram`에 속한 하위 타입을 설정할 수 있다. 아래와 같이 `ProductRepository`에쿼리 메서드를 작성한다.

<br>

```java
// find...By
Optional<Product> findByNumber(Long number);
List<Product> findAllByName(String name);
Product queryByNumber(Long number);
```

<br>



#### exists...By

특정 데이터가 존재하는지 확인하는 키워드이다. 리턴 타입으로는 `boolean` 타입을 사용한다.

<br>

```java
// exists...By
boolean existsByNumber(Long number);
```

<br>



#### count...By

조회 쿼리를 수행한 후 쿼리 결과로 나온 레코드의 개수를 리턴한다.

<br>

```java
// count..By
long countByName(String name);
```

<br>



#### delete...By, remove...By

삭제 쿼리를 수행한다. 리턴 타입이 없거나 삭제한 횟수를 리턴한다.

<br>

```java
// delete...By, remove...By
void deleteByNumber(Long number);
long removeByName(String name);
```

<br>



#### ...First\<number>..., ...Top\<number>...

쿼리를 통해 조회된 결괏값의 개수를 제한하는 키워드이다. 두 키워드는 동일한 동작을 수행하며, 주제와 By 사이에 위치한다. 일반적으로 한 번의 동작으로 여러 건을 조회할 때 사용되며, 단 건으로 조회하기 위해서는 `<number>`를 생략하면 된다.

<br>

```java
// ...First<number>..., Top<Number>...
List<Product> findFirst5ByName(String name);
List<Product> findTop10ByName(String name);
```

<br>



### 쿼리 메서드의 조건자 키워드

JPQL의 서술어 부분에서 사용할 수 있는 몇 가지 조건자 키워드를 소개한다.

<br>



#### Is

값의 일치를 조건으로 사용하는 조건자 키워드이다. 생략되는 경우가 많으며 `Equals`와 동일한 기능을 수행한다.

<br>

```java
// findByNumber 메서드와 동일하게 동작
Product findByNumberIs(Long number);
Product findByNumberEquals(Long number);
```

<br>



#### (Is)Not

값의 불일치를 조건으로 사용하는 조건자 키워드이다. `Is` 는 생략하고 `Not`키워드만 사용할 수도 있다.

<br>

```java
Product findByNumberIsNot(Long number);
Product findByNumberNot(Long number);
```

<br>



#### (Is)Null, (Is)NotNull

값이 null인지 검사하는 조건자 키워드이다.

<br>

```java
List<Product> findByUpdatedAtNull();
List<Product> findByUpdatedAtIsNull();
List<Product> findByUpdatedAtNotNull();
List<Product> findByUpdatedAtIsNotNull();
```

<br>



#### And, Or

여러 조건을 묶을 때 사용한다.

<br>

```java
Product findByNumberAndName(Long number, String name);
Product findByNumberOrName(Long number, String name);
```

<br>



#### (Is)GreaterThan, (Is)LessThan, (Is)Between

숫자나 `datetime` 칼럼을 대상으로 한 비교 연산에 사용할 수 있는 조건자 키워드이다. `GreaterThan`, `LessThan` 키워드는 비교 대상에 대한 초과/미만의 개념으로 비교 연산을 수행하고, 경곗값을 포함하려면 `Equal` 키워드를 추가하면 된다.

<br>

```java
List<Product> findByPriceIsGreaterThan(Long price);
List<Product> findByPriceGreaterThan(Long price);
List<Product> findByPriceGreaterThanEqual(Long price);
List<Product> findByPriceIsLessThan(Long price);
List<Product> findByPriceLessThan(Long price);
List<Product> findByPriceLessThanEqual(Long price);
List<Product> findByPriceIsBetween(Long lowPrice, Long highPrice);
List<Product> findByPriceBetween(Long lowPrice, Long highPrice);
```

<br>



#### (Is)StartingWith(==StartsWith), (Is)EndingWith(==EndsWith), (Is)Containing(==Contains), (Is)Like

칼럼값에서 일부 일치 여부를 확인하는 조건자 키워드이다. SQL 쿼리문에서 값의 일부를 포함하는 값을 추출할 때 사용하는 '%' 키워드와 동일한 역할을 하는 키워드이다. 자동으로 생성되는 SQL문을 보면 `Containing` 키워드는 문자열의 양 끝, `StartingWith` 키워드는 문자열의 앞, `EndingWith` 키워드는 문자열의 끝에 '%'가 배치된다. 여기서 별도로 고려해야 하는 키워드는 `Like` 키워드인데, 이 키워드는 코드 수준에서 메서드를 호출하면서 전달하는 값에 %를 명시적으로 입력해야 한다.

<br>

```java
List<Product> findByNameContains(String name);
List<Product> findByNameContaining(String name);
List<Product> findByNameIsContaining(String name);

List<Product> findByNameStartsWith(String name);
List<Product> findByNameStartingWith(String name);
List<Product> findByNameIsStartingWith(String name);

List<Product> findByNameEndsWith(String name);
List<Product> findByNameEndingWith(String name);
List<Product> findByNameIsEndingWith(String name);
```

<br>
