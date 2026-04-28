# Lab 01: 네트워킹 — VNet, Subnet, Private Endpoint, 게이트웨이, ExpressRoute

[← Azure Labs 목록으로](../README.md)

---

## 📌 개요

Azure 가상 네트워크(VNet) 구성, 서브넷 분리, Private Endpoint를 통한 보안 PaaS 접근, 그리고 다양한 게이트웨이/ExpressRoute를 활용한 하이브리드 연결을 학습합니다.

---

## 🎯 학습 목표

- VNet/Subnet 설계 및 IP 주소 계획
- Private Endpoint를 통한 보안 PaaS 접근
- 네트워크 게이트웨이(인터넷, VPN, NAT) 이해
- VNet Peering 및 ExpressRoute 구성

---

## ✅ 1. VNet (Virtual Network)

### 📌 정의
Azure 가상 네트워크(VNet)는 **Azure 내에서 리소스들이 안전하게 서로 통신할 수 있도록 하는 가상의 네트워크**입니다.

### 🔍 특징
- IP 주소 범위 지정 (CIDR 형식, 예: `10.0.0.0/16`)
- VM, App Service, Redis, Database 등 다양한 리소스를 이 네트워크에 배치 가능
- 외부와는 인터넷 게이트웨이, VPN Gateway, NAT Gateway, Peering 등으로 연결 가능

---

## ✅ 2. Subnet

### 📌 정의
Subnet(서브넷)은 VNet을 세부적으로 나누는 단위로, **리소스를 논리적으로 그룹화하거나 액세스를 분리**할 수 있습니다.

### 🔍 특징
- VNet의 주소 공간 안에서 하위 주소 범위 지정 (예: `10.0.1.0/24`)
- 리소스 간 통신 정책, NSG(Network Security Group) 할당 가능
- 일부 Azure 리소스(예: Redis Premium, Private Endpoint)는 서브넷에 직접 배치되어야 함

---

## ✅ 3. Private Endpoint

### 📌 정의
Private Endpoint는 Azure PaaS 서비스(예: Redis, SQL Database, Storage 등)에 **VNet 내부 IP로 안전하게 접근할 수 있게 해주는 전용 네트워크 인터페이스**입니다.

### 🔍 역할
- 공용 인터넷 없이 내부 네트워크에서 Azure 서비스 접근 가능
- DNS는 `*.privatelink.*` 도메인을 통해 자동 매핑
- Private IP 주소를 사용하므로 보안성이 높음

### 🧩 구성 예
- Azure Redis Premium 인스턴스를 Private Endpoint로 설정하면 외부에서는 접속 불가
- VM 또는 App Service가 같은 VNet에 속하거나 피어링되면 해당 Redis에 접속 가능

---

## 🔐 VNet/Subnet/Private Endpoint 관계도

```
Azure Virtual Network (10.0.0.0/16)
│
├── Subnet: appSubnet (10.0.1.0/24) ────── VM, App Service
│
└── Subnet: redisSubnet (10.0.2.0/24) ─── Redis Premium + Private Endpoint
                                          ↓
                                  Private IP: 10.0.2.5
                                          ↓
                             DNS: myredis.privatelink.redis.cache.windows.net
```

### 보안 및 운영상 이점

| 구성 요소 | 주요 이점 |
|-----------|-----------|
| **VNet** | 리소스를 가상 네트워크로 격리 |
| **Subnet** | 보안 그룹(NSG), 라우팅 적용 가능 |
| **Private Endpoint** | Azure 서비스와의 통신을 내부 IP로 제한, 데이터 노출 차단 |

---

## ✅ 4. Azure 네트워크 게이트웨이

### 🔷 인터넷 게이트웨이 (Internet Gateway)

- Azure에서는 **기본적으로 제공됨**
- 가상 네트워크에 있는 리소스가 **공용 인터넷과 통신**할 수 있게 함
- 퍼블릭 IP가 설정된 VM, Load Balancer 등이 사용

### 🔷 VPN 게이트웨이 (VPN Gateway)

- **온프레미스 네트워크 ↔ Azure 가상 네트워크**를 안전하게 연결
- **Site-to-Site, Point-to-Site, ExpressRoute** 등 다양한 연결 유형
- IPSec 터널을 통해 암호화된 트래픽 제공

### 🔷 NAT 게이트웨이 (NAT Gateway)

