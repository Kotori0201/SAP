# AWS CloudTrail

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **AWS CloudTrail** | Dịch vụ ghi lại lịch sử **API call** và hoạt động trong AWS account (từ Console, SDK, CLI, AWS service), phục vụ quản trị, tuân thủ và audit. |
| **Event** | Bản ghi một hoạt động: ai gọi, gọi API gì, lúc nào, từ IP nào, trên resource nào, kết quả ra sao. |
| **Trail** | Cấu hình gửi event liên tục tới S3 (và tùy chọn CloudWatch Logs) để lưu lâu dài. |
| **Organization trail** | Trail tạo ở management account (hoặc delegated admin), tự ghi log của **mọi account** trong Organization. |
| **CloudTrail Lake** | Kho dữ liệu event được quản lý, truy vấn bằng SQL, lưu tối đa nhiều năm. |
| **Log file integrity validation** | Cơ chế dùng hash SHA-256 và chữ ký số để chứng minh file log không bị sửa/xóa sau khi CloudTrail ghi. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** **Event history** là lịch sử 90 ngày gần nhất của management event, **bật sẵn mặc định** trong mọi account, không cần tạo trail.

- CloudTrail **bật sẵn** (event history 90 ngày)
- Gửi log tới **S3** và/hoặc **CloudWatch Logs**
- Trail áp dụng **tất cả region** (mặc định) hoặc 1 region
- Resource bị xóa, cần biết ai làm → **xem CloudTrail đầu tiên**

### Các loại event

> 📖 **Định nghĩa:** **Management event** là thao tác *trên cấu hình* resource (control plane). **Data event** là thao tác *trên dữ liệu bên trong* resource (data plane). **Insights event** là event bất thường do CloudTrail Insights phát hiện.

| Loại | Mặc định | Ví dụ | Ghi chú |
|---|---|---|---|
| **Management** | ✅ Ghi | `AttachRolePolicy`, `CreateSubnet`, `CreateTrail` | Tách được **Read** / **Write** |
| **Data** | ❌ Không ghi (khối lượng lớn) | S3 `GetObject`/`PutObject`/`DeleteObject`, Lambda `Invoke`, DynamoDB item-level… | Tách được Read / Write; tốn phí |
| **Insights** | ❌ Phải bật | Burst IAM action, chạm service limit | Xem mục dưới |

### CloudTrail Insights

> 📖 **Định nghĩa:** **CloudTrail Insights** là tính năng dùng management event bình thường để xây **baseline**, rồi liên tục phân tích **write event** để phát hiện mẫu hoạt động bất thường.

- Phát hiện: tạo resource sai/bất thường, chạm service limit, **burst IAM actions**, gián đoạn hoạt động định kỳ
- Kết quả: hiện trên console, gửi về **S3**, sinh **EventBridge event** để tự động hóa

### Lưu trữ (retention)

> 📖 **Định nghĩa:** **Retention** là thời gian dữ liệu được giữ lại trước khi bị xóa.

- Event history giữ **90 ngày**
- Lâu hơn → trail gửi về **S3**, phân tích bằng **Athena** (hoặc dùng **CloudTrail Lake**)

### Kiến trúc: gửi log về S3

> 📖 **Định nghĩa:** **SSE-KMS** là mã hóa phía server của S3 dùng khóa trong KMS; **S3 Object Lock** là cơ chế WORM chống xóa/sửa object.

- CloudTrail ghi vào S3 khoảng **mỗi 5 phút**
- Mã hóa: **SSE-S3 (mặc định)** hoặc **SSE-KMS**
- Bảo vệ bucket log: **Versioning**, **MFA Delete**, **Lifecycle** (IA → Glacier), **Object Lock**
- Bật **Log file integrity validation** (SHA-256 hash + ký số)
- S3 Event Notifications → SQS/SNS/Lambda để xử lý tiếp

### Kiến trúc: tập trung log multi-account

> 📖 **Định nghĩa:** **Cross-account delivery** là việc trail ở account A ghi log vào bucket nằm ở account B (thường là Security/Log Archive account).

- Trail ở mỗi account ghi vào bucket ở **Security account**, theo prefix `account-A/`, `account-B/`…
- **Bắt buộc bucket policy** cho phép CloudTrail của các account ghi
- Account A muốn đọc log của mình: (1) **cross-account role** hoặc (2) sửa **bucket policy**
- Cách hiện đại: **Organization trail** từ management account → bucket trung tâm, prefix `o-orgid/<account-id>/`; member account **không tắt/sửa được**

### Kiến trúc: cảnh báo theo API call

> 📖 **Định nghĩa:** **Metric filter** là quy tắc trong CloudWatch Logs đếm số lần xuất hiện một mẫu trong log và biến nó thành CloudWatch metric để đặt alarm.

- CloudTrail → **CloudWatch Logs** → **Metric Filter** → **CloudWatch Alarm** → **SNS**
- Ví dụ: đếm `TerminateInstances`, số API call mỗi user, số lượng call bị **AccessDenied** cao

### Phản ứng với event nhanh nhất?

> 📖 **Định nghĩa:** **Amazon EventBridge** là event bus serverless nhận event từ AWS service (kể cả API call ghi bởi CloudTrail) và định tuyến tới target theo rule.

CloudTrail có thể mất **tới ~15 phút** để ghi event vào trail.

| Cách | Tốc độ | Dùng khi |
|---|---|---|
| **EventBridge** | **Nhanh nhất** – rule cho bất kỳ API call nào | Phản ứng tức thì (vd ai đó `DeleteTable` → SNS) |
| **CloudWatch Logs** | Stream | Metric filter, phát hiện bất thường theo tần suất |
| **S3** | ~5 phút/lần | Lưu dài hạn, kiểm tra integrity, gửi cross-account |

Ví dụ EventBridge: `AssumeRole` → SNS báo; `AuthorizeSecurityGroupIngress` → SNS báo ai mở security group.

## 🎯 Mẹo thi

- "Ai đã xóa resource?" → **CloudTrail**
- "Phản ứng **ngay** khi có API call X" → **EventBridge rule** (không phải S3/CloudWatch Logs)
- "Chứng minh log không bị chỉnh sửa" → **Log file integrity validation**
- "Ghi log mọi account, member không tắt được" → **Organization trail**
- "Truy vấn log CloudTrail nhiều năm bằng SQL" → **S3 + Athena** hoặc **CloudTrail Lake**
- CloudTrail **không** ghi các câu query bên trong database (vd SQL trên RDS)
