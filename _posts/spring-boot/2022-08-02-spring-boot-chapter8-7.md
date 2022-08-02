---
title : "[SpringBoot][Chapter8.6] Spring Data JPA 활용 - JPA Auditing 적용하기"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, JPQL]
toc: true
toc_sticky : true
published : true
date : 2022-08-02
last_modified_at : 2022-08-02
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## JPA Auditing 적용

JPA에서 'Audit' 이란 '감시하다'라는 뜻으로, 각 데이터마다 '누가'', '언제' 데이터를 생성했고 변경했는지 감시한다는 의미로 사용된다. 앞에서 작성한 코드를 보면 알 수 있듯이 엔티티 클래스에는 공통적으로 들어가는 필드가 있다. 예를 들면 '생성 일자'와 '변경 일자' 같은 것이다. 대표적으로 많이 사용되는 필드는 다음과 같다.

- 생성 주체
- 생성 일자
- 변경 주체
- 변경 일자



이러한 필드들은 매번 엔티티를 생성하거나 변경할 때마다 값을 주입해야 하는 번거로움이 있다. 이 같은 번거로움을 해소하기 위해 Spring Data JPA에서는 이러한 값을 자동으로 넣어주는 기능을 제공한다. 더 진행하기에 앞서 이 기능은 꼭 추가해야 하는 기능은 아니지만 9장부터는 이 기능이 적용된 엔티티를 사용할 예정이므로 참고하기 바란다.



### JPA Auditing 기능 활성화

가장 먼저 스프링 부트 애플리케이션 Auditing 기능을 활성화해야 한다. 방법은 간단하다. `main()` 메서드가 있는 클래스에 아래와 같이 `@EnableJpaAuditing` 어노테이션을 추가하면 된다.

<br>

```java
@SpringBootApplication
@EnableJpaAuditing
public class AdvancedJpaApplication {

    public static void main(String[] args) {
        SpringApplication.run(AdvancedJpaApplication.class, args);
    }

}
```

<br>

위와 같이 어노테이션을 추가하면 정상적으로 기능이 활성화되지만 앞으로 다룰 내용인 테스트 코드를 작성해서 애플리케이션을 테스트하는 일부 상황에서는 오류가 발생할 수 있다. 예를 들면, `@WebMvcTest` 어노테이션을 지정해서 테스트를 수행하는 코드를 작성하면 애플리케이션 클래스를 호출하는 과정에서 예외가 발생할 수 있다. 이 같은 문제를 해결하기 위해 아래와 같이 별도 `Configuration` 클래스를 생성해서 애플리케이션 클래스의 기능과 분리해서 활성화할 수 있다. 이 책에서는 이처럼 `Configuration `클래스를 별도로 생성하는 방법을 권장한다.

<br>

```java
@Configuration
@EnableJpaAuditing
public class JpaAuditingConfiguration {
    
}
```

<br>

참고로 위와 같은 방법을 선택했다면 앞선 예제 코드에서 지정한 어노테이션은 지워야 애플리케이션이 정상적으로 동작한다.



### BaseEntity 만들기

코드의 중복을 없애기 위해서는 각 엔티티에 공통으로 들어가게 되는 칼럼(필드)을 하나의 클래스로 빼는 작업을 수행해야 한다.(반드시 그래야 하는 것은 아니지만 자주 활용되는 기법이므로 참고해둘 필요가 있다.). 아직 생성 주체와 변경 추제는 활용할 일이 없기 때문에 제외하고 먼저 생성 일자와 변경일자만 추가해서 아래와 같이 `BaseEntity`를 생성한다.

<br>

```java
@Getter
@Setter
@ToString
@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
public class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;
}
```

<br>

여기서 사용한 주요 어노테이션은 다음과 같다.

- `@MappedSuperclass` : JPA의 엔티티 클래스가 상속받을 경우 자식 클래스에게 매핑 정보를 전달한다.
- `@EntityListeners` : 엔티티를 데이터베이스에 적용하기 전후에 콜백을 요청할 수 있게 하는 어노테이션이다.
- `AuditingEntityListener` : 엔티티의 Auditing 정보를 주입하는 JPA 엔티티 리스너 클래스이다.
- `@CreatedDate` : 데이터 생성 날짜를 자동으로 주입하는 어노테이션이다.
- `@LastModifiedDate` : 데이터 수정 날짜를 자동으로 주입하는 어노테이션이다.

위와 같이 `BaseEntity`를 생성하고 `Product` 엔티티 클래스에서 공통 칼럼을 제거해서 아래와 같이 코드를 수정한다.

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
}
```

<br>

클래스에 추가하는 어노테이션은 필요에 따라 차이가 있을 수 있으나 이 책에서는 실습을 위해 여러 어노테이션을 추가했다. 그중 `@ToString`, `@EqualsAndHashCode` 어노테이션에 적용한 `callSuper`속성은 롬복 설명에서 다뤘다시피 부모 클래스의 필드를 포함하는 역할을 수행한다.

이렇게 섲어하면 기존에 테스트했던 것처럼 매번 `LocalDateTime.now()` 메서드를 사용해 시간을 주입하지 않아도 자동으로 값이 생성되는 것을 볼 수 있다.

아래와 같이 테스트 코드를 작성해서 실행한다.

<br>

```java
@Test
public void auditingTest(){
    Product product = new Product();
    product.setName("펜");
    product.setPrice(1000);
    product.setStock(100);

    Product savedProduct = productRepository.save(product);

    System.out.println("productName : " + savedProduct.getName());
    System.out.println("createdAt : " + savedProduct.getCreatedAt());
}
```

<br>

위 예제에서는 `Product` 엔티티에 생성일자를 입력하지 않은 상태에서 데이터베이스에 저장했다. 10~11번 줄에서 출력된 결과는 다음과 같다.

```
productName : 펜
createdAt : 2022-08-02T22:25:15.504550
```

<br>

직접 일자를 기입하지 않았지만 정상적으로 데이터베이스에는 생성일자가 저장됐으며, 엔티티의 필드를 출력해보면 해당 시간이 출력되는 것을 볼 수 있다.



> **💡 Tip.**
>
> JPA Auditing 기능에는 `@CreatedBy`, `@ModifiedBy` 어노테이션도 존재한다. 누가 엔티티를 생성했고 수정했는지 자동으로 값을 주입하는 기능이다. 이 기능을 사용하려면 `AuditorAware`를 스프링 빈으로 등록할 필요가 있다.



### 정리

이번 장에서는 ORM의 개념을 알아보고 자바의 표준 ORM 기술 스펙인 JPA를 살펴봤다. 데이터를 다루는 영역은 애플리케이션을 개발하면서 가장 중요한 부분이다. 대부분의 로직은 데이터를 가공해서 데이터베이스에 저장하거나 값을 효율적으로 가져오는 부분이 중요하다. 따라서 이번 장에서 다룬 기본기를 잘 다져보고 레퍼런스 뭊서도 살펴보면서 다양한 예제를 스스로 만들어 보는 것이 중요하다.
