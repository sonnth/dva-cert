# Phần 1: Cơ bản về AWS, Quản lý định danh và Phân quyền (IAM)

## 1. Khởi đầu với AWS (Khu vực và Vùng khả dụng - Region & AZ)
- **Khu vực (Region):** Là vị trí địa lý thực tế chứa các trung tâm dữ liệu. Nên triển khai ứng dụng ở khu vực gần với người dùng nhất để giảm độ trễ (latency). 
- Không phải tất cả các dịch vụ đều có mặt ở tất cả các khu vực. Giá cả cũng thay đổi tùy theo từng khu vực.
- **Vùng khả dụng (Availability Zone - AZ):** Mỗi khu vực có nhiều AZ (thường từ 3 đến 6). Mỗi AZ là một hoặc nhiều trung tâm dữ liệu độc lập, tách biệt với nhau để phòng chống thảm họa.
- Hầu hết các dịch vụ AWS gắn với khu vực cụ thể. Các dịch vụ toàn cầu (Global) như IAM, Route 53, CloudFront không yêu cầu chọn khu vực.

## 2. Quản lý định danh và Phân quyền cơ bản (IAM)
- **Users (Người dùng) & Groups (Nhóm):** Có thể gán quyền (IAM policy - định dạng JSON).
- **Chính sách nội tuyến (Inline Policy):** Là chính sách gắn trực tiếp và duy nhất cho một người dùng (không qua nhóm).
- **Cấu trúc của IAM Policy:**
  - `Version`: Phiên bản ngôn ngữ của policy (thường là "2012-10-17").
  - `Id`: Định danh tùy chọn cho policy.
  - `Statement`: Một hoặc nhiều khối quy tắc (bắt buộc). Bao gồm:
    - `Sid`: Định danh tùy chọn.
    - `Effect`: Quyết định Cho phép (Allow) hoặc Từ chối (Deny).
    - `Principal`: Tài khoản/Người dùng/Vai trò (Role) mà policy này áp dụng (thường dùng trong Trust Policy hoặc Resource-based policy).
    - `Action`: Danh sách các hành động được cho phép/từ chối (ví dụ: `s3:GetObject`).
    - `Resource`: Danh sách các tài nguyên chịu tác động của hành động.
    - `Condition`: Các điều kiện áp dụng policy (tùy chọn, ví dụ: kiểm tra IP, MFA).
- **IAM Access Advisor (Truy cập gần nhất):** Xem người dùng đã truy cập dịch vụ nào và khi nào. Giúp tối ưu theo nguyên tắc "Đặc quyền tối thiểu".
- **IAM Credentials Report:** Báo cáo kiểm toán bảo mật cấp tài khoản về tất cả người dùng và chứng chỉ của họ.

## 3. AWS CLI, SDK và EC2 Instance Metadata
- **EC2 Instance Metadata (IMDS):** 
  - Cho phép máy chủ EC2 tự tìm hiểu thông tin về nó (ví dụ IP, Role) mà không cần dùng IAM Role.
  - URL truy cập: `http://169.254.169.254/latest/meta-data`
  - Có thể lấy tên IAM Role nhưng không lấy được nội dung IAM Policy.
  - IMDSv2 bảo mật hơn IMDSv1 nhờ yêu cầu Session Token.
- **AWS CLI (Giao diện dòng lệnh):** 
  - Quản lý cấu hình với các profile: `aws configure --profile my-profile`
  - Hỗ trợ chạy các lệnh bằng profile cụ thể: `aws s3 ls --profile my-profile`
- **Xác thực đa yếu tố (MFA) với CLI:** 
  - Để dùng MFA với CLI, phải tạo một phiên làm việc tạm thời thông qua STS:
  - Lệnh: `aws sts get-session-token --serial-number <ARN_thiết_bị_MFA> --token-code <Mã_MFA> --duration-seconds 3600`
- **Chữ ký AWS (Signature v4):** 
  - Khi gọi HTTP API của AWS, yêu cầu phải được ký (signed) để AWS nhận diện. SDK/CLI sẽ tự động làm điều này bằng quy chuẩn Signature v4 (SigV4).

## 4. Chuỗi cung cấp thông tin xác thực (Credentials Provider Chain)
AWS luôn tìm kiếm khóa xác thực theo một thứ tự ưu tiên nhất định (Chain). Đừng bao giờ hard-code (gắn cứng) thông tin xác thực trong mã nguồn.
- **Thứ tự của AWS CLI:**
  1. Tùy chọn dòng lệnh (Command line options): `--region`, `--profile`.
  2. Biến môi trường (Environment variables): `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`.
  3. File cấu hình thông tin xác thực (`~/.aws/credentials`).
  4. File cấu hình CLI (`~/.aws/config`).
  5. Thông tin xác thực container (cho ECS).
  6. Hồ sơ phiên bản (Instance profile) của máy chủ EC2.
- **Thực hành tốt nhất:** 
  - Khi code chạy trên AWS: Hãy dùng IAM Roles (EC2 Role, ECS Task Role, Lambda Role).
  - Khi code chạy ngoài AWS: Dùng biến môi trường hoặc cấu hình Profile cục bộ.

