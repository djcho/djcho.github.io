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
last_modified_at : 2022-08-16
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
            LOGGER.info("[doFilterInternal] token 값 유효성 체크 완료");
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



### SecurityConfiguration 구현

지금까지 실습을 통해 스프링 시큐리티를 적용하기 위한 컴포넌트를 구현했다. 이제 스프링 시큐리티와 관련된 설정을 진행하겠다. 스프링 시큐리티를 설정하는 대표적인 방법은 `WebSecurityConfigureAdapter`를 상속받는 `Configuration` 클래스를 구현하는 것이다. 전체적인 `SecurityConfiguration` 클래스의 구현은 아래와 같다.

<br>

```java
@Configuration
public class SecurityConfiguration extends WebSecurityConfigurerAdapter {

    private final JwtTokenProvider jwtTokenProvider;

    @Autowired
    public SecurityConfiguration(JwtTokenProvider jwtTokenProvider){
        this.jwtTokenProvider = jwtTokenProvider;
    }

    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception{
        httpSecurity.httpBasic().disable()

                .csrf().disable()

                .sessionManagement()
                .sessionCreationPolicy(
                        SessionCreationPolicy.STATELESS)

                .and()
                .authorizeRequests()
                .antMatchers("*/sign-api/sign-in", "sign-api/sign-up",
                        "/sign-api/exception").permitAll()
                .antMatchers(HttpMethod.GET, "product/**").permitAll()

                .antMatchers("**exception**").permitAll()

                .anyRequest().hasRole("ADMIN")

                .and()
                .exceptionHandling().accessDeniedHandler(new CustomAccessDeniedHandler())
                .and()
                .exceptionHandling().authenticationEntryPoint(new CustomAuthenticationEntryPoint())

                .and()
                .addFilterBefore(new JwtAuthenticationFilter(jwtTokenProvider),
                        UsernamePasswordAuthenticationFilter.class);
    }

    @Override
    public void configure(WebSecurity webSecurity){
        webSecurity.ignoring().antMatchers("/v2/api-docs", "/swagger-resources/**",
                "/swgger-ui.html", "/webjars/**", "/swagger/**", "/sign-api/exception");
    }
}
```

<br>

위 코드의 구조를 살펴보자. `SecurityConfiguration` 클래스의 주요 메서드는 두 가지로, `WebSecurity` 파라미터를 받은 `configure()` 메서드와 `HttpSecurity` 파라미터를 받은 `configure()`메서드이다.

먼저 살펴볼  메서드는 11~39번 줄의 `HttpSecurity`를 설정하는 `configure()` 메서드이다. 스프링 시큐리티의 설정은 대부분 `HttpSecurity`를 통해 진행된다. 대표적인 기능은 다음과 같다.

- 리소스 접근 권한 설정
- 인증 실패 시 발생하는 예외 처리
- 인증 로직 커스터마이징
- csrf, cors 등의 스프링 시큐리티 설정

지금부터 `configure()` 메서드에 작성돼 있는 코드를 설정별로 구분해 설명하겠다. 모든 설정은 전달 받은 `HttpSecurity` 에 설정된다.

<br>

**`httpBasic().disable()`**

UI를 사용하는 것을 기본값으로 가진 시큐리티 설정은 비활성화한다.

<br>

**`csrf().disable()`**

REST API에서는 CSRF 보안이 필요 없기 때문에 비활성화하는 로직이다. CSRF는 Cross-Site Request Forgery의 줄임말로 '사이트 간 요청 위조'를 의미한다. '사이트 간 요청 위조'란 웹 애플리케이션의 취약점 중 하나로서 사용자가 자신의 의지와 무관하게 웹 애플리케이션을 대상으로 공격자가 의도한 행동을 함으로써 특정 페이지의 보안을 취약하게 한다거나 수정, 삭제 등의 작업을 하는 공격 방법이다. 스프링 시큐리티의 `crsf()`메서드는 기본적으로 CSRF 토큰을 발급해서 클라이언트로부터 요청을 받을 때마다 토큰을 검증하는 방식으로 동작한다. 브라우저 사용 환경이 아니라면 비활성화해도 크게 문제가 되지 않는다.

