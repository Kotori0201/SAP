# AWS Global Accelerator

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **AWS Global Accelerator** | Dịch vụ cung cấp **2 địa chỉ IP Anycast tĩnh** toàn cầu, đưa traffic người dùng vào **mạng nội bộ AWS** từ edge location gần nhất rồi tới ứng dụng ở một hoặc nhiều region. |
| **Anycast IP** | Một IP được **nhiều server cùng quảng bá**; client được định tuyến tới server **gần nhất**. (Ngược với **Unicast**: mỗi server một IP riêng.) |
| **Endpoint group** | Nhóm endpoint (ALB, NLB, EC2, EIP) trong một region, có **traffic dial** điều chỉnh % traffic. |
| **Client IP preservation** | Giữ nguyên IP gốc của client khi traffic tới endpoint. |

## Nội dung chính

### Cách hoạt động

> 📖 **Định nghĩa:** **Edge location** là điểm hiện diện của AWS gần người dùng.

- Tạo **2 Anycast IP** cho ứng dụng
- Anycast IP đưa traffic tới **edge location gần nhất**
- Edge location chuyển traffic qua **mạng private của AWS** tới ứng dụng (thay vì Internet công cộng nhiều hop)

### Đặc điểm

> 📖 **Định nghĩa:** **Regional failover** là chuyển traffic sang region khác khi region chính không khỏe.

- Endpoint: **Elastic IP, EC2, ALB, NLB** (public hoặc private)
- **Client IP preservation** (trừ endpoint EIP)
- **Hiệu năng ổn định**: định tuyến thông minh tới độ trễ thấp nhất, **failover regional nhanh**
- **Không bị vấn đề cache DNS phía client** (vì IP không đổi)
- **Health check**: failover **< 1 phút** khi endpoint lỗi → tốt cho **DR**
- **Bảo mật**: chỉ cần **whitelist 2 IP**; chống DDoS nhờ **AWS Shield**
- **Traffic dial** & **endpoint weight** để chuyển traffic dần (blue/green giữa region)

### Global Accelerator vs CloudFront

> 📖 **Định nghĩa:** **CloudFront** là CDN phục vụ nội dung **tại edge** (có cache); **Global Accelerator** là **proxy gói tin** tại edge tới ứng dụng ở region (không cache).

| | **CloudFront** | **Global Accelerator** |
|---|---|---|
| Chung | Mạng toàn cầu AWS, edge location, tích hợp **Shield** | ← giống |
| Nội dung | **Cacheable** (ảnh, video) + **dynamic** (API acceleration) | Không cache |
| Nơi phục vụ | **Tại edge** | Proxy tới app ở **1+ region** |
| Giao thức | HTTP/HTTPS | **TCP, UDP** (mọi ứng dụng) |
| Phù hợp | Website, media, API | **Non-HTTP**: gaming (UDP), IoT (MQTT), **VoIP**; HTTP cần **IP tĩnh**; HTTP cần **failover regional nhanh, xác định** |

## 🎯 Mẹo thi

- "Đối tác chỉ whitelist được vài IP cố định, app đa region" → **Global Accelerator** (2 IP Anycast)
- "Game UDP toàn cầu, giảm độ trễ" → **Global Accelerator**
- "Failover giữa region < 1 phút, không phụ thuộc DNS TTL" → **Global Accelerator**
- "Cache ảnh/video tại edge" → **CloudFront**
