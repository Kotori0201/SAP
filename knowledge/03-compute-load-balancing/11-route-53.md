# Amazon Route 53

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Amazon Route 53** | Dịch vụ **DNS** managed, có tính sẵn sàng cao (SLA 100%), kiêm **domain registrar** và **health checking**. |
| **DNS** | Hệ thống phân giải tên miền thành địa chỉ IP. |
| **Hosted zone** | Container chứa các record định tuyến traffic cho một domain và subdomain. |
| **Record** | Bản ghi DNS: tên, loại (A, AAAA, CNAME, NS, Alias…), giá trị, routing policy, TTL. |
| **TTL (Time To Live)** | Thời gian client/resolver **cache** kết quả DNS. |
| **Alias record** | Record **riêng của Route 53** trỏ tên miền tới tài nguyên AWS; dùng được cho **zone apex**, miễn phí truy vấn. |
| **Zone apex** | Tên miền gốc (vd `example.com`, không có subdomain). |
| **Routing policy** | Cách Route 53 trả lời truy vấn DNS (simple, weighted, latency, failover, geolocation, geoproximity, multi-value, IP-based). |
| **Health check** | Cơ chế Route 53 kiểm tra sức khỏe endpoint để tự động **DNS failover**. |
| **Route 53 Resolver** | DNS resolver mặc định trong VPC (`.2`), có **endpoint inbound/outbound** cho hybrid DNS. |

## Nội dung chính

### Loại record

> 📖 **Định nghĩa:** **A** → IPv4; **AAAA** → IPv6; **CNAME** → hostname khác; **NS** → name server của hosted zone.

- **A** – hostname → **IPv4**
- **AAAA** – hostname → **IPv6**
- **CNAME** – hostname → hostname khác (đích phải có A/AAAA); **không tạo được cho zone apex** (`example.com` ❌, `www.example.com` ✅)
- **NS** – name server của hosted zone

### CNAME vs Alias

> 📖 **Định nghĩa:** Xem bảng trên.

| | **CNAME** | **Alias** |
|---|---|---|
| Trỏ tới | Bất kỳ hostname | **Tài nguyên AWS** |
| Zone apex | ❌ Chỉ subdomain | ✅ Cả root & subdomain |
| Phí | Tính phí truy vấn | **Miễn phí** |
| Health check | – | **Native** (evaluate target health) |
| TTL | Bắt buộc | **Không đặt được** (Route 53 tự quản) |

**Alias target:** ELB · CloudFront · API Gateway · Elastic Beanstalk · **S3 website** · **VPC interface endpoint** · **Global Accelerator** · record Route 53 **cùng hosted zone**.
⚠️ **Không** tạo Alias cho **DNS name của EC2**.

### TTL

> 📖 **Định nghĩa:** Xem bảng trên.

| TTL cao (vd 24h) | TTL thấp (vd 60s) |
|---|---|
| Ít truy vấn Route 53 (rẻ) | Nhiều truy vấn ($$) |
| Record có thể **lỗi thời** lâu | Lỗi thời ngắn, **dễ đổi record** |

- TTL **bắt buộc** với mọi record **trừ Alias**
- 💡 Trước khi migrate/đổi IP: **hạ TTL trước** vài giờ

### Routing Policies

> 📖 **Định nghĩa:** Xem bảng trên.

| Policy | Cách hoạt động | Health check | Use case |
|---|---|---|---|
| **Simple** | Thường 1 tài nguyên; nhiều giá trị → client chọn ngẫu nhiên | ❌ | Đơn giản |
| **Weighted** | Chia **% theo trọng số** | ✅ | Load balance giữa region, **test phiên bản mới**; weight 0 = ngừng gửi |
| **Latency-based** | Tới region có **độ trễ thấp nhất** với user | ✅ (có failover) | Ưu tiên độ trễ (user Đức có thể vào US nếu nhanh hơn) |
| **Failover (active-passive)** | Primary + secondary; **health check bắt buộc** cho primary | ✅ | **Disaster Recovery** |
| **Geolocation** | Theo **vị trí user** (châu lục, quốc gia, bang Mỹ — chọn chính xác nhất); cần record **Default** | ✅ | Bản địa hóa, **giới hạn phân phối nội dung**, tuân thủ |
| **Geoproximity** | Theo vị trí **user và tài nguyên**, dịch chuyển bằng **bias** (1..99 mở rộng, −1..−99 thu hẹp) | ✅ | Dồn thêm traffic sang một region; tài nguyên AWS (region) hoặc ngoài AWS (lat/long) |
| **Multi-Value** | Trả **tối đa 8 record khỏe** | ✅ | Load balancing phía client — **không thay thế ELB** |
| **IP-based** | Theo **CIDR của client** (CIDR collection) | ✅ | Tối ưu hiệu năng, giảm chi phí mạng (vd user ISP X → endpoint Y) |

> ⚠️ **Geolocation ≠ Latency**: geolocation theo *vị trí*, latency theo *độ trễ đo được*.

### Traffic Flow

> 📖 **Định nghĩa:** **Traffic flow policy** là cây quyết định định tuyến được dựng bằng **visual editor**, lưu lại, có version và áp cho nhiều hosted zone.

