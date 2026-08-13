# 라이프사이클

Agents CLI는 **"노트북에서는 잘 동작함"**과 **"프로덕션 라이브"** 사이의 루프라는 한 가지 철학에 집중합니다. 이 페이지는 그 지도 역할을 합니다.

## 단일 조사 과정 살펴보기

일주일 동안 라이브 상태였던 장애 복구 에이전트를 상상해 보세요. 장애 알림이 발생합니다:

<div id="lifecycle-anim-transcript" class="lifecycle-anim" aria-label="Auto-playing transcript of an outage investigation"></div>

이 조사에는 **4.3초**가 걸렸습니다. *에이전트 자체*의 특별한 점은 없습니다 — 대부분의 에이전트 프레임워크가 이를 표현할 수 있습니다. 특별한 것은 그 주변의 모든 환경입니다: 파괴적인 복구 조치를 권장했다면 배포를 허용하지 않았을 평가 루브릭, 런북 검색이 잘못된 섹션을 반환하는 것을 차단했을 CI 검사, 내일 무언가 잘못되었을 때 동일한 조사를 재현할 수 있게 해주는 트레이스입니다.

이것이 바로 루프입니다.

## 순환하는 4가지 CLI 동사

<div id="lifecycle-anim-loop" class="lifecycle-anim" aria-label="The four CLI verbs in a continuous loop"></div>

`scaffold`, `eval`, `deploy`, observe — 영원히 순환합니다. 사용자가 명세를 작성하면, 루프는 배포될 뻔한 오류를 잡아내고, 통과한 코드를 배포하며, 다음 결과를 보여주어 다음 반복을 더 똑똑하게 만듭니다.

## 루프가 없을 때 발생하는 문제

대부분의 에이전트 데모는 프롬프트 단계에서 멈춥니다. 영리한 지시사항을 작성하면 모델이 노트북에서 훌륭해 보이는 결과를 반환하고, 이를 팀에 캡처하여 보여줍니다. 하지만 프로덕션 환경 배포에는 실전의 도전 과제들이 존재합니다.

| | 루프가 없을 때 | Agents CLI 사용 시 |
|---|---|---|
| **환각에 의한 복구 조치 제안** | 머지 후 고객 측에서 사후 발견 | 머지 전 평가 루브릭이 PR을 차단 |
| **툴 API 변경** | 새벽 2시 장애 알림, 에이전트 오동작 | CI 통합 테스트가 스키마 변경 감지 |
| **프로덕션 오용** | 리플레이 및 텔레메트리 없음 | Cloud Trace + BigQuery 분석을 통해 1시간 이내 포착 |
| **과도한 툴 호출로 인한 비용 급증** | 다음 달 청구서가 알림 역할 | 툴별 스팬 카운트를 통해 몇 시간 내 포착 |

## 8단계 프로세스

루프를 천천히 살펴보면 8개 단계로 확장됩니다. 각 단계에는 [스킬](../reference/skills.md)에 인코딩된 철학이 있어 코딩 에이전트가 최선의 선택을 하도록 돕습니다.

| # | 단계 | 역할 | CLI 동사 | 스킬 | 상세 가이드 |
|---|---|---|---|---|---|
| 0 | **Spec (명세)** | `.agents-cli-spec.md`를 작성합니다. 다른 단계들은 이 명세에서 파생됩니다. | — | `google-agents-cli-workflow` | [개발 가이드](development.md) |
| 1 | **Scaffold (스캐폴드)** | 명세를 프로덕션 형태의 프로젝트(~72개 파일)로 변환합니다. | `scaffold create` | `google-agents-cli-scaffold` | [템플릿](templates.md) |
| 2 | **Build (구축)** | 모델, 지시사항, 툴, `App` 래퍼 등 에이전트 본체를 작성합니다. | — | `google-agents-cli-adk-code` | [프로젝트 구조](project-structure.md) |
| 3 | **Orchestrate (오케스트레이션)** | 단일 에이전트가 팀으로 확장될 때 전문 에이전트들을 구성합니다. | — | `google-agents-cli-adk-code` | [프로젝트 구조](project-structure.md) |
| 4 | **Evaluate (평가)** | 모든 배포 전에 데이터셋 대비 에이전트 점수를 측정합니다. | `eval generate`, `eval grade` 및 `eval dataset synthesize`, `eval compare`, `eval analyze`, `eval metric list`, `eval optimize` | `google-agents-cli-eval` | [평가](evaluation.md) |
| 5 | **Deploy (배포)** | Agent Runtime, Cloud Run 또는 GKE에 배포합니다. | `deploy` | `google-agents-cli-deploy` | [배포](deployment.md) |
| 6 | **Publish (게시)** | 다른 에이전트가 검색할 수 있도록 Gemini Enterprise에 등록합니다. | `publish` | `google-agents-cli-publish` | [CI/CD](cicd.md) |
| 7 | **Observe (관찰)** | Cloud Trace + BigQuery 분석을 수행하며, 프로덕션 데이터는 내일의 데이터셋으로 환원됩니다. | — | `google-agents-cli-observability` | [관찰 가능성](observability/index.md) |

