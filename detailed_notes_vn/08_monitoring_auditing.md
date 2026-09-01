# Phần 8: Giám sát, Kiểm toán và Khắc phục sự cố (Monitoring & Auditing)

## 1. Amazon CloudWatch
Dịch vụ giám sát trung tâm cho toàn bộ tài nguyên trên AWS và ứng dụng của bạn.

### CloudWatch Metrics & Alarms (Số liệu & Cảnh báo)
- **Metrics (Số liệu):** Biến số cần theo dõi (VD: CPUUtilization, NetworkIn). Mỗi Metric thuộc về một *Namespace* và có thể phân chia chi tiết theo *Dimensions* (VD: ID máy EC2).
- Mặc định EC2 ghi nhận số liệu mỗi 5 phút. Nếu mua **Detailed Monitoring** (Giám sát chi tiết) sẽ ghi nhận mỗi 1 phút (Free tier có 10 metrics chi tiết).
- **Custom Metrics (Số liệu tùy chỉnh):** Dùng API `PutMetricData` để tự đẩy dữ liệu lên (VD: Lượng RAM trống, số lượng người đang online). Độ phân giải chuẩn là 1 phút, cao nhất (High resolution) là 1 giây. Dữ liệu có thể được gửi lùi về quá khứ tối đa 2 tuần và tiến tới tương lai 2 giờ.
- **CloudWatch Alarms (Cảnh báo):** Gắn với 1 metric. Khi vượt ngưỡng, Alarm đổi trạng thái sang `ALARM` và kích hoạt hành động (VD: Khởi động lại EC2, Kích hoạt Auto Scaling, Gửi email qua SNS). 
  - *Composite Alarms:* Kết hợp nhiều Alarm nhỏ (bằng điều kiện AND/OR) để giảm bớt thông báo rác (Alarm noise).

### CloudWatch Logs
Nơi thu thập và lưu trữ toàn bộ các tập tin nhật ký (Log files) từ các ứng dụng và dịch vụ.
- **Tổ chức:** Nhóm thành *Log Groups* (Nhóm ứng dụng) -> *Log Streams* (Từng máy tính/Container cụ thể).
- Có thể thiết lập thời hạn xóa (Expiration) từ 1 ngày đến 10 năm.
- **CloudWatch Unified Agent:** Ứng dụng nhỏ cài trên EC2/On-premise để đẩy Log files và các thông số chuyên sâu (như RAM, Disk, Swap) lên CloudWatch.
- **Log Insights:** Ngôn ngữ truy vấn mạnh mẽ để lục tìm log (VD: Đếm số lỗi `ERROR` trong 1 tuần qua).
- **Metric Filters (Bộ lọc):** Dùng để tự động đếm các từ khóa trong Log (VD: Đếm số IP tấn công hoặc số lỗi) rồi vẽ thành một Metric riêng (từ đó gắn vào Alarm).
- **Logs Subscription:** Để phân tích log theo thời gian thực (Real-time), CloudWatch có thể "đẩy" luồng log thẳng vào Kinesis, Lambda hoặc OpenSearch. (Tính năng Export to S3 rất chậm, mất tới 12 tiếng nên không phù hợp cho Real-time).

### CloudWatch Synthetics
- Tạo các con bọ (Canaries - chạy bằng trình duyệt giả lập Headless Chrome) liên tục truy cập vào website của bạn để chụp màn hình, kiểm tra link hỏng, đo độ trễ và báo động nếu website sập. 

## 2. AWS X-Ray
Dịch vụ theo dõi chuỗi phân tán (Distributed Tracing). Vô cùng quan trọng cho kiến trúc Microservices.
- **Lợi ích:** Giải quyết câu hỏi "Khách hàng than phiền ứng dụng load chậm, nhưng lỗi nằm ở service nào?". X-Ray vẽ ra *Service Map* (Sơ đồ dịch vụ) trực quan. Nó ghi nhận tỷ lệ lỗi, độ trễ giữa các kết nối.
- **Cấu trúc:** Một yêu cầu đi xuyên hệ thống tạo thành 1 *Trace*. Mỗi dịch vụ ghi lại 1 *Segment* (Phân đoạn), bên trong chứa nhiều *Subsegments* (Chi tiết gọi DB, gọi API ngoài).
- **Cách cài đặt:** 
  1. Chèn *X-Ray SDK* vào mã nguồn (Java, Python, Node...). SDK tự động chặn (intercept) mọi câu lệnh gọi AWS/HTTP/DB.
  2. Cài đặt và bật *X-Ray Daemon* (Bên trong EC2, ECS, Beanstalk) để gửi dữ liệu lên AWS.
