# Xử lý tải cực lớn (Handling Extreme Rates)

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **RPS (Requests Per Second)** | Số request mỗi giây mà một thành phần xử lý được. |
| **Bottleneck** | Thành phần có giới hạn thấp nhất, quyết định năng lực của toàn hệ thống. |
| **Caching layer** | Lớp cache đặt càng gần user càng tốt để giảm số request tới các lớp phía sau. |
| **Soft limit** | Giới hạn mặc định có thể yêu cầu AWS tăng. |

## Nội dung chính

### Giới hạn tham khảo theo lớp

> 📖 **Định nghĩa:** Xem *Bottleneck* ở bảng trên. Các con số là **mặc định/tham khảo**, nhiều cái tăng được.

| Lớp | Dịch vụ | Giới hạn / đặc điểm |
|---|---|---|
| **DNS** | Route 53 | Toàn cầu |
| **Edge cache** | CloudFront | ~**100.000 RPS** (mỗi distribution, tăng được) |
| **Web** | ALB | Scale dần; đỉnh đột ngột cần pre-warm |
| | API Gateway | **10.000 RPS** (account/region) |
| **Compute** | ASG, ECS (EC2) | **Chậm** – phải bootstrap |
| | Fargate | **Nhanh hơn** |
| | Lambda | **1.000 concurrent** (soft) |
| **Cache** | Redis | < **200 node** (replica + sharding) |
| | Memcached | ~**20 node** (sharding) |
| | DAX | ~**10 node** (primary + replica) |
| **Database** | RDS, Aurora, OpenSearch | **Provisioned** (phải chọn cấu hình) |
| | DynamoDB | **Auto scaling, on-demand** |
| **Messaging** | SQS, SNS | **Gần như không giới hạn** |
| | SQS FIFO | **3.000 RPS** (với batching; high-throughput mode cao hơn nhiều) |
| | Kinesis | **1 MB/s vào, 2 MB/s ra mỗi shard** |
| **Storage** | EBS | gp2 16k IOPS, io1 64k IOPS (io2 BE 256k) |
| | Instance Store | ~**hàng triệu IOPS** |
| | EFS | General Purpose / Max I/O |
| | S3 | **3.500 PUT, 5.500 GET / prefix / giây** |
| **Mã hóa** | KMS | **Quota KMS** nếu mã hóa (dễ bị throttle) |

### Nguyên tắc

> 📖 **Định nghĩa:** **Trade-off** là sự đánh đổi giữa các yếu tố (cache, TTL, mạng, tính toán, chi phí, độ trễ).

- **Cache càng gần user càng tốt** (CloudFront → ElastiCache/DAX → DB)
- Mỗi lớp cache giảm tải cho lớp sau; cân nhắc **TTL, mạng, tính toán, chi phí, độ trễ**
- **Decouple bằng queue** (SQS) để hấp thụ đỉnh
- Kiểm tra **mọi soft limit** trên đường đi (API Gateway, Lambda, KMS…)

## 🎯 Mẹo thi

- "Flash sale 100.000 RPS" → **CloudFront cache** + **SQS** hấp thụ ghi + DynamoDB on-demand
- "Upload S3 mã hóa SSE-KMS bị throttle" → **giới hạn KMS** → S3 Bucket Key / tăng quota
- "Ghi đơn hàng đột biến vào RDS" → **SQS** đứng trước để buffer
- "API Gateway trả 429 khi tải cao" → tăng quota / cache / usage plan
