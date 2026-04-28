# Lab 05: 문서 자동화 — PDF 출력물 및 QRCode 기반 업무 연결

[← AWS Labs 목록으로](../README.md)

---

## 📌 개요

템플릿 기반 PDF 자동 생성과 QRCode를 활용한 오프라인-온라인 업무 연결을 구현합니다.

---

## 🎯 학습 목표

- 템플릿 기반 PDF 렌더링 구현
- QRCode 생성 및 업무 연결
- S3 기반 문서 보관 및 수명 주기 관리
- 보안(워터마크, 암호/권한) 적용

---

## 🔧 핵심 기능

### PDF 출력물 제작

- **템플릿 기반 PDF 렌더링**: 표/서식/도장/서명 이미지 포함, 대량 배치 출력
- **생성 이력/버전 관리**: 워터마크, 암호/권한, 감사 로그
- **결과물 보관**: S3 저장 + 다운로드 API + 만료 정책(Lifecycle)

### QRCode 기반 업무 연결

- **즉시 업무 처리**: 문서/자산/현장 설비/출입증/작업지시서에 QRCode 삽입 → 모바일로 즉시 조회/등록/승인
- **보안 URL**: QRCode에 토큰/서명 기반 짧은 URL (만료/1회성/권한 포함) 적용
- **활동 추적**: 스캔 이벤트 로그/업무 히스토리 자동 수집 (현장 DX)

---

## 📋 활용 시나리오

- **현장 점검**: QR 스캔 → 체크리스트 입력 → 사진 업로드 → 분석 → 점검 PDF 자동 생성/배포
- **출입증 발급**: 본인확인 통과 → 출입증 PDF/QR 발급 + 감사로그 저장
- **계약서 배포**: 계약서 PDF 생성 → QRCode 포함 → 서명 확인/이력 추적

---

## 🔗 관련 AWS 서비스

| 서비스 | 역할 |
|--------|------|
| Amazon S3 | PDF/QR 이미지 저장, Lifecycle 정책 |
| AWS Lambda | PDF 렌더링, QR 생성 로직 |
| Amazon DynamoDB | 생성 이력, 스캔 이벤트 로그 |
| Amazon API Gateway | 다운로드/조회 API |
| Amazon CloudFront | PDF 다운로드 가속 |

---

## 📚 참고 자료

- [Amazon S3 Lifecycle Configuration](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html)
- [AWS Lambda with API Gateway](https://docs.aws.amazon.com/lambda/latest/dg/services-apigateway.html)

---

## ☁️ Azure 대응 서비스 비교

> 같은 목표를 Azure에서 구현할 때 사용하는 서비스와 개념을 비교합니다.  
> Azure Lab 참조: [Azure Lab 07 — 앱 개발](../../azure/lab07-app-development/README.md)

### 서비스 대응 매핑

| 기능 | AWS 서비스 | Azure 대응 서비스 |
|------|-----------|-----------------|
| PDF 렌더링 로직 실행 | AWS Lambda | Azure Functions |
| QR 생성 로직 실행 | AWS Lambda | Azure Functions |
| PDF/QR 이미지 저장 | Amazon S3 | Azure Blob Storage |
| 파일 수명 주기 관리 | S3 Lifecycle Policy | Blob Storage Lifecycle Management |
| 다운로드 API | Amazon API Gateway | Azure API Management |
| 전역 다운로드 가속 | Amazon CloudFront | Azure CDN / Azure Front Door |
| 이력/이벤트 로그 저장 | Amazon DynamoDB | Azure Cosmos DB / Azure Table Storage |
| 서명 기반 보안 URL | S3 Pre-signed URL | Blob SAS (Shared Access Signature) |

### 아키텍처 비교

**AWS (S3 + Lambda 기반 PDF/QR 자동화)**
```
API Gateway → Lambda(PDF 생성/QR 생성) → S3 저장 → CloudFront CDN → Pre-signed URL 응답
                                         ↓
                                   DynamoDB(이력 저장)
```

**Azure (Blob Storage + Functions 기반 PDF/QR 자동화)**
```
API Management → Azure Functions(PDF 생성/QR 생성) → Blob Storage → Azure CDN → SAS URL 응답
                                                    ↓
                                             Cosmos DB(이력 저장)
```

### 핵심 개념 차이점

| 항목 | AWS | Azure |
|------|-----|-------|
| 보안 다운로드 URL | S3 Pre-signed URL (시간 만료, IAM 서명) | Blob SAS Token (시간 만료, 권한 제어) |
| 파일 만료 정책 | S3 Lifecycle Rules (일/버전 기반 삭제) | Blob Lifecycle Management (마지막 수정일 기준) |
| 서버리스 실행 단위 | Lambda (zip/container) | Azure Functions (in-process / isolated) |
| CDN 설정 | CloudFront Distribution | Azure CDN Profile + Endpoint |
| 대량 배치 출력 | Lambda + SQS 배치 처리 | Azure Functions + Service Bus / Storage Queue |

### Azure 실습 코드 예시 (Azure Functions — PDF 생성 + Blob 업로드)

```python
import azure.functions as func
from azure.storage.blob import BlobServiceClient, generate_blob_sas, BlobSasPermissions
from reportlab.pdfgen import canvas
from datetime import datetime, timedelta, timezone
import io, os

STORAGE_CONN = os.environ["AZURE_STORAGE_CONNECTION_STRING"]
CONTAINER = "pdfs"
ACCOUNT_NAME = os.environ["STORAGE_ACCOUNT_NAME"]
ACCOUNT_KEY = os.environ["STORAGE_ACCOUNT_KEY"]

def main(req: func.HttpRequest) -> func.HttpResponse:
    data = req.get_json()
    doc_id = data.get("doc_id", "doc001")

    # PDF 생성 (reportlab)
    buffer = io.BytesIO()
    c = canvas.Canvas(buffer)
    c.drawString(100, 750, f"문서 ID: {doc_id}")
    c.drawString(100, 720, f"생성일: {datetime.now().strftime('%Y-%m-%d %H:%M')}")
    c.save()
    buffer.seek(0)

    # Blob Storage 업로드
    blob_client = BlobServiceClient.from_connection_string(STORAGE_CONN)
    blob_name = f"{doc_id}.pdf"
    blob_client.get_container_client(CONTAINER).upload_blob(blob_name, buffer, overwrite=True)

    # SAS URL 생성 (1시간 유효)
    sas_token = generate_blob_sas(
        account_name=ACCOUNT_NAME,
        container_name=CONTAINER,
        blob_name=blob_name,
        account_key=ACCOUNT_KEY,
        permission=BlobSasPermissions(read=True),
        expiry=datetime.now(timezone.utc) + timedelta(hours=1)
    )
    sas_url = f"https://{ACCOUNT_NAME}.blob.core.windows.net/{CONTAINER}/{blob_name}?{sas_token}"

    return func.HttpResponse(sas_url, status_code=200)
```

### 병렬 학습 포인트

1. **보안 URL**: S3 Pre-signed URL ↔ Blob SAS Token — 만료 시간/권한 제어 방식 비교
2. **수명 주기**: S3 Lifecycle ↔ Blob Lifecycle Management — 자동 삭제/Tier 이동 정책 설정
3. **서버리스**: Lambda ↔ Azure Functions — PDF/QR 렌더링 로직을 서버리스로 실행
4. **CDN 가속**: CloudFront ↔ Azure CDN/Front Door — 대용량 파일 다운로드 최적화
5. **배치 처리**: SQS+Lambda ↔ Service Bus+Functions — 대량 PDF 생성 시 큐 기반 처리
