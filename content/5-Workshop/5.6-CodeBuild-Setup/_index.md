+++
title = "CodeBuild Setup"
date = 2026-08-03
weight = 6
chapter = false
+++

## CodeBuild Projects Setup (Week 10)

This section covers the configuration of AWS CodeBuild projects for automated build and deployment of both microservices.

### 5.1 Create CodeBuild Project - API Service

1. **Project name**: `build-api-service`
2. **Source provider**: GitHub
3. **Repository**: `khiem918/VideoPlatformServer`
4. **Webhook filters**: FILE_PATH matches `^api_service/.*$`
5. **Environment configuration**:
   - **Image**: `aws/codebuild/standard:7.0` (Ubuntu 22.04, Docker 20+)
   - **Compute type**: `BUILD_GENERAL1_SMALL`
   - **Privileged mode**: **Enabled** (required for Docker builds)
   - **Service role**: `codebuild-build-api-service-service-role`
6. **Buildspec**: `buildspec-api.yml` (in repository root)
7. **Environment variables**: Add all 6 variables from section 4.4
8. **Artifacts**: None (images pushed to ECR, files to S3)
9. **Logs**: CloudWatch logs enabled

#### Buildspec Location
Ensure `buildspec-api.yml` is placed in the repository root at:
```
VideoPlatformServer/
├── api_service/
│   ├── src/
│   ├── Dockerfile
│   └── docker-compose.api-service.yml
├── search_service/
│   ├── src/
│   ├── Dockerfile
│   └── docker-compose.search-service.yml
├── buildspec-api.yml      <-- Place here
├── buildspec-search.yml   <-- Place here
└── ec2-bootstrap.sh
```

### 5.2 Test Build

Trigger a manual build from CodeBuild console:

**Build #1 Results:**
- **Status**: SUCCESS
- **Duration**: 3m 42s
- **Breakdown**:
  - PRE_BUILD: ECR login (15s)
  - BUILD: Docker build (2m 15s)
  - POST_BUILD: ECR push (30s), S3 sync (5s), SSM trigger (7s)

**Expected build output:**
```
[Container] 2026/08/03 10:30:15 Running command echo "Logging in to Amazon ECR..."
[Container] 2026/08/03 10:30:15 Running command aws ecr get-login-password...
[Container] 2026/08/03 10:30:30 Running command echo "Building Docker image..."
[Container] 2026/08/03 10:30:30 Running command cd api_service && docker build...
[Container] 2026/08/03 10:32:45 Running command echo "Pushing image to ECR..."
[Container] 2026/08/03 10:33:15 Build complete!
```

### 5.3 Verify on EC2

After build completes, verify containers are running on EC2:

![API Service Docker Containers](/my-aws-fcaj-2026-journey-official/images/5-Workshop/01_ec2_api_docker_containers.png)
![Search Service Docker Containers](/my-aws-fcaj-2026-journey-official/images/5-Workshop/02_ec2_search_docker_containers.png)

```bash
# On EC2 instance
sudo docker ps -a
# Expected output: 6+ containers running

# List expected containers:
# - api_service (NestJS GraphQL API on port 3000)
# - search_service (FastAPI search on port 8000)
# - postgres (PostgreSQL database on port 5432)
# - redis (Redis cache on port 6379)
# - rabbitmq (Message queue on port 5672)
# - qdrant (Vector database on port 6333)

# Check API service logs
sudo docker logs api_service
# Expected: "Application is running on: http://0.0.0.0:3000" + no migration errors

# Test API endpoint
curl http://localhost:3000/graphql
# Should return GraphQL playground or schema endpoint

# Check search service health
curl http://localhost:8000/health
# Should return health check response
```

### 5.4 Repeat for Search Service

Create second CodeBuild project with identical settings:

**Project name**: `build-search-service`
- **Webhook filter**: `^search_service/.*$`
- **Buildspec**: `buildspec-search.yml`
- **Environment variables**: Same as API service, but `ECR_REPO_NAME: search-service`
- **Service role**: `codebuild-build-search-service-service-role`

![CodeBuild Search Service Success](/my-aws-fcaj-2026-journey-official/images/5-Workshop/11_codebuild_search_service_success.png)

