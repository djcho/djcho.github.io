---
title: "[AWS] Bedrock 자동 코드 리뷰 시스템: 구조화된 AI 통합 사례"
categories:
  - AWS
tag:
  - [AWS, AI, Bedrock]
toc: true
toc_sticky: true
published: true
date: 2025-04-01 00:00:00
last_modified_at: 2025-04-01 00:00:00
---

# Amazon Bedrock 자동 코드 리뷰 시스템: 구조화된 AI 통합 사례

## 1. Amazon Bedrock 개요

Amazon Bedrock은 AWS에서 제공하는 생성형 AI 서비스로, 대규모 언어 모델(LLM)을 API 형태로 즉시 활용할 수 있게 해준다. 사용자는 모델 학습이나 인프라 구성 없이도, Anthropic Claude, AI21, Cohere, Stability 등 다양한 파운데이션 모델을 선택하여 비즈니스에 필요한 자연어 기반 기능을 손쉽게 구현할 수 있다.

Bedrock은 특히 서버리스 기반으로 동작하기 때문에, 빠르게 프로토타입을 만들거나 실제 운영환경에 통합하기에 매우 용이하다. 코드 리뷰, 문서 요약, 고객 응대 자동화, 보안 감사 등 다양한 분야에서 활용 가능하다.

## 2. 프로젝트 개요: PR Review Bot

이 프로젝트는 AWS Solutions Architect 김범준 님이 설계한 예제 아키텍처를 기반으로 하며, 실제 업무 환경에서 Amazon Bedrock을 활용한 PR(Pull Request) 자동 리뷰 프로세스의 구현 가능성과 확장성을 보여준다.

본 문서에서는 Bedrock의 실질적인 활용 예제로 "PR Review Bot" 프로젝트를 소개한다. 이 프로젝트는 GitHub, GitLab, Bitbucket 등의 PR 이벤트를 트리거로 하여, PR에 포함된 코드 변경사항을 분석하고 Amazon Bedrock Claude 모델을 사용하여 자동 코드 리뷰를 생성하는 시스템이다.

## 3. 전체 아키텍처 및 워크플로우

이 시스템은 Slack 메시지 알림과 GitHub Pull Request 코멘트를 자동 생성하는 기능도 포함한다. Bedrock 응답 결과를 시각적으로 요약하여 다양한 채널에 통합하는 방식이다.

### 3.1 아키텍처 개요

아래 다이어그램은 이 시스템이 AWS 인프라 및 Bedrock을 기반으로 어떻게 구성되는지를 보여준다.

<p align="center">
  <img src="https://github.com/djcho/amazon-bedrock-pr-review-bot/blob/main/docs/img/architecture.png?raw=true" alt="architecture" />
  <br />
  <em>[그림 1] PR Review 시스템 아키텍처 다이어그램</em>
</p>

**주요 흐름 설명:**

1. 개발자가 PR을 생성하면 Git 플랫폼이 이벤트를 발생시킨다.
2. 이 이벤트는 API Gateway를 통해 수신된다.
3. 이벤트는 Step Functions를 트리거하여 리뷰 프로세스를 시작한다.
4. PR의 변경사항이 분석되어 Amazon Bedrock으로 전송된다.
5. Bedrock은 다양한 리전에 있는 모델을 통해 추론을 수행한다 (Cross-region inference).
6. 결과는 다시 Step Functions에서 집계되고 Slack이나 GitHub로 전달된다.
7. 최종적으로 리뷰 결과는 개발자에게 도달한다.

### 3.2 워크플로우 : AWS Step Functions

이 프로젝트는 AWS Step Functions를 활용하여 PR 리뷰 자동화를 구성한다. 각 주요 처리 단계는 Lambda 함수로 구현되어 있으며, Step Functions는 이 Lambda들을 순차적, 병렬적으로 호출하여 전체 리뷰 흐름을 관리한다. 상세한 Lambda 연동 방식은 후반부에서 별도로 다룬다.

