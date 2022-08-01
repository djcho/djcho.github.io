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
    order by
        product0_.number desc
```

<br>

13번 줄에 `order by` 구문이 포함돼 있고 메서드에 이름이 나와 있는 것처럼 `Number`에 대해 오름차순으로 정렬하고 있다.

다른 쿼리 메서드들은 조건 구문에서 조건을 여러 개 사용하기 위해 `And`와 `Or` 키워드를 사용했다. 하지만 정렬 구문은 `And`나 `Or` 키워드를 사용하지 않고 아래와 같이 우선순위를 기준으로 차례대로 작성하면 된다.

<br>

```java
// And룰 붙이지 않음
List<Product> findByNameOrderByPriceAscStockDesc(String name);
```

<br>

2번 줄의 메서드는 먼저 `Price`를 기준으로 오름차순 정렬한 후 후순위로 재고수량을 기준으로 내림차순 정렬을 수행한다. 이렇게 작성한 메서드가 호출되면 하이버네이트에서는 다음과 같이 쿼리를 작성한다.

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
    order by
        product0_.price asc,
        product0_.stock desc
```

<br>

이렇게 쿼리 메서드의 이름에 정렬 키워드를 삽입해서 정렬을 수행하는 것도 가능하지만 메서드의 이름이 길어질수록 가독성이 떨어지는 문제가 생긴다. 이를 해결하기 위해 아래와 같이 매개변수를 활용해 정렬할 수도 있다.

<br>

```java
List<Product> findByName(String name, Sort sort);
```

<br>

위 코드는 앞서 소개한 정렬 키워드가 들어간 메서드와 거의 동일한 기능을 수행한다. 다만 이 메서드는 이름에 키워드를 넣지 않고 `Sort` 객체를 활용해 매개변수로 받아들인 정렬 기준을 가지고 쿼리문을 작성하게 된다. 위 코드의 `Sort` 객체를 테스트해보기 위해 `test/com.springboot.advanced_jpa` 패키지내에 `data/repository` 패키지를 생성한 후 `ProductRepositoryTest`를 생성해서 다음과 같이 작성한다.

<br>

```java
package com.springboot.advanced_jpa.data.repository;


import com.springboot.advanced_jpa.data.entity.Product;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.data.domain.Sort;
import org.springframework.data.domain.Sort.Order;

import java.time.LocalDateTime;

@SpringBootTest
public class ProductRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Test
    void sortingAndPagingTest() {
        Product product1 = new Product();
        product1.setName("펜");
        product1.setPrice(1000);
        product1.setStock(100);
        product1.setCreatedAt(LocalDateTime.now());
        product1.setUpdatedAt(LocalDateTime.now());

        Product product2 = new Product();
        product2.setName("펜");
        product2.setPrice(5000);
        product2.setStock(300);
        product2.setCreatedAt(LocalDateTime.now());
        product2.setUpdatedAt(LocalDateTime.now());

        Product product3 = new Product();
        product3.setName("펜");
        product3.setPrice(500);
        product3.setStock(50);
        product3.setCreatedAt(LocalDateTime.now());
        product3.setUpdatedAt(LocalDateTime.now());

        Product savedProduct1 = productRepository.save(product1);
        Product savedProduct2 = productRepository.save(product2);
        Product savedProduct3 = productRepository.save(product3);
        
        productRepository.findByName("펜", Sort.by(Order.asc("price")));
        productRepository.findByName("펜", Sort.by(Order.asc("price"), Order.desc("stock")));
    }
}
```

<br>

그리고 20번 줄의 sortingAndPagingTest() 메서드 하단에 아래와 같이 작성해서 메서드에 전달할 수 있다.

<br>

```java
productRepository.findByName("펜", Sort.by(Order.asc("price")));
productRepository.findByName("펜", Sort.by(Order.asc("price"), Order.desc("stock")));
```

<br>

위 예제를 포함하여 앞으로 나올 테스트 코드의 결괏값을 확인하고 싶다면 `System.out.println()` 을 붙여주면 된다. `Sort` 클래스는 내부 클래스로 정의돼 있는 `Order` 객체를 활용해 정렬 기준을 생성한다. `Order` 객체에는 `asc`와 `desc` 메서드가 포함돼 있어 이 메서드를 통해 오름차순/내림차순을 지정하며, 여러 정렬 기준을 사용할 경우에는 2번 줄처럼 콤마(,)를 사용해 구분한다. 이렇게 작성된 두 개의 메서드를 실행하면 하이버네이트에서는 다음과 같은 쿼리를 생성한다.

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
    order by
        product0_.price asc
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
    order by
        product0_.price asc,
        product0_.stock desc
