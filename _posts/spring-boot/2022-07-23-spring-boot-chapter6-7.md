---
title : "[SpringBoot][Chapter6.6] DB연동 - 엔티티 설계"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij]
toc: true
toc_sticky : true
published : true
date : 2022-07-23
last_modified_at : 2022-07-23
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## 엔티티 설계

 Spring Data JPA를 사용하면 데이터베이스에 테이블을 생성하기 위해 직접 쿼리를 작성할 필요가 없다. 이 기능을 가능하게 하는 것이 엔티티이다. JPA에서 엔티티는 데이터베이스의 테이블에 대응하는 클래스이다. 엔티티에는 데이터베이스에 쓰일 테이블과 칼럼을 정의한다. 엔티티에 어노테이션을 사용하면 테이블 간 연관관계를 정의할 수 있다.

아래 테이블은 코드로 간단하게 엔티티 클래스로 구현할 수 있다. `data.entity` 패키지를 생성하고 그 안에 `Product`엔티티 클래스를 생성한다.

![image](https://user-images.githubusercontent.com/13410737/180595294-52e8a7ff-c8ca-4d52-8b22-aca1eefe449d.png){: .align-center}

```java
package com.springboot.jpa.data.entity;

import javax.persistence.*;
import java.time.LocalDateTime;

@Entity
@Table(name = "Product")
public class Product {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long number;
    
    @Column(nullable = false)
    private String name;
    
    @Column(nullable = false)
    private Integer price;
    
    @Column(nullable = false)
    private  Integer stock;
    
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    ...getter/setter 메서드 생략
}
```

<br>

위와 같이 클래스를 생성하고 `application.properties`에 정의한 `spring.jpa.hibernate.ddl-auto`의 값을 `create` 같은 테이블을 생성하는 옵션으로 설정하면 쿼리문을 작성하지 않아도 데이터베이스에 테이블이 자동으로 만들어진다.



### 엔티티 관련 기본 어노테이션

엔티티를 작성할 때는 어노테이션을 많이 사용한다. 그 중에는 테이블과 매핑하기 위해 사용하는 어노테이션도 있고, 다른 테이블과의 연관관계를 정의하기 위해 사용하는 어노테이션, 자동으로 값을 주입하기 위한 어노테이션도 있다. 여기서는 먼저 기본적으로 많이 사용하는 어노테이션을 소개하고, 다른 어노테이션은 이후 내용을 진행하면서 필요할 때마다 소개하겠다.



#### @Entity

해당 클래스가 엔티티임을 명시하기 위한 어노테이션이다. 클래스 자체는 테이블과 일대일로 매칭되며, 해당 클래스의 인스턴스는 매핑되는 테이블에서 하나의 레코드를 의미한다.



#### @Table

엔티티 클래스는 테이블과 매핑되므로 특별한 경우가 아니면 `@Table` 어노테이션이 필요하지 않다. `@Table` 어노테이션을 사용할 때는 클래스의 이름과 테이블의 이름을 다르게 지정해야 하는 경우다. `@Table` 어노테이션을 명시하지 않으면 테이블의 이름과 클래스의 이름이 동일하다는 의미이며, 서로 다른 이름을 쓰려면 `@Table(name = 값)` 형태로 데이터베이스의 테이블명을 명시해야 한다. 대체로 자바의 명명법과 데이터베이스가 사용하는 명명법이 다르기 때문에 자주 사용된다.



#### @Id

엔티티 클래스의 필드는 테이블의 칼럼과 매핑된다. `@Id` 어노테이션이 선언된 필드는 테이블의 기본값 역할로 사용된다. 모든 엔티티는 `@Id` 어노테이션이 필요하다. 



#### @GeneratedValue

일반적으로 `@Id` 어노테이션과 함께 사용된다. 이 어노테이션은 해당 필드의 값을 어떤 방식으로 자동으로 생성할지 결정할 때 사용된다. 값 생성 방식은 다음과 같다.

##### GeneratedValue를 사용하지 않은 방식(직접 할당)

- 애플리케이션에서 자체적으로 고유한 기본값을 생성할 경우 사용하는 방식
- 내부에 정해진 규칙에 의해 기본값을 생성하고 식별자로 사용

##### Auto

- `@GeneratedValue`의 기본 설정값
- 기본값을 사용하는 데이터베이스에 맞게 자동 생성한다.

##### IDENTITY

- 기본값 생성을 데이터베이스에 위임하는 방식
- 데이터베이스의 AUTO_INCREMENT를 사용해 기본값을 생성한다.

##### SEQUENCE

- `@SquenceGenerator` 어노테이션으로 식별자 생성기를 설정하고 이를 통해 값을 자동 주입받는다.
- `SequenceGenerator`를 정의할 때는 `name`, `sequenceName`, `allocationSize`를 활용한다.
- `@GeneratedValue`에 생성기를 설정한다.

##### TABLE

- 어떤 DBMS를 사용하더라도 동일하게 동작하기를 원할 경우 사용한다.
- 식별자로 사용할 숫자의 보관 테이블을 별도로 생성해서 엔티티를 생성할 때마다 값을 갱신하며 사용한다.
- `@TableGenerator` 어노테이션으로 테이블 정보를 설정한다.



#### @Column

엔티티 클래스의 필드는 자동으로 테이블 칼럼으로 매핑된다. 그래서 별다른 설정을 하지 않을 예정이라면 이 어노테이션을 명시하지 않아도 괜찮다. `@Column`어노테이션은 아래와 같이 필드에 몇 가지 설정을 더할 때 사용한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180595898-350e2c8a-5e5d-4a43-8376-c44dd1883893.png){: .align-center}

<br>

`@Column` 어노테이션에서 많이 사용하는 요소는 다음과 같다.

- `name` : 데이터베이스의 컬럼명을 설정하는 속성이다. 명시하지 않으면 필드명으로 지정된다.
- `nullable` : 레코드를 생성할 때 컬럼 값에 null 처리가 가능하지를 명시하는 속성이다.
- `length` : 데이터베이스에 저장하는 데이터의 최대 길이를 설정한다.
- `unique` : 해당 컬럼을 유니크로 설정한다.



#### @Transient

엔티티 클래스에는 선언돼 있는 필드지만 데이터베이스에서는 필요 없을 경우 이 어노테이션을 사용해 데이터베이스에서 이용하지 않게 할 수 있다.

