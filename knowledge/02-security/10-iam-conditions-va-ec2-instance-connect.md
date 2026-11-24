# IAM Conditions cho bảo mật, aws:PrincipalOrgID & EC2 Instance Connect

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Condition key** | Thuộc tính của request (IP, region, tag, MFA…) dùng trong khối `Condition` để giới hạn khi nào policy có hiệu lực. |
| **`aws:PrincipalOrgID`** | Condition key chứa ID Organization của principal gọi request; dùng trong resource policy để chỉ cho phép account trong org. |
| **EC2 Instance Connect** | Cách SSH vào EC2 bằng cách đẩy **SSH public key dùng một lần** (hiệu lực 60 giây) qua API `SendSSHPublicKey`, quyền kiểm soát bằng IAM. |

## Nội dung chính

### Các condition key hay dùng

> 📖 **Định nghĩa:** Xem bảng trên.

| Key | Ý nghĩa | Ví dụ dùng |
|---|---|---|
| `aws:SourceIp` | IP client gọi API | Chỉ cho gọi từ IP văn phòng (Deny nếu `NotIpAddress`) |
| `aws:RequestedRegion` | Region API nhắm tới | Chặn region không được phép |
| `ec2:ResourceTag/<key>` / `aws:ResourceTag/<key>` | Tag của resource | Chỉ start/stop instance có tag `Project=DataAnalytics` |
| `aws:PrincipalTag/<key>` | Tag của người gọi | Kết hợp với resource tag cho ABAC (vd `Department` phải khớp) |
| `aws:MultiFactorAuthPresent` | Có MFA không | Bắt buộc MFA khi `StopInstances`/`TerminateInstances` |

> ⚠️ `aws:SourceIp` không hoạt động khi request đi qua **VPC endpoint** hoặc do **AWS service gọi thay bạn** (vd CloudFormation) → cẩn thận khi Deny theo IP (dùng thêm `aws:ViaAWSService`).

### Quyền S3 theo cấp

> 📖 **Định nghĩa:** **Bucket-level permission** áp lên ARN bucket; **object-level permission** áp lên ARN object (`bucket/*`).

- `s3:ListBucket` → `arn:aws:s3:::test`
- `s3:GetObject`, `s3:PutObject`, `s3:DeleteObject` → `arn:aws:s3:::test/*`
- Lỗi thường gặp: gán cả hai loại action vào cùng một ARN → bị từ chối

### `aws:PrincipalOrgID` trong resource policy

> 📖 **Định nghĩa:** **Organization ID** có dạng `o-xxxxxxxxxx`, định danh duy nhất một AWS Organization.

- Dùng trong **mọi resource policy** (S3, KMS, SNS, SQS, Lambda…) để **chỉ cho phép principal thuộc org**
- Không phải liệt kê từng account ID → account mới vào org tự có quyền, account rời org tự mất quyền
- Ví dụ: bucket `2022-financial-data` chỉ cho account trong `o-yyyyyyyyyy`; user ngoài org bị từ chối

```json
{
  "Effect": "Allow",
  "Principal": "*",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::2022-financial-data/*",
  "Condition": { "StringEquals": { "aws:PrincipalOrgID": "o-yyyyyyyyyy" } }
}
```

### EC2 Instance Connect

> 📖 **Định nghĩa:** **`SendSSHPublicKey`** là API đẩy public key tạm thời vào metadata của instance để SSH được trong 60 giây.

- Console/CLI gọi `SendSSHPublicKey` → key **có hiệu lực 60 giây** → SSH vào
- **Security Group phải mở port 22 cho dải IP của dịch vụ EC2 Instance Connect** trong region (lấy từ `ip-ranges.json`, service `EC2_INSTANCE_CONNECT`) — **không phải IP của bạn** khi dùng từ console
- Mọi kết nối **ghi vào CloudTrail**
- **EC2 Instance Connect Endpoint (EIC Endpoint)**: SSH/RDP vào instance **private** không cần bastion, không cần IP public
- So sánh: **SSM Session Manager** không cần mở port 22 nào, ghi log phiên vào S3/CloudWatch → thường là đáp án "an toàn nhất"

## 🎯 Mẹo thi

- "Chỉ account trong org truy cập bucket/key" → **`aws:PrincipalOrgID`**
- "SSH từ console bằng Instance Connect bị timeout" → SG chưa mở port 22 cho **IP range của EC2 Instance Connect**
- "Truy cập shell không mở port, có audit" → **SSM Session Manager**
- "Bắt buộc MFA khi dừng instance" → `aws:MultiFactorAuthPresent`
