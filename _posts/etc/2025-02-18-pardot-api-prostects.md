---
title: "[Salesforce] Pardot API를 활용한 잠재고객 관리 가이드"
categories:
  - Etc
tag:
  - [Etc, Salesforce, laravel, pardot]
toc: true
toc_sticky: true
published: true
date: 2025-02-18 00:00:00
last_modified_at: 2025-02-18 00:00:00
---

# [Salesforce] Pardot API를 활용한 잠재고객 관리 가이드

## 개요

이 문서는 **Salesforce Pardot API**를 활용하여 **Prospect(잠재고객)을 효과적으로 관리하는 방법**을 설명한다. 이 문서는 Salesforce 및 Pardot을 처음 접하는 개발자 및 마케팅 담당자를 대상으로 하며, API를 활용하여 잠재고객을 추가, 조회, 삭제 및 갱신하는 과정을 다룬다.

이를 위해 먼저 **Salesforce와 Pardot의 관계를 이해**하고, Salesforce 시스템에서 고객 데이터가 어떻게 흐르는지를 설명한다. 이후 **Prospect(잠재고객)의 개념 및 수집 방법(Form Handler vs API)을 비교**하고, **API를 사용하기 위한 준비 과정**을 상세히 설명한 후, **실제 API를 호출하는 예제 코드**를 포함하여 Prospect를 관리하는 방법을 다룰 것이다.

## Pardot과 Salesforce의 관계

Salesforce와 Pardot은 같은 Salesforce 생태계에 속해 있지만, 주요 역할이 다르다. 간단히 말하면, **Pardot은 마케팅 자동화를 위한 도구, Salesforce CRM은 영업 및 고객 관리를 위한 도구**이다.

| 항목          | **Pardot (Salesforce Account Engagement)**        | **Salesforce CRM**                 |
| ------------- | ------------------------------------------------- | ---------------------------------- |
| 주요 목적     | 마케팅 자동화 및 리드 육성                        | 영업 및 고객 관리                  |
| 사용자        | 마케팅 팀 (이메일 마케팅, 광고 추적 등)           | 영업 팀 (거래 및 고객 관계 관리)   |
| 주요 기능     | 이메일 캠페인, 양식(Form), 잠재고객 추적(Scoring) | 리드, 기회(Opportunity), 계약 관리 |
| 데이터 흐름   | 웹사이트 방문자 → Prospect 생성                   | Prospect → Lead → 고객(Contact)    |
| API 사용 목적 | 잠재고객 추가, 조회, 삭제, 업데이트               | 리드, 기회, 계약 관리              |

Pardot과 Salesforce CRM은 서로 연동되며, Pardot에서 생성된 Prospect는 Salesforce의 Lead로 전환될 수 있다. 즉, **Pardot에서 마케팅 활동을 통해 수집된 데이터가 Salesforce에서 영업 활동으로 이어지는 역할**을 한다.

## Salesforce 시스템에서 고객 데이터의 흐름

Salesforce에서 고객 데이터는 단계적으로 발전하며, 특정 조건을 충족하면 다음 단계로 전환된다. 아래는 Salesforce에서 Prospect(잠재고객)부터 Customer(고객)까지의 흐름을 시각적으로 표현한 것이다.

```
익명 방문자 (웹사이트 방문)
   ↓
Prospect (Pardot에서 이메일 확보)
   ↓
Lead (Salesforce CRM으로 전환, 영업팀 관리)
   ↓
Contact (Salesforce CRM에서 관계 유지)
   ↓
Opportunity (거래 진행 중)
   ↓
Customer (최종 계약 완료)
```

**단계별 개념 정리**

| 개념                    | 역할                                    | 관리 위치      | 주요 활동                    |
| ----------------------- | --------------------------------------- | -------------- | ---------------------------- |
| 익명 방문자             | 웹사이트를 방문했지만 정보 미제공       | Pardot         | 추적(cookie)                 |
| **Prospect (잠재고객)** | **이메일 및 기본 정보를 제공한 사용자** | **Pardot**     | **이메일 마케팅, 광고 추적** |
| Lead (리드)             | 영업 기회가 있는 고객                   | Salesforce CRM | 영업팀이 직접 연락           |
| Contact (연락처)        | 리드 중에서 장기적으로 관리할 고객      | Salesforce CRM | 고객 관계 관리               |
| Opportunity (기회)      | 실제 계약 진행 중인 고객                | Salesforce CRM | 견적 요청, 가격 협상         |
| Customer (고객)         | 최종적으로 계약이 완료된 고객           | Salesforce CRM | 계약 관리, 후속 지원         |

