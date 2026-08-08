---
title: "Kiểm soát chi phí và khởi tạo EC2"
date: 2026-08-05
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Chọn Region và tạo ngân sách

Sử dụng Region `ap-southeast-1` (Singapore) cho toàn bộ tài nguyên. Trước khi tạo EC2, vào **Billing and Cost Management → Budgets → Create budget**, chọn **Cost budget** và thiết lập:

- Chu kỳ: Monthly.
- Ngân sách: **15 USD** theo cấu hình có bằng chứng trong báo cáo.
- Cảnh báo 80%: 12 USD.
- Cảnh báo 100%: 15 USD.
- Email: địa chỉ được theo dõi trong suốt workshop.

{{% notice warning %}}
AWS Budgets là cơ chế cảnh báo, không phải giới hạn cứng và không tự động dừng EC2. Dữ liệu billing có thể có độ trễ; vẫn phải Stop instance khi không sử dụng.
{{% /notice %}}

## Khởi tạo EC2

Mở **EC2 → Instances → Launch instances** và cấu hình:

|Thuộc tính|Giá trị đề xuất|
|-|-|
|Name|`fcaj-rag-chat`|
|AMI|Ubuntu Server 24.04 LTS|
|Instance type|`t3.medium` (2 vCPU, 4 GiB RAM)|
|Key pair|Tạo mới hoặc chọn key pair đang quản lý an toàn|
|VPC/Subnet|Default VPC, public subnet|
|Public IPv4|Enable cho demo|
|EBS|gp3, 30–60 GiB tùy kích thước tài liệu và ngân sách|

Chờ instance ở trạng thái **Running** và **Status checks 2/2**. Ghi lại Instance ID, Availability Zone và Public IPv4 hiện tại; không công khai các giá trị này trong ảnh báo cáo.

## Cấu hình Security Group

Tạo Security Group `fcaj-rag-chat-sg` với các inbound rule:

|Type|Port|Source|Mục đích|
|-|-|-|-|
|SSH|22|Địa chỉ IP quản trị `/32`|Quản trị máy chủ|
|HTTP|80|`0.0.0.0/0` chỉ trong thời gian demo|Truy cập giao diện web|

Không cần mở trực tiếp cổng `7860`, vì Docker sẽ ánh xạ cổng 80 của máy chủ vào cổng này. Nếu chỉ nhóm nội bộ kiểm thử, hãy giới hạn cổng 80 theo các địa chỉ IP cần thiết thay vì mở toàn internet.

## Kết nối SSH

Trên PowerShell, giới hạn quyền đọc key pair rồi kết nối:

```powershell
ssh -i .\YOUR_KEY.pem ubuntu@YOUR_PUBLIC_IP
```

Nếu SSH timeout, kiểm tra lại Public IPv4 hiện tại, IP mạng của máy quản trị, inbound rule cổng 22 và Security Group đang gắn đúng instance.

## Cài môi trường máy chủ

Trên EC2, cập nhật package và cài Docker, Git, AWS CLI:

```bash
sudo apt-get update
sudo apt-get install -y docker.io git awscli
sudo systemctl enable --now docker
sudo usermod -aG docker ubuntu
```

Đăng xuất rồi SSH lại để quyền nhóm Docker có hiệu lực. Xác nhận:

```bash
docker --version
git --version
aws --version
sudo systemctl is-active docker
```

Kết quả cuối phần này là EC2 hoạt động, SSH chỉ cho phép từ IP quản trị, cổng HTTP được mở đúng phạm vi và môi trường Docker đã sẵn sàng.
