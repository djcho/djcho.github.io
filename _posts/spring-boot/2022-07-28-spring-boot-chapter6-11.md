---
title : "[SpringBoot][Chapter6.11] DB연동 - 롬복(Lombok)"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Lombok]
toc: true
toc_sticky : true
published : true
date : 2022-07-28
last_modified_at : 2022-07-28
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## 반복되는 코드의 작성을 생략하는 방법 - 롬복

<a href="https://projectlombok.org">롬복(Lombok)</a>은 데이터(모델) 클래스를 생성할 때 반복적으로 사용하는 getter/setter 같은 메서드를 어노테이션으로 대체하는 기능을 제공하는 라이브러리이다. 자바에서 데이터 클래스를 작성하면 대개 많은 멤버 변수를 선언하고, 각 멤버 변수별로 getter/setter 메서드를 만들어 코드가 길어지고 가독성이 낮아진다. 인텔리제이 IDEA나 이클립스 같은 IDE에서는 이러한 메서드를 자동으로 생성하는 기능을 제공하긴 하지만 가독성이 떨어진다는 점에서는 마찬가지다.

이러한 경우 롬복을 활용하면 다음과 같은 장점이 있다.

- 어노테이션 기반으로 코드를 자동 생성하므로 생산성이 높아진다.
- 반복되는 코드를 생략할 수 있어 가독성이 좋아진다.
- 롬복을 안다면 간단하게 코드를 유추할 수 있어 유지보수에 용이하다.

반면 몇 가지 이유로 롬복을 사용하는 것을 좋아하지 않는 개발자도 있다. 롬복을 선호하지 않는 가장 큰 이유는 코드를 어노테이션이 자동 생성하기 때문에 메서드를 개발자의 의도대로 정확하게 구현하지 못하는 경우가 발생한다는 것이다.



### 롬복 설치

이 책에서는 프로젝트를 생성하는 단계에서 롬복을 의존성에 추가해 둔 상태이다. `pom.xml` 파일에 아래와 같은 코드가 추가돼 있는지 확인해 보자.

<br>

```xml
</dependencies>
...
	<dependency>
		<groupId>org.projectlombok</groupId>
		<artifactId>lombok</artifactId>
		<optional>true</optional>
	</dependency>
</dependencies>
```

<br>

인텔리제이에서 롬복 라이브러리를 정상적으로 사용하려면 의존성 추가 외에도 몇 가지 설정이 필요하다. 인텔리제이 IDEA 메뉴에서 [File] → [Settings]를 선택해 Setting 창이 나타나면 왼쪽의 [Plugins]를 선택한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181278193-e935bb44-df86-4633-a5eb-af94d6e3a339.png){: .align-center}

<br>

Marketplace에서 'lombok'을 검색해 설치하면 [Installed] 탭에서 Lombok이 활성화돼 있는 것을 확인할 수 있다. Lombok이 설치된 것을 확인했다면 아래와 같이 Setting 창에서 [Build, Execution, Deployment] → [Compiler] → [Annotation Processors]를 차례로 선택한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/181279901-e314b6c5-8dcd-47dc-9a65-49177fe42cfc.png){: .align-center}

<br>

위 화면에서 [Enable annotation processing] 항목에 체크를하고 [OK] 버튼을 눌러야 롬복을 정상적으로 사용할 수 있다.



### 롬복 적용

앞에서 설명한 설치와 설정 과정을 모두 마쳤다면 정상적으로 롬복을 사용할 수 있다. 지금까지 프로젝트 실습을 진행하면서 생성한 데이터 클래스에 롬복을 적용하면서 각 어노테이션의 기능을 살펴보겠다.

먼저 `Product` 엔티티 클래스에 롬복을 적용해 보겠다. 아래 코드는 앞에서 살펴본 `Product` 엔티티 클래스이다.

<br>

```java
package com.springboot.jpa.data.entity;

@Entity
@Table(name = "product")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;

    public Product(Long number, String name, Integer price, Integer stock, LocalDateTime createdAt,
                   LocalDateTime updatedAt){
        this.number = number;
        this.name = name;
        this.price = price;
        this.stock = stock;
        this.createdAt = createdAt;
        this.updatedAt = updatedAt;
    }
    
    public Product(){
    }
    ... getter/setter 메서드 생략...
}
```

<br>

참고로 위 예제에서는 코드를 조금 더 보기 편하게 getter/setter 메서드를 생략했는데, 만약 getter/setter 메서드가 모두 포함돼 있었다면 최소 100줄 이상이 될 것이다. 위 코드를 아래와 같이 어노테이션을 이용해 많은 코드를 대체할 수 있다.

<br>

```java
package com.springboot.jpa.data.entity;

@Entity
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Table(name = "product")
public class Product {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;

    @Column(nullable = false)
    private String name;

    @Column(nullable = false)
    private Integer price;

    @Column(nullable = false)
    private Integer stock;

    private LocalDateTime createdAt;

    private LocalDateTime updatedAt;
}
```

<br>

