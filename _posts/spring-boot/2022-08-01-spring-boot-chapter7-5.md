---
title : "[SpringBoot][Chapter7.5] 테스트 코드 작성하기 - JaCoCo 테스트 커버리지"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, JaCoCo]
toc: true
toc_sticky : true
published : true
date : 2022-08-01
last_modified_at : 2022-08-01
---





## JaCoCo를 활용한 테스트 커버리지 확인

코드 커버리지(code coverage)는 소프트웨어의 테스트 수준이 충분한지를 표현하는 지표 중 하나이다. 테스트를 진행했을 때 대상 코드가 실행됐는지 표현하는 방법으로도 사용된다.

커버리지를 확인하기 위해 다양한 커버리지 도구 중 가장 보편적으로 사용되는 도구는 <a href="https://www.jacoco.org/jacoco/" target="_blank">JaCoCo</a>이다. JaCoCo는 Java Code Coverage의 약자로, JUnit 테스트를 통해 애플리케이션의 코드가 얼마나 테스트 됐는지 Line과 Branch를 기준으로 한 커버리지로 리포트한다. JaCoCo는 런타임으로 테스트 케이스를 실행하고 커버리지를 체크하는 방식으로 동작하며, 리포트는 HTML, XML, CSV 같은 다양한 형식으로 확인할 수 있다.



### JaCoCo 플러그인 설정

JaCoCo의 플러그인 설정은 `pom.xml` 파일에서 한다. 먼저 다음과 같이 의존성을 추가한다.

<br>

```xml
<dependency>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.7</version>
</dependency>
```

<br>

기본적인 설정 형식은 다음과 같다.

<br>

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.7</version>
    <configuration>
        <excludes>
            <exclude>**/ProductServiceImpl.class</exclude>
        </excludes>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>jacoco-report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>jacoco-check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <elment>BUNDLE</elment>
                        <limits>
                            <limit>
                                <counter>INSTRUCTION</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                        </limits>
                        <element>METHOD</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>TOTALCOUNT</value>
                                <maximum>50</maximum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

<br>

위 코드의 플러그인 설정은 `pom.xml` 파일 내 `<build>` 태그 안에 추가한다. 먼저 살펴볼 내용은 `<configuration>` 태그이다.

<br>

```xml
<configuration>
    <excludes>
        <exclude>**/ProductServiceImpl.class</exclude>
    </excludes>
</configuration>
```

<br>

위 설정 내용은 일부 클래스를 커버리지 측정 대상에서 제외하는 것이다. 위 코드에서는 경로와 무관하게 `ProductServiceImpl.class`를 커버리지 측정 대상에서 제외하도록 설정돼 있다.

다음으로 살펴볼 내용은 다음과 같이 `<executions>` 태그에 설정하는 플러그인의 실행 내용이다.

<br>

```xml
<executions>
    <execution>
        <goals>
            <goal>prepare-agent</goal>
        </goals>
    </execution>
    <execution>
        <id>jacoco-report</id>
        <phase>test</phase>
        <goals>
            <goal>report</goal>
        </goals>
    </execution>
    <execution>
        <id>jacoco-check</id>
        <goals>
            <goal>check</goal>
        </goals>
        <configuration>
            <rules>
                <rule>
                    <elment>BUNDLE</elment>
                    <limits>
                        <limit>
                            <counter>INSTRUCTION</counter>
                            <value>COVEREDRATIO</value>
                            <minimum>0.80</minimum>
                        </limit>
                    </limits>
                    <element>METHOD</element>
                    <limits>
                        <limit>
                            <counter>LINE</counter>
                            <value>TOTALCOUNT</value>
                            <maximum>50</maximum>
                        </limit>
                    </limits>
                </rule>
            </rules>
        </configuration>
    </execution>
</executions>
```

<br>

`<excurion>`은 기본적으로 `<goal>`을 포함하며, 설정한 값에 따라 추가 설정이 필요한 내용을 `<configuration>`과 `<rule>`을 통해 작성한다. 먼저 `<execution>`에서 설정할 수 있는 `<goal>`의 속성값은 다음과 같다.



