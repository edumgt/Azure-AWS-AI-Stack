# 🤖 Lab 08 — AI Agent 개발

> Azure OpenAI 서비스 기반 AI Agent 설계·구현 실습 가이드  
> Orchestration · Ingestion · Retrieval (RAG) · Text-2SQL · Context & Policy

---

## 📌 개요

| 항목 | 내용 |
|------|------|
| **목표** | Azure OpenAI를 핵심으로 한 엔터프라이즈 AI Agent 전 영역 구현 |
| **핵심 서비스** | Azure OpenAI, AI Search, Cosmos DB, Azure SQL, Azure Functions, Logic Apps |
| **대상** | AI 솔루션 아키텍트, 백엔드 개발자, MLOps 엔지니어 |

---

## 1️⃣ Orchestration — 프롬프트 파이프라인 설계

### 1.1 개념

Orchestration 레이어는 사용자 요청을 **이해(Intent)** → **분류(Query Processing)** → **실행(Workflow)** 순서로 처리하는 AI Agent의 두뇌 역할을 합니다.

```
사용자 입력
    │
    ▼
┌─────────────────────────────────────┐
│         Intent Classifier           │  ← Azure OpenAI (GPT-4o)
│  (Intent 분류 / Slot Filling)       │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│         Query Processor             │  ← Prompt 전처리 / 정규화
│  (입력 정규화 / 언어 처리)           │
└─────────────────────────────────────┘
    │
    ├──── RAG 검색 ────▶ AI Search
    ├──── SQL 조회 ────▶ Azure SQL
    ├──── API 호출 ────▶ Azure Functions
    └──── 사람 승인 ───▶ HITL 워크플로우
    │
    ▼
┌─────────────────────────────────────┐
│         Response Synthesizer        │  ← Azure OpenAI (GPT-4o)
│  (최종 응답 생성 / 포맷팅)           │
└─────────────────────────────────────┘
```

### 1.2 프롬프트 파이프라인 설계 원칙

| 단계 | 설명 | Azure 서비스 |
|------|------|-------------|
| **System Prompt** | Agent 역할·제약·포맷 정의 | Azure OpenAI Deployments |
| **Few-shot Examples** | 도메인별 예시 주입 | Azure Blob / AI Search |
| **Chain-of-Thought** | 단계별 추론 유도 | OpenAI Function Calling |
| **Output Parsing** | JSON / Structured Output 강제 | Pydantic / JSON Schema |
| **Retry & Fallback** | 실패 시 대체 응답 처리 | Azure Logic Apps |

### 1.3 Intent / Query Processing

```python
# Intent 분류 예시 (Azure OpenAI Function Calling)
functions = [
    {
        "name": "classify_intent",
        "description": "사용자 쿼리를 분류합니다",
        "parameters": {
            "type": "object",
            "properties": {
                "intent": {
                    "type": "string",
                    "enum": ["rag_search", "sql_query", "api_call", "chitchat", "escalate"]
                },
                "confidence": {"type": "number"},
                "entities": {"type": "array", "items": {"type": "string"}}
            },
            "required": ["intent", "confidence"]
        }
    }
]
```

### 1.4 Workflow 구성

- **Azure Logic Apps**: 멀티스텝 워크플로우, 조건 분기, HITL 승인 게이트
- **Azure Durable Functions**: 장기 실행 오케스트레이션, 팬아웃/팬인 패턴
- **Semantic Kernel**: 플러그인 기반 Agent 오케스트레이션 프레임워크

```
Semantic Kernel Planner
    ├── Plugin: SearchPlugin    (AI Search 연동)
    ├── Plugin: SQLPlugin       (Azure SQL 연동)
    ├── Plugin: CalendarPlugin  (Graph API 연동)
    └── Plugin: HRPlugin        (내부 API 연동)
```

---

## 2️⃣ Ingestion — 데이터 전처리 파이프라인

### 2.1 개요

비정형 데이터(PDF, Word, HTML, 이미지 등)를 AI가 검색 가능한 형태로 변환하는 파이프라인입니다.

