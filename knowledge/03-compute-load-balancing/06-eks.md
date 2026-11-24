# Amazon EKS, ECS Anywhere & EKS Anywhere

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Kubernetes (K8s)** | Hệ thống **mã nguồn mở** tự động triển khai, scale và quản lý ứng dụng container; **cloud-agnostic**. |
| **Amazon EKS** | *Elastic Kubernetes Service* – Kubernetes **control plane do AWS quản lý**. |
| **Pod** | Đơn vị triển khai nhỏ nhất của Kubernetes, gồm 1+ container dùng chung mạng/lưu trữ. |
| **Worker node** | Máy (EC2 hoặc Fargate) chạy pod. |
| **Managed node group** | Nhóm node EC2 do EKS tạo và quản lý (ASG do EKS điều khiển). |
| **CSI driver** | *Container Storage Interface* – chuẩn plugin để Kubernetes gắn volume lưu trữ (EBS, EFS, FSx). |
| **IRSA / EKS Pod Identity** | Cơ chế gán **IAM role cho từng pod** (IRSA dùng OIDC; Pod Identity là cách mới, đơn giản hơn). |
| **ECS Anywhere / EKS Anywhere** | Chạy ECS task / Kubernetes cluster trên **hạ tầng của khách hàng** (on-prem, VM). |

## Nội dung chính

### Tổng quan EKS

> 📖 **Định nghĩa:** **Control plane** là thành phần điều khiển cluster (API server, etcd, scheduler); **data plane** là các node chạy workload.

- Cách chạy **Kubernetes managed** trên AWS — thay thế ECS với **API khác**
- Worker: **EC2** hoặc **Fargate** (serverless)
- Use case: công ty **đã dùng Kubernetes** on-prem hoặc cloud khác, muốn migrate sang AWS mà giữ Kubernetes
- **Nhiều region → mỗi region một EKS cluster**
- Log & metric: **CloudWatch Container Insights**
- Kiến trúc: node ở **private subnet** (ASG trải nhiều AZ), **ELB public** cho service public, **ELB private** cho service nội bộ, NAT Gateway cho outbound

### Loại node

> 📖 **Định nghĩa:** Xem bảng trên.

| Loại | Ai quản lý | Ghi chú |
|---|---|---|
| **Managed Node Groups** | EKS tạo & quản lý EC2 (ASG của EKS) | On-Demand hoặc Spot |
| **Self-Managed Nodes** | Bạn tạo, đăng ký vào cluster, ASG của bạn | Dùng **EKS Optimized AMI**; On-Demand hoặc Spot; tùy biến tối đa |
| **AWS Fargate** | AWS | Không có node để bảo trì |
| 🆕 **EKS Auto Mode** | AWS quản lý cả node, scaling (Karpenter), add-on | Ít vận hành nhất trên EC2 |

> 💡 **Karpenter**: autoscaler node mã nguồn mở, chọn instance type phù hợp nhanh hơn Cluster Autoscaler.

### Data Volumes

> 📖 **Định nghĩa:** **StorageClass** là manifest Kubernetes mô tả loại lưu trữ được cấp phát động.

- Khai báo **StorageClass** + driver chuẩn **CSI**
- Hỗ trợ: **EBS**, **EFS** (**chạy được với Fargate**), **FSx for Lustre**, **FSx for NetApp ONTAP**

### ECS Anywhere

> 📖 **Định nghĩa:** **EXTERNAL launch type** là kiểu chạy task ECS trên server bên ngoài AWS đã đăng ký vào cluster.

- Chạy task ECS trên hạ tầng khách hàng (on-prem, VM)
- **Control plane ECS do AWS quản lý**
- Cài **ECS Container Agent** + **SSM Agent** trên server
- Launch type **"EXTERNAL"**
- **Cần kết nối ổn định** tới AWS Region
- Use case: tuân thủ, quy định, độ trễ; chạy gần hệ thống khác on-prem; ML, xử lý video/dữ liệu on-prem

### EKS Anywhere

> 📖 **Định nghĩa:** **EKS Distro** là bản phân phối Kubernetes do AWS đóng gói (cùng bản EKS dùng).

- Tạo & vận hành cluster Kubernetes **ngoài AWS**, dùng **EKS Distro**
- Cài bằng **EKS Anywhere installer**; tùy chọn **EKS Connector** để kết nối về console EKS
- **Fully connected / partially disconnected**: kết nối về AWS, dùng EKS console
- **Fully disconnected**: tự dùng EKS Distro + công cụ mã nguồn mở để quản lý
- ⚠️ Khác ECS Anywhere: **control plane chạy on-prem** → **không bắt buộc kết nối AWS**

## 🎯 Mẹo thi

- "Đang dùng Kubernetes, muốn chuyển lên AWS" → **EKS**
- "Pod cần quyền IAM riêng" → **IRSA / EKS Pod Identity**
- "Volume chia sẻ cho pod trên Fargate" → **EFS**
- "Chạy container on-prem, quản lý từ ECS console" → **ECS Anywhere** (cần kết nối AWS)
- "Kubernetes on-prem, có thể **mất kết nối** hoàn toàn" → **EKS Anywhere**
- "Ít vận hành node nhất trên EKS" → **Fargate** hoặc **EKS Auto Mode**
