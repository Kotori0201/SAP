# 03 · Compute & Load Balancing

> Nguồn: Slide Stephane Maarek SAP v38 (trang 174–317) + section *Compute & Load Balancing* trên Udemy, có bổ sung cập nhật AWS (🆕). Ghi chú viết lại để ôn tập.

**Domain SAP-C03:** Cloud-native architecture design · Resilience · Cost-optimized architecture

## 📖 Định nghĩa tổng quan

- **Compute:** năng lực xử lý chạy ứng dụng — máy ảo (EC2), container (ECS/EKS/Fargate), serverless (Lambda).
- **Load balancing:** phân phối traffic tới nhiều backend, kiểm tra sức khỏe, ẩn backend sau một endpoint.
- **Scalability:** khả năng tăng/giảm năng lực theo tải (scale out/in, scale up/down).
- **Elasticity:** tự động scale theo nhu cầu thực tế để chỉ trả tiền cho phần dùng.
- **DNS routing:** dùng DNS (Route 53) để quyết định client đi tới endpoint nào.
- **Edge / hybrid:** đưa hạ tầng AWS ra gần người dùng hoặc vào data center khách hàng (Global Accelerator, Outposts, Local Zones, Wavelength).

## Mục lục

| # | Chủ đề | File |
|---|---|---|
| 1 | EC2: instance types, placement groups, purchasing, Graviton, recovery | [01-ec2.md](01-ec2.md) |
| 2 | High Performance Computing (EFA, FSx Lustre, Batch, ParallelCluster) | [02-hpc.md](02-hpc.md) |
| 3 | EC2 Auto Scaling | [03-auto-scaling.md](03-auto-scaling.md) |
| 4 | Spot Instances & Spot Fleet | [04-spot-instances-va-spot-fleet.md](04-spot-instances-va-spot-fleet.md) |
| 5 | ECS, Fargate, ECR (+ 🆕 ECS Service Connect) | [05-ecs-fargate-ecr.md](05-ecs-fargate-ecr.md) |
| 6 | EKS, ECS Anywhere, EKS Anywhere | [06-eks.md](06-eks.md) |
| 7 | AWS Lambda | [07-lambda.md](07-lambda.md) |
| 8 | Elastic Load Balancing (CLB, ALB, NLB, GWLB) | [08-elastic-load-balancing.md](08-elastic-load-balancing.md) |
| 9 | API Gateway | [09-api-gateway.md](09-api-gateway.md) |
| 10 | AppSync | [10-appsync.md](10-appsync.md) |
| 11 | Route 53 | [11-route-53.md](11-route-53.md) |
| 12 | Global Accelerator | [12-global-accelerator.md](12-global-accelerator.md) |
| 13 | So sánh kiến trúc Compute + LB | [13-so-sanh-kien-truc.md](13-so-sanh-kien-truc.md) |
| 14 | Outposts, Wavelength, Local Zones | [14-outposts-wavelength-local-zones.md](14-outposts-wavelength-local-zones.md) |
| ⭐ | **Tổng kết, so sánh & tip nhớ nhanh** | [TONG-KET.md](TONG-KET.md) |

## Cheat-sheet: Tình huống → Giải pháp