<br>

**`sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)`**

REST PI 기반 애플리케이션의 동작 방식을 설정한다. 지금 진행 중인 프로젝트에서는 JWT 토큰으로 인증을 처리하며, 세션은 사용하지 않기 때문에 STATELESS로 설정한다.

<br>

**`authorizeRequest()`**

애플리케이션에 들어오는 요청에 대한 사용 권한을 체크한다. 이어서 사용한 `antMatchers()` 메서드는 `antPattern` 을 통해 권한을 설정하는 역할을 한다. 23~29번 줄에 설정된 내용은 다음과 같은 설정을 수행한다.

- '/sign-api/sign-in', '/sign-api/sign-up', '/sign-api/exception' 경로에 대해서는 모두에게 허용한다.
- '/product' 로 시작하는 경로의 GET 요청을 모두 허용한다.
- 'exception' 단어가 들어간 경로는 모두 허용한다.
- 기타 요청은 인증된 권한을 가진 사용자에게 허용한다.

<br>

**`exceptionHandling().accessDenieHandler()`**

 권한을 확인하는 과정에서 통과하지 못하는 예외가 발생할 경우 예외를 전달한다.

<br>

**`exceptionHandling().authenticationEntryPoint()`**

인증 과정에서 예외가 발생할 경우 예외를 전달한다.

<br>

각 메서드는 `CustomAccessDeniedHandler()`와 `CustomAuthenticationEntryPoint`로 예외를 전달한다. 각 클래스는 ㅇ니후 내용에서 살펴보겠다.

앞에서 이야기한 것처럼 스프링 시큐리티는 각각의 역할을 수행하는 필터들이 체인 형태로 구성돼 순서대로 동작한다. 이책에서는 실습을 통해 JWT로 인증하는 필터를 생성했으며, 이 필터의 등록은 `HttpSecurity` 설정에서 진행한다. 37~38번 줄의 `addFilterBefore()` 메서드를 사용해 어느 필터 앞에 추가할 것인지 설정할 수 있는데, 현재 구현돼 있는 설정은 스프링 시큐리티에서 인증을 처리하는 필터인 `UsernamePasswordAuthenticationFilter`앞에 앞에서 생성한 `JwtAuthenticationFilter`를 추가하겠다는 의미이다. 추가된 필터에서 인증이 정상적으로 처리되면 `UsernamePasswordAuthenticationFilter`는 자동으로 통과되기 때문에 위와 같은 구성을 선택했다.

다음으로 볼 부분은 42~45번 줄의 `WebSecurity`를 사용하는 `configure()` 메서드이다. `WebSecurity`는 `HttpSecurity` 앞단에 적용되며, 전체적으로 스프링 시큐리티의 영향권 밖에 있다. 즉, 인증과 인가가 모두 적용되기 전에 동작하는 설정이다. 그렇기 때문에 다양한 곳에서 사용되지 않고 인증과 인가가 적용되지 않는 리소스 접근에 대해서만 사용한다. 예제에서는 Swagger에 적용되는 인증과 인가를 피하기 위해 `ignoring()` 메서드를 사용해 Swagger와 관련된 경로에 대한 예외 처리를 수행한다. 의미상 예외 처리라고 표현했지만 정확하게는 인증, 인가를 무시하는 경로를 설정한 것이다.



#### 커스텀 AccessDeniedhandler, AuthenticationEntryPoint 구현

앞에서 살펴본 예제에서는 인증과 인가 과정의 예외 상황에서 `CustomAccessDeniedHandler`와 `CustomAuthenticationEntryPoint`로 예외를 전달하고 있었다. 이번 절에서는 이렇란 클래스를 작성하는 방법을 알아보겠다.

먼저 `AccessDeniedHandler` 인터페이스의 구현체 클래스를 생성하겠다. 기본적으로 아래와 같이 `handle()` 메서드를 오버라이딩해서 구현하게 된다.

<br>

