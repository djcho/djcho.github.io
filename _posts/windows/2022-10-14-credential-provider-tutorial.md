---
title : "[Windows]CredentialProvider V2 튜토리얼"
categories:
- Winsys
tag :
- [Windows, CredentialProvider V2, C++]
toc: true
toc_sticky : true
published : true
date : 2022-10-14
last_modified_at : 2022-10-14

---









## Windows Credential Provider V2

Windows Credential Provider는 Windows 사용하는 자격 공급자로써 사용자에게 인증 요청해 인가된 사용자에게 자격을 제공하는 윈도우 컴포넌트이다. Windows Credential Provider 는 크게 Windows 데스크탑 진입 전 사용자 로그온 화면과 CredUI라는 사용자 인증 요청 다이얼로그 화면으로 구성된다. 이 컴포넌트는 Windows XP 시절부터 존재했었고 Windows 8 부터 V2로 사용자와 인증 화면(Credential) 을 분리하면서 대폭 개선되었다. 이 문서에서는 Microsoft에서 제공하는 Windows Credential Provider V2(이하 CP) 샘플로 커스텀 Credential 을 띄워 보는 튜토리얼을 설명한다.



CP는 Windows 의 데스크탑 진입 전의 모듈인만큼 문제가 발생 시 최악의 경우에는 데스크탑으로 진입을 못하는 상황이 발생할 수 있기 때문에 해당 바이너리를 테스트할 경우 가상 버신을 사용하는 것을 권고한다. 가상 머신을 사용할 수 없다면 안전모드로 부팅이 가능하게 사전 조취를 취해야 하며, 문제가 발생 했을 때 안전모드로 테스트 바이너리를 제거한다.{: .notice--warning}



### 샘플 프로젝트 다운로드 및 빌드

[![Readme Card](https://github-readme-stats.vercel.app/api/pin/?username=microsoft&repo=Windows-classic-samples)](https://github.com/microsoft/windows-classic-samples/tree/main/Samples/CredentialProvider)

위 링크에서 Microsoft 에서 제공하는 CP 샘플을 다운받고 VisualStudio 를 사용하여 `SampleV2CredentialProvider.sln`을 실행한다.

구성에서 Release/x64로 `SampleV2CredentialProvider` 프로젝트를 빌드한다. 프로젝트 빌드가 성공했다면 x64\Release\SampleV2CredentialProvider.dll 이 생성되는 것을 확인한다.

![image](https://user-images.githubusercontent.com/13410737/195750242-2a96c6fd-7a07-48be-9f62-c721a4b542cc.png){: .align-center}

프로젝트 기본 설정이 `다중 스레드DLL(/MD)`로 설정되어 있기 때문에 빌드한 Visual Studio 의 버전에 따른 재배포 패키지가 테스트 환경에 설치되어야 한다.{: .notice--warning}



### 테스트 환경에 적용

테스트 환경에 적용하기 위해 위에서 빌드한 SampleV2CredcentialProvider.dll 와 프로젝트 내부에 포함되어 있는 register.reg 파일을 테스트 환경으로 복사한다.

#### 커스텀 CP 바이너리 복사

SampleV2CredentialProvider.dll 을 아래 경로에 위치 시킨다.

경로 : C:\Windows\System32



#### COM 등록

register.reg 파일을 실행하여 빌드한 테스트 바이너리를 시스템에 등록한다. register.reg 파일은 아래와 같다. 

```
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers\{5fd3d285-0dd9-4362-8855-e0abaacd4af6}]
@="SampleV2CredentialProvider"

[HKEY_CLASSES_ROOT\CLSID\{5fd3d285-0dd9-4362-8855-e0abaacd4af6}]
@="SampleV2CredentialProvider"

[HKEY_CLASSES_ROOT\CLSID\{5fd3d285-0dd9-4362-8855-e0abaacd4af6}\InprocServer32]
@="SampleV2CredentialProvider.dll"
"ThreadingModel"="Apartment"
```



> **Tip. 💡**
>
> 만약 빌드한 바이너리의 위치를 변경 하고 싶다면 register.reg 파일을 열어 아래 dll 경로를 변경할 절대 경로로 설정해 주면 된다.
>
> ![image](https://user-images.githubusercontent.com/13410737/195749688-8b602fbe-0fa4-47af-bbad-195cec19541e.png){: .align-center}



여기서 등록되는 GUID는 변경이 가능하고 이 GUID를 기준으로 커스텀 CP의 출력 순서가 결정된다. 이 GUID는 guid.h 에 정의되어 있는 GUID와 정확히 일치해야 한다. guid.h는 아래와 같다.

```c++
//
// THIS CODE AND INFORMATION IS PROVIDED "AS IS" WITHOUT WARRANTY OF
// ANY KIND, EITHER EXPRESSED OR IMPLIED, INCLUDING BUT NOT LIMITED TO
// THE IMPLIED WARRANTIES OF MERCHANTABILITY AND/OR FITNESS FOR A
// PARTICULAR PURPOSE.
//
// Copyright (c) Microsoft Corporation. All rights reserved.
//

// 5fd3d285-0dd9-4362-8855-e0abaacd4af6
DEFINE_GUID(CLSID_CSample, 0x5fd3d285, 0x0dd9, 0x4362, 0x88, 0x55, 0xe0, 0xab, 0xaa, 0xcd, 0x4a, 0xf6);
```



등록이 정상적으로 되었다면 Windows 키 + L 로 화면잠금하여 커스텀 CP가 출력되는지 확인한다. 

정상적으로 로드가 되었다면 하단에 [로그인 옵션]이라는 메뉴가 생성 되고 우리가 추가한 커스텀 CP가 출력됨을 확인할 수 있다.



![image](https://user-images.githubusercontent.com/13410737/195765080-84d30aca-6dd5-4bd1-a8ef-5ee4b4144397.png)