이전 코드와 비교해보면 심지어 getter/setter 메서드를 생략한 예제보다 코드 라인 수가 적다. 물론 어노테이션으로 메서드를 자동 생성했기 때문에 필요한 모든 코드는 갖춰져 있다.

> 롬복의 어노테이션이 실제로 어떤 메서드를 생성하는지는 인텔리제이 IDEA 에서 확인 할 수 있다. `Product` 클래스에 마우스 오른쪽 버튼을 클릭한 후 [Refactor] → [Delombok] → [All Lombok annotations]를 클릭하면 롬복의 어노테이션이 실제 코드로 리펙토링된다. 
>
> Delombok을 하지 않고 생성된 메서드가 어떤 종류가 있는지 확인하려면 어노테이션을 작성하고 인텔리제이 IDEA 좌측에 있는 [Structure]를 클릭하면 해당 클래스에 정의된 메서드 목록을 볼 수 있다.



### 롬복의 주요 어노테이션

롬복은 다양한 어노테이션을 제공하고 있다. 그중 많이 사용하는 어노테이션들을 소개한다.

#### @Getter, @Setter

클래스에 선언돼 있는 필드에 대한 getter/setter 메서드를 생성한다. 아래의 `Product` 클래스에서 쓰인 `@Getter`, `@Setter` 를 실제 코드로 추출한 결과는 아래와 같다.

<br>

```java
public Long getNumber() {
    return this.number;
}

public String getName() {
    return this.name;
}

public Integer getPrice() {
    return this.price;
}

public Integer getStock() {
    return this.stock;
}

public LocalDateTime getCreatedAt() {
    return this.createdAt;
}

public LocalDateTime getUpdatedAt() {
    return this.updatedAt;
}

public void setNumber(Long number) {
    this.number = number;
}

public void setName(String name) {
    this.name = name;
}

public void setPrice(Integer price) {
    this.price = price;
}

public void setStock(Integer stock) {
    this.stock = stock;
}

public void setCreatedAt(LocalDateTime createdAt) {
    this.createdAt = createdAt;
}

public void setUpdatedAt(LocalDateTime updatedAt) {
    this.updatedAt = updatedAt;
}
```

<br>

`@Getter/@Setter` 어노테이션을 통해 `Product` 클래스가 가지고 있는 필드에 대해 각각 getter/setter 메서드가 생성되는 것을 볼 수 있다. 인텔리제이 IDEA 등의 IDE가 제공하는 자동 생성 메서드와 기능차이는 없지만 코드의 라인 수를 줄이는 데는 효과적이다.



#### 생성자 자동 생성 어노테이션

데이터 클래스의 초기화를 위한 생성자를 자동으로 만들어주는 어노테이션은 다음의 세 가지가 있다.

- `NoArgsConstructor` : 매개변수가 없는 생성자를 자동 생성한다.
- `AllArgsConstructor` : 모든 필드를 매개변수로 갖는 생성자를 자동 생성한다.
- `RequiredArgsConstructor` : 필드 중 final이나 @NotNull 이 설정된 변수를 매개변수로 갖는 생성자를 자동 생성한다.

현재 `Product` 클래스에는 `@RequiredArgsConstructor` 로 정의될 필드가 없기 때문에 다른 두 개의 어노테이션을 Delombok 해서 나온 코드는 아래와 같다.

<br>

```java
public Product(Long number, String name, Integer price, Integer stock, LocalDateTime createdAt, LocalDateTime updatedAt) {
    this.number = number;
    this.name = name;
    this.price = price;
    this.stock = stock;
    this.createdAt = createdAt;
	this.updatedAt = updatedAt;
}

public Product() {
}
```

<br>

`@AllArgsConstructor` 어노테이션을 통해 1~9번 줄의 생성자가 생성되고, `@NoArgsConstructor` 어노테이션을 통해 11~12번 줄의 생성자가 생성된다.



#### @ToString

이름 그래도 `toString()` 메서드를 생성하는 어노테이션이다. `Product` 클래스에 `@toString()` 을 적용해 Delombok을 수행하면 아래와 같은 코드가 생성된다.

<br>

```java
public String toString() {
	return "Product(number=" + this.getNumber() + ", name=" + this.getName() + ", price=" + this.getPrice() + ", stock=" + this.getStock() + ", createdAt=" + this.getCreatedAt() + ", updatedAt=" + this.getUpdatedAt() + ")";
}
```

<br>

`toString()` 메서드는 필드의 값을 문자열로 조합해서 리턴한다. 또한 민감한 정보처럼 숨겨야할 정보가 있다면 아래와 같이 `@ToString` 어노테이션이 제공하는 `exclude` 속성을 사용해 특정 필드를 자동 생성에서 제외할 수 있다.

<br>

```java
@ToString(exclude = "name")
@Table(name = "product")
public class Product {
	...(생략)
}
```

<br>

#### @EqualsAndHashCode

