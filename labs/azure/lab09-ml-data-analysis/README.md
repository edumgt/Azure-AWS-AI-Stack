# 📊 Lab 09 — ML / 데이터 분석

> Azure 기반 머신러닝 및 데이터 분석 실습 가이드  
> Knowledge Graph · Data Pipeline (ETL) · 가설 기반 심층 데이터 분석

---

## 📌 개요

| 항목 | 내용 |
|------|------|
| **목표** | 비정형·정형 데이터를 결합한 지식 그래프 구축 및 심층 데이터 분석 파이프라인 구현 |
| **핵심 서비스** | Azure Machine Learning, Synapse Analytics, Azure Data Factory, Cosmos DB (Gremlin API), Azure OpenAI |
| **대상** | 데이터 엔지니어, 데이터 사이언티스트, ML 엔지니어 |

---

## 1️⃣ Knowledge Graph — 관계 그래프 개발 및 구축

### 1.1 개념

Knowledge Graph는 엔티티(Node)와 관계(Edge)를 정의하여 비정형 데이터 속의 **숨겨진 연결 관계**를 시각화·탐색할 수 있는 그래프 데이터베이스입니다.

```
Knowledge Graph 구조 예시

[고객: A사]──구매──▶[제품: AI솔루션]──분류──▶[카테고리: SaaS]
    │                        │
    └──담당──▶[담당자: 홍길동]  └──공급──▶[공급사: Microsoft]
                  │
                  └──소속──▶[부서: 영업팀]
```

### 1.2 Azure 구현 아키텍처

```
비정형 데이터 (문서/이메일/계약서)
        │
        ▼
┌──────────────────────────────┐
│   NLP 정보 추출               │
│   Azure AI Language          │
│   - Named Entity Recognition │
│   - Relation Extraction      │
│   - Key Phrase Extraction    │
└──────────────────────────────┘
        │ (Node, Edge 후보)
        ▼
┌──────────────────────────────┐
│   LLM 기반 트리플 추출        │
│   Azure OpenAI GPT-4o        │
│   (Subject, Predicate, Object)│
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│   Cosmos DB (Gremlin API)    │
│   또는 Azure Database for    │
│   PostgreSQL (Apache AGE)    │
└──────────────────────────────┘
        │
        ▼
┌──────────────────────────────┐
│   시각화 / 탐색               │
│   - Cosmos DB Data Explorer  │
│   - Gephi / Neo4j Browser    │
│   - Power BI 연동             │
└──────────────────────────────┘
```

### 1.3 Node / Edge 설계

#### Node (엔티티) 유형 정의

| Node 유형 | 속성 예시 | 설명 |
|----------|----------|------|
| `Person` | name, email, role, department | 사람 (직원, 고객, 담당자) |
| `Organization` | name, industry, region | 조직 (회사, 부서, 팀) |
| `Product` | name, category, version, price | 제품·서비스 |
| `Document` | title, date, source, type | 문서·계약·보고서 |
| `Event` | name, date, location, type | 이벤트·미팅·거래 |
| `Concept` | name, domain, description | 개념·주제·키워드 |

#### Edge (관계) 유형 정의

| Edge 유형 | 방향 | 가중치 | 설명 |
|----------|------|--------|------|
| `WORKS_FOR` | Person → Organization | - | 소속 관계 |
| `MANAGES` | Person → Person | - | 조직 계층 |
| `PURCHASES` | Organization → Product | 거래금액 | 구매 관계 |
| `AUTHORED` | Person → Document | - | 문서 작성 |
| `MENTIONS` | Document → Concept | 빈도 | 문서-개념 연결 |
| `SIMILAR_TO` | Concept → Concept | 유사도(0~1) | 의미 유사성 |
| `PARTICIPATED_IN` | Person → Event | 역할 | 참여 관계 |

### 1.4 LLM 기반 트리플 추출

```python
# Azure OpenAI로 (Subject, Predicate, Object) 트리플 추출
TRIPLE_EXTRACTION_PROMPT = """
다음 텍스트에서 지식 그래프용 트리플을 추출하세요.
형식: [{"subject": "...", "predicate": "...", "object": "..."}]

규칙:
- subject와 object는 명사구(엔티티)
- predicate는 관계 동사
- 최소 5개 이상 추출

텍스트:
{text}
"""

def extract_triples(text: str, openai_client) -> list[dict]:
    response = openai_client.chat.completions.create(
        model="gpt-4o",
        messages=[{"role": "user", "content": TRIPLE_EXTRACTION_PROMPT.format(text=text)}],
        response_format={"type": "json_object"}
    )
    return json.loads(response.choices[0].message.content)
```

