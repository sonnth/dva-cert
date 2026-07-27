# AWS DVA-C02 Exam Tips & Tricks - Domain 2: Security

## 📌 Domain 2: Security (Serverless Security)

### 🧩 Topic: AWS Lambda Code Integrity & Security

#### 🔑 Keywords Nhận Diện
* `unauthorized changes to the code` (thay đổi code trái phép).
* `ensure only trusted code is deployed` (đảm bảo chỉ deploy code đáng tin cậy).
* `digitally sign the Lambda package` (ký số gói deploy).

#### 💡 Tip & Trick Chọn Đáp Án
* **AWS Signer + Lambda Code Signing:** Khi đề bài đề cập đến việc chống chỉnh sửa code Lambda trái phép hoặc chỉ cho phép deploy code đã qua kiểm duyệt/tin tưởng -> Hãy tìm đáp án chứa bộ đôi **AWS Signer** và **Code Signing configuration** trên Lambda.
* **Phân biệt với AWS KMS:** Mặc dù AWS KMS dùng để mã hóa dữ liệu (Encryption), nhưng đối với bài toán xác thực tính toàn vẹn và nguồn gốc của gói code deploy (Code Integrity), bạn phải dùng **AWS Signer** để ký số.
* **Phân biệt với AWS CodeDeploy:** CodeDeploy quản lý luồng triển khai (Linear, Canary, AllAtOnce) chứ không trực tiếp thực hiện ký số hay xác thực tính toàn vẹn mã nguồn trước khi nạp vào Lambda.

### 🧩 Topic: Amazon S3 Server-Side Encryption (SSE-S3 vs SSE-KMS vs SSE-C)

#### 🔑 Keywords Nhận Diện
* `encrypt these objects at rest` (mã hóa dữ liệu S3 ở trạng thái tĩnh).
* `server-side encryption with Amazon S3 managed keys (SSE-S3)` (sử dụng SSE-S3).
* `PutObject API` / `HTTP API` / `HTTP headers`.

#### 💡 Tip & Trick Chọn Đáp Án
* **Sử dụng Header `x-amz-server-side-encryption`:**
  * Để yêu cầu S3 mã hóa đối tượng ngay khi ghi dữ liệu (qua `PutObject` API hoặc HTTP request), ứng dụng phải truyền tiêu đề HTTP (header): `x-amz-server-side-encryption`.
  * **SSE-S3 (S3 Managed Keys):** Đặt giá trị header là `AES256`. S3 sẽ tự động quản lý và tạo khóa mã hóa.
  * **SSE-KMS (KMS Managed Keys):** Đặt giá trị header là `aws:kms` (và tùy chọn thêm `x-amz-server-side-encryption-aws-kms-key-id` để chỉ định KMS key).
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *Tự cung cấp khóa trong header:* Đây là thuộc tính của **SSE-C** (Server-Side Encryption with Customer-Provided Keys), yêu cầu truyền khóa riêng của khách hàng qua header `x-amz-server-side-encryption-customer-key`, không phải SSE-S3.
  * *Tạo và gán KMS key:* Hành động tạo KMS key chỉ dùng cho **SSE-KMS**, không liên quan đến SSE-S3 (vốn sử dụng khóa mặc định do hệ thống S3 tự quản lý).
  * *Áp dụng TLS:* TLS dùng để mã hóa dữ liệu trên đường truyền (encryption in transit), không có tác dụng mã hóa tĩnh trên đĩa của S3 (encryption at rest).

### 🧩 Topic: Amazon S3 Data Security & Encryption Key Access Control

#### 🔑 Keywords Nhận Diện
* `store log files and other unstructured data` (lưu trữ file log và dữ liệu phi cấu trúc tối ưu chi phí).
* `encrypt data in transit and at rest` (mã hóa đường truyền và mã hóa tĩnh).
* `control the encryption keys` + `control who has access to the keys` (kiểm soát khóa mã hóa và phân quyền chi tiết ai được dùng khóa).

