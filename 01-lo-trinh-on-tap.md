# Lộ trình ôn tập kiến thức SAP-C03 (9 tuần)

> **Thời gian học:** Thứ 2–6: 1h/ngày · Thứ 7 & Chủ nhật: 2h/ngày → **9h/tuần, ~80h tổng**
> **Không bao gồm thời gian thi thử** — xem lịch riêng ở [`02-lich-thi-thu.md`](02-lich-thi-thu.md).
> **Thời gian:** 23/11/2026 → 22/01/2027 · **Ngày thi dự kiến:** ~23/01/2027 (trước Tết Nguyên Đán 06/02/2027)

## Nguồn học chính

| Nguồn | Dùng cho |
|---|---|
| Udemy – *Ultimate AWS Certified Solutions Architect Professional* (Stephane Maarek) | Kiến thức cốt lõi (khóa viết cho SAP-C02, slide-based, không hands-on) |
| [Exam guide SAP-C03](https://aws.amazon.com/certification/certified-solutions-architect-professional/) | Danh sách task & dịch vụ in-scope — đối chiếu mỗi tuần |
| AWS Docs / FAQ / Skill Builder | Các chủ đề **mới của C03** mà khóa Stephane chưa có |
| AWS Free Tier / sandbox | Lab nhỏ cuối tuần |

## Phân bổ giờ học

| Hạng mục | Giờ |
|---|---|
| Video Stephane (1.25x + ghi chú) | ~25h (tuần 1–5, thứ 2–6) |
| Chủ đề mới SAP-C03 | ~18h (thứ 7 tuần 1–8) |
| Ôn tập ghi chú / flashcard | ~10h (chủ nhật tuần 1–5) |
| Đào sâu domain yếu, whitepaper, lab | ~20h (tuần 6–8) |
| Tổng ôn & nghỉ trước thi | ~7h (tuần 9) |

## Khung tuần

- **Thứ 2–6 (1h):** xem video + ghi chú dạng *"Tình huống → Giải pháp → Vì sao không chọn phương án khác"*
- **Thứ 7 (2h):** chủ đề mới SAP-C03 (docs + lab nhỏ)
- **Chủ nhật (2h):** ôn lại ghi chú tuần, vẽ lại 1–2 sơ đồ kiến trúc từ trí nhớ

---

## Tuần 1 (23/11 – 29/11) · Identity & Multi-account

- [ ] T2 – Giới thiệu khóa + IAM nâng cao (policy evaluation, permission boundaries)
- [ ] T3 – STS, cross-account roles, Identity Federation (SAML, OIDC)
- [ ] T4 – IAM Identity Center, Directory Service (AD Connector, Managed AD)
- [ ] T5 – AWS Organizations, SCP, (RCP), tag policies
- [ ] T6 – Control Tower, RAM, multi-account strategy
- [ ] T7 – 🆕 Đọc **exam guide SAP-C03**, tô màu task: biết rõ / mơ hồ / chưa biết
- [ ] CN – Ôn tập: vẽ sơ đồ landing zone đa tài khoản từ trí nhớ

## Tuần 2 (30/11 – 06/12) · Security & Networking

- [ ] T2 – KMS (multi-Region keys, key policies, grants), CloudHSM
- [ ] T3 – Secrets Manager, SSM Parameter Store, ACM, WAF, Shield, Firewall Manager
- [ ] T4 – GuardDuty, Security Hub, Inspector, Macie, Detective, Config
- [ ] T5 – VPC nâng cao, Transit Gateway, VPC peering vs TGW
- [ ] T6 – Direct Connect, Site-to-Site VPN, PrivateLink, Route 53 Resolver (DNS hybrid)
- [ ] T7 – 🆕 **VPC Lattice**, **ECS Service Connect**, Network Firewall
- [ ] CN – Ôn tập: sơ đồ kết nối hybrid on-prem ↔ nhiều VPC

## Tuần 3 (07/12 – 13/12) · Compute, Storage, Database

- [ ] T2 – EC2 (placement groups, purchasing options), Auto Scaling, ELB (ALB/NLB/GWLB)
- [ ] T3 – ECS/EKS/Fargate, Lambda, API Gateway
- [ ] T4 – S3 nâng cao, EBS/EFS/FSx, Storage Gateway
- [ ] T5 – CloudFront, Global Accelerator, ElastiCache
- [ ] T6 – RDS, Aurora (Global Database), DynamoDB (Global Tables, DAX)
- [ ] T7 – 🆕 **KMS post-quantum (ML-DSA, ML-KEM)**, **Lake Formation + Apache Iceberg**, **Clean Rooms**
- [ ] CN – Ôn tập: bảng so sánh các lựa chọn database theo use case

## Tuần 4 (14/12 – 20/12) · Integration, Data, Monitoring, Deployment

- [ ] T2 – SQS, SNS, EventBridge, Amazon MQ
- [ ] T3 – Step Functions, Kinesis, MSK, Firehose
- [ ] T4 – Glue, Athena, Redshift, EMR, QuickSight, OpenSearch
- [ ] T5 – CloudWatch, X-Ray, CloudTrail, Health
- [ ] T6 – CloudFormation (StackSets), Elastic Beanstalk, CodePipeline, Systems Manager
- [ ] T7 – 🆕 **Step Functions durable workflows**, **serverless data pipelines**, **kiến trúc SaaS multi-tenant**
- [ ] CN – Ôn tập: sơ đồ event-driven architecture

## Tuần 5 (21/12 – 27/12) · Cost, Migration, DR — Hoàn thành khóa 🎯

- [ ] T2 – Cost Explorer, Budgets, CUR, Savings Plans/RI, Compute Optimizer, Trusted Advisor
- [ ] T3 – Migration: 7R, Migration Hub, Application Discovery Service, MGN
- [ ] T4 – DMS, SCT, DataSync, Transfer Family, Snow Family
- [ ] T5 – Disaster Recovery: Backup & Restore, Pilot Light, Warm Standby, Multi-site; AWS Backup, Elastic Disaster Recovery
- [ ] T6 – Các bài còn lại của khóa + exam tips của Stephane
- [ ] T7 – 🆕 **Fault Injection Service**, **Resilience Hub**, **Application Recovery Controller**, **SSM runbooks**
- [ ] CN – Ôn tập: bảng DR (RPO/RTO/chi phí) + checklist migration

## Tuần 6 (28/12 – 03/01) · GenAI & Agentic AI (mới hoàn toàn trong C03)

- [ ] T2 – Amazon Bedrock: model access, on-demand vs provisioned throughput, cross-Region inference
- [ ] T3 – Bedrock Knowledge Bases & kiến trúc **RAG** (vector store: OpenSearch Serverless, Aurora pgvector…)
- [ ] T4 – **Bedrock Guardrails**, bảo mật & quyền riêng tư dữ liệu cho GenAI
- [ ] T5 – **Bedrock Agents / AgentCore**, human-in-the-loop workflows
- [ ] T6 – Giám sát AI/ML, chi phí GenAI, SageMaker ở mức kiến trúc
- [ ] T7 – Lab: dựng 1 Knowledge Base RAG đơn giản trên Bedrock
- [ ] CN – Ôn tập: sơ đồ kiến trúc RAG + agent trong môi trường enterprise

## Tuần 7 (04/01 – 10/01) · DevSecOps & Operational Excellence

- [ ] T2 – Pipeline triển khai **đa tài khoản** (CodePipeline cross-account, StackSets)
- [ ] T3 – **Pipeline vulnerability scanning**, bảo mật container image (ECR scanning, Inspector)
- [ ] T4 – Blue/green, canary, rollback (CodeDeploy, Lambda alias, ECS)
- [ ] T5 – Observability tập trung đa tài khoản (CloudWatch cross-account, OAM)
- [ ] T6 – Quản trị & tuân thủ: Config conformance packs, Audit Manager, Control Tower controls
- [ ] T7 – Lab: FIS experiment đơn giản **hoặc** VPC Lattice service network
- [ ] CN – Đọc whitepaper: *AWS Well-Architected Framework* (tập trung 6 trụ cột)

## Tuần 8 (11/01 – 17/01) · Đào sâu domain yếu

- [ ] T2 – Domain yếu nhất #1 (dựa trên kết quả thi thử)
- [ ] T3 – Domain yếu nhất #1 (tiếp)
- [ ] T4 – Domain yếu nhất #2
- [ ] T5 – Domain yếu nhất #3
- [ ] T6 – Đọc lại file "lỗi sai" (`notes/mistakes.md`)
- [ ] T7 – Whitepaper: *Disaster Recovery of Workloads on AWS* + *Organizing Your AWS Environment Using Multiple Accounts*
- [ ] CN – Đối chiếu lại exam guide C03: mọi task đã "biết rõ" chưa?

## Tuần 9 (18/01 – 22/01) · Tổng ôn

- [ ] T2 – Ôn Domain 1 & 2 (cloud-native, security/governance)
- [ ] T3 – Ôn Domain 3 & 4 (cost, resilience/migration)
- [ ] T4 – Ôn Domain 5 (operational excellence) + các chủ đề mới C03
- [ ] T5 – Đọc lại toàn bộ file "lỗi sai"
- [ ] T6 – Nghỉ nhẹ, xem lại cheat-sheet, kiểm tra lịch thi & giấy tờ
- [ ] **T7 (23/01) – THI SAP-C03** 🎓

---

## Mẹo

- Phần trùng SAA đã chắc → xem 1.5x hoặc chỉ đọc slide PDF; dồn thời gian cho **multi-account, hybrid networking, migration, GenAI**.
- Tuần 5–6 trùng Giáng sinh & Tết dương lịch — nếu được nghỉ hãy học trước; nếu bận, đây là tuần dễ trễ nhất.
- Trước khi đặt lịch, kiểm tra trong tài khoản AWS Certification: **voucher 50%** từ SAA và **ESL +30 phút** (phải đăng ký trước khi book).
