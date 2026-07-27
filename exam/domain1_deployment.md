# AWS DVA-C02 Exam Tips & Tricks - Domain 1: Deployment

## 📌 Domain 1: Deployment (CI/CD, Beanstalk, CloudFormation)

### 🧩 Topic: CloudFormation Stack Deletion Troubleshooting (DELETE_FAILED)

#### 🔑 Keywords Nhận Diện
* `DELETE_FAILED`: Lỗi xảy ra khi xóa Stack nhưng một hoặc nhiều tài nguyên không thể xóa được (ví dụ: `[ASGInstanceRole12345678]`).
* `retain the resource` / `manually delete`: Đề bài hỏi cách giải quyết để hoàn thành việc xóa Stack và dọn dẹp tài nguyên.

#### 💡 Tip & Trick Chọn Đáp Án
* **Chọn giải pháp Retain + Manual Delete:** Hãy tìm đáp án đề cập đến việc thiết lập chính sách **Retain** (Giữ lại) cho tài nguyên bị lỗi trong template (sử dụng thuộc tính `DeletionPolicy: Retain` hoặc tích chọn Retain trên Console/CLI khi delete lại), sau đó xóa thủ công (manually delete) tài nguyên đó sau khi Stack đã được xóa xong.
* **Loại trừ bẫy `DependsOn`:** `DependsOn` chỉ định thứ tự khởi tạo/xóa tài nguyên trong template chứ không giải quyết được xung đột dependency thực tế khiến việc xóa tài nguyên bị block.
* **Loại trừ bẫy `--force`:** Lệnh CLI của CloudFormation không hỗ trợ tham số force-delete kết hợp với ARN của role cụ thể như mô tả trong các đáp án bẫy.

### 🧩 Topic: CloudFormation Stack Policy (Protecting Resources from Updates)

#### 🔑 Keywords Nhận Diện
* `parameter values were reset` / `ignored the latest changes made by the application` (giá trị cấu hình bị reset về mặc định và ghi đè các thay đổi runtime do ứng dụng tự chỉnh sửa khi cập nhật Stack).
* `avoid resetting the parameter values outside the stack` (tránh việc reset các giá trị này khi update/deploy lại stack).
* `LEAST development effort` (tốn ít công sức phát triển/code nhất).

#### 💡 Tip & Trick Chọn Đáp Án
* **Sử dụng CloudFormation Stack Policy:**
  * Theo mặc định, khi update Stack, CloudFormation sẽ tự động cập nhật mọi tài nguyên được định nghĩa trong template về trạng thái ban đầu của template, dẫn đến việc ghi đè (reset) các thay đổi được thực hiện trực tiếp bên ngoài stack (out-of-band updates).
  * Giải pháp ít tốn công nhất là áp dụng một **Stack Policy** (Chính sách Stack) để cấm hành động cập nhật (**Deny updates**) đối với các tài nguyên cấu hình này (ví dụ: các Systems Manager Parameter Store parameters). Việc này ngăn chặn CloudFormation ghi đè giá trị mới nhất mà không đòi hỏi chỉnh sửa code hay dịch chuyển dữ liệu.
* **Loại trừ các tùy chọn sai/không tối ưu:**
  * *DeletionPolicy: Retain:* Chỉ bảo vệ tài nguyên không bị xóa khi stack bị delete hoặc tài nguyên bị loại bỏ khỏi template, không ngăn chặn việc giá trị bị ghi đè khi cập nhật stack.
  * *Dịch chuyển sang DynamoDB hoặc RDS:* Đây là các giải pháp lưu trữ cấu hình rất tốt, nhưng đòi hỏi công sức phát triển lớn (phải viết code ứng dụng để đọc/ghi DB, tạo mới hạ tầng DB, migrate dữ liệu), không thỏa mãn yêu cầu "LEAST development effort".

### 🧩 Topic: Optimize EC2 Auto Scaling Launch Times (Golden Image vs Runtime Deployment)

#### 🔑 Keywords Nhận Diện
* `EC2 instances are taking a long time to become available` / `UserData script taking a long time` (EC2 khởi chạy lâu trong Auto Scaling do UserData chạy chậm).
* `make the most recent version of the application available` (luôn chạy version mới nhất của app).
* `apply all available security updates` (áp dụng đầy đủ các bản vá bảo mật).
* `minimize the number of images that are created` (giảm thiểu số lượng AMI cần build).
* `images must be validated` (AMI phải được kiểm thử/xác thực).

