# Tóm tắt Khóa học AWS Certified Developer Associate

Tài liệu này tổng hợp các kiến thức cốt lõi và ý chính từ tất cả các ghi chú của khóa học AWS Certified Developer Associate.

## 1. Cơ bản về AWS & Phân quyền (IAM)
- **Khu vực (Regions) & Vùng tính khả dụng (AZs):** Các trung tâm dữ liệu phân bổ toàn cầu. Đa số dịch vụ gắn với một Region cụ thể, ngoại trừ một số dịch vụ toàn cầu như IAM, Route 53, CloudFront.
- **IAM (Identity and Access Management):**
  - Quản lý tài khoản (Users), nhóm (Groups), và chính sách (Policies - định dạng JSON).
  - **IAM Roles:** Dùng để cấp quyền cho các dịch vụ AWS (như EC2, Lambda) thay vì sử dụng Access Key.
  - **Thực hành tốt nhất:** Luôn áp dụng nguyên tắc "đặc quyền tối thiểu" (least privilege), không bao giờ nhúng thẳng (hardcode) secret keys vào code.

## 2. Dịch vụ Máy chủ tính toán (Compute)
- **EC2 & Lưu trữ (EBS/EFS):**
  - **EC2:** Máy chủ ảo. Các loại giá: On-demand (trả theo dùng), Reserved (cam kết), Spot (rẻ nhất, dễ ngắt), Dedicated.
  - **EBS:** Ổ cứng mạng gắn vào một EC2. Hỗ trợ snapshot.
  - **EFS:** Hệ thống file chia sẻ, có thể gắn vào nhiều EC2 cùng lúc.
- **ELB & ASG:**
  - **ALB (Application Load Balancer):** Layer 7 (HTTP/HTTPS), lý tưởng cho microservices, routing linh hoạt.
  - **NLB (Network Load Balancer):** Layer 4, dành cho hiệu năng cực cao và độ trễ siêu thấp.
  - **ASG (Auto Scaling Group):** Tự động mở rộng hoặc thu hẹp số lượng EC2 dựa trên tải (ví dụ: CPU, Network) giúp đảm bảo tính sẵn sàng cao (High Availability).
- **Elastic Beanstalk:** Dịch vụ PaaS. Dev chỉ việc tải code lên, AWS lo cấu hình máy chủ, mạng, cân bằng tải. Các chế độ triển khai đa dạng (All at once, Rolling, Blue/Green, Immutable).
- **Container (ECS, ECR & Fargate):** Quản lý Docker. Fargate cho phép chạy container theo dạng serverless mà không cần quản lý hạ tầng EC2.

## 3. Kiến trúc Serverless & API
- **AWS Lambda:** Hàm thực thi mã nguồn phản hồi theo sự kiện (tối đa 15 phút/lần). Có phiên bản (Version) và bí danh (Alias). Khắc phục Cold Start với Provisioned Concurrency.
- **API Gateway:** Tạo REST API hoặc WebSocket. Serverless, tích hợp hoàn hảo với Lambda, hỗ trợ Versioning, Caching, Throttling và xác thực với Cognito/IAM.
- **SAM (Serverless Application Model):** Khung ứng dụng định dạng YAML (mở rộng từ CloudFormation), có khả năng chạy và kiểm thử nội bộ (local).
- **Step Functions:** Hệ thống điều phối luồng công việc (State Machine) kết nối nhiều dịch vụ AWS. Quản lý xử lý lỗi (Retry/Catch) mạnh mẽ.
- **AppSync:** Dịch vụ API GraphQL được quản lý toàn diện (dành cho lấy dữ liệu theo thời gian thực).

## 4. Lưu trữ (Storage) & Cơ sở dữ liệu (Database)
- **Amazon S3:** Dịch vụ lưu trữ đối tượng (Object storage) siêu bền vững.
  - **Các lớp (Classes):** Standard, Standard-IA, Glacier (Lưu trữ dài hạn)...
  - **Tính năng nổi bật:** Versioning (quản lý phiên bản), Replication (sao chép), Lifecycle rules (quy tắc vòng đời).
  - **Bảo mật:** Bucket/IAM Policies, mã hóa nhiều cấp (SSE-S3, SSE-KMS, SSE-C).
