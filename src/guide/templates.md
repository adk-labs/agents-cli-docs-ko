# 에이전트 템플릿

`agents-cli`는 에이전트 템플릿을 사용하여 프로젝트를 생성합니다. 각 템플릿은 해당 사용 사례에 맞는 의존성, 도구 및 프로젝트 구조를 갖춘 바로 동작 가능한 에이전트를 제공합니다.

---

## 사용 가능한 템플릿

| 템플릿 | 설명 | 사용 사례 |
|----------|-------------|----------|
| `adk` | ADK를 사용하는 ReAct 에이전트 | 도구 사용을 포함한 범용 대화형 에이전트 |

> **RAG는 템플릿이 아닙니다** — 복제 후 학습하여 적용하는 레시피입니다. 아래의 [RAG](#rag-retrieval-augmented-generation)를 참고하세요.

### adk

기본 템플릿입니다. 샘플 도구가 포함된 [Agent Development Kit](https://google.github.io/adk-docs/) 기반의 ReAct 에이전트를 생성합니다. ADK를 처음 사용하거나 범용 에이전트를 구축하는 경우 여기서 시작하세요.

```bash
agents-cli create my-agent --agent adk
```

모든 Python ADK 에이전트는 별도의 설정 없이 [Agent-to-Agent (A2A) 프로토콜](https://a2a-protocol.org)을 즉시 지원합니다. A2A 라우트(에이전트 카드 + JSON-RPC)가 자동으로 마운트됩니다. 에이전트가 다른 프레임워크(LangGraph, CrewAI 등)로 구축된 에이전트와 상호 운용되어야 하거나 분산 멀티 에이전트 시스템을 구축할 때 사용하세요. 별도의 템플릿이나 직접 작성하는 A2A 코드는 필요하지 않습니다.

## RAG (Retrieval-Augmented Generation)

RAG는 템플릿이 **아닙니다** — 복제 후 학습하여 적용하는 레시피입니다. 기본 `adk` 프로젝트를 생성한 후, [google/adk-samples](https://github.com/google/adk-samples)의 RAG 샘플 중 하나를 살펴보고 에이전트에 맞춰 변경하면서 해당 검색기(retriever)와 `infra/terraform/`을 프로젝트로 복사하세요:

- **`rag-vector-search`** — 커스텀 인제스천 파이프라인(임베딩, 유사도 검색)이 포함된 Vertex AI Vector Search 2.0.
- **`rag-agent-search`** — 완전 관리형 GCS 데이터 커넥터가 포함된 Agent Platform Search (Discovery Engine) — 버킷에 파일을 넣기만 하면 되며, 별도의 인제스천 코드를 작성할 필요가 없습니다.

ADK 코드 스킬의 `references/samples.md`에 주요 파일과 함께 두 샘플이 모두 나와 있으며, 각 샘플의 `AGENTS.md`가 학습 및 적용을 위한 가이드입니다. 프로비저닝 및 인제스천은 샘플 자체의 `Makefile`(`make setup-infra`, `make data-ingestion`)에서 실행할 수 있습니다.
