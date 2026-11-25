# Amazon FSx

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Amazon FSx** | Họ dịch vụ **file system bên thứ 3 hiệu năng cao, fully managed**: FSx for Windows File Server, Lustre, NetApp ONTAP, OpenZFS. |
| **SMB** | *Server Message Block* – giao thức chia sẻ file của Windows. |
| **NTFS** | File system của Windows (hỗ trợ ACL, quota). |
| **DFS Namespaces** | *Distributed File System* của Microsoft – gộp nhiều file share thành **một namespace** thống nhất. |
| **Lustre** | File system **song song phân tán** ("Linux" + "cluster") cho tính toán quy mô lớn. |
| **iSCSI** | Giao thức truy cập **block storage** qua mạng IP. |
| **Lazy loading (FSx Lustre)** | Chỉ tải từ S3 **phần dữ liệu thực sự được xử lý**, khi được truy cập lần đầu. |

## Nội dung chính

### FSx for Windows File Server

> 📖 **Định nghĩa:** Xem *SMB, NTFS, DFS* ở bảng trên.

- **Windows file share** fully managed
- **SMB** & **Windows NTFS**
- Tích hợp **Microsoft Active Directory**, ACL, user quota
- **Mount được trên Linux EC2**
- Hỗ trợ **DFS Namespaces** (gộp file trên nhiều file system)
- Scale tới **hàng chục GB/s**, **hàng triệu IOPS**, **hàng trăm PB**
- Lưu trữ: **SSD** (DB, media, analytics – nhạy độ trễ) hoặc **HDD** (home directory, CMS)
- Truy cập từ **on-prem** (VPN/DX)
- Cấu hình **Multi-AZ** (HA)
- **Backup hằng ngày lên S3**

### FSx for Lustre

> 📖 **Định nghĩa:** Xem *Lustre* ở bảng trên.

- Use case: **ML, HPC**, xử lý video, mô hình tài chính, EDA (thiết kế điện tử)
- Scale **hàng trăm GB/s**, **hàng triệu IOPS**, **độ trễ sub-ms**
- Lưu trữ: **SSD** (độ trễ thấp, IOPS cao, file nhỏ/ngẫu nhiên) hoặc **HDD** (throughput, file lớn/tuần tự)
- **Tích hợp S3 liền mạch**: "đọc S3 như file system", **ghi kết quả ngược về S3**
- Dùng được từ on-prem (VPN/DX)

### Deployment options của FSx for Lustre

> 📖 **Định nghĩa:** **Scratch file system** là FS tạm thời, không replicate; **Persistent file system** là FS lâu dài, replicate trong cùng AZ.

| | **Scratch** | **Persistent** |
|---|---|---|
| Lưu trữ | **Tạm thời** | **Lâu dài** |
| Replicate | **Không** (server hỏng → mất dữ liệu) | Trong **cùng AZ**, thay file lỗi trong vài phút |
| Hiệu năng | Burst cao (**6x**, 200 MBps/TiB) | Ổn định |
| Dùng khi | Xử lý **ngắn hạn**, **tối ưu chi phí** | Xử lý **dài hạn**, **dữ liệu nhạy cảm** |

### FSx for Lustre – Lazy Loading

> 📖 **Định nghĩa:** Xem bảng trên.

- Job xử lý với S3 làm nguồn chạy được **không cần tải toàn bộ dataset trước**
- Chỉ tải **dữ liệu thực sự xử lý** → **giảm chi phí và độ trễ**
- Dữ liệu chỉ tải **một lần** → giảm request tới S3

### FSx for NetApp ONTAP

> 📖 **Định nghĩa:** **NetApp ONTAP** là hệ điều hành lưu trữ của NetApp, phổ biến trong NAS doanh nghiệp.

- NetApp ONTAP managed trên AWS
- Giao thức: **NFS, SMB, iSCSI** (đa giao thức)
- **Migrate workload ONTAP hoặc NAS** lên AWS
- Dùng với: Linux, Windows, macOS, **VMware Cloud on AWS**, WorkSpaces, AppStream 2.0, EC2/ECS/EKS
- Dung lượng **tự co giãn**
- Snapshot, replication, chi phí thấp, **nén và khử trùng lặp (deduplication)**
- **Clone tức thì point-in-time** (test workload mới)

### FSx for OpenZFS

> 📖 **Định nghĩa:** **ZFS** là file system/volume manager nổi tiếng về toàn vẹn dữ liệu, snapshot, nén.

- OpenZFS managed trên AWS
- Giao thức: **chỉ NFS** (v3, v4, v4.1, v4.2)
- **Migrate workload ZFS** lên AWS
- Dùng với: Linux, Windows, macOS, VMware Cloud on AWS, WorkSpaces, AppStream, EC2/ECS/EKS
- Tới **1.000.000 IOPS**, độ trễ **< 0,5 ms**
- Snapshot, nén, chi phí thấp, **clone tức thì**
- ⚠️ **Không có deduplication** (khác ONTAP)

### Kiến trúc FSx

> 📖 **Định nghĩa:** **Single-AZ / Multi-AZ** là kiểu triển khai file system trong một hoặc hai AZ (standby tự failover).

- **Chuyển Single-AZ → Multi-AZ:** tạo FSx Multi-AZ mới, **DataSync** đồng bộ dữ liệu (hoặc **backup → restore** vào Multi-AZ)
- **Giảm dung lượng FSx:** chỉ **tăng** được, **không giảm**; restore backup cũng phải cùng size → tạo FSx **nhỏ hơn mới**, **DataSync** đồng bộ, chuyển ứng dụng

### Chọn FSx nào?

> 📖 **Định nghĩa:** **Lift-and-shift** là chuyển workload lên cloud mà gần như không thay đổi.

| Yêu cầu | Chọn |
|---|---|
| Windows share, SMB, AD, NTFS | **FSx for Windows** |
| HPC/ML, song song, liên kết S3 | **FSx for Lustre** |
| Đa giao thức NFS+SMB+iSCSI, migrate NetApp/NAS, dedup | **FSx for NetApp ONTAP** |
| Migrate ZFS, NFS, IOPS cao độ trễ thấp | **FSx for OpenZFS** |
| Linux NFS đơn giản, serverless | **EFS** |

## 🎯 Mẹo thi

- "Windows app cần file share với AD" → **FSx for Windows** (Multi-AZ cho HA)
- "Gộp nhiều file share Windows thành một" → **DFS Namespaces**
- "HPC đọc dataset S3 như file system" → **FSx for Lustre** (lazy loading)
- "Xử lý ngắn hạn, rẻ" → Lustre **Scratch**; "dài hạn, nhạy cảm" → **Persistent**
- "Đang dùng NetApp on-prem" → **FSx for NetApp ONTAP**
- "Giảm dung lượng FSx" → FSx mới nhỏ hơn + **DataSync**
- "Chuyển Single-AZ sang Multi-AZ" → **DataSync** hoặc backup/restore
