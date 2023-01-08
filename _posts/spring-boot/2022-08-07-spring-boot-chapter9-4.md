---
title : "[SpringBoot][Chapter9.4] 연관관계 매핑 - 다대일, 일대다 매핑"
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



# 다대일, 일대다 매핑

상품 테이블과 공급업체 테이블은 아래와 같이 상품 테이블의 입장에서 볼 경우에는 다대일, 공급업체 테이블의 입장에서 볼 경우에는 일대다 관계로 볼 수 있다. 이런 관계는 어떻게 구현해야 할지 직접 매핑하면서 알아보겠다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182394083-ab0b6506-d85b-49a4-b015-074a34444755.png){: .align-center}

<br>



## 다대일 단반향 매핑

먼저 공급업체 테이블에 매핑되는 엔티티 클래스를 만들겠다. 아래와 같이 엔티티 클래스를 생성할 수 있다.

<br>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@EqualsAndHashCode(callSuper = true)
@Table(name = "provider")
public class Provider extends BaseEntity{
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
}
```

<br>

공급업체는 `Provider`라는 도메인을 사용해 정의했다. 공급업체의 정보를 담는다면 더 많은 칼럼이 필요하겠지만 간단한 실습을 위해 필드로는 `id`와 `name`만 작성한다. 여기에 `BaseEntity`를 통해 생성일자와 변경일자를 상속받는다.

상품 엔티티에서는 공급업체의 번호를 받기 위해 다음과 같이 엔티티 필드의 구성을 추가해야 한다. 아래와 같이 상품 엔티티에 필드를 추가하겠다.

<br>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
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
    @ToString.Exclude
    private ProductDetail productDetail;
    
    @ManyToOne
    @JoinColumn(name = "provider_id")
    @ToString.Exclude
    private Provider provider;

}
```

<br>

위 예제의 27~29번 줄은 공급업체 엔티티에 대한 다대일 연관관계를 설정한다. 일반적으로 외래키를 갖는 쪽이 주인의 역할을 수행하기 때문에 이 경우 상품 엔티티가 공급업체 엔티티의 주인이다. 위와 같이 설정한 후 애플리케이션을 가동하면 아래와 같이 `Product` 테이블과 `Provider` 테이블이 생성된다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183254072-20660c29-9577-4ecd-93c6-8b9b443503b0.png){: .align-center}

<br>

