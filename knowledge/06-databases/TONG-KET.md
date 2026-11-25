# ⭐ Tổng kết · Databases

[⬅ Về mục lục section](README.md)

## 1. Bảng tổng hợp kiến thức

| Dịch vụ | Là gì (1 dòng) | Con số phải nhớ | Chọn khi |
|---|---|---|---|
| **DynamoDB** | NoSQL key-value serverless | Item **400 KB** · Streams **24h** · PITR **35 ngày** · 1M rps | Key-value, quy mô lớn, serverless, độ trễ ms |
| **DAX** | Cache chuyên cho DynamoDB | **µs** · TTL **5 phút** · ≤ **10 node** · ≥ 3 node prod | Hot key, đọc nhiều, không sửa code |
| **OpenSearch** | Search & analytics engine | Managed hoặc Serverless | Full-text search, log analytics, vector store RAG |
| **RDS** | SQL managed (6 engine) | Backup **35 ngày** · IAM token **15 phút** | SQL truyền thống, lift-and-shift |
| **RDS Proxy** | Connection pool managed | Không public, Lambda phải trong VPC | Lambda/nhiều kết nối ngắn |
| **Aurora** | SQL cloud-native (MySQL/PG) | **6 bản sao / 3 AZ** · ghi **4/6**, đọc **3/6** · **15 replica** · failover **< 30s** · **256 TB** | SQL hiệu năng cao, HA |
| **Aurora Serverless v2** | Aurora tự scale ACU | Trả theo giây, có thể về 0 | Tải gián đoạn, khó đoán |
| **Aurora Global DB** | 1 primary + secondary region | **10** secondary · **16** replica/region · lag **< 1s** · RTO **< 1 phút** | DR đa region, đọc toàn cầu |
| **Aurora DSQL** 🆕 | SQL phân tán active-active | Ghi đa region | SQL ghi ở nhiều region |

## 2. So sánh quan trọng

### Chọn database nào?

| Yêu cầu | Chọn |
|---|---|
| SQL, JOIN, transaction, lift-and-shift | **RDS** |
| SQL, HA cao, nhiều replica, failover nhanh | **Aurora** |
| Key-value, scale vô hạn, serverless | **DynamoDB** |
| Tìm kiếm văn bản, log analytics | **OpenSearch** |
| Cache cho DynamoDB | **DAX** |
| Cache chung, session, leaderboard | **ElastiCache** |
| Data warehouse, OLAP | **Redshift** (học ở Data Engineering) |

### Multi-AZ vs Read Replica vs Aurora Replica

| | RDS Multi-AZ | RDS Read Replica | Aurora Replica |
|---|---|---|---|
| Mục đích | HA | Scale đọc, DR | HA **và** scale đọc |
| Replication | Sync | Async | Storage dùng chung (lag ~ms) |
| Đọc được | ❌ | ✅ | ✅ |
| Số lượng | 1 standby | Tối đa 15 | Tối đa 15 |
| Cross-region | ❌ | ✅ | Qua Global DB |
| Failover | Tự động | Promote thủ công | Tự động < 30s |

### LSI vs GSI

| | LSI | GSI |
|---|---|---|
| Partition key | Giữ nguyên | Đổi |
| Tạo khi | Chỉ lúc tạo bảng | Bất kỳ lúc nào |
| Capacity | Dùng chung bảng | Riêng |

### DynamoDB Streams vs Kinesis Data Streams for DynamoDB

| | DynamoDB Streams | Kinesis Data Streams |
|---|---|---|
| Retention | **24 giờ** | Tới **365 ngày** |
| Consumer | Lambda, KCL adapter | Nhiều consumer, Firehose, Flink |
| Dùng khi | Trigger đơn giản | Pipeline phân tích, giữ lâu |

### DAX vs ElastiCache

| DAX | ElastiCache |
|---|---|
| Cache item + Query/Scan của DynamoDB | Cache kết quả tổng hợp, mọi nguồn |
| Không sửa code | Sửa code |

## 3. Tip nhớ nhanh

- 🧠 **"4 ghi – 3 đọc – 6 bản – 3 AZ"** → quorum của Aurora.
- 🧠 **"15 – 15 – 30"** → Aurora: 15 replica, RDS 15 replica, failover dưới 30 giây.
- 🧠 **"10 – 16 – 1 – 1"** → Global DB: 10 region phụ, 16 replica/region, lag < 1s, RTO < 1 phút.
- 🧠 **"400 KB – 24 giờ – 35 ngày"** → DynamoDB: item, Streams, PITR.
- 🧠 **"LSI = Lúc tạo, GSI = Gắn sau"**.
- 🧠 **"DAX = DynamoDB Accelerator = µs, không sửa code"**; ElastiCache = phải sửa code.
- 🧠 **"Multi-AZ = Mạnh (HA), Replica = Rộng (đọc)"**; Multi-AZ **Sync**, Replica **Async**.
- 🧠 **"Oracle RAC → EC2"**; **"Oracle backup ra ngoài → RMAN"**; **"MySQL ra ngoài → mysqldump"**.
- 🧠 **"Lambda + RDS = Proxy"** (TooManyConnections).
- 🧠 **"Search = OpenSearch"** — thấy chữ *search/full-text/log analytics* là nghĩ ngay OpenSearch.
- 🧠 **"IAM DB auth = MMP"**: **M**ySQL, **M**ariaDB, **P**ostgreSQL.
- 🧠 **"TDE = Oracle + SQL Server"**.
