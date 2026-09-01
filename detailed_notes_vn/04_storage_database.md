# Phần 4: Lưu trữ (Storage) & Cơ sở dữ liệu (Database)

## 1. Amazon S3 (Simple Storage Service)
S3 là dịch vụ lưu trữ đối tượng, có tính bền bỉ siêu cao (11 số 9 - 99.999999999%).
- **Buckets:** Là các thư mục chính, tên phải là duy nhất trên toàn cầu (không trùng với ai trên toàn AWS). Không được viết hoa, không dùng IP.
- **Objects:** Các tập tin. Được định danh bằng "Key" (chính là đường dẫn file: `folder1/file.txt`). Kích thước tối đa 5TB/file. 
- Không có khái niệm thư mục (directory) thực sự ở backend, chỉ là những cái tên rất dài chứa dấu `/`.

### Bảo mật và Phân quyền S3
- **IAM Policies:** Quy định quyền cho User thao tác với S3.
- **Bucket Policies:** Gắn trực tiếp lên Bucket (viết bằng JSON), dùng để thiết lập quyền truy cập công cộng (Public Access) hoặc chia sẻ Liên tài khoản (Cross-account).
- **MFA Delete:** Yêu cầu mã MFA để xóa vĩnh viễn một object hoặc tắt Versioning (Chỉ tài khoản Root mới bật được).
- **Mã hóa (Encryption):**
  - *SSE-S3:* Mặc định của S3, mã hoá AES-256. AWS tự lo chìa khóa.
  - *SSE-KMS:* Quản lý khóa bởi KMS. Có thể kiểm toán qua CloudTrail, cấp quyền qua KMS Policy. Giới hạn tốc độ theo KMS quota.
  - *SSE-C:* Người dùng tự cấp và quản lý khóa mã hóa. Phải dùng HTTPS.
  - *Client-Side Encryption:* Khách hàng tự mã hóa file trước khi nạp lên S3.
  - *Bắt buộc HTTPS:* Có thể cài luật trong Bucket Policy từ chối mọi yêu cầu `aws:SecureTransport: false`.
- **Pre-signed URLs (URL Ký trước):** Tạo 1 link tạm thời cho phép ai đó có quyền tải xuống (GET) hoặc tải lên (PUT) file (thường có hiệu lực vài phút đến tối đa 7 ngày với CLI). Quyền của link phụ thuộc vào quyền của người tạo link.
- **CORS (Cross-Origin Resource Sharing):** Phải bật CORS trên S3 nếu muốn các website từ tên miền khác có thể dùng JS gọi thẳng tới S3.
- **S3 Access Points:** Tạo ra nhiều điểm truy cập độc lập với policy riêng rẽ trên cùng 1 bucket để dễ quản trị bảo mật ở quy mô lớn. 
- **S3 Object Lambda:** Chạy Lambda ngay khi người dùng gọi file từ S3 để chỉnh sửa file "ngay trong lúc tải về" (Ví dụ: Đóng dấu ảnh tự động, che dữ liệu nhạy cảm).

### Quản lý vòng đời, Tính năng và Hiệu năng
- **Versioning (Quản lý phiên bản):** Khi bật, ghi đè file sẽ tạo ra bản mới (Version 2, 3...) thay vì xóa hẳn. Tránh vô tình xóa (hiện Delete marker).
- **Replication:** Tự động copy file mới. *CRR (Cross-Region)* - chép khác vùng để chống thảm hoạ; *SRR (Same-Region)* - chép cùng vùng để tổng hợp log. Phải bật Versioning trên cả 2 bucket. S3 Batch Replication có thể sao chép cả file cũ.
- **Lớp lưu trữ (Storage Classes):**
  - *Standard:* Nhanh, thường xuyên sử dụng.
  - *Standard-IA (Infrequent Access):* Rẻ hơn nhưng mất phí mỗi lần lấy ra. Dùng cho dữ liệu ít truy cập.
  - *One-Zone IA:* Cực rẻ, chỉ lưu 1 AZ (có thể mất dữ liệu nếu AZ bị hủy).
  - *Glacier Instant / Flexible / Deep Archive:* Lưu trữ siêu rẻ, dành cho đóng băng dữ liệu dài hạn, thời gian lấy lại từ vài ms (Instant) tới tận 12-48h (Deep Archive).
  - *Intelligent Tiering:* AI của AWS tự di chuyển file giữa các lớp dựa trên thói quen sử dụng, tối ưu chi phí tự động.
