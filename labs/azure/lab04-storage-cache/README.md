# Lab 04: 스토리지/캐시 — Azure Storage, Redis Cache, Flask+Redis 실습

[← Azure Labs 목록으로](../README.md)

---

## 📌 개요

Azure Storage의 다양한 서비스(Blob, File, Table, Queue)와 Azure Cache for Redis를 활용한 캐싱, 세션 관리, 그리고 Flask 웹 애플리케이션과의 통합을 학습합니다.

---

## 🎯 학습 목표

- Azure Storage 계정 생성 및 Blob/File 관리
- SAS 토큰을 활용한 보안 접근
- Azure Redis 기본 설정 및 VNet/Private Endpoint 구성
- Flask 애플리케이션에서 Redis 활용

---

## ✅ 1. Azure Storage

Azure Storage는 다양한 유형의 데이터를 저장할 수 있는 범용 클라우드 스토리지 솔루션입니다.

### 🔹 스토리지 계정 종류

| 유형 | 용도 |
|------|------|
| **General-purpose v2** | Blob, File, Queue, Table 모든 기능 지원 (가장 일반적) |
| **Blob Storage** | Blob 데이터 저장에 특화 |
| **FileStorage** | 고성능 Azure Files 제공 |
| **BlockBlobStorage** | Premium 성능이 필요한 Blob 저장용 |

### 🔹 스토리지 계정 생성 (Azure CLI)

```bash
az storage account create \
  --name mystorageaccount \
  --resource-group myresourcegroup \
  --location koreacentral \
  --sku Standard_LRS \
  --kind StorageV2 \
  --enable-hierarchical-namespace true
```

### 🔹 Blob 컨테이너 생성 및 업로드

```bash
az storage container create \
  --name mycontainer \
  --account-name mystorageaccount \
  --public-access off

az storage blob upload \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myfile.txt \
  --file ./myfile.txt
```

### 🔹 SAS 토큰 생성

```bash
az storage container generate-sas \
  --account-name mystorageaccount \
  --name mycontainer \
  --permissions r \
  --expiry 2025-12-31T23:59:00Z \
  --output tsv
```

### 🔹 Static 웹 사이트 호스팅 설정

```bash
az storage blob service-properties update \
  --account-name mystorageaccount \
  --static-website \
  --index-document index.html \
  --error-document error.html

az storage blob upload-batch \
  --account-name mystorageaccount \
  --source ./site \
  --destination \$web
```

### 🔹 파일 공유 (Azure Files)

```bash
az storage share create \
  --name myshare \
  --account-name mystorageaccount

az storage file upload \
  --share-name myshare \
  --source ./localfile.txt \
  --path remotefile.txt \
  --account-name mystorageaccount
```

### 🔹 네트워크 제한 설정

```bash
az storage account update \
  --name mystorageaccount \
  --default-action Deny

az storage account network-rule add \
  --resource-group myresourcegroup \
  --account-name mystorageaccount \
  --vnet-name myvnet \
  --subnet mysubnet
```

### 🔹 Bicep으로 스토리지 계정 정의

```bicep
resource storage 'Microsoft.Storage/storageAccounts@2022-09-01' = {
  name: 'mystorageacct'
  location: resourceGroup().location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
  properties: {
    accessTier: 'Hot'
  }
}
```

---

## ✅ 2. Redis 사용 목적

**Redis**(REmote DIctionary Server)는 인메모리 기반의 NoSQL 키-값 저장소로서, 클라우드 환경에서 다음과 같은 목적으로 활용됩니다:

### 📌 주요 목적
- **캐싱**: 자주 조회되는 데이터를 메모리에 저장하여 DB 부하 감소 및 응답 속도 향상
- **세션 저장소**: 사용자 로그인 세션을 서버 간 공유할 수 있도록 중앙 저장
- **메시지 브로커**: Pub/Sub 구조로 마이크로서비스 간 통신 가능
- **Rate Limiting**: API 요청 횟수를 Redis 키로 제어하여 과도한 요청 방지
- **Queue 시스템**: 작업을 순차 처리하는 큐 구현 가능

### Redis 활용 사례

