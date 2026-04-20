# Lab 07: 앱 개발 — Python 웹 프레임워크, Dapr, Application Insights

[← Azure Labs 목록으로](../README.md)

---

## 📌 개요

Python 기반 웹 프레임워크(Flask/Django)와 Azure 성능 구성 요소(Application Insights, Redis), 그리고 Dapr 분산 런타임을 활용한 마이크로서비스 개발을 학습합니다.

---

## 🎯 학습 목표

- Flask vs Django 특성 이해 및 선택 기준
- Application Insights 통합 (OpenCensus)
- Azure Redis를 활용한 세션/캐시 관리
- TTFB(Time To First Byte) 최적화
- Dapr 사이드카 아키텍처 이해 및 실습

---

## ✅ 1. Flask vs Django 사용 예시

### Flask 예시 (경량 API 서버)

```python
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello from Flask!"

if __name__ == "__main__":
    app.run(debug=True)
```

- 빠른 프로토타이핑, 단일 파일 앱 개발에 적합
- REST API 서버나 소규모 서비스에 활용

### Django 예시 (대규모 웹 앱)

```python
# views.py
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello from Django!")
```

- ORM, 인증, 관리자 페이지 등 내장
- 포털, ERP, LMS 등 복합 비즈니스 로직에 적합

---

## ✅ 2. Application Insights 예시 (Flask 통합)

```python
from opencensus.ext.azure.log_exporter import AzureLogHandler
import logging

logger = logging.getLogger(__name__)
logger.addHandler(AzureLogHandler(connection_string='InstrumentationKey=<your-key>'))

@app.route("/")
def hello():
    logger.warning("Hello route accessed")
    return "Logged to App Insights"
```

- Flask에서도 App Insights 로그 전송 가능
- HTTP 요청, 예외, 사용자 행동 추적 가능

---

## ✅ 3. Azure Redis 설정 (Python)

```python
import redis

r = redis.StrictRedis(
    host='myredis.redis.cache.windows.net',
    port=6380,
    password='your-access-key',
    ssl=True
)

r.set('session:userid:123', 'active')
value = r.get('session:userid:123')
```

- Azure Cache for Redis 생성 후 연결 정보 설정
- 세션, 코드 테이블, 조회 결과 캐싱에 활용

---

## ✅ 4. TTFB (Time To First Byte) 상세 설명

| 항목 | 설명 |
|------|------|
| 정의 | 브라우저가 서버에서 첫 번째 바이트를 수신하는 데 걸리는 시간 |
| 원인 | 느린 백엔드, 미적절한 DB 쿼리, 캐싱 미적용, DNS 지연 등 |

### 개선 방법
- CDN 및 Azure Front Door로 전방 캐시 활용
- API 응답 최적화 및 SQL 쿼리 튜닝
- Redis 등 캐시 시스템 도입
- App Insights로 응답 분해(trace) 및 병목 추적

---

## ✅ 5. Dapr (Distributed Application Runtime)

**Dapr**는 마이크로서비스를 손쉽게 개발할 수 있도록 도와주는 오픈소스 런타임입니다. 사이드카 아키텍처 기반으로, 다양한 언어와 클라우드 환경에서 분산 시스템의 복잡성을 추상화합니다.

### 🔹 주요 기능 (Building Blocks)

| 기능 | 설명 |
|------|------|
| 🔁 Service Invocation | HTTP/gRPC를 통한 서비스 간 통신 추상화 |
| 📦 State Management | Redis, Cosmos DB 등 다양한 저장소 백엔드 지원 |
| 📩 Pub/Sub Messaging | Kafka, Redis Streams, Azure Service Bus 등과 연동 |
| 🪪 Actor 모델 | 분산 Actor 패턴 지원 (가벼운 상태 기반 객체 관리) |
| 🔐 Secrets Management | Vault, AWS Secrets Manager 등에서 비밀 키 관리 |
| 📂 Bindings | 외부 시스템과의 트리거/입출력 연결 |
| 📈 Observability | Zipkin, Prometheus 등과 연계한 분산 트레이싱/로깅/메트릭 수집 |

### 🔹 Dapr 아키텍처

```
+---------------------+          +---------------------+
|  App (order-app)    | <---->   |  Dapr Sidecar       |
+---------------------+          +---------------------+
                                      |
                             +--------+--------+
                             | State Store     |
                             | Pub/Sub         |
                             | Secrets         |
                             +-----------------+
```

### 🔹 Python 예제: 서비스 간 통신

```python
import requests

# Dapr HTTP 포트 기본값은 3500
url = "http://localhost:3500/v1.0/invoke/order-service/method/process"

data = {
    "orderId": 123
}

response = requests.post(url, json=data)
print("응답:", response.text)
```

### 🔹 Node.js 예제: 상태 저장

```javascript
const axios = require('axios');

const daprPort = 3500;
const stateUrl = `http://localhost:${daprPort}/v1.0/state/statestore`;

const saveState = async () => {
  await axios.post(stateUrl, [
    { key: "username", value: "kim_doyoung" }
  ]);
  console.log("State saved.");
};

saveState();
```

### 🔹 Kubernetes 예제: Dapr 애플리케이션 배포

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: python-app
  template:
    metadata:
      labels:
        app: python-app
      annotations:
        dapr.io/enabled: "true"
        dapr.io/app-id: "python-app"
        dapr.io/app-port: "5000"
    spec:
      containers:
      - name: python-app
        image: myregistry/python-app:latest
        ports:
        - containerPort: 5000
```

### 🔹 개발 및 배포 환경

- 로컬: `dapr init`, `dapr run --app-id myapp --app-port 5000 python app.py`
- 운영: Docker, Kubernetes, Helm Chart 사용

---

## 📚 참고 자료

- [Application Insights](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
- [OpenCensus Python](https://github.com/census-instrumentation/opencensus-python)
- [Azure Cache for Redis](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/)
- [Dapr 공식 사이트](https://dapr.io)
- [Dapr GitHub](https://github.com/dapr/dapr)
