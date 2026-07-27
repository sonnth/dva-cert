# AWS DVA-C02 Exam Tips & Tricks - Domain 4: Refactoring & Troubleshooting

## 📌 Domain 4: Refactoring & Troubleshooting (Troubleshooting)

### 🧩 Topic: Auto Scaling + ALB Health Checks

#### 🔑 Keywords Nhận Diện
* `ALB Target Group reports unhealthy instances` / `Auto Scaling does not replace instances`: Target group báo instance unhealthy nhưng Auto Scaling không replace/terminate.
* `Health check mismatch`: Sự khác nhau giữa EC2 Status Checks và ELB (Target Group) Health Checks.

#### 💡 Tip & Trick Chọn Đáp Án
* **Kích hoạt ELB Health Check:** Mặc định Auto Scaling Group chỉ sử dụng **EC2 status checks** (chỉ phát hiện lỗi phần cứng/OS, không phát hiện lỗi app). Cần đổi **Health Check Type = ELB** trên Auto Scaling Group để nó terminate và replace các instance bị unhealthy từ góc nhìn của ALB.
* **Cảnh giác với bẫy:** Tránh chọn các đáp án như *Cooldown*, *Lifecycle Hook*, hoặc *Grace Period* vì chúng không quyết định việc Auto Scaling có xem instance là unhealthy để thay thế hay không.

### 🧩 Topic: CloudWatch Logs Alerts (Metric Filters + Alarms + SNS)

#### 🔑 Keywords Nhận Diện
* `receive notification by email when [word] appears in log lines`: Nhận thông báo qua email khi một từ khóa (ví dụ: `ERROR`) xuất hiện trong log.
* `CloudWatch Logs` + `SNS topic`: Gửi log/metric từ CloudWatch sang SNS để thông báo.

#### 💡 Tip & Trick Chọn Đáp Án
* **Quy trình chuẩn 3 bước:**
  1. Chọn Log Group thích hợp.
  2. Tạo **CloudWatch Metric Filter** với từ khóa cần lọc (ví dụ: `ERROR`). Bộ lọc này sẽ chuyển các sự kiện khớp từ khóa thành một Metric số lượng (Metric Value).
  3. Tạo **CloudWatch Alarm** dựa trên Metric đó (khi giá trị >= 1) và cấu hình action gửi thông báo đến **SNS Topic** đã subscribe email nhóm dev.
  * *CloudWatch Logs Insights:* Là công cụ dùng để truy vấn (query) thủ công và phân tích dữ liệu log lịch sử, không dùng để cấu hình kích hoạt cảnh báo tự động theo thời gian thực (real-time).
  * *Cảnh báo trực tiếp từ Log Group / SNS Subscription Filter:* CloudWatch Alarm không thể kết nối trực tiếp hoặc lọc từ dữ liệu log thô (raw logs) mà bắt buộc phải đi qua **Metric Filter** để chuyển đổi thành dữ liệu dạng Metric trước.

### 🧩 Topic: Lambda VPC Connectivity & Permissions Troubleshooting

#### 🔑 Keywords Nhận Diện
* `Lambda function in VPC mode` + `connect to Amazon RDS` (in private subnet, port 1433/3306/5432).
* `reports an error when it tries to connect` (gặp lỗi kết nối khi kiểm tra).
* `diagnose this issue` / `troubleshoot connectivity`.

#### 💡 Tip & Trick Chọn Đáp Án
* **Phương án kết nối Lambda tới Database (Aurora/RDS) trong Private Subnet:**
  * Lambda function phải được cấu hình chạy trong cùng VPC với Database (truy cập các private subnets).
  * **Cấu hình Security Group (SG) hợp lệ:** Có 2 cách thiết kế SG chuẩn:
    1. **Cách 1 (Dùng chung 1 Security Group - Self-Referencing):** Gắn cùng một Security Group (ví dụ: `SG1`) cho cả Lambda function và Database. Cấu hình quy tắc của `SG1` cho phép Inbound và Outbound TCP trên cổng database (ví dụ: `3306` cho MySQL/Aurora) từ/đến nguồn là chính `SG1` (Self-referencing rule).
    2. **Cách 2 (Dùng 2 Security Groups riêng biệt):** Gắn `SG1` cho Lambda, `SG2` cho Database. **Database SG (SG2)** phải có quy tắc **Inbound** cho phép nhận TCP trên cổng database (ví dụ: `3306`) từ nguồn là `SG1` (Lambda SG). Lambda SG (`SG1`) chỉ cần cho phép Outbound.
* **Loại trừ các bẫy thường gặp:**
  * *Ngược chiều Inbound/Outbound:* Cảnh giác với phương án cấu hình ngược (ví dụ: *"Add inbound rule to Lambda SG (SG1) to allow TCP traffic from DB port"*). Lambda là client khởi tạo kết nối đi ra, nó không cần inbound rule cho cổng database.
  * *Tạo VPC mới (VPC Peering):* Không cần thiết và quá phức tạp khi hoàn toàn có thể chạy Lambda trực tiếp trong VPC hiện tại.
  * *NAT Gateway & Public Access:* Kết nối nội bộ trong VPC không cần NAT Gateway và tuyệt đối không bật Public Access cho database (vừa không cần thiết vừa vi phạm bảo mật).
  * *Xuất dữ liệu ra S3 để query:* Rườm rà, tăng latency lớn, không tối ưu cho truy vấn real-time.
  * *Quyền API quản trị RDS* (ví dụ: `DescribeDBInstances`, `ModifyDBInstance`): Kết nối database là ở tầng mạng qua database driver chứ không phải qua AWS SDK/API quản trị của RDS.
  * *Quyền IAM Execution Role:* Lambda trong VPC cần tạo Elastic Network Interface (ENI) để giao tiếp. Do đó, Execution Role của Lambda phải được cấp các quyền: `ec2:CreateNetworkInterface`, `ec2:DescribeNetworkInterfaces`, và `ec2:DeleteNetworkInterface` (có sẵn trong AWS-managed policy `AWSLambdaVPCAccessExecutionRole`).

