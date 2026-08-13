# 릴리스 노트

이 프로젝트의 주요 변경 사항은 이 문서에 기록됩니다.

## [1.3.1] - 2026-08-04

- agent-plugins.org 플러그인 매니페스트 추가
- `infra cicd` 명령어를 위한 지능형 백오프(backoff) 처리 개선
- `setup` 실행 중 불필요한 에러 메시지를 방지하기 위해 `npx skills` 1.5.9 버전으로 롤백
  - https://github.com/google/agents-cli/issues/59

## [1.3.0] - 2026-07-31

- 프로젝트 전체에서 A2A 1.0을 사용하도록 업데이트되었으며, A2A 0.3 클라이언트를 위한 호환성 레이어 포함
- Cloud Run 배포에 대한 재시도 로직 개선
- `infra cicd` 명령어의 재시도 처리 효율화
- `setup` 및 `update` 중 `npx skills` 출력 포맷팅 수정
- Agent Runtime 배포 시작 시 Cloud Logging URL 출력
- `eval generate`/`run`에 `--concurrency` 및 `--header` 플래그 추가
- `agent_runtime`과 함께 사용될 때 `--session-type` 관련 메시지 조정

## [1.2.1] - 2026-07-23

- 철회(yanked)된 `opentelemetry-resourcedetector-gcp` 버전으로 인해 `uv sync` 시 발생하던 임포트 문제 수정
- `--url`을 사용하는 `eval generate`용 선택적 HTTP 기반 경로 추가 (진행 중인 기능)

## [1.2.0] - 2026-07-21

- **클라우드 텔레메트리가 CLI 및 배포 전반에 걸쳐 ADK의 `otel_to_cloud`로 이동되었습니다.** `playground` 및 `run`에 ADK의 기존 `--otel_to_cloud`를 전달하는 `--otel-to-cloud` 플래그가 추가되었습니다. 기존 `--trace-to-cloud`는 숨겨진 호환용 별칭으로 유지되며 사용 시 경고가 출력됩니다. 배포 측면에서 Agent Runtime은 이제 `otel_to_cloud`를 통해 내보내며(`GOOGLE_CLOUD_AGENT_ENGINE_ENABLE_TELEMETRY`로 제어), 생성된 프로젝트는 런타임 대신 Terraform에서 선언적으로 클라우드 텔레메트리를 구성합니다. 번들로 제공되는 `google-adk`가 `otel-gcp` 엑스트라와 함께 `>=2.2.0`으로 이동되었으며 관찰 가능성(observability) 스킬도 이에 맞춰 업데이트되었습니다.
- 스캐폴드의 기본 모델이 `gemini-3.6-flash`로 변경되었습니다.
- **Agent Analytics가 이제 Agent Runtime에서 GenAI 완성을 수집합니다.** BigQuery 텔레메트리 로그 싱크가 배포 대상별로 구성되어, Agent Runtime 배포의 GenAI 요청/응답 로그가 완성(completions) 뷰를 비워두는 대신 BigQuery로 라우팅됩니다.
- **ADK 코드 스킬에 매니지드 에이전트(Managed Agents) 지침이 추가되었습니다.** 번들 ADK 치트시트에 `ManagedAgent` 사용 시기, Gemini API와 Agent Platform (GEAP) 설정 차이, 실행 가능한 생성 및 사용 예제를 다루는 매니지드 에이전트 섹션이 추가되었습니다.

## [1.1.0] - 2026-07-10

- **새로운 에이전트를 위한 가이드형 브레인스토밍 기능.** 워크플로우 스킬의 Phase 0이 이제 대화형 브레인스토밍 대화로 동작하여 코드 작성 전에 에이전트의 스펙을 구체화하도록 돕고, 질의할 수 없을 때 검토를 위해 가정을 명시합니다.
- `eval generate` 및 `eval grade` 실행 시 결과에 영향을 주지 않던 정상적인 서드파티 경고 및 진행률 표시줄로 출력이 더러워지지 않도록 수정되었습니다. 실패 시에는 해당 출력이 그대로 표시되며 디버깅을 위해 다시 활성화할 수 있습니다.
- 생성된 프로젝트가 이제 기본 스캐폴드 평가 메트릭 모듈 이름을 구현하는 메트릭에 맞춰 `tests/eval/response_quality.py`(기존 `metrics.py`)로 지정합니다.
- CLI 전반에 걸친 광범위한 Windows 호환성 수정.
- 사용자 대상 힌트의 명령어 이름 오타를 수정하여 올바른 `agents-cli` 명령어를 가리키도록 변경.
- 번들 스킬 갱신 (RAG 샘플이 `google/adk-samples`의 `core/python/`을 가리키도록 수정 포함).

