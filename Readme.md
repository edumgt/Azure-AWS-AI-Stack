🌐 **언어 / Language:** [한국어](./Readme.md) | [English](./README_en.md) | [日本語](./README_ja.md) | [中文](./README_zh.md)

---

# AWS AI IaaS 기반 통합 AI 솔루션 제안서(브로셔)
**Lex · Rekognition · Textract · Bedrock · Kendra · Lambda 기반**  
**OCR/문서자동화 · PDF 출력물 · QRCode · 고객사별 AI Agent 제작**

---

## 1. 개요(Executive Summary)

본 문서는 AWS 상의 관리형 AI 서비스와 서버리스/클라우드 네이티브 기술 스택을 결합하여, 고객사별 요구사항(업무 프로세스, 데이터 거버넌스, 보안, SLA)에 맞춘 AI 솔루션을 설계·구축·운영하는 통합 제공 모델을 정리한 제안 브로셔입니다.

핵심 제공 영역은 다음과 같습니다.

1) 대화형 AI(콜센터/챗봇)  
2) 비전 AI(얼굴/객체/텍스트)  
3) 문서 AI(OCR·데이터 추출)  
4) 생성형 AI/RAG(사내 문서 기반 응답)  
5) 자동 문서 출력(PDF)  
6) QRCode 기반 업무 연결  
7) 고객사별 AI Agent 제작  

---

## 2. 솔루션 포트폴리오

### 2.1 대화형 AI - Amazon Lex 기반 업무 챗봇/음성봇

- 멀티턴 대화(인텐트/슬롯)로 업무 파라미터 수집 → Lambda 실행(내부 API/DB 연동)
- 콜센터/IVR/고객지원: FAQ, 주문/배송 조회, 계정/권한, 접수/민원, 예약/변경
- 음성 입출력 확장: Transcribe(음성→텍스트), Polly(텍스트→음성) 연계

### 2.2 비전 AI - Amazon Rekognition 기반 이미지/비디오 분석

- 얼굴 감지/분석: DetectFaces로 얼굴 위치·신뢰도·랜드마크·자세·가림 등 속성 반환
- 얼굴 비교/본인확인: 등록 사진 vs 제출 사진 비교, 중복 등록 탐지, 출입 인증
- 라벨(객체/상황) 감지 및 텍스트 감지로 콘텐츠 자동 태깅/검수 파이프라인 구성

### 2.3 문서 AI - Amazon Textract OCR/문서 데이터화

- 스캔 PDF/이미지에서 텍스트·필기·테이블·양식 데이터를 자동 추출
- 전표/영수증/계약서/신분증/점검표/검수서 등 문서 데이터화 및 검증 워크플로우 제공
- 규칙 기반 정제 + LLM 후처리(필드 표준화/요약/이상치 탐지)로 품질 고도화

### 2.4 생성형 AI/RAG - Bedrock · Kendra 기반 사내지식 응답

- 사내 문서(S3/포털/위키/정책/매뉴얼) 기반 검색(Kendra) + 생성형 응답(Bedrock)
- 답변 근거(출처 링크/문서 스니펫) 제공, 금칙어/PII 등 정책(가드레일) 적용
- 고객사별 테넌트/지식베이스 분리(권한, 암호화, 로깅 포함)

### 2.5 PDF 출력물 제작 - 리포트/증명서/검수서/계약서 자동 생성

- 템플릿 기반 PDF 렌더링(표/서식/도장/서명 이미지 포함), 대량 배치 출력
- 생성 이력/버전 관리, 워터마크, 암호/권한, 감사 로그
- 결과물 보관(S3) + 다운로드 API + 만료 정책(Lifecycle)

### 2.6 QRCode - 오프라인-온라인 업무 연결

- 문서/자산/현장 설비/출입증/작업지시서에 QRCode 삽입 → 모바일로 즉시 조회/등록/승인
- QRCode에는 토큰/서명 기반 짧은 URL(만료/1회성/권한 포함) 적용 가능
- 스캔 이벤트 로그/업무 히스토리 자동 수집(현장 DX)

### 2.7 고객사별 AI Agent 제작 - 업무 오케스트레이션 에이전트