## 4. 주요 기능

이 시스템은 GitHub Pull Request에 대한 코드 리뷰를 자동화하여, 리뷰 품질을 높이고 반복 작업을 줄이는 것을 목표로 한다. 주요 기능은 다음과 같다:

### 4.1 Bedrock 기반 코드 리뷰 자동화

- PR 생성 시 자동 트리거되어 리뷰 프로세스를 시작
- 변경된 파일을 청크 단위로 분할하여 병렬 처리
- 각 청크는 AWS Lambda에서 Amazon Bedrock의 Claude 3.5 Sonnet 모델을 호출해 리뷰 분석 수행
- 분석 항목: 기능 변경 요약, 보안/성능/로직 문제 식별, 위험도 분류
- 전체 리뷰 흐름은 Step Functions 기반으로 안정적으로 관리됨

### 4.2 GitHub PR 코멘트 자동 보고서

<p align="center">
  <img src="https://github.com/djcho/amazon-bedrock-pr-review-bot/blob/main/docs/img/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202025-04-02%2015.37.14.png?raw=true" alt="step1" />
  <img src="https://github.com/djcho/amazon-bedrock-pr-review-bot/blob/main/docs/img/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202025-04-02%2015.37.29.png?raw=true" alt="step2" />
  <br />
  <em>[그림 2] PR Review - PR 코멘트 보고서</em>
</p>

- 분석된 리뷰 결과는 자동으로 GitHub PR 코멘트에 요약 리포트 형태로 등록
- 포함 항목: 변경 요약, 주요 이슈, 파일별 상세 목록

### 4.3 Slack 알림 자동 전송

- 리뷰 결과는 Slack으로도 자동 전송되어 팀 단위 공유 가능
- 주요 이슈 요약, 이슈 수, PR 링크 등을 포함한 카드형 메시지 제공

<p align="center">
  <img src="https://github.com/djcho/amazon-bedrock-pr-review-bot/blob/main/docs/img/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202025-04-02%2015.33.35.png?raw=true" alt="slack-message" />
  <br />
  <em>[그림 4] Slack으로 전송되는 PR Review 결과 메시지 예시</em>
</p>

이러한 기능들을 통해 리뷰 속도와 일관성을 향상시키고, 협업 흐름에 자연스럽게 통합되는 리뷰 자동화 경험을 제공한다.

## 5. Bedrock 프롬프트 설계 및 호출 방식

### 5.1 단순 질문이 아닌, 역할 기반 협업 요청

많은 개발자들이 AI에 "이 코드 어때?"라고 묻는다. 하지만 이 프로젝트는 Claude를 하나의 전문가로 지정하고, 그에게 **역할, 입력, 기대 형식**을 모두 제시한다. 즉, 단순 질의가 아닌 계약 기반 협업 요청이다.

> 📌 Claude에게 요청할 때는 역할 부여 + 맥락 제공 + 응답 포맷을 함께 명시해야 AI를 시스템에 통합할 수 있다.

### 5.2 프롬프트 생성 예시

#### 5.2.1 코드 리뷰 요청

##### 5.2.1.1 프롬프트

````python
prompt = f"""당신은 전문 코드 리뷰어 입니다. Pull Request에서 변경된 코드를 리뷰해주세요.

File Status: {'Primary File' if is_primary else 'Reference File'}
File Path: {file_path}

Code Changes:
```{language}
{content}
```

Detected Patterns:
...
해당 파일의 주요 변경사항을 다음 카테고리별로 분석하고, JSON 형식으로 응답해주세요.

응답 형식:
{
    "summary": {
        "functional_changes": [
            "새로운 기능 A 추가",
            "기존 기능 B 수정"
        ],
        "architectural_changes": [
            "디자인 패턴 X 도입",
            "모듈 구조 Y 변경"
        ],
        "technical_improvements": [
            "성능 최적화 적용",
            "코드 품질 개선"
        ]
    },
    "severity": "CRITICAL/MAJOR/MINOR/NORMAL",
    "review_points": [
        {
            "category": "security/performance/style/logic",
            "severity": "CRITICAL",
            "line_number": "42",
            "description": "SQL 인젝션 취약점 발견",
            "suggestion": "파라미터화된 쿼리 사용 권장"
        }
    ]
}

모든 설명은 한글로 작성하되, 파일명, 메서드명, 클래스명은 원본 그대로 사용해주세요.
변경사항은 실제 코드에서 발견된 내용만 포함해주세요.
"""
````

