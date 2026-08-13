# 튜토리얼: 수동 워크플로우

*코딩 에이전트 없이 모든 명령어를 직접 입력하는 것을 선호하는 개발자를 위한 가이드입니다.*

이 튜토리얼에서는 코딩 에이전트 없이 명령어를 직접 입력하여 ADK 에이전트를 구축, 테스트 및 평가하는 과정을 안내합니다.

!!! tip
    코딩 에이전트에게 작업을 맡기는 방식을 선호하시나요? [튜토리얼: 첫 번째 에이전트 구축하기](quickstart-tutorial.md)를 참고하세요.

---

## 구축할 에이전트

기본 에이전트 템플릿(날씨를 조회하고 시간을 알려주는 어시스턴트)으로 시작하여 새로운 페르소나와 커스텀 툴을 추가해 보겠습니다.

## 사전 요구사항

- Python 3.11+ 및 [uv](https://docs.astral.sh/uv/getting-started/installation/) 설치
- 인증 설정 — [Gemini API 키](authentication.md#option-a-gemini-api-key-google-ai-studio) 또는 [Google Cloud 자격 증명](authentication.md#option-b-google-cloud-vertex-ai)

---

## 1. 프로젝트 생성

```bash
agents-cli create my-first-agent --prototype --yes
cd my-first-agent
agents-cli install
```

- `--prototype` 옵션은 Terraform 및 CI/CD를 건너뛰고 에이전트 코드, 테스트, 평가 세트만 생성합니다.
- `--yes` 옵션은 기본값(ADK 템플릿, 인메모리 세션 저장소)을 자동 승인합니다.
- `agents-cli install`은 `uv sync`를 통해 모든 Python 의존성을 설치합니다.

---

## 2. 프로젝트 구조 살펴보기

프로젝트 구성 항목:

```
my-first-agent/
├── app/
│   ├── __init__.py       # 앱 등록
│   ├── agent.py          # 에이전트 정의 — 메인 로직이 위치하는 곳
│   └── app_utils/        # 텔레메트리 및 유틸리티 코드
├── tests/
│   ├── eval/
│   │   ├── datasets/
│   │   │   └── basic-dataset.json   # 평가용 테스트 케이스
│   │   └── eval_config.yaml         # 메트릭 구성
│   ├── integration/
│   │   └── test_agent.py
│   └── unit/
│       └── test_dummy.py
├── pyproject.toml        # 프로젝트 구성 및 의존성
└── GEMINI.md             # 코딩 에이전트용 안내 파일
```

가장 중요한 파일은 `app/agent.py`입니다. 파일에는 2개의 툴 함수(`get_weather`, `get_current_time`)와 에이전트 정의가 포함되어 있습니다:

```python title="app/agent.py"
root_agent = Agent(
    name="root_agent",
    model=Gemini(model="gemini-3.6-flash"),
    instruction="You are a helpful AI assistant designed to provide accurate and useful information.",
    tools=[get_weather, get_current_time],
)
```

모든 파일에 대한 상세 설명은 [프로젝트 구조](project-structure.md)를 참고하세요.

---

## 3. 로컬에서 에이전트 실행

ADK 웹 플레이그라운드를 시작합니다:

```bash
agents-cli playground
```

브라우저에서 [http://localhost:8080](http://localhost:8080)을 엽니다. 채팅 인터페이스가 표시됩니다. 다음과 같이 입력해 보세요:

> What's the weather in San Francisco?

에이전트는 `get_weather` 툴을 호출하고 *"It's 60 degrees and foggy in San Francisco."*와 같이 응답합니다.

!!! tip
    플레이그라운드는 핫 리로드를 지원합니다 — `app/agent.py` 변경 사항을 저장하면 즉시 반영됩니다.

---

## 4. 터미널에서 테스트

브라우저 없이도 테스트할 수 있습니다:

```bash
agents-cli run "What's the weather in San Francisco?"
```

이 명령어는 단일 프롬프트를 전송하고 에이전트의 응답을 출력합니다.

---

## 5. 에이전트 커스텀하기

에이전트에 성격(personality)을 부여해 봅시다. `app/agent.py`를 열고 지시사항(instruction)을 변경합니다:

```python title="app/agent.py"
root_agent = Agent(
    name="root_agent",
    model=Gemini(
        model="gemini-3.6-flash",
        retry_options=types.HttpRetryOptions(attempts=3),
    ),
    instruction="""You are a cheerful weather reporter who speaks in short, 
    punchy sentences. Always include a fun weather-related pun in your responses. 
    When asked about time, relate it back to weather somehow.""",
    tools=[get_weather, get_current_time],
)
```

파일을 저장합니다. 플레이그라운드가 실행 중이라면 자동으로 재로드됩니다. 동일한 질문을 다시 던져보면 다른 톤으로 응답하는 것을 확인할 수 있습니다.

---

## 6. 커스텀 툴 추가

단어 수를 세는 툴을 추가해 봅시다. `app/agent.py`의 `root_agent` 정의 위에 다음 함수를 추가합니다:

```python title="app/agent.py"
def count_words(text: str) -> str:
    """Count the number of words in the given text.

    Args:
        text: The text to count words in.

    Returns:
        A string with the word count.
    """
    word_count = len(text.split())
    return f"The text contains {word_count} words."
```

그런 다음 에이전트의 `tools` 목록에 등록합니다:

```python
    tools=[get_weather, get_current_time, count_words],
```

테스트 실행:

```bash
agents-cli run "How many words are in: The quick brown fox jumps over the lazy dog"
```

에이전트가 `count_words`를 호출하고 단어 수를 응답합니다.

!!! tip
    ADK 툴은 일반 Python 함수입니다. 함수의 **docstring**이 LLM에 보이는 설명이 되므로 명확하게 작성해 주세요 — 모델이 언제 어떻게 툴을 사용할지 판단하는 기준이 됩니다.

툴 추가에 대한 자세한 내용은 [ADK Tools 문서](https://google.github.io/adk-docs/tools/)를 참고하세요.

---

## 7. 평가 실행

평가는 에이전트가 올바르게 동작하는지 검증합니다. 프로젝트에는 `tests/eval/datasets/basic-dataset.json`에 기본 데이터셋이 포함되어 있습니다:

```json title="tests/eval/datasets/basic-dataset.json"
{
  "eval_cases": [
    {
      "eval_case_id": "greeting",
      "prompt": {
        "role": "user",
        "parts": [{"text": "Hello, what can you help me with?"}]
      }
    }
  ]
}
```

각 평가 케이스는 사용자 메시지를 정의합니다. 평가 시스템은 이 메시지를 에이전트에 보내고 `eval_config.yaml`에 지정된 메트릭을 사용하여 응답을 채점합니다.

평가 실행:

```bash
agents-cli eval generate
agents-cli eval grade
```

출력 결과에는 구성된 메트릭 대비 각 평가 케이스의 점수가 표시됩니다.

테스트 케이스 작성, 메트릭 추가, 평가-수정 루프 및 평가 전체 기능(`eval dataset synthesize`, `eval compare`, `eval analyze`, `eval metric list`, `eval optimize`)에 대한 내용은 [평가 가이드](evaluation.md)를 참고하세요.

---

## 8. Google Cloud에 배포

에이전트가 평가를 통과하면 배포합니다. 먼저 배포 대상을 추가합니다(프로토타입 프로젝트에는 포함되어 있지 않음):

```bash
agents-cli scaffold enhance --deployment-target cloud_run
```

Google Cloud 프로젝트를 설정하고 배포합니다:

```bash
gcloud config set project YOUR_DEV_PROJECT_ID
agents-cli deploy
```

정상 실행 여부 확인:

```bash
agents-cli deploy --status
```

!!! note
    배포 시에는 [Google Cloud 자격 증명](authentication.md#option-b-google-cloud-vertex-ai)이 필요합니다. Agent Runtime, GKE 및 기타 옵션은 [배포 가이드](deployment.md)를 참고하세요.

---

## 9. 에이전트 관찰

Cloud Trace는 기본적으로 활성화되어 있어 별도 설정이 필요하지 않습니다. 에이전트에 몇 가지 요청을 보낸 후 Google Cloud Console에서 [Trace 탐색기](https://console.cloud.google.com/traces)를 엽니다. 각 LLM 호출 및 툴 실행에 대한 지연 시간 세부 스팬(span)을 확인할 수 있습니다.

### 콘텐츠 로그 확인

프로덕션에서 에이전트가 처리하는 실제 프롬프트와 응답을 검사하려면 관찰 가능성 인프라를 프로비저닝하세요:

```bash
agents-cli infra single-project --project YOUR_DEV_PROJECT_ID
```

이 명령어는 Terraform을 실행하여 전용 서비스 계정, GCS 버킷 및 BigQuery 데이터셋을 생성하고 배포된 서비스가 이를 사용하도록 업데이트합니다.

검증 단계, 전체 콘텐츠 캡처 및 BigQuery Agent Analytics는 [관찰 가능성 가이드](observability/index.md)를 참고하세요.

---

## 진행한 작업 요약

| 단계 | 일어난 작업 |
|------|--------------|
| `agents-cli create --prototype --yes` | 에이전트 코드, 테스트 및 평가 세트가 포함된 프로젝트 생성 |
| `agents-cli playground` | 대화형 테스트를 위한 ADK 플레이그라운드 시작 |
| `agents-cli run "..."` | 터미널에서 에이전트 테스트 |
| `agent.py` 수정 | 페르소나 커스텀 및 툴 추가 |
| `agents-cli eval generate` 및 `agents-cli eval grade` | 구조화된 평가로 에이전트 동작 검증 |
| `agents-cli deploy` | Google Cloud에 에이전트 배포 |
| Trace 탐색기 + 콘텐츠 로그 | 트레이싱 검증 및 프롬프트-응답 로깅 설정 |

---

## 다음 단계

- [ADK 커스텀 툴](https://google.github.io/adk-docs/tools/) — 더 많은 툴 패턴 및 고급 활용법
- [평가 가이드](evaluation.md) — 더 나은 평가 작성법 및 메트릭 이해
- [배포 가이드](deployment.md) — Agent Runtime, GKE, 보안 비밀(secrets) 및 CI/CD
- [관찰 가능성 가이드](observability/index.md) — BigQuery Agent Analytics 및 서드파티 통합
