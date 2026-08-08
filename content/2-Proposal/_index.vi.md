---
title: "Bản đề xuất"
date: 2026-06-22
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# TRIỂN KHAI HỆ THỐNG RAG CHAT KOTAEMON TRÊN AWS

### Thành viên nhóm
|MSSV|Tên thành viên|
|-|-|
|3122411020|Đàm Thị Ngọc Châu|
|3122411049|Lê Gia Hân|
|3122411141|Phan Thị Hồng Nhiên|
|3122411173|Võ Hoàng Kim Quyên|
|3122411243|Phan Thị Hải Vân|

### 1. Tóm tắt điều hành  
FCAJ RAG CHAT là ứng dụng hỏi đáp tài liệu (Retrieval-Augmented Generation) dựa trên mã nguồn mở Kotaemon, đã được chạy thành công ở môi trường local bằng Docker, sử dụng Gemini API cho cả mô hình chat và mô hình embedding.

Mục tiêu của dự án là triển khai ứng dụng này lên AWS Cloud dưới dạng container hoá, có lưu trữ dữ liệu bền vững, cấu hình bảo mật cơ bản, giám sát hoạt động, kiểm soát chi phí và quy trình dọn dẹp tài nguyên rõ ràng sau khi demo đáp ứng yêu cầu tối thiểu về số lượng dịch vụ AWS của FCAJ.

Kiến trúc triển khai được đơn giản hoá so với mô hình container hoá đầy đủ (không dùng ECS Fargate, Application Load Balancer, EFS, NAT Gateway) nhằm giảm chi phí và độ phức tạp, phù hợp với quy mô workshop ngắn hạn: EC2 chạy container Docker của Kotaemon, EBS lưu trữ dữ liệu bền vững, S3 lưu backup và minh chứng, CloudWatch giám sát, IAM kiểm soát quyền, AWS Budgets cảnh báo chi phí.  

### 2. Tuyên bố vấn đề  
#### Vấn đề hiện tại
- Ứng dụng chỉ chạy trên máy cá nhân, không thể truy cập từ xa đến demo hoặc chia sẻ.
- Dữ liệu ứng dụng có nguy cơ mất khi container Docker bị restart hoặc rebuild nếu không có lưu trữ bền vững tách rời.
- Chưa có cơ chế giám sát hoạt động (CPU, network, tinh trạng máy chủ) và cảnh báo khi có sự cố.
- Rủi ro bảo mật nếu khoá Gemini API bị lộ hoặc commit nhầm lên GitHub.
- Chưa có cơ chế cảnh báo chi phí, dễ phát sinh chi phí AWS ngoài dự kiến.

#### Giải pháp
- Container hoá và triển khai ứng dụng lên EC2 để truy cập được qua địa chỉ IP công khai.
- Gắn Amazon EBS làm nơi lưu trữ bền vững, mount vào thư mục dữ liệu ứng dụng.
- Sao lưu định kỳ dữ liệu sang Amazon S3.
- Thiết lập CloudWatch để giám sát và cảnh báo (CPU cao, status check thất bại).
- Dùng IAM Role và/hoặc SSM Parameter Store để bảo vệ khoá API và quyền truy cập, tránh hard-code trong mã nguồn.
- Cấu hình AWS Budgets để cảnh báo khi chi phí vượt ngưỡng đề ra  

#### Lợi ích
- Ứng dụng truy cập được từ xa.
- Dữ liệu bền vững qua các lần restart hoặc rebuild container.
- Có khả năng giám sát hoạt động và kiểm soát chi phí chủ động.
- Tuân thủ các thực hành bảo mật cơ bản trên AWS (IAM, security group, quản lý secret).  

### 3. Kiến trúc giải pháp  
Hệ thống sử dụng kiến trúc đơn giản hoá trên một EC2 instance, tách riêng phần lưu trữ bền vững (EBS) và lưu trữ backup/tài sản (S3), có giám sát và kiểm soát chi phí đi kèm.

