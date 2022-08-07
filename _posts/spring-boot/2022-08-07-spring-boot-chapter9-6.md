---
title : "[SpringBoot][Chapter9.6] 연관관계 매핑 - 영속성 전이"
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



## 영속성 전이

영속성 전이(cascade)란 특정 엔티티의 영속성 상태를 변경할 때 그 엔티티와 연관된 엔티티의 영속성에도 영향을 미쳐 영속성 상태를 변경하는 것을 의미한다. 예를 들어 `@OneToMany`어노테이션의 인터페이스를 살펴보면 아래와 같다.

<br>

```java
public @interface OneToMany {
    Class targetEntity() default void.class;

    CascadeType[] cascade() default {};

    FetchType fetch() default FetchType.LAZY;

    String mappedBy() default "";

    boolean orphanRemoval() default false;
}
```

<br>

연관관계와 관련된 어노테이션을 보면 위와 같이 `cascade()`라는 요소를 볼 수 있다. 이 어노테이션은 영속성 전이를 설정하는 데 활용된다. `cascade()` 요소와 함께 사용하는 영속성 전이 타입은 아래와 같다.

<br>

| 종류    | 설명                                                       |
| ------- | ---------------------------------------------------------- |
| ALL     | 모든 영속 상태 변경에 대한 영속성 전이를 적용              |
| PERSIST | 엔티티가 영속화할 때 연관된 엔티티도 함께 영속화           |
| MERGE   | 엔티티를 영속성 컨텍스트에 병합할 때 연관된 엔티티도 병합  |
| REMOVE  | 엔티티를 제거할 때 연관된 엔티티도 제거                    |
| REFRESH | 엔티티를 새로고침할 때 연관된 엔티티도 새로고침            |
| DETACH  | 엔티티를 영속성 컨첵스트에서 제외하면 연관된 엔티티도 제외 |

<br>

여기서 알 수 있듯이 영속성 전이에 사용되는 타입은 엔티티 생명주기와 연관이 있다. 한 엔티티가 위와 같이 `cascade`요소 값으로 주어진 영속 상태의 변경이 일어나면 매핑으로 연관된 엔티티에도 동일한 동작이 일어나도록 전이를 발생시키는 것이다. 위 코드를 보면 `cascade()`요소의 리턴 타입은 배열 형식인 것을 볼 수 있다. 이 말은 개발자가 사용하고자 하는 `cascade` 타입을 골라 각 상황에 적용할 수 있다는 것이다.



### 영속성 전이 적용

이제 영속성 전이를 적용해 보겠다. 여기서 사용할 엔티티는 상품 엔티티와 공급업체 엔티티이다. 예를 들어, 한 가게가 새로운 공급업체와 계약하며 몇 가지 새 상품을 입고시키는 상황에 어떻게 영속성 전이가 적용되는지 살펴보겠다. 우선 엔티티를 데이터베이스에 추가하는 경우로 영속성 전이 타입으로 `PERSIST`를 지정하겠다. 먼저 공급업체 엔티티에 아래의 16번 줄과 같이 영속성 전이 타입을 설정한다.

<br>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "provider")
public class Provider extends BaseEntity{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "provider", fetch = FetchType.EAGER, cascade = CascadeType.PERSIST)
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();

}
```

<br>

영속성 전이 타입을 설정하기 위해서는 `@OneToMany` 어노테이션의 속성을 활용한다. 이렇게 설정한 후에 아래와 같이 코드를 작성한다.

<br>

```java
@Test
void cascadeTest(){
    Provider provider = savedProvider("새로운 공급업체");

    Product product1 = savedProduct("상품1", 1000, 1000);
    Product product2 = savedProduct("상품2", 500, 1500);
    Product product3 = savedProduct("상품3", 750, 500);

    // 연관 관계 설정
    product1.setProvider(provider);
    product2.setProvider(provider);
    product3.setProvider(provider);

    provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

    providerRepository.save(provider);
}

private Provider savedProvider(String name) {
    Provider provider = new Provider();
    provider.setName(name);

    return provider;
}

