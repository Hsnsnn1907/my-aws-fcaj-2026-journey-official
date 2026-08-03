+++
title = "5.4. Cấu hình Dịch vụ AWS"
date = 2026-08-03
weight = 4
chapter = false
+++

## Cấu hình Dịch vụ AWS (Tuần 8)

Phần này bao gồm cấu hình các dịch vụ AWS: ECR repositories, Secrets Manager, S3 bucket và IAM roles.

### Tạo ECR Repositories

Tạo hai private ECR repositories:

- `api-service` (NestJS GraphQL backend)
- `search-service` (FastAPI vector search)

#### Các bước tạo:
1. Vào ECR Console → Create repository
2. Tên: `api-service`, `search-service`
3. Visibility: Private
4. Bật Scan on push
5. Ghi lại repository URIs

### Cấu hình Secrets Manager

Lưu secrets cho cả hai services:

#### API Service Secrets
Secret name: `prod/backend/api-service`
```json
{
  "DATABASE_URL": "postgresql://postgres:postgres@postgres:5432/video_streaming_api?schema=public",
  "RABBITMQ_URL": "amqp://guest:guest@rabbitmq:5672",
  "REDIS_URL": "redis://redis:6379",
  "QDRANT_URL": "http://qdrant:6333",
  "JWT_SECRET": "your-jwt-secret-key-here"
}
```

#### Search Service Secrets
Secret name: `prod/backend/search-service`
```json
{
  "DATABASE_URL": "postgresql://postgres:postgres@postgres:5432/video_streaming_search",
  "RABBITMQ_URL": "amqp://guest:guest@172.17.0.1:5672",
  "REDIS_URL": "redis://redis:6379",
  "QDRANT_URL": "http://qdrant:6333",
  "OPENAI_API_KEY": "sk-...",
  "GEMINI_API_KEY": "AIza..."
}
```

### Tạo S3 Bucket cho Artifacts

Bucket name: `videoplatform-deploy-artifacts-dsk`

**Mục đích:** Lưu trữ Docker Compose files cho EC2 deployment
- Cấu trúc path: `s3://bucket/compose/docker-compose.{api,search}-service.yml`
- Public access: **Block**
- Encryption: SSE-S3

### Cấu hình IAM Roles

#### CodeBuild Service Roles (2 roles)
- `codebuild-build-api-service-service-role`
- `codebuild-build-search-service-service-role`

**Policies:** `AmazonEC2ContainerRegistryPowerUser`, `AmazonS3FullAccess`, `AmazonSSMFullAccess`

#### EC2 Instance Role
- Role: `EC2-Backend-Role`
- Policies: `AmazonSSMManagedInstanceCore`, `SecretsManagerReadWrite`, `AmazonS3ReadOnlyAccess`, `AmazonEC2ContainerRegistryReadOnly`

### Xác minh EC2 Role
```bash
aws sts get-caller-identity
# Expected: arn:aws:sts::772706200692:assumed-role/EC2-Backend-Role/i-...
```

Với các dịch vụ AWS đã cấu hình, bạn sẵn sàng đi đến Dockerfiles và Scripts.