# Phần 3: Kiến trúc Serverless & API

## 1. AWS Lambda
Lambda là dịch vụ tính toán không máy chủ (Serverless). Thay vì thuê máy chủ, bạn chỉ cần tải code lên và AWS sẽ chạy nó.
- **Lợi ích:** 
  - Không cần quản lý máy chủ. Mở rộng hoàn toàn tự động.
  - Tính tiền theo số mili-giây (ms) mã thực thi và số lượt gọi. Không chạy không tính tiền.
  - Cực kỳ rẻ, có mức miễn phí (Free tier) hàng tháng lên tới 1 triệu lượt gọi và 400.000 GB-giây tính toán.
- **Hỗ trợ ngôn ngữ:** Node.js, Python, Java, C#, Ruby, Custom Runtime (Golang, Rust...) và cả Container Image.

### Cấu hình và Hiệu năng
- **RAM:** Từ 128MB đến 10GB. Khi bạn tăng RAM, Lambda tự động tăng tỉ lệ sức mạnh CPU và băng thông mạng. Trên 1792MB, bạn có hơn 1 vCPU và có thể tận dụng đa luồng (multi-threading).
- **Thời gian chạy tối đa (Timeout):** Tối đa 900 giây (15 phút).
- **Không gian lưu trữ tạm (`/tmp`):** Tối đa 10GB. Dùng để chứa file tạm khi xử lý. 
- **Lambda Execution Context:** Sau khi code chạy xong, môi trường thực thi (context) và thư mục `/tmp` có thể được AWS đóng băng (freeze) và giữ lại một thời gian để tái sử dụng cho lượt gọi tiếp theo, giúp tăng tốc độ.
- **Cold Start (Khởi động lạnh):** Khi không có môi trường nào sẵn sàng, AWS mất một chút thời gian tải code và khởi tạo khiến lượt gọi đầu tiên bị chậm.
- **Mẹo thi về Concurrency (Đồng thời):**
  - *Reserved Concurrency (Đồng thời dự trữ):* Thiết lập giới hạn chạy đồng thời tối đa cho 1 hàm. Đảm bảo hàm đó luôn có chỗ để chạy (không bị hàm khác tranh giành) nhưng không hề giúp chống Cold Start. Không tính phí.
  - *Provisioned Concurrency (Đồng thời khởi tạo sẵn):* AWS luôn giữ các môi trường chạy khởi động sẵn. Khắc phục triệt để Cold Start nhưng bị tính phí liên tục.
- **Lambda trong VPC:** Mặc định Lambda chạy ngoài VPC. Khi gắn vào VPC (để gọi RDS/ElastiCache), Lambda sẽ tạo ENI (card mạng). Để Lambda trong VPC truy cập được Internet, cần cấu hình NAT Gateway.

### Gọi đồng bộ (Sync) vs Bất đồng bộ (Async)
- **Đồng bộ (Sync):** Chờ phản hồi trực tiếp (ví dụ: API Gateway, ALB). Máy khách phải tự xử lý lỗi và Retry.
- **Bất đồng bộ (Async):** Sự kiện được đẩy vào hàng đợi (Event queue). Nếu lỗi, Lambda tự động Retry tối đa 2 lần. Phù hợp cho S3, SNS, EventBridge. 
  - **Destinations:** Tính năng gửi kết quả (Thành công/Thất bại) đến SQS, SNS, Lambda hoặc EventBridge thay cho Dead Letter Queue.

### Event Source Mapping
Dùng để kéo (poll) dữ liệu từ các dịch vụ luồng (Kinesis, DynamoDB Streams) hoặc hàng đợi (SQS).
- **Kinesis & DynamoDB Streams:** Lambda đọc theo lô (batch). Kẹt một lỗi sẽ làm chậm cả shard (có tính năng ngắt lô khi lỗi để tiếp tục).
- **SQS Standard:** Lambda tăng dần 60 instance/phút để xử lý siêu tốc (tối đa 1000 batch/lúc).
- **SQS FIFO:** Scale theo số lượng Message Group ID hiện tại. Đảm bảo đúng thứ tự.