| Tình huống trong đề | Giải pháp |
|---|---|
| Độ trễ mạng giữa node thấp nhất (HPC, MPI) | **Cluster placement group + EFA** |
| Cassandra/Kafka/HDFS cô lập lỗi theo rack | **Partition placement group** |
| Vài instance quan trọng trên phần cứng khác nhau | **Spread placement group** (≤ 7/AZ) |
| License theo core/socket | **Dedicated Host** |
| Theo dõi RAM của EC2 | **CloudWatch Agent** |
| Giữ IP khi phần cứng lỗi | **EC2 instance recovery** (alarm `StatusCheckFailed_System`) |
| File system HPC cực nhanh, dữ liệu ở S3 | **FSx for Lustre** |
| Hàng nghìn batch job, ít quản lý | **AWS Batch** (+ Spot) |
| Giữ CPU ~50% | **Target tracking** |
| Tải tăng theo chu kỳ biết trước | **Scheduled / Predictive scaling** |
| Chạy script trước khi terminate instance | **Lifecycle hook** |
| Đổi AMI toàn ASG, giữ capacity tối thiểu | **Instance Refresh** |
| Spot ít bị thu hồi, vẫn rẻ | **`priceCapacityOptimized`** + đa dạng instance type |
| Container serverless | **Fargate** |
| Mỗi task quyền AWS riêng / SG riêng | **Task IAM Role** / **awsvpc** |
| Fargate rẻ hơn cho tải dao động | **FARGATE_SPOT** + base On-Demand |
| Image ở nhiều region/account | **ECR replication** |
| Quét lỗ hổng thư viện trong image | **ECR enhanced scanning (Inspector)** |
| Service ECS gọi nhau, cần retry/metric | **ECS Service Connect** 🆕 |
| Kết nối service đa VPC/account/loại compute | **VPC Lattice** 🆕 |
| Đang dùng Kubernetes | **EKS** |
| Pod cần IAM role riêng | **IRSA / EKS Pod Identity** |
| Container on-prem, control plane AWS | **ECS Anywhere** |
| Kubernetes on-prem có thể mất kết nối | **EKS Anywhere** |
| Tác vụ > 15 phút | **Không phải Lambda** → Fargate/Batch/Step Functions |
| 1 function chiếm hết concurrency | **Reserved concurrency** |
| Loại bỏ cold start | **Provisioned concurrency** / SnapStart |
| Lambda cần IP cố định | **Lambda trong VPC + NAT Gateway (EIP)** |
| Lambda trong VPC không ra Internet | Thiếu **NAT Gateway** |
| Deploy Lambda 10% trước | **CodeDeploy canary** + alias |
| Quá nhiều kết nối DB từ Lambda | **RDS Proxy** |
| Routing theo path/host | **ALB** |
| TCP/UDP, IP tĩnh/EIP | **NLB** |
| IP tĩnh + routing L7 | **NLB → ALB** hoặc **Global Accelerator → ALB** |
| Traffic qua firewall bên thứ 3 trong suốt | **GWLB** (GENEVE 6081) |
| Expose Lambda qua HTTP đơn giản | **ALB + Lambda** |
| Upload file lớn qua API | **Pre-signed URL S3** |
| Lỗi 504 sau 29s | Backend chậm → xử lý **async** |
| Bán API theo gói | **Usage plans + API keys** |
| Chat real-time, server đẩy tin | **API Gateway WebSocket** |
| API chỉ truy cập từ VPC | **Private API** + resource policy `aws:SourceVpce` |
| Ghi SQS/Kinesis qua REST không cần Lambda | **API Gateway AWS service integration** |
| GraphQL, nhiều nguồn dữ liệu, offline sync | **AppSync** |
| Root domain → ALB/CloudFront | **Route 53 Alias** |
| Chuyển % traffic sang bản mới | **Weighted** |
| Active-passive DR qua DNS | **Failover** + health check |
| Nội dung theo quốc gia | **Geolocation** (+ Default) |
| Dồn thêm traffic sang region | **Geoproximity bias** |
| Health check tài nguyên private | **CloudWatch Alarm-based health check** |
| On-prem phân giải private hosted zone | **Resolver Inbound endpoint** |
| VPC phân giải tên on-prem | **Outbound endpoint + forwarding rule** (share qua RAM) |
| Vài IP Anycast cố định cho app đa region | **Global Accelerator** |
| Game UDP / IoT MQTT / VoIP toàn cầu | **Global Accelerator** |
| Dữ liệu ở lại data center nhưng dùng AWS API | **Outposts** |
| Ứng dụng 5G độ trễ ms | **Wavelength** |
| Độ trễ thấp cho user ở một thành phố | **Local Zones** |

## Bẫy hay gặp trong đề

- ❌ "Spread placement group cho 20 instance trong 1 AZ" → tối đa **7/AZ**.
- ❌ "EC2 có sẵn metric RAM" → cần **CloudWatch Agent**.
- ❌ "Health check ASG gọi DB" → DB lỗi làm ASG thay toàn bộ instance.
- ❌ "Database trên Spot" → không.
- ❌ "ECS Service Auto Scaling scale luôn EC2" → cần **capacity provider/ASG** riêng.
- ❌ "EKS Anywhere bắt buộc kết nối AWS" → không (ECS Anywhere mới cần).
- ❌ "Lambda ở public subnet có Internet" → không có IP public, cần **NAT**.
- ❌ "Lambda chạy 30 phút" → tối đa **15 phút**.
- ❌ "API Gateway proxy upload 50 MB" → giới hạn **10 MB**.
- ❌ "Multi-value thay thế ELB" → không.
- ❌ "CNAME cho zone apex" → dùng **Alias**.
- ❌ "Alias tới DNS name của EC2" → không được.
- ❌ "Route 53 HTTP health check tới IP private" → dùng **CloudWatch Alarm**.
- ❌ "Cross-zone NLB miễn phí" → NLB/GWLB **tính phí** inter-AZ.
- ❌ "Global Accelerator cache nội dung" → **không cache**, đó là CloudFront.
- ❌ "AWS chịu trách nhiệm an ninh vật lý Outposts" → **khách hàng**.
