# AWS DVA-C02 Exam Tips & Tricks - Domain 3: Development with AWS Services

## 📌 Domain 3: Development with AWS Services (Serverless & Integration)

### 🧩 Topic: AWS Lambda Invocation Tracking & Messaging

#### 🔑 Keywords Nhận Diện
* `each Lambda function invocation` / `record of each invocation`: Cần ghi nhận lại mọi lượt thực thi của Lambda (cả thành công và thất bại).
* `add a record to an Amazon SQS queue`: Đưa thông tin vào hàng đợi SQS.

#### 💡 Tip & Trick Chọn Đáp Án
* **Bản chất nghiệp vụ (Application Logic):** Khi đề bài yêu cầu ghi nhận thông tin cụ thể hoặc custom chi tiết của *mỗi lượt chạy* vào một dịch vụ khác (như SQS), cách trực tiếp và linh hoạt nhất là sử dụng **AWS SDK** (`SendMessage` API) ngay trong mã nguồn Lambda.
* **Loại trừ DLQ (Dead-Letter Queue):** DLQ **chỉ** hoạt động khi invocation bị **thất bại** (failed) sau khi đã re-try. Nếu đề bài yêu cầu "each invocation" hoặc "successful invocation", loại ngay đáp án có DLQ.
* **Cảnh giác với Destinations:** Lambda Destinations (`OnSuccess` / `OnFailure`) rất tốt cho luồng async event-driven, nhưng nếu đề bài đi kèm các cấu hình rườm rà không cần thiết (như tạo CloudWatch Alarm chỉ để check delivery fail của destination trong khi mục đích chính là record invocation) -> Ưu tiên giải pháp xử lý bằng code SDK ở cuối function để đảm bảo tính tường minh.

### 🧩 Topic: EventBridge Pipes (Filter vs Enrichment vs Input Transformer)

#### 🔑 Keywords Nhận Diện
* `Filter`: Quyết định event nào được đi tiếp (ví dụ: chỉ publish event khi `status = ready`).
* `Lambda Enrichment`: Bổ sung dữ liệu trước khi gửi (query thêm DB/API, thêm metadata).
* `Input Transformer`: Chỉ đổi cấu trúc/format payload JSON, không dùng để filter dữ liệu.
* `EventBridge Rules`: Thực hiện lọc (filter) sau khi event đã đi vào Event Bus (khác với lọc ngay tại Pipe).

#### 💡 Tip & Trick Chọn Đáp Án
* **Chỉ publish event khi thỏa mãn điều kiện (ví dụ: status = ready):** Chọn **Filter** trong EventBridge Pipe.
* **Cần lấy thêm dữ liệu từ nguồn ngoài (DB/API) trước khi gửi:** Chọn **Lambda Enrichment**.
* **Chỉ cần thay đổi cấu trúc payload/JSON:** Chọn **Input Transformer**.

### 🧩 Topic: API Gateway + Lambda Integration (500 Internal Server Error & Permissions)

#### 🔑 Keywords Nhận Diện
* `API Gateway REST API` + `Lambda function integration`.
* `multiple environment stages and stage variables` (sử dụng Stage Variables để gọi Lambda động).
* `500 Internal Server Error` khi test gọi API.
* `grant permissions to call the backend integration`.

#### 💡 Tip & Trick Chọn Đáp Án
* **Lỗi quyền Invoke (Resource-based policy):** Khi API Gateway gọi Lambda qua tích hợp động sử dụng Stage Variables (ví dụ: `arn:aws:lambda:...:${stageVariables.funcName}`), API Gateway không thể tự động cấp quyền Invoke. Bạn phải **cấu hình một resource-based policy trên Lambda function** để cấp quyền `lambda:InvokeFunction` cho API Gateway service.
* Nếu thiếu quyền này, API Gateway sẽ báo **500 Internal Server Error**.
* **Loại trừ các tùy chọn không liên quan:**
  * *Lambda Layer:* Dùng để chia sẻ mã nguồn/thư viện dùng chung giữa các hàm Lambda, không liên quan đến quyền gọi API của API Gateway.
  * *Reserved Concurrency:* Dùng để giới hạn số lượng thực thi đồng thời để tránh làm quá tải hệ thống backend, không giải quyết được lỗi phân quyền.
  * *Availability Zone:* API Gateway và Lambda là các dịch vụ Managed Serverless cấp Region, không cần cấu hình hay ràng buộc theo AZ thủ công.

