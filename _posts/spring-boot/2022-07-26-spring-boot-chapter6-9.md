---
title : "[SpringBoot][Chapter6.9] DB연동 - DAO 설계"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Jpa]
toc: true
toc_sticky : true
published : true
date : 2022-07-26
last_modified_at : 2022-07-26
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## DAO 설계

DAO(Data Access Object)는 데이터베이스에 접근하기 위한 로직을 관리하기 위한 객체이다. 비즈니스 로직의 동작 과정에서 데이터를 조작하는 기능은 DAO 객체가 수행한다. 다만 스프링 데이터 JPA에서 DAO의 개념은 리포지토리가 대체하고 있다.

규모가 작은 서비스에서는 DAO를 별도로 설계하지 않고 바로 서비스 레이어에서 데이터베이스에 접근해서 구현하기도 하지만, 이번 장에서는 DAO를 서비스 레이어와 리포지토리의 중간 계층을 구성하는 역할로 사용할 예정이다. 이 책에서는 간단한 데이터베이스 호출만 다루고 있기 때문에 큰 의미는 없지만 실제로 업무에 필요한 비즈니스 로직을 개발하다 보면 데이터를 다루는 중간 계층을 두는 것이 유지보수 측면에서 용이한 경우가 많다. 물론 서비스 레이어에서 리포지토리의 메서드를 호출하고 그 결과에 대해 처리할 수 있지만 비즈니스 로직을 수행하는 과정에서 데이터베이스에 관한 작업을 처리하는 것은 기능을 분리하고 관리하기에 좋은 코드라고 보기 어렵다.

객체지향적인 설계에서는 서비스와 비즈니스 레이러를 분리해서 서비스 레이어에서는 서비스 로직을 수행하고 비즈니스 레이어에서는 비즈니스 로직을 수행해야 한다는 의견도 많다. 그러나 이번 장에서는 이런 관점은 간단하게만 다루고 서비스 객체가 비즈니스 로직까지 포함하는 방향으로 진행하겠다. 도메인(엔티티) 객체를 중심으로 다뤄지는 로직은 비즈니스 로직으로 볼 수 있다.

> **DAO vs. 리포지토리**
>
> DAO와 리포지토리는 역할이 비슷하다. 그렇기 때문에 아직도 DAO와 리포지토리를 비교하거나 어떤 차이가 있는지 논쟁하는 경우가 많다. 실제로 리포지토리는 Spring Data JPA에서 제공하는 기능이기 때문에 기존의 스프링 프레임워크나 스프링 MVC의 사용자는 리포지토리라는 개념을 사용하지 않고 DAO 객체로 데이터베이스에 접근했다. 이런 측면에서 각 컴포넌트의 역할을 고민하는 시간을 가져보면 좋을 것 같다.



### DAO 클래스 생성

DAO 클래스는 일반적으로 '인터페이스-구현체' 구성으로 생성한다. DAO 클래스는 의존성 결합을 낮추기 위한 디자인 패턴이며, 서비스 레이어에 DA
O 객체를 주입받을 때 인터페이스를 선언하는 방식으로 구성할 수 있다. 아래와 같이 `data.dao.impl` 구조로 패키지를 생성한 후 `dao` 패키지를 생성한 후 `impl`패키지에 각각 `ProductDAO` 인터페이스와 `ProductDAOImpl` 클래스를 생성한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181011610-445fdf2f-8ae6-4f65-96a4-e9346be354a6.png){: .align-center}

<br>

다음으로 인터페이스를 구성한다. 우선 기본적인 CRUD를 다루기 위해 아래와 같이 인터페이스에 메서드를 정의한다.

<br>

```java
package com.springboot.jpa.data.dao;

import com.springboot.jpa.data.entity.Product;

public interface ProductDAO {

    Product insertProduct(Product product);

    Product selectProduct(Long number);

    Product updateProductName(Long number, String name) throws Exception;

    void deleteProduct(Long number) throws Exception;

}
```

<br>

일반적으로 데이터베이스에 접근하는 메서드는 리턴 값으로 데이터 객체를 전달한다. 이때 데이터 객체를 엔티티 객체로 전달할지, DTO 객체로 전달할지에 대해서는 개발자마다 의견이 분분하다. 일반적인 설계 원칙에서 엔티티 객체는 데이터베이스에 접근하는 계층에서만 사용하도록 정의한다. 다른 계층으로 데이터를 전달할 때는 DTO 객체를 사용한다. 그러나 이 부분은 회사나 부서마다 경해 차이가 있으므로 각자 정해진 원칙에 따라 진행하는 것이 좋다.

