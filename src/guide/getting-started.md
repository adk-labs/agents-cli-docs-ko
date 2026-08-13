# 시작하기

**Agent Platform의 Agents CLI**는 Google Cloud 상에서 AI 에이전트를 구축, 평가 및 배포하기 위한 CLI 및 스킬 패키지입니다. 에이전트는 Google의 [Agent Development Kit (ADK)](https://google.github.io/adk-docs/)를 통해 구축되며, Agents CLI는 스캐폴딩, 평가, 배포, 관찰 가능성 등 주변의 모든 프로세스를 처리합니다.

두 가지 방식으로 작동합니다:

1. **코딩 에이전트와 함께 사용** — Antigravity CLI, Claude Code, Codex 등에 스킬을 설치합니다. 코딩 에이전트가 각 단계에서 올바른 결정을 내릴 수 있도록 스킬을 활용합니다.
2. **코딩 에이전트 없이 사용** — 터미널에서 CLI 명령어를 직접 실행합니다. 모든 명령어는 단독으로 동작합니다.

Agents CLI에는 전체 ADK 라이프사이클에 걸친 심도 있는 지식을 코딩 에이전트에 제공하는 **7가지 스킬**이 번들로 포함되어 있습니다:

| 스킬 | 코딩 에이전트가 학습하는 내용 |
|-------|-------------------------------|
| `google-agents-cli-workflow` | 개발 라이프사이클, 코드 보존, 모델 선택 |
| `google-agents-cli-adk-code` | ADK Python API — 에이전트, 툴, 오케스트레이션, 콜백 |
| `google-agents-cli-scaffold` | 프로젝트 스캐폴딩 — `create`, `enhance`, `upgrade` |
| `google-agents-cli-eval` | 평가 라이프사이클 — 데이터셋, 메트릭, 생성/채점, 비교, 분석, 최적화 |
| `google-agents-cli-deploy` | 배포 — Agent Runtime, Cloud Run, GKE, CI/CD |
| `google-agents-cli-publish` | Gemini Enterprise 등록 |
| `google-agents-cli-observability` | Cloud Trace, 로깅, 서드파티 연동 |

---

## 사전 요구사항

**필수:** [Python 3.11+](https://www.python.org/downloads/), [uv](https://docs.astral.sh/uv/getting-started/installation/), [Node.js](https://nodejs.org/en/download) (스킬 설치용)

**선택 (배포용):** [Google Cloud SDK](https://cloud.google.com/sdk/docs/install), [Terraform](https://developer.hashicorp.com/terraform/downloads)

---

## 설치

```bash
uvx google-agents-cli setup
```

이 명령어는 CLI 및 코딩 에이전트를 위한 컨텍스트 인식 스킬을 설치합니다.

??? info "대체 설치 방법"
    **pipx:** `pipx install google-agents-cli && agents-cli setup`

    **venv + pip:** `pip install google-agents-cli && agents-cli setup`

    **스킬만 설치:** `npx skills add google/agents-cli`

**플랫폼 지원:** macOS, Linux 및 Windows (WSL 2). 네이티브 Windows는 공식적으로 지원되지 않습니다.

---

## 인증

이미 `gcloud`로 인증된 경우 그대로 작동합니다 — Agents CLI가 사용자의 Application Default Credentials (ADC)를 자동으로 불러옵니다.

그렇지 않은 경우 가장 빠른 방법은 [AI Studio](https://aistudio.google.com/apikey)에서 Gemini API 키를 받는 것입니다:

```bash
export GEMINI_API_KEY="your-key-here"
```

자세한 내용은 [인증](authentication.md) 문서에서 확인하세요.

---

## 코딩 에이전트로 개발 시작하기

=== "Antigravity CLI"

    1. **Antigravity CLI 실행**

        IDE 또는 터미널에서 Antigravity를 실행합니다.

    2. **스킬 설치 확인**

        현재 환경에서 Agents CLI 스킬을 사용할 수 있는지 확인합니다.

    3. **에이전트 구축 요청**

        ```
        Build a support agent that answers questions from our docs
        ```

        Antigravity가 설치된 스킬을 사용하여 에이전트를 스캐폴딩하고 구축하며 평가합니다.

=== "Claude Code"

    1. **Claude Code 실행**

        ```bash
        claude
        ```

    2. **스킬 설치 확인**

        ```
        /skills
        ```

        `google-agents-cli-workflow` 및 기타 Agents CLI 스킬 목록이 표시되어야 합니다.

    3. **에이전트 구축 요청**

        ```
        Build a support agent that answers questions from our docs
        ```

        Claude가 설치된 스킬을 사용하여 에이전트를 스캐폴딩하고 구축하며 평가합니다.

=== "Codex"

    1. **Codex 실행**

        ```bash
        codex
        ```

    2. **스킬 설치 확인**

        현재 환경에서 Agents CLI 스킬을 사용할 수 있는지 확인합니다.

    3. **에이전트 구축 요청**

        ```
        Build a support agent that answers questions from our docs
        ```

        Codex가 설치된 스킬을 사용하여 에이전트를 스캐폴딩하고 구축하며 평가합니다.

=== "기타 모든 에이전트"

    Agents CLI는 [스킬](https://agentskills.io/what-are-skills)을 지원하는 모든 코딩 에이전트와 함께 사용할 수 있습니다.

    1. **스킬 설치**

        ```bash
        uvx google-agents-cli setup
        ```

    2. **스킬 인식 여부 확인**

        사용 중인 에이전트가 `google-agents-cli-workflow` 및 기타 Agents CLI 스킬을 인식할 수 있는지 확인합니다. 대부분의 에이전트는 `/skills` 명령어나 설정 패널을 통해 이를 제공합니다.

    3. **에이전트 구축 요청**

        ```
        Build a support agent that answers questions from our docs
        ```

        스킬이 설치되어 인식되면 에이전트가 이를 자동으로 활용합니다.

---

## 직접 명령어를 입력하여 진행하시겠습니까?

터미널에서 전체 워크플로우를 직접 실행할 수 있으며, 코딩 에이전트가 필요하지 않습니다.

```bash
# 최소한의 에이전트 프로젝트 생성
agents-cli create my-agent --prototype --yes

# 의존성 설치 및 개발 플레이그라운드 시작
cd my-agent
agents-cli install
agents-cli playground
```

이 명령어는 핫 리로드를 지원하는 ADK 웹 플레이그라운드를 `http://localhost:8080`에서 시작합니다.

전체 단계별 가이드는 [수동 워크플로우 튜토리얼](hands-on-tutorial.md)을 참고하세요.

---

## 데모

<div align="center">
  <iframe width="100%" height="450" src="https://www.youtube.com/embed/ECYKo70pPNc" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

---

## 다음 단계

- [튜토리얼: 첫 번째 에이전트 구축하기](quickstart-tutorial.md) — 코딩 에이전트로 구축, 평가 및 배포하기
- [튜토리얼: 수동 워크플로우](hands-on-tutorial.md) — 모든 명령어를 직접 입력하기
- [유스케이스](use-cases.md) — 실제 에이전트 패턴 아이디어 얻기
- [프로젝트 구조](project-structure.md) — 생성된 각 파일의 역할 이해하기
- [에이전트 템플릿](templates.md) — `adk` 템플릿 및 RAG 학습 레시피
- [개발 가이드](development.md) — 전체 개발 워크플로우
- [CLI 참조 문서](../cli/index.md) — 모든 명령어 및 플래그

---

!!! tip "Agent Starter Pack에서 전환하시나요?"
    [마이그레이션 가이드](../reference/from-agent-starter-pack.md)를 참고하세요.

!!! note "구축한 내용을 공유해 주세요"
    Agents CLI를 사용하여 흥미로운 프로젝트를 만드셨나요? 여러분의 이야기를 듣고 싶습니다! [agents-cli@google.com](mailto:agents-cli@google.com)으로 프로젝트를 공유해 주세요.