### 🧩 Topic: Cost-Effective Serverless REST API with Caching

#### 🔑 Keywords Nhận Diện
* `REST endpoint that clients can call`: Cần cung cấp API endpoint RESTful.
* `limit the number of requests to the backend service`: Cần sử dụng cơ chế lưu nháp (caching) để giảm tải cho backend.
* `small amount of traffic only during testing`: Lưu lượng truy cập rất ít, không thường xuyên (chỉ trong giai đoạn thử nghiệm).
* `MOST cost-effectively`: Tối ưu chi phí nhất có thể.

#### 💡 Tip & Trick Chọn Đáp Án
* **Giải pháp Serverless tối ưu nhất (API Gateway + Lambda):**
  * Với lượng request cực thấp và không liên tục, **AWS Lambda** (pay-per-invocation) là lựa chọn rẻ nhất vì bạn không phải trả phí cho tài nguyên rỗi (idle resource).
  * **Amazon API Gateway** cung cấp tính năng **Caching** ở cấp độ Stage để lưu nháp kết quả phản hồi từ backend (đáp ứng yêu cầu giảm tải requests đến backend).
  * Cặp đôi **API Gateway + AWS Lambda** được định nghĩa bằng **AWS SAM** luôn là câu trả lời chuẩn nhất cho các bài toán POC chi phí thấp.
* **Loại trừ các tùy chọn có chi phí cao:**
  * *Amazon EKS (Kubernetes)* hoặc *Amazon ECS (Containers):* Phải trả phí duy trì các container cluster/tasks chạy liên tục, gây lãng phí lớn khi hệ thống chỉ nhận lưu lượng truy cập rất ít trong giai đoạn thử nghiệm.
  * *AWS Elastic Beanstalk:* Tự động tạo các máy chủ ảo EC2 chạy 24/7, phát sinh chi phí duy trì cố định theo giờ, không tối ưu cho mô hình pay-per-request như Serverless.

### 🧩 Topic: Amazon S3 Throttling Troubleshooting (503 Slow Down)

#### 🔑 Keywords Nhận Diện
* `GET/HEAD requests to the S3 bucket per second` (vài ngàn requests/giây trong thời gian ngắn).
* `503 Slow Down` (lỗi phản hồi từ S3 do vượt quá giới hạn request rate).
* `resolve the error`.

#### 💡 Tip & Trick Chọn Đáp Án
* **Sử dụng Retry với Exponential Backoff:**
  * Giới hạn mặc định của S3 là **5,500 GET/HEAD requests/giây** và **3,500 PUT/POST/DELETE/COPY requests/giây** trên mỗi partitioned prefix.
  * Khi lưu lượng tăng đột biến vượt quá giới hạn này, S3 sẽ trả về lỗi **HTTP 503 Slow Down**.
  * Giải pháp nhanh chóng, hiệu quả và chuẩn AWS nhất là **thêm cơ chế Retry (thử lại) kết hợp với Exponential Backoff** (giãn cách thời gian thử lại lũy thừa) vào code ứng dụng.
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *Tăng quota / Request increase:* Giới hạn request rate của S3 là giới hạn cứng (hard limit) ở mức kiến trúc phần cứng/phân vùng, không thể yêu cầu tăng quota qua support ticket.
  * *Thay đổi Availability Zone của EC2:* AZ của EC2 không ảnh hưởng đến khả năng xử lý request rate của S3 bucket ở cùng Region.
  * *Chuyển đổi sang Directory Bucket (S3 Express One Zone):* Đây là thay đổi kiến trúc lớn, phức tạp và cực kỳ đắt đỏ về chi phí lưu trữ, không phù hợp cho trường hợp chỉ bị nghẽn traffic đột biến trong thời gian ngắn.

### 🧩 Topic: AWS Lambda Startup Optimization (Cold Starts & Provisioned Concurrency)

