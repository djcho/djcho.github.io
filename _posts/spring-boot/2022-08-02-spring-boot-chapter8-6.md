---
title : "[SpringBoot][Chapter8.6] Spring Data JPA 활용 - QueryDSL 적용하기"
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



## QueryDSL 적용하기

앞에서는 `@Query` 어노테이션을 사용해 직접 JPQL의 쿼리를 작성하는 방법을 알아봤다. 메서드의 이름을 기반으로 생성하는 JPQL의 한계는 `@Query` 어노테이션을 통해 대부분 해소할 수 있지만 직접 문자열을 입력하기 때문에 컴파일 시점에 에러를 잡지 못하고 런타임 에러가 발생할 수 있다. 쿼리의 문자열이 잘못된 경우에는 애플리케이션이 실행된 후 로직이 실행되고 나서야 오류를 발견할 수 있다. 이러한 이유로 개발 환경에서는 문제가 없는 것처럼 보이다가 실제 운영 환경에 애플리케이션을 배포하고 나서 오류가 발견되는 리스크를 유발한다.

이 같은 문제를 해결하기 위해 사용되는 것이 QueryDSL 이다. QueryDSL은 문자열이 아니라 코드로 쿼리는 작성할 수 있도록 도와준다.



### QueryDSL

<a href="https://querydsl.com" target="_blank">QueryDSL</a>은 정적 타입을 이용해 SQL과 같은 쿼리를 생성할 수 있도록 지원하는 프레임워크이다. 문자열이나 XML 파일을 통해 쿼리를 작성하는 대신 QueryDSL이 제공하는 플루언트(Fluent) API를 활용해 쿼리를 생성할 수 있다.

<br>

> **💡 Tip.**
>
> QueryDSL에 대하 자세한 내용은 아래 URL에서 확인한다.
>
> - 한글판(4.0.1 버전) : <a href="http://querydsl.com/static/querydsl/4.0.1/reference/ko-KR/html_single/" target="_blank">http://querydsl.com/static/querydsl/4.0.1/reference/ko-KR/html_single/</a>
> - 실습 버전(영문) : <a href="https://querydsl.com/static/querydsl/4.4.0/reference/html/index.html" target="_blank">https://querydsl.com/static/querydsl/4.4.0/reference/html/index.html</a>

<br>



### QueryDSL의 장점

QueryDSL을 사용하면 다음과 같은 장점이 있다.

- IDE가 제공하는 코드 자동 완성 기능을 사용할 수 있다.
- 문법적으로 잘못된 쿼리를 허용하지 않는다. 따라서 정상적으로 활용된 QueryDSL은 문법 오류를 발생시키지 않는다.
- 고정된 SQL 쿼리를 작성하지 않기 때문에 동적으로 쿼리를 생성할 수 있다.
- 코드로 작성하므로 가독성 및 생산성이 향상된다.
- 도메인 타입과 프로퍼티를 안전하게 참조할 수 있다.



### QueryDSL을 사용하기 위한 프로젝트 설정

QueryDSL을 사용하려면 몇 가지 설정이 필요하다. 먼저 `pom.xml`파일에 아래와 같이 의존성을 추가한다.

<br>

```xml
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-apt</artifactId>
    <scope>provided</scope>
</dependency>
<dependency>
    <groupId>com.querydsl</groupId>
    <artifactId>querydsl-jpa</artifactId>
</dependency>
```

<br>

또한 `<plugins>` 태그에 QueryDSL을 사용하기 위한 APT 플러그인을 추가해야 한다. 아래와 같이 플러그인 설정을 추가한다.

<br>

```xml
<plugin>
    <groupId>com.mysema.maven</groupId>
    <artifactId>apt-maven-plugin</artifactId>
    <version>1.1.3</version>
    <executions>
        <execution>
            <goals>
                <goal>process</goal>
            </goals>
            <configuration>
                <outputDirectory>target/generated-sources/java</outputDirectory>
                <processor>com.querydsl.apt.jpa.JPAAnnotationProcessor</processor>
                <options>
                    <querydsl.entityAccessors>true</querydsl.entityAccessors>
                </options>
            </configuration>
        </execution>
    </executions>
</plugin>
```

<br>

