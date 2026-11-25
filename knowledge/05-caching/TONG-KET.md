# ⭐ Tổng kết · Caching

[⬅ Về mục lục section](README.md)

## 1. Bảng tổng hợp kiến thức

| Dịch vụ | Là gì (1 dòng) | Con số phải nhớ | Chọn khi |
|---|---|---|---|
| **CloudFront** | CDN toàn cầu, cache tại edge | **225+ PoP** · Price Class **All / 200 / 100** | Nội dung tĩnh/động toàn cầu, bảo vệ origin |
| **OAC** | Chỉ CloudFront truy cập S3 private | Thay thế OAI | S3 làm origin |
| **VPC Origins** | Origin ALB/NLB/EC2 ở private subnet | – | Không muốn expose ALB |
| **Origin Group** | Primary + secondary origin | Failover theo mã lỗi | HA mức region cho website |
| **Signed URL / Cookie** | Kiểm soát truy cập nội dung CloudFront | Trusted key group | Nội dung trả phí |
| **CloudFront Functions** | JS siêu nhẹ tại edge location | **< 1 ms**, **2 MB**, **10 KB** code | Header, rewrite, redirect, JWT |
| **Lambda@Edge** | Lambda tại Regional Edge Cache | **5s / 30s** · **128 MB / 10 GB** · viết ở **us-east-1** | Gọi mạng, đọc body, đổi origin |
| **ElastiCache Redis/Valkey** | Cache in-memory HA | Multi-AZ, replica, AOF | Cache bền, session, leaderboard |
| **ElastiCache Memcached** | Cache đa luồng, sharding | Không bền vững | Cache đơn giản |

## 2. So sánh quan trọng

### CloudFront Functions vs Lambda@Edge

| | CloudFront Functions | Lambda@Edge |
|---|---|---|
| Ngôn ngữ | JavaScript | Node.js, Python |
| Vị trí | Edge Location | Regional Edge Cache |
| Event | Viewer req/resp | Viewer + **Origin** req/resp |
| Thời gian | < 1 ms | 5s (viewer) / 30s (origin) |
| Mạng / body | ❌ | ✅ |
| Giá | ~1/6 | Đắt hơn |

### CloudFront Signed URL vs S3 Pre-signed URL

| CloudFront Signed URL | S3 Pre-signed URL |
|---|---|
| Truy cập path, mọi origin | Request với quyền người ký |
| Trusted key group | IAM credential |
| Lọc IP, path, ngày | Chỉ thời hạn |
| Có cache | Không cache |

### CloudFront vs S3 CRR

| CloudFront | S3 CRR |
|---|---|
| Toàn cầu, cache theo TTL | Từng region, gần real-time |
| Nội dung tĩnh | Nội dung động, ít region |

### Redis vs Memcached

| Redis/Valkey | Memcached |
|---|---|
| Multi-AZ, failover, replica | Sharding nhiều node |
| Bền vững (AOF, backup) | Không bền vững |
| Cấu trúc dữ liệu phong phú | Đa luồng |

### Ba lớp cache

| Lớp | Dịch vụ | Độ trễ |
|---|---|---|
| Edge | CloudFront | Gần user nhất |
| Ứng dụng | ElastiCache | ms |
| Database | DAX | µs |

## 3. Tip nhớ nhanh

- 🧠 **"Functions = Fast & Front"**: chỉ viewer (phía trước), < 1 ms. **"Lambda@Edge = Lớn & Lâu"**: origin, gọi mạng, đọc body.
- 🧠 **"Không trộn cùng viewer event"** – CF Functions và Lambda@Edge không cùng gắn một viewer event.
- 🧠 **"Lambda@Edge sinh ra ở Virginia"** (us-east-1), cert CloudFront cũng ở us-east-1.
- 🧠 **"Price Class 100 = rẻ nhất = Mỹ + Âu"**.
- 🧠 **"1 file → Signed URL, nhiều file → Signed Cookie"**.
- 🧠 **"Chặn đi đường tắt = Header bí mật"** (custom header + WAF trên ALB).
- 🧠 **"Redis = Robust (HA, bền), Memcached = Multi-thread (đơn giản)"**.
- 🧠 **"Cache gần user nhất trước"**: CloudFront → ElastiCache/DAX → DB.
- 🧠 **"Đỉnh tải → Queue"**: SQS buffer trước DB.
