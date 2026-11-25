# Amazon EBS & EC2 Instance Store

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Block storage** | Lưu trữ dạng khối thô, hệ điều hành định dạng thành file system (như ổ cứng). |
| **Amazon EBS (Elastic Block Store)** | **Ổ đĩa mạng** dạng block gắn vào EC2, **dữ liệu bền vững** độc lập với vòng đời instance, **gắn với một AZ**. |
| **IOPS** | *Input/Output Operations Per Second* – số thao tác đọc/ghi mỗi giây. |
| **Throughput** | Lượng dữ liệu đọc/ghi mỗi giây (MB/s). |
| **EBS Snapshot** | Bản sao lưu **incremental** của volume EBS tại một thời điểm, lưu trong S3 (do AWS quản lý, bạn không thấy bucket). |
| **Amazon Data Lifecycle Manager (DLM)** | Dịch vụ tự động **tạo, giữ và xóa** snapshot EBS và AMI dựa trên tag. |
| **EBS Multi-Attach** | Tính năng gắn **một volume io1/io2** vào **nhiều EC2 trong cùng AZ** cùng lúc. |
| **Instance Store** | Ổ đĩa **vật lý gắn trực tiếp** vào máy chủ chạy EC2; IOPS rất cao nhưng **dữ liệu tạm thời (ephemeral)**. |
| **EBS-optimized instance** | Instance có băng thông riêng dành cho EBS, không tranh chấp với traffic mạng khác. |

## Nội dung chính

### Đặc điểm EBS

> 📖 **Định nghĩa:** **AZ-bound** nghĩa là tài nguyên chỉ dùng được trong AZ nơi nó được tạo.

- Ổ mạng gắn vào **một instance** (trừ Multi-Attach)
- **Gắn với một AZ** → muốn chuyển AZ/region: **snapshot → restore**
- **Resize** được (tăng dung lượng, đổi loại, đổi IOPS khi đang chạy — Elastic Volumes)
- Chọn instance **EBS-optimized** để đạt throughput tối đa

### Loại volume

> 📖 **Định nghĩa:** **SSD** tối ưu IOPS và độ trễ cho thao tác nhỏ, ngẫu nhiên; **HDD** tối ưu throughput cho thao tác lớn, tuần tự.

| Loại | Công nghệ | Đặc điểm | Boot volume | Use case |
|---|---|---|---|---|
| **gp3** | SSD | General purpose; **IOPS và throughput chỉnh độc lập** với dung lượng; rẻ hơn gp2 | ✅ | Hầu hết workload, boot, dev/test |
| **gp2** | SSD | IOPS **tăng theo dung lượng** (3 IOPS/GB, tối đa 16.000) | ✅ | Legacy |
| **io1 / io2 Block Express** | SSD | **Hiệu năng cao nhất**, độ trễ thấp; io2 Block Express tới **256.000 IOPS**; hỗ trợ **Multi-Attach** | ✅ | DB quan trọng, ứng dụng cần IOPS ổn định |
| **st1** | HDD | Throughput cao, giá thấp, truy cập **thường xuyên** | ❌ | Big data, data warehouse, log processing |
| **sc1** | HDD | **Rẻ nhất**, truy cập **ít** | ❌ | Dữ liệu lạnh |

- Volume được mô tả bằng **Size | Throughput | IOPS**
- 🆕 gp3 đã được nâng giới hạn (tới ~80.000 IOPS, 64 TiB) vào 2025 — đối chiếu tài liệu mới nhất

### EBS Snapshots

> 📖 **Định nghĩa:** **Incremental backup** chỉ lưu các block **thay đổi** kể từ snapshot trước. **Pre-warm (initialize)** là đọc toàn bộ block của volume mới restore để tránh độ trễ lần đọc đầu tiên.

- **Incremental** – chỉ backup block thay đổi
- Snapshot **tốn I/O** → tránh chạy lúc tải cao
- Lưu trong **S3** (không thấy trực tiếp)
- **Không bắt buộc detach** volume, nhưng nên làm (đảm bảo nhất quán)
- **Copy cross-region** (cho DR), **share cross-account**
- Tạo **AMI từ snapshot**
- Volume restore từ snapshot cần **pre-warm**: bật **Fast Snapshot Restore (FSR)** (tốn phí) hoặc đọc toàn bộ bằng `fio`/`dd`
- Khác: **Snapshot Archive** (rẻ hơn 75%, restore 24–72h), **Recycle Bin** (chống xóa nhầm), 🆕 **Provisioned rate for volume initialization**

