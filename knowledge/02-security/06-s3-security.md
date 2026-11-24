# S3 Security

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Server-Side Encryption (SSE)** | S3 mã hóa object khi ghi xuống đĩa và giải mã khi đọc. |
| **Client-Side Encryption** | Client tự mã hóa dữ liệu trước khi upload; S3 chỉ lưu dữ liệu đã mã hóa. |
| **Bucket policy** | Resource-based policy JSON gắn vào bucket, áp dụng cho toàn bucket, hỗ trợ cross-account. |
| **ACL (Access Control List)** | Cơ chế phân quyền cũ ở mức bucket/object; nay **mặc định tắt** (Object Ownership = Bucket owner enforced). |
| **Pre-signed URL** | URL có chữ ký tạm thời, cho phép người cầm URL GET/PUT một object với quyền của người tạo URL trong thời hạn nhất định. |
| **WORM** | *Write Once Read Many* – ghi một lần, không sửa/xóa được trong thời gian lưu giữ. |
| **S3 Access Point** | Endpoint có tên + policy riêng gắn vào bucket để đơn giản hóa phân quyền cho từng nhóm người dùng/ứng dụng. |
| **Multi-Region Access Point (MRAP)** | Endpoint toàn cầu duy nhất trải trên các bucket ở nhiều region, định tuyến request tới bucket gần nhất. |
| **S3 Object Lambda** | Cơ chế chạy Lambda để biến đổi object *ngay khi được đọc*, trước khi trả về cho ứng dụng. |

## Nội dung chính

### Mã hóa object

> 📖 **Định nghĩa:** **SSE-S3** dùng khóa do S3 quản lý (AES-256, **mặc định cho mọi object mới**); **SSE-KMS** dùng KMS key; **DSSE-KMS** mã hóa hai lớp bằng KMS; **SSE-C** dùng khóa do khách hàng gửi kèm mỗi request.

| Cách | Ai quản lý khóa | Đặc điểm |
|---|---|---|
| **SSE-S3** | AWS/S3 | Mặc định, header `x-amz-server-side-encryption: AES256` |
| **SSE-KMS** | KMS (bạn kiểm soát key policy) | Việc dùng key **ghi vào CloudTrail**; object public cũng **không đọc được** nếu không có quyền KMS; khi `PutObject` cần quyền **`kms:GenerateDataKey`**, khi đọc cần `kms:Decrypt`; chú ý **quota KMS** → bật **S3 Bucket Key** để giảm call |
| **DSSE-KMS** | KMS | Hai lớp mã hóa, cho yêu cầu tuân thủ khắt khe |
| **SSE-C** | **Bạn** (S3 không lưu khóa) | **Bắt buộc HTTPS**; gửi khóa trong header mỗi request |
| **Client-side** | Bạn | Mã hóa trước khi gửi (vd AWS Encryption SDK) |
| **Glacier** | AWS | Mọi dữ liệu mã hóa AES-256 |

### Mã hóa đường truyền

> 📖 **Định nghĩa:** **`aws:SecureTransport`** là condition key = `true` khi request dùng HTTPS.

- S3 có endpoint HTTP và HTTPS → khuyến nghị HTTPS, **bắt buộc với SSE-C**
- Ép HTTPS bằng bucket policy: `Deny` khi `"Bool": {"aws:SecureTransport": "false"}`

### Theo dõi sự kiện trong bucket

> 📖 **Định nghĩa:** **S3 Event Notifications** là cơ chế S3 gửi thông báo khi có sự kiện (tạo/xóa/restore object, replication).

| Cách | Đặc điểm |
|---|---|
| **S3 Access Logs** | Chi tiết từng request; có thể trễ **vài giờ**, **best effort** (có thể thiếu) |
| **S3 Event Notifications** | Đích: SNS, SQS, Lambda; thường vài giây; bật versioning để không bị gộp thông báo khi ghi đồng thời cùng object |
| **EventBridge** | Bật "Send notifications to EventBridge" trên bucket → rule lọc nâng cao, nhiều đích, archive/replay *(slide cũ ghi cần CloudTrail data event – cách cũ)* |
| **Trusted Advisor** | Kiểm tra bucket có **public** không |
| **CloudTrail data events** | Audit ai đọc/ghi object |

### Kiểm soát truy cập

> 📖 **Định nghĩa:** **User-based** là phân quyền qua IAM policy gắn vào danh tính; **Resource-based** là phân quyền qua policy/ACL gắn vào bucket hoặc object.

- **User-based:** IAM policy
- **Resource-based:** **bucket policy** (cross-account), object ACL, bucket ACL (ít dùng, mặc định tắt)
- **Block Public Access** ở mức account/bucket chặn mọi cấu hình public

**Dùng bucket policy để:** cho phép public, **ép mã hóa khi upload**, cấp quyền cross-account.

