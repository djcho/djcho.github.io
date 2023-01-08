---
title : "[SpringBoot][Chapter9.3] 연관관계 매핑 - 일대일 매핑"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, JPA]
toc: true
toc_sticky : true
published : true
date : 2022-08-06
last_modified_at : 2022-08-06
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



# 일대일 매핑

먼저 두 엔티티 간에 일대일 매핑을 만들어 보겠다. 우선 지금까지 사용해온 `Product` 엔티티를 대상으로 아래와 같이 일대일 매핑될 상품 정보 테이블을 생성한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183249887-b3f3290c-9009-4273-8c9d-a0be6c18977e.png){: .align-center}

<br>

위와 같이 하나의 상품에 하나의 사유ㅜㅁ정보만 매핑되는 구조는 일대일 관계라고 볼 수 있다.



## 일대일 단반향 매핑

프로젝트 `entity` 패키지 안에 아래와 같이 상품정보 엔티티를 작성한다. 상품정보에 대한 도메인은 `ProductDetail`로 설정해서 진행하겠다.

<br>

```java
@Entity
@Table(name = "product_detail")
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class ProductDetail extends BaseEntity{
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String description;
    
    @OneToOne
    @JoinColumn(name = "product_number")
    private Product product;
}
```

<br>

이전에 엔티티를 작성했던 방법 그대로 상품정보 엔티티를 작성한다. 그리고 상품 번호에 매핑하기 위해 16~18번 줄과 같이 작성한다. `@OneToOne` 어노테이션은 다른 엔티티 객체를 필드로 정의했을 때 일대일 연관관계로 매핑하기 위해 사용된다. 뒤이어 `@JoinColumn` 어노테이션을 사용해 매핑할 외래키를 설정한다. `@JoinColumn` 어노테이션은 기본값이 설정돼 있어 자동으로 이름을 매핑하지만 의도한 이름이 들어가지 않기 때문에 `name`속성을 사용해 원하는 칼럼명을 지정하는 것이 좋다. 만약 `@JoinColumn`어노테이션에서 사용할 수 있는 속성을 설명하면 다음과 같다.

- `name` : 매핑할 외래키의 이름을 설정한다.
- `referencedColumnName` : 외래키가 참조할 상대 테이블의 칼럼명을 지정한다.
- `foreignKey` : 외래키를 생성하면서 지정할 제약조건을 설정한다.(`unique`, `nullable`, `insertable`, `updateable` 등).

이렇게 엔티티 클래스를 생성하면 단방향 관계의 일대일 관계 매핑이 완성된다. `hibernate.dll-auto`의 값을 `create`로 설정한 후 애플리케이션을 실행하면 하이버네이트에서 자동으로 테이블을 생성하며 아래와 같이 데이터베이스의 테이블을 확인할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183250978-ffd9ddf5-b9f4-49c0-8c5f-f5796cb91d33.png){: .align-center}

<br>

생성된 상품정보 엔티티 객체들을 사용하기 위해 리포지토리 인터페이스를 생성한다. 아래와 같이 기존에 작성했던 `ProductRepository`와 동일한 형식으로 작성한다.

<br>

```java
public interface ProductDetailRepository extends JpaRepository<ProductDetail, Long> {

}
```

<br>

그럼 연관관계를 활용한 데이터 생성 및 조회 기능을 테스트 코드로 간략하게 작성해보겠다.

<br>

```java
package com.springboot.relationship.data.repository;

import com.springboot.relationship.data.entity.Product;
import com.springboot.relationship.data.entity.ProductDetail;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

@SpringBootTest
public class ProductDetailRepositoryTest {
    
    @Autowired
    ProductDetailRepository productDetailRepository;
    
    @Autowired
    ProductRepository productRepository;
    
    @Test
    public void saveAndReadTest1() {
        Product product = new Product();
        product.setName("스프링 부트 JPA");
        product.setPrice(5000);
        product.setStock(500);
        
        productRepository.save(product);
        
        ProductDetail productDetail = new ProductDetail();
        productDetail.setProduct(product);
        productDetail.setDescription("스프링 부트와 JPA를 함께 볼 수 있는 책");
        
        productDetailRepository.save(productDetail);
        
        // 생성한 데이터 조회
        System.out.println("savedProduct : " + productDetailRepository.findById(productDetail.getId()).get().getProduct());
        
        System.out.println("savedProduct : " + productDetailRepository.findById(productDetail.getId()).get());
    }
}
```

<br>

위와 같은 테스트 코드를 실행하기 위해서는 12~16번 줄과 같이 상품과 상품정보에 매핑된 리포지토리에 대한 의존성 주입을 받아야 한다. 그리고 이 테스트에서 조회할 엔티티 객체를 20~31번 줄과 같이 저장한다. 여기서 주요 코드는 34~35번 줄이다. `ProductDetail` 객체에서 `Product` 객체를 일대일 단방향 연관관계를 설정했기 때문에 `ProductDetailRepository` 에서 `ProductDetail` 객체를 조회한 후 연관 매핑된 `Product` 객체를 조회할 수 있다. 34~35번 줄과 37~38번 줄에서 조회하는 쿼리는 다음과 같이 표현된다.

