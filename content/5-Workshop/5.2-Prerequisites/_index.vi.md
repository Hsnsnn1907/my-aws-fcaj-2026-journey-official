+++
title = "Yêu cầu trước khi bắt đầu"
date = 2026-08-03
weight = 2
chapter = false
+++

## Yêu cầu trước khi bắt đầu Workshop CI/CD Pipeline

Trước khi bắt đầu workshop này, hãy đảm bảo bạn có các yêu cầu sau:

### Yêu cầu tài khoản AWS

- **Tài khoản AWS Free Tier**: Tài khoản AWS đang hoạt động với quyền IAM để tạo tài nguyên
- **AWS Region**: us-east-1 (North Virginia) được khuyến nghị
- **AWS Account ID**: 772706200692 (ví dụ từ workshop, sử dụng của bạn)
- **Billing alerts**: Thiết lập alerts để theo dõi chi phí
- **MFA**: Bật Multi-Factor Authentication cho tài khoản root
- **IAM user**: Tạo IAM user với quyền admin cho các hoạt động hàng ngày

### Công cụ phát triển

- **Tài khoản GitHub**: Quyền truy cập repository và cấu hình webhook
- **AWS CLI v2**: Phiên bản mới nhất đã cài đặt và cấu hình
- **Docker Desktop**: Cho test container local (Docker 20.10+ khuyến nghị)
- **Git**: Hệ thống kiểm soát phiên bản
- **SSH client**: Truy cập an toàn đến EC2 instances
- **Text editor/IDE**: Visual Studio Code, IntelliJ IDEA, hoặc tương tự

### Kiến thức kỹ thuật

- **Hiểu biết cơ bản về Docker**: Dockerfiles, images, containers, Docker Compose
- **Kiến thức Node.js/Python**: Để hiểu codebases của microservices
- **Quen thuộc Linux CLI**: Các thao tác command line cơ bản
- **AWS fundamentals**: Quen thuộc với các dịch vụ AWS core (EC2, S3, IAM)

### Ước tính thời gian và chi phí

#### Cam kết thời gian
- **Thời gian ước tính**: 12 tuần (triển khai part-time)
- **Cam kất hàng tuần**: ~20 giờ/tuần cho triển khai và tài liệu
- **Tổng giờ**: ~240 giờ

#### Ước tính chi phí
- **Chi phí hàng tháng**: ~$23/tháng
  - EC2 t2.medium: ~$17/tháng
  - ECR storage: ~$0.50/tháng  
  - S3 storage: ~$0.50/tháng
  - Secrets Manager: ~$1/tháng
  - Data transfer: ~$3/tháng
  - CloudWatch: ~$0.50/tháng
  - CodeBuild: ~$0 (trong giới hạn Free Tier)

### Repository Workshop

- **Repository**: [khiem918/VideoPlatformServer](https://github.com/khiem918/VideoPlatformServer)
- **Branch**: `feat/CI-CD`
- **Services**: Hai microservices trong các thư mục riêng:
  - `api_service/` - NestJS GraphQL API
  - `search_service/` - FastAPI search service

### Yêu cầu mạng

- **Kết nối internet**: Kết nối ổn định để truy cập dịch vụ AWS
- **Port access**: Khả năng kết nối đến dịch vụ AWS (không có hạn chế firewall)
- **SSH access**: Port 22 cho kết nối EC2 instance

### Checklist trước Workshop

- [ ] Tài khoản AWS đã tạo và xác minh
- [ ] IAM user với quyền admin đã cấu hình
- [ ] AWS CLI v2 đã cài đặt và cấu hình (`aws configure`)
- [ ] Docker Desktop đã cài đặt và đang chạy
- [ ] Git đã cài đặt và cấu hình
- [ ] SSH key pair đã tạo cho truy cập EC2
- [ ] Tài khoản GitHub có thể truy cập
- [ ] Billing alerts đã cấu hình trong AWS Console
- [ ] MFA đã bật cho tài khoản AWS root

Với các điều kiện tiên quyết này, bạn đã sẵn sàng bắt đầu triển khai workshop từ phần Thiết lập tài khoản AWS.