Pardot에서 마케팅 활동을 통해 수집된 Prospect가 일정 조건을 충족하면 Salesforce CRM으로 넘어가고, 영업팀이 관리하게 된다.

## Prospect(잠재고객)이란?

### Prospect(잠재고객)의 정의

**Prospect(잠재고객)은 Pardot에서 관리하는 고객 데이터 단위**로, 기업과 어떤 형태로든 상호작용한 개인 또는 조직을 의미한다. 이들은 아직 정식 고객(Customer)이 아니지만, 기업의 제품이나 **서비스에 관심을 보인 사람들로서 마케팅 및 영업 프로세스의 핵심 대상**이 된다.

### Prospect(잠재고객)이 생성되는 과정

Prospect는 기업과의 상호작용을 기반으로 생성될 수 있으며, 일반적으로 두 가지 방식으로 등록될 수 있다. 일부 시스템에서는 Prospect가 **여러 단계의 고객 인터랙션을 거친 후 생성**될 수도 있고, 다른 경우에는 **단일 이벤트(예: 회원가입, 양식 제출)만으로 즉시 Prospect가 생성**될 수도 있다.

어떤 방식을 사용할지는 **기업의 마케팅 및 영업 전략, 시스템 설계 방식, Salesforce Pardot 설정** 등에 따라 달라질 수 있다.

#### Prospect가 생성되는 두 가지 방식

| Prospect 생성 방식       | 설명                                                                                                  |
| ------------------------ | ----------------------------------------------------------------------------------------------------- |
| **다단계 인터랙션 방식** | Prospect는 웹사이트 방문, 광고 클릭, 이메일 열람 등 여러 단계의 상호작용을 거치면서 점진적으로 생성됨 |
| **단일 이벤트 방식**     | Prospect는 단 한 번의 특정 행동(예: 회원가입, 양식 제출)으로 즉시 생성됨                              |

일반적으로 Salesforce 및 Pardot을 사용하는 많은 조직에서는 Prospect를 **여러 상호작용을 기반으로 생성하는 방식**을 채택하지만, 특정한 경우에는 **단일 이벤트만으로 Prospect를 즉시 생성하는 방식**이 더 적합할 수도 있다.

각 방식의 차이점은 다음과 같다.

##### 회원가입을 통한 Prospect 생성 vs 다단계 방식

| 비교 항목              | **회원가입을 통한 Prospect 생성**                               | **다단계 인터랙션 후 Prospect 생성**                |
| ---------------------- | --------------------------------------------------------------- | --------------------------------------------------- |
| **Prospect 생성 시점** | 회원가입을 완료한 즉시                                          | 특정 기준(점수, 행동 누적 등) 충족 시               |
| **활용 사례**          | SaaS 서비스, B2B 솔루션, 특정 커뮤니티 가입                     | 웹사이트 방문 분석, 리드 육성, 장기적 마케팅 캠페인 |
| **장점**               | Prospect를 빠르게 확보할 수 있으며, 즉시 CRM과 연동 가능        | Prospect의 관심도를 충분히 분석한 후 생성 가능      |
| **단점**               | Prospect의 실질적인 관심도를 충분히 평가하지 못할 가능성이 있음 | Prospect 확보 속도가 느릴 수 있음                   |

Prospect는 기업의 목표와 마케팅 전략에 따라 다르게 생성될 수 있다. 어떤 방식이 더 적절한지는 조직의 마케팅 전략과 시스템 구조에 따라 달라질 수 있으며, Salesforce와 Pardot의 설정을 어떻게 구성하느냐에 따라 Prospect가 생성되는 방식도 다를 수 있다.

#### 💡 **쉽게 이해하기 위한 예제 시나리오**

