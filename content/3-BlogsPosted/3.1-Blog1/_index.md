+++
title = "3.1. Blog 1: Building a CI/CD pipeline with AWS CodeBuild + ECR + S3 + SSM"
date = 2026-07-01
weight = 1
chapter = false
+++

## Blog 1: Building a CI/CD pipeline with AWS CodeBuild + ECR + S3 + SSM

**Introduction**
This blog shares practical experience building the CI/CD pipeline for the VideoPlatform project (a Node.js 18 + NestJS 11 `api_service` and a Python 3.10 + FastAPI `search_service`). The entire flow, from pushing code to GitHub to services running on EC2, takes about 5 minutes, entirely automated without manual SSH. It focuses on CodeBuild, ECR, S3, and SSM.

**Key points:**
- **4-step Architecture:** GitHub webhook $\rightarrow$ CodeBuild $\rightarrow$ ECR push $\rightarrow$ S3 sync $\rightarrow$ SSM `AWS-RunShellScript` trigger.
- **CodeBuild Ephemeral Env:** Billed by the minute (`$0.005/minute`), auto-scaled and auto-cleaned, heavily cost-effective compared to traditional self-hosted Jenkins.
- **Parallel CodeBuild Projects:** Using path-based webhook filtering (`^api_service/.*$`) to separately build microservices rather than rebuilding everything whenever a root level file changes.
- **Troubleshooting:**
  - **Bug 1:** `.dockerignore` conflicting with the build context.
  - **Bug 2:** Forgetting `COPY requirements.txt` before running pip in multi-stage builds.
  - **Bug 3:** Having `prisma/schema.prisma` commented out completely resulting in zero datasources.
  - **Bug 4:** Remaining inside the `search_service` directory during `post_build` ruining the S3 sync. Fixed by returning to `$CODEBUILD_SRC_DIR`.
  - Docker Entrypoints vs Dockerfile RUN: A TCP wait must happen in `entrypoint.sh` using `nc -z db 5432`, not blindly within `Dockerfile RUN`.
- Docker Layer Cache usage allows fixing quick steps iteratively in 1-2 minutes instead of 10-15 minute full rebuilds. Use `--progress=plain` and `docker run --rm -it $IMAGE_ID sh` to visually debug containers.

**Lessons learned**
Never substitute live test pipelines with mere code review. 4 distinct bugs only surfaced during actual CodeBuild triggers at week 10. Run early and run often using local Layer Caches to cut 15 minute iterations into 1. Replicating the Jenkins deployment setup effectively in AWS-native tooling offers immense flexibility.

![Blog 1 Illustration](../../images/blog_1.png)