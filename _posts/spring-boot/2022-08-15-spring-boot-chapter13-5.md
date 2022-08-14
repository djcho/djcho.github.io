---
title : "[SpringBoot][Chapter13.5] 서비스의 인증과 권한 부여 - 스프링 시큐리티와 JWT 적용"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, SpringSecurity, JWT]
toc: true
toc_sticky : true
published : true
date : 2022-08-15
last_modified_at : 2022-08-15
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



### JwtTokenProvider 구현

이제 JWT 토큰을 생성하는 데 필요한 정보를 `UserDetails`에서 가져올 수 있기 때문에 JWT 토큰을 생성하는 `JwtTokenProvider`를 생성한다. 구현 클래스는 아래와 같다.

<br>

```java
@Component
@RequiredArgsConstructor
public class JwtTokenProvider {

    private final Logger LOGGER = LoggerFactory.getLogger(JwtTokenProvider.class);
    private final UserDetailsService userDetailsService;

    @Value("${springboot.jwt.secret}")
    private String secretKey = "secretKey";
    private final long tokenValidMillisecond = 1000L * 60 * 60;

    @PostConstruct
    protected void init() {
        LOGGER.info("[init] JwtTokenProvider 내 secretKey 초기화 시작");
        secretKey = Base64.getEncoder().encodeToString(secretKey.getBytes(StandardCharsets.UTF_8));
        LOGGER.info("[init] JwtTokenProvider 내 secretKey 초기화 완료");
    }

    public String createToken(String userUid, List<String> roles){
        LOGGER.info("[createToken] 토큰 생성 시작");
        Claims claims = Jwts.claims().setSubject(userUid);
        claims.put("roles", roles);
        Date now = new Date();

        String token = Jwts.builder()
                .setClaims(claims)
                .setIssuedAt(now)
                .setExpiration(new Date(now.getTime() + tokenValidMillisecond))
                .signWith(SignatureAlgorithm.HS256, secretKey)
                .compact();

        LOGGER.info("[createToken] 토큰 생성 완료");
        return token;
    }

    public Authentication getAuthentication(String token){
        LOGGER.info("[getAuthentication] 토큰 인증 정보 조회 시작");
        UserDetails userDetails = userDetailsService.loadUserByUsername(this.getUsername(token));
        LOGGER.info("[getAuthentication] 토큰 인증 정보 조회 완료, UserDetails UserName : {}", userDetails.getUsername());
        return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
    }

    public String resolveToken(HttpServletRequest request){
        LOGGER.info("[resolveToken] HTTP 헤더에서 Token 값 추출");
        return request.getHeader("X-AUTH-TOKEN");
    }

    public boolean validateToken(String token){
        LOGGER.info("[validateToken] 토큰 유효 체크 시작");
        try{
            Jws<Claims> claims = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token);

            return !claims.getBody().getExpiration().before(new Date());
        }catch(Exception e) {
            LOGGER.info("[validateToken] 토큰 유효 체크 예외 발생");
            return false;
        }
    }
}
```

<br>

토큰을 생성하기 위해서는 `secretKey`가 필요하므로 8~9번 줄에서 `secretKey` 값을 정의한다. `@Value`의 값은 `application.properties` 파일에서 다음과 같이 정의할 수 있다.

<br>

```properties
springboot.jwt.secret=flature!@#
```

<br>

만약 `application.preperties` 파일에서 값을 가져올 수 없다면 위 코드의 9번 줄에 입력해둔 기본값인 '`secretKey`'를 가져온다.

13번 줄에서는 `init()` 메서드가 아래와 같이 작성돼 있다.

<br>

```java
@PostConstruct
protected void init() {
    LOGGER.info("[init] JwtTokenProvider 내 secretKey 초기화 시작");
    System.out.println(secretKey);
    secretKey = Base64.getEncoder().encodeToString(secretKey.getBytes(StandardCharsets.UTF_8));
    System.out.println(secretKey);
    LOGGER.info("[init] JwtTokenProvider 내 secretKey 초기화 완료");
}
```

<br>

여기서 사용한 `@PostConstruct` 어노테이션은 해당 객체가 빈 객체로 주입된 이후 수행되는 메서드를 가리킨다. `JwtTokenProvider` 클래스에는 `@Component` 어노테이션이 지정돼 있어 애플리케이션이 가동되면서 빈으로 자동 주입된다. 그때 `@PostConstruct`가 지정돼 있는 `init()` 메서드가 자동으로 실행된다. `init()` 메서드에서는 `secretKey`를 Base64 형식으로 인코딩한다. 인코딩 전후의 문자열을 확인하면 다음과 같다.



```
// 인코딩 전 원본 문자열
flature!@#

//Base64 인코딩 결과
ZmxhdHVyZSFAIw==
```

