---
title: "Điều kiện tiên quyết và chạy local"
date: 2026-08-05
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Điều kiện tiên quyết

Trước khi bắt đầu, cần chuẩn bị:

- Tài khoản AWS có quyền tạo EC2, EBS, S3, IAM Role, CloudWatch Alarm và AWS Budget.
- Máy tính có Git, Docker Engine hoặc Docker Desktop và ít nhất 4 GB RAM khả dụng.
- Gemini API key còn hiệu lực và có quota cho mô hình chat/embedding được chọn.
- Kho mã nguồn `fcaj-rag-chat` và một PDF nhỏ, không chứa dữ liệu nhạy cảm, để kiểm thử.
- Địa chỉ email có thể nhận cảnh báo AWS Budgets.


## Chuẩn bị mã nguồn

Clone kho mã nguồn và chuyển vào thư mục dự án:

```bash
git clone REPOSITORY_URL fcaj-rag-chat
cd fcaj-rag-chat
```

Tạo file `.env` theo mẫu của dự án và điền Gemini API key. Tên biến phải theo đúng mã nguồn; không tự đổi tên biến nếu ứng dụng đang đọc một tên cụ thể.

```dotenv
GEMINI_API_KEY=YOUR_GEMINI_API_KEY
```

Đảm bảo `.gitignore` chứa `.env`, sau đó kiểm tra file không nằm trong staging area:

```bash
grep -qxF '.env' .gitignore || echo '.env' >> .gitignore
git status --short
```

## Build và chạy local

Build image từ Dockerfile của dự án:

```bash
docker build -t fcaj-rag-chat:local .
```

Tạo thư mục dữ liệu ngoài container và chạy ứng dụng:

```bash
mkdir -p ./ktem_app_data
docker run -d \
  --name fcaj-rag-chat-local \
  --env-file .env \
  -p 7860:7860 \
  -v "$(pwd)/ktem_app_data:/app/ktem_app_data" \
  fcaj-rag-chat:local
```

Trên PowerShell, có thể thay `$(pwd)` bằng `${PWD}` nếu Docker Desktop không nhận cú pháp Bash. Mở `http://localhost:7860` sau khi container sẵn sàng.

## Kiểm thử baseline

1. Xác nhận container ở trạng thái `Up` bằng `docker ps`.
2. Xem log bằng `docker logs --tail 100 fcaj-rag-chat-local` và bảo đảm không có lỗi model hoặc API key.
3. Đăng nhập Kotaemon, kiểm tra mô hình chat và embedding Gemini.
4. Tải một PDF nhỏ lên và đợi quá trình lập chỉ mục hoàn tất.
5. Đặt ít nhất ba câu hỏi có đáp án trong tài liệu và kiểm tra nguồn trích dẫn.
6. Restart container, mở lại giao diện và xác nhận tài liệu vẫn còn.

```bash
docker restart fcaj-rag-chat-local
docker inspect fcaj-rag-chat-local \
  --format '{{range .Mounts}}{{println .Source "->" .Destination}}{{end}}'
```

## Bằng chứng cần lưu

- Kết quả `docker ps` và log khởi động thành công.
- Trang cấu hình model đã che API key.
- PDF đã được lập chỉ mục và một câu trả lời có citation.
- Kết quả kiểm tra mount và dữ liệu sau restart.

Chỉ chuyển sang AWS sau khi baseline local đạt. Việc này tách lỗi ứng dụng/model khỏi lỗi mạng, IAM hoặc cấu hình EC2.
