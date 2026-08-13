# 평가 가이드

구조화된 평가를 실행하여 에이전트가 올바른 도구를 호출하고, 고품질 응답을 생성하며, 예외 상황(edge cases)을 잘 처리하는지 확인하세요. 내부적으로 평가는 [Gemini Enterprise Agent Platform GenAI Eval SDK](https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/evaluation/agent-evaluation)를 사용하여 평가 채점을 진행합니다.

!!! note "이전 버전의 agents-cli에서 업그레이드하셨나요?"
    프로젝트에 이전 버전의 `tests/eval/evalsets/*.evalset.json` 파일이 여전히 남아 있다면, 새로운 포맷으로 전환하는 방법은 [평가 데이터셋 마이그레이션](../reference/eval-dataset-migration.md) 문서를 참고하세요.

---

## 첫 번째 평가 실행하기

프로젝트에는 `tests/eval/datasets/basic-dataset.json`의 기본 데이터셋과 `tests/eval/eval_config.yaml`의 메트릭 설정이 포함되어 있습니다. 다음 명령으로 실행하세요:

```bash
agents-cli eval generate
agents-cli eval grade
```

출력 결과에는 설정된 메트릭에 대한 각 평가 케이스별 점수가 표시됩니다.

```bash
# 커스텀 데이터셋 및 다른 메트릭으로 실행
agents-cli eval generate --dataset tests/eval/datasets/custom-dataset.json --output custom_traces/
agents-cli eval grade --metrics general_quality --traces custom_traces/

# 로컬에서 실행하는 대신 배포된 에이전트 평가
agents-cli eval generate --url https://my-agent.run.app --app-name app
agents-cli eval grade
```

---

## 평가 케이스 작성 및 메트릭 선택

평가 케이스 스키마 및 사용 가능한 메트릭에 대한 자세한 문서는 [Gemini Enterprise Agent Platform Evaluation 문서](https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/evaluation/agent-evaluation)를 참고하세요.

### 사용 가능한 메트릭 레퍼런스

에이전트의 기능과 수행할 작업에 따라 다양한 기본 제공(built-in) 메트릭 중 선택할 수 있습니다. 사용 가능한 전체 메트릭 목록을 보려면 다음을 실행하세요:

```bash
agents-cli eval metric list
```

#### 자주 사용하는 메트릭 한눈에 보기

가장 많이 사용되는 기본 제공 메트릭 ID에 대한 요약 참고 자료입니다. 설명이 포함된 전체 목록은 `agents-cli eval metric list`를 사용하세요.

| 메트릭 ID | 채점 항목 |
|---|---|
| `general_quality` | 자동 생성된 콘텐츠 기반 기준을 바탕으로 한 전체 응답 품질. 비-에이전트 평가를 위한 추천 시작점. |
| `text_quality` | 언어적 측면: 유창성(fluency), 일관성(coherence), 문법(grammar). |
| `instruction_following` | 응답이 특정 제약 조건과 지침을 얼마나 잘 준수하는지. |
| `tool_use_quality` | 도구 선택, 매개변수 정확도 및 단계 순서의 적절성 (단일 턴). |
| `multi_turn_tool_use_quality` | 다중 턴 대화 전반에 걸친 도구 호출의 기술적 및 의미적 정확성. |
| `multi_turn_trajectory_quality` | 턴 전반에 걸친 순차적 논리, 효율성 및 오류 복구 견고성. |
| `multi_turn_task_success` | 전체 다중 턴 대화를 통해 사용자의 목표가 달성되었는지 여부. |
| `final_response_quality` | 최종 응답 및 중간 도구 사용에 대한 종합적인 평가. |
| `final_response_reference_free` | 레퍼런스 정답 없이 최종 응답 품질 평가 (커스텀 루브릭 필요). |
| `final_response_match` | 에이전트의 최종 응답을 제공된 정답(golden reference answer)과 비교. |
| `hallucination` | 응답을 원자 단위 주장으로 분할하고, 각 주장을 도구에서 반환된 컨텍스트와 검증. |
| `grounding` | 제공된 컨텍스트 대비 사실성 및 일관성. |
| `safety` | 안전 정책 준수 여부 (PII, 혐오 발언, 위험한 콘텐츠, 괴롭힘, 성적 콘텐츠). |

### 평가 설정 (`eval_config.yaml`)

`eval_config.yaml` 파일은 실행할 메트릭을 지정하고 평가 채점을 위한 커스텀 메트릭을 정의합니다.

```yaml
metrics_to_run:
  - response_under_500_chars

custom_metrics:
  - name: response_under_500_chars
    custom_function: |
      def evaluate(instance: dict) -> dict:
          response = instance.get("response") or {}
          text = "".join(
              p.get("text", "") for p in (response.get("parts") or []) if p.get("text")
          )
          passed = len(text) <= 500
          return {
              "score": 1.0 if passed else 0.0,
              "explanation": f"Final response is {len(text)} chars (limit 500).",
          }
  - name: response_quality_rubric
    prompt_template: |
      Rate the agent's response 1-5 for helpfulness and accuracy.
      Prompt: {prompt}
      Final response: {response}
      Full trace (for tool-call and reasoning context): {agent_data}
      Return JSON: {"score": <1|2|3|4|5>, "explanation": "<reason>"}
    judge_model: gemini-3.6-flash
    judge_model_sampling_count: 3
```