![image](https://user-images.githubusercontent.com/13410737/183254039-96b56c4f-ee41-430c-846f-4d715f2ad8ea.png){: .align-center}

<br>

당장은 사용하지 않지만 이후 공급업체 엔티티를 활용할 수 있게 리포지토리를 생성한다. 기존에 리포지토리를 생성했던 방식과 동일하게 아래와 같이 코드를 작성한다.

<br>

```java
public interface ProviderRepository extends JpaRepository<Provider, Long> {
    
}
```

<br>

이제 작성된 코드를 기반으로 테스트를 진행하겠다. 지금 다루고 있는 두 엔티티에서 주인은 `Product`엔티티이기 때문에 `ProductRepository`를 활용해 테스트한다.

<br>

```java
@SpringBootTest
public class ProviderRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Autowired
    ProviderRepository providerRepository;

    @Test
    void relationshipTest1(){
        // 테스트 데이터 생성
        Provider provider = new Provider();
        provider.setName("ㅇㅇ물산");

        providerRepository.save(provider);

        Product product = new Product();
        product.setName("가위");
        product.setPrice(5000);
        product.setStock(500);
        product.setProvider(provider);

        productRepository.save(product);

        // 테스트
        System.out.println(
                "product : " + productRepository.findById(1L)
                .orElseThrow(RuntimeException::new));

        System.out.println("provider : " + productRepository.findById(1L)
        .orElseThrow(RuntimeException::new).getProvider());
    }
}
```

<br>

이제 각 엔티티의 연관관계를 테스트하기 위해 테스트 데이터를 생성해야 한다. 그렇기 때문에 4~8번줄과 같이 두 리포지토리에 대한 의존성 주입을 받았다. 그 뒤 각 리포지토리를 통해 13~24번 줄과 같이 테스트 데이터를 생성한다. 22번 줄은 위에서 생성한 `provider` 객체를 `product`에 추가해서 데이터베이스에 저장하는 코드이다. 애플리케이션을 실행했을 때 하이버네이트로 생성된 쿼리를 보면 다음과 같다.

<br>

```
Hibernate: 
    insert 
    into
        product
        (created_at, updated_at, name, price, provider_id, stock) 
    values
        (?, ?, ?, ?, ?, ?)
```

<br>

쿼리로 데이터를 저장할 때는 `provider_id` 값만 들어가는 것을 볼 수 있다. 이렇게 `product` 테이블에는 `@JoinColumn`에 설정한 이름을 기반으로 자동으로 값을 선정해서 추가하게 된다.

주요 테스트 코드는 27~32번 줄이다 `Product` 엔티티에서 단방향으로 `Provider` 엔티티 연관관계를 맺고 있기 때문에 `ProductRepository`만으로도 `Provider`객체도 조회가 가능하다. 27~29번 줄에서 생성하는 쿼리와 31~32번 줄에서 생성하는 쿼리는 다음과 같이 동일하게 실행된다.

<br>

```java
Hibernate: 
    select
        product0_.number as number1_0_0_,
        product0_.created_at as created_2_0_0_,
        product0_.updated_at as updated_3_0_0_,
        product0_.name as name4_0_0_,
        product0_.price as price5_0_0_,
        product0_.provider_id as provider7_0_0_,
        product0_.stock as stock6_0_0_,
        provider1_.id as id1_2_1_,
        provider1_.created_at as created_2_2_1_,
        provider1_.updated_at as updated_3_2_1_,
        provider1_.name as name4_2_1_,
        productdet2_.id as id1_1_2_,
        productdet2_.created_at as created_2_1_2_,
        productdet2_.updated_at as updated_3_1_2_,
        productdet2_.description as descript4_1_2_,
        productdet2_.product_number as product_5_1_2_ 
    from
        product product0_ 
    left outer join
        provider provider1_ 
            on product0_.provider_id=provider1_.id 
    left outer join
        product_detail productdet2_ 
            on product0_.number=productdet2_.product_number 
    where
        product0_.number=?
```

<br>

이 쿼리가 수행되고 출력된 결과는 다음과 같다.

```
product : Product(super=BaseEntity(createdAt=2022-08-07T00:01:02.562372, updatedAt=2022-08-07T00:01:02.562372), number=1, name=스프링 부트 JPA, price=5000, stock=500), provider =Provider(super=BaseEntity(createdAt=2022-08-07T00:18:23.958813, updatedAt=2022-08-07T00:18:23.958813), id=1, name=ㅇㅇ물산))
```



## 다대일 양방향 매핑

앞에서 상품 엔티티와 공급업체 엔티티 사이에 다대일 단방향 연관관계를 설정했다. 이제 반대로 공급업체를 통해 등록된 상품을 조회하기 위한 일대다 연관관계를 설정해 보겠다. JPA에서는 이처럼 양쪽에서 단방향으로 매핑하는 것이 양방향 매핑 방식이다. 이번에는 아래와 같이 공급업체 엔티티에서만 연관관계를 설정한다.

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

    @OneToMany(mappedBy = "provider", fetch = FetchType.EAGER)
    @ToString.Exclude
    private List<Product> productList = new ArrayList<>();
    
}
```

<br>
일대다 연관관계의 경우 여러 상품 엔티티가 포함될 수 있어 18번 줄과 같이 컬렉션(`Collection`, `List`, `Map`) 형식으로 필드를 생성한다. 이렇게 `@OneToMany`가 붙은 쪽에서 `@JoinColumn` 어노테이션을 사용하면 상대 엔티티에 외래키가 설정된다. 또한 롬복의 `ToString`에 의해 순환참조가 발생할 수 있어 17번 줄과 같이 `ToString`에서 제외 처리를 하는 것이 좋다 16번 줄처럼 '`fetch = FetchType,EAGER`'로 설정한 것은 `@OneToMany`의 기본 fetch 전략이 `Lazy`이기 때문에 즉시 로딩으로 조정한 것이다. 아퓨에서 진행할 테스트에서 지연 로딩 방식을 사용하면 'no Session'으로 에러가 발생하기 때문에 조정했다.

`Provider` 엔티티 클래스를 수정해도 애플리케이션을 가동해보면 칼럼은 변경되지 않는다. `mappedBy`로 설정된 필드는 칼럼에 적용되지 않는다. 즉, 양쪽에서 연관관계를 설정하고 있을 때 RDBMS의 형식처럼 사용하기 위해 `mappedBy`를 통해 한쪽으로 외래키 관리를 위임한 것이다.

<br>

> **💡Tip.  지연로딩과 즉시로딩**
>
> JPA에서 지연로딩(lazy loading)과 즉시로딩(eager loading)은 중요한 개념이다. 엔티티라는 객체의 개념으로 데이터베이스를 구현했기 때문에 연관관계를 가진 각 엔티티 클래스에는 연관관계가 있는 객체들이 필드에 존재하게 된다.
>
> 연관관계와 상관없이 즉각 해당 엔티티의 값만 조회하고 싶거나 연관관계를 가진 테이블의 값도 조회하고 싶은 경우 등 여러 조건들을 만족하기 위해 당장한 개념이 지연로딩과 즉시로딩이다.

<br>

그럼 수정된 공급업체 엔티티를 가지고 연관된 엔티티의 값을 가져올 수 있는지 테스트하겠다. 아래와 같이 테스트 코드를 작성해서 테스트를 진행한다.

<br>

```java
@Autowired
ProductRepository productRepository;