```java
@Component
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    private final Logger LOGGER = LoggerFactory.getLogger(CustomAccessDeniedHandler.class);
    
    @Override
    public void handle(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
                       AccessDeniedException e) throws IOException, ServletException {
        LOGGER.info("[handle] 접근이 막혔을 경우 경로 리다이렉트");
        httpServletResponse.sendRedirect("/sign-api/exception");
    }
}
```

<br>

`AccessDeniedException`은 액세스 권한이 없는 리소스에 접근할 경우 발생하는 예외이다. 이 예외를 처리하기 위해 `AccessDeniedHandler` 인터페이스가 사용되며, `SecurityConfiguration`에도 `exceptionHandling()` 메서드를 통해 추가했다. `AccessDeniedHandler`의 구현 클래스인 `CustomAccessDeniedHandler` 클래스는 `handle()` 메서드를 오버라이딩한다. 이 메서드는 `HttpServletRequest`와 `HttpSeervletResponse`, `AccessDeniedException`을 파라미터로 가져온다.

이번 예제에서는 `response`에서 리다이렉트하는 `sendRedirect()`메서드를 활용하는 방식으로 구현했다. 10번 줄에서 경로를 정의하면 다음과 같이 리다이렉트되어 정의한 예외 메서드가 호출되는 것을 볼 수 있다.

```
[INFO ] [http-nio-8080-exec-2] com.springboot.security.config.security.CustomAccessDeniedHandler
[handle] 접근이 막혔을 경우 경로 리다이렉트
[ERROR] [http-nio-8080-exec-6] com.springboot.security.controller.SignController ExceptionHandler 호출, null, 접근이 금지되었습니다.
```

로그에 출력된 스레드 번호를 보면 리다이렉트됐기 때문에 다른 스레드에서 동작하는 것을 볼 수 있따.

다음은 인증이 실패한 상황을 처리하는 `AuthenticationEntryPoint` 인터페이스를 구현한 `CustomAuthenticationEntryPoint` 클래스이다. 전체 코드는 아래와 같다.

<br>

```java
@Component
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    private final Logger LOGGER = LoggerFactory.getLogger(CustomAuthenticationEntryPoint.class);

    @Override
    public void commence(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
                         AuthenticationException e) throws IOException, ServletException {
        ObjectMapper objectMapper = new ObjectMapper();
        LOGGER.info("[commence] 인증 실패로 response.sendError 발생");

        EntryPointErrorResponse entryPointErrorResponse = new EntryPointErrorResponse();
        entryPointErrorResponse.setMsg("인증이 실패하였습니다.");

        httpServletResponse.setStatus(401);
        httpServletResponse.setContentType("application/json");
        httpServletResponse.setCharacterEncoding("utf-8");
        httpServletResponse.getWriter().write(objectMapper.writeValueAsString(entryPointErrorResponse));
    }
}
```

<br>

위 예제에서 사용된 `EntryPointErrorResponse` 는 `dto` 패키지에 아래와 같이 생성한다.

