# IAM & IAM Policy

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **IAM (Identity and Access Management)** | Dịch vụ toàn cục (global) của AWS để quản lý *ai* được xác thực (authentication) và được *làm gì* (authorization) trên tài nguyên AWS. |
| **Principal** | Thực thể có thể gửi request tới AWS: IAM user, IAM role (session), federated user, AWS service, root user. |
| **IAM User** | Danh tính lâu dài đại diện cho một người hoặc ứng dụng, có credential dài hạn (password, access key). |
| **IAM Group** | Tập hợp IAM user để gán quyền chung; không phải principal. |
| **IAM Role** | Danh tính có quyền nhưng *không có credential dài hạn*; principal được tin cậy sẽ assume để nhận credential tạm thời. |
| **Instance Profile** | Container chứa một IAM role để gắn vào EC2 instance. |
| **Identity-based policy** | Policy gắn vào user/group/role, quy định danh tính đó được làm gì. |
| **Resource-based policy** | Policy gắn vào resource (S3 bucket, KMS key…), có trường `Principal` quy định ai được truy cập resource. |
| **Managed policy / Inline policy** | Managed: policy độc lập, tái sử dụng (AWS hoặc customer quản lý). Inline: policy nhúng vào đúng một entity. |
| **Access Advisor** | Tính năng hiển thị các service mà entity được cấp quyền và lần cuối thực sự sử dụng. |

## IAM nền tảng

| Thành phần | Đặc điểm cần nhớ |
|---|---|
| **User** | Credential **dài hạn** (password, access key) → hạn chế dùng, ưu tiên federation |
| **Group** | Chỉ chứa user, **không lồng group**, không phải principal (không đặt trong policy được) |
| **Role** | Credential **ngắn hạn** qua STS |
| **EC2 Instance Role** | Lấy credential qua instance metadata (IMDS). **Mỗi instance chỉ gắn 1 role** (qua instance profile) |
| **Service Role** | Role cho service dùng thay bạn (CodeDeploy, API Gateway…) |
| **Cross-account Role** | Role cho principal ở account khác assume |

**Các loại policy:**
- **AWS Managed** – AWS tạo & cập nhật (vd: `AdministratorAccess`, `PowerUserAccess`, `ReadOnlyAccess`)
- **Customer Managed** – bạn tạo, tái sử dụng được, có versioning
- **Inline** – nhúng trực tiếp vào 1 user/role/group, quan hệ 1-1, xóa entity là mất
- **Resource-based** – gắn vào resource (S3 bucket policy, SQS queue policy, KMS key policy…), **có trường `Principal`**

## IAM Policy deep dive

### Cấu trúc

> 📖 **Định nghĩa:** **IAM Policy** là tài liệu JSON mô tả *ai* (principal) được *làm gì* (action) trên *tài nguyên nào* (resource) trong *điều kiện nào* (condition), với kết quả Allow hoặc Deny.

`Version` · `Statement[]` → `Sid`, `Effect` (Allow/Deny), `Principal` (chỉ resource-based / trust policy), `Action` / `NotAction`, `Resource` / `NotResource`, `Condition`.

### Nguyên tắc quan trọng

> 📖 **Định nghĩa:** **Least privilege (đặc quyền tối thiểu)** là nguyên tắc chỉ cấp đúng quyền cần thiết để hoàn thành công việc, không hơn. **Explicit Deny** là câu lệnh `Deny` được ghi rõ; **Implicit Deny** là trạng thái mặc định khi không có `Allow` nào khớp.

- **Explicit Deny > Allow > Implicit Deny (mặc định)**
- Least privilege
- **Access Advisor**: xem quyền đã cấp và **lần cuối được dùng** → dọn quyền thừa
- **Access Analyzer**: tìm resource chia sẻ ra ngoài (xem mục 5)

### `NotAction` – ví dụ `PowerUserAccess`

> 📖 **Định nghĩa:** **`NotAction`** là phần tử khớp với *mọi action ngoại trừ* danh sách được liệt kê. Kết hợp với `Allow` nghĩa là “cho phép tất cả trừ…”, nhưng **không** chặn nếu policy khác cấp lại.