![RAG CHAT](/images/2-Proposal/platform_architecture.png)

#### Luồng hoạt động chính
- Users → Browser → EC2 (qua HTTP/public IP)
- EC2 ↔ EBS (lưu trữ bền vững), EC2 → S3 (backup), EC2 → CloudWatch (metrics), EC2 → Gemini API (chat/embedding, IAM Role cấp quyền)
- CloudWatch → CloudWatch Alarm → Dev/Admin (cảnh báo sự cố)
- AWS Budgets → Email alert → Dev/Admin (cảnh báo chi phí)

#### Dịch vụ AWS sử dụng
|**Dịch vụ**|**Vai trò**|  
|-------------|-----------|
|**Amazon EC2**|Chạy container Docker của FCAJ RAG Chat|
|**Amazon EBS**|Lưu ktem_app_data: SQLite DB, vector index, file upload|
|**Amazon S3**|Lưu backup, tài liệu mẫu, ảnh chụp minh chứng báo cáo|
|**CloudWatch Metrics**|Theo dõi CPU, network, status check của EC2|
|**CloudWatch Alarms**|Cảnh báo khi CPU cao hoặc status check thất bại|
|**AWS Budgets**|Gửi email cảnh báo khi chi phí vượt ngưỡng đặt ra|
|**IAM**|IAM Role FCAJ-EC2-S3-Backup-Role gắn cho EC2, cấp quyền truy cập S3/CloudWatch, tránh access key cứng|
|**VPC / Subnet / Security Group**|Kiểm soát truy cập SSH và cổng ứng dụng|
|**SSM Parameter Store**|Lưu an toàn Gemini API key / cấu hình|
|**CloudWatch Logs**|Log ứng dụng / Docker, retention 7 ngày|


#### Thiết kế mạng
- Sử dụng VPC mặc định (không tạo NAT Gateway để tránh chi phí theo giờ không cần thiết).
- Security Group cho phép SSH chỉ từ IP của sinh viên.
- Security Group cho phép cổng ứng dụng 80 (public, HTTP — chưa cấu hình SSL/HTTPS) từ internet, map vào cổng 7860 của container Docker để phục vụ demo công khai.
- EC2 chạy trong subnet công khai (public subnet), dùng IP công khai của EC2 thay vì Load Balancer. 

*Giới hạn đã biết*
- Ứng dụng hiện phục vụ qua HTTP thuần, chưa cấu hình chứng chỉ SSL/HTTPS (do không dùng ALB + ACM trong kiến trúc đơn giản hoá).
- CloudWatch Alarm hiện chưa kết nối với Amazon SNS Topic, nên chỉ hiển thị trạng thái cảnh báo trên Console AWS, chưa tự động gửi email cho Dev/Admin — có thể bổ sung SNS Topic ở giai đoạn sau nếu cần.
- CloudWatch mặc định (EC2 basic monitoring) chỉ theo dõi CPU, network và status check; muốn theo dõi dung lượng ổ đĩa/bộ nhớ bên trong Ubuntu cần cài thêm CloudWatch Agent (tuỳ chọn)

### 4. Triển khai kỹ thuật  
Quá trình triển khai được chia thành các giai đoạn từ baseline local đến báo cáo và dọn dẹp tài nguyên:
* a. Baseline local: xác nhận Docker, mô hình chat/embedding Gemini và luồng upload–hỏi đáp PDF hoạt động ổn định (đã hoàn thành).
* b. Thiết lập tài khoản AWS & an toàn chi phí: chọn region, tạo cảnh báo Budget, xác nhận không dùng các dịch vụ đắt tiền.
* c. Dựng hạ tầng EC2: khởi tạo instance Ubuntu, cấu hình Security Group, key pair, EBS.
* d. Cài đặt môi trường & triển khai ứng dụng: cài Docker/Git/AWS CLI, clone repo, build image và chạy container trên EC2.
* e. Lưu trữ bền vững & backup: mount EBS vào thư mục dữ liệu ứng dụng, kiểm thử persistence, backup định kỳ sang S3.
* f. Giám sát, bảo mật, kiểm thử & tối ưu: cấu hình CloudWatch, IAM, kiểm thử end-to-end, dọn dẹp tài nguyên và viết báo cáo workshop.

