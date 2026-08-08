---
title: "Blog 3"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 3.3. </b> "
---

# 🔐 Mình từng hardcode API Key ngay trong task definition ECS – đây là cách tụi mình sửa lại bằng AWS Systems Manager Parameter Store

Xin chào mọi người 👋

Trong lúc deploy con RAG chatbot (fork từ Kotaemon) lên ECS Fargate, tụi mình cần cấu hình Gemini API Key cho container backend. Bình thường tụi mình hay làm là: gõ thẳng API Key vào phần environment variables trong task definition, rồi deploy lên.

Chạy thì chạy được thật, mọi thứ hoạt động ngon lành. Nhưng đến lúc ngồi review lại trước khi nộp báo cáo, mình mới giật mình nhận ra một vấn đề khá nghiêm trọng.

#### 😬 Vấn đề: Key nằm "trần trụi" ở khắp nơi
Task definition trên ECS không phải là chỗ giấu bí mật tốt. Nó hiện rõ mồn một trong console, ai có quyền xem task definition là đọc được API Key ngay lập tức, kể cả những người chỉ cần quyền xem chứ không cần quyền chỉnh sửa gì cả. Chưa kể, mỗi lần cần đổi key (ví dụ key cũ bị lộ hoặc hết hạn), tụi mình lại phải sửa trực tiếp trong task definition rồi tạo revision mới, khá thủ công và dễ quên.

Ngồi nghĩ lại, mình thấy đây đúng kiểu "để cửa nhà mở toang nhưng khóa cổng thật kỹ" – tụi mình lo bảo mật ở tầng network (Security Group, ALB...) nhưng lại quên mất chính API Key mới là thứ nhạy cảm nhất.

#### 🛠️ Giải pháp: chuyển hết secrets qua AWS Systems Manager Parameter Store

Sau khi tìm hiểu, tụi mình quyết định dọn hết các giá trị nhạy cảm (Gemini API Key, một vài config nội bộ khác) ra khỏi task definition và lưu vào SSM Parameter Store dưới dạng SecureString – tức là được mã hoá bằng AWS KMS ngay khi lưu, chỉ giải mã lúc thật sự cần dùng.

Quy trình tụi mình làm gồm 3 bước chính:

**Tạo parameter kiểu SecureString**

Thay vì gõ giá trị thẳng vào task definition, tụi mình tạo một parameter riêng, đặt tên theo dạng phân cấp cho dễ quản lý, ví dụ /fcaj-rag-chat/prod/gemini-api-key, và chọn kiểu SecureString để AWS tự mã hoá bằng KMS key mặc định.

**Cấp quyền cho Task Execution Role**

Đây là phần tụi mình từng bị vướng khá lâu: Container không tự nhiên đọc được Parameter Store. Khi sử dụng mục secrets trong ECS task definition, ECS/Fargate agent sẽ dùng Task Execution Role để lấy giá trị của parameter trước khi container khởi động. Role này cần quyền ssm:GetParameters đối với đúng parameter. Nếu parameter được mã hóa bằng customer managed KMS key, role cần thêm quyền kms:Decrypt đối với KMS key đó. Nếu thiếu quyền cần thiết, task có thể dừng ngay trong giai đoạn khởi động và xuất hiện lỗi như ResourceInitializationError..

**Tham chiếu parameter trong task definition thay vì gõ giá trị thật**

Trong container definition, thay vì khai báo environment với giá trị thật, tụi mình khai báo ở mục secrets, trỏ tới ARN của parameter. Lúc container khởi động, ECS agent sẽ tự động lấy giá trị từ Parameter Store và inject vào container dưới dạng biến môi trường. Trong task definition lúc này chỉ còn lưu tên hoặc ARN của parameter, không còn lưu trực tiếp API Key dạng plaintext. Tuy nhiên, vì secret vẫn được đưa vào container dưới dạng biến môi trường, ứng dụng cần tránh in toàn bộ environment variables ra log và cần kiểm soát chặt quyền truy cập vào container.

#### ✅ Kết quả sau khi đổi
- Xem task definition trên console giờ chỉ thấy đường dẫn tới parameter, không còn thấy giá trị thật của API Key nữa.
- Muốn đổi key mới thì chỉ cần update giá trị trong Parameter Store rồi force new deployment cho service, không cần đụng vào task definition.
- An tâm hơn hẳn khi cần chia sẻ task definition hoặc file cấu hình cho thành viên khác trong nhóm để cùng review.

#### 💡 Vài điều tụi mình rút ra
- Đừng nghĩ "chạy được là xong" – bảo mật thường bị bỏ quên đúng ở những chỗ tưởng chừng nhỏ nhặt như biến môi trường.
- Task Execution Role và Task Role là hai role khác nhau, dễ nhầm lẫn lúc mới làm quen ECS – quyền đọc Parameter Store phải gắn đúng vào Task Execution Role.
- Với những secret cần quản lý version, vòng đời hoặc xây dựng cơ chế rotation định kỳ, AWS Secrets Manager thường phù hợp hơn. Tuy nhiên, khả năng rotation tự động còn phụ thuộc vào loại secret và trong một số trường hợp cần triển khai thêm Lambda rotation function. Với nhu cầu đơn giản như tụi mình, Parameter Store SecureString là đủ dùng.

Nhóm nào đang triển khai container trên ECS mà vẫn còn hardcode key/secret ngay trong task definition thì thử đổi qua cách này xem, khá nhẹ nhàng mà an tâm hơn hẳn. Có ai trong group từng gặp lỗi liên quan tới quyền đọc Parameter Store/Secrets Manager trên ECS chưa? Chia sẻ kinh nghiệm với tụi mình nhé! 🙌

![Blog 3](/images/blog3.png)

#### 📚 Tài liệu tham khảo:
- Pass sensitive data to an Amazon ECS container – Amazon ECS Developer Guide: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/specifying-sensitive-data.html
- Pass Systems Manager parameters through Amazon ECS environment variables – Amazon ECS Developer Guide: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/secrets-envvar-ssm-paramstore.html
- SecureString parameters – AWS Systems Manager User Guide: https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-paramstore-securestring.html
- AWS KMS encryption for SecureString parameters – AWS Systems Manager User Guide: https://docs.aws.amazon.com/systems-manager/latest/userguide/secure-string-parameter-kms-encryption.html
- Creating a Parameter Store parameter using the AWS CLI – AWS Systems Manager User Guide: https://docs.aws.amazon.com/systems-manager/latest/userguide/param-create-cli.html
= AWS Systems Manager Parameter Store overview: https://docs.aws.amazon.com/systems-manager/latest/userguide/systems-manager-parameter-store.html