---
title : "[Windows]CredentialProvider V2 with C#"
categories:
- Winsys
tag :
- [Windows, CredentialProvider V2, C#]
toc: true
toc_sticky : true
published : true
date : 2022-10-14
last_modified_at : 2022-10-14

---





## Windows Credential Provider V2 with C#

이 문서에서는 Microsoft 에서 제공하는 Windows Credential Provider(이하 CP) C++샘플 코드를 C# 샘플 코드로 포팅하는 방법에 대해 설명한다. Windows Credential Provider 의 기본 배경 지식은 설명하지 않고 기존 C++ 샘플 코드로 커스텀 CP를 띄워보는 방법은 아래 포스팅을 참고한다.

- https://djcho.github.io/winsys/credential-provider-tutorial/



Microsoft 에서 제공하는 CP의 샘플코드는  C++ 구현되어 있다.  물론 Windows 의 시스템 프로그래밍을 하기엔 C++가 편하지만 아무래 언어 특성 상 생산성과 유지보수에 있어서 그렇게 좋지 못하다. 그래서 베이스 코드를 C#으로 포팅하여 .NET 환경에서 CP 개발을 하면 어떨까? 라는 생각이 문득 들었고, 관련하여 구글링 해보니 이미 CredUI 시나리오에 한하여 C#으로 CP를 띄우는데까지 성공 시킨 시킨 Github 를 발견하였다. 이 저장소와 관련된 github 저장소들을 참고하여 로그온과 화면잠금 비밀번호 변경 시나리오에서 C#으로 빌드된 CP 바이너리가 로드될 수 있도록 구현하였다.



Windows 의 데스크탑 진입 전의 모듈인만큼 문제가 발생 시 최악의 경우에는 데스크탑으로 진입을 못하는 상황이 발생할 수 있기 때문에 해당 바이너리를 테스트할 경우 가상 버신을 사용하는 것을 권고한다. 가상 머신을 사용할 수 없다면 안전모드로 부팅이 가능하게 사전 조취를 취해야 하며, 문제가 발생 했을 때  안전모드로 테스트 바이너리를 제거한다. 📢
{: .notice--warning}



### COM Interop 파일 생성

#### idl 파일 수정

Windows SDK 에서 제공하는 기본 credentialprovider.idl을 그대로 사용할 경우에는 일부 인터페이스(예를 들어 ICredentialProviderCredential2)가 Export되지 않기 때문에  파일을 수정해야 한다. 

idl 파일 경로 : C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\um\credentialprovider.idl 

위 경로의 idl파일을 임의의 경로로 복사한 뒤 아래와 같이 수정한다. 수정이 필요한 항목은 아래와 같다.

- 라이브러리 선언부를 최상위로 옮겨 모든 항목이 Export 되도록 수정

```c++
//중간에 존재하는 위 코드 조각을 파일 가장 윗 부분으로 옮겨야 한다.
[
    uuid(d545db01-e522-4a63-af83-d8ddf954004f), // LIBID_CredentialProviders
]
library CredentialProviders
{
```

- `HBITMAP` 타입을 `HANDLE`로 변경



#### TypeLibrary 파일 생성

위에서 수정한 credentialprovider.idl 파일이 준비되었다면 **x64 Native Tools Command Prompt VS 2022** 를 실행하여 아래와 같이 명령어를 입력하고 tlb 파일을 생성한다.

Developer Command Prompt for VS 2022 가 아님을 주의 해야하며 관리자 권한이 필요하다. 📢
{: .notice--warning}

```> midl /target NT100 /x64 "C:\Program Files (x86)\Windows Kits\10\Include\10.0.19041.0\um\credentialprovider.idl"```

![image](https://user-images.githubusercontent.com/13410737/196323157-c40a7204-8d3a-4b58-98e2-db8fbb9c1b32.png){: .align-center}



CredentialProvider는 Windows 의 LogonUI.exe 에 의해 호출되므로 Windows 에 아키텍쳐를 따른다. 더이상 x86 Windows는 없기 때문에 x64로 명시해야 하며 Windows10 을 대상으로 하는 NT100 옵션을 추가한다. CredentialProvider V2 는 Vista 이후에 추가되었기 때문에 반드시 NT60 이상으로 명시해야 한다.



`.tlb`파일이 생성되었다면 VisualStudio 로 해당 파일을 열어 개체 브라우저에서 필요한 인터페이스들이 잘 Export 되었는지 확인한다.  아래 그림과 같이 인터페이스가 존재하는지 확인한다.

![image](https://user-images.githubusercontent.com/13410737/196322863-f561a3da-eb39-4aa8-b1b9-115f25f8d8ea.png){: .align-center}

#### TypeLibraryImporter2 빌드

기본  Microsoft 에서 제공하는 tlbimp.exe 를 사용하여 컴파일할 경우에는 HRESULT 반환 유형을 생략하고 .NET의 예외(Exception)을 사용하도록 변경되므로 Winlogon 또는 credUI 호스트앱이 예외를 발생시키면서 프로스가 종료되는 이슈가 있다. 때문에 이를 해결하려면 tlbimp2.exe 를 사용해 컴파일 해야 한다.

TypeLibraryImporter2 Github 저장소 : https://github.com/clrinterop/TypeLibraryImporter



위 저장소에서 프로젝트를 받은 뒤 TlbImp2.sln 을 열어 Tlbmp2 프로젝트를 빌드한다.

![image](https://user-images.githubusercontent.com/13410737/196321910-bae02c54-a8ef-46f7-820a-381975ef9c5f.png){: .align-center}



프로젝트 빌드 시 아래와 같이 파일이 생긴다. TlbImp2.exe 가 실행되려면 아래 3개의 dll 모두 필요하다.

![image](https://user-images.githubusercontent.com/13410737/196322208-99757655-f0df-4104-b450-4c64fe0927a2.png){: .align-center}

### Interop 파일 생성

tlbimp2.exe 가 준비되었다면 이제 위에서 만든 tlb 파일로 Interop.dll 을 생성할 차례이다. 위에서와 마찬가지로 **x64 Native Tools Command Prompt VS 2022** 를 관리자로 실행하여 아래와 같이 명령을 입력한다.

```> tlbImp2.exe credentialprovider.tlb /out:CredentialProvider.Interop.dll /unsafe /verbose /preservesig```

![image](https://user-images.githubusercontent.com/13410737/196323307-81079865-398f-42c7-ab05-1f3de6e4932c.png){: .align-center}



이제 VisualStudio 에서 CredentialProvider.Interop.dll 파일을 [참조 추가]하면 CP관련 인터페이스를 호출할 수 있게되고 C# CredentialProvider 를 개발할 준비가 끝났다.



### 설치 방법

regass.exe /codebase {테스트 CP바이너리 경로}



### 레퍼런스

- https://github.com/phaetto/windows-credentials-provider
- https://stackoverflow.com/questions/36425318/windows-credential-provider-in-c-sharp
- https://stackoverflow.com/questions/16092696/windows-credential-provider-with-c-sharp/23496878#23496878
- https://syfuhs.net/2017/10/15/creating-custom-windows-credential-providers-in-net/
- https://learn.microsoft.com/en-us/windows/win32/secauthn/credential-providers-in-windows
- https://learn.microsoft.com/en-us/samples/microsoft/windows-classic-samples/credential-provider/
