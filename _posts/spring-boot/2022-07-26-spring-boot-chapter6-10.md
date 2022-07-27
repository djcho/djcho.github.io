---
title : "[SpringBoot][Chapter6.10] DB연동 - DAO 연동을 위한 컨트롤러와 서비스 설계"
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



## DAO 연동을 위한 컨트롤러와 서비스 설계

앞에서 설계한 구성 요소들을 클라이언트의 요청과 연결하려면 컨트롤러와 서비스를 생성해야 한다.

이를 위해 먼저 DAO의 메서드를 호출하고 그 외 비즈니스 로직을 수행하는 서비스 레이어를 생성한 후 컨트롤러를 생성하겠다.



### 서비스 클래스 만들기

서비스 레이어에서는 도메인 모델(Domain Model)을 활용해 애플리케이션에서 제공하는 핵심 기능을 제공한다. 여기서 말하는 핵심 기능을 구현하려면 세부 기능을 정의해야 한다. 세부 기능이 모여 핵심 기능을 구현하기 때문이다. 이러한 모든 로직을 서비스 레이어에서 포함하기란 쉽지 않은 일이다. 이 같은 아키텍처의 한계를 극복하기 위해 이키텍처를 서비스 로직과 비즈니스 로직으로 분리하기도 한다. 도메인을 활용한 세부 기능들을 비즈니스 레이어의 로직에서 구현하고, 서비스 레이어에서는 기능들을 종합해서 핵심 기능을 전달하도록 구성하는 경우가 대표적이다.

다만 이 책의 목적은 과도한 기능 구현보다는 어떻게 프로젝트를 구성하고 스프링 부트의 기능을 온전히 사용할 수 있는지를 고민하는 것이므로 서비스 레이어에서 비즈니스 로직을 처리하는 아키텍처로 진행한다.

서비스 객체는 DAO와 마찬가지로 추상화해서 구성한다. 아래와 같이 `service` 패키지와 클래스, 인터페이스를 구성한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181048955-5f248698-79e2-4e03-9502-2ddb1a81e255.png){: .align-center}

<br>

서비스 인터페이스를 작성하기 전에 필요한 DTO 클래스를 생성하겠다. `data` 패키지 안에 `dto` 패키지를 생성하고 `ProductDto`와 `ProductResponseDto` 클래스를 생성한다.

<br>

```java
public class ProductDto {
    
    private String name;
    private int price;
    private int stock;

    public ProductDto(String name, int price, int stock) {
        this.name = name;
        this.price = price;
        this.stock = stock;
    }

    public String getName(){
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public int getStock() {
        return stock;
    }

    public void setStock(int stock) {
        this.stock = stock;
    }
}
```

<br>

```java
public class ProductResponseDto {

    private Long number;
    private String name;
    private int price;
    private int stock;

    public ProductResponseDto() {}

    public ProductResponseDto(Long number, String name, int price, int stock) {
        this.number = number;
        this.name = name;
        this.price = price;
        this.stock = stock;
    }

    public Long getNumber() {
        return number;
    }

    public void setNumber(Long number){
        this.number = number;
    }

    public void setName(String name){
        this.name = name;
    }

    public int getPrice(){
        return price;
    }

    public void setPrice(int price){
        this.price = price;
    }

    public int getStock(){
        return stock;
    }

    public void setStock(int stock){
        this.stock = stock;
    }
}
```

<br>

필요에 따라 빌더 메서드와 `hashCode/equals` 메서드로 추가할 수 있다.



