# AWS Outposts, Wavelength & Local Zones

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Hybrid cloud** | Mô hình doanh nghiệp giữ hạ tầng on-prem song song với hạ tầng cloud. |
| **AWS Outposts** | **Tủ rack (hoặc server) phần cứng AWS** lắp đặt tại data center của khách hàng, chạy cùng hạ tầng, dịch vụ, API và công cụ AWS; AWS lắp đặt và quản lý. |
| **AWS Wavelength** | Hạ tầng AWS đặt **bên trong data center của nhà mạng viễn thông**, tại biên mạng **5G**. |
| **AWS Local Zones** | Phần mở rộng của một AWS Region, đặt compute/storage/DB **gần người dùng cuối** ở các thành phố lớn. |
| **Carrier Gateway** | Gateway kết nối subnet trong Wavelength Zone với mạng của nhà mạng. |

## Nội dung chính

### AWS Outposts

> 📖 **Định nghĩa:** **Data residency** là yêu cầu dữ liệu phải lưu trữ trong một vị trí địa lý/pháp lý cụ thể.

- Hybrid cloud thường phải vận hành **2 cách** (AWS console/API vs on-prem) → Outposts **thống nhất** về một cách
- AWS **setup và quản lý** Outposts Rack trong data center của bạn
- **Bạn chịu trách nhiệm an ninh vật lý** của rack
- **Lợi ích**: độ trễ thấp tới hệ thống on-prem, **xử lý dữ liệu local**, **data residency**, migrate dễ hơn, fully managed
- **Dịch vụ chạy trên Outposts**: EC2, EBS, S3, EKS, ECS, RDS, EMR…

### S3 on Outposts

> 📖 **Định nghĩa:** **S3 Outposts storage class** là lớp lưu trữ S3 đặt dữ liệu trên Outposts.

- Dùng **S3 API** lưu/đọc dữ liệu **local** trên Outposts
- Giữ dữ liệu gần ứng dụng on-prem, **giảm truyền dữ liệu** về region
- Storage class **S3 Outposts**; mã hóa mặc định **SSE-S3**
- Truy cập qua **S3 Access Point** từ VPC; đồng bộ về S3 region bằng **DataSync**

### AWS Wavelength

> 📖 **Định nghĩa:** **CSP (Communication Service Provider)** là nhà cung cấp dịch vụ viễn thông.

- Hạ tầng nhúng trong data center nhà mạng tại **biên 5G**
- Dịch vụ: EC2, EBS, VPC…
- **Độ trễ cực thấp qua 5G**; traffic **không rời mạng nhà mạng**
- Kết nối băng thông cao, an toàn tới **region cha**
- **Không phí/hợp đồng bổ sung**
- Use case: smart city, chẩn đoán có ML, **xe kết nối**, **live video tương tác**, **AR/VR**, **game real-time**

### AWS Local Zones

> 📖 **Định nghĩa:** Xem bảng trên.

- Đặt compute, storage, DB… **gần user cuối** cho ứng dụng nhạy độ trễ
- **Mở rộng VPC** sang vị trí mới ("extension of an AWS Region") — tạo subnet trong Local Zone
- Hỗ trợ: EC2, RDS, ECS, EBS, ElastiCache, Direct Connect…
- Ví dụ: region us-east-1 → Local Zone Boston, Chicago, Dallas, Houston, Miami…

### So sánh nhanh

> 📖 **Định nghĩa:** **Edge computing** là xử lý dữ liệu gần nơi phát sinh/người dùng thay vì ở trung tâm.

| | **Outposts** | **Local Zones** | **Wavelength** |
|---|---|---|---|
| Đặt ở | **Data center của bạn** | Thành phố lớn (AWS vận hành) | **Data center nhà mạng 5G** |
| Mục tiêu | Hybrid, data residency, độ trễ tới on-prem | Độ trễ thấp cho user ở thành phố | Độ trễ cực thấp cho thiết bị di động 5G |
| An ninh vật lý | **Bạn** | AWS | AWS/nhà mạng |

## 🎯 Mẹo thi

- "Dữ liệu phải ở lại data center công ty nhưng dùng API AWS" → **Outposts**
- "Ứng dụng AR/game cho thiết bị di động 5G, độ trễ ms" → **Wavelength**
- "Render/game cho user ở Los Angeles, độ trễ thấp hơn region" → **Local Zones**
- "S3 API on-prem, đồng bộ về region" → **S3 on Outposts + DataSync**
