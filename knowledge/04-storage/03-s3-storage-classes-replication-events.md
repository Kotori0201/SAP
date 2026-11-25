# Amazon S3 – Storage Classes, Replication & Event Notifications

[⬅ Về mục lục section](README.md)

> Phần bảo mật S3 (mã hóa, bucket policy, Object Lock, Access Points…) xem tại [02-security/06-s3-security.md](../02-security/06-s3-security.md).

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Amazon S3 (Simple Storage Service)** | Dịch vụ **object storage** serverless, dung lượng không giới hạn, trả theo dùng; truy cập object qua **key**. |
| **Object storage** | Lưu dữ liệu dưới dạng **object** (dữ liệu + metadata + key) trong một không gian phẳng, không phải cây thư mục thực sự. |
| **Storage class** | Lớp lưu trữ S3 với mức giá, độ sẵn sàng, phí truy xuất và thời gian lưu tối thiểu khác nhau. |
| **Durability** | Xác suất dữ liệu **không bị mất**; S3 đạt **11 số 9** (99,999999999%). |
| **Availability** | Tỷ lệ thời gian dữ liệu **truy cập được**. |
| **Lifecycle policy** | Quy tắc tự động **chuyển storage class** hoặc **xóa** object theo tuổi/điều kiện. |
| **Replication (CRR/SRR)** | Sao chép object **bất đồng bộ** sang bucket khác: **Cross-Region** hoặc **Same-Region**; yêu cầu **versioning**. |
| **S3 Replication Time Control (RTC)** | Tùy chọn replication có **SLA**: 99,99% object được sao chép trong **15 phút**. |

## Nội dung chính

### Tổng quan S3

> 📖 **Định nghĩa:** **Anti-pattern** là cách dùng không phù hợp với thiết kế của dịch vụ.

- Object storage, serverless, không giới hạn, trả theo dùng
- Tốt cho **nội dung tĩnh** (ảnh, video)
- Truy cập theo **key**, **không có index/tìm kiếm**
- **Không phải file system**, không mount native trên EC2 (🆕 có **Mountpoint for Amazon S3** để mount và đọc/ghi tuần tự — vẫn không phải POSIX đầy đủ)
- **Anti-pattern:**
  - Rất nhiều file nhỏ
  - Cần **POSIX file system, file lock** → dùng **EFS**
  - Tìm kiếm, query, dữ liệu thay đổi liên tục
  - Website **nội dung động**

### So sánh Storage Classes

> 📖 **Định nghĩa:** **Minimum storage duration** là số ngày tối thiểu bị tính phí dù xóa sớm hơn; **minimum billable object size** là kích thước tối thiểu tính phí cho mỗi object.

| | Standard | Intelligent-Tiering | Standard-IA | One Zone-IA | Glacier Instant | Glacier Flexible | Glacier Deep Archive |
|---|---|---|---|---|---|---|---|
| Durability | 11 9's | 11 9's | 11 9's | 11 9's | 11 9's | 11 9's | 11 9's |
| Availability | 99,99% | 99,9% | 99,9% | **99,5%** | 99,9% | 99,99% | 99,99% |
| Số AZ | ≥ 3 | ≥ 3 | ≥ 3 | **1** | ≥ 3 | ≥ 3 | ≥ 3 |
| Lưu tối thiểu | – | – | 30 ngày | 30 ngày | 90 ngày | 90 ngày | **180 ngày** |
| Object tối thiểu tính phí | – | – | 128 KB | 128 KB | 128 KB | 40 KB | 40 KB |
| Phí truy xuất | – | – | Theo GB | Theo GB | Theo GB | Theo GB | Theo GB |
| Thời gian lấy dữ liệu | ms | ms | ms | ms | **ms** | phút – 12 giờ | **12 – 48 giờ** |

- **Intelligent-Tiering**: tự chuyển tier theo mức truy cập, phí giám sát nhỏ/object, **không phí truy xuất**
- 🆕 **S3 Express One Zone**: lớp hiệu năng cao, độ trễ **mili giây một chữ số**, trong **một AZ** (directory bucket) – cho ML training, analytics tương tác
- Chuyển tier/xóa bằng **S3 Lifecycle Policies**

### Replication

> 📖 **Định nghĩa:** Xem bảng trên.

- **CRR (Cross-Region)** và **SRR (Same-Region)** — **bắt buộc bật versioning** ở cả hai bucket
- Kết hợp **Lifecycle rule** (vd bucket đích chuyển sang Glacier)
- Lợi ích: **giảm độ trễ, DR, bảo mật** (copy sang account khác), tuân thủ
- **S3 RTC**: hầu hết object sao chép trong vài giây, **99,99% trong 15 phút** (có SLA, metric)
- Lưu ý:
  - Chỉ object **mới** sau khi bật được replicate → object cũ dùng **S3 Batch Replication**
  - Delete marker replicate **tùy chọn**; xóa theo version ID **không** replicate
  - **Không replicate chuỗi** (A→B→C không tự sang C)

### S3 Event Notifications

> 📖 **Định nghĩa:** **Event Notification** là cơ chế S3 gửi thông báo khi có sự kiện trên object tới SNS, SQS, Lambda hoặc EventBridge.

- Sự kiện: `s3:ObjectCreated`, `s3:ObjectRemoved`, `s3:ObjectRestore`, `s3:Replication`…
- **Lọc theo tên object** (prefix/suffix, vd `*.jpg`)
- Use case: **tạo thumbnail** ảnh mới upload
- Tạo bao nhiêu event config cũng được
- Thường giao trong **vài giây**, đôi khi **một phút hoặc hơn**
- Đích trực tiếp: **SNS, SQS, Lambda** (cần resource policy cho phép S3)

### S3 Event Notifications với EventBridge

> 📖 **Định nghĩa:** **Archive & replay** là khả năng của EventBridge lưu lại event và phát lại sau.

- Bật trên bucket → **mọi event** gửi tới EventBridge → rule định tuyến
- **Lọc nâng cao** bằng JSON rule (metadata, kích thước object, tên…)
- **Nhiều đích**: Step Functions, Kinesis Data Streams/Firehose… (hơn 18 dịch vụ)
- Tính năng EventBridge: **archive, replay**, giao tin cậy

## 🎯 Mẹo thi

- "Truy cập không đoán được, tối ưu chi phí tự động" → **Intelligent-Tiering**
- "Dữ liệu có thể tái tạo, truy cập ít, rẻ" → **One Zone-IA**
- "Lưu trữ 7–10 năm, lấy trong 48 giờ, rẻ nhất" → **Glacier Deep Archive**
- "Archive cần lấy ngay trong ms" → **Glacier Instant Retrieval**
- "Replication có SLA 15 phút" → **S3 RTC**
- "Replicate object có sẵn trước khi bật replication" → **S3 Batch Replication**
- "Một sự kiện S3 tới nhiều đích, lọc theo kích thước" → **EventBridge**
- "Độ trễ mili giây một chữ số cho ML training" → **S3 Express One Zone** 🆕