| 사용처 | 설명 |
|--------|------|
| 웹 애플리케이션 캐시 | 로그인 정보, 상품 목록, 인기 콘텐츠 등 캐싱 |
| 마이크로서비스 통신 | Redis Pub/Sub 기능을 활용한 이벤트 브로커 |
| 게임 서버 | 사용자 점수, 상태 등을 빠르게 저장 및 조회 |
| 실시간 분석 | 실시간 클릭 수, 방문 수 등을 저장하여 통계 처리 |
| 분산 세션 저장소 | 사용자 인증 상태를 여러 서버에서 공유 |

---

## ✅ 3. Azure Redis 설정 (CLI)

```bash
# 변수 설정
RESOURCE_GROUP=myResourceGroup
REDIS_NAME=myRedisCache
LOCATION=koreacentral
SKU=Standard
VM_SIZE=C1

# 리소스 그룹 생성
az group create --name $RESOURCE_GROUP --location $LOCATION

# Redis 인스턴스 생성
az redis create \
  --name $REDIS_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku $SKU \
  --vm-size $VM_SIZE \
  --enable-non-ssl-port false
```

### ARM 템플릿 예시

```json
{
  "type": "Microsoft.Cache/Redis",
  "apiVersion": "2023-04-01",
  "name": "[parameters('redisCacheName')]",
  "location": "[parameters('location')]",
  "properties": {
    "enableNonSslPort": false
  },
  "sku": {
    "name": "[parameters('skuName')]",
    "family": "C",
    "capacity": "[parameters('skuCapacity')]"
  }
}
```

---

## ✅ 4. Redis + VNet + Private Endpoint (Azure CLI)

```bash
# 변수 설정
RESOURCE_GROUP=myRedisRG
LOCATION=koreacentral
REDIS_NAME=myPrivateRedis
VNET_NAME=myRedisVNet
SUBNET_NAME=myRedisSubnet
PE_NAME=myRedisPrivateEndpoint

# VNet/Subnet 생성
az network vnet create \
  --resource-group $RESOURCE_GROUP \
  --name $VNET_NAME \
  --subnet-name $SUBNET_NAME \
  --location $LOCATION \
  --address-prefix 10.0.0.0/16 \
  --subnet-prefix 10.0.1.0/24

# Subnet 수정
az network vnet subnet update \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name $SUBNET_NAME \
  --disable-private-endpoint-network-policies true

# Redis 생성
az redis create \
  --name $REDIS_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Premium \
  --vm-size P1 \
  --subnet-id <subnet-id> \
  --enable-non-ssl-port false

# Private Endpoint 생성
az network private-endpoint create \
  --name $PE_NAME \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --subnet $SUBNET_NAME \
  --private-connection-resource-id $(az redis show --name $REDIS_NAME --resource-group $RESOURCE_GROUP --query id -o tsv) \
  --group-id redisCache \
  --connection-name "${REDIS_NAME}-pe"
```

---

## ✅ 5. Redis Bicep 템플릿 (VNet + Private Endpoint + DNS)

