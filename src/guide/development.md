# 개발 가이드

이 가이드는 구축하려는 내용을 정의하는 것부터 프로덕션 모니터링까지 전체 개발 워크플로우를 다룹니다. 코딩 에이전트가 `google-agents-cli-workflow` 스킬을 통해 사용하는 것과 동일한 단계를 따릅니다.

---

## 0단계: 이해 (Understand)

코드를 작성하기 전에 어떤 것을 구축할지 정의합니다.

코딩 에이전트와 함께 작업하는 경우 다음 질문을 자동으로 수행합니다. 수동으로 작업하는 경우 스스로 답변해 보세요:

1. **에이전트가 어떤 문제를 해결하는가?** — 핵심 목적 및 기능
2. **필요한 외부 API 또는 데이터 소스는 무엇인가?** — 툴, 연동, 인증 요구사항
3. **안전 제약 조건은 무엇인가?** — 에이전트가 절대 수행하면 안 되는 행동
4. **선호하는 배포 환경은 무엇인가?** — 프로토타입 우선, 또는 전체 배포(Agent Runtime, Cloud Run, GKE)?

현재 디렉토리의 `.agents-cli-spec.md`에 답변을 저장하세요 — 개요, 유스케이스 예시, 필요한 툴, 제약 조건, 성공 기준 등을 포함합니다.

---

## 1단계: 스캐폴드 (Scaffold)

템플릿에서 새 프로젝트를 생성합니다:

```bash
agents-cli create my-agent
```

생성 과정에서 에이전트 템플릿(`adk`)과 배포 대상을 선택합니다. (RAG는 복사 및 학습용 레시피입니다 — [템플릿](templates.md#rag-retrieval-augmented-generation) 참고.) 인프라 결정 없이 빠른 프로토타이핑을 하려면 다음을 실행합니다:

```bash
agents-cli create my-agent --prototype --yes
```

추후 `agents-cli scaffold enhance`를 실행하여 배포 지원을 추가할 수 있습니다.

모든 옵션은 [에이전트 템플릿](templates.md)을 참고하세요.

---

## 2단계: 구축 및 반복 (Build & Iterate)

### 코딩 에이전트 사용 시

코딩 에이전트를 열고 워크플로우 스킬을 활성화합니다:

```
/google-agents-cli-workflow
```

구축하려는 내용을 설명하세요. 코딩 에이전트가 설치된 스킬을 사용하여 ADK 모범 사례를 준수하는 에이전트 로직 작성, 툴 생성 및 변경 사항 테스트를 수행합니다.

### 수동 진행 시

`app/agent.py`에서 에이전트 로직을 수정하고 다음 명령어로 테스트합니다:

- `agents-cli playground` — `localhost:8080`에서 핫 리로드를 지원하는 ADK 웹 플레이그라운드 실행
- `agents-cli run "your prompt"` — 터미널에서 빠른 스모크 테스트 실행

### 코드 품질 검사

```bash
agents-cli lint                                # Ruff 검사 및 포맷팅
uv run pytest tests/unit tests/integration     # 단위 및 통합 테스트 실행
```

### 패키지 관리

[uv](https://docs.astral.sh/uv/)를 사용하여 의존성을 추가하거나 제거합니다:

- `uv add <package>`
- `uv remove <package>`

---

## 3단계: 평가 (Evaluate)

에이전트 동작을 검증하기 위해 구조화된 평가를 실행합니다. 내부적으로 [GenAI Eval SDK](https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/evaluation/agent-evaluation)를 활용합니다.

```bash
agents-cli eval generate
agents-cli eval grade
```

에이전트가 평가를 안정적으로 통과하기까지 평가-수정 루프를 **5~10회 이상 반복**하게 될 것으로 예상해야 합니다. 1~2개의 핵심 평가 케이스로 시작하여 실패 케이스를 수정하고 검증 범위를 확장하세요.

메트릭, 데이터셋 스키마 및 전체 방법론은 [평가 가이드](evaluation.md)를 참고하세요.

---

## 4단계: 배포 (Deploy)

평가 임계값을 충족하면 Google Cloud에 배포합니다.

1. **배포 대상 추가** (`--prototype`으로 시작한 경우):

    ```bash
    agents-cli scaffold enhance --deployment-target cloud_run
    ```

2. **배포 실행**:

    ```bash
    agents-cli deploy
    ```

!!! tip
    관찰 가능성 기능(프롬프트-응답 로깅, 콘텐츠 로그)을 활성화하려면 배포 후 `agents-cli infra single-project`를 실행하세요. 자세한 내용은 [관찰 가능성 가이드](observability/index.md)를 참고하세요.

스테이징, 승인 단계 및 CI/CD를 갖춘 프로덕션 파이프라인은 [배포](deployment.md) 및 [CI/CD 및 프로덕션](cicd.md)을 참고하세요.

---

## 5단계: 게시 (Publish - 선택 사항)

배포된 에이전트를 Gemini Enterprise에 등록합니다:

```bash
agents-cli publish gemini-enterprise
```

모든 에이전트에 필요한 것은 아니며, Gemini Enterprise를 통해 배포하는 경우에만 수행합니다.

---

## 6단계: 관찰 (Observe)

프로덕션에서 에이전트를 모니터링합니다. Cloud Trace는 배포된 모든 에이전트에 기본으로 활성화되어 있으며 별도 설정이 필요하지 않습니다.

- **Cloud Trace** — 분산 트레이싱, 지연 시간 분석, 에러 관찰
- **BigQuery Agent Analytics** — 토큰 사용량, 대화 패턴 및 LLM 판정관 채점을 위한 선택적 고급 분석

설정 및 사용법은 [관찰 가능성 가이드](observability/index.md)를 참고하세요.

---

모든 명령어와 플래그는 [CLI 참조 문서](../cli/index.md)를 참고하세요. 각 단계에서 코딩 에이전트가 사용하는 스킬에 대한 자세한 내용은 [스킬 참조 문서](../reference/skills.md)를 참고하세요.
