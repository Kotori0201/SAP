# ⭐ Tổng kết · Identity & Federation

[⬅ Về mục lục section](README.md)

## 1. Bảng tổng hợp kiến thức

| Dịch vụ / Khái niệm | Là gì (1 dòng) | Con số phải nhớ | Chọn khi |
|---|---|---|---|
| **IAM Policy** | JSON quy định ai làm gì trên đâu | Explicit Deny > Allow > Implicit Deny | Mọi phân quyền |
| **Permission Boundary** | Giới hạn quyền tối đa của user/role | Không cho group | Ủy quyền tạo role an toàn |
| **Access Analyzer** | Phân tích truy cập & sinh policy | Policy generation từ CloudTrail **90 ngày** | Tìm truy cập ngoài, least privilege |
| **STS AssumeRole** | Lấy credential tạm của role | **15 phút – 12 giờ** · role chaining **1 giờ** | Cross-account, service |
| **External ID** | Chuỗi chống confused deputy | Bên thứ 3 tạo | Cấp quyền cho SaaS |
| **SAML 2.0 Federation** | IdP doanh nghiệp → AWS | `AssumeRoleWithSAML` | ADFS/Okta (cách cũ) |
| **Custom Identity Broker** | Tự viết broker | `GetFederationToken`/`AssumeRole` | IdP **không** SAML |
| **Cognito Identity Pool** | Đổi danh tính web/mobile lấy AWS credential | Hỗ trợ guest | App mobile/web |
| **Managed Microsoft AD** | AD thật trên AWS | Multi-AZ, trust on-prem | RDS SQL Server, FSx Windows, trust |
| **AD Connector** | Proxy về AD on-prem | Không cache, cần VPN/DX | User ở on-prem |
| **Simple AD** | AD tương thích giá rẻ | ~500 / 5.000 user, không MFA | Nhỏ, rẻ |
| **Organizations** | Quản lý nhiều account | Consolidated Billing / **All Features** | Multi-account |
| **SCP** | Giới hạn quyền tối đa account/OU | Không áp management account, không áp SLR | Cấm service/region |
| **RCP** 🆕 | Giới hạn quyền tối đa trên resource | Áp cho mọi principal | Data perimeter |
| **IAM Identity Center** | SSO workforce | Permission set → IAM role | Nhiều account + SaaS |
| **Control Tower** | Landing zone + guardrails | Preventive (SCP) · Detective (Config) · Proactive (CFN Hooks) | Dựng multi-account chuẩn |
| **RAM** | Chia sẻ resource giữa account | VPC sharing **chỉ trong org** | Subnet, TGW, Resolver rules |

## 2. So sánh quan trọng

### Role vs Resource-based policy

| Assume Role | Resource-based policy |
|---|---|
| Bỏ quyền gốc | Giữ quyền gốc |
| Mọi service | Chỉ service hỗ trợ (S3, SQS, SNS, Lambda, KMS…) |

### Các lớp giới hạn quyền

| Lớp | Áp lên | Cấp quyền? |
|---|---|---|
| SCP | Account/OU (principal) | ❌ |
| RCP 🆕 | Resource trong org | ❌ |
| Permission boundary | User/role | ❌ |
| Session policy | Role session | ❌ |
| Identity / Resource policy | Principal / resource | ✅ |

### STS API

| API | Nguồn danh tính |
|---|---|
| `AssumeRole` | IAM principal |
| `AssumeRoleWithSAML` | SAML IdP |
| `AssumeRoleWithWebIdentity` | OIDC (khuyên dùng Cognito) |
| `GetSessionToken` | IAM user + MFA |
| `GetFederationToken` | Custom broker |

### Federation

| Tình huống | Chọn |
|---|---|
| Workforce, nhiều account | **IAM Identity Center** |
| IdP SAML, ít account | SAML 2.0 |
| IdP không SAML | Custom broker |
| App web/mobile | Cognito |

### Directory Services

| | Managed AD | AD Connector | Simple AD |
|---|---|---|---|
| User ở | AWS | On-prem | AWS |
| Trust | ✅ | ❌ | ❌ |
| MFA | ✅ | ✅ | ❌ |

### Cognito

| User Pool | Identity Pool |
|---|---|
| Đăng nhập, trả JWT | Đổi danh tính lấy AWS credential |

### Control Tower guardrails

| Preventive | Detective | Proactive |
|---|---|---|
| SCP | Config | CloudFormation Hooks |
| Ngăn trước | Phát hiện sau | Chặn trước khi tạo |

## 3. Tip nhớ nhanh

- 🧠 **"Deny luôn thắng"** – explicit Deny ở bất kỳ đâu = từ chối.
- 🧠 **"SCP = Chỉ Chặn, Không Cấp"** và **"SCP không đụng sếp (management account) và không đụng SLR"**.
- 🧠 **"Boundary = Biên giới"** – chỉ user & role, không group.
- 🧠 **"External ID = bên ngoài tạo"** – chống confused deputy.
- 🧠 **"Service làm deputy → aws:SourceArn / aws:SourceAccount"**.
- 🧠 **"Đọc A + ghi B cùng lúc → resource policy ở B"** (không assume role).
- 🧠 **"AD Connector = Cầu nối, không lưu, không trust"**; **"Simple AD = Simple, không MFA"**.
- 🧠 **"Trust ≠ Sync"** – trust không copy user.
- 🧠 **"Invite → tự tạo OrganizationAccountAccessRole"**.
- 🧠 **"All Features = một chiều"** – không quay về Consolidated Billing.
- 🧠 **"Tag Policy = định dạng tag; SCP + Null = bắt buộc có tag"**.
- 🧠 **"Nhiều account + SSO = Identity Center"**; **"Okta/Entra + đồng bộ user = SAML + SCIM"**.
- 🧠 **"Guardrail: P-D-P = Preventive SCP, Detective Config, Proactive Hooks"**.
- 🧠 **"VPC sharing = chỉ trong org, không share default VPC"**.
- 🧠 **"ForAllValues rỗng = true"** → kết hợp Null để bắt buộc có tag.
