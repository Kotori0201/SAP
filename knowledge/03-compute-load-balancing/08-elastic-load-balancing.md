# Elastic Load Balancing (CLB, ALB, NLB, GWLB)

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Load balancer** | Thành phần phân phối traffic tới nhiều backend (target), kiểm tra sức khỏe và ẩn backend sau một endpoint duy nhất. |
| **Elastic Load Balancing (ELB)** | Họ load balancer managed của AWS: **CLB, ALB, NLB, GWLB**. |
| **Listener** | Tiến trình kiểm tra kết nối đến theo **protocol + port**, áp dụng rule để chuyển tới target group. |
| **Target group** | Nhóm target (EC2, IP, Lambda, ALB) cùng health check, nhận traffic từ listener rule. |
| **Cross-zone load balancing** | Mỗi node LB phân phối đều traffic tới **mọi target ở mọi AZ**, thay vì chỉ target trong AZ của nó. |
| **Sticky session (session affinity)** | Cùng một client luôn được chuyển tới cùng một target (bằng cookie). |
| **OSI Layer 4 / Layer 7** | Layer 4 = tầng vận chuyển (TCP/UDP); Layer 7 = tầng ứng dụng (HTTP). |

## Nội dung chính

### Các loại load balancer

> 📖 **Định nghĩa:** **Internal LB** chỉ có IP private (dùng trong VPC); **internet-facing LB** có endpoint public.

| LB | Năm | Layer | Protocol |
|---|---|---|---|
| **CLB** (v1, cũ) | 2009 | 4 & 7 | HTTP, HTTPS, TCP, SSL |
| **ALB** (v2) | 2016 | **7** | HTTP, HTTPS, **WebSocket**, HTTP/2, gRPC |
| **NLB** (v2) | 2017 | **4** | **TCP, TLS, UDP** |
| **GWLB** | 2020 | **3** | **IP** (GENEVE) |

→ Khuyến nghị dùng thế hệ mới; LB có thể **internal** hoặc **external**.

### Classic Load Balancer (v1)

> 📖 **Định nghĩa:** **SAN (Subject Alternative Name)** là danh sách tên miền bổ sung trong một chứng chỉ.

- Health check HTTP (L7) hoặc TCP (L4)
- **Chỉ 1 chứng chỉ SSL** (nhiều SAN nhưng phải thay cert mỗi lần đổi SAN) → nên dùng **ALB + SNI**
- Muốn nhiều cert riêng → nhiều CLB
- TCP → TCP: chuyển toàn bộ traffic tới EC2 → (theo slide) cách duy nhất cho **2-way SSL** — hiện nay **ALB đã hỗ trợ mTLS** 🆕 và NLB TCP passthrough cũng làm được

### Application Load Balancer (v2)

> 📖 **Định nghĩa:** **Routing rule** là điều kiện (path, host, header, query string, method, source IP) để chọn target group.

- **Layer 7**, load balance nhiều ứng dụng HTTP (nhiều target group), nhiều app trên **cùng máy** (container, **dynamic port mapping**)
- HTTP/2, **WebSocket**, **redirect** (HTTP → HTTPS)
- Rule theo **path** (`/user`, `/search`), **host header**, **query string**, **HTTP header**
- **Target**: EC2 (ASG), **ECS task**, **Lambda** (request → JSON event), **IP private**
- Health check **ở mức target group**
- 🆕 Tính năng thêm: **weighted target groups** (blue/green), **authentication** (Cognito/OIDC) ngay trên ALB, **mTLS**, WAF

### Network Load Balancer (v2)

> 📖 **Định nghĩa:** **Static IP per AZ** là mỗi AZ của NLB có một IP cố định, có thể gán **Elastic IP**.

- **Layer 4**: TCP & UDP, **hàng triệu request/giây**, độ trễ thấp (~100 ms vs ~400 ms của ALB)
- **Một IP tĩnh mỗi AZ**, hỗ trợ **Elastic IP** → tiện **whitelist IP**
- Dùng cho **hiệu năng cực cao, TCP/UDP**
- **Target**: EC2, **IP private** (kể cả on-prem qua VPN/DX), **ALB** (NLB đứng trước ALB để có IP tĩnh + routing L7)
- Giữ nguyên **IP client** (client IP preservation)
- 🆕 NLB đã hỗ trợ **Security Group**
- Không có trong free tier

