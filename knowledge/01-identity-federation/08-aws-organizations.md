# AWS Organizations

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **AWS Organizations** | Dịch vụ quản lý tập trung nhiều AWS account: gom nhóm, gộp hóa đơn và áp chính sách. |
| **Management account** | Account tạo ra Organization, trả tiền cho toàn bộ org (payer), không bị SCP ràng buộc. |
| **Member account** | Account thuộc Organization, chịu các policy của org. |
| **Organizational Unit (OU)** | Nhóm logic chứa account hoặc OU con, dùng để áp policy theo cấp. |
| **Root** | Container gốc của mọi OU và account trong Organization. |
| **Consolidated billing** | Gộp hóa đơn mọi account vào management account, hưởng giảm giá theo tổng sử dụng. |

## Nội dung chính

- **Management account** (payer) + **member accounts**, gom thành cây **OU** từ **Root**
- Mỗi account chỉ thuộc **1 organization** tại một thời điểm

### OrganizationAccountAccessRole

> 📖 **Định nghĩa:** **OrganizationAccountAccessRole** là IAM role mặc định trong member account, tin tưởng management account và có quyền `AdministratorAccess`.

- Role **full admin** trong member account, cho phép management account assume để quản trị
- **Tự tạo** khi tạo account mới qua Organizations
- **Phải tạo thủ công** nếu **mời (invite)** account có sẵn

### Chiến lược multi-account

> 📖 **Định nghĩa:** **Multi-account strategy** là cách chia workload ra nhiều AWS account để cô lập bảo mật, giới hạn **blast radius** (phạm vi ảnh hưởng khi có sự cố), tách hóa đơn và service quota.

- Tách account theo: phòng ban, cost center, dev/test/prod, yêu cầu tuân thủ (SCP), **cô lập resource**, **service quota riêng**, account riêng cho **logging** và **security**
- Multi-account vs single account multi-VPC: multi-account cô lập mạnh hơn (blast radius, quota, billing)
- Tag chuẩn hóa cho billing
- **CloudTrail organization trail** → gửi log về S3 ở **Log Archive account**
- CloudWatch Logs tập trung về logging account

**Ví dụ cấu trúc OU:** theo Business Unit (Sales/Retail/Finance), theo môi trường (Prod/Dev/Test), theo dự án. Khuyến nghị hiện nay: OU **Security**, **Infrastructure**, **Workloads (Prod/Non-prod)**, **Sandbox**, **Suspended**, **Policy Staging**.

### Feature modes

> 📖 **Định nghĩa:** **Feature mode** là chế độ hoạt động của Organization: chỉ gộp hóa đơn (**Consolidated Billing**) hoặc đầy đủ tính năng quản trị (**All Features**).

| Consolidated Billing only | **All Features** (mặc định) |
|---|---|
| Gộp hóa đơn, 1 phương thức thanh toán | Gồm consolidated billing **+ SCP và các policy khác** |
| Giảm giá theo tổng usage (volume discount EC2, S3…) | Account được mời **phải đồng ý** bật All Features |
| | Có thể dùng SCP **chặn member rời khỏi org** |
| | **Không thể quay về** Consolidated Billing only |

### Reserved Instances / Savings Plans

> 📖 **Định nghĩa:** **Reserved Instance (RI)** và **Savings Plans** là hình thức cam kết sử dụng 1–3 năm để được giảm giá; trong Organization, mức giảm có thể **chia sẻ** giữa các account.

- Consolidated billing coi cả org như **1 account** → RI/SP mua ở account nào cũng được áp dụng cho account khác
- Management account **tắt được việc chia sẻ discount** cho bất kỳ account nào (kể cả chính nó)
- Muốn chia sẻ giữa 2 account → **cả hai phải bật sharing**

### Di chuyển account sang org khác

> 📖 **Định nghĩa:** **Invitation (lời mời)** là cơ chế đưa account đã tồn tại vào Organization; account phải chủ động chấp nhận.

1. (Rời org cũ – nếu SCP cho phép)
2. Org mới **gửi invite**
3. Member account **chấp nhận invite**
