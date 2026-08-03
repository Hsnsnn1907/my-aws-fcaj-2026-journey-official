+++
title = "Security"
date = 2026-08-03
weight = 9
chapter = false
+++

## Security Best Practices

This section covers security best practices for securing the CI/CD pipeline, AWS infrastructure, and containerized applications.

### 8.1 Secrets Management

Implementing secure secrets management throughout the pipeline:

#### Best Practices Checklist

- **No secrets in code**: Ensure no hardcoded secrets in source code
- **Secrets stored in Secrets Manager**: All secrets in AWS Secrets Manager
- **IAM role-based access**: No long-term credentials, use IAM roles
- **Runtime secret injection**: Secrets fetched at runtime only
- **Encryption at rest**: Use AWS-managed encryption keys
- **Secret rotation**: Enable automatic rotation (90-day recommended)

#### Secrets Manager Security Features

| Feature | Implementation | Status |
|---------|----------------|--------|
| Encryption | AWS-managed keys (aws/secretsmanager) | Enabled |
| Access control | IAM policies on EC2 role | Configured |
| Audit logging | CloudTrail enabled | Required |
| Rotation | 90-day schedule | Recommended |

#### Secrets Access Pattern

```python
# Secrets are fetched at runtime by deploy.sh
import boto3
import json

def get_secret(secret_id):
    client = boto3.client('secretsmanager')
    response = client.get_secret_value(SecretId=secret_id)
    return json.loads(response['SecretString'])

# No credentials in code - only EC2 IAM role
secret = get_secret('prod/backend/api-service')
```

### 8.2 Network Security

Securing network access and communication:

#### Security Group Configuration

| Port | Source | Purpose | Notes |
|------|--------|---------|-------|
| 22 | Your IP only | SSH | Restrict to trusted IPs |
| 80 | Anywhere | HTTP | For web applications |
| 3000 | Anywhere | API service | Internal service port |
| 8000 | Anywhere | Search service | Internal service port |
| 5432 | Docker network | PostgreSQL | Container-to-container |
| 6379 | Docker network | Redis | Container-to-container |
| 5672 | Docker network | RabbitMQ | Container-to-container |
| 6333 | Docker network | Qdrant | Container-to-container |

#### Network Security Checklist

- **EC2 in default VPC**: No public subnets for databases
- **Security groups**: Minimal port exposure, least privilege
- **IMDSv2**: Required for EC2 metadata protection
- **ECR private repositories**: No public access
- **S3 bucket**: Block all public access
- **SSH access**: Restricted to specific IPs

#### Security Group Commands

```bash
# Create security group
aws ec2 create-security-group \
  --group-name video-platform-sg \
  --description "Security group for video platform" \
  --vpc-id vpc-xxxxxxxx

# Authorize SSH from specific IP
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp \
  --port 22 \
  --cidr your-ip-address/32

# Authorize application ports
aws ec2 authorize-security-group-ingress \
  --group-id sg-xxxxxxxx \
  --protocol tcp \
  --port 3000 \
  --cidr 0.0.0.0/0
```

### 8.3 Container Security

Securing container images and runtime:

#### Container Security Checklist

- **Multi-stage builds**: Minimize attack surface
- **Non-root user**: Run containers as non-root (where possible)
- **ECR image scanning**: Enabled for vulnerability detection
- **Minimal base images**: Use alpine/slim variants
- **No credentials in images**: All secrets injected at runtime
- **Regular scanning**: Weekly vulnerability scans

#### Dockerfile Security Best Practices

```dockerfile
# Use specific version, not 'latest'
FROM node:18-alpine

# Create non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

# Copy files with correct ownership
COPY --chown=nodejs:nodejs . .

# Switch to non-root user
USER nodejs

# Don't run as root
EXPOSE 3000
```

#### ECR Image Scanning

```bash
# Enable scan on push
aws ecr put-image-scanning-configuration \
  --repository-name api-service \
  --image-scanning-configuration scanOnPush=true

# View scan findings
aws ecr describe-image-scan-findings \
  --repository-name api-service \
  --image-id imageTag=latest
```

