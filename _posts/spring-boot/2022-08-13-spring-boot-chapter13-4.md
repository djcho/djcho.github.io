---
title : "[SpringBoot][Chapter13.4] 서비스의 인증과 권한 부여 - JWT"
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







# JWT

JWT(JSON Web Token)은 당사자 간에 정보를 JSON 형태로 안전하게 전송하기 위한 토큰이다. JWT는 URL로 이용할 수 있는 문자열로만 구성돼 있으며, 디지털 서명이 적용돼 있어 신뢰할 수 있다. JWT는 주고 서버와의 통신에서 권한 인가를 위해 사용된다. URL에서 사용할 수 있는 문자열로만 구성돼 있기 때문에 HTTP 구성요소 어디든 위치할 수 있다.



## JWT 구조

JWT는 점('.')으로 구분된 아래의 세 부분으로 구성된다.

- 헤더(Header)
- 내용(Payload)
- 서명(Signature)

<br>

따라서 JWT는 일반적으로 아래와 같은 형식을 띠고 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/184501127-1736c050-25df-45f9-9756-b4029bc524de.png){: .align-center}

<br>

### 헤더

JWT의 헤더는 검증과 관련된 내용을 담고 있다. 헤더에는 아래와 같이 두 가지 정보를 포함하고 있는데, 바로 `alg`와 `typ` 속성이다.

```json
{
    "alg" : "HS256",
    "typ" : "JWT"
}
```

<br>

`alg` 속성에서는 해싱 알고리즘을 지정한다. 해싱 알고리즘은 보통 SHA256 또는 RSA를 사용하며, 토큰을 검증할 때 사용되는 서명 부분에서 사용된다. 위 예제에 작성돼 있는 HS256은 'HMAC SHA256' 알고리즘을 사용한다는 의미이다. 그리소 `typ` 속성에는 토큰의 타입을 지정한다.

이렇게 완성된 헤더는 Base64Url 형식으로 인코딩돼 사용된다.



### 내용

JWT의 내용에는 토큰에 담는 정보를 포함한다. 이곳에 포함된 속성들은 클레임(Claim)이라 하며, 크게 세 가지로 분류된다.

- 등록된 클레임(Registered Claims)
- 공개 클레인(Public Claims)
- 비공개 클레인(Private Claims)

<br>

등록된 클레임은 필수는 아니지만 토큰에 대한 정보를 담기 위해 이미 이름이 정해져 있는 클레임을 뜻한다. 등록된 클레임은 다음과 같이 정의돼 있다.

- `iss`: JWT의 발급자(Issuer) 주체를 나타낸다. `iss`의 값은 문자열이나 URI를 포함하는 대소문자를 구분하는 문자열이다.
- `sub` : JWT의 제목(Subject)이다.
- `aud` JWT의 수신인(Audience)이다. JWT를 처리하려는 각 주체는 해당 값으로 자신을 식별해야 한다. 요청을 처리하는 주체가 'aud' 값으로 자신을 식별하지 않으면 JWT는 거부된다.
- `exp` : JWT의 만료시간(Expiration)이다. 시간은 `NubericDate` 형식으로 지정해야 한다.
- `nbf` : 'Not Before' 를 의미한다.
- `iat` : JWT가 발급된 시간(Issued at)이다.
- `jti`: JWT의 식별자(JWT ID)이다. 주고 중복 처리를 방지하기 위해 사용된다.

<br>

공개 클레임은 키 값을 마음대로 정의할 수 있다. 다만 충돌이 발생하지 않을 이름으로 설정해야 한다.

비공개 클레임은 통신 간에 상호 합의되고 등록된 클레임과 공개된 클레임이 아닌 클레임을 의미한다. 내용의 예는 아래와 같다.

<br>

```json
{
    "sub" : "wikibooks payload",
    "exp" : "1602076408",
    "userId" : "wikibooks",
    "username" : "flature"
}
```

<br>

이렇게 완성된 내용은 Base64Url 형식으로 인코딩되어 사용된다.



### 서명

JWT의 서명 부분은 인코딩된 헤더, 인코딩된 내용, 비밀키, 헤더의 알고리즘 속성값을 가져와 생성된다. 예를 들어 HMAC SHA256 알고리즘을 사용해서 서명을 생성한다면 아래와 같은 방식으로 생성된다.

<br>

```java
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret
  )
```

<br>

서명은 토큰의 값들을 포함해서 암호화하기 때문에 세시지가 도중에 변경되지 않았는지를 확인할 때 사용된다.

<br>

## JWT 디버거 사용하기

JWT 공식 사이트에서는 더욱 쉽게 JWT를 생성해볼 수 있따. 웹 브라우저에서 다음 URL로 접속하면 아래와 같은 화면을 볼 수 있다.

- <a href="https://jwt.io/#debugger-io">https://jwt.io/#debugger-io</a>

![image](https://user-images.githubusercontent.com/13410737/184501651-e5fb13db-6ad3-4d65-ad8a-77f049b4b512.png){: .align-center}

<br>

이 화면은 Encoded와 Decoded로 나눠져 있으며, 양측의 내용이 일치하는지 사이트에서 확인할 수도 있고 Decoded의 내용을 변경하면 Encoded의 콘텐츠가 자동으로 반영된다.

<br>

> **💡 Tip.**
>
> JWT와 관련된 자세한 내용은 공식 사이트에서 확인할 수 있다.
>
> - <a href="https://jwt.io/introduction/">https://jwt.io/introduction/</a>

