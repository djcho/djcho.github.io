---
title : "[SpringBoot][Chapter13.5] 서비스의 인증과 권한 부여 - 스프링 시큐리티와 JWT 적용"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, SpringSecurity, JWT]
toc: true
toc_sticky : true
published : true
date : 2022-08-14
last_modified_at : 2022-08-14
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}







## 스프링 시큐리티와 JWT 적용

이제 애플리케이션에 스프링 시큐리티와 JWT를 적용해보겠다. 먼저 프로젝트를 생성하겠다. 먼저 다음과 같은 설정으로 프로젝트를 생성한다.

- groupId : com.springboot
- artifactId : security
- name : security
- Developer Tools : Lombok, Spring Configuration Processor
- Web : Spring Web
- SQL : Spring Data JPA, MariaDB Driver

<br>

그리고 이전 장에서 사용했던 `SwaggerConfiguration` 클래스와 그에 따른 의존성을 추가한다. 기본 프로젝트 틀을 가져가기 위해 7장에서 사용한 프로젝트 코드를 다음과 같이 가져온다.

<br>

![image](https://user-images.githubusercontent.com/13410737/184502205-4af85cef-7eea-4a22-97d1-43843e9bd237.png){: .align-center}

<br>

그리고 인증과 인가 코드를 작성하기 위해 아래와 같이 의존성을 구성한다.

<br>

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<dependency>
   <groupId>io.jsonwebtoken</groupId>
   <artifactId>jjwt</artifactId>
   <version>0.9.1</version>
</dependency>
```

<br>

스프링 시큐리티는 기본적으로 `UsernamePasswordAuthenticationFilter`를 통해 인증을 수행하도록 구성돼 있다. 참고로 이 필터에서는 인증이 실패하면 로그인 폼이 포함된 화면을 전달하게 되는데, 이 책의 실습 프로젝트에는 이러한 화면이 없다. 따라서 JWT를 사용하는 인증 필터를 구현하고 `UsernamePasswordAuthenticationFilter` 앞에 인증 필터를 배치해서 인증 주체를 변경하는 작업을 수행하는 방식으로 구성하겠다.

<br>



### UserDetails와 UserDetailsService 구현

먼저 아래와 같이 사용자 정보를 담는 엔티티를 생성한다.

<br>

```java
@Entity
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
@Table
public class User implements UserDetails {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;

    @Column(nullable = false, unique = true)
    private String uid;

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Column(nullable = false)
    private String password;

    @Column(nullable = false)
    private String name;

    @ElementCollection(fetch = FetchType.EAGER)
    @Builder.Default
    private List<String> roles = new ArrayList();

    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return this.roles.stream().map(SimpleGrantedAuthority::new).collect(Collectors.toList());
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public String getUsername(){
        return this.uid;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isAccountNonExpired(){
        return true;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isCredentialsNonExpired() {
        return false;
    }

    @JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
    @Override
    public boolean isEnabled() {
        return false;
    }
}
```

<br>

`User` 엔티티는 `UserDetails` 인터페이스를 구현하고 있다. `UserDetails`는 앞에서 봤듯이 `UserDetailsService`를 통해 입력된 로그인 정보를 가지고 데이터베이스에서 사용자 정보를 가져오는 역할을 수행한다. `UserDetails`인터페이스는 아래와 같은 메서드를 가지고 있다.

<br>

```java
public interface UserDetails extends Serializable {

    Collection<? extends GrantedAuthority> getAuthorities();

    String getPassword();

    String getUsername();

    boolean isAccountNonExpired();

    boolean isAccountNonLocked();

    boolean isCredentialsNonExpired();

    boolean isEnabled();
}
```

<br>

각 메서드의 용도를 정리하면 다음과 같다.

- `getAuthorities()` : 계정이 가지고 있는 권한 목록을 리턴한다.
- `getPassword()` : 계정의 비밀번호를 리턴한다.
- `getUsername()` : 계정의 이름을 리턴한다. 일반적으로 아이디를 리턴한다.
- `isAccountNonExpired()` : 계정이 만료됐는지 리턴한다. `true`는 만료되지 않았다는 의미이다.
- `isAccountNonLocked()` : 계정이 잠겨있는지 리턴한다. `true`는 잠기지 않았다는 의미이다.
- `isCredentialNonExpired()` 비밀번호가 만료됐는지 리턴한다. `true`는 만료되지 않았다는 의미이다.
- `isEnable()` : 계정이 활성화돼 있는지 리턴한다. `true`는 활성화 상태를 의미한다.

<br>

이번 예제에서는 계정의 상태 변경을 다루지 않을 예정이므로  `true`로 리턴한다. 이 엔티티는 앞으로 토큰을 생성할 때 토큰의 정보로 사용될 정보와 권한 정보를 갖게 된다.

이번에는 앞에서 살펴본 엔티티를 조회하는 기능을 구현하기 위해 리포지토리와 서비스를 구현하겠다. 리포지토리의 구현은 아래와 같다.

<br>

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    User getByUid(String uid);
    
}
```

<br>

`UserRepository` 를 작성하는 것은 기존에 리포지토리를 작성하던 방법과 동일하다. `JpaRepository`를 상속받고 `User` 엔티티에 대해 설정하면 된다. 그리고 형재 ID 값은 인덱스 값이기 때문에 `id`값을 토큰 생성 정보로 사용하기 위해 3번 줄과 같이 `getByUid()` 메서드를 생성한다.

이제 리포지토리를 통해 `User` 엔티티의 `id`를 가져오는 서비스를 생성한다. 아래와 같이 `UserDetailsServiceImpl`을 생성한다.

<br>

```java
@RequiredArgsConstructor
@Service
public class UserDetailsServiceImpl implements UserDetailsService {

    private final Logger LOGGER = LoggerFactory.getLogger(UserDetailsServiceImpl.class);

    private final UserRepository userRepository;


    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        LOGGER.info("[loadUserByUsername] loadUserByUsername 수행, username : {}", username);
        return userRepository.getByUid(username);
    }
}
```

<br>

위 코드의 3번 줄을 보면 `UserDetailsService` 인터페이스를 구현하도록 설정돼 있다. `UserDetailsService`는 아래와 같이  `loadUserByUsername()` 메서드를 구현하도록 정의돼 있다.

<br>

```java
public interface UserDetailsService{
    
    UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
    
}
```

<br>

`UserDetails`는 스프링 시큐리티에서 제공하는 개념으로, `UserDetails`의 `username`은 각 사용자를 구분할 수 있는 ID를 의미한다. 3번 줄의 메서드를 보면 `username`을 가지고 `UserDetails` 객체를 리턴하게 끔 정의돼 있는데, `UserDetils`의 구현체로 `User`엔티티를 생성했기 때문에 `User` 객체를 리턴하게 끔 구현한 것이다.
