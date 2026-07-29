+++
title = "Workshop"
date = 2026-07-27
weight = 5
chapter = false
+++## 1) Workshop scope

This workshop documents how the project moved from manual deployment to an AWS-native CI/CD flow for two backend services:

- `api_service` (NestJS GraphQL + Prisma)
- `search_service` (FastAPI + Qdrant + RabbitMQ consumer)

## 2) Architecture overview

![Architecture overview](../images/workshop/architecture-overview.svg)

![CI/CD flow](../images/workshop/cicd-flow.svg)

### Core AWS services used

| Service | Usage in project |
|---|---|
| EC2 | Runtime host for all containers |
| ECR | Private registry for service images |
| CodeBuild | Build/test/push automation |
| S3 | Stores deploy compose artifacts |
| SSM | Remotely triggers deploy script on EC2 |
| Secrets Manager | Runtime secret source |
| CloudWatch | Logs, alarms, dashboard |
| IAM | Access control via roles/policies |

## 3) Infrastructure and application flow

1. Push code to `feat/CI-CD`.
2. Path filter triggers proper CodeBuild project.
3. Build image and push to ECR.
4. Sync compose files to S3.
5. Trigger EC2 deploy through SSM.
6. Pull image, inject runtime secrets, run `docker compose up -d`.
7. Verify health and monitor via CloudWatch.

## 4) Code snippets

### Buildspec post-build working directory reset

```yaml
post_build:
  commands:
    - docker push $REPO_URI:latest
    - docker push $REPO_URI:$IMAGE_TAG
    - cd $CODEBUILD_SRC_DIR
    - aws s3 sync docker/ s3://$DEPLOY_S3_BUCKET/$DEPLOY_S3_PREFIX/ --delete
    - aws ssm send-command --document-name "AWS-RunShellScript" --targets "Key=instanceids,Values=$EC2_INSTANCE_ID" --parameters '{"commands":["/app/deploy.sh search"]}'
```

### EC2 deploy secret parsing (structured JSON)

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

### Docker entrypoint dependency gate

```bash
while ! nc -z postgres 5432; do
  sleep 0.5
done
npx prisma migrate deploy
exec "$@"
```

## 5) Illustrative images

- Architecture overview: `../images/workshop/architecture-overview.svg`
- CI/CD pipeline map: `../images/workshop/cicd-flow.svg`

## 6) File attachments

- Dockerfiles:
  - `sever code/VideoPlatformServer/api_service/Dockerfile`
  - `sever code/VideoPlatformServer/search_service/Dockerfile`
- Buildspecs:
  - `sever code/VideoPlatformServer/buildspec-api.yml`
  - `sever code/VideoPlatformServer/buildspec-search.yml`
- Deployment scripts:
  - `sever code/VideoPlatformServer/script/deploy.sh`
  - `sever code/VideoPlatformServer/script/ec2-bootstrap.sh`
  - `sever code/VideoPlatformServer/script/sync-compose-to-s3.sh`
- Compose files:
  - `sever code/VideoPlatformServer/docker/docker-compose.api-service.yml`
  - `sever code/VideoPlatformServer/docker/docker-compose.search-service.yml`
- gRPC contract:
  - `sever code/VideoPlatformServer/proto/video_metadata.proto`

## 7) Security and reliability checklist

- IAM Role on EC2 instead of long-term keys
- IMDSv2 required
- Secrets from Secrets Manager only at runtime
- ECR image scan-on-push enabled
- CloudWatch alarms and backup strategy in place

