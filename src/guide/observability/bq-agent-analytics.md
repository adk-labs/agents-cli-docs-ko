# BigQuery Agent Analytics 플러그인

*에이전트 동작, 토큰 사용량 및 대화 패턴에 대한 SQL 기반 분석을 원하는 팀을 위한 가이드입니다.*

## 개요

BigQuery Agent Analytics 플러그인은 상세한 에이전트 이벤트를 BigQuery에 직접 기록하여 확장된 관측 가능성을 제공합니다. 이를 통해 시간이 흐름에 따른 에이전트의 동작, 상호작용 및 성능에 대해 풍부한 SQL 기반 분석을 수행할 수 있습니다.

이 기능은 **옵트인(opt-in)** 기능이며, **ADK 기반 에이전트**에서만 사용할 수 있습니다.

---

## 사용 시기

다음과 같은 경우 이 플러그인을 활성화하세요:

*   **BigQuery의 고급 LLM 기능 활용**: `AI.Search`, `AI.Score`, `AI.Generate_text`를 사용하여 대화를 의미론적으로 그룹화하고, 순위를 매기며, 오류를 식별하거나 LLM-as-a-judge 평가를 수행하는 등 에이전트의 시맨틱 분석에 적용하려는 경우.
*   **BigQuery 대화형 분석 활용**: 다른 대화형 에이전트를 사용하여 복잡한 SQL 쿼리를 직접 작성할 필요 없이 에이전트를 분석하려는 경우.
*   **커스텀 대시보드 및 리포트 생성**: 에이전트 성능, 도구 사용량, 토큰 소비량에 대한 대시보드 및 보고서를 작성하려는 경우.
*   **구조화된 쿼리 가능 이력 보존**: 감사, 파인 튜닝 또는 다른 비즈니스 데이터와의 결합을 위해 에이전트 이벤트 이력을 저장하려는 경우.

항상 켜져 있는 [Cloud Trace 텔레메트리](cloud-trace.md)와 비교했을 때, 이 플러그인은 오프라인 분석을 위해 설계된 구조화된 테이블 형태로 보다 세분화된 데이터를 제공합니다.

---

## 사전 요구사항

*   ADK 기반 템플릿(예: `adk`)으로 생성된 에이전트 프로젝트.
*   `google-adk` 버전 `>=1.21.0` (플러그인을 활성화할 때 자동으로 추가됨).
*   BigQuery API 및 BigQuery Storage API가 활성화된 Google Cloud 프로젝트 (일반적으로 Terraform에 의해 처리됨).

---

## 플러그인 활성화하기

프로젝트를 생성할 때 `--bq-analytics` 플래그를 사용하세요:

```bash
agents-cli create my-agent \
  -a adk \
  -d cloud_run \
  --bq-analytics
```

이 플래그는 `app/agent.py`에 플러그인 초기화 코드를 포함시키고 Terraform에 환경 변수를 설정합니다.

---

## 설정

플러그인은 `app/agent.py` 파일에서 설정합니다:

```python
from google.adk.plugins.bigquery_agent_analytics_plugin import (
    BigQueryAgentAnalyticsPlugin,
    BigQueryLoggerConfig,
)

bq_config = BigQueryLoggerConfig(
    enabled=True,
    gcs_bucket_name=os.environ.get("BQ_ANALYTICS_GCS_BUCKET"),
    connection_id=os.environ.get("BQ_ANALYTICS_CONNECTION_ID"),
    log_multi_modal_content=True,
    max_content_length=500 * 1024,
    table_id="agent_events",
)

bq_analytics_plugin = BigQueryAgentAnalyticsPlugin(
    project_id=os.environ.get("GOOGLE_CLOUD_PROJECT"),
    dataset_id=os.environ.get("BQ_ANALYTICS_DATASET_ID", "adk_agent_analytics"),
    table_id=bq_config.table_id,
    config=bq_config,
    location=os.environ.get("GOOGLE_CLOUD_LOCATION", "US"),
)

app = App(
    name="my-agent",
    root_agent=root_agent,
    plugins=[bq_analytics_plugin],
)
```

**주요 `BigQueryLoggerConfig` 옵션:**

*   **`enabled`**: 플러그인 활성화/비활성화 스위치.
*   **`gcs_bucket_name`** (선택 사항): 대용량/바이너리 콘텐츠 오프로드를 위한 GCS 버킷. 멀티모달 데이터에만 필요합니다.
*   **`connection_id`** (선택 사항): GCS 접근을 위한 BigQuery Connection ID. 멀티모달 데이터에만 필요합니다.
*   **`log_multi_modal_content`**: 콘텐츠 파트를 처리하고 GCS로 오프로드할지 여부.
*   **`max_content_length`**: GCS로 텍스트를 오프로드하는 임계값.
*   **`table_id`**: BigQuery 테이블 이름 (기본값: `agent_events`).
*   **`event_allowlist`** / **`event_denylist`**: 기록할 이벤트 유형 필터링.
*   **`batch_size`**: 쓰기 전 배치 처리할 행 수.

---

## 인프라

Terraform(`agents-cli infra single-project`)으로 배포할 때:

*   **데이터셋:** `{project_name}_telemetry`라는 이름의 BigQuery 데이터셋이 생성됩니다.
*   **GCS 버킷** (선택 사항): 콘텐츠 오프로드용 `{project_id}-{project_name}-logs`.
*   **BigQuery 커넥션** (선택 사항): BigQuery에서 GCS로 접근하기 위한 `{project_name}-genai-telemetry`.
*   **테이블:** `agent_events` 테이블은 첫 번째 이벤트 발생 시 플러그인에 의해 **자동 생성**됩니다.

---

## 쿼리 예시

본인의 `YOUR_PROJECT_ID` 및 `YOUR_AGENT_NAME`으로 적절히 교체하세요.

**최근 이벤트:**

```sql
SELECT *
FROM `YOUR_PROJECT_ID.YOUR_AGENT_NAME_telemetry.agent_events`
ORDER BY timestamp DESC
LIMIT 100;
```

**도구 호출 및 오류:**

```sql
SELECT
  timestamp,
  JSON_VALUE(content, '$.tool') AS tool_name,
  JSON_VALUE(content, '$.args') AS tool_args,
  status,
  error_message
FROM `YOUR_PROJECT_ID.YOUR_AGENT_NAME_telemetry.agent_events`
WHERE event_type IN ('TOOL_COMPLETED', 'TOOL_ERROR')
ORDER BY timestamp DESC;
```

**LLM 토큰 사용량:**

```sql
SELECT
  agent,
  JSON_VALUE(attributes, '$.model') AS model,
  SUM(CAST(JSON_VALUE(attributes, '$.usage_metadata.prompt') AS INT64)) AS total_prompt_tokens,
  SUM(CAST(JSON_VALUE(attributes, '$.usage_metadata.completion') AS INT64)) AS total_completion_tokens
FROM `YOUR_PROJECT_ID.YOUR_AGENT_NAME_telemetry.agent_events`
WHERE event_type = 'LLM_RESPONSE'
  AND JSON_VALUE(attributes, '$.usage_metadata.prompt') IS NOT NULL
GROUP BY agent, model;
```
