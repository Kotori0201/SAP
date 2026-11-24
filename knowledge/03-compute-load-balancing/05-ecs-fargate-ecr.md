# Containers: Docker, Amazon ECS, AWS Fargate, Amazon ECR

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Docker / Container** | Nền tảng đóng gói ứng dụng cùng toàn bộ phụ thuộc thành **container image**, chạy giống nhau ở mọi môi trường; nhẹ và khởi động nhanh hơn VM vì dùng chung kernel host. |
| **Amazon ECS** | *Elastic Container Service* – dịch vụ điều phối container **của riêng AWS**. |
| **AWS Fargate** | Nền tảng chạy container **serverless**: không quản lý EC2, chỉ khai báo CPU/RAM; dùng được với ECS và EKS. |
| **Amazon ECR** | *Elastic Container Registry* – kho lưu trữ container image (private/public) do AWS quản lý. |
| **Task definition** | Bản mô tả JSON cách chạy container: image, CPU, RAM, port, biến môi trường, IAM role, log… |
| **Task** | Một lần chạy cụ thể của task definition (1 hoặc nhiều container). |
| **Service** | Thành phần ECS duy trì **số lượng task mong muốn**, tích hợp load balancer, rolling update. |
| **Cluster** | Nhóm logic các tài nguyên (EC2 hoặc Fargate) chạy task. |
| **Capacity provider** | Cấu hình nguồn capacity cho cluster (ASG, FARGATE, FARGATE_SPOT) và cách phân bổ task giữa chúng. |

## Nội dung chính

### Quản lý container trên AWS

> 📖 **Định nghĩa:** **Container orchestration** là việc tự động triển khai, scale, kết nối và phục hồi container trên nhiều máy.

- **ECS** – nền tảng của AWS
- **EKS** – Kubernetes managed (mã nguồn mở)
- **Fargate** – serverless, chạy cho cả ECS và EKS

### Use case của ECS

> 📖 **Định nghĩa:** **Microservices** là kiến trúc chia ứng dụng thành nhiều service nhỏ, độc lập, giao tiếp qua API.

- **Microservices**: nhiều container trên một máy, service discovery, tích hợp ALB/NLB, auto scaling
- **Batch / scheduled task**: chạy task trên On-Demand/Reserved/Spot (lập lịch bằng EventBridge Scheduler)
- **Migrate lên cloud**: Docker hóa ứng dụng legacy → chạy trên ECS

### Khái niệm & IAM Role

> 📖 **Định nghĩa:** **Task IAM role** là role gắn cho từng task để container gọi AWS API; **Task execution role** là role để ECS agent kéo image từ ECR, đẩy log, đọc secret.

- **EC2 Instance Profile** (EC2 launch type): instance gọi ECS API, gửi log, kéo image
- **Task IAM Role**: **mỗi task một role riêng** → least privilege (vd service A đọc S3, service B ghi DynamoDB)
- **Task execution role**: pull ECR, CloudWatch Logs, lấy secret từ Secrets Manager/SSM

### Tích hợp ALB – Dynamic Port Mapping

> 📖 **Định nghĩa:** **Dynamic port mapping** là việc ECS gán **port ngẫu nhiên trên host** cho mỗi container; ALB tự tìm đúng port để chuyển traffic.

- Chạy **nhiều bản sao** cùng ứng dụng trên **cùng EC2**
- Lợi ích: tăng sức chịu lỗi, **tận dụng CPU/core**, **rolling upgrade** không downtime
- Security group của EC2 phải cho phép **mọi port** từ SG của ALB

### AWS Fargate

> 📖 **Định nghĩa:** Xem bảng trên.

- Không provision hạ tầng, **serverless**
- Tạo task definition → AWS chạy container theo CPU/RAM khai báo
- Scale = **tăng số task**

### Bảo mật & Networking

> 📖 **Định nghĩa:** **awsvpc network mode** là chế độ mạng cấp cho **mỗi task một ENI và private IP riêng**.

- Inject **secret & cấu hình** làm biến môi trường từ **SSM Parameter Store** và **Secrets Manager**
- Network mode:

| Mode | Ý nghĩa |
|---|---|
| `none` | Không có mạng |
| `bridge` | Mạng ảo của Docker |
| `host` | Dùng thẳng card mạng của host (bỏ qua Docker network) |
| **`awsvpc`** | Mỗi task **ENI + private IP riêng** → **Security Group cho từng task**, VPC Flow Logs; **mặc định và bắt buộc với Fargate** |

> ⚠️ `awsvpc` trên EC2 bị giới hạn số ENI/instance → bật **ENI trunking**.

### ECS Service Auto Scaling

