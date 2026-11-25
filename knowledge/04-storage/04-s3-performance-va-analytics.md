# Amazon S3 – Performance, Storage Class Analysis & Storage Lens

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Prefix** | Phần đường dẫn giữa tên bucket và tên object (vd `bucket/folder1/sub1/file` → prefix `/folder1/sub1/`); S3 scale hiệu năng **theo prefix**. |
| **Multi-part upload** | Chia file lớn thành nhiều phần upload **song song**, ghép lại ở S3. |
| **S3 Transfer Acceleration** | Upload/download qua **edge location** của CloudFront rồi đi mạng private AWS tới bucket ở region đích. |
| **Byte-range fetch** | Request GET chỉ lấy **một đoạn byte** của object (header `Range`). |
| **Storage Class Analysis (S3 Analytics)** | Tính năng phân tích mẫu truy cập để gợi ý khi nào chuyển object sang Standard-IA. |
| **S3 Storage Lens** | Công cụ phân tích & tối ưu lưu trữ S3 trên **toàn bộ Organization** với dashboard và metric. |

## Nội dung chính

### Hiệu năng cơ bản

> 📖 **Định nghĩa:** Xem *Prefix* ở bảng trên.

- S3 tự scale request rate cao, độ trễ **100–200 ms**
- Mỗi **prefix**: ít nhất **3.500 PUT/COPY/POST/DELETE** và **5.500 GET/HEAD** request/giây
- **Không giới hạn số prefix** trong bucket
- Ví dụ: chia đọc đều 4 prefix → **22.000 GET/HEAD/giây**

### Tăng tốc upload/download

> 📖 **Định nghĩa:** Xem bảng trên.

- **Multi-part upload**: khuyến nghị cho file **> 100 MB**, **bắt buộc > 5 GB**; song song hóa → nhanh hơn
- **Transfer Acceleration**: upload vào edge gần user → mạng private AWS → bucket xa (vd user Mỹ → bucket Úc); tương thích multi-part
- **Byte-range fetch**: tải song song nhiều đoạn để **tăng tốc download**, **chịu lỗi tốt hơn** (chỉ tải lại đoạn lỗi); hoặc chỉ lấy **phần đầu file** (header)

### Dọn multi-part upload dở dang

> 📖 **Định nghĩa:** **Incomplete multi-part upload** là các phần đã upload nhưng chưa được ghép (complete); chúng **vẫn bị tính phí lưu trữ**.

- Liệt kê bằng CLI: `aws s3api list-multipart-uploads`
- **Lifecycle policy** tự **abort & xóa** multi-part upload dở dang sau X ngày (`AbortIncompleteMultipartUpload`)

### Storage Class Analysis

> 📖 **Định nghĩa:** Xem bảng trên.

- Giúp quyết định **khi nào chuyển** object sang storage class phù hợp
- Gợi ý cho **Standard và Standard-IA** — **không** cho One Zone-IA hay Glacier
- Báo cáo cập nhật **hằng ngày**, bắt đầu có dữ liệu sau **24–48 giờ**
- Xuất CSV, trực quan bằng **QuickSight**
- **Bước đầu tốt để viết Lifecycle Rules**

### S3 Storage Lens

> 📖 **Định nghĩa:** Xem bảng trên.

- Hiểu, phân tích, tối ưu lưu trữ trên **toàn Organization**
- Phát hiện bất thường, tối ưu chi phí, áp dụng best practice bảo vệ dữ liệu (dữ liệu **30 ngày**)
- Gộp theo **Organization, account, region, bucket, prefix**
- **Default dashboard** (multi-region, multi-account, do S3 cấu hình sẵn, **không xóa được, chỉ tắt**) hoặc dashboard tự tạo
- **Xuất metric hằng ngày** vào bucket S3 (CSV, Parquet)

**Nhóm metric:**

| Nhóm | Ví dụ | Dùng để |
|---|---|---|
| **Summary** | `StorageBytes`, `ObjectCount` | Bucket/prefix tăng nhanh nhất hoặc không dùng |
| **Cost-optimization** | `NonCurrentVersionStorageBytes`, `IncompleteMultipartUploadStorageBytes` | Tìm multi-part dở dang > 7 ngày, object nên chuyển class rẻ |
| **Data-protection** | `VersioningEnabledBucketCount`, `MFADeleteEnabledBucketCount`, `SSEKMSEnabledBucketCount`, `CrossRegionReplicationRuleCount` | Bucket không theo best practice bảo vệ dữ liệu |
| **Access-management** | `ObjectOwnershipBucketOwnerEnforcedBucketCount` | Thiết lập Object Ownership |
| **Event** | `EventNotificationEnabledBucketCount` | Bucket có event notification |
| **Performance** | `TransferAccelerationEnabledBucketCount` | Bucket bật Transfer Acceleration |
| **Activity** | `AllRequests`, `GetRequests`, `BytesDownloaded` | Cách storage được truy cập |
| **Status code** | `200OKStatusCount`, `403ForbiddenErrorCount`, `404NotFoundErrorCount` | Lỗi truy cập |

**Free vs Paid:**

| Free | Advanced (trả phí) |
|---|---|
| ~28 usage metric, tự động cho mọi khách hàng | Activity, advanced cost optimization, advanced data protection, status code |
| Dữ liệu truy vấn **14 ngày** | Dữ liệu **15 tháng** |
| – | **Publish lên CloudWatch** (không phí thêm), **prefix aggregation**, recommendations |

## 🎯 Mẹo thi

- "Upload file 10 GB" → **multi-part upload** (bắt buộc > 5 GB)
- "User ở xa upload chậm vào bucket" → **Transfer Acceleration**
- "Tăng tốc download / chỉ đọc header file lớn" → **byte-range fetch**
- "Request rate cao bị throttle 503" → **chia nhiều prefix**
- "Chi phí tăng do upload dở dang" → **lifecycle abort incomplete multi-part**
- "Biết khi nào nên chuyển sang Standard-IA" → **Storage Class Analysis**
- "Bucket nào trong cả org chưa bật versioning/mã hóa" → **Storage Lens**
- "Giữ metric Storage Lens 15 tháng, prefix-level" → **Advanced metrics**
