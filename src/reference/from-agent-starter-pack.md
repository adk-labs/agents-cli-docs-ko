# Agent Starter Pack에서 마이그레이션

`agents-cli`는 Agent Starter Pack (ASP)의 후속 도구입니다. 동일한 기반 위에서 주요 개선 사항들을 더해 구축되었습니다.

---

## 변경된 사항

**코딩 에이전트 우선 (Coding agent first).** ASP는 대화형 CLI를 실행하는 사람(개발자)을 위해 개발되었습니다. 반면 `agents-cli`는 코딩 에이전트를 위해 개발되었으며, ADK, 평가, 배포 및 관측 가능성에 대한 심도 있는 컨텍스트를 제공하는 7개의 번들 스킬이 포함되어 있습니다. 물론 모든 명령은 터미널에서도 동일하게 동작합니다.

**Makefile을 대체하는 CLI.** ASP는 `make` 타깃(`make dev`, `make eval`, `make deploy`)을 사용했습니다. `agents-cli`는 이를 플래그, 도움말 텍스트 및 구조화된 출력을 통해 전체 라이프사이클을 아우르는 통합 CLI로 대체합니다.

**새로운 기능들.** `agents-cli`에는 ASP에 존재하지 않았던 명령들이 추가되었습니다: `playground`, `run`, `deploy`, 전체 평가 기능(`eval generate`, `eval grade`, `eval dataset synthesize`, `eval compare`, `eval analyze`, `eval metric list`, `eval optimize`), `lint`, `login`, 그리고 스킬 관리(`setup`, `update`).

### 명령 매핑

| Agent Starter Pack | agents-cli |
|---|---|
| `create` | `create` (`scaffold create`의 별칭) |
| `enhance` | `scaffold enhance` |
| `upgrade` | `scaffold upgrade` |
| `setup-cicd` | `infra cicd` |
| `register-gemini-enterprise` | `publish gemini-enterprise` |

### 설정 키

설정이 `pyproject.toml`의 `[tool.agent-starter-pack]`에서 전용 `agents-cli-manifest.yaml`로 이동했습니다:

**이전 (`pyproject.toml`)**
```toml
[tool.agent-starter-pack]
agent_directory = "app"

[tool.agent-starter-pack.create_params]
deployment_target = "cloud_run"
```

**이후 (`agents-cli-manifest.yaml`)**
```yaml
name: my-agent
agent_directory: app
create_params:
  deployment_target: cloud_run
```

### 템플릿 커버리지

`agents-cli`는 `adk` 템플릿(Python)을 지원하며, 모든 ADK 에이전트에 A2A가 내장되어 있습니다 (독립형 `adk_a2a` 템플릿이 `adk`로 통합됨). RAG는 템플릿이라기보다는 복제 후 학습하여 적용하는 레시피입니다 (이전의 `agentic_rag` 템플릿은 제거되었으며, 대신 `rag-vector-search` / `rag-agent-search` 샘플을 적용하세요). ASP에는 `agents-cli`에서 아직 지원되지 않는 추가 템플릿들(`adk_go`, `adk_java`, `adk_ts`, `adk_live`, `custom_a2a`)이 존재했습니다. 이들에 대한 지원도 계획되어 있습니다.

### 동일하게 유지되는 사항

- **템플릿** — 동일한 `adk` 에이전트 템플릿(RAG는 이제 복제 후 학습하여 적용하는 레시피), 동일한 배포 타깃, 동일한 세션 저장소 옵션
- **프로젝트 구조** — 생성된 프로젝트는 동일한 레이아웃을 가지며 `app/agent.py` 코드는 변경되지 않음
- **Terraform** — `deployment/terraform/` 아래의 동일한 IaC(infrastructure-as-code)
- **CI/CD 파이프라인** — 동일한 Cloud Build 및 GitHub Actions 설정

---

## 기존 프로젝트 마이그레이션하기

기존 ASP 프로젝트는 완전한 호환성을 유지합니다. 필요한 유일한 변경 사항은 `pyproject.toml`에서 설정 섹션의 이름을 변경하는 것입니다.

**Step 1: agents-cli 설치**

```bash
uvx google-agents-cli setup
```

**Step 2: 설정 섹션 이름 변경**

```bash
sed -i '' 's/tool.agent-starter-pack/tool.agents-cli/g' pyproject.toml
```

다음에 설정을 읽을 때 `agents-cli-manifest.yaml`로의 마이그레이션이 트리거되고 `pyproject.toml`에서 `tool.agents-cli` 섹션이 제거됩니다.

**Step 3: 검증**

```bash
agents-cli info
```

프로젝트 설정을 보여주며 `agents-cli`가 해당 설정을 정상적으로 읽을 수 있음을 확인합니다. 에이전트 코드, 테스트, Terraform, CI/CD 파이프라인 모두 이전과 동일하게 작동합니다.

!!! note "`tests/eval/evalsets/` 아래에 기존 평가 케이스가 있나요?"
    ASP의 기본 에이전트 템플릿은 ADK `EvalSet` 스키마를 사용하는 `basic.evalset.json`을 포함했습니다. `agents-cli`의 평가 기능은 `tests/eval/datasets/`에서 다른 포맷을 읽어옵니다. 변환 방법은 [평가 데이터셋 마이그레이션](eval-dataset-migration.md) 문서를 참고하세요.
