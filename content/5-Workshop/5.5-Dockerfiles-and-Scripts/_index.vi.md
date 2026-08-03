+++
title = "5.5. Dockerfiles và Scripts"
date = 2026-08-03
weight = 5
chapter = false
+++

## Dockerfiles và CI/CD Scripts (Tuần 9)

Phần này trình bày việc đóng gói hóa container cho cả hai microservice sử dụng Docker multi-stage builds và các script CI/CD điều khiển pipeline triển khai.

### 4.1 Dockerfile cho API Service

Multi-stage build cho ứng dụng NestJS để giảm thiểu kích thước image:

```dockerfile
# Builder stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npx prisma generate
RUN npm run build
RUN npm prune --omit=dev

# Runtime stage
FROM node:18-alpine
RUN apk add --no-cache ffmpeg openssl
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/prisma ./prisma
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
EXPOSE 3000
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["node", "dist/main.js"]
```

**Các khía cạnh chính của Dockerfile này:**
- **Multi-stage build**: Builder stage biên dịch TypeScript, runtime stage chỉ chứa các artifact production
- **Alpine base**: Kích thước image tối thiểu để deploy nhanh hơn
- **Prisma generation**: Client database được sinh ra trong quá trình build
- **FFmpeg installed**: Yêu cầu cho các tính năng xử lý video
- **Entrypoint script**: Xử lý database readiness và migrations

### 4.2 Dockerfile cho Search Service

Multi-stage build cho ứng dụng FastAPI:

```dockerfile
# Builder stage
FROM python:3.10-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip wheel --no-cache-dir --no-deps --wheel-dir /wheels -r requirements.txt

# Runtime stage
FROM python:3.10-slim
RUN apt-get update && apt-get install -y ffmpeg libpq5 && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY --from=builder /wheels /wheels
COPY requirements.txt .
RUN pip install --no-cache-dir --no-index --find-links=/wheels -r requirements.txt && rm -rf /wheels
COPY . .
RUN prisma generate --schema=prisma/schema.prisma
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
EXPOSE 8000
ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

**Các khía cạnh chính:**
- **Python wheel caching**: Wheel được build sẵn để deploy nhanh hơn
- **FFmpeg và libpq5**: Yêu cầu cho xử lý video và kết nối PostgreSQL
- **Prisma client**: Được sinh ra cho các thao tác database type-safe
- **Uvicorn ASGI server**: Server sẵn sàng production cho FastAPI

### 4.3 Script Entrypoint Docker

**Mục đích:** Chờ PostgreSQL sẵn sàng, chạy Prisma migrations, sau đó khởi động service

```bash
#!/bin/sh
set -e

echo "Waiting for PostgreSQL..."
timeout=60
while ! nc -z $DATABASE_URL_HOST $DATABASE_URL_PORT; do
  timeout=$((timeout - 1))
  if [ $timeout -le 0 ]; then
    echo "Timeout waiting for PostgreSQL"
    exit 1
  fi
  sleep 0.5
done

echo "PostgreSQL is ready. Running migrations..."
npx prisma migrate deploy

echo "Starting application..."
exec "$@"
```

**Cách hoạt động:**
1. **TCP probe**: Sử dụng netcat để kiểm tra khả dụng của PostgreSQL
2. **Timeout handling**: Fail sau 60 giây nếu database chưa sẵn sàng
3. **Migration execution**: Chạy Prisma migrations theo thứ tự
4. **Exec replacement**: Thay thế shell process bằng application

### 4.4 File Buildspec

#### buildspec-api.yml (Hướng dẫn CodeBuild cho API Service)

```yaml
version: 0.2

env:
  variables:
    ECR_REPO_NAME: api-service
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
      - cd api_service
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
      - aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=instanceids,Values=$EC2_INSTANCE_ID" --parameters '{"commands":["/app/deploy.sh api"]}'
```

#### Các biến môi trường bắt buộc trong CodeBuild

| Variable | Value |
|----------|-------|
| AWS_DEFAULT_REGION | us-east-1 |
| AWS_ACCOUNT_ID | 772706200692 |
| ECR_REPO_NAME | api-service (hoặc search-service) |
| DEPLOY_S3_BUCKET | videoplatform-deploy-artifacts-dsk |
| DEPLOY_S3_PREFIX | compose |
| EC2_INSTANCE_ID | i-037a4cd636a68eb7e |

### 4.5 Script triển khai EC2

**deploy.sh** chạy trên EC2, được trigger bởi SSM:

```bash
#!/bin/bash
set -e

