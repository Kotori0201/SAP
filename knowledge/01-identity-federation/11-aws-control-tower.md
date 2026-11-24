# AWS Control Tower

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **AWS Control Tower** | Dịch vụ tự động dựng và quản trị môi trường multi-account (landing zone) theo best practice, chạy trên Organizations. |
| **Landing zone** | Môi trường multi-account được cấu hình sẵn: OU, account log/audit, identity, logging tập trung. |
| **Control (guardrail)** | Quy tắc quản trị áp liên tục: preventive (SCP), detective (Config), proactive (CloudFormation Hooks). |
| **Account Factory** | Cơ chế tạo account mới theo baseline chuẩn, dựa trên Service Catalog. |

## Nội dung chính

- Dựng & quản trị môi trường **multi-account an toàn, tuân thủ** theo best practice — **chạy trên AWS Organizations**
- Lợi ích: setup vài click, quản lý policy liên tục bằng **guardrails (controls)**, phát hiện & khắc phục vi phạm, **dashboard** tuân thủ
- Landing zone mặc định có: OU Security với **Log Archive** account & **Audit** account, IAM Identity Center, CloudTrail/Config tập trung

### Account Factory

> 📖 **Định nghĩa:** **Account Factory** là tính năng của Control Tower để tạo AWS account mới theo mẫu cấu hình đã chuẩn hóa. **Landing zone** là môi trường multi-account đã được thiết lập sẵn theo best practice.

- Tự động cấp account mới với **baseline đã phê duyệt** (VPC mặc định, subnet, region…)
- Dùng **AWS Service Catalog** để provision account
- Mở rộng: **Account Factory for Terraform (AFT)**, **Account Factory Customization (AFC)**, Customizations for Control Tower (CfCT)

### Guardrails / Controls

> 📖 **Định nghĩa:** **Guardrail (nay gọi là Control)** là quy tắc quản trị cấp cao áp dụng liên tục lên các account trong Control Tower, ở dạng phòng ngừa, phát hiện hoặc chủ động.

| Loại | Cơ chế | Ví dụ |
|---|---|---|
| **Preventive** | **SCP** (và RCP/declarative) | Cấm tạo access key cho root |
| **Detective** | **AWS Config** rules | Phát hiện root chưa bật MFA, resource chưa gắn tag |
| **Proactive** | **CloudFormation Hooks** – chặn *trước khi* tạo resource không tuân thủ | Bắt buộc mã hóa khi tạo bucket bằng CFN |

Khắc phục: Config phát hiện NON_COMPLIANT → **SNS** báo admin → **Lambda** tự sửa (vd thêm tag).

| Mức | Ý nghĩa | Ví dụ |
|---|---|---|
| **Mandatory** | Luôn bật, bắt buộc | Không cho public read Log Archive |
| **Strongly Recommended** | Best practice (tùy chọn) | Bật mã hóa EBS |
| **Elective** | Doanh nghiệp hay dùng (tùy chọn) | Cấm xóa object S3 khi không có MFA |

Kết hợp với on-prem AD: Account Factory dựng account → IAM Identity Center xác thực qua **Managed Microsoft AD** có **two-way trust** với AD on-prem qua VPN/DX.
