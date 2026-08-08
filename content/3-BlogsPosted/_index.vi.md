---
title: "Các bài blogs đã đăng"
date: 2026-06-22
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Tại đây sẽ là phần liệt kê, giới thiệu các blogs mà các bạn đã đăng trên [AWS Study Group](https://www.facebook.com/groups/awsstudygroupfcj). Ví dụ:

###  [Blog 1 - EC2 tưởng đơn giản mà không đơn giản và vài điều mình học được khi làm lab AWS](3.1-blog1/)
Bài viết chia sẻ những kinh nghiệm thực tế mà tác giả rút ra trong quá trình thực hành với Amazon EC2 trên AWS. Nội dung tập trung làm rõ một số điểm mà người mới học Cloud thường dễ nhầm lẫn, như sự khác biệt giữa **Stop** và **Terminate**, cách hoạt động của **AWS Free Tier** đối với EC2, cũng như các phương pháp giúp hạn chế phát sinh chi phí ngoài ý muốn thông qua việc tự động hóa và quản lý tài nguyên. Bên cạnh đó, bài viết còn tổng hợp một số thói quen hữu ích sau mỗi buổi lab nhằm giúp người học sử dụng EC2 hiệu quả, an toàn và tiết kiệm hơn.

###  [Blog 2 - "Dính" lỗi HTTP 429 khi gọi Gemini API trên AWS – và cách nhóm mình fix](3.2-blog2/)
Bài viết chia sẻ kinh nghiệm thực tế của nhóm trong quá trình phát triển và triển khai chatbot RAG trên Amazon EC2 có tích hợp Gemini API, khi gặp phải lỗi **HTTP 429 (Too Many Requests)** do vượt quá giới hạn số lượng yêu cầu. Nội dung phân tích nguyên nhân khiến lỗi chỉ xuất hiện trong môi trường triển khai thực tế, những rủi ro nếu không xử lý đúng cách, đồng thời trình bày giải pháp theo hướng **resilient** thông qua cơ chế retry với exponential backoff và jitter, tối ưu số lượng yêu cầu bằng batching và chunking, sử dụng Amazon Bedrock làm phương án dự phòng, và giám sát hệ thống bằng Amazon CloudWatch. Qua đó, bài viết tổng kết những bài học kinh nghiệm trong việc xây dựng các ứng dụng Cloud có khả năng chịu lỗi tốt và vận hành ổn định khi tích hợp với các dịch vụ bên thứ ba.

###  [Blog 3 - 🔐 Mình từng hardcode API Key ngay trong task definition ECS – đây là cách tụi mình sửa lại bằng AWS Systems Manager Parameter Store](3.3-blog3/)
Bài viết chia sẻ kinh nghiệm thực tế của nhóm trong việc cải thiện bảo mật khi triển khai chatbot RAG trên Amazon ECS Fargate bằng cách thay thế việc hardcode Gemini API Key trong ECS Task Definition bằng **AWS Systems Manager Parameter Store**. Nội dung phân tích những rủi ro khi lưu trữ thông tin nhạy cảm trực tiếp trong biến môi trường của Task Definition, đồng thời trình bày quy trình chuyển sang sử dụng **SecureString**, cấu hình **Task Execution Role** để truy cập tham số an toàn, và tham chiếu secret thông qua ECS. Bên cạnh đó, bài viết cũng tổng kết những bài học kinh nghiệm về quản lý secrets trên AWS, sự khác biệt giữa Task Execution Role và Task Role, cũng như khi nào nên sử dụng Parameter Store hoặc AWS Secrets Manager để tăng cường tính bảo mật và khả năng quản lý thông tin nhạy cảm trong các ứng dụng Cloud.