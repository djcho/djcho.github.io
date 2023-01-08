---
title : "[SpringBoot][Chapter6.1] DB연동 - MariaDB 설치"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java, Intelij, MariaDB]
toc: true
toc_sticky : true
published : true
date : 2022-07-20
last_modified_at : 2022-07-20
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



# MariaDB 설치

마리아 DB를 내려받기 위해 다운로드 페이지(<a href="https://mariadb.org/download" target="_blank">https://mariadb.org/download/</a>)에 접속하면 아래와 같 은 화면을 볼 수 있다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179999209-118513a4-8064-4f77-b47d-86341043897c.png){: .align-center}

<br>

개발 환경에 맞춰 다음과 같이 설정하고 설치 파일을 내려받는다.

- MariaDB Server Version: MariaDB Server 10.6.8
- Operating System : Windows
- Architecture : x86_x64
- Package Type : MSI Package

만약 마리아DB 서버 버전이 위에 명시한 버전(10.6.5)으로 표시되지 않는다면 [MariaDB Server Version]항목 하단의 [Display older release]에 체크해서 버전을 동일하게 맞추는 것이 좋다.

설치 프로그램 다운로드가 완료되면 설치를 진행한다. 아래와 같은 창이 나오면 별도의 조작 없이 [Next]를 눌러 진행한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179999660-0cc99a41-af93-4a5a-9622-4c1810120024.png){: .align-center}

<br>

계속 단계를 넘어가다 보면 아래와 같은 화면이 나온다. 처음 데이터베이스를 설치하면 `root` 계정이 부여되는데, 그 계정의 패스워드를 지정하는 단계이다. 패스워드는 원하는 대로 설정하되 꼭 잊지 않게 보관해야 한다. 참고로 실무에서는 보안상 `root` 패스워드는 사용하지 않지만 여기서는 실습용으로 데이터베이스를 사용할 것이디 때문에 그대로 `root`계정을 사용한다.

다음으로 다장 대중적으로 사용되는 문자 인코딩 방식인 UTF-8을 기본값으로 설장하기 위해 [Use UTF8 as defult server's character set]에 체크한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180593780-84f24b68-4048-4f61-85a7-d22c1b55233d.png){: .align-center}

<br>

다음 단계에서는 아래와 같이 서버 이름과 포트 번호를 설정한다. 이 단계에서는 따로 설정을 수정할 필요 없이 그림과 동일하지 확인한 후 넘어가면 된다. 마리아DB는 기본 포트 번호로 3306을 사용하며, 만약 별도로 데이터베이스를 설치한 적이 있다면 포트가 겹치기 때문에 자동으로 3307번으로 매핑된다.

<br>

![image](https://user-images.githubusercontent.com/13410737/179999867-6297c01c-d0ad-4bbd-acd3-2d86f834ee7e.png){: .align-center}

<br>

그 밖에 단계는 모드 [Next]를 눌러 기본 설정을 그대로 유지한 채로 설치를 마친다. 마리아DB를 이 방식으로 설치하면 서드파티 도구로 <a href="https://www.heidisql.com">HeidiSQL</a>이 함께 설치된다. HeidiSQL은 데이터베이스에 접속해서 관리하는 GUI 도구로 앞으로 계속 사용할 예정이다. 이 프로그램을 실행하면 아래와 같은 창이 나타난다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180001338-ea1d89d4-2b81-43cf-a983-11854180da1a.png){: .align-center}

<br>

앞으로 사용할 데이터베이스의 접속 정보를 등록하기 위해 좌측 하단의 [신규] 버튼을 누른다. 아래와 같이 세션 목록에 새 항목이 등록되면서 우측에 접속 정보를 설정할 수 있는 항목들이 등장한다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180001643-2f9c656c-54b5-4884-be43-39a4c3e9d46d.png){: .align-center}

<br>



세션 목록에 나온 항목에 이름을 지정하고 우측의 입력 항목에 다음과 같이 입력하고 좌측 하단의 [저장]버튼을 클릭한다.

- 세션 이름 : springboot(다른 이름으로 지정해도 상관 없음)
- 호스트명 / IP : 127.0.0.1 또는 localhost
- 사용자명 : root
- 암호 : 마리아DB 설치 단계에서 지정한 패스워드
- 포트 : 3306(다를 경우 설치 단계에서 확인한 포드를 입력)

다음으로 [열기] 버튼을 클릭하면 아래와 같이 데이터베이스에 정상적으로 접속된다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180002085-d19d725d-03f5-475b-866c-eec817e91bad.png){: .align-center}

<br>

데이터베이스에 접속하면 기본적으로 데이터베이스 4개가 존재한다. 이 4개의 데이터베이스에는 마리아DB의 환경 설정 등과 같은 정보를 담고 있어 데이터베이스를 튜닝할 때 외에는 건드리지 않는다. 이 화면에서 스프링 부트 애플리케이션에서 사용할 데이터베이스를 하나 생성한다.

아래와 같이 데이터베이스를 생성하는 쿼리를 입력하고 도구모음 창에 위치한 파란색 세모 모양의 실행버튼을 누르거나 F9 키를 누르면 쿼리가 실행되면서 데이터베이스가 생성된다.

<br>

![image](https://user-images.githubusercontent.com/13410737/180002983-31ad9e85-dec9-4a23-a6e5-5d0b35813194.png){: .align-center}

<br>