#### 💡 Tip & Trick Chọn Đáp Án
* **Kết hợp Golden Image (AMI) và Deploy tại Runtime:**
  * Để tối ưu hóa thời gian scale-out: Sử dụng **EC2 Image Builder** để tạo và xác thực (**validate**) một "Golden Image" (AMI) chứa sẵn hệ điều hành, các bản vá bảo mật (security patches) và các agent quản lý cần thiết. Điều này giúp loại bỏ pha chạy cài đặt hệ thống nặng nề từ UserData.
  * Để giảm thiểu số lượng AMI phải tạo (tránh việc cứ mỗi lần đổi code app lại phải build lại AMI): **Không bake code ứng dụng vào AMI**. Thay vào đó, hãy sử dụng **AWS CodeDeploy** để tự động triển khai phiên bản code mới nhất của ứng dụng vào EC2 instance tại runtime (thời điểm EC2 khởi chạy xong).
* **Loại trừ các tùy chọn sai/không tối ưu:**
  * *Bake ứng dụng trực tiếp vào AMI:* Mỗi khi cập nhật code ứng dụng sẽ bắt buộc phải build lại một AMI mới, làm tăng số lượng AMI được tạo, vi phạm yêu cầu "minimize the number of images".
  * *Loại bỏ các lệnh OS patching khỏi UserData:* Giúp tăng tốc boot nhưng vi phạm yêu cầu bắt buộc phải cài đặt các bản vá bảo mật mới nhất cho hệ thống.
  * *Sử dụng AWS CodePipeline để deploy tại runtime:* CodePipeline là dịch vụ điều phối luồng phát hành (CI/CD orchestrator), nó không trực tiếp thực hiện nhiệm vụ deploy code lên EC2 tại runtime (CodeDeploy mới là dịch vụ thực thi việc này).

### 🧩 Topic: CloudFront Cache Invalidation during Deployment (S3 + CloudFront Caching)

#### 🔑 Keywords Nhận Diện
* `static artifacts are stored in an Amazon S3 bucket` (các file tĩnh lưu trữ trong S3 bucket).
* `deploys some changes and can see new artifacts in S3` (đã deploy và thấy file mới trong S3).
* `changes do not appear on the webpage that the CloudFront distribution delivers` (nhưng trang web qua CloudFront vẫn hiển thị nội dung cũ).
* `What should the developer configure the buildspec.yml file to do to resolve this issue?` (cần cấu hình buildspec.yml làm gì để giải quyết lỗi cache).

#### 💡 Tip & Trick Chọn Đáp Án
* **Sử dụng CloudFront Cache Invalidation:**
  * CloudFront lưu trữ tạm thời (cached) các đối tượng tĩnh tại các Edge Location để giảm độ trễ truy cập. Khi bạn upload phiên bản mới của file lên S3 với cùng tên file cũ (ví dụ: `index.html`), CloudFront vẫn sẽ phân phối bản cũ đã cache cho đến khi thời gian sống (TTL) của cache đó hết hạn.
  * Để cập nhật nội dung mới ngay lập tức cho người dùng, giải pháp chuẩn là thực hiện hành động **Invalidate the cache (Vô hiệu hóa cache/Xóa cache)** trên CloudFront distribution cho các file vừa thay đổi (ví dụ: `/*` hoặc `/index.html`).
  * **Tự động hóa trong CI/CD pipeline (`buildspec.yml`):** Khi sử dụng CodePipeline và CodeBuild để build và deploy static website lên S3, bạn cần cấu hình file `buildspec.yml` chạy câu lệnh AWS CLI để vô hiệu hóa cache (`aws cloudfront create-invalidation --distribution-id <ID> --paths "/*"`) ngay sau khi deploy.
