# IAM Role vs Resource-based Policy

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Assume role** | Hành động lấy credential tạm thời của một role; trong phiên đó principal *chỉ* có quyền của role. |
| **Resource-based policy** | Policy gắn trực tiếp vào resource, cho phép principal (kể cả account khác) truy cập mà không cần đổi danh tính. |
| **Cross-account access** | Truy cập tài nguyên nằm ở một AWS account khác với account của principal. |

## Nội dung chính

| | Assume Role (role làm proxy) | Resource-based policy |
|---|---|---|
| Quyền gốc | **Bỏ quyền gốc**, chỉ có quyền của role | **Giữ nguyên** quyền gốc |
| Cross-account | Principal gọi `sts:AssumeRole` | Principal gọi trực tiếp resource |
| Dùng khi | Service không hỗ trợ resource policy, cần tách biệt quyền | Cần thao tác ở **2 account cùng lúc** |

**Ví dụ kinh điển:** User ở Account A cần **đọc DynamoDB ở A** rồi **ghi vào S3 ở B** → dùng **S3 bucket policy ở B** cho phép user A. Nếu assume role sang B thì user mất quyền đọc DynamoDB ở A.

**Service hỗ trợ resource-based policy:** S3, SNS, SQS, Lambda, ECR, AWS Backup vault, EFS, S3 Glacier vault, Secrets Manager, ACM PCA, **KMS (key policy – bắt buộc)**, CloudWatch Logs destinations, API Gateway, EventBridge event bus, OpenSearch…

> 💡 **EventBridge target:** Lambda/SNS/SQS/S3 dùng **resource-based policy**; Kinesis Data Streams, ECS task, Step Functions… dùng **IAM role**.
