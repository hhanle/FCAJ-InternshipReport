---
title: "Worklog Tuần 2"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 1.2. </b> "
---

### Mục tiêu tuần 2:

* Thực hành quản lý truy cập người dùng an toàn với AWS IAM.
* Hiểu kiến trúc mạng Amazon VPC và các lớp bảo mật mạng.
* Khởi động dự án nhóm.

### Các công việc trong tuần 2:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | **- Thực hành Lab 4:** Quản trị quyền truy cập với AWS Identity and Access Management (IAM) <br>&emsp;  + Tạo IAM User và IAM Group <br>&emsp; + Tạo IAM Role <br>&emsp; + Chuyển đổi Role                                                                                         | 29/06/2026   | 29/06/2026      | [Lab 4](https://000002.awsstudygroup.com/vi/) |
| 3   | **- Thực hành Lab 5:** Bắt đầu với Amazon Virtual Private Cloud (VPC) và AWS Site-to-Site VPN <br>&emsp; + Giới thiệu Amazon VPC: Subnets, Route Table, Internet Gateway, NAT Gateway <br>&emsp; + Network Security với Security Groups và Network ACLs | 30/06/2026   | 30/06/2026      | [Lab 5 Giới thiệu](https://000003.awsstudygroup.com/vi/1-introduce/) <br> [Lab 5 Tường lửa trong VPC](https://000003.awsstudygroup.com/vi/2-firewallinvpc/) |
| 4   | **- Tiếp tục thực hành Lab 5:** <br>&emsp; + Chuẩn bị Môi trường: Tạo VPC, Tạo Subnet, Tạo Internet Gateway, Tạo Route Table, Tạo Security Group, Kích hoạt VPC Flow Logs <br>&emsp; + Triển khai Amazon EC2 Instance: Tạo EC2 Instances, Kiểm thử Phương thức Kết nối, Tạo Multi-AZ NAT Gateway, Sử dụng Reachability Analyzer, Cấu hình EIC Endpoint, AWS Systems Manager Session Manager, CloudWatch Monitoring & Alerting | 01/07/2026   | 01/07/2026      | [Lab 5 Các bước chuẩn bị](https://000003.awsstudygroup.com/vi/3-prerequisite/) <br> [Lab 5 Triển khai Amazon EC2 Instances](https://000003.awsstudygroup.com/vi/4-createec2server/) |
| 5   | **- Tiếp tục thực hành Lab 5:** <br>&emsp; + Thiết lập AWS Site-to-Site VPN: <br>&emsp; * Tạo môi trường VPN: Tạo VPC cho VPN, Tạo EC2 Instance <br>&emsp; * Cấu hình kết nối VPN: Tạo Virtual Private Gateway, Tạo Customer Gateway, Tạo kết nối VPN, Cấu hình Customer Gateway, Tùy chỉnh AWS VPN Tunnel, Cấu hình VPN Nâng cao, Hướng dẫn Troubleshooting VPN | 02/07/2026   | 02/07/2026      | [Lab 5 Tạo môi trường VPN](https://000003.awsstudygroup.com/vi/5-vpnsitetosite/5.1-createvpnenv/) <br> [Lab 5 Cấu hình kết nối VPN](https://000003.awsstudygroup.com/vi/5-vpnsitetosite/5.2-vpnsitetosite/) |
| 6   | - Team project: lên ý tưởng, phân tích, nghiên cứu để chọn lọc idea phù hợp | 03/07/2026   | 03/07/2026      | |


### Kết quả đạt được tuần 2:

* **Quản lý quyền truy cập AWS:** Nắm bắt được kinh nghiệm thực tế về bảo mật đám mây thông qua việc cấu hình thành công các IAM User, Group và Role, đảm bảo áp dụng tốt nguyên tắc quyền hạn tối thiểu.

* **Mạng & Bảo mật Đám mây:** Phát triển tư duy và kỹ năng thiết kế, cách ly các môi trường mạng đám mây sử dụng Amazon VPC; đồng thời áp dụng các biện pháp bảo mật chặt chẽ thông qua Security Groups và Network ACLs.

* **Điện toán & Kết nối Mạng:** Cấp phát và khởi tạo thành công các tài nguyên điện toán (Amazon EC2) cũng như thiết lập các kết nối mạng riêng tư, an toàn giữa các môi trường bằng AWS Site-to-Site VPN.

* **Lập kế hoạch & Phân tích Dự án:** Lên ý tưởng và nghiên cứu cho dự án nhóm, phân tích kỹ lưỡng các yêu cầu thực tế để chốt được hướng phát triển khả thi cho dự án.