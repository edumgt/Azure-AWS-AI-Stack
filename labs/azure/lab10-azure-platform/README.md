# ☁️ Lab 10 — Azure 플랫폼 시스템 환경

> Azure 클라우드 데이터·AI 플랫폼 핵심 인프라 실습 가이드  
> Microsoft Fabric · Azure Data Lake · Azure OpenAI 통합 아키텍처

---

## 📌 개요

| 항목 | 내용 |
|------|------|
| **목표** | Azure 데이터·AI 통합 플랫폼 환경 구성 및 운영 역량 확보 |
| **핵심 서비스** | Microsoft Fabric, Azure Data Lake Storage Gen2, Azure Synapse Analytics, Azure OpenAI, Azure Machine Learning |
| **대상** | 클라우드 아키텍트, 데이터 엔지니어, AI 플랫폼 엔지니어 |

---

## 1️⃣ Microsoft Fabric — 통합 분석 플랫폼

### 1.1 개요

Microsoft Fabric은 데이터 엔지니어링, 데이터 과학, 비즈니스 인텔리전스, 실시간 분석을 **단일 SaaS 플랫폼**으로 통합한 엔터프라이즈 분석 솔루션입니다.

```
Microsoft Fabric 구조

┌─────────────────────────────────────────────────────────────┐
│                    Microsoft Fabric                          │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌───────────┐  │
│  │ Data     │  │ Data     │  │ Data     │  │ Real-Time │  │
│  │Engineering│ │ Science  │  │Warehouse │  │Analytics  │  │
│  │(Spark)   │  │(ML/AI)   │  │(SQL)     │  │(KQL)      │  │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └─────┬─────┘  │
│       │             │             │               │         │
│  ┌────▼─────────────▼─────────────▼───────────────▼──────┐ │
│  │              OneLake (통합 데이터 레이크)               │ │
│  │          Delta Lake 형식, 단일 스토리지 계층            │ │
│  └──────────────────────────────────────────────────────┘ │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │           Power BI (시각화 / 보고)                   │  │
│  └─────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

### 1.2 Fabric 핵심 구성 요소

| 구성 요소 | 역할 | 특징 |
|----------|------|------|
| **OneLake** | 통합 데이터 레이크 | 테넌트당 1개, Delta Parquet 형식, 멀티클라우드 바로가기 |
| **Lakehouse** | 데이터 레이크 + SQL 결합 | Spark + SQL 동시 사용, Delta Lake 기반 |
| **Data Warehouse** | 엔터프라이즈 DW | T-SQL 완전 지원, 서버리스, 자동 확장 |
| **Data Factory** | ETL / 데이터 이동 | 900+ 커넥터, 코드리스 변환 |
| **Spark Notebooks** | 데이터 엔지니어링 | PySpark / Scala / R, MLflow 통합 |
| **KQL Database** | 실시간 분석 | 로그/시계열 고속 쿼리, EventStream 연동 |
| **Power BI** | 비즈니스 인텔리전스 | Direct Lake 모드로 OneLake 직접 접근 |
| **Data Activator** | 실시간 알림 | 데이터 조건 기반 자동 액션 트리거 |

### 1.3 Fabric Lakehouse 아키텍처

```
데이터 레이어 구성 (Medallion Architecture)

┌─────────────────────────────────────────────┐
│              Bronze Layer (Raw)              │
│  원본 데이터 그대로 적재 (스키마 미적용)       │
│  형식: Parquet / JSON / CSV                  │
└────────────────────┬────────────────────────┘
                     │ (변환·정제)
┌────────────────────▼────────────────────────┐
│              Silver Layer (Cleansed)         │
│  정제된 데이터 (중복 제거, 타입 변환, 조인)   │
│  형식: Delta Lake                            │
└────────────────────┬────────────────────────┘
                     │ (비즈니스 룰 적용)
┌────────────────────▼────────────────────────┐
│               Gold Layer (Curated)           │
│  분석 최적화 (Fact/Dimension, Aggregation)   │
│  Power BI Direct Lake 연결                   │
└─────────────────────────────────────────────┘
```

### 1.4 Fabric 보안 모델

| 보안 계층 | 설명 |
|----------|------|
| **Workspace RBAC** | Admin / Member / Contributor / Viewer 역할 |
| **OneLake 데이터 접근** | 아이템별 읽기/쓰기 권한, Row-level Security |
| **열 수준 보안** | SQL Warehouse에서 민감 컬럼별 접근 제어 |
| **Private Link** | 공용 인터넷 차단, VNet 통합 |
| **Customer Managed Key** | 고객 관리 암호화 키 (Azure Key Vault) |
| **Purview 통합** | 데이터 거버넌스, 데이터 계보(Lineage) 추적 |

---

## 2️⃣ Azure Data Lake Storage Gen2 (ADLS Gen2)

### 2.1 개요

ADLS Gen2는 Azure Blob Storage 위에 **계층적 네임스페이스(Hierarchical Namespace)**를 추가하여 빅데이터 분석에 최적화된 스토리지 서비스입니다.

```
ADLS Gen2 구조

