# Phần 5: Tích hợp, Giao tiếp (Messaging) và Các dịch vụ khác

## 1. Amazon SQS (Hàng đợi thông điệp)
Dịch vụ hàng đợi lưu trữ tin nhắn giúp giải quyết bài toán "Quá tải" (Giảm chấn) và chia tách hệ thống (Decoupling) bằng mô hình Bất đồng bộ (Asynchronous).
- Người gửi (Producer) đẩy tin nhắn vào SQS -> Người nhận (Consumer / EC2 / Lambda) kéo (poll) tin nhắn về xử lý -> Xử lý xong thì phải gọi lệnh XÓA (DeleteMessage).
- Tích hợp cực tốt với Auto Scaling Group (Dùng số lượng tin nhắn trong hàng đợi để kích hoạt tự động thêm máy chủ EC2).

### Các loại SQS và tính năng
- **Standard (Tiêu chuẩn):** Thông lượng (Throughput) không giới hạn, nhưng có thể bị *trùng lặp* tin nhắn và *không đảm bảo thứ tự*.
- **FIFO (Vào trước ra trước):** Đảm bảo thứ tự tuyệt đối và không bị trùng (chống trùng lặp bằng ID). Tuy nhiên, thông lượng bị giới hạn (300 msg/s hoặc 3000 msg/s nếu gom nhóm). Cần truyền vào `Message Group ID`.
- **Visibility Timeout (Thời gian ẩn):** Khi một máy ảo (Consumer) lấy tin nhắn, tin nhắn đó bị "ẩn" đi (mặc định 30s) để các máy khác không lấy nhầm. Nếu máy đó chết hoặc xử lý quá 30s mà chưa xóa tin, tin nhắn sẽ hiện lại và bị xử lý lần 2. (Có thể xin thêm thời gian bằng API `ChangeMessageVisibility`).
- **Dead Letter Queue (DLQ):** Khi một tin nhắn bị xử lý lỗi quá nhiều lần (VD: 5 lần), nó sẽ tự động bị ném sang DLQ để kỹ sư vào kiểm tra, tránh làm kẹt hàng đợi chính.
- **Delay Queues:** Tạm ẩn tin nhắn mới gửi lên đến 15 phút rồi mới cho Consumer thấy.
- **Long Polling (Kéo dài thời gian đợi):** Consumer thay vì hỏi liên tục gây tốn tiền API, có thể thiết lập đợi 20 giây ở SQS để chờ tin nhắn. Giảm chi phí và giảm độ trễ rất hiệu quả.
- Tin nhắn tối đa 256KB. Dùng *SQS Extended Client* (tích hợp S3) nếu muốn gửi tin nhắn lớn hơn.

## 2. Amazon SNS (Dịch vụ thông báo)
Dịch vụ phát sóng theo mô hình "Xuất bản / Đăng ký" (Pub/Sub).
- Một tin nhắn gửi vào SNS (Topic) sẽ được "phát" (Push) ngay lập tức tới TẤT CẢ những người đăng ký (Email, SMS, HTTP Endpoint, SQS, Lambda).
- Không lưu trữ tin nhắn (Tin gửi đi nếu không ai nhận sẽ mất). 

### Fan-Out Pattern (Mô hình rẽ quạt)
Kết hợp sức mạnh của SNS và SQS: Đẩy 1 tin nhắn vào SNS -> SNS phát xuống 5 hàng đợi SQS khác nhau -> 5 nhóm máy chủ khác nhau nhận được cùng 1 tin nhắn và xử lý lưu trữ an toàn, có thể thử lại nếu lỗi.
- **Message Filtering:** Bạn có thể đặt luật (JSON) ở SNS để một hàng đợi SQS chỉ nhận các tin nhắn có thuộc tính nhất định (VD: chỉ nhận tin nhắn có `type = error`).

