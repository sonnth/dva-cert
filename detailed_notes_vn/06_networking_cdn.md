# Phần 6: Mạng lưới (Networking) và Phân phối nội dung (CDN)

## 1. Amazon VPC (Virtual Private Cloud)
Mạng đám mây riêng ảo. Nền tảng cốt lõi bao bọc mọi tài nguyên trong AWS để tạo ra một môi trường mạng tách biệt, an toàn.

### Kiến trúc cơ bản
- **VPC & Subnets:** Chia VPC thành các mạng con (Subnets) nằm trong các Vùng khả dụng (AZ) khác nhau.
- **Public Subnet:** Mạng con có gắn *Internet Gateway (IGW)*, tài nguyên trong đây có IP Public và ra vào Internet bình thường.
- **Private Subnet:** Mạng con KHÔNG gắn IGW. Máy chủ ở đây tuyệt đối an toàn với Internet. Để máy chủ tải các bản cập nhật từ mạng, phải dùng *NAT Gateway* (đặt ở Public Subnet) làm người phiên dịch trung gian.

### Bảo mật và Theo dõi Mạng
- **Network ACL (NACL):** Tường lửa mức độ Mạng con (Subnet). Khác với Security Group, NACL có cả luật CHO PHÉP (ALLOW) và TỪ CHỐI (DENY).
- **Security Groups (Tường lửa):** Hoạt động ở mức độ Máy chủ (ENI/EC2). Chỉ có luật CHO PHÉP.
- **VPC Flow Logs:** "Máy ghi hình" ghi lại toàn bộ IP và Port đã truy cập vào mạng VPC của bạn. Cực kỳ quan trọng để điều tra lỗi mạng bị chặn (chặn bởi SG hay chặn bởi NACL). Dữ liệu có thể đẩy vào S3 hoặc CloudWatch.

### Kết nối Mạng
- **VPC Peering:** Kết nối 2 VPC (cùng/khác tài khoản hoặc Region) với nhau qua hạ tầng cáp quang riêng của AWS. Không có tính chất bắc cầu (A kết nối B, B kết nối C không có nghĩa là A tự kết nối được C).
- **VPC Endpoints:** Cổng kết nối siêu tốc nội bộ. Giúp EC2 trong mạng Private có thể truy cập thẳng vào S3 hay DynamoDB bằng đường truyền riêng của AWS (không cần đi qua Internet). Rất an toàn và độ trễ thấp.
- **Site-to-Site VPN & Direct Connect (DX):** Kết nối mạng nội bộ công ty với AWS VPC. VPN đi qua mạng Internet (mã hóa), DX đi qua đường cáp quang kéo vật lý (đắt, cực nhanh, mất 1 tháng để thiết lập).

## 2. Amazon Route 53 (Dịch vụ DNS)
Dịch vụ Quản lý Tên miền (DNS) toàn cầu, tính sẵn sàng 100% SLA. Biến tên miền con người dễ đọc (`google.com`) thành địa chỉ IP.

### Bản ghi (Records)
- **A Record:** Trỏ tên miền tới địa chỉ IPv4.
- **AAAA Record:** Trỏ tới địa chỉ IPv6.
- **CNAME:** Trỏ tên miền tới một Tên miền khác (VD: `app.domain.com` -> `anything.com`). KHÔNG THỂ trỏ cho tên miền gốc (Zone Apex - `domain.com`).
- **Alias (Tính năng độc quyền):** Trỏ bất kỳ tên miền nào (Kể cả gốc hay phụ) tới thẳng các tài nguyên AWS (Load Balancer, CloudFront, S3 Website). Miễn phí truy vấn, tự động cập nhật IP nếu tài nguyên AWS thay đổi IP. *Lưu ý: Không dùng Alias để trỏ tới EC2 DNS name.*
- **TTL (Thời gian sống):** Quyết định bao lâu thì máy khách (Client) hỏi lại DNS (Cache). TTL thấp (60s) giúp đổi IP nhanh nhưng tốn phí hỏi Route 53 nhiều. Bí danh (Alias) không phải tốn tiền khai báo TTL.

