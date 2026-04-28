# Lab 03: 문서 AI — Amazon Textract OCR/문서 데이터화

[← AWS Labs 목록으로](../README.md)

---

## 📌 개요

Amazon Textract를 활용하여 스캔 PDF/이미지에서 텍스트, 테이블, 양식 데이터를 자동 추출하는 문서 AI 파이프라인을 구축합니다.

---

## 🎯 학습 목표

- Textract의 OCR 및 문서 분석 기능 이해
- 문서 데이터 추출 및 검증 워크플로우 구성
- LLM 후처리를 활용한 품질 고도화
- 이벤트 기반 문서 처리 파이프라인 구축

---

## 🔧 핵심 기능

- **자동 데이터 추출**: 스캔 PDF/이미지에서 텍스트·필기·테이블·양식 데이터를 자동 추출
- **문서 데이터화**: 전표/영수증/계약서/신분증/점검표/검수서 등 문서 데이터화 및 검증 워크플로우 제공
- **품질 고도화**: 규칙 기반 정제 + LLM 후처리 (필드 표준화/요약/이상치 탐지)

---

## 🏗️ 레퍼런스 아키텍처

### 이벤트 기반 문서 처리 파이프라인

```
S3 업로드 → (S3 이벤트) Lambda → Textract → 결과 저장(DynamoDB/RDS) → 알림(SNS)
```

- 트래픽 급증/재처리 필요 시: `S3 → SQS → Lambda` (버퍼링, DLQ, 재시도) 패턴 적용
- 비동기 처리: Textract StartDocumentAnalysis → SNS 알림 → Lambda 후처리

---

## 📋 활용 시나리오

- **문서 자동화**: 계약서/전표 PDF 업로드 → Textract 추출 → 필드 검증 → RAG로 관련 규정/가이드 자동 제시
- **현장 점검**: QR 스캔 → 체크리스트 입력 → 사진 업로드 → Textract 자동 분석 → 점검 PDF 자동 생성/배포

---

## 🔗 관련 AWS 서비스

| 서비스 | 역할 |
|--------|------|
| Amazon Textract | OCR 및 문서 구조 분석 |
| Amazon S3 | 원본 문서 저장 |
| AWS Lambda | 이벤트 기반 처리 로직 |
| Amazon DynamoDB | 추출 결과 메타데이터 저장 |
| Amazon SNS | 비동기 처리 완료 알림 |

---

## 📚 참고 자료

- [Amazon Textract OCR](https://aws.amazon.com/textract/ocr/)
- [Amazon Textract Overview](https://aws.amazon.com/textract/)

---

## ☁️ Azure 대응 서비스 비교

> 같은 목표를 Azure에서 구현할 때 사용하는 서비스와 개념을 비교합니다.  
> Azure Lab 참조: [Azure Lab 08 — AI Agent 개발](../../azure/lab08-ai-agent/README.md)

### 서비스 대응 매핑

| 기능 | AWS 서비스 | Azure 대응 서비스 |
|------|-----------|-----------------|
| OCR / 텍스트 추출 | Textract — DetectDocumentText | Azure AI Document Intelligence — Read |
| 양식(Form) 데이터 추출 | Textract — AnalyzeDocument (FORMS) | Azure AI Document Intelligence — General Document / Prebuilt |
| 테이블 구조 추출 | Textract — AnalyzeDocument (TABLES) | Azure AI Document Intelligence — Layout |
| 영수증/계약서 등 특화 추출 | Textract — Expense/Identity Analysis | Azure AI Document Intelligence — Prebuilt (Invoice, Receipt, ID) |
| 커스텀 문서 모델 학습 | Textract + A2I (Human Review) | Azure AI Document Intelligence — Custom Model |
| 이벤트 기반 처리 | S3 Event → Lambda | Blob Storage Event Grid → Azure Functions |
| 비동기 처리 완료 알림 | Textract SNS 콜백 | Azure Functions Durable (상태 조회) |
| 결과 저장 | DynamoDB / RDS | Cosmos DB / Azure SQL |

### 아키텍처 비교

**AWS (Textract 기반 문서 처리)**
```
S3 업로드 → Lambda → Textract StartDocumentAnalysis → SNS 완료 알림 → Lambda 후처리 → DynamoDB
```

**Azure (Document Intelligence 기반 문서 처리)**
```
Blob Storage 업로드 → Event Grid → Azure Functions → Document Intelligence → Cosmos DB
```

### 핵심 개념 차이점

| 항목 | Amazon Textract | Azure AI Document Intelligence |
|------|----------------|-------------------------------|
| 동기/비동기 | DetectDocumentText(동기) / StartDocumentAnalysis(비동기) | Analyze Document(비동기, polling 방식) |
| 사전 학습 모델 | Expense, Identity, Lending | Invoice, Receipt, ID Document, Business Card |
| 커스텀 모델 | Custom Queries (추가 설정) | Custom Template / Neural Model |
| 인간 검토 통합 | Amazon A2I (Augmented AI) | Azure AI Document Intelligence + 수동 검토 연계 |
| 출력 형식 | JSON (Block 기반) | JSON (Analyze Result, 필드/테이블 구조화) |

### Azure 실습 코드 예시 (Document Intelligence — 영수증 분석)

```python
from azure.ai.formrecognizer import DocumentAnalysisClient
from azure.core.credentials import AzureKeyCredential

def analyze_receipt(blob_url: str):
    client = DocumentAnalysisClient(
        endpoint="https://<your-endpoint>.cognitiveservices.azure.com/",
        credential=AzureKeyCredential("<your-key>")
    )

    poller = client.begin_analyze_document_from_url("prebuilt-receipt", blob_url)
    result = poller.result()

    for receipt in result.documents:
        merchant = receipt.fields.get("MerchantName")
        total = receipt.fields.get("Total")
        print(f"상점명: {merchant.value if merchant else 'N/A'}")
        print(f"합계: {total.value if total else 'N/A'}")

        items = receipt.fields.get("Items")
        if items:
            for item in items.value:
                name = item.value.get("Description")
                price = item.value.get("TotalPrice")
                print(f"  - {name.value if name else '?'}: {price.value if price else '?'}")
```

### 병렬 학습 포인트

1. **비동기 처리**: Textract StartDocumentAnalysis+SNS 콜백 ↔ Document Intelligence Analyze+polling — 장문서 처리 패턴 비교
2. **사전 학습 모델**: Textract Expense ↔ Document Intelligence Prebuilt-Receipt — 영수증/전표 필드 자동 추출
3. **커스텀 모델**: Textract Custom Queries ↔ Document Intelligence Custom Model — 도메인 특화 양식 학습
4. **LLM 후처리**: Textract → Bedrock ↔ Document Intelligence → Azure OpenAI — 추출 결과 요약/검증
5. **이벤트 파이프라인**: S3+SQS+Lambda ↔ Blob Storage+Event Grid+Functions — 대량 문서 배치 처리 구조