- `help` : `jacoco-maven-plugin`에 대한 도움말을 보여준다.
- `prepare-agent` : 테스트 중인 애플리케이션에 VM 인수를 전달하는 JaCoCo 런타임 에이전트 속성을 준비한다. 에이전트는 `maven-surefire-plugin`을 통해 테스트한 결과를 가져오는 역할을 수행한다.
- `prepare-agent-integration` : `prepare-agent`와 유사하지만 통합 테스트에 적합한 기본값을 제공한다.
- `merge` : 실행 데이터 파일 세트(.exec)를 단일 파일로 병합한다.
- `report` : 단일 프로젝트 테스트를 마치면 생성되는 코드 검사 보고서를 다양한 형식(HTML, XML, CSV) 중에서 선택할 수 있게 한다.
- `report-integration` : `report`와 유사하나 통합 테스트에 적합한 기본값을 제공한다.
- `report-aggregate` : Reactor 내의 여러 프로젝트에서 구조화된 보고서(HTML, XML, CSV)를 생성한다. 보고서는 해당 프로젝트가 의존하는 모듈에서 생성된다.
- `check` : 코드 커버리지의 메트릭 충족 여부를 검사한다. 메트릭은 테스트 커버리지를 측정하는 데 필요한 지표를 의미한다. 메트릭은 `check`가 설정된 `<exceution>` 태그 내에서 `<rule>`을 통해 설정한다.
- `dump` : TCP 서버 모드에서 실행 중인 JaCoCo 에이전트에서 TCP/IP를 통한 덤프를 생성한다.
- `instrument` : 오프라인 특정을 수행하는 명령이다. 테스트를 실행한 후 `restore-instrumented-classes` Coal로 원본 클래스 파일들을 저장해야 한다.
- `restore-instrumented-class` : 오프라인 측정 전 원본 파일을 저장하는 기능을 수행한다.



다음으로 JaCoCo에서 설정할 수 있는 Rule을 살펴보겠다. `<cofiguration>` 태그 안에 설정하며, 다양한 속성을 활용할 수 있다.

먼저 `Element`는 코드 커버리지를 체크하는 데 필요한 범위 기준을 설정한다. 사용 가능한 속성은 총 6가지이다.



- BUNDLE(기본값) : 패키지 번들(프로젝트 내 모든 파일)
- PACKAGE : 패키지
- CLASS : 클래스
- GROUP : 논리적 번들 그룹
- SOURCEFILE : 소스 파일
- METHOD : 메서드



값을 지정하지 않은 상태의 기본값은 `BUNDLE`이다. `BUNDLE`은 `Element`를 기준으로 `<limits>` 태그 내 `<counter>`와 `<value>`를 활용해 커버리지 측정 단위와 방식을 설정한다.

다음으로 `Counter`는 커버리지를 측정하는 데 사용하는 지표이다. `Counter`에서 사용할 수 있는 커버리지 측정 단위는 총 6가지이다.



- LINE : 빈 줄을 제외한 실제 코드의 라인 수
- BRANCH : 조건문 등의 분기 수
- CLASS : 클래스 수
- METHOD : 메서드 수
- INSTRUCTION(기본값) : 자바의 바이트코드 명령 수
- COMPLEXITY : 복잡도, 복잡도는  <a href="https://en.wikipedia.org/wiki/Cyclomatic_complexity" target="_blank">맥케이브 순환 복잡도</a> 정의를 따른다.
- TOTALCOUNT : 전체 개수
- MISSDCOUNT : 커버되지 않은 개수
- COVEREDCOUNT : 커버된 개수
- MISSEDRATIO : 커버되지 않은 비율
- COVEREDRATIO(기본값) : 커버된 비율



`Value`의 속성을 지정하지 않는 경우의 기본값은 `COVEREDRATIO`이다. 위 내용을 기반으로 아래 코드를 보자.

<br>

```xml
<configuration>
    <rules>
        <rule>
            <elment>BUNDLE</elment>
            <limits>
                <limit>
                    <counter>INSTRUCTION</counter>
                    <value>COVEREDRATIO</value>
                    <minimum>0.80</minimum>
                </limit>
            </limits>
            <element>METHOD</element>
            <limits>
                <limit>
                    <counter>LINE</counter>
                    <value>TOTALCOUNT</value>
                    <maximum>50</maximum>
                </limit>
            </limits>
        </rule>
    </rules>
</configuration>
```

<br>

예제에서는 `limit`이 각 `Element` 단위로 설정돼 있다. 4~11번 줄을 보면 패키지 번들 단위 바이트코드 명령 수를 기준으로 커버리지가 최소한 80% 달성하는 것을 `limit`으로 설정했다. 그리고 12~19번 줄에서는 메서드 단위로 전체 라인 수를 최대 50줄로 설정했다. 만약 설정한 기준을 벗어나는 경우에는 에러가 발생한다.



