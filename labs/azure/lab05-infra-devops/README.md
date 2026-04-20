# Lab 05: 인프라/DevOps — Key Vault, ARM/Bicep, Azure Arc, GitHub Actions, KQL

[← Azure Labs 목록으로](../README.md)

---

## 📌 개요

Azure 인프라 핵심 서비스(Key Vault, Application Gateway, Load Balancer)와 IaC(ARM/Bicep), 하이브리드 클라우드(Azure Arc), CI/CD(GitHub Actions), 그리고 모니터링 쿼리(KQL)를 학습합니다.

---

## 🎯 학습 목표

- Key Vault를 통한 시크릿 관리
- ARM Template / Bicep을 활용한 인프라 자동화
- Azure Arc를 통한 하이브리드 클라우드 관리
- GitHub Actions 기반 CI/CD 파이프라인 구축
- KQL을 활용한 로그 분석

---

## ✅ 1. Key Vault

- 암호, API 키, 인증서 등 보안 정보 저장
- RBAC 또는 정책 기반 액세스 제어
- 구성 예: `https://<vault-name>.vault.azure.net/secrets/dbPassword/`

---

## ✅ 2. Application Gateway + WAF

- L7(Web) 기반 로드 밸런서로 SSL 종료, 쿠키 기반 세션 유지
- WAF(Web Application Firewall) 포함: OWASP 룰 기반 공격 차단
- 경로 기반 라우팅, 인증서 관리 가능

---

## ✅ 3. Azure Load Balancer vs Traffic Manager

| 항목 | Load Balancer | Traffic Manager |
|------|---------------|-----------------|
| 계층 | L4 (TCP/UDP) | DNS 기반 |
| 사용 위치 | VM 간 부하 분산 | 글로벌 앱 라우팅 |
| 라우팅 방식 | IP, 포트 기반 | 가중치, 성능, 장애 조치 |
| 내부/외부 | 둘 다 가능 | 외부 트래픽 전용 |
| 특징 | 빠른 응답, 저지연 | 글로벌 고가용성 구성 |

---

## ✅ 4. ARM Template 예시

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2022-09-01",
      "name": "mystorageacct",
      "location": "koreacentral",
      "sku": { "name": "Standard_LRS" },
      "kind": "StorageV2",
      "properties": {}
    }
  ]
}
```

---

## ✅ 5. Bicep 예시

```bicep
resource storage 'Microsoft.Storage/storageAccounts@2022-09-01' = {
  name: 'mystorageacct'
  location: 'koreacentral'
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {}
}
```

---

## ✅ 6. Log Analytics 쿼리 (KQL)

```kusto
AzureActivity
| where ResourceGroup == "prod-rg"
| where ActivityStatus == "Failed"
| summarize Count = count() by ResourceProviderName, bin(TimeGenerated, 1h)
| order by Count desc
```

- 활동 로그에서 특정 그룹 내 실패 이벤트 분석
- `summarize`, `join`, `render` 등 다양한 시각화 가능

---

## ✅ 7. Blueprint

- Azure Policy + Role Assignment + Resource Template을 패키지화
- 예: 표준 태그, 위치 제한, 모니터링 설정 포함
- ID 기반으로 버전 관리 가능

```json
{
  "properties": {
    "displayName": "KISA ISMS 표준 정책",
    "description": "조직 보안 규정 준수용 블루프린트",
    "targetScope": "subscription",
    "parameters": {},
    "resourceGroups": {
      "core-rg": {
        "description": "공통 리소스 그룹"
      }
    }
  }
}
```

---

## ✅ 8. Reserved Instance

- VM, SQL, Cosmos DB 등에 1년 또는 3년 약정으로 비용 절감
- 최대 72% 할인 가능
- 예약은 서브스크립션 단위, 해지 가능 (수수료 발생)
- 예: Standard_D2s_v3 VM - 약정 시 시간당 비용 절감

---

## ✅ 9. Azure Arc

### 📌 정의
**Azure Arc**는 Azure 서비스 및 관리 기능을 **온프레미스, 다른 클라우드, 엣지 환경의 인프라에 확장**할 수 있도록 해주는 플랫폼입니다.

### 🔍 주요 기능

| 기능 | 설명 |
|------|------|
| **서버 관리** | 온프레미스 또는 AWS/GCP VM을 Azure 리소스처럼 등록하고 모니터링 가능 |
| **Kubernetes 관리** | 외부 K8s 클러스터를 AKS처럼 관리 |
| **데이터 서비스** | Azure SQL Managed Instance, PostgreSQL 등 온프레미스 운영 가능 |
| **정책 적용** | Azure Policy를 Arc 대상에 적용 가능 |
| **보안 통합** | Defender for Cloud, Log Analytics, Azure Monitor 연동 |

### 📦 지원 대상
- 물리 서버 / 가상 머신 (on-prem / AWS / GCP)
- Kubernetes 클러스터 (EKS, GKE 포함)
- 데이터 서비스 (SQL, PostgreSQL 등)

---

## ✅ 10. GitHub Actions + Python 사용 사례

### 📁 프로젝트 구조 예시

```
my-python-app/
├── app.py
├── requirements.txt
└── .github/
    └── workflows/
        └── python-app.yml
