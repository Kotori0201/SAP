# AWS Directory Services

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Microsoft Active Directory** | Dịch vụ thư mục quản lý tập trung user, máy tính, nhóm và chính sách trong mạng Windows. |
| **AWS Managed Microsoft AD** | Microsoft AD thật do AWS vận hành trong VPC của bạn, multi-AZ, hỗ trợ trust với AD on-prem. |
| **AD Connector** | Directory gateway (proxy) chuyển tiếp request xác thực về AD on-prem; không lưu user trên AWS. |
| **Simple AD** | Thư mục tương thích AD giá rẻ chạy Samba 4, tính năng cơ bản. |
| **Forest trust** | Quan hệ tin cậy giữa hai AD forest; khác với replication (sao chép dữ liệu). |
| **Seamless domain join** | Tự động join EC2 instance vào domain khi khởi chạy, không cần thao tác thủ công. |

## Nội dung chính

### Nền tảng

> 📖 **Định nghĩa:** **Active Directory (AD)** là dịch vụ thư mục của Microsoft lưu trữ và quản lý tập trung các object (user, máy tính, nhóm…). **Domain Controller (DC)** là server chạy AD. **Forest trust** là quan hệ tin cậy giữa hai forest AD cho phép user ở forest này xác thực tài nguyên ở forest kia. **ADFS** là thành phần của Windows Server cung cấp SSO qua SAML.

- **Microsoft AD**: CSDL object (user, computer, printer, file share, security group) trên Windows Server có AD DS. Object tổ chức thành **tree**, nhiều tree = **forest**. Server chạy AD = **Domain Controller**.
- **ADFS**: cung cấp **SSO qua SAML** cho app bên thứ 3 (AWS Console, Office 365, Dropbox…).

### So sánh 3 lựa chọn

> 📖 **Định nghĩa:** **AWS Directory Service** là họ dịch vụ cung cấp thư mục tương thích Microsoft AD trên AWS, gồm 3 lựa chọn: AWS Managed Microsoft AD, AD Connector và Simple AD.

| | **AWS Managed Microsoft AD** | **AD Connector** | **Simple AD** |
|---|---|---|---|
| Bản chất | AD thật do AWS quản lý trong VPC | **Proxy/gateway** chuyển request về AD on-prem | Directory tương thích AD (Samba 4) |
| User quản lý ở | AWS (có thể trust on-prem) | **On-prem** | AWS |
| Trust với on-prem | ✅ one-way / two-way forest trust | ❌ (chỉ proxy) | ❌ |
| MFA | ✅ | ✅ | ❌ |
| Cache | Có (là AD thật) | **Không cache** → phụ thuộc kết nối | – |
| Seamless domain join EC2 | ✅ (cả multi-account, multi-VPC) | ✅ (EC2 join domain on-prem qua connector) | ✅ |
| RDS SQL Server, FSx Windows, WorkSpaces, QuickSight | ✅ | Hạn chế (không RDS SQL Server, không share directory) | ❌ |
| Quy mô / giá | Cao nhất | Trung bình (Small/Large) | Rẻ nhất · Small ~500, Large ~5.000 user |
| Kết nối on-prem | Cần VPN/DX khi trust | **Bắt buộc VPN/DX** | Không |

**Đặc điểm Managed Microsoft AD:**
- Multi-AZ (tối thiểu 2 DC ở 2 AZ), tăng số DC để scale
- Backup tự động, **multi-Region replication**
- Domain join seamless EC2 từ **nhiều account & VPC** (chia sẻ directory)
- Tích hợp: RDS for SQL Server, WorkSpaces, QuickSight, Amazon Connect, FSx for Windows, **IAM Identity Center**

**Trust với on-prem AD** (cần **VPN hoặc Direct Connect**):
- One-way: AWS → on-prem
- One-way: on-prem → AWS
- **Two-way forest trust**
- ⚠️ **Trust ≠ đồng bộ (replication)**: user không được copy sang

**Kiến trúc giảm độ trễ / tăng sẵn sàng:** dựng **replica AD on-prem trên EC2** (self-managed, cùng domain on-prem) → replication với on-prem; thiết lập **trust** giữa Managed AD và replica EC2. Khi VPN/DX đứt, xác thực vẫn chạy trong cloud.

> 🎯 Từ khóa:
> - "Quản lý user **hoàn toàn on-prem**, không muốn đồng bộ/lưu trên AWS" → **AD Connector**
> - "Cần **trust**, RDS SQL Server, FSx Windows, nhiều tính năng AD" → **Managed Microsoft AD**
> - "Rẻ, ít user, không cần MFA/trust" → **Simple AD**