@Autowired
ProviderRepository providerRepository;

@Test
void relationshipTest() {
    // 테스트 데이터 생성
    Provider provider = new Provider();
    provider.setName("ㅇㅇ상사");
    
    providerRepository.save(provider);
    
    Product product1 = new Product();
    product1.setName("펜");
    product1.setPrice(2000);
    product1.setStock(100);
    product1.setProvider(provider);
    
    Product product2 = new Product();
    product2.setName("가방");
    product2.setPrice(20000);
    product2.setStock(200);
    product2.setProvider(provider);

    Product product3 = new Product();
    product3.setName("노트");
    product3.setPrice(3000);
    product3.setStock(1000);
    product3.setProvider(provider);
    
    productRepository.save(product1);
    productRepository.save(product2);
    productRepository.save(product3);
    
    List<Product> products = providerRepository.findById(provider.getId()).get()
            .getProductList();
    
    for(Product product : products) {
        System.out.println(product);
    }
    
}
```

<br>

`Provider` 엔티티 클래스는 `Product` 엔티티와의 연관관계에서 주인이 아니기 때문에 외래키를 관리할 수 없다. 그렇기 때문에 테스트 데이터를 생성하는 11~36번 줄과 같이 `Provider`를 등록한 후 각 `Product`에 객체를 설정하는 작업을 통해 데이터베이스에 저장한다. 만약 앞의 예제에서 테스트 데이터를 생성하는 방식이 아니라 `Provider`엔티티에 정의한 `productList` 필드에 아래와 같이 `Product`엔티티를 추가하는 방식으로 데이터베이스에 레코드를 저장하게 되면 `Provider` 엔티티 클래스는 연관관계의 주인이 아니기 때문에 해당 데이터는 데이터베이스에 반영되지 않는다.

<br>

```java
provider.getProductList.add(product1); // 무시
provider.getProductList.add(product2); // 무시
provider.getProductList.add(product3); // 무시
```

<br>

38~43번 줄에서는 `ProviderRepository`를 통해 연관관계가 매핑된 `Product` 리스트를 가져와 출력한다. 이때 생성되는 `select`쿼리는 다음과 같다.

<br>

```
Hibernate: 
    select
        provider0_.id as id1_2_0_,
        provider0_.created_at as created_2_2_0_,
        provider0_.updated_at as updated_3_2_0_,
        provider0_.name as name4_2_0_,
        productlis1_.provider_id as provider7_0_1_,
        productlis1_.number as number1_0_1_,
        productlis1_.number as number1_0_2_,
        productlis1_.created_at as created_2_0_2_,
        productlis1_.updated_at as updated_3_0_2_,
        productlis1_.name as name4_0_2_,
        productlis1_.price as price5_0_2_,
        productlis1_.provider_id as provider7_0_2_,
        productlis1_.stock as stock6_0_2_,
        productdet2_.id as id1_1_3_,
        productdet2_.created_at as created_2_1_3_,
        productdet2_.updated_at as updated_3_1_3_,
        productdet2_.description as descript4_1_3_,
        productdet2_.product_number as product_5_1_3_ 
    from
        provider provider0_ 
    left outer join
        product productlis1_ 
            on provider0_.id=productlis1_.provider_id 
    left outer join
        product_detail productdet2_ 
            on productlis1_.number=productdet2_.product_number 
    where
        provider0_.id=?