> **Tip. 빌더 메서드**
>
> 빌더 메서드는 빌더(Builder) 패턴을 따르는 메서드이다. 데이터 클래스를 사용할 때 생성자로 초기화할 경우 모든 필드에 값을 넣거나 null을 명시적으로 사용해야 한다. 이러한 단점을 보완하기 위해 나온 패턴이 빌더 패턴이며, 이 패턴을 이용하면 필요한 데이터만 설정할 수 있어 유연성을 확보할 수 있다. 빌더 패턴을 따르는 빌더 메서드는 다음과 같은 방법으로 작성할 수 있다.
>
> ```java
> public class ProductResponseDto {
>     
>     private Long number;
>     private String name;
>     private int price;
>     private int stock;
>     
>     public static ProductResponseDtoBuilder builder() {
>         return new ProductResponseDtoBuilder();
>     }
>     
>     public static class ProductResponseDtoBuilder {
>         private Long number;
>         private String name;
>         private int price;
>         private int stock;
>         
>         ProductResponseDtoBuilder(){
>         }
>         
>         public ProductResponseDtoBuilder number(Long number) {
>             this.number = number;
>             return this;
>         }
>         
>         public ProductResponseDtoBuilder name(String name) {
>             this.name = name;
>             return this;
>         }
>         
>         public ProductResponseDtoBuilder price(int price) {
>             this.price = price;
>             return this;
>         }
>         
>         public ProductResponseDtoBuilder stock(int stock) {
>             this.stock = stock;
>             return this;
>         }
>         
>         public ProductResponseDto build() {
>             return new ProductResponseDto(number, name, price, stock);
>         }
>         
>         public String toString() {
>             return "ProductResponseDto.ProductResponseDtoBuilder(number=" + this.number + ", name=" + this.name + ", price =" + this.price + ", stock=" + this.stock + ")";
>         }
>     }
> }
> ```

<br>

그리고 서비스 인터페이스를 작성한다. 기본적인 CRUD의 기능을 호출하기 위해 간단한 메서드를 정의하겠다. 아래와 같이 코드를 작성한다.

<br>

```java
public interface ProductService {
    
    ProductResponseDto getProduct(Long number);
    
    ProductResponseDto saveProduct(ProductDto productDto);
    
    ProductResponseDto changeProductName(Long number, String name) throws  Exception;
    
    void deleteProduct(Long number) throws Exception;
}
```

<br>

서비스에서는 클라이언트가 요청한 데이터를 적절하게 가공헤서 컨트롤러에게 넘기는 역할을 한다. 이 과정에서 여러 메서드를 사용하는데, 지금은 간단하게 CRUD만을 구현하기 때문에 코드가 단순해 보일 수 있다.

위 코드를 보면 리턴 타입이 DTO 객체인 것을 볼 수 있다. DAO 객체에서 엔티티 타입을 사용하는 것을 고려하면 서비스 레이어에서 DTO 객체와 엔티티 객체를 각 레이어에 변환해서 전달하는 역할도 수행한다고 볼 수 있다. 다만 이 부분은 실무 환경에서 내부적으로 어떻게 정의하느냐에 따라 달라질 수 있다.

정리해보면 데이터베이스와 밀접한 관련이 있는 데이터 액세스 레이어까지는 엔티티 객체를 사용하고, 클라이언트와 가까워지는 다른 레이어에서는 데이터를 교환하는 데 DTO 객체를 사용하는 것이 일반적이다. 이 책에서 구현하는 스프링 부트 애플리케이션의 구조는 아래와 같다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181041278-e082a621-6e9e-4983-a762-6d8ec8b89263.png){: .align-center}

<br>

위 그림은 서비스와 DAO의 사이에서 엔티티로 데이터를 전달하는 것으로 표현했지만 회사나 개발 그룹 내 규정에 따라 DTO를 사용하기도 한다. 위 구조는 각 레이어 사이의 큰 데이터의 전달을 표현한 것이고, 단일 데이터나 소량의 데이터를 전달하는 경우 DTO나 엔티티를 사용하지 않기도 한다.

지금까지는 서비스 인터페이스를 생성했다. 이제 구현체 클래스를 작성해 보자. 구현체 클래스의 기본 형태는 아래와 같다.

<br>