private Product savedProduct(String name, Integer price, Integer stock) {
    Product product = new Product();
    product.setName(name);
    product.setPrice(price);
    product.setStock(stock);

    return product;
}
```

<br>

위 코드에서는 테스트를 수행하기 위해 3~7번 줄과 같이 공급업체 하나와 상품 객체를 3개 생성한다. 영속성 전이를 테스트하기 위해 객체에는 영속화 작업을 수행하지 않고 10~14번 줄처럼 연관관계만 설정한다. 영속성 전이가 수행되는 부분은 16번 줄이다. 16번 줄에서 데이터베이스에 저장하는 쿼리를 보면 다음과 같다.

<br>

```
Hibernate: 
    insert 
    into
        provider
        (created_at, updated_at, name) 
    values
        (?, ?, ?)
Hibernate: 
    insert 
    into
        product
        (created_at, updated_at, name, price, provider_id, stock) 
    values
        (?, ?, ?, ?, ?, ?)
Hibernate: 
    insert 
    into
        product
        (created_at, updated_at, name, price, provider_id, stock) 
    values
        (?, ?, ?, ?, ?, ?)
Hibernate: 
    insert 
    into
        product
        (created_at, updated_at, name, price, provider_id, stock) 
    values
        (?, ?, ?, ?, ?, ?)
```

<br>

지금까지는 엔티티를 데이터베이스에 저장하기 위해 각 엔티티를 저장하는 코드를 작성해야 했다. 하지만 영속성 전이를 사용하면 부모 엔티티가 되는 `Provider`엔티티만 저장하면 코드에 작성돼 있는 `Cascade.PERSIST`에 맞춰 상품 엔티티도 함께 저장할 수 있다.

이처럼 특정 상황에 맞춰 영속성 전이 타입을 설정하면 영속 상태의 변화에 따라 연관된 엔티티들의 동작도 함께 수핼할 수 있어 개발의 생산성이 높아진다. 다만 자동 설정으로 동작하는 코드들이 정확히 어떤 영향을 미치는지 파악할 필요가 있다. 예를 들어, `REMOVE`와 `REMOVE`를 포함하는 `ALL` 같은 타입을 무분별하게 사용하면 연관된 엩니니가 의도치 않게 모두 삭제될 수 있기 때문에 다른 타입보다 더궁 사이트 이펙트(side effect)를 고려해서 사용해야 한다.



### 고아 객체

JPA에서 고아(orphan)란 부모 엔티티와 연관관계가 끊어진 엔티티를 의미한다 JPA에는 이러한 고아 객체를 자동으로 제거하는 기능이 있다. 물론 자식 엔티티가 다른 엔티티와 연관관계를 가지고 있다면 이 기능은 사용하지 않는 것이 좋다. 현재 예제에서 사용되는 상품 엔티티는 다른 엔티티와 연관관계가 많이 설정돼 있지만 그 부분은 예외로 두고 테스트를 진행하겠다.

고아 객체를 제거하는 기능을 사용하기 위해서는 공급업체 엔티티를 아래와 같이 작성한다.

<br>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "provider")
public class Provider extends BaseEntity{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String name;

    @OneToMany(mappedBy = "provider", fetch = FetchType.EAGER, cascade = CascadeType.PERSIST, orphanRemoval = true)
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();

}
```

<br>

예제의 16번 줄에 있는 '`orphanRemoval = true`'속성은 고아 객체를 제거하는 기능이다. 이 설정을 추가하고 정상적으로 동작하는지 확인하기 위해 아래와 같이 테스트 코드를 작성한다.

<br>

```java
@Test
@Transactional
void orphanRemovalTest() {
    Provider provider = savedProvider("새로운 공급업체");

    Product product1 = savedProduct("상품1", 1000, 1000);
    Product product2 = savedProduct("상품2", 500, 1500);
    Product product3 = savedProduct("상품3", 750, 500);

    product1.setProvider(provider);
    product2.setProvider(provider);
    product3.setProvider(provider);

    provider.getProductList().addAll(Lists.newArrayList(product1, product2, product3));

    providerRepository.saveAndFlush(provider);

    providerRepository.findAll().forEach(System.out::println);
    productRepository.findAll().forEach(System.out::println);

    Provider foundProvider = providerRepository.findById(1L).get();
    foundProvider.getProductList().remove(0);

    providerRepository.findAll().forEach(System.out::println);
    productRepository.findAll().forEach(System.out::println);
}
```

<br>

