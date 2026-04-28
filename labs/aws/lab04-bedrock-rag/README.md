# Lab 04: 생성형 AI/RAG — Bedrock · Kendra 기반 사내지식 응답

[← AWS Labs 목록으로](../README.md)

---

## 📌 개요

Amazon Bedrock과 Kendra를 활용하여 사내 문서(S3/포털/위키/정책/매뉴얼) 기반의 검색 + 생성형 응답(RAG) 시스템을 구축합니다.

---

## 🎯 학습 목표

- RAG(Retrieval-Augmented Generation) 아키텍처 이해
- Kendra 인덱싱 및 검색 구성
- Bedrock 기반 생성형 응답 구현
- 정책(가드레일) 적용 및 테넌트 분리

---

## 🔧 핵심 기능

- **사내 지식 기반 응답**: 사내 문서(S3/포털/위키/정책/매뉴얼) 기반 검색(Kendra) + 생성형 응답(Bedrock)
- **답변 근거 제공**: 출처 링크/문서 스니펫 제공
- **정책 적용**: 금칙어/PII 등 정책(가드레일) 적용
- **멀티테넌트**: 고객사별 테넌트/지식베이스 분리 (권한, 암호화, 로깅 포함)

---

## 🏗️ 레퍼런스 아키텍처

### RAG 지식봇 (사내 문서 기반)

```
문서 수집/정제 → 인덱싱(Kendra/Vector) → 질의 → 검색 결과 → Bedrock 응답 생성 + 정책(가드레일)
```

---

## 📋 활용 시나리오

- **사내 지식봇**: 정책/매뉴얼 기반 검색+요약 응답(Bedrock/Kendra) + 근거 링크 제공
- **규정 안내**: "이 계약서에 적용되는 규정은?" → Kendra 검색 → Bedrock 요약 응답
- **신입사원 온보딩**: 사내 규정/프로세스 질의 → 자연어 응답 + 출처 제공

---

## 🔗 관련 AWS 서비스

| 서비스 | 역할 |
|--------|------|
| Amazon Bedrock | LLM 기반 생성형 응답 |
| Amazon Kendra | 사내 문서 검색 엔진 |
| Amazon S3 | 문서 원본 저장 |
| AWS Lambda | 검색-생성 오케스트레이션 |
| Amazon CloudWatch | 사용량/비용 모니터링 |

---

## 📚 참고 자료

- [Amazon Bedrock](https://aws.amazon.com/bedrock/)
- [Amazon Kendra](https://aws.amazon.com/kendra/)
- [RAG Architecture Patterns](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html)

---

## ☁️ Azure 대응 서비스 비교

> 같은 목표를 Azure에서 구현할 때 사용하는 서비스와 개념을 비교합니다.  
> Azure Lab 참조: [Azure Lab 08 — AI Agent 개발](../../azure/lab08-ai-agent/README.md)

### 서비스 대응 매핑

| 기능 | AWS 서비스 | Azure 대응 서비스 |
|------|-----------|-----------------|
| LLM 추론 (생성형 응답) | Amazon Bedrock (Claude/Titan/Llama) | Azure OpenAI Service (GPT-4o/GPT-4) |
| 문서 검색 엔진 | Amazon Kendra | Azure AI Search (Cognitive Search) |
| 벡터 임베딩 저장 | Bedrock Knowledge Base + OpenSearch | Azure AI Search (Vector Search) |
| 문서 원본 저장 | Amazon S3 | Azure Blob Storage |
| 오케스트레이션 로직 | AWS Lambda | Azure Functions / Azure AI Foundry |
| 정책/가드레일 | Bedrock Guardrails | Azure AI Content Safety + System Prompt |
| 멀티테넌트 분리 | Bedrock Knowledge Base (분리) | AI Search Index (테넌트별 분리) |
| 사용량/비용 모니터링 | Amazon CloudWatch | Azure Monitor + Cost Management |

### 아키텍처 비교

**AWS (Bedrock + Kendra RAG)**
```
문서(S3) → Kendra 인덱싱 → 사용자 질의 → Kendra 검색 → Bedrock(Claude) 응답 생성 + Guardrails
```

**Azure (Azure OpenAI + AI Search RAG)**
```
문서(Blob) → AI Search 인덱싱(벡터 임베딩) → 사용자 질의 → AI Search 검색 → Azure OpenAI(GPT-4o) 응답 생성 + Content Safety
```

### 핵심 개념 차이점

| 항목 | AWS (Bedrock + Kendra) | Azure (OpenAI + AI Search) |
|------|----------------------|--------------------------|
| 검색 방식 | Kendra: 시맨틱 검색 (ML 기반) | AI Search: 벡터 + 키워드 하이브리드 검색 |
| 임베딩 생성 | Bedrock Embeddings (Titan) | Azure OpenAI text-embedding-ada-002 |
| LLM 선택 | Bedrock: Claude, Titan, Llama 등 멀티 모델 | Azure OpenAI: GPT-4o, GPT-4 Turbo |
| 정책 적용 | Bedrock Guardrails (금칙어/PII 필터) | System Prompt + Azure AI Content Safety |
| 인용/출처 제공 | Bedrock Knowledge Base 자동 인용 | AI Search 검색 결과 참조 필드 |

### Azure 실습 코드 예시 (Azure OpenAI + AI Search RAG)

```python
from openai import AzureOpenAI
from azure.search.documents import SearchClient
from azure.core.credentials import AzureKeyCredential

search_client = SearchClient(
    endpoint="https://<search>.search.windows.net",
    index_name="knowledge-index",
    credential=AzureKeyCredential("<search-key>")
)

openai_client = AzureOpenAI(
    azure_endpoint="https://<your-endpoint>.openai.azure.com/",
    api_key="<your-key>",
    api_version="2024-02-01"
)

def rag_query(user_question: str) -> str:
    # 1. AI Search로 관련 문서 검색
    results = search_client.search(user_question, top=3)
    context = "\n\n".join([r["content"] for r in results])

    # 2. Azure OpenAI로 응답 생성
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": f"다음 문서를 참고하여 답변하세요:\n\n{context}"},
            {"role": "user", "content": user_question}
        ]
    )
    return response.choices[0].message.content

answer = rag_query("연차 신청 절차를 알려줘")
print(answer)
```

### 병렬 학습 포인트

1. **검색 엔진**: Kendra(시맨틱) ↔ AI Search(벡터+하이브리드) — 검색 품질과 설정 복잡도 비교
2. **LLM 선택**: Bedrock 멀티모델(Claude/Titan/Llama) ↔ Azure OpenAI(GPT-4o) — 모델 라우팅/선택 전략
3. **임베딩**: Bedrock Titan Embeddings ↔ text-embedding-ada-002 — 벡터 인덱스 구축 방법
4. **가드레일/안전**: Bedrock Guardrails ↔ System Prompt+Content Safety — 응답 품질 제어 방식
5. **멀티테넌트**: Bedrock Knowledge Base 분리 ↔ AI Search Index per Tenant — 고객사별 지식 격리