```java
@Service
public class ProductServiceImpl implements ProductService {
    
    private final ProductDAO productDAO;

    @Autowired
    public ProductServiceImpl(ProductDAO productDAO) {
        this.productDAO = productDAO;
    }

    @Override
    public ProductResponseDto getProduct(Long number) {
        return null;
    }

    @Override
    public ProductResponseDto saveProduct(ProductDto productDto) {
        return null;
    }

    @Override
    public ProductResponseDto changeProductName(Long number, String name) throws Exception {
        return null;
    }

    @Override
    public void deleteProduct(Long number) throws Exception {

    }
}
```

<br>

인터페이스 구현체 클래스에서는 4~9번 줄과 같이 DAO 인터페이스를 선언하고 `@Autowired`를 지정한 생성자를 통해 의존성을 주입받는다. 그리고 인터페이스에서 정의한 메서드를 오버라이딩한다.

이제 오버라이딩된 메서드를 구현할 차례이다. 먼저 조회 메서드에 해당하는 `getProduct()` 메서드를 구현하겠다. `getProduct()` 메서드는 아래와 같이 구현할 수 있다.

<br>

```java
@Override
public ProductResponseDto getProduct(Long number) {
    Product product = productDAO.selectProduct(number);

    ProductResponseDto productResponseDto = new ProductResponseDto();
    productResponseDto.setNumber(product.getNumber());
    productResponseDto.setName(product.getName());
    productResponseDto.setPrice(product.getPrice());
    productResponseDto.setStock(product.getStock());

    return productResponseDto;
}
```

<br> 현재 서비스 레이어에는 DTO 객체와 엔티티 객체가 공존하도록 설계돼 있어 변환 작업이 필요하다. 6~9번 줄에서는 DTO 객체를 생성하고 값을 넣어 초기화 작업을 수행하는데, 이런 부분은 빌더(Builder) 패털을 활용하거나 엔티티 객체나 DTO 객체 내부에 변환하는 메서드를 추가해서 간단하게 전환할 수 있다.

다음으로 `ProductServiceImpl`에서 저장 메서드에 해당하는 `saveProduct()` 메서드를 구현하겠다.

<br>

```java
@Override
public ProductResponseDto saveProduct(ProductDto productDto) {
    Product product = new Product();
    product.setName(productDto.getName());
    product.setPrice(productDto.getPrice());
    product.setStock(productDto.getStock());
    product.setCreatedAt(LocalDateTime.now());
    product.setUpdatedAt(LocalDateTime.now());

    Product savedProduct = productDAO.insertProduct(product);

    ProductResponseDto productResponseDto = new ProductResponseDto();
    productResponseDto.setNumber(savedProduct.getNumber());
    productResponseDto.setName(savedProduct.getName());
    productResponseDto.setPrice(savedProduct.getPrice());
    productResponseDto.setStock(savedProduct.getStock());

    return productResponseDto;
}
```

<br>

저장 메서드는 로직이 간단하다. 전달받은 DTO 객체를 통해 엔티티 객체를 생성해서 초기화한 후 DAO 객체로 전달하면 된다. 다만 저장 메서드의 리턴 타입을 어떻게 지정할지는 고민해야 한다. 일반적으로 저장 메서드는 `void` 타입으로 작성하거나 작업의 성공 여부를 나타내는 `boolean` 타입으로 지정하는 경우가 많다.  리턴 타입은 해당 비즈니스 로직이 어떤 성격을 띠느냐에 따라 결정하는 것이 바람직하다.

`savedProduct()` 메서드는 상품 정보를 전달하고 애플리케이션을 거쳐 데이터베이스에 저장하는 역할을 수행한다. 현재 데이터를 조회하는 메서드는 데이터베이스에서 인덱스를 통해 값을 찾아야 하는데, `void` 로 저장 메서드를 구현할 경우에는 클라이언트가 저장한 데이터의 인덱스 값을 알 방법이 없다. 그렇기 때문에 데이터를 저장하면서 가져온 인덱스를 DTO에 담에 결괏값으로 클라이언트에 전달하는 12~16번 줄의 코드를 작성했다. 만약 이 같은 방식이 아니라 `void` 형식으로 메서드를 작성한다면 조회 메서드를 추가로 구현하고 클라이언트에서 한 번 더 요청해야 한다.