### Amazon Data Lifecycle Manager vs AWS Backup

> 📖 **Định nghĩa:** **AWS Backup** là dịch vụ backup **tập trung** cho nhiều dịch vụ AWS (EBS, EFS, RDS, DynamoDB, FSx, S3…) với backup plan, vault, cross-region/cross-account copy.

| | **Data Lifecycle Manager** | **AWS Backup** |
|---|---|---|
| Phạm vi | **Chỉ EBS snapshot & EBS-backed AMI** | **Nhiều dịch vụ** từ một chỗ |
| Chọn tài nguyên | **Theo tag** (EC2, EBS) | Tag hoặc resource ID |
| Tính năng | Lịch, giữ, xóa; **copy cross-account/region** | Backup plan, **Backup Vault Lock** (WORM), Organizations backup policy, audit |
| Hạn chế | Không quản lý snapshot/AMI **tạo ngoài DLM**; không cho AMI **instance-store backed** | – |

### Mã hóa EBS – thiết lập mức account

> 📖 **Định nghĩa:** **Encryption by default** là thiết lập của account để **mọi volume và snapshot EBS mới** được mã hóa tự động.

- Volume EBS mới **mặc định không mã hóa**
- Bật **encryption by default** ở mức account → mọi volume & snapshot mới tự mã hóa
- Thiết lập **theo từng region**
- Mã hóa bằng KMS: dữ liệu at rest, in-flight giữa instance và volume, snapshot, volume tạo từ snapshot

### EBS Multi-Attach (io1/io2)

> 📖 **Định nghĩa:** **Cluster-aware file system** là file system điều phối được việc ghi đồng thời từ nhiều máy (vd GFS2, OCFS2).

- Gắn **cùng volume** vào **nhiều EC2 trong cùng AZ**
- Mỗi instance có **toàn quyền đọc/ghi**
- Use case: tăng sẵn sàng cho **ứng dụng Linux dạng cluster** (vd Teradata)
- Ứng dụng phải **tự quản lý ghi đồng thời**
- Phải dùng **file system cluster-aware** (**không** phải XFS, EXT4…)

### Instance Store vs EBS

> 📖 **Định nghĩa:** **Ephemeral storage** là lưu trữ tạm thời, mất dữ liệu khi instance stop/terminate.

| | **Instance Store** | **EBS** |
|---|---|---|
| Bản chất | Ổ **vật lý** trên host | Ổ **mạng** |
| Bền vững | **Mất khi stop/terminate** (giữ khi reboot) | **Bền vững** |
| Hiệu năng | **Rất cao** (hàng triệu IOPS) | gp2 16.000, io1 64.000, io2 BE 256.000 IOPS |
| Resize | ❌ | ✅ |
| Backup | **Tự làm** | Snapshot |
| Dung lượng | Đĩa tới ~7,5 TiB, gộp tới ~60 TiB (thay đổi theo thời gian) | Tới 64 TiB/volume |
| Use case | **Buffer, cache, scratch, dữ liệu tạm**, DB tự replicate (Cassandra) | Dữ liệu bền vững, boot |

## 🎯 Mẹo thi

- "Volume ở AZ khác / region khác" → **snapshot → copy → restore**
- "DB cần IOPS ổn định > 16.000" → **io2 (Block Express)** hoặc gp3 cấu hình cao
- "Big data đọc tuần tự, rẻ" → **st1**; "dữ liệu lạnh rẻ nhất" → **sc1**
- "Restore volume nhanh, không có độ trễ lần đọc đầu" → **Fast Snapshot Restore**
- "Tự động snapshot EBS theo tag" → **DLM**; "backup tập trung nhiều dịch vụ" → **AWS Backup**
- "Mọi volume mới phải mã hóa" → **EBS encryption by default** (từng region)
- "Nhiều instance cùng ghi một block volume" → **io2 Multi-Attach** + cluster-aware FS
- "Scratch/cache cần IOPS cao nhất" → **Instance Store**