## 5. Giới hạn AWS (Limits & Quotas) và Exponential Backoff
- **Giới hạn tốc độ gọi API (API Rate Limits):** 
  - Ví dụ: `DescribeInstances` (EC2) giới hạn 100 lần/giây, `GetObject` (S3) giới hạn 5500 lần/giây/prefix.
  - **Lỗi không liên tục (Intermittent Errors):** Do gửi quá nhiều yêu cầu cùng lúc gây ra lỗi điều tiết `ThrottlingException` (Lỗi 429). Khắc phục bằng kỹ thuật **Exponential Backoff** (Tăng lùi khoảng thời gian thử lại - SDK mặc định đã có). 
    - Chỉ áp dụng thử lại cho lỗi máy chủ (5xx) và lỗi điều tiết (Throttling). Không thử lại với lỗi máy khách (4xx - sai tham số, cấm quyền).
  - **Lỗi nhất quán (Consistent Errors):** Gửi vé yêu cầu AWS nâng hạn mức API (Quota increase).

## 6. AWS STS (Security Token Service)
- Cho phép cấp quyền truy cập giới hạn và tạm thời vào các tài nguyên (thời hạn 15 phút đến 1 tiếng).
- Các lệnh gọi API phổ biến:
  - `AssumeRole`: Đóng vai trò (Role) trong cùng tài khoản hoặc liên tài khoản (Cross-account).
  - `AssumeRoleWithSAML` / `AssumeRoleWithWebIdentity`: Trả về quyền cho người dùng xác thực qua bên thứ 3 (Facebook, Google...). AWS khuyên dùng Cognito thay vì gọi trực tiếp API này.
  - `GetSessionToken`: Lấy token tạm thời (Dành cho tài khoản MFA hoặc tài khoản root).
  - `GetCallerIdentity`: Lấy thông tin về người dùng/Role đang thực hiện lời gọi API hiện tại (dùng để debug xem mình đang dùng user nào).

## 7. Định danh nâng cao (Advanced Identity)
### Đánh giá chính sách (Policy Evaluation Logic)
Nguyên tắc xét duyệt quyền của AWS:
1. Mặc định là TỪ CHỐI (DENY).
2. Nếu có bất kỳ dòng nào chỉ định TỪ CHỐI RÕ RÀNG (Explicit DENY) -> Lập tức bị từ chối (Ghi đè mọi ALLOW).
3. Nếu không có lệnh TỪ CHỐI RÕ RÀNG, và có ít nhất một lệnh CHO PHÉP RÕ RÀNG (Explicit ALLOW) -> Quyết định là CHO PHÉP.
4. Còn lại -> Từ chối.

### Tương tác giữa IAM Policy và S3 Bucket Policy
- S3 cho phép truy cập nếu: Có quyền IAM ALLOW **HOẶC** S3 Bucket Policy ALLOW. 
- Nhưng KHÔNG ĐƯỢC CÓ lệnh Explicit DENY ở cả hai nơi.
- Ví dụ: IAM Role không có quyền S3, nhưng Bucket Policy của S3 cho phép IAM Role đó đọc/ghi -> Role ĐƯỢC PHÉP.
- Ví dụ: IAM Role có quyền S3, nhưng Bucket Policy của S3 từ chối (Explicit DENY) IAM Role đó -> Role BỊ TỪ CHỐI.

### Chính sách động (Dynamic Policies)
- Thay vì tạo hàng trăm policy riêng cho từng thư mục của người dùng trong S3, có thể tạo 1 policy dùng biến môi trường nội bộ như `${aws:username}`.

### So sánh Managed Policy vs Inline Policy
- **AWS Managed Policy:** Do AWS duy trì, tự động cập nhật khi có dịch vụ mới (tốt cho admin/power user).
- **Customer Managed Policy:** Do người dùng tự tạo, dễ dàng tái sử dụng, kiểm soát phiên bản (Version Controlled), là **thực hành tốt nhất**.
- **Inline Policy:** Liên kết tỷ lệ 1-1 chặt chẽ với một Principal (người dùng/role). Sẽ bị xóa khi xóa principal đó.

### Truyền vai trò (PassRole)
- Để cấu hình cho các dịch vụ (EC2, Lambda, CodePipeline) sử dụng được Role, người thiết lập phải có quyền `iam:PassRole`. 
- Kèm theo thường là quyền `iam:GetRole` để xem role đang được truyền.
- Các dịch vụ (EC2, Lambda) chỉ có thể đảm nhận các role nếu role đó có định nghĩa **Trust Policy** cho phép dịch vụ cụ thể đó gọi `sts:AssumeRole`.

## 8. AWS Directory Service (AD)
Cung cấp quản lý thư mục người dùng cho hệ thống.
- **Microsoft Active Directory (AD):** Quản lý tập trung tài khoản, máy tính, cấu trúc dạng cây và rừng.
- **AWS Managed Microsoft AD:** Chạy AD đầy đủ trên AWS. Có thể thiết lập kết nối tin cậy (trust) với AD tại doanh nghiệp (on-premise).
- **AD Connector:** Là máy chủ proxy trung gian, chuyển hướng truy vấn xác thực về máy chủ AD tại doanh nghiệp. Không lưu dữ liệu trên AWS.
- **Simple AD:** Thư mục giá rẻ, tương thích cơ bản với AD trên AWS nhưng không thể kết nối hoặc thiết lập "trust" với AD nội bộ doanh nghiệp.
