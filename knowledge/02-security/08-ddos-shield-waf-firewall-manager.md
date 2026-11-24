# DDoS, AWS Shield, AWS WAF & Firewall Manager

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **DDoS (Distributed Denial of Service)** | Tấn công từ chối dịch vụ phân tán: rất nhiều máy (botnet) gửi lượng request khổng lồ làm hệ thống quá tải, người dùng hợp lệ không truy cập được. |
| **AWS Shield** | Dịch vụ chống DDoS: **Standard** (miễn phí, tự động) và **Advanced** (trả phí, nâng cao). |
| **AWS WAF** | *Web Application Firewall* – tường lửa **Layer 7** lọc request HTTP(S) theo rule. |
| **Web ACL** | Tập rule của WAF gắn vào resource (CloudFront, ALB, API Gateway, AppSync, Cognito, App Runner, Verified Access). |
| **AWS Firewall Manager** | Dịch vụ quản lý tập trung rule bảo mật (WAF, Shield Advanced, Security Group, Network Firewall, DNS Firewall) cho **mọi account trong Organization**. |
| **Rate-based rule** | Rule WAF đếm số request theo nguồn trong khoảng thời gian và chặn khi vượt ngưỡng. |

## Nội dung chính

### Các loại tấn công

> 📖 **Định nghĩa:** **Layer 3/4 attack** nhắm vào tầng mạng/vận chuyển (IP, TCP, UDP); **Layer 7 attack** nhắm vào tầng ứng dụng (HTTP).

- **SYN Flood (L4):** gửi quá nhiều yêu cầu mở kết nối TCP
- **UDP Reflection (L4):** lợi dụng server khác gửi gói UDP lớn tới nạn nhân
- **DNS flood:** làm quá tải DNS để user không phân giải được domain
- **Slow Loris:** mở và giữ rất nhiều kết nối HTTP
- **Application-level (L7):** phức tạp hơn, ví dụ **cache busting** – vô hiệu hóa cache để dồn tải xuống DB

### Bảo vệ DDoS trên AWS

> 📖 **Định nghĩa:** **Edge location** là điểm hiện diện toàn cầu của CloudFront/Route 53, hấp thụ và lọc traffic gần người dùng.

- **Shield Standard** (miễn phí, mọi khách hàng)
- **Shield Advanced** (24/7, nâng cao)
- **WAF** lọc request theo rule
- **CloudFront + Route 53**: mạng edge toàn cầu + Shield → giảm thiểu tấn công **tại edge**
- **Sẵn sàng scale**: Auto Scaling
- **Tách static (S3/CloudFront) khỏi dynamic (EC2/ALB)**
- Tham khảo whitepaper *AWS Best Practices for DDoS Resiliency*

### AWS Shield

> 📖 **Định nghĩa:** **Shield Response Team (SRT)** là đội chuyên gia DDoS của AWS hỗ trợ khách hàng Shield Advanced khi bị tấn công.

| | **Shield Standard** | **Shield Advanced** |
|---|---|---|
| Giá | Miễn phí, tự bật | **~$3.000/tháng/organization** (cam kết 1 năm) + phí data |
| Bảo vệ | L3/L4: SYN/UDP flood, reflection | Tấn công tinh vi hơn, kể cả **L7** (kết hợp WAF, **tự động tạo rule WAF**) |
| Resource | Mọi resource | **EC2 (EIP), ELB, CloudFront, Global Accelerator, Route 53** |
| Hỗ trợ | – | **SRT 24/7** (trước gọi DRT) |
| Chi phí | – | **Bảo vệ chi phí** tăng đột biến do DDoS (cost protection) |
| Khác | – | Health-based detection, báo cáo nâng cao, WAF & Firewall Manager không tính phí thêm cho resource được bảo vệ |

### AWS WAF

> 📖 **Định nghĩa:** **SQL injection** chèn câu lệnh SQL độc hại vào input; **XSS (Cross-Site Scripting)** chèn script độc hại vào trang web người khác xem.

- Bảo vệ khỏi khai thác web phổ biến (**Layer 7**)
- Gắn vào: **ALB** (rule cục bộ), **API Gateway** (regional/edge), **CloudFront** (global, tại edge), **AppSync** (GraphQL), Cognito User Pool…
- Muốn bảo vệ CLB, EC2, custom origin, S3 website → đặt **CloudFront + WAF** phía trước
- ⚠️ **WAF không phải công cụ chống DDoS** (chống DDoS là Shield; WAF hỗ trợ L7 bằng rate-based rule)
- **Web ACL rule** dựa trên: IP, HTTP header, body, URI string, **SQLi, XSS**, size constraint, **geo match**, **rate-based**
- **Rule action:** Count | Allow | Block | **CAPTCHA** | **Challenge**

### WAF Managed Rules

> 📖 **Định nghĩa:** **Managed rule group** là bộ rule dựng sẵn do AWS hoặc AWS Marketplace seller duy trì và cập nhật.

