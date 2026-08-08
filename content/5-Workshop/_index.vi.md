---
title: "Workshop"
date: 2026-08-05
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# TRIỂN KHAI HỆ THỐNG KOTAEMON RAG CHAT TRÊN AWS

Workshop này hướng dẫn đưa ứng dụng RAG Chat dựa trên Kotaemon từ máy cá nhân lên AWS theo kiến trúc một máy chủ, ưu tiên khả năng trình diễn, dữ liệu bền vững và chi phí thấp. Ứng dụng chạy trong Docker trên Amazon EC2; dữ liệu được tách khỏi container bằng Amazon EBS; bản sao lưu được lưu trong Amazon S3; IAM, Security Group, Amazon CloudWatch và AWS Budgets hỗ trợ bảo mật, giám sát và kiểm soát chi phí.

Sau khi hoàn thành, bạn có thể:

- Chạy Kotaemon với Gemini cho mô hình chat và embedding ở local và trên EC2.
- Duy trì PDF, SQLite, Chroma, LanceDB và chỉ mục sau khi container được khởi động lại hoặc tạo lại.
- Sao lưu dữ liệu từ EBS lên S3 và kiểm thử khôi phục vào một thư mục độc lập.
- Theo dõi EC2, xử lý các lỗi thường gặp và dọn dẹp tài nguyên có kiểm soát.


#### Nội dung

1. [Giới thiệu](5.1-introduction/)
2. [Kiến trúc](5.2-architecture/)
3. [Điều kiện tiên quyết và chạy local](5.3-prerequisites-local/)
4. [Kiểm soát chi phí và khởi tạo EC2](5.4-cost-control-ec2/)
5. [Triển khai và lưu dữ liệu bền vững](5.5-deployment-persistence/)
6. [Sao lưu và khôi phục](5.6-backup-restore/)
7. [Giám sát và bảo mật](5.7-monitoring-security/)
8. [Vận hành và xử lý sự cố](5.8-operations-troubleshooting/)
9. [Chi phí, dọn dẹp và hướng phát triển](5.9-cost-cleanup-roadmap/)
