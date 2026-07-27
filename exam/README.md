# 🎓 AWS Certified Developer – Associate (DVA-C02) Study Notes

Đây là tài liệu tổng hợp lộ trình ôn thi và các **Tip & Trick** được đúc kết trực tiếp từ quá trình luyện đề thực tế của bạn.

> [!NOTE]
> Hiện tại tôi đang luyện đề. Mỗi câu hỏi tôi gửi lên sẽ có những tip & trick để làm dựa trên những keywords trong câu hỏi. Các tip này đang được lưu trong các file markdown chia theo từng Domain tương ứng dưới đây. Sau mỗi câu hỏi tôi gửi lên, hãy update lại file tương ứng để sau khi luyện đề xong có thể đọc lại.

---

## 📌 1. Cấu Trúc Đề Thi DVA-C02

| Domain | Tỷ lệ | Dịch vụ chính |
| :--- | :---: | :--- |
| **Domain 1: Deployment** | 22% | CI/CD (CodePipeline, CodeBuild, CodeDeploy), Elastic Beanstalk, CloudFormation, SAM. |
| **Domain 2: Security** | 26% | IAM, KMS, Secrets Manager, Cognito, SSM Parameter Store. |
| **Domain 3: Development with AWS Services** | 30% | Serverless (Lambda, API Gateway, Step Functions), DynamoDB, SQS, SNS, Kinesis, EventBridge. |
| **Domain 4: Refactoring & Troubleshooting** | 22% | CloudWatch, X-Ray, CloudTrail. |

---

## 💡 2. Tài liệu học tập & Tips (Chi tiết từng Domain)
* 📁 [Domain 1: Deployment](./domain1_deployment.md) — Tổng hợp Tips về CI/CD, Beanstalk, CloudFormation.
* 📁 [Domain 2: Security](./domain2_security.md) — Tổng hợp Tips về IAM, KMS, Secrets Manager, Cognito, SSM.
* 📁 [Domain 3: Development with AWS Services](./domain3_development.md) — Tổng hợp Tips về Serverless, DynamoDB, SQS, SNS, Kinesis.
* 📁 [Domain 4: Refactoring & Troubleshooting](./domain4_troubleshooting.md) — Tổng hợp Tips về CloudWatch, X-Ray, CloudTrail.

---

## 🚀 3. Các Dịch Vụ "Trọng Tâm" Cần Nằm Lòng

### ⚡ Serverless & Compute
* **AWS Lambda**:
  * *Execution timeout*: Tối đa 15 phút (900 giây).
  * *Features*: Environment Variables, Layers, Concurrency (Reserved vs Provisioned), Dead Letter Queues (DLQ).
* **Amazon API Gateway**:
  * *Features*: Stages, Throttling (tùy chỉnh giới hạn request), Caching.
  * *Authorizers*: Cognito Authorizer vs Lambda Authorizer.
* **AWS Step Functions**: Quản lý state machine phối hợp các workflow phức tạp.

### 💾 Databases & Storage
* **Amazon DynamoDB** *(Trọng tâm lớn)*:
  * Phân biệt **LSI** (Local Secondary Index) và **GSI** (Global Secondary Index).
  * DynamoDB Streams + Lambda trigger.
  * TTL (Time to Live) tự động xóa data cũ.
  * Cách tính toán **RCU / WCU** (Read/Write Capacity Units) cho chế độ Provisioned.
* **Amazon S3**:
  * Pre-signed URLs (download/upload bảo mật có thời hạn).
  * Lifecycle Policies (chuyển đổi storage class, xóa tự động).
  * CORS (Cross-Origin Resource Sharing).
  * Encryption: SSE-S3 (AWS managed keys), SSE-KMS (KMS keys), SSE-C (Customer provided keys).

### 🔒 Security & Configuration
* **AWS KMS**: Quản lý khóa mã hóa. Phân biệt AWS Managed Keys và Customer Managed Keys (CMK).
* **Amazon Cognito**:
  * *User Pools*: Đăng ký, đăng nhập và xác thực người dùng (Authentication).
  * *Identity Pools*: Cấp quyền AWS credentials tạm thời (Authorization) để truy cập trực tiếp tài nguyên AWS (như S3, DynamoDB).
* **AWS Secrets Manager** vs **SSM Parameter Store**:
  * *Secrets Manager*: Hỗ trợ tự động xoay vòng khóa (rotation), có tính phí.
  * *Parameter Store*: Dùng cho cấu hình dạng text/plain hoặc encrypted string (SecureString), miễn phí đối với standard parameters.

### 🔄 CI/CD & Dev Tools
* **AppSpec.yml** (CodeDeploy) vs **Buildspec.yml** (CodeBuild): Cần phân biệt rõ file cấu hình của dịch vụ nào và các phase/hooks tương ứng bên trong.
* **AWS CloudFormation & SAM**: SAM (Serverless Application Model) là phần mở rộng của CloudFormation chuyên tối ưu cho việc viết và deploy các ứng dụng Serverless.