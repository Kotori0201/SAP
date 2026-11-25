# ⭐ Tổng kết · Compute & Load Balancing

[⬅ Về mục lục section](README.md)

## 1. Bảng tổng hợp kiến thức

| Dịch vụ | Là gì (1 dòng) | Con số phải nhớ | Chọn khi |
|---|---|---|---|
| **EC2 placement – Cluster** | Instance sát nhau, 1 AZ | Độ trễ thấp nhất | HPC, MPI |
| **EC2 placement – Spread** | Mỗi instance phần cứng riêng | **7 instance/AZ** | Vài instance quan trọng |
| **EC2 placement – Partition** | Nhóm rack tách biệt | **7 partition/AZ** | Cassandra, Kafka, HDFS |
| **Spot** | Dung lượng dư, rẻ | Giảm **90%**, báo trước **2 phút** | Batch, stateless |
| **Dedicated Host** | Nguyên server vật lý | – | License theo core/socket |
| **EFA** | Card mạng HPC bypass OS | Linux, MPI | Tightly coupled HPC |
| **ASG** | Tự scale EC2 | Target tracking / step / scheduled / predictive | Web tier |
| **ECS / Fargate** | Container AWS / serverless | awsvpc = ENI/task | Container |
| **EKS** | Kubernetes managed | 1 cluster/region | Đã dùng K8s |
| **Lambda** | Serverless function | **15 phút** · **10 GB** RAM · **1.000** concurrency · **6 MB** sync | Event-driven ngắn |
| **ALB** | LB Layer 7 | Path/host/header routing, Lambda target | HTTP, microservices |
| **NLB** | LB Layer 4 | **IP tĩnh/AZ, EIP**, triệu rps | TCP/UDP, whitelist IP |
| **GWLB** | LB Layer 3 cho appliance | **GENEVE 6081** | Firewall/IDS bên thứ 3 |
| **API Gateway** | API managed serverless | **29s** timeout · **10 MB** payload · **10.000 rps** · cache **300s** | REST/WebSocket API |
| **AppSync** | GraphQL managed | Real-time, offline | GraphQL |
| **Route 53** | DNS + health check | SLA 100% · ~15 health checker · multi-value **8 record** | DNS, failover |
| **Global Accelerator** | 2 Anycast IP tĩnh | Failover **< 1 phút** | TCP/UDP toàn cầu, IP cố định |
| **Outposts / Local Zones / Wavelength** | AWS tại DC khách / thành phố / nhà mạng 5G | – | Hybrid / độ trễ thấp |

## 2. So sánh quan trọng

### Placement Groups

| Cluster | Spread | Partition |
|---|---|---|
| Hiệu năng | An toàn | Cân bằng |
| 1 AZ | Nhiều AZ, 7/AZ | Nhiều AZ, 7 partition/AZ |
| Rack hỏng → tất cả hỏng | Mỗi instance cô lập | Mỗi partition cô lập |

### ALB vs NLB vs GWLB vs CLB

| | ALB | NLB | GWLB | CLB |
|---|---|---|---|---|
| Layer | 7 | 4 | 3 | 4 & 7 |
| Giao thức | HTTP/S, WebSocket, gRPC | TCP, UDP, TLS | IP (GENEVE) | HTTP/S, TCP |
| IP tĩnh | ❌ | ✅ (EIP) | – | ❌ |
| Target Lambda | ✅ | ❌ | ❌ | ❌ |
| Cross-zone mặc định | Bật, free | Tắt, tính phí | Tắt, tính phí | Tắt, free |
| SNI | ✅ | ✅ | – | ❌ |

### ECS vs EKS vs Fargate vs Lambda

| | ECS (EC2) | EKS | Fargate | Lambda |
|---|---|---|---|---|
| Quản lý server | Có | Có (trừ Fargate/Auto Mode) | Không | Không |
| Thời gian chạy | Không giới hạn | Không giới hạn | Không giới hạn | 15 phút |
| Phù hợp | Container AWS-native | Kubernetes | Container serverless | Event-driven ngắn |

### Lambda sync vs async

| Sync | Async |
|---|---|
| API Gateway, ALB, SDK | S3, SNS, EventBridge |
| Client retry | Lambda retry 3 lần → DLQ/Destination |

### Route 53 routing policy

| Policy | Một câu |
|---|---|
| Simple | Một đích |
| Weighted | Chia % |
| Latency | Nhanh nhất |
| Failover | Chính – phụ |
| Geolocation | Theo **vị trí user** |
| Geoproximity | Theo **khoảng cách + bias** |
| Multi-value | ≤ 8 IP khỏe |
| IP-based | Theo CIDR client |

### CNAME vs Alias

| CNAME | Alias |
|---|---|
| Không dùng cho root | Dùng cho root |
| Có phí | Miễn phí |
| Bất kỳ hostname | Tài nguyên AWS (không phải EC2 DNS) |

### Global Accelerator vs CloudFront

| Global Accelerator | CloudFront |
|---|---|
| TCP/UDP, không cache | HTTP, cache |
| 2 IP Anycast tĩnh | Domain CloudFront |
| Game, IoT, VoIP, failover nhanh | Website, media, API |

### API Gateway endpoint types

| Edge-optimized | Regional | Private |
|---|---|---|
| Client toàn cầu (qua CloudFront) | Client cùng region | Chỉ từ VPC (interface endpoint) |

### Outposts vs Local Zones vs Wavelength

| Outposts | Local Zones | Wavelength |
|---|---|---|
| DC của bạn | Thành phố lớn | Nhà mạng 5G |
| Bạn lo an ninh vật lý | AWS lo | AWS/nhà mạng lo |

## 3. Tip nhớ nhanh

- 🧠 **"Cluster = Chen chúc (nhanh), Spread = Sợ chết (an toàn, 7/AZ), Partition = Phân khu (Cassandra/Kafka)"**.
- 🧠 **"Spot = 90% rẻ, 2 phút báo"**; chiến lược tốt nhất **priceCapacityOptimized**.
- 🧠 **"RAM không có sẵn → CloudWatch Agent"**.
- 🧠 **"Lambda 15 – 10 – 1000 – 6"**: 15 phút, 10 GB, 1.000 concurrent, 6 MB sync.
- 🧠 **"Lambda trong VPC muốn ra Internet = NAT"** (public subnet cũng không có Internet).
- 🧠 **"API Gateway 29 – 10 – 10k"**: 29 giây, 10 MB, 10.000 rps.
- 🧠 **"Lỗi 429 = Quá nhiều, 504 = Quá lâu, 502 = Trả sai"**.
- 🧠 **"ALB = Application (7), NLB = Network (4), GWLB = Gateway (3)"** — số layer giảm dần.
- 🧠 **"NLB = IP tĩnh"**; **"GWLB = 6081"**.
- 🧠 **"Cross-zone: ALB free, NLB/GWLB tính tiền"**.
- 🧠 **"Root domain → Alias"**; **"Alias không trỏ EC2 DNS"**.
- 🧠 **"Geolocation = Ở đâu, Geoproximity = Bao xa (bias)"**.
- 🧠 **"Private health check → CloudWatch Alarm"**.
- 🧠 **"Inbound = vào AWS, Outbound = ra on-prem"** (Resolver endpoint).
- 🧠 **"Global Accelerator = 2 IP + UDP + < 1 phút"**.
- 🧠 **"ECS Anywhere cần mạng, EKS Anywhere không cần"**.
- 🧠 **"Outposts = Ở nhà mình, Wavelength = Sóng 5G, Local Zones = Lân cận thành phố"**.
