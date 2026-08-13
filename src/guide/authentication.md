# 인증

agents-cli는 각각의 인증 요구사항을 가진 여러 도구 상위에서 동작합니다. 이 문서에서는 어떤 인증을 왜 수행해야 하는지 명확히 이해할 수 있도록 3가지 상이한 인증 단계를 소개합니다.

---

## 1단계: 코딩 에이전트 인증

사용 중인 코딩 에이전트(Antigravity CLI, Claude Code, Codex 등)가 동작하려면 자체 인증이 필요합니다. **agents-cli는 이를 제어하지 않습니다** — 각 코딩 에이전트가 자체 자격 증명을 관리합니다.

| 코딩 에이전트 | 인증 방법 |
|-------------|---------------------|
| [Antigravity CLI](https://antigravity.google/) | Google 계정 |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | Anthropic 계정 또는 API 키 |
| [Codex](https://github.com/openai/codex) | OpenAI API 키 |

설정 방법은 사용 중인 코딩 에이전트의 문서를 참고하세요. 이는 agents-cli와 독립적입니다.

---

## 2단계: 모델 인증

사용자가 *구축 중인* 에이전트는 응답을 생성하기 위해 LLM을 호출합니다. 이를 위해 코딩 에이전트와 별개의 자격 증명이 필요합니다.

ADK는 [다양한 모델 제공업체](https://adk.dev/agents/models/)(Gemini, Claude, LiteLLM, Ollama 등)를 지원합니다. [Gemini 모델](https://adk.dev/agents/models/google-gemini/)을 위한 가장 대표적인 두 가지 설정 방식은 아래와 같습니다.

### 옵션 A: Gemini API 키 (Google AI Studio)

Google Cloud 프로젝트가 필요하지 않습니다.

1. [AI Studio](https://aistudio.google.com/apikey)에서 API 키를 생성합니다.
2. 내보내기 설정:

    ```bash
    # 스캐폴드된 Python 프로젝트에는 .env가 포함되어 있습니다. AI Studio를 사용하려면 파일 편집:
    #   GOOGLE_* 행을 주석 처리하고 GEMINI_API_KEY 주석 해제 (GOOGLE_API_KEY도 허용됨)
    GEMINI_API_KEY="your-key-here"
    ```

3. 파일을 저장합니다 — `agents-cli` 개발 명령어를 실행할 때 `.env`가 자동으로 로드됩니다.

!!! note
    API 키는 `dev`, `run`, `eval`과 같은 로컬 개발 명령어를 지원합니다. Google Cloud에 배포하려면 3단계 인증이 필요합니다.

### 옵션 B: Google Cloud (Vertex AI)

Vertex AI 모델, 엔터프라이즈 기능 및 배포를 위해 필수적입니다.

```bash
agents-cli login -i
# 또는 직접 실행: gcloud auth application-default login
```

이 명령어는 OAuth를 위한 브라우저를 열고 Application Default Credentials (ADC)를 설정합니다.

프로젝트 및 위치 설정:

```bash
gcloud config set project YOUR_PROJECT_ID
export GOOGLE_CLOUD_LOCATION="us-east1"
export GOOGLE_GENAI_USE_VERTEXAI=TRUE
```

---

## 3단계: 배포 인증

2단계의 옵션 B(Vertex AI)를 설정했다면 배포를 위한 인증이 이미 완료된 상태입니다 — 동일한 ADC 자격 증명을 공유합니다. ADC는 모델 접근 권한 외에도 다음 기능을 활성화합니다:

- `agents-cli deploy` — Agent Runtime, Cloud Run 또는 GKE에 배포
- `agents-cli infra single-project` / `agents-cli infra cicd` — Terraform을 사용해 인프라 및 CI/CD 프로비저닝

배포를 위해서는 결제가 활성화되어 있고 적절한 IAM 권한(배포 대상별로 다름)을 갖춘 Google Cloud 프로젝트가 필요합니다.

---

## 인증 상태 확인

```bash
agents-cli login --status
```

현재 어떤 인증 방식이 활성화되어 있는지와 설정된 프로젝트를 표시합니다.
