# Lab 03: 보안/거버넌스 — 인증, RBAC, 정책, 컴플라이언스

[← Azure Labs 목록으로](../README.md)

---

## 📌 개요

Azure AD 기반 인증/인가, RBAC 정책 설계, 그리고 거버넌스 및 컴플라이언스 체계를 학습합니다.

---

## 🎯 학습 목표

- Azure AD 인증 및 MSAL OAuth2 구현
- RBAC 기반 권한 관리 및 Custom Role 설계
- Azure Policy, Blueprint를 활용한 거버넌스 자동화
- 컴플라이언스 프레임워크(ISO 27001, GDPR, KISA ISMS) 이해

---

## ✅ 1. Azure AD 기반 사용자 인증 (Authentication)

### 🔹 개요
Azure Active Directory(Azure AD)는 사용자 인증, 디바이스 인증, 멀티팩터 인증(MFA), SSO를 포함한 중앙 인증 서비스를 제공합니다.

### 🔹 주요 기능 요약

- **SSO (Single Sign-On)**: 한 번 로그인으로 M365, 자체 앱 등 연동
- **MFA**: OTP, 전화, 앱 인증 등 다단계 인증
- **Conditional Access**: 조건별 정책 설정 (IP, 디바이스, 앱)
- **B2B / B2C 인증**: 외부 협력사 초대 또는 고객용 로그인 처리
- **ID 연동**: Google, Okta, ADFS 등 외부 IdP 연계

---

## ✅ 2. MSAL 기반 OAuth2 인증 앱 예제 (Python Flask)

```python
from flask import Flask, redirect, url_for, session
from msal import ConfidentialClientApplication

app = Flask(__name__)
app.secret_key = 'YOUR_SECRET'

CLIENT_ID = 'your-client-id'
CLIENT_SECRET = 'your-client-secret'
AUTHORITY = 'https://login.microsoftonline.com/<tenant-id>'
REDIRECT_URI = 'http://localhost:5000/getAToken'
SCOPE = ['User.Read']

@app.route("/")
def index():
    return '<a href="/login">Login with Azure AD</a>'

@app.route("/login")
def login():
    app = ConfidentialClientApplication(CLIENT_ID, authority=AUTHORITY, client_credential=CLIENT_SECRET)
    auth_url = app.get_authorization_request_url(SCOPE, redirect_uri=REDIRECT_URI)
    return redirect(auth_url)

@app.route("/getAToken")
def authorized():
    return "Access Token received (handle parsing here)"
```

---

## ✅ 3. Azure RBAC 기반 사용자 인가 (Authorization)

### 🔹 주요 개념

- **Role**: 권한 집합 (예: Reader, Contributor, Custom)
- **Scope**: 권한 적용 범위 (Subscription, RG, 리소스 등)
- **Assignment**: 사용자/그룹에 Role + Scope 할당

### 🔹 기본 역할 예시

| 역할 | 설명 |
|------|------|
| Owner | 전체 권한 + 권한 부여 가능 |
| Contributor | 생성/수정 가능, 권한 부여는 불가 |
| Reader | 읽기 전용 |
| User Access Admin | 권한 관리 가능 |

### RBAC 정책 JSON 템플릿 (Custom Role 예)

```json
{
  "Name": "Read Storage Only",
  "IsCustom": true,
  "Description": "스토리지 계정만 읽기 가능한 역할",
  "Actions": [
    "Microsoft.Storage/storageAccounts/read"
  ],
  "NotActions": [],
  "AssignableScopes": [
    "/subscriptions/<subscription-id>"
  ]
}
```

### 인증/인가 흐름 요약

1. 사용자가 앱에 로그인 요청
2. Azure AD가 인증 후 ID Token/Access Token 발급
3. 앱은 Access Token을 포함하여 리소스 요청
4. Azure는 RBAC 정책 확인 후 접근 허용/거부

> 📌 최소 권한 원칙을 따르고, 그룹 기반 RBAC 및 관리형 ID를 통해 보안 수준을 향상시킬 것.

---

## ✅ 4. 거버넌스 (Governance)

### 🔎 정의
클라우드 자원의 사용, 관리, 보안, 비용 통제를 위한 정책, 역할, 구조를 설계하고 실행하는 체계

### 🔧 주요 구성 요소