#### 🔑 Keywords Nhận Diện
* `optimize the startup time` / `optimize the initialization of the function` (tối ưu hóa thời gian khởi tạo hoặc khởi động của Lambda).
* `infrequently invoked by multiple clients at the same time` (gọi không thường xuyên nhưng khi gọi thì nhiều client gọi cùng lúc - nguyên nhân chính gây ra Cold Starts trên nhiều container đồng thời).

#### 💡 Tip & Trick Chọn Đáp Án
* **Kích hoạt Provisioned Concurrency:**
  * Để loại bỏ hoàn toàn hiện tượng Cold Start (trễ khởi tạo môi trường chạy), hãy sử dụng tính năng **Provisioned Concurrency**. Tính năng này chuẩn bị sẵn (pre-warm) một số lượng môi trường thực thi đã được khởi tạo toàn bộ, giúp Lambda phản hồi ngay lập tức khi có request đến.
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *API Gateway Caching:* Mặc dù giúp giảm tải và trả phản hồi nhanh cho client bằng dữ liệu đệm, nhưng nó không giải quyết được thời gian khởi tạo thực tế của chính hàm Lambda (khi cache miss hoặc cho request động).
  * *Lambda Proxy Integration:* Chỉ là phương thức truyền dữ liệu thô (raw format) giữa API Gateway và Lambda, không ảnh hưởng đến thời gian khởi chạy/khởi tạo container của Lambda.
  * *AWS Global Accelerator:* Tối ưu hóa định tuyến mạng toàn cầu để giảm độ trễ đường truyền internet (network latency), không liên quan đến thời gian khởi động của container Lambda.

### 🧩 Topic: Event-Driven Fan-Out Architecture (SNS + SQS + Lambda)

#### 🔑 Keywords Nhận Diện
* `automate actions` + `run asynchronously` (tự động hóa và chạy bất đồng bộ).
* `multiple actions` / `multiple consumers` (ví dụ: cập nhật DB trong MySQL/ECS, index dữ liệu trong OpenSearch qua Lambda, gửi email thông báo qua Lambda/SES).
* `must not slow down the application` (không làm chậm ứng dụng chính).

#### 💡 Tip & Trick Chọn Đáp Án
* **Mô hình Pub/Sub Fan-Out chuẩn (SNS + SQS + Lambda):**
  * Để phân phối một sự kiện (event) tới nhiều xử lý bất đồng bộ, độc lập và song song: sử dụng **Amazon SNS Topic** làm đầu mối nhận tin nhắn (Publisher).
  * Đăng ký (Subscribe) trực tiếp các **Lambda functions** và một **Amazon SQS Queue** vào SNS Topic đó (cơ chế Fan-Out).
  * Với các dịch vụ dạng container/máy chủ như **Amazon ECS** hoặc EC2 vốn không hỗ trợ trigger trực tiếp từ SNS: hãy dùng **Amazon SQS** để hứng tin nhắn từ SNS, sau đó cho ứng dụng ECS poll tin nhắn từ SQS để xử lý.
* **Loại trừ các tùy chọn thiết kế tồi:**
  * *Xử lý tuần tự trong 1 Lambda:* Gộp tất cả các hành động vào 1 Lambda chạy tuần tự sẽ tăng độ trễ (latency), tăng thời gian thực thi của Lambda, không đảm bảo tính bất đồng bộ và tạo ra Single Point of Failure (nếu 1 dịch vụ lỗi sẽ kéo theo lỗi toàn bộ).
  * *Triển khai chuỗi gọi nối tiếp (Chaining):* Cấu hình Lambda này gọi trực tiếp Lambda kia sẽ làm tăng độ phức tạp kết nối (tight coupling) và khó quản lý lỗi.
  * *Sử dụng cơ chế Polling (quét định kỳ DB/EventBridge):* Quét định kỳ MySQL DB sẽ làm chậm DB, tăng độ trễ thông tin (không real-time) và không tối ưu bằng mô hình đẩy sự kiện (push-based event-driven).

### 🧩 Topic: Amazon Kinesis Data Firehose Data Transformation (on-the-fly Transformation)

