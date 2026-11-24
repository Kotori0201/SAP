# IAM Access Analyzer

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **IAM Access Analyzer** | Dịch vụ dùng suy luận logic (automated reasoning) để phân tích policy, phát hiện truy cập ngoài ý muốn, quyền không dùng và kiểm tra/sinh policy. |
| **Zone of trust** | Phạm vi được coi là tin cậy khi tạo analyzer: một AWS account hoặc cả AWS Organization. |
| **Finding** | Kết quả phát hiện của Access Analyzer, vd một bucket cho phép principal ngoài zone of trust truy cập. |

## Nội dung chính

| Tính năng | Mô tả |
|---|---|
| **External access findings** | Tìm resource chia sẻ ra **ngoài Zone of Trust** (= account hoặc organization). Resource: S3, IAM roles, KMS keys, Lambda functions/layers, SQS, Secrets Manager, SNS, EBS/RDS snapshots, ECR, EFS, DynamoDB streams/tables… |
| **Unused access findings** | Tìm role, access key, password, permission **không dùng** |
| **Internal access findings** | Ai *bên trong* org truy cập được resource quan trọng *(mới, 2025)* |
| **Policy validation** | Kiểm tra policy theo grammar & best practices → error, security warning, warning, suggestion |
| **Policy generation** | Sinh policy least-privilege từ **CloudTrail logs (tối đa 90 ngày)** |
| **Custom policy checks** | Kiểm tra policy mới có cấp thêm quyền so với baseline không (dùng trong CI/CD) |

> 🎯 Đề hỏi "*tạo policy least privilege dựa trên hoạt động thực tế*" → **Access Analyzer policy generation**.