먼저 테스트 데이터를 생성하고 10~14번 줄처럼 연관관계 매핑을 수행한다. 연관관계가 매핑된 각 엔티티들을 저장한 후 18~19번 줄에서 각 엔티티를 출력하면 다음과 같이 4~8번 줄에서 생성한 공급업체 엔티티 1개, 상품 엔티티3개가 출력되는 것을 볼 수 있다.

<br>

```
Hibernate: 
    select
        provider0_.id as id1_6_,
        provider0_.created_at as created_2_6_,
        provider0_.updated_at as updated_3_6_,
        provider0_.name as name4_6_ 
    from
        provider provider0_
Provider(super=BaseEntity(createdAt=2022-08-07T15:55:59.206071, updatedAt=2022-08-07T15:55:59.206071), id=1, name=새로운 공급업체)
Hibernate: 
    select
        product0_.number as number1_3_,
        product0_.created_at as created_2_3_,
        product0_.updated_at as updated_3_3_,
        product0_.name as name4_3_,
        product0_.price as price5_3_,
        product0_.provider_id as provider7_3_,
        product0_.stock as stock6_3_ 
    from
        product product0_
Product(super=BaseEntity(createdAt=2022-08-07T15:55:59.246255, updatedAt=2022-08-07T15:55:59.246255), number=1, name=상품1, price=1000, stock=1000)
Product(super=BaseEntity(createdAt=2022-08-07T15:55:59.257967, updatedAt=2022-08-07T15:55:59.257967), number=2, name=상품2, price=500, stock=1500)
Product(super=BaseEntity(createdAt=2022-08-07T15:55:59.259254, updatedAt=2022-08-07T15:55:59.259254), number=3, name=상품3, price=750, stock=500)
```

<br>

그리고 고아 객체를 생성하기 위해 위 코드 21~22번 줄에 생성한 공급업체 엔티티를 가져올 후 첫 번째로 매핑돼 있는 상품 엔티티의 연관관계를 제거했다. 그리고 전체 조회 코드를 24~25번 줄에서 수행하면 다음과 같이 연관관계가 끊긴 상품의 엔티티가 제거된 것을 확인할 수 있다.

<br>

```
Hibernate: 
    select
        provider0_.id as id1_6_,
        provider0_.created_at as created_2_6_,
        provider0_.updated_at as updated_3_6_,
        provider0_.name as name4_6_ 
    from
        provider provider0_
Provider(super=BaseEntity(createdAt=2022-08-07T15:55:59.206071, updatedAt=2022-08-07T15:55:59.206071), id=1, name=새로운 공급업체)
Hibernate: 
    delete 
    from
        product 
    where
        number=?
Hibernate: 
    select
        product0_.number as number1_3_,
        product0_.created_at as created_2_3_,
        product0_.updated_at as updated_3_3_,
        product0_.name as name4_3_,
        product0_.price as price5_3_,
        product0_.provider_id as provider7_3_,
        product0_.stock as stock6_3_ 
    from
        product product0_
Product(super=BaseEntity(createdAt=2022-08-07T15:55:59.257967, updatedAt=2022-08-07T15:55:59.257967), number=2, name=상품2, price=500, stock=1500)
Product(super=BaseEntity(createdAt=2022-08-07T15:55:59.259254, updatedAt=2022-08-07T15:55:59.259254), number=3, name=상품3, price=750, stock=500)
```

<br>

출력 결과에서 10~15번 줄을 보면 실제로 연관관계가 제거되면서 하이버네이트에서는 상태 감지를 통해 삭제하는 쿼리가 수행되는 것을 볼 수 있다.



### 정리

이번 장에서는 연관관계를 설정하는 방법과 영속성 전이라는 개념을 알아봤다. JPA를 사용할 때 영속이라는 개념은 매우 중요하다. 코드를 통해 편리하게 데이터베이스에 접근하기 위해서는 중간에서 엔티티의 상태를 조율하는 영속성 컨텍스트가 어떻게 동작하는지 이해해야 한다.

이 과정에서 하이버네이트를 직접 사용하는 것과 Spring Data JPA를 사용하는 데는 차이가 있다. 따라서 이 책에서 다루지 않았지만 하이버네이트만 사용하는 JPA도 함게 공부하는 것을 권장한다. JPA 자체를 그대로 다뤄보는 경험을 해보면 DAO와 리포지토리의 차이에 대해서도 더 알 수 있게 되고 스프링 부트 JPA에 대해서도 좀 더 폭넓게 이해할 수 있게 된다.