```

<br>

매개변수를 활용한 쿼리 메서드를 사용하면 쿼리 메서드를 정의하는 단계에서 코드가 줄어드는 장점이 있다. 그러나 호출하는 위치에서는 여전히 정렬 기준이 길어져 가독성이 떨어진다. 해당 코드는 정렬 기준을 설정하기 위한 필수적인 구문이기 때문에 코드의 양을 줄이기는 어렵다. 하지만 아래와 같이 `Sort` 부분을 하나의 메서드로 분리해서 쿼리 메서드를 호출하는 코드를 작성하는 방법도 가능하다.

<br>

```java
@SpringBootTest
public class ProductRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Test
    void sortingAndPagingTest() {
        ..상단 코드 생략..
            
        productRepository.findByName("펜", getSort());
    }
    
    private Sort getSort() {
        return Sort.by(
                Order.asc("price"),
                Order.desc("stock")
        );
    }
}
```

<br>



### 페이징 처리

페이징이란 데이터베이스의 레코드를 개수로 나눠 페이지를 구분하는 것을 의미한다. 예를 들면, 25개의 레코드가 있다면 레코드를 7개씩, 총 4개의 페이지로 구분하고 그중에서 특정 페이지를 가져오는 것이다. 흔히 볼 수 있는 웹 페이지에서 각 페이지를 구분해서 데이터를 제공할 때 그에 맞게 데이터를 요청하는 것이라고 생각하면 된다.

JPA에서는 이 같은 페이징 처리를 위해 `Page` 와 `Pageable`을 사용한다. 아래와 같이 페이징 처리가 가능한 쿼리 메서드를 작성할 수 있다.

<br>

```java
Page<Product> findByName(String name, Pageable pageable);
```

<br>

위와 같이 리턴 타입으로 `Page`를 설정하고 매개변수에는 `Pageable` 타일의 객체를 정의한다. 해당 메서드를 사용하기 위해서는 아래와 같이 호출한다.

<br>

```java
Page<Product> productPage = productRepository.findByName("펜", PageRequest.of(0, 2));
```

<br>

위 코드에서는 메서드를 호출할 때 리턴 타입으로 `Page` 객체를 받아야 하기 때문에 `Page<Product>`로 타입을 선언해서 객체를 리턴받았다. 그리고 `Pageable` 파라미터를 전달하기 위해 `PageRequest` 클래스는 사용했다. `PageRequest`는 `Pageable`의 구현체이다.

일반적으로 `PageRequest`는 `of` 메서드를 통해 `PageRequest` 객체를 생성한다. `of` 메서드는 매개변수에 따라 다양한 형태로 오버로딩돼 있는데 다음과 같은 매개변수 조합을 지원한다.

<br>

| of 메서드                                                  | 매개변수 설명                                            | 비고                                         |
| ---------------------------------------------------------- | -------------------------------------------------------- | -------------------------------------------- |
| `of(int page, int size)`                                   | 페이지 번호(0부터 시작), 페이지당 데이터 개수            | 데이터를 정렬하지 않음                       |
| `of(int page, int size, Sort)`                             | 페이지 번호, 페이지당 데이터 개수, 정렬                  | `sort`에 의해 정렬                           |
| `of(int page, int size, Direction, String ... properties)` | 페이지 번호, 페이지당 데이터 개수, 정렬 방향, 속성(칼럼) | `Sort.by(direction, properties)`에 의해 정렬 |

<br>

위 메서드가 호출될 때 하이버네이트에서 생성하는 쿼리는 다음과 같다.

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
        product0_.name=? limit ?
Hibernate: 
    select
        count(product0_.number) as col_0_0_ 
    from
        product product0_ 
    where
        product0_.name=?
```

<br>

쿼리 로그를 보면 `select` 쿼리에 `limit` 쿼리가 포함돼 있는 것을 볼 수 있다. 만약 페이지 번호를 0이 아닌 1 이상의 숫자로 설정하면 `offset` 키워드도 포함되어 레코드 목록을 구분해서 가져오게 된다. 이렇게 리턴받은 객체를 출력하면 다음과 같은 출력 결과를 확인할 수 있다.

<br>

```
Page 1 of 2 containing com.springboot.advanced_jpa.data.entity.Product instances
```

<br>

`Page`객체를 그대로 출력하면 해당 객체의 값을 보여주지 않고 위와 같이 몇 번째 페이지에 해당하는지만 확인할 수 있다. 각 페이지를 구성하는 세부적인 값을 보려면 아래와 같이 작성한다.

<br>

```java
Page<Product> productPage = productRepository.findByName("펜", PageRequest.of(0, 2));
System.out.println(productPage.getContent());
```

<br>

2번 줄과 같이 `getContent()` 메서드를 사용해 출력하면 배열 형태로 값이 출력된다.

