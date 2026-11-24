# EC2 Spot Instances & Spot Fleet

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Spot Instance** | Instance dùng **dung lượng EC2 dư thừa** của AWS với giá giảm tới **90%**, nhưng AWS có thể **thu hồi** bất kỳ lúc nào (báo trước 2 phút). |
| **Spot price** | Giá Spot theo giờ, thay đổi theo cung cầu của từng capacity pool. |
| **Spot interruption** | Việc AWS thu hồi Spot instance (stop / hibernate / terminate) kèm **thông báo trước 2 phút**. |
| **Capacity pool** | Tập instance cùng **instance type + AZ + OS** — mỗi pool có giá và dung lượng riêng. |
| **Spot Fleet** | Tập hợp Spot Instance (+ tùy chọn On-Demand) mà AWS tự quản lý để đạt **target capacity** với ràng buộc giá. |
| **EC2 Fleet** | Phiên bản API mới hơn của fleet, dùng chung khái niệm (ASG mixed instances policy dùng cơ chế tương tự). |

## Nội dung chính

### Spot Instances

> 📖 **Định nghĩa:** **Max price** là giá tối đa bạn sẵn sàng trả; mặc định là giá On-Demand.

- Đặt **max price**, nhận instance khi spot price < max price
- Giá thay đổi theo cung cầu (thực tế ổn định hơn trước, gián đoạn chủ yếu do **thiếu capacity**)
- Khi bị thu hồi: chọn **stop**, **hibernate** hoặc **terminate**, có **2 phút** để xử lý
- Bắt tín hiệu: instance metadata (`spot/instance-action`) hoặc **EventBridge** "EC2 Spot Instance Interruption Warning"; 🆕 **rebalance recommendation** báo sớm hơn
- Dùng cho: **batch job, phân tích dữ liệu, CI/CD, render, workload chịu lỗi, stateless**
- **Không** dùng cho: job quan trọng không ngắt được, **database**

### Spot Fleets

> 📖 **Định nghĩa:** **Launch pool** là một tổ hợp lựa chọn (instance type, OS, AZ) mà fleet có thể dùng để khởi chạy instance.

- Fleet = Spot (+ On-Demand tùy chọn) để đạt **target capacity**
- Định nghĩa **nhiều launch pool** → fleet tự chọn
- Dừng khởi chạy khi đạt **capacity** hoặc **max cost**

**Chiến lược phân bổ Spot:**

| Chiến lược | Ý nghĩa | Khi nào |
|---|---|---|
| `lowestPrice` | Pool rẻ nhất | Tối ưu chi phí, workload **ngắn** |
| `diversified` | Rải đều mọi pool | **Sẵn sàng cao**, workload **dài** |
| `capacityOptimized` | Pool còn nhiều capacity nhất | Giảm khả năng bị thu hồi |
| **`priceCapacityOptimized`** (khuyến nghị) | Pool nhiều capacity nhất, rồi chọn giá thấp nhất trong đó | **Lựa chọn tốt nhất cho đa số workload** |

### Best practice

> 📖 **Định nghĩa:** **Instance flexibility** là việc chấp nhận nhiều instance type/AZ để tăng số capacity pool có thể dùng.

- **Đa dạng hóa** instance type và AZ (càng nhiều pool càng ít bị gián đoạn)
- Dùng **attribute-based instance type selection** (chọn theo vCPU/RAM thay vì liệt kê type)
- Thiết kế **checkpoint** để tiếp tục sau gián đoạn
- Kết hợp **On-Demand baseline + Spot** cho phần tăng thêm

## 🎯 Mẹo thi

- "Rẻ nhất cho batch chịu được gián đoạn" → **Spot**
- "Spot nhưng hạn chế bị thu hồi, chi phí vẫn thấp" → **`priceCapacityOptimized`** + đa dạng instance type
- "Workload dài, cần sẵn sàng" → `diversified` (hoặc mix On-Demand)
- "Lưu trạng thái trước khi Spot bị thu hồi" → bắt **interruption notice** (EventBridge/metadata) và checkpoint
- "Database trên Spot" → ❌ sai
