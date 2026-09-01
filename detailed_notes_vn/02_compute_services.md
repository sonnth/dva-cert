# Phần 2: Dịch vụ Máy chủ tính toán (Compute)

## 1. Cơ bản về EC2 (Elastic Compute Cloud)
EC2 cung cấp cơ sở hạ tầng dưới dạng dịch vụ (IaaS). Bạn có thể thuê máy chủ ảo, cấu hình CPU, RAM, ổ cứng (EBS), mạng lưới, và thiết lập tường lửa (Security Group).
- **Bootstrap Script (User Data):** Tập lệnh chạy duy nhất 1 lần khi EC2 khởi động lần đầu (bằng quyền root). Thường dùng để cập nhật hệ điều hành, cài đặt ứng dụng, tải mã nguồn.

### Các loại máy chủ EC2 (Instance Types)
Cách đặt tên: Ví dụ `m5.2xlarge` (`m`: dòng máy, `5`: thế hệ thứ 5, `2xlarge`: kích cỡ phần cứng).
- **General Purpose (Đa dụng - dòng M, T):** Cân bằng giữa Compute, Memory và Network. Dành cho web server, code repository.
- **Compute Optimized (Tối ưu tính toán - dòng C):** Bộ xử lý hiệu năng cao. Dành cho xử lý hàng loạt (batch), render video, AI/ML.
- **Memory Optimized (Tối ưu bộ nhớ - dòng R, X):** Dành cho xử lý tập dữ liệu lớn trong RAM. Phù hợp cho Database in-memory (Redis), xử lý dữ liệu lớn (Big Data).
- **Storage Optimized (Tối ưu lưu trữ - dòng I, D):** Đọc/ghi (I/O) tuần tự tốc độ cao vào ổ đĩa. Dành cho kho dữ liệu (Data warehouse), NoSQL, hệ thống phân tán.

### Tùy chọn mua EC2 (Purchasing Options)
- **On-demand (Theo nhu cầu):** Trả tiền theo giây/giờ, không cam kết. Tốt cho các công việc ngắn hạn, khó dự đoán, không bị gián đoạn. Đắt nhất.
- **Reserved Instances (Dự trữ):** Cam kết 1 hoặc 3 năm. Giảm giá đến 72%. Tốt cho CSDL, ứng dụng chạy 24/7.
  - *Convertible:* Cho phép thay đổi loại máy tính (OS, gia đình máy). Giảm giá thấp hơn (66%).
- **Savings Plans:** Cam kết tiêu một số tiền cố định (ví dụ: $10/giờ) trong 1 hoặc 3 năm. Linh hoạt vượt giới hạn sẽ tính giá on-demand.
- **Spot Instances:** Giảm giá đến 90%. Nhưng AWS có thể thu hồi máy bất cứ lúc nào (sau khi báo trước 2 phút) nếu giá thị trường vượt giá bạn đặt. Phù hợp cho tính toán linh hoạt, Batch job, render video. Không dùng cho CSDL hoặc web server cần chạy liên tục.
- **Dedicated Hosts/Instances:** Dành riêng phần cứng vật lý. Hosts cho phép kiểm soát phần cứng vật lý cấp thấp để dùng giấy phép phần mềm hiện có của công ty (BYOL). Rất đắt.
- **Capacity Reservations:** Đặt trước dung lượng ở một AZ cụ thể, đảm bảo lúc nào cũng có thể khởi chạy máy tính. 

## 2. Security Groups (Tường lửa mạng)
- Kiểm soát lưu lượng truy cập (Inbound / Outbound).
- Mặc định: Cho phép TẤT CẢ các kết nối đi ra (Outbound) và chặn TẤT CẢ kết nối đi vào (Inbound).
- Các luật (rules) chỉ là "CHO PHÉP" (ALLOW). Không thể tạo luật chặn.
- Có thể tham chiếu theo IP hoặc tham chiếu tới một Security Group khác (cho phép tất cả máy EC2 nằm trong SG đó được truy cập).
- Tồn tại tách biệt bên ngoài EC2, một SG có thể gắn cho nhiều EC2 và ngược lại.
- **Khắc phục sự cố:** 
  - Bị `Timeout`: Chắc chắn là lỗi do Security Group hoặc cấu hình mạng chặn.
  - Bị `Connection refused`: Do lỗi ứng dụng trong máy chủ chưa chạy hoặc sai cổng, không phải do SG.

