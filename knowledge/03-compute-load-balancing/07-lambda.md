# AWS Lambda

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **AWS Lambda** | Dịch vụ **serverless compute**: chạy code theo **sự kiện (event)**, không quản lý server, tự scale, trả tiền theo số lần gọi và thời gian chạy (ms). |
| **Runtime** | Môi trường ngôn ngữ để chạy function (Node.js, Python, Java, .NET, Ruby, custom runtime). |
| **Concurrency** | Số lần thực thi function **đang chạy đồng thời** tại một thời điểm. |
| **Reserved concurrency** | Số concurrency **dành riêng (và cũng là giới hạn tối đa)** cho một function. |
| **Provisioned concurrency** | Số môi trường thực thi được **khởi tạo sẵn** để tránh cold start. |
| **Throttle** | Việc Lambda từ chối lời gọi vì vượt giới hạn concurrency (lỗi 429 với sync). |
| **Cold start** | Độ trễ khi Lambda phải tạo môi trường thực thi mới (tải code, khởi tạo runtime). |
| **Alias** | Con trỏ có tên (vd `PROD`) trỏ tới một hoặc hai **version** của function, cho phép chia traffic. |
| **DLQ (Dead-Letter Queue)** | Hàng đợi (SQS/SNS) nhận sự kiện xử lý thất bại sau khi hết lượt retry. |
| **Idempotent** | Xử lý cùng một sự kiện nhiều lần cho kết quả như một lần — cần thiết khi có retry. |

## Nội dung chính

### Tích hợp chính

> 📖 **Định nghĩa:** **Event source** là dịch vụ phát sinh sự kiện kích hoạt Lambda.

API Gateway · Kinesis · DynamoDB Streams · S3 · IoT · EventBridge · CloudWatch Logs · SNS · Cognito · SQS · ALB…

**Ví dụ:**
- **Tạo thumbnail serverless:** ảnh mới vào S3 → trigger Lambda → tạo thumbnail lưu S3 + ghi metadata (tên, kích thước, ngày) vào DynamoDB
- **Serverless CRON:** **EventBridge (Scheduler)** mỗi giờ → Lambda thực hiện tác vụ

### Runtime

> 📖 **Định nghĩa:** **Lambda container image** là function đóng gói dạng image (tối đa 10 GB), phải triển khai **Lambda Runtime API**.

- Node.js, Python, Java, C#/.NET, PowerShell, Ruby
- **Custom Runtime API** (Rust, Go…)
- **Container image** phải implement **Lambda Runtime API**
- Chạy **Docker image tùy ý** → ưu tiên **ECS/Fargate**

### Giới hạn cần nhớ

> 📖 **Định nghĩa:** **Soft limit** là giới hạn tăng được qua Service Quotas; **hard limit** không tăng được.

| Hạng mục | Giới hạn |
|---|---|
| RAM | **128 MB – 10.240 MB (10 GB)** |
| CPU | **Gắn theo RAM** (không chỉnh riêng): ~2 vCPU ở 1.769 MB, **6 vCPU ở 10.240 MB** |
| Timeout | **15 phút** |
| `/tmp` (ephemeral) | **10.240 MB** |
| Deployment package | **50 MB (zip)**, **250 MB (unzip, gồm layers)** |
| Container image | **10 GB** |
| Concurrency | **1.000** (soft, tăng được) |
| Payload | **6 MB sync**, **256 KB async** (🆕 async đã tăng lên 1 MB năm 2025 — kiểm tra lại) |

> 🎯 Tác vụ > 15 phút / cần GPU / cần nhiều RAM hơn → **ECS/Fargate, Batch, Step Functions** chia nhỏ.

### Concurrency & Throttling

> 📖 **Định nghĩa:** Xem *Reserved/Provisioned concurrency* ở bảng trên.

- Mặc định **1.000 concurrent executions / region / account** (dùng chung mọi function)
- **Reserved concurrency** ở mức function = giới hạn riêng
- Vượt giới hạn → **Throttle**
- **Vấn đề:** nếu không giới hạn, một function bị spam (vd qua ALB) dùng hết 1.000 → các function khác (API Gateway, SDK) **bị throttle** → đặt reserved concurrency để cô lập
- **Provisioned concurrency** → loại bỏ cold start (tốn phí); 🆕 **SnapStart** (Java, Python, .NET) giảm cold start bằng snapshot

### Lambda & CodeDeploy