**Condition hay dùng:**
- `aws:SourceIp` – IP **public** (hoặc EIP); `aws:VpcSourceIp` – IP **private** (qua VPC endpoint)
- `aws:SourceVpce` – một/vài VPC endpoint cụ thể; `aws:SourceVpc` – mọi endpoint trong VPC
- **CloudFront OAC** (Origin Access Control, thay thế OAI cũ)
- MFA, `aws:PrincipalOrgID`

**Quyền theo cấp:**
- `s3:ListBucket` → ARN **bucket** `arn:aws:s3:::test`
- `s3:GetObject`, `PutObject`, `DeleteObject` → ARN **object** `arn:aws:s3:::test/*`

### Pre-signed URL

> 📖 **Định nghĩa:** Xem bảng định nghĩa ở trên.

- Tạo bằng SDK hoặc CLI (download dễ bằng CLI; upload cần SDK)
- Mặc định **3600 giây**, đổi bằng `--expires-in`
- Người dùng URL **thừa hưởng quyền của người tạo URL**
- Use case: chỉ user đăng nhập mới tải video premium; danh sách user thay đổi liên tục; cho phép upload tạm vào đúng vị trí

### VPC Endpoint Gateway cho S3

> 📖 **Định nghĩa:** **Gateway endpoint** là loại VPC endpoint miễn phí cho S3 và DynamoDB, thêm route vào route table để traffic tới S3 đi trong mạng AWS, không qua Internet.

- Instance public đi qua IGW → bucket policy lọc bằng `aws:SourceIp` (IP public)
- Instance private đi qua **VPC endpoint** → lọc bằng `aws:SourceVpce` (1 vài endpoint) hoặc `aws:SourceVpc` (mọi endpoint của VPC)
- ⚠️ `aws:SourceIp` **không dùng được** với traffic qua VPC endpoint (IP là private)

### S3 Object Lock & Glacier Vault Lock

> 📖 **Định nghĩa:** **Retention period** là khoảng thời gian object bị khóa; **Legal hold** là khóa không thời hạn cho đến khi gỡ.

- **S3 Object Lock** (cần versioning): chặn xóa/ghi đè **phiên bản object** trong thời gian quy định
  - **Compliance mode**: không ai (kể cả root) xóa hay rút ngắn được
  - **Governance mode**: user có quyền đặc biệt (`s3:BypassGovernanceRetention`) vẫn xóa được
- **Glacier Vault Lock**: khóa **vault lock policy** → **không sửa policy được nữa** → object không bị xóa
- Dùng cho **tuân thủ và lưu giữ dữ liệu** (SEC 17a-4, FINRA…)

### S3 Access Points

> 📖 **Định nghĩa:** Xem bảng định nghĩa ở trên.

- Mỗi access point có **DNS riêng** và **access point policy** riêng (giống bucket policy)
- Ví dụ: AP *Finance* R/W prefix `/finance`, AP *Sales* R/W `/sales`, AP *Analytics* chỉ đọc toàn bucket → bucket policy chỉ cần ủy quyền cho access point
- **VPC origin**: AP chỉ truy cập được **từ trong VPC** → cần VPC endpoint (gateway/interface); **endpoint policy phải cho phép cả bucket và access point**
- Ba lớp policy: endpoint policy → access point policy → bucket policy

### S3 Multi-Region Access Points

> 📖 **Định nghĩa:** **Failover control** là cơ chế chuyển traffic của MRAP giữa các bucket trong vài phút (active-active hoặc active-passive).

- Một endpoint **toàn cầu** trên nhiều bucket ở nhiều region
- Định tuyến request tới bucket **độ trễ thấp nhất**
- Tạo rule **Cross-Region Replication hai chiều** để đồng bộ
- **Failover controls**: chuyển request giữa region trong **vài phút** (active/active hoặc active/passive)

### S3 Object Lambda

> 📖 **Định nghĩa:** Xem bảng định nghĩa ở trên.

- Chỉ cần **1 bucket** → tạo **S3 Access Point** + **S3 Object Lambda Access Point**
- Use case: **ẩn PII** cho analytics/môi trường non-prod; chuyển định dạng (XML → JSON); resize/watermark ảnh theo người yêu cầu; làm giàu dữ liệu từ DB khác
- 🆕 Lưu ý: AWS đã hạn chế Object Lambda với khách hàng mới (2025) — đề vẫn có thể hỏi khái niệm

## 🎯 Mẹo thi

- "Audit ai dùng khóa mã hóa object" → **SSE-KMS** (CloudTrail)
- "Upload lỗi AccessDenied dù có `s3:PutObject`" → thiếu **`kms:GenerateDataKey`**
- "SSE-KMS bị throttle khi upload nhiều" → bật **S3 Bucket Key** / tăng quota KMS
- "Chỉ cho truy cập bucket từ VPC endpoint X" → `aws:SourceVpce`
- "Không ai, kể cả root, được xóa trong 7 năm" → **Object Lock compliance mode**
- "Nhiều team, policy bucket quá phức tạp" → **Access Points**
- "Endpoint toàn cầu, tự failover giữa region" → **Multi-Region Access Point**
- "Trả về dữ liệu đã ẩn PII mà không lưu bản sao" → **S3 Object Lambda**