## [1.0.0] - 2026-06-30

**Agents CLI가 1.0 정식 버전(GA)으로 출시되었습니다.** Google Cloud 상에서 ADK 에이전트를 스캐폴딩, 평가 및 배포하기 위한 안정적이고 프로덕션 환경에 적합한 첫 번째 GA 릴리스입니다.

- 재배포 시 미지정 설정을 초기화하는 대신 Agent Runtime 및 Cloud Run의 기존 배포 스펙을 유지합니다.
- Agent Runtime 배포 시 소스 패키징에서 `.gcloudignore` 및 `.gitignore`를 준수하여 업로드에 무시된 파일이 포함되지 않도록 변경되었습니다.
- RAG가 이제 복사 및 학습용 레시피 형태로 제공됩니다. `google/adk-samples`의 `rag-vector-search` / `rag-agent-search` 샘플(워크플로우 스킬을 통해 확인 가능)부터 시작할 수 있습니다. `agentic_rag` 템플릿, `--datastore` 플래그, `infra datastore` / `data-ingestion` 명령어는 제거되었으며 리다이렉션 안내를 출력합니다.
- 생성된 프로젝트가 이제 Python 환경 설정을 단일 템플릿 `.env` 파일로 통합합니다.
- 평가 명령어 실행 시 평가 메트릭 메타데이터를 내부 검사할 때 ADK 툴셋을 허용하도록 개선되어, 툴셋을 사용하는 에이전트가 메타데이터 수집에 실패하지 않습니다.
- GKE Cloud Build 배포가 이제 로그 스트리밍 제한에 강건하게 동작하며 빌드 로그 스트림이 잘려도 실패하지 않습니다.
- 번들 스킬 갱신: RAG 샘플이 adk-samples `main` 브랜치의 `core/`를 가리키도록 수정되었고, 상시 활성화 워크플로우 스킬이 일반화 및 정돈되었으며, ADK 코드 지침에 Agent Runtime 디버깅을 위한 `streaming_agent_run_with_events`가 명시되었습니다.

## [0.6.1] - 2026-06-28

- `publish gemini-enterprise`가 이제 기본적으로 ADK를 통해 Agent Runtime 배포를 등록하며, Gemini Enterprise가 이를 네이티브하고 안정적으로 호출합니다. Cloud Run 및 GKE의 경우 A2A 등록이 기본값으로 유지되며, Agent Runtime에서 A2A를 요청할 경우 경고와 함께 ADK를 권장합니다. A2A 에이전트를 재게시해도 더 이상 중복 등록이 생성되지 않으며, 첫 배포 시 A2A 에이전트 카드가 올바른 공개 URL을 전달합니다.
- `agents-cli update` 실행 중 스킬 업데이트 실패 시, 오해의 소지가 있는 초록색 "Skills updated." 배너를 표시하는 대신 비정상 종료 코드(non-zero)를 반환하고 에러를 명확히 보고하도록 수정되었습니다. 또한 Windows PowerShell에서 실패 메시지가 잘못된 색상으로 렌더링되던 문제를 수정했습니다.
- 모든 템플릿 에이전트에 대해 생성된 프로젝트의 `uv.lock` 파일을 갱신하고 번들된 `google-adk`를 2.2.0에서 2.3.0으로 업데이트했습니다.

## [0.6.0] - 2026-06-23

- Agent Runtime 배포가 이제 단일 통합 컨테이너 앱에서 ADK 웹, A2A 및 추론 엔진을 제공합니다.
- Cloud Trace 스팬이 더 이상 LLM 프롬프트 및 응답을 캡처하지 않아 민감한 콘텐츠가 트레이스에 포함되지 않습니다.
- 번들 스킬 갱신: 정확성 수정, 중복 제거, 더 가벼워진 상시 활성화 워크플로우 가이드 제공, ADK 코드 치트시트에 a2ui 문서화.

## [0.5.1] - 2026-06-18

- Windows 환경에서의 run 및 playground 명령어 오류 수정
  - https://github.com/google/agents-cli/issues/34
  - https://github.com/google/agents-cli/issues/35
  - 이를 발견하고 보고해 주신 @Abdullah-k0de 님께 감사드립니다!
- 실패 조사 가이드의 오래된 GCS 버킷 경로 수정
- publish 스킬에 Agent Registry 플릿 관리 추가

## [0.5.0] - 2026-06-15