인터페이스의 설계를 마쳤다면 해당 인터페이스의 구현체를 만들어야 한다. 우선 기능 구현을 위해 아래와 같은 구현체 클래스를 작성한다. (import문 생략)

<br>

```java
package com.springboot.jpa.data.dao.impl;

@Component
public class ProductDAOImpl implements ProductDAO {

    private final ProductRepository productRepository;

    @Autowired
    public ProductDAOImpl(ProductRepository productRepository){
        this.productRepository = productRepository;
    }

    @Override
    public Product insertProduct(Product product) {
        return null;
    }

    @Override
    public Product selectProduct(Long number) {
        return null;
    }

    @Override
    public Product updateProductName(Long number, String name) throws Exception {
        return null;
    }

    @Override
    public void deleteProduct(Long number) throws Exception {

    }
}
```

<br>
위 `ProductDAOImpl` 클래스를 스프링이 관리하는 빈으로 등록하려면 3번 줄과 같이 `@Component` 또는 `@Service` 어노테이션으로 지정해야 한다. 빈으로 등록된 객체는 다른 클래스가 인터페이스를 가지고 의존성을 주입받을 때 이 구현체를 찾아 주입하게 된다.

마찬가지로 DAO 객체에서도 데이터베이스에 접근하기 위해 리포지토리 인터페이스를 사용해 의존성 주입을 받아야 한다. 6~11번 줄과 같이 리포지토리를 정의하고 생성자를 통해 의존성 주입을 받으면 된다.

이제 인터페이스에 정의한 메서드를 구현해야 한다. 먼저 13~15번 줄에 정의돼 있는 `insertProduct()` 메서드를 구현하겠다. 이 메서드에서는 `Product` 엔티티를 데이터베이스에 저장하는 기능을 수행하며, 아래와 같이 작성할 수 있다.

<br>

```java
@Override
public Product insertProduct(Product product) {
    Product savedProduct = productRepository.save(product);
    return savedProduct;
}
```

<br>

3번 줄과 5번 줄을 합쳐서 좀 더 간결하게 작성할 수도 있지만 예외 처리를 하거나 코드 사이에 로그를 삽입해야 할 수 있게 때문에 서로 구분해서 작성했다.이전에서 리포지토리를 생성할 때 인터페이스에서 따로 메서드를 구현하지 않아도 JPA에서 기본 메서드를 제공하므로 3번 줄과 같이 `save` 메서드를 활용할 수 있다.

다음으로 조회 메서드를 작성한다. 조회 메서드에 해당하는 `selectProduct()` 메서드는 아래와 같다.

<br>

```java
@Override
public Product selectProduct(Long number) {
    Product selectProduct = productRepository.getById(number);
    return selectProduct;
}
```

<br>

`selectProduct()` 메서드가 사용한 리포지토리의 메서드는 `getById()`이다. 리포지토리에서는 단건 조회를 위한 기본 메서드로 두 가지를 제공하는데, 바로 `getById()` 메서드와 `findById()` 메서드이다. 두 메서드는 조회한다는 기능 측면에서는 동일하지만 세부 내용이 다르다. 각 메서드의 자세한 설명은 다음과 같다.

- **`getById()`**

  - 내부적으로 `EntityManager` 의 `getReference()` 메서드를 호출한다. `getReference()` 메서드를 호출하면 프락시 객체를 리턴한다. 실제 쿼리는 프락시 객체를 통해 최초로 데이터에 접근하는 시점에 실행된다. 이때 데이터가 존재하지 않는 경우에는 `EntityNotFoundException`이 발생한다. `JpaRepository`의 실제 구현체인 `SimpleJpaRepository`의 `getById()` 메서드는 아래와 같다.

    ```java
    @Override
    public T getById(ID id) {
        Assert.notNull(id, ID_MUST_NOT_BE_NULL);
        return em.getReference(getDomainClass(), id);
    }
    ```

- **`findById()`**

  - 내부적으로 `EntityManager`의 `find()` 메서드를 호출한다. 이 메서드는 영속성 컨텍스트의 캐시에서 값을 조회한 후 영속성 컨텍스트에 걊이 존재하지 않는다면 실제 데이터베이스에서 데이터를 조회한다. 리터턴 값으로 `Optional` 객체를 전달한다. `SimpleJpaRepository`의 `findById()` 메서드는 아래와 같다.

    ```java
    @Override
    public Optional<T> findById(ID id) {
        Assert.notNull(id, ID_MUST_NOT_BE_NULL);
    
        Class<T> domainType = getDomainClass();
    
        if(metadata == null) {
          return Optional.ofNullable(em.find(domainType, id));
        }
    
        LockModeType type = metadata.getLockModeType();
    
        Map<String, Object> hints = new HashMap<>();
        getQueryHints().withFetchGraphs(em).forEach(hints::put);
    
        return Optional.ofNullable(type == null ? em.find(domainType, id, hints) : em.find(domainType, id, type, hints));
    }
    ```

