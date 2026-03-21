🌐 **Language:** [한국어](./Readme.md) | [English](./README_en.md) | [日本語](./README_ja.md) | [中文](./README_zh.md)

---

# Integrated AI Solution Proposal (Brochure) — AWS AI IaaS
**Based on Lex · Rekognition · Textract · Bedrock · Kendra · Lambda**  
**OCR / Document Automation · PDF Output · QR Code · Customer-Specific AI Agent Development**

---

## 1. Executive Summary

This document is a proposal brochure outlining an integrated delivery model that combines AWS managed AI services with serverless/cloud-native technology stacks to design, build, and operate AI solutions tailored to each customer's requirements (business processes, data governance, security, and SLA).

Core service areas are as follows:

1. Conversational AI (Call Center / Chatbot)  
2. Vision AI (Face / Object / Text Recognition)  
3. Document AI (OCR · Data Extraction)  
4. Generative AI / RAG (Responses Based on Internal Documents)  
5. Automated Document Output (PDF)  
6. QR Code–Based Workflow Integration  
7. Customer-Specific AI Agent Development  

---

## 2. Solution Portfolio

### 2.1 Conversational AI — Amazon Lex-Based Business Chatbot / Voice Bot

- Multi-turn dialogue (intents/slots) to collect business parameters → Lambda execution (internal API/DB integration)
- Call center / IVR / customer support: FAQ, order/delivery lookup, account/permission management, request/complaint handling, reservation/changes
- Voice I/O extension: Transcribe (speech → text), Polly (text → speech) integration

### 2.2 Vision AI — Amazon Rekognition-Based Image / Video Analysis

- Face detection/analysis: DetectFaces returns face position, confidence, landmarks, pose, occlusion, and other attributes
- Face comparison / identity verification: Registered photo vs. submitted photo comparison, duplicate registration detection, access authentication
- Label (object/scene) detection and text detection for automatic content tagging/inspection pipeline

### 2.3 Document AI — Amazon Textract OCR / Document Digitization

- Automatic extraction of text, handwriting, tables, and form data from scanned PDFs/images
- Document digitization and validation workflows for receipts, invoices, contracts, IDs, inspection sheets, and acceptance forms
- Quality enhancement with rule-based refinement + LLM post-processing (field standardization / summarization / anomaly detection)

### 2.4 Generative AI / RAG — Bedrock · Kendra-Based Internal Knowledge Q&A

- Internal document (S3 / portal / wiki / policies / manuals) search (Kendra) + generative responses (Bedrock)
- Provides answer sources (reference links / document snippets); policy guardrails (prohibited words / PII, etc.) applied
- Per-customer tenant / knowledge base separation (including permissions, encryption, and logging)

### 2.5 PDF Output — Automatic Generation of Reports / Certificates / Inspection Sheets / Contracts

- Template-based PDF rendering (tables / forms / seal images / signatures included), bulk batch output
- Generation history / version management, watermarks, password/permission controls, audit logs
- Result storage (S3) + download API + expiration policy (Lifecycle)

### 2.6 QR Code — Offline-to-Online Workflow Integration

- Insert QR codes into documents / assets / field equipment / access badges / work orders → instant mobile lookup/registration/approval
- QR codes support token/signature-based short URLs (with expiry / single-use / permission controls)
- Automatic collection of scan event logs / business history (Field DX)

### 2.7 Customer-Specific AI Agent Development — Business Orchestration Agent

- Securely connects customer systems (API / DB / groupware) tools to enable business execution "through conversation"
- Examples: "Summarize the contract and submit it for approval", "Generate the field inspection PDF and distribute it to the person in charge"
- Policy-based tool usage (permissions / approval / audit), tenant separation, prompt / knowledge / toolset customization

---

## 3. Reference Architecture (Representative Configurations)

### 3.1 Event-Driven Vision / Document Processing Pipeline

- S3 upload → (S3 event) Lambda → Rekognition / Textract → Store results (DynamoDB / RDS / OpenSearch) → Notification (SNS)
- For traffic spikes / reprocessing: S3 → SQS → Lambda (buffering, **DLQ**, retry) pattern applied

### 3.2 Lex Business Chatbot + Internal System Integration

- Lex (intent/slot) → Fulfillment Lambda → Internal API (ERP / CRM / groupware) → Response
- Long-running tasks: Progress update messages + asynchronous jobs (queue / Step Functions) for improved UX

### 3.3 RAG Knowledge Bot (Based on Internal Documents)

- Document collection/refinement → Indexing (Kendra / Vector) → Query → Search results → Bedrock response generation + policy (guardrails)

---

## 4. Implementation / Operation Methodology

- **Phase 1 (2–4 weeks)**: Requirements / security / data classification, PoC (accuracy / latency / cost)
- **Phase 2 (4–8 weeks)**: MVP build (core scenarios / permissions / audit / deployment automation)
- **Phase 3 (ongoing)**: Enhancement (quality, user feedback, knowledge/tool expansion, operations metric-based optimization)

---

## 5. Required AWS Technology Stack (Checklist)

### 5.1 Compute / Integration
- AWS Lambda, (optional) Step Functions, EventBridge
- API Gateway / ALB, SQS, SNS

### 5.2 Data / Storage
- S3 (source / output), DynamoDB (metadata / history), RDS (business data), (optional) OpenSearch (search)
- KMS (encryption), S3 Lifecycle / versioning

### 5.3 AI Services
- Lex (conversation), Rekognition (vision), Textract (OCR), Bedrock (generative AI), Kendra (search / RAG)
- (optional) Transcribe / Polly (voice)

### 5.4 Security / Networking
- IAM (least privilege), VPC / security groups, (if possible) VPC Endpoints / PrivateLink, WAF
- Secrets Manager / Parameter Store, CloudTrail (audit)

### 5.5 Observability / Operations
- CloudWatch Logs / Metrics / Alarms, X-Ray (distributed tracing), failure alert integration

### 5.6 DevOps / IaC
- CloudFormation / CDK / Terraform, CodeBuild / CodePipeline or GitHub Actions

---

## 6. Representative Application Scenarios

- **Field Inspection**: QR scan → checklist input → photo upload → automatic Rekognition/Textract analysis → automatic inspection PDF generation/distribution
- **Access / Identity Verification**: Photograph via app → Rekognition face comparison → access certificate PDF/QR issuance upon pass + audit log storage
- **Document Automation**: Contract/invoice PDF upload → Textract extraction → field validation → automatic presentation of related regulations/guidelines via RAG
- **Customer Support**: Lex voice bot receives inquiry → internal system ticket creation → status updates provided via chat
- **Internal Knowledge Bot**: Policy/manual-based search + summary responses (Bedrock/Kendra) + reference links provided

---

## 7. References (Official Documentation / Service Descriptions)

- [Open Proposal PDF](./AWS_AI_Solution_Company_Brochure_ko.pdf)

- Amazon Rekognition DetectFaces API: https://docs.aws.amazon.com/rekognition/latest/APIReference/API_DetectFaces.html  
- AWS Lambda — Process Amazon S3 event notifications with Lambda: https://docs.aws.amazon.com/lambda/latest/dg/with-s3.html  
- Amazon Lex V2 — Integrating Lambda: https://docs.aws.amazon.com/lexv2/latest/dg/lambda.html  
- Amazon Textract OCR: https://aws.amazon.com/textract/ocr/  
- Amazon Textract overview: https://aws.amazon.com/textract/  
