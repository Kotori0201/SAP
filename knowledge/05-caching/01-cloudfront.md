# Amazon CloudFront

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **CDN (Content Delivery Network)** | Mạng máy chủ phân tán toàn cầu **cache nội dung gần người dùng** để tăng tốc truy cập và giảm tải cho origin. |
| **Amazon CloudFront** | Dịch vụ CDN của AWS, cache nội dung tại **edge location**, tích hợp Shield, WAF, Route 53. |
| **Point of Presence (PoP)** | Điểm hiện diện của CloudFront: gồm **edge location** (gần user) và **Regional Edge Cache** (lớp cache trung gian lớn hơn). |
| **Origin** | Nơi CloudFront lấy nội dung gốc (S3, ALB, EC2, HTTP server…). |
| **Distribution** | Cấu hình CloudFront gồm origin, cache behavior, domain, chứng chỉ… |
| **TTL (cache)** | Thời gian object được giữ trong cache trước khi CloudFront hỏi lại origin. |
| **Cache key** | Tập thuộc tính (URL, header, cookie, query string) dùng để xác định một object trong cache. |
| **OAC (Origin Access Control)** | Cơ chế cho phép **chỉ CloudFront** đọc/ghi S3 bucket private (thay thế OAI cũ). |
| **Signed URL / Signed Cookie** | URL/cookie có chữ ký và hạn dùng, giới hạn ai được truy cập nội dung CloudFront. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** Xem bảng trên.

- CDN → **tăng hiệu năng đọc**, cache tại edge
- **225+ PoP** (215+ edge location & 13 Regional Edge Cache) – con số tăng theo thời gian
- Chống tấn công tầng mạng & ứng dụng (DDoS), tích hợp **Shield, WAF, Route 53**
- Expose HTTPS ra ngoài và nói chuyện HTTPS với backend nội bộ
- Hỗ trợ **WebSocket**

### Các loại Origin

> 📖 **Định nghĩa:** **VPC Origin** là tính năng cho CloudFront kết nối tới ALB/NLB/EC2 nằm trong **private subnet**.

| Origin | Dùng cho |
|---|---|
| **S3 bucket** | Phân phối file; **upload file vào S3** qua CloudFront (ingress); bảo mật bằng **OAC** |
| **MediaStore / MediaPackage** | Video on demand, live streaming (AWS Media Services) |
| **VPC Origin** | ALB/NLB/EC2 trong **private subnet** – không phải expose ra Internet |
| **Custom origin (HTTP)** | API Gateway (muốn kiểm soát nhiều hơn edge-optimized), **S3 static website**, bất kỳ HTTP backend |

### S3 làm origin

> 📖 **Định nghĩa:** Xem *OAC* ở bảng trên.

- User → edge (Internet công cộng) → origin S3 (mạng private AWS)
- **OAC + S3 bucket policy** chỉ cho CloudFront distribution truy cập bucket
- Bucket giữ **private**, không bật public access

### CloudFront vs S3 Cross-Region Replication

> 📖 **Định nghĩa:** **Cache (CloudFront)** giữ bản sao tạm thời theo TTL; **replication (CRR)** tạo bản sao đầy đủ, gần real-time ở region khác.

| **CloudFront** | **S3 CRR** |
|---|---|
| Mạng edge **toàn cầu** | Phải cấu hình **cho từng region** |
| File cache theo **TTL** (vd 1 ngày) | File cập nhật **gần real-time** |
| Tốt cho nội dung **tĩnh** cần có ở mọi nơi | **Read-only**, tốt cho nội dung **động** cần độ trễ thấp ở **vài region** |

### ALB/EC2 làm origin

> 📖 **Định nghĩa:** **CloudFront origin-facing prefix list** là AWS-managed prefix list chứa dải IP mà CloudFront dùng để gọi origin.

**Cách 1 – VPC Origins (khuyến nghị):** ALB/NLB/EC2 ở **private subnet**, không expose Internet.

**Cách 2 – origin public:**
- EC2 làm origin → EC2 **phải public**, SG cho phép **IP public của edge location**
- ALB làm origin → ALB **phải public** (SG cho phép IP edge), **EC2 phía sau có thể private** (SG cho phép SG của ALB)
- Danh sách IP: dùng **managed prefix list** `com.amazonaws.global.cloudfront.origin-facing`

### Chặn truy cập trực tiếp vào ALB / custom origin

> 📖 **Định nghĩa:** **Custom origin header** là header HTTP bí mật CloudFront thêm vào mọi request gửi tới origin.