12번 줄의 `JPAAnnotationProcessor`는 `@Entity` 어노테이션으로 정의된 엔티티 클래스를 찾아서 쿼리 타입을 생성한다.

<br>

> **💡 Tip. APT란?**
>
> APT(Annotation Processing Tool)는 어노테이션으로 정의된 코드를 기반으로 새로운 코드를 생성하는 기능이다. JDK 1.6부터 도입된 기능이며, 클래스를 컴파일하는 기능도 제공한다.

<br>

이렇게 설정을 마치고 아래와 같이 메이븐이 compile 단계를 클릭해 빌드 작업을 수행한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182161155-59f358f3-39c4-4e71-8739-53a6e7fd7f06.png){: .align-center}

<br>

빌드가 완료되면 위 11번 줄에 작성했던 `outputDirectory`에 설정한 `generated-source` 경로에 아래와 같이 Q도메인 클래스가 생성된 것을 볼 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182162051-ae5f98eb-6de3-4a4f-8d72-90e622ad2fa2.png){: .align-center}

<br>

QueryDSL은 지금까지 작성했던 엔티티 클래스와 Q도메인(QDomain)이라는 쿼리 타입의 클래스를 자체적으로 생성해서 메타데이터로 사용하는데, 이를 통해 SQL과 같은 쿼리를 생성해서 제공한다.

만약 Q도메인 클래스(`QProduct`)가 제대로 생성되지 않았다면 프로젝트 폴더를 마우스 오른쪽 버튼으로 클릭한 후 [Maven] → [Generate Sources and Update Folders]를 선택한다.

또한 코드가 정상적으로 동작하지 않는다면 IDE의 설정을 조정해야 한다. 인텔리제이 IDEA 에서 [Ctrl + Alt + Shift + S]를 클릭하거나 메뉴에서 [File] → [Project Structure]를 차례로 선택해 설정 창을 연 다음 [Modules] 탭을 클릭한다. 그러고 나서 아래와 같이 `generated-sources`를 눌러 [Mark as] 항목에 있는 [Sources]를 눌러 IDE에서 소스파일로 인식할 수 있게 설정한다.

