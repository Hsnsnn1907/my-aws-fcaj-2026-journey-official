+++
title = "5.11. Next Steps"
date = 2026-08-03
weight = 11
chapter = false
+++

## Next Steps

This section covers cost optimization strategies and future improvements for the CI/CD pipeline.

### Cost Optimization Tips

Reduce AWS costs while maintaining reliability:

#### 1. EC2 Schedule Start/Stop

Save ~50% on compute costs by stopping EC2 during off-hours:

```bash
# Manual stop/start
aws ec2 stop-instances --instance-ids i-037a4cd636a68eb7e
aws ec2 start-instances --instance-ids i-037a4cd636a68eb7e
```

**Cost impact**: ~$17/month → ~$8.50/month (if stopped 12 hours/day)

#### 2. EC2 Instance Scheduler (Automated)

Configure AWS Systems Manager for automated start/stop:

1. Navigate to Systems Manager → Instance Scheduler
2. Create schedule: Weekdays 8AM-6PM EST
3. Associate with EC2 instance

**Schedule configuration:**
- **Start**: 08:00 (weekdays)
- **Stop**: 18:00 (weekdays)
- **Timezone**: EST (UTC-5)
- **Exclude dates**: Holidays

#### 3. S3 Lifecycle Policy

Delete old compose files to reduce storage costs:

```json
{
  "Rules": [
    {
      "ID": "DeleteOldComposeFiles",
      "Status": "Enabled",
      "Filter": {
        "Prefix": "compose/"
      },
      "Expiration": {
        "Days": 30
      },
      "Transitions": [
        {
          "Days": 7,
          "StorageClass": "GLACIER"
        }
      ]
    }
  ]
}
```

**Cost impact**: Reduce S3 storage from ~$0.50 → ~$0.10/month

#### 4. ECR Lifecycle Policy

Keep only recent images to reduce storage:

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 10 images",
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

**Cost impact**: Reduce ECR storage from ~$0.50 → ~$0.25/month

#### 5. Right-Size EC2 Instance

Monitor CloudWatch metrics and downsize if underutilized:

```bash
# Check average CPU utilization
aws cloudwatch get-metric-statistics \
  --namespace AWS/EC2 \
  --metric-name CPUUtilization \
  --dimensions Name=InstanceId,Value=i-037a4cd636a68eb7e \
  --start-time $(date -d '30 days ago' --iso-8601=seconds) \
  --end-time $(date --iso-8601=seconds) \
  --period 86400 \
  --statistic Average

# If CPU < 20%, consider t2.small
```

**Cost impact**: t2.medium ($17) → t2.small ($8.50/month)

#### 6. Spot Instances (Non-Production)

For development/staging environments:

- **Savings**: Up to 70% compared to on-demand
- **Trade-off**: Can be interrupted with 2-minute notice
- **Use case**: Development environments, CI/CD runners

#### 7. Reserved Instances

For stable production workloads:

- **Commitment**: 1-year or 3-year term
- **Savings**: ~40% compared to on-demand
- **Requirements**: Consistent, predictable usage

### Future Improvements

Enhance the CI/CD pipeline with these advanced features:

#### Multi-AZ Deployment

Deploy across availability zones for high availability:

- **Current**: Single EC2 instance in one AZ
- **Target**: 2+ instances across AZs with load balancer
- **Benefits**: Fault tolerance, improved availability
- **AWS services**: Application Load Balancer, Auto Scaling

**Architecture:**
```
[ALB] --> [EC2-AZ1] --> [Containers]
      --> [EC2-AZ2] --> [Containers]
```

#### Application Load Balancer

Replace direct EC2 access with ALB:

- **SSL termination**: Managed certificates via ACM
- **Health checks**: Automatic instance health detection
- **Traffic distribution**: Round-robin across instances
- **Security**: WAF integration available

**Implementation:**
```yaml
# Future enhancement
alb:
  type: aws_alb
  name: video-platform-alb
  listeners:
    - port: 443
      protocol: HTTPS
      certificate: arn:aws:acm:...
  target_groups:
    - name: api-target
      port: 3000
      health_check:
        path: /health
```

#### RDS instead of Containerized Database

Migrate from containerized PostgreSQL to Amazon RDS:

**Benefits:**
- Automated backups and point-in-time recovery
- Automatic failover for Multi-AZ
- Automated patching
- Better durability and performance

**Migration steps:**
1. Create RDS PostgreSQL instance
2. Set up replication from containerized DB
3. Migrate data during maintenance window
4. Update application connection strings
5. Decommission containerized database

#### ElastiCache for Redis

Replace containerized Redis with Amazon ElastiCache:

