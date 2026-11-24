# AWS KMS (Key Management Service)

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **AWS KMS** | Dịch vụ managed để tạo, lưu trữ và kiểm soát việc sử dụng **khóa mật mã**; khóa nằm trong HSM đạt FIPS 140-3 và **không bao giờ rời KMS ở dạng chưa mã hóa**. |
| **KMS key** | Đối tượng khóa logic trong KMS (trước gọi là CMK), gồm metadata, key policy và key material. |
| **Key material** | Chuỗi bit bí mật thực sự dùng để mã hóa/giải mã. |
| **Key policy** | Resource-based policy **bắt buộc** của mỗi KMS key, quyết định ai được dùng/quản lý key. |
| **Envelope encryption** | Mã hóa dữ liệu bằng **data key**, rồi mã hóa data key bằng KMS key; lưu data key đã mã hóa cạnh dữ liệu. |
| **Data key** | Khóa đối xứng do KMS sinh (`GenerateDataKey`) để mã hóa dữ liệu lớn bên ngoài KMS. |
| **Symmetric / Asymmetric key** | Đối xứng: 1 khóa dùng cho cả mã hóa và giải mã. Bất đối xứng: cặp public/private key. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** **Encryption at rest** là mã hóa dữ liệu khi được lưu trữ; trong AWS, hầu hết dịch vụ dùng KMS cho việc này.

- Nghe "**encryption**" trong đề → gần như luôn là **KMS**
- Tích hợp **IAM** để phân quyền; mọi lần dùng key được ghi vào **CloudTrail**
- Tích hợp: EBS, S3, RDS, Redshift, SSM Parameter Store, Secrets Manager, DynamoDB…
- **KMS key là tài nguyên theo region** – chỉ dùng được trong region tạo ra (trừ multi-Region key)
- Giới hạn: `Encrypt`/`Decrypt` trực tiếp tối đa **4 KB** → dữ liệu lớn dùng **envelope encryption**

### Loại khóa theo thuật toán

> 📖 **Định nghĩa:** **AES-256** là thuật toán mã hóa đối xứng 256-bit; **RSA** và **ECC** là họ thuật toán bất đối xứng.

| | **Symmetric (AES-256)** | **Asymmetric (RSA, ECC)** |
|---|---|---|
| Khóa | 1 khóa | Cặp public/private |
| Thao tác | Encrypt/Decrypt, GenerateDataKey | Encrypt/Decrypt **hoặc** Sign/Verify |
| Truy cập khóa | Không bao giờ lấy được, phải gọi KMS API | **Public key tải về được**; private key không |
| Dùng cho | AWS service tích hợp, envelope encryption | Mã hóa **bên ngoài AWS** bởi người không gọi được KMS API; ký số |

### Loại khóa theo quyền sở hữu

> 📖 **Định nghĩa:** **Customer managed key** do bạn tạo và quản lý; **AWS managed key** do AWS tạo cho từng service trong account bạn (`aws/s3`…); **AWS owned key** do AWS sở hữu, dùng chung nhiều account, nằm ngoài account bạn.

| | Customer managed | AWS managed | AWS owned |
|---|---|---|---|
| Xem metadata | ✅ | ✅ | ❌ |
| Quản lý (key policy, enable/disable) | ✅ | ❌ (chỉ xem) | ❌ |
| Chỉ dùng cho account mình | ✅ | ✅ | ❌ |
| Audit CloudTrail | ✅ | ✅ | ❌ |
| Xoay vòng tự động | Tùy chọn, **90–2560 ngày** (mặc định 365) | Bắt buộc **mỗi năm** | Tùy service |
| Xoay vòng on-demand | ✅ | ❌ | ❌ |
| Chia sẻ cross-account | ✅ (qua key policy) | ❌ | ❌ |

> 💡 **Rotation** giữ nguyên key ID/ARN, tạo key material mới; dữ liệu cũ vẫn giải mã được bằng material cũ (KMS tự giữ lại).

### Nguồn key material (Key Material Origin)

> 📖 **Định nghĩa:** **Key material origin** cho biết key material được sinh ra ở đâu; **không đổi được** sau khi tạo key.

