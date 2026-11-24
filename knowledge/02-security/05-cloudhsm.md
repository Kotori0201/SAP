# AWS CloudHSM

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **HSM (Hardware Security Module)** | Thiết bị phần cứng chuyên dụng, chống can thiệp vật lý (tamper-resistant), để sinh, lưu và sử dụng khóa mật mã an toàn. |
| **AWS CloudHSM** | Dịch vụ cung cấp **HSM chuyên dụng single-tenant** trong VPC của bạn; AWS quản lý phần cứng, **bạn quản lý hoàn toàn khóa và user**. |
| **CloudHSM cluster** | Nhóm nhiều HSM (trải nhiều AZ) đồng bộ khóa với nhau để đảm bảo sẵn sàng cao. |
| **Crypto User (CU)** | User trên HSM có quyền tạo/sử dụng khóa; khác **Crypto Officer (CO)** – quản trị user trên HSM. |
| **FIPS 140-2/140-3 Level 3** | Chuẩn bảo mật của Mỹ cho module mật mã; Level 3 yêu cầu chống can thiệp vật lý và xác thực theo danh tính. |

## Nội dung chính

### KMS vs CloudHSM – khác biệt cốt lõi

> 📖 **Định nghĩa:** **Single-tenant** là phần cứng dành riêng cho một khách hàng; **multi-tenant** là hạ tầng dùng chung nhiều khách hàng (có cô lập logic).

- **KMS** → AWS quản lý **phần mềm** mã hóa
- **CloudHSM** → AWS chỉ cấp **phần cứng**, bạn tự quản lý khóa (AWS **không** truy cập được khóa; mất credential = mất khóa)
- Hỗ trợ **symmetric & asymmetric** (cả SSL/TLS key)
- **Không có free tier**; phải dùng **CloudHSM Client Software** (hoặc thư viện PKCS#11, JCE, OpenSSL Dynamic Engine, KSP/CNG)
- **Redshift** hỗ trợ CloudHSM cho mã hóa database
- Hợp với **SSE-C** (bạn tự giữ khóa, gửi kèm request S3)

### Phân chia trách nhiệm

> 📖 **Định nghĩa:** **Shared responsibility** là mô hình chia trách nhiệm bảo mật giữa AWS và khách hàng.

| AWS (qua IAM) | Bạn (qua CloudHSM software) |
|---|---|
| Tạo/xóa/sửa **cluster**, thêm HSM | Quản lý **khóa** |
| Vận hành phần cứng, vá firmware | Quản lý **user** trên HSM (CO, CU) |

### Sẵn sàng cao

> 📖 **Định nghĩa:** **High availability (HA)** là khả năng hệ thống tiếp tục hoạt động khi một thành phần hỏng.

- Cluster **trải nhiều AZ**; khóa tự đồng bộ giữa các HSM
- Khuyến nghị **≥ 2 HSM ở 2 AZ** cho production (bắt buộc khi dùng làm KMS Custom Key Store)

### So sánh KMS vs CloudHSM

> 📖 **Định nghĩa:** **Cryptographic acceleration** là việc dùng phần cứng chuyên dụng để tăng tốc các phép toán mật mã (vd TLS handshake).

| Tiêu chí | **AWS KMS** | **AWS CloudHSM** |
|---|---|---|
| Tenancy | Multi-tenant | **Single-tenant** |
| Chuẩn | FIPS 140-3 Level 3 | FIPS 140-3 Level 3 (tùy loại HSM) |
| Loại khóa | AWS owned / AWS managed / Customer managed | Customer managed |
| Thuật toán | Symmetric, asymmetric, ký số, HMAC | Symmetric, asymmetric, ký số **và hashing** |
| Phạm vi | Theo region (có multi-Region key) | Trong **VPC**; truy cập từ VPC khác qua peering |
| Tăng tốc mật mã | Không | **SSL/TLS acceleration**, **Oracle TDE acceleration** |
| Xác thực & phân quyền | **IAM** | **User trên HSM do bạn quản lý** |
| HA | AWS lo | **Bạn thêm HSM ở nhiều AZ** |
| Audit | CloudTrail, CloudWatch | CloudTrail, CloudWatch, **hỗ trợ MFA** (quorum auth) |
| Free tier | Có | Không |
| Tích hợp AWS service | Rất rộng | Hạn chế (qua KMS Custom Key Store để tích hợp) |

## 🎯 Mẹo thi

- "Single-tenant", "tự quản lý khóa hoàn toàn, AWS không truy cập được", "FIPS 140 Level 3 với khóa riêng" → **CloudHSM**
- "Tăng tốc SSL/TLS" hoặc "Oracle TDE" → **CloudHSM**
- "Muốn dùng KMS với các AWS service nhưng khóa nằm trong HSM riêng" → **KMS Custom Key Store (CloudHSM)**
- "Ít vận hành nhất, tích hợp sẵn dịch vụ AWS" → **KMS**
