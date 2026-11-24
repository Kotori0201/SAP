# Amazon API Gateway

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Amazon API Gateway** | Dịch vụ **serverless** để tạo, công bố, bảo mật, giám sát API ở quy mô lớn, đứng trước Lambda, HTTP backend hoặc AWS service. |
| **REST API / HTTP API / WebSocket API** | Ba loại API của API Gateway: REST (đầy đủ tính năng), HTTP (đơn giản, rẻ, nhanh hơn), WebSocket (kết nối hai chiều lâu dài). |
| **Stage** | Bản triển khai có tên của API (dev, test, prod), có URL và cấu hình riêng. |
| **Integration** | Backend mà API Gateway chuyển request tới (Lambda, HTTP, AWS service, mock). |
| **Authorizer** | Cơ chế xác thực request: IAM, Lambda authorizer, Cognito User Pool (JWT). |
| **Usage plan / API key** | Usage plan định nghĩa ai dùng được API, **bao nhiêu và nhanh cỡ nào** (throttle, quota); API key định danh khách hàng. |
| **CORS** | *Cross-Origin Resource Sharing* – cơ chế trình duyệt kiểm soát domain nào được gọi API. |
| **Throttling** | Giới hạn tốc độ request (rate + burst); vượt → lỗi **429**. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** **OpenAPI (Swagger)** là chuẩn mô tả API dạng YAML/JSON, import/export được vào API Gateway.

- Expose **Lambda, HTTP, AWS service** thành API
- Versioning, authorization, quản lý traffic (API key, throttle), scale lớn, serverless, biến đổi request/response, **OpenAPI**, **CORS**
- **Giới hạn:**
  - Timeout **29 giây** (🆕 REST API regional/private **tăng được** qua quota, đổi lại giảm throttle)
  - Payload tối đa **10 MB**

### Deployment Stages

> 📖 **Định nghĩa:** **Canary release (stage)** là tính năng chuyển một % traffic của stage sang bản deploy mới.

- Thay đổi phải **deploy vào stage** mới có hiệu lực
- Đặt tên stage tùy ý; giữ **lịch sử deployment** → rollback được
- Stage variables + Lambda alias: vd Prod stage 95% → V1, 5% → V2

### Integrations

> 📖 **Định nghĩa:** **Proxy integration** chuyển nguyên request tới backend; **non-proxy** cho phép mapping template biến đổi request/response.

| Loại | Ví dụ | Vì sao dùng |
|---|---|---|
| **HTTP** | API nội bộ on-prem, ALB | Thêm rate limit, cache, xác thực, API key |
| **Lambda** | Gọi function | Cách dễ nhất để có REST API serverless |
| **AWS Service** | Start Step Functions, gửi message SQS, Kinesis PutRecord | Thêm xác thực, public an toàn, kiểm soát tốc độ — **không cần Lambda** |

### Kiến trúc: API Gateway trước S3

> 📖 **Định nghĩa:** **Pre-signed URL** là URL S3 có chữ ký tạm thời cho phép upload/download trực tiếp.

- Proxy upload file qua API Gateway → **vướng giới hạn 10 MB**
- **Kiến trúc tốt hơn:** client gọi API → Lambda **sinh pre-signed URL** → client **upload thẳng lên S3**

### Endpoint Types

> 📖 **Định nghĩa:** **Edge-optimized** đi qua CloudFront edge; **Regional** cho client cùng region; **Private** chỉ truy cập từ VPC qua interface endpoint.

| Loại | Đặc điểm |
|---|---|
| **Edge-Optimized** (mặc định) | Cho **client toàn cầu**; request đi qua **CloudFront edge**; API vẫn ở **1 region** |
| **Regional** | Client **cùng region**; có thể tự ghép **CloudFront** để kiểm soát cache/distribution tốt hơn |
| **Private** | Chỉ truy cập **từ VPC** qua **interface VPC endpoint (ENI)**; dùng **resource policy** kiểm soát |

### Caching

> 📖 **Định nghĩa:** **API cache** lưu response tại API Gateway để giảm số lần gọi backend.

- TTL mặc định **300 giây** (0 – 3600s)
- Cache **theo stage**, override được **theo method**
- Client invalidate bằng header `Cache-Control: max-age=0` (cần quyền IAM)
- Flush toàn bộ cache ngay lập tức
- Mã hóa cache; dung lượng **0,5 GB – 237 GB**

### Lỗi thường gặp

> 📖 **Định nghĩa:** **4xx** là lỗi phía client; **5xx** là lỗi phía server.

| Mã | Ý nghĩa |
|---|---|
| 400 | Bad Request |
| **403** | Access Denied, **bị WAF lọc** |
| **429** | **Quota exceeded / Throttle** |
| **502** | Bad Gateway – output **không tương thích** từ Lambda proxy, đôi khi do tải cao |
| 503 | Service Unavailable |
| **504** | Integration failure – **Endpoint Request Timed-out** (quá **29 giây**) |

