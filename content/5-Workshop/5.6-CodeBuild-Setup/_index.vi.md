+++
title = "Thiết lập CodeBuild"
date = 2026-08-03
weight = 6
chapter = false
+++

## Thiết lập CodeBuild Projects (Tuần 10)

### Tạo CodeBuild Project - API Service

1. **Project name**: `build-api-service`
2. **Source provider**: GitHub
3. **Repository**: `khiem918/VideoPlatformServer`
4. **Webhook filters**: FILE_PATH matches `^api_service/.*$`
5. **Environment**:
   - Image: `aws/codebuild/standard:7.0` (Ubuntu 22.04)
   - Compute: `BUILD_GENERAL1_SMALL`
   - Privileged mode: **Enabled**
   - Service role: `codebuild-build-api-service-service-role`
6. **Buildspec**: `buildspec-api.yml`
7. **Environment variables**: AWS_DEFAULT_REGION, AWS_ACCOUNT_ID, ECR_REPO_NAME, DEPLOY_S3_BUCKET, DEPLOY_S3_PREFIX, EC2_INSTANCE_ID

### Test Build

Build #1 Results:
- **Status**: SUCCESS
- **Duration**: 3m 42s
- **Breakdown**: PRE_BUILD (15s), BUILD (2m 15s), POST_BUILD (30s + 5s + 7s)

### Xác minh trên EC2

```bash
# Kiểm tra containers đang chạy
sudo docker ps -a

# Logs API service
sudo docker logs api_service
# Expected: "Application is running on: http://0.0.0.0:3000"

# Test API endpoint
curl http://localhost:3000/graphql
```

### Lặp lại cho Search Service

Create project: `build-search-service`
- Webhook filter: `^search_service/.*$`
- Buildspec: `buildspec-search.yml`
- Service role: `codebuild-build-search-service-service-role`

### Configure GitHub Webhooks

1. CodeBuild project → Build triggers
2. Enable webhook
3. Repository URL: `https://github.com/khiem918/VideoPlatformServer`
4. Secret: Generate webhook secret
5. Filter patterns: PUSH event, branch `feat/CI-CD`, file paths `^api_service/.*$`

### Quy trình Build và Deploy

1. **Push code**: Developer push changes lên `feat/CI-CD`
2. **Webhook trigger**: GitHub trigger CodeBuild project
3. **ECR auth**: CodeBuild authenticate với ECR
4. **Docker build**: Multi-stage build tạo container image
5. **Push image**: Build image push lên ECR với `latest` và git SHA tags
6. **Artifact sync**: Docker Compose files sync lên S3
7. **SSM command**: CodeBuild trigger SSM chạy deploy.sh trên EC2
8. **EC2 deploy**: EC2 pull image mới, fetch secrets, restart containers
9. **Health verify**: Services được validate và CloudWatch monitoring

Với CodeBuild projects đã cấu hình, hãy đi đến Deploy sản xuất.