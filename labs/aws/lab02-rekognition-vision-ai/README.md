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
