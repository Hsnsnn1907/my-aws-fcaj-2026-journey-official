+++
title = "Production Deployment"
date = 2026-08-03
weight = 7
chapter = false
+++

## Production Deployment (Weeks 11-12)

This section covers the complete production deployment workflow, verification procedures, and rollback capabilities for the CI/CD pipeline.

### 6.1 Full Pipeline Test

Push a code change to test the complete deployment pipeline:

```bash
# Make a code change
git add api_service/src/video/video.service.ts
git commit -m "feat: add video transcoding queue"
git push origin feat/CI-CD
```

**Complete pipeline execution timeline:**

| Step | Duration | Action |
|------|----------|--------|
| 1 | < 5 sec | GitHub webhook triggers CodeBuild |
| 2 | 2-3 min | CodeBuild builds Docker image |
| 3 | 30 sec | Push to ECR |
| 4 | 5 sec | Sync compose files to S3 |
| 5 | 10 sec | SSM triggers EC2 deploy.sh |
| 6 | 1 min | EC2 pulls image and restarts containers |
| **Total** | **~4-5 min** | Full deployment |

**Total time: ~4-5 minutes** (vs. manual deployment: 20-30 minutes)

![Production Deployment](/my-aws-fcaj-2026-journey-official/images/5-Workshop/12_ec2_production_deployment.png)

### 6.2 Verification Checklist

After deployment, verify all components are functioning correctly:

#### Infrastructure Verification
- [ ] **Services healthy**: `docker ps` shows all containers running
- [ ] **No crash loops**: Container restart count is low (0-1)
- [ ] **Resource usage**: CPU and memory within expected ranges
- [ ] **Network connectivity**: Services can reach each other

#### Application Verification
- [ ] **API responds**: `curl http://98.81.144.110:3000/graphql`
- [ ] **Search service connected**: Check consumer logs for RabbitMQ connection
- [ ] **Database migrations applied**: Check Prisma migrate logs
- [ ] **Secrets loaded correctly**: Verify environment variables inside containers

#### AWS Service Verification
- [ ] **CloudWatch metrics**: CPU, memory, network within expected ranges
- [ ] **S3 artifacts**: Verify compose files synced correctly
- [ ] **ECR images**: Latest image pushed with correct tags
- [ ] **SSM commands**: Command history shows successful execution

#### Verification Commands

```bash
# Check all containers are running
sudo docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Verify API service is responding
curl -s http://localhost:3000/graphql -H "Content-Type: application/json" \
  -d '{"query":"{ __typename }"}'

# Check search service health
curl http://localhost:8000/health

# View recent logs for any errors
sudo docker logs --tail 50 api_service
sudo docker logs --tail 50 search_service

# Verify environment variables are loaded
sudo docker exec api_service env | grep -E "(DATABASE_URL|JWT_SECRET)"

# Check CloudWatch metrics in console
# Verify CPU, memory, and network metrics
```

### 6.3 Rollback Procedure

If deployment fails or issues are detected, execute rollback:

#### Quick Rollback Steps

```bash
# On EC2 instance
cd /app/compose

# Identify previous working version
docker images 772706200692.dkr.ecr.us-east-1.amazonaws.com/api-service

# Or use Docker Compose rollback
docker compose -f docker-compose.api-service.yml down
docker compose -f docker-compose.api-service.yml up -d

# Verify rollback was successful
sudo docker ps
curl http://localhost:3000/health
```

#### Rollback via Previous ECR Image

```bash
# List available image tags
aws ecr list-images --repository-name api-service --region us-east-1

# Pull previous image
docker pull 772706200692.dkr.ecr.us-east-1.amazonaws.com/api-service:<previous-tag>

# Update compose file with previous tag
# Edit docker-compose.api-service.yml to use specific tag

# Redeploy
docker compose -f docker-compose.api-service.yml up -d
```

#### ECR Lifecycle Policy for Rollback

Configure lifecycle policy to keep previous images:

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 10 images for rollback",
      "selection": {
        "tagStatus": "any",
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

### 6.4 Deployment Strategies

#### Blue/Green Deployment (Future Enhancement)

For zero-downtime deployments:
1. Deploy new version alongside existing version
2. Test new version with production traffic
3. Switch traffic to new version
4. Keep old version for quick rollback

#### Canary Deployment (Future Enhancement)

Gradual rollout strategy:
1. Deploy to 10% of traffic initially
2. Monitor metrics and errors
3. Gradually increase traffic
4. Full deployment or automatic rollback

#### Blue/Green with Application Load Balancer

```yaml
# Future enhancement - ALB configuration
services:
  api_service_green:
    image: ${ECR_URL}/api-service:green
    # ...
  api_service_blue:
    image: ${ECR_URL}/api-service:blue
    # ...
```

### 6.5 Deployment Best Practices

1. **Monitor during deployment**: Watch logs and metrics throughout
2. **Health checks**: Configure and rely on container health checks
3. **Graceful shutdown**: Allow containers to complete requests before stopping
4. **Notification setup**: Configure alerts for deployment failures
5. **Documentation**: Document any issues and fixes encountered

### 6.6 Post-Deployment Checklist

After every deployment:

```bash
# 1. Verify all services are healthy
sudo docker-compose -f docker-compose.api-service.yml ps
sudo docker-compose -f docker-compose.search-service.yml ps

# 2. Check for errors in logs
sudo docker logs api_service 2>&1 | grep -i error | tail -20
sudo docker logs search_service 2>&1 | grep -i error | tail -20

# 3. Test critical endpoints
curl -f http://localhost:3000/health || exit 1
curl -f http://localhost:8000/health || exit 1

# 4. Verify database connectivity
sudo docker exec api_service npx prisma db ping

# 5. Check CloudWatch for any alarms
# Review: CPU, Memory, Disk, Network metrics
```

### Common Deployment Issues

| Issue | Cause | Solution |
|-------|-------|----------|
| Container won't start | Missing secret | Verify Secrets Manager access |
| Database connection failed | Network issue | Check Docker network configuration |
| Port already in use | Previous container running | Clean up with `docker down` |
| Image pull fails | ECR authentication expired | Re-authenticate with ECR |

### Deployment Automation

Set up automated deployment notifications:

```bash
# Add to deploy.sh for notifications
if [ $? -eq 0 ]; then
    # Send success notification (Slack, SNS, etc.)
    aws sns publish --topic-arn arn:aws:sns:us-east-1:772706200692:deployment-notifications \
        --message "Deployment successful: ${SERVICE}-service"
else
    # Send failure notification
    aws sns publish --topic-arn arn:aws:sns:us-east-1:772706200692:deployment-notifications \
        --message "Deployment FAILED: ${SERVICE}-service"
fi
```

### Next Steps

With production deployment configured, proceed to Monitoring where you'll set up CloudWatch logs, alarms, and cost monitoring.