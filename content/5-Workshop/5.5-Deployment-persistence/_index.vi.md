---
title: "Triển khai và lưu dữ liệu bền vững"
date: 2026-08-05
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Clone và cấu hình ứng dụng

Trên EC2, clone kho mã nguồn vào home directory:

```bash
cd ~
git clone REPOSITORY_URL fcaj-rag-chat
cd fcaj-rag-chat
```

Tạo `.env` theo mẫu của dự án và điền Gemini API key. Chỉ cấp quyền đọc/ghi cho chủ sở hữu:

```bash
chmod 600 .env
```

Kiểm tra `.env` đã được loại khỏi Git:

```bash
git check-ignore .env
git status --short
```

## Chuẩn bị dữ liệu trên EBS

Trong cấu hình một volume đơn giản, root volume của EC2 đã là EBS. Tạo thư mục chuyên dụng trên EBS và gán quyền cho user Ubuntu:

```bash
sudo mkdir -p /opt/fcaj/ktem_app_data
sudo chown -R ubuntu:ubuntu /opt/fcaj
df -h /opt/fcaj
```

Thư mục này sẽ chứa SQLite, Chroma, LanceDB, PDF tải lên, cache và index. Nó nằm ngoài lớp ghi của container nên không bị xóa khi container được recreate.

{{% notice note %}}
EBS bảo vệ dữ liệu khỏi vòng đời container, nhưng không tự bảo vệ dữ liệu khi volume bị xóa hoặc instance bị terminate cùng root volume. S3 backup ở mục 5.6 cung cấp lớp bảo vệ độc lập.
{{% /notice %}}

## Build image

Build image trực tiếp trên EC2:

```bash
docker build -t fcaj-rag-chat:v1 .
docker images fcaj-rag-chat
```

Nếu build bị dừng, dùng `free -h`, `df -h` và `docker system df` để kiểm tra RAM và dung lượng trước khi chạy lại.

## Chạy container

```bash
docker run -d \
  --name fcaj-rag-chat \
  --restart unless-stopped \
  --env-file ~/fcaj-rag-chat/.env \
  -p 80:7860 \
  -v /opt/fcaj/ktem_app_data:/app/ktem_app_data \
  fcaj-rag-chat:v1
```

Xác nhận container và bind mount:

```bash
docker ps
docker inspect fcaj-rag-chat \
  --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
docker logs --tail 100 fcaj-rag-chat
curl -I http://127.0.0.1
```

Kết quả mount phải hiển thị:

```text
/opt/fcaj/ktem_app_data -> /app/ktem_app_data
```

Mở `http://YOUR_PUBLIC_IP` trên trình duyệt. Ứng dụng có thể cần vài phút để tải thư viện và khởi tạo lần đầu.

## Kiểm thử dữ liệu bền vững

1. Tải một PDF lên, đợi lập chỉ mục và đặt một câu hỏi.
2. Ghi lại danh sách tài liệu và câu trả lời có citation.
3. Restart container bằng `docker restart fcaj-rag-chat`.
4. Mở lại ứng dụng và xác nhận PDF, index và truy vấn cũ vẫn hoạt động.
5. Để kiểm tra mạnh hơn, xóa **chỉ container**, sau đó chạy lại đúng lệnh `docker run` với bind mount cũ.

```bash
docker stop fcaj-rag-chat
docker rm fcaj-rag-chat
# Chạy lại lệnh docker run ở trên với cùng thư mục /opt/fcaj/ktem_app_data
```

{{% notice warning %}}
Không xóa `/opt/fcaj/ktem_app_data` trong kiểm thử. Trước khi recreate container, kiểm tra lại đường dẫn mount để tránh khởi động ứng dụng với một thư mục rỗng ngoài ý muốn.
{{% /notice %}}