Storage Account (계층적 네임스페이스 활성화)
│
├── Container: raw-data
│   ├── year=2025/
│   │   ├── month=01/
│   │   │   └── sales_20250101.parquet
│   │   └── month=02/
│   └── year=2026/
│
├── Container: processed-data
│   ├── silver/
│   └── gold/
│
└── Container: ml-artifacts
    ├── models/
    └── experiments/
```

### 2.2 ADLS Gen2 핵심 기능

| 기능 | 설명 |
|------|------|
| **계층적 네임스페이스** | 폴더 단위 POSIX ACL, 원자적 디렉토리 연산 |
| **Hive 파티셔닝** | `key=value` 형식 파티션으로 쿼리 최적화 |
| **수명 주기 관리** | 데이터 접근 패턴 기반 자동 Tier 이동 (Hot → Cool → Archive) |
| **내결함성** | ZRS(영역 중복) / GRS(지역 중복) 복제 |
| **통합 보안** | Azure AD + RBAC + POSIX ACL + SAS Token |
| **성능** | 초당 수백 GB 처리량, Hadoop 호환 ABFS 드라이버 |

### 2.3 데이터 접근 패턴별 Tier 전략

| Tier | 접근 빈도 | 저장 비용 | 접근 비용 | 적합한 데이터 |
|------|----------|----------|----------|-------------|
| **Hot** | 자주 (매일) | 높음 | 낮음 | Raw 수집 데이터, 운영 로그 |
| **Cool** | 가끔 (월 1회) | 중간 | 중간 | 실버 레이어, 과거 분석 결과 |
| **Cold** | 드물게 (연 1회) | 낮음 | 높음 | 6개월~2년 이상 과거 데이터 |
| **Archive** | 거의 없음 | 매우 낮음 | 매우 높음 | 규제 준수 보관 데이터 |

### 2.4 ADLS Gen2 + Synapse 통합

```python
# Azure Synapse에서 ADLS Gen2 접근 (Serverless SQL Pool)
-- 외부 테이블로 ADLS Gen2 Parquet 직접 쿼리
CREATE EXTERNAL TABLE sales_external (
    order_date      DATE,
    product_id      VARCHAR(50),
    region          VARCHAR(50),
    revenue         DECIMAL(18,2)
)
WITH (
    LOCATION = 'raw-data/year=2025/**',
    DATA_SOURCE = adls_gen2_source,
    FILE_FORMAT = parquet_format
);

-- 쿼리 실행 (서버리스, 비용은 스캔 데이터량 기준)
SELECT region, SUM(revenue) AS total_revenue
FROM sales_external
WHERE order_date >= '2025-01-01'
GROUP BY region
ORDER BY total_revenue DESC;
```

---

## 3️⃣ Azure AI · 분석 통합 인프라

### 3.1 전체 데이터·AI 플랫폼 아키텍처

```
┌─────────────────────────────────────────────────────────────────┐
│                    Azure Data & AI Platform                      │
│                                                                  │
│  ┌──────────────────────────────────────────────────────────┐   │
│  │                    수집 레이어                            │   │
│  │  Event Hub │ IoT Hub │ ADF │ Synapse Pipeline │ Logic Apps│   │
│  └──────────────────────┬───────────────────────────────────┘   │
│                          │                                       │
│  ┌───────────────────────▼──────────────────────────────────┐   │
│  │              저장 레이어 (OneLake / ADLS Gen2)            │   │
│  │    Bronze ──▶ Silver ──▶ Gold (Delta Lake / Parquet)     │   │
│  └───────────────────────┬──────────────────────────────────┘   │
│                          │                                       │
│  ┌───────────────────────▼──────────────────────────────────┐   │
│  │                 처리·분석 레이어                           │   │
│  │  Fabric Spark │ Synapse Spark │ Databricks │ Serverless   │   │
│  └───────────────┬───────────────────────────┬──────────────┘   │
│                  │                           │                   │
│  ┌───────────────▼────────┐  ┌──────────────▼───────────────┐  │
│  │    ML / AI 레이어       │  │    서빙 레이어                │  │
│  │  Azure ML │ OpenAI     │  │  Power BI │ API Management   │  │
│  │  Fabric DS │ AI Search │  │  Azure App Service │ AKS      │  │
│  └────────────────────────┘  └──────────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

### 3.2 Azure Synapse Analytics 구성 요소