#### 💡 Tip & Trick Chọn Đáp Án
* **Sự kết hợp giữa Amazon S3 và SSE-KMS:**
  * **Lưu trữ tối ưu:** S3 là lựa chọn lưu trữ rẻ và phù hợp nhất cho file logs/unstructured data (DynamoDB chỉ phù hợp cho dữ liệu cấu hình/Structured NoSQL dung lượng nhỏ).
  * **Kiểm soát và phân quyền khóa:** Chỉ có **SSE-KMS** (mã hóa sử dụng AWS KMS keys) mới cho phép khách hàng tự kiểm soát khóa và cấu hình **KMS Key Policy** để định nghĩa chi tiết những ai (IAM Principals) có quyền sử dụng khóa để mã hóa/giải mã và hành động cụ thể được phép làm.
  * **Enforce mã hóa trên Bucket:** Sử dụng **Bucket Policy** kết hợp với từ chối (`Deny`) các request upload (`PutObject`) nếu tiêu đề HTTP không chứa cấu hình mã hóa KMS tương ứng (ví dụ: `x-amz-server-side-encryption` là `aws:kms`).
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *Mã hóa SSE-S3:* Sử dụng khóa do S3 tự quản lý hoàn toàn, khách hàng không thể kiểm soát khóa hay phân quyền chi tiết cho người dùng trên khóa.
  * *Lưu trữ log trong DynamoDB:* Không tối ưu chi phí và không phù hợp để lưu trữ file log lớn/phi cấu trúc.
  * *AWS Secrets Manager / AWS Certificate Manager (ACM) để mã hóa DB:* Secrets Manager chỉ dùng quản lý secrets (mật khẩu/credentials), ACM chỉ dùng quản lý SSL/TLS certificates, cả hai đều không dùng làm Key Management Service (KMS) để tạo và quản lý khóa mã hóa dữ liệu tĩnh trực tiếp trong DB.

### 🧩 Topic: Secure ECS Database Credentials (Secrets Manager & Task Execution Role)

#### 🔑 Keywords Nhận Diện
* `improve security for database credentials in an Amazon ECS environment` (tăng tính bảo mật cho thông tin đăng nhập database trong môi trường ECS).
* `stores credentials directly in ECS task definition environment variables` (hiện tại lưu plaintext trong biến môi trường của task definition).
* `LEAST development effort` (tốn ít công sức phát triển/sửa code nhất).

#### 💡 Tip & Trick Chọn Đáp Án
* **Lưu Secret trong Secrets Manager + inject tự động qua Task Execution Role:**
  * Để bảo mật mà tốn ít công sức sửa code ứng dụng nhất: Lưu credentials trong **AWS Secrets Manager** (hoặc SSM Parameter Store).
  * Trong **ECS Task Definition**, khai báo thuộc tính `secrets` và liên kết với ARN của secret. Khi ECS khởi chạy container, ECS agent sẽ tự động fetch giá trị secret này từ Secrets Manager và inject trực tiếp thành biến môi trường có tên tương ứng vào trong container. Do đó, code ứng dụng đọc từ biến môi trường hoàn toàn không phải thay đổi.
  * Vì hành động fetch secret này do ECS container agent thực hiện lúc khởi động chứ không phải do code ứng dụng chạy bên trong container, bạn **bắt buộc** phải cấp quyền truy cập secret (`secretsmanager:GetSecretValue`) cho **Task Execution IAM Role** (không phải Task IAM Role).
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *Cấp quyền cho Task IAM Role để fetch secret:* Task IAM Role cấp quyền cho code ứng dụng chạy bên trong container (ví dụ gọi AWS SDK). Việc sử dụng role này để kéo secret từ định nghĩa task definition là sai kiến trúc và gây lỗi access denied khi container agent kéo secret lúc khởi động.
  * *Sử dụng AWS KMS key mã hóa thủ công:* Mã hóa thủ công credentials rồi đưa vào biến môi trường yêu cầu bạn phải sửa code ứng dụng để load khóa KMS và giải mã thủ công, tốn nhiều công sức phát triển (vi phạm yêu cầu LEAST development effort).

### 🧩 Topic: AWS KMS Envelope Encryption & GenerateDataKey API

#### 🔑 Keywords Nhận Diện
* `PDF file could be more than 1 MB` / `large file` (dữ liệu cần mã hóa lớn hơn 4 KB).
* `GenerateDataKey API` (sử dụng API tạo khóa dữ liệu).
* `symmetric customer managed key` (sử dụng KMS key đối xứng để quản lý).