```

<br>



## 일대다 단방향 매핑

앞에서 다대일 연관관계에서의 단방향과 양방향 매핑을 살펴봤다. 이번에는 일대다 단방향 매핑 방법을 알아보겠다. 참고로 일대다 양방향 매핑은 다루지 않을 예정이다. 그 이유는 `@OneToMany`를 사용하는 입장에서는 어느 엔티티 클래스도 연관관계의 주인이 될 수 없기 때문이다. `@OneToMany`관계에서 주인이 될 수 없는 이유는 이번 절에서 함께 다루겠다.

먼저 실습을 위해 새로운 엔티티를 생성하겠다. 아래와 같이 상품분류 테이블을 추가한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183257126-49ad2c78-b3bc-4025-8b71-1a88b4b6317b.png){: .align-center}

<br>

위의 테이블 구조와 맞추기 위한 상품 분류 엔티티를 아래와 같이 생성한다. 상품 분류의 도메인 이름은 `Category`로 하겠다.

<br>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString
@EqualsAndHashCode
@Table(name = "category")
public class Category {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(unique = true)
    private String code;
    
    private String name;
    
    @OneToMany(fetch = FetchType.EAGER)
    @JoinColumn(name = "category_id")
    private List<Product> products = new ArrayList<>();
    
}
```

<br>

위와 같이 상품 분류 엔티티 클래스를 생성하고 애플리케이션을 실행하면 아래와 같이 상품 분류(`category`) 테이블이 생성되고 상품 테이블에 외래키가 추가되는 것을 확인할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183257424-c3c0ff56-29a8-4c71-84ec-ca4a2592972b.png){: .align-center}

<br>

