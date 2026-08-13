# 유스케이스

Agents CLI는 코딩 에이전트에 전달한 설명을 바탕으로 에이전트를 스캐폴딩, 평가 및 배포합니다. 다음과 같은 에이전트를 구축해 보세요:

- **스케줄링 봇.** RSS 피드에서 데이터를 가져와 LLM으로 요약하고 Cloud Scheduler 트리거를 통해 Google Chat 또는 이메일로 발송합니다.
- **조사 에이전트.** 로그를 읽고 배포 트레이스를 분석하며 과거 장애와 연관 지어 근본 원인 분석(RCA)을 수행합니다.
- **지식 에이전트.** 대화, 이메일, 설계 문서를 인덱싱하여 동일한 주제가 반복될 때 과거의 결정 사항을 탐색할 수 있도록 합니다.
- **A2A 멀티 에이전트 시스템.** 장애 대응, 코드 마이그레이션, 보안 감사 전반에 걸쳐 전문 에이전트들을 협업시킵니다.

---

## 패턴 선택

<table class="use-case-grid">
<tr>
<td align="center" width="33%"><h3><a href="#일일-뉴스-봇">일일 뉴스 봇</a></h3></td>
<td align="center" width="33%"><h3><a href="#업계-동향-모니터링">업계 동향 모니터링</a></h3></td>
<td align="center" width="33%"><h3><a href="#자체-튜닝-지원-에이전트">자체 튜닝 지원 에이전트</a></h3></td>
</tr>
<tr>
<td align="center"><h3><a href="#기술-조사-에이전트">기술 조사 에이전트</a></h3></td>
<td align="center"><h3><a href="#성능-저하-감지기">성능 저하 감지기</a></h3></td>
<td align="center"><h3><a href="#조직-메모리">조직 메모리</a></h3></td>
</tr>
<tr>
<td align="center"><h3><a href="#사내-지식-네비게이터">사내 지식 네비게이터</a></h3></td>
<td align="center"><h3><a href="#실사-에이전트">실사 에이전트</a></h3></td>
<td align="center"><h3><a href="#보안-감사-에이전트">보안 감사 에이전트</a></h3></td>
</tr>
<tr>
<td align="center"><h3><a href="#rfp-제안서-생성기">RFP 제안서 생성기</a></h3></td>
<td align="center"><h3><a href="#장애-대응-조율">장애 대응 조율</a></h3></td>
<td align="center"><h3><a href="#분산-코드-마이그레이션">분산 코드 마이그레이션</a></h3></td>
</tr>
</table>

!!! note "아직 지원되지 않는 기능"

    - **실시간 음성 및 비디오**
    - **Python 이외의 에이전트** (Go, Java, TypeScript)
    - **멀티 클라우드 배포** — Google Cloud에 집중되어 있습니다. 타 클라우드와의 상호작용에는 별도의 인프라 및 스킬이 필요할 수 있습니다.

---

## 초급

에이전트 간 조율이 필요 없는 단일 에이전트 패턴입니다. 첫 프로젝트로 적합합니다.

### 일일 뉴스 봇

*초급 · `adk`*

지정된 RSS 피드 세트에서 헤드라인을 가져와 LLM으로 가장 관련성 높은 항목을 선택하고 Google Chat 또는 이메일로 발송합니다. Cloud Scheduler로 스케줄링합니다.

```
Build me a daily news bot that pulls these RSS feeds, summarizes the top 5 stories, and posts to Google Chat every morning.
```

스케줄링 및 롤아웃은 [배포](deployment.md) 및 [CI/CD](cicd.md)를 참고하세요.

### 업계 동향 모니터링

*초급 · `adk`*

업계 전반의 공용 릴리스 노트, 문서 업데이트, 채용 공고 및 컨퍼런스 발표를 추적합니다. 배포된 기능과 채용 트렌드를 포착합니다. 수집된 결과를 검색 가능한 저장소에 지속적으로 저장하여 주간 검토에 활용합니다.

```
Track these companies' public docs, releases, and job postings daily. Surface shipped features and hiring trends.
```

---

## 중급

피드백 루프, RAG(검색 증강 생성) 또는 주요 툴 연동과 결합된 단일 에이전트 패턴입니다.

### 자체 튜닝 지원 에이전트

*중급 · `adk`*

각 대화 후 평가를 실행하여 지식이나 동작의 범위를 파악하고, 부족한 응답에 대해 새로운 평가 케이스 초안을 작성합니다. 실제 고객 질문에 맞춰 평가 범위가 유기적으로 적용됩니다.

```
Build a support agent that runs eval after each conversation, drafts new eval cases for weak answers, and surfaces documentation gaps.
```

[평가 가이드](evaluation.md)에서 평가 및 수정 루프를 다룹니다. 프로덕션 트레이스를 재현하려면 [관찰 가능성](observability/index.md)과 함께 활용하세요.

### 기술 조사 에이전트

*중급 · `adk`*

"지난달 결제 서비스 지연 시간이 증가한 이유는 무엇인가요?"와 같은 질문을 처리합니다. 로그를 읽고 배포 트레이스를 분석하며 과거 장애와 연관 지어 타임라인 및 근본 원인 분석(RCA)을 생성합니다.

```
Build an investigation agent. I ask questions like "why did X break last week" and it pulls from logs, deploy history, and past incidents to produce a writeup.
```

### 성능 저하 감지기

*중급 · `adk`*