- **Annotations & Metadata:** *Annotations* là dạng key-value dùng để MỞ RỘNG TÌM KIẾM (VD: Gắn mã số GameID để dễ tìm trace của game đó). *Metadata* chỉ dùng lưu thêm thông tin chứ không thể tìm kiếm.
- **Sampling Rules (Luật lấy mẫu):** Mặc định, X-Ray chỉ ghi 1 request mỗi giây (Reservoir) và 5% request vượt mức (Rate). Giúp tiết kiệm chi phí cực lớn mà vẫn phát hiện được lỗi. Có thể chỉnh sửa luật lấy mẫu ngay trên console mà không cần sửa code.

## 3. AWS CloudTrail
Dịch vụ kiểm toán (Auditing). "Ai đã làm gì, vào lúc nào, ở đâu?".
- Ghi nhận toàn bộ lệnh gọi API vào hệ thống AWS (Từ Bảng điều khiển Console, CLI hay SDK). Bật sẵn mặc định.
- Vô cùng hữu ích để truy vết thủ phạm đã vô tình xóa Database hay thay đổi cấu hình bảo mật.
- **Các loại sự kiện:**
  - *Management Events (Sự kiện quản lý):* Ghi nhận thao tác đổi cấu hình, tạo xóa tài nguyên (Bật mặc định).
  - *Data Events (Sự kiện dữ liệu):* Các thao tác mức độ cao như đọc/ghi 1 file cụ thể trên S3, hoặc thực thi 1 hàm Lambda. (Mặc định tắt, vì sinh ra tỷ tỷ log, nếu bật sẽ tính phí rất lớn).
  - *CloudTrail Insights:* Dùng Trí tuệ nhân tạo (AI) học thói quen dùng AWS của bạn. Nếu phát hiện hành vi dùng API đột biến (bị hack) sẽ cảnh báo.
- Dữ liệu giữ lại miễn phí trong 90 ngày. Có thể đẩy sang S3 (Và dùng Athena để truy vấn SQL) để lưu trữ vĩnh viễn và kiểm toán pháp lý.

## 4. Amazon EventBridge (Trước đây là CloudWatch Events)
Trái tim của kiến trúc hướng sự kiện (Event-driven).
- **Schedule (Lịch trình):** Tạo cronjob để hẹn giờ gọi Lambda (VD: Mỗi 8h sáng chạy dọn rác).
- **Event Rules (Luật sự kiện):** Bắt ngay lập tức một sự kiện từ các dịch vụ (VD: Khi một file upload lên S3, hoặc một EC2 bị tắt) để kích hoạt tự động (Gọi Lambda, gửi tin SNS/SQS).
- **Schema Registry:** Tính năng tự động học cấu trúc của sự kiện và sinh ra code (Java/Python/TypeScript) để Developer chỉ việc sử dụng thẳng trong ứng dụng, tránh mất thời gian dò dẫm cấu trúc JSON.
- Hỗ trợ bắt sự kiện từ các phần mềm quản lý công ty (SaaS) và hỗ trợ gom sự kiện từ nhiều tài khoản AWS khác nhau (Cross-account).

## 💡 Trích xuất trọng tâm luyện thi DVA-C02
- **Cảnh báo từ khóa log qua Email:** Quy trình chuẩn: Log Group -> Tạo **Metric Filter** đếm từ khóa (VD: ERROR) -> Gắn Metric vào **CloudWatch Alarm** -> Alarm gửi **SNS Topic**. Không dùng Logs Insights cho cảnh báo real-time.
- **Troubleshoot 503 Error phân tán:** Khi CloudWatch log không đủ chi tiết lỗi của Microservices, dùng **AWS X-Ray**. Sơ đồ **Service Map** sẽ hiển thị điểm nghẽn, và dùng **Custom Subsegments** trong code để mổ xẻ chính xác đoạn truy vấn nào gây lỗi.
- **Phân tích tổng hợp (Aggregate):** Khi cần tính toán (VD: Đếm số user bị lỗi mỗi ngày) trực tiếp từ Logs, hãy dùng **CloudWatch Logs Insights** để truy vấn và ghim kết quả lên **CloudWatch Dashboard**.
- **Lỗi kết nối Lambda VPC đến RDS:** Lambda trong Private Subnet gọi RDS/Aurora cùng VPC không cần NAT Gateway. Lỗi Timeout thường do Security Group. Cách cấu hình SG chuẩn: SG của Lambda cho phép Outbound TCP tới port DB, và SG của Database cho phép Inbound TCP từ port DB với nguồn là SG của Lambda (Và Lambda phải có quyền IAM tạo ENI `AWSLambdaVPCAccessExecutionRole`).