```
원본 데이터 소스
(PDF / Word / HTML / DB)
        │
        ▼
┌───────────────────┐
│   Document Loader  │  ← Azure Blob Storage, SharePoint, OneDrive
└───────────────────┘
        │
        ▼
┌───────────────────┐
│  Preprocessing    │  ← 언어 감지, 노이즈 제거, 표/이미지 추출
│  (전처리)         │     Azure AI Document Intelligence
└───────────────────┘
        │
        ▼
┌───────────────────┐
│    Chunking       │  ← 청크 크기/오버랩 전략 결정
└───────────────────┘
        │
        ▼
┌───────────────────┐
│    Embedding      │  ← Azure OpenAI Embeddings (text-embedding-3-large)
└───────────────────┘
        │
        ▼
┌───────────────────┐
│  VectorDB 저장    │  ← Azure AI Search (벡터 인덱스)
└───────────────────┘
```

### 2.2 Chunking 전략

| 전략 | 설명 | 권장 상황 |
|------|------|----------|
| **Fixed-size** | 고정 토큰 수로 분할 (e.g., 512 tokens) | 균일한 텍스트 문서 |
| **Semantic** | 의미 단위(문단/섹션)로 분할 | 구조화된 문서 (PDF, Word) |
| **Recursive** | 구분자 계층별 재귀 분할 | 다양한 형식의 혼합 문서 |
| **Sliding Window** | 오버랩(예: 10~20%) 포함 분할 | 문맥 연속성이 중요한 경우 |
| **Parent-Child** | 부모 청크 + 자식 청크 이중 저장 | 정밀 검색 + 풍부한 컨텍스트 |

```python
# Parent-Child Chunking 예시
parent_chunk_size = 2048  # 토큰
child_chunk_size  = 256   # 토큰
overlap           = 32    # 토큰
```

### 2.3 Embedding

| 모델 | 차원 | 특징 |
|------|------|------|
| `text-embedding-3-large` | 3072 | 고정밀, 다국어 지원 |
| `text-embedding-3-small` | 1536 | 경량, 저비용 |
| `text-embedding-ada-002` | 1536 | 레거시, 호환성 |

```python
from openai import AzureOpenAI

client = AzureOpenAI(
    api_key=os.getenv("AZURE_OPENAI_KEY"),
    api_version="2024-02-01",
    azure_endpoint=os.getenv("AZURE_OPENAI_ENDPOINT")
)

response = client.embeddings.create(
    input=chunk_text,
    model="text-embedding-3-large"
)
embedding_vector = response.data[0].embedding
```

### 2.4 VectorDB Collection 구성 (Azure AI Search)

```json
{
  "name": "knowledge-index",
  "fields": [
    {"name": "id",        "type": "Edm.String",            "key": true},
    {"name": "content",   "type": "Edm.String",            "searchable": true},
    {"name": "source",    "type": "Edm.String",            "filterable": true},
    {"name": "category",  "type": "Edm.String",            "filterable": true, "facetable": true},
    {"name": "embedding", "type": "Collection(Edm.Single)", "dimensions": 3072,
     "vectorSearchProfile": "hnsw-profile"}
  ],
  "vectorSearch": {
    "algorithms": [{"name": "hnsw-algo", "kind": "hnsw",
                    "parameters": {"m": 4, "efConstruction": 400, "metric": "cosine"}}],
    "profiles": [{"name": "hnsw-profile", "algorithm": "hnsw-algo"}]
  },
  "semantic": {
    "configurations": [{"name": "semantic-config",
                        "prioritizedFields": {"contentFields": [{"fieldName": "content"}]}}]
  }
}
```

---

## 3️⃣ Retrieval — RAG 파이프라인 구축

### 3.1 RAG 아키텍처

```
사용자 질문
    │
    ▼ (Embedding)
┌──────────────────────────────────┐
│        Azure AI Search           │
│  ┌─────────────┐ ┌────────────┐  │
│  │ 벡터 검색   │ │ 키워드 검색│  │
│  │ (Semantic)  │ │ (BM25)     │  │
│  └─────────────┘ └────────────┘  │
│         └────── Hybrid ──────┘   │
│              + Re-ranking        │
└──────────────────────────────────┘
    │  Top-K 청크 (Context)
    ▼
┌──────────────────────────────────┐
│        Azure OpenAI GPT-4o       │
│  System Prompt + Context + Query │
│  → 최종 응답 생성                 │
└──────────────────────────────────┘
    │
    ▼
응답 + 출처(Citations)
```

