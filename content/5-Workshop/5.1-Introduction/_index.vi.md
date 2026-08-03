+++
title = "Giới thiệu"
date = 2026-08-03
weight = 1
chapter = false
+++

## Giới thiệu về Workshop CI/CD Pipeline

Workshop này hướng dẫn bạn triển khai pipeline CI/CD sản xuất cho hai microservices: service API NestJS GraphQL và service tìm kiếm FastAPI. Giải pháp sử dụng các dịch vụ AWS native để tạo luồng deploy tự động, loại bỏ can thiệp thủ công, cải thiện bảo mật và giảm thời gian deploy.

### Tổng quan Workshop

Workshop 12 tuần này ghi lại hành trình chuyển đổi CI/CD toàn diện từ deploy thủ công sang tự động hóa AWS-native cho backend nền tảng video streaming. Quá trình từ một quy trình thủ công tốn thời gian đã phát triển thành một pipeline tinh gọn, tự động, triển khai hai microservices với bảo mật, giám sát và độ tin cậy cấp sản xuất.

Hành trình này mang lại giá trị kinh doanh đáng kể bằng cách giảm thời gian deploy từ 20-30 phút xuống chỉ còn 4-5 phút mỗi service, loại bỏ hoàn toàn các lỗi cấu hình thủ công trước đây gây ra sự cố sản xuất. Bằng cách tập trung quản lý secrets trong AWS Secrets Manager và triển khai quyền truy cập dựa trên IAM, nền tảng đã đạt được bảo mật cấp doanh nghiệp mà không ảnh hưởng đến tốc độ phát triển của developer.

### Những gì bạn sẽ học được

Về mặt kỹ thuật, workshop này bao gồm:
- Container hóa service API NestJS GraphQL và service tìm kiếm FastAPI
- Tích hợp với 8 dịch vụ AWS (EC2, ECR, CodeBuild, S3, Secrets Manager, SSM, CloudWatch, IAM)
- Triển khai pipeline deploy đầy đủ từ commit code đến deploy sản xuất
- Xử lý các yêu cầu phức tạp bao gồm migrations database, phụ thuộc service và inject secret runtime

### Đối tượng mục tiêu

Workshop này được thiết kế cho các developer và kỹ sư DevOps muốn triển khai pipeline CI/CD mạnh mẽ trong môi trường AWS. Bạn sẽ có kinh nghiệm thực hành với:
- Thiết kế kiến trúc AWS
- Docker multi-stage builds
- Best practices bảo mật sản xuất
- Chiến lược tối ưu chi phí
- Khắc phục sự cố deploy thực tế

### Các thành tựu chính

- **Thời gian deploy**: Giảm từ 20-30 phút → 4-5 phút (cải thiện 83%)
- **Tỷ lệ lỗi**: Loại bỏ các lỗi deploy thủ công và drift cấu hình
- **Bảo mật**: Quản lý secrets tập trung, IAM roles, không có credentials trong code, ECR scanning
- **Khả năng mở rộng**: Sẵn sàng scale lên nhiều môi trường (staging, production, feature branches)
- **Giám sát**: CloudWatch monitoring cho tất cả components với alarms có thể hành động
- **Chi phí**: ~$22/tháng (trong giới hạn Free Tier, tiết kiệm 60% so với dịch vụ managed)
- **Độ tin cậy**: Khả năng rollback tự động, health checks, quản lý phụ thuộc

### Thời gian biểu hành trình

- **Tuần 2-3**: Thiết lập tài khoản AWS và launch EC2 instance
- **Tuần 8**: Cấu hình dịch vụ AWS (ECR, Secrets Manager, S3, IAM)
- **Tuần 9**: Phát triển Dockerfiles và CI/CD scripts
- **Tuần 10**: Thiết lập và test CodeBuild projects
- **Tuần 11-12**: Deploy sản xuất và tối ưu hóa

### Bước tiếp theo

Bắt đầu với phần Prerequisites để đảm bảo bạn có tất cả các công cụ cần thiết và thiết lập tài khoản AWS, sau đó tiến qua từng phần để xây dựng pipeline CI/CD hoàn chỉnh.