# Amazon Aurora

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Amazon Aurora** | Database quan hệ **cloud-native** do AWS xây, **tương thích MySQL và PostgreSQL**, tách compute khỏi **storage dùng chung, tự mở rộng, 6 bản sao trên 3 AZ**. |
| **Aurora DB cluster** | Gồm **1 writer (primary)** + tối đa **15 Aurora Replica** dùng chung một **cluster volume**. |
| **Endpoint** | Địa chỉ host + port để kết nối; Aurora có **cluster (writer), reader, custom, instance** endpoint. |
| **Aurora Serverless** | Chế độ Aurora **tự scale capacity** (ACU) theo tải thực tế, trả theo giây. |
| **Data API** | Endpoint **HTTPS** để chạy SQL trên Aurora mà **không cần kết nối JDBC/driver**. |
| **Aurora Global Database** | Một **primary region** (đọc/ghi) + tối đa **10 secondary region** chỉ đọc, replication **< 1 giây**. |
| **Write forwarding** | Secondary cluster **chuyển câu lệnh ghi** về primary cluster thay vì từ chối. |
| **Performance Insights** | Công cụ phân tích hiệu năng DB theo **wait, SQL statement, host, user**. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** **Cross-region read replica** là bản sao toàn bộ DB ở region khác.

- Engine: **PostgreSQL-compatible** & **MySQL-compatible**
- Storage **tự tăng tới 256 TB** (con số tăng theo thời gian), **6 bản sao**, multi-AZ
- **Tối đa 15 read replica**, một **reader endpoint** truy cập tất cả
- **Cross-region RR**: **sao chép toàn bộ DB** (không chọn từng bảng)
- **Load/offload dữ liệu trực tiếp từ/tới S3**
- Backup, snapshot, restore giống RDS
- 🆕 **Aurora I/O-Optimized** (không tính phí I/O, cho workload I/O nặng), **zero-ETL tới Redshift**

### High Availability & Read Scaling

> 📖 **Định nghĩa:** **Quorum** là số bản sao tối thiểu phải xác nhận để thao tác thành công.

- **6 bản sao trên 3 AZ**:
  - **4/6** cần cho **ghi**
  - **3/6** cần cho **đọc**
- **Tự chữa lành** bằng replication peer-to-peer
- Storage **striped** trên hàng trăm volume
- **Failover master < 30 giây**
- Master + tối đa **15 replica** phục vụ đọc; hỗ trợ cross-region replication

### Endpoints

> 📖 **Định nghĩa:** Xem *Endpoint* ở bảng trên.

| Endpoint | Trỏ tới | Dùng cho |
|---|---|---|
| **Cluster (Writer)** | **Primary hiện tại** | Mọi thao tác **ghi** (và đọc) |
| **Reader** | **Load balance** kết nối tới mọi replica | Chỉ **đọc** |
| **Custom** | **Tập instance bạn chọn** | Nhóm instance có cấu hình/capacity khác (vd replica lớn cho **analytics**) |
| **Instance** | Một instance cụ thể | Chẩn đoán, tinh chỉnh một instance |

- Ví dụ custom endpoint: 2 replica `db.r5.2xlarge` cho **truy vấn phân tích**, replica nhỏ cho truy vấn thường → sau khi có custom endpoint thường **không dùng reader endpoint** nữa
- Reader có thể **auto scaling**

### Logs & Troubleshooting

> 📖 **Định nghĩa:** **Enhanced Monitoring** là metric **mức hệ điều hành** của host DB (theo process, mỗi giây); **slow query log** ghi các câu truy vấn chạy lâu.

- Log Aurora MySQL: **error log, slow query log, general log, audit log** → download hoặc **publish lên CloudWatch Logs**
- Troubleshoot hiệu năng RDS & Aurora:
  - **Performance Insights** – theo **wait, SQL, host, user**
  - **CloudWatch Metrics** – CPU, Memory, Swap
  - **Enhanced Monitoring** – mức host, process, **từng giây**
  - **Slow query logs**

### Aurora Serverless

> 📖 **Định nghĩa:** **ACU (Aurora Capacity Unit)** là đơn vị capacity (~2 GB RAM + CPU, mạng tương ứng).