![image](https://user-images.githubusercontent.com/13410737/183257437-d29c1b61-00e5-4a62-8665-39d48d9a70c5.png){: .align-center}

<br>

상품 분류 엔티티에서 `@OneToMany`와 `@JoinColumn`을 사용하면 상품 엔티티에서 별도의 설정을 하지 않아도 일대다 단방향 연관관계가 매핑된다. 앞에서 언급한 것처럼 `@JoinColumn`어노테이션은 필수 사항은 아니다. 이 어노테이션을 사용하지 않으면 중간 테이블로 join 테이블이 생성되는 전략이 채택된다.

위 그림과 같은 화면이 나오면 단방향 매핑이 완료된 상태이다. 지금 같은 일대다 단방향 관계의 단점은 매핑의 주체가 아닌 반대 테이블에 외래키가 추가된다는 점이다. 이 방식은 다대일 구조와 다르게 외래키를 설정하기 위해 다른 테이블에 대한 `update`쿼리를 발생시킨다 테스트를 통해 확인해 보겠다. 아래와 같이 `CategoryRepository`를 생성한다.

<br>

```java
public interface CategoryRepository extends JpaRepository<Category, Long> {
    
}
```

<br>

이제 연관관계를 활용한 테스트 코드를 작성해보겠다.

<br>

```java
@SpringBootTest
public class CategoryRepositoryTest {

    @Autowired
    ProductRepository productRepository;

    @Autowired
    CategoryRepository categoryRepository;

    @Test
    void relationshipTest(){
        // 테스트 데이터 생성
        Product product = new Product();
        product.setName("펜");
        product.setPrice(2000);
        product.setStock(100);

        productRepository.save(product);

        Category category = new Category();
        category.setCode("S1");
        category.setName("도서");
        category.getProducts().add(product);
        
        categoryRepository.save(category);
        
        // 테스트 코드
        List<Product> products = categoryRepository.findById(1L).get().getProducts();
        
        for(Product foundProduct : products){
            System.out.println(product);
        }
    }
}
```

<br>

테스트 데이터 생성을 위해 `ProductRepository`의 의존성도 함께 주입받겠다. 위 예제에서 23번 줄과 같이 `Product` 객체를 `Category`에서 생성된 리스트 객체에 추가해서 연관관계를 설정한다. 우선 13~25번 줄의 테스트 데이터 생성 쿼리는 다음과 같이 생성된다.

<br>

```
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
        category
        (code, name) 
    values
        (?, ?)
Hibernate: 
    update
        product 
    set
        category_id=? 
    where
        number=?
```

<br>

일대다 연관관계에서는 위와 같이 연관관계 설정을 위한 `update`쿼리가 발생한다. 이 같은 문제를 해결하기 위해서는 일대다 연관관계를 사용하기보다는 다대일 연관관계를 사용하는 것이 좋다. 이렇게 테스트 데이터를 생성한 뒤에 `CategoryRepository`를 활용해 상품정보를 가져오는 테스트 코드를 실행하면 다음과 같은 쿼리가 생성된다.

<br>

```
Hibernate: 
    select
        category0_.id as id1_0_0_,
        category0_.code as code2_0_0_,
        category0_.name as name3_0_0_,
        products1_.category_id as category8_1_1_,
        products1_.number as number1_1_1_,
        products1_.number as number1_1_2_,
        products1_.created_at as created_2_1_2_,
        products1_.updated_at as updated_3_1_2_,
        products1_.name as name4_1_2_,
        products1_.price as price5_1_2_,
        products1_.provider_id as provider7_1_2_,
        products1_.stock as stock6_1_2_,
        provider2_.id as id1_3_3_,
        provider2_.created_at as created_2_3_3_,
        provider2_.updated_at as updated_3_3_3_,
        provider2_.name as name4_3_3_,
        productdet3_.id as id1_2_4_,
        productdet3_.created_at as created_2_2_4_,
        productdet3_.updated_at as updated_3_2_4_,
        productdet3_.description as descript4_2_4_,
        productdet3_.product_number as product_5_2_4_ 
    from
        category category0_ 
    left outer join
        product products1_ 
            on category0_.id=products1_.category_id 
    left outer join
        provider provider2_ 
            on products1_.provider_id=provider2_.id 
    left outer join
        product_detail productdet3_ 
            on products1_.number=productdet3_.product_number 
    where
        category0_.id=?
```

<br>

알대다 연관관계에서는 이처럼 `category`와 `product`의 조인이 발생해서 상품 데이터를 정상적으로 가져온다.

