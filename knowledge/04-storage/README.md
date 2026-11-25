# 04 · Storage

> Nguồn: Slide Stephane Maarek SAP v38 (trang 318–374) + section *Storage* trên Udemy, có bổ sung cập nhật AWS (🆕). Ghi chú viết lại để ôn tập.

**Domain SAP-C03:** Cost-optimized architecture · Resilience, migration & business continuity · Cloud-native architecture

## 📖 Định nghĩa tổng quan

- **Block storage:** lưu dạng khối, gắn vào một máy như ổ đĩa (EBS, Instance Store).
- **File storage:** hệ thống file chia sẻ qua mạng cho nhiều máy (EFS – NFS, FSx – SMB/NFS/Lustre/iSCSI).
- **Object storage:** lưu object theo key trong không gian phẳng, truy cập qua API (S3).
- **Durability vs Availability:** độ bền = không mất dữ liệu; độ sẵn sàng = truy cập được khi cần.
- **Data transfer service:** dịch vụ di chuyển dữ liệu vào/ra/giữa các dịch vụ lưu trữ (DataSync, Transfer Family, Data Exchange).
- **Storage tiering:** chuyển dữ liệu sang lớp rẻ hơn theo tần suất truy cập (S3 lifecycle, EFS lifecycle).

## Mục lục

| # | Chủ đề | File |
|---|---|---|
| 1 | EBS & Instance Store | [01-ebs-va-instance-store.md](01-ebs-va-instance-store.md) |
| 2 | EFS | [02-efs.md](02-efs.md) |
| 3 | S3 – Storage classes, Replication, Event Notifications | [03-s3-storage-classes-replication-events.md](03-s3-storage-classes-replication-events.md) |
| 4 | S3 – Performance, Storage Class Analysis, Storage Lens | [04-s3-performance-va-analytics.md](04-s3-performance-va-analytics.md) |
| 5 | FSx (Windows, Lustre, NetApp ONTAP, OpenZFS) | [05-fsx.md](05-fsx.md) |
| 6 | DataSync | [06-datasync.md](06-datasync.md) |
| 7 | Data Exchange & Transfer Family | [07-data-exchange-va-transfer-family.md](07-data-exchange-va-transfer-family.md) |
| 8 | Kiến trúc lưu trữ & chi phí | [08-kien-truc-va-chi-phi-storage.md](08-kien-truc-va-chi-phi-storage.md) |

> Bảo mật S3 (mã hóa, Object Lock, Access Points, MRAP, Object Lambda): [02-security/06-s3-security.md](../02-security/06-s3-security.md)

## Cheat-sheet: Tình huống → Giải pháp

| Tình huống trong đề | Giải pháp |
|---|---|
| Chuyển volume EBS sang AZ/region khác | **Snapshot → copy → restore** |
| IOPS ổn định rất cao cho DB | **io2 Block Express** |
| Big data đọc tuần tự, rẻ | **st1** |
| Dữ liệu lạnh trên EBS rẻ nhất | **sc1** |
| Restore volume không bị chậm lần đọc đầu | **Fast Snapshot Restore** |
| Tự động snapshot EBS theo tag | **Data Lifecycle Manager** |
| Backup tập trung nhiều dịch vụ, WORM | **AWS Backup** (+ Vault Lock) |
| Mọi volume EBS mới phải mã hóa | **EBS encryption by default** (từng region) |
| Nhiều EC2 cùng ghi một block volume | **io2 Multi-Attach** + cluster-aware FS |
| Scratch/cache IOPS cao nhất | **Instance Store** |
| Linux nhiều AZ chia sẻ file | **EFS** |
| Throughput EFS khó đoán | **EFS Elastic throughput** |
| EFS dev/test rẻ nhất | **EFS One Zone-IA** |
| Mỗi app chỉ thấy thư mục riêng trên EFS | **EFS Access Points** |
| DR file system sang region khác | **EFS replication** |
| Truy cập S3 không đoán được | **S3 Intelligent-Tiering** |
| Lưu trữ nhiều năm, rẻ nhất | **Glacier Deep Archive** |
| Archive nhưng lấy ngay trong ms | **Glacier Instant Retrieval** |
| Replication có SLA 15 phút | **S3 RTC** |
| Replicate object có sẵn | **S3 Batch Replication** |
| Một event S3 tới nhiều đích, lọc nâng cao | **S3 → EventBridge** |
| Upload file > 5 GB | **Multi-part upload** |
| User ở xa upload chậm | **S3 Transfer Acceleration** |
| Tăng tốc download / đọc header file | **Byte-range fetch** |
| Throttle 503 trên S3 | **Chia nhiều prefix** |
| Chi phí do upload dở dang | **Lifecycle abort incomplete MPU** |
| Khi nào chuyển sang Standard-IA | **Storage Class Analysis** |
| Bucket nào trong org chưa theo best practice | **S3 Storage Lens** |
| Windows file share với AD | **FSx for Windows** (Multi-AZ) |
| HPC đọc dataset S3 như file system | **FSx for Lustre** (lazy loading) |
| Migrate NetApp, đa giao thức, dedup | **FSx for NetApp ONTAP** |
| Migrate ZFS | **FSx for OpenZFS** |
| Giảm dung lượng FSx / Single → Multi-AZ | FSx mới + **DataSync** |
| Migrate NFS/SMB on-prem, giữ quyền file | **DataSync** (agent) |
| Sao chép giữa S3/EFS/FSx | **DataSync** (không agent) |
| DataSync không qua Internet | **Private VIF + VPC endpoint** |
| Mua dữ liệu bên thứ 3 | **Data Exchange** |
| Đối tác gửi file SFTP vào S3 | **Transfer Family** |
| SFTP cần IP tĩnh + lọc IP nguồn | **Transfer Family VPC internet-facing (EIP)** |
| Tìm object S3 theo thuộc tính | **Index metadata trong DynamoDB** / S3 Metadata 🆕 |
| Nội dung tĩnh toàn cầu rẻ nhất | **CloudFront + S3 (OAC)** |
| Độ trễ ms một chữ số cho ML/analytics trên S3 | **S3 Express One Zone** 🆕 |

## Bẫy hay gặp trong đề

- ❌ "st1/sc1 làm boot volume" → chỉ **gp2/gp3, io1/io2**.
- ❌ "EBS gắn sang AZ khác" → phải **snapshot → restore**.
- ❌ "EBS mặc định mã hóa" → phải bật **encryption by default** (từng region).
- ❌ "Multi-Attach với gp3 / EXT4" → chỉ **io1/io2**, cần **cluster-aware FS**.
- ❌ "Instance Store giữ dữ liệu khi stop" → **mất** (chỉ giữ khi reboot).
- ❌ "DLM quản lý snapshot tạo thủ công" → **không**.
- ❌ "EFS cho Windows" → dùng **FSx for Windows**.
- ❌ "S3 làm POSIX file system có file lock" → dùng **EFS**.
- ❌ "Replication chép cả object cũ" → chỉ object mới; cũ dùng **Batch Replication**.
- ❌ "Storage Class Analysis gợi ý Glacier/One Zone-IA" → chỉ **Standard ↔ Standard-IA**.
- ❌ "Giảm dung lượng FSx tại chỗ" → **không được**.
- ❌ "FSx for OpenZFS hỗ trợ SMB/dedup" → **chỉ NFS**, không dedup.
- ❌ "Lustre Scratch có replicate" → **không**.
- ❌ "Transfer Family public endpoint có IP tĩnh" → IP **thay đổi**, không allow list được.