<br>

```
Hibernate: 
    select
        productdet0_.id as id1_1_0_,
        productdet0_.created_at as created_2_1_0_,
        productdet0_.updated_at as updated_3_1_0_,
        productdet0_.description as descript4_1_0_,
        productdet0_.product_number as product_5_1_0_,
        product1_.number as number1_0_1_,
        product1_.created_at as created_2_0_1_,
        product1_.updated_at as updated_3_0_1_,
        product1_.name as name4_0_1_,
        product1_.price as price5_0_1_,
        product1_.stock as stock6_0_1_ 
    from
        product_detail productdet0_ 
    left outer join
        product product1_ 
            on productdet0_.product_number=product1_.number 
    where
        productdet0_.id=?
```

<br>

`select` 구문을 보면 `ProductDetail` 객체와 `Product` 객체가 함께 조회되는 것을 볼 수 있다. 이처럼 엔티티를 조회할 때 연관된 엔티티도 함께 조회하는 것을 '즉시 로딩' 이라고 한다. 그리고 16~18번 줄에서 'left outer join'이 수행되는 것을 볼 수 있다. 여기서 left outer join이 수행되는 이유는 `@OneToOne` 어노테이션 때문이다. 아래에서 `@OneToOne` 어노테이션 인터페이스를 확인하겠다.

<br>

```java
public @interface OneToOne {
    Class targetEntity() default void.class;

    CascadeType[] cascade() default {};

    FetchType fetch() default FetchType.EAGER;

    boolean optional() default true;

    String mappedBy() default "";

    boolean orphanRemoval() default false;
}
```

<br>

이후에 더 자세히 살펴볼 예정이므로 여기서는 `fetch()` 요소와 `optional()` 요소만 보겠다. `@OneToOne` 어노테이션은 기본 fetch 전략으로 `EAGER`, 즉 즉시 로깅 전략이 채택된 것을 볼 수 있다. 그리고 `optional()` 메서드는 기본값으로 `true` 가 설정돼 있다. 기본값이 `true`인 상태는 매핑되는 값이 `nullable`이라는 것을 의미한다. 반드시 값이 있어야 한다면 `ProductDetail` 엔티티에서 속성값에 아래와 같이 설정할 수 있다.

<br>

```java
@Entity
@Table(name = "product_detail")
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
public class ProductDetail extends BaseEntity{

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String description;

    @OneToOne(optional = false)
    @JoinColumn(name = "product_number")
    private Product product;
    
}
```

<br>

위와 같이 `@OneToOne`어노테이션에 '`optional = false`' 속성을 설정하면 `product`가 `null`인 값을 허용하지 않게 된다. 위와 같이 설정하고 애플리케이션을 실행하면 다음과 같이 테이블을 생성하는 쿼리에서 `not null`이 설정되는 것을 확인할 수 있다.

<br>

```java
Hibernate: 
    create table product_detail (
       id bigint not null auto_increment,
        created_at datetime(6),
        updated_at datetime(6),
        description varchar(255),
        product_number bigint not null,
        primary key (id)
    ) engine=InnoDB
```

<br>

그리고 앞에서 작성한 예제를 실행하면 다음과 같이 쿼리문이 바뀌어 실행된다.

<br>

```java
Hibernate: 
    select
        productdet0_.id as id1_1_0_,
        productdet0_.created_at as created_2_1_0_,
        productdet0_.updated_at as updated_3_1_0_,
        productdet0_.description as descript4_1_0_,
        productdet0_.product_number as product_5_1_0_,
        product1_.number as number1_0_1_,
        product1_.created_at as created_2_0_1_,
        product1_.updated_at as updated_3_0_1_,
        product1_.name as name4_0_1_,
        product1_.price as price5_0_1_,
        product1_.stock as stock6_0_1_ 
    from
        product_detail productdet0_ 
    inner join
        product product1_ 
            on productdet0_.product_number=product1_.number 
    where
        productdet0_.id=?
```

<br>

즉, `@OneToOne` 어노테이션에 '`optional = false`' 속성을 지정한 경우에는 16~18번 줄과 같이 left outer join 이 inner join으로 바뀌어 실행된다. 이처럼 객체에 대한 설정에 따라 JPA는 최적의 쿼리를 생성해서 실행한다.

이후 내용을 진행하기 위해 `@OneToOne`에 적용했던 '`optional = false`' 속성은 제거하겠다.

> optional = false 경우 일피 하는 레코드들만 조인되어 null 데이터가 존재할 수 없는  Inner join 으로 수행된다. 반대로 optional = true 경우에는 일치 하지 않은 레코드가 존재할 경우 null로 채우는 특성을 가진 outer join을 수행한다.



