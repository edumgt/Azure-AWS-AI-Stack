# Lab 02: 컴퓨팅/컨테이너 — App Service, AKS, Container Apps, ACR, VM

[← Azure Labs 목록으로](../README.md)

---

## 📌 개요

Azure의 다양한 컴퓨팅 서비스를 학습합니다. PaaS(App Service), 컨테이너 오케스트레이션(AKS), 서버리스 컨테이너(Container Apps), 컨테이너 레지스트리(ACR), 가상 머신(VM)을 포함합니다.

---

## 🎯 학습 목표

- App Service와 Front Door를 활용한 글로벌 웹 서비스 구성
- AKS 기반 Kubernetes 클러스터 배포 및 운영
- Container Apps를 활용한 서버리스 마이크로서비스 배포
- ACR로 프라이빗 컨테이너 이미지 관리
- VM SKU 선택 및 비용 최적화

---

## ✅ 1. Azure App Service

Azure App Service는 웹 애플리케이션, REST API, 모바일 백엔드를 위한 완전 관리형 플랫폼(PaaS)입니다.

### 주요 기능
- 다양한 언어 지원: Python, .NET, Java, Node.js 등
- 자동 확장 및 로드 밸런싱
- GitHub, Azure DevOps 등의 지속적 배포 통합
- TLS/SSL, 사용자 지정 도메인, Azure AD 통합 등 보안 기능
- Application Insights를 통한 모니터링 및 진단

### Python 예제 (Flask 앱)

```python
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello from Azure App Service!'

if __name__ == '__main__':
    app.run()
```

---

## ✅ 2. Azure Front Door

Azure Front Door는 글로벌 고성능 애플리케이션 전송 네트워크로, CDN과 웹 애플리케이션 방화벽(WAF)을 포함합니다.

### 주요 기능
- 글로벌 로드 밸런싱 (가장 가까운 백엔드로 라우팅)
- 동적 콘텐츠 전송 가속
- WAF를 통한 보안 기능 (OWASP Top 10 보호)
- 종단 간 SSL/TLS 암호화
- 자동 장애 조치

### 🧩 통합 아키텍처 예시

```
[사용자]
   |
   v
[Azure Front Door]
   |
   v
[Azure App Service (Flask 앱)]
```

---

## ✅ 3. Azure Kubernetes Service (AKS)

**AKS**는 Microsoft Azure에서 제공하는 **완전관리형 Kubernetes 클러스터 서비스**입니다.

### 주요 구성 요소

| 구성 요소 | 설명 |
|-----------|------|
| **컨트롤 플레인** | Azure가 관리. 클러스터 상태 모니터링, 작업 스케줄링 등 수행 |
| **노드 풀(Node Pool)** | 실제 애플리케이션이 실행되는 VM 노드들 |
| **Pod** | 컨테이너가 실행되는 기본 단위 |
| **서비스(Service)** | Pod에 대한 네트워크 액세스를 안정적으로 제공 |
| **Persistent Volume** | 스토리지를 Pod에 연결해 상태 저장 가능 |
| **Helm** | Kubernetes 앱의 패키징 및 배포 도구 |

### 주요 기능 및 이점
- 완전관리형 Control Plane: Kubernetes 마스터 노드 관리 불필요
- 자동 패치, 업그레이드, 보안 관리 포함

### AKS 배포 예제 (Azure CLI)

```bash
# 리소스 그룹 생성
az group create --name myAKSGroup --location koreacentral

# AKS 클러스터 생성
az aks create \
  --resource-group myAKSGroup \
  --name myAKSCluster \
  --node-count 2 \
  --enable-addons monitoring \
  --generate-ssh-keys

# kubectl 연결
az aks get-credentials --resource-group myAKSGroup --name myAKSCluster
```

---

## ✅ 4. Azure Container Apps

**Azure Container Apps**는 마이크로서비스 및 컨테이너 기반 애플리케이션을 서버리스 방식으로 실행할 수 있는 **PaaS**입니다.

### 주요 특징