`@EqualsAndHashCode`는객체의 동등성(Equality)와 동일성(Identity)을 비교하는 연산 메서드를 생성한다. `Product` 클래스에 `@EqualsAndHashCode` 어노테시션을 적용한 후 Delombok을 수행하면 아래와 같다.

<br>

```java
public boolean equals(final Object o) {
    if (o == this) return true;
    if (!(o instanceof Product)) return false;
    final Product other = (Product) o;
    if (!other.canEqual((Object) this)) return false;
    final Object this$number = this.getNumber();
    final Object other$number = other.getNumber();
    if (this$number == null ? other$number != null : !this$number.equals(other$number)) return false;
    final Object this$name = this.getName();
    final Object other$name = other.getName();
    if (this$name == null ? other$name != null : !this$name.equals(other$name)) return false;
    final Object this$price = this.getPrice();
    final Object other$price = other.getPrice();
    if (this$price == null ? other$price != null : !this$price.equals(other$price)) return false;
    final Object this$stock = this.getStock();
    final Object other$stock = other.getStock();
    if (this$stock == null ? other$stock != null : !this$stock.equals(other$stock)) return false;
    final Object this$createdAt = this.getCreatedAt();
    final Object other$createdAt = other.getCreatedAt();
    if (this$createdAt == null ? other$createdAt != null : !this$createdAt.equals(other$createdAt)) return false;
    final Object this$updatedAt = this.getUpdatedAt();
    final Object other$updatedAt = other.getUpdatedAt();
    if (this$updatedAt == null ? other$updatedAt != null : !this$updatedAt.equals(other$updatedAt)) return false;
    return true;
}

protected boolean canEqual(final Object other) {
    return other instanceof Product;
}

public int hashCode() {
    final int PRIME = 59;
    int result = 1;
    final Object $number = this.getNumber();
    result = result * PRIME + ($number == null ? 43 : $number.hashCode());
    final Object $name = this.getName();
    result = result * PRIME + ($name == null ? 43 : $name.hashCode());
    final Object $price = this.getPrice();
    result = result * PRIME + ($price == null ? 43 : $price.hashCode());
    final Object $stock = this.getStock();
    result = result * PRIME + ($stock == null ? 43 : $stock.hashCode());
    final Object $createdAt = this.getCreatedAt();
    result = result * PRIME + ($createdAt == null ? 43 : $createdAt.hashCode());
    final Object $updatedAt = this.getUpdatedAt();
    result = result * PRIME + ($updatedAt == null ? 43 : $updatedAt.hashCode());
    return result;
}
```

<br>

두 메서드는 각각 다음과 같은 연산을 수행한다.

- `equals` : 두 객체의 내용이 같은지 동등성을 비교한다.
- `hashCode` : 두 객체가 같은 객체인지 동일성을 비교한다.



만약 부모 클래스가 있어서 상속을 받는 상황이라면 부모 클래스의 필드까지 비교할 필요가 있는 경우도 발생한다. 이 경우에는 `@EqualsAndHashCode` 에서 제공하는 `callSuper` 속성을 아래와 같이 설정하면 부모 클래스의 필드를 비교 대상에 포함할 수 있다.

<br>

```java
@Entity
@EqualsAndHashCode(callSuper = true)
@Table(name = "product")
public class Product extends BaseEntity {
	...(생략)
}
```

<br>

`callSuper`의 기본값은 `false` 이며, `true`일 경우 부모 객체의 값도 비교 대상에 포함한다.

> **동등성과 동일성**
>
> `equals`와 `hashCode`  를 공부할 때 반드시 나오는 개념으로 '동등성(equality)'과 '동일성(Identity)이 있다. 동긍성 비교 대상이 되는 두 객체가 가진 값이 같음을 의미하고, 동일성은 두 객체가 같은 객체임을 의미한다.'
>
> 두 메서드는 일반적으로 클래스 단위의 객체를 비교하는 데 사용하고 `Object` 클래스의 메서드를 오버라이딩해 구현한다.'
>
> 동등성과 동일성이 원시(primitive) 타입의 자료형과 레퍼런스(reference) 타입의 자료형에서 어떻게 사용되는지 알아보면 자바를 이해하는데 도움이 된다. 그중 `String` 타입은 특별한 사례이다. `String` 은 레퍼런스 타입이지만 원시 타입처럼 사용된다. `String` 에서의 동등성과 동일성의 비교 원리도 알아보길 바란다.



#### @Data

`@Data` 는 앞서 설먕힌 `@Getter/Setter`, `@RequiredArgsConstructor`, `@ToString`, `@EqualsAndHashCode` 를 모두 포괄하는 어노테이션이다. 즉, 앞에서 살펴본 각각의 어노테이션에서 생성하는 대부분의 코드가 필요하다면 `@Data` 어노테이션으로 앞에서 설명한 코드를 전부 한 번에 생성할 수 있다.

> 롬복과 관련된 자세한 기능은 공식 사이트의 `features` 항목(<a href="https://projectlombok.org/features/all">href="https://projectlombok.org/features/all</a>)에서 확인할 수 있다.