| 구성 요소 | 설명 | 사용 시나리오 |
|----------|------|-------------|
| **Serverless SQL Pool** | 온디맨드 쿼리 (ADLS 직접 쿼리) | 탐색적 분석, 데이터 프로파일링 |
| **Dedicated SQL Pool** | 프로비저닝 DW (고성능 OLAP) | 정기 리포트, 대시보드 |
| **Apache Spark Pool** | Spark 클러스터 | ETL, ML, 스트리밍 처리 |
| **Data Explorer Pool** | KQL 기반 분석 | 로그·시계열 실시간 분석 |
| **Synapse Link** | CosmosDB → Synapse 실시간 분석 | 운영 DB 분석 (HTAP) |
| **Synapse Pipeline** | ADF 통합 ETL | 데이터 오케스트레이션 |

### 3.3 실시간 분석 파이프라인 (Streaming)

```
이벤트 소스                처리                    저장·서빙
────────────              ─────────               ──────────────
IoT 센서 ──▶
앱 로그  ──▶  Azure Event Hub  ──▶  Stream Analytics  ──▶  Power BI (실시간)
API 이벤트──▶  (파티션 32개)       또는 Fabric EventStream  ──▶  ADLS Gen2 (Cold Path)
                                    └──▶  Azure Functions  ──▶  CosmosDB (Hot Path)
```

### 3.4 Azure OpenAI 엔터프라이즈 통합

| 패턴 | 설명 | Azure 서비스 |
|------|------|-------------|
| **Private Endpoint** | 인터넷 차단, VNet 내부 통신 | Azure Private Link |
| **APIM 게이트웨이** | 인증, 속도 제한, 로깅 중앙화 | Azure API Management |
| **Content Safety** | 입출력 콘텐츠 필터링 | Azure AI Content Safety |
| **Prompt Flow** | 프롬프트 개발·평가·배포 | Azure AI Studio / Prompt Flow |
| **모니터링** | 토큰 사용량, 지연시간, 오류율 | Azure Monitor + Application Insights |
| **비용 최적화** | PTU(프로비저닝 처리 단위) vs. 종량제 | Azure OpenAI Quota 관리 |

```
┌────────────────────────────────────────────────────┐
│           Azure OpenAI 엔터프라이즈 구성             │
│                                                    │
│  클라이언트 앱                                      │
│      │                                             │
│      ▼                                             │
│  ┌──────────────┐                                  │
│  │ API Management│  ← 인증(AAD), 속도제한, 로깅      │
│  └──────┬───────┘                                  │
│         │ (Private Endpoint)                       │
│  ┌──────▼───────┐                                  │
│  │Azure OpenAI  │  ← GPT-4o, Embedding, DALL-E    │
│  │(Private)     │                                  │
│  └──────┬───────┘                                  │
│         │                                          │
│  ┌──────▼───────┐  ┌──────────────┐               │
│  │ AI Search    │  │ Cosmos DB    │               │
│  │ (Vector DB)  │  │ (세션/이력)  │               │
│  └──────────────┘  └──────────────┘               │
└────────────────────────────────────────────────────┘
```

---

## 4️⃣ 인프라 운영 및 거버넌스

### 4.1 비용 최적화 전략

| 전략 | 설명 | 대상 서비스 |
|------|------|------------|
| **자동 일시정지** | 미사용 시 Dedicated Pool 자동 중단 | Synapse Dedicated Pool |
| **Spot 인스턴스** | 학습 작업에 저비용 Spot VM 사용 | Azure ML Compute |
| **Reserved Instance** | 1~3년 약정으로 최대 65% 절감 | Azure OpenAI PTU, VM |
| **Data Lifecycle** | 오래된 데이터 자동 Cool/Archive 이동 | ADLS Gen2 |
| **쿼리 최적화** | 파티션 프루닝, 통계 업데이트 | Synapse SQL Pool |
| **태그 기반 비용 추적** | 프로젝트/팀별 비용 분리 | Azure Cost Management |

### 4.2 모니터링 체계

```
┌──────────────────────────────────────────────────────────┐
│                 모니터링 스택                              │
│                                                          │
│  메트릭 수집                                              │
│  ├── Azure Monitor (플랫폼 메트릭)                        │
│  ├── Application Insights (애플리케이션 성능)             │
│  └── Log Analytics Workspace (로그 중앙화)               │
│                                                          │
│  알림 / 대응                                              │
│  ├── Alert Rules → Action Groups → 이메일/SMS/Teams       │
│  ├── Logic Apps → 자동 스케일링 / 재시작                   │
│  └── Azure Automation → 자동 복구 스크립트               │
│                                                          │
│  시각화                                                   │
│  ├── Azure Monitor Dashboards                            │
│  ├── Grafana (Azure Managed Grafana)                     │
│  └── Power BI (비즈니스 KPI)                             │
└──────────────────────────────────────────────────────────┘
```

### 4.3 MLOps / LLMOps 운영 체계

