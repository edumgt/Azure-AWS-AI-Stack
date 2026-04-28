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

---

## ☁️ AWS ↔ Azure 전체 기술 스택 비교

> AWS와 Azure의 인프라·AI 서비스 전체를 카테고리별로 비교합니다.  
> Azure Lab 참조: [Azure Lab 05 — 인프라/DevOps](../../azure/lab05-infra-devops/README.md) | [Azure Lab 06 — 아키텍처/성능](../../azure/lab06-architecture-performance/README.md)

### 카테고리별 서비스 대응표

| 카테고리 | AWS 서비스 | Azure 대응 서비스 |
|---------|-----------|-----------------|
| **네트워킹** | VPC, Subnet, Security Group | VNet, Subnet, NSG |
| **프라이빗 연결** | VPC Endpoints, PrivateLink | Private Endpoint, Private Link |
| **하이브리드 연결** | AWS Direct Connect | Azure ExpressRoute |
| **VPN** | AWS VPN Gateway | Azure VPN Gateway |
| **NAT** | NAT Gateway | NAT Gateway |
| **네트워크 간 연결** | VPC Peering, Transit Gateway | VNet Peering, Virtual WAN |
| **WAF/DDoS** | AWS WAF, AWS Shield | Azure WAF, Azure DDoS Protection |
| **서버리스 컴퓨팅** | AWS Lambda | Azure Functions |
| **컨테이너 오케스트레이션** | Amazon EKS | Azure Kubernetes Service (AKS) |
| **서버리스 컨테이너** | AWS Fargate, App Runner | Azure Container Apps |
| **컨테이너 레지스트리** | Amazon ECR | Azure Container Registry (ACR) |
| **VM** | Amazon EC2 | Azure Virtual Machine |
| **PaaS 앱 호스팅** | AWS Elastic Beanstalk | Azure App Service |
| **오브젝트 스토리지** | Amazon S3 | Azure Blob Storage |
| **파일 스토리지** | Amazon EFS | Azure Files |
| **인메모리 캐시** | Amazon ElastiCache (Redis) | Azure Cache for Redis |
| **NoSQL DB** | Amazon DynamoDB | Azure Cosmos DB |
| **관계형 DB** | Amazon RDS | Azure SQL Database / Azure Database for PostgreSQL |
| **검색 엔진** | Amazon OpenSearch / Kendra | Azure AI Search (Cognitive Search) |
| **메시지 큐** | Amazon SQS | Azure Service Bus / Storage Queue |
| **메시지 브로커** | Amazon SNS | Azure Event Grid / Service Bus |
| **이벤트 스트리밍** | Amazon Kinesis | Azure Event Hubs |
| **워크플로우 오케스트레이션** | AWS Step Functions | Azure Durable Functions / Logic Apps |
| **인증/인가** | AWS IAM, Amazon Cognito | Azure AD (Entra ID), RBAC |
| **비밀 관리** | AWS Secrets Manager | Azure Key Vault |
| **감사 로그** | AWS CloudTrail | Azure Activity Log / Monitor |
| **모니터링/메트릭** | Amazon CloudWatch | Azure Monitor + Log Analytics |
| **분산 추적** | AWS X-Ray | Azure Application Insights |
| **IaC** | CloudFormation, CDK, Terraform | ARM/Bicep, Terraform |
| **CI/CD** | CodeBuild, CodePipeline | Azure DevOps, GitHub Actions |
| **대화형 AI** | Amazon Lex | Azure AI Language (CLU) + Bot Service |
| **비전 AI** | Amazon Rekognition | Azure AI Vision + Face |
| **문서 AI/OCR** | Amazon Textract | Azure AI Document Intelligence |
| **음성 인식** | Amazon Transcribe | Azure AI Speech (STT) |
| **음성 합성** | Amazon Polly | Azure AI Speech (TTS) |
| **LLM/생성형 AI** | Amazon Bedrock | Azure OpenAI Service |
| **AI Agent** | Bedrock Agents | Azure AI Agent Service (AI Foundry) |
| **ML 플랫폼** | Amazon SageMaker | Azure Machine Learning |
| **데이터 파이프라인** | AWS Glue, Athena | Azure Data Factory, Synapse Analytics |
| **CDN** | Amazon CloudFront | Azure CDN / Azure Front Door |
| **DNS** | Amazon Route 53 | Azure DNS |
| **글로벌 로드밸런싱** | AWS Global Accelerator | Azure Front Door |

### Well-Architected Framework 비교

| 축 | AWS | Azure |
|----|-----|-------|
| 운영 우수성 | Operational Excellence Pillar | Operational Excellence |
| 보안 | Security Pillar | Security |
| 안정성 | Reliability Pillar | Reliability |
| 성능 효율성 | Performance Efficiency Pillar | Performance Efficiency |
| 비용 최적화 | Cost Optimization Pillar | Cost Optimization |
| 지속 가능성 | Sustainability Pillar | Sustainability |

> AWS Well-Architected Framework ↔ Azure Well-Architected Framework — 6개 축이 동일하게 대응

### DevOps 파이프라인 비교

**AWS CI/CD**
```
GitHub → CodeBuild(빌드/테스트) → CodePipeline → ECR(이미지) → EKS/Lambda(배포)
```

**Azure CI/CD**
```
GitHub → Azure DevOps / GitHub Actions(빌드/테스트) → ACR(이미지) → AKS/Functions(배포)
```

### 병렬 학습 포인트

1. **네트워킹 기반**: VPC+Security Group ↔ VNet+NSG — 네트워크 격리 설계 원칙 동일 (Azure Lab 01 병행)
2. **컨테이너 오케스트레이션**: EKS ↔ AKS — Kubernetes 기반 동일, 관리형 Control Plane 차이 (Azure Lab 02 병행)
3. **보안 기반**: IAM+Cognito ↔ Azure AD+RBAC — 인증/인가 체계 비교 (Azure Lab 03 병행)
4. **IaC**: CloudFormation/CDK ↔ ARM/Bicep — 선언형 인프라 코드 비교 (Azure Lab 05 병행)
5. **Well-Architected**: AWS WAF ↔ Azure WAF — 6개 설계 축을 동일 기준으로 적용 (Azure Lab 06 병행)
