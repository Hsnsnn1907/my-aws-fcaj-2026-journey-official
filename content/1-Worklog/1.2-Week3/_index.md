+++
title = "Week 3 Worklog"
date = 2026-05-29
weight = 3
chapter = false
pre = "<b>1.2.</b>"
+++

## Week 3: EC2, IAM and Deploying the First Instance

### Completed Tasks

- Learned EC2 architecture: AMI, instance types, EBS, key pairs, security groups, elastic IP.
- Learned IAM: users, groups, roles, policies; distinguished between IAM Users (long-term credentials) vs IAM Roles (temporary credentials for EC2).
- Created an IAM Role `EC2-Backend-Role` with policies `AmazonS3ReadOnlyAccess`, `AmazonEC2ContainerRegistryPowerUser`, and `AmazonSSMFullAccess` (will be used for deployment scripts later).
- Launched an EC2 instance `t3.small` running Ubuntu 22.04, configured Security Group to open ports 22 (SSH), 80 (HTTP), and 443 (HTTPS).
- Read basic VPC documentation (subnets, route tables, internet gateways) in preparation for deploying multiple services in subsequent weeks.

### Key Tasks Completed

| Day | Task | Start Date | Completion Date |
|---|---|---|---|
| 7 | Learned EC2: AMI, instance type, EBS, key pair, security group | 25/05/2026 | 25/05/2026 |
| 8 | Learned IAM: User/Group/Role/Policy; credential model distinction | 26/05/2026 | 26/05/2026 |
| 9 | Created IAM Role `EC2-Backend-Role` with S3/ECR/SSM policies | 27/05/2026 | 27/05/2026 |
| 10 | Launched EC2 `t3.small` Ubuntu 22.04, configured Security Group | 28/05/2026 | 28/05/2026 |
| 11 | Read VPC docs: subnets, route tables, internet gateways | 29/05/2026 | 29/05/2026 |

### Outcomes

- First EC2 instance running stably, successful SSH connection via key pair.
- IAM Role successfully attached to the instance, enabling AWS API calls from EC2 without storing access keys on disk (a best practice).
- Clear understanding of why Roles are used instead of User credentials for long-running services on EC2.

### Notes

*Important lesson: never store AWS access keys in EC2 user data or repositories. Always use IAM Roles + metadata service (IMDSv2). This EC2 instance will serve as the backend deployment host for the upcoming weeks.*