### 8.4 IAM Least Privilege

Implementing least privilege access controls:

#### IAM Best Practices

- **Separate roles**: Different roles for CodeBuild and EC2
- **Resource-specific policies**: Avoid wildcards where possible
- **No AWS access keys**: Use IAM roles only
- **Regular policy review**: Monthly audit of IAM policies
- **MFA enforcement**: Require MFA for all IAM users

#### CodeBuild Role Policies (Minimal)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ecr:BatchCheckLayerAvailability",
        "ecr:CompleteLayerUpload",
        "ecr:GetDownloadUrlForLayer",
        "ecr:InitiateLayerUpload",
        "ecr:PutImage",
        "ecr:UploadLayerPart"
      ],
      "Resource": "arn:aws:ecr:us-east-1:772706200692:repository/api-service"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::videoplatform-deploy-artifacts-dsk",
        "arn:aws:s3:::videoplatform-deploy-artifacts-dsk/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "ssm:SendCommand"
      ],
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "ssm:instance-id": "i-037a4cd636a68eb7e"
        }
      }
    }
  ]
}
```

#### EC2 Role Policies

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue"
      ],
      "Resource": [
        "arn:aws:secretsmanager:us-east-1:772706200692:secret:prod/backend/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::videoplatform-deploy-artifacts-dsk/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "ecr:GetDownloadUrlForLayer",
        "ecr:BatchGetImage"
      ],
      "Resource": "*"
    }
  ]
}
```

### 8.5 Security Audit

Regular security audits and compliance checks:

#### Weekly Security Checklist

- [ ] Review CloudTrail for unusual API calls
- [ ] Check IAM access analyzer for public resources
- [ ] Review ECR scan findings
- [ ] Verify security group rules
- [ ] Check for unused IAM credentials
- [ ] Review VPC flow logs for unusual traffic

#### Security Audit Commands

```bash
# Check for unused access keys
aws iam list-access-keys --user-name <username>

# Review IAM policies
aws iam list-policies --scope AWS

# Check security groups for wide-open rules
aws ec2 describe-security-groups --filters "Name=ip-permission.cidr,Values=0.0.0.0/0"

# Review CloudTrail events
aws cloudtrail lookup-events \
  --start-time $(date -d '7 days ago' --iso-8601=seconds) \
  --event-name "CreateUser"
```

### 8.6 Incident Response

Prepare for security incidents:

#### Response Steps

1. **Identify**: Use CloudWatch alarms and logs
2. **Contain**: Isolate affected resources
3. **Investigate**: Review logs and CloudTrail
4. **Remediate**: Apply fixes
5. **Document**: Record incident details
6. **Review**: Update security controls

#### Common Security Incidents

| Incident | Indicators | Response |
|----------|------------|----------|
| Unauthorized access | Unusual API calls, IAM changes | Revoke credentials, rotate secrets |
| Data exfiltration | High NetworkOut | Block traffic, investigate source |
| Malware | Container crashes, resource spikes | Isolate, rebuild container |

### 8.7 Compliance Considerations

Ensure compliance with security frameworks:

- **Encryption**: All data encrypted at rest and in transit
- **Access control**: IAM-based access, no shared credentials
- **Audit trail**: CloudTrail enabled for all API calls
- **Monitoring**: CloudWatch for operational monitoring
- **Vulnerability management**: ECR scanning enabled

### Security Best Practices Summary

| Area | Practice | Status |
|------|----------|--------|
| Secrets | AWS Secrets Manager + IAM roles | Implemented |
| Network | Security groups + private subnets | Implemented |
| Containers | Multi-stage builds + ECR scanning | Implemented |
| IAM | Least privilege + separate roles | Implemented |
| Monitoring | CloudWatch + alarms | Implemented |

### Next Steps

With security best practices in place, proceed to Troubleshooting for solutions to common issues encountered during implementation.