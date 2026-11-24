# SSM Parameter Store & AWS Secrets Manager

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **SSM Parameter Store** | Tính năng của AWS Systems Manager lưu trữ **cấu hình và bí mật** dạng key-value theo cấu trúc phân cấp, serverless, tùy chọn mã hóa bằng KMS. |
| **SecureString** | Loại parameter được mã hóa bằng KMS key. |
| **AWS Secrets Manager** | Dịch vụ chuyên lưu trữ **secret** (mật khẩu, API key…), hỗ trợ **xoay vòng tự động** bằng Lambda, mã hóa KMS bắt buộc. |
| **Secret rotation** | Quá trình định kỳ tạo giá trị secret mới và cập nhật đồng thời ở nơi lưu secret lẫn hệ thống đích (vd đổi password DB). |
| **Parameter policy** | Quy tắc gắn vào *advanced parameter* để đặt hạn sử dụng hoặc nhắc nhở thay đổi. |

## Nội dung chính

### SSM Parameter Store

> 📖 **Định nghĩa:** **Parameter** là một cặp tên–giá trị; tên có thể theo dạng đường dẫn (`/dept/app/env/key`) để tổ chức phân cấp.

- Lưu cấu hình (plaintext) và bí mật (SecureString – mã hóa KMS)
- Serverless, bền vững, SDK đơn giản, có **version tracking**
- Bảo mật bằng **IAM**; nhận thông báo qua **EventBridge**; tích hợp **CloudFormation**
- Phân cấp: `GetParameters` hoặc **`GetParametersByPath`** lấy cả cây (vd Lambda dev đọc `/my-app/dev/*`)
- Tham chiếu đặc biệt:
  - `/aws/reference/secretsmanager/<secret_id>` → đọc **secret của Secrets Manager qua API Parameter Store**
  - `/aws/service/ami-amazon-linux-latest/...` → **parameter public** do AWS cung cấp (vd AMI mới nhất)

### Standard vs Advanced tier

> 📖 **Định nghĩa:** **Tier** là gói dịch vụ của parameter, quyết định giới hạn kích thước, số lượng và tính năng.

| | Standard | Advanced |
|---|---|---|
| Số parameter (mỗi account/region) | 10.000 | 100.000 |
| Kích thước tối đa | 4 KB | 8 KB |
| Parameter policies | ❌ | ✅ |
| Chi phí | Miễn phí | ~$0.05/parameter/tháng |

### Parameter Policies (chỉ advanced)

> 📖 **Định nghĩa:** **TTL (Time To Live)** là thời hạn tồn tại; hết hạn thì parameter bị xử lý theo policy.

- **Expiration** – xóa parameter khi hết hạn (ép đổi password)
- **ExpirationNotification** – gửi EventBridge event trước khi hết hạn
- **NoChangeNotification** – gửi EventBridge nếu parameter không được đổi sau X ngày
- Gắn được nhiều policy cùng lúc

### AWS Secrets Manager

> 📖 **Định nghĩa:** **Secret** là thông tin nhạy cảm cần bảo vệ (credential DB, API key, token) kèm metadata và các phiên bản (`AWSCURRENT`, `AWSPENDING`, `AWSPREVIOUS`).

- **Xoay vòng secret tự động** mỗi X ngày, sinh secret mới bằng **Lambda**
- **Hỗ trợ sẵn** (Lambda có sẵn): **Amazon RDS** (mọi engine), **Aurora**, **Redshift**, **DocumentDB**
- DB/service khác → tự viết **custom Lambda**
- Kiểm soát truy cập bằng **resource-based policy**
- Tích hợp đọc secret trực tiếp: CloudFormation, CodeBuild, **ECS/Fargate** (inject làm biến môi trường), EKS, EMR, Parameter Store…
- **Multi-Region secrets**: replicate secret sang region khác, đồng bộ với primary → phục vụ DR

**Với CloudFormation:** `AWS::SecretsManager::Secret` sinh password → tham chiếu trong `AWS::RDS::DBInstance` (`{{resolve:secretsmanager:...}}`) → `SecretTargetAttachment` liên kết secret với DB để rotation hoạt động. Hoặc dùng `ManageMasterUserPassword` để RDS tự quản lý secret.

### Chia sẻ secret cross-account

> 📖 **Định nghĩa:** **`kms:ViaService`** là condition key giới hạn KMS key chỉ được dùng khi request đi qua một AWS service cụ thể.

Cần **2 lớp quyền**:
1. **Resource-based policy của secret**: cho account đích `secretsmanager:GetSecretValue`
2. **Key policy của KMS key** (customer managed): cho account đích `kms:Decrypt` với điều kiện `kms:ViaService = secretsmanager.<region>.amazonaws.com`

> ⚠️ Secret mã hóa bằng **AWS managed key `aws/secretsmanager` không chia sẻ cross-account được** → phải dùng customer managed key.

### So sánh Parameter Store vs Secrets Manager

> 📖 **Định nghĩa:** Cả hai đều là kho lưu trữ giá trị nhạy cảm; khác biệt chính là **khả năng xoay vòng tích hợp** và **chi phí**.

| | **Secrets Manager** ($$$) | **Parameter Store** ($) |
|---|---|---|
| Xoay vòng tự động | ✅ Tích hợp, Lambda có sẵn cho RDS/Redshift/DocumentDB | ❌ Tự làm: **EventBridge (lịch) → Lambda** đổi giá trị |
| Mã hóa KMS | **Bắt buộc** | Tùy chọn (SecureString) |
| CloudFormation | ✅ | ✅ |
| Cross-account | ✅ resource policy | ❌ (advanced parameter share được qua RAM) |
| Multi-Region replicate | ✅ | ❌ |
| Đọc lẫn nhau | – | Đọc được secret của Secrets Manager qua API Parameter Store |
| Kích thước | 64 KB | 4 KB / 8 KB |

## 🎯 Mẹo thi

- "Tự động xoay mật khẩu RDS **mỗi 30 ngày**, ít vận hành nhất" → **Secrets Manager**
- "Lưu cấu hình/biến môi trường rẻ, phân cấp theo môi trường" → **Parameter Store**
- "Ép password hết hạn sau X ngày trong Parameter Store" → **Advanced tier + Expiration policy**
- "ECS task cần password DB" → Secrets Manager, inject qua task definition (`secrets`)
- "Chia sẻ secret cho account khác" → resource policy + **customer managed KMS key**
- "Secret cần dùng ở region DR" → **Multi-Region secret (replica)**