#### 🔑 Keywords Nhận Diện
* `Kinesis Data Firehose delivery stream` (dòng truyền phân phối dữ liệu Firehose).
* `remove pattern-based customer identifiers` / `remove PII` / `transform data` (loại bỏ thông tin định danh/biến đổi dữ liệu trực tiếp).
* `store modified data in Amazon S3` (lưu dữ liệu sau khi biến đổi vào S3 bucket).

#### 💡 Tip & Trick Chọn Đáp Án
* **Sử dụng Lambda làm Data Transformation trong Firehose:**
  * Amazon Kinesis Data Firehose có tính năng tích hợp sẵn cho phép biến đổi dữ liệu trực tiếp trên luồng truyền (**Data Transformation**) trước khi ghi vào destination (như S3, Redshift, OpenSearch).
  * Giải pháp chuẩn và tối ưu nhất là kích hoạt tính năng này và liên kết với một **AWS Lambda function**. Lambda sẽ nhận các batch dữ liệu từ Firehose, thực hiện xử lý (ví dụ: dùng regex để lọc/bóc tách thông tin cá nhân PII như customer ID), sau đó trả về dữ liệu đã biến đổi cho Firehose để ghi xuống **Amazon S3** bucket.
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *Triển khai máy chủ EC2:* Việc dựng EC2 chạy ứng dụng xử lý dữ liệu thủ công là phức tạp, tốn kém chi phí hạ tầng rỗi và không tận dụng được cơ chế serverless tích hợp sẵn của Firehose. Hơn nữa, EC2 không phải là một "destination" hợp lệ mà Firehose hỗ trợ phân phối trực tiếp.
  * *Sử dụng OpenSearch làm destination để replace:* Việc cấu hình đẩy dữ liệu thô vào OpenSearch, thực hiện search-and-replace rồi xuất ra S3 là quá phức tạp, dư thừa tài nguyên và tốn chi phí lớn.
  * *Sử dụng AWS Step Functions làm destination:* Step Functions là dịch vụ quản lý trạng thái workflow, không hỗ trợ làm đích đến (destination) trực tiếp cho Kinesis Data Firehose delivery stream.

### 🧩 Topic: Amazon API Gateway Mock Integration for Testing

#### 🔑 Keywords Nhận Diện
* `Logic tier built with API Gateway and Lambda` (backend xây dựng bằng API Gateway & Lambda).
* `develop integration tests for the frontend` (phát triển các bài test tích hợp cho frontend).
* `cover both positive and negative scenarios` (test cả trường hợp thành công và thất bại với các HTTP status codes).
* `LEAST effort` (tốn ít công sức nhất).

#### 💡 Tip & Trick Chọn Đáp Án
* **Sử dụng API Gateway Mock Integration:**
  * Để cho phép frontend phát triển và test độc lập song song khi backend chưa xây dựng xong, giải pháp tốn ít công sức nhất là tạo **Mock Integration** trực tiếp trên API Gateway.
  * Trong cấu hình Mock Integration, bạn có thể thiết lập logic đơn giản ở **Integration Request** (sử dụng Mapping Template VTL để lấy tham số đầu vào như query string hoặc header để quyết định status code trả về) và cấu hình **Integration Response** tương ứng để trả về payload JSON mẫu kèm HTTP status codes mong muốn.
  * Việc này hoàn toàn không cần viết, deploy hay quản lý bất kỳ hàm Lambda backend nào, giúp tiết kiệm công sức nhất.
* **Loại trừ các tùy chọn sai/không tối ưu:**
  * *Tạo các hàm Lambda để giả lập response (Mock Lambda):* Việc viết code Lambda, cấu hình IAM policy và liên kết tích hợp với API Gateway mất nhiều công sức phát triển và quản trị hơn rất nhiều so với tính năng Mock native có sẵn của API Gateway.
  * *Tạo nhiều resource mock riêng biệt cho từng mã lỗi:* Thay vì tạo nhiều resource/route trùng lặp gây rườm rà, ta chỉ cần 1 API method duy nhất sử dụng Mock Integration động để phân tích request và trả về response tương ứng.

### 🧩 Topic: Dynamic Configuration & Feature Flags Management (AWS AppConfig & AppConfig Agent)