### Quản lý phiên bản và Triển khai
- **Versions:** Các phiên bản đã xuất bản là bất biến (immutable), không thể thay đổi, có đánh số (1,2,3...). Bản đang code là `$LATEST`.
- **Aliases (Bí danh):** Các nhãn (ví dụ: `PROD`, `DEV`) trỏ tới một phiên bản cụ thể. Hỗ trợ "Canary deployment" bằng cách chia trọng số (ví dụ: 90% truy cập vào bản 1, 10% vào bản 2).
- Tích hợp rất tốt với **CodeDeploy** để chuyển đổi lượng truy cập (Traffic Shifting) tự động và có Pre/Post-hook.

### Lambda@Edge vs CloudFront Functions
- **CloudFront Functions:** Viết bằng JS. Chạy siêu nhẹ, khởi động cực nhanh (sub-ms) để thay đổi nhỏ các request/response (đổi header, viết lại URL) ở cấp độ mạng biên.
- **Lambda@Edge:** Viết bằng NodeJS/Python. Thời gian chạy lâu hơn, cho phép truy cập mạng ngoài, xử lý phức tạp (gọi SDK, truy cập file) nhưng chậm hơn chút so với CloudFront Functions.

## 2. API Gateway
Dịch vụ tạo và quản lý API RESTful, HTTP hoặc WebSocket. Thường đi cặp với Lambda để tạo backend Serverless.
- **Endpoint Types (Loại cổng kết nối):**
  - *Edge-Optimized (Mặc định):* Dành cho client toàn cầu, request đi qua CloudFront.
  - *Regional:* Dành cho client trong cùng 1 khu vực (tránh đi đường vòng qua CloudFront).
  - *Private:* Chỉ có thể truy cập qua nội bộ VPC Endpoint.
- **Deployment & Stages:** Phải "Deploy" (Triển khai) API vào một "Stage" (ví dụ: `dev`, `prod`) thì mới sử dụng được. Có thể dùng **Stage Variables** (Biến môi trường) để trỏ đến các Lambda Alias khác nhau (Ví dụ: Stage `prod` gọi Lambda Alias `PROD`).
- **Canary Deployments:** Đẩy một phần trăm nhỏ lượng người dùng sang Stage cấu hình mới để test.

### Tích hợp và Mapping
- **Lambda Proxy Integration (HTTP_PROXY / AWS_PROXY):** API Gateway chuyển tiếp "toàn bộ" request gốc (headers, query strings) thẳng vào Lambda mà không làm gì thêm. Code Lambda tự bóc tách.
- **Mapping Templates (VTL):** Trích xuất, thay đổi hoặc làm sạch dữ liệu trước khi chuyển cho backend (hoặc trước khi trả về client). Giúp chuyển đổi từ XML sang JSON.
- Có khả năng cấu hình Swagger / OpenAPI, tự động sinh SDK.

### Bộ đệm (Caching) và Giới hạn (Throttling)
- **Caching:** API Gateway có thể lưu tạm kết quả (mặc định 300s) để giảm tải cho backend. Bộ đệm tính tiền theo giờ và kích thước. Client có thể ép làm mới (Invalidate cache) bằng header `Cache-Control: max-age=0` (nếu được cấp quyền).
- **Throttling (Giới hạn tỷ lệ):** Chống tấn công DDOS.
- **Mẹo thi về Mã Lỗi (Error Codes):** 
  - `429 Too Many Requests`: Vượt quá giới hạn (Throttling).
  - `502 Bad Gateway`: Lỗi do Backend (Lambda bị sập, hoặc Lambda trả về JSON sai định dạng).
  - `504 Gateway Timeout`: Phản hồi quá lâu (Do Lambda xử lý vượt quá 29 giây của API Gateway).
- **Usage Plans & API Keys:** Dùng để đóng gói API thành sản phẩm bán cho khách hàng. Phát API Key cho khách, gán vào Usage Plan để giới hạn số lượt truy cập trong tháng (Quota) và tốc độ gọi (Rate).

### Bảo mật & Xác thực (Authentication)
- **IAM Permissions:** Xác thực bằng tài khoản/Role AWS (SigV4). Phù hợp cho giao tiếp nội bộ trong AWS.
- **Cognito User Pools:** API Gateway tự động xác thực token từ Cognito. Dễ dàng, không cần viết code xác thực.
- **Lambda Authorizer (Custom Authorizer):** Gateway gọi một hàm Lambda trung gian để kiểm tra Token (JWT, OAuth) và trả về IAM Policy. Chuyên dùng cho các hệ thống đăng nhập của bên thứ 3 hoặc hệ thống kế thừa. Linh hoạt nhất nhưng tốn chi phí gọi Lambda thêm 1 lần.