```bicep
@description('Azure Region')
param location string = 'koreacentral'

@description('Redis Cache 이름')
param redisName string = 'myPrivateRedis'

@description('VNet 이름')
param vnetName string = 'myRedisVNet'

@description('Subnet 이름')
param subnetName string = 'myRedisSubnet'

@description('Private Endpoint 이름')
param peName string = 'myRedisPrivateEndpoint'

@description('Private DNS Zone 이름')
param privateDnsZoneName string = 'privatelink.redis.cache.windows.net'

var vnetAddressPrefix = '10.0.0.0/16'
var subnetPrefix = '10.0.1.0/24'

resource vnet 'Microsoft.Network/virtualNetworks@2023-04-01' = {
  name: vnetName
  location: location
  properties: {
    addressSpace: {
      addressPrefixes: [vnetAddressPrefix]
    }
    subnets: [
      {
        name: subnetName
        properties: {
          addressPrefix: subnetPrefix
          privateEndpointNetworkPolicies: 'Disabled'
        }
      }
    ]
  }
}

resource redis 'Microsoft.Cache/Redis@2023-04-01' = {
  name: redisName
  location: location
  sku: {
    name: 'Premium'
    family: 'P'
    capacity: 1
  }
  properties: {
    subnetId: vnet::subnets[0].id
    enableNonSslPort: false
    minimumTlsVersion: '1.2'
  }
}

resource privateDnsZone 'Microsoft.Network/privateDnsZones@2023-04-01' = {
  name: privateDnsZoneName
  location: 'global'
  properties: {}
}

resource vnetLink 'Microsoft.Network/privateDnsZones/virtualNetworkLinks@2023-04-01' = {
  name: 'vnetLink'
  parent: privateDnsZone
  location: 'global'
  properties: {
    virtualNetwork: {
      id: vnet.id
    }
    registrationEnabled: false
  }
}

resource privateEndpoint 'Microsoft.Network/privateEndpoints@2023-04-01' = {
  name: peName
  location: location
  properties: {
    subnet: {
      id: vnet::subnets[0].id
    }
    privateLinkServiceConnections: [
      {
        name: 'redisPrivateConnection'
        properties: {
          privateLinkServiceId: redis.id
          groupIds: [ 'redisCache' ]
        }
      }
    ]
  }
}

resource dnsZoneGroup 'Microsoft.Network/privateEndpoints/dnsZoneGroups@2023-04-01' = {
  name: 'default'
  parent: privateEndpoint
  properties: {
    privateDnsZoneConfigs: [
      {
        name: 'dnsconfig'
        properties: {
          privateDnsZoneId: privateDnsZone.id
        }
      }
    ]
  }
}
```

---

## ✅ 6. Flask + Azure Redis 실습 예제

### 🔹 목적
Flask API를 통해 Azure Redis에 데이터를 저장하고 조회합니다.

### 🔹 사전 준비
- Azure Redis 인스턴스 생성 (Premium/Standard)
- Python 환경: `pip install flask redis`

### 🔹 예제 코드 (`app.py`)

```python
from flask import Flask, request, jsonify
import redis
import os

app = Flask(__name__)

# Redis 연결 정보
redis_host = os.getenv('REDIS_HOST', 'your-redis-host.redis.cache.windows.net')
redis_port = 6380
redis_password = os.getenv('REDIS_PASSWORD', 'your-redis-access-key')

r = redis.StrictRedis(
    host=redis_host,
    port=redis_port,
    password=redis_password,
    ssl=True,
    decode_responses=True
)

@app.route('/set_gender', methods=['POST'])
def set_gender():
    data = request.json
    user_id = data.get('user_id')
    gender = data.get('gender')

    if not user_id or not gender:
        return jsonify({'error': 'Missing user_id or gender'}), 400

    r.set(f"user:{user_id}:gender", gender)
    return jsonify({'message': 'Gender saved'})

@app.route('/get_gender/<user_id>', methods=['GET'])
def get_gender(user_id):
    gender = r.get(f"user:{user_id}:gender")
    if gender:
        return jsonify({'user_id': user_id, 'gender': gender})
    else:
        return jsonify({'error': 'User not found'}), 404

if __name__ == '__main__':
    app.run(debug=True)
```

### Azure Redis 설정 (Python)

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

---

## 📚 참고 자료

