# 01 · Identity & Federation

> Nguồn: Slide Stephane Maarek SAP v38 (trang 9–79) + section *Identity & Federation* trên Udemy, có bổ sung cập nhật AWS (🆕). Ghi chú viết lại để ôn tập.

**Domain SAP-C03:** Security, compliance & governance · Cloud-native architecture (multi-account)

## 📖 Định nghĩa tổng quan

- **Identity (danh tính):** đại diện cho một người, ứng dụng hoặc service gửi request tới AWS.
- **Authentication (xác thực):** chứng minh “bạn là ai”.
- **Authorization (ủy quyền):** quyết định “bạn được làm gì”.
- **Federation (liên kết danh tính):** tin tưởng một hệ thống danh tính bên ngoài AWS thay vì tạo IAM user.
- **Multi-account governance:** quản trị tập trung nhiều AWS account bằng Organizations, SCP, Identity Center, Control Tower.

## Mục lục

| # | Chủ đề | File |
|---|---|---|
| 1 | IAM & IAM Policy | [01-iam-va-policy.md](01-iam-va-policy.md) |
| 2 | IAM Role vs Resource-based Policy | [02-role-vs-resource-based-policy.md](02-role-vs-resource-based-policy.md) |
| 3 | Permission Boundaries | [03-permission-boundaries.md](03-permission-boundaries.md) |
| 4 | IAM Access Analyzer | [04-iam-access-analyzer.md](04-iam-access-analyzer.md) |
| 5 | AWS STS (Security Token Service) | [05-aws-sts.md](05-aws-sts.md) |
| 6 | Identity Federation | [06-identity-federation.md](06-identity-federation.md) |
| 7 | AWS Directory Services | [07-directory-services.md](07-directory-services.md) |
| 8 | AWS Organizations | [08-aws-organizations.md](08-aws-organizations.md) |
| 9 | SCP, các policy của Organizations & Policy Evaluation Logic | [09-scp-va-policy-evaluation.md](09-scp-va-policy-evaluation.md) |
| 10 | IAM Identity Center | [10-iam-identity-center.md](10-iam-identity-center.md) |
| 11 | AWS Control Tower | [11-aws-control-tower.md](11-aws-control-tower.md) |
| 12 | AWS Resource Access Manager (RAM) | [12-aws-ram.md](12-aws-ram.md) |
| ⭐ | **Tổng kết, so sánh & tip nhớ nhanh** | [TONG-KET.md](TONG-KET.md) |

## Cheat-sheet: Tình huống → Giải pháp

| Tình huống trong đề | Giải pháp |
|---|---|
| Cấp quyền cho bên thứ 3 (SaaS) truy cập account | Cross-account role + **External ID** |
| AWS service ghi vào resource của bạn, chống confused deputy | Resource policy + `aws:SourceArn` / `aws:SourceAccount` |
| User A cần đọc ở account A và ghi vào account B cùng lúc | **Resource-based policy** ở B (không assume role) |
| Cho lead dev tạo role mà không leo thang quyền | **Permission boundary** + điều kiện `iam:PermissionsBoundary` |
| Cấm cả account (kể cả root) dùng service/region | **SCP** |
| Chỉ principal trong org được truy cập bucket/KMS key | `aws:PrincipalOrgID` trong resource policy, hoặc **RCP** |
| Tìm resource đang chia sẻ ra ngoài org | **IAM Access Analyzer** (zone of trust = org) |
| Sinh policy least privilege theo thực tế | **Access Analyzer policy generation** (CloudTrail 90 ngày) |
| SSO workforce cho 50 account + Salesforce | **IAM Identity Center** + permission sets |
| Dùng Okta/Entra ID làm nguồn user cho AWS | Identity Center + **External IdP (SAML) + SCIM** |
| IdP doanh nghiệp không hỗ trợ SAML | **Custom identity broker** (`GetFederationToken`/`AssumeRole`) |
| App mobile cho user đăng nhập Google/Facebook, truy cập S3 theo user | **Cognito Identity Pool** + biến `${cognito-identity.amazonaws.com:sub}` |
| App cho phép khách (guest) không đăng nhập | **Cognito Identity Pool** (unauthenticated identities) |
| User giữ nguyên trên AD on-prem, đăng nhập Console/WorkSpaces | **AD Connector** (cần VPN/DX) |
| RDS SQL Server / FSx Windows dùng Windows auth, trust AD on-prem | **AWS Managed Microsoft AD** + forest trust |
| Xác thực vẫn chạy khi VPN/DX đứt | Replica AD trên EC2 / Managed AD trong AWS |
| Directory rẻ, ít user, không MFA | **Simple AD** |
| Dựng landing zone multi-account chuẩn, có guardrail | **AWS Control Tower** |
| Tạo account mới tự động với baseline | **Control Tower Account Factory** (Service Catalog / AFT) |
| Nhiều account dùng chung subnet, giao tiếp private IP | **RAM – VPC sharing** |
| Nhiều account cần forward DNS về on-prem | Share **Route 53 Resolver rules** qua RAM |
| Cập nhật whitelist CIDR cho SG ở nhiều account | **Customer-managed prefix list** + RAM |
| Thu hồi ngay credential tạm bị lộ của role | **Revoke active sessions** (`AWSRevokeOlderSessions`) |
| Bắt buộc mọi EC2 có tag CostCenter | **SCP** Deny + `Null` condition `aws:RequestTag/CostCenter` |
| Chuẩn hóa giá trị tag toàn org | **Tag Policies** |
| Không cho AI services dùng dữ liệu của mình | **AI services opt-out policy** |
| Ép backup plan cho toàn org | **Backup policies** |
| Member account mời vào không có role admin cho management | Tạo thủ công **OrganizationAccountAccessRole** |
| Không cho account tự ý rời org | SCP Deny `organizations:LeaveOrganization` (cần All Features) |

## Bẫy hay gặp trong đề

- ❌ "Dùng SCP để **cấp** quyền" → SCP **không cấp** quyền, chỉ giới hạn.
- ❌ "SCP chặn management account" → **không áp dụng** cho management account.
- ❌ "SCP chặn service-linked role" → **không**.
- ❌ "Permission boundary áp cho group" → chỉ **user và role**.
- ❌ "AD Connector có cache / tạo trust" → **không cache, không trust**.
- ❌ "Simple AD hỗ trợ MFA / trust on-prem" → **không**.
- ❌ "Tag policy bắt buộc resource phải có tag" → dùng **SCP**.
- ❌ "Chia sẻ VPC subnet với account ngoài org" → VPC sharing **chỉ trong org**.
- ❌ "External ID do bạn tạo" → **bên thứ 3 tạo**, duy nhất cho mỗi khách hàng.
- ❌ "Web Identity Federation trực tiếp là best practice" → AWS khuyến nghị **Cognito**.
- ❌ "SAML thủ công cho từng account" khi có nhiều account → ưu tiên **IAM Identity Center**.
- ✅ Gặp từ khóa *"least operational overhead"* → ưu tiên managed: Identity Center > SAML thủ công > custom broker; Control Tower > tự dựng Organizations + Config.