<br>

그리고 19~34번 줄에 있는 `createToken()` 메서드를 아래에서 보겠다.

<br>

```java
public String createToken(String userUid, List<String> roles){
    LOGGER.info("[createToken] 토큰 생성 시작");
    Claims claims = Jwts.claims().setSubject(userUid);
    claims.put("roles", roles);
    Date now = new Date();

    String token = Jwts.builder()
            .setClaims(claims)
            .setIssuedAt(now)
            .setExpiration(new Date(now.getTime() + tokenValidMillisecond))
            .signWith(SignatureAlgorithm.HS256, secretKey)
            .compact();

    LOGGER.info("[createToken] 토큰 생성 완료");
    return token;
}
```

<br>

3번 줄에서는 JWT 토큰의 내용에 값을 넣기 위해 `Claims` 객체를 생성한다. `setSubject()` 메서드를 통해 `sub` 속성에 값을 추가하려면 `User`의 `uid` 값을 사용한다. 4번 줄에서는 해당 토큰을 사용하는 사용자의 권한을 확인할 수 있는 `role`값을 별개로 추가했다. 그리고 7~12번 줄처럼 `Jwts.builder()` 를 사용해 토큰을 생성한다.

다음으로 볼 내용은 `getAuthentication()` 메서드이다. `getAuthentication()` 메서드는 아래와 같이 작성돼 있다.

<br>

```java
public Authentication getAuthentication(String token){
    LOGGER.info("[getAuthentication] 토큰 인증 정보 조회 시작");
    UserDetails userDetails = userDetailsService.loadUserByUsername(this.getUsername(token));
    LOGGER.info("[getAuthentication] 토큰 인증 정보 조회 완료, UserDetails UserName : {}", userDetails.getUsername());
    return new UsernamePasswordAuthenticationToken(userDetails, "", userDetails.getAuthorities());
}
```

<br>

이 메서드는 필터에서 인증이 성공했을 때 `SecurityContextHolder`에 저장할 `Authentication`을 생성하는 역할을 한다. `Authentication`을 구현하는 편한 방법은 `UsernamePasswordAuthenticationToken`을 사용하는 것이다. `UsernamePasswordAuthenticationToken`의 구조는 아래와 같다.

<br>

