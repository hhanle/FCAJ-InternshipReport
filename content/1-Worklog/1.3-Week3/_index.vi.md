---
title: "Worklog Tuần 3"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 1.3. </b> "
---

### Mục tiêu tuần 3:

* Nắm vững cách thiết lập và quản lý kết nối Site-to-Site VPN với strongSwan và AWS Transit Gateway.
* Hiểu và triển khai mô hình Hybrid DNS bằng Route 53 Resolver để phân giải tên miền giữa môi trường On-premises và AWS.
* Làm chủ các mô hình kết nối giữa nhiều VPC thông qua VPC Peering và AWS Transit Gateway.
* Hệ thống hóa kiến thức về các dịch vụ Compute trên AWS, bao gồm EC2, AMI, EBS, Instance Store, User Data, Metadata và Auto Scaling.

### Các công việc trong tuần 3:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | **- Tiếp tục thực hành Lab 5:** <br>&emsp; + Thiết lập AWS Site-to-Site VPN: <br>&emsp; * Cấu hình VPN bằng strongSwan với Transit Gateway: Tạo Customer Gateway, Tạo Transit Gateway, Tạo kết nối VPN, Tạo Transit Gateway Attachment, Cấu hình Route Tables, Cấu hình Customer Gateway <br>&emsp; + Dọn dẹp Tài nguyên <br>&emsp; + Infrastructure as Code Templates| 06/07/2026   | 06/07/2026      | [Lab 5 Cấu hình VPN bằng strongSwan với Transit Gateway](https://000003.awsstudygroup.com/vi/5-vpnsitetosite/5.3-vpnsitetosite-optional/) <br> [Lab 5 Dọn dẹp Tài nguyên](https://000003.awsstudygroup.com/vi/6-cleanup/) <br> [Lab 5 Infrastructure as Code Templates](https://000003.awsstudygroup.com/vi/7-infrastructureascode/)|
| 3   | **- Thực hành Lab 6:** Quản lý DNS lai với Amazon Route 53 <br>&emsp; + Các bước chuẩn bị: Tạo Key Pair, Khởi tạo CloudFormation Template, Cấu hình Security Group <br>&emsp; + Kết nối đến RDGW <br>&emsp; + Triển khai Microsoft AD <br>&emsp; + Thiết lập DNS: Tạo Route 53 Outbound Endpoint, Tạo Route 53 Resolver Rules, Tạo Route 53 Inbound Endpoints, Thử nghiệm kết quả <br>&emsp; + Dọn dẹp tài nguyên | 07/07/2026   | 07/07/2026      | [Lab 6 Quản lý DNS lai với Amazon Route 53](https://000010.awsstudygroup.com/vi/) |
| 4   | **- Thực hành Lab 7:** Tích hợp mạng với VPC Peering <br>&emsp; + Các bước chuẩn bị: Khởi tạo CloudFormation Template, Tạo Security Group, Tạo EC2 instance <br>&emsp; + Cập nhật Network ACL <br>&emsp; + Tạo kết nối Peering <br>&emsp; + Kích hoạt Cross-Peer DNS <br>&emsp; + Dọn dẹp tài nguyên | 08/07/2026   | 08/07/2026      | [Lab 7 Tích hợp mạng với VPC Peering](https://000019.awsstudygroup.com/vi/) |
| 5   | **- Thực hành Lab 8:** Quản lý mạng tập trung với AWS Transit Gateway <br>&emsp; + Các bước chuẩn bị: Tạo Key Pair, Khởi tạo CloudFormation Template <br>&emsp; + Tạo Transit Gateway <br>&emsp; + Tạo Transit Gateway Attachments <br>&emsp; + Tạo Transit Gateway Route Tables <br>&emsp; + Thêm Transit Gateway Routes vào VPC Route Tables <br>&emsp; + Dọn dẹp tài nguyên | 09/07/2026   | 09/07/2026      | [Lab 8 Quản lý mạng tập trung với AWS Transit Gateway](https://000020.awsstudygroup.com/vi/) |
| 6   | - Tìm hiểu dịch vụ Compute trên AWS (Module 03): EC2, AMI, EBS, Instance Store, User Data, Metadata và Auto Scaling | 10/07/2026   | 10/07/2026      | [Youtube AWS Study Group](https://www.youtube.com/@AWSStudyGroup) |


### Kết quả đạt được tuần 3:

* Hoàn thành cấu hình AWS Site-to-Site VPN sử dụng strongSwan kết hợp với AWS Transit Gateway.
* Triển khai thành công mô hình Hybrid DNS với Amazon Route 53 Resolver, bao gồm Inbound Endpoint, Outbound Endpoint và Resolver Rules.
* Thiết lập và kiểm thử kết nối giữa các VPC thông qua VPC Peering và AWS Transit Gateway.
* Hiểu được cách quản lý định tuyến mạng bằng Route Tables, Transit Gateway Attachments và Cross-Peer DNS.
* Củng cố kiến thức về các dịch vụ Compute trên AWS, bao gồm EC2, AMI, EBS, Instance Store, User Data, Instance Metadata và Auto Scaling.


