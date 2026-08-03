+++
title = "5.11. Các bước tiếp theo"
date = 2026-08-03
weight = 11
chapter = false
+++

## Các bước tiếp theo

Phần này bao gồm các chiến lược tối ưu chi phí và cải tiến tương lai cho CI/CD pipeline.

### Mẹo tối ưu chi phí

Giảm chi phí AWS trong khi vẫn duy trì độ tin cậy:

#### 1. EC2 Schedule Start/Stop

Tiết kiệm ~50% chi phí compute bằng cách stop EC2 ngoài giờ làm việc:

```bash
# Manual stop/start
aws ec2 stop-instances --instance-ids i-037a4cd636a68eb7e
aws ec2 start-instances --instance-ids i-037a4cd636a68eb7e
```

**Cost impact**: ~$17/tháng → ~$8.50/tháng (nếu stop 12 giờ/ngày)

#### 2. EC2 Instance Scheduler (Automated)

Cấu hình AWS Systems Manager để tự động start/stop:

1. Điều hướng đến Systems Manager → Instance Scheduler
2. Tạo schedule: Weekdays 8AM-6PM EST
3. Liên kết với EC2 instance

**Schedule configuration:**
- **Start**: 08:00 (weekdays)
- **Stop**: 18:00 (weekdays)
- **Timezone**: EST (UTC-5)
- **Exclude dates**: Holidays

#### 3. S3 Lifecycle Policy

Xóa các compose file cũ để giảm chi phí lưu trữ:

```json
{
  "Rules": [
    {
      "ID": "DeleteOldComposeFiles",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "compose/"
      },
      "Expiration": {
        "Days": 30
      },
      "Transitions": [
        {
          "Days": 7,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
```

**Cost impact**: Giảm S3 storage từ ~$0.50 → ~$0.10/tháng

#### 4. ECR Lifecycle Policy