### 5. Lộ trình & Mốc triển khai  
|STT|Giai đoạn|Thời gian|
|-|-|-|
|1|Baseline local|0.5–1 ngày|
|2|Thiết lập AWS & an toàn chi phí|0.5 ngày|
|3|Dựng EC2 instance|0.5–1 ngày|
|4|Cấu hình Security Group|0.5 ngày|
|5|Cài đặt môi trường server|0.5–1 ngày|
|6|Triển khai ứng dụng lên EC2|1 ngày|
|7|Lưu trữ bền vững với EBS|0.5–1 ngày|
|8|S3 lưu trữ tài sản & backup|0.5 ngày|
|9|Backup thủ công sang S3|0.5 ngày|
|10|Giám sát với CloudWatch|0.5–1 ngày|
|11|Rà soát bảo mật & cấu hình|0.5 ngày|
|12|Kiểm thử end-to-end trên AWS|1 ngày|
|13|Tối ưu chi phí & dọn dẹp|0.5 ngày|
|14|Viết báo cáo workshop|2–4 ngày|

### 6. Ước tính ngân sách  
* Cấu hình đề xuất: EC2 t3.medium (2 vCPU / 4 GB RAM) để đảm bảo ổn định khi index PDF, chi phí dự kiến thực tế khoảng **15.35 USD/tháng** cho demo ~120 giờ/tháng. 
* AWS Budgets được cấu hình cảnh báo qua email khi chi phí chạm ngưỡng 20 USD (có biên an toàn so với chi phí dự kiến). Các khoản mục khác (S3, CloudWatch metrics cơ bản, Gemini API free tier) có chi phí không đáng kể hoặc bằng 0.

### 7. Đánh giá rủi ro  
|Rủi ro|Mức độ ảnh hưởng|
|-|-|
|Mất dữ liệu sau khi rebuild/restart container|Cao|
|Lộ khoá Gemini API lên GitHub|Cao|
|Phát sinh chi phí AWS ngoài dự kiến|Cao|
|Bỏ sót bước dọn dẹp sau demo|Cao|
|EC2 thiếu RAM / container dừng đột ngột|Trung bình|
|IP công khai đổi khi restart / Security Group sai|Trung bình|
|Model Gemini không hỗ trợ / vượt quota API|Trung bình|
|EBS bị xoá nhầm / S3 bucket bị public|Trung bình|
|Thiếu log CloudWatch khi bị lỗi|Thấp|
|Xử lý PDF dung lượng lớn bị chậm|Trung bình|
|Môi trường Local và EC2 khác biệt|Trung bình|

### 8. Kết quả kỳ vọng  
*Kỹ thuật*
- Triển khai thành công ứng dụng RAG trên AWS với kiến trúc container hoá, đơn giản và tiết kiệm chi phí.
- Đảm bảo dữ liệu bền vững qua các lần restart/rebuild bằng EBS, có backup sang S3.
- Thiết lập giám sát cơ bản (CloudWatch) và kiểm soát chi phí (AWS Budgets).
- Áp dụng thực hành bảo mật cơ bản: IAM, Security Group, quản lý secret không hard-code.

*Giá trị*
- Nâng cao kỹ năng triển khai và vận hành hệ thống trên AWS.
- Hiểu rõ cách đơn giản hoá kiến trúc cloud để phù hợp với ngân sách và quy mô demo/workshop.
- Có tài liệu và minh chứng đầy đủ phục vụ báo cáo workshop FCAJ, có thể mở rộng thành giải pháp triển khai đầy đủ hơn (ECS/ALB/EFS) trong tương lai nếu cần.