### JaCoCo 테스트 커버리지 확인

JaCoCo 플러그인으로 테스트 커버리지를 측정하려면 메이븐의 테스트 단계가 선행돼야 한다. 메이븐의 생명주기는 아래와 같이 Maven 탭에서 확인할 수 있으며, JaCoCo는 test 단계 뒤에 있는 package 단계에서 실행할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182033699-f04fe4a2-b995-4b1a-9b51-0555e2f4a597.png){: .align-center}

<br>

위 그림에서 package를 더블클릭해서 빌드를 진행하면 아래와 같이 `target` 폴더 내 `site → jacoco`폴더가 생성된다. 만약 정상적으로 동작하지 않는다면 프로젝트 경로에 한글이 없는지 확인해야 한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182033787-213f2c55-21db-4ef4-a4f3-37ae2b5ee81f.png){: .align-center}

<br>

기본적으로 JaCoCo 리포트 파일은 HTML, CSV, XMl 형식으로 제공된다. 일반적으로 곧바로 보고서를 보기 위해서는 HTML 파일을 주로 이용한다. 웹 브라우저를 통해 HTML 파일을 열면 아래와 같이 리포트 결과를 확인할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182033935-85e5f4ad-1440-415d-be7c-36cfbb24f9e0.png){: .align-center}

<br>

위 페이지를 구성하는 각 칼럼은 다음과 같은 의미를 가지고 있다.

- Element : 우측 테스트 커버리지를 측정한 단위를 표현한다. 링크를 따라 들어가면 세부 사항을 볼 수 있다.
- Missed Instructions - Cov.(Coverage) : 테스트를 수행한 후 바이트코드의 커버리지를 퍼센티지와 바(Bar) 형식으로 제공한다.
- Missed Branches - Cov.(Coverage) : 분기에 대한 테스트 커버리지를 퍼센티지와 바 형식으로 제공한다.
- Missed - Cxty(Complexity) : 복잡도에 대한 커버리지 대상 개수와 커버되지 않은 수를 제공한다.
- Missed - Lines : 테스트 대상 라인 수와 커버되지 않은 라인 수를 제공한다.
- Missed - Methods : 테스트 대상 메서드 수와 커버되지 않은 메서드 수를 제공한다.
- Messed - Classes : 테스트 대상 클래스 수와 커버되지 않은 메서드 수를 제공한다.

<br>

위 리포트 첫 페이지는 Element가 패키지 단위로 표현돼 있다. 총 5개 단위의 패키지가 보이는데, 앞선 설정에서 테스트 대상에서 서비스 클래스를 제외했기 때문에 리포트 페이지에서 제외된것을 볼 수 있다. 해당 설정을 삭제하고 다시 빌드를 수행하면 서비스 클래스도 테스트 범위에 해당되는 것을 볼 수 있다.

JaCoCo의 리포트를 통해 볼 수 있는 세부 내용을 살펴보기 위해 `ProductController`를 살펴보겠다. HTML 파일에서 `com.springboot.test.controller` 패키지를 누르면 `ProductController`클래스가 Element에 표시되며 테스트 커버리지 통계를 보여준다. 다시 해당 부분을 클릭하면 컨트롤러에 작성돼 있는 Method가 Element의 기준이 되고, 여기서 메서드를 클릭하면 아래와 같이 코드 레벨에서의 테스트 커버리지를 확인할 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/182034212-5f9827e5-0933-4821-800e-6c9cdca74b24.png){: .align-center}

<br>

여기서 각 코드 라인은 초록색과 빨간색으로 표시돼 있다. 테스트 대상이 단순한 예제 코드라 색깔이 분명하게 출력됐으나 `if`문 같은 조건문이 있다면 코드가 분기되는 지점에 노란색이 칠해지는 등 좀 더 복잡한 결과물이 출력된다.

초록색은 테스트에서 실행됐다는 의미이고, 빨간색은 테스트 코드에서 실행되지 않은 라인을 의미한다. 조건문의 경우 `true`와 `false`에 대한 모든 케이스가 테스트됐다면 초록색으로 표시되며, 둘 중 하나만 테스트됐다면 노란색으로 표시된다.

<br>

> **💡 Tip.**
>
> JaCoCo와 관련된 내용을 더 알고 싶다면 공식 문서를 참고한다.
>
> - <a href="https://www.jacoco.org/jacoco/trunk/doc/" target="_blank">https://www.jacoco.org/jacoco/trunk/doc/</a>
