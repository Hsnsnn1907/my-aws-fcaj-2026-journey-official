+++
title = "Prerequisites"
date = 2026-08-03
weight = 2
chapter = false
+++

## Prerequisites for CI/CD Pipeline Workshop

Before starting this workshop, ensure you have the following requirements in place:

### AWS Account Requirements

- **AWS Free Tier account**: Active AWS account with IAM permissions to create resources
- **AWS Region**: us-east-1 (North Virginia) recommended
- **AWS Account ID**: 772706200692 (example from workshop, use your own)
- **Billing alerts**: Set up billing alerts to monitor costs
- **MFA**: Enable Multi-Factor Authentication for the root account
- **IAM user**: Create an IAM user with administrative privileges for day-to-day operations

### Development Tools

- **GitHub account**: Repository access and webhook configuration permissions
- **AWS CLI v2**: Latest version installed and configured
- **Docker Desktop**: For local container testing (Docker 20.10+ recommended)
- **Git**: Version control system
- **SSH client**: For secure access to EC2 instances
- **Text editor/IDE**: Visual Studio Code, IntelliJ IDEA, or similar

### Technical Knowledge

- **Basic understanding of Docker**: Dockerfiles, images, containers, Docker Compose
- **Node.js/Python knowledge**: For understanding the microservice codebases
- **Linux CLI familiarity**: Basic command line operations
- **AWS fundamentals**: Familiarity with core AWS services (EC2, S3, IAM)

### Time and Cost Estimates

#### Time Commitment
- **Estimated duration**: 12 weeks (part-time implementation)
- **Weekly commitment**: ~20 hours/week for implementation and documentation
- **Total hours**: ~240 hours

#### Cost Estimates
- **Monthly cost**: ~$23/month
  - EC2 t2.medium: ~$17/month
  - ECR storage: ~$0.50/month  
  - S3 storage: ~$0.50/month
  - Secrets Manager: ~$1/month
  - Data transfer: ~$3/month
  - CloudWatch: ~$0.50/month
  - CodeBuild: ~$0 (within Free Tier limits)

### Workshop Repository

- **Repository**: [khiem918/VideoPlatformServer](https://github.com/khiem918/VideoPlatformServer)
- **Branch**: `feat/CI-CD`
- **Services**: Two microservices in separate directories:
  - `api_service/` - NestJS GraphQL API
  - `search_service/` - FastAPI search service

### Network Requirements

- **Internet connectivity**: Stable internet connection for AWS service access
- **Port access**: Ability to connect to AWS services (no corporate firewall restrictions)
- **SSH access**: Port 22 access for EC2 instance connection

### Development Workstation

- **Operating System**: Windows 10+, macOS 10.15+, or Linux
- **Memory**: 8GB RAM minimum (16GB recommended)
- **Storage**: 10GB free space for Docker images and tools
- **CPU**: Multi-core processor for Docker builds

### Pre-Workshop Checklist

Before starting the implementation:

- [ ] AWS account created and verified
- [ ] IAM user with admin permissions configured
- [ ] AWS CLI v2 installed and configured (`aws configure`)
- [ ] Docker Desktop installed and running
- [ ] Git installed and configured
- [ ] SSH key pair generated for EC2 access
- [ ] GitHub account accessible
- [ ] Billing alerts configured in AWS Console
- [ ] MFA enabled for AWS root account

### Account Verification

Test your AWS setup with the following commands:

```bash
# Verify AWS CLI configuration
aws sts get-caller-identity

# Check Docker installation
docker --version
docker run hello-world

# Verify Git
git --version

# Test SSH key generation
ssh-keygen -t rsa -b 4096 -f ~/.ssh/video-platform -q -N ""
```

### Troubleshooting Setup Issues

If you encounter setup issues:

1. **AWS CLI authentication**: Ensure you have valid credentials and correct region
2. **Docker permission issues**: Add your user to docker group or run with sudo
3. **GitHub access**: Verify SSH keys are added to GitHub account
4. **Port conflicts**: Check for existing services on ports 3000, 8000, 5432, etc.
5. **Firewall restrictions**: Ensure outbound connections to AWS services are allowed

### Cost Monitoring Setup

Set up cost monitoring before starting:

1. **AWS Cost Explorer**: Enable in Billing & Cost Management
2. **Billing alarms**: Create CloudWatch alarms for budget thresholds
3. **AWS Budgets**: Set up monthly budget with notifications
4. **Cost allocation tags**: Tag resources for tracking

### Regional Considerations

- **Recommended region**: us-east-1 (North Virginia)
- **Alternative regions**: us-west-2 (Oregon), eu-west-1 (Ireland)
- **Considerations**: Service availability, pricing differences, latency

### Security Considerations

- **Never share AWS credentials**: Use IAM roles instead
- **Secure SSH keys**: Protect private key files with appropriate permissions
- **Regular backups**: Back up important configuration files
- **Audit trail**: Enable AWS CloudTrail for activity logging

With these prerequisites in place, you're ready to begin the workshop implementation starting with AWS Account Setup.