> **예제 1: 회원가입을 통한 Prospect 생성** (단일 이벤트 방식)
>
> - 사용자가 웹사이트에서 회원가입을 완료
>
> - 회원가입 과정에서 이메일, 이름, 회사 정보를 입력
>
> - Pardot API를 호출하여 입력된 정보를 Prospect로 등록
>
> - Prospect는 특정 캠페인(예: "신규 가입자 캠페인")과 자동으로 연결됨
>
>   📌 **적합한 Prospect 생성 방식: `Pardot API`**
>   ✔ **즉시 Prospect를 생성하고 백엔드에서 데이터를 조작해야 할 경우** API가 적합
>   ✔ **CRM, ERP 등과 연동하여 Prospect 데이터를 추가로 관리해야 할 경우** API를 활용
>   ✔ **회원가입 이후 특정한 비즈니스 로직(예: 자동 태깅, 점수 조정 등)이 필요할 경우** API가 유리

> **예제 2: 웹사이트 내 여러 행동을 거쳐 Prospect가 생성되는 경우** (다단계 인터랙션 방식)
>
> - 사용자가 기업의 블로그를 방문하고 여러 페이지를 탐색
>
> - 이후 "무료 eBook 다운로드" 페이지에서 이메일을 입력하고 다운로드 요청
>
> - Pardot은 이메일을 등록하고 Prospect로 저장
>
> - Prospect는 이후 **추가적인 행동(예: 뉴스레터 열람, 제품 페이지 방문 등)**을 분석하여 마케팅 자동화에 활용됨
>
>   📌 **적합한 Prospect 생성 방식: `Pardot Form Handler + Pardot Tracking`**
>   ✔ **웹사이트에서 여러 상호작용을 분석하고 Prospect를 자동 등록하려면 Form Handler가 적합**
>   ✔ **Pardot Tracking 코드를 웹사이트에 삽입하여 사용자의 행동을 분석 가능**
>   ✔ **다운로드, 클릭 등의 행동 데이터를 기반으로 점수를 부여할 수 있음**

> **예제 3: 이메일 마케팅 반응을 통해 Prospect가 생성되는 경우**
>
> - 기업이 뉴스레터를 통해 새로운 제품 정보를 발송
>
> - 사용자가 이메일을 열어보고 포함된 링크를 클릭하여 제품 페이지 방문
>
> - Pardot은 이메일 오픈 및 클릭 이벤트를 추적
>
> - 일정 점수를 넘으면 Prospect로 자동 등록
>
>   📌 **적합한 Prospect 생성 방식: `Pardot Tracking + Automation Rules`**
>   ✔ **이메일을 열어본 사용자를 자동으로 Prospect로 변환하려면 Pardot Tracking을 활용**
>   ✔ **Pardot 내 자동화 규칙(Automation Rules)을 사용하여 특정 기준을 충족한 사용자를 Prospect로 등록 가능**
>   ✔ **이메일 마케팅의 효과 분석이 중요한 경우 활용**

이제, Prospect를 생성하는 구체적인 방법인 **Pardot API를 활용한 Prospect 추가 방식**을 살펴보자. 🚀

## Pardot API 사용 준비

Pardot API를 사용하여 Prospect를 추가, 조회, 삭제, 갱신하려면 먼저 **인증 및 환경 설정을 완료해야 한다.** Salesforce는 Pardot API를 보호하기 위해 OAuth 2.0 인증을 요구하며, API 요청을 보내기 전에 **올바른 권한과 설정**이 필요하다.

### Pardot API 사용을 위한 필수 준비 사항

1. Salesforce 계정 및 Pardot 액세스 권한
2. Connected App 생성 (API 인증을 위한 Client ID & Secret 확보)
3. OAuth 2.0을 사용한 Access Token 발급 방법 이해
4. Pardot Business Unit ID 확인
5. API 사용을 위한 Salesforce 권한 설정

각 단계별로 필요한 설정 방법을 설명한다.

### 1. Salesforce 계정 및 Pardot 액세스 권한

