# ☁️ AI Solution Technology Stack — Lab Repository

> AWS AI 서비스 및 Azure 클라우드 인프라를 활용한 통합 솔루션 실습 가이드

🌐 **언어 / Language:** [English](./i18n/README_en.md) | [日本語](./i18n/README_ja.md) | [中文](./i18n/README_zh.md)

---

## 📁 Repository 구조

```
.
├── README.md                          ← 현재 문서 (메인 인덱스)
├── labs/
│   ├── aws/                           ← AWS AI 솔루션 Labs
│   │   ├── README.md                  ← AWS Labs 개요
│   │   ├── lab01-lex-conversational-ai/
│   │   ├── lab02-rekognition-vision-ai/
│   │   ├── lab03-textract-document-ai/
│   │   ├── lab04-bedrock-rag/
│   │   ├── lab05-pdf-qrcode-automation/
│   │   ├── lab06-ai-agent-orchestration/
│   │   └── lab07-architecture-devops/
│   └── azure/                         ← Azure 클라우드 Labs
│       ├── README.md                  ← Azure Labs 개요
│       ├── lab01-networking/
│       ├── lab02-compute-containers/
│       ├── lab03-security-governance/
│       ├── lab04-storage-cache/
│       ├── lab05-infra-devops/
│       ├── lab06-architecture-performance/
│       ├── lab07-app-development/
│       ├── lab08-ai-agent/            ← AI Agent 개발 (신규)
│       ├── lab09-ml-data-analysis/    ← ML / 데이터 분석 (신규)
│       └── lab10-azure-platform/      ← Azure 플랫폼 환경 (신규)
├── docs/
│   ├── intro.md                       ← 전문가 이력/프로필
│   └── assets/                        ← PDF 등 첨부 자료
├── i18n/                              ← 다국어 번역 (EN, JA, ZH)
```

---

## ☁️ AWS AI Solution Labs

AWS 관리형 AI 서비스를 결합한 **통합 AI 솔루션** 실습 가이드입니다.

| Lab | 주제 | 핵심 서비스 |
|-----|------|------------|
| [Lab 01](./labs/aws/lab01-lex-conversational-ai/README.md) | 대화형 AI (챗봇/음성봇) | Amazon Lex, Lambda, Transcribe, Polly |
| [Lab 02](./labs/aws/lab02-rekognition-vision-ai/README.md) | 비전 AI (얼굴/객체 분석) | Amazon Rekognition |
| [Lab 03](./labs/aws/lab03-textract-document-ai/README.md) | 문서 AI (OCR/데이터 추출) | Amazon Textract |
| [Lab 04](./labs/aws/lab04-bedrock-rag/README.md) | 생성형 AI / RAG | Amazon Bedrock, Kendra |
| [Lab 05](./labs/aws/lab05-pdf-qrcode-automation/README.md) | PDF/QRCode 문서 자동화 | S3, Lambda |
| [Lab 06](./labs/aws/lab06-ai-agent-orchestration/README.md) | AI Agent 오케스트레이션 | Bedrock Agents, Step Functions |
| [Lab 07](./labs/aws/lab07-architecture-devops/README.md) | 아키텍처 / DevOps | CloudFormation, CDK, CodePipeline |

> 📄 [AWS AI 솔루션 제안서 (PDF)](./docs/assets/AWS_AI_Solution_Company_Brochure_ko.pdf)

---

## ☁️ Azure Cloud Labs

Azure 클라우드 인프라 구성요소별 실습 가이드입니다.

| Lab | 주제 | 핵심 서비스 |
|-----|------|------------|
| [Lab 01](./labs/azure/lab01-networking/README.md) | 네트워킹 | VNet, Subnet, Private Endpoint, Gateway, ExpressRoute |
| [Lab 02](./labs/azure/lab02-compute-containers/README.md) | 컴퓨팅/컨테이너 | App Service, AKS, Container Apps, ACR, VM |
| [Lab 03](./labs/azure/lab03-security-governance/README.md) | 보안/거버넌스 | Azure AD, RBAC, Policy, Compliance |
| [Lab 04](./labs/azure/lab04-storage-cache/README.md) | 스토리지/캐시 | Storage Account, Redis Cache |
| [Lab 05](./labs/azure/lab05-infra-devops/README.md) | 인프라/DevOps | Key Vault, ARM/Bicep, Azure Arc, GitHub Actions |
| [Lab 06](./labs/azure/lab06-architecture-performance/README.md) | 아키텍처/성능 | Well-Architected Framework, Front Door, CDN |
| [Lab 07](./labs/azure/lab07-app-development/README.md) | 앱 개발 | Flask, Django, Dapr, Application Insights |
| [Lab 08](./labs/azure/lab08-ai-agent/README.md) | **AI Agent 개발** | Azure OpenAI, AI Search, Durable Functions, Logic Apps |
| [Lab 09](./labs/azure/lab09-ml-data-analysis/README.md) | **ML / 데이터 분석** | Azure ML, Synapse, Data Factory, Cosmos DB (Gremlin) |
| [Lab 10](./labs/azure/lab10-azure-platform/README.md) | **Azure 플랫폼 환경** | Microsoft Fabric, ADLS Gen2, Synapse, Power BI |

---

## 📋 문서 구조 변경 요약

기존의 루트 레벨 단일 파일들을 **Lab 스타일**로 재구성하였습니다:

| 변경 유형 | 설명 |
|-----------|------|
| **AWS (분할)** | 단일 `Readme.md`의 AWS AI 제안서를 7개 개별 Lab으로 분할 |
| **Azure (병합)** | 19개 개별 Azure .md 파일을 7개 주제별 Lab으로 병합 |
| **다국어** | `README_en/ja/zh.md`를 `i18n/` 디렉토리로 이동 |
| **지원 문서** | `Intro.md` → `docs/intro.md`, PDF → `docs/assets/` |