![image](https://user-images.githubusercontent.com/13410737/182163525-14de4330-099e-4897-8079-5d3e520dd672.png){: .align-center}

<br>



### 기본적인 QueryDSL 사용하기

앞의 프로젝트 설정을 마치면 QueryDSL을 사용할 준비가 끝났다. 우선 테스트 코드로 기본적인 QueryDSL 사용법을 알아보겠다. 아래와 같이 테스트 코드를 작성해서 QueryDSL의 동작을 확인할 수 있다.

<br>

```java
@PersistenceContext
EntityManager entityManager;

@Test
void queryDslTest() {
    JPAQuery<Product> query = new JPAQuery(entityManager);
    QProduct qProduct = QProduct.product;

    List<Product> productList = query
        .from(qProduct)
        .where(qProduct.name.eq("펜"))
        .orderBy(qProduct.price.asc())
        .fetch();

    for(Product product : productList) {
        System.out.println("------------------");
        System.out.println();
        System.out.println("Product Number : " + product.getNumber());
        System.out.println("Product Name : " + product.getName());
        System.out.println("Product Price : " + product.getPrice());
        System.out.println("Product Stock : " + product.getStock());
        System.out.println();
        System.out.println("------------------");
    }
}
```

<br>

위 코드는 QueryDSL에 의해 생성된 Q도메인 클래스를 활용하는 코드이다. 다만 Q도메인 클래스와 대응되는 테스트 클래스가 없으므로 엔티티 클래스에 대응되는 리포지토리의 테스트 클래스(`ProductRepositoryTest`)에 포함해도 무관하다.

그럼 위 코드를 자세히 살펴보겠다. QueryDSL을 사용하기 위해서는 6번 줄과 같이 `JPAQuery`객체를 사용한다. `JPAQuery`는 엔티티 매니저(EntityManager)를 활용해 생성한다. 이렇게 생성된 `JPAQuery`는 9~13번 줄과 같이 빌더 형식으로 쿼리를 작성한다. 빌더 메서드에서 확인할 수 있듯이 SQL 쿼리에서 사용되는 키뭐으도 메서드가 구성돼 있다. 그렇게 때문에 메서드를 활용해 좀 더 손쉽게 코드를 작성할 수 있다.

`List` 타입으로 값을 리턴받기 위해서는 13번 줄과 같이 `fetch()` 메서드를 사용해야 하는데, 만약 4.0.1 이전 버전의 QueryDSL을 설정한다면 `list()` 메서드를 사용해야 한다. 반환 메서드로 사용할 수 있는 메서드는 다음과 같다.

- `List<T> fetch()` : 조회 결과를 리스트로 반환한다.
- `T fetchOne` : 단 건의 조회 결과를 반환한다.
- `T fetchFirst()` : 여러 건의 조회 결과 중 1건을 반환한다. 내부 로직을 살펴보면 '`.limit(1).fetchOne()`' 으로 구현돼 있다.
- `Long fetchCount()` : 조회 결과의 개수를 반환한다.
- `QeuryResult<T> fetchResults()` : 조회 결과 리스트와 개수를 포함한 `QueryResults`를 반환한다.

<br>

위 코드 처럼 `JPAQuery` 객체를 사용해서 코드를 작성하는 방법 외 다른 방법도 있다. 아래는 `JPAQueryFactory`를 활용해서 작성한 코드이다.

<br>

```java
@Test
void queryDslTest2() {
    JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(entityManager);
    QProduct qProduct = QProduct.product;

    List<Product> productList = jpaQueryFactory.selectFrom(qProduct)
        .where(qProduct.name.eq("펜"))
        .orderBy(qProduct.price.asc())
        .fetch();

    for(Product product : productList) {
        System.out.println("------------------");
        System.out.println();
        System.out.println("Product Number : " + product.getNumber());
        System.out.println("Product Name : " + product.getName());
        System.out.println("Product Price : " + product.getPrice());
        System.out.println("Product Stock : " + product.getStock());
        System.out.println();
        System.out.println("------------------");
    }
}
```

<br>

위 코드에서는 `JPAQueryFactory`를 활용해 쿼리를 작성했다. `JPAQuery`를 사용했을 때와 달리 `JPAQueryFactory`에서는 `select` 절부터 작성 가능하다.

만약 전체 칼럼을 조회하지 않고 일부만 조회하고 싶다면 아래와 같이 `selectFrom()`이 아닌 `select()`와 `from()` 메서드를 구분해서 사용하면 된다.

<br>

```java
@Test
void queryDslTest3() {
    JPAQueryFactory jpaQueryFactory = new JPAQueryFactory(entityManager);
    QProduct qProduct = QProduct.product;

    List<String> productList = jpaQueryFactory
        .select(qProduct.name)
        .from(qProduct)
        .where(qProduct.name.eq("펜"))
        .orderBy(qProduct.price.asc())
        .fetch();

    for(String product : productList) {
        System.out.println("------------------");
        System.out.println("Product Name : " + product);
        System.out.println("------------------");
    }

    List<Tuple> tupleList = jpaQueryFactory
        .select(qProduct.name, qProduct.price)
        .from(qProduct)
        .where(qProduct.name.eq("펜"))
        .orderBy(qProduct.price.asc())
        .fetch();

    for(Tuple product : tupleList) {
        System.out.println("------------------");
        System.out.println("Product Name : " + product.get(qProduct.name));
        System.out.println("Product price : " + product.get(qProduct.price));
        System.out.println("------------------");
    }
}
```

<br>

위 코드의 3~17번 줄은 `select` 대상이 하나인 경우이다. 만약 조회 대상이 여러 개일 경우에는 20번 줄과 같이 쉽표(,)로 구분해서 작성하면 되고, 리턴 타입을 `List<String>` 타입이 아닌 `List<Tuple>` 타입으로 지정한다.

지금까지 테스트 코드를 활용해 QueryDSL의 기본 사용법을 소개했다. 이제 QueryDSL을 실제 비즈니스 로직에서 활용할 수 있게 설정해보겠다. 먼저 아래와 같이 컨피그 클래스를 생성한다.

<br>

```java
@Configuration
public class QueryDSLConfiguration {
    
    @PersistenceContext
    EntityManager entityManager;
    
    @Bean
    public JPAQueryFactory jpaQueryFactory(){
        return new JPAQueryFactory(entityManager);
    }
}
```

<br>

위와 같이 `JPAQueryFactory` 객체를 `@Bean` 객체로 등록해두면 앞에서 작성한 예제 처럼 매번 `JPAQueryFactory`를 초기화 하지 않고 스프링 컨테이너에서 가져다 쓸 수 있다. 이렇게 생성한 컨피그 클래스는 아래와 같이 사용할 수 있다.

<br>

```java
@Autowired
JPAQueryFactory jpaQueryFactory;

@Test
void queryDslTest4() {
    QProduct qProduct = QProduct.product;

    List<String> productList = jpaQueryFactory
        .select(qProduct.name)
        .from(qProduct)
        .where(qProduct.name.eq("펜"))
        .orderBy(qProduct.price.asc())
        .fetch();

    for(String product : productList) {
        System.out.println("------------------");
        System.out.println("Product Name : " + product);
        System.out.println("------------------");
    }
}
```

<br>



### QuerydslPredicateExecutor, QuerydslRepositorySupport 활용

스프링 데이터 JPA 에서는 QueryDSL을 더욱 편하게 사용할 수 있게 `QuerydslPredicateExecutor` 인터페이스와 `QuerydslRepositorySupport` 클래스를 제공한다. 이번 절에서는 이 두 클래스의 활용법을 살펴보겠다.



#### QuerydslPredicateExecutor

`QuerydslPredicateExecutor`는 `JpaRepository`와 함께 리포지토리에서 QueryDSL을 사용할 수 있게 인터페이스를 제공한다. 아래와 같이 리포지토리를 보자. 기존 리포지토리를 그대로 이용해도 되지만 예제를 구분하기 위해 `QProductRepository`라는 이름의 클래스를 생성했다.

<br>

```java
public interface QProductRepository extends JpaRepository<Product, Long>,
        QuerydslPredicateExecutor<Product> {
}
```

<br>

위 코드는 `QuerydslPredicateExecutor`를 상속받도록 설정한 `Product` 엔티티에 대한 리포지토리이다. ` QuerydslPredicateExecutor` 인터페이스를 보면 아래와 같은 다양한 메서드를 제공한다.

<br>

```java
Optional<T> findOne(Predicate predicate);

Iterable<T> findAll(Predicate predicate);

Iterable<T> findAll(Predicate predicate, Sort sort);

Iterable<T> findAll(Predicate predicate, OrderSpecifier<?>... orders);

Iterable<T> findAll(OrderSpecifier<?>... orders);

Page<T> findAll(Predicate predicate, Pageable pageable);

long count(Predicate predicate);

boolean exists(Predicate predicate);
```

<br>

보다시피 `QuerydslPredicateExecutor` 인터페이스의 메서드는 대부분 `Predicate` 타입을 매개변수로 받는다. `Predicate`는 표현식을 작성할 수 있게 QueryDSL에서 제공하는 인터페이스이다. `QProductRepositiory`에 대한 실습 코드를 작성하기 위해 `test` 디렉터리에 다음과 같이 `QProductRepositoryTest` 클래스를 생성한다.

<br>

```java
@SpringBootTest
public class QProductRepositoryTest {
    
    @Autowired
    QProductRepository qProductRepository;
    
}
```

<br>

`Predicate`를 이용해 `findOne()` 메서드를 호출하는 방법은 아래와 같다.

<br>

```java
@Test
public void queryDSLTest1() {
    Predicate predicate = QProduct.product.name.containsIgnoreCase("펜")
        .and(QProduct.product.price.between(1000, 2000));

    Optional<Product> foundProduct = qProductRepository.findOne(predicate);

    if(foundProduct.isPresent()) {
        Product product = foundProduct.get();
        System.out.println(product.getNumber());
        System.out.println(product.getName());
        System.out.println(product.getPrice());
        System.out.println(product.getStock());
    }
}
```

<br>

`Predicate`는 간단하게 표현식으로 정의하는 쿼리로 생각하면 된다. 앞의 예제에서는 `Predicate`를 명시적으로 정의하고 사용헸지만 아래와 같이 서술부만 가져다 사용할 수도 있다.

<br>

```java
@Test
public void queryDSLTest2() {
    QProduct qProduct = QProduct.product;

    Iterable<Product> productList = qProductRepository.findAll(
        qProduct.name.contains("펜")
        .and(qProduct.price.between(550, 1500))
    );

    for (Product product : productList) {
        System.out.println(product.getNumber());
        System.out.println(product.getName());
        System.out.println(product.getPrice());
        System.out.println(product.getStock());
    }
}
```

<br>

지금까지 간단하게 `QuerydslPredicateExecutor`의 사용법을 알아봤다. `QuerydslPredicateExecutor`를 활용하면 더욱 편하게 QueryDSL을 사용할 수 있지만 `join`이나 `fetch` 기능은 사용할 수 없다는 단점이 있다.

<br>

> **💡 Tip.**
>
> `QuerydslPredicateExecutor`의 공식 문서는 다음 URL을 참고한다.
>
> - <a href="https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/querydsl/QuerydslPredicateExecutor.html" target="_blank">https://docs.spring.io/spring-data/commons/docs/current/api/org/springframework/data/querydsl/QuerydslPredicateExecutor.html</a>



#### QuerydslRepositorySupport 추상 클래스 사용하기

`QuerydslRepositorySupport` 클래스 역시 QueryDSL 라이브러리를 사용하는 데 유용한 기능을 제공한다. 이번 절에서는 `QuerydslRepositorySupport` 클래스를 사용하는 여러 방법 중 가장 일반적인 사용법을 소개하겠다.

가장 보편적으로 사용하는 방식은 `CustomRepository`를 활용해 리포지토리를 구현하는 방식이다. 지금까지 예로 든 `Product` 엔티티를 활용하기 위한 객체들의 상속 구조를 살펴보면 아래 그림과 같다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182179595-20b0e640-cc43-4341-b880-789c38088cdf.png){: .align-center}

