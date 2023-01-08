---
title : "[SpringBoot][Chapter6.5] DB연동 - 영속성 컨텍스트"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij]
toc: true
toc_sticky : true
published : true
date : 2022-07-21
last_modified_at : 2022-07-21
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



# 영속성 컨텍스트

영속성 컨텍스트(Persistence Context)는 애플리케이션과 데이터베이스 사이에서 엔티티와 레코드의 괴리를 해소하는 기능과 객체를 보관하는 기능을 수행한다. 엔티티 객체가 영속성 컨텍스트에 들어오면 JPA는 엔티티 객체의 매핑 정보를 데이터베이스에 반영하는 작업을 수행안다. 이처럼 엔티티 객체가 영속성 컨텍스트에 들어와 JPA의 관리 대상이 되는 시점부터는 해당 객체를 영속 객체(Persistence Object)라고 부른다. 간단하게 애플리케이션과 데이터베이스와의 관계를 표현하면 아래와 같다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180015608-3b908a03-0616-45c9-a83b-7931af42a245.png){: .align-center}

<br>

영속성 컨텍스트는 세션 단위의 생명주기를 가진다. 데이터베이스에 접근하기 위한 세션이 생성되면 영속성 컨텍스트가 만들어지고, 세션이 종료되면 영속성 컨텍스트도 없어진다. 엔티티 매니저는 이러한 일련의 과정에서 영속성 컨텍스트에 접근하기 위한 수단으로 사용된다.

<br>

## 엔티티 매니저

엔티티 매니저(Entity Manager)는 이름 그대로 엔티티를 관리하는 객치이다. 엔티티 매니저는 데이터베이스에 접근해서 CRUD 작업을 수행한다. Spring Data JPA를 사용하면 리포지토리를 사용해서 데이터베이스를 접근하는데, 실제 내부 구현체인 `SimpleJpaRepository` 가 아래와 같이 리포지토리에서 엔티티 매니저를 사용하는 것을 알 수 있다.

<br>

```java
public SimpleJpaRepository(JpaEntityInformation<T, ?> entityInformation, EntityManager entityManager){
	Assert.notNull(entityInformation, "JpaEntityInformation must not be null");
	Assert.notNull(entityManager, "EntityManager must not be null");
	
	this.entityInformation = entityInformation;
	this.em = entityManager;
	this.provider = PersistenceProvider.fromEntityManager(entityManager);
}
```

<br>

엔티티 매니저는 엔티티 매니저 팩토리(`EntityManagerFactory`)가 만든다. 엔티티 매니저 팩토리는 데이터베이스에 대응하는 객체로서 스프링 부트에서는 자동 설정 기능이 있기 때문에 `application.properties`에서 작성한 최소한의 설정만으로도 동작하지만 JPA의 구현체 중 하나인 하이버네이트에서는 `persistence.xml` 이라는 설정 파일을 구성하고 사용해야 하는 객체이다. 아래는 `persistence.xml`파일의 예를 보여 준다.

<br>

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<persistence xmlns="http://xmlns.jcp.org./xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmls.jcp.org/xml/ns/persistence_2_1.xsd" version="2.1">
    
    <persistence-unit name="entity_manager_factory" transaction-type="RESOURCE_LOCAL">
    
        <properties>
            <property name="javax.persistence.jdbc.driver" value="org.mariadb.jdbc.Driver"/>
            <property name="javax.persistence.jdbc.user" value="root"/>
            <property name="javax.persistence.jdbc.password" value="password"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:mariadb://localhost:3306"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.MariaDB103Dialect"/>
            
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            
        </properties>
        
    </persistence-unit>
    
</persistence>
```

<br>

8번 줄과 같이 `persistence-unit` 을 설정하면 해당 유닛의 이름을 가진 엔티티 매니저 팩토리가 생성된다. 엔티티 매니저 팩토리는 애플리케이션에서 단 하나만 생성되며, 모든 엔티티가 공유해서 사용한다.

엔티티 매니저 팩토리로 생성된 엔티티 매니저는 엔티티를 영속성 컨텍스트에 추가하여 영속 객체로 만드는 작업을 수행하고 영속성 컨텍스트와 데이터베이스를 비교하면서 실제 데이터베이스를 대상으로 작업을 수행한다.

<br>

## 엔티티의 생명주기

엔티티 객체는 영속성 컨텍스트에서 아래와 같은 4가지 상태로 구분된다.

- **비영속(New)**
  - 영속성 컨텍스트에 추가되지 않은 엔티티 객체의 상태를 의미한다.
- **영속(Managed)**
  - 영속성 컨텍스트에 의해 엔티티 객체가 관리되는 상태이다.
- **준영속(Detached)**
  - 영속성 컨텍스트에 의해 관리되면 엔티티 객체가 컨첵스트와 분리된 상태이다.
- **삭제(Removed)**
  - 데이터베이스에서 레코드를 삭제하기 위해 영속성 컨텍스트에 삭제 요청을 한 상태이다.