### 0 · Spec (명세)

`.agents-cli-spec.md`는 에이전트의 툴, 제약 조건 및 성공 기준을 정의합니다. 라이프사이클의 나머지 모든 부분이 이 명세를 참조합니다: 스캐폴드 플래그, 평가 루브릭, 안전 가드레일, 프로덕션에서 모니터링할 트레이스 속성 등입니다. 빈 화면에서 시작하지 말고, [Agent Garden](https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/agent-garden)에서 원하는 형태와 가까운 기존 템플릿을 찾아 커스텀하세요.

전형적인 명세는 한 화면 분량의 마크다운입니다:

```markdown
# .agents-cli-spec.md — outage-recovery-bot

## Tools

| Tool                                    | Backing service       |
| --------------------------------------- | --------------------- |
| `query_logs(service, severity)`         | Cloud Logging         |
| `check_metrics(service, metric)`        | Cloud Monitoring      |
| `search_runbook(query)`                 | Vector Search         |

## Constraints

1. Always cite the runbook section consulted.
2. Never recommend a destructive remediation unless the runbook
   explicitly sanctions it for the observed symptom.

## Success criteria

- ≥ 80% of incidents get a diagnosis whose root cause matches ground truth
- 100% of recommendations cite a runbook section
- 0 destructive recommendations without runbook sanction
```

### 1 · Scaffold (스캐폴드)

단 하나의 명령어로 명세를 읽어 에이전트 코드, 테스트, 평가 보일러플레이트, Terraform, CI/CD 워크플로우, 배포 매니페스트 등 프로젝트를 생성합니다. 플래그는 불필요하게 제공되지 않으며, 지정된 라이프사이클에 맞게 스캐폴드를 확장하거나 축소합니다.

<div id="lifecycle-anim-scaffold" class="lifecycle-anim" aria-label="Scaffold wizard — toggle flags, watch the command and file count update"></div>

전체 설정 시 에이전트 코드, 평가 보일러플레이트, Terraform, GitHub Actions 워크플로우, 배포 매니페스트 전반에 걸쳐 **~72개 파일**이 생성됩니다. 필요하지 않은 요소를 건너뛰어 슬림하게 유지할 수 있습니다. 전체 목록은 [템플릿](templates.md)을 참고하세요.

### 2 · Build (구축)

모든 ADK 에이전트는 모델, 지시사항, 툴 목록, 그리고 이를 감싸는 `App`이라는 4가지 핵심 요소로 귀결됩니다. 에이전트 본체는 약 30줄 정도의 코드이며, 중요한 작업은 툴 내부에서 일어납니다.

```python
from google.adk.agents import Agent
from google.adk.apps import App
from google.adk.models import Gemini

root_agent = Agent(
    name="root_agent",
    model=Gemini(model="gemini-3.6-flash"),
    instruction="You are an SRE outage-recovery assistant...",
    tools=[query_logs, check_metrics, search_runbook],
)

app = App(root_agent=root_agent, name="app")
```

