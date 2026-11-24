# 02 · Security

> Nguồn: Slide Stephane Maarek SAP v38 (trang 80–173) + section *Security* trên Udemy, có bổ sung cập nhật AWS (🆕). Ghi chú viết lại để ôn tập.

**Domain SAP-C03:** Security, compliance & governance

## 📖 Định nghĩa tổng quan

- **Encryption at rest:** mã hóa dữ liệu khi lưu trữ (đĩa, snapshot, object).
- **Encryption in transit (in-flight):** mã hóa dữ liệu khi truyền qua mạng (TLS).
- **Audit:** ghi nhận và kiểm tra lại ai đã làm gì, khi nào (CloudTrail, Config).
- **Threat detection:** phát hiện hoạt động độc hại hoặc bất thường (GuardDuty, Inspector, Macie).
- **Perimeter protection:** bảo vệ biên ứng dụng trước tấn công từ Internet (Shield, WAF, Firewall Manager).
- **Secret management:** lưu trữ và xoay vòng thông tin nhạy cảm (Secrets Manager, Parameter Store).

## Mục lục

| # | Chủ đề | File |
|---|---|---|
| 1 | AWS CloudTrail | [01-cloudtrail.md](01-cloudtrail.md) |
| 2 | AWS KMS (+ 🆕 post-quantum ML-DSA/ML-KEM) | [02-kms.md](02-kms.md) |
| 3 | SSM Parameter Store & Secrets Manager | [03-parameter-store-va-secrets-manager.md](03-parameter-store-va-secrets-manager.md) |
| 4 | SSL/TLS & ACM | [04-ssl-tls-acm.md](04-ssl-tls-acm.md) |
| 5 | CloudHSM | [05-cloudhsm.md](05-cloudhsm.md) |
| 6 | S3 Security | [06-s3-security.md](06-s3-security.md) |
| 7 | RDS Security | [07-rds-security.md](07-rds-security.md) |
| 8 | DDoS, Shield, WAF, Firewall Manager | [08-ddos-shield-waf-firewall-manager.md](08-ddos-shield-waf-firewall-manager.md) |
| 9 | Inspector, Config, GuardDuty | [09-inspector-config-guardduty.md](09-inspector-config-guardduty.md) |
| 10 | IAM Conditions, aws:PrincipalOrgID, EC2 Instance Connect | [10-iam-conditions-va-ec2-instance-connect.md](10-iam-conditions-va-ec2-instance-connect.md) |
| 11 | Security Hub & Detective | [11-security-hub-va-detective.md](11-security-hub-va-detective.md) |

## Cheat-sheet: Tình huống → Giải pháp