- `deploy` 실행 시 Agent Runtime 및 Cloud Run의 머신 사양 파라미터를 플래그로 제공합니다.
- `deploy`에 `--service-name` 오버라이드 플래그가 추가되었습니다.
- `run` 실행 시 세션 푸터에 복사하여 붙여넣을 수 있는 재개(resume) 명령어를 출력합니다.
- `run` 실행 시 일반 실행에서는 재사용된 로컬 서버를 종료하지 않습니다.
- `scaffold upgrade`가 이제 `uvx`를 통해 이전 버전 템플릿을 빌드합니다.
- 대용량 `npx` 출력 시 스킬 설정/업데이트가 중단되던 파이프 버퍼 교착 상태(deadlock) 이슈를 수정했습니다.
- 프로젝트 루트 안내 메시지가 실제로 디렉토리가 변경될 때만 출력되도록 수정했습니다.
- 번들 스킬 및 생성된 프로젝트 README의 기존 불일치 사항을 수정했습니다.
- 소스 코드가 공개 GitHub 리포지토리에 게시되었습니다: https://github.com/google/agents-cli

## [0.4.0] - 2026-06-10

- 스캐폴드된 Python 템플릿이 이제 **ADK 2.0 GA**를 사용합니다. 새로운 `adk`, `adk_a2a` 및 `agentic_rag` 프로젝트는 `google-adk[gcp]>=2.0.0,<3.0.0`을 고정하며, `[gcp]` 엑스트라는 OpenTelemetry GCP 익스포터를 복원하고 BigQuery 클라이언트를 번들로 제공하므로 별도의 `[bigquery-analytics]` 엑스트라가 필요하지 않습니다. Cloud Run 및 GKE의 Cloud SQL 세션은 2.0에서도 계속 동작합니다. 번들된 ADK 코딩 스킬 및 참조 문서가 2.0에 맞춰 갱신되었습니다.
  - https://github.com/google/agents-cli/issues/24
- Agent Runtime 배포가 Cloud Run 동작과 동일하게 `--update-env-vars`를 통해 전달된 사용자의 `AGENT_VERSION`(또는 `NUM_WORKERS`)을 더 이상 덮어쓰지 않습니다. "버전을 찾을 수 없음" 경고는 이제 설정할 `pyproject.toml` 필드명을 안내합니다.
- Cloud Trace 관찰 가능성 가이드의 오래된 `deployment/terraform/dev/` 경로를 현재의 `single-project` terraform 구조에 맞춰 수정했습니다.

## [0.3.1] - 2026-06-04

- `eval generate`가 `VertexAiSearchTool`과 같은 내장 도구를 사용하는 ADK 2.x 프로젝트에서 동작하도록 수정되었습니다. SDK 수정 사항이 포함된 `google-cloud-aiplatform` 최소 버전을 1.156.0으로 상향했습니다.
  - https://github.com/google/agents-cli/issues/27
- `agents-cli setup`을 통해 설치된 스킬을 이제 Antigravity에서 확인할 수 있습니다. 전역 스킬이 Antigravity 스킬 디렉토리에 미러링됩니다.
  - https://github.com/google/agents-cli/issues/26
- `update` 실행 시 에러를 조용히 무시하지 않고 명확하게 출력합니다.
- 에이전트 배포 시 손상되거나 잘못된 형태의 `deployment_metadata.json`이 있어도 크래시 없이 정상 처리합니다.
- 배포 타임스탬프가 이제 타임존을 인식합니다.
- 잘못된 형식의 `AGENTS_CLI_EXPERIMENTS` 값이 있어도 CLI가 크래시되지 않습니다.
- `agents-cli install`이 이제 `--locked` 옵션으로 실행되어, 변경된 `uv.lock`이 있을 경우 조용히 새 의존성 버전을 구하는 대신 빠르게 실패합니다.

## [0.3.0] - 2026-05-29

### 주요 변경 사항 (Breaking Changes)

- 평가 데이터 형식이 ADK `EvalSet`에서 Vertex AI `EvaluationDataset`으로 변경되었습니다. 기존 `tests/eval/evalsets/*.evalset.json` 파일은 더 이상 `agents-cli eval generate` 및 관련 기능에서 읽히지 않습니다. 변환 방법은 [평가 데이터셋 마이그레이션](docs/src/reference/eval-dataset-migration.md)을 참고하세요. 레거시 파일이 감지되면 `scaffold upgrade`가 안내 메시지를 출력합니다.

### 평가 - 품질 플라이휠 (Quality Flywheel - 프리뷰)

