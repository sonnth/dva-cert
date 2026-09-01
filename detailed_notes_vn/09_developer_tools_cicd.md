# Phần 9: CI/CD & Cơ sở hạ tầng dưới dạng mã (Infrastructure as Code)

## 1. Cơ sở hạ tầng dưới dạng mã (IaC)

### AWS CloudFormation
Dịch vụ giúp định nghĩa và triển khai tự động toàn bộ cơ sở hạ tầng mạng, máy chủ trên AWS thông qua các tập tin code (YAML hoặc JSON).
- **Lợi ích:** Đưa hệ thống vào quản lý phiên bản (Version Control). Xóa/tạo lại tài nguyên chỉ bằng 1 nút bấm (Stack). Ước lượng chi phí trước khi triển khai. Giữ sự nhất quán cao độ.
- **Thành phần chính của Template:**
  - `Resources` (Bắt buộc): Các tài nguyên AWS cần tạo (VD: EC2, S3, RDS...).
  - `Parameters`: Các biến đầu vào (VD: Chọn loại máy t2.micro hay m5.large lúc bắt đầu chạy).
  - `Mappings`: Bảng giá trị cứng (VD: Nếu Region là us-east-1 thì dùng AMI id X, nếu us-west-1 thì dùng AMI Y).
  - `Outputs`: Các thông tin xuất ra sau khi chạy xong. Có thể `Export` để các Stack CloudFormation khác `ImportValue` sử dụng (Rất phổ biến để chia nhỏ hệ thống lớn thành nhiều mạng lưới độc lập).
  - `Conditions`: Điều kiện if/else để tạo hoặc không tạo tài nguyên (VD: Nếu là môi trường PROD thì tạo DB Multi-AZ).
- **Intrinsic Functions (Các hàm tích hợp):**
  - `!Ref`: Tham chiếu tới Parameter hoặc trả về ID vật lý của Resource (VD: EC2 Instance ID).
  - `!GetAtt`: Lấy một thuộc tính cụ thể của Resource (VD: Lấy địa chỉ IP Public của EC2, hoặc URL của S3).
  - `!Sub`: Dùng để nối chuỗi (Thay thế biến vào trong String).
- **Xử lý lỗi (Rollbacks):** Mặc định, nếu gặp lỗi giữa chừng, CloudFormation sẽ tự động HỦY (xóa) toàn bộ các resource vừa tạo trước đó để giữ hệ thống sạch sẽ.
- **Bảo vệ tài nguyên:** Sử dụng thuộc tính `DeletionPolicy` (Giá trị: `Retain` để giữ lại không bị xóa khi Stack bị xóa, `Snapshot` để chụp ảnh lưu lại trước khi xóa). Dùng `TerminationProtection` để chặn ai đó vô tình xóa nguyên Stack.
- **Custom Resources (Tài nguyên tùy chỉnh):** Cho phép gọi Lambda Function trong quá trình chạy CloudFormation để tạo ra các logic phức tạp (Ví dụ: Chạy Lambda để dọn rác S3 bucket trước khi CloudFormation có thể xóa Bucket đó).
- **StackSets:** Tính năng cho phép triển khai CloudFormation đồng thời tới *nhiều tài khoản AWS* và *nhiều khu vực* cùng một lúc.

