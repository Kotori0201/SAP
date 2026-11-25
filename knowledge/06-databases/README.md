# 06 · Databases

> Nguồn: Slide Stephane Maarek SAP v38 (trang 408–444) + section *Databases* trên Udemy, có bổ sung cập nhật AWS (🆕). Ghi chú viết lại để ôn tập.

**Domain SAP-C03:** Cloud-native architecture · Resilience, migration & business continuity · Cost-optimized architecture

## 📖 Định nghĩa tổng quan

- **Relational database (SQL):** dữ liệu theo bảng có schema cố định, quan hệ, JOIN, transaction (RDS, Aurora).
- **NoSQL database:** schema linh hoạt, scale ngang, truy cập theo key (DynamoDB).
- **Search engine:** đánh chỉ mục văn bản để tìm kiếm & phân tích (OpenSearch).
- **High availability:** DB tiếp tục phục vụ khi một AZ/instance lỗi (Multi-AZ, Aurora 6 bản sao).
- **Read scaling:** tăng năng lực đọc bằng replica hoặc cache.
- **Replication sync vs async:** đồng bộ = chờ bản sao xác nhận (không mất dữ liệu, chậm hơn); bất đồng bộ = không chờ (nhanh, có lag).

## Mục lục

| # | Chủ đề | File |
|---|---|---|
| 1 | DynamoDB & DAX | [01-dynamodb.md](01-dynamodb.md) |
| 2 | OpenSearch | [02-opensearch.md](02-opensearch.md) |
| 3 | RDS | [03-rds.md](03-rds.md) |
| 4 | Aurora (+ 🆕 DSQL, Limitless) | [04-aurora.md](04-aurora.md) |
| ⭐ | **Tổng kết, so sánh & tip nhớ nhanh** | [TONG-KET.md](TONG-KET.md) |

## Cheat-sheet: Tình huống → Giải pháp

| Tình huống trong đề | Giải pháp |
|---|---|
| NoSQL serverless, quy mô cực lớn | **DynamoDB** |
| Tải không đoán được | **DynamoDB on-demand** |
| Query theo thuộc tính khác, thêm sau | **GSI** |
| Tự xóa dữ liệu hết hạn | **DynamoDB TTL** |
| Phản ứng khi item thay đổi | **DynamoDB Streams + Lambda** |
| Change log > 24h | **Kinesis Data Streams for DynamoDB** |
| Ghi ở nhiều region (NoSQL) | **DynamoDB Global Tables** |
| Đọc micro giây, hot key, không sửa code | **DAX** |
| Cache kết quả tổng hợp | **ElastiCache** |
| Full-text search | **OpenSearch** |
| Tìm kiếm trên dữ liệu DynamoDB | **Streams → Lambda → OpenSearch** / zero-ETL 🆕 |
| CloudWatch Logs → OpenSearch ít vận hành | **Subscription filter → Firehose** |
| HA, failover tự động cho RDS | **Multi-AZ** |
| Scale đọc RDS | **Read Replicas** (+ Route 53 weighted) |
| Lambda gây TooManyConnections | **RDS Proxy** |
| Không lưu password DB | **IAM DB auth** / Secrets Manager |
| Oracle RAC | **Oracle trên EC2** |
| Backup Oracle RDS ra ngoài | **RMAN** |
| Migrate MySQL RDS ra on-prem | **mysqldump + replication** |
| Failover RDS sang region khác | Alarm → Lambda **promote replica** → cập nhật DNS |
| Replica riêng cho analytics trên Aurora | **Custom endpoint** |
| DB cho workload gián đoạn | **Aurora Serverless v2** |
| SQL qua HTTPS, không quản lý kết nối | **Aurora Data API** |
| DR đa region RPO ~1s, RTO < 1 phút | **Aurora Global Database** |
| Secondary region cần ghi | **Write forwarding** |
| RDS → Aurora ít downtime | **Aurora read replica → promote** |
| Tìm câu SQL gây chậm | **Performance Insights** |
| SQL active-active đa region | **Aurora DSQL** 🆕 |

## Bẫy hay gặp trong đề

- ❌ "DynamoDB item 1 MB" → tối đa **400 KB**.
- ❌ "Thêm LSI sau khi tạo bảng" → **chỉ lúc tạo**; thêm sau dùng **GSI**.
- ❌ "DynamoDB query tùy ý theo cột" → chỉ theo **PK + sort key** (trên bảng/index).
- ❌ "DynamoDB Streams giữ 7 ngày" → **24 giờ**.
- ❌ "Global Tables không cần Streams" → **phải bật Streams**.
- ❌ "Đọc từ standby Multi-AZ" (instance cổ điển) → **không**; dùng read replica.
- ❌ "Read replica replicate đồng bộ" → **bất đồng bộ**.
- ❌ "RDS for Oracle hỗ trợ RAC" → **không**.
- ❌ "RDS Proxy public" → **không bao giờ public**, Lambda phải trong VPC.
- ❌ "Aurora cross-region replica chọn từng bảng" → **sao chép toàn bộ DB**.
- ❌ "CloudTrail ghi query SQL" → **không**.
