# Lab 06: AI Agent — 고객사별 업무 오케스트레이션 에이전트

[← AWS Labs 목록으로](../README.md)

---

## 📌 개요

고객사 시스템(API/DB/그룹웨어)의 도구들을 안전하게 연결하여 업무를 "대화로 실행"하는 AI Agent를 구축합니다.

---

## 🎯 학습 목표

- AI Agent 아키텍처 이해 (도구/액션/지식 구성)
- 고객사별 테넌트 분리 구현
- 정책 기반 도구 사용 (권한/승인/감사)
- 프롬프트/지식/툴셋 커스터마이징

---

## 🔧 핵심 기능

- **업무 대화 실행**: 고객사 시스템(API/DB/그룹웨어) 도구들을 안전하게 연결해 업무를 "대화로 실행"
- **복합 업무 처리**:
  - 예: "계약서 요약해 결재 올려줘"
  - 예: "현장 점검 PDF 생성하고 담당자에게 배포"
- **정책 기반 실행**: 권한/승인/감사 기반 도구 사용
- **테넌트 분리**: 고객사별 프롬프트/지식/툴셋 커스터마이징

---

## 🏗️ 아키텍처

```
사용자 대화 → AI Agent (Bedrock)
                ├── Tool 1: 문서 조회 (S3/Kendra)
                ├── Tool 2: 데이터 처리 (Lambda → DB)
                ├── Tool 3: PDF 생성 (Lambda)
                ├── Tool 4: 결재 요청 (그룹웨어 API)
                └── Tool 5: 알림 발송 (SNS/이메일)
```

---

## 📋 활용 시나리오

- **계약서 워크플로우**: "계약서 요약해 결재 올려줘" → 문서 검색 → 요약 → 결재 시스템 연동
- **현장 점검 자동화**: "현장 점검 PDF 생성하고 담당자에게 배포" → 데이터 수집 → PDF 생성 → 이메일/알림
- **고객 문의 처리**: "이 고객의 최근 주문 상태 확인하고 안내해줘" → CRM 조회 → 안내 메시지 발송

---

## 🔗 관련 AWS 서비스

| 서비스 | 역할 |
|--------|------|
| Amazon Bedrock Agents | AI Agent 실행 런타임 |
| AWS Lambda | Tool/Action 실행 로직 |
| AWS Step Functions | 복합 워크플로우 오케스트레이션 |
| Amazon Kendra | 지식 검색 |
| Amazon S3 | 문서 저장 |

---

## 📚 참고 자료

- [Amazon Bedrock Agents](https://docs.aws.amazon.com/bedrock/latest/userguide/agents.html)
- [AWS Step Functions](https://aws.amazon.com/step-functions/)