API를 사용하려면 **Pardot (Salesforce Account Engagement)**을 사용할 수 있는 **Salesforce 계정**이 필요하다. 또한, API를 호출하는 계정은 **Pardot API에 접근할 수 있는 권한**이 있어야 한다.

📌 **필요한 사항**
✔ Salesforce 계정이 활성화되어 있어야 한다.
✔ 사용자가 **Pardot Admin 역할**을 가지고 있어야 한다.
✔ API 요청을 실행할 **Salesforce 사용자가 SSO(Single Sign-On)를 통해 Pardot에 로그인할 수 있어야 한다.**

🛠️**Salesforce 권한 확인 방법**

1. **Salesforce 로그인 후 "설정" 이동**

   ![image](https://github.com/user-attachments/assets/9dce85df-f5f5-409f-a3a8-4fe339b57068)

2. **"사용자" 메뉴에서 API를 사용할 사용자 선택**

   ![image](https://github.com/user-attachments/assets/9874159a-b7d4-46ff-9cd7-721661589908)

3. **사용자의 "Profile"에서 Pardot API 관련 권한 확인**

   ![image](https://github.com/user-attachments/assets/0c881968-c018-4ed5-a5d8-629bbc1d3f40)

4. **필요 시 "API Enabled" 권한 부여**

   ![image](https://github.com/user-attachments/assets/0eeff55b-335c-4741-bbf8-8c1078f4ad25)

---

### 2. Connected App 생성 (API 인증을 위한 Client ID & Secret 확보)

Pardot API를 호출하려면 **Salesforce의 Connected App을 생성하여 Client ID 및 Client Secret을 발급받아야 한다.** 이 정보는 OAuth 2.0을 사용하여 **Access Token**을 얻는 데 필요하다.

🛠️ **Connected App 생성 방법**

1. **Salesforce에 로그인 후 "설정" 이동**

   ![image](https://github.com/user-attachments/assets/9dce85df-f5f5-409f-a3a8-4fe339b57068)

2. **"App Manager" 검색 후 이동**

   ![image](https://github.com/user-attachments/assets/16659a25-8944-4c0a-a1c9-db0608e966e8)

3. **"새 연결된 앱" 버튼 클릭**

   ![image](https://github.com/user-attachments/assets/4298b746-177e-4100-b550-ba28583053b3)

   ![image](https://github.com/user-attachments/assets/e5cdd33b-5d67-4630-8cd3-ed8adb748dea)

4. 다음 설정을 입력

   - **App Name**: `Pardot API Integration`

   - **API (OAuth 설정 활성화) - OAuth 설정 활성화**

   - **Callback URL**: `https://login.salesforce.com/services/oauth2/callback`

   - **Selected OAuth Scopes**

     - `API를 통해 사용자 데이터 관리 (API)`
     - `언제든지 요청 수행 (refresh_token, offline_access)`
     - `Pardot 서비스 관리(pardot_api)`

     ![image](https://github.com/user-attachments/assets/0498126a-6f58-423a-948e-5a58f1cbc0fa)

5. **Connected App 생성 후 "고객 키 (Client ID)" 및 "고객 암호 (Client Secret)" 저장**

   - 위 정보를 다시 보고 싶다면 [앱 관리자] > 생성했던 "연결된 앱"의 [우측 화살표] > [보기] 버튼 클릭하여 아래 [고객 세부 사항 관리] 버튼 클릭

   ![image](https://github.com/user-attachments/assets/5c1722c0-3726-4367-aaf1-8b2698c66dba)

### 3. Pardot Business Unit ID 확인

Pardot API를 호출할 때 **Business Unit ID**가 필요하며, 이는 **Salesforce Setup**에서 확인할 수 있다.

🛠️ **Pardot Business Unit ID 확인 방법**

1. **Salesforce에 로그인 후 "설정" 이동**

   ![image](https://github.com/user-attachments/assets/9dce85df-f5f5-409f-a3a8-4fe339b57068)

2. **"사업부 설정" 검색 후 선택**

   ![image](https://github.com/user-attachments/assets/5f93ef50-45d6-407f-9868-f540d50f7810)

3. **"사업부 ID" 값을 확인** (`0Uv2u000000000pCAA` 형식)

   ![Image](https://github.com/user-attachments/assets/d868f043-bfe1-4e0b-a34a-1f7d9e8e0cf4)

## Pardot API 사용 방법 (Laravel PHP)

이제 Laravel을 사용하여 **Pardot API를 호출하는 예제 코드**를 제공한다. 아래의 코드에서는 **Access Token 발급 → Prospect 추가 → 수정 → 조회 → 삭제**의 전체 프로세스를 포함한다.

잠재고객의 현황은 `https://pi.pardot.com/prospect`에서 확인 가능하다.

![Image](https://github.com/user-attachments/assets/1245cb0c-f435-4a8a-89d6-8606ac2c07da)

📌 **전제 조건**
✔ Laravel 프로젝트가 설정되어 있어야 한다.
✔ `Http` Facade (`use Illuminate\Support\Facades\Http;`)을 사용하여 API 호출을 수행한다.
✔ `Pardot API v4`를 사용하며, `OAuth 2.0` 인증 방식을 따른다. Pardot App의 패키지 버전이 5버전 대역일 경우는 `v5` API를 사용해야 하며, 공식 문서를 확인하면 된다.

---

## 🔹 1️⃣ 로그인 (Access Token 발급)

Pardot API를 호출하려면 **OAuth 2.0 Access Token**을 먼저 발급받아야 한다.
Access Token은 모든 API 요청에서 **헤더에 포함**해야 하며, 만료되면 새로 발급받아야 한다.

### 1. 로그인 (Access **Token** 발급)

Pardot API v4를 호출하려면 **OAuth 2.0 Access Token**을 먼저 발급받아야 한다. Access Token은 모든 API 요청에서 **헤더에 포함**해야 하며, 만료되면 새로 발급받아야 한다.

```php
use Illuminate\Support\Facades\Http;

function getPardotAccessToken() {
    $response = Http::asForm()->post('https://login.salesforce.com/services/oauth2/token', [
        'grant_type'    => 'password',
        'client_id'     => '<YOUR_CLIENT_ID>',
        'client_secret' => '<YOUR_CLIENT_SECRET>',
        'username'      => '<YOUR_SALESFORCE_USERNAME>',
        'password'      => '<YOUR_SALESFORCE_PASSWORD>',
    ]);

    if ($response->successful()) {
        return $response->json()['access_token'];
    } else {
        throw new Exception('Access Token 발급 실패: ' . $response->body());
    }
}

// Access Token 가져오기
$access_token = getPardotAccessToken();
```

**설명**

- `Http::asForm()->post(...)`를 사용하여 Access Token을 요청한다.
- `client_id`, `client_secret`, `username`, `password` 정보를 입력해야 한다.
- 응답 성공 시 `access_token`을 반환하며, 이후 모든 API 요청에서 사용된다.

### 2. 잠재고객 추가 (Prospect Create)

Pardot에 **새로운 Prospect(잠재고객)을 추가하는 API**를 호출한다.
이 예제에서는 **기본 필드 + 커스텀 필드**를 함께 추가한다.

```php
function createProspect($access_token) {
    $url = "https://pi.pardot.com/api/prospect/version/4/do/create?format=json";

    $prospect_data = [
        'email'       => 'testuser@example.com',
        'first_name'  => 'John',
        'last_name'   => 'Doe',
        'company'     => 'Example Corp',
        'phone'       => '123-456-7890',
        'custom_field__c' => 'Custom Value'
    ];

    $response = Http::asForm()->withHeaders([
        'Authorization' => "Bearer $access_token",
        'Pardot-Business-Unit-Id' => '<YOUR_BUSNIESS_UNIT_ID>',
        'Content-Type' => 'application/x-www-form-urlencoded',
    ])->post($url, $prospect_data);

    return $response->json();
}

// Prospect 생성
$new_prospect = createProspect($access_token);
print_r($new_prospect);
```

📌 **설명**
✔ `POST /api/v5/prospects`를 호출하여 새로운 Prospect를 추가한다.
✔ `custom_field__c`와 같은 **커스텀 필드(Custom Field)**도 포함할 수 있다.
✔ `Pardot-Business-Unit-Id`를 헤더에 반드시 포함해야 한다.

## 3. 잠재고객 수정 (Prospect Update)

기존 Prospect의 **정보를 업데이트**한다.
이 예제에서는 **기본 필드(회사명) 및 커스텀 필드(Custom Field)**를 수정한다.

```php
function updateProspect($access_token, $prospect_id) {
    $url = "https://pi.pardot.com/api/prospect/version/4/do/update/id/$prospect_id?format=json";

    $updated_data = [
        'company' => 'Updated Example Corp',
        'custom_field__c' => 'Updated Custom Value'
    ];

    $response = Http::asForm()->withHeaders([
        'Authorization' => "Bearer $access_token",
        'Pardot-Business-Unit-Id' => '<YOUR_BUSNIESS_UNIT_ID>',
        'Content-Type' => 'application/x-www-form-urlencoded',
    ])->post($url, $updated_data);

    return $response->json();
}

// Prospect 수정
$updated_prospect = updateProspect($access_token, 123456);
print_r($updated_prospect);
```

📌 **설명**
✔ `POST /api/prospect/version/4/do/update/id/{prospect_id}`를 호출하여 Prospect 정보를 업데이트한다.
✔ 회사 정보(`company`) 및 커스텀 필드(`custom_field__c`) 값을 수정할 수 있다.

## 3. 잠재고객 조회 (Prospect Retrieve)

특정 Prospect의 정보를 조회한다.
이 예제에서는 **Prospect ID** 또는 **이메일을 사용하여 검색**할 수 있다.

```php
function getProspectById($access_token, $prospect_id) {
    $url = "https://pi.pardot.com/api/prospect/version/4/do/read/id/$prospect_id?format=json";

    $response = Http::withHeaders([
        'Authorization' => "Bearer $access_token",
        'Pardot-Business-Unit-Id' => '0Uv2u000000000pCAA',
    ])->get($url);

    return $response->json();
}

// Prospect 조회
$prospect = getProspectById($access_token, 123456);
print_r($prospect);
```

```php
function getProspectByEmail($access_token, $email) {
    $url = "https://pi.pardot.com/api/prospect/version/4/do/read/email/$email?format=json";

    $response = Http::withHeaders([
        'Authorization' => "Bearer $access_token",
        'Pardot-Business-Unit-Id' => '0Uv2u000000000pCAA',
    ])->get($url);

    return $response->json();
}

// Prospect 이메일 조회
$prospect = getProspectByEmail($access_token, 'testuser@example.com');
print_r($prospect);
```

📌 **설명**

✔ `GET /api/prospect/version/4/do/read/id/{prospect_id}`를 호출하여 특정 Prospect 정보를 조회한다.

✔ `GET /api/prospect/version/4/do/read/email/{email}`을 사용하여 이메일로 조회할 수도 있다.

## 4. 잠재고객 삭제 (Prospect Delete)

특정 Prospect를 삭제하는 API를 호출한다.

```php
function deleteProspect($access_token, $prospect_id) {
    $url = "https://pi.pardot.com/api/prospect/version/4/do/delete/id/$prospect_id?format=json";

    $response = Http::withHeaders([
        'Authorization' => "Bearer $access_token",
        'Pardot-Business-Unit-Id' => '0Uv2u000000000pCAA',
    ])->delete($url);

    return $response->successful() ? "✅ Prospect 삭제 완료" : "❌ Prospect 삭제 실패";
}

// Prospect 삭제
$result = deleteProspect($access_token, 123456);
echo $result;
```

📌 **설명**

✔ `DELETE /api/prospect/version/4/do/delete/id/{prospect_id}`를 호출하여 특정 Prospect를 삭제한다.

## 마무리 및 정리

이 문서는 **Salesforce Pardot API v4를 활용하여 Prospect(잠재고객)를 관리하는 방법**을 설명했다. 처음 Pardot API를 사용하는 개발자 및 마케팅 담당자를 위해 **개념부터 API 호출 방법까지** 단계별로 정리했다.

### 핵심 요약

**1. Pardot과 Salesforce의 관계**

- Pardot은 **마케팅 자동화 도구**이며, Prospect 데이터를 기반으로 리드(Lead) 및 고객 전환을 관리한다.
- Prospect 데이터는 Salesforce CRM과 연동될 수 있으며, **영업팀과 마케팅팀이 데이터를 공유하여 활용**할 수 있다.

**2. Prospect(잠재고객) 개념 및 수집 방법**

- Prospect는 기업과의 다양한 상호작용(회원가입, 광고 클릭, 이메일 반응 등)을 통해 생성된다.
- Prospect 수집 방식으로 **Pardot Form Handler 및 Pardot API**를 비교하였으며, **백엔드에서 유연한 Prospect 관리를 위해 API 사용이 권장됨**을 확인했다.

**3. Pardot API 사용 준비**

- API를 호출하려면 **OAuth 2.0 인증을 통해 Access Token을 발급받아야 한다.**
- API 호출 시 **Pardot Business Unit ID를 헤더에 포함해야 한다.**
- **Bearer Token 방식**을 사용하여 Pardot API v4를 호출할 수 있다.

**4. Laravel 기반 Pardot API 구현**

| 기능                  | API 엔드포인트                                       | 구현 방식                          |
| --------------------- | ---------------------------------------------------- | ---------------------------------- |
| **Access Token 발급** | `/services/oauth2/token`                             | `POST` 요청 (OAuth 2.0)            |
| **Prospect 추가**     | `/api/prospect/version/4/do/create`                  | `POST` 요청 (Bearer Token 사용)    |
| **Prospect 수정**     | `/api/prospect/version/4/do/update/id/{prospect_id}` | `POST` 요청 (업데이트 데이터 포함) |
| **Prospect 조회**     | `/api/prospect/version/4/do/read/id/{prospect_id}`   | `GET` 요청 (ID 기반 조회)          |
| **Prospect 삭제**     | `/api/prospect/version/4/do/delete/id/{prospect_id}` | `DELETE` 요청 (ID 기반 삭제)       |

### 다음 단계 (추가 개발 가능성)

이 문서는 **Prospect 데이터의 생성, 수정, 조회, 삭제**와 같은 **기본적인 API 활용법**을 다루었다. 하지만 Pardot API를 활용하여 **보다 고도화된 기능을 구현할 수도 있다.**

**1. Prospect Score(점수) 및 Engagement History 분석**

- Prospect의 행동을 기반으로 점수를 매기고, **고객의 관심도를 정량적으로 분석**하는 기능 추가 가능
- `GET /api/prospect/version/4/do/query` API를 사용하여 특정 조건의 Prospect 목록을 조회 가능

**2. Pardot 캠페인 및 이메일 자동화 연동**

- 특정 Prospect가 생성되었을 때 자동으로 **Pardot 이메일 캠페인에 추가하는 기능**
- `POST /api/campaign/version/4/do/create` API 활용 가능

**3. Salesforce CRM과 실시간 데이터 동기화**

- Prospect 데이터가 업데이트될 때 **Salesforce Lead로 자동 변환하는 기능**
- Salesforce API를 활용하여 **Pardot Prospect와 CRM Lead 데이터 연동** 가능

**4. 웹사이트 사용자 행동 기반 Prospect 세분화**

- Pardot Tracking 기능을 활용하여 Prospect의 **웹사이트 방문, 클릭, 다운로드 이력 분석**
- Prospect의 행동 데이터 기반으로 **자동 분류 및 리드 점수 조정 가능**

### 결론

- **Pardot API를 활용하면 Prospect 데이터를 자동으로 관리할 수 있다.**
- **OAuth 2.0 인증을 통해 Access Token을 발급하고, Bearer Token 방식으로 API를 호출한다.**
- **Prospect 생성, 수정, 조회, 삭제 기능을 Laravel을 활용하여 쉽게 구현할 수 있다.**
- **향후 Prospect 데이터를 기반으로 보다 정교한 마케팅 자동화 기능을 개발할 수 있다.**

이 문서를 바탕으로 Pardot API를 활용하여 Prospect 데이터 관리 시스템을 구축할 수 있으며, 추가적인 개발을 통해 보다 강력한 마케팅 및 영업 자동화 프로세스를 구현할 수 있을 것이다.