SERVICE=$1  # "api" hoặc "search"
S3_BUCKET="${DEPLOY_S3_BUCKET:-videoplatform-deploy-artifacts-dsk}"
SECRET_ID="prod/backend/${SERVICE}-service"

echo "Downloading compose file from S3..."
aws s3 cp "s3://$S3_BUCKET/compose/docker-compose.${SERVICE}-service.yml" /app/compose/

echo "Fetching secrets from Secrets Manager..."
SECRET_JSON=$(aws secretsmanager get-secret-value --secret-id "$SECRET_ID" --query SecretString --output text)

echo "Writing .env file..."
python3 - <<'PYEOF'
import json, os
data = json.loads(os.environ["SECRET_JSON"])
with open(f"/app/compose/.env.{os.environ['SERVICE']}", "w", encoding="utf-8") as f:
    # Parse DATABASE_URL for entrypoint TCP probe
    db_url = data.get("DATABASE_URL", "")
    if "postgresql://" in db_url:
        host_port = db_url.split("@")[1].split("/")[0]
        host, port = host_port.split(":")
        f.write(f"DATABASE_URL_HOST={host}\n")
        f.write(f"DATABASE_URL_PORT={port}\n")
    
    for k, v in data.items():
        # Only quote values with special chars
        v_str = str(v).replace("\n", "\\n")
        if any(c in v_str for c in [" ", "#", '"', "\\"]):
            v_str = v_str.replace('"', '\\"')
            f.write(f'{k}="{v_str}"\n')
        else:
            f.write(f"{k}={v_str}\n")
PYEOF

echo "Logging in to ECR..."
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 772706200692.dkr.ecr.us-east-1.amazonaws.com

echo "Pulling latest images..."
cd /app/compose
docker compose -f docker-compose.${SERVICE}-service.yml pull

echo "Deploying containers..."
docker compose -f docker-compose.${SERVICE}-service.yml up -d

echo "Deployment complete for ${SERVICE}-service"
```

**Các tính năng chính:**
- **Secret injection**: Fetch secrets tại runtime từ Secrets Manager
- **Environment parsing**: Trích xuất host/port database cho entrypoint health checks
- **ECR authentication**: Đảm bảo Docker có thể pull từ private ECR
- **Compose-based deployment**: Sử dụng Docker Compose cho orchestration đa container

### 4.6 File Docker Compose

#### docker-compose.api-service.yml (đoạn trích)

```yaml
version: '3.8'

services:
  api_service:
    image: ${ECR_URL}/api-service:latest
    container_name: api_service
    restart: unless-stopped
    ports:
      - "3000:3000"
    env_file:
      - .env.api
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
      rabbitmq:
        condition: service_started
    networks:
      - video-platform
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - video-platform
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
    driver: local

networks:
  video-platform:
    driver: bridge
```

### Các vấn đề đã giải quyết trong quá trình triển khai

1. **Bug #1 (Tuần 10)**: Entrypoint bị bỏ qua bởi `.dockerignore`
   - **Root cause**: `docker-entrypoint.sh` nằm trong `.dockerignore`
   - **Cách sửa**: Loại bỏ `docker-entrypoint.sh` khỏi `.dockerignore`, thêm comment giải thích lý do nó cần thiết

2. **Bug #2 (Tuần 10)**: `requirements.txt` không được copy trong runtime stage
   - **Root cause**: Thiếu `COPY requirements.txt .` trước pip install
   - **Cách sửa**: Thêm `COPY requirements.txt .` trước pip install trong Dockerfile

3. **Bug #3 (Tuần 10)**: Prisma schema bị comment out
   - **Root cause**: Production schema bị comment trong `prisma/schema.prisma`
   - **Cách sửa**: Bỏ comment production schema, giữ development schema cho local dev

4. **Bug #4 (Tuần 10)**: Build context bị kẹt trong thư mục `search_service/`
   - **Root cause**: `cd $CODEBUILD_SRC_DIR` không được gọi trước S3 sync trong post_build
   - **Cách sửa**: Thêm `cd $CODEBUILD_SRC_DIR` trước lệnh `aws s3 sync`

### Best Practices đã áp dụng

- **Multi-stage builds**: Giảm thiểu kích thước image cuối cùng
- **No secrets in images**: Secrets được inject tại runtime
- **Health checks**: Xác minh sức khỏe container
- **Dependency ordering**: Service khởi động theo thứ tự đúng
- **Non-root containers**: Khi có thể
- **Resource limits**: Giới hạn memory và CPU

### Các bước tiếp theo

Sau khi đã tạo Dockerfiles và scripts, tiếp tục với CodeBuild Setup, nơi bạn sẽ cấu hình các build project và webhook.
