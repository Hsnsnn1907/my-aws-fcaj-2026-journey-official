+++
title = "5.1. Giới thiệu"
date = 2026-08-03
weight = 1
chapter = false
+++

## Giới thiệu về Triển khai CI/CD Pipeline

Workshop này hướng dẫn bạn triển khai một CI/CD pipeline sẵn sàng cho production cho hai microservice: một NestJS GraphQL API service và một FastAPI search service. Giải pháp tận dụng các dịch vụ AWS native để tạo ra một quy trình deploy tự động, loại bỏ can thiệp thủ công, cải thiện bảo mật và giảm thời gian deploy.

### Tổng quan Workshop

Workshop kéo dài 12 tuần này ghi lại hành trình chuyển đổi CI/CD toàn diện từ quy trình deploy thủ công sang tự động hóa AWS-native cho backend của nền tảng streaming video. Quy trình thủ công tốn thời gian ban đầu đã phát triển thành một pipeline tự động hóa, hợp lý, có khả năng deploy hai microservice với bảo mật, giám sát và độ tin cậy cấp production.

Hành trình mang lại giá trị kinh doanh đáng kể bằng cách giảm thời gian deploy từ 20-30 phút xuống chỉ còn 4-5 phút cho mỗi service, loại bỏ hoàn toàn các lỗi cấu hình thủ công trước đây gây ra sự cố production. Bằng cách tập trung quản lý secrets trong AWS Secrets Manager và triển khai truy cập dựa trên IAM role, nền tảng đã đạt được bảo mật cấp doanh nghiệp mà không ảnh hưởng đến tốc độ phát triển.

### Những gì bạn sẽ học

Về mặt kỹ thuật, workshop này bao gồm:
- Đóng gói hóa NestJS GraphQL API service và FastAPI search service
- Tích hợp với 8 dịch vụ AWS (EC2, ECR, CodeBuild, S3, Secrets Manager, SSM, CloudWatch, IAM)
- Triển khai pipeline deploy đầy đủ từ commit code đến deploy production
- Các yêu cầu phức tạp bao gồm database migration, phụ thuộc giữa các service và inject secret runtime

### Đối tượng mục tiêu

Workshop này được thiết kế cho các developer và DevOps engineer muốn triển khai pipeline CI/CD mạnh mẽ trong môi trường AWS. Bạn sẽ có được kinh nghiệm thực hành với:
- Thiết kế kiến trúc AWS
- Docker multi-stage builds
- Các best practice bảo mật production
- Chiến lược tối ưu chi phí
- Khắc phục các sự cố deploy thực tế

### Thành tựu chính

Khi hoàn thành workshop này, bạn sẽ có:
- **Thời gian deploy**: Giảm từ 20-30 phút → 4-5 phút (cải thiện 83%)
- **Tỷ lệ lỗi**: Loại bỏ các sai sót khi deploy thủ công và configuration drift
- **Bảo mật**: Quản lý secrets tập trung, IAM roles, không có credentials trong code, ECR scanning
- **Khả năng mở rộng**: Sẵn sàng mở rộng cho nhiều môi trường (staging, production, feature branches)
- **Observability**: CloudWatch monitoring cho tất cả thành phần với alarms có thể hành động
- **Chi phí**: ~$22/tháng (trong giới hạn Free Tier, ít hơn 60% so với các managed service tương đương)
- **Độ tin cậy**: Khả năng rollback tự động, health checks, quản lý phụ thuộc

### Tech Stack

- **Version Control**: GitHub
- **Build System**: AWS CodeBuild
- **Container Registry**: Amazon ECR
- **Orchestration**: Docker Compose trên EC2
- **Infrastructure**: Amazon EC2 (t2.medium)
- **Secrets Management**: AWS Secrets Manager
- **Monitoring**: Amazon CloudWatch
- **Infrastructure Automation**: AWS Systems Manager (SSM)
- **Artifact Storage**: Amazon S3

### Mốc thời gian hành trình

- **Tuần 2-3**: Thiết lập tài khoản AWS và khởi chạy EC2 instance
- **Tuần 8**: Cấu hình các dịch vụ AWS (ECR, Secrets Manager, S3, IAM)
- **Tuần 9**: Phát triển Dockerfiles và các script CI/CD
- **Tuần 10**: Thiết lập và kiểm thử các project CodeBuild
- **Tuần 11-12**: Deploy production và tối ưu hóa

### Bước tiếp theo

Bắt đầu với phần Prerequisites để đảm bảo bạn có tất cả công cụ cần thiết và tài khoản AWS đã được thiết lập, sau đó tiếp tục qua từng phần theo thứ tự để xây dựng CI/CD pipeline hoàn chỉnh của bạn.