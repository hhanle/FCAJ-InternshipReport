---
title: "Event 2"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 4.2. </b> "
---

# Bài thu hoạch “FCAJ x Agentic AI Build Week: Show Up. Build. Pitch. WIN!"

### Mục Đích Của Sự Kiện

- Tiếp cận quy trình xây dựng các sản phẩm ứng dụng AI thực tiễn trong thời gian ngắn.
- Học hỏi kinh nghiệm xử lý vấn đề, quản lý tài nguyên và làm việc nhóm từ các đội thi hackathon.
- Khám phá cách kết hợp AI với các hệ thống backend, cloud và giao diện người dùng để giải quyết bài toán thực tế.
- Lắng nghe những chia sẻ định hướng công nghệ trong ngành.

### Danh Sách Diễn Giả

- **One Team** - Quán quân với giải pháp AI Chatbot đặt thức ăn đa kênh
- **Signal Scout** - Á quân với nền tảng phân tích chiến lược doanh nghiệp
- **Plan V** - Trợ lý AI tạo kiến trúc hệ thống tự động cho Solution Architect
- **3KA** - Hệ thống AI Camera điều phối và giám sát đám đông
- **Six Pillars** - Hệ thống Multi-agent phòng chống rửa tiền (AML) cho ngân hàng

### Nội Dung Nổi Bật

#### Đặt Món Thông Minh Đa Kênh (One Team)

- Giải pháp giải quyết trực tiếp rào cản thao tác của khách hàng khi phải tải app hoặc điền thông tin phức tạp. 
- Chatbot đa kênh (Zalo, WhatsApp) giữ người dùng ở lại môi trường hội thoại quen thuộc, mang lại trải nghiệm liền mạch. 
- Hệ thống sử dụng Agent Core để ghi nhớ ngữ cảnh, kết hợp Web Scraping thu thập thực đơn, giúp giảm thiểu đáng kể chi phí hạ tầng.

#### Phân Tích Chiến Lược Doanh Nghiệp (Signal Scout)

- Dự án xây dựng nền tảng thu thập và tổng hợp dữ liệu rải rác để đánh giá chiến lược đối thủ. 
- Hệ thống kết hợp các Crawling Agent, Data Processing lọc nhiễu chống hallucination và Dashboard React trên AWS Amplify để hiển thị trực quan các dự báo ROI.

#### AI Generative Assistant Cho Kiến Trúc Cloud (Plan V)

- Công cụ tự động hóa quy trình thiết kế hạ tầng dành cho các kỹ sư kiến trúc hệ thống (SA). 
- Chỉ với mô tả bằng ngôn ngữ tự nhiên, AI có thể tự động vẽ sơ đồ kiến trúc, báo giá và sinh mã hạ tầng Terraform để triển khai thẳng lên AWS.

#### Các Giải Pháp Giám Sát Và Bảo Mật (3KA & Six Pillars)

- **3KA**: Tích hợp Computer Vision với AWS Fargate theo dõi dòng người theo thời gian thực, tự động cảnh báo khu vực có nguy cơ ùn tắc tại sân bay hoặc sự kiện.
- **Six Pillars**: Ứng dụng Adaptive Workflow Engine với 3 AI Agent (kiểm tra hồ sơ, dòng tiền, danh sách cấm vận) để lọc các cảnh báo rửa tiền sai lệch, tiết kiệm hàng giờ phân tích thủ công cho ngân hàng.


### Những Gì Học Được

#### Tư Duy Trải Nghiệm Người Dùng (UX/UI)

- Bài học lớn nhất đến từ giải pháp của One Team: công nghệ AI phức tạp phía sau phải phục vụ mục tiêu cuối cùng là giảm bớt rào cản cho người dùng cuối. 
- Việc không bắt khách hàng tải app riêng mà đặt món trực tiếp qua Zalo/WhatsApp là một minh chứng xuất sắc cho tư duy thiết kế lấy người dùng làm trung tâm.

#### Kiến Trúc Phần Mềm Và Cloud

