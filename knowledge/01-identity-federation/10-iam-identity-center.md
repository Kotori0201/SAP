# IAM Identity Center

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **IAM Identity Center** | Dịch vụ SSO cho workforce: một lần đăng nhập để truy cập nhiều AWS account và ứng dụng SaaS/SAML (trước đây là AWS SSO). |
| **Single Sign-On (SSO)** | Đăng nhập một lần, truy cập nhiều hệ thống mà không phải đăng nhập lại. |
| **Identity source** | Nơi lưu user/group cho Identity Center: built-in store, Active Directory, hoặc external IdP. |
| **Permission set** | Mẫu quyền gồm các IAM policy; khi gán vào account, Identity Center tạo IAM role tương ứng. |
| **Delegated administrator** | Member account được ủy quyền quản trị một service thay cho management account. |

## Nội dung chính

*(trước đây là AWS SSO)*

- **Một lần đăng nhập** cho:
  - Mọi **AWS account trong Organizations**
  - **SaaS** (Salesforce, Box, Microsoft 365…)
  - App hỗ trợ **SAML 2.0** tùy chỉnh
  - EC2 Windows instances
- **Identity source** (chọn 1): **built-in identity store** · **Active Directory** (Managed AD hoặc AD Connector) · **External IdP** (Okta, Entra ID, OneLogin… qua SAML 2.0 + **SCIM** để đồng bộ user/group)

### Thành phần

> 📖 **Định nghĩa:** **Permission set** là mẫu quyền (tập IAM policy) định nghĩa trong Identity Center; khi gán cho account, nó được triển khai thành IAM role. **SCIM (System for Cross-domain Identity Management)** là chuẩn tự động đồng bộ user/group từ IdP bên ngoài.

- **Permission Set**: tập 1+ IAM policy; khi gán (group/user × account × permission set) → Identity Center **tự tạo IAM role** trong account đích
- **Application assignments**: SSO vào SaaS/SAML apps (cung cấp URL, certificate, metadata)
- **ABAC**: dùng attribute của user (cost center, title, locale…) trong identity store làm **session tag** → định nghĩa quyền một lần, đổi quyền bằng cách đổi attribute

Ví dụ: group *Developers* → gán permission set `FullAccess` cho Dev accounts và `ReadOnlyAccess` cho Prod accounts.

> 🆕 Ngoài slide: nên đặt Identity Center ở **delegated admin account** (không phải management account); **Trusted identity propagation** truyền danh tính người dùng tới Redshift, Athena, Lake Formation, Q Business…; có **account instance** cho app đơn lẻ.

> 🎯 "Nhiều account + SSO workforce + quản lý tập trung" → **IAM Identity Center** (không phải SAML thủ công từng account).
