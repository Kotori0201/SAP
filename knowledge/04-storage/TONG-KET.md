# ⭐ Tổng kết · Storage

[⬅ Về mục lục section](README.md)

## 1. Bảng tổng hợp kiến thức

| Dịch vụ | Là gì (1 dòng) | Con số phải nhớ | Chọn khi |
|---|---|---|---|
| **EBS gp3** | SSD đa dụng, IOPS/throughput chỉnh riêng | Baseline 3.000 IOPS | Mặc định, boot |
| **EBS gp2** | SSD đa dụng (cũ) | 3 IOPS/GB, max **16.000** | Legacy |
| **EBS io1/io2 BE** | SSD hiệu năng cao | io1 **64.000**, io2 BE **256.000** IOPS · Multi-Attach | DB quan trọng |
| **EBS st1 / sc1** | HDD throughput / lạnh | Không làm boot | Big data / dữ liệu lạnh |
| **Instance Store** | Ổ vật lý trên host | Hàng **triệu** IOPS · mất khi stop | Cache, scratch |
| **EFS** | NFS managed, Linux, nhiều AZ | ~3x gp2 · NFSv4.1 · One Zone-IA tiết kiệm **> 90%** | File share Linux |
| **S3** | Object storage | **11 9's** · **3.500 PUT / 5.500 GET** / prefix · MPU **> 5 GB bắt buộc** | Object, tĩnh, data lake |
| **S3 Glacier Deep Archive** | Archive rẻ nhất | Lấy **12–48h** · tối thiểu **180 ngày** | Lưu trữ nhiều năm |
| **S3 RTC** | Replication có SLA | **99,99% trong 15 phút** | Tuân thủ replication |
| **FSx Windows** | Windows file share (SMB, AD) | Multi-AZ, DFS | App Windows |
| **FSx Lustre** | FS song song cho HPC | Scratch / Persistent · lazy load S3 | HPC, ML |
| **FSx ONTAP** | NetApp managed | NFS + SMB + iSCSI, dedup | Migrate NetApp/NAS |
| **FSx OpenZFS** | ZFS managed | **NFS only**, 1M IOPS < 0,5 ms | Migrate ZFS |
| **DataSync** | Di chuyển/đồng bộ dữ liệu online | **10 Gbps/agent**, giữ metadata | Migrate NFS/SMB, AWS↔AWS |
| **Transfer Family** | SFTP/FTPS/FTP vào S3/EFS | FTP chỉ trong VPC | Đối tác gửi file |
| **Data Exchange** | Chợ dữ liệu bên thứ 3 | S3, Redshift, API | Mua dữ liệu |

## 2. So sánh quan trọng

### Block vs File vs Object

| | Block (EBS) | File (EFS/FSx) | Object (S3) |
|---|---|---|---|
| Truy cập | 1 instance (trừ Multi-Attach), 1 AZ | Nhiều instance, nhiều AZ | API/HTTP, toàn cầu |
| Giao thức | Gắn như ổ đĩa | NFS / SMB / Lustre | REST |
| Use case | Boot, DB | File chia sẻ | Static, backup, data lake |

### EBS vs Instance Store

| EBS | Instance Store |
|---|---|
| Ổ mạng, bền vững | Ổ vật lý, tạm thời |
| Snapshot | Tự backup |
| Resize được | Không resize |

### EFS vs FSx

| Nhu cầu | Chọn |
|---|---|
| Linux NFS serverless | **EFS** |
| Windows SMB + AD | **FSx Windows** |
| HPC song song | **FSx Lustre** |
| Đa giao thức, NetApp | **FSx ONTAP** |
| ZFS | **FSx OpenZFS** |

### S3 Storage Classes (nhớ theo cột)

| Class | AZ | Lưu tối thiểu | Lấy dữ liệu |
|---|---|---|---|
| Standard | ≥3 | – | ms |
| Intelligent-Tiering | ≥3 | – | ms |
| Standard-IA | ≥3 | 30 ngày | ms |
| One Zone-IA | **1** | 30 ngày | ms |
| Glacier Instant | ≥3 | 90 ngày | ms |
| Glacier Flexible | ≥3 | 90 ngày | phút – 12h |
| Glacier Deep Archive | ≥3 | **180 ngày** | 12 – 48h |

### DLM vs AWS Backup

| DLM | AWS Backup |
|---|---|
| Chỉ EBS snapshot & AMI, theo tag | Nhiều dịch vụ, tập trung, Vault Lock |

### Di chuyển dữ liệu

| Nhu cầu | Chọn |
|---|---|
| Online, theo lịch, giữ quyền file | **DataSync** |
| PB, mạng kém | **Snowball** |
| On-prem truy cập liên tục dữ liệu AWS | **Storage Gateway** |
| Đối tác SFTP | **Transfer Family** |

## 3. Tip nhớ nhanh

- 🧠 **"Boot chỉ SSD"**: gp2/gp3/io1/io2 – không st1/sc1.
- 🧠 **"16 – 64 – 256"** nghìn IOPS: gp2 – io1 – io2 Block Express.
- 🧠 **"Multi-Attach = io + cùng AZ + cluster-aware FS"**.
- 🧠 **"Instance Store = Ở trọ"** – stop là mất đồ.
- 🧠 **"EFS = Linux, FSx Windows = Windows"**.
- 🧠 **"Lustre = Linux + Cluster = HPC"**; **Scratch = tạm, Persistent = bền**.
- 🧠 **"ONTAP = Ôm Tất (NFS + SMB + iSCSI), OpenZFS = Only NFS"**.
- 🧠 **"FSx chỉ lớn lên, không nhỏ lại"** → thu nhỏ = FSx mới + DataSync.
- 🧠 **"3.5 – 5.5"** nghìn request/giây/prefix: PUT – GET.
- 🧠 **"100 MB nên, 5 GB phải"** multi-part upload.
- 🧠 **"30 – 90 – 180"** ngày tối thiểu: IA – Glacier – Deep Archive.
- 🧠 **"Deep Archive = 12–48 giờ"**.
- 🧠 **"RTC = 15 phút"**; replication **chỉ object mới** → cũ dùng Batch Replication.
- 🧠 **"Storage Class Analysis chỉ Standard ↔ IA"**; **"Storage Lens = cả Organization"**.
- 🧠 **"DataSync = Đồng bộ, Snowball = Ship hàng"**.