- Hơn **190** managed rule
- **Baseline:** `AWSManagedRulesCommonRuleSet`, `AdminProtectionRuleSet`…
- **Theo use case:** `SQLiRuleSet`, `WindowsRuleSet`, `PHPRuleSet`, `WordPressRuleSet`…
- **IP reputation:** `AmazonIpReputationList`, `AnonymousIpList` (chặn VPN, Tor, hosting)
- **Bot Control:** `BotControlRuleSet`
- 🆕 Ngoài ra: **Account Takeover Prevention (ATP)**, **Account Creation Fraud Prevention (ACFP)**, DDoS protection rule group L7

### Logging của WAF

> 📖 **Định nghĩa:** **Web ACL logging** ghi lại thông tin chi tiết từng request được Web ACL đánh giá.

| Đích | Giới hạn |
|---|---|
| **CloudWatch Logs** | 5 MB/giây |
| **S3** | Ghi theo lô 5 phút |
| **Kinesis Data Firehose** | Theo quota Firehose → tiếp tới S3, Redshift, OpenSearch… |

### Kiến trúc: bảo vệ origin của CloudFront

> 📖 **Định nghĩa:** **Origin** là nơi CloudFront lấy nội dung gốc (ALB, S3, server tùy chỉnh).

Vấn đề: người dùng có thể **gọi thẳng ALB**, bỏ qua CloudFront + WAF.
Giải pháp:
1. CloudFront thêm **custom HTTP header** bí mật (vd `X-Origin-Verify: <secret>`) khi gửi tới origin
2. **WAF trên ALB** chỉ cho phép request có header đúng
3. Lưu secret trong **Secrets Manager**, **Lambda xoay vòng** định kỳ và cập nhật cả CloudFront lẫn WAF rule
4. Bổ sung: Security Group của ALB chỉ cho phép **AWS-managed prefix list của CloudFront** (`com.amazonaws.global.cloudfront.origin-facing`); hoặc dùng **CloudFront VPC origins** (ALB private)

### AWS Firewall Manager

> 📖 **Định nghĩa:** **Security policy** (của Firewall Manager) là tập rule bảo mật chung áp dụng tự động lên các account/resource trong phạm vi được chọn.

- Quản lý rule cho **mọi account trong Organization**
- Quản lý: **WAF rules** (ALB, API Gateway, CloudFront), **Shield Advanced** (ALB, CLB, NLB, EIP, CloudFront), **Security Groups** (EC2, ALB, ENI), **Network Firewall** (VPC), **Route 53 Resolver DNS Firewall**, third-party firewall
- Policy tạo theo **region**
- **Tự động áp dụng cho resource mới** và **account mới** → tốt cho tuân thủ
- Yêu cầu: **Organizations (All Features)** + **AWS Config** bật

### WAF vs Firewall Manager vs Shield

> 📖 **Định nghĩa:** Ba dịch vụ bổ trợ nhau: WAF định nghĩa rule, Firewall Manager phân phối rule, Shield chống DDoS.

- **Bảo vệ chi tiết một vài resource** → **WAF** riêng
- **WAF trên nhiều account, tự bảo vệ resource mới** → **Firewall Manager + WAF**
- **Thường xuyên bị DDoS, cần SRT, báo cáo nâng cao, bảo vệ chi phí** → **Shield Advanced**

### Chặn một địa chỉ IP

> 📖 **Định nghĩa:** **NACL (Network ACL)** là tường lửa **stateless** ở mức subnet, có cả rule **Allow và Deny**; **Security Group** là tường lửa **stateful** ở mức ENI, **chỉ có Allow**.

| Kiến trúc | Chặn IP ở đâu |
|---|---|
| Client → EC2 (public) | **NACL** (Deny); SG chỉ có Allow nên không deny được IP; firewall phần mềm trên EC2 (tùy chọn) |
| Client → **ALB** → EC2 | **NACL** của subnet ALB; ALB **kết thúc kết nối** nên EC2 chỉ thấy IP private của ALB → chặn ở EC2 vô ích |
| Client → **NLB** → EC2 | **NACL**; NLB giữ nguyên IP client (client IP preservation) |
| Client → ALB + **WAF** | **WAF IP set** (lọc IP) + NACL |
| Client → **CloudFront** → ALB + WAF | **NACL không giúp được** (ALB chỉ thấy IP của CloudFront) → dùng **WAF trên CloudFront** (IP filtering) hoặc **CloudFront Geo Restriction** |

## 🎯 Mẹo thi

- "SQL injection / XSS" → **WAF**
- "Rate limit theo IP" → **WAF rate-based rule**
- "DDoS L3/L4 miễn phí" → **Shield Standard**; "cần SRT 24/7, bảo vệ chi phí" → **Shield Advanced**
- "Áp WAF/SG thống nhất cho 100 account, tự áp account mới" → **Firewall Manager**
- "Chặn quốc gia" → **WAF geo match** hoặc **CloudFront geo restriction**
- "Ngăn truy cập thẳng ALB bỏ qua CloudFront" → **custom header + WAF** hoặc **prefix list CloudFront** trong SG
- "Chặn IP khi có CloudFront phía trước" → **WAF trên CloudFront**, không phải NACL
