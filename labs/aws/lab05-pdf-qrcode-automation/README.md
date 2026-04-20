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
