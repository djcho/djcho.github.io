---
title : "[SpringBoot][Chapter5.2] API 작성법 - GET API만들기"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij]
toc: true
toc_sticky : true
published : true
date : 2022-07-19
last_modified_at : 2022-07-19
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



# GET API 만들기

GET API는 웹 애플리케이션 서버에서 값을 가져올 때 사용하는  API이다. GET API를 작성하는 방법은 다양하며, 이번 장에서는 애플리케이션으로 들어오는 여러 요청에 대한 처리 방법의 하나로서 소개한다.

실무에서는 HTTP 메서드에 따라 컨트롤러 클래스를 구분하지 않지만 여기서는 메서드별로 클래스를 생성한다. 아래와 같이 `controller` 패키지를 생성하고 `GetController` 클래스를 생성한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179662114-e2a45643-ef47-486b-ba53-19ef4d8f3117.png){: .align-center}

<br>

그리고 아래와 같이 컨트롤러에 `@RestController`  와 `@RequestMapping`을 붙여 내부에 선언되는 메서드에서 사용할 공통 URL을 설정한다.

<br>

```java
//컨트롤러 클래스에 @RestController와 @RequestMapping 설정
package com.springboot.api.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/get-api")
public class GetController {
}
```

<br>

클래스 수준에서 `@RequestMapping`을 설정하면 내부에 선언한 메서드의 URL 리소스 앞에 `@RequestMapping`의 값이 공통 값으로 추가된다.

<br>

## @RequestMapping으로 구현하기

`@RequestMapping` 어노테이션을 별다른 설정 없이 선언하면 HTTP의 모든 요청을 받는다. 그러나 GET형식의 요청만 받기 위해서는 어노테이션에 별도 설정이 필요하다. 아래와 같이 `@RequestMapping` 어노테이션의 `method`요소의 값을 `RequestMethod.Get`으로 설정하면 요청 형식을 GET으로만 설정할 수 있다.

<br>

```java
package com.springboot.api.controller;

import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/api/v1/get-api")
public class GetController {
    //http://localhost:8080/api/v1/get-api/hello
    @RequestMapping(value = "/hello", method = RequestMethod.GET)
    public String getHello(){
        return "Hello World";
    }
}
```

<br>

스프링 4.3 버전 이후로는 새로 나온 아래의 어노테이션을 사용하기 때문에 `@RequestMapping` 어노테이션은 더 이상 사용되지 않는다. 앞으로 예제에서는 특별히 `@RequestMapping`을 활용해야 하는 내용이 아니라면 아래의 각 HTTP 메서드에 맞는 어노테이션을 사용할 예정이다.

- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`

<br>

예제에서 작성한 메서드를 호출하고 싶다면 Talend API Tester에 아래와 같이 설정하고 'Send'를 눌러주면 된다. 주소는 예제에 작성된 메서드 위에 있는 주석과 동일하다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179663084-4cad9bc2-c53d-4cd0-9c45-bc1a92e2cf29.png){: .align-center}

<br>



## 매개변수가 없는 GET메서드 구현

별도의 매개변수 없이 GET API를 구현하는 경우 아래와 같이 코드를 작성할 수 있다.

<br>

```java
    //http://localhost:8080/api/v1/get-api/name
    @GetMapping(value = "/name")
    public String getName(){
        return "Flature";
    }
```

<br>

매개변수가 없는 요청은 위 예제 코드의 1번 줄에 나와 있는 URL을 그대로 입력하고 요청할 때 스프링 부터 애플리케이션이 정해진 응답을 반환한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179663463-1d072f43-1b29-4041-9154-101692ea940a.png){: .align-center}

<br>



## @PathVariable을 활요한 GET메서드 구현

실무 환경에서는 매개변수를 받지 않는 메서드가 거의 쓰이지 않는다.. 웹 통신의 기본 목적은 데이터를 주고 받는 것이기 때문에 대부분 매개변수를 받는 메서드를 작성하게 된다.  매개변수를 받을 때 자주 쓰이는 방법 중 하나는 URL 자체에 값을 담아 요청하는 것이다. 아래는 URL에 값을 담아 전달되는 요청을 처리하는 방법을 보여준다.

<br>

```java
    //http://localhost:8080/api/v1/get-api/variable1/{String 값}
    @GetMapping(value = "variable1/{variable}")
    public String getVariable1(@PathVariable String variable){
        return variable;
    }