- **Management Group**: 여러 서브스크립션을 묶어 조직 단위의 정책을 적용
- **Subscription**: 비용/부서/환경(Dev, QA, Prod) 별 구분 기준
- **Resource Group**: 논리적으로 리소스를 묶는 단위 (배포/권한 적용 단위)
- **Azure Policy**: 리전 제한, 태그 필수 등 규정 위반 리소스 생성을 방지
- **RBAC**: 최소 권한 원칙에 기반한 역할 분리
- **Tags**: 비용/부서/환경 구분을 위한 메타데이터
- **Blueprints**: 정책, 역할, 리소스 템플릿을 패키지화하여 표준 배포
- **Cost Management + Budget**: 예산 설정, 비용 추적, 알림 설정

### 📋 실무 적용 예시

#### 📁 부서별 관리 구조

```
Management Group
├── Finance Subscription
│   └── RG: finance-prod, finance-dev
├── HR Subscription
│   └── RG: hr-app, hr-data
```

#### 📜 정책 적용 예
- Azure Policy: `location == koreaCentral`
- Tag Policy: `{"Department": "required", "Project": "required"}`

#### 👥 RBAC 역할 분리
- 구독 수준: Owner (Infra팀)
- RG 수준: Contributor (개발팀)
- 자원 수준: Reader (모니터링 팀)

---

## ✅ 5. 컴플라이언스 (Compliance)

### 🔎 정의
내부 보안 정책, 산업 표준, 국가 법률에 부합하도록 클라우드 환경을 설정하고 감사 가능한 형태로 유지하는 것

### 📚 Azure 도구

- **Azure Policy**: 리소스 준수 여부 평가 및 제한
- **Defender for Cloud**: 보안 점수, 정책 준수도 제공
- **Compliance Manager**: 인증 보고서 확인
- **Audit Logs / Activity Logs**: 활동 기록 추적
- **Customer Lockbox**: Microsoft 직원 접근 통제
- **Private Link / VNet Integration**: 데이터 사설망 구성

### 📜 인증/규제 예시

| 인증/규제 | 설명 |
|-----------|------|
| ISO 27001 | 정보 보안 경영 시스템 |
| GDPR | 유럽 개인정보 보호법 |
| HIPAA | 미국 의료정보 보호법 |
| KISA ISMS | 한국 정보보호 관리체계 |
| FedRAMP | 미국 정부 클라우드 기준 |

---

## ✅ 6. 거버넌스와 컴플라이언스 통합 전략

| 전략 항목 | 설명 |
|-----------|------|
| 초기 설계 시 포함 | 아키텍처 설계 초기부터 정책 포함 |
| 정책 자동화 | Azure Policy, Blueprint로 자동화 |
| 감사 로그 보관 | 최소 90일 이상 저장, SIEM 연동 권장 |
| 최소 권한 원칙 | RBAC + PIM(Azure AD Privileged Identity Management) 병행 |
| 정기 점검 | 분기별 Compliance Score 보고서 작성 |

### 🧩 실무 요약

| 항목 | 도구/전략 |
|------|-----------|
| 표준화된 리소스 배포 | Blueprint, ARM, Bicep |
| 리소스 준수 여부 확인 | Azure Policy, Compliance Score |
| 비용 추적/관리 | Cost Management, Budget Alert |
| 사용자 접근 권한 관리 | RBAC, Azure AD Group |
| 규제 인증 대응 | Compliance Manager, 감사 로그, 정책 자동화 |

---

## 📚 참고 자료

