---
title : "[SpringBoot][Chapter11.3] 액추에이터 - 액추에이터 기능 살펴보기"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, Actuator]
toc: true
toc_sticky : true
published : true
date : 2022-08-10
last_modified_at : 2022-08-10
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}





## 액추에이터 기능 살펴보기

액추에이터를 활성화하고 노출 지점도 설정하고 나면 애플리케이션에서 해당 기능을 사용할 수 있다. 모든 기능을 살펴보기 위해서는 다른 의존성을 추가하거나 몇 가지 설정을 추가해야 하기 때문에 이번 절에서는 기능 추가 없이 액추에이터 설정만으로 볼 수 있는 기능 위주로 살펴보겠다.



### 애플리케이션 기본 정보(/info)

액추에이터의 `/info` 엔드포인트를 활용하면 가동 중인 애플리케이션의 정보를 볼 수 있다. 제공하는 정보의 범위는 애플리케이션에서 몇 가지 방법을 거쳐 제공할 수 있으나 `application.properties`파일에 '`info.`' 로 시작하는 속성 값들을 정의하는 것이 가장 쉬운 방법이다. 간단한 예로 아래와 같이 애플리케이션의 정보를 작성할 수 있다.

<br>

```properties
## 액추에이터 info 정보 설정
info.organization.name=wikibooks
info.contact.email=thinkground.flature@email.com
info.contact.phoneNumber=010-1234-5678
```

<br>

그러고 나서 애플리케이션을 가동한 후 브라우저에서 아래 URL에 접근하면 아래와 같은 결괏값이 확인된다.

<br>

------

<a href="http://localhost:8080/actuator/info">`http://localhost:8080/actuator/info`</a>

------

```json
{
   "organization":{
      "name":"wikibooks"
   },
   "contact":{
      "email":"thinkground.flature@email.com",
      "phoneNumber":"010-1234-5678"
   }
}
```

<br>