- **Lifecycle Rules (Luật vòng đời):** Tự động chuyển lớp file (ví dụ sau 30 ngày chuyển sang IA) hoặc xóa hẳn file cũ/bản version cũ.
- **Hiệu năng:** 
  - Đạt 3500 PUT / 5500 GET mỗi giây cho mỗi tiền tố (prefix/thư mục).
  - Tải lên file > 100MB nên dùng *Multi-part upload* (bắt buộc với file > 5GB).
  - *Transfer Acceleration:* Tải file thông qua mạng lưới CDN Edge Location của AWS để tăng tốc độ cho người dùng ở xa.
  - *Byte-Range Fetches:* Chỉ tải một đoạn nhỏ của file (ví dụ tải tiêu đề file) hoặc dùng tải song song nhiều phần.
- **Event Notifications:** Gọi SQS, SNS, Lambda khi có file mới. Gửi sự kiện cho mọi lần tải lên thành công.
- **Metadata & Tags:** `x-amz-meta-*` dùng gán thông tin phụ. Tag dùng phân quyền và thống kê chi phí. Không thể tìm kiếm file bằng Tag, phải dùng DB ngoài.

## 2. Amazon DynamoDB
DynamoDB là cơ sở dữ liệu NoSQL Serverless, có khả năng mở rộng không giới hạn với độ trễ một chữ số mili-giây.

### Khoá chính và Chế độ thanh toán
- Dữ liệu lưu dưới dạng Table > Item (Dòng) > Attribute (Cột). Kích thước tối đa 1 dòng là 400KB.
- **Primary Key:** 
  - Chỉ có Partition Key (Hash key).
  - Hoặc Partition Key + Sort Key (Range key) để nhóm dữ liệu.
- **Provisioned (Cấp phát trước):** Mua trước số RCU và WCU mong muốn. 
  - *WCU (Ghi):* 1 WCU = Ghi 1 Item lên tới 1KB mỗi giây. (Kích thước làm tròn lên).
  - *RCU (Đọc):* 1 RCU = Đọc Strongly Consistent 1 Item lên tới 4KB mỗi giây (Hoặc 2 Eventually Consistent).
  - **Mẹo thi:** Đi thi luôn có bài tập tính WCU/RCU. Công thức: (Kích thước item / 1KB hoặc 4KB làm tròn lên) * Số request mỗi giây.
- **On-Demand:** Không cần đoán dung lượng, tự động scale. Trả tiền theo từng lượt đọc ghi (Rất đắt - gấp 2.5 lần Provisioned nhưng an toàn cho tải thay đổi đột ngột).
- **Throttling (Nghẽn):** Khi vượt giới hạn RCU/WCU (Đặc biệt do "Hot Keys" - một dữ liệu bị đọc quá nhiều). Cần Exponential Backoff hoặc phân tán Partition key (Write sharding).

### Các thao tác (Operations)
- **Ghi:** `PutItem` (thêm mới hoặc ghi đè toàn bộ), `UpdateItem` (Cập nhật 1 vài cột, hữu ích làm bộ đếm Atomic).
- **Đọc:** 
  - `GetItem` (Lấy bằng đúng Primary Key).
  - `Query` (Truy vấn theo Partition Key cụ thể và dùng phép điều kiện (>, <, begins_with) trên Sort Key). 
  - `Scan` (Quét toàn bộ bảng - Cực kỳ tốn kém RCU, rất chậm). Sử dụng Parallel Scan để tăng tốc scan.
  - **Mẹo thi Pagination (Phân trang):** Cả Query và Scan đều giới hạn trả về tối đa **1MB dữ liệu**. AWS sẽ trả về tham số `LastEvaluatedKey`. Ứng dụng phải dùng tham số này để gọi API lần 2 lấy trang tiếp theo.
- **Xóa:** `DeleteItem` xóa từng dòng, `DeleteTable` xóa toàn bộ bảng không tốn WCU.
- **Batch:** `BatchWriteItem` (lên tới 25 dòng), `BatchGetItem` (100 dòng) làm song song để giảm thời gian phản hồi. Nếu thất bại 1 vài dòng, danh sách sẽ nằm trong `UnprocessedItems` để thử lại.
- **Conditional Writes (Ghi có điều kiện):** Chỉ ghi khi điều kiện thỏa (VD: Tên cột không tồn tại, phiên bản chưa đổi - *Optimistic Locking*). Khắc phục ghi đè đồng thời.