## 3. Serverless Application Model (SAM)
- Framework lập trình dựa trên CloudFormation (YAML) chuyên dành cho kiến trúc Serverless.
- Khai báo nhanh gọn các resource đặc biệt: `AWS::Serverless::Function`, `AWS::Serverless::Api`, `AWS::Serverless::SimpleTable` (DynamoDB).
- Lệnh quan trọng: 
  - `sam build` -> `sam deploy` (hoặc `sam package`).
  - `sam sync --watch`: Đồng bộ code lên AWS theo thời gian thực (Bỏ qua CloudFormation nếu chỉ đổi code).
  - `sam local start-api` / `sam local invoke`: Chạy thử API Gateway và Lambda cục bộ (local) trên Docker.
- Hỗ trợ triển khai an toàn qua CodeDeploy và quản lý chính sách bảo mật có sẵn (Policy Templates).

## 4. AWS Step Functions
Hệ thống điều phối các dịch vụ AWS thành luồng trạng thái (State Machine).
- **Sử dụng:** Quy trình thanh toán, xử lý đơn hàng, điều phối hàng chục Lambda function làm việc với nhau.
- **Các loại State (Trạng thái):** 
  - `Task`: Gọi 1 dịch vụ AWS (Lambda, ECS, DynamoDB...).
  - `Choice`: Rẽ nhánh điều kiện (if/else).
  - `Wait`: Tạm dừng quy trình chờ tới mốc thời gian.
  - `Map`: Vòng lặp. `Parallel`: Chạy song song nhánh.
- **Xử lý lỗi:** Hỗ trợ `Retry` (thử lại) và `Catch` (chuyển sang bước khác nếu lỗi).
- **Wait for Task Token:** Step Function tạm dừng tại một bước và gửi Token ra hệ thống bên ngoài (ví dụ gửi email xin duyệt). Nó sẽ chờ tới 1 năm cho đến khi bên ngoài gọi API trả Token lại báo thành công.
- **Standard vs Express:**
  - *Standard:* Lịch sử thực thi lưu 90 ngày, thời gian tối đa 1 năm. Thanh toán theo mỗi lần chuyển state (Transitions). Chạy cho quy trình chuẩn.
  - *Express:* Lưu log ở CloudWatch, tối đa chạy 5 phút. Thanh toán theo số lần chạy, bộ nhớ. Dùng cho luồng dữ liệu (IoT, Streaming) siêu nhanh (hàng trăm ngàn lần/s).

## 5. AppSync & Amplify
- **AppSync:** Dịch vụ API Serverless nhưng sử dụng **GraphQL** (thay vì REST). Hỗ trợ lấy dữ liệu từ DynamoDB, Aurora, Lambda. Có khả năng trả dữ liệu thời gian thực (WebSockets) hoặc đồng bộ dữ liệu cho ứng dụng di động khi rớt mạng.
- **AWS Amplify:** Bộ công cụ front-end (React, Vue, iOS) để phát triển ứng dụng di động/web nhanh chóng. Cung cấp Hosting (CI/CD), Authentication (gắn với Cognito) và API (gắn với AppSync/DynamoDB). Hỗ trợ kiểm thử e2e (Cypress).

## 💡 Trích xuất trọng tâm luyện thi DVA-C02
- **Lỗi 500 khi API Gateway gọi Lambda động:** Khi dùng Stage Variables để trỏ đến các Lambda/Alias khác nhau, API Gateway mất khả năng tự động xin quyền. Bạn phải tự thiết lập **Resource-based policy** trên Lambda để cấp quyền `lambda:InvokeFunction` cho API Gateway.
- **Mock Integration cho API Gateway:** Phương án tốn ít công sức nhất để team frontend test luồng thành công/thất bại mà không cần team backend code Lambda là sử dụng **Mock Integration** ngay trên API Gateway.
- **Quản lý Feature Flags động:** Sử dụng **AWS AppConfig** kết hợp với **AppConfig Agent** (trên EC2/ECS). Agent sẽ tự động làm nhiệm vụ polling và caching cấu hình tại máy cục bộ, giải phóng dev khỏi việc tự viết code.
- **Khắc phục Cold Start Lambda khi có đợt tăng traffic (Sale):** Dùng **Provisioned Concurrency**, nhưng để tối ưu chi phí, hãy kết hợp với Application Auto Scaling để tăng/giảm Provisioned Concurrency **theo lịch trình** (on a schedule).
