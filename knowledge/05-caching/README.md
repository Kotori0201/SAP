# 05 · Caching

> Nguồn: Slide Stephane Maarek SAP v38 (trang 375–407) + section *Caching* trên Udemy, có bổ sung cập nhật AWS (🆕). Ghi chú viết lại để ôn tập.

**Domain SAP-C03:** Cloud-native architecture · Cost-optimized architecture · Resilience

## 📖 Định nghĩa tổng quan

- **Caching:** lưu tạm bản sao dữ liệu ở nơi truy cập nhanh hơn để giảm độ trễ và tải cho nguồn gốc.
- **Edge caching:** cache tại các điểm gần người dùng trên toàn cầu (CloudFront).
- **In-memory caching:** cache trong RAM cạnh ứng dụng/DB (ElastiCache, DAX).
- **Edge computing:** chạy logic tại edge để biến đổi request/response (CloudFront Functions, Lambda@Edge).
- **TTL:** thời gian dữ liệu được giữ trong cache trước khi làm mới.
- **Cache hit ratio:** tỷ lệ request được phục vụ từ cache — càng cao càng tốt.

## Mục lục

| # | Chủ đề | File |
|---|---|---|
| 1 | Amazon CloudFront | [01-cloudfront.md](01-cloudfront.md) |
| 2 | CloudFront Functions & Lambda@Edge | [02-cloudfront-functions-va-lambda-edge.md](02-cloudfront-functions-va-lambda-edge.md) |
| 3 | Amazon ElastiCache | [03-elasticache.md](03-elasticache.md) |
| 4 | Xử lý tải cực lớn | [04-xu-ly-tai-cuc-lon.md](04-xu-ly-tai-cuc-lon.md) |

## Cheat-sheet: Tình huống → Giải pháp

| Tình huống trong đề | Giải pháp |
|---|---|
| Nội dung tĩnh toàn cầu, bucket private | **CloudFront + OAC** |
| ALB/EC2 không được expose Internet | **CloudFront VPC Origins** |
| Ngăn gọi thẳng ALB bỏ qua CloudFront | **Custom header** + WAF/listener rule, prefix list CloudFront |
| Website tĩnh HA khi region S3 lỗi | **Origin group** + S3 CRR |
| Chặn quốc gia vì bản quyền | **Geo restriction** |
| Giảm chi phí CDN, user ở Mỹ/EU | **Price Class 100** |
| Nội dung premium, một file | **CloudFront Signed URL** |
| Nội dung premium, nhiều file | **CloudFront Signed Cookies** |
| Truy cập tạm vào object S3 với quyền người ký | **S3 pre-signed URL** |
| Trang lỗi thân thiện khi origin lỗi | **Custom error pages** |
| Nội dung động độ trễ thấp ở vài region | **S3 CRR** (không phải CloudFront) |
| Rewrite URL, header, redirect, quy mô lớn | **CloudFront Functions** |
| Kiểm tra JWT tại edge | **CloudFront Functions** |
| Gọi dịch vụ ngoài / đọc body / AWS SDK tại edge | **Lambda@Edge** |
| Đổi origin theo vị trí user | **Lambda@Edge origin request** |
| Ảnh theo loại thiết bị | **CloudFront Functions** (User-Agent) |
| Giảm tải đọc cho RDS | **ElastiCache** |
| Session dùng chung cho mọi instance | **ElastiCache** (hoặc DynamoDB) |
| Cache cần HA, bền vững, backup | **Redis/Valkey** |
| Cache đơn giản, đa luồng | **Memcached** |
| Leaderboard real-time | **Redis Sorted Sets** |
| Đỉnh tải cực lớn | **CloudFront** + **SQS** buffer + DynamoDB on-demand |
| SSE-KMS bị throttle | **Quota KMS** → S3 Bucket Key |

## Bẫy hay gặp trong đề

- ❌ "CloudFront Functions đổi origin request" → chỉ **viewer** events.
- ❌ "Dùng CloudFront Functions và Lambda@Edge trên cùng viewer event" → **không được**.
- ❌ "Lambda@Edge deploy ở mọi region" → viết ở **us-east-1**, CloudFront tự replicate.
- ❌ "CloudFront Functions gọi API ngoài" → **không có network access**.
- ❌ "Signed URL CloudFront dùng IAM credential như S3" → dùng **trusted key group** (key pair).
- ❌ "Memcached có Multi-AZ failover/replica" → đó là **Redis**.
- ❌ "ElastiCache không cần sửa code" → **cần sửa nhiều**; muốn ít sửa với DynamoDB → **DAX**.
- ❌ "CloudFront thay thế được S3 CRR cho dữ liệu cập nhật liên tục" → CloudFront cache theo TTL.