- Đơn giản hóa cấu hình lớn, phức tạp (kết hợp nhiều routing policy lồng nhau)
- **Bắt buộc** dùng Traffic Flow cho **Geoproximity** (trên console cũ; nay đã tạo được record trực tiếp)

### Hosted Zones

> 📖 **Định nghĩa:** **Public hosted zone** định tuyến traffic trên Internet; **private hosted zone** định tuyến trong một hoặc nhiều VPC.

- **Public**: `application1.mypublicdomain.com` → resource public
- **Private**: `application1.company.internal` → IP private trong VPC
- Private hosted zone: VPC phải bật **`enableDnsHostnames`** và **`enableDnsSupport`**
- Associate private zone với VPC **account khác** (CLI/API, cần authorization)

### Good to know

> 📖 **Định nghĩa:** **DNSSEC** là mở rộng bảo mật DNS dùng chữ ký số để bảo vệ tính toàn vẹn và nguồn gốc dữ liệu DNS.

- **DNSSEC**: chống **MITM**; Route 53 hỗ trợ DNSSEC cho **đăng ký domain** và **DNSSEC signing** — **chỉ public hosted zone**
- Mua domain ở **registrar khác**, dùng Route 53 làm DNS → **cập nhật NS record** ở registrar đó

### Health Checks

> 📖 **Định nghĩa:** **Calculated health check** kết hợp kết quả nhiều health check con bằng OR/AND/NOT.

Ba loại:
1. **Theo dõi endpoint** (app, server, tài nguyên AWS)
   - ~**15 health checker toàn cầu**
   - Pass khi trả **2xx/3xx**; có thể kiểm tra **text trong 5.120 byte đầu**
   - **Phải cho phép IP của health checker** (từ `ip-ranges.json`)
   - ⚠️ Health check HTTP **chỉ cho tài nguyên public**
2. **Calculated health check**: tối đa **256 health check con**, chỉ định bao nhiêu con phải pass → bảo trì website mà không fail toàn bộ
3. **Theo dõi CloudWatch Alarm** (kiểm soát hoàn toàn): throttle DynamoDB, alarm RDS, custom metric → **dùng cho tài nguyên private** (health checker ở ngoài VPC)

- Health check tích hợp CloudWatch metrics

**Kiến trúc: failover RDS đa region**
- Cách 1: health check HTTP tới route `/health-db` của app
- Cách 2: **CloudWatch Alarm** trên RDS → gắn với health check
- Khi lỗi: EventBridge/SNS → Lambda **promote read replica** ở region khác → **cập nhật DNS**

### Hybrid DNS & Resolver Endpoints

> 📖 **Định nghĩa:** **Inbound endpoint** cho DNS on-prem **hỏi vào** Route 53 Resolver; **Outbound endpoint** cho Route 53 Resolver **chuyển tiếp ra** DNS on-prem.

- Mặc định Route 53 Resolver trả lời: tên local của EC2, record **private hosted zone**, record **public**
- **Hybrid DNS**: phân giải giữa VPC và mạng khác (VPC peered, **on-prem qua DX/VPN**)

| Endpoint | Chiều | Dùng khi |
|---|---|---|
| **Inbound** | On-prem → AWS | DNS on-prem forward truy vấn `*.aws.private` tới IP của inbound endpoint để phân giải tài nguyên AWS/private hosted zone |
| **Outbound** | AWS → On-prem | Resolver **conditional forward** `*.onprem.private` tới DNS on-prem theo **Resolver Rules** |

- Endpoint gắn với 1+ VPC **cùng region**; tạo ở **2 AZ** để HA
- Mỗi endpoint ~**10.000 truy vấn/giây/IP**

### Resolver Rules

> 📖 **Định nghĩa:** **Conditional forwarding rule** chuyển truy vấn của một domain (và subdomain) tới IP chỉ định; **system rule** ghi đè để giữ phân giải trong AWS.

- **Forwarding rules**: forward domain + subdomain tới IP đích
- **System rules**: ghi đè chọn lọc (vd **không** forward `acme.example.com`)
- **Auto-defined system rules**: domain nội bộ AWS, private hosted zone
- Nhiều rule khớp → chọn **khớp cụ thể nhất**
- **Chia sẻ qua AWS RAM** → quản lý tập trung một account, nhiều VPC dùng chung

## 🎯 Mẹo thi

- "Trỏ `example.com` (root) tới ALB/CloudFront" → **Alias**
- "Chuyển 10% traffic sang bản mới" → **Weighted**
- "Active-passive DR" → **Failover** + health check
- "Nội dung theo quốc gia / chặn quốc gia ở DNS" → **Geolocation** (+ Default)
- "Dồn thêm traffic sang region A" → **Geoproximity bias**
- "Health check tài nguyên **private**" → **CloudWatch Alarm-based health check**
- "On-prem phân giải tên trong private hosted zone" → **Inbound endpoint**
- "EC2 phân giải tên on-prem" → **Outbound endpoint + forwarding rule**
- "Nhiều account dùng chung rule DNS on-prem" → **share Resolver rule qua RAM**
- "Multi-value thay ELB" → ❌
