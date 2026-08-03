+++
title = "5.2. Yêu cầu trước"
date = 2026-08-03
weight = 2
chapter = false
+++

## Yêu cầu trước cho Workshop CI/CD Pipeline

Trước khi bắt đầu workshop này, hãy đảm bảo bạn đã có sẵn các yêu cầu sau:

### Yêu cầu về tài khoản AWS

- **AWS Free Tier account**: Tài khoản AWS đang hoạt động với quyền IAM để tạo resource
- **AWS Region**: us-east-1 (North Virginia) khuyến nghị
- **AWS Account ID**: 772706200692 (ví dụ từ workshop, hãy dùng tài khoản của bạn)
- **Billing alerts**: Thiết lập cảnh báo billing để theo dõi chi phí
- **MFA**: Bật Multi-Factor Authentication cho tài khoản root
- **IAM user**: Tạo một IAM user với quyền admin cho các thao tác hằng ngày

### Công cụ phát triển

- **GitHub account**: Quyền truy cập repository và cấu hình webhook
- **AWS CLI v2**: Phiên bản mới nhất đã cài đặt và cấu hình
- **Docker Desktop**: Dùng để test container local (khuyến nghị Docker 20.10+)
- **Git**: Hệ thống version control
- **SSH client**: Dùng để truy cập bảo mật vào các EC2 instance
- **Text editor/IDE**: Visual Studio Code, IntelliJ IDEA hoặc tương đương

### Kiến thức kỹ thuật

- **Hiểu biết cơ bản về Docker**: Dockerfile, image, container, Docker Compose
- **Kiến thức Node.js/Python**: Để hiểu codebase của các microservice
- **Quen thuộc với Linux CLI**: Các thao tác command line cơ bản
- **AWS fundamentals**: Quen thuộc với các dịch vụ AWS cốt lõi (EC2, S3, IAM)

### Ước tính thời gian và chi phí

#### Cam kết thời gian
- **Thời lượng ước tính**: 12 tuần (triển khai bán thời gian)
- **Cam kết hằng tuần**: ~20 giờ/tuần cho triển khai và tài liệu
- **Tổng số giờ**: ~240 giờ

#### Ước tính chi phí
- **Chi phí hằng tháng**: ~$23/tháng
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
- **Services**: Hai microservice trong các thư mục riêng biệt:
  - `api_service/` - NestJS GraphQL API
  - `search_service/` - FastAPI search service

### Yêu cầu về mạng

- **Kết nối Internet**: Kết nối internet ổn định để truy cập các dịch vụ AWS
- **Truy cập cổng (Port)**: Khả năng kết nối tới các dịch vụ AWS (không bị firewall công ty hạn chế)
- **Truy cập SSH**: Truy cập cổng 22 để kết nối EC2 instance

### Máy trạm phát triển

- **Hệ điều hành**: Windows 10+, macOS 10.15+, hoặc Linux
- **Bộ nhớ (RAM)**: Tối thiểu 8GB (khuyến nghị 16GB)
- **Lưu trữ**: 10GB trống cho Docker image và công cụ
- **CPU**: Bộ xử lý đa nhân để build Docker

### Checklist trước Workshop

Trước khi bắt đầu triển khai:

- [ ] Đã tạo và xác minh tài khoản AWS
- [ ] Đã cấu hình IAM user với quyền admin
- [ ] Đã cài đặt và cấu hình AWS CLI v2 (`aws configure`)
- [ ] Đã cài đặt và chạy Docker Desktop
- [ ] Đã cài đặt và cấu hình Git
- [ ] Đã sinh cặp SSH key để truy cập EC2
- [ ] Có thể truy cập tài khoản GitHub
- [ ] Đã cấu hình billing alerts trong AWS Console
- [ ] Đã bật MFA cho tài khoản AWS root

### Xác minh tài khoản

Kiểm tra thiết lập AWS của bạn bằng các lệnh sau:

```bash
# Verify AWS CLI configuration
aws sts get-caller-identity

# Check Docker installation
docker --version
docker run hello-world

# Verify Git
git --version

# Test SSH key generation
ssh-keygen -t rsa -b 4096 -f ~/.ssh/video-platform -q -N ""
```

### Khắc phục sự cố thiết lập

Nếu bạn gặp vấn đề khi thiết lập:

1. **Xác thực AWS CLI**: Đảm bảo bạn có credentials hợp lệ và region chính xác
2. **Sự cố quyền Docker**: Thêm user của bạn vào group docker hoặc chạy với sudo
3. **Truy cập GitHub**: Xác minh SSH key đã được thêm vào tài khoản GitHub
4. **Xung đột cổng (Port)**: Kiểm tra các service đang chạy trên các cổng 3000, 8000, 5432, v.v.
5. **Hạn chế firewall**: Đảm bảo các kết nối đi ra tới dịch vụ AWS được phép

### Thiết lập giám sát chi phí

Thiết lập giám sát chi phí trước khi bắt đầu:

1. **AWS Cost Explorer**: Bật trong Billing & Cost Management
2. **Billing alarms**: Tạo các CloudWatch alarm cho ngưỡng ngân sách
3. **AWS Budgets**: Thiết lập ngân sách hằng tháng với thông báo
4. **Cost allocation tags**: Gắn tag cho resource để theo dõi

### Cân nhắc về region

- **Region khuyến nghị**: us-east-1 (North Virginia)
- **Region thay thế**: us-west-2 (Oregon), eu-west-1 (Ireland)
- **Cân nhắc**: Khả dụng của dịch vụ, chênh lệch giá, độ trễ

### Cân nhắc về bảo mật

- **Không bao giờ chia sẻ AWS credentials**: Sử dụng IAM role thay thế
- **Bảo mật SSH key**: Bảo vệ file private key với quyền phù hợp
- **Sao lưu định kỳ**: Sao lưu các file cấu hình quan trọng
- **Audit trail**: Bật AWS CloudTrail để ghi nhật ký hoạt động

Với các yêu cầu trước này đã sẵn sàng, bạn đã có thể bắt đầu triển khai workshop bắt đầu từ phần AWS Account Setup.