| Origin | Mô tả | Lưu ý |
|---|---|---|
| **AWS_KMS** (mặc định) | KMS tạo và quản lý material | Đơn giản nhất |
| **EXTERNAL (BYOK)** | **Bạn import** material vào KMS | Bạn chịu trách nhiệm bảo mật, sẵn sàng, bền vững bên ngoài AWS; **không có automatic rotation** (xoay thủ công); hỗ trợ symmetric & asymmetric; có thể đặt **ngày hết hạn** |
| **AWS_CLOUDHSM (Custom Key Store)** | Material nằm trong **CloudHSM cluster của bạn** | Thao tác mật mã chạy trong HSM của bạn; cần **≥ 2 HSM active** ở các AZ khác nhau |
| **External Key Store (XKS)** | Material nằm trong **HSM bên ngoài AWS** (on-prem) | Cho yêu cầu "khóa không bao giờ nằm trong AWS" |

**Quy trình BYOK:** tạo KMS key (origin EXTERNAL) → tải **public key + import token** → mã hóa key material bằng public key → import material đã mã hóa + token.

**Khi nào dùng Custom Key Store (CloudHSM):** cần **kiểm soát trực tiếp HSM**, key phải nằm trong **HSM single-tenant chuyên dụng**.

### Multi-Region Keys

> 📖 **Định nghĩa:** **Multi-Region key** là tập các KMS key ở nhiều region có **cùng key ID và key material**, dùng thay thế cho nhau: mã hóa ở region này, giải mã ở region khác mà không cần gọi cross-region.

- ID dạng `mrk-...`; 1 **primary** + nhiều **replica**, đồng bộ key material & rotation
- **Không phải global**: mỗi key được quản lý độc lập (key policy riêng)
- Chỉ có **1 primary** tại một thời điểm; có thể **promote replica** thành primary
- Use case: **Disaster Recovery**, quản lý dữ liệu toàn cầu (vd **DynamoDB Global Tables**, Aurora Global với client-side encryption), app **active-active** đa region, **ký số phân tán**
- Không khuyến nghị dùng tràn lan — chỉ khi thật cần (yêu cầu bảo mật thường muốn khóa cô lập theo region)

### Chia sẻ cross-account & snapshot mã hóa

> 📖 **Định nghĩa:** **Grant** là cơ chế ủy quyền tạm thời, chi tiết cho một principal dùng KMS key, hay được AWS service dùng thay bạn.

- Chia sẻ snapshot EBS/RDS mã hóa sang account khác: **phải dùng customer managed key** + sửa **key policy** cho account đích → account đích copy snapshot và **re-encrypt bằng key của mình**
- Snapshot mã hóa bằng **AWS managed key không chia sẻ được**
- Copy snapshot sang region khác → **re-encrypt bằng key ở region đích**

### 🆕 Mật mã hậu lượng tử (chủ đề mới SAP-C03)

> 📖 **Định nghĩa:** **Post-quantum cryptography (PQC)** là các thuật toán được thiết kế để vẫn an toàn trước máy tính lượng tử. **ML-KEM** (FIPS 203) là thuật toán *trao đổi khóa*; **ML-DSA** (FIPS 204) là thuật toán *chữ ký số*.

- **ML-DSA trong KMS:** tạo asymmetric key loại ML-DSA để **ký/xác minh** chữ ký số chống lượng tử (vd ký firmware, tài liệu cần hiệu lực lâu dài)
- **ML-KEM:** dùng trong **hybrid post-quantum TLS** khi kết nối tới endpoint KMS, ACM, Secrets Manager → chống tấn công "**harvest now, decrypt later**" (thu thập traffic bây giờ, giải mã sau khi có máy lượng tử)
- Đề có thể hỏi: dữ liệu cần bảo mật **dài hạn** trước rủi ro lượng tử → bật hybrid PQ TLS trong SDK + dùng ML-DSA cho chữ ký
- ⚠️ Phần này dựa trên kiến thức đến giữa 2026 — đối chiếu tài liệu KMS mới nhất

## 🎯 Mẹo thi

- Dữ liệu > 4 KB → **envelope encryption** (`GenerateDataKey`)
- "Khách hàng ngoài AWS cần mã hóa dữ liệu gửi cho bạn" → **asymmetric key**, phát **public key**
- "Kiểm soát hoàn toàn HSM / single-tenant / FIPS level 3 với key quản lý riêng" → **CloudHSM** hoặc **Custom Key Store**
- "Key không bao giờ ở trong AWS" → **External Key Store (XKS)**
- "Import key của mình" → **BYOK (EXTERNAL)** – nhớ **không tự xoay vòng**
- "Mã hóa ở region A, giải mã ở region B không độ trễ cross-region" → **Multi-Region key**
- Chia sẻ snapshot mã hóa → **customer managed key** + key policy
