# Lab 06: AI Agent — 고객사별 업무 오케스트레이션 에이전트

[← AWS Labs 목록으로](../README.md)

---

## 📌 개요

고객사 시스템(API/DB/그룹웨어)의 도구들을 안전하게 연결하여 업무를 "대화로 실행"하는 AI Agent를 구축합니다.

---

## 🎯 학습 목표

- AI Agent 아키텍처 이해 (도구/액션/지식 구성)
- 고객사별 테넌트 분리 구현
- 정책 기반 도구 사용 (권한/승인/감사)
- 프롬프트/지식/툴셋 커스터마이징

---

## 🔧 핵심 기능

- **업무 대화 실행**: 고객사 시스템(API/DB/그룹웨어) 도구들을 안전하게 연결해 업무를 "대화로 실행"
- **복합 업무 처리**:
  - 예: "계약서 요약해 결재 올려줘"
  - 예: "현장 점검 PDF 생성하고 담당자에게 배포"
- **정책 기반 실행**: 권한/승인/감사 기반 도구 사용
- **테넌트 분리**: 고객사별 프롬프트/지식/툴셋 커스터마이징

---

## 🏗️ 아키텍처

```
사용자 대화 → AI Agent (Bedrock)
                ├── Tool 1: 문서 조회 (S3/Kendra)
                ├── Tool 2: 데이터 처리 (Lambda → DB)
                ├── Tool 3: PDF 생성 (Lambda)
                ├── Tool 4: 결재 요청 (그룹웨어 API)
                └── Tool 5: 알림 발송 (SNS/이메일)
```

---

## 📋 활용 시나리오

- **계약서 워크플로우**: "계약서 요약해 결재 올려줘" → 문서 검색 → 요약 → 결재 시스템 연동
- **현장 점검 자동화**: "현장 점검 PDF 생성하고 담당자에게 배포" → 데이터 수집 → PDF 생성 → 이메일/알림
- **고객 문의 처리**: "이 고객의 최근 주문 상태 확인하고 안내해줘" → CRM 조회 → 안내 메시지 발송

---

## 🔗 관련 AWS 서비스

| 서비스 | 역할 |
|--------|------|
| Amazon Bedrock Agents | AI Agent 실행 런타임 |
| AWS Lambda | Tool/Action 실행 로직 |
| AWS Step Functions | 복합 워크플로우 오케스트레이션 |
| Amazon Kendra | 지식 검색 |
| Amazon S3 | 문서 저장 |

---

## 📚 참고 자료

- [Amazon Bedrock Agents](https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html)
- [AWS Step Functions](https://aws.amazon.com/step-functions/)

---

## ☁️ Azure 대응 서비스 비교

> 같은 목표를 Azure에서 구현할 때 사용하는 서비스와 개념을 비교합니다.  
> Azure Lab 참조: [Azure Lab 08 — AI Agent 개발](../../azure/lab08-ai-agent/README.md)

### 서비스 대응 매핑

| 기능 | AWS 서비스 | Azure 대응 서비스 |
|------|-----------|-----------------|
| AI Agent 실행 런타임 | Amazon Bedrock Agents | Azure AI Agent Service / Azure AI Foundry |
| Tool/Action 로직 실행 | AWS Lambda | Azure Functions |
| 복합 워크플로우 오케스트레이션 | AWS Step Functions | Azure Durable Functions |
| 지식 검색 (RAG) | Amazon Kendra + Bedrock Knowledge Base | Azure AI Search + Azure OpenAI |
| 에이전트 프롬프트/도구 관리 | Bedrock Agents 콘솔 | Azure AI Foundry (Prompt Flow) |
| 문서 저장 | Amazon S3 | Azure Blob Storage |
| 승인/알림 발송 | Amazon SNS / SES | Azure Communication Services / Event Grid |
| 감사 로그 | AWS CloudTrail + CloudWatch | Azure Monitor + Activity Log |

### 아키텍처 비교

**AWS (Bedrock Agents 기반 AI Agent)**
```
사용자 대화 → Bedrock Agent → (Tool Use) → Lambda(Tool 실행) → 내부 API/DB/S3
                              ↑                ↓
                         Step Functions(복합 워크플로우)
```

**Azure (Azure AI Agent 기반 AI Agent)**
```
사용자 대화 → Azure AI Agent → (Function Call) → Azure Functions(Tool 실행) → 내부 API/DB/Blob
                               ↑                   ↓
                          Durable Functions(복합 워크플로우 오케스트레이션)
```

### 핵심 개념 차이점

| 항목 | Amazon Bedrock Agents | Azure AI Agent Service |
|------|----------------------|----------------------|
| Tool 정의 방식 | Action Group (Lambda ARN + OpenAPI 스키마) | Function 정의 (JSON Schema) |
| 워크플로우 오케스트레이션 | Step Functions (State Machine) | Durable Functions (Orchestrator/Activity) |
| 메모리/컨텍스트 | Bedrock Agent 세션 | AI Agent Thread / Conversation |
| 다중 에이전트 협업 | Multi-Agent 설정 (Supervisor) | AutoGen / Semantic Kernel 멀티에이전트 |
| 코드 실행 도구 | Lambda 호출 | Azure Functions / Code Interpreter |
| Human-in-the-loop | Step Functions Wait State | Durable Functions External Event |

### Azure 실습 코드 예시 (Durable Functions — 업무 오케스트레이션)

```python
import azure.durable_functions as df

def orchestrator(context: df.DurableOrchestrationContext):
    # 1. 문서 검색 (AI Search)
    search_result = yield context.call_activity("SearchDocument", "계약서 표준 규정")

    # 2. LLM으로 요약 생성 (Azure OpenAI)
    summary = yield context.call_activity("SummarizeWithLLM", search_result)

    # 3. 결재 요청 (그룹웨어 API)
    approval_request = {"document": summary, "approver": "manager@company.com"}
    yield context.call_activity("RequestApproval", approval_request)

    # 4. 인간 승인 대기 (Human-in-the-loop)
    approval_event = context.wait_for_external_event("ApprovalDecision")
    approved = yield approval_event

    if approved:
        # 5. PDF 생성 및 배포
        pdf_url = yield context.call_activity("GeneratePDF", summary)
        yield context.call_activity("NotifyTeam", pdf_url)

    return {"status": "completed", "approved": approved}

main = df.Orchestrator.create(orchestrator)
```

### 병렬 학습 포인트

1. **Tool Use 패턴**: Bedrock Agents Action Group ↔ Azure AI Agent Function Call — LLM이 도구를 선택하고 실행하는 구조
2. **워크플로우**: Step Functions State Machine ↔ Durable Functions Orchestrator — 복합 업무 흐름 정의 방식
3. **Human-in-the-loop**: Step Functions Wait State ↔ Durable External Event — 인간 승인 대기 패턴
4. **멀티에이전트**: Bedrock Supervisor Agent ↔ AutoGen/Semantic Kernel — 에이전트 간 협업 구조
5. **감사/모니터링**: CloudTrail+CloudWatch ↔ Azure Monitor+Activity Log — Tool 실행 추적 및 감사
