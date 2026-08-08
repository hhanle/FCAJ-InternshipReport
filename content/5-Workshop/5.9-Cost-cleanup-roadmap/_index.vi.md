---
title: "Chi phí, dọn dẹp và hướng phát triển"
date: 2026-08-05
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

## Ước tính chi phí

Chi phí phụ thuộc vào thời gian chạy EC2, dung lượng EBS, Public IPv4, S3 và cấu hình CloudWatch. Báo cáo nguồn đưa ra hai cách tính: dự toán ban đầu với EBS 60 GiB và 1 GB CloudWatch Logs, và bản đối chiếu với EBS khoảng 30 GiB, không tính Logs vì chưa triển khai.

|Kịch bản|Dự toán EBS 60 GiB + Logs|Đối chiếu EBS ~30 GiB, không Logs|Nhận xét|
|-|-|-|-|
|Demo 60 giờ/tháng|11,36 USD|Khoảng 7,48 USD|Dưới ngân sách 15 USD|
|Demo 120 giờ/tháng|15,35 USD|Khoảng 11,47 USD|Bản dự toán ban đầu vượt nhẹ ngân sách|
|Chạy 24/7|55,89 USD|Khoảng 52,00 USD|Không phù hợp ngân sách demo|

Các số liệu này là ước tính trong tài liệu dự án, không phải hóa đơn. Trước khi nộp báo cáo, cập nhật lại AWS Pricing Calculator và đối chiếu **Billing/Cost Explorer** vì giá và mức sử dụng thực tế có thể khác.

## Cách giảm chi phí

- Stop EC2 khi không build, kiểm thử hoặc demo.
- Chỉ chọn `t3.medium` trong giai đoạn cần đủ RAM; đo tải trước khi giảm cấu hình.
- Chọn dung lượng EBS theo dữ liệu thật và xóa snapshot thừa.
- Không tạo NAT Gateway, ALB, EFS, ECR hoặc CloudWatch Logs nếu chưa có nhu cầu và ngân sách.
- Giữ S3 lifecycle/retention rõ ràng, không lưu nhiều bản sao không cần thiết.
- Theo dõi budget 80% và 100%, nhưng không phụ thuộc hoàn toàn vào cảnh báo.

## Trình tự dọn dẹp

Chỉ dọn dẹp sau khi đã lưu bằng chứng và kiểm thử bản sao cần giữ:

1. Lưu ảnh cuối cùng đã che thông tin và xác nhận backup S3.
2. Stop EC2 nếu còn cần kiểm tra; terminate khi dự án đã kết thúc.
3. Giải phóng Elastic IP/Public IPv4 tĩnh không còn gắn tài nguyên.
4. Xác nhận backup rồi xóa EBS volume không còn dùng.
5. Xóa snapshot thử nghiệm hoặc snapshot thừa.
6. Xóa object tạm; empty và xóa bucket S3 nếu không cần lưu bản sao.
7. Xóa CloudWatch alarm, SNS topic/subscription và log group thử nghiệm nếu đã tạo.
8. Xóa Security Group tùy chỉnh sau khi không còn resource tham chiếu.
9. Gỡ IAM Role và policy không còn sử dụng.
10. Xóa `.env` khỏi máy bị hủy và luân phiên Gemini API key nếu cần.
11. Kiểm tra Billing và Cost Explorer sau dọn dẹp để tìm tài nguyên còn phát sinh phí.

{{% notice warning %}}
Không xóa EBS hoặc S3 trước khi kiểm tra bản sao và xác nhận dữ liệu không còn cần thiết. Ghi lại trạng thái từng mục; không đánh dấu cleanup hoàn thành chỉ vì EC2 đã được Stop.
{{% /notice %}}

## Đánh giá hiện trạng

|Khía cạnh|Đánh giá|Cơ sở|
|-|-|-|
|Phù hợp demo|Tốt|Một EC2 dễ triển khai và trình bày|
|Dữ liệu bền vững|Khá|Có EBS và backup S3; cần biên bản restore test|
|Bảo mật|Cơ bản|Có IAM Role, SG và S3 riêng tư; còn HTTP và `.env`|
|Khả năng quan sát|Cơ bản|Có metrics/alarm; thiếu action, RAM, disk và app log|
|Kiểm soát chi phí|Khá|Có budget và tránh dịch vụ đắt; cleanup cần hoàn tất|
|Khả năng mở rộng|Thấp|Một EC2 là điểm lỗi đơn|

## Hướng phát triển

|Ưu tiên|Hành động|Điều kiện hoàn thành|
|-|-|-|
|P0|Che dữ liệu nhạy cảm, đổi credential mặc định, luân phiên key|Không còn account ID, ARN, IP hoặc password trong bản công khai|
|P0|Hoàn tất persistence test và restore test|Có bằng chứng trước/sau restart và khôi phục vào thư mục mới|
|P0|Hoàn tất cleanup sau khi nộp|Billing không còn tài nguyên ngoài kế hoạch|
|P1|Bổ sung HTTPS|Trình duyệt truy cập bằng TLS hợp lệ|
|P1|Kết nối alarm với SNS|Nhận được thông báo kiểm thử|
|P1|Đánh giá RAG bằng bộ câu hỏi chuẩn|Có phương pháp và bảng kết quả|
|P2|Tự động hóa backup và retention|Có lịch, log, cảnh báo lỗi và restore test định kỳ|
|P2|Bổ sung CloudWatch Agent/log|Có RAM, disk, application log và retention|
|P3|Xem xét ALB, ECS/ECR và EFS|Chỉ triển khai khi có nhu cầu nhiều instance, SLA và ngân sách|

Kết luận của workshop là hệ thống đã đạt mức nguyên mẫu có thể trình diễn: ứng dụng chạy trên EC2, dữ liệu tách khỏi container bằng EBS, có backup S3, IAM Role, giám sát cơ bản và kiểm soát ngân sách. Các mục trong roadmap là điều kiện để tiến gần môi trường production, không phải thành phần đã triển khai.
