# ☁️ AWS AI Solution Labs

> AWS 관리형 AI 서비스와 서버리스/클라우드 네이티브 기술 스택을 결합한 통합 AI 솔루션 실습 가이드

🌐 **언어 / Language:** [한국어](../../Readme.md) | [English](../../i18n/README_en.md) | [日本語](../../i18n/README_ja.md) | [中文](../../i18n/README_zh.md)

📄 [제안서 PDF 열기](../../docs/assets/AWS_AI_Solution_Company_Brochure_ko.pdf)

---

## 📚 Lab 목록

| Lab | 주제 | 핵심 서비스 | 설명 |
|-----|------|------------|------|
| [Lab 01](./lab01-lex-conversational-ai/README.md) | 대화형 AI | Amazon Lex, Lambda, Transcribe, Polly | 업무 챗봇/음성봇 구축 |
| [Lab 02](./lab02-rekognition-vision-ai/README.md) | 비전 AI | Amazon Rekognition | 이미지/비디오 분석 (얼굴, 객체, 텍스트) |
| [Lab 03](./lab03-textract-document-ai/README.md) | 문서 AI | Amazon Textract | OCR/문서 데이터 자동 추출 |
| [Lab 04](./lab04-bedrock-rag/README.md) | 생성형 AI/RAG | Amazon Bedrock, Kendra | 사내 지식 기반 응답 시스템 |
| [Lab 05](./lab05-pdf-qrcode-automation/README.md) | 문서 자동화 | S3, Lambda | PDF 출력물 및 QRCode 기반 업무 연결 |
| [Lab 06](./lab06-ai-agent-orchestration/README.md) | AI Agent | Bedrock Agents, Step Functions | 고객사별 업무 오케스트레이션 에이전트 |
| [Lab 07](./lab07-architecture-devops/README.md) | 아키텍처/DevOps | CloudFormation, CDK, CodePipeline | 레퍼런스 아키텍처, 기술 스택, 운영 방법론 |

---

## 🏗️ 핵심 제공 영역

1. **대화형 AI** — 콜센터/챗봇 (Lex 기반)
2. **비전 AI** — 얼굴/객체/텍스트 인식 (Rekognition 기반)
3. **문서 AI** — OCR·데이터 추출 (Textract 기반)
4. **생성형 AI/RAG** — 사내 문서 기반 응답 (Bedrock·Kendra 기반)
5. **자동 문서 출력** — PDF 리포트/증명서 자동 생성
6. **QRCode 기반 업무 연결** — 오프라인-온라인 연결
7. **고객사별 AI Agent** — 업무 오케스트레이션 에이전트

---

## 🔧 필수 AWS 기술 스택

| 분류 | 서비스 |
|------|--------|
| Compute/Integration | Lambda, Step Functions, EventBridge, API Gateway, ALB, SQS, SNS |
| Data/Storage | S3, DynamoDB, RDS, OpenSearch, KMS |
| AI Services | Lex, Rekognition, Textract, Bedrock, Kendra, Transcribe, Polly |
| Security/Networking | IAM, VPC, VPC Endpoints/PrivateLink, WAF, Secrets Manager, CloudTrail |
| Observability | CloudWatch, X-Ray |
| DevOps/IaC | CloudFormation, CDK, Terraform, CodeBuild, CodePipeline, GitHub Actions |