`PowerUserAccess` = Allow **mọi thứ trừ** `iam:*`, `organizations:*`, `account:*` (dùng `NotAction` + `Allow`), rồi thêm một statement Allow lại vài action IAM cần thiết (`iam:CreateServiceLinkedRole`, `iam:ListRoles`…).

> 💡 **`NotAction` + `Allow` ≠ `Deny`**. `NotAction` chỉ là "không cấp", nếu policy khác Allow `iam:*` thì user vẫn có quyền. Còn `Deny` thì chặn tuyệt đối.

### Condition

> 📖 **Định nghĩa:** **Condition** là khối điều kiện tùy chọn trong policy; statement chỉ có hiệu lực khi mọi điều kiện đều đúng. Gồm **operator** (cách so sánh), **condition key** (thuộc tính của request) và **value**.

Cú pháp: `"Condition": { "<operator>": { "<key>": "<value>" } }`

| Nhóm operator | Ví dụ |
|---|---|
| String | `StringEquals`, `StringNotEquals`, `StringLike` (hỗ trợ `*`, `?`) |
| Numeric | `NumericLessThan`… |
| Date | `DateGreaterThan`… (vd giới hạn theo `aws:CurrentTime`) |
| Bool | `aws:SecureTransport` (bắt buộc HTTPS), `aws:MultiFactorAuthPresent` (bắt buộc MFA) |
| IP | `IpAddress` / `NotIpAddress` với `aws:SourceIp` |
| ARN | `ArnEquals`, `ArnLike` (vd `aws:SourceArn`) |
| Null | `"Null": {"aws:RequestTag/Project": "true"}` → **key không tồn tại** |
| Multi-value | `ForAllValues:` / `ForAnyValue:` (xem mục Tag) |

Ví dụ bắt buộc MFA khi xóa:

```json
{
  "Effect": "Deny",
  "Action": ["ec2:TerminateInstances", "s3:DeleteObject"],
  "Resource": "*",
  "Condition": { "BoolIfExists": { "aws:MultiFactorAuthPresent": "false" } }
}
```

### Policy Variables & Tags

> 📖 **Định nghĩa:** **Policy variable** là placeholder dạng `${...}` được thay bằng giá trị thực của request lúc đánh giá (vd `${aws:username}`). **ABAC (Attribute-Based Access Control)** là mô hình phân quyền dựa trên *thuộc tính/tag* của principal và resource thay vì liệt kê từng resource.

- `${aws:username}` → vd mỗi user chỉ truy cập `arn:aws:s3:::mybucket/${aws:username}/*`
- **Global keys:** `aws:CurrentTime`, `aws:TokenIssueTime`, `aws:PrincipalType`, `aws:SecureTransport`, `aws:SourceIp`, `aws:userid`, `aws:RequestedRegion`, `aws:PrincipalOrgID`, `aws:SourceVpce`, `aws:SourceVpc`
- **Service-specific:** `s3:prefix`, `s3:x-amz-acl`, `sns:Endpoint`, `ec2:SourceInstanceARN`…
- **Tag-based (ABAC):**
  - `aws:PrincipalTag/<key>` – tag của người gọi
  - `aws:ResourceTag/<key>` – tag của resource bị tác động
  - `aws:RequestTag/<key>` – tag được truyền trong request (lúc tạo resource)
  - `aws:TagKeys` – danh sách key tag trong request

### Giới hạn tag bằng `aws:TagKeys`

> 📖 **Định nghĩa:** **`aws:TagKeys`** là condition key chứa danh sách *tên* tag key có trong request. **`ForAllValues` / `ForAnyValue`** là set operator để so sánh một key nhiều giá trị với danh sách cho phép.

- `ForAllValues:StringEquals` → **mọi** tag key trong request phải nằm trong danh sách cho phép
- `ForAnyValue:StringEquals` → **ít nhất một** tag key phải nằm trong danh sách

> ⚠️ `ForAllValues` trả về **true nếu request không có tag nào** → muốn bắt buộc phải có tag thì kết hợp thêm `Null` condition.