이번에는 업데이트 메서드를 구현한다. 업데이트 메서드에 해당하는 `changeProductName()` 메서드는 아래와 같이 구현할 수 있다.

<br>

```java
@Override
public ProductResponseDto changeProductName(Long number, String name) throws Exception {
    Product changedProduct = productDAO.updateProductName(number, name);

    ProductResponseDto productResponseDto = new ProductResponseDto();
    productResponseDto.setNumber(changedProduct.getNumber());
    productResponseDto.setName(changedProduct.getName());
    productResponseDto.setPrice(changedProduct.getPrice());
    productResponseDto.setStock(changedProduct.getStock());

    return productResponseDto;
}
```

<br>

`changeProductName()` 메서드는 상품정보 중 이름을 변경하는 작업을 수행한다. 이름을 변경하기 위해 먼저 클라이언트로부터 개상을 식별할 수 있는 인덱스 값과 변경하려는 이름을 받아온다. 좀 더 견고하게 코드를 작성하기 위해서는 기존 이름도 받아와 식별자로 가져온 상품정보와 일치라는지 검증하는 단계를 추가하기도 한다.

이 기능의 핵심이 되는 비즈니스 로직은 레코드의 이름 칼럼을 변경하는 것이다. 실제 레코드 값을 변경하는 작업은 DAO에서 진행하기 때문에 서비스 레이어에서는 해당 메섣를 호출해서 결괏값만 받아온다.

마지막으로 삭제 메서드를 구현한다. 삭제 메서드에도 동일하게 검증 로직을 추가해도 되지만 우선 삭제 기능만 수행하도록 구현한다.

<br>

```java
@Override
public void deleteProduct(Long number) throws Exception {
productDAO.deleteProduct(number);
}
```

<br>

상품정보를 삭제하는 메서드는 리포지토리에서 제공하는 `delete()` 메서드를 사용할 경우 리턴받는 타입이 지정돼 있지 않기 때문에 리턴 타입을 `void`로 지정해 메서드를 구현한다.



### 컨트롤러 생성

서비스 객체의 설계를 마친 후에는 비즈니스 로직과 클라이언트의 요청을 연결하는 컨트롤러를 생성해야한다. 앞에서 컨트롤러를 생성하는 방법을 이미 다뤘으므로 여기서는 간단하게 설명하고 넘어가겠다. 컨트롤러의 패키지와 컨트롤러는 아래와 같이 구성한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181049557-08b87aef-4cda-4419-b0fd-0c7da77fd0c1.png){: .align-center}

<br>

컨트롤러는 클라이언트로부터 요청을 받고 해당 요청에 대해 서비스 레이어에 구현된 적절한 메서드를 호출해서 결괏값을 받는다. 이처럼 컨트롤러는 요청과 응답을 전달하는 역할만 맡는 것이 좋다.

컨트롤러는 아래와 같이 작성한다.

<br>

```java
@RestController
@RequestMapping("/product")
public class ProductController {
    
    private final ProductService productService;
    
    @Autowired
    public ProductController(ProductService productService){
        this.productService = productService;
    }
    
    @GetMapping()
    public ResponseEntity<ProductResponseDto> getProduct(Long number){
        ProductResponseDto productResponseDto = productService.getProduct(number);
        
        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }
    
    @PostMapping()
    public ResponseEntity<ProductResponseDto> createProduct(@RequestBody ProductDto productDto){
        ProductResponseDto productResponseDto = productService.saveProduct(productDto);
        
        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
    }
    
    @PutMapping()
    public ResponseEntity<ProductResponseDto> changeProductName(
            @RequestBody ChangeProductNameDto changeProductNameDto) throws Exception {
        ProductResponseDto productResponseDto = productService.changeProductName(
                changeProductNameDto.getNuber(),
                changeProductNameDto.getName());

        return ResponseEntity.status(HttpStatus.OK).body(productResponseDto);
        
    }
    
    @DeleteMapping()
    public ResponseEntity<String> deleteProduct(Long number) throws Exception {
        productService.deleteProduct(number);
        
        return ResponseEntity.status(HttpStatus.OK).body("정상적으로 삭제되었습니다.");
    }
    
}
```

