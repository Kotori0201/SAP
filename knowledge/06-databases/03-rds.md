# Amazon RDS

[⬅ Về mục lục section](README.md)

> Phần mã hóa và audit RDS chi tiết: [02-security/07-rds-security.md](../02-security/07-rds-security.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Amazon RDS (Relational Database Service)** | Dịch vụ **database quan hệ managed**: AWS lo provisioning, backup, vá lỗi, giám sát. |
| **Multi-AZ** | Mô hình có **standby instance** ở AZ khác, **replicate đồng bộ (sync)**, tự **failover** qua cùng một DNS name. |
| **Read Replica** | Bản sao **chỉ đọc**, **replicate bất đồng bộ (async)**, dùng để **scale đọc**; có thể ở region khác. |
| **Eventual consistency** | Dữ liệu trên replica **có thể trễ** so với primary một khoảng ngắn. |
| **Automated backup / PITR** | Backup tự động hằng ngày + transaction log, khôi phục về **bất kỳ thời điểm nào** trong khoảng retention (tối đa 35 ngày). |
| **Snapshot (RDS)** | Bản sao lưu **thủ công**, giữ đến khi bạn xóa, **copy được sang region khác**. |
| **RDS Proxy** | Proxy managed **gom và tái sử dụng kết nối** DB (connection pooling), tăng sức chịu lỗi, hỗ trợ IAM auth. |
| **IAM DB authentication** | Đăng nhập DB bằng **token 15 phút** sinh từ IAM thay cho mật khẩu. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** **DB engine** là phần mềm database (MySQL, PostgreSQL…) chạy trên RDS.

- Engine: **PostgreSQL, MySQL, MariaDB, IBM Db2, Oracle, SQL Server**
- Managed: provisioning, backup, patching, monitoring
- Chạy trong **VPC**, thường **private subnet**, kiểm soát mạng bằng **Security Group** (quan trọng khi dùng Lambda)
- Lưu trữ trên **EBS**, **storage auto-scaling**
- **Backup tự động + PITR** (backup hết hạn theo retention)
- **Snapshot thủ công**, copy **cross-region**
- **RDS Events** → thông báo qua **SNS** (thao tác, sự cố…)
- 🆕 **RDS Blue/Green Deployments**: tạo môi trường staging đồng bộ, chuyển đổi nâng cấp version với downtime tối thiểu

### Multi-AZ vs Read Replicas

> 📖 **Định nghĩa:** Xem bảng trên.

| | **Multi-AZ** | **Read Replicas** |
|---|---|---|
| Mục đích | **Sẵn sàng cao** (failover) | **Scale đọc** |
| Replication | **Đồng bộ (SYNC)** | **Bất đồng bộ (ASYNC)** → eventual consistency |
| Đọc từ standby | ❌ (instance Multi-AZ cổ điển) | ✅ |
| Endpoint | **Một DNS name**, failover tự động | Mỗi replica **endpoint riêng** |
| Cross-region | ❌ | ✅ |
| Promote | Tự động | **Thủ công** thành DB độc lập |

> 💡 **Multi-AZ DB cluster** (1 writer + **2 reader đọc được**) là kiểu mới cho MySQL/PostgreSQL.

### Phân phối đọc qua các replica

> 📖 **Định nghĩa:** **Weighted record set** là tập record Route 53 chia traffic theo trọng số.

- Route 53 **weighted record** trỏ tới endpoint từng replica (vd mỗi cái 25%) + **health check**
- (Aurora có sẵn **reader endpoint** – không cần cách này)

### Bảo mật (nhắc lại)

> 📖 **Định nghĩa:** **TDE** là mã hóa trong suốt của engine (Oracle, SQL Server).

- **KMS** mã hóa EBS/snapshot at rest; **TDE** cho Oracle & SQL Server; **SSL** in-flight cho mọi engine
- **IAM authentication** cho MySQL, PostgreSQL, MariaDB — **authorization vẫn trong RDS**
- **Copy snapshot chưa mã hóa thành snapshot mã hóa** (cách mã hóa DB cũ)
- ⚠️ **CloudTrail không theo dõi query** trong RDS

### IAM Authentication

> 📖 **Định nghĩa:** Xem bảng trên.

- Dùng cho **MariaDB, MySQL, PostgreSQL**
- **Không cần mật khẩu**, chỉ cần **auth token** (gọi IAM & RDS API) – hạn **15 phút**
- Lợi ích: kết nối **bắt buộc SSL**; **quản lý user tập trung bằng IAM**; dùng **IAM Role / EC2 instance profile** dễ dàng

### RDS for Oracle – Exam tips

> 📖 **Định nghĩa:** **RMAN** là *Oracle Recovery Manager* – công cụ backup/restore native của Oracle; **RAC** là *Real Application Clusters* – cụm nhiều node Oracle active-active.

- **Backup:** dùng **RDS Backups** để backup & restore **vào RDS for Oracle**; dùng **Oracle RMAN** để backup & restore **ra ngoài RDS** (restore RMAN *vào* RDS **không hỗ trợ**)
- **RAC:** RDS for Oracle **KHÔNG hỗ trợ RAC** → muốn RAC thì chạy **Oracle trên EC2** (toàn quyền kiểm soát)
- RDS for Oracle hỗ trợ **TDE**
- **DMS** hoạt động với Oracle RDS (migrate on-prem → RDS)

### RDS for MySQL

> 📖 **Định nghĩa:** **mysqldump** là công cụ native của MySQL để export/import dữ liệu dạng SQL.

- Dùng **mysqldump** để migrate MySQL RDS → **non-RDS** (on-prem hoặc EC2)
- Quy trình: export bằng mysqldump → import vào đích → **bật replication** → **dừng replication** khi hoàn tất

### RDS Proxy cho Lambda

> 📖 **Định nghĩa:** **Connection pooling** là tái sử dụng một tập kết nối DB cho nhiều client thay vì mở kết nối mới mỗi lần.

- Lambda mở & giữ kết nối DB → dễ lỗi **"TooManyConnections"**
- **RDS Proxy**: không cần code dọn kết nối idle và quản lý pool
- Hỗ trợ **IAM auth** hoặc DB auth, **auto-scaling**
- ⚠️ Lambda **phải ở trong VPC** – RDS Proxy **không bao giờ public**
- Giảm thời gian failover (tới ~66%)

### Kiến trúc: failover cross-region

> 📖 **Định nghĩa:** **Promote read replica** là biến replica thành DB độc lập, nhận cả ghi.

- Option 1: Route 53 health check gọi **HTTP `/health-db`** của app
- Option 2: **CloudWatch Alarm** trên RDS → gắn vào health check
- Khi lỗi: alarm → **EventBridge/SNS** → Lambda **promote read replica** (us-west-2) → **cập nhật DNS**

## 🎯 Mẹo thi

- "HA, failover tự động" → **Multi-AZ**; "scale đọc / DR region khác" → **Read Replica**
- "Lambda gây TooManyConnections" → **RDS Proxy** (Lambda trong VPC)
- "Không lưu password DB" → **IAM DB auth** (MySQL/PostgreSQL/MariaDB) hoặc **Secrets Manager**
- "Oracle RAC" → **EC2**, không phải RDS
- "Backup Oracle RDS ra ngoài" → **RMAN**; "migrate MySQL RDS ra on-prem" → **mysqldump + replication**
- "Nâng cấp version DB downtime thấp" → **RDS Blue/Green Deployments** 🆕
- "Mã hóa RDS chưa mã hóa" → snapshot → **copy có mã hóa** → restore
