---
title: "Giám sát và bảo mật"
date: 2026-08-05
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

## Giám sát EC2 bằng CloudWatch

EC2 basic monitoring cung cấp các chỉ số như `CPUUtilization`, `NetworkIn`, `NetworkOut` và `StatusCheckFailed`. Mở **EC2 → Instances → Monitoring** hoặc **CloudWatch → Metrics → EC2 → Per-Instance Metrics**, chọn đúng Instance ID và quan sát tải trong lúc build image, lập chỉ mục PDF và đặt câu hỏi.

Các chỉ số này giúp trả lời:

- CPU có duy trì ở mức cao trong thời gian dài hay không.
- Network có lưu lượng khi tải dependency, gọi Gemini hoặc sao lưu S3 hay không.
- EC2 status check có thất bại do sự cố hệ thống hoặc instance hay không.

Basic monitoring không cung cấp RAM, dung lượng filesystem hoặc application log. Muốn có các dữ liệu đó cần cài CloudWatch Agent hoặc một giải pháp log khác; phần này chưa thuộc bản triển khai hiện tại.

## Tạo CloudWatch Alarm

Tạo alarm cho `CPUUtilization` của instance:

1. Chọn metric `CPUUtilization` của đúng Instance ID.
2. Statistic: **Average**; Period: **5 minutes**.
3. Threshold: **Greater than 70%** trong khoảng đánh giá phù hợp.
4. Đặt tên, ví dụ `fcaj-rag-chat-high-cpu`.
5. Kiểm tra trạng thái `OK`, `In alarm` hoặc `Insufficient data` sau khi tạo.


Nên bổ sung một alarm cho `StatusCheckFailed >= 1`. Khi có hành động SNS, thực hiện kiểm thử thông báo và lưu bằng chứng email đã nhận thay vì chỉ chụp màn hình cấu hình.

## Rà soát IAM và S3

- EC2 phải sử dụng IAM Role thay vì Access Key/Secret Key dài hạn.
- Role chỉ có `ListBucket`, `GetObject` và `PutObject` trên bucket/prefix cần thiết.
- Bucket bật Block Public Access; không có bucket policy hoặc ACL cấp quyền công khai.
- Kiểm tra danh tính thực tế bằng `aws sts get-caller-identity`.
- Không cấp `s3:*` hoặc quyền trên `*` nếu quy trình chỉ cần một bucket.

## Bảo vệ secret

Giữ Gemini API key trong `.env` với quyền file `600`, loại file khỏi Git và không chép key vào Docker image. Trước khi chia sẻ kho mã nguồn, chạy:

```bash
git check-ignore .env
git log --all -- .env
git grep -n -i 'gemini_api_key' -- ':!*.md'
```

Nếu phát hiện key từng được commit, xóa key khỏi nơi sử dụng, luân phiên key và xử lý lịch sử Git theo quy trình của nhóm. Trong giai đoạn sau có thể chuyển secret sang SSM Parameter Store; không mô tả dịch vụ này là đã triển khai khi chưa có bằng chứng.

## Checklist bảo mật trước demo

- [ ] SSH cổng 22 chỉ cho phép IP quản trị `/32`.
- [ ] Cổng 7860 không mở trực tiếp.
- [ ] Mật khẩu mặc định của Kotaemon đã được đổi.
- [ ] `.env`, API key và thông tin đăng nhập không xuất hiện trong Git hoặc ảnh.
- [ ] S3 Block Public Access đang bật.
- [ ] IAM Role gắn đúng EC2 và policy giới hạn đúng bucket.
- [ ] Account ID, ARN, IP, Instance ID và username được che trong tài liệu công khai.
- [ ] HTTP công khai chỉ mở trong thời gian cần thiết.