### Chính sách định tuyến (Routing Policies)
Quyết định cách Route 53 trả lời IP cho người dùng:
- **Simple:** Trả về IP tĩnh. Không hỗ trợ Health check.
- **Weighted (Trọng số):** Chia tỷ lệ % lượng người dùng. VD: 70% tới máy chủ cũ, 30% tới bản test mới (Canary/A-B Testing).
- **Latency-based (Độ trễ):** Đo tốc độ mạng, tự động trỏ người dùng tới khu vực (Region) máy chủ có tốc độ kết nối nhanh nhất.
- **Failover (Dự phòng):** Có 1 máy chính, 1 máy phụ. Kết hợp với *Health Checks*, khi máy chính chết sẽ tự động trỏ tên miền sang máy phụ.
- **Geolocation (Vị trí địa lý):** Ép người dùng ở Châu lục/Quốc gia cụ thể phải vào một máy chủ nhất định (Ví dụ phục vụ Luật bản quyền Châu Âu).
- **Geoproximity:** Dựa trên vị trí người dùng và vị trí máy chủ để tính toán, có thể thêm "Bias" (Mở rộng/Thu hẹp vòng ảnh hưởng) để kéo người dùng sang máy chủ xa hơn chút xíu nhằm san sẻ bớt tải. 
- **Multi-Value:** Trả về nhiều IP ngẫu nhiên cùng lúc (Có kiểm tra sức khỏe Health check) để san tải nhỏ lẻ ở mức DNS (không thay thế được ALB).

### Health Checks (Kiểm tra sức khỏe)
Hệ thống mạng lưới bot của AWS trên toàn cầu liên tục ping vào máy chủ của bạn để xem nó có sống không. Cực kỳ quan trọng để tích hợp với Định tuyến Failover.

## 3. Amazon CloudFront (CDN)
Mạng phân phối nội dung. Bộ nhớ đệm (Cache) khổng lồ đặt tại hàng trăm trạm trung chuyển (Edge Locations) khắp thế giới. Giúp tải ảnh, video, dữ liệu web cực nhanh.

### Origins (Nguồn dữ liệu gốc)
- **S3 Bucket:** Tuyệt vời để phục vụ ảnh, web tĩnh. Dùng *Origin Access Control (OAC)* để khóa S3 lại, chỉ cho phép CloudFront lấy file. (Tốt hơn S3 Cross-Region Replication ở chỗ CDN bao phủ toàn cầu và chỉ dùng để đọc).
- **HTTP/ALB/EC2:** Phục vụ nội dung động (API).

### Caching (Bộ đệm) & Policies
- **Cache Key:** Quyết định một file có được cache lại hay không. Mặc định dựa vào URL. Bạn có thể thêm Header, Cookies, Query String vào cấu hình để tạo Cache Key phức tạp.
- **Origin Request Policy:** Chọn những Headers/Cookies muốn đưa vào cho Backend (Máy chủ gốc) đọc, nhưng không tính vào việc sinh Cache.
- Tối đa hóa tỷ lệ Cache (Cache Hit) bằng cách tách riêng các đường dẫn (Behavior). Ví dụ: `/*` để động không lưu cache, `/images/*` thì lưu cache 1 năm.
- **Cache Invalidation:** Lệnh ép CloudFront xóa ngay lập tức file cũ trong bộ nhớ đệm để tải file mới từ gốc lên (Rất tốn kém).

### Bảo mật & Truy cập riêng tư
- Tích hợp với **AWS WAF (Tường lửa ứng dụng web)** để chống hack, chống Bot. Tích hợp **AWS Shield** chống DDoS toàn cầu.
- **Geo Restriction:** Chặn không cho người dùng từ một số quốc gia truy cập nội dung.
- **Signed URLs / Signed Cookies:** (URL Ký trước). Bán nội dung trả tiền (Khóa học, Video, Phim). 
  - Tạo URL có hạn sử dụng (1 phút đến vài tháng), ép IP hoặc ngày tháng được quyền xem.
  - Sử dụng *Trusted Key Group*.
  - Khác biệt lớn với *S3 Pre-signed URL* là: CloudFront Signed URL có tận dụng tính năng CDN Cache, và cho phép sử dụng đa dạng mọi loại máy chủ gốc, không chỉ S3.

## 💡 Trích xuất trọng tâm luyện thi DVA-C02
- **Lỗi Cache CloudFront sau khi Deploy S3:** Khi dùng CodePipeline/CodeBuild deploy web tĩnh lên S3, nếu người dùng vẫn thấy giao diện cũ do CloudFront cache, bắt buộc phải cấu hình `buildspec.yml` gọi lệnh AWS CLI để xóa cache (**Create Invalidation**) ngay sau bước deploy.
