# Amazon EFS (Elastic File System)

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Amazon EFS** | **NFS file system managed**, serverless, tự co giãn, mount đồng thời trên **nhiều EC2 ở nhiều AZ** (và on-prem), trả tiền theo GB sử dụng. |
| **NFS (Network File System)** | Giao thức chia sẻ file qua mạng; EFS dùng **NFSv4.1**. |
| **POSIX** | Chuẩn giao diện hệ điều hành kiểu Unix (quyền user/group, file API chuẩn). |
| **Mount target** | ENI trong một subnet/AZ mà client kết nối tới để mount EFS. |
| **EFS Access Point** | Điểm vào ứng dụng cụ thể: **ép POSIX user/group** và **giới hạn thư mục gốc** khi truy cập file system. |
| **File system policy** | Resource-based policy của EFS (giống bucket policy), kiểm soát quyền truy cập NFS bằng IAM. |
| **Lifecycle management** | Tự động chuyển file sang lớp lưu trữ rẻ hơn khi không truy cập sau N ngày. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** Xem bảng trên.

- Mount trên **nhiều EC2**, **multi-AZ**, và **on-prem** (qua DX/VPN)
- Sẵn sàng cao, scale tốt, **đắt (~3x gp2)**, trả theo GB dùng
- Use case: **content management, web serving, chia sẻ dữ liệu, WordPress**
- **Chỉ Linux** (không Windows), POSIX-compliant
- Kiểm soát truy cập mạng bằng **Security Group**
- Mã hóa at rest bằng **KMS**
- Tự scale, **không cần capacity planning**

### Hiệu năng

> 📖 **Định nghĩa:** **Performance mode** quyết định độ trễ/khả năng song song (chọn **lúc tạo**); **Throughput mode** quyết định cách tính throughput.

- Scale: **hàng nghìn NFS client**, **10+ GB/s**, tới **PB**
- **Performance mode** (chọn khi tạo):
  - **General Purpose** (mặc định) – nhạy độ trễ (web server, CMS)
  - **Max I/O** – độ trễ cao hơn, throughput cao, song song lớn (big data, media) *(legacy; khuyến nghị General Purpose + Elastic)*
- **Throughput mode:**
  - **Bursting** – 1 TB = 50 MiB/s + burst tới 100 MiB/s
  - **Provisioned** – đặt throughput **không phụ thuộc dung lượng** (vd 1 GiB/s cho 1 TB)
  - **Elastic** (khuyến nghị) – tự scale theo workload, tới ~3 GiB/s đọc / 1 GiB/s ghi (cao hơn theo region) → **workload khó đoán**

### Storage classes

> 📖 **Định nghĩa:** **EFS-IA (Infrequent Access)** lưu rẻ nhưng tính phí truy xuất; **Archive** cho dữ liệu truy cập vài lần mỗi năm; **One Zone** lưu trong một AZ.

- **Tier** (lifecycle policy chuyển file sau N ngày):
  - **Standard** – truy cập thường xuyên
  - **Infrequent Access (IA)** – lưu rẻ hơn, **phí truy xuất**
  - **Archive** – truy cập rất hiếm, rẻ hơn IA ~50%
- **Sẵn sàng & độ bền:**
  - **Standard (Regional)** – Multi-AZ, cho **production**
  - **One Zone** – một AZ, cho **dev**, backup bật mặc định, kết hợp IA (**One Zone-IA**) → **tiết kiệm > 90%**

### Truy cập từ on-prem & VPC peering

> 📖 **Định nghĩa:** **Mount by IP** là mount EFS bằng IP của mount target thay vì DNS name (DNS của EFS chỉ phân giải trong VPC).

- On-prem qua **Direct Connect** hoặc **Site-to-Site VPN** (dự phòng DX/DX hoặc DX/VPN)
- VPC khác qua **VPC peering / Transit Gateway**
- ⚠️ Mount bằng **IPv4 của mount target** (không phải DNS) khi từ ngoài VPC, hoặc tự cấu hình DNS
- Dự phòng: **mount target ở nhiều AZ**

### EFS Access Points

> 📖 **Định nghĩa:** Xem bảng trên.

- Quản lý truy cập ứng dụng vào môi trường NFS dễ hơn
- **Ép POSIX UID/GID** khi truy cập
- **Giới hạn vào một thư mục**, tùy chọn đặt **root directory khác** (vd AP1 → `/config` UID 1001, AP2 → `/data` UID 1002)
- Kết hợp **IAM policy** để giới hạn NFS client nào được dùng access point nào

### File System Policies

> 📖 **Định nghĩa:** Xem bảng trên.

- Resource-based policy (giống S3 bucket policy)
- **Mặc định cấp full access cho mọi client** (có mạng tới)
- Dùng để: cấp R/W cho IAM principal cụ thể, **ép mã hóa in-transit** (`aws:SecureTransport`), **chặn root access**, chặn anonymous

### Cross-Region Replication

> 📖 **Định nghĩa:** **RPO/RTO** – *Recovery Point Objective* (lượng dữ liệu tối đa chấp nhận mất) và *Recovery Time Objective* (thời gian tối đa để khôi phục).

- Replicate file system sang **region khác**
- Áp dụng cho file system **mới hoặc có sẵn**
- **RPO & RTO tính bằng phút**
- **Không ảnh hưởng** provisioned throughput của file system nguồn
- Use case: **tuân thủ, business continuity/DR**

## 🎯 Mẹo thi

- "Nhiều EC2 Linux ở nhiều AZ chia sẻ file" → **EFS**
- "Windows file share" → **FSx for Windows**, không phải EFS
- "Workload throughput khó đoán" → **Elastic throughput**
- "Dev/test, giảm chi phí tối đa" → **One Zone-IA**
- "Mỗi ứng dụng chỉ thấy thư mục riêng, user POSIX cố định" → **Access Points**
- "Ép mã hóa khi mount" → **file system policy** `aws:SecureTransport`
- "DR file system sang region khác, RPO phút" → **EFS replication**
- "Container Fargate/EKS cần volume chia sẻ" → **EFS**