- Tự khởi tạo & **auto scaling theo tải thực**
- Tốt cho workload **không thường xuyên, gián đoạn, khó đoán**
- **Không cần capacity planning**, **trả theo giây**
- Kiến trúc: client → **proxy fleet** (Aurora quản lý) → instance → storage dùng chung
- 🆕 **Serverless v2** scale tức thì theo bước nhỏ, hỗ trợ read replica, Global Database, và **scale về 0 ACU** (tự pause)

### Data API

> 📖 **Định nghĩa:** Xem bảng trên.

- Truy cập Aurora qua **API endpoint đơn giản** (không JDBC)
- Endpoint **HTTPS bảo mật** chạy SQL; **không quản lý kết nối lâu dài**
- **Không cần DB credential trong app**: user cần quyền **Data API** và **Secrets Manager** (nơi kiểm tra credential)
- 🆕 Đã hỗ trợ **Aurora Serverless v2 và provisioned** (không chỉ Serverless v1)

### RDS Proxy cho Aurora

> 📖 **Định nghĩa:** **Read-only proxy endpoint** là endpoint của RDS Proxy chỉ kết nối tới các read replica.

- App/Lambda → **RDS Proxy** → Aurora primary (đọc & ghi)
- Tạo thêm **read-only endpoint** trên proxy → chỉ kết nối **Aurora Read Replicas**

### Global Aurora

> 📖 **Định nghĩa:** Xem *Aurora Global Database* ở bảng trên.

| **Cross-Region Read Replicas** | **Aurora Global Database** (khuyến nghị) |
|---|---|
| Hữu ích cho DR, **dễ triển khai** | **1 primary region** (đọc/ghi) |
| | Tối đa **10 secondary region** (chỉ đọc), **lag < 1 giây** |
| | Tối đa **16 read replica / secondary region** |
| | **Giảm độ trễ** cho user toàn cầu |
| | **Promote region khác (DR) RTO < 1 phút** |
| | Quản lý được **RPO** (Aurora PostgreSQL) |

> 💡 **Switchover** (có kế hoạch, không mất dữ liệu) vs **failover** (sự cố, có thể mất dữ liệu trong khoảng lag).

### Write Forwarding

> 📖 **Định nghĩa:** Xem bảng trên.

- Secondary cluster **chuyển SQL ghi** tới **primary cluster**
- Dữ liệu luôn được **đổi ở primary trước**, rồi replicate về secondary
- Primary luôn có bản **mới nhất**
- **Giảm số endpoint** ứng dụng phải quản lý

### Chuyển RDS sang Aurora

> 📖 **Định nghĩa:** **Aurora read replica of RDS** là bản sao Aurora replicate từ một RDS MySQL/PostgreSQL instance.

| Cách | Downtime |
|---|---|
| **Snapshot RDS → restore thành Aurora** | Có (dừng ghi trong lúc snapshot & restore) |
| **Tạo Aurora Read Replica từ RDS** → đợi lag = 0 → **promote** | **Tối thiểu** |

### 🆕 Aurora DSQL & Limitless (bổ sung ngoài slide)

> 📖 **Định nghĩa:** **Aurora DSQL** là database SQL **phân tán, serverless**, tương thích PostgreSQL, **active-active đa region** với strong consistency. **Aurora Limitless Database** là Aurora PostgreSQL **tự sharding** để scale ghi vượt giới hạn một instance.

- DSQL: ghi được ở **nhiều region cùng lúc**, sẵn sàng 99,999% đa region — chủ đề có thể xuất hiện ở C03
- Limitless: scale **hàng triệu ghi/giây**, petabyte
- ⚠️ Đối chiếu tài liệu mới nhất (ra mắt 2024–2025)

## 🎯 Mẹo thi

- "Failover < 30s, 15 replica, storage tự tăng" → **Aurora**
- "Một số replica lớn riêng cho báo cáo" → **Custom endpoint**
- "Workload gián đoạn, khó đoán, dev/test" → **Aurora Serverless (v2)**
- "Lambda gọi SQL không quản lý kết nối/credential" → **Data API** (hoặc RDS Proxy)
- "DR đa region, RPO ~1s, RTO < 1 phút" → **Aurora Global Database**
- "Secondary region cần ghi mà không đổi endpoint" → **Write forwarding**
- "Chuyển RDS MySQL sang Aurora ít downtime" → **Aurora read replica rồi promote**
- "Tìm câu SQL gây chậm" → **Performance Insights**
- "Ghi active-active nhiều region, SQL" → **Aurora DSQL** 🆕 (hoặc DynamoDB Global Tables nếu NoSQL)