<br>

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class EntryPointErrorResponse {

    private String msg;
}
```

<br>

클래스 구조는 앞에서 본 `AccessDeniedHandler`와 크게 다르지 않으며, `commence()` 메서드를 오버라이딩해서 코드를 구현한다. 이 `commence()` 메서드는 `HttpServletRequest`, `HttpServletResponse`, `AuthenticationException`을 매개변수로 받는데, 이번 예제에서는 예외 처리를 위해 리다이렉트가 아니라 직접 `Response`를 생성해서 클라이언트에게 응답하는 방식으로 구현돼 있다.

컨트롤러에서는 응답을 위한 설정들이 자동으로 구현되기 때문에 별도의 작업이 필요하지 않았지만 여기서는 응닶값을 설정할 필요가 있다. 메시지를 담기 위해 `EntryPointErrorResponse` 객체를 사용해 메시지를 설정하고 ,`response`에 상태 코드(status)와 콘텐츠 타입(Content-type)등을 설정한 후 `ObjectMapper`를 사용해 `EntryPointErrorResponse` 객체를 바디 값으로 파싱한다.

굳이 메시지를 설정할 필요가 없다면 `commence()` 메서드 내부에 아래와 같이 한 줄만 작성하는 식으로 인증 실패 코드만 전달할 수 있다.

<br>

```java
@Override
public void commence(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse,
                     AuthenticationException e) throws IOException, ServletException {
    httpServletResponse.sendError(HttpServletResponse.SC_UNAUTHORIZED);
}
```

<br>

전체적인 코드 구조가 `AccessDeniedHandler`와 동일하기 때문에 지금까지 소개한 세 가지 응답을 구성하는 방식을 각 메서드에 혼용할 수도 있다.



### 회원가입과 로그인 구현

앞절에서 인증에 사용되는 `UserDetails` 인터페이스 구현제 클래스로 `User`엔티티를 생성했다. 지금까지는 `User` 객체를 통해 인증하는 방법을 구현했는데, 이번 절에서는 `User` 객체를 생성하기 위해 회원가입을 구현하고 `User` 객체로 인증을 시도하는 로그인을 구현하겠다.

회원가입과 로그인의 도메인은 Sign으로 통합해서 표현할 예정이며, 각각 Sign-up, Sign-in 으로 구분해서 기능을 구현한다. 먼저 서비스 레이어를 구현하겠다. `SignService` 인터페이스에 정의된 메서드는 아래와 같다.

<br>

```java
public interface SignService {
    
    SignUpResultDto signUp(String id, String password, String name, String role);
    
    SignInResultDto signIn(String id, String password) throws RuntimeException;
}
```

<br>

`SignService`인터페이스를 구현한 `SignServiceImpl` 클래스의 전체 코드는 다음과 같다.

<br>

```java
@Service
public class SignServiceImpl implements SignService {

    private final Logger LOGGER = LoggerFactory.getLogger(SignServiceImpl.class);

    public UserRepository userRepository;
    public JwtTokenProvider jwtTokenProvider;
    public PasswordEncoder passwordEncoder;

    @Autowired
    public SignServiceImpl(UserRepository userRepository, JwtTokenProvider
            jwtTokenProvider, PasswordEncoder passwordEncoder){
        this.userRepository = userRepository;
        this.jwtTokenProvider = jwtTokenProvider;
        this.passwordEncoder = passwordEncoder;
    }

    @Override
    public SignUpResultDto signUp(String id, String password, String name, String role) {
        LOGGER.info("[getSignUpResult] 회원 가입 정보 전달");
        User user;
        if(role.equalsIgnoreCase("admin")){
            user = User.builder()
                    .uid(id)
                    .name(name)
                    .password(password)
                    .roles(Collections.singletonList("ROLE_ADMIN"))
                    .build();
        }else {
            user = User.builder()
                    .uid(id)
                    .name(name)
                    .password(passwordEncoder.encode(password))
                    .roles(Collections.singletonList("ROLE_USER"))
                    .build();
        }

        User savedUser = userRepository.save(user);
        SignUpResultDto signUpResultDto = new SignInResultDto();

        LOGGER.info("[getSignUpResult] userEntity 값이 들어왔는지 확인 후 결과값 주입");
        if(!savedUser.getName().isEmpty()){
            LOGGER.info("[getSignUpResult] 정상 처리 완료");
            setSuccessResult(signUpResultDto);
        } else{
            LOGGER.info("[getSignUpResult] 실패 처리 완료");
            setFailResult(signUpResultDto);
        }
        return signUpResultDto;
    }

    @Override
    public SignInResultDto signIn(String id, String password) throws RuntimeException {
        LOGGER.info("[getSignInResult] signDataHandler 로 회원 정보 요청");
        User user = userRepository.getByUid(id);
        LOGGER.info("[getSignInResult] Id : {}", id);
        
        LOGGER.info("[getSignInResult] 패스워드 비교 수행");
        if(!passwordEncoder.matches(password, user.getPassword())){
            throw new RuntimeException();
        }
        LOGGER.info("[getSignInResult] 패스워드 일치");
        
        LOGGER.info("[getSignInResult] SignInResultDto 객체 생성");
        SignInResultDto signInResultDto = SignInResultDto.builder()
                .token(jwtTokenProvider.createToken(String.valueOf(user.getUid()),
                        user.getRoles()))
                .build();
        
        LOGGER.info("[getSignInResult] SignInResultDto 객체에 값 주입");
        setSuccessResult(signInResultDto);
        
        return signInResultDto;
    }
    
