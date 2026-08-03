+++
title = "5.3. Thiết lập Tài khoản AWS"
date = 2026-08-03
weight = 3
chapter = false
+++

## Thiết lập Tài khoản AWS (Tuần 2-3)

Phần này bao gồm việc thiết lập cơ sở hạ tầng AWS: tạo tài khoản, launch EC2 instance và cài đặt prerequisites.

### Tạo Tài khoản AWS Free Tier

1. **Tạo tài khoản**: Truy cập https://aws.amazon.com/free và tạo tài khoản mới
2. **Xác minh danh tính**: Hoàn thành quy trình xác minh qua điện thoại
3. **Cấu hình quan trọng**: 
   - Thiết lập billing alerts
   - Bật MFA cho tài khoản root
   - Tạo IAM user với quyền admin

### Launch EC2 Instance

EC2 instance phục vụ như host sản xuất cho tất cả containers:

#### Cấu hình EC2
- **Instance type**: t2.medium (2 vCPU, 4 GB RAM)
- **AMI**: Ubuntu 22.04 LTS
- **Public IP**: 98.81.144.110
- **Storage**: 20 GB GP2
- **Security groups**: SSH (port 22), HTTP (port 80), ports ứng dụng (3000, 8000, 5432, 6379, 5672, 6333)
- **IAM role**: `EC2-Backend-Role` (truy cập SSM, Secrets Manager, S3, ECR)

### Cài đặt Prerequisites trên EC2

Kết nối EC2 qua SSH và chạy bootstrap script:

```bash
# Kết nối EC2
ssh -i ~/.ssh/video-platform.pem ubuntu@98.81.144.110

# Chạy bootstrap script
sudo bash ec2-bootstrap.sh
```

#### Script bootstrap cài đặt:
- Docker Engine (20.10+) và Docker Compose v2
- AWS CLI v2
- jq cho JSON parsing
- Công cụ mạng (netcat, curl)
- Cập nhật hệ thống

#### Xác minh cài đặt:
```bash
docker --version
docker-compose --version
aws --version
aws sts get-caller-identity
```

### Cấu hình bảo mật

- SSH hardening: Cập nhật SSH config
- Firewall rules: Đảm bảo security groups chỉ cho phép traffic cần thiết
- Backup: Cấu hình backups cho EC2

### Troubleshooting

| Vấn đề | Giải pháp |
|--------|-----------|
| SSH không kết nối | Kiểm tra security group, verify key pair |
| Docker permission denied | Thoát và đăng nhập lại |
| IAM role không gắn | Dừng instance, gắn role, khởi động lại |

Với EC2 đã cấu hình và prerequisites đã cài đặt, bạn sẵn sàng đi đến Cấu hình Dịch vụ AWS.