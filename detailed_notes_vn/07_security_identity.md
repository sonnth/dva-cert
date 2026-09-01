# Phần 7: Bảo mật, Nhận dạng và Mã hóa (Security & Identity)

## 1. Cơ bản về Mã hóa (Encryption)
- **In-flight Encryption (Mã hóa đường truyền):** Mã hóa dữ liệu trước khi gửi và giải mã sau khi nhận (TLS/HTTPS). Chống bị nghe lén (Man In The Middle).
- **At-rest Encryption (Mã hóa tại chỗ):** Dữ liệu được mã hóa và lưu trữ ở dạng ổ đĩa (Server-side). Dữ liệu chỉ được giải mã khi có người dùng hợp lệ yêu cầu tải xuống.
- **Client-side Encryption (Mã hóa phía máy khách):** Khách hàng tự mã hóa file trước khi đẩy lên hệ thống của AWS. Kể cả nhân viên AWS cũng không thể giải mã được.

## 2. AWS KMS (Key Management Service)
Dịch vụ quản lý Khóa (Chìa khóa mã hóa). Mọi dịch vụ lưu trữ dữ liệu của AWS (S3, EBS, RDS...) đều được tích hợp sâu với KMS để mã hóa bằng 1 nút bấm (SSE-KMS).
- Gắn liền với **IAM** để cấp quyền cực kỳ chi tiết (Ai có chìa khóa mới đọc được file).
- Ghi nhật ký tất cả các lần sử dụng chìa khóa bằng **CloudTrail** để kiểm toán.

### Các loại khóa KMS
- **Symmetric (Khóa Đối xứng - AES-256):** Dùng một chìa khóa duy nhất để mã hóa và giải mã. Mặc định các dịch vụ AWS đều dùng loại này. Người dùng không bao giờ lấy được chìa khóa này ra khỏi AWS, chỉ có thể gọi lệnh API để AWS làm giúp.
- **Asymmetric (Khóa Bất đối xứng - RSA):** Cặp Public/Private key (Khóa công khai/bí mật). Dùng để mã hóa bên ngoài AWS bởi người dùng không có kết nối API. Khóa Public có thể tải về.
- **Các nhóm khóa & Chi phí:** 
  - Khóa AWS sở hữu (miễn phí), Khóa AWS quản lý tự động (Miễn phí), Khóa do Khách hàng tạo (Customer Managed Key - 1$/tháng, linh hoạt chia sẻ nhất).
- Phí gọi API là cực kỳ nhỏ (0.03$ / 10000 cuộc gọi API).

### Envelope Encryption (Mã hóa bao thư)
- API của KMS chỉ cho phép mã hóa 1 file dữ liệu **nhỏ hơn 4KB**.
- Với các file lớn hơn (Ảnh, Video, Dữ liệu GBs), hệ thống sử dụng **Envelope Encryption** (Lệnh `GenerateDataKey`).
  1. Yêu cầu KMS sinh ra một "Khóa dữ liệu" (DEK) gửi xuống app.
  2. KMS sẽ gửi lại 2 bản: Bản thô (Plaintext DEK) và Bản bị mã hóa (Encrypted DEK - mã hóa bằng Khóa chính của KMS).
  3. Ứng dụng dùng Bản thô để tự khóa cái file Video siêu to lại siêu tốc ngay tại máy của mình.
  4. Sau đó vứt Bản thô đi, nhét chung Bản bị mã hóa vào cùng file Video đem cất ở S3.
  5. Khi cần mở khóa: Lấy Bản bị mã hóa gửi lên cho KMS nhờ nó dịch ra bản thô lại. Sau đó lấy về để mở khóa video.

### Cấu trúc và Quản lý
- **AWS Encryption SDK:** Gói thư viện lập trình tự động làm quy trình mã hóa bao thư ở trên (Tích hợp tính năng *Data Key Caching* - giữ lại chìa khóa trong RAM để dùng lại cho hàng ngàn file nhằm tiết kiệm chi phí gọi KMS và tăng tốc).
- **KMS Key Policies:** Tương tự S3 Bucket Policy, là thứ **Bắt buộc** để kiểm soát xem ai (Role nào) có quyền dùng chìa khóa. Mặc định là quyền quản lý cho tài khoản Root. Để chia sẻ chìa khóa (ví dụ: Chép ổ cứng AMI qua tài khoản AWS của người khác), bắt buộc phải cập nhật Key Policy để cấp quyền liên tài khoản (Cross-account).

### Tối ưu chi phí S3 với KMS
- Tính năng **S3 Bucket Key**: S3 sẽ gọi KMS một lần để xin chìa khóa tạo ra Khóa gốc (Bucket Key). Sau đó tự S3 sinh ra hàng ngàn chìa khóa con cho mỗi file ảnh. Tính năng này giảm 99% lượng lệnh gọi API về KMS -> Giảm 99% chi phí KMS khổng lồ khi áp dụng mã hóa vào S3.