<br>

여기서 `JpaRepository`와 `QuerydslRepositorySupport` 는 Spring Data JPA에서 제공하는 인터페이스와 클래스이다. 나머지 `ProductRepository`와 `ProductRepositoryCustom`, `ProductRepositoryCustomImpl`은 직접 구현해야 한다. 간단하게 구조를 설명하자면 다음과 같다.

- 먼저 앞에서 사용했던 방식처럼  `JpaRepository`를 상속받는 `ProductRepository`를 생성한다.
- 이때 직접 구현한 쿼리를 사용하기 위해서는 `JpaRepository`를 상속받지 않는 리포지토리 인터페이스인 `ProductRepositoryCustom`을 생성한다. 이 인터페이스에 정의하고자 하는 기능들을 메서드로 정의한다.
- `ProductRepositoryCustom`에서 정의한 메서드를 사용하기 위해 `ProductRepository`에서 `ProductRepositoryCustom`을 상속받는다.
-  `ProductRepositoryCustom`에서 정의된 메서드를 기반으로 실제 쿼리 작성을 하기 위해 구현체인 `ProductRepositoryCustomImpl` 클래스를 생성한다.
- `ProductRepositoryCustomImpl` 클래스에서는 다양한 방법으로 쿼리를 구현할 수 있지만 QueryDSL을 사용하기 위해 `QuerydslRepositorySupport`를 상속받는다.

