# 프로젝트 구조

*생성된 에이전트 프로젝트의 구조를 이해하려는 개발자를 위한 가이드입니다.*

`agents-cli create my-agent --prototype --yes` 명령을 실행하면 바로 실행할 수 있는 프로젝트가 생성됩니다. 이 페이지에서는 각 파일의 역할을 설명합니다.

---

## 디렉터리 구조

```
my-agent/
├── app/                          # 에이전트 코드
│   ├── __init__.py               # 앱 등록 (export `app`)
│   ├── agent.py                  # 에이전트 정의 — 지침(instructions), 모델, 도구
│   ├── fast_api_app.py           # FastAPI 서버 — 텔레메트리 설정, 피드백/A2A 라우트
│   └── app_utils/                # 유틸리티
│       ├── services.py           # 공유 세션 및 아티팩트 서비스
│       ├── a2a.py                # A2A 라우트 연결
│       └── typing.py             # 요청/응답 Pydantic 모델
│
├── tests/
│   ├── eval/                     # 평가 테스트 케이스
│   │   ├── datasets/
│   │   │   └── basic-dataset.json    # 기본 평가 케이스
│   │   └── eval_config.yaml          # 평가 메트릭 설정
│   ├── integration/
│   │   └── test_agent.py         # 통합 테스트 (에이전트를 엔드투엔드로 실행)
│   └── unit/
│       └── test_dummy.py         # 단위 테스트용 플레이스홀더
│
├── pyproject.toml                # 프로젝트 설정 및 의존성
├── agents-cli-manifest.yaml      # agents-cli 설정
├── GEMINI.md                     # 코딩 에이전트를 위한 안내 파일
├── Makefile                      # 단축 명령 (make dev, make eval 등)
├── .env                          # 환경 변수 (프로젝트 ID, 위치)
└── uv.lock                       # 의존성 버전 잠금 파일
```

---

## 주요 파일

### `app/agent.py`

에이전트가 정의되는 핵심 파일입니다. 기본 템플릿은 다음과 같습니다:

```python title="app/agent.py"
from google.adk.agents import Agent
from google.adk.apps import App
from google.adk.models import Gemini
from google.genai import types

MODEL = "gemini-3.6-flash"


def get_weather(query: str) -> str:
    """Simulates a web search. Use it get information on weather."""
    if "sf" in query.lower() or "san francisco" in query.lower():
        return "It's 60 degrees and foggy."
    return "It's 90 degrees and sunny."


def get_current_time(query: str) -> str:
    """Simulates getting the current time for a city."""
    # ... implementation
    return f"The current time is ..."


root_agent = Agent(
    name="root_agent",
    model=Gemini(
        model=MODEL,
        retry_options=types.HttpRetryOptions(attempts=3),
    ),
    instruction="You are a helpful AI assistant.",
    tools=[get_weather, get_current_time],
)

app = App(
    root_agent=root_agent,
    name="app",  # Must match the agent directory name
)
```

네 가지 핵심 요소:

1. **도구 함수 (Tool functions)** — 독스트링(docstring)이 포함된 일반 Python 함수입니다. 독스트링은 LLM에 해당 도구를 언제 사용해야 하는지 알려줍니다.
2. **`Agent`** — 모델, 지침(시스템 프롬프트), 도구를 결합합니다.
3. **`App`** — 서빙을 위해 에이전트를 래핑합니다. `name`은 에이전트 디렉터리 이름(`app`)과 일치해야 합니다.
4. **모델 (Model)** — 기본값은 `gemini-3.6-flash`입니다. `agent.py` 상단의 `MODEL` 상수를 통해 변경할 수 있습니다.

### `pyproject.toml`

Python 프로젝트 메타데이터 및 의존성을 포함합니다:

```toml title="pyproject.toml"
[project]
name = "my-agent"
version = "0.0.1"
requires-python = ">=3.11"
dependencies = [
    "google-adk[gcp]>=2.0.0,<3.0.0",
    # ... 기타 의존성
]
```

### `agents-cli-manifest.yaml`

agents-cli 프로젝트 메타데이터 및 설정을 포함합니다:

```yaml title="agents-cli-manifest.yaml"
name: my-agent
agent_directory: app
create_params:
  deployment_target: none
  session_type: in_memory
```

- **`agent_directory`** — `agents-cli` 명령에 에이전트 코드가 위치한 경로를 알립니다.
- **`create_params`** — 프로젝트가 어떻게 생성되었는지 기록합니다. `agents-cli scaffold upgrade` 명령이 설정을 보존하는 데 사용됩니다.

### `tests/eval/datasets/basic-dataset.json`

기본 평가 케이스입니다. 각 케이스는 사용자 메시지와 이를 실행하기 위한 세션 컨텍스트를 정의합니다. 전체 스키마는 [평가 가이드](evaluation.md)를 참고하세요.

### `GEMINI.md`

코딩 에이전트(Antigravity CLI, Claude Code 등)가 자동으로 읽는 안내 파일입니다. ADK 패턴, 코딩 컨벤션, 워크플로 가이드 등 프로젝트 전용 지침이 포함되어 있습니다. 코딩 에이전트가 프로젝트와 작동하는 방식을 맞춤 설정하려는 경우가 아니라면 이 파일을 직접 읽거나 수정할 필요는 없습니다.

### `.env`

로컬 개발을 위한 환경 변수입니다:

```bash title=".env"
GOOGLE_CLOUD_PROJECT=your-project-id
GOOGLE_CLOUD_LOCATION=us-east1
```

런타임 시 에이전트가 이 변수들을 읽어옵니다. 본인의 Google Cloud 프로젝트에 맞게 설정하거나, Gemini API 키를 사용하는 경우 비워 두세요.

---

## 배포 인프라 포함 시

배포 타깃을 지정하여 프로젝트를 생성하거나(`agents-cli scaffold enhance`로 추가), 추가 디렉터리가 생성됩니다:

```
my-agent/
├── deployment/
│   └── terraform/
│       ├── dev/              # 개발(Dev) 환경 Terraform
│       ├── staging/          # 스테이징(Staging) Terraform
│       ├── prod/             # 운영(Production) Terraform
│       └── variables.tf      # 공유 변수
│
├── .github/                  # GitHub Actions CI/CD (선택한 경우)
│   └── workflows/
│       ├── pr_checks.yaml
│       ├── staging.yaml
│       └── deploy-to-prod.yaml
│
└── .cloudbuild/              # Cloud Build CI/CD (선택한 경우)
    ├── pr_checks.yaml
    ├── staging.yaml
    └── deploy-to-prod.yaml
```

### 나중에 인프라 추가하기

프로토타입으로 시작한 후 필요할 때 인프라를 추가하세요:

```bash
# Cloud Run 배포 추가
agents-cli scaffold enhance --deployment-target cloud_run

# 적용하지 않고 변경 사항 미리보기
agents-cli scaffold enhance --deployment-target cloud_run --dry-run
```