- **VNet 내부의 사설 IP 리소스들이 인터넷에 나갈 수 있도록 하는 구성**
- **들어오는 트래픽은 차단**, 나가는 트래픽만 허용
- 다수의 리소스에 대해 **고정된 공용 IP** 사용 가능
- 보안상 **인바운드 트래픽을 막고 아웃바운드만 허용**할 때 유용

### 🔷 VNet Peering

- 서로 다른 **VNet 간에 프라이빗 IP를 통한 통신**을 가능하게 함
- **다른 리전 간 Peering**도 가능 (Global VNet Peering)
- 동일 구독 또는 다른 구독 간에도 연결 가능
- 트래픽은 Azure 백본망을 통해 전달되므로 **빠르고 안전**

### 네트워크 게이트웨이 용도 요약

| 구성요소 | 주요 용도 |
|----------|-----------|
| 인터넷 게이트웨이 | Azure VM 등의 인터넷 접속 제공 |
| VPN 게이트웨이 | 온프레미스 ↔ Azure 간 보안 터널 |
| NAT 게이트웨이 | 사설 리소스의 아웃바운드 트래픽만 허용 |
| VNet Peering | VNet 간 내부 통신을 위한 연결 |

---

## ✅ 5. ExpressRoute

### 📌 정의
**ExpressRoute**는 Azure 데이터 센터와 **온프레미스 네트워크를 전용 회선을 통해 연결**하는 서비스입니다.  
일반 인터넷 대신 **사설 네트워크**로 연결되기 때문에 **보안성, 신뢰성, 속도**가 뛰어납니다.

### 🔍 주요 특징
- 인터넷을 통하지 않고 Azure와 통신 (예: VPN과 차이점)
- **높은 대역폭** 및 **낮은 지연 시간** 제공
- **BGP 기반 라우팅** 지원
- Microsoft 365, Dynamics 365도 ExpressRoute로 연결 가능 (옵션 설정 시)

### ExpressRoute 연결 유형

| 유형 | 설명 |
|------|------|
| **Private Peering** | 온프레미스 ↔ VNet 연결 (가상 머신, Redis 등) |
| **Microsoft Peering** | 온프레미스 ↔ Microsoft 공용 서비스 (Azure Storage, SQL, 365 등) |

### VPN Gateway vs ExpressRoute

| 항목 | VPN Gateway | ExpressRoute |
|------|-------------|--------------|
| 연결 경로 | 공용 인터넷 | 사설 전용망 |
| 보안성 | 암호화된 인터넷 경로 | 전용 회선 (더 안전) |
| 성능 | 중간 ~ 낮음 | 매우 높음 (10Gbps 이상 가능) |
| SLA | 낮음 | 높음 (엔터프라이즈 급) |
| 비용 | 상대적으로 저렴 | 비교적 고가 (전용 회선 요금 포함) |

### ExpressRoute를 사용하는 이유
- **보안 데이터 전송 필요** (예: 금융, 의료)
- **대량의 데이터 이동 필요** (백업, 분석 등)
- **하이브리드 클라우드** 환경 구축

---

## 📚 참고 자료