#### buildspec-search.yml

```yaml
version: 0.2

env:
  variables:
    ECR_REPO_NAME: search-service
    DEPLOY_S3_PREFIX: compose

phases:
  pre_build:
    commands:
      - echo "Logging in to Amazon ECR..."
      - aws ecr get-login-password --region $AWS_DEFAULT_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com
      - REPO_URI=$AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$ECR_REPO_NAME
      - IMAGE_TAG=$(git rev-parse --short HEAD)
      
  build:
    commands:
      - echo "Building Docker image..."
      - cd search_service
      - docker build -t $REPO_URI:latest .
      - docker tag $REPO_URI:latest $REPO_URI:$IMAGE_TAG
      
  post_build:
    commands:
      - echo "Pushing image to ECR..."
      - docker push $REPO_URI:latest
      - docker push $REPO_URI:$IMAGE_TAG
      - cd $CODEBUILD_SRC_DIR
      - echo "Syncing compose files to S3..."
      - aws s3 sync docker/ s3://$DEPLOY_S3_BUCKET/$DEPLOY_S3_PREFIX/ --delete
      - echo "Triggering EC2 deployment..."
      - aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=instanceids,Values=$EC2_INSTANCE_ID" --parameters '{"commands":["/app/deploy.sh search"]}'
```

### 5.5 Configure GitHub Webhooks

For automatic build triggering on code push:

1. **Navigate to CodeBuild project** → Build triggers
2. **Enable webhook**: Check "Enable webhooks"
3. **Repository URL**: `https://github.com/khiem918/VideoPlatformServer`
4. **Secret**: Generate webhook secret
5. **Filter patterns**:
   - **Event type**: PUSH
   - **Branch**: feat/CI-CD
   - **File paths**: `^api_service/.*$` (for API project) or `^search_service/.*$` (for Search project)

**GitHub webhook configuration:**
- **Payload URL**: `https://codebuild.us-east-1.amazonaws.com/webhooks...`
- **Content type**: application/json
- **Secret**: Same as configured in CodeBuild
- **Events**: Push events only

### 5.6 Build Pipeline Flow

The complete build and deployment flow:

1. **Code Push**: Developer pushes changes to `feat/CI-CD` branch
2. **Webhook Trigger**: GitHub webhook triggers appropriate CodeBuild project
3. **ECR Authentication**: CodeBuild authenticates with ECR
4. **Docker Build**: Multi-stage build creates container image
5. **Image Push**: Build image pushed to ECR with `latest` and git SHA tags
6. **Artifact Sync**: Docker Compose files synchronized to S3
7. **SSM Command**: CodeBuild triggers SSM to execute deploy.sh on EC2
8. **EC2 Deployment**: EC2 pulls latest image, fetches secrets, restarts containers
9. **Health Verification**: Services validated and monitoring begins

### Testing the Complete Pipeline

```bash
# Make a code change
git add api_service/src/video/video.service.ts
git commit -m "feat: add video transcoding queue"
git push origin feat/CI-CD

# Monitor build in CodeBuild console
# Expected: Build starts within 5 seconds of push

# Monitor SSM command execution
aws ssm list-command-invocations --instance-id i-037a4cd636a68eb7e --details

# Verify deployment on EC2
ssh -i ~/.ssh/video-platform.pem ubuntu@98.81.144.110
sudo docker logs api_service | tail -20
```

### Troubleshooting Build Issues

#### Issue: Build fails with "permission denied while trying to connect"
**Solution**: Check IAM role permissions for CodeBuild, ensure ECR repository policy allows the role

#### Issue: Docker build fails with "no such file or directory"
**Solution**: Verify build context path in buildspec, check COPY commands in Dockerfile

#### Issue: S3 sync fails with "access denied"
**Solution**: Verify S3 bucket policy allows CodeBuild role, check environment variables

#### Issue: SSM command not delivered
**Solution**: Ensure EC2 has SSM managed instance core policy, check SSM agent status

### Next Steps

With CodeBuild projects configured, proceed to Production Deployment where you'll test the complete pipeline and establish deployment procedures.