<br>

조회 기능을 구현하기 위해서는 어떤 메서드를 사용하더라도 무관하다. 비즈니스 로직을 구현하는 데 적합한 방식을 선정해 활용하면 된다.

다음으로 업데이트 메서드를 구현한다. 여기서는 `Product` 데이터의 상품명을 업데이트하는 기능을 구현한다. 업데이트 메서드에 해당하는 `updateProductName()` 메서드는 아래와 같이 작성할 수 있다.

<br>

```java
@Override
public Product updateProductName(Long number, String name) throws Exception {
    Optional<Product> selectedProduct = productRepository.findById(number);

    Product updatedProduct;
    if(selectedProduct.isPresent()) {
        Product product = selectedProduct.get();

        product.setName(name);
        product.setUpdatedAt(LocalDateTime.now());

        updatedProduct = productRepository.save(product);
    } else {
        throw new Exception();
    }

    return updatedProduct;
}
```

<br>

JPA에서 데이터의 값을 변경할 때는 다른 메서드와는 다른 점이 있다. JPA는 값을 갱신할 때 `update`라는 키워드를 사용하지 않는다. 여기서는 영속성 컨텍스트를 활용해 값을 갱신하는데, `find()` 메서드를 통해 데이터베이스에서 값을 가져오면 가져온 객체가 영속성 컨텍스트에 추가된다. 영속성 컨텍스트가 유지되는 상황에서 객체의 값을 변경하고 다시 `save()` 를 실행하면 JPA에서는 더티 체크(DirtyCheck)라고 하는 변경 감지를 수행한다. `SimpleJpaRepository`에 구현돼 있는 `save()` 메서드를 살펴보면 아래와 같다.

<br>

```java
@Transactional
@Override
public <S extends T> S save(S entity) {
    Assert.notNull(entity, "Entity must not be null.");
  
    if(entotyInformation.isNew(entity)) {
        em.persist(entity);
        return entity;
    } else {
      return em.merge(entity);
    }
}
```

<br>

위 코드의 1번 줄에는 `@Transactional` 어노테이션이 선언돼 있다. 이 어노테이션이 지정돼 있으면 메서드 내 작업을 마칠 경우 자동으로 `flush()` 메서드를 실행한다. 이 과정에서 변경이 감지되면 대상 객체에 해당하는 데이터베이스의 레코드를 업데이트하는 쿼리가 실행된다.

다음으로 삭제 메서드를 구현한다. 삭제 메서드에 해당하는 `deleteProduct()` 메서드는 아래와 같다.

<br>

```java
@Override
public void deleteProduct(Long number) throws Exception {
    Optional<Product> selectedProduct = productRepository.findById(number);

    if(selectedProduct.isPresent()){
        Product product = selectedProduct.get();

        productRepository.delete(product);
    } else {
        throw new Exception();
    }
}
```

<br>

데이터베이스의 레코드를 삭제하기 위해서는 삭제하고자 하는 레코드와 매핑된 영속 객체를 영속성 컨텍스트에 가져와야 한다. `deleteProduct()` 메서드는 `findById()` 메서드를 통해 객체를 가져오는 작업을 수행하고 `delete()` 메서드를 통해 해강 객체를 삭제하게끔 삭제 요청을 한다. `SimpleJpaRepository`의 `delete()` 메서드는 아래와 같다.

<br>

```java
@Override
@Transactional
@SuppressWarnings("unchecked")
public void delete(T entity) {
    
    Assert.notNull(entity, "Entity must not be null!");
    
    if (entityInformation.isNew(entity)) {
        return;
    }
    
    Class<?> type = ProxyUtils.getUserClass(entity);
    
    T existing = (T) em.find(type, entityInformation.getId(entity));
    
    // if the entity to be deleted doesn't exist, delete is a NOOP
    if (existing == null) {
        return;
    }
    
    em.remove(em.contains(entity) ? entity : em.merge(entity));
}
```

<br>

`SimpleJpaRepository`의 `delete()` 메서드는 21번 줄에서 `delete()` 메서드로 전달받은 엔티티가 영속성 컨텍스트에 있는지 파악하고, 해당 엔티티를 영속성 컨텍스트에 영속화하는 작업을 거쳐 데이터베이스의 레코드와 매핑한다. 그렇게 매핑된 영속 객체를 대상으로 삭제 요청을 수행하는 메서드를 실행해 작업을 마치고 커밋(commit) 단계에서 삭제를 진행한다.