### 1.5 Gremlin API 쿼리 예시 (Cosmos DB)

```groovy
// Node 추가
g.addV('Person').property('name', '홍길동').property('department', '영업팀')

// Edge 추가
g.V().has('Person', 'name', '홍길동')
 .addE('WORKS_FOR')
 .to(g.V().has('Organization', 'name', 'Microsoft Korea'))

// 2-hop 관계 탐색: 특정 회사와 연관된 모든 제품
g.V().has('Organization', 'name', 'Microsoft Korea')
 .out('OFFERS')
 .hasLabel('Product')
 .values('name')

// 최단 경로 찾기
g.V('person_001').repeat(both().simplePath()).until(has('name', '이순신')).path().limit(1)
```

### 1.6 GraphRAG 통합

Knowledge Graph를 RAG 파이프라인과 결합하여 **구조화된 관계 정보**를 LLM 컨텍스트에 주입합니다.

```
사용자 질문
    │
    ├──▶ 벡터 검색 (Azure AI Search) ──▶ 관련 청크
    └──▶ 그래프 탐색 (Cosmos DB)   ──▶ 연관 엔티티·관계
            │
            └──▶ 통합 컨텍스트 → GPT-4o → 응답
```

---

## 2️⃣ Data Pipeline — ETL 개발 및 Fact-table 구성

### 2.1 ETL 아키텍처

```
데이터 소스 (Source)
├── RDBMS (Oracle / MySQL / MSSQL)
├── SaaS API (Salesforce / SAP / 카카오)
├── 파일 (CSV / Excel / JSON / Parquet)
└── 실시간 스트림 (Kafka / Event Hub)
        │
        ▼ (Extract)
┌───────────────────────────────┐
│    Azure Data Factory (ADF)   │
│    또는 Azure Synapse Pipeline │
│    - Linked Service 구성       │
│    - Copy Activity             │
│    - 스케줄링 / 트리거          │
└───────────────────────────────┘
        │
        ▼ (Transform)
┌───────────────────────────────┐
│    Azure Databricks           │
│    또는 Synapse Spark Pool    │
│    - 데이터 정제 / 정규화       │
│    - 비즈니스 룰 적용           │
│    - Slowly Changing Dimension│
└───────────────────────────────┘
        │
        ▼ (Load)
┌───────────────────────────────┐
│    Azure Synapse Analytics    │
│    (Dedicated SQL Pool)       │
│    - Star Schema              │
│    - Fact Table               │
│    - Dimension Table          │
└───────────────────────────────┘
        │
        ▼ (Serve)
┌───────────────────────────────┐
│    Power BI / Azure Analysis  │
│    Services                   │
│    - 시각화 / 리포팅           │
└───────────────────────────────┘
```

### 2.2 Star Schema 설계 (Fact-table 구성)

```
                    ┌─────────────────┐
                    │  Dim_Date       │
                    │  date_key (PK)  │
                    │  year           │
                    │  quarter        │
                    │  month          │
                    │  day            │
                    └────────┬────────┘
                             │
┌──────────────┐    ┌────────▼────────┐    ┌──────────────┐
│  Dim_Product │    │  Fact_Sales     │    │ Dim_Customer │
│  product_key │◄───│  date_key (FK)  │───►│ customer_key │
│  name        │    │  product_key(FK)│    │ name         │
│  category    │    │  customer_key   │    │ region       │
│  brand       │    │  region_key (FK)│    │ segment      │
│  price_band  │    │  quantity       │    └──────────────┘
└──────────────┘    │  revenue        │
                    │  cost           │    ┌──────────────┐
                    │  profit         │    │  Dim_Region  │
                    │  discount_rate  │───►│  region_key  │
                    └─────────────────┘    │  country     │
                                           │  city        │
                                           └──────────────┘
```

### 2.3 SCD (Slowly Changing Dimension) 처리