<br>

ChangeProductNameDto 를 추가한다.

<br>

```java
package com.springboot.jpa.data.dto;

public class ChangeProductNameDto {

    private Long number;
    private String name;

    public ChangeProductNameDto(Long number, String name) {
        this.number = number;
        this.name = name;
    }

    public ChangeProductNameDto(){
    }
    
    public Long getNumber(){
        return this.number;
    }
    
    public String getName() {
        return this.name;
    }
    
    public void setNumber(Long number){
        this.number = number;
    }
    
    public void setName(String name) {
        this.name = name;
    }
    
}
```

<br>



### Swagger API를 통한 동작 확인

여기까지 진행하면 아래와 같이 프로젝트 구조가 완성된다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181052960-bdfebfd0-25a3-42b8-b83c-51eafd6839fe.png){: .align-center}

<br>

지금까지 구현한 코드에는 상품정보를 조회, 저장, 삭제할 수 있는 기능을 비롯해 상품정보 중 상품의 이름을 수정하는 기능이 포함돼 있다. 각 기능에 대한 요청은 '컨트롤러 - 서비스 - DAO - 리포지토리'의 계층을 따라 이동하고, 그것의 역순으로 응답을 전달하는 구조이다.

그럼 Swagger API를 통해 애플리케이션의 클라이언트 입장에서 기능을 요청해보고 어떻게 결과가 나타나는지 살펴보겠다. 앞의 5.6절을 참조해서 컨트롤러의 각 기능에 API명세를 작성한다. 작성 방법은 전과 동일하며, `basePackage`를 정의하는 코드에서 패키지명을 현재 실급 중인 패키지 경로로 수정하면 된다. 이후 Swagger API를 사용하기 위해 아래와 같이 애플리케이션을 실행하고 웹 브라우저를 통해 Swagger  페이지(<a href="http://localhost:8080/swagger-ui.html" target="_blank">http://localhost:8080/swagger-ui.html</a>)로 접속한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181054727-6f1c63db-8cb7-4d6d-a1cc-9fbba459910f.png){: .align-center}

<br>

먼저 상품 정보를 저장하겠다. 상품정보를 저장하기 위해서는 POST 메서드를 사용하는 `createProduct()` 메서드를 사용해야 한다. 위 크림에서 POST API를 클릭하고 [Try it out]을 눌러 아래와 같이 입력한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181055744-5ad2e402-6fdd-4e70-8e6d-f870a7a3d031.png){: .align-center}

<br>

```
Hibernate: 
    insert 
    into
        product
        (created_at, name, price, stock, updated_at) 
    values
        (?, ?, ?, ?, ?)
```

<br>

정상적으로 `insert` 쿼리가 생성되어 실행된 것을 볼 수 있다. 실제로 데이터베이스에 데이터가 저장 됐는지 HeidiSQL을 통해 데이터베이스를 확인한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181268123-51ec2e3f-f8c7-4157-86db-bfbbb1eb7d1e.png){: .align-center}

<br>

`createProduct` 명령어를 최초로 실행한 상태라면 `number` 칼럼의 값은 1로 나올 것이다. 그리고 Swagger에서 입력한 이름과 가격, 재고수량이 정상적으로 입력되고 `ProductService` 에서 구현했던 `saveProduct()` 메서드를 통해 `created_at`과 `updated_at` 칼럼에 시간이 포함된 데이터가 추가됐다.

이제 이 값을 가져와본다. 앞에서 만든 조회 메서드는 `number` 의 값을 가지고 데이터를 조회하기 때문에 Swagger 페이지에서 아래와 같이 데이터베이스에서 확인한 값을 입력한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181269281-139eed2d-7c89-4a1d-adaa-43142a71836b.png){: .align-center}

<br>

