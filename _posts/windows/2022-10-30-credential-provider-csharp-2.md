---
title : "[Windows]CredentialProvider V2 with C# - 포팅(Porting)"
categories:
- Winsys
tag :
- [Windows, CredentialProvider V2, C#]
toc: true
toc_sticky : true
published : true
date : 2022-10-30
last_modified_at : 2022-10-30

---



## Windows Credential Provider V2 with C# - 포팅(Porting)

[![Readme Card](https://github-readme-stats.vercel.app/api/pin/?username=djcho&repo=windows-credential-provider-dotnet)](https://github.com/djcho/windows-credential-provider-dotnet)

이 문서에서는 Microsoft 에서 제공하는 Windows Credential Provider(이하 CP) C++샘플 코드를 C# 샘플 코드로 포팅하는 방법에 대해 설명한다. 포팅을 하기 위해 필요한 사전 작업들은 아래 포스팅에서 확인하고 자세한 코드는 위 Repository 를 확인한다.

- <a href="https://djcho.github.io/winsys/credential-provider-csharp/" target="_blank">https://djcho.github.io/winsys/credential-provider-csharp/</a>



### 포팅

#### 프로젝트 생성

CP의 Interop.dll 생성에 성공했다면 본격적으로 C++ 샘플 프로젝트를 C# 프로젝트로 포팅하는 방법에 대해 알아보겠다. LogonUI.exe에서 COM 인터페이스로 호출할 수 있도록 dll 형태의 클래스라이브러리 C# 프로젝트를 생성한다.

![image](https://user-images.githubusercontent.com/13410737/198545561-ad080943-d9d2-453b-a20e-ff418c27f67a.png){: .align-center}

위와 비슷한 프로젝트 중 .NET Standard를 대상으로 하는 클래스 라이브러리 프로젝트가 존재하는데, 해당 프로젝트로 혼동하여 생성하지 않도록 주의한다. 📢
{: .notice--warning}



#### CredentialProvider.Interop.dll 참조 추가

프로젝트를 생성한 뒤 CP Interface 를 호출하기 위해  위에서 생성한 CredentialProvider.Interop.dll를 참조 추가 한다.

![image](https://user-images.githubusercontent.com/13410737/198545774-2d3fd1dd-e2ea-47cc-953b-b76c510ef110.png){: .align-center}

위와 같이 참조를 추가하면 이제 해당 프로젝트에서 `credentialprovider.idl`파일에서 export하고 있는 인터페이스를 `using`키워드로 import하여 사용할 수 있다.



#### Provider 클래스 구현

Provider 클래스는 여러개의 인증 수단 타일(Credential)을 제공할 수 있는 개념의 인증 제공자로 ICredentialProvider 인터페이스를 재정의해야 한다. 

##### ICredentialProvider 인터페이스 구현

```C#
[ComVisible(true)]
[Guid(Constants.CredentialProviderUID)]
[ClassInterface(ClassInterfaceType.None)]
[ProgId("CSharpProvider.CSharpSampleProvider")]
public class CSharpSampleProvider : ICredentialProvider, ICredentialProviderSetUserArray
{
  ...
}
```

CP는 COM 인터페이스를 통해 LogonUI.exe 가 호출해 주기 때문에 `Comvisible` 속성을 부여하여 COM 노출을 한다. 그리고 CP는 Windows 시스템에서 각 CP들(Builtin CP든 Custom CP든..)을 식별하고 로드할 때 Guid를 사용하므로 Guid 속성을 사용하여 해당 Provider 의 식별자를 지정해 준다. 추후 CP를 시스템에 등록할 때 이 Guid를 사용한다.

> **Tip. 💡**
>
> 이 Guid 로 각 Provider들이 제공하는 Credential 타일들의 출력 순서도 결정되므로 여러 Provider 들을 운영할 때 타일의 순서를 변경하고 싶다면 각 Provider의 Guid를 변경하면 된다. 순서는 2-4-1-3 크기 순으로 출력된다. (Windows 10 기준)



`ICredentialProvder` 인터페이스의 가장 핵심적인 부분은 `GetCredentialCount()`와 `GetCredentialAt()` 메서드이다. 이 Provider에서 출력하고 싶은 타일(Credential)의 갯수를 LogonUI에게 알려주고, LogonUI가 화면에 타일을 열거할 때(`GetCredentialAt()`이 호출될 때)우리가 생성하는 Credential 객체를 반환 시키면 된다. (샘플에서는 `GetCredentialCount()`가 호출되면 Credential 객체를 생성하지만 꼭 그러지 않아도 된다. )

```c#
// Returns the credential at the index specified by dwIndex. This function is called by logonUI to enumerate
// the tiles.
public int GetCredentialAt(uint dwIndex, out ICredentialProviderCredential ppcpc)
{
  Log.LogMethodCall();

  if (_pCredential == null)
  {
    _pCredential = new CSharpSampleCredential();
  }

  ppcpc = (ICredentialProviderCredential)_pCredential;
  return HResultValues.S_OK;
}
```



샘플에서는 Credential 의 갯수를 1로 고정했지만 실제로 개발할 때는 모든 사용자에게 타일을 보여주는 것이 일반적이라  `ICredentialProviderSetUserArray` 인터페이스에 의해 넘어오는 UserArray의 사용자 정보의 갯수만큼 설정하여 리턴해야 한다.



#### Credential 클래스 구현

Provider를 구현했다면 사용자 화면에 타일로 보여지는 Credential 를 구현해야 한다. Credential은 Provider에서 정의한 UI 필드에 대한 구체적인 동작을 LogonUI에게 전달하며, 실제로 인증을 수행해야 한다.



##### ICredentialProviderCredential2 인터페이스 구현

```C#
[ComVisible(true)]
[ClassInterface(ClassInterfaceType.None)]
public class CSharpSampleCredential : ICredentialProviderCredential2, ICredentialProviderCredentialWithFieldOptions
{    
}
```

마찬가지로 COM노출이 되어야하기 때문에 Provider와 동일하게 `ComVisible` 속성과 `ClassInterface` 속성을 부여해야 한다. 

##### 필드 관련 메서드

필드 관련 메서드들은 아래 코드와 처럼 `dwFieldId`에 해당하는 필드id를 필드 자료구조에서 찾아 값을 설정해 주거나 리턴해 주도록 구현하면 된다.

```c#
// Sets ppwsz to the string value of the field at the index dwFieldID
public int GetStringValue(uint dwFieldID, out string ppsz)
{
  ppsz = null;
  int hr = HResultValues.S_OK;

  if(dwFieldID < _rgCredProvFieldDescriptors.Length)
  {
    // Make a copy of the string and return that. The caller
    // is responsible for freeing it.
    ppsz = _rgFieldStrings[dwFieldID];
  }
  else
  {
    hr = HResultValues.E_INVALIDARG;
  }

  return hr;
}

// Sets the value of a field which can accept a string as a value.
// This is called on each keystroke when a user types into an edit field
public int SetStringValue(uint dwFieldID, string psz)
{
  int hr = HResultValues.S_OK;

  if((dwFieldID < _rgCredProvFieldDescriptors.Length) && 
     (_CREDENTIAL_PROVIDER_FIELD_TYPE.CPFT_EDIT_TEXT == _rgCredProvFieldDescriptors[dwFieldID].cpft ||
      _CREDENTIAL_PROVIDER_FIELD_TYPE.CPFT_PASSWORD_TEXT == _rgCredProvFieldDescriptors[dwFieldID].cpft))
  {
    _rgFieldStrings[dwFieldID] = psz;
    hr = HResultValues.S_OK;
  }
  else
  {
    hr = HResultValues.E_INVALIDARG;
  }

  return hr;
}
```

그 중 `Initiialize()` 메서드는 CP인터페이스와는 무관한 메서드이지만 해당 타일이 초기 어떤 모양으로 화면에 출력될지를 정의할 수 있도록 샘플에서 아주 잘 만들어 둔 메서드이므로 각 필드 값이 최초 화면에서 어떻게 표시될지 적절하게 값을 설정한다.



##### 인증 관련 메서드

인증 관련 메서드는 실제로 인증을 위해 필요한 데이터를 직렬화하여 LSA에게 인증 요청 패킷을 보내는 `GetSerialization()`, 인증 결과를 통보 받는 `ReportResult()` 메서드가 있다.

`GetSerialization()` 메서드에서는 `_CREDENTIAL_PROVIDER_CREDENTIAL_SERIALIZATION` 구조체 모양으로 인증에 필요한 각종 데이터를 직렬화 시켜야 한다. 아래와 같이 메모리를 구성하면 된다. 순서대로 `KERB_INTERACTIVE_UNLOCK_LOGON` + {도메인 이름} + {유저 이름} + {암호화된 비밀번호} 로 패킷을 생성하면 되는데 뒤에 붙는 세가지의 문자열은 아래와 같은 규칙을 지켜야 한다.

- 각 문자열을 복사한다.
- 문자열이 시작되는 버퍼 포인터를 버퍼의 시작점 기준 오프셋으로 수정한다.

패킷을 정상적으로 생성했다면 메모리 형태는 아래와 같은 모양이어야 한다. 위 작업들을 해주는 hepler.cs 의 메서드는 아래와 같다.

![image](https://user-images.githubusercontent.com/13410737/198884319-5ce86ba9-3aba-41b7-bca3-bf06f04dcc54.png){: .align-center}

```c#
internal static int KerbInteractiveUnlockLogonPack(PInvoke.KERB_INTERACTIVE_UNLOCK_LOGON kiul, out IntPtr rgbSerialization, out uint cbSerialization)
{
  // set up the Logon structure within the KERB_INTERACTIVE_UNLOCK_LOGON
  PInvoke.KERB_INTERACTIVE_UNLOCK_LOGON kiulOut = kiul;
  KERB_INTERACTIVE_LOGON kil = kiulOut.Logon;

  // alloc space for struct plus extra for the three strings
  int cb = Marshal.SizeOf(kiulOut) +
    kil.LogonDomainName.Length +
    kil.UserName.Length +
    kil.Password.Length;

  //
  // copy each string,
  // fix up appropriate buffer pointer to be offset,
  // advance buffer pointer over copied characters in extra space
  //
  IntPtr buffer = Marshal.AllocHGlobal(cb);
  IntPtr structBuffer = Marshal.AllocHGlobal(Marshal.SizeOf(kiulOut));

  int pos = Marshal.SizeOf(kiulOut);
  CopyMemory(buffer + pos, kil.LogonDomainName.Buffer, kil.LogonDomainName.Length);
  kiulOut.Logon.LogonDomainName.Buffer = (IntPtr)((buffer.ToInt64() + (Int64)pos) - buffer.ToInt64());

  pos += kil.LogonDomainName.Length;
  CopyMemory(buffer + pos, kil.UserName.Buffer, kil.UserName.Length);
  kiulOut.Logon.UserName.Buffer = (IntPtr)((buffer.ToInt64() + (Int64)pos) - buffer.ToInt64());

  pos +=  kil.UserName.Length;
  CopyMemory(buffer + pos, kil.Password.Buffer, kil.Password.Length);
  kiulOut.Logon.Password.Buffer = (IntPtr)((buffer.ToInt64() + (Int64)pos) - buffer.ToInt64());

  Marshal.StructureToPtr(kiulOut, structBuffer, false);
  CopyMemory(buffer, structBuffer, Marshal.SizeOf(kiulOut));

  rgbSerialization = Marshal.AllocHGlobal(cb);
  CopyMemory(rgbSerialization, buffer, cb);

  Marshal.FreeHGlobal(buffer);

  cbSerialization = (uint)cb;

  return HResultValues.S_OK;
}
```



### 설치 방법

포팅이 완료된 C#용 CP 바이너리를 테스트하기 위해서는 우선 COM으로 해당 바이너리를 등록 시킨 후 Windows 시스템에 Guid를 등록해야 한다.

우선 빌드된 CP 바이너리를 `C:\Windows\System32` 폴더에 복사한 아래 과정을 따른다.



###### COM등록

regass.exe /codebase {테스트 CP바이너리 경로}

```bash
C:\Windows\Microsoft.NET\Framework64\v4.0.30319\RegAsm.exe /codebase "CSharpCredentialProvider.dll"
```



###### CP 등록

CP를 시스템에 등록하기 위해 위 Provider 클래스에서 지정한 Guid를 레지스트리에 등록 시킨다. 아래 코드를 .reg 파일로 따로 생성하여 실행하도록 한다.

```bash
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Authentication\Credential Providers\{298D9F84-9BC5-435C-9FC2-EB3746625954}]
@="CSharp Provider"
```



설치가 정상적으로 되었다면 화면 잠금 화면으로 돌입 시 아래 화면과 같이 샘플 CP 가 출력되는 것을 확인할 수 있다.

![image](https://user-images.githubusercontent.com/13410737/195765080-84d30aca-6dd5-4bd1-a8ef-5ee4b4144397.png)
