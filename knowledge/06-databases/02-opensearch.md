# Amazon OpenSearch Service

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Amazon OpenSearch Service** | Dịch vụ managed cho **OpenSearch** (fork mã nguồn mở của Elasticsearch) – công cụ **tìm kiếm & phân tích** dữ liệu, đặc biệt **full-text search** và log analytics. |
| **OpenSearch Dashboards** | Công cụ dashboard/trực quan hóa trên dữ liệu OpenSearch (trước gọi là **Kibana**). |
| **Logstash** | Công cụ **thu thập & đưa log vào** OpenSearch (dùng Logstash agent). |
| **Index (OpenSearch)** | Tập document đã được đánh chỉ mục để tìm kiếm nhanh. |
| **Full-text search** | Tìm kiếm theo nội dung văn bản (từ khóa, độ liên quan), không chỉ khớp chính xác. |
| **Subscription filter** | Bộ lọc của CloudWatch Logs để **stream log** tới Lambda, Kinesis, Firehose hoặc OpenSearch. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** **Managed cluster** là cụm node bạn chọn cấu hình; **Serverless collection** là chế độ tự scale, không quản lý node.

- Tên mới: Elasticsearch → **OpenSearch**, Kibana → **OpenSearch Dashboards**
- Hai chế độ: **Managed cluster** hoặc **Serverless**
- Use case: **log analytics**, giám sát ứng dụng real-time, **security analytics**, **full-text search**, **clickstream analytics**, indexing
- 🆕 **Vector engine**: lưu & tìm kiếm **vector embedding** → làm **vector store cho RAG** (Bedrock Knowledge Bases)

### OpenSearch + Dashboards + Logstash

> 📖 **Định nghĩa:** Xem bảng trên.

| Thành phần | Vai trò | Thay thế cho |
|---|---|---|
| **OpenSearch** | Tìm kiếm & đánh chỉ mục | – |
| **OpenSearch Dashboards** | Dashboard real-time trên dữ liệu OpenSearch | **CloudWatch Dashboards** (mạnh hơn) |
| **Logstash** | Thu thập log bằng agent | **CloudWatch Logs** (bạn tự quyết retention, độ chi tiết) |

### Các mẫu kiến trúc

> 📖 **Định nghĩa:** **Real time** là xử lý từng sự kiện ngay; **near real time** là xử lý theo lô nhỏ có độ trễ vài giây – phút.

**DynamoDB + OpenSearch (tìm kiếm item):**
- CRUD → **DynamoDB Table** → **DynamoDB Stream** → **Lambda** → **OpenSearch**
- API **tìm kiếm** trên OpenSearch → lấy ID → API **đọc item đầy đủ** từ DynamoDB
- 🆕 Có **zero-ETL integration** DynamoDB → OpenSearch (không cần tự viết Lambda)

**CloudWatch Logs → OpenSearch:**
- **Real time:** CloudWatch Logs → **Subscription Filter** → **Lambda** (AWS quản lý) → OpenSearch
- **Near real time:** CloudWatch Logs → **Subscription Filter** → **Kinesis Data Firehose** → OpenSearch

## 🎯 Mẹo thi

- "Tìm kiếm full-text / tìm theo một phần tên" → **OpenSearch** (không phải DynamoDB/RDS)
- "Tìm kiếm trên dữ liệu DynamoDB" → **Streams → Lambda → OpenSearch** (hoặc zero-ETL)
- "Phân tích log nâng cao, dashboard mạnh hơn CloudWatch" → **OpenSearch + Dashboards**
- "Đưa CloudWatch Logs vào OpenSearch gần real-time, ít vận hành" → **Subscription filter → Firehose**
- "Vector store cho RAG" → **OpenSearch Serverless (vector)** 🆕