    private void setSuccessResult(SignUpResultDto result){
        result.setSuccess(true);
        result.setCode(CommonResponse.SUCCESS.getCode());
        result.setMsg(CommonResponse.SUCCESS.getMsg());
    }
    
    private void setFailResult(SignUpResultDto result){
        result.setSuccess(false);
        result.setCode(CommonResponse.FAIL.getCode());
        result.setMsg(CommonResponse.FAIL.getMsg());
    }
}
```

<br>

예제의 6~16번 줄에서는 회원가입과 로그인을 구현하기 위해 세 가지 객체에 대한 의존성 주입을 받는다.

18~36번 줄에서는 회원가입을 구현한다. 현재 애플리케이션에서는 `ADMIN`과 `USER`로 권한을 구분하고 있다. `signUp()` 메서드는 그에 맞게 전달받은 `role`객체를 확인해 `User` 엔티티의 `roles` 변수에 추가해서 엔티티를 생성한다. 패스워드는 암호화해서 저장해야 하기 때문에 `PasswordEncoder`를 활용해 인코딩을 수행한다. `PasswordEncoder`는 아래와 같이 별도의 `@Configuration` 클래스를 생성하고 `@Bean` 객체로 등록하도록 구현했다.

<br>

```java
@Configuration
public class PasswordEncodeConfiguration {

    @Bean
    public PasswordEncoder pAsswordEncoder(){
        return PasswordEncoderFactories.createDelegatingPasswordEncoder();
    }
}
```

<br>

위 예제는 빈 객체를 등록하기 위해서 생성된 클래스이기 때문에 `SecurityConfiguration` 클래스 같이 이미 생성된 `@Configuration` 클래스 내부에 4~7번 줄의 `passwordEncoder()` 메서드를 정의해도 충분하다.

이렇게 생성된 엔티티를 `UserRepository` 를 통해 저장한다. 실제 엔터프라이즈 환경에서는 회원가입을 위한 필드도 많고 코드도 복잡하겠지만 이 책에서는 부가적인 사항들은 모두 배제하고 회원가입 자체만 구현하겠다.

이제 회원으로 가입한 사용자의 아이디와 패스워드를 가지고 로그인을 수행할 수 있다. 로그인 메서드는 52~74번 줄에 구현돼 있다.

로그인은 미리 저장돼 있는 계정 정보와 요청을 통해 전달된 계정 정보가 일치하는지 확인하는 작업이다. `signIn()`메서드는 아이디와 패스워드를 입력받아 처리하게 된다. 내부 로직을 좀 더 자세히 살펴보겠다.

1. `id`를 기반으로 `UserRepository`에서 `User` 엔티티를 가져온다.
2. `PasswordEncoder`를 사용해 데이터베이스에 저장돼 있던 패스워드와 입력받은 패스워드가 일치하는지 확인하는 작업을 수행한다. 이번 예제에서는 패스워드가 일치하지 않아 예외를 발생시키는 `RuntimeException`을 사용했지만 별도의 커스텀 예외를 만들어서 사용하기도 한다.
3. 패스워드가 일치해서 인증을 통과하면 `JwtTokenProvider`를 통해 `id`와 `role` 값을 전달해서 토큰을 생성한 후 `Response`에 담아 전달한다.

<br>

76~86번 줄은 결과 데이터를 설정하는 메서드이다. 회원기입와 로그인 메서드에서 사용할 수 있게 설정돼 있으며, 각 메서드는 DTO를 전달받아 값을 설정한다. 이때 사용된 `CommonResponse` 클래스는 다음과 같이 작성돼 있다.

<br>

```java
public enum CommonResponse {

    SUCCESS(0, "Success"), FAIL(-1, "Fail");