위와 같이 구성하면 DAO나 서비스에서 리포지토리에 접근하기 위해 `ProductRepository`를 사용한다. `ProductRepository`를 활용함으로써 QueryDSL의 기능도 사용할 수 있게 된다.

그럼 이전에 만들었던 인터페이스의 이름과 겹이지 않게 아래와 같이 `repository` 패키지 안에 `support` 패키지를 만들어서 그 안에서 구현하겠다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182180339-4eef4610-64a9-402a-9324-ee3641f39c59.png){: .align-center}

<br>

위와 같이 `ProductRepository`, `ProductRepositoryCustom`, `ProductRepositoryCustomImpl`로 총 3개의 인터페이스와 클래스를 생성한다. 그럼 `ProductRepositoryCustom` 인터페이스 먼저 살펴보겠다.

<br>

```java
public interface ProductRepositoryCustom {
    
    List<Product> findByName(String name);
    
}
```

<br>

위와 같이 인터페이스를 생성하고 쿼리로 구현하고자 하는 메서드를 정의하는 작업을 수행한다. 여기서는 간단하게 `findByName()`을 정의하고 사용해보겠다. `ProductRepositoryCustom` 인터페이스의 구현체인 `ProductRepositoryCustomImpl` 클래스를 아래와 같이 작성한다.

