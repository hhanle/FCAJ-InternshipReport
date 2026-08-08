---
title: "Giới thiệu"
date: 2026-08-05
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Mục tiêu của workshop

Kotaemon là ứng dụng RAG mã nguồn mở cho phép người dùng tải tài liệu lên, lập chỉ mục, đặt câu hỏi và nhận câu trả lời có trích dẫn. Dự án này không xây dựng lại thuật toán RAG cốt lõi; phần đóng góp tập trung vào cấu hình Gemini, đóng gói Docker, triển khai trên AWS, thiết kế lưu trữ bền vững, sao lưu, giám sát, bảo mật và quy trình vận hành.

Vấn đề cần giải quyết là một ứng dụng chỉ chạy trên máy cá nhân khó chia sẻ và dữ liệu trong lớp ghi của container có thể mất khi container bị xóa hoặc tạo lại. Giải pháp của workshop sử dụng một EC2 để chạy ứng dụng, EBS để giữ dữ liệu hoạt động và S3 để giữ bản sao độc lập.

## Luồng xử lý RAG

Hệ thống xử lý tài liệu theo hai giai đoạn:

1. **Lập chỉ mục:** đọc PDF hoặc URL, chuẩn hóa nội dung, chia đoạn, tạo embedding bằng Gemini và lưu vector cùng siêu dữ liệu.
2. **Hỏi đáp:** nhận câu hỏi, truy xuất các đoạn liên quan bằng hybrid search, ghép ngữ cảnh và gửi đến mô hình Gemini chat để sinh câu trả lời kèm nguồn.

Theo cấu hình được ghi nhận trong báo cáo, tài liệu được chia thành đoạn 1.024 token với phần chồng lấn 256 token; Chroma lưu embedding, LanceDB lưu tài liệu và SQLite lưu cấu hình/siêu dữ liệu. Truy xuất mặc định lấy 10 đoạn và chưa bật reranker chuyên dụng.

## Phạm vi triển khai

|Trong phạm vi|Ngoài phạm vi hiện tại|
|-|-|
|Một EC2 chạy Docker và Kotaemon|Cụm nhiều máy hoặc Auto Scaling|
|EBS lưu `ktem_app_data`|EFS hoặc cơ sở dữ liệu được quản lý|
|S3 lưu bản sao riêng tư|Sao lưu tự động và kiểm thử phục hồi định kỳ|
|CloudWatch metrics cơ bản và CPU alarm|RAM, dung lượng đĩa, CloudWatch Agent và log tập trung|
|HTTP cổng 80 cho demo|HTTPS, tên miền, WAF và ALB|
|Gemini API cho chat và embedding|Amazon Bedrock trong luồng hiện tại|

## Tiêu chí hoàn thành

- Giao diện Kotaemon truy cập được từ trình duyệt.
- Container chạy ổn định và tự khởi động lại với chính sách `unless-stopped`.
- Dữ liệu còn nguyên sau khi restart hoặc recreate container.
- EC2 sao lưu được dữ liệu lên S3 bằng IAM Role, không dùng Access Key dài hạn.
- CloudWatch hiển thị metrics và alarm; AWS Budgets theo dõi ngân sách.
- Có quy trình khởi động, kiểm tra, khôi phục, xử lý lỗi và dọn dẹp.


