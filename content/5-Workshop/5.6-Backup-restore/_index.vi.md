---
title: "Sao lưu và khôi phục"
date: 2026-08-05
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Tạo bucket S3 riêng tư

Tại Region `ap-southeast-1`, tạo một bucket có tên duy nhất toàn cầu, ví dụ `fcaj-rag-chat-backup-UNIQUE_SUFFIX`. Giữ **Block all public access** ở trạng thái bật và sử dụng mã hóa mặc định SSE-S3. Không dùng bucket này để public website.

Tổ chức object theo prefix:

```text
s3://YOUR_BACKUP_BUCKET/
├── ktem_app_data/
└── evidence/
```

## Cấp quyền bằng IAM Role

Tạo IAM Role dành cho EC2, ví dụ `FCAJ-EC2-S3-Backup-Role`, và giới hạn policy vào đúng bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:ListBucket"],
      "Resource": "arn:aws:s3:::YOUR_BACKUP_BUCKET"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:PutObject"],
      "Resource": "arn:aws:s3:::YOUR_BACKUP_BUCKET/*"
    }
  ]
}
```

Gắn role vào EC2 qua **Actions → Security → Modify IAM role**. Trên máy chủ, kiểm tra danh tính và quyền:

```bash
aws sts get-caller-identity
aws s3 ls s3://YOUR_BACKUP_BUCKET --region ap-southeast-1
```

## Sao lưu thủ công

Để bản sao SQLite và index nhất quán, dừng container trước khi sync trong khoảng bảo trì ngắn:

```bash
docker stop fcaj-rag-chat
aws s3 sync /opt/fcaj/ktem_app_data/ \
  s3://YOUR_BACKUP_BUCKET/ktem_app_data/ \
  --region ap-southeast-1
docker start fcaj-rag-chat
```

Xác nhận số object và dung lượng:

```bash
aws s3 ls s3://YOUR_BACKUP_BUCKET/ktem_app_data/ \
  --recursive --summarize \
  --region ap-southeast-1
```

{{% notice warning %}}
Không thêm `--delete` vào lệnh sync thông thường. Tùy chọn đó có thể xóa object trên S3 khi file nguồn bị mất hoặc đường dẫn nguồn bị chọn sai.
{{% /notice %}}

## Kiểm thử khôi phục an toàn

Khôi phục vào một thư mục trống, không ghi đè dữ liệu đang vận hành:

```bash
sudo mkdir -p /opt/fcaj/restore_test
sudo chown ubuntu:ubuntu /opt/fcaj/restore_test
aws s3 sync s3://YOUR_BACKUP_BUCKET/ktem_app_data/ \
  /opt/fcaj/restore_test/ \
  --region ap-southeast-1
du -sh /opt/fcaj/ktem_app_data /opt/fcaj/restore_test
find /opt/fcaj/ktem_app_data -type f | wc -l
find /opt/fcaj/restore_test -type f | wc -l
```

Để xác nhận đầy đủ, dừng container và chạy một container kiểm thử với `/opt/fcaj/restore_test` được mount vào `/app/ktem_app_data`. Kiểm tra danh sách PDF, index và một câu hỏi đã dùng trước đó. Sau khi kiểm thử, dừng container kiểm thử và khởi động lại container chính.

## Quy trình khôi phục khi có sự cố

1. Dừng ứng dụng để không ghi thêm dữ liệu.
2. Đổi tên thư mục lỗi để giữ khả năng rollback.
3. Tạo thư mục `ktem_app_data` mới và sync dữ liệu từ S3.
4. Kiểm tra quyền file và mount bằng `docker inspect`.
5. Khởi động ứng dụng, kiểm tra log, PDF, index và truy vấn.
6. Chỉ xóa thư mục cũ sau khi bản khôi phục được xác nhận.

Lưu ảnh hoặc log của lần restore test. Báo cáo nguồn mới chỉ có bằng chứng backup; kiểm thử restore cần được thực hiện trước khi tuyên bố quy trình sao lưu hoàn chỉnh.