## 3. Lưu trữ EC2: EBS, Instance Store & EFS
### Amazon EBS (Elastic Block Store)
- Là ổ cứng lưu trữ khối nối mạng gắn vào EC2. Dữ liệu vẫn được giữ lại dù máy ảo bị dừng.
- Gắn được vào 1 máy EC2 tại 1 thời điểm (ngoại trừ chế độ Multi-Attach cho phép io1/io2 gắn vào 16 EC2).
- Bị khóa trong một Vùng Khả Dụng (AZ). Ổ EBS ở `us-east-1a` không thể cắm vào máy ảo ở `us-east-1b`.
- **EBS Volume Types:**
  - `gp2/gp3` (SSD): Đa dụng, cân bằng hiệu năng và giá. 
  - `io1/io2` (SSD): Hiệu năng cao (Provisioned IOPS). Dùng cho CSDL cường độ lớn. Có hỗ trợ Multi-Attach.
  - `st1` / `sc1` (HDD): Rẻ, không thể làm ổ khởi động. Dành cho Big data hoặc kho lưu trữ ít truy cập.
- **EBS Snapshots:** Chụp bản sao lưu tại một thời điểm (Point-in-time backup). Có thể sao chép Snapshot sang AZ hoặc Region khác, giúp di dời ổ cứng. Có tính năng Archive (lưu trữ siêu rẻ) và Recycle Bin (khôi phục snapshot lỡ xóa).

### EC2 Instance Store
- Ổ cứng phần cứng cắm trực tiếp vào máy chủ vật lý chứa EC2.
- Hiệu suất (I/O) cực cao.
- Là lưu trữ tạm thời (Ephemeral): Dữ liệu MẤT SACH khi máy ảo bị dừng (stop) hoặc tắt, phần cứng hỏng. Thích hợp cho bộ nhớ đệm (Cache), buffer.

### EFS (Elastic File System)
- Hệ thống file chia sẻ dạng mạng (NFS). Có thể mount (gắn) vào hàng trăm máy EC2 trên nhiều AZ cùng một lúc. Rất đắt (gấp 3 lần gp2).
- Tự động mở rộng mà không cần cấp phát trước.
- **Chế độ hiệu năng:** General Purpose (mặc định) và Max I/O.
- Dùng cho chia sẻ dữ liệu nội bộ, web serving chung, Content Management System (WordPress). 
- Chỉ tương thích với EC2 chạy Linux.

## 4. Cân bằng tải (ELB) & Tự động mở rộng (ASG)
- **Scale Up (Vertical):** Nâng cấp cấu hình (RAM, CPU).
- **Scale Out (Horizontal):** Tăng số lượng máy ảo. 
- **High Availability (HA):** Luôn chạy ứng dụng ở ít nhất 2 vùng khả dụng (AZ) để chống thảm hoạ chết trung tâm dữ liệu.

### Elastic Load Balancers (ELB)
- **ALB (Application Load Balancer):** Layer 7 (HTTP/HTTPS/gRPC). Cân bằng tải chuyên dụng cho ứng dụng web, microservices. Hỗ trợ chuyển hướng dựa trên đường dẫn (path), hostname, headers. IP của máy khách được truyền qua header `X-Forwarded-For`.
- **NLB (Network Load Balancer):** Layer 4 (TCP/UDP). Dùng cho ứng dụng cần hiệu suất siêu khủng, xử lý hàng triệu truy vấn/giây, độ trễ siêu thấp. Cung cấp **IP tĩnh (Static IP)** cho mỗi AZ.
- **GWLB (Gateway Load Balancer):** Layer 3 (Mạng). Dùng cho các máy chủ quét bảo mật (Tường lửa thế hệ 3, Intrusion Detection). Dùng giao thức GENEVE (cổng 6081).
- **Tính năng mở rộng:**
  - **Sticky Sessions:** Gắn chặt một người dùng vào cùng 1 máy EC2 trong một khoảng thời gian bằng Cookie. Giúp không bị mất session đăng nhập, nhưng có thể gây mất cân bằng tải.
  - **Cross-Zone Load Balancing:** Đảm bảo lượng truy cập được chia đều cho tất cả máy EC2 trên TẤT CẢ các AZ, thay vì chia đều cho mỗi AZ. Bật mặc định cho ALB, tắt mặc định cho NLB/GWLB.
  - **SSL/TLS Certificates:** Tích hợp chứng chỉ với AWS ACM. ALB/NLB hỗ trợ **SNI (Server Name Indication)** giúp phục vụ nhiều chứng chỉ SSL trên cùng 1 ALB cho nhiều tên miền.
  - **Connection Draining (Deregistration Delay):** Khi một máy EC2 bị đánh dấu là unhealthy hoặc đang gỡ bỏ, ELB ngừng gửi truy vấn mới và chờ N giây (mặc định 300s) để máy tính này hoàn tất xử lý nốt các "yêu cầu đang dang dở".

