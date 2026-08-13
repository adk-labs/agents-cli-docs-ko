# 관측 가능성 (Observability)

모든 `agents-cli` 프로젝트에는 OpenTelemetry 계측(instrumentation)이 포함되어 있어 트레이스(trace)를 **Cloud Trace**로 자동 내보냅니다. 이를 통해 다음을 얻을 수 있습니다:

- **분산 트레이싱 (Distributed tracing)** — LLM 호출 및 도구 실행을 통과하는 요청의 흐름을 추적합니다.
- **지연 시간 분석 (Latency analysis)** — 스팬(span) 지속 시간을 분석하여 성능 병목 현상을 식별합니다.
- **오류 시각화 (Error visibility)** — 트레이스가 오류를 캡처하여 오류 발생 위치를 정확히 짚어냅니다.
- **별도 설정 불필요** — 모든 환경에서 추가 설정 없이 바로 작동합니다.

ADK 기반 에이전트의 경우, **프롬프트-응답 로깅**이 전체 모델 상호작용(프롬프트, 응답, 토큰)을 캡처하여 **GCS**(JSONL) 및 **BigQuery**의 `completions` 테이블로 업로드합니다. 로그 버킷이 설정되어 있으면(`LOGS_BUCKET_NAME` + `OTEL_INSTRUMENTATION_GENAI_*` 업로드 변수) 항상 활성화되며, Terraform으로 프로비저닝된 배포에서는 기본적으로 이 작업이 수행됩니다.

> **두 개의 독립적인 계층.** 프롬프트-응답 로깅(GCS/BigQuery completions)은 **전체 콘텐츠**를 캡처합니다. 콘텐츠가 **Cloud Trace 스팬 / Cloud Logging 이벤트**에도 나타날지 여부는 `OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT`(기본값 `NO_CONTENT` — 트레이스/이벤트에 콘텐츠 **제외**) 및 `ADK_CAPTURE_MESSAGE_CONTENT_IN_SPANS=false`에 의해 별도로 제어됩니다. 따라서 기본 설정은 다음과 같습니다: **GCS/BigQuery에는 전체 콘텐츠 포함, 트레이스에는 콘텐츠 제외.**

### 환경별 로깅 동작

| 환경 | Cloud Trace 스팬 | 프롬프트-응답 로깅 (GCS/BigQuery) |
|---|---|---|
| **로컬** (`agents-cli playground`) | 활성화됨, 콘텐츠 없음 | 꺼짐 (`LOGS_BUCKET_NAME` 없음) |
| **배포 환경** (Terraform 프로비저닝) | 활성화됨, 콘텐츠 없음 | **켜짐 — 전체 프롬프트/응답** |
| **배포 환경** (단순 `agents-cli deploy`, 버킷 없음) | 활성화됨, 콘텐츠 없음 | 꺼짐 (`LOGS_BUCKET_NAME` 없음) |

---

## Cloud Trace

기본으로 제공되는 관측 가능성 메서드입니다. 설정 및 사용 방법은 [Cloud Trace](cloud-trace.md)를 참고하세요.

---

## BigQuery Agent Analytics

고급 분석(대화 전반의 패턴 쿼리, 토큰 사용량 대시보드, 프로덕션 트래픽에 대한 LLM-as-a-judge 채점 등)을 위해 사용합니다. 프로젝트 생성 시 `--bq-analytics` 플래그를 사용하여 옵트인(opt-in)할 수 있습니다.

자세한 내용은 [BigQuery Agent Analytics](bq-agent-analytics.md)를 참고하세요.