- Sự kiện cho thấy sức mạnh của việc kết hợp các kiến trúc phân tán (như microservices) với nền tảng AWS. 
- Cách các đội sử dụng API Gateway, Lambda, Fargate và Dashboard React cho thấy một workflow hoàn chỉnh từ backend xử lý AI đến giao diện frontend hiển thị dữ liệu trực quan.

#### Kỹ Năng Quản Lý Dự Án Ngắn Hạn

- Các đội thi đã chứng minh rằng trong một khoảng thời gian giới hạn, việc khoanh vùng phạm vi dự án (scope limit) là tối quan trọng. 
- Xây dựng một sản phẩm cốt lõi chạy được (MVP) mang lại giá trị thuyết phục hơn nhiều so với một ý tưởng vĩ mô nhưng không thể hoạt động thực tế.

### Ứng Dụng Vào Công Việc

- **Phát Triển Frontend:** Vận dụng React và React Native để xây dựng các giao diện hiển thị dữ liệu AI (dashboard) đa nền tảng, mượt mà và trực quan như cách các đội ứng dụng AWS Amplify.
- **Kiến Trúc Microservices:** Áp dụng tư duy chia nhỏ các tác vụ thành các agent độc lập, kết nối qua API Gateway để tối ưu hiệu năng cho các hệ thống phức tạp quy mô lớn.
- **Tối Ưu Trải Nghiệm:** Ứng dụng các công cụ AI vào quy trình phân tích và thiết kế UX/UI trên Figma, đồng thời tích hợp AI vào mã nguồn để tự động hóa các bài kiểm thử phần mềm (automated testing).
- **Lên Kế Hoạch Kỹ Thuật:** Sử dụng các công cụ sơ đồ kỹ thuật như PlantUML để mô hình hóa hệ thống và luồng dữ liệu trước khi viết code, học hỏi từ quy trình chặt chẽ của hệ thống multi-agent.

### Trải nghiệm trong event

- Giai đoạn năm tư đại học đòi hỏi những góc nhìn thực tế về thị trường, và sự kiện hackathon này đã đáp ứng trọn vẹn điều đó. 
- Việc trực tiếp quan sát các đội trình bày demo đã kết nối những lý thuyết về phát triển phần mềm, tích hợp API và thiết kế giao diện thành những sản phẩm có tính thương mại hóa cao. 
- Không khí cạnh tranh, cường độ làm việc cao và tinh thần xử lý lỗi trực tiếp trên sân khấu mang lại nguồn cảm hứng rất lớn để tiếp tục đào sâu vào chuyên môn phát triển sản phẩm.

#### Bài học rút ra
- Cốt lõi của một sản phẩm công nghệ thành công là khả năng giải quyết đúng "nỗi đau" (pain point) của thị trường, chứ không phải việc nhồi nhét những công nghệ phức tạp nhất.
- Kiểm soát tài nguyên đầu vào, từ dữ liệu, token AI đến giới hạn phạm vi tính năng là kỹ năng bắt buộc để dự án đi đến đích.
- Giao diện người dùng mượt mà và trực quan chính là cầu nối quan trọng nhất để đưa các giải pháp AI phức tạp tiếp cận đại chúng.
- Kỹ năng làm việc nhóm, nhượng bộ và hướng tới mục tiêu chung quyết định sự thành bại trong các dự án công nghệ tốc độ cao.

#### Một số hình ảnh khi tham gia sự kiện
![Event2](/images/event2.jpg)
> Tổng thể, AGENTIC AI BUILD WEEK là buổi workshop có ý nghĩa thực tiễn. Qua việc phân tích các sản phẩm từ AI Chatbot đến nền tảng giám sát tài chính, tôi nhận thấy rõ bức tranh toàn cảnh về cách xây dựng một kiến trúc hệ thống hiện đại: linh hoạt ở backend, thông minh với AI và tối ưu tuyệt đối ở giao diện frontend. Đây là bước đệm tuyệt vời để hoàn thiện tư duy phát triển sản phẩm công nghệ toàn diện.
