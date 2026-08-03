+++
title = "5.4. Cấu hình Dịch vụ AWS"
date = 2026-08-03
weight = 4
chapter = false
+++

## Cấu hình Dịch vụ AWS (AWS Services Configuration - Tuần 8)

{{< notice warning >}}
Lưu ý: Thông tin dưới đây chỉ mang tính tham khảo. Vui lòng không sao chép nguyên văn cho báo cáo của bạn, bao gồm cả cảnh báo này.
{{< /notice >}}

Phần này bao gồm cấu hình các dịch vụ AWS cốt lõi cần thiết cho CI/CD pipeline: ECR repositories, Secrets Manager, S3 bucket và IAM roles.

### 3.1 Tạo ECR Repositories

Tạo hai private ECR repositories cho container images:

- `api-service` (NestJS GraphQL backend)
- `search-service` (FastAPI vector search)

![ECR Repository Creation](/my-aws-fcaj-2026-journey-official/images/5-Workshop/14_ecr_repository_creation.png)
![ECR Repository Policy](/my-aws-fcaj-2026-journey-official/images/5-Workshop/13_ecr_repository_policy.png)

#### Hướng dẫn từng bước thiết lập ECR (Step-by-Step ECR Setup)

1. **Điều hướng đến ECR Console** → Create repository
2. **Repository 1**: `api-service`
   - Name: `api-service`
   - Visibility: Private
   - Bật: Scan on push (để phát hiện lỗ hổng bảo mật)
   - Bật: Tag immutability (tùy chọn, ngăn ghi đè tag)
3. **Repository 2**: `search-service`
   - Name: `search-service`
   - Cùng cài đặt như trên
4. **Ghi lại repository URIs**: 
   - `772706200692.dkr.ecr.us-east-1.amazonaws.com/api-service`
   - `772706200692.dkr.ecr.us-east-1.amazonaws.com/search-service`
5. **Cấu hình repository policies** để cho phép CodeBuild push access

#### ECR Repository Policy

Cấu hình repository policy để cho phép CodeBuild service roles push images:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCodeBuildPush",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::772706200692:role/codebuild-build-api-service-service-role",
          "arn:aws:iam::772706200692:role/codebuild-build-search-service-service-role"
        ]
      },
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage",
        "ecr:PutImage",
        "ecr:InitiateLayerUpload",
        "ecr:UploadLayerPart",
        "ecr:CompleteLayerUpload"
      ]
    }
  ]
}
```

### 3.2 Cấu hình Secrets Manager

Lưu trữ runtime secrets cho cả hai services trong AWS Secrets Manager:

#### API Service Secrets

![Secrets Manager - API Service](/my-aws-fcaj-2026-journey-official/images/5-Workshop/03_secrets_manager_api_service.png)

**Secret name**: `prod/backend/api-service`
```json
{
  "DATABASE_URL": "postgresql://postgres:postgres@postgres:5432/video_streaming_api?schema=public",
  "RABBITMQ_URL": "amqp://guest:guest@rabbitmq:5672",
  "REDIS_URL": "redis://redis:6379",
  "QDRANT_URL": "http://qdrant:6333",
  "JWT_SECRET": "your-jwt-secret-key-here",
  "FIREBASE_PRIVATE_KEY": "-----BEGIN PRIVATE KEY-----\\nMIIEvQIBADANBgk...\\n-----END PRIVATE KEY-----",
  "FIREBASE_CLIENT_EMAIL": "firebase-adminsdk@project.iam.gserviceaccount.com",
  "AWS_ACCESS_KEY_ID": "",
  "AWS_SECRET_ACCESS_KEY": "",
  "AWS_REGION": "us-east-1",
  "CLOUDINARY_CLOUD_NAME": "your-cloud-name",
  "CLOUDINARY_API_KEY": "your-api-key",
  "CLOUDINARY_API_SECRET": "your-api-secret"
}
```

#### Search Service Secrets

![Secrets Manager - Search Service](/my-aws-fcaj-2026-journey-official/images/5-Workshop/04_secrets_manager_search_service.png)

**Secret name**: `prod/backend/search-service`
```json
{
  "DATABASE_URL": "postgresql://postgres:postgres@postgres:5432/video_streaming_search",
  "RABBITMQ_URL": "amqp://guest:guest@172.17.0.1:5672",
  "QDRANT_URL": "http://qdrant:6333",
  "REDIS_URL": "redis://redis:6379",
  "OPENAI_API_KEY": "sk-...",
  "GEMINI_API_KEY": "AIza...",
  "AWS_ACCESS_KEY_ID": "",
  "AWS_SECRET_ACCESS_KEY": "",
  "AWS_REGION": "us-east-1"
}
```

#### Lưu ý quan trọng cho Secrets Manager (Important Notes)

1. **Ký tự xuống dòng**: Sử dụng `\\n` cho newline trong JSON values (bắt buộc cho private keys)
2. **Encryption key**: Sử dụng `aws/secretsmanager` (default AWS-managed key)
3. **Access control**: Secrets được truy cập qua IAM role trên EC2 - không có credentials trong code
4. **Runtime fetching**: Secrets được lấy tại runtime trong quá trình deployment, không bake vào images
5. **Rotation**: Bật automatic secret rotation cho bảo mật production

#### Các bước tạo Secret (Secret Creation Steps)

1. Điều hướng đến Secrets Manager Console → Store a new secret
2. Chọn "Other type of secret" → Plaintext
3. Paste nội dung JSON cho service tương ứng
4. **Secret name**: Sử dụng format `prod/backend/{service}-service`
5. **Encryption key**: Default AWS-managed key
6. **Cấu hình rotation**: Tùy chọn, nhưng khuyến nghị cho production
7. **Review và store**: Lưu secret

### 3.3 Tạo S3 Bucket cho Artifacts

Tạo một S3 bucket để lưu trữ deployment artifacts (Docker Compose files):

**Bucket name**: `videoplatform-deploy-artifacts-dsk`

![S3 Deploy Artifacts Bucket](/my-aws-fcaj-2026-journey-official/images/5-Workshop/09_s3_deploy_artifacts_bucket.png)

#### Cấu hình S3 Bucket (S3 Bucket Configuration)

- **Bucket name**: `videoplatform-deploy-artifacts-dsk` (phải globally unique)
- **Region**: us-east-1 (giống các dịch vụ khác)
- **Public access**: **Block all public access** (truy cập qua IAM role only)
- **Versioning**: Disabled (không cần cho compose files)
- **Encryption**: SSE-S3 (AWS-managed encryption)
- **Lifecycle rules**: Tùy chọn - xóa các file cũ hơn 30 ngày

#### Cấu trúc Path (Path Structure)
```
s3://videoplatform-deploy-artifacts-dsk/
├── compose/
│   ├── docker-compose.api-service.yml
│   └── docker-compose.search-service.yml
└── backups/  # Tùy chọn, cho các file backup
```

#### Bucket Policy cho CodeBuild Access

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowCodeBuildSync",
      "Effect": "Allow",
      "Principal": {
        "AWS": [
          "arn:aws:iam::772706200692:role/codebuild-build-api-service-service-role",
          "arn:aws:iam::772706200692:role/codebuild-build-search-service-service-role"
        ]
      },
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::videoplatform-deploy-artifacts-dsk",
        "arn:aws:s3:::videoplatform-deploy-artifacts-dsk/*"
      ]
    }
  ]
}
```

