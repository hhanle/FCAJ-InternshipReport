---
title: "Blog 1"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# EC2 tưởng đơn giản mà không đơn giản và vài điều mình học được khi làm lab AWS

Xin chào mọi người! 

Trong quá trình làm các bài lab thực hành trên AWS, EC2 chắc chắn là dịch vụ mà bất kỳ ai học Cloud cũng phải đụng tới đầu tiên. Nghe thì đơn giản như chỉ là "thuê một cái máy ảo", nhưng càng làm nhiều lab, mình càng nhận ra có vài điều dễ hiểu lầm nếu không để ý kỹ. Hôm nay mình xin chia sẻ lại vài điều mình rút ra được.

### 1. Stop khác với Terminate
Ban đầu mình cứ nghĩ tắt máy trong hệ điều hành là xong, nhưng thực ra instance vẫn tồn tại ở trạng thái **Stopped**, và EBS volume đính kèm vẫn có thể phát sinh chi phí lưu trữ dù không còn tính phí compute nữa. Chỉ khi **Terminate** thì instance mới bị xoá hẳn (và volume mặc định cũng bị xoá theo, trừ khi bạn cấu hình giữ lại).

Đây là điểm mà người mới rất dễ nhầm, vì **Stop** tạo cảm giác an toàn tuyệt đối, trong khi thực tế vẫn có thể phát sinh phí nhỏ nếu không dọn dẹp volume.

### 2. Free Tier tính theo tổng số giờ, không phải theo từng instance
AWS Free Tier cho **750 giờ EC2 mỗi tháng**, nhưng đây là tổng cộng cho tất cả instance thuộc loại được hỗ trợ, cộng dồn lại, chứ không phải mỗi instance được cấp riêng 750 giờ. Nếu trong lúc làm lab bạn chạy song song nhiều instance để test, hoặc quên tắt sau khi dùng xong, số giờ miễn phí sẽ hao hụt nhanh hơn nhiều so với tưởng tượng ban đầu.

### 3. Tự động hoá việc tắt instance thay vì tin vào trí nhớ
Một cách mình thấy khá hay ho khi tìm hiểu thêm là kết hợp **Amazon EventBridge** và **AWS Lambda** để tự động dừng instance vào một khung giờ cố định mỗi ngày (ví dụ 23h đêm), thay vì phải nhớ tắt tay sau mỗi buổi lab. Với các bạn mới học và hay quên như mình, đây là cách giảm rủi ro phát sinh chi phí ngoài ý muốn khá hiệu quả.

### Vài điều nên làm sau mỗi buổi lab
* ✅ Kiểm tra lại EC2 Dashboard xem còn instance nào đang **Running** không
* ✅ Đặt tên rõ ràng cho instance (kèm ngày tạo) để dễ nhận biết cái nào cần xoá
* ✅ Cân nhắc tự động hoá việc stop bằng **Lambda** + **EventBridge** cho các instance chỉ dùng để test ngắn hạn
* ✅ Bật **Billing Alerts** để được cảnh báo sớm nếu chi phí vượt ngưỡng mong muốn

![Blog 1](/images/blog1.jpg)

Nói chung, EC2 là nền tảng quan trọng để hiểu về AWS, nhưng cũng vì "quen thuộc" mà nhiều bạn mới (trong đó có mình) dễ chủ quan bỏ qua vài chi tiết nhỏ. Hy vọng những điều trên hữu ích cho các bạn đang trong quá trình học Cloud như mình!