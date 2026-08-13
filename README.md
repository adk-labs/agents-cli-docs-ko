<div align="center">
  <img src="https://raw.githubusercontent.com/google/agents-cli/main/docs/src/assets/logo_sm.png" alt="agents-cli logo" width="120" />
  <h1><code>agents-cli</code></h1>
  <p>Gemini Enterprise Agent Platform에서 에이전트를 구축하기 위한 CLI 및 스킬입니다.</p>

  <p>
    <a href="#시작하기">시작하기</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="#agent-skills-에이전트-스킬">스킬</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="#cli-명령어">명령어</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://pypi.org/project/google-agents-cli/">PyPI</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://github.com/google/agents-cli/issues">이슈</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://google.github.io/agents-cli/">문서</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://github.com/google/agents-cli/blob/main/RELEASE_NOTES.md">릴리스 노트</a> &nbsp;&nbsp;|&nbsp;&nbsp;
    <a href="https://github.com/google/agents-cli">Star 누르기</a>
  </p>
</div>

---

자주 사용하는 코딩 어시스턴트를 Google Cloud 상에서 에이전트를 구축하고 배포하는 전문가로 전환해 보세요.

**Agent Platform의 Agents CLI** (`agents-cli`)는 코딩 에이전트에 엔터프라이즈급 에이전트를 구축, 확장, 거버넌스 및 최적화할 수 있는 스킬과 명령어를 제공하므로, 모든 CLI와 서비스를 직접 배울 필요가 없습니다.