- 고객사 시스템(API/DB/그룹웨어) 도구들을 안전하게 연결해 업무를 “대화로 실행”
- 예: “계약서 요약해 결재 올려줘”, “현장 점검 PDF 생성하고 담당자에게 배포”
- 정책 기반 도구 사용(권한/승인/감사), 테넌트 분리, 프롬프트/지식/툴셋 커스터마이징

---

## 3. 레퍼런스 아키텍처(대표 구성)

### 3.1 이벤트 기반 비전/문서 처리 파이프라인

- S3 업로드 → (S3 이벤트) Lambda → Rekognition/Textract → 결과 저장(DynamoDB/RDS/OpenSearch) → 알림(SNS)
- 트래픽 급증/재처리 필요 시: S3 → SQS → Lambda(버퍼링, **DLQ**, 재시도) 패턴 적용

### 3.2 Lex 업무 챗봇 + 내부 시스템 연동

- Lex(인텐트/슬롯) → Fulfillment Lambda → 내부 API(ERP/CRM/그룹웨어) → 결과 응답
- 장시간 작업: 진행상태 메시지(Progress update) + 비동기 Job(큐/Step Functions)로 UX 개선

### 3.3 RAG 지식봇(사내 문서 기반)

- 문서 수집/정제 → 인덱싱(Kendra/Vector) → 질의 → 검색 결과 → Bedrock 응답 생성 + 정책(가드레일)

---

## 4. 구축/운영 방법론

- **1단계(2~4주)**: 요구사항/보안/데이터 분류, PoC(정확도/지연/비용)
- **2단계(4~8주)**: MVP 구축(핵심 시나리오/권한/감사/배포 자동화)
- **3단계(지속)**: 고도화(품질, 사용자 피드백, 지식/툴 확장, 운영 지표 기반 최적화)

---

## 5. 필수 AWS 기술 스택(체크리스트)

### 5.1 Compute/Integration
- AWS Lambda, (선택) Step Functions, EventBridge
- API Gateway/ALB, SQS, SNS

### 5.2 Data/Storage
- S3(원본/결과물), DynamoDB(메타/이력), RDS(업무데이터), (선택) OpenSearch(검색)
- KMS(암호화), S3 Lifecycle/버전 관리

### 5.3 AI Services
- Lex(대화), Rekognition(비전), Textract(OCR), Bedrock(생성형), Kendra(검색/RAG)
- (선택) Transcribe/Polly(음성)

### 5.4 Security/Networking
- IAM(최소권한), VPC/보안그룹, (가능 시) VPC Endpoints/PrivateLink, WAF
- Secrets Manager/Parameter Store, CloudTrail(감사)

### 5.5 Observability/Operations
- CloudWatch Logs/Metrics/Alarms, X-Ray(분산추적), 장애 알림 연계

### 5.6 DevOps/IaC
- CloudFormation/CDK/Terraform, CodeBuild/CodePipeline 또는 GitHub Actions

---

## 6. 대표 적용 시나리오

- 현장 점검: QR 스캔 → 체크리스트 입력 → 사진 업로드 → Rekognition/Textract 자동 분석 → 점검 PDF 자동 생성/배포
- 출입/본인확인: 앱에서 촬영 → Rekognition 얼굴 비교 → 통과 시 출입증 PDF/QR 발급 + 감사로그 저장
- 문서 자동화: 계약서/전표 PDF 업로드 → Textract 추출 → 필드 검증 → RAG로 관련 규정/가이드 자동 제시
- 고객지원: Lex 음성봇이 문의 접수 → 내부 시스템 티켓 생성 → 진행상태를 챗으로 안내
- 사내 지식봇: 정책/매뉴얼 기반 검색+요약 응답(Bedrock/Kendra) + 근거 링크 제공

---

## 7. 참고(공식 문서/서비스 설명)

- [제안서 PDF 열기](./AWS_AI_Solution_Company_Brochure_ko.pdf)

- Amazon Rekognition DetectFaces API: https://docs.aws.amazon.com/rekognition/latest/APIReference/API_DetectFaces.html  
- AWS Lambda - Process Amazon S3 event notifications with Lambda: https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html  
- Amazon Lex V2 - Integrating Lambda: https://docs.aws.amazon.com/lexv2/latest/dg/lambda.html  
- Amazon Textract OCR: https://aws.amazon.com/textract/ocr/  
- Amazon Textract overview: https://aws.amazon.com/textract/  