```

<br>

1번 줄에 있는 요청 예시 URL을 보면 이 메서드는 중괄호(`{}`)로 표시된 위치의 값을 받아 요청하는 것을 알 수 있다.(실제 요청 시 중괄호는 들어가지 않으며 값만 존재) 값을 간단히 전달할 때 주로 사용하는 방법이며, GET 요청에서 많이 사용 된다.

이러한 방식으로 코드를 작성할 때는 몇 가지 지켜야 할 규칙이 있다. `@GetMapping` 어노테이션의 값으로 URL을 입력할 때 중괄호를 사용해 어느 위치에서 값을 받을 수 지정해야 한다. 또한 메서드의 매개변수와 그 값을 연결하기 위해 3번 줄과 같이 `@PathVariable`을 명시하며, `@GetMapping` 어노테이션과 `@PathVariable`에 지정된 변수의 이름을 동일하게 맞춰야 한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179664089-79feaf4e-81aa-48df-8296-6ceffccfafd0.png){: .align-center}

<br>

만약 `@GetMapping` 어노테이션에 지정한 변수의 이름과 메서드 매개변수의 이름을 동일하게 맞추기 어렵다면 `@PathVariable` 뒤에 괄호를 열어 `@GetMpping` 어노테이션의 변수명을 지정한다.

<br>

```java
    //http://localhost:8080/api/v1/get-api/variable2/{String 값}
    @GetMapping(value = "variable2/{variable}")
    public String getVariable2(@PathVariable("variable") String var){
        return var;
    }