### Bảo mật

> 📖 **Định nghĩa:** **Resource policy (API Gateway)** là policy JSON gắn vào API, quy định account, IP/CIDR, VPC hoặc VPC endpoint nào được gọi.

- Custom domain: cert (ACM) + Route 53 (CNAME/Alias)
- **Resource policy**: kiểm soát theo AWS account, IP/CIDR, **VPC / VPC endpoint**; **cross-account**
- **IAM execution role** để API Gateway gọi Lambda/AWS service
- **CORS**: kiểm soát domain trình duyệt được phép gọi
- Kết hợp **WAF** (REST API)

### Authentication

> 📖 **Định nghĩa:** **SigV4** là cơ chế ký request bằng IAM credential; **Lambda authorizer** là function tự viết trả về IAM policy cho request; **Cognito User Pool authorizer** xác minh JWT do Cognito cấp.

| Cách | Khi nào |
|---|---|
| **IAM (AWS_IAM)** | Truy cập **nội bộ hạ tầng AWS** (user/role); ký **SigV4** |
| **Lambda Authorizer** | Xác thực **OAuth / SAML / bên thứ 3** tùy biến; kết quả cache được |
| **Cognito User Pools** | User đăng nhập Cognito → gửi token → API Gateway **tự xác minh** (không cần code) |

> 💡 HTTP API hỗ trợ **JWT authorizer** (OIDC/OAuth2 bất kỳ) native.

### Logging, Monitoring, Tracing

> 📖 **Định nghĩa:** **Access log** ghi lại ai gọi API, khi nào, kết quả; **execution log** ghi chi tiết quá trình xử lý.

- **CloudWatch Logs** ở mức stage (ERROR/INFO), log full request/response, **access logs** tùy biến
- Gửi log thẳng tới **Kinesis Data Firehose**
- **CloudWatch Metrics** theo stage: `IntegrationLatency`, `Latency`, `CacheHitCount`, `CacheMissCount`
- **X-Ray**: tracing API Gateway + Lambda → bức tranh đầy đủ

### Usage Plans & API Keys

> 📖 **Định nghĩa:** Xem bảng trên. **Quota** là tổng số request tối đa trong một khoảng (ngày/tuần/tháng).

- Bán API cho khách hàng ($)
- **Usage plan**: ai truy cập stage/method nào, **bao nhiêu & nhanh cỡ nào**, dùng **API key** để định danh & đo
- **Throttling** áp lên API key; **quota** là tổng số request tối đa
- **429 Too Many Requests**: throttle mức account trên mọi API trong region (mặc định 10.000 rps, burst 5.000) → client phải **retry (exponential backoff)**

### WebSocket API

> 📖 **Định nghĩa:** **WebSocket** là giao thức giao tiếp **hai chiều, kết nối lâu dài** giữa trình duyệt và server; server **chủ động đẩy** dữ liệu cho client.

- Ứng dụng **stateful, real-time**: chat, cộng tác, game nhiều người chơi, giao dịch tài chính
- Route: `$connect` (onConnect), `sendMessage`, `$disconnect` → Lambda; lưu `connectionId` vào **DynamoDB**
- URL: `wss://abcdef.execute-api.<region>.amazonaws.com/dev`
- **Server → client**: backend gọi **HTTP POST (IAM SigV4)** tới **Connection URL** `.../@connections/{connectionId}` (POST gửi tin, GET lấy trạng thái, DELETE ngắt kết nối)

### Private APIs

> 📖 **Định nghĩa:** **Interface VPC endpoint** (`execute-api`) là ENI trong subnet cho phép truy cập API Gateway private.

- Chỉ truy cập **từ VPC qua interface endpoint**
- **Một endpoint dùng cho nhiều Private API**
- Resource policy cho phép/chặn theo VPC / VPC endpoint, **kể cả cross-account** — condition `aws:SourceVpc`, `aws:SourceVpce`
- Endpoint policy kiểm soát thêm ở phía VPC

## 🎯 Mẹo thi

- "Upload file lớn qua API" → **pre-signed URL S3**, không proxy qua API Gateway
- "Lỗi 504 sau ~29s" → backend quá chậm → xử lý **async** (SQS/Step Functions) và trả ngay
- "Lỗi 429" → throttle → **usage plan / tăng quota / retry**
- "Bán API theo gói cho khách" → **Usage plans + API keys**
- "Chat real-time, server đẩy tin" → **WebSocket API** + DynamoDB lưu connectionId
- "API chỉ truy cập từ VPC (kể cả account khác)" → **Private API** + resource policy `aws:SourceVpce`
- "Ghi vào SQS/Kinesis qua REST không cần Lambda" → **AWS Service integration**
- "Xác thực bằng IdP bên thứ 3 tùy biến" → **Lambda authorizer**
