# Lab 01: 대화형 AI — Amazon Lex 기반 업무 챗봇/음성봇

[← AWS Labs 목록으로](../README.md)

---

## 📌 개요

Amazon Lex를 활용하여 멀티턴 대화형 업무 챗봇 및 음성봇을 구축합니다.  
인텐트/슬롯 기반의 대화 흐름으로 업무 파라미터를 수집하고, Lambda를 통해 내부 시스템과 연동합니다.

---

## 🎯 학습 목표

- Amazon Lex의 인텐트/슬롯 모델 이해
- Lambda Fulfillment를 통한 내부 API/DB 연동
- 음성 입출력 확장 (Transcribe/Polly)
- 콜센터/IVR 시나리오 구현

---

## 🔧 핵심 기능

- **멀티턴 대화**: 인텐트/슬롯으로 업무 파라미터 수집 → Lambda 실행 (내부 API/DB 연동)
- **콜센터/IVR/고객지원**: FAQ, 주문/배송 조회, 계정/권한, 접수/민원, 예약/변경
- **음성 입출력 확장**: Transcribe(음성→텍스트), Polly(텍스트→음성) 연계

---

## 🏗️ 레퍼런스 아키텍처

### Lex 업무 챗봇 + 내부 시스템 연동

```
Lex(인텐트/슬롯) → Fulfillment Lambda → 내부 API(ERP/CRM/그룹웨어) → 결과 응답
```

- 장시간 작업: 진행상태 메시지(Progress update) + 비동기 Job(큐/Step Functions)로 UX 개선

---

## 📋 활용 시나리오

- **고객지원**: Lex 음성봇이 문의 접수 → 내부 시스템 티켓 생성 → 진행상태를 챗으로 안내
- **주문 조회**: "주문번호 12345 배송 상태 알려줘" → Lex가 파라미터 추출 → Lambda가 ERP 조회

---

## 🔗 관련 AWS 서비스

| 서비스 | 역할 |
|--------|------|
| Amazon Lex V2 | 대화 흐름 관리 (인텐트/슬롯) |
| AWS Lambda | Fulfillment 로직 실행 |
| Amazon Transcribe | 음성 → 텍스트 변환 |
| Amazon Polly | 텍스트 → 음성 변환 |
| API Gateway | 외부 클라이언트와 연동 |

---

## 📚 참고 자료

- [Amazon Lex V2 - Integrating Lambda](https://docs.aws.amazon.com/lexv2/latest/dg/lambda.html)
- [Amazon Transcribe](https://aws.amazon.com/transcribe/)
- [Amazon Polly](https://aws.amazon.com/polly/)

---

## ☁️ Azure 대응 서비스 비교

> 같은 목표를 Azure에서 구현할 때 사용하는 서비스와 개념을 비교합니다.  
> Azure Lab 참조: [Azure Lab 08 — AI Agent 개발](../../azure/lab08-ai-agent/README.md)

### 서비스 대응 매핑

| 기능 | AWS 서비스 | Azure 대응 서비스 |
|------|-----------|-----------------|
| 대화 흐름 관리 (인텐트/슬롯) | Amazon Lex V2 | Azure AI Language — CLU (Conversational Language Understanding) |
| FAQ / 지식 기반 응답 | Amazon Lex + Kendra | Azure AI Language — Question Answering (QnA Maker 후속) |
| 챗봇 채널 통합 (Slack, Teams 등) | Amazon Lex 채널 통합 | Azure Bot Service (Bot Framework) |
| 음성 → 텍스트 | Amazon Transcribe | Azure AI Speech — Speech-to-Text |
| 텍스트 → 음성 | Amazon Polly | Azure AI Speech — Text-to-Speech |
| Fulfillment 로직 실행 | AWS Lambda | Azure Functions |
| 외부 클라이언트 API | Amazon API Gateway | Azure API Management |

### 아키텍처 비교

**AWS (Lex 기반 챗봇)**
```
[사용자/채널] → Lex(인텐트/슬롯) → Lambda(Fulfillment) → 내부 API/DB → 응답
                     ↓
              Transcribe/Polly (음성 확장)
```

**Azure (Bot Service + CLU 기반 챗봇)**
```
[사용자/채널] → Bot Service → Azure AI Language(CLU) → Azure Functions → 내부 API/DB → 응답
                                     ↓
                         Speech-to-Text / Text-to-Speech (음성 확장)
```

### 핵심 개념 차이점

| 항목 | Amazon Lex | Azure AI Language (CLU) |
|------|-----------|------------------------|
| 학습 단위 | Intent + Slot | Intent + Entity |
| 대화 흐름 관리 | Built-in (Lex 엔진) | Bot Framework SDK로 직접 구현 |
| 채널 통합 | 자체 채널 연결 | Bot Service가 채널 통합 담당 |
| LLM 확장 | Bedrock 연동 | Azure OpenAI 연동 |
| 멀티턴 컨텍스트 | Lex 세션 | Bot Framework ConversationState |

### Azure 실습 코드 예시 (Azure Functions — Fulfillment)

```python
import azure.functions as func
import json

def main(req: func.HttpRequest) -> func.HttpResponse:
    body = req.get_json()
    intent = body.get("$instance", {}).get("intent", "")

    if intent == "OrderStatus":
        order_id = body.get("orderId")
        # 내부 ERP/CRM API 호출
        status = query_order_status(order_id)
        return func.HttpResponse(
            json.dumps({"status": status}),
            mimetype="application/json"
        )
    return func.HttpResponse("Intent not handled", status_code=400)

def query_order_status(order_id):
    # 실제 구현에서는 DB/API 호출
    return f"주문 {order_id}은 배송 중입니다."
```

### 병렬 학습 포인트

1. **인텐트 설계**: Lex의 Intent ↔ CLU의 Intent — 둘 다 사용자 의도를 분류하는 동일한 개념
2. **슬롯/엔티티 추출**: Lex Slot ↔ CLU Entity — 대화에서 필요한 파라미터 추출
3. **Fulfillment**: Lambda ↔ Azure Functions — 서버리스 로직 실행 구조 동일
4. **채널 배포**: Lex 채널 설정 ↔ Bot Service 채널 등록 — Slack/Teams/웹 등 동일 채널 지원
5. **음성 확장**: Transcribe+Polly ↔ Azure AI Speech — STT/TTS 파이프라인 구성 동일
