# Amazon DynamoDB & DAX

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Amazon DynamoDB** | Database **NoSQL key-value & document**, serverless, fully managed, độ trễ mili giây một chữ số ở mọi quy mô. |
| **NoSQL** | Nhóm database **không dùng mô hình bảng quan hệ** cố định; schema linh hoạt, scale ngang tốt. |
| **Table / Item / Attribute** | Table = bảng; Item = một dòng (tối đa **400 KB**); Attribute = một trường (thêm dần, có thể null). |
| **Partition key (HASH)** | Thuộc tính quyết định **item nằm ở partition nào**; phải **đa dạng** để dữ liệu phân bố đều. |
| **Sort key (RANGE)** | Thuộc tính sắp xếp các item **trong cùng partition key**. |
| **RCU / WCU** | *Read/Write Capacity Unit* – đơn vị đo throughput đọc/ghi ở chế độ provisioned. |
| **LSI / GSI** | *Local Secondary Index* (cùng partition key, sort key khác) / *Global Secondary Index* (partition key khác). |
| **DynamoDB Streams** | Luồng **thay đổi theo item** của bảng (insert/update/delete), giữ **24 giờ**. |
| **Global Tables** | Bảng **replicate active-active** giữa nhiều region. |
| **DAX (DynamoDB Accelerator)** | **Cache in-memory chuyên cho DynamoDB**, đọc ở mức **micro giây**, không phải sửa code ứng dụng. |
| **Hot key / hot partition** | Một key/partition nhận quá nhiều request so với phần còn lại → bị throttle. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** **ACID transaction** là giao dịch đảm bảo *Atomicity, Consistency, Isolation, Durability* — tất cả thao tác cùng thành công hoặc cùng thất bại.

- NoSQL, fully managed, quy mô cực lớn (**1.000.000 rps**)
- Giống **Apache Cassandra** (migrate được sang DynamoDB)
- **Không cần provision dung lượng đĩa**; item tối đa **400 KB**
- **Capacity mode:** **provisioned** (WCU, RCU + Auto Scaling) hoặc **on-demand**
- CRUD; đọc **eventually consistent** hoặc **strongly consistent**
- **Transaction đa bảng (ACID)**
- **Backup** & **Point-in-Time Recovery (PITR)** tới 35 ngày
- **Table class**: **Standard** và **Standard-IA** (lưu rẻ hơn, đọc/ghi đắt hơn)

### Kiểu dữ liệu

> 📖 **Định nghĩa:** **Scalar** là giá trị đơn; **Document** là cấu trúc lồng (List, Map); **Set** là tập giá trị không trùng.

- **Scalar**: String, Number, Binary, Boolean, Null
- **Document**: List, Map
- **Set**: String Set, Number Set, Binary Set
- **Primary key phải chọn khi tạo bảng**

### Primary Key

> 📖 **Định nghĩa:** **Composite key** là primary key gồm **partition key + sort key**; tổ hợp phải duy nhất.

| Lựa chọn | Quy tắc | Ví dụ |
|---|---|---|
| **Partition key only** | Partition key **duy nhất** cho mỗi item, phải **đa dạng** | `user_id` trong bảng users |
| **Partition key + Sort key** | **Tổ hợp** phải duy nhất; dữ liệu **nhóm theo partition key** | `user_id` + `game_id` trong bảng users-games; sort key tốt: **timestamp** |

### Indexes

> 📖 **Định nghĩa:** Xem *LSI/GSI* ở bảng trên.

| | **LSI** | **GSI** |
|---|---|---|
| Partition key | **Giữ nguyên** | **Đổi** (và tùy chọn sort key) |
| Sort key | Chọn sort key **khác** | Tùy chọn |
| Khi tạo | **Chỉ lúc tạo bảng** | **Thêm sau** được |
| Capacity | Dùng chung bảng | **Riêng** (throttle GSI có thể throttle bảng) |

⚠️ Chỉ **query theo PK + sort key** trên bảng và index (≠ RDS – không query tùy ý theo cột). Truy vấn khác → **Scan** (đắt, chậm).

### Tính năng quan trọng

