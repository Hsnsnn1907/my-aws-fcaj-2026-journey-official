+++
title = "5.3. Thiết lập tài khoản AWS (AWS Account Setup)"
date = 2026-08-03
weight = 3
chapter = false
+++

## Thiết lập tài khoản AWS (AWS Account Setup) (Tuần 2-3)

Phần này trình bày việc thiết lập nền tảng cho hạ tầng AWS của bạn, bao gồm tạo tài khoản, khởi chạy EC2 instance và cài đặt các prerequisite.

### 2.1 Tạo tài khoản AWS Free Tier

Bắt đầu bằng việc tạo tài khoản AWS Free Tier tại https://aws.amazon.com/free. Thực hiện theo các bước sau:

1. **Account creation**: Cung cấp email, mật khẩu và thông tin liên hệ
2. **Payment method**: Thêm phương thức thanh toán hợp lệ (Free Tier vẫn yêu cầu bước này)
3. **Identity verification**: Hoàn tất quy trình xác minh qua điện thoại
4. **Support plan**: Chọn gói hỗ trợ "Basic" (Miễn phí)

**Các cấu hình quan trọng sau khi tạo tài khoản:**
- **Billing alerts**: Cấu hình cảnh báo ở mức 50%, 80% và 100% ngân sách
- **MFA**: Bật Multi-Factor Authentication cho tài khoản root
- **IAM user**: Tạo một user quản trị cho các thao tác hằng ngày

### 2.2 Khởi chạy EC2 Instance

EC2 instance đóng vai trò là production host cho tất cả container. Cấu hình instance với các thông số sau:

#### Cấu hình EC2 Instance (EC2 Instance Configuration)

- **Instance type**: t2.medium (2 vCPU, 4 GB RAM) - đủ cho workload development/production
- **AMI**: Ubuntu 22.04 LTS (ami-0c02fb55956c7d316) - bản phát hành LTS ổn định
- **Public IP**: 98.81.144.110 (được tự động gán trong workshop)
- **Storage**: 20 GB GP2 root volume
- **Key pair**: Tạo SSH key mới tên `video-platform` để truy cập an toàn

#### Cấu hình Network Security (Network Security Configuration)

![EC2 Network Config 1](/my-aws-fcaj-2026-journey-official/images/5-Workshop/07_ec2_network_config_1.png)
![EC2 Network Config 2](/my-aws-fcaj-2026-journey-official/images/5-Workshop/08_ec2_network_config_2.png)

**Cấu hình security groups:**
- **SSH (port 22)**: Chỉ cho phép từ IP của bạn
- **HTTP (port 80)**: Cho phép từ bất kỳ đâu (cho web application)
- **Application ports**: Cho phép từ bất kỳ đâu:
  - 3000 (API service)
  - 8000 (Search service) 
  - 5432 (PostgreSQL)
  - 6379 (Redis)
  - 5672 (RabbitMQ)
  - 6333 (Qdrant)

#### Gắn IAM Role (IAM Role Attachment)

Gắn IAM role cho EC2 instance để truy cập các dịch vụ AWS:
- **Role name**: `EC2-Backend-Role`
- **Permissions**: SSM, Secrets Manager, S3, ECR access
- **Policy**: `AmazonSSMManagedInstanceCore`, `SecretsManagerReadWrite`, `AmazonS3ReadOnlyAccess`, `AmazonEC2ContainerRegistryReadOnly`

### Các bước khởi chạy EC2 chi tiết (Detailed EC2 Launch Steps)

1. Điều hướng đến EC2 Console → Launch instance
2. **Name**: `video-platform-production`
3. **AMI**: Chọn Ubuntu 22.04 LTS
4. **Instance type**: Chọn t2.medium
5. **Key pair**: Tạo key pair mới `video-platform`, tải về file .pem
6. **Network settings**: Default VPC, bật public IP
7. **Security groups**: Cấu hình như danh sách ở trên
8. **Advanced details**: Gắn IAM role `EC2-Backend-Role`
9. **Launch instance**: Ghi nhận instance ID: `i-037a4cd636a68eb7e` (ví dụ trong workshop)

### 2.3 Cài đặt Prerequisites trên EC2

