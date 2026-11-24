# EC2 Auto Scaling

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Auto Scaling Group (ASG)** | Nhóm EC2 instance tự động **tăng/giảm số lượng** theo nhu cầu, thay thế instance lỗi, giữ số lượng trong khoảng **min – max**, mục tiêu **desired**. |
| **Launch template** | Mẫu cấu hình để khởi chạy instance (AMI, instance type, SG, user data…), có versioning. Thay thế *launch configuration* (cũ, không còn khuyến nghị). |
| **Scaling policy** | Quy tắc quyết định khi nào và bao nhiêu instance được thêm/bớt. |
| **Lifecycle hook** | Điểm dừng cho phép chạy hành động tùy chỉnh **trước khi** instance vào phục vụ hoặc **trước khi** bị terminate. |
| **Instance refresh** | Tính năng thay thế dần toàn bộ instance trong ASG theo launch template mới. |
| **Warm-up time** | Thời gian chờ instance mới sẵn sàng trước khi tính vào metric scaling. |
| **Cooldown** | Khoảng thời gian ASG chờ sau một lần scale trước khi scale tiếp. |

## Nội dung chính

### Dynamic Scaling Policies

> 📖 **Định nghĩa:** **Dynamic scaling** là scale **phản ứng** theo metric thực tế trong CloudWatch.

| Policy | Cách hoạt động | Ví dụ |
|---|---|---|
| **Target Tracking** | Đơn giản nhất: giữ một metric quanh giá trị mục tiêu | CPU trung bình ASG ~40% |
| **Simple / Step Scaling** | Theo CloudWatch Alarm, thêm/bớt theo bậc | CPU > 70% → +2; CPU < 30% → −1 |
| **Scheduled Actions** | Theo lịch đã biết | Thứ 6, 17h tăng min lên 10 |

### Predictive Scaling

> 📖 **Định nghĩa:** **Predictive scaling** dùng ML **dự báo tải** từ lịch sử và **lên lịch scale trước** khi tải tới.

- Phù hợp tải **có tính chu kỳ** (theo ngày/tuần); kết hợp được với dynamic scaling

### Metric tốt để scale

> 📖 **Định nghĩa:** **RequestCountPerTarget** là số request trung bình mà mỗi target của ALB nhận.

- **CPUUtilization** – CPU trung bình
- **RequestCountPerTarget** – giữ số request/instance ổn định
- **Average Network In/Out** – ứng dụng nặng mạng
- **Custom metric** đẩy lên CloudWatch (vd độ dài hàng đợi SQS / số instance → *backlog per instance*)

### Những điều cần biết

> 📖 **Định nghĩa:** Xem định nghĩa *Lifecycle hook* ở bảng trên.

- Hỗ trợ **mix Spot + On-Demand** (mixed instances policy)
- **Lifecycle hooks**: cleanup, trích log, health check đặc biệt trước khi phục vụ/terminate
- Nâng AMI: cập nhật launch template → terminate instance cũ thủ công (CloudFormation hỗ trợ) **hoặc dùng Instance Refresh**
- 🆕 **Warm pools**: giữ sẵn instance đã khởi tạo (stopped/hibernated) để scale-out nhanh

### Instance Refresh

> 📖 **Định nghĩa:** Xem bảng trên.

- Gọi `StartInstanceRefresh` sau khi đổi launch template
- Cài **minimum healthy percentage** (vd 60%) và **warm-up time**
- Hỗ trợ checkpoint, rollback, skip matching

### Scaling Processes

> 📖 **Định nghĩa:** **Scaling process** là từng tiến trình nội bộ của ASG; có thể **suspend (tạm dừng)** riêng lẻ.

| Process | Chức năng |
|---|---|
| `Launch` | Thêm instance |
| `Terminate` | Bớt instance |
| `HealthCheck` | Kiểm tra sức khỏe |
| `ReplaceUnhealthy` | Terminate và tạo lại instance lỗi |
| `AZRebalance` | Cân bằng số instance giữa các AZ |
| `AlarmNotification` | Nhận thông báo từ CloudWatch |
| `ScheduledActions` | Chạy scheduled action |
| `AddToLoadBalancer` | Đưa instance vào LB/target group |
| `InstanceRefresh` | Chạy instance refresh |

> 💡 Ví dụ: suspend `ReplaceUnhealthy` để debug instance lỗi; suspend `AZRebalance` để ASG không terminate instance khi cân bằng AZ. Hoặc dùng **standby** / **scale-in protection** cho từng instance.

### Health Checks

> 📖 **Định nghĩa:** **Health check** là cơ chế xác định instance còn khỏe không; instance không khỏe sẽ bị ASG thay thế.

- **EC2 status checks** (mặc định)
- **ELB health checks** (HTTP) – nên bật khi đứng sau LB
- **Custom health check**: gửi trạng thái qua CLI/SDK `set-instance-health`
- ASG **terminate rồi launch mới** instance lỗi (không restart)
- ⚠️ Health check phải **đơn giản, kiểm tra đúng thứ**: endpoint `/health` của server, **không** gọi DB (nếu DB lỗi, tất cả instance bị coi là lỗi và bị thay liên tục)

### Cập nhật ứng dụng – kiến trúc triển khai

> 📖 **Định nghĩa:** **Blue/green deployment** là chạy song song môi trường cũ (blue) và mới (green), chuyển traffic dần hoặc một lần; **canary** là chuyển một phần nhỏ traffic sang bản mới trước.

| Cách | Mô tả | Ưu/nhược |
|---|---|---|
| **Cùng ASG, cùng target group** | Đổi launch template v1 → v2, instance mới dần thay instance cũ | Đơn giản; khó kiểm soát % traffic |
| **2 ASG, 2 target group, cùng ALB** | ALB **weighted target groups** chia traffic giữa v1 và v2 | Kiểm soát %; rollback nhanh |
| **2 ALB, Route 53 weighted (CNAME)** | Mỗi phiên bản một ALB + ASG; Route 53 chia traffic | Test thủ công/load test riêng; phụ thuộc DNS TTL và client cache |

## 🎯 Mẹo thi

- "Giữ CPU ~50%, ít cấu hình nhất" → **Target tracking**
- "Biết trước tải tăng lúc 9h sáng thứ 2" → **Scheduled** hoặc **Predictive scaling**
- "Chạy script trích log trước khi instance bị terminate" → **Lifecycle hook**
- "Đổi AMI cho toàn bộ ASG, giữ tối thiểu 80% capacity" → **Instance Refresh**
- "ASG thay instance liên tục khi DB lỗi" → health check đang **phụ thuộc DB**
- "Khởi động instance chậm (bootstrap lâu)" → **AMI dựng sẵn (golden AMI)**, **warm pools**
- "Scale theo backlog SQS" → custom metric *backlog per instance* + target tracking