### Auto Scaling Groups (ASG)
- Mục đích: Tự động Tăng (Scale out) hoặc Giảm (Scale in) số lượng máy tính EC2 dựa trên tải thực tế, và tự thay thế máy hỏng. Hoàn toàn miễn phí, chỉ trả tiền cho EC2 tạo ra.
- Sử dụng **Launch Template** (Khuôn mẫu): Khai báo AMI, loại máy, Keypair, Security Group, IAM Role, User data.
- **Chính sách mở rộng (Scaling Policies):**
  - **Dynamic Tracking (Theo dõi mục tiêu):** Ví dụ: "Luôn giữ CPU trung bình toàn nhóm ở mức 40%". Rất tiện lợi.
  - **Step / Simple Scaling:** Dựa trên cảnh báo CloudWatch Alarm.
  - **Scheduled Scaling:** Mở rộng theo lịch trình (ví dụ: tăng máy vào 5h chiều thứ Sáu).
  - **Predictive Scaling:** Dùng AI dự đoán theo thói quen để scale trước khi hệ thống quá tải.
- **Cooldown (Thời gian chờ):** Sau mỗi hành động tăng/giảm, ASG sẽ dừng khoảng 300s để thông số ổn định trước khi quyết định scale tiếp.

## 5. Elastic Beanstalk
- Là giải pháp **PaaS (Platform as a Service)**. Cực kỳ tiện cho Developer: Chỉ cần nạp mã nguồn, AWS lo toàn bộ cơ sở hạ tầng (EC2, ASG, ELB), theo dõi, cấu hình và scaling.
- Quản lý theo cấu trúc: Application -> Environments -> Versions.
- Hỗ trợ nhiều nền tảng: Java, .NET, Node.js, PHP, Python, Docker...
- Tùy biến thông qua thư mục `.ebextensions/` (chứa các file `.config` định dạng YAML/JSON) để tùy chỉnh hạ tầng (cài biến môi trường, thiết lập thêm RDS, ElastiCache...).

### Các phương pháp triển khai (Deployment Modes)
- **All at once:** Triển khai cùng lúc cho tất cả máy. Nhanh nhất nhưng gây mất dịch vụ (Downtime).
- **Rolling:** Chia máy thành nhiều nhóm (buckets). Cập nhật từng nhóm một. Ứng dụng chạy song song bản cũ/mới, hoạt động dưới công suất trong lúc cập nhật.
- **Rolling with additional batches:** Tạo thêm nhóm mới dự phòng để cập nhật, đảm bảo hệ thống không bị hụt công suất so với bình thường.
- **Immutable:** Tạo một ASG hoàn toàn mới, thử nghiệm xong sẽ thay thế cái cũ. Không mất dịch vụ (Zero downtime), an toàn nhất, nhưng chạy lâu và tốn gấp đôi chi phí tạm thời.
- **Blue/Green:** Tạo 1 Environment hoàn toàn mới, test thử, sau đó swap URL (đổi tên miền DNS qua Route 53) sang môi trường mới. Rất phổ biến.
- **Traffic Splitting (Canary):** Đưa trước 10% lượng truy cập vào phiên bản mới để test lỗi. Nếu tốt thì mới chuyển hẳn.

