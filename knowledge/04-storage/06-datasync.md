# AWS DataSync

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **AWS DataSync** | Dịch vụ online **di chuyển và đồng bộ lượng lớn dữ liệu** giữa on-prem/cloud khác và AWS, hoặc giữa các dịch vụ lưu trữ AWS, giữ nguyên quyền và metadata. |
| **DataSync agent** | Máy ảo/phần mềm cài **gần nguồn dữ liệu** (on-prem) để đọc dữ liệu và gửi lên DataSync. |
| **Task** | Cấu hình một lần chuyển dữ liệu: nguồn, đích, lịch chạy, bộ lọc, giới hạn băng thông. |
| **VIF (Virtual Interface)** | Giao diện logic trên Direct Connect: **Public VIF** tới dịch vụ AWS public, **Private VIF** tới VPC. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** **Metadata** ở đây là thông tin đi kèm file: quyền POSIX, quyền SMB/ACL, timestamp…

- Di chuyển dữ liệu lớn **vào và ra** AWS:
  - **On-prem / cloud khác → AWS** (NFS, SMB, HDFS, S3 API…) → **cần agent**
  - **AWS → AWS** (giữa các dịch vụ lưu trữ) → **không cần agent**
- Đồng bộ tới:
  - **S3** (mọi storage class, **kể cả Glacier**)
  - **EFS**
  - **FSx** (Windows, Lustre, NetApp ONTAP, OpenZFS)
- Lên lịch **hằng giờ, hằng ngày, hằng tuần**
- **Giữ nguyên quyền file và metadata** (NFS POSIX, SMB…)
- Một task agent dùng tới **10 Gbps**, có **giới hạn băng thông**
- Dữ liệu truyền được mã hóa **TLS**, kiểm tra toàn vẹn

### Kiến trúc

> 📖 **Định nghĩa:** **PrivateLink / Interface VPC endpoint** cho phép truy cập dịch vụ AWS qua IP private trong VPC.

- **NFS/SMB on-prem → AWS:** Server NFS/SMB → **DataSync Agent** → TLS → DataSync → S3/EFS/FSx
- **Giữa dịch vụ AWS:** S3 ↔ EFS ↔ FSx, sao chép dữ liệu & metadata
- **Qua Direct Connect riêng tư:** Agent on-prem → **Private VIF** → **Interface VPC Endpoint (PrivateLink)** cho DataSync → không đi Internet công cộng (Public VIF cũng dùng được tới endpoint public)

### DataSync vs các lựa chọn khác

> 📖 **Định nghĩa:** **Offline transfer** là chuyển dữ liệu bằng thiết bị vật lý (Snow Family) thay vì qua mạng.

| Nhu cầu | Chọn |
|---|---|
| Đồng bộ/di chuyển file **qua mạng**, theo lịch, giữ metadata | **DataSync** |
| Dữ liệu **PB**, mạng chậm/không có | **Snowball** (offline) |
| On-prem truy cập liên tục dữ liệu trên AWS (cache cục bộ) | **Storage Gateway** |
| Đối tác gửi file qua **SFTP/FTP** | **Transfer Family** |

## 🎯 Mẹo thi

- "Migrate NFS/SMB on-prem lên EFS/FSx/S3, giữ quyền file" → **DataSync** (agent)
- "Sao chép EFS sang FSx / S3 sang EFS định kỳ" → **DataSync** (không agent)
- "Chuyển qua Direct Connect, không đi Internet" → **Private VIF + VPC endpoint cho DataSync**
- "Giới hạn băng thông để không ảnh hưởng mạng công ty" → **bandwidth limit** trên task
