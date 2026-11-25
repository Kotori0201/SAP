# ⭐ Tổng kết · Security

[⬅ Về mục lục section](README.md)

## 1. Bảng tổng hợp kiến thức

| Dịch vụ | Là gì (1 dòng) | Con số phải nhớ | Chọn khi |
|---|---|---|---|
| **CloudTrail** | Ghi lịch sử API call | Event history **90 ngày** · ghi S3 ~**5 phút** · trễ tới **15 phút** | Audit "ai làm gì" |
| **KMS** | Quản lý khóa mã hóa | Encrypt trực tiếp **4 KB** · rotation **90–2560 ngày** (mặc định 365) | Mã hóa mọi dịch vụ |
| **CloudHSM** | HSM single-tenant | FIPS 140 Level 3 · ≥ 2 HSM/2 AZ | Tự quản lý khóa, SSL offload, Oracle TDE |
| **Parameter Store** | Cấu hình & secret phân cấp | **4 KB / 8 KB** · 10k / 100k parameter | Cấu hình rẻ |
| **Secrets Manager** | Secret + xoay vòng tự động | **64 KB** · rotation bằng Lambda | Password DB, xoay vòng |
| **ACM** | Chứng chỉ TLS miễn phí, tự gia hạn | Theo region · CloudFront = **us-east-1** | HTTPS trên ELB/CloudFront/API GW |
| **Shield Standard** | Chống DDoS L3/L4 | Miễn phí | Mặc định |
| **Shield Advanced** | Chống DDoS nâng cao | **~$3.000/tháng/org**, SRT 24/7 | Bị DDoS thường xuyên |
| **WAF** | Tường lửa L7 | Rule: IP, SQLi, XSS, geo, **rate-based** | Lọc request HTTP |
| **Firewall Manager** | Quản lý rule toàn org | Cần Organizations + Config | WAF/SG thống nhất nhiều account |
| **Inspector** | Quét lỗ hổng CVE | EC2, **ECR**, **Lambda** | Vulnerability scanning |
| **Config** | Ghi cấu hình & tuân thủ | Theo region, aggregator | Compliance, phát hiện sai cấu hình |
| **GuardDuty** | Phát hiện mối đe dọa ML | Nguồn: CloudTrail, VPC Flow Logs, DNS | Threat detection |
| **Security Hub** | Dashboard bảo mật tập trung | Cần Config | Gom finding nhiều account |
| **Detective** | Điều tra root cause | Behavior graph | Sau khi có finding |
| **Macie** | Tìm PII trong S3 | ML | Dữ liệu nhạy cảm |

## 2. So sánh quan trọng

### Loại KMS key theo quyền sở hữu

| | Customer managed | AWS managed | AWS owned |
|---|---|---|---|
| Quản lý policy | ✅ | ❌ | ❌ |
| Rotation | Tùy chọn | Mỗi năm | Tùy service |
| Cross-account | ✅ | ❌ | ❌ |

### Nguồn key material

| AWS_KMS | EXTERNAL (BYOK) | CloudHSM key store | External key store (XKS) |
|---|---|---|---|
| Mặc định | Tự import, **không auto-rotate** | Khóa trong HSM của bạn | Khóa ngoài AWS |

### KMS vs CloudHSM

| KMS | CloudHSM |
|---|---|
| Multi-tenant, IAM | Single-tenant, user trên HSM |
| Tích hợp rộng | SSL/TLS & Oracle TDE acceleration |
| AWS lo HA | Bạn thêm HSM nhiều AZ |

### Parameter Store vs Secrets Manager

| Parameter Store | Secrets Manager |
|---|---|
| $ / miễn phí | $$$ |
| Không auto-rotate | **Auto-rotate** (Lambda) |
| KMS tùy chọn | KMS bắt buộc |
| Không multi-region | Multi-region replica |

### Shield vs WAF vs Firewall Manager

| Shield | WAF | Firewall Manager |
|---|---|---|
| DDoS L3/L4 (+L7 Advanced) | Lọc L7 theo rule | Phân phối rule toàn org |

### Inspector vs GuardDuty vs Config vs Macie vs Security Hub vs Detective

| Dịch vụ | Câu hỏi nó trả lời |
|---|---|
| **Inspector** | Có **lỗ hổng phần mềm** (CVE) không? |
| **GuardDuty** | Có **ai đang tấn công / hành vi độc hại** không? |
| **Config** | Cấu hình có **đúng chuẩn** không, ai đổi gì? |
| **Macie** | Có **dữ liệu nhạy cảm** trong S3 không? |
| **Security Hub** | **Tổng hợp tất cả** ở một chỗ |
| **Detective** | **Vì sao / từ đâu** sự cố xảy ra? |

### Mã hóa S3

| SSE-S3 | SSE-KMS | DSSE-KMS | SSE-C | Client-side |
|---|---|---|---|---|
| Mặc định | Audit CloudTrail, cần `kms:GenerateDataKey` | 2 lớp | Bạn giữ khóa, bắt buộc HTTPS | Mã hóa trước khi gửi |

### Chặn IP

| Kiến trúc | Chặn ở |
|---|---|
| EC2 public | NACL |
| ALB | NACL / WAF |
| NLB | NACL |
| CloudFront | **WAF trên CloudFront** / Geo restriction |

## 3. Tip nhớ nhanh

- 🧠 **"Ai xóa? → CloudTrail. Phản ứng ngay? → EventBridge"**.
- 🧠 **"KMS 4 KB → lớn hơn thì Envelope"**.
- 🧠 **"BYOK = Bring Your Own Key = Bạn Yêu cầu tự xoay Khóa"** (không auto-rotate).
- 🧠 **"Multi-Region key = mrk-"**, primary + replica, **không phải global**.
- 🧠 **"Chia sẻ snapshot mã hóa = Customer managed key"**.
- 🧠 **"Secrets = Sẽ tự xoay, Parameter = Phải tự làm"**.
- 🧠 **"CloudFront cert = Virginia (us-east-1)"**; ACM **không tự gia hạn cert import**.
- 🧠 **"HSM = Hardware, Single-tenant, Mình giữ khóa"**.
- 🧠 **"SSE-C = C(ustomer) giữ khóa + HTTPS bắt buộc"**.
- 🧠 **"Object Lock Compliance = Cả root cũng chịu"**, Governance = có quyền đặc biệt thì xóa được.
- 🧠 **"Shield chống lũ, WAF lọc rác, Firewall Manager phát luật"**.
- 🧠 **"SG không có Deny → chặn IP dùng NACL/WAF"**.
- 🧠 **"Inspector = Lỗ hổng, GuardDuty = Kẻ gian, Config = Cấu hình, Macie = Dữ liệu nhạy cảm, Detective = Thám tử"**.
- 🧠 **"Config phát hiện, không ngăn chặn"** → ngăn chặn = SCP/IAM.
- 🧠 **"Inspector chỉ quét 3 thứ: EC2, ECR, Lambda"**.
- 🧠 **"Org-only resource → aws:PrincipalOrgID"**.