## 3. Amazon Kinesis
Dịch vụ xử lý luồng dữ liệu theo thời gian thực (Real-time Streaming) dùng cho Big Data.
- Kinesis Data Streams (KDS) và SQS giống nhau nhưng khác ở chỗ: SQS khi đọc xong thì tin nhắn mất đi và không thể 2 nhóm đọc cùng 1 hàng đợi (Ngoại trừ Fan-out). KDS lưu dữ liệu lại 365 ngày, hàng ngàn nhóm Consumer có thể cùng đọc và tua lại (Replay) dữ liệu cũ vô tư. 
- **Kinesis Data Streams (KDS):** Viết code để xử lý dữ liệu luồng (Analytics, AI). Khả năng mở rộng bằng cách thêm *Shard* (Phân mảnh). Đảm bảo thứ tự theo `Partition Key`.
- **Kinesis Data Firehose (KDF):** KHÔNG cần viết code. Chỉ nhận dữ liệu và đẩy thẳng (hoặc chuyển đổi định dạng CSV sang Parquet, nén Gzip) vào kho lưu trữ (S3, Redshift, OpenSearch).
- **Amazon Managed Service for Apache Flink:** Dịch vụ để chạy các kịch bản phân tích, tính toán dữ liệu trực tiếp trong lúc nó đang chảy (Streaming Analytics) bằng SQL, Java.

## 4. Các dịch vụ tiện ích khác (Other Services)
- **AWS SES (Simple Email Service):** Dịch vụ gửi (và nhận) email số lượng lớn chuyên nghiệp qua SMTP hoặc API.
- **Amazon OpenSearch (Trước là ElasticSearch):** Máy chủ tìm kiếm cực nhanh. Trái với DynamoDB (chỉ tìm theo khóa chính), OpenSearch cho phép tìm kiếm theo bất kỳ từ khóa nào, tìm gần giống, tìm theo bộ lọc phức tạp.
- **Amazon Athena:** Dịch vụ truy vấn (Query) không máy chủ. Cho phép bạn gõ câu lệnh SQL chuẩn để tìm kiếm dữ liệu thẳng trên các file CSV/JSON/Parquet đang cất trong S3 mà không cần chép vào cơ sở dữ liệu. Tính tiền theo số GB quét. (Lưu dữ liệu dạng cột Parquet giúp tiết kiệm 90% chi phí Athena).
- **Amazon MSK (Managed Streaming for Kafka):** Tương tự Kinesis nhưng dùng mã nguồn mở Apache Kafka, dành cho các công ty muốn chuyển hệ thống từ On-premise lên đám mây mà không đổi code.
- **AWS AppConfig:** Dịch vụ giúp bạn thay đổi biến môi trường, bật/tắt tính năng (Feature flags) cho hệ thống ứng dụng ngay lúc đang chạy mà không cần khởi động lại máy chủ hay tải lại mã nguồn. Hỗ trợ validate tính hợp lệ trước khi áp dụng.

## 💡 Trích xuất trọng tâm luyện thi DVA-C02
- **Theo dõi mọi lượt chạy của Lambda:** Nếu đề bài yêu cầu ghi lại TẤT CẢ sự kiện (cả thành công lẫn thất bại) vào SQS, giải pháp tốt nhất là dùng thẳng **AWS SDK (SendMessage API)** bên trong mã nguồn Lambda. DLQ chỉ lưu các lần chạy thất bại.
- **Kiến trúc Pub/Sub (Event-Driven Fan-Out):** Nếu muốn một event kích hoạt nhiều xử lý bất đồng bộ, dùng **SNS Topic -> SQS Queues -> Lambda** (hoặc ứng dụng trên ECS lấy từ SQS). Đừng bao giờ nối Lambda gọi nhau trực tiếp (Chaining).
- **Biến đổi dữ liệu Kinesis Firehose:** Để che giấu/lọc PII, hãy bật tính năng Data Transformation của Firehose để gọi một hàm **Lambda** biến đổi dữ liệu on-the-fly trước khi ghi xuống S3.
- **Giới hạn số lượng tin nhắn SQS:** Không có giới hạn tối đa số lượng tin nhắn được lưu trữ trong hàng đợi SQS (**No limit**).
- **Xử lý tin nhắn lỗi liên tục trong SQS:** Cấu hình **Dead-Letter Queue (DLQ)** thông qua Redrive Policy để tự động cách ly các tin nhắn xử lý lỗi nhiều lần ra khỏi hàng đợi chính.
- **EventBridge Pipes:** Để lọc event không cần thiết dùng tính năng *Filter*. Để query thêm dữ liệu từ DB dùng *Lambda Enrichment*. Để đổi format payload dùng *Input Transformer*.