Kết nối tới EC2 qua SSH và chạy bootstrap script để cài đặt các công cụ cần thiết:

#### Kết nối SSH (SSH Connection)

```bash
# Connect to EC2 (replace with your instance IP)
ssh -i ~/.ssh/video-platform.pem ubuntu@98.81.144.110

# Run bootstrap script
sudo bash ec2-bootstrap.sh
```

#### Nội dung Bootstrap Script (Bootstrap Script Contents) (ec2-bootstrap.sh)

Bootstrap script sẽ cài đặt tất cả các prerequisite cần thiết:

```bash
#!/bin/bash
set -e

echo "Updating system packages..."
sudo apt-get update
sudo apt-get upgrade -y

echo "Installing Docker Engine..."
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "Adding Docker repository..."
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

echo "Adding user to docker group..."
sudo usermod -aG docker $USER

echo "Installing AWS CLI v2..."
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --update

echo "Installing utilities..."
sudo apt-get install -y jq netcat curl wget git

echo "Creating deployment directory structure..."
sudo mkdir -p /app/compose
sudo mkdir -p /app/logs

echo "Setting permissions..."
sudo chown -R ubuntu:ubuntu /app

echo "Bootstrap complete! Please log out and back in for group changes to take effect."
```

#### ec2-bootstrap.sh cài đặt những gì (What ec2-bootstrap.sh Installs)

1. **Docker Engine (20.10+)** và Docker Compose v2
2. **AWS CLI v2** để tương tác với các dịch vụ AWS
3. **jq** để parse JSON trong deployment script
4. **Network tools** (netcat, curl) cho health check
5. **System updates** và security patch
6. **Directory structure** cho deployment artifact

### Xác minh sau cài đặt (Post-Installation Verification)

Sau khi bootstrap script hoàn tất, hãy xác minh cài đặt:

```bash
# Log out and reconnect
exit
ssh -i ~/.ssh/video-platform.pem ubuntu@98.81.144.110

# Verify installations
docker --version
docker-compose --version
aws --version
jq --version

# Test Docker without sudo
docker run hello-world

# Verify IAM role attachment
aws sts get-caller-identity
# Expected output: arn:aws:sts::772706200692:assumed-role/EC2-Backend-Role/i-037a4cd636a68eb7e
```

### Xác minh IAM Role (IAM Role Verification)

EC2 instance cần được gắn đúng IAM role để truy cập các dịch vụ AWS:

```bash
# On EC2 instance
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
# Should return the IAM role name

# Test Secrets Manager access
aws secretsmanager list-secrets --region us-east-1 --max-results 1
# Should return list of secrets (or empty if none created yet)
```

### Cấu hình bảo mật (Security Configuration)

1. **SSH hardening**: Cập nhật cấu hình SSH để bảo mật tốt hơn
2. **Firewall rules**: Xác minh security group chỉ cho phép traffic cần thiết
3. **System monitoring**: Cài đặt các công cụ giám sát cơ bản
4. **Backup configuration**: Thiết lập backup cho EC2 instance

### Xử lý sự cố thường gặp (Troubleshooting Common Issues)

#### Sự cố (Issue): SSH Connection Failed
**Giải pháp (Solution)**: Kiểm tra security group rule, xác minh quyền của key pair, đảm bảo instance đang chạy

#### Sự cố (Issue): Docker Permission Denied
**Giải pháp (Solution)**: Đăng xuất và đăng nhập lại sau khi thêm user vào docker group, hoặc sử dụng `sudo`

#### Sự cố (Issue): IAM Role Not Attached
**Giải pháp (Solution)**: Dừng instance, gắn đúng IAM role, khởi động lại instance

#### Sự cố (Issue): Bootstrap Script Errors
**Giải pháp (Solution)**: Chạy thủ công từng lệnh trong script, kiểm tra kết nối mạng

### Bước tiếp theo (Next Steps)

Sau khi EC2 instance đã được cấu hình và các prerequisite được cài đặt, bạn đã sẵn sàng để tiếp tục với phần AWS Services Configuration, nơi bạn sẽ thiết lập ECR, Secrets Manager, S3 và IAM role cho toàn bộ CI/CD pipeline.
