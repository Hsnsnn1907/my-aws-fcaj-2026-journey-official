+++
title = "5.9. Bảo mật"
date = 2026-08-03
weight = 9
chapter = false
+++

## Best Practices Bảo mật (Security Best Practices)

Phần này đề cập đến các best practice bảo mật để bảo vệ CI/CD pipeline, hạ tầng AWS và các ứng dụng container hóa.

### 8.1 Quản lý Secrets (Secrets Management)

Triển khai quản lý secrets an toàn xuyên suốt pipeline:

#### Checklist Best Practices

- **Không có secrets trong code**: Đảm bảo không có hardcoded secrets trong source code
- **Secrets lưu trong Secrets Manager**: Tất cả secrets trong AWS Secrets Manager
- **Truy cập dựa trên IAM role**: Không có long-term credentials, sử dụng IAM roles
- **Inject secret runtime**: Chỉ lấy secrets khi runtime
- **Mã hóa khi lưu trữ (at rest)**: Sử dụng AWS-managed encryption keys
- **Secret rotation**: Bật automatic rotation (khuyến nghị 90 ngày)

#### Tính năng bảo mật của Secrets Manager

| Tính năng | Triển khai | Trạng thái |
|---------|----------------|--------|
| Mã hóa | AWS-managed keys (aws/secretsmanager) | Đã bật |
| Kiểm soát truy cập | IAM policies trên EC2 role | Đã cấu hình |
| Audit logging | CloudTrail đã bật | Bắt buộc |
| Rotation | Lịch 90 ngày | Khuyến nghị |

#### Mẫu truy cập Secrets

```python
# Secrets được lấy tại runtime bởi deploy.sh
import boto3
import json

def get_secret(secret_id):
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_id)
    return json.loads(response['SecretString'])

# Không có credentials trong code - chỉ EC2 IAM role
secret = get_secret('prod/backend/api-service')
```

### 8.2 Bảo mật mạng (Network Security)

Bảo vệ truy cập và giao tiếp mạng:

#### Cấu hình Security Group

| Port | Source | Mục đích | Ghi chú |
|------|--------|---------|-------|
| 22 | Chỉ IP của bạn | SSH | Giới hạn cho các IP tin cậy |
| 80 | Bất kỳ | HTTP | Cho web applications |
| 3000 | Bất kỳ | API service | Internal service port |
| 8000 | Bất kỳ | Search service | Internal service port |
| 5432 | Docker network | PostgreSQL | Container-to-container |
| 6379 | Docker network | Redis | Container-to-container |
| 5672 | Docker network | RabbitMQ | Container-to-container |
| 6333 | Docker network | Qdrant | Container-to-container |

#### Checklist bảo mật mạng

- **EC2 trong default VPC**: Không có public subnets cho databases
- **Security groups**: Mở tối thiểu các port, least privilege
- **IMDSv2**: Bắt buộc để bảo vệ EC2 metadata
- **ECR private repositories**: Không truy cập public
- **S3 bucket**: Chặn tất cả truy cập public
- **SSH access**: Giới hạn cho các IP cụ thể

#### Các lệnh Security Group

```bash
# Tạo security group
aws ec2 create-security-group \
  --group-name video-platform-sg \
  --description "Security group for video platform" \
  --vpc-id vpc-xxxxxxxx

# Cho phép SSH từ IP cụ thể
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp \
  --port 22 \
  --cidr your-ip-address/32

# Cho phép các port ứng dụng
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp \
  --port 3000 \
  --cidr 0.0.0.0/0
```

### 8.3 Bảo mật container (Container Security)

Bảo vệ container images và runtime:

#### Checklist bảo mật container

- **Multi-stage builds**: Giảm thiểu attack surface
- **Non-root user**: Chạy container với non-root (nếu có thể)
- **ECR image scanning**: Bật để phát hiện lỗ hổng
- **Minimal base images**: Sử dụng các biến thể alpine/slim
- **Không có credentials trong images**: Tất cả secrets được inject tại runtime
- **Quét thường xuyên**: Quét lỗ hổng hàng tuần

#### Best Practices bảo mật Dockerfile

```dockerfile
# Sử dụng version cụ thể, không dùng 'latest'
FROM node:18-alpine

# Tạo non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy file với đúng ownership
COPY --chown=nodejs:nodejs . .

# Chuyển sang non-root user
USER nodejs

# Không chạy với root
EXPOSE 3000
```

#### ECR Image Scanning

```bash
# Bật scan on push
aws ecr put-image-scanning-configuration \
  --repository-name api-service \
  --image-scanning-configuration scanOnPush=true

# Xem scan findings
aws ecr describe-image-scan-findings \
  --repository-name api-service \
  --image-id imageTag=latest
```