```

<br>2번 중에 적혀 있는 변수명인 `variable`과 3번 줄에 적힌 매개변수명인 `var`가 서로 일치하지 않는 상황에서 두 값을 매핑하는 방법을 보여준다. `@PathVariable`에는 변수의 이름을 특정할 수 있는 `value`요소가 존재하며,  이 위치에 변수 이름을 정의하면 매개변수와 매핑할 수 있다. 3번 줄의 `@PathVariable`사용법을 좀 더 풀어쓰면 다음과 같다.

<br>

```java
public String getVariable2(@PathVariable(value = "variable") String var) {
```

![image](https://user-images.githubusercontent.com/13410737/179664582-b1c527e1-941c-4029-a7a8-c53a2c09ec82.png){: .align-center}<br>

<br>

## @RequestParam을 활용한 GET 메서드 구현

GET 요청을 구현할 때 앞에서 살펴본 방법처럼 URL 경로에 값을 담에 요청을 보내는 방법 외에도 쿼리 형식으로 값을 전달할 수도 있다. 즉, URI에서 `'?'`를 기준으로 우측에 `'{키} = {값}'` 형태로 구성된 요청을 전송하는 방법이다. 애플리케이션에서 이 같은 형식을 처리하려면 `@RequestParam`을 활용하면 되는데, 아래와 같이 매개변수 부분에 `@RequestParam` 어노테이션을 명시해 쿼리 값과 매핑하면 된다.

<br>

```java
    //http://localhost:8080/api/v1/get-api/request1?name=value1&email=value2&organization=value3
    @GetMapping(value = "variable2/request1")
    public String getRequestParam1(
            @RequestParam String name,
            @RequestParam String email,
            @RequestParam String organization) {
        return name + " " + email + " " + organization;
    }
```

<br>

1번 줄을 보면 '?' 오른쪽에 쿼리스트링(query string)이 명시되어 있다. 쿼리스트링에는 키(변수의 이름)가 모두 적혀 있기 때문에 이 값을 기준으로 메서드의 매개변수에 이름을 매핑하면 값을 가져올 수 있다. 키와 `@RequestParam`뒤에 적는 이름을 동일하게 설정하기 어렵다면 `@PathVariable` 예제에서 사용한 방법처럼 `value` 요소로 매핑한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179665489-7c4c4317-aef3-4cfe-9245-9c27f1c9e3de.png){: .align-center}

<br>

만약 쿼리스트링에 어떤 값이 들어올지 모른다면 아래와 같이`Map`객체를 활용할 수도 있다.

<br>

```java
    //http://localhost:8080/api/v1/get-api/request2?key1=value1&key2=value2
    @GetMapping(value = "/request2")
    public String getRequestParam2(@RequestParam Map<String, String> param){
        StringBuilder sb = new StringBuilder();
        
        param.entrySet().forEach(map -> {
            sb.append(map.getKey() + " : " + map.getValue() + "\n");
        });
        
        return sb.toString();
    }
```

<br>

위의 형태로 코드를 작성하면 값에 상관없이 요청을 받을 수 있다. 예를 들어, 회원 가입 관련 API에서 사용자는 회원 가입을 하면서 ID 값은 필수 항목이 아닌 취미 값은 선택 항목에 대해서는 값을 기입하지 않는 경우가 있다. 이러한 경우에는 매개변수의 항목이 일정하지 않을 수 있어 Map 객체로 받는 것이 효율적이다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179665971-3880c4da-06a4-49af-9c5b-17767ac50266.png){: .align-center}

<br>

> **💡 Tip. URI와 URL의 차이**
>
> URL은 우리가 흔히 말하는 웹 주소를 의미하며, 리소스가 어디에 있는지 알려주기 위한 경로를 의미한다. 반면 URI는 특정 리소스를 식별할 수 있는 식별자를 의미한다.
>
> 웹에서는 URL을 통해 리소스가 어느 서버에 위치해 있는지 알 수 있으며, 그 서버에 접근해서 리소스에 접근하기 위해서는 대부분 URI가 필요하다.

<br>

## DTO 객체를 활용한 GET 메서드 구현

### DTO란?

DTO는 Data Transfer Object의 약자로, 다른 레이어 간의 데이터 교환에 활용된다. 간략하게 설명하자면 클래스 및 인터페이스를 호출하면서 전달하는 매개변수로 사용되는 데이터 객체이다.

DTO는 데이터를 교환하는 용도로만 사용하는 객체이기 때문에 DTO에는 별도의 로직이 포함되지 않는다. 이 책에서는 6장의 데이터베이스를 연동하는 작업을 시작으로 본격적인 데이터를 다룰 예정이므로 DTO 클래스의 역할은 이때 좀 더 다루도록 하겠다.

<br>

> **💡 Tip. DTO와 VO**
>
> DTO와 VO(Value Object)의 역할을 서로 엄밀하게 구분하지 않고 새용할 때가 많습니다. 이렇게 해도 대부분의 상황에서는 큰 문제가 발생하지 않지만 정확하게 구분하자면 역할과 사용법에서 차이가 있다.
>
> 먼저 VO는 데이터 그 자체로 의미가 있는 객체를 의미한다. VO의 가장 특징적인 부분은 읽기전용(Read-Only)으로 설계한다는 점이다. 즉, VO는 값을 변경할 수 없게 만들어 데이터의 신뢰성을 유지해야 한다.
>
> DTO는 데이터 전송을 위해 사용되는 데이터 컨테이너로 볼 수 있다. 즉, 같은 애플리케이션 내부에서 사용되는 것이 아니라 다른 서버(시스템)으로 전달하는 경우에 사용된다.
>
> 본문에서는  DTO가 다른 에이어 간 데이터 교환에 활용된다고 설명했다. 여기서 레이어는 애플리케이션 내부에 정의된 레이어일 수도 있고 인프라 관점에서의 서버 아키텍처 상의 레이어일 수도 있다. 이러한 개념의 혼용이 DTO와 VO의 차이를 흐리게 만든다.
>
> 이 같은 용어와 개념을 정확하게 사용하는 것도 중요하지만 팀 내부적으로 용어나 개념의 역할 범위를 설정하고 합의해서 사용 한다면 업무를 효율적으로 처리하는 데 도움이 된다.

DTO 객체는 아래와 같이 장성할 수 있다. 이 클래스 파일은 예제 프로젝트에서 com.springboot.api 패키지 하단에 `dto`라는 패키지를 생성한 후 그 안에 생성하면 된다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179666977-5647fa6e-ccd5-488f-80d8-00db27ac4ea9.png){: .align-center}

<br>

```java
package com.springboot.api.dto;

public class MemberDto {
    private String name;
    private String email;
    private String organization;
    
    public String getName(){
        return name;
    }
    
    public void setName(String name){
        this.name = name;
    }
    
    public String getEmail(){
        return email;
    }
    
    public void setEmail(String email){
        this.email = email;
    }
    
    public String getOrganization(){
        return organization;
    }
    
    public void setOrganization(String organization){
        this.organization = organization;
    }
    
    @Override
    public String toString(){
        return "MemberDto{" + 
                "name='" + name + '\'' + 
                ", email ='" + email + '\'' + 
                ", organization='" + organization + '\'' +
                '}';
    }
}
```

<br>

DTO 클래스에는 전달하고자 하는 필드 객체를 선언하고 getter/setter 메서드를 구현한다. DTO 클래스에 선언된 필드는 컨트롤러의 메서드에서 쿼리 파라미터의 키와 매핑된다. 즉, 쿼리스트링의 키가 정해져 있지만 받아야 할 파라미터가 많을 경우에는 아래와 같이 DTO 객체를 활용해 코드의 가독성을 높일 수 있다.

<br>

```java
    //http://localhost:8080/api/v1/get-api/request3?name=value1&email=value2&organization=value3
    @GetMapping(value="/request3")
    public String getRequestParam3(MemberDto memberDto){
        //return memberDto.getName() + " " + memberDto.getEmail() + " " + memberDto.getOrganization();
        return memberDto.toString();
    }
```

<br>

1번 줄은 이전 예제와 동일한 형태의 쿼리스트링을 가진다. 다만 3번 줄과 같이 코드의 양을 줄일 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179668088-75a1b543-2c16-4b4b-89d2-bd77e650be44.png){: .align-center}

<br>