* **Loại trừ các tùy chọn sai/không phù hợp:**
  * *S3 Object Lock:* Đây là tính năng ghi một lần đọc nhiều lần (WORM) để chống xóa/sửa đổi file trong S3 vì lý do tuân thủ pháp lý, hoàn toàn không liên quan đến cơ chế cache của CloudFront.
  * *Xóa toàn bộ đối tượng cũ trong S3 trước khi upload hoặc re-sync:* Việc này chỉ tác động ở tầng S3 chứ không giải phóng hay làm mới các file đã được cache tại các Edge Location của CloudFront.
  * *Sửa đổi Origin hoặc cấu hình CORS của S3:* Origin và CORS không liên quan đến việc cache CDN bị lỗi thời sau khi deploy.

### 🧩 Topic: AWS Elastic Beanstalk Deployment Policies (Speed vs Availability)

#### 🔑 Keywords Nhận Diện
* `Elastic Beanstalk` (triển khai Beanstalk).
* `test a new version of an application in a test environment` (thử nghiệm app trong môi trường test/dev).
* `FASTEST deployment` (phương thức deploy nhanh nhất).

#### 💡 Tip & Trick Chọn Đáp Án
* **Phương thức "All at once" nhanh nhất:**
  * **All at once:** Triển khai phiên bản mới lên toàn bộ các EC2 instance đồng thời. Đây là phương thức **nhanh nhất** vì không thực hiện phân đợt (batching) hay khởi tạo tài nguyên mới. Tuy nhiên, nó sẽ gây ra một khoảng thời gian downtime ngắn khi tất cả các instance khởi động lại app. Rất phù hợp cho môi trường kiểm thử/dev nơi downtime không ảnh hưởng khách hàng.
* **Hiểu các phương thức khác để phân biệt:**
  * *Rolling:* Deploy theo từng đợt (batch). Sẽ mất nhiều thời gian hơn All at once và làm giảm công suất (capacity) của hệ thống trong quá trình deploy.
  * *Rolling with additional batch:* Khởi tạo thêm một đợt instance mới trước rồi mới deploy cuốn chiếu. Duy trì 100% capacity nhưng chậm hơn Rolling do mất thời gian khởi động EC2 mới.
  * *Immutable:* Khởi tạo một group EC2 instance mới song song để deploy, kiểm tra thành công rồi mới switch traffic và xóa group cũ. Cực kỳ an toàn, tránh downtime nhưng là phương thức **chậm nhất** do phải khởi tạo lượng tài nguyên lớn gấp đôi.

### 🧩 Topic: AWS SAM Local Testing & Event Generation (sam local generate-event)

#### 🔑 Keywords Nhận Diện
* `AWS Serverless Application Model (AWS SAM)` (mô hình ứng dụng serverless).
* `test AWS Lambda functions locally` (test hàm Lambda cục bộ).
* `test event payloads to match the actual events that AWS services create` (cần cấu trúc payload giống hệt sự kiện thực tế của các dịch vụ AWS tạo ra).
* `LEAST development effort` (tốn ít công sức phát triển nhất).

#### 💡 Tip & Trick Chọn Đáp Án
* **Sử dụng lệnh `sam local generate-event`:**
  * AWS SAM CLI tích hợp sẵn một câu lệnh vô cùng mạnh mẽ để sinh tự động các payload sự kiện giả lập của các dịch vụ AWS khác nhau (như S3, SQS, API Gateway, DynamoDB Streams...): **`sam local generate-event`**.
  * Ví dụ: `sam local generate-event s3 put` sẽ sinh ra cấu hình JSON chuẩn của sự kiện S3 PutObject.
  * Việc này giúp nhà phát triển có ngay payload chuẩn để chạy test cục bộ (`sam local invoke`) mà không cần tốn bất kỳ công sức nào để tìm kiếm, tự viết cấu trúc JSON phức tạp bằng tay.
* **Loại trừ các tùy chọn sai/không tối ưu:**
  * *Tự viết thủ công (manually create) rồi lưu ở local hoặc S3:* Cấu trúc JSON sự kiện của AWS (như API Gateway AWS Proxy) cực kỳ phức tạp và dài dòng. Việc tự viết tay hay đi copy thủ công tốn rất nhiều công sức (vi phạm yêu cầu LEAST development effort).
  * *Lưu file test ở Amazon S3:* Không cần thiết và làm phức tạp quá trình kiểm thử cục bộ ngoại tuyến (offline).
