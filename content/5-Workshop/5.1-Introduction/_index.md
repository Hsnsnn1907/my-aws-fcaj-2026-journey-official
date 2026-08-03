+++
title = "5.1. Introduction"
date = 2026-08-03
weight = 1
chapter = false
+++

## Introduction to CI/CD Pipeline Implementation

This workshop guides you through implementing a production-ready CI/CD pipeline for two microservices: a NestJS GraphQL API service and a FastAPI search service. The solution leverages AWS native services to create an automated deployment workflow that eliminates manual intervention, improves security, and reduces deployment time.

### Workshop Overview

This 12-week workshop documents a comprehensive CI/CD transformation journey from manual deployment to AWS-native automation for a video streaming platform backend. What began as a time-consuming manual process evolved into a streamlined, automated pipeline that deploys two microservices with production-grade security, monitoring, and reliability.

The journey delivers significant business value by reducing deployment time from 20-30 minutes to just 4-5 minutes per service, completely eliminating manual configuration errors that previously caused production outages. By centralizing secrets management in AWS Secrets Manager and implementing IAM role-based access, the platform achieved enterprise-grade security without compromising developer velocity.

### What You'll Learn

Technically, this workshop covers:
- Containerization of a NestJS GraphQL API service and FastAPI search service
- Integration with 8 AWS services (EC2, ECR, CodeBuild, S3, Secrets Manager, SSM, CloudWatch, IAM)
- Implementation of a full deployment pipeline from code commit to production deployment
- Complex requirements including database migrations, service dependencies, and runtime secret injection

### Target Audience

This workshop is designed for developers and DevOps engineers looking to implement robust CI/CD pipelines in AWS environments. You will gain hands-on experience with:
- AWS architecture design
- Docker multi-stage builds
- Production security best practices
- Cost optimization strategies
- Troubleshooting real-world deployment issues

### Key Achievements

By completing this workshop, you will have:
- **Deployment time**: Reduced from 20-30 minutes → 4-5 minutes (83% improvement)
- **Error rate**: Eliminated manual deployment mistakes and configuration drift
- **Security**: Centralized secrets management, IAM roles, no credentials in code, ECR scanning
- **Scalability**: Ready to scale to multiple environments (staging, production, feature branches)
- **Observability**: CloudWatch monitoring for all components with actionable alarms
- **Cost**: ~$22/month (within Free Tier limits, 60% less than comparable managed services)
- **Reliability**: Automated rollback capability, health checks, dependency management

### Technical Stack

- **Version Control**: GitHub
- **Build System**: AWS CodeBuild
- **Container Registry**: Amazon ECR
- **Orchestration**: Docker Compose on EC2
- **Infrastructure**: Amazon EC2 (t2.medium)
- **Secrets Management**: AWS Secrets Manager
- **Monitoring**: Amazon CloudWatch
- **Infrastructure Automation**: AWS Systems Manager (SSM)
- **Artifact Storage**: Amazon S3

### Journey Timeline

- **Weeks 2-3**: AWS Account Setup and EC2 instance launch
- **Week 8**: AWS Services configuration (ECR, Secrets Manager, S3, IAM)
- **Week 9**: Dockerfiles and CI/CD scripts development
- **Week 10**: CodeBuild projects setup and testing
- **Weeks 11-12**: Production deployment and optimization

### Next Steps

Start with the Prerequisites section to ensure you have all necessary tools and AWS account setup, then proceed through each section in order to build your complete CI/CD pipeline.