- LLM 기반 사용자 시뮬레이션 데이터셋 생성을 위한 `eval dataset synthesize` 추가.
- `EvaluationDataset`에 대해 에이전트 추론을 실행하고 트레이스를 출력하는 `eval generate` 추가.
- 내장 또는 커스텀 메트릭 대비 에이전트 트레이스 점수를 측정하는 `eval grade` 추가.
- Vertex AI Eval Service에서 엔드투엔드 클라우드 측 평가 실행을 제출하는 `eval submit` 추가.
- 완료된 클라우드 평가 실행에서 결과를 가져오는 `eval results` 추가.
- 채점된 결과에 대해 실패 모드를 분석하는 `eval analyze` 추가.
- 내장 평가 메트릭을 탐색하는 `eval metric list` 추가.
- Quality Flywheel 워크플로우(데이터셋, 생성, 채점, 분석, 최적화)를 다루도록 `eval` 스킬을 전체 재작성.

### 기타

- 소규모 스킬 일관성 수정.

## [0.2.1] - 2026-05-28

- `--dry-run` 별칭으로 `--dryrun` 추가
- 스킬 설치 로직 개선
  - https://github.com/google/agents-cli/issues/23
- 성능 향상을 위해 자격 증명 캐싱
- gcloud 없이도 `is_authenticated`가 동작하도록 수정
  - https://github.com/google/agents-cli/issues/16
- Agent Runtime 배포 에러 메시지를 더 명확하게 수정
- 더 이상 필요하지 않은 gcloud 명령어에서 'beta' 제거
- 깨진 문서 링크 수정
- 배포 시 내보내기 전 누락된 lockfile 자동 생성
  - https://github.com/google/agents-cli/issues/17

## [0.2.0] - 2026-05-15

- agent-cli 프로젝트 구성을 언어 독립적인 agents-cli-manifest.yaml 파일로 이동
  - pyproject.toml에 임베딩된 기존 구성은 `agents-cli scaffold upgrade`를 통해 자동으로 마이그레이션 가능
- `eval optimize` 명령어 추가
- deploy에 `--network-attachment` 및 `--dns-peering-*` 플래그 추가
- 기타 시작 성능 향상
- 터미널 인코딩 관련 크래시 방지
  - https://github.com/google/agents-cli/issues/15 수정
- 특히 Windows에서의 도구 경로 확인 로직 개선
  - https://github.com/google/agents-cli/issues/14 수정
- 의존성 버전 잠금 업데이트
  - https://github.com/google/agents-cli/issues/13 수정
- Claude 및 Gemini CLI 플러그인 지원을 위한 매니페스트 지원 추가
- 스캐폴딩, 향상 및/또는 업그레이드 시 올바른 구성 메타데이터 보존 관련 버그 수정
- 기타 문서 및 스킬 수정
- https://google.github.io/agents-cli/ 에 Agents CLI 라이프사이클을 설명하는 시각적 설명 페이지 추가
- 미사용 템플릿 코드 정리

## [0.1.3] - 2026-05-06

- `infra` 명령어 기본값을 apply 대신 terraform plan으로 변경
- `playground`가 Cloud Shell 및 유사 환경에서 동작하도록 수정하고 기본 명령어 투명성 강화
- cloud sql 역할의 필요성을 다루도록 스킬 업데이트
- 쉬운 버그 리포팅을 위해 `agents-cli info`가 OS 정보를 출력하도록 수정
- `--start-server` 옵션으로 요청 시에만 `run`이 백그라운드 서버를 시작하도록 변경
- ADC 인증용 표시 문자열을 더 명확하게 수정
- 깨진 문서 링크 수정
- agent_runtime용 누락된 대상 설명 수정

## [0.1.2] - 2026-04-29

- 문서 및 이미지 수정
- 프로젝트 메타데이터 수정
- completions_view BigQuery SQL에서 멀티홉 트레이스 유지
- setup 실행 중 레거시 ADK 스킬 감지
- 인라인 아티팩트를 .google-agents-cli/artifacts/에 저장
- 일부 Windows 셸 상호작용 이슈 수정
- `deploy`에서 미처리 패스스루 인자 제거, 스킬 및 --help 텍스트 업데이트
- 인증 만료 시 agents-cli가 사용자를 인증 상태로 간주하던 버그 수정
- 에러 발생 시 로컬 `run` 서버 자동 중지

## [0.1.1] - 2026-04-22

- 특히 CLI 시작 시간에 대한 성능 향상
- 문서 정리

## [0.1.0] - 2026-04-21

- 최초 공개 릴리스
