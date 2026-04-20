# Lab 04: 생성형 AI/RAG — Bedrock · Kendra 기반 사내지식 응답

[← AWS Labs 목록으로](../README.md)

---

## 📌 개요

Amazon Bedrock과 Kendra를 활용하여 사내 문서(S3/포털/위키/정책/매뉴얼) 기반의 검색 + 생성형 응답(RAG) 시스템을 구축합니다.

---

## 🎯 학습 목표

- RAG(Retrieval-Augmented Generation) 아키텍처 이해
- Kendra 인덱싱 및 검색 구성
- Bedrock 기반 생성형 응답 구현
- 정책(가드레일) 적용 및 테넌트 분리

---

## 🔧 핵심 기능

- **사내 지식 기반 응답**: 사내 문서(S3/포털/위키/정책/매뉴얼) 기반 검색(Kendra) + 생성형 응답(Bedrock)
- **답변 근거 제공**: 출처 링크/문서 스니펫 제공
- **정책 적용**: 금칙어/PII 등 정책(가드레일) 적용
- **멀티테넌트**: 고객사별 테넌트/지식베이스 분리 (권한, 암호화, 로깅 포함)

---

## 🏗️ 레퍼런스 아키텍처

### RAG 지식봇 (사내 문서 기반)

```
문서 수집/정제 → 인덱싱(Kendra/Vector) → 질의 → 검색 결과 → Bedrock 응답 생성 + 정책(가드레일)
```

---

## 📋 활용 시나리오

- **사내 지식봇**: 정책/매뉴얼 기반 검색+요약 응답(Bedrock/Kendra) + 근거 링크 제공
- **규정 안내**: "이 계약서에 적용되는 규정은?" → Kendra 검색 → Bedrock 요약 응답
- **신입사원 온보딩**: 사내 규정/프로세스 질의 → 자연어 응답 + 출처 제공

---

## 🔗 관련 AWS 서비스

| 서비스 | 역할 |
|--------|------|
| Amazon Bedrock | LLM 기반 생성형 응답 |
| Amazon Kendra | 사내 문서 검색 엔진 |
| Amazon S3 | 문서 원본 저장 |
| AWS Lambda | 검색-생성 오케스트레이션 |
| Amazon CloudWatch | 사용량/비용 모니터링 |

---

## 📚 참고 자료

- [Amazon Bedrock](https://aws.amazon.com/bedrock/)
- [Amazon Kendra](https://aws.amazon.com/kendra/)
- [RAG Architecture Patterns](https://docs.aws.amazon.com/bedrock/latest/userguide/knowledge-base.html)