#### 💡 Tip & Trick Chọn Đáp Án
* **Quy trình Mã hóa phong bì (Envelope Encryption) chuẩn:**
  1. Gọi API **`GenerateDataKey`** của AWS KMS bằng cách truyền vào KMS CMK.
  2. API sẽ trả về 2 giá trị khóa: **Plaintext data key** (khóa dữ liệu dạng rõ) và **Encrypted data key** (khóa dữ liệu đã được mã hóa bởi KMS CMK).
  3. Sử dụng **Plaintext data key** để mã hóa file cục bộ bằng thuật toán mã hóa đối xứng (như AES-256) trên ứng dụng.
  4. **Lưu trữ an toàn:** Ghi **Encrypted data key** vào ổ đĩa/storage (cùng với file đã mã hóa) để dùng giải mã sau này. **Xóa Plaintext data key** khỏi bộ nhớ RAM ngay lập tức và tuyệt đối không bao giờ ghi Plaintext key xuống đĩa.
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *Sử dụng trực tiếp API KMS Encrypt để mã hóa file:* API `Encrypt`/`Decrypt` trực tiếp của KMS chỉ hỗ trợ dữ liệu tối đa **4 KB** (4096 bytes). Với file lớn hơn 4 KB (ví dụ > 1 MB), bắt buộc phải dùng Envelope Encryption.
  * *Ghi Plaintext data key xuống đĩa:* Vi phạm nghiêm trọng nguyên tắc bảo mật.
  * *Dùng Encrypted data key để mã hóa file:* Khóa đã bị mã hóa không thể dùng trực tiếp để thực hiện thuật toán mã hóa dữ liệu.

### 🧩 Topic: AWS Encryption SDK & Data Key Management

#### 🔑 Keywords Nhận Diện
* `AWS Encryption SDK` (thư viện mã hóa mã nguồn mở của AWS).
* `keep track of the data encryption keys used to encrypt data` (làm thế nào để theo dõi/quản lý các khóa mã hóa dữ liệu đã dùng).

#### 💡 Tip & Trick Chọn Đáp Án
* **SDK tự động lưu Encrypted Data Key vào Ciphertext:**
  * Khi sử dụng **AWS Encryption SDK** để thực hiện Envelope Encryption, nhà phát triển **không cần tự quản lý hay lưu trữ cơ sở dữ liệu về các khóa đã dùng**.
  * SDK sẽ tự động mã hóa khóa dữ liệu (data key) và lưu trữ trực tiếp khóa đã mã hóa đó dưới dạng **siêu dữ liệu (metadata) nằm trong tiêu đề (header) của chuỗi ciphertext (bản mã)** trả về. Khi cần giải mã, chỉ cần truyền chuỗi ciphertext này vào SDK, nó sẽ tự động đọc header để lấy khóa bị mã hóa và gửi lên KMS để giải thô.
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *Nhà phát triển phải tự quản lý thủ công:* Sai, SDK sinh ra để tự động hóa toàn bộ việc đóng gói siêu dữ liệu khóa này.
  * *Lưu tự động vào Amazon S3:* SDK là thư viện client-side cục bộ, không tự kết nối hay lưu khóa vào S3.
  * *Lưu trong EC2 Userdata:* Userdata chỉ dùng để chạy script khi EC2 khởi tạo lần đầu, không liên quan đến việc lưu trữ khóa động của từng đối tượng dữ liệu.

### 🧩 Topic: Database Credentials Security & Automatic Rotation (Secrets Manager vs Parameter Store)

#### 🔑 Keywords Nhận Diện
* `open connections to an Amazon RDS` (kết nối đến cơ sở dữ liệu RDS).
* `automatically rotate the credentials` (tự động xoay vòng/cập nhật mật khẩu định kỳ).
* `does not want to store the credentials in the code` (không lưu plaintext mật khẩu trong source code).
* `MOST secure way` (phương án bảo mật tốt nhất).

#### 💡 Tip & Trick Chọn Đáp Án
* **Sự kết hợp giữa AWS Secrets Manager & Automatic Rotation:**
  * Để lưu trữ thông tin đăng nhập DB bảo mật và tự động xoay vòng mật khẩu, dịch vụ được lựa chọn luôn là **AWS Secrets Manager**.
  * Secrets Manager tích hợp sẵn tính năng tự động xoay vòng khóa (**Automatic Rotation**) kết hợp với AWS Lambda để định kỳ đổi mật khẩu cả trên Secrets Manager lẫn trên database đích (như RDS SQL Server) mà không cần sự can thiệp thủ công.
  * Ứng dụng chạy trên EC2 sẽ lấy thông tin thông qua gọi API của Secrets Manager lúc runtime bằng AWS SDK (sử dụng IAM role của EC2 instance để xác thực), tránh tuyệt đối việc lưu cứng mật khẩu trong code.
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *AWS Systems Manager Parameter Store:* Parameter Store chỉ là kho lưu trữ tham số tĩnh (Key-Value), nó **không hỗ trợ tính năng tự động xoay vòng mật khẩu tích hợp sẵn** cho RDS giống như Secrets Manager.
  * *Lưu mật khẩu trên Amazon S3:* Không an toàn, khó phân quyền chi tiết và không hỗ trợ tính năng tự động xoay vòng mật khẩu.
  * *Lưu trong file cấu hình của Git repository:* Vi phạm nghiêm trọng nguyên tắc bảo mật thông tin nhạy cảm.