| 항목 | 설명 |
|------|------|
| 서버리스 컨테이너 | 인프라 관리 없이 컨테이너 배포 |
| 자동 스케일링 | 요청 수, 큐 길이, 이벤트 수신량 등에 따라 자동 확장/축소 |
| HTTP 기반 라우팅 | Envoy를 이용한 외부 트래픽 라우팅 지원 |
| Dapr 통합 | 상태 관리, 서비스 호출, pub/sub 등 기능 제공 |

### Azure CLI로 배포

```bash
az containerapp env create \
  --name my-environment \
  --resource-group my-resource-group \
  --location koreacentral

az containerapp create \
  --name my-api \
  --resource-group my-resource-group \
  --environment my-environment \
  --image mcr.microsoft.com/azuredocs/containerapps-helloworld:latest \
  --target-port 80 \
  --ingress 'external' \
  --cpu 0.5 --memory 1.0Gi
```

### YAML로 배포 정의

```yaml
apiVersion: '2022-03-01'
kind: ContainerApp
location: koreacentral
name: my-api
properties:
  environmentId: /subscriptions/<sub-id>/resourceGroups/my-resource-group/providers/Microsoft.App/managedEnvironments/my-environment
  configuration:
    ingress:
      external: true
      targetPort: 80
  template:
    containers:
      - image: mcr.microsoft.com/azuredocs/containerapps-helloworld:latest
        name: my-api
        resources:
          cpu: 0.5
          memory: 1.0Gi
```

### Dapr 연동

```bash
az containerapp create \
  --name my-dapr-app \
  --resource-group my-resource-group \
  --environment my-environment \
  --image <your-image> \
  --target-port 80 \
  --ingress external \
  --enable-dapr \
  --dapr-app-id my-dapr-app \
  --dapr-app-port 80
```

---

## ✅ 5. Azure Container Registry (ACR)

ACR은 Microsoft Azure에서 제공하는 Docker 컨테이너 이미지 저장소입니다.

### 주요 기능
- **프라이빗 레지스트리**: 조직 전용 컨테이너 이미지 저장소
- **Azure AD 통합**: 인증 및 권한 관리
- **Webhooks**: CI/CD 파이프라인과 통합 가능
- **Geo-replication**: 전 세계 여러 지역에서 동기화 가능
- **Content Trust**: 이미지 서명 및 보안 강화
- **헬름 차트 저장소**: Kubernetes 헬름 배포를 위한 차트 저장소

### SKU 종류
- **Basic**: 소규모 개발팀에 적합
- **Standard**: 대부분의 일반적인 워크로드에 적합
- **Premium**: 고급 보안, 가용성 및 성능 (zone redundancy, geo-replication 등)

---

## ✅ 6. Azure Virtual Machines (VM)

Azure Virtual Machines는 Azure에서 제공하는 IaaS 서비스로, 클라우드 기반 가상 서버를 제공합니다.

### 주요 기능
- **다양한 OS 지원**: Windows, Linux 등
- **유연한 크기 조정**: B, D, E, F, H, N 등 다양한 VM 시리즈 제공
- **확장성**: 가상 머신 스케일셋(VMSS)을 통해 수평 확장 가능
- **고가용성**: 가용성 집합(Availability Set), 가용성 영역(Availability Zone) 지원

### 비용 관리 및 최적화
- **예약 인스턴스**: 1년 또는 3년 단위 예약으로 최대 72% 절감
- **스팟 인스턴스**: 남는 용량을 저렴하게 활용
- **자동 정지/시작 스크립트**: 비업무 시간 동안 VM 비용 절감

---

## 📚 참고 자료

- [Azure App Service Docs](https://learn.microsoft.com/en-us/azure/app-service/)
- [Azure Front Door Docs](https://learn.microsoft.com/en-us/azure/frontdoor/)
- [Azure Kubernetes Service](https://learn.microsoft.com/en-us/azure/aks/)
- [Azure Container Apps](https://learn.microsoft.com/en-us/azure/container-apps/)
- [Azure Container Registry](https://learn.microsoft.com/en-us/azure/container-registry/)
- [Azure Virtual Machines](https://learn.microsoft.com/en-us/azure/virtual-machines/)
