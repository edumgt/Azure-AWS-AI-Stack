# Lab 02: 비전 AI — Amazon Rekognition 기반 이미지/비디오 분석

[← AWS Labs 목록으로](../README.md)

---

## 📌 개요

Amazon Rekognition을 활용하여 이미지 및 비디오에서 얼굴, 객체, 텍스트를 감지·분석하는 비전 AI 파이프라인을 구축합니다.

---

## 🎯 학습 목표

- Rekognition DetectFaces API 활용
- 얼굴 비교/본인확인 시나리오 구현
- 라벨(객체/상황) 감지 및 콘텐츠 태깅 파이프라인
- S3 이벤트 기반 자동 처리 파이프라인 구성

---

## 🔧 핵심 기능

- **얼굴 감지/분석**: DetectFaces로 얼굴 위치·신뢰도·랜드마크·자세·가림 등 속성 반환
- **얼굴 비교/본인확인**: 등록 사진 vs 제출 사진 비교, 중복 등록 탐지, 출입 인증
- **라벨(객체/상황) 감지**: 콘텐츠 자동 태깅/검수 파이프라인 구성
- **텍스트 감지**: 이미지 내 텍스트 자동 인식

---

## 🏗️ 레퍼런스 아키텍처

### 이벤트 기반 비전 처리 파이프라인

```
S3 업로드 → (S3 이벤트) Lambda → Rekognition → 결과 저장(DynamoDB/RDS/OpenSearch) → 알림(SNS)
```

- 트래픽 급증/재처리 필요 시: `S3 → SQS → Lambda` (버퍼링, DLQ, 재시도) 패턴 적용

---

## 📋 활용 시나리오

- **출입/본인확인**: 앱에서 촬영 → Rekognition 얼굴 비교 → 통과 시 출입증 PDF/QR 발급 + 감사로그 저장
- **현장 점검**: QR 스캔 → 사진 업로드 → Rekognition 자동 분석 → 점검 보고서 생성
- **콘텐츠 검수**: 업로드된 이미지의 라벨/텍스트 자동 태깅

---

## 🔗 관련 AWS 서비스

| 서비스 | 역할 |
|--------|------|
| Amazon Rekognition | 이미지/비디오 분석 (얼굴, 객체, 텍스트) |
| Amazon S3 | 원본 이미지/비디오 저장 |
| AWS Lambda | 이벤트 기반 처리 로직 |
| Amazon DynamoDB | 분석 결과 메타데이터 저장 |
| Amazon SNS | 알림 발송 |

---

## 📚 참고 자료

- [Amazon Rekognition DetectFaces API](https://docs.aws.amazon.com/rekognition/latest/APIReference/API_DetectFaces.html)
- [AWS Lambda - Process Amazon S3 event notifications](https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html)

---

## ☁️ Azure 대응 서비스 비교

> 같은 목표를 Azure에서 구현할 때 사용하는 서비스와 개념을 비교합니다.  
> Azure Lab 참조: [Azure Lab 08 — AI Agent 개발](../../azure/lab08-ai-agent/README.md)

### 서비스 대응 매핑

| 기능 | AWS 서비스 | Azure 대응 서비스 |
|------|-----------|-----------------|
| 얼굴 감지/분석 | Rekognition — DetectFaces | Azure AI Vision — Face Detection |
| 얼굴 비교/본인확인 | Rekognition — CompareFaces | Azure AI Face — Verify / Find Similar |
| 객체/라벨 감지 | Rekognition — DetectLabels | Azure AI Vision — Analyze Image (Tags/Objects) |
| 이미지 내 텍스트 감지 | Rekognition — DetectText | Azure AI Vision — OCR (Read API) |
| 콘텐츠 안전성 검사 | Rekognition — DetectModerationLabels | Azure AI Content Safety |
| 커스텀 객체 인식 학습 | Rekognition Custom Labels | Azure Custom Vision |
| 이벤트 기반 파이프라인 | S3 Event → Lambda | Blob Storage Event Grid → Azure Functions |
| 분석 결과 저장 | Amazon DynamoDB | Azure Cosmos DB |
| 알림 발송 | Amazon SNS | Azure Event Grid / Service Bus |

### 아키텍처 비교

**AWS (Rekognition 기반 비전 파이프라인)**
```
S3 업로드 → S3 Event → Lambda → Rekognition(DetectFaces/Labels) → DynamoDB → SNS 알림
```

**Azure (AI Vision 기반 비전 파이프라인)**
```
Blob Storage 업로드 → Event Grid → Azure Functions → Azure AI Vision → Cosmos DB → Service Bus 알림
```

### 핵심 개념 차이점

| 항목 | Amazon Rekognition | Azure AI Vision / Face |
|------|-------------------|----------------------|
| 얼굴 등록 컬렉션 | Collection (IndexFaces) | Person Group / Large Person Group |
| 얼굴 식별 | SearchFacesByImage | Identify API |
| 비디오 분석 | StartFaceDetection (비동기 Job) | Video Indexer (별도 서비스) |
| 커스텀 라벨 학습 | Rekognition Custom Labels | Azure Custom Vision |
| 콘텐츠 안전성 | Moderation Labels 내장 | Azure AI Content Safety (별도 서비스) |

### Azure 실습 코드 예시 (Azure Functions — 이미지 분석)

```python
import azure.functions as func
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential
import json

def main(event: func.EventGridEvent):
    blob_url = event.data["url"]

    client = ImageAnalysisClient(
        endpoint="https://<your-endpoint>.cognitiveservices.azure.com/",
        credential=AzureKeyCredential("<your-key>")
    )

    result = client.analyze_from_url(
        image_url=blob_url,
        visual_features=[VisualFeatures.TAGS, VisualFeatures.OBJECTS, VisualFeatures.PEOPLE]
    )

    tags = [{"name": t.name, "confidence": t.confidence} for t in result.tags.list]
    print(f"감지된 태그: {json.dumps(tags, ensure_ascii=False)}")
```

### 병렬 학습 포인트

1. **이벤트 파이프라인**: S3 Event+Lambda ↔ Event Grid+Azure Functions — 이미지 업로드 시 자동 처리 구조 동일
2. **라벨 감지**: DetectLabels ↔ Analyze Image(Tags) — 이미지 콘텐츠 자동 태깅 개념 동일
3. **얼굴 등록/식별**: Collection+IndexFaces ↔ PersonGroup+Person — 얼굴 DB 구축 방식 비교
4. **커스텀 모델**: Rekognition Custom Labels ↔ Azure Custom Vision — 도메인 특화 객체 인식 학습
5. **비동기 처리**: SQS 버퍼링+DLQ ↔ Service Bus+DLQ — 재처리/실패 패턴 동일