### 8.4 Nguyên tắc đặc quyền tối thiểu IAM (IAM Least Privilege)

Triển khai kiểm soát truy cập đặc quyền tối thiểu:

#### IAM Best Practices

- **Tách riêng roles**: Khác roles cho CodeBuild và EC2
- **Resource-specific policies**: Tránh wildcards khi có thể
- **Không có AWS access keys**: Chỉ sử dụng IAM roles
- **Review policy thường xuyên**: Audit IAM policies hàng tháng
- **Bắt buộc MFA**: Yêu cầu MFA cho tất cả IAM users

#### CodeBuild Role Policies (Tối thiểu)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:CompleteLayerUpload",
        "ecr:GetDownloadUrlForLayer",
        "ecr:InitiateLayerUpload",
        "ecr:PutImage",
        "ecr:UploadLayerPart"
      ],
      "Resource": "arn:aws:ecr:us-east-1:772706200692:repository/api-service"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::videoplatform-deploy-artifacts-dsk",
        "arn:aws:s3:::videoplatform-deploy-artifacts-dsk/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ssm:SendCommand"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ssm:instance-id": "i-037a4cd636a68eb7e"
        }
      }
    }
  ]
}
```

#### EC2 Role Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": [
        "arn:aws:secretsmanager:us-east-1:772706200692:secret:prod/backend/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::videoplatform-deploy-artifacts-dsk/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage"
      ],
      "Resource": "*"
    }
  ]
}
```

### 8.5 Security Audit

Audit bảo mật và kiểm tra tuân thủ thường xuyên:

#### Checklist bảo mật hàng tuần

- [ ] Review CloudTrail để tìm API calls bất thường
- [ ] Kiểm tra IAM access analyzer cho public resources
- [ ] Review ECR scan findings
- [ ] Xác minh security group rules
- [ ] Kiểm tra IAM credentials không sử dụng
- [ ] Review VPC flow logs cho traffic bất thường

#### Các lệnh Security Audit

```bash
# Kiểm tra access keys không sử dụng
aws iam list-access-keys --user-name <username>

# Review IAM policies
aws iam list-policies --scope AWS

# Kiểm tra security groups cho wide-open rules
aws ec2 describe-security-groups --filters "Name=ip-permission.cidr,Values=0.0.0.0/0"

# Review CloudTrail events
aws cloudtrail lookup-events \
  --start-time $(date -d '7 days ago' --iso-8601=seconds) \
  --event-name "CreateUser"
```

### 8.6 Incident Response

Chuẩn bị cho các sự cố bảo mật:

#### Các bước phản hồi

1. **Xác định (Identify)**: Sử dụng CloudWatch alarms và logs
2. **Cô lập (Contain)**: Cô lập các resources bị ảnh hưởng
3. **Điều tra (Investigate)**: Review logs và CloudTrail
4. **Khắc phục (Remediate)**: Áp dụng các fix
5. **Tài liệu hóa (Document)**: Ghi lại chi tiết sự cố
6. **Review**: Cập nhật security controls

#### Các sự cố bảo mật thường gặp

| Sự cố | Dấu hiệu | Phản hồi |
|----------|------------|----------|
| Truy cập trái phép | API calls bất thường, thay đổi IAM | Thu hồi credentials, rotate secrets |
| Data exfiltration | NetworkOut cao | Chặn traffic, điều tra nguồn |
| Malware | Container crashes, resource spikes | Cô lập, rebuild container |

### 8.7 Cân nhắc tuân thủ (Compliance Considerations)

Đảm bảo tuân thủ các security framework:

- **Mã hóa**: Tất cả data được mã hóa at rest và in transit
- **Kiểm soát truy cập**: Truy cập dựa trên IAM, không có credentials chung
- **Audit trail**: CloudTrail đã bật cho tất cả API calls
- **Giám sát**: CloudWatch cho operational monitoring
- **Quản lý lỗ hổng**: ECR scanning đã bật

### Tóm tắt Best Practices Bảo mật

| Lĩnh vực | Best practice | Trạng thái |
|------|----------|--------|
| Secrets | AWS Secrets Manager + IAM roles | Đã triển khai |
| Mạng | Security groups + private subnets | Đã triển khai |
| Containers | Multi-stage builds + ECR scanning | Đã triển khai |
| IAM | Least privilege + separate roles | Đã triển khai |
| Giám sát | CloudWatch + alarms | Đã triển khai |

### Bước tiếp theo

Với các best practice bảo mật đã được áp dụng, tiếp tục đến phần Troubleshooting để tìm giải pháp cho các sự cố thường gặp trong quá trình triển khai.