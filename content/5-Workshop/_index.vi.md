+++
title = "5. Workshop"
date = 2026-07-27
weight = 5
chapter = false
+++## 1) Phạm vi workshop

Workshop này mô tả quá trình chuyển từ deploy thủ công sang luồng CI/CD AWS-native cho 2 backend service:

- `api_service` (NestJS GraphQL + Prisma)
- `search_service` (FastAPI + Qdrant + RabbitMQ consumer)

## 2) Tổng quan kiến trúc

![Sơ đồ kiến trúc AWS VideoPlatform](/my-aws-fcaj-2026-journey-official/images/5-Workshop/architecture_diagram.png)

### Các dịch vụ AWS chính

| Dịch vụ | Vai trò trong dự án |
|---|---|
| EC2 | Máy chủ runtime chạy toàn bộ container |
| ECR | Private registry cho image |
| CodeBuild | Tự động build/test/push |
| S3 | Lưu deploy artifact (compose files) |
| SSM | Trigger deploy script từ xa trên EC2 |
| Secrets Manager | Nguồn secret runtime |
| CloudWatch | Logs, alarms, dashboard |
| IAM | Phân quyền bằng role/policy |

## 3) Luồng hạ tầng và ứng dụng

1. Push code lên `feat/CI-CD`.
2. Path filter gọi đúng project CodeBuild.
3. Build image và push lên ECR.
4. Sync compose file lên S3.
5. Trigger deploy EC2 qua SSM.
6. Pull image, inject secret runtime, chạy `docker compose up -d`.
7. Verify health và monitor qua CloudWatch.

## 4) Code snippet

### Reset working directory ở post-build

```yaml
post_build:
  commands:
    - docker push $REPO_URI:latest
    - docker push $REPO_URI:$IMAGE_TAG
    - cd $CODEBUILD_SRC_DIR
    - aws s3 sync docker/ s3://$DEPLOY_S3_BUCKET/$DEPLOY_S3_PREFIX/ --delete
    - aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=instanceids,Values=$EC2_INSTANCE_ID" --parameters '{"commands":["/app/deploy.sh search"]}'
```

### Parse secret JSON an toàn trên EC2

```bash
SECRET_JSON="$(aws secretsmanager get-secret-value --secret-id "$SECRET_ID" --query SecretString --output text)"
python3 - <<'PYEOF'
import json, os
data = json.loads(os.environ["SECRET_JSON"])
with open("/opt/deploy/.env", "w", encoding="utf-8") as f:
    for k, v in data.items():
        f.write(f"{k}={str(v).replace(chr(10), '\\n')}\n")
PYEOF
```

### Chờ dependency trước khi migrate

```bash
while ! nc -z postgres 5432; do
  sleep 0.5
done
npx prisma migrate deploy
exec "$@"
```

## 5) Hình ảnh minh họa

- Sơ đồ kiến trúc: `../images/workshop/architecture-overview.svg`
- Sơ đồ pipeline CI/CD: `../images/workshop/cicd-flow.svg`

## 6) File đính kèm

- Dockerfile:
  - `sever code/VideoPlatformServer/api_service/Dockerfile`
  - `sever code/VideoPlatformServer/search_service/Dockerfile`
- Buildspec:
  - `sever code/VideoPlatformServer/buildspec-api.yml`
  - `sever code/VideoPlatformServer/buildspec-search.yml`
- Script triển khai:
  - `sever code/VideoPlatformServer/script/deploy.sh`
  - `sever code/VideoPlatformServer/script/ec2-bootstrap.sh`
  - `sever code/VideoPlatformServer/script/sync-compose-to-s3.sh`
- Compose:
  - `sever code/VideoPlatformServer/docker/docker-compose.api-service.yml`
  - `sever code/VideoPlatformServer/docker/docker-compose.search-service.yml`
- gRPC contract:
  - `sever code/VideoPlatformServer/proto/video_metadata.proto`

## 7) Checklist bảo mật và ổn định

- Dùng IAM Role trên EC2 thay cho access key dài hạn
- Bật IMDSv2 bắt buộc
- Secret chỉ inject runtime từ Secrets Manager
- Bật ECR image scan-on-push
- Có CloudWatch alarm và backup strategy

