# 스킬 레퍼런스

스킬은 `agents-cli setup`을 통해 코딩 에이전트(Antigravity CLI, Claude Code, GitHub Copilot 등)에 설치되는 컨텍스트 파일입니다. 생성된 에이전트 프로젝트로 작업할 때 도메인 관련 지침을 제공합니다.

```bash
agents-cli setup      # 모든 스킬 설치
agents-cli update     # 스킬 재설치 / 업데이트
```

---

## `google-agents-cli-adk-code`

에이전트 유형, 도구 정의, 오케스트레이션 패턴, 콜백 및 상태 관리에 대한 빠른 참조를 제공합니다.

---

## `google-agents-cli-deploy`

배포 워크플로, 서비스 계정, 롤백 및 프로덕션 인프라를 다룹니다. Google ADK (Agent Development Kit) 스킬 수트의 일부입니다.

---

## `google-agents-cli-eval`

평가 라이프사이클 전체(데이터셋 스키마, 트레이스 생성 및 채점, 실행 결과 비교, 실패 클러스터 분석, 메트릭 검색, 프롬프트 최적화, LLM-as-a-judge 설정, 주요 실패 원인)를 다룹니다. Google ADK (Agent Development Kit) 스킬 수트의 일부입니다.

---

## `google-agents-cli-observability`

Cloud Trace, 프롬프트-응답 로깅, BigQuery Agent Analytics, 서드파티 연동(AgentOps, Phoenix, MLflow 등) 및 트러블슈팅을 다룹니다. Google ADK (Agent Development Kit) 스킬 수트의 일부입니다.

---

## `google-agents-cli-publish`

ADK 대 A2A 등록 모드, 프로그래밍 방식 및 대화형 사용법, 플래그 레퍼런스, 배포 메타데이터에서의 자동 감지, 트러블슈팅을 다룹니다. Google ADK (Agent Development Kit) 스킬 수트의 일부입니다.

---

## `google-agents-cli-scaffold`

`agents-cli scaffold create`, `scaffold enhance`, `scaffold upgrade` 명령, 템플릿 옵션, 배포 타깃, 프로토타입 우선 워크플로를 다룹니다.

---

## `google-agents-cli-workflow`

항상 활성화되어 있음 — ADK 또는 모든 에이전트 개발을 위한 전체 워크플로(스캐폴드, 빌드, 평가, 배포, 게시, 관측), 코드 보존 규칙, 모델 선택 가이드, 트러블슈팅 단계를 제공합니다.