### 3.2 검색 전략

| 전략 | 설명 | 효과 |
|------|------|------|
| **Vector Search** | 의미 기반 유사도 검색 (코사인 유사도) | 동의어·맥락 대응 |
| **Keyword Search** | BM25 기반 정확도 매칭 | 고유명사·코드 검색 |
| **Hybrid Search** | 벡터 + 키워드 결합 (RRF 방식) | 정밀도·재현율 균형 |
| **Semantic Reranking** | AI 기반 관련성 재정렬 | 최종 품질 향상 |
| **HyDE** | 가상 문서 생성 후 검색 | 추상적 질문 대응 |

### 3.3 RAG 품질 개선 기법

- **Query Rewriting**: 원본 질문을 검색에 최적화된 형태로 재작성
- **Multi-Query**: 동일 질문을 여러 각도로 변환하여 다중 검색
- **Self-RAG**: 검색 결과의 관련성을 LLM이 자체 평가
- **Contextual Compression**: 검색된 청크 중 관련 부분만 압축 추출
- **Citation Grounding**: 응답에 출처 청크 링크 포함

```python
# Hybrid Search + Semantic Reranking
results = search_client.search(
    search_text=query,
    vector_queries=[VectorizedQuery(
        vector=query_embedding,
        k_nearest_neighbors=50,
        fields="embedding"
    )],
    query_type=QueryType.SEMANTIC,
    semantic_configuration_name="semantic-config",
    top=5,
    select=["id", "content", "source"]
)
```

---

## 4️⃣ Text-2SQL — 자연어 → SQL 변환

### 4.1 아키텍처

```
자연어 질문
    │
    ▼
┌─────────────────────────────────────┐
│         Schema Mapper               │
│  DB 스키마 + Viewtable 메타데이터    │
│  (테이블명, 컬럼, 관계, 샘플 데이터) │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│      Azure OpenAI (GPT-4o)          │
│  → SQL 쿼리 생성                     │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│         Guardrail Layer             │
│  - SQL 파싱·유효성 검사              │
│  - 위험 키워드 차단 (DROP/DELETE 등) │
│  - 결과 범위 제한 (LIMIT 강제)       │
└─────────────────────────────────────┘
    │
    ▼
┌─────────────────────────────────────┐
│       Query Execution               │
│       Azure SQL / Synapse           │
└─────────────────────────────────────┘
    │
    ▼
결과 → 자연어 응답 변환 (GPT-4o)
```

### 4.2 Viewtable 설계 전략

| 원칙 | 설명 |
|------|------|
| **읽기 전용 View 사용** | 원본 테이블 직접 접근 차단, View 레이어만 노출 |
| **비즈니스 명칭 사용** | 기술적 컬럼명 → 비즈니스 친화적 별칭 |
| **민감 컬럼 제외** | 개인정보(주민번호, 연락처 등) 마스킹/제외 |
| **조인 사전 처리** | 복잡한 조인은 View에서 처리하여 복잡도 감소 |
| **필터 기본값 내장** | 활성 데이터 필터, 기간 제한 등 기본 조건 포함 |

```sql
-- 예: 비즈니스 친화적 Viewtable
CREATE VIEW vw_sales_summary AS
SELECT
    o.order_date                    AS 주문일자,
    p.product_name                  AS 상품명,
    c.region                        AS 지역,
    SUM(o.quantity)                 AS 총_판매수량,
    SUM(o.quantity * p.unit_price)  AS 총_매출액
FROM orders o
JOIN products p ON o.product_id = p.id
JOIN customers c ON o.customer_id = c.id
WHERE o.status = 'COMPLETED'
GROUP BY o.order_date, p.product_name, c.region;
```

### 4.3 Guardrail (환각 방지) 적용