1. CloudFront thêm **custom HTTP header** (vd `X-Custom-Header: <secret>`) khi gửi request tới ALB
2. ALB (listener rule hoặc **WAF**) chỉ chuyển request **có header đúng**
3. Giữ tên và giá trị header **bí mật** (xoay vòng bằng Secrets Manager + Lambda)
4. Có thể kết hợp: chỉ cho phép **IP của CloudFront** (prefix list)

### Origin Groups

> 📖 **Định nghĩa:** **Origin group** gồm một **primary** và một **secondary origin**; khi primary trả mã lỗi, CloudFront tự thử lại với secondary.

- Tăng **sẵn sàng cao** và **failover**
- Origin có thể **ở region khác nhau**
- Ví dụ: S3 (us-east-1) + S3 (eu-west-1) có **CRR** → **HA mức region** cho website tĩnh
- Failover dựa trên **mã lỗi** (vd 500, 502, 503, 504, 403, 404) — chỉ cho request **GET/HEAD/OPTIONS**

### Geo Restriction

> 📖 **Định nghĩa:** **Geo-IP database** là cơ sở dữ liệu ánh xạ IP → quốc gia.

- **Allow list**: chỉ cho user ở các quốc gia được duyệt
- **Block list**: chặn user ở các quốc gia bị cấm
- Quốc gia xác định bằng Geo-IP database của bên thứ 3
- Use case: **luật bản quyền** kiểm soát truy cập nội dung
- Header `CloudFront-Viewer-Country` dùng được trong Lambda@Edge / CloudFront Functions

### Pricing & Price Classes

> 📖 **Định nghĩa:** **Price class** là nhóm edge location mà distribution được phép dùng, để cân bằng chi phí và hiệu năng.

- Phí data out **khác nhau theo vị trí edge**
- Giảm chi phí bằng cách **giảm số edge location**:
  1. **Price Class All** – mọi region, **hiệu năng tốt nhất**
  2. **Price Class 200** – hầu hết region, **loại các region đắt nhất**
  3. **Price Class 100** – **chỉ các region rẻ nhất** (Bắc Mỹ, châu Âu)
- 🆕 Có thêm gói **flat-rate pricing** (2025) gộp CDN + WAF + DDoS theo tháng — kiểm tra lại

### Signed URL / Signed Cookies

> 📖 **Định nghĩa:** **Trusted signer / trusted key group** là khóa công khai đăng ký với CloudFront để xác minh chữ ký trên URL/cookie.

- Luồng: client xác thực với **application server** → server dùng SDK **sinh Signed URL** → client dùng URL truy cập CloudFront → CloudFront lấy object từ S3 (OAC)
- **Signed URL**: một file; **Signed Cookies**: nhiều file

### CloudFront Signed URL vs S3 Pre-Signed URL

> 📖 **Định nghĩa:** Xem *Signed URL* ở bảng trên và *Pre-signed URL* ở phần S3 Security.

| **CloudFront Signed URL** | **S3 Pre-Signed URL** |
|---|---|
| Cho phép truy cập một **path**, **bất kể origin** | Request thực hiện **với quyền người ký** |
| Khóa: **trusted key group** quản lý bằng IAM (cách cũ: key pair của **root**) | Dùng **IAM credential** của principal ký |
| Lọc theo **IP, path, ngày, thời hạn** | Thời hạn giới hạn |
| **Tận dụng cache** | Đi thẳng vào S3, không cache |

### Custom Error Pages

> 📖 **Định nghĩa:** **Error Caching Minimum TTL** là thời gian CloudFront cache **response lỗi** hoặc trang lỗi tùy chỉnh.

- Trả object (vd `.html`) cho viewer khi origin trả **4xx/5xx**
- Trang lỗi lưu ở S3, CloudFront cache theo **Error Caching Minimum TTL**

## 🎯 Mẹo thi

- "Nội dung tĩnh toàn cầu, bucket private" → **CloudFront + OAC**
- "ALB không được public" → **CloudFront VPC Origins**
- "Ngăn bỏ qua CloudFront gọi thẳng ALB" → **custom header + WAF/listener rule** hoặc prefix list
- "Website tĩnh HA khi region S3 lỗi" → **Origin group** + S3 CRR
- "Chặn quốc gia vì bản quyền" → **Geo restriction**
- "Giảm chi phí CloudFront, user chủ yếu Mỹ/EU" → **Price Class 100**
- "Nội dung premium cho user trả phí, nhiều file" → **Signed Cookies**; một file → **Signed URL**
- "Nội dung tĩnh ở mọi nơi" → CloudFront; "nội dung động độ trễ thấp ở vài region" → **S3 CRR**