> 📖 **Định nghĩa:** **Application Auto Scaling** là dịch vụ scale cho tài nguyên ngoài EC2 (ECS service, DynamoDB, Aurora replica, Lambda concurrency…).

- Tăng/giảm **desired number of tasks** qua **Application Auto Scaling**
- Metric CPU/RAM ở mức **ECS service** (và `ALBRequestCountPerTarget`)
- Target Tracking / Step Scaling / Scheduled Scaling
- ⚠️ **ECS Service Auto Scaling (task) ≠ EC2 Auto Scaling (instance)** → với EC2 launch type cần thêm **ECS Cluster Capacity Provider** (managed scaling cho ASG)
- Fargate scale đơn giản hơn nhiều (serverless)

### ECS với Spot

> 📖 **Định nghĩa:** **Draining** là trạng thái instance ngừng nhận task mới và các task hiện tại được dừng/chuyển đi.

- **EC2 launch type**: EC2 Spot trong ASG; instance chuyển **draining** khi bị thu hồi → tiết kiệm nhưng giảm độ tin cậy
- **Fargate**: đặt **số task tối thiểu On-Demand** (baseline) + thêm task **FARGATE_SPOT** để tiết kiệm (capacity provider strategy: `base` + `weight`)

### Amazon ECR

> 📖 **Định nghĩa:** **Lifecycle policy (ECR)** là quy tắc tự xóa image cũ theo số lượng hoặc tuổi.

- Repository **private** và **public** (ECR Public Gallery)
- Tích hợp ECS/EKS; truy cập kiểm soát bằng **IAM** (lỗi pull → kiểm tra policy)
- Hỗ trợ image scanning, versioning, tag (có **tag immutability**), lifecycle policy
- **Replication**: **cross-Region** và **cross-account** (vd CodeBuild push vào us-east-1 → tự replicate sang us-west-1, eu-west-1)
- 🆕 **Pull-through cache**: cache image từ registry ngoài (Docker Hub, Quay, GitHub, ECR Public…)

### ECR Image Scanning

> 📖 **Định nghĩa:** **Basic scanning** dùng CVE database phổ biến; **Enhanced scanning** dùng **Amazon Inspector** quét lỗ hổng **OS và thư viện ngôn ngữ lập trình**, liên tục.

| | Basic | Enhanced |
|---|---|---|
| Engine | CVE database (Clair trước đây, nay AWS native) | **Amazon Inspector** |
| Phạm vi | Package OS | OS + **programming language packages** |
| Kích hoạt | Manual hoặc **scan on push** | Scan on push + **continuous** |
| Kết quả | Console, **EventBridge** | Console, Inspector, **EventBridge** |

### 🆕 ECS Service Connect (chủ đề mới SAP-C03)

> 📖 **Định nghĩa:** **ECS Service Connect** là tính năng giúp các service ECS **tìm thấy và gọi nhau bằng tên ngắn** (service-to-service), ECS tự chèn **proxy sidecar** để xử lý retry, timeout, load balancing và metric.

- Dựa trên **AWS Cloud Map** namespace; không cần ALB nội bộ hay tự cài service mesh
- Cung cấp metric traffic (request, lỗi, độ trễ) trong ECS console/CloudWatch
- Hỗ trợ **TLS** giữa service (dùng AWS Private CA)
- So sánh nhanh:
  - **Service Discovery (Cloud Map DNS)**: chỉ phân giải DNS, không có retry/metric
  - **Service Connect**: discovery + proxy + metric, **trong phạm vi ECS**
  - **VPC Lattice**: kết nối service **đa dịch vụ (ECS, EKS, Lambda, EC2), đa VPC, đa account**, có auth policy (sẽ học ở phần Networking)
  - **App Mesh**: ⚠️ AWS đã thông báo **ngừng hỗ trợ** (2026) → chuyển sang Service Connect/VPC Lattice

## 🎯 Mẹo thi

- "Container serverless, không quản lý server" → **Fargate**
- "Mỗi container cần quyền AWS khác nhau" → **Task IAM Role**
- "Mỗi task cần Security Group riêng" → **awsvpc**
- "Nhiều container cùng app trên 1 EC2 sau ALB" → **dynamic port mapping**
- "Tiết kiệm chi phí Fargate cho phần tải dao động" → **FARGATE_SPOT** + base On-Demand
- "Image phải có ở nhiều region/account" → **ECR replication**
- "Quét lỗ hổng thư viện trong image liên tục" → **ECR enhanced scanning (Inspector)**
- "Các service ECS gọi nhau, cần retry & metric, ít vận hành" → **ECS Service Connect**
- "Kết nối service giữa nhiều VPC/account và nhiều loại compute" → **VPC Lattice**