- [Azure Storage 소개](https://learn.microsoft.com/ko-kr/azure/storage/common/storage-introduction)
- [Azure CLI Storage 명령어](https://learn.microsoft.com/ko-kr/cli/azure/storage)
- [Azure Cache for Redis](https://learn.microsoft.com/en-us/azure/azure-cache-for-redis/)

---

## ☁️ AWS 대응 서비스 비교

> 이 Lab의 Azure 스토리지/캐시 개념을 AWS에서 구현할 때 사용하는 서비스와 비교합니다.  
> AWS Lab 참조: [AWS Lab 07 — 아키텍처/DevOps](../../aws/lab07-architecture-devops/README.md)

### 서비스 대응 매핑

| Azure 서비스 | AWS 대응 서비스 | 차이점 |
|------------|---------------|--------|
| Azure Blob Storage | Amazon S3 | 가장 직접적 대응. S3는 더 많은 스토리지 클래스 |
| Azure Files | Amazon EFS (NFS) / FSx (SMB/NFS) | Files는 SMB/NFS 동시 지원 |
| Azure Table Storage | Amazon DynamoDB (단순 KV) | DynamoDB가 기능/성능 더 풍부 |
| Azure Queue Storage | Amazon SQS | 메시지 큐 동일 목적 |
| Azure Data Lake Storage Gen2 | Amazon S3 + AWS Lake Formation | ADLS는 Blob 위에 계층 파일시스템 제공 |
| Azure Cache for Redis | Amazon ElastiCache for Redis | 기능 동일, 설정 방식 차이 |
| Azure CDN | Amazon CloudFront | 글로벌 엣지 캐시 개념 동일 |
| Storage Account (계층) | S3 스토리지 클래스 | Hot→Warm→Cool→Archive ↔ Standard→IA→Glacier |

### 스토리지 계층(Tier) 비교

| 용도 | Azure Blob | Amazon S3 |
|------|-----------|-----------|
| 자주 접근하는 데이터 | Hot 티어 | S3 Standard |
| 가끔 접근 (30일 이상) | Cool 티어 | S3 Standard-IA |
| 드물게 접근 (90일 이상) | Cold 티어 | S3 Glacier Instant Retrieval |
| 장기 보관 (180일 이상) | Archive 티어 | S3 Glacier Flexible / Deep Archive |

### Redis 설정 비교

**Azure Cache for Redis (Python)**
```python
import redis

r = redis.StrictRedis(
    host='myredis.redis.cache.windows.net',
    port=6380,
    password='<your-access-key>',
    ssl=True
)
r.set('session:123', 'active', ex=3600)
print(r.get('session:123'))
```

**AWS ElastiCache for Redis (Python)**
```python
import redis

r = redis.StrictRedis(
    host='my-cluster.xxxxxx.ng.0001.apn2.cache.amazonaws.com',
    port=6379,
    ssl=True,
    ssl_cert_reqs=None
)
r.set('session:123', 'active', ex=3600)
print(r.get('session:123'))
```

### S3 vs Blob Storage 주요 차이

| 항목 | Amazon S3 | Azure Blob Storage |
|------|----------|-------------------|
| 버킷/컨테이너 네임스페이스 | 글로벌 고유 | 스토리지 계정 내 고유 |
| 정적 웹 호스팅 | S3 Static Website | Static Website (Blob) |
| 이벤트 트리거 | S3 Event Notifications | Event Grid |
| 오브젝트 잠금 | S3 Object Lock (WORM) | Blob Immutable Storage |
| 복제 | S3 Cross-Region Replication | Blob Geo-Redundancy (GRS/GZRS) |
| 보안 URL | Pre-signed URL | SAS Token |
| 수명 주기 | S3 Lifecycle Rules | Blob Lifecycle Management |

### AWS CLI 예시 (S3 기본 작업)

```bash
# 버킷 생성
aws s3api create-bucket \
  --bucket my-data-bucket \
  --region ap-northeast-2 \
  --create-bucket-configuration LocationConstraint=ap-northeast-2

# 파일 업로드
aws s3 cp local-file.pdf s3://my-data-bucket/documents/

# Pre-signed URL 생성 (1시간 유효)
aws s3 presign s3://my-data-bucket/documents/local-file.pdf \
  --expires-in 3600

# Lifecycle 정책 설정 (30일 후 IA, 90일 후 Glacier)
aws s3api put-bucket-lifecycle-configuration \
  --bucket my-data-bucket \
  --lifecycle-configuration file://lifecycle.json
```

### 병렬 학습 포인트

1. **오브젝트 스토리지**: Blob Storage ↔ S3 — 업로드/다운로드/SAS URL/Pre-signed URL 패턴 비교
2. **스토리지 계층**: Hot/Cool/Archive ↔ Standard/IA/Glacier — 비용 최적화 수명 주기 설계
3. **인메모리 캐시**: Azure Redis ↔ ElastiCache Redis — 세션 캐싱, pub/sub, 클러스터 모드 비교
4. **이벤트 트리거**: Event Grid ↔ S3 Event Notification — 파일 업로드 시 자동 처리 파이프라인
5. **파일 공유**: Azure Files ↔ Amazon EFS/FSx — NFS/SMB 기반 공유 스토리지 마운트 비교