### 🧩 Topic: Distributed Tracing with AWS X-Ray (Service Map & Custom Subsegments)

#### 🔑 Keywords Nhận Diện
* `microservices-based application` (ứng dụng phân tán dựa trên kiến trúc microservices).
* `intermittent HTTP 503` / `errors during execution` (lỗi 5xx hoặc 503 xuất hiện không liên tục).
* `logs do not provide enough detail to determine the specific cause` (log hệ thống không có đủ chi tiết để biết nguyên nhân cụ thể).
* `determine the root cause of the errors`.

#### 💡 Tip & Trick Chọn Đáp Án
* **Sử dụng X-Ray Service Map + Custom Subsegments:**
  * Khi các file log thông thường (CloudWatch Logs) thiếu chi tiết/không đủ thông tin để tìm ra lỗi của một ứng dụng phân tán, ta cần dùng **AWS X-Ray** để trace (theo dõi) toàn bộ đường đi của request.
  * **Service Map** trong X-Ray sẽ hiển thị bản đồ trực quan các node dịch vụ kết nối với nhau và khoanh vùng node nào đang bị lỗi/đỏ/timeout.
  * Việc tích hợp thêm **Custom Subsegments** (phân đoạn nhỏ tùy chỉnh trong code) giúp chia nhỏ luồng xử lý bên trong một microservice (ví dụ: chia thành subsegment gọi DB, subsegment gọi API bên thứ ba). Từ đó chỉ ra chính xác phần logic hay truy vấn nào trong code đang bị lỗi hoặc gây trễ/timeout (nguyên nhân của lỗi 503).
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *Chỉ cấu hình X-Ray với Default Sampling Rate:* Mặc định sampling rate của X-Ray chỉ kiểm soát số lượng request được trace (1 request/giây + 5%), nó không tự động bổ sung chi tiết sâu bên trong code nếu không được viết code instrument với **Custom Subsegments**.
  * *Tạo CloudWatch Alarm:* Chỉ có tác dụng cảnh báo khi tỉ lệ lỗi vượt ngưỡng, hoàn toàn không hỗ trợ phân tích tìm nguyên nhân gốc rễ (root cause) của lỗi.
  * *Quét/Filter CloudWatch Logs:* Đề bài đã cho sẵn dữ kiện là logs không có đủ chi tiết, nên việc lọc hay phân tích logs cũ sẽ không mang lại kết quả.

### 🧩 Topic: Aggregate Log Analysis (CloudWatch Logs Insights & Dashboards)

#### 🔑 Keywords Nhận Diện
* `log and process the incoming orders` (sử dụng Lambda hoặc ứng dụng khác để ghi log sự kiện/lỗi vào CloudWatch Logs).
* `show how many unique customers this problem affects each day` (yêu cầu đếm số lượng thực thể duy nhất như unique customers bị ảnh hưởng theo ngày/khoảng thời gian).
* `create a dashboard` (tạo bảng điều khiển trực quan hóa kết quả).

#### 💡 Tip & Trick Chọn Đáp Án
* **Sử dụng CloudWatch Logs Insights làm nguồn cho Dashboard:**
  * Khi cần phân tích, tổng hợp dữ liệu ứng dụng nâng cao từ logs (như đếm số lượng khách hàng duy nhất `count_distinct(customerId)` bị lỗi theo chu kỳ thời gian `bin(1d)`), **CloudWatch Logs Insights** là công cụ truy vấn tối ưu và nhanh chóng nhất.
  * Kết quả truy vấn từ Logs Insights có thể dễ dàng được pin/add trực tiếp vào **CloudWatch Dashboard** để tạo biểu đồ theo dõi hàng ngày.
* **Loại trừ các tùy chọn sai/không tối ưu:**
  * *Tạo Custom CloudWatch Metrics trên DynamoDB Streams:* DynamoDB Streams chỉ gửi dữ liệu thay đổi của bảng, không hỗ trợ sinh các custom metric ở tầng ứng dụng (như phân tích unique customer ID) để tự động tính toán aggregate ngay trên stream.
  * *Sử dụng AWS CloudTrail + Athena:* CloudTrail chỉ ghi nhận các cuộc gọi API quản trị hoặc quản lý tài nguyên của AWS (Management/Data Events), không ghi nhận nội dung logs nghiệp vụ chi tiết của ứng dụng (như quantity, customerId bên trong payload).
  * *Sử dụng Amazon EventBridge:* EventBridge dùng để định tuyến sự kiện (routing events) đến các target, không hỗ trợ chức năng lưu trữ, truy vấn aggregate (gom nhóm unique) hay vẽ biểu đồ trực tiếp lên Dashboard.

