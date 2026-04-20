# Lab 01: 대화형 AI — Amazon Lex 기반 업무 챗봇/음성봇

[← AWS Labs 목록으로](../README.md)

---

## 📌 개요

Amazon Lex를 활용하여 멀티턴 대화형 업무 챗봇 및 음성봇을 구축합니다.  
인텐트/슬롯 기반의 대화 흐름으로 업무 파라미터를 수집하고, Lambda를 통해 내부 시스템과 연동합니다.

---

## 🎯 학습 목표

- Amazon Lex의 인텐트/슬롯 모델 이해
- Lambda Fulfillment를 통한 내부 API/DB 연동
- 음성 입출력 확장 (Transcribe/Polly)
- 콜센터/IVR 시나리오 구현

---

## 🔧 핵심 기능

- **멀티턴 대화**: 인텐트/슬롯으로 업무 파라미터 수집 → Lambda 실행 (내부 API/DB 연동)
- **콜센터/IVR/고객지원**: FAQ, 주문/배송 조회, 계정/권한, 접수/민원, 예약/변경
- **음성 입출력 확장**: Transcribe(음성→텍스트), Polly(텍스트→음성) 연계

---

## 🏗️ 레퍼런스 아키텍처

### Lex 업무 챗봇 + 내부 시스템 연동

```
Lex(인텐트/슬롯) → Fulfillment Lambda → 내부 API(ERP/CRM/그룹웨어) → 결과 응답
```

- 장시간 작업: 진행상태 메시지(Progress update) + 비동기 Job(큐/Step Functions)로 UX 개선

---

## 📋 활용 시나리오

- **고객지원**: Lex 음성봇이 문의 접수 → 내부 시스템 티켓 생성 → 진행상태를 챗으로 안내
- **주문 조회**: "주문번호 12345 배송 상태 알려줘" → Lex가 파라미터 추출 → Lambda가 ERP 조회

---

## 🔗 관련 AWS 서비스

| 서비스 | 역할 |
|--------|------|
| Amazon Lex V2 | 대화 흐름 관리 (인텐트/슬롯) |
| AWS Lambda | Fulfillment 로직 실행 |
| Amazon Transcribe | 음성 → 텍스트 변환 |
| Amazon Polly | 텍스트 → 음성 변환 |
| API Gateway | 외부 클라이언트와 연동 |

---

## 📚 참고 자료

- [Amazon Lex V2 - Integrating Lambda](https://docs.aws.amazon.com/lexv2/latest/dg/lambda.html)
- [Amazon Transcribe](https://aws.amazon.com/transcribe/)
- [Amazon Polly](https://aws.amazon.com/polly/)