현재 메트릭 및 로그 패턴을 과거 장애 발생 전 징후와 비교합니다. 현재 동작이 알려진 성능 저하(regression) 패턴과 일치하면 예방적 이슈를 생성합니다. 매일 밤 스케줄로 실행됩니다.

```
Build an agent that runs nightly, looks for metric/log patterns that match historical pre-incident signatures, and files preventive bugs.
```

### 조직 메모리

*중급 · RAG 레시피*

의사결정 기록을 위해 Google Chat, 이메일, 설계 문서 및 회의록을 인덱싱합니다. 동일한 제안이 다시 등장할 때(예: "세션 관리에 Redis 사용") 팀이 도달했던 원래 쓰레드와 결정을 제시합니다.

```
Build a RAG agent that indexes Google Chat, email, and design docs nightly. Surface past decisions when someone proposes something we've already discussed.
```

RAG는 복사 및 학습용 레시피입니다 — 활용 가능한 샘플은 [에이전트 템플릿](templates.md#rag-retrieval-augmented-generation)을 참고하세요.

### 사내 지식 네비게이터

*중급 · RAG 레시피 · Gemini Enterprise*

Drive, Google Chat, 이메일에 대한 권한이 부여된 상태로 Gemini Enterprise에 배포합니다. "프로덕션 데이터베이스 접근 권한은 어떻게 얻나요?"와 같은 질문에 대해 공식 문서화된 절차와 실제 운용 현황을 함께 응답합니다.

```
Build a RAG agent for new-hire questions that knows both official docs and how things actually work. Publish it to Gemini Enterprise.
```

등록 세부 정보는 [`google-agents-cli-publish`](../reference/skills.md) 스킬을 참고하세요.

---

## 고급

장시간 실행되는 워크플로우 또는 멀티 에이전트 조율 패턴입니다. 전용 인프라와 확장된 개발 작업이 필요합니다.

### 실사 에이전트

*고급 · RAG 레시피*

약 50만 줄 규모의 타겟 코드베이스를 인덱싱합니다. 기술 부채, 보안 취약점, 라이선스 준수 여부 및 배포 복잡성을 분석합니다. 라인 번호, 의존성 그래프, CVE 참조가 포함된 위험 보고서를 작성합니다. 며칠에 걸친 분석에는 Agent Runtime의 확장된 세션 및 체크포인팅 기능이 유용합니다.

```
Build a due-diligence agent that indexes a target codebase, runs security and license scans, and produces a risk report with citations.
```

### 보안 감사 에이전트

*고급 · `adk`*

GDPR, HIPAA 또는 SOC2 규정 준수 여부를 검증하기 위해 코드베이스 전반의 데이터 흐름을 매핑합니다. 수집부터 삭제까지 민감한 데이터의 이동을 추적합니다. 사용자 데이터를 설정된 보유 정책 이상으로 보관하는 분석 로그 등의 허점을 식별합니다.

```
Build a compliance-audit agent that traces sensitive data flows across our codebase and flags retention/policy gaps with file:line citations.
```

감사 추적 완성도를 모니터링하려면 [BigQuery Agent Analytics](observability/bq-agent-analytics.md)를 활용하세요.

### RFP 제안서 생성기

*고급 · RAG 레시피*

과거 프로젝트 기록, 현재 리소스 가용성 및 가격 책정 모델을 추출합니다. 타임라인과 예산을 추정하고 기술적 접근 방식을 작성하여 사람이 검토할 제안서 패키지를 생성합니다.

```
Build a RAG agent that drafts RFP responses by pulling from past proposals, current resourcing, and pricing models.
```

### 장애 대응 조율

*고급 (A2A) · `adk`*

장애 발생 시 전담 에이전트들을 병렬로 실행합니다: 최근 변경 사항 이분 탐색, 서비스 간 에러 연관 분석, 과거 장애 검색, 고객 공지문 초안 작성 등을 각각 담당합니다. 병렬 조사를 통해 순차적 트러블슈팅 대비 근본 원인 파악 시간을 단축시킵니다.

```
Build an A2A multi-agent system for incident response. Specialists for bisection, error correlation, past-incident lookup, and customer comms — coordinated in parallel.
```

A2A 프로토콜은 [`adk` 템플릿](templates.md)(A2A 기본 내장)에서 제공됩니다. 각 전문 에이전트는 서비스로 실행되며 코디네이터가 전체 실행을 오케스트레이션합니다.

### 분산 코드 마이그레이션

*고급 (A2A) · `adk`*

대규모 프레임워크 마이그레이션을 위해 전문 에이전트들을 실행합니다: 데이터 모델, API 계약, 테스트, 검증을 각각 담당합니다. 에이전트들은 A2A를 통해 호환성을 깨뜨리는 변경 사항(breaking changes)에 대한 분석 결과를 공유합니다. 다수의 동시 전문 에이전트 인스턴스를 실행할 때는 GKE 런타임을 권장합니다.

```
Build A2A specialist agents for a large framework migration: data models, API contracts, tests, validation.
```

---

## 다음 단계

- [튜토리얼: 첫 번째 에이전트 구축하기](quickstart-tutorial.md) — 코딩 에이전트로 구축, 평가 및 배포하기
- [프로젝트 구조](project-structure.md) — 생성된 각 파일의 역할 이해하기
- [에이전트 템플릿](templates.md) — `adk` 템플릿 및 RAG 학습 레시피
- [개발 가이드](development.md) — 전체 개발 워크플로우
- [CLI 참조 문서](../cli/index.md) — 모든 명령어 및 플래그
