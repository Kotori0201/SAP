# AWS AppSync

[⬅ Về mục lục section](README.md)

## 📖 Định nghĩa

| Thuật ngữ | Định nghĩa |
|---|---|
| **AWS AppSync** | Dịch vụ managed cung cấp **GraphQL API** (và Events API real-time), kết nối nhiều nguồn dữ liệu, hỗ trợ real-time và offline sync. |
| **GraphQL** | Ngôn ngữ truy vấn API cho phép client **lấy đúng dữ liệu cần**, kết hợp nhiều nguồn trong **một request**. |
| **GraphQL schema** | Định nghĩa kiểu dữ liệu, query, mutation, subscription của API — điểm bắt đầu của AppSync. |
| **Resolver** | Logic ánh xạ một trường GraphQL tới nguồn dữ liệu (DynamoDB, Aurora, OpenSearch, Lambda, HTTP). |
| **Subscription** | Loại thao tác GraphQL cho phép client **nhận cập nhật real-time** qua WebSocket. |

## Nội dung chính

### Tổng quan

> 📖 **Định nghĩa:** Xem bảng trên.

- Dùng **GraphQL** → app lấy đúng dữ liệu cần, **gộp từ nhiều nguồn**: NoSQL, quan hệ, HTTP API…
- Tích hợp **DynamoDB, Aurora, OpenSearch**…; nguồn tùy biến qua **Lambda**
- **Real-time** qua WebSocket hoặc MQTT over WebSocket
- **Mobile**: truy cập dữ liệu local & **đồng bộ offline** (Amplify DataStore)
- Bắt đầu bằng **upload GraphQL schema**
- Client: web app, mobile app, dashboard real-time; metric & log vào **CloudWatch**
- 🆕 **AppSync Events**: API pub/sub WebSocket serverless không cần GraphQL

### Tích hợp Cognito & bảo mật

> 📖 **Định nghĩa:** **Cognito group** là nhóm user trong User Pool; AppSync dùng nhóm này để phân quyền trong schema.

- Phân quyền theo **group Cognito** của user
- Khai báo bảo mật ngay **trong GraphQL schema** (directive `@aws_auth`, `@aws_cognito_user_pools`)
- Client đăng nhập Cognito → nhận **JWT** → AppSync authorize → resolver dùng IAM permission truy cập DynamoDB
- Chế độ auth khác: **API key**, **IAM**, **OIDC**, **Lambda authorizer**

## 🎯 Mẹo thi

- "GraphQL", "lấy dữ liệu từ nhiều nguồn trong một request" → **AppSync**
- "App mobile cần real-time update và offline sync" → **AppSync** (+ Amplify)
- "REST API thông thường" → **API Gateway**, không phải AppSync