- **DynamoDB:** CSDL NoSQL serverless. Hiệu năng tính bằng mili-giây. Dùng Partition/Sort Key. Có DynamoDB Streams (dành cho luồng thay đổi) và DAX (bộ đệm cực nhanh).
- **RDS, Aurora & ElastiCache:**
  - **RDS:** Quản lý CSDL quan hệ (MySQL, PostgreSQL,...). Hỗ trợ Read Replicas & Multi-AZ.
  - **Aurora:** CSDL quan hệ do AWS phát triển tối ưu hoá trên Cloud, hiệu năng gấp 5 lần MySQL. Tự động lưu 6 bản sao lưu trên 3 vùng khả dụng.
  - **ElastiCache:** Hệ thống Cache (Redis, Memcached) giảm tải đọc cho CSDL chính.

## 5. Tích hợp & Thông điệp (Integration & Messaging)
- Hỗ trợ xây dựng kiến trúc tách rời (Decoupled architecture).
- **SQS (Simple Queue Service):** Hàng đợi thông điệp. Standard (không giới hạn, có thể lặp/sai thứ tự) và FIFO (giữ đúng thứ tự, không lặp). Hỗ trợ Dead Letter Queues và Long Polling.
- **SNS (Simple Notification Service):** Mô hình Pub/Sub. Đẩy (push) thông điệp đến nhiều nguồn đăng ký.
- **Kinesis:** Phân tích dữ liệu luồng khối lượng lớn (Real-time). Gồm Kinesis Data Streams (cần viết code đọc/ghi) và Kinesis Data Firehose (nạp dữ liệu tự động vào S3/Redshift...).

## 6. Mạng lưới & Phân phối (Networking & CDN)
- **VPC (Virtual Private Cloud):** Mạng riêng ảo trên Cloud.
  - **Bảo mật mạng:** Security Groups (tường lửa cho EC2, chỉ ALLOW) và NACL (tường lửa cho Subnet, có ALLOW/DENY).
  - VPC Flow Logs (lưu log mạng) và VPC Peering (kết nối giữa các mạng).
- **Route 53:** Dịch vụ tên miền (DNS) tin cậy 100%. Các chính sách định tuyến (Simple, Weighted, Latency, Geolocation,...).
- **CloudFront:** Mạng phân phối nội dung (CDN) toàn cầu để giảm độ trễ, lưu vào bộ đệm ở vùng biên (Edge).

## 7. Bảo mật (Security & Encryption)
- **KMS (Key Management Service):** Dịch vụ quản lý khóa mã hóa. Dùng mô hình Envelope Encryption với dữ liệu > 4KB.
- **SSM Parameter Store:** Lưu trữ cấu hình/bảo mật đơn giản (có mã hóa).
- **Secrets Manager:** Quản lý cấu hình mật, mạnh mẽ nhất khi cần thiết lập đổi mật khẩu định kỳ tự động (ví dụ với RDS).
- **Cognito:** 
  - **User Pools:** Chức năng đăng nhập, đăng ký cho người dùng.
  - **Identity Pools:** Cấp chứng chỉ AWS tạm thời để người dùng ứng dụng di động/web được phép truy cập các dịch vụ AWS ẩn danh.

## 8. Giám sát & Đo lường (Monitoring & Observability)
- **CloudWatch:** Thu thập Metric (chỉ số), tạo Alarm (cảnh báo) và CloudWatch Logs (lưu log ứng dụng).
- **X-Ray:** Công cụ theo dõi dấu vết (Distributed Tracing), giúp lập bản đồ các cuộc gọi giữa các dịch vụ trong hệ thống Microservices, phân tích gỡ rối các điểm nghẽn cổ chai (bottleneck).
- **CloudTrail:** Ghi log quản trị viên, cung cấp dấu vết Audit toàn bộ các thao tác API trong tài khoản.

## 9. Công cụ phát triển & CI/CD (Developer Tools)
- **CodeCommit:** Kho lưu trữ git riêng tư.
- **CodeBuild:** Biên dịch, test và đóng gói mã nguồn.
- **CodeDeploy:** Triển khai tự động bản build tới EC2, Lambda hoặc ECS.
- **CodePipeline:** Trình điều phối luồng công việc CI/CD tự động từ kho lưu trữ đến hệ thống thực thi.
- **CloudFormation:** Quản lý hạ tầng bằng code (IaC) thông qua khai báo YAML/JSON. Tái tạo nhanh môi trường làm việc.
- **CDK:** Phát triển/cấu hình AWS thông qua ngôn ngữ lập trình lập trình hiện đại (TypeScript, Python...) thay vì YAML thô.
