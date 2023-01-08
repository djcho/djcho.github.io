---
title : "[SpringBoot][Chapter11.2] 액추에이터 - 엔드포인트"
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





# 엔드포인트

액추에이터의 엔드포인트는 애플리케이션의 모니터링을 사용하는 경로이다. 스프링 부트에는 여러 내장 엔드포인트가 포함돼 있으며, 커스텀 엔드포인트를 추가할 수도 있다. 액푸에이터를 추가하면 기본적으로 엔드포인트 URL로 `/actuator`가 추가되며 이 뒤에 경로를 추가해 상세 내역에 접근한다. 만약 `/actuator` 경로가 아닌 다른 경로를 사용하고 싶다면 아래와 같이 `application.properties`파일에 작성한다.

<br>

```yaml
management.endpoints.web.base-path=/custom-path
```

<br>

자주 활용되는 액추에이터의 엔드포인트는 다음과 같다.

| ID                 | 설명                                                         |
| ------------------ | ------------------------------------------------------------ |
| `auditevents`      | 호출된 Audit 이벤트 정보를 표시한다. `AuditEventRepository` 빈이 필요하다. |
| `beans`            | 애플리케이션에 있는 모든 스프링 빈 리스트를 표시한다.        |
| `caches`           | 사용 가능한 캐시를 표시한다.                                 |
| `confitions`       | 자동 구성 조건 내역을 생성한다.                              |
| `configprops`      | `@ConfigurationProperties`의 속성 리스트를 표시한다.         |
| `env`              | 애플리케이션에서 사용할 수 있는 환경 속성을 표시한다.        |
| `health`           | 애플리케이션의 상태 정보를 표시한다.                         |
| `httptrace`        | 가장 최근에 이뤄진 100건의 요청 기록을 표시한다. `HttpTraceRepository` 빈이 필요하다. |
| `info`             | 애플리케이션의 정보를 표시한다.                              |
| `integrationgraph` | 스프링 통합 그래프를 표시한다. spring-integration-core 모듈에 대한 의존성을 추개해야 동작한다. |
| `loggers`          | 애플리케이션의 로거 구성을 표시하고 수정한다.                |
| `metrics`          | 애플리케이션의 메트릭 정보를 표시한다.                       |
| `mappings`         | 모든 `@RequestMapping`의 매핑 정보를 표시한다.               |
| `quartz`           | Quartz 스케줄러 작업에 대한 정보를 표시한다.                 |
| `scheduledtasks`   | 애플리케이션에서 예약된 작업을 표시한다.                     |
| `sessions`         | 스프링 세션 저장소에서 사용자의 세션을 검색하고 삭제할 수 있다. 스프링 세션을 사용하는 서블릿 기반 웹 애플리케이션이 필요하다. |
| `shutdown`         | 애플리케이션을 정상적으로 종료할 수 있다. 기본값은 비활성화 상태이다. |
| `startup`          | 애플리케이션이 시작될 때 수집된 시작 단계 데이터를 표시한다.<br />`BufferingApplicationStartup`으로 구성된 스프링 애플리케이션이 필요하다. |
| `threaddump`       | 스레드 덤프를 수행한다.                                      |

<br>

만약 Spring MVC, Spring WebFlus, jersey을 사용한다면 추가로 다음과 같은 엔드포인트를 사용할 수 있다.

<br>

| ID           | 설명                                                         |
| ------------ | ------------------------------------------------------------ |
| `heapdump`   | 힙 덤프 파일을 반환한다. 핫스팟(HotSpot) VM 상에서 hprof 포맷의 파일이 반환되며, OpenJ9JVM에서는 PHD 포맷 파일을 반환한다. |
| `jolokia`    | Jolokia가 클래스패스에 있을 때 HTTP를 통해 JMX 빈을 표시한다. jolokia-core 모듈에 대한 의존성 추가가 필요하며, WebFlux에서는 사용할 수 없다. |
| `logfile`    | `logging.file.name` 또는 `logging.file.path`속성이 설정돼 있는 경우 로그 파일의 내용을 반환한다. |
| `Prometheus` | Prometheus 서버에서 스크랩할 수 있는 형식으로 메트릭을 표시한다. Micrometer-registry-prometheus 모듈의 의존성 추가가 필요하다. |

<br>

엔드포인트 활성화 여부와 노출 여부를 설정할 수 있다. 활성화는 기능 자체를 활성화할 것인지를 결정하는 것으로, 비활성화된 엔트포인트는 애플리케이션 컨텍스트에서 완전히 제거된다. 엔트포인트를 활성화하려면 `application.propertis` 파일에 속성을 추가하면 된다. 간단한 예로 아래와 같이 작성할 수 있다.

<br>

```yaml
## 엔드포인트 활성화
management.endpoint.shutdown.enabled=true
management.endpoint.caches.enabled=true
```

<br>

위 예제의 설정은 엔드포인트의 `shutdown`기능은 활성화하고 `caches` 기능은 비활성화하겠다는 의미이다.

또한 액추에이터 설정을 통해 기능 활성화/비활성화가 아니라 엔드포인트의 노출 여부만 설정하는 것도 가능하다. 노출 여부는 JMX를 통한 노출과 HTTP를 통한 노출이 있어 아래와 같이 설정이 구분된다.

<br>

```yaml
## 엔드포인트 노출 설정
## HTTP 설정
management.endpoints.web.exposure.include=*
management.endpoints.web.exposure.exclude=threaddump, heapdump

## JMX 설정
management.endpoints.jmx.exposure.include=*
management.endpoints.jmx.exposure.exclude=threaddump, heapdump
```

<br>

위 설정을 해석하면 web과 jmx 환경에서 엔드포인트를 전체적으로 노출하며, 스레드 덤프(thread dump)와 힙 덤프(heap dump) 기능은 제외하겠다는 의미이다.

엔드포인트는 애플리케이션에 관한 민감한 정보를 포함하고 있으므로 노출 설정을 신중하게 고려해야한다. 특히나 공개적으로 노출되는 애플리케이션이라면 더욱 신중히 고려해서 사용해야 한다. 노출 설정에 대한 기본값은 아래 표와 같다.

<br>

| ID                 | JMX       | WEB  |
| ------------------ | --------- | ---- |
| `auditevents`      | O         | X    |
| `bean`             | O         | X    |
| `caches`           | O         | X    |
| `conditions`       | O         | X    |
| `configprops`      | O         | X    |
| `env`              | O         | X    |
| `flyway`           | O         | X    |
| `health`           | O         | O    |
| `heapdump`         | 해당 없음 | X    |
| `httptrace`        | O         | X    |
| `info`             | O         | X    |
| `integrationgraph` | O         | X    |
| `jolokia`          | 해당 없음 | X    |
| `logfile`          | 해당 없음 | X    |
| `loggers`          | O         | X    |
| `liquibase`        | O         | X    |
| `metrics`          | O         | X    |
| `mappings`         | O         | X    |
| `Prometheus`       | 해당 없음 |      |
| `quarts`           | O         | X    |
| `scheduledtasks`   | O         | X    |
| `sessions`         | O         | X    |
| `shutdown`         | O         | X    |
| `startup`          | O         | X    |
| `threaddump`       | O         | X    |