참고로 출력 결과가 한 줄로 나와 보기 힘들 경우에는 JSON Formatter 사이트(<a href="https://jsonformatter.curiousconcept.com/" target="_blank">https://jsonformatter.curiousconcept.com/</a>)를 통해 출력 결과를 좀 더 보기 쉽게 확인할 수 있다.



#### 애플리케이션 상태(/health)

`/health` 엔드포인트를 활용하면 애플리케이션의 상태를 확인할 수 있다. 별도의 설정 없이 다음 URL에 접근하면 아래와 같은 결과를 확인할 수 있다.

<br>

------

<a href="http://localhost:8080/actuator/health">`http://localhost:8080/actuator/health`</a>

------

```json
{"status":"UP"}
```

<br>

예제에서는 UP만 표시되지만 `status` 속성에서 확인할 수 있는 상태 지표는 다음과 같다.

- `UP`
- `DOWN`
- `UNKNOWN`
- `OUT_OF_SERVICE`

<br>

이 결과는 주로 네트워크 계층 중 L4(Loadbalancing) 레벨에서 애플리케이션의 상태를 확인하기 위해 사용된다. 상세 상태를 확인하고 싶다면 아래와 같이 설정하면 된다.

<br>

```properties
## 액추에이터 health 상세 내역 활성화
management.endpoint.health.show-details=always
```

<br>

위 예제의 2번 줄에 있는 `show-details` 속성에서 설정할 수 있는 값은 다음과 같다.

- `never(기본값)` : 세부 사항은 표시하지 않는다.
- `when-authorized` : 승인된 사용자에게만 세부 상태를 표시한다. 확인 권한은 `application.properties` 에 추가한 `management.endpoint.health.roles` 속성으로 부여할 수 있다.
- `always` : 모든 사용자에게 세부 상태를 표시한다.

<br>

`always`로 설정을 변경하고 애플리케이션을 재가동한 후 애플리케이션 상태를 확인하면 아래와 같은 결괏값을 얻을 수 있다.

<br>

```json
{
   "status":"UP",
   "components":{
      "diskSpace":{
         "status":"UP",
         "details":{
            "total":511013888000,
            "free":214821752832,
            "threshold":10485760,
            "exists":true
         }
      },
      "ping":{
         "status":"UP"
      }
   }
}
```

<br>

위 예제에서는 별다른 기능이 없는 애플리케이션에서 실습을 진행하고 있어 간단한 내용만 출력된다. 만약 애플리케이션에 데이터베이스가 연동 돼 있으면 인프라 관련 상태까지 확인할 수 있다.

그리고 예제의 내용을 살펴보면 중간중간 계속해서 `status` 속성 값이 보이는데, 모든 `status`의 값이 `UP`이어야 애플리케이션의 상태가 UP으로 표시된다. 만약 `DOWN` 상태인 항목이 있다면 애플리케이션의 상태도 `DOWN`으로 표기되며 HTTP 상태 코드도 변경된다.



### 빈 정보 확인(/beans)

액추애이터의 `/beans` 엔드포인트를 사용하면 스프링 컨테이너에 등록된 스프링 빈의 전체 목록을 표시할 수 있다. 이 엔드포인트는 JSON 형식으로 빈의 정보를 반환한다. 다만 스프링은 워낙 많은 빈이 자동으로 등록되어 운영되기 때문에 실제로 내용을 출력해서 육안으로 내용을 파악하기는 어렵다. 간단하게 출력된 내용을 보면 아래와 같다.

<br>

------

<a href="http://localhost:8080/actuator/beans">`http://localhost:8080/actuator/beans`</a>

------



```json
{
  "contexts":{
    "application":{
      "beans":{
          "endpointCachingOperationInvokerAdvisor":{
             "aliases":[],
             "scope":"singleton",
             "type":"org.springframework.boot.actuate.endpoint.invoker.cache.CachingOperationInvokerAdvisor", ...
      },
      "parentId":null
    }
  }
}
```

<br>



### 스프링 부트의 자동설정 내역 확인(/conditions)

스프링 부트의 자동설정(AutoConfiguration) 조건 내역을 확인하려면 '`/conditions`' 엔드포인트를 사용한다. 다음 URL로 접근하면 아래와 같은 내용을 확인할 수 있다.

------

<a href="http://localhost:8080/actuator/conditions">`http://localhost:8080/actuator/conditions`</a>

------

```json
{
  "contexts":{
    "application":{
      "positiveMatches":{
        "AuditEventsEndpointAutoConfiguration":[
          {
            "condition":"OnAvailableEndpointCondition",
            "message":"@ConditionalOnAvailableEndpoint no property management.endpoint.auditevents.enabled found so using endpoint default; @ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.jmx.exposure' property"
          }
        ],
        "BeansEndpointAutoConfiguration":[
          {
            "condition":"OnAvailableEndpointCondition",
            "message":"@ConditionalOnAvailableEndpoint no property management.endpoint.beans.enabled found so using endpoint default; @ConditionalOnAvailableEndpoint marked as exposed by a 'management.endpoints.jmx.exposure' property"
          }
        ],
        "BeansEndpointAutoConfiguration#beansEndpoint":[
          {
            "condition":"OnBeanCondition",
            "message":"@ConditionalOnMissingBean (types: org.springframework.boot.actuate.beans.BeansEndpoint; SearchStrategy: all) did not find any beans"
          }
        ],...
      }
    }
  }
}
```

<br>

출력 내용은 크게 `positiveMatches` 와 `negativeMatches` 속성으로 구분되는데, 자동설정의 `@Conditional`에 따라 평가된 내용을 표시한다.



### 스프링 환경변수 정보(/env)

`/env` 엔드포인트는 스프링의 환경변수 정보를 확인하는 데 사용된다. 기본적으로 `application.properties` 파일의 변수들이 표시되며, OS, JVM의 환경변수도 함께 표시된다. 다음 URL로 접근하면 아래와 같은 결과를 확인할 수 있다. 참고로 `/env` 엔드포인트의 출력값은 내용이 매우 복잡하기 때문에 일부 내용만 발췌했다.

------

<a href="http://localhost:8080/actuator/env">`http://localhost:8080/actuator/env`</a>

------

```json
{
  "activeProfiles":[
    
  ],
  "propertySources":[
    {
      "name":"server.ports",
      "properties":{
        "local.server.port":{
          "value":8080
        }
      }
    },
    {
      "name":"servletContextInitParams",
      "properties":{
        
      }
    },
    {
      "name":"systemProperties",
      "properties":{
        "sun.desktop":{
          "value":"windows"
        },
        "awt.toolkit":{
          "value":"sun.awt.windows.WToolkit"
        },
        "java.specification.version":{
          "value":"11"
        },
        "sun.cpu.isalist":{
          "value":"amd64"
        },
        "sun.jnu.encoding":{
          "value":"MS949"
        },
        ...
      }
    }
  }
}
```

<br>

만약 일부 내용에 포함된 민감한 정보를 가리기 위해서는 `management.endpoint.env.keys-to-sanitize` 속성을 사용하면 된다. 해당 속성에 넣을 수 있는 값은 단순 문자열이나 정규식을 활용한다.



### 로깅 레벨 확인(/loggers)

애플리케이션의 로깅 레벨 수준이 어떻게 설정돼 있는지 확인하려면 `/loggers` 엔드포인트를 사용할 수 있다. 다음 URL에 접근하면 아래와 같은 결과가 출력된다. 참고로 출력 결과가 매우 길기 때문에 일부 내용만 발췌했다.

<br>

------

<a href="http://localhost:8080/actuator/loggers">`http://localhost:8080/actuator/loggers`</a>

------

```json
{
  "levels":[
    "OFF",
    "ERROR",
    "WARN",
    "INFO",
    "DEBUG",
    "TRACE"
  ],
  "loggers":{
    "ROOT":{
      "configuredLevel":"INFO",
      "effectiveLevel":"INFO"
    },
    "_org":{
      "configuredLevel":null,
      "effectiveLevel":"INFO"
    },
    "_org.springframework":{
      "configuredLevel":null,
      "effectiveLevel":"INFO"
    },
    "_org.springframework.web":{
      "configuredLevel":null,
      "effectiveLevel":"INFO"
    },
    "_org.springframework.web.servlet":{
      "configuredLevel":null,
      "effectiveLevel":"INFO"
    },
    "groups":{
    "web":{
      "configuredLevel":null,
      "members":[
        "org.springframework.core.codec",
        "org.springframework.http",
        "org.springframework.web",
        "org.springframework.boot.actuate.endpoint.web",
        "org.springframework.boot.web.servlet.ServletContextInitializerBeans"
      ]
    },
    "sql":{
      "configuredLevel":null,
      "members":[
        "org.springframework.jdbc.core",
        "org.hibernate.SQL",
        "org.jooq.tools.LoggerListener"
      ]
    }
  }
}
```

<br>

위 예제는 GET 메서드로 호출한 결과이며, POST 형식으로 호출하면 로깅 레벨을 변경하는 것도 가능하다.

