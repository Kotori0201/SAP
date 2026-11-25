# AWS Data Exchange & AWS Transfer Family

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **AWS Data Exchange** | Chợ dữ liệu: **tìm, đăng ký (subscribe) và sử dụng dữ liệu của bên thứ 3** trên cloud (file, bảng Redshift, API). |
| **Data product** | Gói dữ liệu do nhà cung cấp công bố trên Data Exchange, khách hàng subscribe theo điều khoản. |
| **AWS Transfer Family** | Dịch vụ fully managed để **truyền file vào/ra S3 hoặc EFS** qua giao thức **SFTP, FTPS, FTP** (và AS2). |
| **SFTP / FTPS / FTP** | SFTP: truyền file qua SSH; FTPS: FTP qua SSL/TLS; FTP: giao thức truyền file không mã hóa. |

## Nội dung chính

### AWS Data Exchange

> 📖 **Định nghĩa:** Xem bảng trên.

- Ví dụ nhà cung cấp: **Reuters** (tin tức), **Change Healthcare** (giao dịch y tế ẩn danh), **Dun & Bradstreet** (hồ sơ doanh nghiệp), **Foursquare** (dữ liệu vị trí)
- Sau khi subscribe, dùng **Data Exchange API** tải dữ liệu **thẳng vào S3** → phân tích bằng analytics/ML (vd **SageMaker**)
- Sản phẩm khác:
  - **Data Exchange for Redshift**: subscribe dữ liệu bên thứ 3 và **query ngay trong Redshift** trong vài phút; cũng để **bán/cấp phép** dữ liệu Redshift của bạn
  - **Data Exchange for APIs**: subscribe API bên thứ 3, truy cập nhất quán qua **AWS SDK**, xác thực & quản trị kiểu AWS
  - Data Exchange for S3 (truy cập trực tiếp bucket của nhà cung cấp), Lake Formation data permissions

### AWS Transfer Family

> 📖 **Định nghĩa:** Xem bảng trên.

- Giao thức: **FTP**, **FTPS**, **SFTP** (và **AS2** cho B2B)
- Hạ tầng managed, scale, tin cậy, **Multi-AZ**
- Giá: **theo endpoint provisioned mỗi giờ + GB truyền**
- Lưu & quản lý **credential user** trong dịch vụ, hoặc tích hợp **AD, LDAP, Okta, Cognito, custom** (Lambda/API Gateway)
- Transfer Family dùng **IAM role** để truy cập S3/EFS
- Tùy chọn **Route 53** đặt hostname thân thiện
- ⚠️ **FTP chỉ trong VPC** (không qua Internet)
- Use case: chia sẻ file với đối tác, dataset public, CRM, ERP…
- 🆕 **Transfer Family web apps**: giao diện web cho user truy cập S3

### Endpoint Types

> 📖 **Định nghĩa:** **Allow list** là danh sách IP nguồn được phép kết nối.

| Loại | IP | Kiểm soát truy cập |
|---|---|---|
| **Public endpoint** | IP do AWS quản lý (**có thể thay đổi**, dùng DNS name) | **Không** đặt allow list theo IP nguồn được |
| **VPC endpoint – internal access** | **Private IP tĩnh** | Allow list bằng **SG & NACL**; truy cập từ VPC, on-prem qua VPN/DX |
| **VPC endpoint – internet-facing** | **Public IP tĩnh (EIP)** + private IP | **Security Group** lọc IP nguồn |

## 🎯 Mẹo thi

- "Mua dữ liệu bên thứ 3 nạp vào S3/Redshift" → **Data Exchange**
- "Đối tác gửi file bằng SFTP vào S3, không quản lý server" → **Transfer Family**
- "SFTP cần IP tĩnh để đối tác whitelist, lọc IP nguồn" → **VPC endpoint internet-facing (EIP)** + SG
- "SFTP xác thực bằng AD công ty" → Transfer Family + **AD**
- "Giao thức FTP (không mã hóa)" → chỉ **trong VPC**