Gemini에만 국한되지 않습니다 — 모델 구문을 ADK가 지원하는 다른 제공업체로 교체할 수 있습니다([Model Garden](https://cloud.google.com/model-garden)에서 Anthropic Claude, OpenAI GPT 등을 지원). 교체하더라도 라이프사이클의 나머지 동작 방식은 동일합니다.

상태 유지가 필요한 에이전트는 Agent Platform의 두 가지 추가 기능을 사용합니다:

- **매니지드 세션 저장소 (Managed session storage)**: 재시작 시에도 유지되고 수평 확장이 가능한 대화 상태 관리 기능으로, 스캐폴드 시 기본 인메모리 대신 `--session-type agent_platform_sessions` 옵션으로 선택할 수 있습니다.
- **[Memory Bank](https://cloud.google.com/agent-builder/docs/memory)**: 세션을 뛰어넘는 *장기* 메모리로, SRE 봇이 "지난 분기 장애와 유사함"을 인식할 수 있게 해줍니다. `from google.adk.memory import VertexAiMemoryBankService`를 통해 연결하면 사용자, 세션, 앱 키에 지정된 영구 저장소를 얻게 됩니다.

단일 HTTP 요청에 다 담기지 않는 워크플로우(긴 조사, 다단계 배치 작업)의 경우, Agent Runtime이 에이전트 상태를 유지하므로 배포나 재시작 시에도 진행 상황을 잃지 않습니다.

<div id="lifecycle-anim-models" class="lifecycle-anim" aria-label="Same prompt, three model providers — illustrative side-by-side"></div>

동일한 에이전트 본체가 다른 장애 시나리오에 응답하는 전체 과정입니다:

<div id="lifecycle-anim-playground" class="lifecycle-anim" aria-label="Inline playground — payments triage scenario, click to step through"></div>

### 3 · Orchestrate (오케스트레이션)

단일 에이전트 구조는 문제 규모가 작을 때 유효합니다. 실제 프로덕션 에이전트는 **팀** 단위로 성장합니다 — 오케스트레이터가 각각 좁은 역할과 툴을 가진 전담 에이전트들에게 작업을 라우팅하는 구조입니다.

<div id="lifecycle-anim-team" class="lifecycle-anim" aria-label="Team diagram — orchestrator routes work to investigator, diagnoser, and remediator"></div>

팀 분할은 평가, 배포, 관찰 시 세 가지 이점을 줍니다: 더 작고 명확한 프롬프트는 에이전트 신뢰성을 향상시키며, 독립된 툴 영역을 통해 에이전트별 가드레일을 적용할 수 있고, 트레이스를 통해 어떤 서브 에이전트가 오동작했는지 정확히 추적할 수 있습니다.

팀이 프로세스 경계를 넘어서거나 타 팀이 소유한 에이전트를 호출해야 하는 경우, 통신 프로토콜로 **[A2A 프로토콜](https://a2a-protocol.org/)**을 사용하세요. A2A는 모든 ADK 에이전트에 내장되어 있으므로 정상적으로 스캐폴딩(`--agent adk`)하면 됩니다. Agents CLI로 만들어졌든 아니든 A2A 호환 에이전트라면 서로 자유롭게 호출할 수 있습니다.

### 4 · Evaluate (평가)

대부분의 에이전트 데모가 건너뛰는 단계입니다. `agents-cli eval generate` 실행 후 `agents-cli eval grade`를 실행하면 라이브 에이전트에 대해 데이터셋을 실행하고, LLM 판정관(judge)을 통해 루브릭 대비 각 응답을 채점하여 검증 가능한 객관적 수치를 제공합니다.

<div id="lifecycle-anim-eval" class="lifecycle-anim" aria-label="Eval-fix loop — click 'apply fix' to see one case flip from failing to passing"></div>

`agents-cli eval grade` 루프를 5~10회 이상 반복 수행하게 됩니다. 수정할 때마다 점수가 향상되고, 재실행을 거쳐 목표 임계값을 넘으면 배포하게 됩니다. 아래는 루브릭이 가장 자주 잡아내는 4가지 실패 유형입니다.

<div id="lifecycle-anim-failures" class="lifecycle-anim" aria-label="Common agent failures and the eval rubric that catches each"></div>

메트릭, 데이터셋 스키마 및 전체 방법론은 [평가 가이드](evaluation.md)를 참고하세요.

### 5 · Deploy (배포)

동일한 에이전트 코드가 3가지 다른 환경에 배포될 수 있습니다. `agents-cli deploy`는 스캐폴딩 시 선택한 대상에 따라 작동합니다. **환경을 선택하여 `--dry-run` 출력 내용과 다음 후속 단계를 확인하세요:**

<div id="lifecycle-anim-deploy" class="lifecycle-anim" aria-label="Deploy target picker — choose a runtime to see the dry-run + pipeline"></div>

```bash
agents-cli deploy --dry-run        # 파이프라인 미리보기
agents-cli deploy                  # 배포 실행
agents-cli deploy --no-wait        # 즉시 반환; 나중에 --status 로 확인
```

각 배포 대상은 주변의 프로덕션 기본 기능들을 상속받습니다:

- **에이전트별 서비스 계정**: `agents-cli deploy --agent-identity` 옵션을 통해 배포된 에이전트가 전용 GCP IAM 보안 주체로 실행됩니다. 일반 IAM을 사용하여 접근 가능한 대상(BigQuery 데이터셋, 버킷, API 등)을 최소 권한으로 설정하세요. 파괴적 복구를 차단하는 평가 루브릭에는 마지막 안전장치가 있습니다: 권한이 없으면 에이전트가 `kubectl delete`를 물리적으로 실행할 수 없습니다.
- **[Identity-Aware Proxy (IAP)](https://cloud.google.com/iap)**: `--iap` 플래그를 사용하여 Cloud Run 배포를 Google Workspace SSO 뒤에 보호할 수 있습니다. 내부 전용 에이전트가 공용 인터넷에 노출될 우려가 사라집니다.
- **[Workload Identity Federation (WIF)](https://cloud.google.com/iam/docs/workload-identity-federation)**: 스캐폴드된 `pr_checks.yaml`이 WIF를 통해 GitHub Actions를 GCP에 인증하므로 서비스 계정 키를 리포지토리에 저장할 필요가 없습니다.

대상별 전체 단계는 [배포 가이드](deployment.md)를 참고하세요.

### 6 · Publish (게시)

에이전트 배포를 통해 URL로 접근할 수 있게 됩니다. 게시(Publish)는 다른 에이전트(또는 카탈로그를 탐색하는 사용자)가 이 에이전트를 실제로 찾을 수 있도록 Gemini Enterprise에 등록하는 별도의 단계입니다.

<div id="lifecycle-anim-publish" class="lifecycle-anim" aria-label="The agent's listing in Gemini Enterprise after publish"></div>

두 가지 등록 모드: **ADK** (배포된 Agent Runtime 인스턴스 게시) 및 **[A2A](https://a2a-protocol.org/)** (A2A 호환 HTTP 엔드포인트 게시, ADK 필요 없음 — 모든 프레임워크 기반 에이전트 지원).

### 7 · Observe (관찰)

에이전트가 라이브 상태가 되면, 모든 호출은 Cloud Trace 스팬을 내보냅니다. 모든 툴 호출, 모델 생성 및 서브 에이전트 핸드오프를 시각적으로 확인할 수 있습니다. **아래 스팬 위에 마우스를 올려 속성을 검사해 보세요.**

<div id="lifecycle-anim-trace" class="lifecycle-anim" aria-label="Trace waterfall — bars draw in left-to-right showing the orchestrator and its sub-agents; hover to inspect"></div>

프로덕션에서 실행되는 에이전트에는 관찰 가능성이 필수적입니다. 평가에서 놓쳤을 수 있는 성능 저하, 과도한 호출 툴로 인한 비용 급증, 사용자가 안전 프롬프트를 우회하는 사례 등을 포착할 수 있습니다. 스캐폴드 시 `--bq-analytics` 옵션을 켜면 모든 프롬프트와 응답이 오프라인 분석을 위해 BigQuery에 저장됩니다.

이 데이터는 다시 루프를 완성합니다: 프로덕션 트래픽이 내일의 데이터셋으로 환원됩니다. 평가 점수가 지속적으로 재계산되므로 성능 저하가 수개월이 아닌 며칠 만에 포착됩니다.

<div id="lifecycle-anim-rolling" class="lifecycle-anim" aria-label="Rolling production eval score over the last ten days, with annotated regression and deploy events"></div>

전체 설정 방법은 [관찰 가능성 가이드](observability/index.md)를 참고하세요.

## 실행하는 두 가지 방법

<div class="lc-tabs-bare" markdown>

=== "코딩 에이전트에 요청하기"

    가장 추천하는 표준적인 방법입니다. 코딩 에이전트가 스킬을 읽고 각 단계에 맞는 올바른 CLI 명령어를 선택합니다.

    ```
    Build me an outage-recovery agent. It should investigate incidents
    using logs, metrics, and runbooks, and recommend remediations
    that cite a runbook section. Deploy it to Agent Runtime.
    ```

    코딩 에이전트의 수행 과정:

    1. 툴과 제약 조건을 정의하는 `.agents-cli-spec.md` 작성
    2. `agents-cli scaffold create … --agent adk --deployment-target agent_runtime` 실행 (RAG는 복사 및 학습용 레시피입니다 — RAG 샘플 활용; [템플릿](templates.md#rag-retrieval-augmented-generation) 참고)
    3. 에이전트 본체 및 툴 구현
    4. 데이터셋 테스트 케이스 작성
    5. `agents-cli eval generate` 및 `agents-cli eval grade` 실행 후, 점수가 임계값을 넘을 때까지 `eval grade` 반복
    6. `agents-cli deploy` 실행
    7. 트레이스 및 분석 연결 후 사용자에게 URL 전달

=== "CLI 직접 실행하기"

    모든 명령어는 단독으로 실행 가능합니다. 직접 명령어를 입력하려면 코딩 에이전트를 건너뛰어도 됩니다.

    ```bash
    # Phase 1: scaffold (스캐폴드)
    agents-cli scaffold create outage-recovery-bot \
      --agent adk \
      --deployment-target agent_runtime \
      --cicd-runner github_actions \
      --bq-analytics
    cd outage-recovery-bot && agents-cli install

    # Phase 2-3: build & orchestrate (app/agent.py 수정)
    agents-cli playground       # :8080 포트에서 로컬 웹 플레이그라운드 실행

    # Phase 4: evaluate (평가)
    agents-cli eval dataset synthesize --count 10  # 선택 사항: 데이터셋 콜드스타트 생성
    agents-cli eval generate
    agents-cli eval grade                          # 평가 점수가 임계값을 넘을 때까지 반복
    agents-cli eval compare prev.json latest.json  # 수정을 통해 실제 성능이 향상되었는지 확인
    agents-cli eval analyze --eval-result latest.json  # 남은 실패 케이스 클러스터링
    agents-cli eval optimize                       # 선택 사항: 평가 데이터로 프롬프트 자동 튜닝

    # Phase 5: deploy (배포)
    agents-cli deploy --dry-run
    agents-cli deploy

    # Phase 6: publish (게시 - 선택 사항)
    agents-cli publish gemini-enterprise
    ```

    전체 과정은 [수동 워크플로우 튜토리얼](hands-on-tutorial.md)을 참고하세요.

</div>

## 더 자세히 알아보기

- [템플릿](templates.md) — 스캐폴드 템플릿(`adk`) 및 RAG 학습 레시피
- [프로젝트 구조](project-structure.md) — 생성된 각 파일의 역할
- [개발 가이드](development.md) — 일상적인 개발 워크플로우
- [평가 가이드](evaluation.md) — 데이터셋 스키마 및 평가-수정 루프
- [배포](deployment.md) — 배포 대상별 세부 가이드
- [CI/CD 및 프로덕션](cicd.md) — PR에서 프로덕션까지의 전체 경로
- [관찰 가능성](observability/index.md) — Cloud Trace, BigQuery 분석 및 서드파티 도구
- [CLI 참조 문서](../cli/index.md) — 모든 명령어 및 플래그
