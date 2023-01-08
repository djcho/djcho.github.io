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



# 다대다 매핑

다대다(M:N) 연관관계는 실무에서 거의 사용되지 않는 구성이다. 다대다 연관관계를 상품과 생산업체의 예로 들자면 한 종류의 상품이 여러 생산업체를 통해 생산될 수 있고, 생산업체 한 곳이 여러 상품을 생산할 수도 있다.

![image](https://user-images.githubusercontent.com/13410737/183258315-1ad034a6-9a71-46bc-8b9b-6fef105d373e.png){: .align-center}

다대다 연관관계에서는 각 엔티티에서 서로를 리스트로 가지는 구조가 만들어진다. 이런 경우에는 교차 엔티티라고 부르는 중간 테이블을 생성해서 다대다 관계를 일대다 또는 다대일 관계로 해소한다.



## 다대다 단방향 매핑

앞의 그림과 같은 연관관계를 가진 생산업체 엔티티를 생성해보겠다 생산업체에 매핑되는 도메인은 `Producer`라고 가정하고 아래와 같이 작성한다.

<br>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@ToString(callSuper = true)
@EqualsAndHashCode(callSuper = true)
@Table(name = "producer")
public class Producer extends BaseEntity{
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String code;
    
    private String name;
    
    @ManyToMany
    @ToString.Exclude
    private List<Product> products = new ArrayList<>();
    public void addProduct(Product product){
        products.add(product);
    }
    
}
```

<br>

위 예제에서 18~19번 줄에서 상품 엔티티에 적용한 것과 같이 다대다 연관관계는 `@ManyToMany` 어노테이션으로 설정한다. 리스트로 필드를 가지는 객체에서는 외래키를 가지지 않기 때문에 별도의 `@JoinColumn`은 설정하지 않아도 된다. 이렇게 엔티티를 생성하고 애플리케이션을 실해0ㅇ하면 아래와 같이 생산업체 테이블이 생성된다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183275224-7dd10ef6-7dc7-4639-b6c8-6d515a8dd0cb.png){: .align-center}

<br>

생산업체 테이블에는 별도의 외래키가 추가되지 않은 것을 볼 수 있다. 그리고 데이터베이스에 추가로 중산 테이블이 생성돼 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183275242-7a0a832c-ddae-4d3e-b35a-0fba90ab93ff.png){: .align-center}

<br>

별도의 설정을 하지 않았따면 테이블은 `producer_products`라는 이름으로 설정되며, 만약 테이블의 이름을 관리하고 싶다면 위 코드의 18번째 줄에 있는 `@ManyToMany` 어노테이션 아래에 `@JoinTable(name = "이름")`의 형식으로 어노테이션을 정의하면 된다.

`producer_products` 테이블의 경우 상품 테이블과 생산업체 테이블에서 `id`값을 가져와 두 개의 외래키가 설정되는 것을 볼 수 있다. 그럼 이러한 연관관계를 테스트하기 위해 아래와 같이 생산업체 엔티티에 대한 리포지토리를 생성해보겠다.

<br>

```java
public interface ProducerRepository extends JpaRepository<Producer, Long> {
    
}
```

<br>

이 같은 리포지토리를 생성하면 생산업체에 대한 기본적인 데이터베이스 조작이 가능하다. 그럼 아래 예제와 같이 테스트 코드를 작성해서 앞에서 설정한 연관관계가 정상적으로 동작하는지 확인하겠다.

<br>

```java
@Autowired
ProducerRepository producerRepository;

@Autowired
ProductRepository productRepository;

@Test
@Transactional
void relationshipTest(){

    Product product1 = saveProduct("동글펜", 500, 1000);
    Product product2 = saveProduct("네모 공책", 100, 2000);
    Product product3 = saveProduct("지우개", 152, 1234);

    Producer producer1 = saveProducer("flature");
    Producer producer2 = saveProducer("wikibooks");

    producer1.addProduct(product1);
    producer1.addProduct(product2);

    producer2.addProduct(product2);
    producer2.addProduct(product3);

    producerRepository.saveAll(Lists.newArrayList(producer1, producer2));

    System.out.println(producerRepository.findById(1L).get().getProducts());

}

private Product saveProduct(String name, Integer price, Integer stock) {
    Product product = new Product();
    product.setName(name);
    product.setPrice(price);
    product.setStock(stock);

    return productRepository.save(product);
}

private Producer saveProducer(String name) {
    Producer producer = new Producer();
    producer.setName(name);

    return producerRepository.save(producer);
}
```

<br>

위 예제에서는 가독성을 위해 리포지토리를 통해 테스트 데이터를 생성하는 부분을 별도 메서드로 구현했다. 이 경우 리포지토리를 사용하게 되면 매번 트랜잭션이 끊어져 생산업체 엔티티에서 상품 리스트를 가져오는 작업이 불가능하다. 이 같은 문제를 해소하기 위헤 메서드에 `@Transactional`어노테이션을 지정해 트랜잭션이 유지되도록 구성해서 테스트를 진행한다.

26번줄의 생산업체 엔티티와 연관관계가 설정된 상품 데이터 리스트를 출력하면 다음과 같은 내용이 출력된다.

<br>

```
[Product(super=BaseEntity(createdAt=2022-08-07T13:59:53.639112, updatedAt=2022-08-07T13:59:53.639112), number=2, name=동글펜, price=500, stock=1000), Product(super=BaseEntity(createdAt=2022-08-07T13:59:53.642471, updatedAt=2022-08-07T13:59:53.642471), number=3, name=네모 공책, price=100, stock=2000)]
```

<br>

앞서 18~22번 줄에서 연관관계를 설정했기 때문에 정상적으로 생산업체 엔티티에서 상품 리스트를 가져오는 것을 볼 수 있다.

앞의 테스트를 통해 테스트 데이터를 생성하면 `product`테이블과 `producer`테이블에 레코드가 추가되지만 보여지는 내용만으로는 연관관계 설정 여부를 확인하기 어렵다. 그 이유는 다대다 연관관계 설정을 통해 생성된 중간 테이블에 연관관계 매핑이 돼 있기 때문이다. 중간 테이블에 생성된 레코드를 확인하면 아래와 같다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183276550-e06f6a5f-525b-4abb-83f4-96e5284e986f.png){: .align-center}

<br>

`producer_products`라는 이름의 중간 테이블에는 18~22번 줄에서 설정한 연관관계에 맞춰 양 테이블의 기본키를 매핑한 레코드가 생성된 것을 볼 수 있다.



## 다대다 양방향 매핑

다대다 단방향 매핑의 개념을 이해했다면 양방향 매핑을 하는 방법은 간단하다. 상품 엔티티에서 아래와 같이 작성한다.

<br>

```java
Entity
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

    @ManyToMany
    @ToString.Exclude
    private List<Producer> producers = new ArrayList<>();

    public void addProducer(Producer producer){
        this.producers.add(producer);
    }

}
```

<br>

이 예제에서 32~34번 줄은 생산업체에 대한 다대다 연관관계를 설정한다. 필요에 따라 `mappedBy` 속성을 사용해 두 엔티티 간 연관관계의 주인을 설정할 수도 있다. 이렇게 설정한 후 애플리케이션을 실행하면 데이터베이스의 테이블 구조는 변경되지 않는다. 중간 테이블이 연관관계를 성정하고 있기 때문이다. 이렇게 구성한 후 아래와 같이 테스트 코드를 작성해보자

<br>

```java
@Test
@Transactional
void relationshipTest2(){
    Product product1 = saveProduct("동글펜", 500, 1000);
    Product product2 = saveProduct("네모 공책", 100, 2000);
    Product product3 = saveProduct("지우개", 152, 1234);

    Producer producer1 = saveProducer("flature");
    Producer producer2 = saveProducer("wikibooks");

    producer1.addProduct(product1);
    producer1.addProduct(product2);
    producer2.addProduct(product2);
    producer2.addProduct(product3);
    
    product1.addProducer(producer1);
    product2.addProducer(producer1);
    product2.addProducer(producer2);
    product3.addProducer(producer2);
    
    producerRepository.saveAll(Lists.newArrayList(producer1, producer2));
    productRepository.saveAll(Lists.newArrayList(product1, product2, product3));
    
    System.out.println("products : " + producerRepository.findById(1L).get().getProducts());
    
    System.out.println("producers : " + productRepository.findById(2L).get().getProducers());
    
}
```

<br>

위 테스트에서 사용되는 테스트 데이터는 이전에 작성한 메서드를 활용했다. 여기서 양방향 연관관계 설정을 위해 17~20번의 연관관계 설정 코드를 추가했다. 연관관계를 설정하고 아래의 25~27번 줄처럼 각 엔티티에 연관된 엔티티를 출력하면 다음과 같이 정상적으로 출력되는 것을 볼 수 있다.

<br>

```
[Product(super=BaseEntity(createdAt=2022-08-07T15:15:44.662646, updatedAt=2022-08-07T15:15:44.662646), number=1, name=동글펜, price=500, stock=1000), Product(super=BaseEntity(createdAt=2022-08-07T15:15:45.794736, updatedAt=2022-08-07T15:15:45.794736), number=2, name=네모 공책, price=100, stock=2000)]
[Producer(super=BaseEntity(createdAt=2022-08-07T15:15:52.537577, updatedAt=2022-08-07T15:15:52.537577), id=1, code=null, name=flature), Producer(super=BaseEntity(createdAt=2022-08-07T15:15:52.600543, updatedAt=2022-08-07T15:15:52.600543), id=2, code=null, name=wikibooks)]
```

<br>

이렇게 다대다 연관관계를 설정하면 중간 테이블을 통해 연관된 엔티티의 값을 가져올 수 있다. 다만 다대다 연관관계에서는 중간 테이블이 생성되기 때문에 예기치 못한 쿼리가 생길 수 있다. 즉, 관리하기 힘든 포인트가 발생한다는 문제가 있다. 그렇게 때문에 이러한 다대다 연관관계의 한계를 극복하기 위해서는 중간 테이블을 생성하는 일대다 다대일로 연관관계를 맺을 수 있는 중간 엔티티로 승격시켜 JPA에서 관리할 수 있게 생생하는 것이 좋다.