- **Managed service**: Automatic failover, patching
- **Scalability**: Easy vertical scaling
- **Monitoring**: Enhanced CloudWatch metrics
- **Security**: Encryption at rest and in transit

#### ECS/Fargate instead of EC2

Migrate to serverless containers:

**Benefits:**
- No EC2 management
- Automatic scaling
- Pay per use
- Better integration with AWS services

**Migration effort:**
- Rewrite deployment scripts for ECS
- Update CodeBuild buildspec
- Modify IAM roles for ECS tasks
- Set up ECS service with desired count

#### Infrastructure as Code

Implement AWS CDK or Terraform:

**Current state:** Manual AWS Console setup
**Target state:** Infrastructure as Code

**Advantages:**
- Reproducible deployments
- Version control for infrastructure
- Environment parity
- Disaster recovery

**Example (CDK):**
```typescript
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

export class VideoPlatformStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // ECR Repository
    new ecr.Repository(this, 'ApiServiceRepo', {
      repositoryName: 'api-service',
      imageScanOnPush: true,
    });

    // EC2 Instance
    new ec2.Instance(this, 'Production', {
      instanceType: ec2.InstanceType.of(
        ec2.InstanceClass.T2,
        ec2.InstanceSize.MEDIUM
      ),
      vpc: /* ... */,
    });
  }
}
```

#### Blue/Green Deployments

Implement zero-downtime deployments:

- **Current**: In-place deployment with brief downtime
- **Target**: Blue/green with instant cutover

**AWS services:** CodeDeploy, Application Load Balancer

**Benefits:**
- Zero downtime deployments
- Instant rollback capability
- Traffic shifting strategies

#### Integration Tests in CodeBuild

Add automated testing before deployment:

```yaml
# In buildspec.yml, add test phase
phases:
  pre_build:
    commands:
      - echo "Running unit tests..."
      - cd api_service && npm test
      - echo "Unit tests passed"

  build:
    commands:
      - echo "Building Docker image..."
      # ... existing build commands
```

**Test types:**
- Unit tests (Jest, pytest)
- Integration tests (test database connectivity)
- Security scans (Trivy, security headers)

#### Canary Deployments

Gradual rollout for new versions:

1. Deploy to 10% of traffic
2. Monitor metrics and errors
3. Gradually increase to 50%, 100%
4. Automatic rollback if error rate increases

**AWS services:** CodeDeploy, CloudWatch Alarms

#### Service Mesh (AWS App Mesh)

Advanced traffic management:

- **Features**: Traffic splitting, retries, timeouts
- **Observability**: Distributed tracing (X-Ray)
- **Security**: mTLS between services

#### API Gateway

Centralized API management:

- **Features**: Rate limiting, API keys, throttling
- **Authentication**: Cognito, Lambda authorizers
- **Monitoring**: Detailed API metrics

#### X-Ray Tracing

Distributed tracing for debugging:

- **Visualize**: Request flow across services
- **Debug**: Identify latency bottlenecks
- **Integrates**: Lambda, ECS, API Gateway

### Implementation Roadmap

| Priority | Feature | Effort | Impact |
|----------|---------|--------|--------|
| High | S3 Lifecycle Policy | Low | Cost reduction |
| High | ECR Lifecycle Policy | Low | Cost reduction |
| Medium | EC2 Scheduler | Low | Cost reduction |
| Medium | Integration Tests | Medium | Quality |
| Medium | Infrastructure as Code | High | Maintainability |
| Low | Multi-AZ Deployment | High | Reliability |
| Low | RDS Migration | Medium | Reliability |

### Recommended Next Steps

1. **Immediately**: Implement S3 and ECR lifecycle policies
2. **This month**: Set up EC2 scheduler for off-hours
3. **Next quarter**: Add integration tests to CodeBuild
4. **Future**: Plan infrastructure-as-code migration

### Conclusion

This workshop has provided a solid foundation for CI/CD automation on AWS. The implemented pipeline reduces deployment time by 83% and establishes enterprise-grade security practices. The future improvements outlined here will further enhance reliability, reduce costs, and improve maintainability as the platform scales.

### Workshop Summary

| Metric | Before | After |
|--------|--------|-------|
| Deployment time | 20-30 min | 4-5 min |
| Monthly cost | - | ~$22 |
| AWS services used | - | 8 |
| Total builds | - | 42+ |
| Success rate | - | 100% (after Week 10 fixes) |

**Repository:** [khiem918/VideoPlatformServer](https://github.com/khiem918/VideoPlatformServer)  
**AWS Account:** 772706200692 (us-east-1)  
**Production URL:** http://98.81.144.110:3000/graphql