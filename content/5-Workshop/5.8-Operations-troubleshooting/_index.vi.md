---
title: "Vận hành và xử lý sự cố"
date: 2026-08-05
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Quy trình khởi động trước demo

Thực hiện tuần tự và chỉ chuyển bước khi kết quả đạt yêu cầu:

|Bước|Thao tác|Kết quả mong đợi|
|-|-|-|
|1|Chọn Region Singapore và Start EC2|`Running`, status checks `2/2`|
|2|Lấy Public IPv4 hiện tại|Có địa chỉ mới sau Stop/Start|
|3|Kiểm tra Security Group|SSH 22 và HTTP 80 đúng source|
|4|SSH vào EC2|Có shell Ubuntu|
|5|Kiểm tra Docker service|Trạng thái `active`|
|6|Kiểm tra container|`Up`, ánh xạ `80 → 7860`|
|7|Kiểm tra mount và log|Đúng đường dẫn EBS, không có lỗi nghiêm trọng|
|8|Gọi ứng dụng từ EC2|Nhận phản hồi HTTP|
|9|Mở giao diện từ trình duyệt|Kotaemon hiển thị|
|10|Kiểm tra PDF và một câu hỏi|Dữ liệu cũ và citation hoạt động|

## Bộ lệnh kiểm tra nhanh

```bash
sudo systemctl is-active docker
docker start fcaj-rag-chat 2>/dev/null || true
docker ps
docker inspect fcaj-rag-chat \
  --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
docker logs --tail 100 fcaj-rag-chat
curl -I http://127.0.0.1
df -h /opt/fcaj
free -h
aws s3 ls s3://YOUR_BACKUP_BUCKET/ktem_app_data/ \
  --recursive --summarize --region ap-southeast-1
```

Trên máy quản trị Windows, kiểm tra cổng trước khi mở trình duyệt:

```powershell
Test-NetConnection YOUR_PUBLIC_IP -Port 80
```

## Bảng xử lý lỗi nhanh

|Hiện tượng|Nguyên nhân thường gặp|Cách kiểm tra và xử lý|
|-|-|-|
|SSH timeout|Public IP đổi hoặc rule 22 sai source|Lấy IP mới; kiểm tra IP mạng quản trị và Security Group|
|Web timeout|Cổng 80 đóng hoặc container chưa chạy|`Test-NetConnection`, `docker ps`, `curl` trên EC2|
|Container `Exited`|Thiếu RAM, lỗi `.env`, model hoặc dependency|`docker logs`, `free -h`, kiểm tra biến môi trường|
|Giao diện trả lỗi sau khi start|Ứng dụng còn khởi tạo|Theo dõi log và đợi readiness thay vì restart liên tục|
|Mất PDF/index|Bind mount sai hoặc thư mục host rỗng|`docker inspect`; kiểm tra `/opt/fcaj/ktem_app_data`|
|S3 `AccessDenied`|Role chưa gắn hoặc policy thiếu quyền|`aws sts get-caller-identity`; kiểm tra instance profile/policy|
|Gemini 401/403|API key sai, hết hạn hoặc không được phép|Kiểm tra `.env`, model name và luân phiên key|
|Gemini 429|Vượt quota|Giảm tần suất, dùng PDF nhỏ, chờ quota và chuẩn bị minh chứng dự phòng|
|CPU cao kéo dài|Build/index PDF lớn hoặc nhiều truy vấn|Xem CloudWatch, log, giảm kích thước tài liệu hoặc Stop tác vụ|
|Alarm không gửi email|Alarm chưa có action hoặc SNS chưa xác nhận|Kiểm tra tab Actions và subscription SNS|

## Quy trình cập nhật ứng dụng

1. Sao lưu dữ liệu lên S3 và xác nhận object.
2. Lưu tên image đang chạy để rollback.
3. Pull mã nguồn và build image với tag mới, ví dụ `v2`.
4. Dừng container cũ nhưng giữ nguyên thư mục EBS.
5. Chạy container mới với đúng `.env`, port và bind mount.
6. Kiểm tra log, giao diện, PDF và một truy vấn.
7. Nếu lỗi, chạy lại image cũ với cùng bind mount.

Không dùng tag `latest` làm bằng chứng duy nhất; tag phiên bản giúp xác định image đang chạy và rollback rõ ràng.

## Kiểm thử end-to-end

Một vòng kiểm thử hoàn chỉnh gồm: mở giao diện → tải PDF → lập chỉ mục → hỏi đáp → kiểm tra citation → restart container → kiểm tra dữ liệu → backup S3 → restore vào thư mục mới. Với mỗi bước, ghi thời gian, kết quả và bằng chứng đã che thông tin nhạy cảm.

Không ghi tỷ lệ chính xác về chất lượng RAG khi chưa có tập câu hỏi chuẩn. Nếu cần đánh giá, dùng 20–30 câu hỏi trên nhiều tài liệu và đo tỷ lệ truy xuất đúng, độ bám nguồn, citation, thời gian phản hồi và thời gian lập chỉ mục.
