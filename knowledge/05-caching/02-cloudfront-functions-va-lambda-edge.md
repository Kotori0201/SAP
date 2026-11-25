# CloudFront Functions & Lambda@Edge

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Edge function** | Đoạn code gắn vào CloudFront distribution, chạy **gần người dùng** để thay đổi request/response; **không có cache riêng**. |
| **CloudFront Functions** | Function **JavaScript siêu nhẹ** chạy tại **edge location**, thời gian khởi động dưới 1 ms, hàng triệu request/giây. |
| **Lambda@Edge** | Lambda function (**Node.js, Python**) chạy tại **Regional Edge Cache**, mạnh hơn (truy cập mạng, body request, thời gian lâu hơn). |
| **Viewer request / response** | Sự kiện khi CloudFront **nhận request từ user** / **trước khi trả response cho user**. |
| **Origin request / response** | Sự kiện **trước khi CloudFront gửi request tới origin** / **sau khi nhận response từ origin** (chỉ chạy khi cache miss). |
| **Cache key normalization** | Chuẩn hóa header/cookie/query string để nhiều request tương đương dùng chung một cache key → tăng cache hit. |

## Nội dung chính

### Tùy biến tại edge

> 📖 **Định nghĩa:** **A/B testing** là chia user thành nhóm để thử phiên bản khác nhau và so sánh kết quả.

- Use case: thao tác HTTP request/response, **lọc request** trước khi tới ứng dụng, **xác thực & phân quyền** user, **sinh response tại edge**, **A/B testing**, **chặn bot** tại edge
- Không quản lý server, **triển khai toàn cầu**

### CloudFront Functions

> 📖 **Định nghĩa:** **Process-based isolation** là cô lập các function bằng tiến trình (nhẹ, nhanh hơn VM).

- **JavaScript**, nhẹ, cho tùy biến CDN **quy mô lớn, nhạy độ trễ**
- Khởi động **< 1 ms**, **hàng triệu request/giây**
- Chạy tại **edge location**, process-based isolation
- Chỉ đổi **viewer request** và **viewer response**
- Tính năng **native** của CloudFront (quản lý code trong CloudFront)
- 🆕 **CloudFront KeyValueStore**: kho key-value đọc được từ CloudFront Functions (vd bảng redirect, feature flag)

### Lambda@Edge

> 📖 **Định nghĩa:** **VM-based isolation** là cô lập bằng máy ảo (nặng hơn nhưng mạnh hơn).

- **Node.js hoặc Python**, scale **hàng nghìn request/giây**
- Chạy tại **Regional Edge Cache** gần nhất, VM-based isolation
- Đổi được **cả 4 sự kiện**: viewer request, **origin request**, **origin response**, viewer response
- Viết function ở **một region (us-east-1)**, CloudFront **replicate** ra các vị trí

### Kết hợp hai loại

> 📖 **Định nghĩa:** **Trigger** là sự kiện CloudFront gọi function.

- Dùng **cùng lúc** được: CloudFront Functions cho **viewer request/response**, Lambda@Edge cho **origin request/response**
- ⚠️ **Không kết hợp** CloudFront Functions và Lambda@Edge trên **cùng một viewer event**
- Chỉ dùng Lambda@Edge khi cần khả năng CloudFront Functions không có (thời gian chạy lâu, truy cập mạng…)

### So sánh

> 📖 **Định nghĩa:** Xem bảng định nghĩa ở trên.

| | **CloudFront Functions** | **Lambda@Edge** |
|---|---|---|
| Runtime | JavaScript | Node.js, Python |
| Chạy ở | **Edge Locations** | **Regional Edge Caches** |
| Trigger | Viewer Request/Response | Viewer **và Origin** Request/Response |
| Isolation | Process-based | VM-based |
| Thời gian tối đa | **< 1 ms** | **5 s** (viewer), **30 s** (origin) |
| Bộ nhớ tối đa | **2 MB** | 128 MB (viewer), **10 GB** (origin) |
| Kích thước package | **10 KB** | 1 MB (viewer), 50 MB (origin) |
| Truy cập mạng, file system | ❌ | ✅ |
| Đọc request body | ❌ | ✅ |
| Giá | Có free tier, **~1/6 giá Lambda@Edge** | Không free tier, tính theo request & duration |

### Use cases

> 📖 **Định nghĩa:** **JWT (JSON Web Token)** là token chuẩn chứa claim có chữ ký, dùng để xác thực stateless.

| **CloudFront Functions** | **Lambda@Edge** |
|---|---|
| **Cache key normalization** (biến đổi header, cookie, query string, URL) | Thời gian chạy dài hơn (vài ms trở lên) |
| **Thao tác header** (thêm/sửa/xóa) | Điều chỉnh CPU/RAM |
| **URL rewrite / redirect** | Code phụ thuộc **thư viện bên thứ 3** (vd AWS SDK gọi dịch vụ khác) |
| **Xác thực request**: tạo & **kiểm tra JWT** | **Truy cập mạng** tới dịch vụ ngoài (vd gọi **Cognito / OIDC** bên thứ 3) |
| | Truy cập **file system** hoặc **body** request |

### Các kiến trúc mẫu

> 📖 **Định nghĩa:** **Device-based content** là trả nội dung khác nhau theo loại thiết bị (User-Agent).

- **Ảnh theo thiết bị (CloudFront Functions):** đọc header `User-Agent` → rewrite `cat-1920-1080.jpg` thành `cat-640-320.jpg` cho iPhone
- **Ứng dụng toàn cầu (Lambda@Edge):** website tĩnh trên S3 + request API động → Lambda@Edge **query DynamoDB** → cache response
- **Định tuyến sang origin khác (Lambda@Edge origin request):** user châu Âu → function đổi origin sang **bucket eu-west-2** (có S3 CRR từ us-east-1)

## 🎯 Mẹo thi

- "Rewrite URL / thêm security header / redirect, hàng triệu rps, rẻ" → **CloudFront Functions**
- "Kiểm tra JWT tại edge" → **CloudFront Functions** (không cần gọi mạng)
- "Xác thực bằng cách gọi Cognito/OIDC bên ngoài" → **Lambda@Edge**
- "Đổi origin theo vị trí user / truy vấn DynamoDB ở edge" → **Lambda@Edge (origin request)**
- "Cần đọc body request" → **Lambda@Edge**
- "Tăng cache hit bằng cách chuẩn hóa query string" → **CloudFront Functions** (cache key normalization)
