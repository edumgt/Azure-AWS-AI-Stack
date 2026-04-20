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