### Các tính năng nâng cao
- **Secondary Indexes (Chỉ mục phụ):**
  - *LSI (Local Secondary Index):* Giữ nguyên Partition Key, đổi Sort Key. Chỉ tạo được khi khởi tạo bảng. Dùng chung RCU/WCU với bảng chính.
  - *GSI (Global Secondary Index):* Đổi cả Partition Key. Tạo bất cứ lúc nào. Có pool RCU/WCU riêng (nếu GSI bị nghẽn, bảng chính cũng sẽ bị nghẽn).
- **DAX (DynamoDB Accelerator):** Cụm In-memory cache độc quyền. Siêu nhanh (microsecond), không cần sửa code app, giải quyết dứt điểm vấn đề "Hot Key" (quá tải đọc). 
- **DynamoDB Streams:** Ghi lại mọi sự thay đổi (Thêm/sửa/xóa) trong 24 giờ. Rất hữu ích khi đẩy vào Lambda để thực hiện tác vụ thời gian thực (ví dụ: Báo email ngay khi có dữ liệu mới).
- **TTL (Time to Live):** Tự động xóa dữ liệu quá hạn, không tốn chi phí WCU. Dữ liệu xóa sẽ xuất hiện trong Dynamo Streams.
- **Transactions (Giao dịch):** Đảm bảo ACID, cập nhật đồng thời nhiều bảng (Tất cả thành công hoặc thất bại). Tốn gấp đôi RCU và WCU (do có prepare & commit).

## 3. Cơ sở dữ liệu Quan hệ & Cache (RDS, Aurora, ElastiCache)
- **RDS:** Cơ sở dữ liệu quan hệ quản lý sẵn (MySQL, PostgreSQL, MariaDB...). Hỗ trợ tự động dự phòng, bảo trì, vá lỗi (Khác xa so với việc tự cài DB trên EC2).
  - *Read Replicas:* Sinh bản sao (chỉ đọc) để giảm tải cho DB chính khi ứng dụng bị đọc nhiều. Tối đa 15 bản.
  - *Multi-AZ:* Chạy song song ổ cứng sang vùng khác làm bản dự phòng chống thảm họa.
- **Aurora:** Kiến trúc CSDL do AWS tối ưu cho môi trường đám mây. Tự động lưu 6 bản copy qua 3 AZ. Khôi phục thảm họa siêu tốc (<30s). Có Reader Endpoint tự động cân bằng tải đọc.
- **RDS Proxy:** Chứa sẵn kết nối Database để dùng chung (Connection Pooling) -> Không thể thiếu khi tích hợp với AWS Lambda để tránh sập DB vì quá tải kết nối.
- **ElastiCache:** Dịch vụ bộ nhớ RAM đám mây (Redis hoặc Memcached). Làm giảm tải rất nhiều lượng truy vấn trực tiếp vào DB, đồng thời làm Session store giúp web backend chuyển sang kiến trúc Stateless (phi trạng thái).
  - *Redis:* Hỗ trợ sao chép đa vùng (Multi-AZ), cấu trúc dữ liệu phức tạp.
  - *Memcached:* Hỗ trợ Sharding (phân mảnh đa node), đa luồng, dùng đơn giản.
  - **Chiến lược Caching:** *Lazy Loading* (Chỉ nạp vào cache khi đọc thiếu - Cache Miss), *Write-through* (Ghi thẳng vào cache khi có ghi dữ liệu mới). Cần sử dụng TTL để tự động xóa dữ liệu rác cũ.

## 💡 Trích xuất trọng tâm luyện thi DVA-C02
- **Mã hóa S3 Server-Side (SSE):** Dùng header `x-amz-server-side-encryption`. Giá trị `AES256` là cho SSE-S3, giá trị `aws:kms` là cho SSE-KMS. Không được nhầm lẫn với SSE-C.
- **Bảo mật file nhạy cảm/Logs trên S3:** Kết hợp S3 + SSE-KMS + KMS Key Policy. Đây là giải pháp duy nhất cho phép bạn kiểm soát chính xác từng user/role IAM nào có quyền dùng khóa để giải mã dữ liệu.
- **Nghẽn S3 (503 Slow Down):** Khi vượt giới hạn cứng (5500 GET / 3500 PUT mỗi giây), S3 sẽ báo lỗi 503. Giải pháp thi chuẩn xác nhất là dùng thuật toán **Retry with Exponential Backoff** trong code ứng dụng. Không có cách nào yêu cầu AWS tăng quota này.
- **Hiệu năng EBS gp2:** Tỉ lệ là 3 IOPS trên mỗi GiB. Giới hạn IOPS tối đa của ổ là 16,000 IOPS. Để đạt max hiệu năng, kích thước ổ phải là: `16000 / 3 = 5,333 GiB` (~5.3 TiB).
