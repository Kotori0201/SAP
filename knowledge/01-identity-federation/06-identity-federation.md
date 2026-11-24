# Identity Federation

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **Identity Federation** | Cơ chế cho phép user được quản lý *bên ngoài AWS* truy cập tài nguyên AWS bằng cách tin tưởng một Identity Provider. |
| **Identity Provider (IdP)** | Hệ thống xác thực và phát danh tính cho user (AD/ADFS, Okta, Entra ID, Google…). |
| **SAML 2.0** | Chuẩn XML trao đổi xác thực/ủy quyền giữa IdP và Service Provider, phổ biến cho workforce doanh nghiệp. |
| **OIDC (OpenID Connect)** | Lớp danh tính trên OAuth 2.0, dùng JWT token; phổ biến cho web/mobile và workload (GitHub Actions, EKS). |
| **Amazon Cognito** | Dịch vụ danh tính cho ứng dụng: User Pool (đăng ký/đăng nhập) và Identity Pool (cấp AWS credential). |
| **Token Vending Machine (TVM)** | Mẫu kiến trúc cũ: server tự viết để cấp credential tạm cho client; nay thay bằng Cognito Identity Pool. |

## Nội dung chính

**Ý tưởng:** user quản lý **ngoài AWS** (AD, Okta, Google…), không cần tạo IAM user. IdP xác thực → AWS tin IdP (trust relationship) → cấp credential tạm.

**Các "flavor":**

| Loại | Khi nào | API |
|---|---|---|
| **SAML 2.0** | IdP doanh nghiệp tương thích SAML (ADFS, Okta, Entra ID…) | `AssumeRoleWithSAML` |
| **Custom Identity Broker** | IdP **KHÔNG** tương thích SAML | `AssumeRole` / `GetFederationToken` |
| **Web Identity (không Cognito)** | App dùng OIDC IdP – **không khuyến nghị** | `AssumeRoleWithWebIdentity` |
| **Web Identity + Cognito** | App web/mobile – **khuyến nghị** | Cognito Identity Pools → STS |
| **IAM Identity Center** | Workforce SSO cho nhiều account + SaaS app – **cách mới, ưu tiên** | (managed) |

### SAML 2.0

> 📖 **Định nghĩa:** **SAML 2.0 (Security Assertion Markup Language)** là chuẩn mở dựa trên XML để trao đổi thông tin xác thực giữa **Identity Provider (IdP)** – nơi xác thực user – và **Service Provider (SP)** – ở đây là AWS. **SAML assertion** là tài liệu XML có chữ ký do IdP phát ra, chứng nhận user đã đăng nhập và mang thuộc tính (role được phép).

- Trust **2 chiều**: tạo **SAML identity provider** trong IAM (upload metadata của IdP) + cấu hình AWS là relying party trên IdP
- Truy cập **Console, CLI, API** bằng credential tạm
- Là "cách cũ"; **IAM Identity Center là cách mới, đơn giản hơn**

**Luồng API/CLI:** App → IdP (xác thực qua LDAP/AD) → nhận **SAML assertion** → gọi `AssumeRoleWithSAML` → STS trả credential → gọi AWS API.

**Luồng Console:** User → portal IdP → nhận SAML assertion → **POST tới `https://signin.aws.amazon.com/saml`** → AWS sign-in lấy credential từ STS → redirect vào Console.

### Custom Identity Broker

> 📖 **Định nghĩa:** **Identity broker** là ứng dụng trung gian do bạn tự viết: xác thực user với hệ thống nội bộ, rồi thay mặt user gọi STS để lấy credential tạm thời.

- Chỉ dùng khi IdP **không** hỗ trợ SAML 2.0
- Broker tự xác thực user, **tự quyết định role/quyền phù hợp**, gọi STS, trả về token hoặc **sign-in URL** cho Console
- Phải tự code & bảo trì → nhiều công vận hành

### Web Identity Federation

> 📖 **Định nghĩa:** **Web Identity Federation** là cơ chế cho user đăng nhập bằng IdP công cộng chuẩn **OIDC (OpenID Connect)** như Google, Facebook, Amazon rồi đổi token lấy AWS credential. **Amazon Cognito** là dịch vụ quản lý danh tính cho app web/mobile: **User Pool** (thư mục user, đăng nhập) và **Identity Pool** (đổi danh tính lấy AWS credential).

- **Không Cognito:** client login IdP (Google/Facebook/OIDC) → web identity token → `AssumeRoleWithWebIdentity` → credential tạm
- **Có Cognito (khuyến nghị):** client login IdP → token → **Cognito Identity Pool** đổi thành Cognito token → STS credential tạm
  - Ưu điểm: hỗ trợ **guest/anonymous**, MFA, ánh xạ role theo rule/claim, thay thế **Token Vending Machine (TVM)** tự viết
- Định danh user trong policy bằng biến: `${cognito-identity.amazonaws.com:sub}`, `${www.amazon.com:user_id}`, `${graph.facebook.com:id}`, `${accounts.google.com:sub}`

```json
{
  "Effect": "Allow",
  "Action": ["s3:GetObject", "s3:PutObject"],
  "Resource": "arn:aws:s3:::app-bucket/${cognito-identity.amazonaws.com:sub}/*"
}
```

> 💡 Phân biệt Cognito: **User Pool** = xác thực (sign-up/sign-in, trả JWT); **Identity Pool** = đổi danh tính lấy **AWS credentials**.
