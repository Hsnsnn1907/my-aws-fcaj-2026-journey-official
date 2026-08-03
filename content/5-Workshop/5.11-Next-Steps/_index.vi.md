+++
title = "Các bước tiếp theo"
date = 2026-08-03
weight = 11
chapter = false
+++

## Các bước tiếp theo và Tối ưu hóa

### Mẹo tối ưu chi phí

#### 1. Bật/tắt EC2 theo lịch trình

Lưu ~50% chi phí compute:

```bash
aws ec2 stop-instances --instance-ids i-037a4cd636a68eb7e
aws ec2 start-instances --instance-ids i-037a4cd636a68eb7e
```

#### 2. EC2 Instance Scheduler

Cấu hình Systems Manager để tự động start/stop:
- **Start**: 8AM (thứ 2 - thứ 6)
- **Stop**: 6PM (thứ 2 - thứ 6)
- **Timezone**: EST

#### 3. S3 Lifecycle Policy

Xóa compose files cũ sau 30 ngày:

```json
{
  "Rules": [{
    "ID": "DeleteOldComposeFiles",
    "Status": "Enabled",
    "Filter": {"Prefix": "compose/"},
    "Expiration": {"Days": 30}
  }]
}
```

#### 4. ECR Lifecycle Policy

Chỉ giữ 10 images gần nhất:

```json
{
  "rules": [{
    "rulePriority": 1,
    "selection": {
      "tagStatus": "any",
      "countNumber": 10
    },
    "action": {"type": "expire"}
  }]
}
```

#### 5. Right-size EC2 instance

Giảm từ t2.medium sang t2.small nếu CPU < 20%

### Các cải tiến tương lai

| Cải tiến | Mô tả |
|----------|-------|
| Multi-AZ deployment | High availability qua availability zones |
| Application Load Balancer | Traffic distribution và SSL termination |
| RDS thay vì PostgreSQL container | Độ bền tốt hơn, backups tự động |
| ElastiCache cho Redis | Managed service |
| ECS/Fargate thay vì EC2 | Serverless containers |
| AWS CDK/Terraform | Infrastructure-as-code |
| Blue/Green deployments | Zero-downtime updates |
| Integration tests trong CodeBuild | Chạy trước deploy |
| Canary deployments | Gradual rollout |
| API Gateway | API management và rate limiting |
| X-Ray tracing | Distributed tracing |

### Lộ trình triển khai

| Ưu tiên | Tính năng | Effort | Ảnh hưởng |
|---------|-----------|--------|-----------|
| Cao | S3 Lifecycle Policy | Thấp | Giảm chi phí |
| Cao | ECR Lifecycle Policy | Thấp | Giảm chi phí |
| Trung bình | EC2 Scheduler | Thấp | Giảm chi phí |
| Trung bình | Integration tests | Trung bình | Chất lượng |
| Trung bình | Infrastructure as Code | Cao | Bảo trì |
| Thấp | Multi-AZ | Cao | Độ tin cậy |

Với các bước tiếp theo được xác định, bạn đã hoàn thành workshop CI/CD!