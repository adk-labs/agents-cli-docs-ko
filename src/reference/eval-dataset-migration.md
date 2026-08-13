# 평가 데이터셋 마이그레이션

[Gemini Enterprise Agent Platform GenAI Eval SDK](https://docs.cloud.google.com/gemini-enterprise-agent-platform/optimize/evaluation/agent-evaluation)를 중심으로 평가 기능이 재구축되기 전에 `agents-cli`를 사용하기 시작했다면, 이전 ADK `EvalSet` 스키마를 사용하는 평가 파일이 `tests/eval/evalsets/` 아래에 남아 있을 수 있습니다. 이러한 파일은 `agents-cli eval generate` 및 관련 명령에서 더 이상 읽어오지 않습니다. 이 페이지에서는 이를 새로운 포맷으로 변환하는 방법을 안내합니다.

프로젝트에 `tests/eval/evalsets/` 디렉터리가 없다면 추가로 수행할 작업이 없습니다.

---

## 자동 마이그레이션

`agents-cli scaffold upgrade` 명령은 기존 `*.evalset.json` 파일을 감지하여 자동으로 새 포맷으로 변환합니다. 변환은 다음 규칙을 따릅니다: `tests/eval/datasets/` 아래에 새 파일을 작성하고, 이미 존재하는 대상 파일은 건너뛰며, 삭제 전 직접 검증할 수 있도록 기존 디렉터리를 그대로 유지합니다. `eval generate`는 다음 실행 시 실제 에이전트로부터 `agent_data.agents`를 채워 넣으므로, 마이그레이터가 스텁(stub)을 따로 작성하지는 않습니다.

수동으로 변환을 진행하려면 이 페이지의 나머지 부분을 참고하여 스키마 변경 사항을 확인하세요.

---

## 변경된 이유

새로운 평가 기능(`eval generate`, `eval grade`, `eval dataset synthesize`, `eval compare`, `eval analyze`, `eval metric list`, `eval optimize`)은 Gemini Enterprise Agent Platform GenAI Eval SDK의 `EvaluationDataset` / `EvalCase` 타입을 기반으로 구축되었습니다. 플랫폼 자체 스키마를 채택함으로써 `agents-cli`가 두 가지 서로 다른 데이터 형태 간을 브리지할 필요 없이 Agent Platform의 광범위한 평가 기능 세트(기본 제공 및 커스텀 메트릭, LLM-as-a-judge 채점, 데이터셋 합성, 회귀 비교, 실패 모드 분석, 프롬프트 최적화 등)를 그대로 활용할 수 있습니다.

---

## 한눈에 보는 변경 사항

| | 이전 (ADK `EvalSet`) | 신규 (Agent Platform `EvaluationDataset`) |
|---|---|---|
| 디렉터리 | `tests/eval/evalsets/` | `tests/eval/datasets/` |
| 파일명 | `*.evalset.json` | `*-dataset.json` |
| 기본 파일 | `basic.evalset.json` | `basic-dataset.json` |
| 스키마 소스 | `google.adk.evaluation` | `vertexai._genai.types.EvaluationDataset` |

`agents-cli eval generate`는 기본적으로 `tests/eval/datasets/basic-dataset.json`을 찾습니다. 다른 파일을 가리키려면 `--dataset PATH`를 사용하세요.

---

## 스키마 변경 사항

> **유효한 두 가지 입력 형태.** 새 포맷의 각 평가 케이스는 다음 중 **하나**를 제공해야 합니다:
>
> - **형태 A — 단일 프롬프트 케이스 (single-prompt case):** 최상위 `prompt` 필드 (단일 사용자 메시지). 케이스가 원샷(one-shot) 사용자 쿼리인 경우 사용합니다.
> - **형태 B — 대화 이어서 진행 케이스 (continued-conversation case, "N+1" 패턴):** 턴의 끝이 사용자 메시지로 끝나는 `agent_data` 블록. `agents-cli eval generate`가 그 뒤에 다음 에이전트 응답을 추가합니다.
>
> 이전 `EvalSet` 스키마의 *단일 턴* 케이스는 **형태 A**에 매핑됩니다. 이전 *다중 턴* 케이스는 **형태 B**에 매핑됩니다: 기록된 이전 턴들이 `agent_data.turns`가 되고, 에이전트가 응답하길 원하는 사용자 메시지로 끝납니다. 아래 섹션에서 두 가지 모두 설명합니다.

### 엔벨로프 (Envelope)

최상위 래퍼가 더 단순해졌습니다. `eval_set_id`, `name`, `description`은 제거되었으며 `eval_cases`만 남습니다.

**이전:**
```json
{
  "eval_set_id": "basic_eval",
  "name": "Basic Agent Evaluation",
  "description": "Sample evaluation set for testing core agent functionality.",
  "eval_cases": [ ... ]
}
```

**신규:**
```json
{
  "eval_cases": [ ... ]
}
```

### 단일 턴 케이스

케이스당 3가지 변경 사항:

- `eval_id` → `eval_case_id`.
- 첫 번째 턴의 `conversation[0].user_content`가 최상위 `prompt`로 승격되었습니다.
- `session_input`이 제거되었습니다. 에이전트 상태 초기화는 평가 데이터에 선언되는 대신 에이전트 코드(`app/agent.py`) 내부로 이동했습니다.

**이전:**
```json
{
  "eval_id": "greeting",
  "conversation": [
    {
      "user_content": {
        "parts": [{"text": "Hello, what can you help me with?"}]
      }
    }
  ],
  "session_input": {
    "app_name": "app",
    "user_id": "eval_user",
    "state": {}
  }
}
```

**신규:**
```json
{
  "eval_case_id": "greeting",
  "prompt": {
    "role": "user",
    "parts": [{"text": "Hello, what can you help me with?"}]
  }
}
```

프롬프트에 `"role": "user"`가 추가된 점에 유의하세요. 이는 Agent Platform `Content` 타입에서 필수 항목입니다.

### 다중 턴 케이스 (형태 B)

이전 스키마에서 다중 턴 대화는 `conversation` 아래 턴들의 리스트였습니다. 새 스키마에서는 **형태 B**로 매핑됩니다: 이전 턴들은 `agent_data.turns` 아래에 위치하고, 해당 이력의 마지막 사용자 메시지에 대해 `eval generate`가 응답을 생성합니다 (별도의 최상위 `prompt` 없음).

`agent_data.turns[].events` 아래의 각 항목은 `author`(`"user"` 또는 `agent_data.agents`에 선언된 에이전트 ID 중 하나)와 `content`(사용자 턴의 경우 `role: "user"`, 에이전트 턴의 경우 `role: "model"`)를 갖는 이벤트입니다.

**이전 (2개 턴 대화):**
```json
{
  "eval_id": "follow_up",
  "conversation": [
    {
      "user_content": {
        "parts": [{"text": "Book a flight to Paris."}]
      },
      "final_response": {
        "parts": [{"text": "What dates are you flying?"}]
      }
    },
    {
      "user_content": {
        "parts": [{"text": "Next Monday, returning Friday."}]
      }
    }
  ]
}
```

**신규 (형태 B):**
```json
{
  "eval_case_id": "follow_up",
  "agent_data": {
    "agents": {
      "flight_booker": {
        "agent_id": "flight_booker",
        "agent_type": "llm_agent",
        "description": "Books flights and answers itinerary questions.",
        "instruction": "Help the user book flights. Ask clarifying questions about dates, origin, and passenger count before calling any booking tool.",
        "tools": [
          {
            "function_declarations": [
              {"name": "search_flights", "description": "Search available flights."},
              {"name": "book_flight",    "description": "Book a flight by ID."}
            ]
          }
        ],
        "sub_agents": []
      }
    },
    "turns": [
      {
        "turn_index": 0,
        "events": [
          {
            "author": "user",
            "content": {
              "role": "user",
              "parts": [{"text": "Book a flight to Paris."}]
            }
          },
          {
            "author": "flight_booker",
            "content": {
              "role": "model",
              "parts": [{"text": "What dates are you flying?"}]
            }
          },
          {
            "author": "user",
            "content": {
              "role": "user",
              "parts": [{"text": "Next Monday, returning Friday."}]
            }
          }
        ]
      }
    ]
  }
}
```

> **`agent_data.agents`에 대하여.** `agents` 맵은 평가 대상 에이전트 시스템의 토폴로지를 선언합니다. 에이전트 ID를 키로 사용하며, 각 항목은 해당 에이전트의 설정(`agent_type`, `description`, `instruction`, `tools`(에이전트가 호출할 수 있는 함수 선언, `google.genai.types.Tool`에서 사용하는 것과 동일한 포맷), `sub_agents`)을 담고 있습니다. 각 이벤트의 `author`는 `"user"`이거나 이 맵에 존재하는 에이전트 ID 중 하나여야 하며, 채점 시 멀티 에이전트 시스템이 응답과 도구 호출을 올바른 서브 에이전트에 귀속시키는 기준이 됩니다. `tools` 블록은 채점기가 에이전트가 타당한 인자와 함께 올바른 도구를 선택했는지 검증할 수 있게 하므로, 에이전트에 호출 가능한 도구가 있는 경우 반드시 포함해야 합니다. 단일 에이전트 프로젝트의 경우 위에 표시된 것처럼 하나의 항목만 선언할 수 있고, 멀티 에이전트 시스템의 경우 각 에이전트를 나열하고 `sub_agents`를 사용하여 토폴로지를 표현합니다. 여기에 표시된 값은 예시이므로 본인의 프로젝트에 맞게 수정하세요.

`eval generate`는 이 이력을 바탕으로 에이전트를 실행하고 해당 응답을 다음 에이전트 이벤트로 추가하여 `eval grade`에 사용할 수 있는 트레이스를 생성합니다.

이전 케이스에서 정답(gold answer)을 표현하기 위해 **마지막** 턴(채점 대상 턴)에 `final_response`가 설정되어 있었다면, 이는 다른 개념입니다 — `agent_data.turns`에 섞어 넣는 대신 최상위 `reference` 필드에 기술하세요. 과거의 실제 응답은 턴 이력에 들어가고, 마지막 사용자 메시지에 대한 목표 정답은 `reference`에 들어갑니다.

---

## 단계별 변환 절차

단일 파일 `tests/eval/evalsets/basic.evalset.json`의 경우:

1. 새 디렉터리 생성: `mkdir -p tests/eval/datasets`
2. 파일 복사: `cp tests/eval/evalsets/basic.evalset.json tests/eval/datasets/basic-dataset.json`
3. 에디터에서 `tests/eval/datasets/basic-dataset.json` 열기.
4. 최상위에서 `eval_set_id`, `name`, `description` 삭제.
5. `eval_cases` 아래의 각 항목에 대해 `eval_id`를 `eval_case_id`로 이름을 변경한 후 **형태 A** 또는 **형태 B** 선택:
    - **단일 턴 (형태 A):** 유일한 턴의 `user_content`를 최상위 `prompt`로 이동하고 `"role": "user"`를 추가합니다. `conversation` 배열과 `session_input` 블록을 삭제합니다.
    - **다중 턴 (형태 B):** `agent_data.agents`(에이전트 ID 대 해당 `AgentConfig` 맵)에 에이전트 토폴로지를 선언한 다음, 에이전트가 응답하길 원하는 사용자 메시지가 마지막 항목이 되도록 `agent_data.turns[0].events` 목록을 작성합니다. 각 이전 턴의 `user_content`를 `author: "user"`(`role: "user"`)인 이벤트로 변환하고, 기록된 에이전트 응답을 해당 에이전트 ID가 `author`인 이벤트(`role: "model"`)로 변환합니다. `conversation` 배열과 `session_input` 블록을 삭제합니다. 형태 B에서는 최상위 `prompt`를 설정하지 **않습니다**.
6. 저장 후 `agents-cli eval generate`로 검증 — 파일이 자동으로 검색되어야 합니다.

모든 것이 정상 작동하면 이전 `tests/eval/evalsets/` 디렉터리를 삭제하세요.

### 다중 파일

각 `*.evalset.json`에 대해 위 단계를 반복합니다. `tests/eval/datasets/` 내의 파일명은 `*-dataset.json` 컨벤션을 따라야 합니다 (예: `flight_booking.evalset.json` -> `flight_booking-dataset.json`).

---

## 검증

```bash
agents-cli eval generate
```

기본 이름(`basic-dataset.json`)을 사용한 경우 `eval generate`가 이를 자동으로 선택합니다. 다른 파일명의 경우:

```bash
agents-cli eval generate --dataset tests/eval/datasets/your-file-dataset.json
```

성공적으로 실행되면 `agents-cli eval grade`에 전달할 수 있는 트레이스 파일이 작성됩니다.