### 3.4 Cấu hình IAM Roles

Tạo IAM roles với quyền least-privilege cho CodeBuild và EC2:

#### CodeBuild Service Roles (2 roles)

**Role 1**: `codebuild-build-api-service-service-role`

![CodeBuild API Service Role](/my-aws-fcaj-2026-journey-official/images/5-Workshop/05_iam_codebuild_api_service_role.png)

**Attached Policies:**
- `AmazonEC2ContainerRegistryPowerUser` (push images lên ECR)
- `AmazonS3FullAccess` (sync compose files lên S3)
- `AmazonSSMFullAccess` (trigger EC2 deployment qua SSM)
- `CodeBuildBasePolicy-build-api-service-*` (CloudWatch Logs, các quyền cơ bản)

**Role 2**: `codebuild-build-search-service-service-role`

![CodeBuild Search Service Role](/my-aws-fcaj-2026-journey-official/images/5-Workshop/06_iam_codebuild_search_service_role.png)

**Attached Policies:** (giống như trên, nhưng cho search service)

#### EC2 Instance Role

**Role name**: `EC2-Backend-Role`

**Attached Policies:**
- `AmazonSSMManagedInstanceCore` (nhận SSM commands)
- `SecretsManagerReadWrite` (fetch runtime secrets)
- `AmazonS3ReadOnlyAccess` (download compose files)
- `AmazonEC2ContainerRegistryReadOnly` (pull Docker images)

#### Xác minh EC2 Role Attachment (Verify EC2 Role Attachment)

```bash
# Trên EC2 instance
aws sts get-caller-identity
# Expected output: arn:aws:sts::772706200692:assumed-role/EC2-Backend-Role/i-037a4cd636a68eb7e

# Test permissions
aws s3 ls s3://videoplatform-deploy-artifacts-dsk
aws secretsmanager list-secrets --region us-east-1
```

### Các bước tạo IAM Role (IAM Role Creation Steps)

1. **Điều hướng đến IAM Console** → Roles → Create role
2. **Trusted entity type**: AWS service
3. **Use case**: 
   - Cho CodeBuild: `CodeBuild`
   - Cho EC2: `EC2`
4. **Attach policies**: Chọn các policies đã liệt kê ở trên
5. **Role name**: Sử dụng các tên đã chỉ định
6. **Review và create**: Hoàn tất tạo role

### Best Practices Bảo mật (Security Best Practices)

1. **Least privilege**: Chỉ cấp các quyền cần thiết
2. **Role-based access**: Không hardcode credentials
3. **Regular review**: Audit IAM policies hàng tháng
4. **MFA enforcement**: Yêu cầu MFA cho IAM users
5. **Access logging**: Bật CloudTrail cho API calls

### Kiểm thử Cấu hình (Testing Configuration)

Sau khi thiết lập tất cả các dịch vụ, kiểm thử cấu hình:

```bash
# Test ECR login (từ CodeBuild role context)
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 772706200692.dkr.ecr.us-east-1.amazonaws.com

# Test S3 access
aws s3 cp docker-compose.api-service.yml s3://videoplatform-deploy-artifacts-dsk/compose/

# Test Secrets Manager access
aws secretsmanager get-secret-value --secret-id prod/backend/api-service --query SecretString --output text
```

### Troubleshooting

#### Issue: ECR Push Permission Denied
**Solution**: Xác minh repository policy cho phép CodeBuild role, kiểm tra IAM permissions

#### Issue: Secrets Manager Access Denied
**Solution**: Đảm bảo EC2 role có policy `SecretsManagerReadWrite`

#### Issue: S3 Access Denied
**Solution**: Kiểm tra bucket policy và IAM role permissions

#### Issue: SSM Commands Not Delivered
**Solution**: Xác minh EC2 có policy `AmazonSSMManagedInstanceCore`

### Next Steps

Với các dịch vụ AWS đã được cấu hình, bạn đã sẵn sàng để tiếp tục đến phần Dockerfiles & Scripts, nơi bạn sẽ tạo code containerization và deployment automation.
