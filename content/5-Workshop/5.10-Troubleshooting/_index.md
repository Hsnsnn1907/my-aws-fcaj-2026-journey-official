+++
title = "Troubleshooting"
date = 2026-08-03
weight = 10
chapter = false
+++

## Troubleshooting Guide

This section covers common issues encountered during CI/CD pipeline implementation and their solutions.

### Issue 1: CodeBuild Fails with "docker-entrypoint.sh not found"

**Error message:**
```
COPY failed: file not found in context: path not found: docker-entrypoint.sh
```

**Root cause:** The `docker-entrypoint.sh` file was excluded by `.dockerignore`

**Fix:** Remove `docker-entrypoint.sh` from `.dockerignore`

Before (in .dockerignore):
```
node_modules
dist
docker-entrypoint.sh  <-- Remove this line
```

After:
```
node_modules
dist
# docker-entrypoint.sh - kept in context for container startup
```

**Verification:**
```bash
# Rebuild in CodeBuild
# Entrypoint should now be included in Docker context
```

### Issue 2: Prisma Migrations Fail in Container

**Error message:**
```
Error: P1001: Can't reach database server at postgresql://postgres:5432
```

**Root cause:** The entrypoint script needs database host/port parsed from DATABASE_URL

**Fix:** Parse DATABASE_URL in deploy.sh and write environment variables

```bash
# In deploy.sh, add Python script to parse DATABASE_URL:
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

**Verification:**
```bash
# Check env file on EC2
cat /app/compose/.env.api
# Should contain DATABASE_URL_HOST and DATABASE_URL_PORT
```

### Issue 3: S3 Sync Fails with "docker/ does not exist"

**Error message:**
```
fatal error: cannot start container: could not get containerd socket:
```

**Root cause:** The `cd search_service` command in BUILD phase persists into POST_BUILD, making the S3 sync path relative to wrong directory

**Fix:** Add `cd $CODEBUILD_SRC_DIR` before S3 sync command

```yaml
post_build:
  commands:
    - echo "Pushing image to ECR..."
    - docker push $REPO_URI:latest
    - cd $CODEBUILD_SRC_DIR  # <-- Add this line
    - echo "Syncing compose files to S3..."
    - aws s3 sync docker/ s3://$DEPLOY_S3_BUCKET/$DEPLOY_S3_PREFIX/ --delete
```

**Verification:**
```bash
# Check S3 bucket for compose files
aws s3 ls s3://videoplatform-deploy-artifacts-dsk/compose/
```

### Issue 4: EC2 Deployment Stuck "Pulling image..."

**Error message:**
```
Pulling image (latest) from 772706200692.dkr.ecr.us-east-1.amazonaws.com/api-service...
```

**Root cause:** Docker Hub rate limit for anonymous pulls, or ECR authentication expired

**Fix:** Ensure ECR login before pull in deploy.sh

```bash
# Add to deploy.sh before docker compose pull:
echo "Logging in to ECR..."
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 772706200692.dkr.ecr.us-east-1.amazonaws.com

echo "Pulling latest images..."
docker compose -f docker-compose.${SERVICE}-service.yml pull
```

**Verification:**
```bash
# Check ECR authentication
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 772706200692.dkr.ecr.us-east-1.amazonaws.com
# Should succeed without error
```

### Issue 5: SSM Command Times Out

**Error message:**
```
Command failed: Status: Failed
Timed out waiting for command execution
```

**Root cause:** EC2 not registered with SSM or IAM role missing SSM permissions

**Fix:** Verify EC2 has correct IAM role

```bash
# Check if EC2 is registered with SSM
aws ssm describe-instance-information --filters "Key=InstanceIds,Values=i-037a4cd636a68eb7e"

# Verify IAM role on EC2
aws sts get-caller-identity
# Should show EC2-Backend-Role

# Check IAM policies
aws iam list-attached-role-policies --role-name EC2-Backend-Role
# Should show AmazonSSMManagedInstanceCore
```

**Resolution:**
```bash
# If role missing, stop instance and attach role
aws ec2 stop-instances --instance-ids i-037a4cd636a68eb7e
aws ec2 associate-iam-instance-profile \
  --instance-id i-037a4cd636a68eb7e \
  --iam-instance-profile Name=EC2-Backend-Role
aws ec2 start-instances --instance-ids i-037a4cd636a68eb7e
```

### Issue 6: Secrets Manager Access Denied

**Error message:**
```
An error occurred (AccessDeniedException) when calling the GetSecretValue operation
```

**Root cause:** EC2 IAM role missing Secrets Manager permissions

**Fix:** Add SecretsManagerReadWrite policy to EC2 role

```bash
# Verify current policies
aws iam list-attached-role-policies --role-name EC2-Backend-Role

# If missing, attach policy
aws iam attach-role-policy \
  --role-name EC2-Backend-Role \
  --policy-arn arn:aws:iam::aws:policy/SecretsManagerReadWrite
```

### Issue 7: Docker Build Fails - Missing Package

**Error message:**
```
exec /usr/local/bin/docker-entrypoint.sh: no such file or directory
```

**Root cause:** Build context issues or incorrect file permissions

**Fix:** Ensure proper file permissions and build context

```dockerfile
# In Dockerfile, ensure entrypoint has execute permissions
COPY docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh
ENTRYPOINT ["docker-entrypoint.sh"]
```

### Issue 8: CodeBuild - Build Context Too Large

**Error message:**
```
failed to solve: failed to prepare context: context size exceeded
```

**Root cause:** Build context contains too many files

**Fix:** Optimize .dockerignore and use multi-stage builds

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

### Issue 9: Container Cannot Connect to Database

**Error message:**
```
Connection to 5432 refused. Is the server running?
```

**Root cause:** Database container not healthy when API service starts

**Fix:** Use health checks and depends_on conditions

```yaml
# In docker-compose.yml
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

### Issue 10: CloudWatch Logs Not Appearing

**Error message:**
```
No logs found in CloudWatch for build
```

**Root cause:** Missing CloudWatch logs configuration or permissions

**Fix:** Enable logs in CodeBuild and verify IAM role

```bash
# Check CodeBuild project logs configuration
aws codebuild batch-get-projects --names build-api-service

# Verify CloudWatch logs permissions in IAM role
aws iam get-role-policy --role-name codebuild-build-api-service-service-role
```

### Debug Commands Reference

```bash
# Check CodeBuild build details
aws codebuild batch-get-builds --ids build-id

# Check SSM command status
aws ssm list-commands \
  --instance-id i-037a4cd636a68eb7e \
  --state-pending

# View SSM command output
aws ssm get-command-invocation \
  --command-id command-id \
  --instance-id i-037a4cd636a68eb7e

# Check EC2 status
aws ec2 describe-instances \
  --instance-ids i-037a4cd636a68eb7e

# View Docker logs on EC2
sudo docker logs api_service --tail 100

# Check container networking
sudo docker network ls
sudo docker network inspect video-platform_video-platform
```

### General Troubleshooting Tips

1. **Start with logs**: Always check CodeBuild logs first
2. **Check IAM**: Many issues stem from missing permissions
3. **Verify network**: Ensure security groups allow required traffic
4. **Test in isolation**: Reproduce locally if possible
5. **Use AWS Health**: Check AWS service status

### Next Steps

With troubleshooting knowledge in place, proceed to Next Steps for cost optimization tips and future improvements.