# RDS Security

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Encryption at rest (RDS)** | Mã hóa volume EBS, snapshot, backup, read replica của DB bằng KMS key. |
| **TDE (Transparent Data Encryption)** | Tính năng mã hóa của chính engine DB (**Oracle, SQL Server**), mã hóa dữ liệu trong file DB một cách trong suốt với ứng dụng. |
| **IAM database authentication** | Đăng nhập DB bằng **token xác thực ngắn hạn** (15 phút) sinh từ IAM thay vì mật khẩu. |
| **Encryption in transit** | Mã hóa kết nối client ↔ DB bằng SSL/TLS. |

## Nội dung chính

### Mã hóa

> 📖 **Định nghĩa:** Xem bảng trên.

- **KMS** mã hóa at rest cho EBS volume & snapshot — **phải bật khi tạo DB**
- **TDE** cho **Oracle và SQL Server** (có thể kết hợp CloudHSM với Oracle TDE)
- **SSL/TLS** in-flight cho **mọi engine**; ép bắt buộc: PostgreSQL `rds.force_ssl=1`, MySQL `REQUIRE SSL` cho user
- **Mã hóa DB đang chạy chưa mã hóa:** snapshot → **copy snapshot với mã hóa** → restore thành DB mới → chuyển ứng dụng

### Xác thực

> 📖 **Định nghĩa:** Xem bảng trên.

- **IAM authentication** cho **MySQL, PostgreSQL, MariaDB** (và Aurora tương ứng)
- **Authorization vẫn nằm trong RDS** (GRANT trong DB), không trong IAM
- Kerberos/AD qua **AWS Managed Microsoft AD** cho SQL Server, Oracle, MySQL, PostgreSQL
- Mật khẩu master nên quản lý bằng **Secrets Manager** (xoay vòng tự động)

### Audit

> 📖 **Định nghĩa:** **Database Activity Streams** là luồng gần thời gian thực các hoạt động trong DB (Aurora, RDS Oracle/SQL Server) đẩy vào Kinesis, phục vụ audit.

- **CloudTrail không ghi các câu query trong RDS** (chỉ ghi API quản lý như `CreateDBInstance`)
- Audit query: bật log của engine (vd pgaudit, MySQL audit plugin) → CloudWatch Logs, hoặc **Database Activity Streams**

## 🎯 Mẹo thi

- "Mã hóa RDS đang chạy" → snapshot → copy có mã hóa → restore
- "Không muốn lưu password DB trong code" → **IAM DB auth** hoặc **Secrets Manager**
- "Audit các câu lệnh SQL" → **không phải CloudTrail** → Database Activity Streams / audit log
