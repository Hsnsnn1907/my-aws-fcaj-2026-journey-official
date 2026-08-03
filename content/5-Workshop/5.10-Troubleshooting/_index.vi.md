+++
title = "5.10. Khắc phục sự cố"
date = 2026-08-03
weight = 10
chapter = false
+++

## Hướng dẫn Khắc phục sự cố

Phần này bao gồm các vấn đề phổ biến gặp phải khi triển khai CI/CD pipeline và các giải pháp.

### Vấn đề 1: CodeBuild thất bại với "docker-entrypoint.sh not found"

**Error message:**
```
COPY failed: file not found in context: path not found: docker-entrypoint.sh
```

**Nguyên nhân gốc:** File `docker-entrypoint.sh` bị loại trừ bởi `.dockerignore`

**Khắc phục:** Xóa `docker-entrypoint.sh` khỏi `.dockerignore`

Trước (trong .dockerignore):
```
node_modules
dist
docker-entrypoint.sh  <-- Xóa dòng này
```

Sau:
```
node_modules
dist
# docker-entrypoint.sh - giữ trong context để container startup
```

**Xác minh:**
```bash
# Rebuild trong CodeBuild
# Entrypoint giờ đã được include trong Docker context
```

### Vấn đề 2: Prisma Migrations thất bại trong container

**Error message:**
```
Error: P1001: Can't reach database server at postgresql://postgres:5432
```

**Nguyên nhân gốc:** Entrypoint script cần database host/port được parse từ DATABASE_URL

**Khắc phục:** Parse DATABASE_URL trong deploy.sh và ghi environment variables

```bash
# Trong deploy.sh, thêm Python script để parse DATABASE_URL:
python3 - <<'PYEOF'
import json, os
data = json.loads(os.environ["SECRET_JSON"])
with open(f"/app/compose/.env.{os.environ['SERVICE']}", "w", encoding="utf-8") as f:
    db_url = data.get("DATABASE_URL", "")
    if "postgresql://" in db_url:
        host_port = db_url.split("@")[1].split("/")[0]
        host, port = host_port.split(":")
        f.write(f"DATABASE_URL_HOST={host}\n")
        f.write(f"DATABASE_URL_PORT={port}\n")
    # ... rest of secrets
PYEOF
```

**Xác minh:**
```bash
# Kiểm tra env file trên EC2
cat /app/compose/.env.api
# Nên chứa DATABASE_URL_HOST và DATABASE_URL_PORT
```

### Vấn đề 3: S3 Sync thất bại với "docker/ does not exist"

**Error message:**
```
fatal error: cannot start container: could not get containerd socket:
```

**Nguyên nhân gốc:** Lệnh `cd search_service` trong phase BUILD vẫn tồn tại trong POST_BUILD, khiến path S3 sync relative đến thư mục sai

**Khắc phục:** Thêm `cd $CODEBUILD_SRC_DIR` trước lệnh S3 sync

```yaml
post_build:
  commands:
    - echo "Pushing image to ECR..."
    - docker push $REPO_URI:latest
    - cd $CODEBUILD_SRC_DIR  # <-- Thêm dòng này
    - echo "Syncing compose files to S3..."
    - aws s3 sync docker/ s3://$DEPLOY_S3_BUCKET/$DEPLOY_S3_PREFIX/ --delete
```

**Xác minh:**
```bash
# Kiểm tra S3 bucket cho compose files
aws s3 ls s3://videoplatform-deploy-artifacts-dsk/compose/
```

### Vấn đề 4: EC2 Deployment bị stuck "Pulling image..."

**Error message:**
```
Pulling image (latest) from 772706200692.dkr.ecr.us-east-1.amazonaws.com/api-service...
```

**Nguyên nhân gốc:** Docker Hub rate limit cho anonymous pulls, hoặc ECR authentication hết hạn

**Khắc phục:** Đảm bảo ECR login trước pull trong deploy.sh

```bash
# Thêm vào deploy.sh trước docker compose pull:
echo "Logging in to ECR..."
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 772706200692.dkr.ecr.us-east-1.amazonaws.com

echo "Pulling latest images..."
docker compose -f docker-compose.${SERVICE}-service.yml pull
```

**Xác minh:**
```bash
# Kiểm tra ECR authentication
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 772706200692.dkr.ecr.us-east-1.amazonaws.com
# Nên thành công không lỗi
```

### Vấn đề 5: SSM Command hết giờ (Timeout)

**Error message:**
```
Command failed: Status: Failed
Timed out waiting for command execution
```

**Nguyên nhân gốc:** EC2 không đăng ký với SSM hoặc IAM role thiếu SSM permissions

**Khắc phục:** Xác minh EC2 có IAM role đúng

