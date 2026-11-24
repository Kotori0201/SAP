# AWS Resource Access Manager (RAM)

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **AWS RAM** | Dịch vụ chia sẻ resource bạn sở hữu cho account khác, OU hoặc cả Organization mà không cần sao chép. |
| **Resource share** | Đối tượng RAM gồm danh sách resource được chia sẻ và danh sách principal nhận. |
| **Owner / Participant** | Owner: account sở hữu resource gốc. Participant: account nhận chia sẻ và dùng resource. |

## Nội dung chính

- **Chia sẻ resource bạn sở hữu** với account khác (bất kỳ account, hoặc trong org) → **tránh trùng lặp**
- Resource chia sẻ được (tiêu biểu): **VPC subnets**, **Transit Gateway**, **Route 53 Resolver rules** & DNS Firewall rule groups, **managed prefix lists**, License Manager configurations, Aurora DB clusters, **ACM Private CA**, CodeBuild projects, EC2 Dedicated Hosts & Capacity Reservations, Glue catalog/database/table, Network Firewall policies, Resource Groups, Incident Manager, Outposts, **VPC IPAM pools**, **VPC Lattice service networks**…

### VPC Sharing (chia sẻ subnet)

> 📖 **Định nghĩa:** **VPC sharing** là việc chia sẻ subnet của một VPC (do **owner account** sở hữu) cho các **participant account** trong cùng Organization để họ đặt resource vào đó.

- **Chỉ trong cùng AWS Organization**
- **Không share** default VPC & default security group
- **Participant** tạo & quản lý resource của mình trong subnet được share; **không xem/sửa/xóa** resource của owner hoặc participant khác
- Mạng dùng chung → giao tiếp bằng **private IP**, không cần peering/TGW; **tham chiếu security group** của account khác
- Use case: app cùng trust boundary, **liên kết chặt** (high interconnectivity); giảm số VPC & CIDR phải quản lý
- Owner chịu trách nhiệm VPC/subnet/route table/NACL; participant trả tiền resource của mình

### Managed Prefix List

> 📖 **Định nghĩa:** **Prefix list** là tập hợp có tên gồm một hoặc nhiều dải CIDR, được tham chiếu trong security group và route table như một đối tượng duy nhất.

- Tập các CIDR để dùng trong **security group** và **route table**
- **Customer-managed**: tự định nghĩa, **share qua RAM**, sửa 1 chỗ → cập nhật mọi SG tham chiếu
- **AWS-managed**: CIDR của dịch vụ AWS (vd S3, CloudFront origin-facing) – không tạo/sửa/share được

### Route 53 Resolver rules

> 📖 **Định nghĩa:** **Route 53 Resolver** là DNS resolver mặc định của VPC. **Forwarding rule** chỉ định domain nào sẽ được chuyển tiếp tới DNS server khác (vd on-prem) thông qua **Outbound Endpoint**.

- Tạo **forwarding rule** (vd `corp.example.com` → DNS on-prem 172.16.0.10) + **Outbound Endpoint** ở 1 account
- **Share rule qua RAM** → account khác accept → **associate** rule với VPC của họ
- Scale DNS hybrid cho nhiều account/VPC mà không cần nhiều endpoint