##### 5.2.1.2 리뷰 요청 Bedrock 호출

```python
body = json.dumps({
  "anthropic_version": "bedrock-2023-05-31",
  "max_tokens": self.config.get('max_tokens', 4096),
  "temperature": self.config.get('temperature', 0.7),
  "top_p": self.config.get('top_p', 0.9),
  "system": "당신은 senior code reviewer입니다. 피드백을 구체적이고 실행 가능한 내용으로 작성하는 데 집중하세요. 요청된 JSON 형식으로 응답을 반환하세요.",
  "messages": [
    {
      "role": "user",
      "content": prompt
    }
  ]
})

response = self.bedrock.invoke_model(
  modelId='apac.anthropic.claude-3-5-sonnet-20241022-v2:0',
  contentType='application/json',
  accept='application/json',
  body=prompt_body.encode()
)
```

#### 5.2.2 리뷰 결과 요약 요청

`aggregate-results` Lambda는 병렬 청크 처리 결과를 통합하고, Claude를 다시 활용하여 변경사항 요약을 생성한다.

##### 5.2.2.1 프롬프트

```python
prompt = """다음 변경사항들을 각 카테고리별로 5문장 이내로 요약해주세요.
원본 변경사항:

🔄 Functional Changes:
"""
for change in changes.get('functional_changes', []):
    prompt += f"- {change}\n"

prompt += "\n🏗 Architectural Changes:\n"
for change in changes.get('architectural_changes', []):
    prompt += f"- {change}\n"

prompt += "\n🔧 Technical Improvements:\n"
for change in changes.get('technical_improvements', []):
    prompt += f"- {change}\n"

prompt += """
위 변경사항들을 다음 형식으로 요약해주세요:

    {
        "summary": {
            "functional_changes": "기능적 변경사항 요약",
            "architectural_changes": "아키텍처 변경사항 요약",
            "technical_improvements": "기술적 개선사항 요약"
        }
    }

    각 요약은 한글로 작성하고, 전문 용어나 고유명사는 원문 그대로 사용해주세요."""
```

##### 5.2.1.2 리뷰 요청 Bedrock 호출

```python
bedrock = boto3.client('bedrock-runtime')
            prompt = self._prepare_summary_prompt(changes)

            body = json.dumps({
                "anthropic_version": "bedrock-2023-05-31",
                "max_tokens": 1000,
                "temperature": 0.7,
                "top_p": 0.9,
                "system": "5문장 이내로 간결하게 요약하는 전문 리뷰어입니다.",
                "messages": [
                    {
                        "role": "user",
                        "content": prompt
                    }
                ]
            })

            response = bedrock.invoke_model(
                modelId=self.config['model'],
                contentType='application/json',
                accept='application/json',
                body=body.encode()
            )
```

이 요약은 Slack 메시지, PR 코멘트, Markdown Report에 모두 사용된다.

## 6. Step Functions와 Lambda 연동 방식 (CDK 기반)

`lib/constructs/step-functions.ts` 파일에서 Step Functions 상태 머신은 다음과 같이 Lambda 함수들을 순차적 혹은 병렬로 실행하도록 구성된다.