#### 🔑 Keywords Nhận Diện
* `use dynamic feature flags` (sử dụng các cờ tính năng động).
* `shared with other applications` (chia sẻ chung với các ứng dụng khác).
* `poll on an interval for new feature flag values` (phải quét định kỳ để lấy giá trị cờ mới).
* `values must be cached when they are retrieved` (giá trị phải được cache lại).
* `MOST operationally efficient way` (phương án tối ưu nhất về mặt vận hành).

#### 💡 Tip & Trick Chọn Đáp Án
* **Sử dụng AWS AppConfig kết hợp AWS AppConfig Agent:**
  * **AWS AppConfig** là dịch vụ thiết kế chuyên dụng của AWS để quản lý, lưu trữ và triển khai các cấu hình động và cờ tính năng (Feature Flags).
  * **AWS AppConfig Agent:** Là một helper process chạy trực tiếp trên EC2 instance (hoặc sidecar trong ECS/EKS). Agent này tự động xử lý toàn bộ việc quét định kỳ (polling) và quản lý bộ nhớ đệm (caching) cục bộ cho cờ tính năng. Ứng dụng chỉ cần gọi một HTTP GET request nội bộ đơn giản đến `http://localhost:2772` của Agent để lấy giá trị ngay lập tức với độ trễ cực thấp.
  * Giải pháp này giải phóng hoàn toàn lập trình viên khỏi việc tự viết code polling, code caching hoặc tự duy trì hạ tầng cache.
* **Loại trừ các tùy chọn sai/không tối ưu:**
  * *Dựng ElastiCache (Redis/Memcached) hoặc DynamoDB Accelerator (DAX):* Đây là các dịch vụ cache độc lập, đòi hỏi thiết lập cluster và viết code tích hợp phức tạp, tốn rất nhiều công vận hành và chi phí duy trì, không tối ưu cho dữ liệu cấu hình nhỏ như feature flags.
  * *Dùng SSM Parameter Store và tự viết code polling/in-memory cache:* Mặc dù Parameter Store lưu trữ được tham số, nhưng yêu cầu ứng dụng phải tự viết code polling định kỳ và tự quản lý cache trong memory bằng AWS SDK, tốn nhiều công sức phát triển và dễ phát sinh lỗi hơn so với giải pháp out-of-the-box của AppConfig Agent.

### 🧩 Topic: Amazon EBS gp2 Volume Performance Calculations (Baseline IOPS vs Max IOPS)

#### 🔑 Keywords Nhận Diện
* `General purpose SSD volume` / `gp2` (ổ cứng SSD mục đích chung thế hệ gp2).
* `hit the max IOPS` / `maximum performance limit` (đạt giới hạn hiệu năng IOPS tối đa).
* `volume size` (kích thước ổ cứng cần tính toán).

#### 💡 Tip & Trick Chọn Đáp Án
* **Công thức tính hiệu năng của EBS gp2:**
  * **Tỉ lệ baseline:** gp2 cung cấp hiệu năng cơ sở là **3 IOPS trên mỗi GiB** kích thước ổ đĩa.
    $$\text{Baseline IOPS} = \text{Volume Size (GiB)} \times 3$$
  * **Giới hạn tối đa (Max IOPS):** Hiệu năng tối đa của một ổ đĩa gp2 là **16,000 IOPS**. Dù kích thước ổ đĩa có lớn hơn nữa, IOPS cũng sẽ bị giới hạn ở con số này.
  * **Tính toán kích thước đạt Max IOPS:**
    $$\text{Size (GiB)} \times 3 = 16,000\text{ IOPS}$$
    $$\text{Size (GiB)} = \frac{16,000}{3} \approx 5,333.33\text{ GiB}$$
    * Quy đổi sang TiB (hệ nhị phân hoặc thập phân dùng làm tròn trong đề thi): $5,334\text{ GiB} \approx \mathbf{5.3\text{ TiB}}$ (hoặc $5.33\text{ TiB}$).
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *16 TiB:* Đây là kích thước ổ đĩa tối đa (max volume size) mà gp2 hỗ trợ, chứ không phải kích thước tối thiểu để đạt max IOPS.
  * *10.6 TiB, 2.7 TiB:* Các con số sai lệch do tính toán sai tỉ lệ IOPS/GiB.