- [Azure Virtual Network](https://learn.microsoft.com/en-us/azure/virtual-network/)
- [Azure Private Endpoint](https://learn.microsoft.com/en-us/azure/private-link/private-endpoint-overview)
- [Azure ExpressRoute](https://learn.microsoft.com/en-us/azure/expressroute/)
- [Azure VPN Gateway](https://learn.microsoft.com/en-us/azure/vpn-gateway/)

---

## ☁️ AWS 대응 서비스 비교

> 이 Lab의 Azure 네트워킹 개념을 AWS에서 구현할 때 사용하는 서비스와 비교합니다.  
> AWS Lab 참조: [AWS Lab 07 — 아키텍처/DevOps](../../aws/lab07-architecture-devops/README.md)

### 서비스 대응 매핑

| Azure 개념/서비스 | AWS 대응 서비스 | 차이점 |
|-----------------|---------------|--------|
| VNet (Virtual Network) | VPC (Virtual Private Cloud) | 개념 동일, Azure VNet은 구독 스코프 |
| Subnet | Subnet | 동일. AWS는 AZ별로 서브넷 생성 필수 |
| NSG (Network Security Group) | Security Group (인스턴스 레벨) + NACL (서브넷 레벨) | AWS는 Stateful SG + Stateless NACL 이중 구조 |
| Private Endpoint | VPC Endpoint (Interface Endpoint) | 동일 개념, AWS PrivateLink 기반 |
| VPN Gateway | AWS VPN Gateway + Customer Gateway | 설정 방식 유사 |
| NAT Gateway | NAT Gateway | 동일 개념 |
| VNet Peering | VPC Peering | 동일 개념, AWS는 Transit Gateway로 허브-스포크 구성 |
| ExpressRoute | AWS Direct Connect | 전용 회선 기반 하이브리드 연결, 동일 목적 |
| Azure Front Door | AWS Global Accelerator + CloudFront | 글로벌 라우팅 + CDN 역할 |
| Azure Firewall | AWS Network Firewall | 중앙 집중형 네트워크 방화벽 |

### 네트워크 구성 비교

**Azure 네트워크 구성**
```
Azure Virtual Network (10.0.0.0/16)
├── Subnet: appSubnet (10.0.1.0/24) — NSG 적용
│     └── App Service, VM
└── Subnet: dataSubnet (10.0.2.0/24) — NSG 적용
      └── Azure SQL, Redis + Private Endpoint
```

**AWS 네트워크 구성**
```
VPC (10.0.0.0/16)
├── Public Subnet (10.0.1.0/24) [AZ-a] — Internet Gateway, NAT Gateway
│     └── ALB, Bastion Host
├── Private Subnet (10.0.2.0/24) [AZ-a] — Security Group
│     └── EC2, Lambda, EKS Node
└── Private Subnet (10.0.3.0/24) [AZ-b] — Security Group
      └── RDS, ElastiCache + VPC Endpoint
```

### 핵심 차이점 상세

| 항목 | Azure | AWS |
|------|-------|-----|
| 서브넷 AZ 분리 | VNet 내 자유롭게 배치 (AZ 무관) | 서브넷은 반드시 단일 AZ에 속함 |
| 보안 그룹 적용 단위 | NSG → Subnet 또는 NIC | Security Group → 인스턴스/ENI |
| 인터넷 게이트웨이 | 기본 제공 (별도 리소스 없음) | Internet Gateway를 VPC에 명시적 연결 필요 |
| 프라이빗 서비스 접근 | Private Endpoint (DNS 자동 매핑) | VPC Endpoint Interface (Route 53 Resolver) |
| 전용 회선 속도 | ExpressRoute: 최대 100Gbps | Direct Connect: 최대 100Gbps (동일) |
| VNet/VPC 간 연결 | VNet Peering / Virtual WAN | VPC Peering / Transit Gateway |

### AWS CLI 예시 (VPC + Subnet + Private 연결 구성)

```bash
# VPC 생성
aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications \
  'ResourceType=vpc,Tags=[{Key=Name,Value=my-vpc}]'

# 퍼블릭 서브넷 생성
aws ec2 create-subnet \
  --vpc-id vpc-xxxx \
  --cidr-block 10.0.1.0/24 \
  --availability-zone ap-northeast-2a

# 프라이빗 서브넷 생성
aws ec2 create-subnet \
  --vpc-id vpc-xxxx \
  --cidr-block 10.0.2.0/24 \
  --availability-zone ap-northeast-2a

# VPC Endpoint (S3 Interface Endpoint) 생성
aws ec2 create-vpc-endpoint \
  --vpc-id vpc-xxxx \
  --vpc-endpoint-type Interface \
  --service-name com.amazonaws.ap-northeast-2.s3 \
  --subnet-ids subnet-xxxx \
  --security-group-ids sg-xxxx
```

### 병렬 학습 포인트

1. **격리 단위**: VNet/Subnet ↔ VPC/Subnet — 리소스 네트워크 격리 설계 원칙 동일
2. **보안 정책**: NSG ↔ Security Group + NACL — Stateful vs Stateless 차이 이해
3. **프라이빗 접근**: Private Endpoint ↔ VPC Interface Endpoint — PaaS 서비스를 내부 IP로 접근
4. **하이브리드 연결**: ExpressRoute ↔ Direct Connect — 전용 회선 설계 시 고려 요소 동일
5. **멀티 VNet/VPC**: VNet Peering ↔ Transit Gateway — 대규모 네트워크 허브-스포크 구조 비교