<br>

```java
@Component
public class ProductRepositoryCustomImpl extends QuerydslRepositorySupport implements
        ProductRepositoryCustom {

    public ProductRepositoryCustomImpl() {
        super(Product.class);
    }

    @Override
    public List<Product> findByName(String name) {
        QProduct product = QProduct.product;

        List<Product> productList = from(product)
                .where(product.name.eq(name))
                .select(product)
                .fetch();

        return productList;
    }
}
```

<br>

`ProductRepositoryCustomImpl` 클래스에서는 QueryDSL을 사용하기 위해 `QuerydslRepositorySupport`를 상속받고 앞서 생성한 `ProductRepositoryCustom` 인터페이스를 구현한다. `QuerydslRepositorySupport`를 상속받으면 5~7번 줄과 같이 생성자를 통해 도메인 클래스를 부모 클래스에 전달해야 한다.

그리고 9~19번 줄과 같이 인터페이스에 정의한 메서드를 구현한다. 이 과정에서 Q도메인 클래스인 `QProduct`를 사용해 `QuerydslRepositorySupport`가 제공하는 기능을 사용한다. 대표적인 예로 13번 줄의 `from()` 메서드가 있다. `from()` 메서드는 이름에서 유추할 수 있듯이 어떤 도메인에 접근할 것인지 지정하는 역할을 수행하고 JPAQuery를 리턴한다. 이 `from()` 메서드는 11번줄에서 생성한 `QProduct`의 이름을 매개변수로 사용한다. 그리고 차례대로 쿼리 키워드에 매핑되는 QueryDSL의 메서드를 사용해 쿼리를 생성하면 된다.

여기서 기존에 `Product` 엔티티 클래스와 매핑해서 사용하던 `ProductRepository`가 있다면 `ProductRepositoryCustom`을 상속받아 사용할 수 있다. 다만 이번 예제에서는 예제를 구분하기 위해 별도로 생성했다 .새로 생성한 `ProductRepository`는 아래와 같다.

<br>

```java
@Repository("productRepositorySupport")
public interface ProductRepository extends JpaRepository<Product, Long>, ProductRepositoryCustom {
    
}
```

<br>

기존에 리포지토리를 생성하는 것과 동일하게 `JpaRepository`를 상속받아 구성하면 된다. 이미 이전 예제에서 `ProductRepository`라는 이름이 이미 사용되고 있기 때문에 빈 생성 시 충돌이 발생해서 1번 줄과 같이 별도로 빈 이름을 설정해야 한다. 기존의 리포지토리를 그대로 사용한다면 따로 작성하지 않아도 무방하다.

이 코드를 사용할 때는 `ProductRepository`만 이용하면 된다. 기본적으로 `JpaRepository`에서 제공하는 메서드도 사용할 수 있고, 별도로 `ProductRepositoryCustom` 인터페이스에서 정의한 메서드도 구현체를 통해 사용할 수 있다. 기본적인 CURD 사용법은 앞에서 다뤘기 때문에 `ProductRepositoryCustom` 인터페이스에서 정의한 `findByName()` 메서드만 아래와 같이 호출해 보겠다. 이번 테스트 코드를 작성하기 위해 `test/com.springboot.advanced_jpa/data/repository` 패키지 내에 `support` 패키지를 생성하고 `ProductRepositoryTest` 클래스를 생성한다.

<br>

```java
@Autowired
ProductRepository productRepository;

@Test
void findByNameTest() {
    List<Product> productList = productRepository.findByName("펜");

    for(Product product : productList){
        System.out.println(product.getNumber());
        System.out.println(product.getName());
        System.out.println(product.getPrice());
        System.out.println(product.getStock());
    }
}
```

<br>

리포지토리를 구성하면서 모든 로직을 구현했기 때문에 `findByName()` 메서드를 사용할 때는 위와 같이 간단하게 구현해서 사용할 수 있다.

<br>

> **💡 Tip.**
>
> `QuerydslRepositorySupport`의 공식 문서는 다음 URL을 참고한다.
>
> - <a href="https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/support/QuerydslRepositorySupport.html" target="_blank">https://docs.spring.io/spring-data/jpa/docs/current/api/org/springframework/data/jpa/repository/support/QuerydslRepositorySupport.html</a>

