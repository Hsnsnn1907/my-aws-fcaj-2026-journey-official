+++
title = "AWS Account Setup"
date = 2026-08-03
weight = 3
chapter = false
+++

## AWS Account Setup (Week 2-3)

This section covers the foundational setup of your AWS infrastructure including account creation, EC2 instance launch, and prerequisite installations.

### 2.1 Create AWS Free Tier Account

Begin by creating an AWS Free Tier account at https://aws.amazon.com/free. Follow these steps:

1. **Account creation**: Provide email, password, and contact information
2. **Payment method**: Add a valid payment method (Free Tier still requires this)
3. **Identity verification**: Complete phone verification process
4. **Support plan**: Select "Basic" (Free) support plan

**Important configurations after account creation:**
- **Billing alerts**: Configure alerts at 50%, 80%, and 100% of your budget
- **MFA**: Enable Multi-Factor Authentication for the root account
- **IAM user**: Create an administrative user for day-to-day operations

### 2.2 Launch EC2 Instance

The EC2 instance serves as the production host for all containers. Configure it with the following specifications:

#### EC2 Instance Configuration

- **Instance type**: t2.medium (2 vCPU, 4 GB RAM) - sufficient for development/production workloads
- **AMI**: Ubuntu 22.04 LTS (ami-0c02fb55956c7d316) - stable LTS release
- **Public IP**: 98.81.144.110 (automatically assigned in workshop)
- **Storage**: 20 GB GP2 root volume
- **Key pair**: Create new SSH key named `video-platform` for secure access

#### Network Security Configuration

![EC2 Network Config 1](/my-aws-fcaj-2026-journey-official/images/5-Workshop/07_ec2_network_config_1.png)
![EC2 Network Config 2](/my-aws-fcaj-2026-journey-official/images/5-Workshop/08_ec2_network_config_2.png)

**Security groups configuration:**
- **SSH (port 22)**: Allow from your IP only
- **HTTP (port 80)**: Allow from anywhere (for web applications)
- **Application ports**: Allow from anywhere:
  - 3000 (API service)
  - 8000 (Search service) 
  - 5432 (PostgreSQL)
  - 6379 (Redis)
  - 5672 (RabbitMQ)
  - 6333 (Qdrant)

#### IAM Role Attachment

Attach IAM role to EC2 instance for AWS service access:
- **Role name**: `EC2-Backend-Role`
- **Permissions**: SSM, Secrets Manager, S3, ECR access
- **Policy**: `AmazonSSMManagedInstanceCore`, `SecretsManagerReadWrite`, `AmazonS3ReadOnlyAccess`, `AmazonEC2ContainerRegistryReadOnly`

### Detailed EC2 Launch Steps

1. Navigate to EC2 Console → Launch instance
2. **Name**: `video-platform-production`
3. **AMI**: Select Ubuntu 22.04 LTS
4. **Instance type**: Choose t2.medium
5. **Key pair**: Create new key pair `video-platform`, download .pem file
6. **Network settings**: Default VPC, enable public IP
7. **Security groups**: Configure as listed above
8. **Advanced details**: Attach IAM role `EC2-Backend-Role`
9. **Launch instance**: Note instance ID: `i-037a4cd636a68eb7e` (workshop example)

### 2.3 Install Prerequisites on EC2

Connect to EC2 via SSH and run the bootstrap script to install necessary tools:

#### SSH Connection

```bash
# Connect to EC2 (replace with your instance IP)
ssh -i ~/.ssh/video-platform.pem ubuntu@98.81.144.110

# Run bootstrap script
sudo bash ec2-bootstrap.sh
```

#### Bootstrap Script Contents (ec2-bootstrap.sh)

The bootstrap script installs all necessary prerequisites:

```bash
#!/bin/bash
set -e

echo "Updating system packages..."
sudo apt-get update
sudo apt-get upgrade -y

echo "Installing Docker Engine..."
sudo apt-get install -y ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "Adding Docker repository..."
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

echo "Adding user to docker group..."
sudo usermod -aG docker $USER

echo "Installing AWS CLI v2..."
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install --update

echo "Installing utilities..."
sudo apt-get install -y jq netcat curl wget git

echo "Creating deployment directory structure..."
sudo mkdir -p /app/compose
sudo mkdir -p /app/logs

echo "Setting permissions..."
sudo chown -R ubuntu:ubuntu /app

echo "Bootstrap complete! Please log out and back in for group changes to take effect."
```

#### What ec2-bootstrap.sh Installs

1. **Docker Engine (20.10+)** and Docker Compose v2
2. **AWS CLI v2** for AWS service interactions
3. **jq** for JSON parsing in deployment scripts
4. **Network tools** (netcat, curl) for health checks
5. **System updates** and security patches
6. **Directory structure** for deployment artifacts

### Post-Installation Verification

After bootstrap script completes, verify the installation:

```bash
# Log out and reconnect
exit
ssh -i ~/.ssh/video-platform.pem ubuntu@98.81.144.110

# Verify installations
docker --version
docker-compose --version
aws --version
jq --version

# Test Docker without sudo
docker run hello-world

# Verify IAM role attachment
aws sts get-caller-identity
# Expected output: arn:aws:sts::772706200692:assumed-role/EC2-Backend-Role/i-037a4cd636a68eb7e
```

### IAM Role Verification

The EC2 instance should have the correct IAM role attached for accessing AWS services:

```bash
# On EC2 instance
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
# Should return the IAM role name

# Test Secrets Manager access
aws secretsmanager list-secrets --region us-east-1 --max-results 1
# Should return list of secrets (or empty if none created yet)
```

### Security Configuration

1. **SSH hardening**: Update SSH configuration for better security
2. **Firewall rules**: Verify security groups allow necessary traffic only
3. **System monitoring**: Install basic monitoring tools
4. **Backup configuration**: Set up EC2 instance backups

### Troubleshooting Common Issues

#### Issue: SSH Connection Failed
**Solution**: Check security group rules, verify key pair permissions, ensure instance is running

#### Issue: Docker Permission Denied
**Solution**: Log out and back in after adding user to docker group, or use `sudo`

#### Issue: IAM Role Not Attached
**Solution**: Stop instance, attach correct IAM role, restart instance

#### Issue: Bootstrap Script Errors
**Solution**: Run commands manually from script, check network connectivity

### Next Steps

With the EC2 instance configured and prerequisites installed, you're ready to proceed to AWS Services Configuration where you'll set up ECR, Secrets Manager, S3, and IAM roles for the complete CI/CD pipeline.
