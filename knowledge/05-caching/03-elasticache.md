# Amazon ElastiCache

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Cache** | Kho lưu dữ liệu **trong bộ nhớ (in-memory)**, hiệu năng rất cao, độ trễ thấp, dùng để giảm tải cho database. |
| **Amazon ElastiCache** | Dịch vụ cache managed cho **Redis OSS, Valkey và Memcached** (giống RDS nhưng cho cache). |
| **Cache hit / cache miss** | Hit: dữ liệu có sẵn trong cache. Miss: không có, phải đọc từ DB. |
| **Lazy loading (cache-aside)** | Chiến lược: đọc cache trước, miss thì đọc DB rồi **ghi vào cache**. |
| **Write-through** | Chiến lược: mỗi lần ghi DB thì **ghi luôn vào cache**. |
| **Cache invalidation** | Cơ chế làm mới/xóa dữ liệu cũ trong cache (TTL, xóa key khi dữ liệu thay đổi). |
| **Sharding** | Chia dữ liệu ra nhiều node (partition) để scale ghi và dung lượng. |
| **AOF (Append Only File)** | Cơ chế lưu bền của Redis: ghi mọi thao tác ghi vào file để khôi phục. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** **Stateless application** là ứng dụng không giữ trạng thái phiên trên server.

- **RDS cho DB quan hệ** ⇔ **ElastiCache cho Redis/Memcached**
- In-memory DB hiệu năng cao, **độ trễ thấp**
- **Giảm tải DB** cho workload đọc nhiều
- Giúp ứng dụng **stateless** (lưu session ngoài)
- AWS lo: bảo trì OS, vá lỗi, tối ưu, cài đặt, giám sát, khôi phục lỗi, backup
- ⚠️ Dùng ElastiCache **đòi hỏi sửa code ứng dụng nhiều**
- 🆕 **ElastiCache Serverless** (không quản lý node) và engine **Valkey** (fork mã nguồn mở của Redis, rẻ hơn)

### Kiến trúc: DB cache

> 📖 **Định nghĩa:** Xem *Lazy loading* và *Cache invalidation* ở bảng trên.

- App query ElastiCache → nếu **miss**, đọc RDS rồi **ghi vào cache**
- Giảm tải RDS
- **Bắt buộc có chiến lược invalidation** để chỉ dùng dữ liệu mới nhất

### Kiến trúc: User Session Store

> 📖 **Định nghĩa:** **Session store** là nơi lưu dữ liệu phiên đăng nhập dùng chung cho mọi instance ứng dụng.

- User đăng nhập vào **bất kỳ instance** → app ghi session vào ElastiCache
- User sang **instance khác** → đọc session từ ElastiCache → **vẫn đăng nhập**
- Thay thế sticky session của load balancer

### Redis vs Memcached

> 📖 **Định nghĩa:** **Replication** sao chép dữ liệu sang replica để HA/scale đọc; **multi-threaded** là xử lý song song nhiều luồng trên một node.

| **Redis / Valkey** | **Memcached** |
|---|---|
| **Multi-AZ + auto-failover** | **Nhiều node để sharding** dữ liệu |
| **Read replica** scale đọc, HA | **Không bền vững** (non-persistent) |
| **Bền vững**: AOF, backup & restore | Backup & restore (chỉ bản **Serverless**) |
| Cấu trúc dữ liệu phong phú (sorted set, pub/sub, geospatial…) | **Multi-threaded** |
| **Replication + sharding** (cluster mode) | Chỉ sharding |

> 💡 Redis Sorted Sets → **leaderboard** real-time.

## 🎯 Mẹo thi

- "Giảm tải đọc cho RDS" → **ElastiCache** (hoặc RDS read replica nếu không muốn sửa code)
- "Lưu session để ứng dụng stateless" → **ElastiCache** (hoặc DynamoDB)
- "Cache cần HA, failover, bền vững, backup" → **Redis/Valkey**
- "Cache đơn giản, đa luồng, không cần bền vững" → **Memcached**
- "Leaderboard game" → **Redis Sorted Sets**
- "Cache cho DynamoDB không sửa code nhiều" → **DAX** (xem phần Database)