그러고 나서 [Execute] 버튼을 누르면 결과 페이지에서 아래와 같이 정상적으로 응답 Body에 값이 담긴 것을 확인할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181269828-ef044f89-2fcf-4bc7-8560-e362de3cee5e.png){: .align-center}

<br>

인텔리제이 IDEA 에서 로그를 확인해보면 다음과 같은 `select` 쿼리가 실행된 것을 확인할 수 있다.

<br>

```
Hibernate: 
    select
        product0_.number as number1_0_0_,
        product0_.created_at as created_2_0_0_,
        product0_.name as name3_0_0_,
        product0_.price as price4_0_0_,
        product0_.stock as stock5_0_0_,
        product0_.updated_at as updated_6_0_0_ 
    from
        product product0_ 
    where
        product0_.number=?
```

<br>

그름 이번에는 `updateProductName()` 메서드를 통해 상품의 이름을 변경하겠다. Swagger 페이지에서 아래와 같이 값을 입력한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181270720-fa3ec349-a5ab-4065-9919-9846e247a63c.png){: .align-center}

<br>

위와 같이 Body 값에 식별자 번호를 입력하고 바꾸고자 하는 이름을 기입한 후 [Execute] 버튼을 클릭한다. 그럼 이름이 변경된 상태의 값이 응답으로 오는 것을 확인할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181271341-ea762824-3272-4850-863b-c9bc35a356c2.png){: .align-center}

<br>

마찬가지로 인텔리제이 IDEA에서 로그를 확인해보면 다음과 같은 쿼리가 실행된 것을 확인할 수 있다.

<br>

```
Hibernate: 
    select
        product0_.number as number1_0_0_,
        product0_.created_at as created_2_0_0_,
        product0_.name as name3_0_0_,
        product0_.price as price4_0_0_,
        product0_.stock as stock5_0_0_,
        product0_.updated_at as updated_6_0_0_ 
    from
        product product0_ 
    where
        product0_.number=?
Hibernate: 
    update
        product 
    set
        created_at=?,
        name=?,
        price=?,
        stock=?,
        updated_at=? 
    where
        number=?
```

<br>

쿼리를 보면 업데이트를 위해 대상 영속 객체를 조회한 후 갱신을 위한 `update` 쿼리를 실행하는 것을 볼 수 있다.

다음으로 앞에서 생성했던 상품정보를 삭제하겠다. 아래와 같이 데이터베이스에서 확인했던 `number` 값을 파라미터에 입력한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181272524-b93bae54-a509-4987-a142-dfe5fbca77bb.png){: .align-center}

<br>

값을 입력한 후 [Execute] 버튼을 누르고 결과 화면을 확인한다.

![image](https://user-images.githubusercontent.com/13410737/181272938-8ec59330-39f6-4367-91ab-ed7aae9aedfe.png){: .align-center}

<br>

컨트롤러에서 `deleteProduct()` 메서드를 작성할 때 정상적으로 삭제가 되면 문자열 값을 Body 값에 담아 전달하도록 구현했기 때문에 'Response Body' 부분에 삭제 확인 메시지가 담겨 있다. 상품 정보를 삭제하기 위해 생성된 쿼리는 다음과 같다.

<br>

```
Hibernate: 
    select
        product0_.number as number1_0_0_,
        product0_.created_at as created_2_0_0_,
        product0_.name as name3_0_0_,
        product0_.price as price4_0_0_,
        product0_.stock as stock5_0_0_,
        product0_.updated_at as updated_6_0_0_ 
    from
        product product0_ 
    where
        product0_.number=?
Hibernate: 
    delete 
    from
        product 
    where
        number=?
```

<br>

삭제할 데이터를 특정하기 위해 `select` 쿼리를 통해 데이터를 영속성 컨텍스트로 가져오고, 해당 객체를 삭제 요청해서 `commit` 단계에서 정상적으로 삭제하는 동작이 수행됐다.

이렇게 해서 제품 정보에 대한 기본 CRUD 조작을 실습했다. 다음 장에서는 조금 더 복잡한 JPA의 기능을 사용할 예정이다.
