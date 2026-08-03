+++
title = "5.4. AWS Services Configuration"
date = 2026-08-03
weight = 4
chapter = false
+++

## AWS Services Configuration (Week 8)

This section covers the configuration of core AWS services required for the CI/CD pipeline: ECR repositories, Secrets Manager, S3 bucket, and IAM roles.

### 3.1 Create ECR Repositories

Create two private ECR repositories for container images:

- `api-service` (NestJS GraphQL backend)
- `search-service` (FastAPI vector search)

![ECR Repository Creation](/my-aws-fcaj-2026-journey-official/images/5-Workshop/14_ecr_repository_creation.png)
![ECR Repository Policy](/my-aws-fcaj-2026-journey-official/images/5-Workshop/13_ecr_repository_policy.png)

#### Step-by-Step ECR Setup

1. **Navigate to ECR Console** → Create repository
2. **Repository 1**: `api-service`
   - Name: `api-service`
   - Visibility: Private
   - Enable: Scan on push (for security vulnerability detection)
   - Enable: Tag immutability (optional, prevents overwriting tags)
3. **Repository 2**: `search-service`
   - Name: `search-service`
   - Same settings as above
4. **Note repository URIs**: 
   - `772706200692.dkr.ecr.us-east-1.amazonaws.com/api-service`
   - `772706200692.dkr.ecr.us-east-1.amazonaws.com/search-service`
5. **Configure repository policies** to allow CodeBuild push access

#### ECR Repository Policy

Configure repository policy to allow CodeBuild service roles to push images:

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

### 3.2 Configure Secrets Manager

Store runtime secrets for both services in AWS Secrets Manager:

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

#### Important Notes for Secrets Manager

1. **Newline characters**: Use `\\n` for newlines in JSON values (required for private keys)
2. **Encryption key**: Use `aws/secretsmanager` (default AWS-managed key)
3. **Access control**: Secrets are accessed via IAM role on EC2 - no hardcoded credentials in code
4. **Runtime fetching**: Secrets are fetched at runtime during deployment, not baked into images
5. **Rotation**: Enable automatic secret rotation for production security

#### Secret Creation Steps

1. Navigate to Secrets Manager Console → Store a new secret
2. Select "Other type of secret" → Plaintext
3. Paste JSON content for the appropriate service
4. **Secret name**: Use the format `prod/backend/{service}-service`
5. **Encryption key**: Default AWS-managed key
6. **Configure rotation**: Optional, but recommended for production
7. **Review and store**: Save the secret

### 3.3 Create S3 Bucket for Artifacts

Create an S3 bucket to store deployment artifacts (Docker Compose files):

**Bucket name**: `videoplatform-deploy-artifacts-dsk`

![S3 Deploy Artifacts Bucket](/my-aws-fcaj-2026-journey-official/images/5-Workshop/09_s3_deploy_artifacts_bucket.png)

#### S3 Bucket Configuration

- **Bucket name**: `videoplatform-deploy-artifacts-dsk` (must be globally unique)
- **Region**: us-east-1 (same as other services)
- **Public access**: **Block all public access** (access via IAM role only)
- **Versioning**: Disabled (not needed for compose files)
- **Encryption**: SSE-S3 (AWS-managed encryption)
- **Lifecycle rules**: Optional - delete files older than 30 days

#### Path Structure
```
s3://videoplatform-deploy-artifacts-dsk/
├── compose/
│   ├── docker-compose.api-service.yml
│   └── docker-compose.search-service.yml
└── backups/  # Optional, for backup files
```

#### Bucket Policy for CodeBuild Access

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

### 3.4 Configure IAM Roles

Create IAM roles with least-privilege permissions for CodeBuild and EC2:

#### CodeBuild Service Roles (2 roles)

**Role 1**: `codebuild-build-api-service-service-role`

![CodeBuild API Service Role](/my-aws-fcaj-2026-journey-official/images/5-Workshop/05_iam_codebuild_api_service_role.png)

**Attached Policies:**
- `AmazonEC2ContainerRegistryPowerUser` (push images to ECR)
- `AmazonS3FullAccess` (sync compose files to S3)
- `AmazonSSMFullAccess` (trigger EC2 deployment via SSM)
- `CodeBuildBasePolicy-build-api-service-*` (CloudWatch Logs, basic permissions)

**Role 2**: `codebuild-build-search-service-service-role`

![CodeBuild Search Service Role](/my-aws-fcaj-2026-journey-official/images/5-Workshop/06_iam_codebuild_search_service_role.png)

**Attached Policies:** (same as above, but for search service)

#### EC2 Instance Role

**Role name**: `EC2-Backend-Role`

**Attached Policies:**
- `AmazonSSMManagedInstanceCore` (receive SSM commands)
- `SecretsManagerReadWrite` (fetch runtime secrets)
- `AmazonS3ReadOnlyAccess` (download compose files)
- `AmazonEC2ContainerRegistryReadOnly` (pull Docker images)

#### Verify EC2 Role Attachment

```bash
# On EC2 instance
aws sts get-caller-identity
# Expected output: arn:aws:sts::772706200692:assumed-role/EC2-Backend-Role/i-037a4cd636a68eb7e

# Test permissions
aws s3 ls s3://videoplatform-deploy-artifacts-dsk
aws secretsmanager list-secrets --region us-east-1
```

### IAM Role Creation Steps

1. **Navigate to IAM Console** → Roles → Create role
2. **Trusted entity type**: AWS service
3. **Use case**: 
   - For CodeBuild: `CodeBuild`
   - For EC2: `EC2`
4. **Attach policies**: Select the policies listed above
5. **Role name**: Use the names specified
6. **Review and create**: Complete role creation

### Security Best Practices

1. **Least privilege**: Only grant necessary permissions
2. **Role-based access**: No hardcoded credentials
3. **Regular review**: Audit IAM policies monthly
4. **MFA enforcement**: Require MFA for IAM users
5. **Access logging**: Enable CloudTrail for API calls

### Testing Configuration

After setting up all services, test the configuration:

```bash
# Test ECR login (from CodeBuild role context)
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 772706200692.dkr.ecr.us-east-1.amazonaws.com

# Test S3 access
aws s3 cp docker-compose.api-service.yml s3://videoplatform-deploy-artifacts-dsk/compose/

# Test Secrets Manager access
aws secretsmanager get-secret-value --secret-id prod/backend/api-service --query SecretString --output text
```

### Troubleshooting

#### Issue: ECR Push Permission Denied
**Solution**: Verify repository policy allows CodeBuild role, check IAM permissions

#### Issue: Secrets Manager Access Denied
**Solution**: Ensure EC2 role has `SecretsManagerReadWrite` policy

#### Issue: S3 Access Denied
**Solution**: Check bucket policy and IAM role permissions

#### Issue: SSM Commands Not Delivered
**Solution**: Verify EC2 has `AmazonSSMManagedInstanceCore` policy

### Next Steps

With AWS services configured, you're ready to proceed to Dockerfiles & Scripts where you'll create the containerization and deployment automation code.