**다음 도구들과 원활하게 연동됩니다:**
[Antigravity CLI](https://antigravity.google/) &nbsp;•&nbsp; [Claude Code](https://docs.anthropic.com/en/docs/claude-code) &nbsp;•&nbsp; [Codex](https://github.com/openai/codex) &nbsp;•&nbsp; *그리고 기타 모든 코딩 에이전트.*

## 시작하기

**사전 요구사항:** Python 3.11+, [uv](https://docs.astral.sh/uv/getting-started/installation/), 및 [Node.js](https://nodejs.org/en/download).

### 1. 설치

```bash
uvx google-agents-cli setup
```

<details>
<summary>스킬만 설치하는 경우 — 사용 중인 코딩 에이전트가 나머지를 처리합니다</summary>

```bash
npx skills add google/agents-cli
```

</details>

### 2. 코딩 에이전트 실행

[Antigravity CLI](https://antigravity.google/), [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Codex](https://github.com/openai/codex) 또는 선호하는 코딩 에이전트를 실행합니다.

### 3. 첫 번째 에이전트 구축

코딩 에이전트에 에이전트 구축을 요청하세요. 예: *"Use agents-cli to build a caveman-style agent that compresses verbose text into terse, technical grunts"*

단계별 가이드는 [전체 튜토리얼](https://google.github.io/agents-cli/guide/quickstart-tutorial/)을 참고하세요.

**[전체 문서 둘러보기 →](https://google.github.io/agents-cli/)**

---

## Agent Skills (에이전트 스킬)

| 스킬 | 코딩 에이전트가 학습하는 내용 |
|-------|-------------------------------|
| `google-agents-cli-workflow` | 개발 라이프사이클, 코드 보존 규칙, 모델 선택 |
| `google-agents-cli-adk-code` | ADK Python API — 에이전트, 툴, 오케스트레이션, 콜백, 상태 관리 |
| `google-agents-cli-scaffold` | 프로젝트 스캐폴딩 — `create`, `enhance`, `upgrade` |
| `google-agents-cli-eval` | 평가 방법론 — 메트릭, 데이터셋, LLM-as-judge, 적응형 루브릭 |
| `google-agents-cli-deploy` | 배포 — [Agent Runtime](https://docs.cloud.google.com/gemini-enterprise-agent-platform/scale), [Cloud Run](https://cloud.google.com/run), [GKE](https://cloud.google.com/kubernetes-engine), CI/CD, 보안 비밀(secrets) |
| `google-agents-cli-publish` | Gemini Enterprise 등록 |
| `google-agents-cli-observability` | 관찰 가능성 — Cloud Trace, 로깅, 서드파티 연동 |

---

## CLI 명령어

| 명령어 | 역할 |
|---------|-------------|
| `agents-cli setup` | CLI 및 코딩 에이전트용 스킬 설치 |
| `agents-cli scaffold <name>` | 새 에이전트 프로젝트 생성 |
| `agents-cli eval generate` | 평가 데이터셋에서 에이전트 실행 및 트레이스 생성 |
| `agents-cli eval grade` | 트레이스 기반 에이전트 평가 실행 |
| `agents-cli deploy` | Google Cloud에 배포 |
| `agents-cli publish gemini-enterprise` | Gemini Enterprise에 등록 |

<details>
<summary>전체 명령어 보기</summary>

| 명령어 | 설명 |
|---------|-------------|
| `agents-cli login` | Google Cloud 또는 AI Studio 인증 |
| `agents-cli login --status` | 인증 상태 표시 |
| **스캐폴드 (Scaffold)** | |
| `agents-cli scaffold <name>` | 새 에이전트 프로젝트 생성 |
| `agents-cli scaffold enhance` | 기존 프로젝트에 배포, CI/CD 또는 RAG 추가 |
| `agents-cli scaffold upgrade` | 프로젝트를 최신 agents-cli 버전으로 업그레이드 |
| **개발 (Develop)** | |
| `agents-cli run "prompt"` | 단일 프롬프트로 에이전트 실행 |
| `agents-cli install` | 프로젝트 의존성 설치 |
| `agents-cli lint` | 코드 품질 검사 실행 (Ruff) |
| **평가 (Evaluate)** | |
| `agents-cli eval generate` | 평가 케이스에 대한 에이전트 추론 실행 |
| `agents-cli eval grade` | 생성된 트레이스를 메트릭 대비 채점 |
| `agents-cli eval dataset synthesize` | 로컬 에이전트를 위한 멀티턴 평가 시나리오 합성 |
| `agents-cli eval compare` | 두 개의 평가 결과 파일 비교 |
| `agents-cli eval analyze` | 채점 결과에서 실패 모드 클러스터링 |
| `agents-cli eval metric list` | 사용 가능한 메트릭 목록 조회 |
| `agents-cli eval optimize` | 평가 데이터를 사용하여 에이전트 프롬프트 자동 튜닝 |
| **배포 및 게시 (Deploy & Publish)** | |
| `agents-cli deploy` | Google Cloud에 배포 |
| `agents-cli publish gemini-enterprise` | Gemini Enterprise에 등록 |
| `agents-cli infra single-project` | 단일 프로젝트 인프라 프로비저닝 |
| `agents-cli infra cicd` | CI/CD 파이프라인 및 스테이징/프로덕션 인프라 설정 |
| **데이터 (Data)** | |
| `agents-cli infra datastore` | RAG용 데이터스토어 인프라 프로비저닝 |
| `agents-cli data-ingestion` | 데이터 수집 파이프라인 실행 |
| **기타 (Other)** | |
| `agents-cli info` | 프로젝트 설정 및 CLI 버전 표시 |
| `agents-cli update` | 모든 IDE에 스킬 강제 재설치 |

</details>

## 작동 방식

<div align="center">
  <a href="https://youtu.be/ECYKo70pPNc">
    <img src="https://img.youtube.com/vi/ECYKo70pPNc/maxresdefault.jpg" alt="agents-cli demo video" width="100%" />
  </a>
</div>

---

## 아키텍처

`agents-cli`가 기반으로 하는 Google Cloud 에이전트 스택:

![Architecture](https://raw.githubusercontent.com/google/agents-cli/main/docs/src/assets/architecture.png "Architecture")

## 자주 묻는 질문 (FAQ)

**이 도구가 Antigravity CLI, Claude Code, Codex를 대체하나요?**<br>
아닙니다. **`agents-cli`는 코딩 에이전트 *자체*가 아니라 코딩 에이전트를 *위한* 도구입니다.** Google Cloud에서 ADK 에이전트를 구축, 평가, 배포하는 능력을 향상시키는 CLI 명령어와 스킬을 제공합니다.

**`adk`를 직접 사용하는 것과 어떻게 다른가요?**<br>
[ADK](https://adk.dev)는 에이전트 프레임워크입니다. `agents-cli`는 코딩 에이전트가 ADK 에이전트를 엔드투엔드로 구축, 평가, 배포할 수 있는 스킬과 도구를 제공합니다.

**Google Cloud가 반드시 필요한가요?**<br>
로컬 개발(`create`, `run`, `eval`)의 경우에는 필요하지 않습니다. [AI Studio API 키](https://aistudio.google.com/apikey)를 사용하여 로컬에서 [ADK](https://adk.dev)로 Gemini를 실행할 수 있습니다. 배포 및 클라우드 기능의 경우에는 Google Cloud가 필요합니다.

**기존 에이전트 프로젝트에 사용할 수 있나요?**<br>
네, 가능합니다. `agents-cli scaffold enhance`를 실행하면 기존 프로젝트에 배포 및 CI/CD 설정을 추가할 수 있습니다.

**코딩 에이전트 없이 `agents-cli`를 사용할 수 있나요?**<br>
네, 가능합니다. CLI는 단독으로 동작하므로 터미널에서 `agents-cli scaffold`, `eval`, `deploy` 및 기타 모든 명령어를 직접 실행할 수 있습니다. 스킬은 코딩 에이전트가 이러한 작업을 더 쉽게 수행할 수 있도록 지원합니다.

**`agents-cli`를 다른 스킬로 확장하려면 어떻게 해야 하나요?**<br>
`agents-cli` 스킬은 에이전트 구축 라이프사이클(스캐폴드, ADK 코드 패턴, 평가, 배포, 게시, 관찰 가능성)을 다룹니다. 관련 추가 작업의 경우 다른 스킬 제품군을 함께 설치할 수 있습니다. 예를 들어, [agent-skills](https://github.com/addyosmani/agent-skills)는 일반적인 소프트웨어 공학 워크플로우(아이디어 구상, 스펙 검증, 기획, 코드 리뷰)를 다루며, [google/skills](https://github.com/google/skills)는 Google Cloud 기초 기능(BigQuery, Cloud Run, Firebase, GKE)을 다룹니다.

## 피드백

사용자의 피드백은 커뮤니티를 위한 `agents-cli` 개선에 큰 도움이 됩니다.

- **버그 및 기능 요청:** [이슈 생성](https://github.com/google/agents-cli/issues/new) — 우선순위 적용을 원하는 이슈에 👍를 눌러주세요.
- **구축한 프로젝트 공유:** 여러분의 프로젝트 이야기를 환영합니다! 에이전트를 공유하거나 피드백을 제공하려면 <a href="mailto:agents-cli@google.com">agents-cli@google.com</a>으로 연락해 주세요.

## 기여하기

기여하는 가장 좋은 방법은 피드백 제공입니다. [이슈](https://github.com/google/agents-cli/issues)를 통해 버그 리포트, 기능 요청, 아이디어를 공유하여 로드맵 형성에 직접 참여할 수 있습니다.

자세한 내용은 [기여 가이드](CONTRIBUTING.md)를 참고하세요.

## 서비스 약관

`agents-cli`는 Google Cloud API를 활용합니다. 에이전트를 배포하면 사용자 소유의 Google Cloud 프로젝트에 리소스가 배포되며 해당 리소스에 대한 책임은 사용자에게 있습니다. 자세한 내용은 [Google Cloud 서비스 약관](https://cloud.google.com/terms/service-terms)을 확인하세요.
