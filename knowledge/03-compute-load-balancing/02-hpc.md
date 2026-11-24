# High Performance Computing (HPC) trên AWS

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **HPC (High Performance Computing)** | Tính toán hiệu năng cao: dùng rất nhiều tài nguyên tính toán song song để giải bài toán lớn (gen học, mô phỏng hóa học, mô hình rủi ro tài chính, dự báo thời tiết, ML/DL, xe tự hành). |
| **Tightly coupled workload** | Workload mà các node phải **liên lạc liên tục** với nhau trong quá trình tính toán (vd mô phỏng dùng MPI). |
| **Loosely coupled workload** | Workload chia thành nhiều tác vụ **độc lập**, ít giao tiếp (vd render từng frame). |
| **Enhanced Networking (SR-IOV)** | Công nghệ ảo hóa mạng cho băng thông cao, PPS cao, độ trễ thấp. |
| **ENA (Elastic Network Adapter)** | Card mạng ảo hỗ trợ Enhanced Networking tới **100 Gbps** (và hơn trên instance mới). |
| **EFA (Elastic Fabric Adapter)** | ENA nâng cấp cho HPC, **bỏ qua kernel OS** (OS-bypass) để đạt độ trễ cực thấp, hỗ trợ **MPI**. |
| **MPI (Message Passing Interface)** | Chuẩn trao đổi thông điệp giữa các tiến trình trên nhiều node trong tính toán song song. |

## Nội dung chính

### Vì sao HPC phù hợp với cloud

> 📖 **Định nghĩa:** **Time to results** là thời gian từ lúc bắt đầu đến lúc có kết quả tính toán.

- Tạo **rất nhiều tài nguyên** trong thời gian ngắn
- Thêm tài nguyên để **rút ngắn time to results**
- Chỉ trả tiền cho phần đã dùng

### Chuyển dữ liệu

> 📖 **Định nghĩa:** **Data transfer service** là dịch vụ đưa dữ liệu từ on-prem lên AWS (qua mạng hoặc thiết bị vật lý).

- **Direct Connect**: chuyển GB/s qua mạng riêng an toàn
- **Snowball**: chuyển **PB** dữ liệu bằng thiết bị vật lý
- **DataSync**: chuyển lượng lớn dữ liệu giữa on-prem và S3/EFS/FSx

### Compute & Networking

> 📖 **Định nghĩa:** **PPS (packets per second)** là số gói tin mạng xử lý được mỗi giây.

- EC2 **tối ưu CPU / GPU**; **Spot / Spot Fleet** + Auto Scaling để tiết kiệm
- **Cluster placement group** cho mạng tốt nhất
- **Enhanced Networking**: ENA (tới 100 Gbps); Intel 82599 VF (10 Gbps – legacy)
- **EFA**: chỉ **Linux**, tốt cho giao tiếp **giữa các node**, workload **tightly coupled**, dùng **MPI**, bypass OS

> 🎯 "Workload MPI tightly coupled cần độ trễ thấp nhất" → **EFA + cluster placement group**

### Storage cho HPC

> 📖 **Định nghĩa:** **Amazon FSx for Lustre** là hệ thống file phân tán hiệu năng cao (Lustre) do AWS quản lý, tối ưu cho HPC, liên kết với S3.

| Loại | Dịch vụ | Đặc điểm |
|---|---|---|
| Gắn với instance | **EBS** | io2 Block Express tới **256.000 IOPS** |
| | **Instance Store** | **Hàng triệu IOPS**, độ trễ thấp, mất dữ liệu khi stop |
| Lưu trữ mạng | **S3** | Blob lớn, không phải file system |
| | **EFS** | IOPS tăng theo dung lượng hoặc provisioned/elastic throughput |
| | **FSx for Lustre** | File system phân tán cho HPC, **hàng triệu IOPS**, **backed by S3** |

### Automation & Orchestration

> 📖 **Định nghĩa:** **AWS Batch** là dịch vụ managed chạy batch job: tự lập lịch job và khởi chạy EC2/Fargate tương ứng. **AWS ParallelCluster** là công cụ **mã nguồn mở** để triển khai cluster HPC trên AWS.

- **AWS Batch**: hỗ trợ **multi-node parallel jobs** (1 job chạy trên nhiều EC2); tự lập lịch và scale
- **AWS ParallelCluster**: cấu hình bằng file text, tự tạo VPC, subnet, loại cluster, loại instance; hỗ trợ scheduler như Slurm; **bật EFA được**

## 🎯 Mẹo thi

- "Mô phỏng MPI, độ trễ giữa node thấp nhất" → **EFA** + **cluster placement group**
- "File system cực nhanh cho HPC, dữ liệu gốc ở S3" → **FSx for Lustre**
- "Chạy hàng nghìn batch job, tự quản lý hạ tầng tối thiểu" → **AWS Batch** (+ Spot)
- "Dựng cluster HPC kiểu truyền thống (Slurm) bằng file cấu hình" → **ParallelCluster**
