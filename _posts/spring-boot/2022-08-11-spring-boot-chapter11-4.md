---
title : "[SpringBoot][Chapter11.4] 액추에이터 - 액추에이터 커스텀 기능 만들기"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Actuator]
toc: true
toc_sticky : true
published : true
date : 2022-08-11
last_modified_at : 2022-08-11
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





## 액추에이터 커스텀 기능 만들기

앞에서 살펴봤듯이 액추에이터는 다양한 정보를 가공해서 제공한다. 그 밖에 개발자의 요구사항에 맞춘 커스텀 기능 설정도 제공한다. 커스텀 기능을 개발하는 방식에는 크게 두 가지가 있다. 첫 번째는 기존 기능에 내용을 추가하는 방식이고, 두 번째는 새로운 엔드포인트를 개발하는 방식이다.



#### 정보 제공 인터페이스의 구현체 생성

액추에이터를 커스터마이징하는 가장 간단한 방법은 앞에서 `/info` 엔드포인트의 내용을 추가한 것처럼 `application.properties` 파일 내에 내용을 추가하는 것이다. 그러나 이 방법은 많은 내용을 담을 때는 관리 측면이 좋지 않다.

그래서 커스텀 기능을 설정할 때는 별도의 구현체 클래스를 작성해서 내용을 추가하는 방법을 많이 활용된다. 액추에이터에서는 `InfoContributor` 인터페이스를 제공하고 있는데, 이 인터페이스를 구현하는 클래스를 생성하면 된다. 아래와 같이 `InfoContributor` 인터페이스에 대한 구현 클래스를 생성한다.

<br>

```java
@Component
public class CustomInfoContributor implements InfoContributor {
    @Override
    public void contribute(Info.Builder builder) {
        Map<String, Object> content = new HashMap<>();
        content.put("code-info", "InfoContributor 구현체에서 정의한 정보입니다.");
        builder.withDetail("custom-info-contributor", content);
    }
}
```

<br>

새로 생성한 클래스를 `InfoContributor` 인터페이스의 구현체로 설정하면 `contributor` 메서드를 오버라이딩할 수 있게 된다. 이 메서드에서 파라미터로 받은 `Builder`객체는 액추에이터 패키지의 `Info`클래스 안에 정의돼 있는 클래스로서 `info` 엔드포인트에서 보여줄 내용을 담는 역할을 수행한다. 이렇게 객체를 가져와 6~8번 줄처럼 콘텐츠를 담아 `builder`에 포함하면 엔드포인트 출력 결과에서 확인할 수 있다. 아래와 같이 설정한 후 애플리케이션을 재가동해서 엔드포인트를 호출하면 아래와 같은 결과를 볼 수 있다.

<br>

```json
{
  "organization":{
    "name":"wikibooks"
  },
  "contact":{
    "email":"thinkground.flature@email.com",
    "phoneNumber":"010-1234-5678"
  },
  "custom-info-contributor":{
    "code-info":"InfoContributor 구현체에서 정의한 정보입니다."
  }
}
```

<br>

보다시피 기존 `application.properties` 에서 정의했던 속성값을 비롯해 구현체 클래스에서 포함한 내용이 추가된 것을 볼 수 있다.



### 커스텀 엔드포인트 생성

`@EndPoint` 어노테이션으로 빈에 추가된 객체들은 `@ReadOperation`, `@WriteOperation`, `@DeleteOperation` 어노테이션을 사용해 JMX나 HTTP를 통해 커스텀 엔드포인트를 노출시킬 수 있다. 만약 JMX에서만 사용하거나 HTTP에서만 사용하는 것을 제한하고 싶다면 `@JmxEndpoint`, `@WebEndpoint` 어노테이션을 사용하면 된다.

이 책에서는 간단하게 애플리케이션에 메모 기록을 남길 수 있는 기능을 엔드포인트로 생성하겠다. 아래와 같이 엔드포인트 클래스를 생성한다.

```java
@Component
@Endpoint(id = "note")
public class NoteEndpoint {
    
    private Map<String, Object> noteContent = new HashMap<>();
    
    @ReadOperation
    public Map<String, Object> getNote(){
        return noteContent;
    }
    
    @WriteOperation
    public Map<String, Object> writeNote(String key, Object value){
        noteContent.put(key, value);
        return noteContent;
    }
    
    @DeleteOperation
    public Map<String, Object> deleteNote(String key){
        noteContent.remove(key);
        return noteContent;
    }
}
```

<br>

위 코드의 2번 줄에서는 `@Endpoint` 어노테이션을 사용하고 있다. 이 어노테이션을 선언하면 액추에이터에 엔드포인트로 자동으로 등록되며 `id` 속성값으로 경로를 정의할 수 있다. 또한 `enableByDefault`라는 속성으로 현재 생성하는 엔드포인트의 기본 활성화 여부도 설정 가능하다. `enableByDefault`속성의 기본값은 `true`로서 값을 별도로 설정하지 않으면 활성화된다.

엔드포인트를 설정하는 클래스에는 `@ReadOpertation`, `@WriteOperation`, `@DeleteOperation` 어노테이션을 사용해 각 동작 메서드를 생성할 수 있다.

7~10번 줄에서는 `@ReadOperation` 어노테이션을 정의해 HTTP, GET 요청을 반응하는 메서드를 생성했다. 이 클래스에서는 `noteContent`라고 하는 `Map`타입의 객체를 전달하고 있다. 애플리케이셔을 재가동한 후 다음 엔드포인트를 호출해 보겠다.

<br>

------

<a href="http://localhost:8080/actuator/note">`http://localhost:8080/actuator/note`</a>

------

```json
{}
```

<br>

보다시피 아직 값을 넣지 않은 상태라서 JSON 형태의 빈 값이 표현된다. 그럼 `@WriteOperation` 동작을 확인하기 위해 아래와 같이 Talend API Tester를 POST호출을 시도한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183940337-d3879208-66eb-4433-bca6-c4390c7a5dcc.png){: .align-center}

<br>

위와 같이 메서드에서 받고자 하는 파라미터의 이름을 JSON의 키 값으로 설정하고, 그에 해당하는 값을 후가할 수 있다. 이렇게 값을 추가하고 다시 GET 메서드로 커스텀 엔드포인트인 `/note`를 호출하면 다음과 같은 결괏값이 확인된다.

<br>

```json
{
  "description": "설명 부분을 기입한다."
}
```

<br>

이번에는 `@DeleteOperation` 어노테이션이 선언된 메서드를 호출해 보겠다. `@DeleteOperation` 어노테이션이 선언된 메서드는 DELETE 호출을 통해 사용할 수 있으며, [Query Parameters] 에 값을 넣어 호출한다. 현재 구성된 메서드에서는 `key` 값을 받아 `Map`객체에서 해당 값을 삭제하는 작업을 수행하게 된다. 아래와 같이 호출을 시도한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/183941473-adda824d-9131-46ef-a996-326b617c37a0.png){: .align-center}

<br>

위와 같이 호출하면 아래와 같이 지정한 키 값이 상제된 결과를 확인할 수 있다.

지즘까지의 액추에이터의 커스텀 엔드포인트를 생성해봤다. 커스텀 엔드포인트는 코드로 구현되기 때문에 더욱 확장성 있는 기능을 개발할 수 있다.

<br>

> **💡 Tip.** 
>
> 스프링 부트 액추에이터의 자세한 내용은 공식 페이지에서 확인할 수 있다.
>
> - <a href="https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html">https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html</a>