```

### 📄 `.github/workflows/python-app.yml`

```yaml
name: Python CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - name: 코드 체크아웃
      uses: actions/checkout@v3

    - name: Python 설치
      uses: actions/setup-python@v4
      with:
        python-version: '3.10'

    - name: 의존성 설치
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt

    - name: 유닛 테스트 실행
      run: |
        pytest

    - name: 애플리케이션 실행
      run: |
        python app.py
```

### 📄 `app.py`

```python
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from GitHub Actions + Python!"

if __name__ == "__main__":
    app.run()
```

### GitHub Actions 사용 효과

| 장점 | 설명 |
|------|------|
| CI/CD 자동화 | 코드 푸시 시 테스트 및 배포 자동 수행 |
| 협업 최적화 | Pull Request 병합 전 자동 검증 |
| Azure 연동 | Azure App Service, ACR 등과 통합 가능 |
| 유연한 런타임 | Python, Node.js, Java 등 지원 |

---

## ✅ 11. Bicep, KQL, Python 문법 비교

### Bicep (Azure Resource Manager DSL)

```bicep
param location string = resourceGroup().location

resource storageAccount 'Microsoft.Storage/storageAccounts@2022-09-01' = {
  name: 'mystorageacct'
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}
```

주요 키워드: `param`, `var`, `resource`, `output`, `module`

### KQL (Kusto Query Language)

```kql
AzureActivity
| where OperationName == "Create or Update Virtual Machine"
| summarize count() by ResourceGroup, bin(TimeGenerated, 1h)
```

주요 키워드: `where`, `summarize`, `project`, `extend`, `join`, `let`

### 비교 요약

| 요소 | Bicep | KQL | Python |
|------|-------|-----|--------|
| 목적 | Azure 리소스 정의 | 로그/데이터 분석 | 범용 프로그래밍 |
| 구조 | 선언형 | 쿼리 기반 | 절차적/객체지향 |
| 주석 | `//` | `//` | `#`, `'''...'''` |
| 변수 선언 | `param`, `var` | `let` | `=`, `def` 내부 |
| 문자열 | `'value'` | `"value"` | `'value'`, `"value"` |

---

## 📚 참고 자료

- [Azure Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/)
- [Azure ARM Templates](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/)
- [Bicep Language](https://learn.microsoft.com/en-us/azure/azure-resource-manager/bicep/)
- [Azure Arc](https://learn.microsoft.com/en-us/azure/azure-arc/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [KQL Reference](https://learn.microsoft.com/en-us/azure/data-explorer/kql-quick-reference)