## 3. Các hệ thống bảo mật & lưu trữ khóa nâng cao
- **AWS CloudHSM:** Tương tự KMS, nhưng AWS cấp nguyên một phần cứng độc quyền chống giả mạo (FIPS 140-2 Level 3) cho công ty của bạn. Tự bạn quản lý toàn bộ 100%. (Khác với KMS là phần mềm dùng chung - Multi-tenant). Yêu cầu đối với quy định pháp lý ngành tài chính khắt khe.
- **SSM Parameter Store:** Dịch vụ lưu trữ biến số, chuỗi cấu hình. Miễn phí (Standard) lên tới 10.000 cấu hình, chuỗi siêu nhẹ. Tùy chọn tích hợp KMS để mã hóa mật khẩu.
- **AWS Secrets Manager:** Sinh ra chuyên để chứa "Mật khẩu CSDL", "API Token". Tính phí $$. *Tuyệt chiêu:* Tự động gắn với Lambda để kết nối trực tiếp vào RDS và tự động đổi mật khẩu CSDL ngẫu nhiên sau mỗi 30 ngày (Rotation). An toàn tuyệt đối.
- **AWS Nitro Enclaves:** Tạo môi trường máy ảo cực kỳ cô lập (Không mạng, không lưu trữ vĩnh viễn, không SSH truy cập). Dùng để bóc tách thông tin cá nhân (Thẻ tín dụng, sức khỏe y tế) để tính toán mà không sợ một đoạn mã độc nào trong app truyền dữ liệu ra ngoài Internet.
- **Amazon Macie:** AI tự động quét tất cả các file trong S3. Nếu phát hiện file nào chứa số thẻ tín dụng hay số CMND bị cấu hình hớ hênh công khai, lập tức báo động đỏ.

## 4. Amazon Cognito
Dịch vụ Quản lý Định danh, Cấp quyền và Đăng nhập cho người dùng hệ thống (Web App, Mobile App).

### Cognito User Pools (Hồ sơ Người dùng - Xác thực / Đăng nhập)
- Nơi lưu trữ thông tin Username, Password của khách hàng ứng dụng. Quản lý việc Đăng ký, Đăng nhập, Quên mật khẩu, Xác thực 2 bước (MFA).
- Hỗ trợ Mạng xã hội: Cho phép Đăng nhập bằng Google, Facebook...
- **Lambda Triggers:** Khi khách hàng bấm đăng ký, có thể cấu hình kích hoạt Lambda (Ví dụ: Pre Sign-up để kiểm tra xem tài khoản email đó có bị nằm trong sổ đen không, Post Confirmation để tự động gửi email chào mừng).
- Sau khi đăng nhập thành công, Cognito trả về mã **JSON Web Token (JWT)** cho ứng dụng để chứng minh đã đăng nhập.
- **Adaptive Authentication:** AI của Cognito đánh giá điểm rủi ro. Nếu IP đăng nhập từ quốc gia lạ, lập tức tự kích hoạt xác thực 2 bước MFA.
- Tích hợp chặt chẽ trực tiếp vào **API Gateway** hoặc **ALB (Load Balancer)** (Không cần viết code xác thực backend).

### Cognito Identity Pools (Hồ sơ Nhận dạng - Ủy quyền API)
- Chuyên dùng để đổi lấy "Quyền IAM tạm thời" cho thiết bị của khách hàng.
- Quy trình: Người dùng đăng nhập thành công qua Google (Hoặc Cognito User Pool) -> Lấy Token mang qua Identity Pool nộp -> Identity Pool cung cấp cho thiết bị di động đó 1 IAM Role (Ví dụ: Role cho phép tài khoản A được quyền Upload file trực tiếp lên thư mục `/S3/User_A/`).
- Khách quan trọng: Identity Pools hỗ trợ luôn cả việc cấp quyền IAM tối thiểu cho **Người dùng khách (Guest / Unauthenticated)** chưa đăng nhập (VD: Chỉ cho tải ảnh nền từ S3).

**Tóm tắt Cognito:** `User Pools` (Quản lý Đăng nhập, trả về Token) + `Identity Pools` (Cấp quyền IAM cho thiết bị lấy dữ liệu AWS trực tiếp). 2 dịch vụ này hoạt động bổ trợ cho nhau.

## 💡 Trích xuất trọng tâm luyện thi DVA-C02
- **Bảo mật và tính toàn vẹn code Lambda:** Để đảm bảo chỉ có code đáng tin cậy/được duyệt mới được deploy lên Lambda, phải dùng **AWS Signer** kết hợp tính năng **Code Signing** của Lambda (Không dùng KMS cho việc ký mã nguồn).
- **Mã hóa phong bì KMS (Envelope Encryption):** Áp dụng bắt buộc cho file > 4KB. Dùng API `GenerateDataKey`. Nguyên tắc: Lấy Data Key bản thô (Plaintext) để mã hóa file, sau đó XÓA bản thô ngay lập tức khỏi RAM và tuyệt đối không ghi xuống đĩa. Lưu bản Data Key đã mã hóa (Encrypted) cùng với file tĩnh.
- **AWS Encryption SDK:** Khi dùng SDK này, bạn không cần phải tự quản lý hay thiết kế Database để theo dõi các Data Key đã sử dụng. SDK tự động đóng gói Encrypted Data Key dưới dạng Metadata thẳng vào trong Header của bản mã hóa (Ciphertext).
- **Tự động xoay vòng mật khẩu CSDL (Automatic Rotation):** Để lưu credentials của RDS an toàn và có thể tự động đổi mật khẩu định kỳ mà không cần can thiệp thủ công, luôn chọn **AWS Secrets Manager** (nó tích hợp sẵn Lambda xoay vòng) thay vì SSM Parameter Store.