| Guardrail 유형 | 적용 방법 |
|---------------|----------|
| **SQL 파싱 검증** | `sqlparse` / `sqlglot` 라이브러리로 구문 유효성 확인 |
| **DML 차단** | `INSERT`, `UPDATE`, `DELETE`, `DROP` 포함 시 실행 거부 |
| **Row Limit 강제** | 모든 쿼리에 `TOP 1000` 또는 `LIMIT 1000` 자동 삽입 |
| **Execution Timeout** | 쿼리 실행 시간 제한 (기본: 30초) |
| **결과 검증** | 빈 결과 또는 이상 결과 시 재질문 유도 |
| **Confidence Score** | LLM 자체 신뢰도 평가 후 임계값 미달 시 사람 검토 요청 |

```python
import sqlglot

def validate_sql(sql: str) -> tuple[bool, str]:
    forbidden = {"DROP", "DELETE", "INSERT", "UPDATE", "TRUNCATE", "ALTER"}
    try:
        parsed = sqlglot.parse_one(sql, dialect="tsql")
        tokens = {token.upper() for token in sql.split()}
        blocked = forbidden & tokens
        if blocked:
            return False, f"위험 키워드 감지: {blocked}"
        return True, "OK"
    except Exception as e:
        return False, f"SQL 파싱 오류: {e}"
```

---

## 5️⃣ Context & Policy — 세션 컨텍스트 / HITL / Policy Layer

### 5.1 Session Context 구현

```
대화 세션
    │
    ├── Short-term Memory  ← 최근 N턴 대화 히스토리 (Redis / Cosmos DB)
    ├── Long-term Memory   ← 사용자 선호·도메인 지식 (Azure AI Search)
    └── Working Memory     ← 현재 Task 진행 상태 (In-memory / Durable Functions)
```

| 컴포넌트 | 저장소 | TTL | 설명 |
|---------|--------|-----|------|
| **대화 히스토리** | Azure Cosmos DB (NoSQL) | 세션 종료 | 멀티턴 대화 유지 |
| **사용자 프로필** | Azure Cosmos DB | 영구 | 사용자별 설정·이력 |
| **검색 캐시** | Azure Cache for Redis | 1시간 | 동일 질문 재검색 방지 |
| **태스크 상태** | Durable Functions State | Task 완료 | 장기 워크플로우 진행 상태 |

```python
# Cosmos DB 세션 컨텍스트 예시
class SessionContext:
    def __init__(self, session_id: str, cosmos_client):
        self.session_id = session_id
        self.container = cosmos_client.get_container("sessions")

    def add_turn(self, role: str, content: str):
        self.container.upsert_item({
            "id": self.session_id,
            "history": self._get_history() + [{"role": role, "content": content}],
            "updated_at": datetime.utcnow().isoformat()
        })

    def get_context_window(self, max_turns: int = 10) -> list:
        item = self.container.read_item(self.session_id, self.session_id)
        return item.get("history", [])[-max_turns * 2:]
```

### 5.2 HITL (Human-in-the-Loop) 시스템

HITL은 AI 판단의 불확실성이 높거나, 높은 위험·비용이 수반되는 액션에 **사람의 검토 및 승인**을 삽입하는 패턴입니다.

```
AI Agent 응답 생성
        │
        ▼
┌──────────────────────────────────┐
│       Risk Assessment            │
│  - Confidence < 0.7?             │
│  - 민감 데이터 포함?              │
│  - 비가역적 액션(삭제/결제 등)?   │
└──────────────────────────────────┘
    │ 위험 감지 시          │ 안전 시
    ▼                      ▼
┌─────────────┐      ┌─────────────┐
│ HITL 게이트  │      │ 자동 실행   │
│ (사람 검토) │      └─────────────┘
└─────────────┘
    │
    ├── 승인 → 액션 실행
    └── 거부 → 재시도 / 대체 응답
```

**Azure 구현 방법:**

| 방법 | 서비스 | 설명 |
|------|--------|------|
| **이메일 승인** | Logic Apps + Outlook | 담당자 이메일로 승인 요청 |
| **Teams 알림** | Logic Apps + Teams | Teams 채널에 승인 카드 발송 |
| **대시보드 검토** | Azure Static Web Apps | 관리자 웹 UI에서 검토 |
| **비동기 대기** | Durable Functions (외부 이벤트) | 승인 신호 수신까지 워크플로우 대기 |