![image](https://user-images.githubusercontent.com/13410737/184543386-f3833a89-ea93-4cc4-9798-ee546b7790a7.png){: .align-center}

<br>

`UsernamePasswordAuthenticationToken`은 `AbstractAuthenticationToken`을 상속받고 있는데 `AbstractAuthenticationToken`은 `Authentication` 인터페이스의 구현체이다.

이 토큰 클래스를 사용하려면 초기화를 위한 `UserDetails`가 필요하다. 이 객체는 `UserDetailsService`를 통해 가져오게 된다. 이때 사용되는 `Username` 값은 아래와 같이 구현된다.

<br>

```java
public String getUsername(String token){
    LOGGER.info("[getUsername] 토큰 기발 회원 구별 정보 추출");
    String info = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token).getBody().getSubject();
    LOGGER.info("[getUsername] 토큰 기반 회원 구별 정보 추출 완료, info : {}", info);
    return info;
}
```

<br>

`Jwts.parser()`를 통해 `secretKey`를 설정하고 클레임을 추출해서 토큰을 생성할 때 넣었던 `sub`값을 추출한다.

그다음으로 살펴볼 메서드는 `resolveToken()` 이다. `resolveToken()` 메서드는 아래와 같이 구현돼 있다.

<br>

```java
public String resolveToken(HttpServletRequest request){
    LOGGER.info("[resolveToken] HTTP 헤더에서 Token 값 추출");
    return request.getHeader("X-AUTH-TOKEN");
}
```

<br>

이 메서드는 `HttpSevletRequest`를 파라미터로 받아 헤데 값으로 전달된 '`X-AUTH-TOKEN`' 값을 가져와 리턴한다. 클라이언트가 헤더를 통해 애플리케이션 서버로 JWT 토큰 값을 전달해야 정상적인 추출이 가능하다. 헤더의 이름은 임의로 변경할 수 있다.

마지막으로 볼 메서드는 `validateToken()` 메서드로 아래와 같이 작성돼 있다.

<br>

```java
public boolean validateToken(String token){
    LOGGER.info("[validateToken] 토큰 유효 체크 시작");
    try{
        Jws<Claims> claims = Jwts.parser().setSigningKey(secretKey).parseClaimsJws(token);

        return !claims.getBody().getExpiration().before(new Date());
    }catch(Exception e) {
        LOGGER.info("[validateToken] 토큰 유효 체크 예외 발생");
        return false;
    }
}
```

<br>

이 메서드는 토큰을 전달받아 클레임의 유효기간을 체크하고 `boolean` 타입의 값을 리턴하는 역할을 한다.



### JwtAuthenticationFilter 구현

`JwtAuthenticationFilter`는 JWT 토큰을 인증하고 `SecurityContextHolder`에 추가하는 필터를 설정하는 클래스이다. 우선 전체 코드를 보면 아래와 같아.

<br>

```java
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final Logger LOGGER = LoggerFactory.getLogger(JwtAuthenticationFilter.class);
    private final JwtTokenProvider jwtTokenProvider;

    public JwtAuthenticationFilter(JwtTokenProvider jwtTokenProvider){
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest servletRequest,
                                    HttpServletResponse servletResponse,
                                    FilterChain filterChain) throws ServletException, IOException {
        String token = jwtTokenProvider.resolveToken(servletRequest);
        LOGGER.info("[doFilterInternal] token 값 추출 완료. token : {}", token);

        LOGGER.info("[doFilterInternal] token 값 유효성 체크 시작");
        if(token != null && jwtTokenProvider.validateToken(token)){
            Authentication authentication = jwtTokenProvider.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(authentication);
            LOGGER.info("[doFilterInternal] token 값 유효성 체크 완료")
        }

        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

<br>

코드는 비교적 간단하다. 먼저 살펴볼 부분은 1번 줄의 `OncePerReqeustFilter`이다. 스프링 부트에서는 필터를 여러 방법으로 구현할 수 있는데, 가장 편한 구현 방법은 필터를 상속받아 사용하는  것이다. 대표적으로 많이 사용되는 상속 객체는 `GenericFilterBean`과 `OncePerReqeustFilter`이다. `GenericFilterBean`을 상속 받아 구현하면 아래와 같이 구현할 수 있다.

<br>

```java
public class JwtAuthenticationFilter extends GenericFilterBean{
    
    private final Logger LOGGER = LoggerFactory.getLogger(JwtAuthenticationFilter.class);
    private final JwtTokenProvider jwtTokenProvider;
    
    public JwtAuthenticationFilter(JwtTokenProvider jwtTokenProvider){
        this.jwtTokenProvider = jwtTokenProvider;
    }
    
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
                        FilterChain filterChain) throws IOException, ServletException{
        String token = jwtTokenProvider.resolveToken((HttpServletReqeust)servletRequest);
        LOGGER.info("[doFilterInternal] token 값 추출 완료. token : {}", token);
        
        LOGGER.info("[doFilterInternal] token 값 유효성 체크 시작");
        if(token != null && jwtTokenProvider.validateToken(token)){
            Authentication authentication = jwtTokenProvider.getAuthentication(token);
            SecurityContextHolder.getContext().setAuthentication(authentication);
            LOGGER.info("[doFilterInternal] token 값 유효성 체크 완료")
        }

        filterChain.doFilter(servletRequest, servletResponse);
    }
}
```

<br>

이번 예제에서는 `OncePerRequestFilter` 를 상속받아 코드를 사용하겠다.

<br>

> `GenericFilterBean`은 기존 필터에서 가져올 수 없는 스프링의 설정 정보를 가져올 수 있게 확장된 추상 클래스이다. 다만 서블릿은 사용자의 요청을 받으면 서블릿을 생성해서 메모리에 저장해두고 동일한 클라이언트의 요청을 받으면 재활용하는 구조여서 `GenericFilterBean`을 상속받으면 `RequestDispatcher`에 의해 다른 서블릿으로 디스패치되면서 필터가 두 번 실행되는 현상이 발생할 수 있다.
>
> 이 같은 문제를 해결하기 위해 등장한 것이 `OncePerRequestFilter`이며, 이 클래스 역히 `GenericFilterBean`을 상속받고 있다. 다만 이 클래스를 상속받아 구현한 필터는 매 요청마다 한 번만 실행되게끔 구현된다.
>
> 자세한 `GenericFilterBean`과 `OncePerRequestFilter`의 차이에 대해 알아보는 것을 권장한다.

<br>

10~25번 줄을 보면 `OncePerRequestFilter`로 부터 오버라이딩한 `doFilterInternal()` 메서드가 있다. 24번 줄의 `doFilter()`메서드는 서블릿을 실행하는 메서드인데, `doFilter()`메서드를 기준으로 앞에 작성한 코드는 서블릿이 실행되기 전에 실행되고, 뒤에 작성한 코드는 서블릿이 실행된 후 실행된다.

메서드의 내부 로직을 보면 `JwtTokenProvider`를 통해 `servletRequest`에서 토큰을 추출하고, 토큰에 대한 유효성을 검사한다. 토큰이 유효하다면 `Authentication` 객체를 생성해서 `SecurityContextHolder`에 추가하는 작업을 수행한다.