| Tình huống trong đề | Giải pháp |
|---|---|
| Biết ai đã xóa resource | **CloudTrail** |
| Phản ứng ngay khi có API call nhạy cảm | **EventBridge rule** (từ CloudTrail event) |
| Log CloudTrail mọi account, member không tắt được | **Organization trail** |
| Chứng minh log không bị sửa | **CloudTrail log file integrity validation** |
| Cảnh báo khi có nhiều lỗi AccessDenied | CloudTrail → CloudWatch Logs **metric filter** → alarm |
| Mã hóa dữ liệu lớn bằng KMS | **Envelope encryption** (`GenerateDataKey`) |
| Bên ngoài AWS mã hóa dữ liệu gửi cho bạn | **KMS asymmetric key**, phát public key |
| Tự mang khóa vào AWS | **KMS BYOK (EXTERNAL)** – xoay thủ công |
| Khóa phải nằm trong HSM single-tenant của mình | **CloudHSM** / **KMS Custom Key Store** |
| Khóa không bao giờ nằm trong AWS | **KMS External Key Store (XKS)** |
| Mã hóa ở region A, giải mã ở region B (DR, Global Tables) | **KMS Multi-Region key** |
| Chia sẻ snapshot mã hóa sang account khác | **Customer managed key** + key policy |
| Chữ ký số chống máy tính lượng tử | **KMS ML-DSA key** 🆕 |
| Chống "harvest now, decrypt later" khi gọi KMS/Secrets Manager | **Hybrid post-quantum TLS (ML-KEM)** 🆕 |
| Tự xoay mật khẩu RDS định kỳ | **Secrets Manager** |
| Cấu hình phân cấp, rẻ | **Parameter Store** |
| Chia sẻ secret cross-account | Secret resource policy + **customer managed KMS key** |
| Nhiều HTTPS domain trên một ALB | **SNI** |
| Cert cho CloudFront | ACM ở **us-east-1** |
| App nhiều region | Cert ACM ở **từng region** |
| TLS acceleration / private key không rời phần cứng | **CloudHSM SSL offloading** |
| Ép HTTPS khi truy cập S3 | Bucket policy `aws:SecureTransport` |
| Audit việc dùng khóa mã hóa object S3 | **SSE-KMS** |
| Chỉ truy cập bucket từ VPC endpoint X | `aws:SourceVpce` |
| Không ai xóa được object trong N năm | **S3 Object Lock – compliance mode** |
| Nhiều team dùng chung bucket, policy phức tạp | **S3 Access Points** |
| Endpoint S3 toàn cầu có failover | **Multi-Region Access Point** |
| Ẩn PII khi đọc object, không lưu bản sao | **S3 Object Lambda** |
| Mã hóa RDS đang chạy | Snapshot → copy có mã hóa → restore |
| Audit query SQL trong RDS | Database Activity Streams / audit log (không phải CloudTrail) |
| Chặn SQLi/XSS, rate limit | **WAF** |
| DDoS L3/L4 miễn phí | **Shield Standard** |
| Cần SRT 24/7, bảo vệ chi phí khi bị DDoS | **Shield Advanced** |
| Áp WAF/SG thống nhất toàn org, tự áp account mới | **Firewall Manager** |
| Ngăn truy cập thẳng ALB bỏ qua CloudFront | Custom header + WAF (secret xoay bằng Secrets Manager) / prefix list CloudFront |
| Chặn IP khi đi qua ALB | **NACL** hoặc **WAF** (không chặn ở EC2) |
| Chặn IP khi có CloudFront phía trước | **WAF trên CloudFront** / Geo restriction |
| Quét CVE cho EC2, ECR, Lambda | **Inspector** |
| Quét lỗ hổng code/dependency trong pipeline | **Inspector** (ECR scan, code security) 🆕 |
| Theo dõi cấu hình & tuân thủ, tự sửa | **Config rules + SSM Automation remediation** |
| Phát hiện crypto mining, IP độc hại, instance bị chiếm | **GuardDuty** |
| Malware trong file upload S3 | **GuardDuty Malware Protection for S3** |
| Bật GuardDuty cho toàn org | **Delegated administrator** |
| Chỉ principal trong org truy cập resource | **`aws:PrincipalOrgID`** |
| Shell vào instance không mở port 22 | **SSM Session Manager** |
| Một dashboard bảo mật cho nhiều account | **Security Hub** |
| Điều tra root cause finding | **Detective** |
| Tìm PII trong S3 | **Macie** |

## Bẫy hay gặp trong đề

- ❌ "Config rule ngăn chặn hành động" → Config chỉ **phát hiện** (có thể tự sửa sau đó).
- ❌ "WAF chống DDoS" → chống DDoS là **Shield**; WAF lọc L7.
- ❌ "Chặn IP bằng Security Group" → SG **không có Deny**; dùng NACL/WAF.
- ❌ "Chặn IP ở EC2 sau ALB" → EC2 chỉ thấy IP của ALB.
- ❌ "ACM tự gia hạn cert import" → **chỉ cert do ACM cấp**.
- ❌ "Copy cert ACM sang region khác" → **không được**, cấp mới.
- ❌ "BYOK có automatic rotation" → **không**.
- ❌ "Chia sẻ snapshot mã hóa bằng AWS managed key" → phải dùng **customer managed key**.
- ❌ "CloudTrail ghi query SQL" → **không**.
- ❌ "Multi-Region key là global key" → là primary + replica quản lý độc lập.
- ❌ "Inspector quét RDS/S3" → chỉ **EC2, ECR, Lambda**.
- ❌ "`aws:SourceIp` lọc traffic qua VPC endpoint" → dùng `aws:SourceVpce`/`aws:VpcSourceIp`.