### AWS Cloud Development Kit (CDK)
- Thay vì viết YAML/JSON rối rắm và dài dòng như CloudFormation, CDK cho phép bạn định nghĩa hạ tầng bằng ngôn ngữ lập trình "xịn" (TypeScript, Python, Java, C#).
- Trình biên dịch sẽ chuyển code ngôn ngữ bậc cao thành file CloudFormation tương ứng, sau đó tự động triển khai.
- **Cấp độ Constructs:**
  - *L1 (Cấp 1):* Ánh xạ 1:1 trực tiếp với các định nghĩa JSON/YAML gốc của CloudFormation. (Ký hiệu tiền tố: `Cfn`).
  - *L2 (Cấp 2):* Bọc sẵn các phương thức hỗ trợ (VD: Thay vì viết policy JSON phức tạp, chỉ cần gọi hàm `bucket.addLifecycleRule()`).
  - *L3 (Cấp 3 - Patterns):* Gói toàn bộ hệ thống (VD: 1 dòng code tạo ra luôn VPC, ECS Fargate, ALB Load Balancer).
- **Bootstrapping:** Trước khi chạy CDK lần đầu tiên trong 1 Region/Account, phải chạy lệnh `cdk bootstrap` để nó tạo S3 chứa code và gán quyền IAM cần thiết.

## 2. CI/CD (Tích hợp & Giao hàng liên tục)
Bộ công cụ của AWS giúp tự động hóa quy trình phần mềm từ lúc dev bấm "Git push" đến khi lên môi trường Production hoàn chỉnh (CodeCommit -> CodeBuild -> CodeDeploy -> CodePipeline).

### AWS CodeCommit
- Dịch vụ lưu trữ mã nguồn tư nhân (Git repository) quản lý hoàn toàn.
- Không giới hạn kích thước, độ ổn định cực cao. Mã hóa tự động bằng KMS.
- Xác thực bằng HTTPS (yêu cầu IAM credential) hoặc SSH (tải khóa pub/priv lên IAM). Không bao giờ cấp thẳng thông tin tài khoản root.

### AWS CodeBuild
- Dịch vụ máy chủ biên dịch (Build & Test). Hỗ trợ Java, Python, Node, Docker...
- Tự động lấy code, bật một máy chủ tạm thời, làm việc, sau đó tắt máy và tính tiền theo phút. 
- Yêu cầu tập tin **`buildspec.yml`** đặt tại thư mục gốc mã nguồn.
  - Chứa các `phases` (Giai đoạn): `install` (cài thư viện), `pre_build` (đăng nhập docker), `build` (chạy compile/test), `post_build` (nén zip).
  - Thuộc tính `artifacts` chỉ định những file sẽ được đẩy lên S3 sau khi build xong.
  - Có thể nén cache các thư viện để lần chạy tiếp theo siêu tốc.
- Hỗ trợ cắm biến môi trường (Environment variables) thẳng từ AWS Parameter Store hoặc Secrets Manager cực kỳ an toàn.

### AWS CodeDeploy
- Dịch vụ tự động triển khai phiên bản phần mềm lên EC2, On-Premises, Lambda và ECS.
- **Triển khai lên EC2/ASG:** 
  - Đòi hỏi cài đặt phần mềm *CodeDeploy Agent* trên máy ảo đích.
  - Sử dụng file **`appspec.yml`** khai báo đường dẫn copy file và các Hooks (các kịch bản chạy trước/sau khi tắt phần mềm cũ, bật phần mềm mới). 
    - **Mẹo thi:** Phải nhớ CHÍNH XÁC thứ tự chạy các Hooks quan trọng: `ApplicationStop` -> `DownloadBundle` -> `BeforeInstall` -> `Install` -> `AfterInstall` -> `ApplicationStart` -> `ValidateService`.
  - Hỗ trợ triển khai *In-place* (Tải code đè lên máy cũ - Nhanh nhưng có gián đoạn) hoặc *Blue/Green* (Tạo nguyên một nhóm máy ảo mới, cài code, sau đó nắn lượng truy cập qua bên mới - Cực kỳ an toàn).
- Hỗ trợ Rollback (Quay về bản cũ) tự động ngay khi triển khai bị lỗi hoặc khi CloudWatch Alarm bị kích hoạt.
- Có khả năng cấu hình tốc độ triển khai (*AllAtOnce*, *HalfAtATime*, *OneAtATime*).

### AWS CodePipeline
- Trình điều phối luồng quy trình (Orchestration).
- Ghép nối CodeCommit, CodeBuild, CodeDeploy và các bên thứ ba (GitHub, Jenkins) thành một đường ống.
- Hỗ trợ **Manual Approval** (Xin phép phê duyệt thủ công): Có thể cấu hình hệ thống dừng lại, gửi thông báo qua SNS (Email, Slack), chờ một quản lý bấm nút "Đồng ý" thì luồng mới chạy tiếp.

### Các công cụ lập trình khác (Developer Tools)
- **AWS CodeArtifact:** Nơi lưu trữ, công bố và chia sẻ các gói thư viện nội bộ công ty (như npm registry, maven, pip). Hỗ trợ kéo về lưu các gói từ bên ngoài giúp công ty không bị sập nếu kho ngoài bảo trì, đồng thời kiểm soát mã độc.
- **AWS CodeGuru:** Máy học tự động review mã nguồn. Gồm *Reviewer* (Dò tìm lỗi ẩn, hổng bảo mật rò rỉ bộ nhớ lúc đang code) và *Profiler* (Tìm nguyên nhân gây tiêu hao nhiều CPU, tốn thời gian chạy nhất trên hệ thống đang chạy thực tế). Mức độ dò cực sâu (MaxStackDepth).

## 💡 Trích xuất trọng tâm luyện thi DVA-C02
- **Lỗi xóa CloudFormation Stack (DELETE_FAILED):** Sử dụng thiết lập `DeletionPolicy: Retain` cho tài nguyên bị lỗi để Stack có thể tiếp tục xóa thành công, sau đó xóa thủ công tài nguyên đó (Manual delete). Không dùng `DependsOn` hay lệnh force.
- **Bảo vệ tài nguyên CloudFormation:** Để ngăn việc cập nhật Stack ghi đè (reset) lại các cấu hình đã được thay đổi bên ngoài (ví dụ SSM Parameter), giải pháp tốn ít công sức nhất là dùng **Stack Policy** (Deny updates) thay vì sửa đổi kiến trúc.
- **Tối ưu tốc độ EC2 Auto Scaling:** Dùng EC2 Image Builder tạo "Golden Image" (AMI) chứa OS và security patches để giảm thời gian khởi động từ UserData. KHÔNG bake code ứng dụng vào AMI, mà dùng CodeDeploy triển khai code tại runtime để tối thiểu hóa số lượng AMI phải quản lý.
- **Tốc độ Deploy Elastic Beanstalk:** Phương thức **All at once** là nhanh nhất (phù hợp test). Phương thức **Immutable** là an toàn (không rớt mạng) nhưng chậm nhất.
- **Kiểm thử SAM Local:** Sử dụng lệnh `sam local generate-event` để tự động sinh cấu trúc payload JSON (của S3, API Gateway...) giống hệt sự kiện thực tế mà tốn ít công sức nhất.
