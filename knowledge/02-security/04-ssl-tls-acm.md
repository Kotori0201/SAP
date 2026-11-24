# SSL/TLS & AWS Certificate Manager (ACM)

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **SSL / TLS** | Giao thức mã hóa kết nối mạng (encryption in transit). **SSL** (Secure Sockets Layer) là bản cũ; **TLS** (Transport Layer Security) là bản mới – thực tế dùng TLS nhưng vẫn quen gọi "SSL". |
| **Certificate (chứng chỉ)** | Tài liệu số gắn **public key** với một tên miền, được ký bởi **Certificate Authority (CA)** để chứng minh danh tính server. |
| **Certificate Authority (CA)** | Tổ chức tin cậy phát hành chứng chỉ (DigiCert, GlobalSign, Let's Encrypt, Amazon Trust Services…). |
| **SNI (Server Name Indication)** | Phần mở rộng TLS: client gửi **hostname** ngay trong handshake để server chọn đúng chứng chỉ khi một server phục vụ nhiều website. |
| **Man-in-the-Middle (MITM)** | Tấn công kẻ đứng giữa chặn, đọc hoặc sửa traffic giữa client và server. |
| **DNSSEC** | Mở rộng bảo mật cho DNS dùng chữ ký số để chống giả mạo phản hồi DNS. |
| **AWS Certificate Manager (ACM)** | Dịch vụ cấp, lưu và **tự gia hạn** chứng chỉ TLS cho các dịch vụ AWS. |
| **SSL termination / offloading** | Giải mã TLS tại load balancer (hoặc thiết bị chuyên dụng) thay vì tại backend để giảm tải CPU. |

## Nội dung chính

### TLS hoạt động thế nào

> 📖 **Định nghĩa:** **Handshake** là giai đoạn đầu kết nối TLS, nơi client và server thỏa thuận thuật toán, xác minh chứng chỉ và trao đổi khóa phiên.

1. Client gửi *hello*, danh sách cipher suite, số ngẫu nhiên
2. Server trả về số ngẫu nhiên + **chứng chỉ (public key)**
3. Client **xác minh chứng chỉ** (với CA tin cậy)
4. Client sinh **master key (đối xứng)**, mã hóa bằng public key, gửi server
5. (Tùy chọn) server xác minh **chứng chỉ client** → **mutual TLS (mTLS)**
6. Server giải mã master key bằng private key
7. Hai bên giao tiếp bằng **mã hóa đối xứng**

> 💡 Bất đối xứng thì đắt → chỉ dùng trong handshake để trao đổi khóa; dữ liệu thì mã hóa đối xứng (rẻ).

### SNI

> 📖 **Định nghĩa:** Xem bảng trên – SNI cho phép **nhiều chứng chỉ trên một listener**.

- Hỗ trợ bởi **ALB, NLB, CloudFront**; **không** hỗ trợ CLB (thế hệ cũ)
- ALB/NLB: một listener HTTPS gắn nhiều cert, mỗi cert cho một domain → định tuyến tới target group tương ứng

### Tấn công MITM và cách phòng

> 📖 **Định nghĩa:** **DNS spoofing** là giả mạo phản hồi DNS để đưa client tới server của kẻ tấn công.

- Dùng **HTTPS** (không dùng HTTP public)
- Dùng DNS có **DNSSEC** – **Route 53 hỗ trợ DNSSEC** cho cả đăng ký domain và DNS service (ký bằng KMS key)
- Hoặc tự chạy DNS server trên EC2 (Bind, dnsmasq, KnotDNS, PowerDNS)

### AWS Certificate Manager (ACM)

> 📖 **Định nghĩa:** **Public certificate** do CA công khai cấp, trình duyệt tin sẵn; **Private certificate** do **Private CA** của bạn cấp, chỉ hệ thống tin CA đó mới chấp nhận.

- Cách có cert public trên AWS: **tự mua rồi upload** (CLI) hoặc **ACM cấp miễn phí và tự gia hạn**
- ACM gắn cert vào: **ELB** (kể cả do Elastic Beanstalk tạo), **CloudFront**, **API Gateway** (và một số dịch vụ khác)
- **Không** cài trực tiếp cert ACM public lên EC2 (không export được private key của cert public… trừ loại *exportable public certificate* mới, có phí)
- Public cert: phải **xác minh domain** (DNS validation – khuyến nghị, hoặc email)
- Private cert: dùng **AWS Private CA** cho app nội bộ; app phải tin Private CA
- **Gia hạn:** tự động nếu ACM cấp; cert upload thủ công → **tự gia hạn và upload lại** (ACM gửi cảnh báo qua EventBridge/Config trước khi hết hạn)
- **ACM là dịch vụ theo region**: app global (nhiều ALB ở nhiều region) → **cấp cert ở từng region**; **không copy cert giữa region**
- ⚠️ CloudFront chỉ dùng cert ACM ở **us-east-1**

### Kiến trúc SSL

> 📖 **Định nghĩa:** **End-to-end encryption** là mã hóa suốt đường đi từ client tới backend, không giải mã ở giữa.

| Kiến trúc | Cách làm | Khi nào |
|---|---|---|
| **SSL trên ALB** | Cert ACM trên ALB, ALB → EC2 bằng HTTP (hoặc HTTPS lại) | Phổ biến nhất, đơn giản |
| **SSL trên EC2 qua NLB** | NLB **TCP passthrough**, EC2 tự giải mã; EC2 lấy private key lúc boot (user data) từ **SSM Parameter Store** bằng IAM role | Yêu cầu mã hóa end-to-end, cert do mình quản lý; tốn CPU EC2 |
| **CloudHSM SSL offloading** | Web server (NGINX, Apache, IIS) dùng **CloudHSM** để xử lý TLS; **private key không bao giờ rời HSM** | Yêu cầu bảo mật private key cao nhất, tăng tốc TLS; cần tạo **Crypto User (CU)** trên HSM |

## 🎯 Mẹo thi

- "Nhiều domain HTTPS trên một ALB" → **SNI** với nhiều cert
- "Cert cho CloudFront" → ACM ở **us-east-1**
- "App ở 3 region, mỗi region một ALB" → cấp cert ACM **ở cả 3 region**
- "Cert tự mua sắp hết hạn" → ACM **không tự gia hạn cert import** → dùng EventBridge/Config nhắc, hoặc chuyển sang cert ACM
- "Private key không bao giờ được rời thiết bị phần cứng" → **CloudHSM SSL offloading**
- "Mã hóa tới tận instance" → NLB TCP passthrough hoặc ALB → HTTPS tới target