| 영역 | 도구 | 설명 |
|------|------|------|
| **버전 관리** | MLflow (Azure ML 통합) | 모델/데이터/코드 버전 추적 |
| **CI/CD** | GitHub Actions + Azure ML SDK | 모델 학습·평가·배포 자동화 |
| **모델 레지스트리** | Azure ML Model Registry | 승인 게이트, 환경별 프로모션 |
| **A/B 테스트** | Azure ML Managed Endpoint | 트래픽 분할, 챔피언-챌린저 |
| **데이터 드리프트** | Azure ML Data Monitor | 입력 데이터 분포 변화 감지 |
| **프롬프트 관리** | Azure AI Studio / Prompt Flow | 프롬프트 버전, 평가, 배포 |
| **비용 모니터링** | Azure Cost Management + 커스텀 대시보드 | 모델·서비스별 비용 추적 |

### 4.4 Azure Well-Architected Framework 적용

| 기둥 | Azure 데이터·AI 플랫폼 적용 |
|------|---------------------------|
| **신뢰성** | ZRS/GRS 스토리지, 자동 장애 조치, 백업 정책 |
| **보안** | Private Endpoint, Managed Identity, Key Vault |
| **비용 최적화** | 자동 스케일링, Spot 인스턴스, 수명 주기 관리 |
| **운영 우수성** | IaC (Bicep/Terraform), GitOps, 자동화 테스트 |
| **성능 효율성** | 파티셔닝, 캐싱, CDN, 적절한 SKU 선택 |

---

## 5️⃣ 환경 구성 체크리스트

### 5.1 신규 프로젝트 환경 구성 순서

```
1. Azure 구독 및 리소스 그룹 설계
   └── 네이밍 컨벤션 수립 (rg-{env}-{project}-{region})

2. 네트워크 기반 구성
   ├── VNet / Subnet 설계
   ├── Private Endpoint 구성
   └── NSG / Azure Firewall 설정

3. 보안 기반 구성
   ├── Azure AD 그룹 및 RBAC 역할 정의
   ├── Key Vault 생성 및 접근 정책 설정
   └── Defender for Cloud 활성화

4. 스토리지 레이어 구성
   ├── ADLS Gen2 컨테이너 및 폴더 구조 설계
   ├── 수명 주기 정책 적용
   └── 진단 로그 → Log Analytics 연결

5. 처리 레이어 구성
   ├── Synapse Workspace 또는 Fabric Workspace 생성
   ├── Spark Pool 크기 및 자동 스케일링 설정
   └── ETL 파이프라인 구성

6. AI 레이어 구성
   ├── Azure OpenAI 배포 (모델 선택, 할당량 설정)
   ├── Azure AI Search 인덱스 생성
   └── Azure ML Workspace 생성

7. 서빙 레이어 구성
   ├── API Management 게이트웨이 설정
   ├── Power BI 워크스페이스 연결
   └── Application Insights 연동

8. 모니터링 및 거버넌스
   ├── Log Analytics Workspace 중앙화
   ├── Alert Rules 설정
   └── Azure Policy / Purview 거버넌스 설정
```

---

## 🔗 관련 Azure 서비스

| 서비스 | 역할 |
|--------|------|
| **Microsoft Fabric** | 통합 분석 플랫폼 (ETL·ML·BI 일체형) |
| **Azure Data Lake Storage Gen2** | 확장 가능한 빅데이터 저장소 |
| **Azure Synapse Analytics** | 엔터프라이즈 데이터 웨어하우스 |
| **Azure Databricks** | 고성능 Spark 기반 데이터 처리 |
| **Azure Machine Learning** | ML 실험·배포·MLOps |
| **Azure OpenAI Service** | LLM 기반 AI 기능 |
| **Azure AI Search** | 엔터프라이즈 검색 / RAG |
| **Azure API Management** | API 게이트웨이·보안·정책 |
| **Azure Monitor + Log Analytics** | 통합 모니터링 |
| **Microsoft Purview** | 데이터 거버넌스·카탈로그·계보 |
| **Azure Cost Management** | 클라우드 비용 추적·최적화 |

---

## 📚 참고 자료

- [Microsoft Fabric 공식 문서](https://learn.microsoft.com/fabric/)
- [Azure Data Lake Storage Gen2](https://learn.microsoft.com/azure/storage/blobs/data-lake-storage-introduction)
- [Azure Synapse Analytics](https://learn.microsoft.com/azure/synapse-analytics/)
- [Azure OpenAI 엔터프라이즈 구성](https://learn.microsoft.com/azure/ai-services/openai/how-to/managed-identity)
- [Azure Well-Architected Framework](https://learn.microsoft.com/azure/well-architected/)
- [Microsoft Purview 거버넌스](https://learn.microsoft.com/azure/purview/)

---

_Lab 10 | Azure 플랫폼 시스템 환경 | 최종 수정: 2026-04_
