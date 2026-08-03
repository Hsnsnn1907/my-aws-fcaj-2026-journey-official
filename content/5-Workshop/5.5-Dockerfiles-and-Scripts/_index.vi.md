+++
title = "Dockerfiles và Scripts"
date = 2026-08-03
weight = 5
chapter = false
+++

## Dockerfiles và CI/CD Scripts (Tuần 9)

Phần này bao gồm containerization của hai microservices và CI/CD scripts.

### Dockerfile API Service

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

### Dockerfile Search Service

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

### Docker Entrypoint Script

Chờ PostgreSQL sẵn sàng, chạy migrations, rồi start service:

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

### Buildspec Files

#### buildspec-api.yml

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

### EC2 Deployment Script (deploy.sh)

```bash
#!/bin/bash
set -e

SERVICE=$1  # "api" or "search"
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
    db_url = data.get("DATABASE_URL", "")
    if "postgresql://" in db_url:
        host_port = db_url.split("@")[1].split("/")[0]
        host, port = host_port.split(":")
        f.write(f"DATABASE_URL_HOST={host}\n")
        f.write(f"DATABASE_URL_PORT={port}\n")
    for k, v in data.items():
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

### Các Bug đã khắc phục

1. **Bug #1**: Entrypoint bị `.dockerignore` loại bỏ → Xóa dòng khỏi `.dockerignore`
2. **Bug #2**: `requirements.txt` không được copy trong runtime stage → Thêm COPY requirements.txt
3. **Bug #3**: Prisma schema bị comment → Uncomment schema production
4. **Bug #4**: Build context bị stuck trong `search_service/` → Thêm `cd $CODEBUILD_SRC_DIR` trước S3 sync

Với Dockerfiles và scripts đã tạo, hãy đi đến Thiết lập CodeBuild.