```python
# Durable Functions HITL 패턴
@app.orchestration_trigger(context_name="context")
def hitl_orchestrator(context: df.DurableOrchestrationContext):
    # 1. AI 응답 초안 생성
    draft = yield context.call_activity("generate_draft", context.get_input())

    # 2. 위험도 평가
    risk = yield context.call_activity("assess_risk", draft)

    if risk["score"] > 0.7:
        # 3. 사람 승인 대기 (최대 24시간)
        approval = yield context.wait_for_external_event(
            "ApprovalEvent",
            timeout=timedelta(hours=24)
        )
        if not approval["approved"]:
            return {"status": "rejected", "reason": approval["reason"]}

    # 4. 최종 실행
    result = yield context.call_activity("execute_action", draft)
    return result
```

### 5.3 Policy Layer

Policy Layer는 Agent의 모든 입출력에 적용되는 **규칙·제약·윤리 가이드라인** 레이어입니다.

| Policy 유형 | 설명 | 구현 방법 |
|------------|------|----------|
| **Content Safety** | 유해·혐오·불법 콘텐츠 차단 | Azure AI Content Safety API |
| **PII Detection** | 개인식별정보 감지·마스킹 | Azure AI Language (PII) |
| **Rate Limiting** | 사용자/세션별 호출 제한 | API Management Policy |
| **Scope Control** | 도메인 외 질문 거부 | System Prompt + Intent 분류 |
| **Audit Logging** | 모든 상호작용 로깅·감사 | Azure Monitor + Log Analytics |
| **Data Residency** | 데이터 지역 제한 | Azure Region 정책 |

```python
# Policy Pipeline 예시
class PolicyPipeline:
    def __init__(self):
        self.content_safety = ContentSafetyClient(...)
        self.pii_recognizer = TextAnalyticsClient(...)

    async def check_input(self, user_input: str) -> PolicyResult:
        # 1. Content Safety 검사
        safety = await self.content_safety.analyze_text(user_input)
        if safety.hate_result.severity > 2:
            return PolicyResult(blocked=True, reason="유해 콘텐츠")

        # 2. PII 감지
        pii = await self.pii_recognizer.recognize_pii_entities([user_input])
        if pii[0].entities:
            masked = self._mask_pii(user_input, pii[0].entities)
            return PolicyResult(blocked=False, sanitized_input=masked)

        return PolicyResult(blocked=False, sanitized_input=user_input)
```

---

## 🔗 관련 Azure 서비스

| 서비스 | 역할 |
|--------|------|
| **Azure OpenAI Service** | LLM 추론 (GPT-4o, Embedding) |
| **Azure AI Search** | 벡터 인덱스 / 하이브리드 검색 |
| **Azure AI Document Intelligence** | 문서 파싱 / OCR |
| **Azure AI Content Safety** | 콘텐츠 정책 적용 |
| **Azure Cosmos DB** | 세션 컨텍스트 / 대화 이력 저장 |
| **Azure Cache for Redis** | 검색 캐시 / 세션 TTL 관리 |
| **Azure SQL / Synapse Analytics** | Text-2SQL 대상 데이터베이스 |
| **Azure Durable Functions** | 오케스트레이션 / HITL 대기 |
| **Azure Logic Apps** | 멀티채널 알림 / 승인 워크플로우 |
| **Azure API Management** | Policy 적용 / Rate Limiting |
| **Azure Monitor + Log Analytics** | 감사 로깅 / 모니터링 |

---

## 📚 참고 자료

- [Azure OpenAI Service 공식 문서](https://learn.microsoft.com/azure/ai-services/openai/)
- [Azure AI Search 벡터 검색](https://learn.microsoft.com/azure/search/vector-search-overview)
- [Semantic Kernel 공식 문서](https://learn.microsoft.com/semantic-kernel/)
- [Azure AI Content Safety](https://learn.microsoft.com/azure/ai-services/content-safety/)
- [Durable Functions 패턴 — 사람 상호작용](https://learn.microsoft.com/azure/azure-functions/durable/durable-functions-overview)

---

_Lab 08 | Azure AI Agent 개발 | 최종 수정: 2026-04_