| 유형 | 전략 | 사용 예 |
|------|------|--------|
| **Type 1** | 덮어쓰기 (이력 없음) | 오탈자 수정 |
| **Type 2** | 새 행 삽입 (이력 보존) | 고객 등급 변경, 가격 변경 |
| **Type 3** | 이전 값 컬럼 추가 | 현재/이전 담당자 |

```sql
-- SCD Type 2: Effective Date 패턴
CREATE TABLE Dim_Customer (
    customer_key    INT IDENTITY PRIMARY KEY,
    customer_id     VARCHAR(20),        -- 비즈니스 키
    customer_name   NVARCHAR(100),
    tier            VARCHAR(20),
    effective_from  DATE NOT NULL,
    effective_to    DATE,               -- NULL = 현재 유효
    is_current      BIT DEFAULT 1
);

-- 변경 발생 시: 기존 행 만료 처리 + 새 행 삽입
UPDATE Dim_Customer
SET effective_to = GETDATE(), is_current = 0
WHERE customer_id = 'C001' AND is_current = 1;

INSERT INTO Dim_Customer (customer_id, customer_name, tier, effective_from)
VALUES ('C001', '홍길동', 'GOLD', GETDATE());
```

### 2.4 Azure Data Factory 파이프라인 구성 요소

| 구성 요소 | 설명 |
|----------|------|
| **Linked Service** | 외부 데이터 소스/싱크 연결 정보 |
| **Dataset** | 데이터 구조 정의 (스키마, 포맷) |
| **Pipeline** | Activity의 논리적 묶음 |
| **Copy Activity** | 소스 → 싱크 데이터 복사 |
| **Data Flow** | 시각적 ETL 변환 (Spark 기반) |
| **Trigger** | 스케줄/이벤트/윈도우 기반 실행 |
| **Integration Runtime** | 실행 환경 (Azure IR / Self-hosted IR) |

---

## 3️⃣ Data Analysis — 가설 기반 심층 데이터 분석

### 3.1 분석 방법론

```
비즈니스 문제 정의
        │
        ▼
가설 수립 (Hypothesis)
"A 요인이 B 결과에 영향을 미친다"
        │
        ▼
데이터 수집 및 탐색 (EDA)
- 분포 확인, 이상치 탐지, 결측치 처리
        │
        ▼
통계적 검정 (Statistical Testing)
- t-test, ANOVA, Chi-Square, 상관 분석
        │
        ▼
모델링 / 예측 (Modeling)
- 회귀 / 분류 / 시계열
        │
        ▼
해석 및 인사이트 도출
- Feature Importance, SHAP, 비즈니스 번역
        │
        ▼
보고 및 의사결정 지원
- Power BI 대시보드, 경영진 보고서
```

### 3.2 Azure ML 기반 분석 워크플로우

| 단계 | 도구 | 설명 |
|------|------|------|
| **데이터 탐색** | Azure ML Notebooks (Jupyter) | EDA, 시각화, 프로파일링 |
| **데이터 전처리** | Azure ML Pipelines | 피처 엔지니어링, 스케일링 |
| **모델 학습** | Azure ML Jobs (Compute Cluster) | 분류/회귀/클러스터링 |
| **AutoML** | Azure AutoML | 자동 모델 선택 및 하이퍼파라미터 튜닝 |
| **실험 추적** | MLflow (Azure ML 통합) | 파라미터/메트릭/아티팩트 관리 |
| **모델 등록** | Azure ML Model Registry | 버전 관리, 스테이징 |
| **배포** | Azure ML Managed Endpoint | REST API 서빙 |
| **모니터링** | Azure ML Data Drift | 모델 품질 지속 모니터링 |

### 3.3 시계열 분석 (Time Series)

```python
# Azure ML AutoML 시계열 예측 예시
from azure.ai.ml import MLClient
from azure.ai.ml.automl import forecasting

forecasting_job = forecasting(
    training_data=train_data,
    target_column_name="revenue",
    primary_metric="normalized_root_mean_squared_error",
    time_column_name="date",
    time_series_id_column_names=["region", "product_category"],
    forecast_horizon=30,        # 30일 예측
    frequency="D",              # 일별
    seasonality=7,              # 주간 계절성
    allowed_training_algorithms=["Prophet", "TCNForecaster", "AutoArima"],
    max_trials=20,
    experiment_timeout_minutes=60
)

returned_job = ml_client.jobs.create_or_update(forecasting_job)
```

### 3.4 가설 검정 프레임워크

