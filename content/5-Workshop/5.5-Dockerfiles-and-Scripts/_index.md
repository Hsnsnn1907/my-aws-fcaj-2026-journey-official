+++
title = "5.5. Dockerfiles and Scripts"
date = 2026-08-03
weight = 5
chapter = false
+++

## Dockerfiles and CI/CD Scripts (Week 9)

This section covers the containerization of both microservices using multi-stage Docker builds and the CI/CD scripts that drive the deployment pipeline.

### 4.1 API Service Dockerfile

Multi-stage build for NestJS application to minimize image size:

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

**Key aspects of this Dockerfile:**
- **Multi-stage build**: Builder stage compiles TypeScript, runtime stage contains only production artifacts
- **Alpine base**: Minimal image size for faster deployments
- **Prisma generation**: Database client generated during build
- **FFmpeg installed**: Required for video processing features
- **Entrypoint script**: Handles database readiness and migrations

### 4.2 Search Service Dockerfile

Multi-stage build for FastAPI application:

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

**Key aspects:**
- **Python wheel caching**: Pre-built wheels for faster deployments
- **FFmpeg and libpq5**: Required for video processing and PostgreSQL connectivity
- **Prisma client**: Generated for type-safe database operations
- **Uvicorn ASGI server**: Production-ready server for FastAPI

### 4.3 Docker Entrypoint Script

**Purpose:** Wait for PostgreSQL readiness, run Prisma migrations, then start the service

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

**How it works:**
1. **TCP probe**: Uses netcat to check PostgreSQL availability
2. **Timeout handling**: Fails after 60 seconds if database not ready
3. **Migration execution**: Runs Prisma migrations in order
4. **Exec replacement**: Replaces shell process with application

### 4.4 Buildspec Files

#### buildspec-api.yml (CodeBuild instructions for API Service)

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

#### Required Environment Variables in CodeBuild

| Variable | Value |
|----------|-------|
| AWS_DEFAULT_REGION | us-east-1 |
| AWS_ACCOUNT_ID | 772706200692 |
| ECR_REPO_NAME | api-service (or search-service) |
| DEPLOY_S3_BUCKET | videoplatform-deploy-artifacts-dsk |
| DEPLOY_S3_PREFIX | compose |
| EC2_INSTANCE_ID | i-037a4cd636a68eb7e |

### 4.5 EC2 Deployment Script

**deploy.sh** runs on EC2, triggered by SSM:

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

**Key features:**
- **Secret injection**: Fetches secrets at runtime from Secrets Manager
- **Environment parsing**: Extracts database host/port for entrypoint health checks
- **ECR authentication**: Ensures Docker can pull from private ECR
- **Compose-based deployment**: Uses Docker Compose for multi-container orchestration

### 4.6 Docker Compose Files

#### docker-compose.api-service.yml (excerpt)

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

### Issues Resolved During Implementation

1. **Bug #1 (Week 10)**: Entrypoint ignored by `.dockerignore`
   - **Root cause**: `docker-entrypoint.sh` was in `.dockerignore`
   - **Fix**: Removed `docker-entrypoint.sh` from `.dockerignore`, added comment explaining why it's needed

2. **Bug #2 (Week 10)**: `requirements.txt` not copied in runtime stage
   - **Root cause**: Missing `COPY requirements.txt .` before pip install
   - **Fix**: Added `COPY requirements.txt .` before pip install in Dockerfile

3. **Bug #3 (Week 10)**: Prisma schema commented out
   - **Root cause**: Production schema was commented in `prisma/schema.prisma`
   - **Fix**: Uncommented production schema, kept development schema for local dev

4. **Bug #4 (Week 10)**: Build context stuck in `search_service/` directory
   - **Root cause**: `cd $CODEBUILD_SRC_DIR` not called before S3 sync in post_build
   - **Fix**: Added `cd $CODEBUILD_SRC_DIR` before `aws s3 sync` command

### Best Practices Applied

- **Multi-stage builds**: Minimize final image size
- **No secrets in images**: Secrets injected at runtime
- **Health checks**: Container health verification
- **Dependency ordering**: Services start in correct order
- **Non-root containers**: Where possible
- **Resource limits**: Memory and CPU constraints

### Next Steps

With Dockerfiles and scripts created, proceed to CodeBuild Setup where you'll configure the build projects and webhooks.