각 커스텀 메트릭은 **Code Execution Metric** 또는 **LLM-as-a-Judge Metric**(`LLMMetric`) 스키마 중 하나를 따라야 합니다:
- **Code Execution Metric**: 평가를 위해 커스텀 Python 코드를 실행할 때 사용합니다. `name`과 `custom_function`(`def evaluate(instance):` 시그니처 포함)이 포함되어야 합니다. 기본적으로 해당 함수는 **CLI 프로세스에서 로컬로** 실행되므로 GCP 프로젝트나 리전이 필요하지 않지만, 사용자 지정 코드가 CLI의 권한으로 실행됩니다. Vertex AI의 샌드박스 환경인 `CodeExecutionMetric`(서버 측)을 사용하려면 `"execution": "remote"`를 추가해야 하며, 이 경우 설정된 GCP 프로젝트와 리전이 필요합니다.
- **LLM-as-a-Judge Metric**: LLM 저지를 사용하여 응답을 평가할 때 사용합니다. `name`과 `prompt_template`이 포함되어야 합니다. 선택적 필드로는 `rubric_group_name`, `judge_model`(예: `gemini-3.6-flash`), `judge_model_sampling_count`(`1`에서 `32` 사이)가 있습니다.

### 주요 시나리오별 빠른 참조

- **커스텀 함수 도구를 사용하는 에이전트** — `tool_use_quality`(단일 턴용) 또는 `multi_turn_tool_use_quality` + `multi_turn_trajectory_quality`(다중 턴용)를 사용하세요.
- **RAG 에이전트** — `grounding` + `hallucination` + `safety`를 사용하세요.
- **대화형 어시스턴트** — `general_quality` 또는 `multi_turn_general_quality`를 사용하세요.
- **목표 지향적 에이전트** — `multi_turn_task_success`를 사용하세요.

---

## 평가-수정 루프 (Eval-Fix Loop)

평가는 반복적인 과정입니다. 에이전트가 일관되게 통과하기까지 5~10회 이상의 사이클을 거치는 것이 일반적입니다.

1. **1~2개의 핵심 평가 케이스 작성**: 가장 중요한 동작을 아우르는 케이스를 작성합니다.
2. **실행**: `agents-cli eval generate` 실행 후 `agents-cli eval grade` 실행.
3. **결과 확인**: 어떤 케이스가 실패했는지, 왜 실패했는지 분석합니다.
4. **수정**: 에이전트의 지침(instruction), 도구 또는 로직을 조정합니다.
5. **재실행**: `agents-cli eval generate` 및 `agents-cli eval grade` 재실행.
6. **확장**: 핵심 케이스가 통과하면 예외 상황과 새로운 시나리오를 추가합니다.

---

## generate 및 grade 그 이상

`generate`와 `grade`가 이너 루프(inner loop)를 구성하지만, 평가 기능에는 알아두면 유용한 추가 명령들이 있습니다. 각 명령은 평가 설정이 성숙해짐에 따라 단계별로 활용할 수 있습니다.

### `agents-cli eval dataset synthesize`

로컬 ADK 에이전트를 검사하여 입력을 위한 별도의 파일 없이도 다중 턴 대화 시나리오를 자동 생성해 데이터셋을 초기화합니다. 새로운 에이전트에 대한 평가를 빠르게 시작하거나 모든 케이스를 수동으로 작성하지 않고 커버리지를 확장할 때 유용합니다. 생성된 각 케이스에는 시작 사용자 메시지, 대화 계획, LLM 기반 사용자 시뮬레이터를 상대로 시나리오를 진행하여 생성된 전체 에이전트 트레이스가 포함됩니다.

```bash
agents-cli eval dataset synthesize --count 10
```

`--instruction` (예: `"사용자가 마음을 바꾸는 시나리오"`) 및 `--environment-context` (예: `"오늘은 월요일입니다. 파리행 항공편이 있습니다."`)를 통해 생성되는 내용을 제어할 수 있습니다. 출력 결과는 수정 및 커밋이 가능한 일반 `*-dataset.json` 파일이며, `eval grade`에 바로 전달할 수 있습니다(트레이스가 이미 채워져 있으므로 `eval generate` 단계를 건너뛸 수 있습니다).

### `agents-cli eval compare`

두 개의 채점 결과를 나란히 비교하여 변경 사항이 실제로 개선을 이끌어냈는지 확인할 수 있습니다.

```bash
agents-cli eval compare baseline_results.json candidate_results.json
```

평가-수정 루프 중에 "수정 전" 실행 결과와 "수정 후" 실행 결과를 비교할 때 주로 사용됩니다.

### `agents-cli eval analyze`

채점 결과 파일의 실패 모드를 주제별로 클러스터링하여 개별 케이스를 일일이 훑어보는 대신 어떤 종류의 문제가 발생하고 있는지 확인할 수 있습니다.

```bash
agents-cli eval analyze --eval-result grade_results.json
```

### `agents-cli eval metric list`

SDK가 지원하는 모든 기본 제공 메트릭과 간단한 설명을 출력합니다. 위의 주요 메트릭 테이블 외에 추가로 사용할 수 있는 메트릭을 확인하고 싶을 때 시작하는 지점입니다.

### `agents-cli eval optimize`

평가 설정이 준비되면, `eval optimize`는 해당 평가를 활용해 에이전트의 프롬프트를 자동으로 튜닝합니다.

```bash
agents-cli eval optimize
```

실행에는 데이터셋 크기와 메트릭 복잡도에 따라 수 분에서 수 시간이 소요되므로 반복해서 실행할 명령은 아닙니다. 더 간단한 접근 방식(프롬프트 직접 재작성, 메트릭 조정, 실패한 케이스 수동 수정 등)을 충분히 시도한 후에 활용하세요.

---

평가 케이스 스키마, 메트릭 및 사용자 시뮬레이션에 대한 자세한 문서는 [Gemini Enterprise Agent Platform Evaluation 문서](https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/evaluation/agent-evaluation)를 참고하세요.