    int code;
    String msg;

    CommonResponse(int code, String msg){
        this.code = code;
        this.msg = msg;
    }

    public int getCode() {
        return code;
    }

    public String getMsg() {
        return msg;
    }
}
```

<br>

이제 회원가입과 로그인을 API로 노출하는 컨트롤러를 생성해야 하는데 사실상 서비스 레이어로 요청을 전달하고 응답하는 역할만 수행하기 때문에 코드만 소개하겠다. `SignController`의 전체 코드는 아래와 같다.

<br>

```java
@RestController
@RequestMapping("/sign-api")
public class SignController {

    private final Logger LOGGER = LoggerFactory.getLogger(SignController.class);
    private final SignService signService;
    
    @Autowired
    public SignController(SignService signService){
        this.signService = signService;
    }
    
    @PostMapping(value = "/sign-in")
    public SignInResultDto signIn(
            @ApiParam(value = "ID", required = true) @RequestParam String id,
            @ApiParam(value = "Password", required = true) @RequestParam String password) throws RuntimeException{
        LOGGER.info("[signIn] 로그인을 시도하고 있습니다. id : {}, pw : ****", id);
        SignInResultDto signInResultDto = signService.signIn(id, password);
        
        if(signInResultDto.getCode() == 0){
            LOGGER.info("[signIn] 정상적으로 로그인되었습니다. id : {}, token : {}", id, 
                    signInResultDto.getToken());
        }
        return signInResultDto;
    }
    
    @PostMapping(value = "/sign-up")
    public SignUpResultDto signUp(
            @ApiParam(value = "ID", required = true) @RequestParam String id,
            @ApiParam(value = "비밀번호", required = true) @RequestParam String password,
            @ApiParam(value = "이름", required = true) @RequestParam String name,
            @ApiParam(value = "권한", required = true) @RequestParam String role) {
        LOGGER.info("[signUp] 회원가입을 수행합니다. id = {}, password : ****, name : {}, role : {}",
                id, name, role);
        SignUpResultDto signUpResultDto = signService.signUp(id, password, name, role);

        LOGGER.info("[signUp] 회원가입을 완료했습니다. id : {}", id);
        return signUpResultDto;
    }
    
    @GetMapping(value = "/exception")
    public void exceptionTest() throws RuntimeException{
        throw new RuntimeException("접근이 금지되었습니다.");
    }
    
    @ExceptionHandler(value = RuntimeException.class)
    public ResponseEntity<Map<String, String>> ExceptionHandler(RuntimeException e){
        HttpHeaders responseHeaders = new HttpHeaders();
        //responseHeaders.add(HttpHeaders.CONTENT_TYPE, "application/json");
        HttpStatus httpStatus = HttpStatus.BAD_REQUEST;
        
        LOGGER.error("ExceptionHandler 호출, {}, {}", e.getCause(), e.getMessage());
        
        Map<String, String> map = new HashMap<>();
        map.put("error type", httpStatus.getReasonPhrase());
        map.put("code", "400");
        map.put("message", "에러 발생");
        
        return new ResponseEntity<>(map, responseHeaders, httpStatus);
    }
}
```

<br>

클라이언트는 위와 같이 계정을 생성하고 로그인 과정을 거쳐 토큰값을 전달받음으로써 이 애플리케이션에서 제공하는 API 서비스를 사용할 준비를 마친다. `Response` 로 전달되는 `SignUpResultDto`와 `SignInResultDto` 클래스는 각각 아래와 같다.

<br>

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class SignUpResultDto {

    private boolean success;

    private int code;

    private String msg;
}
```

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@ToString
public class SignInResultDto extends SignUpResultDto{
    private String token;

    @Builder
    public SignInResultDto(boolean success, int code, String msg, String token) {
        super(success, code, msg);
        this.token = token;
    }

}
```

<br>

여기까지 구현이 완료되면 정상적으로 스프링 시큐리티가 동작하는 애플리케이션 환경이 완성된 것이다. 다음 절에서는 애플리케이션의 동작을 확인하겠다.