```python
import pandas as pd
from scipy import stats
import numpy as np

class HypothesisTester:
    """가설 기반 통계 검정 프레임워크"""

    def t_test(self, group_a: pd.Series, group_b: pd.Series,
               alpha: float = 0.05) -> dict:
        """독립 표본 t-검정: 두 그룹 평균 차이 검정"""
        stat, p_value = stats.ttest_ind(group_a, group_b)
        effect_size = (group_a.mean() - group_b.mean()) / np.sqrt(
            (group_a.std()**2 + group_b.std()**2) / 2
        )  # Cohen's d
        return {
            "test": "Independent t-test",
            "statistic": stat,
            "p_value": p_value,
            "significant": p_value < alpha,
            "effect_size_cohens_d": effect_size,
            "interpretation": self._interpret_effect(abs(effect_size))
        }

    def chi_square(self, contingency_table: pd.DataFrame,
                   alpha: float = 0.05) -> dict:
        """카이제곱 검정: 범주형 변수 독립성 검정"""
        chi2, p_value, dof, expected = stats.chi2_contingency(contingency_table)
        return {
            "test": "Chi-Square",
            "chi2_statistic": chi2,
            "p_value": p_value,
            "degrees_of_freedom": dof,
            "significant": p_value < alpha
        }

    @staticmethod
    def _interpret_effect(d: float) -> str:
        if d < 0.2: return "무시 가능한 효과"
        if d < 0.5: return "소 효과"
        if d < 0.8: return "중 효과"
        return "대 효과"
```

### 3.5 피처 중요도 및 모델 해석 (SHAP)

```python
import shap
import matplotlib.pyplot as plt

# SHAP 값 계산 (Azure ML 학습된 모델)
explainer = shap.TreeExplainer(trained_model)
shap_values = explainer.shap_values(X_test)

# 전역 피처 중요도 시각화
shap.summary_plot(shap_values, X_test, feature_names=feature_names)
plt.savefig("shap_summary.png")

# 개별 예측 설명 (Waterfall Chart)
shap.waterfall_plot(
    shap.Explanation(values=shap_values[0], base_values=explainer.expected_value,
                     data=X_test.iloc[0], feature_names=feature_names)
)
```

### 3.6 분석 결과 리포팅

| 구성 요소 | 도구 | 설명 |
|----------|------|------|
| **탐색 리포트** | Power BI / Azure Data Explorer | 인터랙티브 대시보드 |
| **통계 리포트** | Azure ML Jupyter Notebooks | 분석 과정 문서화 |
| **경영진 요약** | Azure OpenAI (자동 요약) | 자연어 인사이트 생성 |
| **알림/모니터링** | Azure Monitor + Logic Apps | KPI 임계치 알림 |

---

## 🔗 관련 Azure 서비스

| 서비스 | 역할 |
|--------|------|
| **Azure Machine Learning** | ML 실험, 학습, 배포, MLOps |
| **Azure Synapse Analytics** | 대규모 데이터 웨어하우스, Spark 분석 |
| **Azure Data Factory** | ETL 파이프라인, 데이터 이동 |
| **Azure Databricks** | Apache Spark 기반 빅데이터 처리 |
| **Azure Cosmos DB (Gremlin)** | 그래프 데이터베이스 |
| **Azure OpenAI** | LLM 기반 정보 추출, 자동 요약 |
| **Azure AI Language** | NER, 관계 추출, 감성 분석 |
| **Power BI** | 데이터 시각화, 비즈니스 리포팅 |
| **Azure Data Explorer** | 시계열/로그 고속 분석 |
| **Azure Purview** | 데이터 거버넌스, 카탈로그 |

---

## 📚 참고 자료

- [Azure Machine Learning 공식 문서](https://learn.microsoft.com/azure/machine-learning/)
- [Azure Synapse Analytics 개요](https://learn.microsoft.com/azure/synapse-analytics/)
- [Azure Data Factory 문서](https://learn.microsoft.com/azure/data-factory/)
- [Cosmos DB Gremlin API](https://learn.microsoft.com/azure/cosmos-db/gremlin/)
- [MLflow on Azure ML](https://learn.microsoft.com/azure/machine-learning/how-to-use-mlflow-cli-runs)

---

_Lab 09 | ML / 데이터 분석 | 최종 수정: 2026-04_
