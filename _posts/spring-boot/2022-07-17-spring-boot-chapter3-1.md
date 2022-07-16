---
title : "[SpringBoot][Chapter3.1] 개발 환경 구성 - 자바 JDK 설치"
categories:
- SpringBoot
tag :
- [SpringBoot, Spring, Java]
toc: true
toc_sticky : true
published : true
date : 2022-07-17 
last_modified_at : 2022-07-17
---





장정우님이 지음, [스프링부트 핵심가이드 :: 스프링 부트를 활용한 애플리케이션 개발 실무] 책을 읽고 정리한 필기입니다.📢
{: .notice--warning}



## 자바 JDK 설치

자바 JDK를 Azul에서 제공하는 Open JDK를 사용한다.  Azul 공식 사이트의 [다운로드 페이지] 를 방분한 후 페이지 하단에서 아래와 같은 화면이 나오면 현재 사용 중인 시스템에 해당하는 항목을 선택한다.

[다운로드 페이지]: https://www.azul.com/downloads/?package=jdk



![image](https://user-images.githubusercontent.com/13410737/179361336-117351a7-8a41-495d-a571-b2a036739393.png){: .align-center}

윈도우 10 64비트 환경에서의 항목 설정은 다음과 같다. 현재 사용 중인 컴퓨터 사양에 따라 운영체제와 아키텍처를 선택한다.

- Java Version : Java 11(LTS)
- Operating System : Windows
- Architecture : x86 64-bit
- Java Package : JDK



아래 그림과 같이 JDK 다운로드 항목이 표시되면 .msi 나 .zip 형식의 설치 파일을 내려 받는다.

![image](https://user-images.githubusercontent.com/13410737/179361378-25866414-e515-4e26-b51e-97c29f4701c7.png){: .align-center}

.msi 파일을 실행하면 아래와 같은 설치 화면이 나온다. 별도의 조작 없이 [Next]만 눌러 설치를 완료한다.

![image](https://user-images.githubusercontent.com/13410737/179361428-4341424d-2e2e-4458-9e70-fc426e75bf2f.png){: .align-center}

설치가 완료되면 윈도우에서 정상적으로 JDk를 사용하기 위해서는 환경 변수를 추가해야 한다. 그런데 .msi 파일로 자바를 설치하면 환경변수에 자바 경로가 자동으로 추가되기 대문에 직접 등록하는 작업을 하지 않아도 된다.

그러나 종종 환경변수가 정상적으로 추가되지 않는 경우도 발생하므로 함게 시스템 환경변수를 확인하겠다. 윈도우에서 [제어판] → [시스템 및 보안] → [시스템]으로 차례로 이동하면 아래 같은 화면이 나온다. 이 화면의 가장 오른쪽에 위치한 [관련 설정] 영역에서 [고급 시스템 설정]을 선택한다.

![image](https://user-images.githubusercontent.com/13410737/179361594-bf927dfe-9836-4e79-869e-7feda2153b78.png){: .align-center}

 아래와 같이 '시스템 속성' 창이 나타나면 [고급]탭으로 이동한 후 [환경 변수] 버튼을 클릭한다.

![image](https://user-images.githubusercontent.com/13410737/179361622-3590f026-3dd4-4800-9e84-d710866631d4.png){: .align-center}

아래와 같은 화면이 나오면 아래 위치한 [시스템 변수] 항목에서 변수 이름이 'Path'인 항목을 찾아 [편집] 버튼을 클릭한다.

![image](https://user-images.githubusercontent.com/13410737/179361679-fcd51e44-c7e1-45bc-a2fd-3cf70b4ebba0.png){: .align-center}

환경 변수 편집 창에서 아래와 같이 zulu-11에 대한 경로가 설정돼 있다면 정상적으로 JDK 설치가 완료된 것이다. 만약 설정돼 있지 않다면 오른쪽에 위치한 [새로 만들기] 버튼을 클릭하고 컴퓨터에서 JDK가 설치된 위치를 찾아 bin 경로(예: C:\Program Files\Zulu\zulu-11\bin)를 입력한다. 그러고 나서 [확인] 버튼을 눌러 변경된 사항을 적용한다.

![image](https://user-images.githubusercontent.com/13410737/179361766-10e09355-9bac-4e6e-8534-f0834205e5c9.png){: .align-center}



