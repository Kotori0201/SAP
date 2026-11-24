# Amazon Inspector, AWS Config & Amazon GuardDuty

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Amazon Inspector** | Dịch vụ **quét lỗ hổng tự động, liên tục** cho EC2, container image trong ECR và Lambda function. |
| **CVE** | *Common Vulnerabilities and Exposures* – danh mục công khai các lỗ hổng bảo mật đã biết. |
| **AWS Config** | Dịch vụ **ghi lại cấu hình** resource và thay đổi theo thời gian, đánh giá **tuân thủ** theo rule. |
| **Config rule** | Quy tắc đánh giá cấu hình resource là COMPLIANT hay NON_COMPLIANT (managed hoặc custom). |
| **Remediation** | Hành động khắc phục resource không tuân thủ (tự động hoặc thủ công) qua **SSM Automation**. |
| **Amazon GuardDuty** | Dịch vụ **phát hiện mối đe dọa thông minh** dùng ML, phát hiện bất thường và threat intelligence trên log của account. |
| **Finding** | Kết quả phát hiện (lỗ hổng, vi phạm, mối đe dọa) mà dịch vụ bảo mật tạo ra. |

## Nội dung chính

### Amazon Inspector

> 📖 **Định nghĩa:** **Network reachability** là phân tích đường mạng có thể truy cập tới instance từ bên ngoài (port mở, route).

- **EC2**: dùng **SSM Agent** (hoặc **quét agentless** qua EBS snapshot) → phân tích **network reachability** + **lỗ hổng OS/package**
- **Container image trong ECR**: quét **khi push** và liên tục
- **Lambda**: lỗ hổng trong **code và dependency**, quét khi deploy
- **Quét liên tục** khi có thay đổi hoặc CVE mới
- Mỗi lỗ hổng có **risk score** để ưu tiên
- Gửi finding tới **Security Hub** và **EventBridge**
- 🆕 **Inspector code security**: quét mã nguồn/IaC/dependency trong repo (GitHub, GitLab) – liên quan **DevSecOps pipeline scanning** của SAP-C03
- ⚠️ **Chỉ** cho EC2, ECR image, Lambda — không phải RDS, S3…

### AWS Config

> 📖 **Định nghĩa:** **Configuration item** là bản ghi trạng thái cấu hình của một resource tại một thời điểm; **Configuration timeline** là chuỗi các bản ghi đó theo thời gian.

- **Audit & ghi nhận tuân thủ**; xem cấu hình và thay đổi theo thời gian
- **Config rule KHÔNG ngăn hành động** (không deny) → chỉ **phát hiện**; muốn ngăn → SCP/IAM
- Trả lời các câu hỏi: SG có mở SSH ra 0.0.0.0/0? Bucket có public? Cấu hình ALB thay đổi thế nào?
- Thông báo **SNS** khi có thay đổi
- **Theo region**; **aggregate** được nhiều region và account (**Config Aggregator**)
- Với mỗi resource: xem **tuân thủ theo thời gian**, **cấu hình theo thời gian**, và **API call CloudTrail** liên quan

**Config Rules:**
- **AWS managed rules** (hàng trăm rule) hoặc **custom rule** (Lambda hoặc **Guard** – policy-as-code)
- Kích hoạt: **khi cấu hình thay đổi** và/hoặc **định kỳ**
- NON_COMPLIANT → **EventBridge** → Lambda…
- **Auto remediation** bằng **SSM Automation documents** (vd sửa SG, dừng instance thiếu tag), có retry
- **Conformance pack**: gói nhiều rule + remediation triển khai một lần cho account/org

### AWS managed logs – log nào đi đâu

> 📖 **Định nghĩa:** **Access log** là nhật ký ghi lại từng request mà dịch vụ nhận được.

| Log | Đích |
|---|---|
| **ELB access logs** (ALB, NLB, CLB) | S3 |
| **CloudTrail** | S3, CloudWatch Logs |
| **VPC Flow Logs** | S3, CloudWatch Logs, Kinesis Data Firehose |
| **Route 53 query logs** | CloudWatch Logs |
| **S3 access logs** | S3 |
| **CloudFront access logs** | S3 (và real-time logs → Kinesis) |
| **AWS Config** | S3 |

### Amazon GuardDuty

> 📖 **Định nghĩa:** **Threat intelligence** là dữ liệu về IP, domain độc hại đã biết từ AWS và bên thứ 3; **anomaly detection** là phát hiện hành vi lệch khỏi mẫu bình thường.

- Bật **1 click** (dùng thử 30 ngày), **không cần cài phần mềm**
- **Nguồn dữ liệu nền tảng:**
  - **CloudTrail management events** – API bất thường, triển khai trái phép
  - **VPC Flow Logs** – traffic nội bộ/IP bất thường
  - **DNS logs** – instance bị chiếm gửi dữ liệu mã hóa qua DNS (DNS exfiltration)
- **Protection plan tùy chọn:** **S3 Protection** (CloudTrail S3 data events), **EKS Protection** (audit log), **Runtime Monitoring** (EKS, ECS, EC2), **Malware Protection** (EBS, **S3 upload**), **RDS Protection** (login bất thường Aurora/RDS), **Lambda Protection** (network activity)
- Có finding riêng cho **đào tiền mã hóa (crypto mining)**
- Thông báo: **EventBridge rule** → Lambda / SNS
- 🆕 **Extended Threat Detection**: tương quan nhiều tín hiệu thành **attack sequence** nhiều bước

**Delegated Administrator:**
- Management account chỉ định một **member account** làm **GuardDuty delegated admin**
- Delegated admin bật & quản lý GuardDuty cho **mọi account trong org** (tự bật cho account mới)
- Chỉ **management account** mới chỉ định được delegated admin

## 🎯 Mẹo thi

- "Quét lỗ hổng CVE cho EC2/ECR/Lambda" → **Inspector**
- "Quét image khi push lên ECR trong pipeline" → **Inspector** (enhanced scanning)
- "Theo dõi cấu hình thay đổi, ai đổi gì, có tuân thủ không" → **AWS Config**
- "Tự động sửa SG mở 0.0.0.0/0" → **Config rule + SSM Automation remediation**
- "**Ngăn** tạo resource sai" → **không phải Config** (SCP, IAM, CloudFormation Hooks/Control Tower proactive)
- "Phát hiện instance bị chiếm, crypto mining, IP độc hại" → **GuardDuty**
- "Phát hiện malware trong file upload lên S3" → **GuardDuty Malware Protection for S3**
- "Bật GuardDuty cho mọi account, kể cả account mới" → **delegated admin** + auto-enable