> 📖 **Định nghĩa:** **TTL (DynamoDB)** là thuộc tính chứa thời điểm (epoch) mà sau đó item **tự động bị xóa**, miễn phí.

- **TTL**: tự xóa item sau thời điểm epoch chỉ định (session, dữ liệu tạm)
- **DynamoDB Streams**: phản ứng thay đổi **real-time**; đọc bằng **Lambda**, EC2…; giữ **24 giờ**
  - Use case: đẩy sang **OpenSearch** để tìm kiếm, gửi email chào mừng, analytics
- **Global Tables**: replicate **active-active** nhiều region; **phải bật Streams**; dùng cho **độ trễ thấp toàn cầu & DR**
  - 🆕 Global Tables hỗ trợ **multi-Region strong consistency** (2025) — đối chiếu tài liệu
- Khác: **Export to S3** (không tốn RCU, dùng PITR), **Import from S3**, **PartiQL** (cú pháp SQL)

### Kinesis Data Streams for DynamoDB

> 📖 **Định nghĩa:** **Change Data Capture (CDC)** là ghi nhận thay đổi dữ liệu theo thời gian thực để đẩy sang hệ thống khác.

- Dùng **Kinesis Data Streams** bắt thay đổi item-level của bảng
- **Retention dài hơn** (tới 365 ngày, so với 24h của DynamoDB Streams), nhiều consumer hơn
- Pipeline: DynamoDB → **KDS** → **Firehose** → S3 / Redshift / OpenSearch; hoặc → Lambda / **Managed Flink** để tính toán real-time (lọc, tổng hợp, biến đổi)

### Kiến trúc: index metadata object S3

> 📖 **Định nghĩa:** Xem phần Storage – S3 không có index, DynamoDB đóng vai trò bảng tra cứu.

- S3 ghi object → event → **Lambda** → ghi metadata vào **DynamoDB**
- API tra cứu: theo ngày, tổng dung lượng của khách hàng, object theo thuộc tính, object trong khoảng thời gian

### DAX – DynamoDB Accelerator

> 📖 **Định nghĩa:** Xem *DAX* ở bảng trên.

- Cache **liền mạch**, **không viết lại ứng dụng** (API tương thích DynamoDB)
- **Ghi đi qua DAX** xuống DynamoDB (write-through)
- Độ trễ **micro giây** cho đọc & query đã cache
- Giải quyết **Hot Key** (quá nhiều đọc cùng key)
- TTL cache mặc định **5 phút**
- Tối đa **10 node** trong cluster
- **Multi-AZ** (khuyến nghị **≥ 3 node** cho production)
- Bảo mật: KMS at rest, VPC, IAM, CloudTrail

### DAX vs ElastiCache

> 📖 **Định nghĩa:** **Aggregation result** là kết quả tính toán tổng hợp từ nhiều item (tổng, đếm, báo cáo).

| **DAX** | **ElastiCache** |
|---|---|
| Cache **từng object** (item cache) và **kết quả Query/Scan** | Lưu **kết quả tổng hợp/tính toán** của ứng dụng |
| Đứng giữa app và DynamoDB, **không sửa code** | App tự quản lý cache (sửa code) |
| **Chỉ cho DynamoDB** | Dùng cho mọi nguồn dữ liệu |

## 🎯 Mẹo thi

- "Key-value/NoSQL, serverless, quy mô lớn, độ trễ ms" → **DynamoDB**
- "Tải không đoán được" → **on-demand**; "ổn định, tối ưu chi phí" → **provisioned + auto scaling**
- "Truy vấn theo thuộc tính khác partition key, thêm sau" → **GSI**
- "Tự xóa session hết hạn" → **TTL**
- "Phản ứng khi item thay đổi" → **DynamoDB Streams + Lambda**
- "Giữ change log > 24h, nhiều consumer" → **Kinesis Data Streams for DynamoDB**
- "Tìm kiếm full-text trên dữ liệu DynamoDB" → **Streams → Lambda → OpenSearch**
- "Ứng dụng toàn cầu ghi ở nhiều region" → **Global Tables**
- "Đọc micro giây, hot key, không sửa code" → **DAX**
- "Item > 400 KB" → lưu object vào **S3**, metadata/pointer trong DynamoDB