## 일대일 양방향 매핑

이번에는 앞에서 생성한 일대일 단방향 설정을 양방향 설정으로 변경해 보겠다. 사실 객체에서의 양방향 개념은 양쪽에서 단방향으로 서로를 매핑하는 것을 의미한다. 일대일 양방향 매핑을 위해서는 아래와 같이 `Product` 엔티티를 추가한다.

<br>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString
@EqualsAndHashCode(callSuper = true)
@Table(name = "product")
public class Product extends BaseEntity{
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;
    
    @OneToOne
    private ProductDetail productDetail;
    
}
```

<br>

추가된 코드는 23~24번 줄이다. 이렇게 설정하고 애플리케이션을 실행하면 아래와 같이 `Product`테이블에도 칼럼이 생성되는 것을 볼 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183252754-d377a7da-f3eb-4f70-a930-57be719dfbcc.png){: .align-center}

<br>

그리고 위 예제를 실행하면 다음과 같은 쿼리가 생성되는 것을 볼 수 있다.

<br>

```java
Hibernate: 
    select
        productdet0_.id as id1_1_0_,
        productdet0_.created_at as created_2_1_0_,
        productdet0_.updated_at as updated_3_1_0_,
        productdet0_.description as descript4_1_0_,
        productdet0_.product_number as product_5_1_0_,
        product1_.number as number1_0_1_,
        product1_.created_at as created_2_0_1_,
        product1_.updated_at as updated_3_0_1_,
        product1_.name as name4_0_1_,
        product1_.price as price5_0_1_,
        product1_.product_detail_id as product_7_0_1_,
        product1_.stock as stock6_0_1_,
        productdet2_.id as id1_1_2_,
        productdet2_.created_at as created_2_1_2_,
        productdet2_.updated_at as updated_3_1_2_,
        productdet2_.description as descript4_1_2_,
        productdet2_.product_number as product_5_1_2_ 
    from
        product_detail productdet0_ 
    left outer join
        product product1_ 
            on productdet0_.product_number=product1_.number 
    left outer join
        product_detail productdet2_ 
            on product1_.product_detail_id=productdet2_.id 
    where
        productdet0_.id=?
```

<br>

여러 테이블끼리 연관관계가 설정돼 있어 여러 left outer join이 설정되는 것은 괜찮으나 위와 같이 양쪽에서 외래키를 가지고 left outer join이 두번이나 수행되는 경우는 효율성이 떨어진다. 실제 데이터베이스에서도 테이블 간 연관관계를 맺으면 한쪽 테이블이 외래키를 가지는 구조로 이뤄진다. 바로 앞에서 언급한 '주인(Owner)' 개념이다.

JPA에서도 실제 데이터베이스의 연관관계를 반영해서 한쪽의 테이블에서만 외래키를 바꿀 수 있도록 정하는 것이 좋다. 이 경우 엔티티는 양방향으로 매핑하되 한쪽에게만 외래키를 줘야 하는데, 이때 사용되는 속성 값이 `mappedBy`이다. `mappedBy`는 어떤 객체가 주인인지 표시하는 속성이라고 볼 수 있다. 아래와 같이 `Product` 객체에 `mappedBy` 속성을 추가해 보겠다.

<br>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString
@EqualsAndHashCode(callSuper = true)
@Table(name = "product")
public class Product extends BaseEntity{
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    @OneToOne(mappedBy = "product")
    private ProductDetail productDetail;

}
```

<br>

23~24번 줄을 보면 `@OneToOne` 어노테이션에 `mappedBy` 속성값을 사용했다. `mappedBy`에 들어가는 값은 연관관계를 갖고 있는 상대 엔티티에 있는 연관관계 필드의 이름이 된다. 이 설정을 마치면 `ProductDetail` 엔티티가 `Product` 엔티티의 주인이 되는 것이다. 애플리케이션을 실행하고 데이터베이스 테이블을 보면 아래와 같이 외래키가 사라진 것을 볼 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183253140-0b073aea-806d-423d-865b-1cf6912d5161.png){: .align-center}

<br>

그리고 다시 테스트 코드를 실행하면 `toString` 을 실행하는 시점에서 `StackOverflowError`가발생하는 것을 볼 수 있다. 양방향으로 연관관계가 설정되면 `ToString`을 사욜할 때 순환참조가 발생하기 때문이다. 그렇게 때문에 필요한 경우가 아니라면 대체로 단방향으로 연관관계를 설정하거나 양방향 설정이 필요할 경우에는 순환참조 제거를 위해 아래와 같이 `exclude`를사용해 `ToString`에서 제외 설정을 하는 것이 좋다.

<br>

```java
@OneToOne(mappedBy = "product")
@ToString.Exclude
private ProductDetail productDetail;
```

<br>

위와 같이 `Product` 엔티티 클래스의 코드를 수정하면 기존 테스트 코드가 정상적으로 동작하는 것을 볼 수 있다.