```bash
# Kiểm tra EC2 có đăng ký SSM không
aws ssm describe-instance-information --filters "Key=InstanceIds,Values=i-037a4cd636a68eb7e"

# Xác minh IAM role trên EC2
aws sts get-caller-identity
# Nên show EC2-Backend-Role

# Kiểm tra IAM policies
aws iam list-attached-role-policies --role-name EC2-Backend-Role
# Nên show AmazonSSMManagedInstanceCore
```

**Giải pháp:**
```bash
# Nếu role thiếu, stop instance và attach role
aws ec2 stop-instances --instance-ids i-037a4cd636a68eb7e
aws ec2 associate-iam-instance-profile \
  --instance-id i-037a4cd636a68eb7e \
  --iam-instance-profile Name=EC2-Backend-Role
aws ec2 start-instances --instance-ids i-037a4cd636a68eb7e
```

### Vấn đề 6: Secrets Manager Access Denied

**Error message:**
```
An error occurred (AccessDeniedException) when calling the GetSecretValue operation
```

**Nguyên nhân gốc:** EC2 IAM role thiếu Secrets Manager permissions

**Khắc phục:** Thêm SecretsManagerReadWrite policy vào EC2 role

```bash
# Xác minh policies hiện tại
aws iam list-attached-role-policies --role-name EC2-Backend-Role

# Nếu thiếu, attach policy
aws iam attach-role-policy \
  --role-name EC2-Backend-Role \
  --policy-arn arn:aws:iam::aws:policy/SecretsManagerReadWrite
```

### Vấn đề 7: Docker Build Fails - Missing Package

**Error message:**
```
exec /usr/local/bin/docker-entrypoint.sh: no such file or directory
```

**Nguyên nhân gốc:** Build context issues hoặc incorrect file permissions

**Khắc phục:** Đảm bảo file permissions đúng và build context

```dockerfile
# Trong Dockerfile, đảm bảo entrypoint có execute permissions
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["docker-entrypoint.sh"]
```

### Vấn đề 8: CodeBuild - Build Context Too Large

**Error message:**
```
failed to solve: failed to prepare context: context size exceeded
```

**Nguyên nhân gốc:** Build context chứa quá nhiều file

**Khắc phục:** Tối ưu .dockerignore và dùng multi-stage builds

```text
# .dockerignore
node_modules/
dist/
*.log
.git/
.github/
tests/
docs/
*.md
.dockerignore
Dockerfile
.gitignore
```

### Vấn đề 9: Container Không kết nối được Database

**Error message:**
```
Connection to 5432 refused. Is the server running?
```

**Nguyên nhân gốc:** Database container chưa healthy khi API service khởi động

**Khắc phục:** Dùng health checks và depends_on conditions

```yaml
# Trong docker-compose.yml
services:
  postgres:
    image: postgres:15-alpine
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

  api_service:
    depends_on:
      postgres:
        condition: service_healthy
```

### Vấn đề 10: CloudWatch Logs Không hiển thị

**Error message:**
```
No logs found in CloudWatch for build
```

**Nguyên nhân gốc:** Thiếu CloudWatch logs configuration hoặc permissions

**Khắc phục:** Bật logs trong CodeBuild và xác minh IAM role

```bash
# Kiểm tra CodeBuild project logs configuration
aws codebuild batch-get-projects --names build-api-service

# Xác minh CloudWatch logs permissions trong IAM role
aws iam get-role-policy --role-name codebuild-build-api-service-service-role
```

### Các lệnh Debug tham khảo

```bash
# Kiểm tra CodeBuild build details
aws codebuild batch-get-builds --ids build-id

# Kiểm tra SSM command status
aws ssm list-commands \
  --instance-id i-037a4cd636a68eb7e \
  --state-pending

# Xem SSM command output
aws ssm get-command-invocation \
  --command-id command-id \
  --instance-id i-037a4cd636a68eb7e

# Kiểm tra EC2 status
aws ec2 describe-instances \
  --instance-ids i-037a4cd636a68eb7e

# Xem Docker logs trên EC2
sudo docker logs api_service --tail 100

# Kiểm tra container networking
sudo docker network ls
sudo docker network inspect video-platform_video-platform
```

### Mẹo khắc phục sự cố chung

1. **Bắt đầu từ logs**: Luôn kiểm tra CodeBuild logs trước
2. **Kiểm tra IAM**: Nhiều vấn đề xuất phát từ missing permissions
3. **Xác minh network**: Đảm bảo security groups cho phép traffic cần thiết
4. **Test isolated**: Reproduce locally nếu có thể
5. **Dùng AWS Health**: Kiểm tra AWS service status

### Bước tiếp theo

Với kiến thức khắc phục sự cố, chuyển sang phần Next Steps để xem các mẹo tối ưu chi phí và cải tiến tương lai.