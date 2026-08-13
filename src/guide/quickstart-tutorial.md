# 튜토리얼: 첫 번째 에이전트 구축하기

*코딩 에이전트를 사용하여 에이전트를 구축, 평가 및 배포하려는 입문자를 위한 가이드입니다.*

이 튜토리얼에서는 Agent Platform에서의 Agents CLI의 모든 활용 방식을 다룹니다 — 사용자는 코딩 에이전트에 대화만 하면 되며, 코딩 에이전트가 사용자를 위해 ADK 에이전트를 구축, 평가 및 배포합니다.

사용자는 장황한 텍스트를 압축하여 간결한 원시인 스타일의 요약으로 바꾸는 에이전트인 **원시인 압축기(caveman compressor)**를 작성하게 됩니다. [caveman](https://github.com/JuliusBrussee/caveman) 프로젝트에서 영감을 받았습니다.

엔드투엔드 작동 모습은 다음과 같습니다:

![agents-cli demo](https://raw.githubusercontent.com/google/agents-cli/assets/agents-cli-demo.gif)

---

## 환경 설정

사용자가 직접 실행하는 유일한 명령어입니다. 그 외의 모든 작업은 코딩 에이전트를 통해 진행됩니다.

```bash
uvx google-agents-cli setup
```

그런 다음 선호하는 코딩 에이전트 — [Antigravity CLI](https://antigravity.google/), [Claude Code](https://docs.anthropic.com/en/docs/claude-code), [Codex](https://github.com/openai/codex) 등을 실행합니다.

---

## 1. 스캐폴드 (Scaffold)

코딩 에이전트에 다음과 같이 요청하세요:

> *"Use agents-cli to build a caveman-style agent that compresses verbose text into terse, technical grunts"*

코딩 에이전트는 `google-agents-cli-workflow` 및 `google-agents-cli-scaffold` 스킬을 활성화합니다. 그리고 다음 작업을 수행합니다:

- 확인을 위한 질문 수행 (배포 대상, 안전 제약 조건 등)
- 에이전트 목적을 정의하는 `.agents-cli-spec.md` 명세 저장
- 프로젝트 스캐폴딩 실행:

```
agents-cli create caveman-agent --prototype --yes
cd caveman-agent && agents-cli install
```

이제 에이전트 보일러플레이트 코드, 테스트 및 평가 세트가 포함된 작동하는 프로젝트가 생성되었습니다.

---

## 2. 구축 (Build)

코딩 에이전트가 `app/agent.py`를 편집하여 기본 에이전트를 원시인 압축기 에이전트로 교체합니다. 이때 ADK 패턴을 위해 `google-agents-cli-adk-code` 스킬을 활용합니다.

에이전트 정의 코드는 대략 다음과 같이 작성됩니다:

```python title="app/agent.py"
root_agent = Agent(
    name="caveman_agent",
    model=Gemini(model="gemini-3.6-flash"),
    instruction="""You caveman compressor. Human give long words,
    you make short. Rules:
    - No articles. No filler. No fluff.
    - Short grunts. Simple words.
    - Keep technical terms but grunt around them.
    - Funny but meaning stays.

    Example input:  "I would like to deploy the application to production"
    Example output: "Me deploy. Production. Now."
    """,
)
```

이후 코딩 에이전트가 스모크 테스트를 실행합니다:

```
agents-cli run "Please help me understand the deployment options available for my project"
```

출력 예시:

```
Deploy options: Agent Runtime, Cloud Run, GKE. Pick one. Ship.
```

---

## 3. 평가 (Evaluate)

코딩 에이전트에 다음과 같이 요청하세요:

> *"Write evals for the caveman agent and run them"*

코딩 에이전트는 `google-agents-cli-eval` 스킬을 활성화하고 다음을 수행합니다:

- 테스트 케이스가 담긴 `tests/eval/datasets/caveman-dataset.json` 생성 (압축 품질, 기술 용어 보존, 원시인 톤앤매너)
- 평가 실행:

```bash
agents-cli eval generate
agents-cli eval grade
```

테스트 케이스가 실패하면 수정할 내용을 코딩 에이전트에 지시합니다:

> *"The response to the greeting test is too polite. Make it more caveman."*

코딩 에이전트가 지시사항을 수정하고 평가를 재실행하며, 품질 임계값을 통과할 때까지 반복(iterate)합니다.

평가 기능은 `generate` 및 `grade` 외에도 다양하게 제공됩니다 — `eval dataset synthesize`, `eval compare`, `eval analyze`, `eval optimize` 명령어는합성 케이스 생성, 리그레션 비교, 실패 분석 및 프롬프트 자동 튜닝을 다룹니다. 전체 기능은 [평가 가이드](evaluation.md#beyond-generate-and-grade)를 참고하세요.

---

## 4. 배포 (Deploy)

코딩 에이전트에 다음과 같이 요청하세요:

> *"Deploy this to Cloud Run"*

코딩 에이전트는 `google-agents-cli-deploy` 스킬을 활성화하고 다음을 수행합니다:

- 배포 인프라 구성 추가:

```
agents-cli scaffold enhance --deployment-target cloud_run
```

- 배포 실행:

```
agents-cli deploy
```

이제 원시인 에이전트가 라이브 상태가 되었습니다. 출력 결과에 Cloud Run URL이 표시됩니다.

---

## 5. 관찰 (Observe)

Cloud Trace는 기본적으로 활성화되어 있어 별도 설정이 필요하지 않습니다. Google Cloud Console에서 [Trace 탐색기](https://console.cloud.google.com/traces)를 열고 에이전트에 몇 가지 요청을 보냅니다. 각 LLM 호출 및 툴 실행에 대한 스팬(span)을 확인할 수 있습니다.

한 걸음 더 나아가 프로덕션에서 에이전트가 처리하는 실제 프롬프트와 응답을 검사하려면 코딩 에이전트에 다음과 같이 요청하세요:

> *"Set up observability infrastructure for my agent"*

코딩 에이전트가 `infra single-project`를 실행하여 서비스 계정, GCS 버킷 및 BigQuery 데이터셋을 프로비저닝하고 배포된 서비스가 이를 사용하도록 업데이트합니다. 검증 단계 및 고급 옵션은 [관찰 가능성 가이드](observability/index.md)를 참고하세요.

---

## 일어난 작업 정리

내부적으로 각 프롬프트가 어떤 작업을 실행했는지 정리한 내용입니다:

| 사용자 요청 | 코딩 에이전트의 수행 작업 |
|----------|----------------------|
| *"Build a caveman compressor agent"* | 프로젝트 스캐폴딩, 에이전트 코드 작성, 로컬 테스트 |
| *"Write evals and run them"* | 데이터셋 생성, `generate` 및 `grade`를 사용한 평가 실행 |
| *"Deploy this to Cloud Run"* | 배포 대상 추가, Cloud Run 배포 실행 |
| *"Set up observability"* | 서비스 계정, GCS 버킷 및 BigQuery 데이터셋 프로비저닝 |

스킬을 통해 코딩 에이전트는 각 단계에서 올바른 결정을 내릴 수 있는 컨텍스트(사용할 ADK 패턴, 평가 구성 방법, 전달할 배포 대상 플래그 등)를 전달받았습니다.

---

## 다음 단계

더 복잡한 에이전트 구축을 시도해 보세요:

- 툴 추가 — *"Add a Google Search tool so the caveman can grunt about current events"*
- 멀티 에이전트 — *"Create an A2A agent that other agents can talk to"* (`adk` 템플릿 사용 — A2A가 기본 내장되어 있음)
- RAG — *"Build an agent that answers questions from our docs"* (RAG 샘플 학습 — [에이전트 템플릿](templates.md#rag-retrieval-augmented-generation) 참고)

전체 옵션은 [에이전트 템플릿](templates.md)을 참고하거나, 전체 워크플로우를 보려면 [개발 가이드](development.md)로 이동하세요.