- [Azure Active Directory](https://learn.microsoft.com/en-us/azure/active-directory/)
- [Azure RBAC](https://learn.microsoft.com/en-us/azure/role-based-access-control/)
- [Azure Policy](https://learn.microsoft.com/en-us/azure/governance/policy/)
- [Azure Compliance](https://learn.microsoft.com/en-us/azure/compliance/)

---

## ☁️ AWS 대응 서비스 비교

> 이 Lab의 Azure 보안/거버넌스 개념을 AWS에서 구현할 때 사용하는 서비스와 비교합니다.  
> AWS Lab 참조: [AWS Lab 07 — 아키텍처/DevOps](../../aws/lab07-architecture-devops/README.md)

### 서비스 대응 매핑

| Azure 개념/서비스 | AWS 대응 서비스 | 차이점 |
|-----------------|---------------|--------|
| Azure AD (Entra ID) | AWS IAM Identity Center (SSO) + Amazon Cognito | Azure AD는 엔터프라이즈 IdP 통합이 더 광범위 |
| RBAC (역할 기반 접근 제어) | AWS IAM (Policy + Role) | AWS는 Policy 기반, Azure는 Role Assignment 기반 |
| Custom Role | AWS Custom IAM Policy | 개념 동일, 문법 차이 |
| Managed Identity | IAM Role for EC2/Lambda (Instance Profile) | 코드 없이 서비스 인증, 개념 동일 |
| Azure Policy | AWS Config Rules + Service Control Policy (SCP) | Azure Policy가 더 세밀한 리소스 제어 |
| Management Group | AWS Organizations (OU 구조) | 계층형 정책 상속 개념 동일 |
| Blueprint | AWS Control Tower | 표준화된 환경 자동 배포 |
| Microsoft Defender for Cloud | AWS Security Hub + GuardDuty | 통합 보안 점수, 위협 탐지 |
| Azure Key Vault | AWS Secrets Manager + KMS | 비밀/인증서/키 관리, 기능 유사 |
| Azure AD Conditional Access | AWS IAM Condition Keys + Cognito MFA | 조건부 접근 정책 |
| Azure AD B2C | Amazon Cognito User Pools | 외부 고객 인증 (B2C) |

### 인증/인가 아키텍처 비교

**Azure (Azure AD 기반)**
```
사용자 → Azure AD (Entra ID) → MSAL OAuth2 토큰 발급 → App Service/API
                ↓
         Conditional Access (IP/디바이스/MFA 조건)
                ↓
         RBAC (Subscription/RG/Resource 스코프별 권한)
```

**AWS (IAM 기반)**
```
사용자 → IAM Identity Center (SSO) / Cognito → JWT 토큰 발급 → API Gateway/ALB
                ↓
         SCPs (Organization 수준 가드레일)
                ↓
         IAM Policy (Service/Resource/Condition 기반 권한)
```

### RBAC vs IAM Policy 비교

| 항목 | Azure RBAC | AWS IAM |
|------|-----------|---------|
| 권한 정의 방식 | Role (Actions 집합) → Scope에 Assignment | Policy (Effect/Action/Resource/Condition) |
| 최소 권한 원칙 | Custom Role로 세밀하게 정의 | Inline/Managed Policy로 정의 |
| 리소스 스코프 | Management Group → Subscription → RG → Resource | Organization → Account → Service → Resource |
| 상속 구조 | 상위 스코프에서 하위로 상속 | SCPs → 계정 → IAM Policy |
| 서비스 주체 | Service Principal / Managed Identity | IAM Role (Trust Policy) |

### AWS IAM 예시 (읽기 전용 S3 Custom Policy)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ],
      "Condition": {
        "StringEquals": {
          "aws:RequestedRegion": "ap-northeast-2"
        }
      }
    }
  ]
}
```

### 거버넌스 도구 비교

| 거버넌스 항목 | Azure | AWS |
|-------------|-------|-----|
| 계정/구독 계층 관리 | Management Group | AWS Organizations |
| 정책 자동 적용 | Azure Policy | AWS Config Rules + SCP |
| 표준 환경 배포 | Azure Blueprint | AWS Control Tower |
| 보안 점수 | Microsoft Defender for Cloud (Secure Score) | AWS Security Hub (Security Score) |
| 규제 컴플라이언스 | Compliance Manager | AWS Artifact + Security Hub Standards |
| 비용 거버넌스 | Azure Cost Management | AWS Cost Explorer + Budgets |

### 병렬 학습 포인트

1. **인증 흐름**: Azure AD + MSAL ↔ IAM Identity Center + Cognito — OAuth2/OIDC 토큰 기반 인증
2. **역할/정책 설계**: RBAC Custom Role ↔ IAM Custom Policy — 최소 권한 원칙 적용 방법
3. **서비스 인증**: Managed Identity ↔ IAM Role (Instance Profile) — 코드 없는 서비스 간 인증
4. **계층형 거버넌스**: Management Group ↔ AWS Organizations — 정책 상속 구조 비교
5. **컴플라이언스 자동화**: Azure Policy ↔ AWS Config Rules — 리소스 규정 준수 자동 검사
