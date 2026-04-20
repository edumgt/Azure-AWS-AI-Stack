# Lab 07: 아키텍처/DevOps — 레퍼런스 아키텍처, 기술 스택, 운영 방법론

[← AWS Labs 목록으로](../README.md)

---

## 📌 개요

AWS AI 솔루션 구축에 필요한 전체 기술 스택, 레퍼런스 아키텍처, 구축/운영 방법론, 그리고 대표 적용 시나리오를 정리합니다.

---

## 🏗️ 레퍼런스 아키텍처 (대표 구성)

### 1. 이벤트 기반 비전/문서 처리 파이프라인

```
S3 업로드 → (S3 이벤트) Lambda → Rekognition/Textract → 결과 저장(DynamoDB/RDS/OpenSearch) → 알림(SNS)
```

- 트래픽 급증/재처리 필요 시: `S3 → SQS → Lambda` (버퍼링, DLQ, 재시도) 패턴 적용

### 2. Lex 업무 챗봇 + 내부 시스템 연동

```
Lex(인텐트/슬롯) → Fulfillment Lambda → 내부 API(ERP/CRM/그룹웨어) → 결과 응답
```

- 장시간 작업: 진행상태 메시지 + 비동기 Job(큐/Step Functions)으로 UX 개선

### 3. RAG 지식봇 (사내 문서 기반)

```
문서 수집/정제 → 인덱싱(Kendra/Vector) → 질의 → 검색 결과 → Bedrock 응답 생성 + 정책(가드레일)
```

---

## 📋 구축/운영 방법론

| 단계 | 기간 | 내용 |
|------|------|------|
| **1단계** | 2~4주 | 요구사항/보안/데이터 분류, PoC (정확도/지연/비용) |
| **2단계** | 4~8주 | MVP 구축 (핵심 시나리오/권한/감사/배포 자동화) |
| **3단계** | 지속 | 고도화 (품질, 사용자 피드백, 지식/툴 확장, 운영 지표 기반 최적화) |

---

## 🔧 필수 AWS 기술 스택 (체크리스트)

### Compute/Integration
- AWS Lambda, (선택) Step Functions, EventBridge
- API Gateway/ALB, SQS, SNS

### Data/Storage
- S3 (원본/결과물), DynamoDB (메타/이력), RDS (업무데이터), (선택) OpenSearch (검색)
- KMS (암호화), S3 Lifecycle/버전 관리

### AI Services
- Lex (대화), Rekognition (비전), Textract (OCR), Bedrock (생성형), Kendra (검색/RAG)
- (선택) Transcribe/Polly (음성)

### Security/Networking
- IAM (최소권한), VPC/보안그룹, (가능 시) VPC Endpoints/PrivateLink, WAF
- Secrets Manager/Parameter Store, CloudTrail (감사)

### Observability/Operations
- CloudWatch Logs/Metrics/Alarms, X-Ray (분산추적), 장애 알림 연계

### DevOps/IaC
- CloudFormation/CDK/Terraform, CodeBuild/CodePipeline 또는 GitHub Actions

---

## 🎯 대표 적용 시나리오

| 시나리오 | 구성 |
|----------|------|
| **현장 점검** | QR 스캔 → 체크리스트 입력 → 사진 업로드 → Rekognition/Textract 자동 분석 → 점검 PDF 자동 생성/배포 |
| **출입/본인확인** | 앱에서 촬영 → Rekognition 얼굴 비교 → 통과 시 출입증 PDF/QR 발급 + 감사로그 저장 |
| **문서 자동화** | 계약서/전표 PDF 업로드 → Textract 추출 → 필드 검증 → RAG로 관련 규정/가이드 자동 제시 |
| **고객지원** | Lex 음성봇이 문의 접수 → 내부 시스템 티켓 생성 → 진행상태를 챗으로 안내 |
| **사내 지식봇** | 정책/매뉴얼 기반 검색+요약 응답(Bedrock/Kendra) + 근거 링크 제공 |

---

## 📚 참고 자료

- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [AWS Architecture Center](https://aws.amazon.com/architecture/)
- [제안서 PDF 열기](../../../docs/assets/AWS_AI_Solution_Company_Brochure_ko.pdf)