<p align="center">
  <img 
    src="https://github.com/djcho/amazon-bedrock-pr-review-bot/blob/main/docs/img/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA%202025-04-02%2014.48.00.png?raw=true" alt="stepfunction-flow"/>
  <br />
  <em>[그림 4] PR Review 시스템 StepFunction 흐름도</em>
</p>

#### 6.1 단일 Lambda 실행 예시

```ts
const splitPr = new tasks.LambdaInvoke(this, "SplitPRIntoChunks", {
  lambdaFunction: props.functions.splitPr,
  inputPath: "$.processingResult.body.data",
  payloadResponseOnly: true,
});
```

#### 6.2 병렬 Lambda 실행 (Map 상태)

```ts
const processChunks = new stepfunctions.Map(this, "ProcessChunks", {
  itemsPath: "$.chunks",
  maxConcurrency: 3,
}).itemProcessor(
  new tasks.LambdaInvoke(this, "ProcessChunk", {
    lambdaFunction: props.functions.processChunk,
    payloadResponseOnly: true,
  })
);
```

이러한 구조는 Step Function을 통해 **Bedrock 호출 작업을 Lambda 단위로 안전하고 유연하게 조정**할 수 있게 한다. 각 Lambda는 자신에게 전달된 payload를 기반으로 PR의 일부 파일을 분석하고, 개별적인 Claude 호출을 수행한다.

## 7. 이런 것도 가능하지 않을까?

Bedrock은 코드 리뷰 외에도 다양한 형태로 활용될 수 있다. 예를 들어:

- 조직도 정규화 유틸 구현 (비정형 조직 데이터 → 내부 포맷 자동 변환)
- 테스트 자동 생성 보조 (기능 흐름 기반 테스트 시나리오 초안 작성)
- 문서화 및 사내 지식 관리 (기술 위키 기반 Q&A 챗봇, API 문서 자동화)
- 로그 기반 장애 요약 (운영 로그에서 이슈 요약 및 영향도 추론)

이처럼 "의미를 이해하고 문장으로 바꿀 수 있는 일"이라면, 다양한 업무에서 Bedrock이 실질적인 보조 역할을 할 수 있다.

## 8. 결론: 생성형 AI와 개발 자동화

본 문서는 Amazon Bedrock을 기반으로 한 PR 자동 리뷰 시스템의 구조, 프롬프트 설계, Step Functions 기반의 오케스트레이션 방식까지 상세히 소개하였다. 이 시스템은 단순한 AI 연동을 넘어서, **“역할 기반 프롬프트 설계 + 병렬 처리를 위한 인프라 아키텍처 + 실무 연계 자동화”** 라는 관점에서 생성형 AI를 실제 업무에 녹여낸 대표 사례라 할 수 있다.

이를 통해 얻을 수 있는 핵심 인사이트는 다음과 같다:

- **AI는 도구가 아니라 협업자다:** 명확한 역할과 기대치를 프롬프트에 반영할 때, 일관되고 신뢰할 수 있는 응답을 얻을 수 있다.
- **자동화는 구조로 시작한다:** Step Functions를 통한 작업 흐름 분할과 병렬화는 대규모 PR도 안정적으로 처리하게 해준다.

Amazon Bedrock은 단순한 LLM API가 아닌, **생성형 AI를 안전하고 확장 가능하게 시스템에 통합할 수 있는 플랫폼**이다. 앞으로도 본 프로젝트에서 소개한 구조적 접근 방식을 응용하면, 테스트 자동화, 문서화, 로그 분석 등 다양한 영역에서 실질적인 생산성 향상을 기대할 수 있다.

---

## 🔗 참고 및 코드 저장소

이 문서에서 소개한 PR 자동 리뷰 시스템의 전체 코드는 아래 GitHub 저장소에서 확인할 수 있다:

[https://github.com/djcho/amazon-bedrock-pr-review-bot](https://github.com/djcho/amazon-bedrock-pr-review-bot)
