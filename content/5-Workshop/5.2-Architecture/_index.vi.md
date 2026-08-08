---
title: "Kiến trúc"
date: 2026-08-05
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Kiến trúc tổng thể

Hệ thống chạy tại Region `ap-southeast-1` (Singapore) trong default VPC. EC2 Ubuntu `t3.medium` nằm trong public subnet và nhận yêu cầu HTTP từ người dùng qua Public IPv4. Docker chuyển tiếp cổng `80` của máy chủ đến cổng `7860` của Kotaemon.

![Kiến trúc triển khai RAG Chat trên AWS](/images/2-Proposal/platform_architecture.png)

## Luồng truy cập và dữ liệu

|Luồng|Mô tả|
|-|-|
|Truy cập web|Browser → Public IPv4 → Security Group → EC2 cổng 80 → container cổng 7860|
|Dữ liệu hoạt động|Kotaemon → `/app/ktem_app_data` → bind mount → `/opt/fcaj/ktem_app_data` trên EBS|
|Sao lưu|EC2 → AWS CLI → IAM Role → S3 prefix `ktem_app_data/`|
|Giám sát|EC2 → CloudWatch metrics cơ bản → CloudWatch Alarm → người vận hành xem trên Console|
|Chi phí|Chi phí tài khoản → AWS Budgets → ngưỡng cảnh báo|
|Mô hình|Kotaemon trên EC2 → Gemini API bên ngoài AWS → embedding hoặc câu trả lời|

## Vai trò của các thành phần

|Thành phần|Vai trò|Trạng thái|
|-|-|-|
|Amazon EC2|Chạy Ubuntu, Docker và Kotaemon|Đã triển khai|
|Amazon EBS|Lưu PDF, SQLite, Chroma, LanceDB, cache và index|Đã triển khai|
|Amazon S3|Lưu bản sao độc lập trong bucket riêng tư|Đã triển khai|
|Amazon CloudWatch|Theo dõi CPU, network, status check và CPU alarm|Triển khai một phần|
|AWS IAM|Cấp quyền tạm thời từ EC2 đến S3|Đã triển khai|
|VPC và Security Group|Kiểm soát SSH cổng 22 và HTTP cổng 80|Đã triển khai|
|AWS Budgets|Theo dõi ngân sách tài khoản|Đã triển khai|
|Gemini API|Cung cấp mô hình chat và embedding|Dịch vụ ngoài AWS|

## Lựa chọn kiến trúc

Kiến trúc một EC2 được chọn vì dễ triển khai, dễ quan sát và phù hợp ngân sách demo. EBS thay cho EFS vì chỉ có một máy chủ. Image được build trực tiếp trên EC2 nên chưa cần ECR hoặc ECS Fargate. Public subnet loại bỏ nhu cầu NAT Gateway, và truy cập trực tiếp Public IPv4 loại bỏ chi phí ALB.

Đánh đổi của lựa chọn này là EC2 trở thành điểm lỗi đơn, địa chỉ IP có thể đổi sau Stop/Start, kết nối HTTP không được mã hóa và hệ thống không tự mở rộng. Những giới hạn này cần được giữ rõ trong báo cáo và được xử lý trong lộ trình phát triển nếu chuyển sang production.

## Nguyên tắc bảo mật

- Không lưu AWS Access Key/Secret Key dài hạn trên EC2; sử dụng IAM Role.
- Giới hạn SSH theo địa chỉ quản trị `/32`.
- Bật S3 Block Public Access và chỉ cấp quyền đúng bucket sao lưu.
- Không commit `.env` hoặc `GEMINI_API_KEY` vào Git.
- Chỉ mở HTTP công khai trong thời gian demo và đổi mật khẩu mặc định trước khi mở cổng.