> 📖 **Định nghĩa:** **Traffic shifting** là chuyển dần tỷ lệ request từ version cũ sang version mới qua alias.

- CodeDeploy tự động chuyển traffic của **alias** (tích hợp **SAM**)
- **Linear**: tăng đều mỗi N phút (`Linear10PercentEvery3Minutes`, `Linear10PercentEvery10Minutes`)
- **Canary**: X% trước, sau đó 100% (`Canary10Percent5Minutes`, `Canary10Percent30Minutes`)
- **AllAtOnce**: ngay lập tức
- **Pre/Post traffic hooks** kiểm tra sức khỏe; rollback theo **CloudWatch Alarm**

### Logging, Monitoring, Tracing

> 📖 **Định nghĩa:** **Execution role** là IAM role Lambda dùng khi chạy để gọi AWS service khác.

- **CloudWatch Logs**: log thực thi — execution role cần quyền ghi CloudWatch Logs
- **CloudWatch Metrics**: invocations, errors, duration, throttles, timeouts…
- **X-Ray**: bật trong cấu hình (Lambda chạy X-Ray daemon), dùng SDK trong code, role cần quyền X-Ray

### Lambda trong VPC

> 📖 **Định nghĩa:** **Hyperplane ENI** là ENI dùng chung mà Lambda tạo trong subnet để function truy cập tài nguyên private.

| Triển khai | Truy cập Internet | Truy cập RDS private |
|---|---|---|
| **Mặc định (ngoài VPC)** | ✅ | ❌ |
| **Trong VPC, private subnet** | ❌ trừ khi có **NAT Gateway** (Lambda ở private subnet, NAT ở public subnet) | ✅ (gắn **Security Group**) |
| Trong VPC + **VPC Endpoint** | Truy cập DynamoDB/S3 qua endpoint không cần NAT | ✅ |

- ⚠️ Lambda trong **public subnet không có Internet** (không có IP public)
- Lambda → CloudWatch Logs **vẫn hoạt động** dù không có endpoint/NAT
- **IP public cố định** cho API bên ngoài whitelist → Lambda trong private subnet → **NAT Gateway có Elastic IP** → IGW
- Lambda → RDS nhiều kết nối → dùng **RDS Proxy**

### Synchronous vs Asynchronous

> 📖 **Định nghĩa:** **Sync invocation** chờ kết quả trả về ngay; **async invocation** đưa sự kiện vào hàng đợi nội bộ và trả về ngay, Lambda xử lý sau.

| | **Synchronous** | **Asynchronous** |
|---|---|---|
| Nguồn | CLI, SDK, **API Gateway**, ALB | **S3, SNS, EventBridge**… |
| Kết quả | Trả ngay | Không trả cho caller |
| Xử lý lỗi | **Phía client** (retry, exponential backoff) | Lambda **tự retry (tổng 3 lần)**; cần **idempotent** |
| Lỗi cuối | Client nhận lỗi | **DLQ (SQS/SNS)** hoặc **Lambda Destinations** (khuyến nghị) |

> 💡 **Event source mapping** (SQS, Kinesis, DynamoDB Streams): Lambda **poll** nguồn; lỗi xử lý theo batch (bisect, on-failure destination).

### Thảo luận kiến trúc

> 📖 **Định nghĩa:** **Fan-out** là một sự kiện được phân phát tới nhiều consumer xử lý song song.

- **S3 → SNS → Lambda**: chạy **ngay**, **song song**
- **S3 → SNS → SQS → Lambda**: **theo batch**, **có độ trễ**, chịu tải đột biến tốt hơn, retry bền vững

## 🎯 Mẹo thi

- "Tác vụ chạy > 15 phút" → **không phải Lambda** (Fargate/Batch/Step Functions)
- "Function khác bị throttle vì 1 function chiếm hết concurrency" → **reserved concurrency**
- "Loại bỏ cold start cho API nhạy độ trễ" → **provisioned concurrency** (hoặc SnapStart)
- "Lambda cần IP cố định để đối tác whitelist" → **VPC + NAT Gateway + EIP**
- "Lambda trong VPC không gọi được Internet" → thiếu **NAT Gateway**
- "Deploy Lambda an toàn, 10% traffic trước" → **CodeDeploy canary** qua alias
- "Sự kiện async lỗi không được mất" → **DLQ / on-failure destination**
- "Quá nhiều kết nối DB từ Lambda" → **RDS Proxy**
