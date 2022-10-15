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



CP는 Windows 의 데스크탑 진입 전의 모듈인만큼 문제가 발생 시 최악의 경우에는 데스크탑으로 진입을 못하는 상황이 발생할 수 있기 때문에 해당 바이너리를 테스트할 경우 가상 버신을 사용하는 것을 권고한다. 가상 머신을 사용할 수 없다면 안전모드로 부팅이 가능하게 사전 조취를 취해야 하며, 문제가 발생 했을 때 안전모드로 테스트 바이너리를 제거한다.
{: .notice--warning}



### 샘플 프로젝트 다운로드 및 빌드

[![Readme Card](https://github-readme-stats.vercel.app/api/pin/?username=microsoft&repo=Windows-classic-samples)](https://github.com/microsoft/windows-classic-samples/tree/main/Samples/CredentialProvider)

위 링크에서 Microsoft 에서 제공하는 CP 샘플을 다운받고 VisualStudio 를 사용하여 `SampleV2CredentialProvider.sln`을 실행한다.

구성에서 Release/x64로 `SampleV2CredentialProvider` 프로젝트를 빌드한다. 프로젝트 빌드가 성공했다면 x64\Release\SampleV2CredentialProvider.dll 이 생성되는 것을 확인한다.

![image](https://user-images.githubusercontent.com/13410737/195750242-2a96c6fd-7a07-48be-9f62-c721a4b542cc.png){: .align-center}



### 테스트 환경에 적용

테스트 환경에 적용하기 위해 위에서 빌드한 SampleV2CredcentialProvider.dll 와 프로젝트 내부에 포함되어 있는 register.reg 파일을 테스트 환경으로 복사한다.

프로젝트 기본 설정이 `다중 스레드DLL(/MD)`로 설정되어 있기 때문에 빌드한 Visual Studio 의 버전에 따른 재배포 패키지가 테스트 환경에 설치되어야 한다.
{: .notice--warning}



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
> 만약 빌드한 바이너리의 위치를 변경 하고 싶다면 register.reg 파일을 열어 아래 dll 경로를 변경할 절대 경로로 설정해 주면 된다. 경로 중에 역슬래쉬(`\`)가 들어간다면 두번 " `\\`" 입력해야 정상적으로 경로가 등록된다.
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



### 샘플 프로젝트 분석

#### ICredentialProvider 인터페이스

이 인터페이스에서는 커스텀 CP에서 다루고자하는 UI 정보들을 LogonUI 에서 얻어갈 수 있도록 메서드들이 구성되어 있다. 

```c++
IFACEMETHODIMP SetUsageScenario(CREDENTIAL_PROVIDER_USAGE_SCENARIO cpus, DWORD dwFlags);
IFACEMETHODIMP SetSerialization(_In_ CREDENTIAL_PROVIDER_CREDENTIAL_SERIALIZATION const *pcpcs);

IFACEMETHODIMP GetFieldDescriptorCount(_Out_ DWORD *pdwCount);
IFACEMETHODIMP GetFieldDescriptorAt(DWORD dwIndex,  _Outptr_result_nullonfailure_ CREDENTIAL_PROVIDER_FIELD_DESCRIPTOR **ppcpfd);

IFACEMETHODIMP GetCredentialCount(_Out_ DWORD *pdwCount,
_Out_ DWORD *pdwDefault,
_Out_ BOOL *pbAutoLogonWithDefault);
IFACEMETHODIMP GetCredentialAt(DWORD dwIndex,
_Outptr_result_nullonfailure_ ICredentialProviderCredential **ppcpc);

IFACEMETHODIMP SetUserArray(_In_ ICredentialProviderUserArray *users);
```



- `SetUsageScenario()` : 현재 CP가 어떤 시나리오 상에서 로드되었는지가 입력된다. CP가 처음 로드될 때 호출되며 대표적인 시나리오는 아래와 같다.
  - CPUS_LOGON : 사용자의 데스크탑이 아직 열리지 않은 상태에서의 로그인할 때의 시나리오
  - CPUS_UNLOCK_WORKSTATION : 사용자의 데스크탑이 이미 열려 있는 상태에서 화면 잠금 했을 때 시나리오
  - CPUS_CHANGE_PASSWORD : 사용자의 데스크탑에서 Alt+Ctrl+Del 키 로 [암호 변경] 메뉴를 눌러 사용자 비밀번호 변경 화면이 출력될 때의 시나리오
  - CPUS_CREDUI :  Windows API 중`CredUIPromptForWindowsCredentials()` 를 호출하여 해당 CP가 로드되어 있는 PC의 자격증명을 요청했을 경우 시나리오
- `SetSerialization()` : CPUS_CREDUI 시나리오로 CP가 로드되었을 경우 또는 원격 데스크탑으로 인증을 요청한 경우 호출된다.
- `GetFieldDescriptorCount()` : CP에서 관리되는 필드들의 총 갯수를 반환해야 한다. CP가 로드될 때 호출된다.
- `GetFieldDescriptorAt()` : CP에서 관리되는 필드들 중 입력된 인덱스에 해당되는 필드를 반환해야 한다. CP가 로드될 때 호출된다.
- `GetCredentialCount()` : CP에서 관리하는 Credential(타일)의 갯수가 총 몇개인지 반환해야 한다. CP가 로드될 때 호출된다.
- `GetCredentialAt()` : CP에서 관리하는 Credential(타일)의 중 입력된 인덱스에 해당하는 Credential을 반환해야 한다. CP가 로드될 때 호출된다.
- `SetUserArray()` : CP 화면에서 출력할 수 있는 현재 시스템에 등록된 사용자들의 정보가 입력된다.



CPUS_UNLOCK_WORKSTATION은 CredentialProvider V1의 잔재로 V2에서는 실제로 화면 잠금 시나리오에서도 CPUS_LOGON으로 입력된다.{: .notice--warning}

#### ICredentialProviderCredential2 인터페이스

CP는 LogonUI.exe 에 의해 호출되는 입장이기 때문에 구성된 메서드들은 모두 LogonUI.exe 에게 정보를 반환하는 모양을 띈다. 주요 인터페이스 메서드들은 아래와 같으며 메서드에 맞게 기능을 구현해 줘야 정상 동작한다. 크게 CP UI를 제어하기 위해 정보를 얻는 메서드군과 인증 파트 메서드 군으로 나눌 수 있다. 

```c++
// UI 정보 파트 메서드
IFACEMETHODIMP GetStringValue(DWORD dwFieldID, _Outptr_result_nullonfailure_ PWSTR *ppwsz);
IFACEMETHODIMP GetBitmapValue(DWORD dwFieldID, _Outptr_result_nullonfailure_ HBITMAP *phbmp);
IFACEMETHODIMP GetCheckboxValue(DWORD dwFieldID, _Out_ BOOL *pbChecked, _Outptr_result_nullonfailure_ PWSTR *ppwszLabel);
IFACEMETHODIMP GetComboBoxValueCount(DWORD dwFieldID, _Out_ DWORD *pcItems, _Deref_out_range_(<, *pcItems) _Out_ DWORD *pdwSelectedItem);
IFACEMETHODIMP GetComboBoxValueAt(DWORD dwFieldID, DWORD dwItem, _Outptr_result_nullonfailure_ PWSTR *ppwszItem);
IFACEMETHODIMP GetSubmitButtonValue(DWORD dwFieldID, _Out_ DWORD *pdwAdjacentTo);

IFACEMETHODIMP SetStringValue(DWORD dwFieldID, _In_ PCWSTR pwz);
IFACEMETHODIMP SetCheckboxValue(DWORD dwFieldID, BOOL bChecked);
IFACEMETHODIMP SetComboBoxSelectedValue(DWORD dwFieldID, DWORD dwSelectedItem);
IFACEMETHODIMP CommandLinkClicked(DWORD dwFieldID);

// 인증 파트 메서드
IFACEMETHODIMP GetSerialization(_Out_ CREDENTIAL_PROVIDER_GET_SERIALIZATION_RESPONSE *pcpgsr,
                                _Out_ CREDENTIAL_PROVIDER_CREDENTIAL_SERIALIZATION *pcpcs,
                                _Outptr_result_maybenull_ PWSTR *ppwszOptionalStatusText,
                                _Out_ CREDENTIAL_PROVIDER_STATUS_ICON *pcpsiOptionalStatusIcon);
IFACEMETHODIMP ReportResult(NTSTATUS ntsStatus,
                            NTSTATUS ntsSubstatus,
                            _Outptr_result_maybenull_ PWSTR *ppwszOptionalStatusText,
                            _Out_ CREDENTIAL_PROVIDER_STATUS_ICON *pcpsiOptionalStatusIcon);

public:
HRESULT Initialize(CREDENTIAL_PROVIDER_USAGE_SCENARIO cpus,
                   _In_ CREDENTIAL_PROVIDER_FIELD_DESCRIPTOR const *rgcpfd,
                   _In_ FIELD_STATE_PAIR const *rgfsp,
                   _In_ ICredentialProviderUser *pcpUser);

```

UI 관련 메서드들은 모두 CP UI가 그려질 때 호출되기 때문에 UI를 구성하고 있는 필드의 ID를 키로 정보를 설정하고 반환할 수 있도록 구현되어야 한다. 

- `GetStringValue()` : 입력된 아이디에 해당하는 필드의 문자열을 반환해야 한다. 
- `GetBitmapValue()` : 입력된 아이디에 해당하는 필드의 비트맵 정보를 반환해야 한다. 보통 타일 이미지에 대한 비트맵을 반환하여 타일을 원하는 이미지로 표시하게 할 수 있다.
- `GetCheckboxValue()` : 입력된 아이디에 해당하는 체크 박스 필드의 값을 반환해야 한다.
- `GetComboBoxValueCount()` : 입력된 아이디에 해당하는 콤보 박스에 필드가 몇개 있는지 반환해야 한다.
- `GetSubmitButtonValue()` : 로그인 버튼(화살표)이 어느 필드 옆에 도킹되었는지 LogonUI 쪽에서 확인하는 메서드로 도킹할 필드 아이디를 반환하면 로그인 버튼의 위치를 변경할 수 있다.
- `SetStringValue()` : 입력된 아이디에 해당하는 필드에 문자열을 설정되도록 구현해야 한다.
- `SetCheckboxValue()` : 입력된 아이디에 해당하는 체크 박스의 값이 설정되도록 구현해야 한다.
- `SetComboBoxSelectedValue()` : 사용자가 콤보박스의 아이템을 선택했을 때 호출된다.
- `CommandLinkClicked()` : 사용자가 링크 버튼을 클릭했을 때 호출된다.
- `Initialize()` : 이 메서드는 인터페이스에 속한 메서드는 아니지만 Provider에서 각 사용자마다 타일을 생성할 때 처음 호출되는 메서드로 CP의 최초 화면 구성이 어떻게 되어야하는지를 정의하는 메서드이다. 샘플코드에서는 CP 가 처음 로드되면 호출되도록 구현되어 있다.



인증 파트 메서드는 Windows의 인증을 담당하는 LSA(Local Security Authority) 컴포넌트로 인증을 요청하기 전과 요청 후에 호출되는 메서드들로 구성되어 있다.

- `GetSerialization` : LSA 컴포넌트에 인증 요청 데이터를 전송하기 위해 필요한 구조체를 반환해야 한다.  사용자가 로그인 버튼(화살표 버튼)을 클릭했을 때 호출된다.
- `ReportResult` : LSA 인증이 요청 후 결과를 통보 받는다. 인증 성공 여부가 입력된다. 로그인 수행이 완료되었을 때 호출된다.