## 6. Containers: ECS, ECR & Fargate
Container hóa ứng dụng giúp chạy ổn định trên mọi môi trường.
- **ECR (Elastic Container Registry):** Nơi lưu trữ Docker Image của AWS. Bảo mật qua IAM.
- **ECS (Elastic Container Service):** Dịch vụ quản lý dàn xếp Container (Orchestration) của AWS.
- **Fargate:** Giải pháp **Serverless** để chạy container. Bạn không cần thiết lập máy ảo EC2 nào, chỉ khai báo cấu hình CPU/RAM container cần, AWS lo mọi thứ hạ tầng phía dưới. Cực kỳ tiện lợi.

### Cấu trúc và khái niệm ECS
- **Task Definition (Định nghĩa tác vụ):** Khai báo Docker Image, CPU/RAM, biến môi trường, port, IAM Role cho container (tối đa 10 container / 1 task).
- **ECS IAM Roles (Mẹo thi):** Phải phân biệt rõ 3 loại quyền:
  - *EC2 Instance Profile:* Quyền cho máy ảo EC2 cấp dưới hoạt động.
  - *ECS Task Execution Role:* Quyền để công cụ của ECS có thể kéo (pull) Docker Image từ ECR về và đẩy log lên CloudWatch.
  - *ECS Task Role:* Quyền ĐÍCH THỰC của ứng dụng nằm bên trong Container (VD: để ứng dụng gọi API lấy file từ S3, đọc DynamoDB). Tuân thủ nguyên tắc đặc quyền tối thiểu.
- **Data Volumes (Gắn kết ổ đĩa):** Dễ dàng gắn EFS vào cả EC2 hoặc Fargate để chia sẻ dữ liệu liên tục cho các container trên nhiều AZ.
- **Load Balancing (Cân bằng tải):** Dynamic Host Port Mapping. Nếu dùng EC2 launch type, ALB sẽ tự động tìm kiếm cổng động đang chạy trên EC2 để chuyển truy vấn vào container tương ứng.

### Auto Scaling & Task Placement trong ECS
- ECS Auto Scaling dùng chung cơ chế Application Auto Scaling (Tracking theo CPU, RAM hoặc số truy vấn tới đích).
- **Task Placement Strategies (Chiến lược sắp xếp - chỉ áp dụng với ECS dạng EC2):**
  - **Binpack:** Sắp xếp nhồi nhét sao cho sử dụng ít số lượng EC2 nhất có thể (tiết kiệm chi phí).
  - **Spread:** Phân bổ đều các container ra các vùng AZ khác nhau (tối đa hoá độ tin cậy).
  - **Random:** Xếp ngẫu nhiên.
  - Có thể ràng buộc (Constraints) như `distinctInstance` (không xếp chung 2 container lên cùng 1 máy chủ).

### AWS Copilot
- Công cụ giao diện dòng lệnh (CLI tool) để nhanh chóng xây dựng và ra mắt ứng dụng container (trên ECS, Fargate, AppRunner) chỉ qua vài thao tác, tự động thiết lập CICD và hạ tầng (VPC, ALB, ECR). Không phải là một dịch vụ AWS riêng biệt.

### EKS (Elastic Kubernetes Service)
- Dịch vụ quản lý cụm Kubernetes (mã nguồn mở) được lưu trữ trên AWS. Có thể chạy worker node bằng EC2 hoặc Fargate. Phù hợp cho công ty muốn dùng Kubernetes để tránh bị khóa vào hệ sinh thái AWS (cloud-agnostic).

## 💡 Trích xuất trọng tâm luyện thi DVA-C02
- **Lưu trữ mật khẩu DB cho ứng dụng ECS:** Sử dụng AWS Secrets Manager. Để ECS tự động truyền mật khẩu này thành biến môi trường cho container, bạn khai báo trong thuộc tính `secrets` của Task Definition và CHẮC CHẮN rằng quyền đọc secret được cấp cho **Task Execution Role** (không phải Task Role).
- **Lỗi Auto Scaling không thay máy hỏng:** Nếu ALB báo một máy EC2 bị unhealthy (hỏng ứng dụng) nhưng ASG không chịu xóa máy đó đi, nguyên nhân là ASG đang dùng loại Health Check mặc định (EC2). Bạn phải đổi cấu hình Health Check Type của ASG sang **ELB**.
