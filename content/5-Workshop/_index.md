+++
title = "Workshop"
date = 2026-08-03
weight = 5
chapter = false
+++

# CI/CD Pipeline Implementation Workshop

{{< notice warning >}}
Note: The information below is for reference purposes only. Please do not copy verbatim for your report, including this warning.
{{< /notice >}}

## Overview

This 12-week workshop documents a comprehensive CI/CD transformation journey from manual deployment to AWS-native automation for a video streaming platform backend. What began as a time-consuming manual process evolved into a streamlined, automated pipeline that deploys two microservices with production-grade security, monitoring, and reliability.

The journey delivers significant business value by reducing deployment time from 20-30 minutes to just 4-5 minutes per service, completely eliminating manual configuration errors that previously caused production outages. By centralizing secrets management in AWS Secrets Manager and implementing IAM role-based access, the platform achieved enterprise-grade security without compromising developer velocity.

Technically, this workshop covers containerization of a NestJS GraphQL API service and FastAPI search service, integration with 8 AWS services (EC2, ECR, CodeBuild, S3, Secrets Manager, SSM, CloudWatch, IAM), and implementation of a full deployment pipeline from code commit to production deployment. The solution handles complex requirements including database migrations, service dependencies, and runtime secret injection.

Participants will gain hands-on experience with AWS architecture design, Docker multi-stage builds, production security best practices, cost optimization strategies, and troubleshooting real-world deployment issues. This workshop is designed for developers and DevOps engineers looking to implement robust CI/CD pipelines in AWS environments.

## Architecture

![CI/CD Architecture](/my-aws-fcaj-2026-journey-official/images/5-Workshop/architecture_diagram.png)

### Components Overview

| AWS Service | Role in Pipeline | Configuration Details |
|-------------|------------------|----------------------|
| **GitHub** | Source control | Branch: `feat/CI-CD`, Path filters: `^api_service/.*$` and `^search_service/.*$` |
| **CodeBuild** | Build automation | 2 projects: `build-api-service`, `build-search-service`, Ubuntu 22.04, Docker privileged mode |
| **ECR** | Container registry | 2 private repos: `api-service`, `search-service`, scan-on-push enabled |
| **S3** | Artifact storage | `videoplatform-deploy-artifacts-dsk`, compose files path: `compose/docker-compose.*.yml` |
| **Secrets Manager** | Runtime secrets | 2 secrets: `prod/backend/api-service`, `prod/backend/search-service`, JSON format |
| **EC2** | Production host | t2.medium (2 vCPU, 4 GB RAM), Ubuntu 22.04, Public IP: 98.81.144.110 |
| **SSM** | Remote deployment trigger | `AWS-RunShellScript` document, targets EC2 instance: `i-037a4cd636a68eb7e` |
| **CloudWatch** | Monitoring | Build logs, SSM command logs, EC2 metrics, alarms for CPU/Disk thresholds |

### Data Flow

1. **Code Push**: Developer pushes changes to `feat/CI-CD` branch in GitHub repository
2. **Webhook Trigger**: GitHub webhook triggers appropriate CodeBuild project based on path filter
3. **Build Phase**: CodeBuild clones repository, builds Docker image using multi-stage builds
4. **Registry Push**: Build image is pushed to ECR repository with `latest` and git SHA tags
5. **Artifact Sync**: Docker Compose files are synchronized to S3 bucket for deployment
6. **Deployment Trigger**: SSM command sent to EC2 instance to execute deployment script
7. **EC2 Deployment**: EC2 pulls latest image from ECR, fetches secrets from Secrets Manager
8. **Container Launch**: Docker Compose starts containers with injected environment variables
9. **Health Verification**: Services are validated and CloudWatch monitoring begins
10. **Pipeline Complete**: Total deployment time: 4-5 minutes per service

## Prerequisites

- **AWS Free Tier account**: Active AWS account with IAM permissions to create resources
- **GitHub account**: Repository access and webhook configuration permissions
- **Technical knowledge**: Basic understanding of Docker, Node.js, Python, Linux CLI
- **Local tools**: AWS CLI v2, Docker Desktop, Git, SSH client
- **Estimated time**: 12 weeks (part-time, ~20 hours/week for implementation and documentation)
- **Estimated cost**: ~$23/month (EC2 t2.medium + ECR storage + S3 + Secrets Manager + minimal data transfer)
- **AWS Region**: us-east-1 (North Virginia)
- **AWS Account ID**: 772706200692

---

## Workshop Sections

Navigate through the sections below to follow the complete workshop:

1. **Introduction** - Overview of the workshop and what you'll learn
2. **Prerequisites** - AWS account setup, tools, and requirements
3. **AWS Account Setup** - EC2 instance launch and bootstrap configuration
4. **AWS Services Configuration** - ECR, Secrets Manager, S3, and IAM setup
5. **Dockerfiles and Scripts** - Container builds and deployment automation
6. **CodeBuild Setup** - Build project configuration and webhook setup
7. **Production Deployment** - Complete pipeline testing and verification
8. **Monitoring** - CloudWatch logs, alarms, and cost tracking
9. **Security** - Best practices for securing the pipeline
10. **Troubleshooting** - Common issues and solutions
11. **Next Steps** - Cost optimization and future improvements

---

## Key Achievements

- **Deployment time**: Reduced from 20-30 minutes → 4-5 minutes (83% improvement)
- **Error rate**: Eliminated manual deployment mistakes and configuration drift
- **Security**: Centralized secrets management, IAM roles, no credentials in code, ECR scanning
- **Scalability**: Ready to scale to multiple environments (staging, production, feature branches)
- **Observability**: CloudWatch monitoring for all components with actionable alarms
- **Cost**: ~$22/month (within Free Tier limits, 60% less than comparable managed services)
- **Reliability**: Automated rollback capability, health checks, dependency management

**Technical accomplishments:**
- Containerized 2 microservices with multi-stage Docker builds
- Implemented 8 AWS service integration with proper IAM permissions
- Created automated pipeline from code commit to production deployment
- Solved 8 blocking issues and 6 audit findings during implementation
- Established production-grade security and monitoring practices

![GitHub Repository](/my-aws-fcaj-2026-journey-official/images/5-Workshop/10_github_repository.png)

---

## Repository Information

**Repository:** [khiem918/VideoPlatformServer](https://github.com/khiem918/VideoPlatformServer)  
**AWS Account:** 772706200692 (us-east-1)  
**Production URL:** http://98.81.144.110:3000/graphql

---

**Last Updated:** August 3, 2026  
**Author:** Ho Khang - FCAJ Internship 2026  
**Duration:** 12 weeks (May 12 - July 31, 2026)  
**AWS Services Used:** 8  
**Total Builds:** 42 successful builds  
**Deployment Success Rate:** 100% after Week 10 fixes