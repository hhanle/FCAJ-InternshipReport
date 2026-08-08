---
title: "Blog 2"
date: 2026-06-22
weight: 1
chapter: false
pre: " <b> 3.2. </b> "
---

# "Dính" lỗi HTTP 429 khi gọi Gemini API trên AWS – và cách nhóm mình fix

Xin chào mọi người 👋

Trong lúc làm capstone project (chatbot RAG chạy trên EC2, gọi tới Gemini API), tụi mình gặp một lỗi mà chắc ai làm về tích hợp API bên thứ ba cũng từng "ăn" ít nhất một lần: HTTP 429 – Too Many Requests.

Lúc mới thấy lỗi này, tụi mình khá bối rối, vì trên máy local mọi thứ chạy ngon lành, không hề có dấu hiệu gì bất thường. Nhưng vừa deploy lên EC2 để test với nhiều người dùng cùng lúc thì y như rằng, requests dồn dập và server bên Gemini "từ chối khéo" bằng lỗi 429.

### 🤔 Tại sao local chạy ngon mà lên EC2 lại "toang"?

Ngồi phân tích lại thì tụi mình nhận ra vấn đề không nằm ở EC2, mà nằm ở việc tải tăng đột biến:
- **Ở local**: chỉ có 1 mình mình test, may ra vài request/phút → API nào chịu nổi cũng dư sức.
- **Trên EC2**: nhiều người cùng dùng, nhiều tiến trình worker chạy song song → request bắn ra dồn dập trong thời gian ngắn → chạm ngay ngưỡng rate limit → 429 xuất hiện.

Nói vui là: một mình mình xài thì API "chiều" hết mình, nhưng cả nhóm cùng xài một lúc thì nó "giận dỗi" ngay 😄

### ❗ Hệ lụy nếu không xử lý lỗi này

Ban đầu tụi mình chủ quan, nghĩ 429 chỉ là lỗi vặt, retry vài lần là xong. Nhưng đọc kỹ lại thì mới thấy nếu không có cơ chế xử lý đàng hoàng, nó có thể kéo theo một loạt vấn đề khác:
- Tác vụ bị dở dang giữa chừng: đang đọc file → gọi API → ghi vào database, mà bị đứt ngay giữa bước gọi API thì dữ liệu dễ bị thiếu hoặc sai lệch.
- Nếu cứ để lỗi này "rớt" tự do mà không bắt lại, nó có thể làm hỏng cả luồng xử lý của worker đang chạy trên EC2.
- Người dùng cuối thì chỉ thấy một lỗi 500 chung chung, chẳng biết đường nào mà lần.

### 🛠️ Cách tụi mình xử lý

Thay vì chỉ vá tạm ở một chỗ, tụi mình quyết định thiết kế lại cả luồng gọi API theo hướng "chịu lỗi tốt hơn" (resilient), gồm mấy phần chính sau:

#### 1️⃣ Retry với Exponential Backoff + Jitter
Thay vì gọi lại ngay lập tức (dễ làm tình trạng nghẽn thêm nặng), tụi mình cho hệ thống chờ lâu dần sau mỗi lần thất bại:
- Thất bại lần 1: chờ **2 giây** rồi thử lại.
- Thất bại lần 2: chờ **4 giây**.
- Thất bại lần 3: chờ **8 giây**.

Và có thêm một chút "jitter" – tức là cộng thêm một khoảng thời gian ngẫu nhiên nho nhỏ (tầm ±0.5 giây) – để tránh việc tất cả worker cùng retry đúng một thời điểm, gây nghẽn lần nữa.

#### 2️⃣ Gom nhỏ lại: Batching và Chunking

Để đỡ chạm ngưỡng giới hạn quá nhanh, tụi mình:
- Gom nhiều mục nhỏ vào chung một request nếu API cho phép, giảm tổng số lần gọi.
- Chia nhỏ dữ liệu lớn thành từng phần vừa phải, thay vì nhồi hết vào một request to đùng.
Ngoài ra cũng dọn bớt các trường dữ liệu dư thừa trong payload cho gọn.

#### 3️⃣ Có "phương án B": chuyển sang Amazon Bedrock

Nếu Gemini API cứ liên tục lỗi (do sập tạm thời hoặc do hết quota), hệ thống sẽ tự động chuyển hướng gọi sang Amazon Bedrock – một dịch vụ AI managed của AWS.

Lưu ý nhỏ mà tụi mình rút ra: chuyển sang Bedrock không có nghĩa là hết lo về quota nhé, Bedrock cũng có giới hạn riêng theo tài khoản, khu vực và mức sử dụng token. Nhưng ít nhất mình không còn phụ thuộc hoàn toàn vào một nhà cung cấp duy nhất.

#### 4️⃣ Theo dõi bằng CloudWatch cho yên tâm

Tụi mình đẩy hết log lỗi (429, timeout...) từ EC2 về **CloudWatch Logs**, sau đó tạo **Metric Filter** để đếm số lần lỗi xuất hiện. 

Nếu số lỗi vượt ngưỡng trong một khoảng thời gian, **CloudWatch Alarm** sẽ tự bắn thông báo qua SNS về email của nhóm – khỏi phải ngồi canh log thủ công nữa.

### 💡 Vài điều tụi mình rút ra được
- Đừng bao giờ nghĩ **chạy ngon ở local** nghĩa là **chạy ngon ở production**. Khi tải và số lượng người dùng tăng lên, những vấn đề mà local không bao giờ gặp sẽ lộ ra hết.
- Retry không phải "cây đũa thần" – chỉ nên dùng cho lỗi tạm thời. Nếu lỗi kéo dài, retry mãi chỉ tổ tốn tài nguyên và làm hệ thống chậm thêm thôi.
- Thao tác được retry thì nên thiết kế **idempotent** (gọi lại nhiều lần vẫn ra kết quả đúng), không thì dữ liệu dễ loạn.
- Luôn có **phương án B** cho những dịch vụ bên ngoài mà mình không kiểm soát được.

Đây là lần đầu nhóm mình thật sự ngồi thiết kế lại một luồng xử lý theo hướng **resilient** thay vì chỉ code cho chạy được, nên cũng học được khá nhiều. Không biết mọi người trong group đã từng gặp lỗi rate limit khi tích hợp API bên thứ ba trên AWS chưa? Xử lý sao vậy, chia sẻ cho tụi mình học hỏi với! 🙌

![Blog 2](/images/blog2.jpg)

### 📚 Tài liệu chúng mình tham khảo:
- Exponential Backoff and Jitter – AWS Architecture Blog: https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
- REL05-BP03 Control and limit retry calls – AWS Well-Architected Framework: https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/rel_mitigate_interaction_failure_limit_retries.html
- Retry behavior – AWS SDKs and Tools Reference Guide: https://docs.aws.amazon.com/sdkref/latest/guide/feature-retry-behavior.html
- Identity-based policy examples for Amazon Bedrock (IAM permissions, bedrock:InvokeModel): https://docs.aws.amazon.com/.../security_iam_id-based
- InvokeModel API Reference – Amazon Bedrock: https://docs.aws.amazon.com/bedrock/latest/APIReference/API_runtime_InvokeModel.html
- Alarming on logs – Amazon CloudWatch User Guide: https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/Alarm-On-Logs.html
- Creating a metric filter for a log group – Amazon CloudWatch Logs: https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CreateMetricFilterProcedure.html