Chỉ giữ các image gần đây để giảm storage:

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 10 images",
      "selection": {
        "tagStatus": "any",
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

**Cost impact**: Giảm ECR storage từ ~$0.50 → ~$0.25/tháng

#### 5. Right-Size EC2 Instance

Giám sát CloudWatch metrics và downsize nếu underutilized:

```bash
# Kiểm tra average CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-037a4cd636a68eb7e \
  --start-time $(date -d '30 days ago' --iso-8601=seconds) \
  --end-time $(date --iso-8601=seconds) \
  --period 86400 \
  --statistic Average

# Nếu CPU < 20%, cân nhắc dùng t2.small
```

**Cost impact**: t2.medium ($17) → t2.small ($8.50/tháng)

#### 6. Spot Instances (Non-Production)

Cho development/staging environments:

- **Savings**: Tiết kiệm tới 70% so với on-demand
- **Trade-off**: Có thể bị interrupt với 2 phút notice
- **Use case**: Development environments, CI/CD runners

#### 7. Reserved Instances

Cho stable production workloads:

- **Commitment**: 1-year hoặc 3-year term
- **Savings**: ~40% so với on-demand
- **Requirements**: Sử dụng ổn định, predictable

### Cải tiến tương lai

Nâng cấp CI/CD pipeline với các tính năng nâng cao:

#### Multi-AZ Deployment

Triển khai qua các availability zone để có high availability:

- **Hiện tại**: Single EC2 instance trong một AZ
- **Mục tiêu**: 2+ instances qua các AZ với load balancer
- **Lợi ích**: Fault tolerance, improved availability
- **AWS services**: Application Load Balancer, Auto Scaling

**Architecture:**
```
[ALB] --> [EC2-AZ1] --> [Containers]
      --> [EC2-AZ2] --> [Containers]
```

#### Application Load Balancer

Thay thế truy cập EC2 trực tiếp bằng ALB:

- **SSL termination**: Managed certificates qua ACM
- **Health checks**: Automatic instance health detection
- **Traffic distribution**: Round-robin qua các instances
- **Security**: Tích hợp WAF

**Implementation:**
```yaml
# Future enhancement
alb:
  type: aws_alb
  name: video-platform-alb
  listeners:
    - port: 443
      protocol: HTTPS
      certificate: arn:aws:acm:...
  target_groups:
    - name: api-target
      port: 3000
      health_check:
        path: /health
```

#### RDS thay vì Containerized Database

Di chuyển từ containerized PostgreSQL sang Amazon RDS:

**Lợi ích:**
- Automated backups và point-in-time recovery
- Automatic failover cho Multi-AZ
- Automated patching
- Durability và performance tốt hơn

**Các bước migration:**
1. Tạo RDS PostgreSQL instance
2. Set up replication từ containerized DB
3. Migrate data trong maintenance window
4. Update application connection strings
5. Decommission containerized database

#### ElastiCache cho Redis

Thay thế containerized Redis bằng Amazon ElastiCache:

- **Managed service**: Automatic failover, patching
- **Scalability**: Dễ dàng vertical scaling
- **Monitoring**: CloudWatch metrics nâng cao
- **Security**: Encryption at rest và in transit

#### ECS/Fargate thay vì EC2

Di chuyển sang serverless containers:

**Lợi ích:**
- Không cần quản lý EC2
- Automatic scaling
- Pay per use
- Tích hợp tốt hơn với AWS services

**Migration effort:**
- Viết lại deployment scripts cho ECS
- Update CodeBuild buildspec
- Sửa IAM roles cho ECS tasks
- Set up ECS service với desired count

#### Infrastructure as Code

Triển khai AWS CDK hoặc Terraform:

**Trạng thái hiện tại:** Manual AWS Console setup
**Trạng thái mục tiêu:** Infrastructure as Code

**Lợi thế:**
- Reproducible deployments
- Version control cho infrastructure
- Environment parity
- Disaster recovery

**Ví dụ (CDK):**
```typescript
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

export class VideoPlatformStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // ECR Repository
    new ecr.Repository(this, 'ApiServiceRepo', {
      repositoryName: 'api-service',
      imageScanOnPush: true,
    });

    // EC2 Instance
    new ec2.Instance(this, 'Production', {
      instanceType: ec2.InstanceType.of(
        ec2.InstanceClass.T2,
        ec2.InstanceSize.MEDIUM
      ),
      vpc: /* ... */,
    });
  }
}
```

#### Blue/Green Deployments

Triển khai zero-downtime deployments:

- **Hiện tại**: In-place deployment với downtime ngắn
- **Mục tiêu**: Blue/green với instant cutover

**AWS services:** CodeDeploy, Application Load Balancer

**Lợi ích:**
- Zero downtime deployments
- Instant rollback capability
- Traffic shifting strategies

#### Integration Tests trong CodeBuild

Thêm automated testing trước deployment:

```yaml
# Trong buildspec.yml, thêm test phase
phases:
  pre_build:
    commands:
      - echo "Running unit tests..."
      - cd api_service && npm test
      - echo "Unit tests passed"

  build:
    commands:
      - echo "Building Docker image..."
      # ... existing build commands
```

**Các loại test:**
- Unit tests (Jest, pytest)
- Integration tests (test database connectivity)
- Security scans (Trivy, security headers)

#### Canary Deployments

Triển khai dần dần cho phiên bản mới:

1. Deploy tới 10% traffic
2. Giám sát metrics và errors
3. Dần tăng lên 50%, 100%
4. Tự động rollback nếu error rate tăng

**AWS services:** CodeDeploy, CloudWatch Alarms

#### Service Mesh (AWS App Mesh)

Advanced traffic management:

- **Features**: Traffic splitting, retries, timeouts
- **Observability**: Distributed tracing (X-Ray)
- **Security**: mTLS giữa các services

#### API Gateway

Centralized API management:

- **Features**: Rate limiting, API keys, throttling
- **Authentication**: Cognito, Lambda authorizers
- **Monitoring**: Detailed API metrics

#### X-Ray Tracing

Distributed tracing để debug:

- **Visualize**: Request flow qua các services
- **Debug**: Xác định latency bottlenecks
- **Integrates**: Lambda, ECS, API Gateway

### Lộ trình triển khai

| Ưu tiên | Tính năng | Effort | Ảnh hưởng |
|---------|-----------|--------|-----------|
| Cao | S3 Lifecycle Policy | Thấp | Giảm chi phí |
| Cao | ECR Lifecycle Policy | Thấp | Giảm chi phí |
| Trung bình | EC2 Scheduler | Thấp | Giảm chi phí |
| Trung bình | Integration Tests | Trung bình | Chất lượng |
| Trung bình | Infrastructure as Code | Cao | Maintainability |
| Thấp | Multi-AZ Deployment | Cao | Độ tin cậy |
| Thấp | RDS Migration | Trung bình | Độ tin cậy |

### Các bước tiếp theo được khuyến nghị

1. **Ngay lập tức**: Triển khai S3 và ECR lifecycle policies
2. **Tháng này**: Set up EC2 scheduler cho ngoài giờ làm việc
3. **Quý tới**: Thêm integration tests vào CodeBuild
4. **Tương lai**: Lên kế hoạch infrastructure-as-code migration

### Kết luận

Workshop này đã cung cấp một nền tảng vững chắc cho CI/CD automation trên AWS. Pipeline đã triển khai giảm deployment time 83% và thiết lập các best practice bảo mật cấp doanh nghiệp. Các cải tiến tương lai được nêu ở đây sẽ tiếp tục nâng cao độ tin cậy, giảm chi phí và cải thiện khả năng bảo trì khi nền tảng mở rộng.

### Tóm tắt Workshop

| Metric | Trước | Sau |
|--------|--------|-------|
| Thời gian deploy | 20-30 phút | 4-5 phút |
| Chi phí hàng tháng | - | ~$22 |
| Số dịch vụ AWS sử dụng | - | 8 |
| Tổng số build | - | 42+ |
| Tỷ lệ thành công | - | 100% (sau các fix Tuần 10) |

**Repository:** [khiem918/VideoPlatformServer](https://github.com/khiem918/VideoPlatformServer)  
**AWS Account:** 772706200692 (us-east-1)  
**Production URL:** http://98.81.144.110:3000/graphql