**Zonal DNS name:**
- DNS regional trả về IP của **mọi node** ở mọi AZ
- Mỗi node có **DNS riêng theo AZ** (`us-east-1a.my-nlb-...`) → client chọn node cùng AZ để **giảm độ trễ và phí data cross-AZ** (cần logic riêng trong app)

### Gateway Load Balancer

> 📖 **Định nghĩa:** **Network virtual appliance** là phần mềm mạng chạy trên EC2 (firewall, IDS/IPS, deep packet inspection); **GENEVE** là giao thức đóng gói gói tin dùng **port 6081**.

- Triển khai, scale, quản lý fleet **appliance bên thứ 3** (firewall, **IDS/IPS**, **deep packet inspection**, chỉnh sửa payload)
- **Layer 3** (gói IP)
- Kết hợp: **Transparent Network Gateway** (một điểm vào/ra cho mọi traffic) + **Load Balancer** (phân phối tới appliance)
- Dùng **GENEVE port 6081**
- Route table của VPC đẩy traffic qua **GWLB endpoint** → appliance → quay lại → đích
- **Target**: EC2, IP private

### Cross-Zone Load Balancing

> 📖 **Định nghĩa:** Xem bảng trên. Ví dụ: AZ1 có 2 instance, AZ2 có 8 instance; **bật** → mỗi instance nhận 10%; **tắt** → instance AZ1 nhận 25%, AZ2 nhận 6,25%.

| LB | Mặc định | Phí data inter-AZ |
|---|---|---|
| **CLB** | Tắt | Không tính |
| **ALB** | **Luôn bật** (có thể tắt ở mức target group) | **Không tính** |
| **NLB** | Tắt | **Có tính** nếu bật |
| **GWLB** | Tắt | **Có tính** nếu bật |

### Sticky Sessions

> 📖 **Định nghĩa:** Xem bảng trên.

- Hoạt động với **CLB, ALB** (và NLB theo source IP)
- Cookie có **thời hạn** do bạn kiểm soát (duration-based hoặc application-based cookie)
- Use case: user **không mất session**
- ⚠️ Có thể gây **mất cân bằng tải** → tốt hơn: lưu session ra **ElastiCache/DynamoDB** (stateless)

### Thuật toán định tuyến request

> 📖 **Định nghĩa:** **Routing algorithm** là cách LB chọn target cho request/kết nối tiếp theo.

| Thuật toán | Cách chọn | LB |
|---|---|---|
| **Least Outstanding Requests** | Target có **ít request đang chờ** nhất | **ALB**, CLB (HTTP/HTTPS) |
| **Round Robin** | Chia đều lần lượt | **ALB**, CLB (TCP) |
| **Flow Hash** | Hash theo protocol, IP/port nguồn & đích, TCP sequence → **mỗi kết nối gắn 1 target suốt vòng đời** | **NLB** |
| 🆕 **Weighted random** (+ anomaly mitigation) | Ngẫu nhiên có trọng số, tự giảm traffic tới target bất thường | **ALB** |

## 🎯 Mẹo thi

- "Định tuyến theo path/host, microservices, container" → **ALB**
- "TCP/UDP, hiệu năng cực cao, **IP tĩnh/EIP** để whitelist" → **NLB**
- "Vừa cần IP tĩnh vừa cần routing L7" → **NLB → ALB** (ALB làm target của NLB) hoặc **Global Accelerator → ALB**
- "Đưa traffic qua firewall bên thứ 3 trong suốt" → **GWLB** (GENEVE 6081)
- "Gọi Lambda qua HTTP không cần tính năng API Gateway" → **ALB + Lambda**
- "Tải không đều giữa AZ" → bật **cross-zone** (NLB/GWLB tính phí)
- "Request xử lý lâu ngắn khác nhau, muốn tránh dồn" → **Least Outstanding Requests**
- "Nhiều domain HTTPS" → **ALB/NLB + SNI** (không phải CLB)
