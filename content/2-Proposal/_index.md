+++
title = "Project Proposal"
date = 2026-06-26
weight = 2
chapter = false
+++

# Project Proposal

## Project Overview

**VideoPlatform** is a semantic video search & sharing platform that lets users upload videos, auto-processes them (transcode, frame extraction, AI-generated captions), and supports content-based search beyond simple tags/titles. As the FCAJ internship deliverable, I designed and shipped an **end-to-end AWS CI/CD pipeline** for the backend: 2 microservices (`api_service` NestJS GraphQL gateway + `search_service` FastAPI vector search) running on EC2, containerized via Docker, with automated build & deploy through AWS CodeBuild → ECR → S3 → SSM. Real-world use case: a video platform for Vietnamese content creators; core value is reducing "code commit → production deploy" time from several hours of manual work down to ~5 minutes automated, while ensuring reproducibility (immutable images, secrets via Secrets Manager, single-command rollback).

## Objectives

- Design and ship an end-to-end AWS CI/CD pipeline for the VideoPlatform backend (CodeBuild + ECR + S3 + EC2), with git-push-to-deploy time ≤ 10 minutes, idempotent, and rollback-capable.
- Split the NestJS monolith backend into 2 microservices by responsibility: `api_service` (gateway, GraphQL, video processing) and `search_service` (FastAPI + Qdrant + Prisma multi-schema Postgres), communicating over gRPC and RabbitMQ.
- Apply AWS security best practices: IAM Role instead of access keys, secrets via AWS Secrets Manager, container image scan on push, key-based SSH + ufw firewall on EC2, CloudWatch alarms for CPU/Memory/health.
- Deliver full observability: centralized logs via CloudWatch (30-day retention), a 4-widget dashboard (CPU/Memory, request count, error rate, container restart count), and a post-deploy smoke-test script verifying all 6 containers are healthy.

## Problem Statement

Before this project, the VideoPlatform team faced 4 major pain points:

1. The NestJS backend monolith (`Server/`) handled GraphQL + video upload + FFmpeg transcode + vector search + background jobs all at once, causing CPU contention and 5-10x GraphQL latency spikes when multiple uploads arrived simultaneously.
2. Deploy was entirely manual — a developer had to SSH into EC2, run `git pull`, `npm install`, `docker compose up`, taking 30-60 minutes and prone to human error.
3. No monitoring — service crashes were only discovered when users complained.
4. Secrets (Firebase private key, DB password) lived in local `.env` files and were manually copied to the server, posing a serious leak risk.

The team chose AWS over on-prem because they needed elasticity (scale with video traffic), managed services (RDS, S3, ECR reduce operational overhead), and an integrated CI/CD ecosystem (CodeBuild, CodePipeline).

## Solution Architecture

![Solution architecture](/images/workshop/architecture-overview.svg)

The solution uses 5 main AWS components:

1. **GitHub** (source) — webhook triggers CodeBuild on push to `feat/CI-CD` with path filters `^api_service/.*$` or `^search_service/.*$`.
2. **AWS CodeBuild** (build) — 2 dedicated projects (`build-api-service`, `build-search-service`) running `buildspec-*.yml` with multiple phases (`install` → `pre_build` → `build` → `post_build`) that build multi-stage Docker images and push to ECR.
3. **Amazon ECR** (image registry) — 2 private repos (`videoplatform/api-service`, `videoplatform/search-service`) with scan-on-push + tag immutability.
4. **Amazon S3** (deploy artifacts) — the bucket `videoplatform-deploy-artifacts-dsk` stores `docker-compose.*.yml` files synced by CodeBuild after every build.
5. **Amazon EC2** (compute) — a single `t3.small` Ubuntu 22.04 instance with IAM Role `EC2-Backend-Role` (S3 read + ECR pull + SSM), running 6 containers via Docker Compose v2: `postgres` (Prisma multi-schema `core`/`search`), `redis` (cache + BullMQ), `qdrant` (vector DB), `rabbitmq` (message broker), `api_service` (NestJS GraphQL gateway, port 3000), `search_service` (FastAPI, port 8000).

An SSM `AWS-RunShellScript` from CodeBuild triggers `deploy.sh` on EC2, which fetches secrets from Secrets Manager, writes `.env`, pulls the new image from ECR, syncs the compose file from S3, and runs `docker compose up -d`.

**Data flow:** client uploads video to Cloudflare R2 (via worker) → publishes an event via RabbitMQ → `api_service` consumes it → triggers FFmpeg transcode + Prisma insert → publishes the result via RabbitMQ → `search_service` embeds & indexes into Qdrant → client queries GraphQL → `api_service` calls `search_service` via gRPC (port 50051).

**Rationale:** CodeBuild over Jenkins because zero server maintenance + native AWS integration; ECR over DockerHub because it's private + IAM-authenticated + VPC-resident; multiple services on one EC2 rather than ECS/EKS because of internship budget and low traffic; Prisma multi-schema over 2 DB instances to reduce overhead and simplify Prisma migrations.

## Timeline

| Phase | Tasks | Deliverable |
|---|---|---|
| Phase 1 (Week 2-3) | Learn AWS fundamentals (Compute, Storage, Networking, Database, Security); create AWS Free Tier account, install AWS CLI v2; launch first EC2 instance; create IAM Role `EC2-Backend-Role`. | AWS Cloud Practitioner assessment report; EC2 running stably with key-based SSH. |
| Phase 2 (Week 4-6) | Explore `VideoPlatformClient` (React 19) + `VideoPlatformServer` (NestJS) codebase; initialize `structure.md`, `workflow.md`, `memory.md`; establish Claude working workflow (5 permanent rules). | `structure.md` + `workflow.md` as canonical reference; standardized GitHub push process. |
| Phase 3 (Week 7-9) | Design 2-microservice architecture (`api_service` + `search_service`), gRPC + RabbitMQ; create ECR repos, S3 bucket, Secrets Manager secret; switch local branch to `feat/CI-CD`; write multi-stage `Dockerfile` + `entrypoint` + `deploy.sh` + `buildspec-*.yml`; commit + push `d3f106e` + `0972102`. | All CI/CD artifacts (12 new + 4 modified files) live on `origin/feat/CI-CD`; docker-compose with all 6 services. |
| Phase 4 (Week 10) | Create 2 CodeBuild projects on AWS Console; bootstrap EC2 (fix `sudo $USER` bug); trigger `build-search-service` first time; fix 4 consecutive build bugs (.dockerignore, Dockerfile requirements.txt, prisma schema uncomment, buildspec `cd $CODEBUILD_SRC_DIR`). | 4 commits `f16d5ea`, `f4f2bd3`, `1c52793`, `1bf28e7` on GitHub; `build-search-service` and `build-api-service` both BUILD SUCCEEDED. |
| Phase 5 (Week 11) | End-to-end verify on EC2: 6 containers running healthy, GraphQL + FastAPI + MinIO response OK; fix Redis hostname bug (`localhost` → `redis`); write `smoke-test.sh`; set up CloudWatch Logs/Alarms/Dashboard; EC2 security hardening; daily Postgres backup to S3. | CI/CD pipeline running stably from GitHub push to EC2 deploy in ~5 minutes; full monitoring; automated backup. |
| Phase 6 (Week 12) | Rewrite `README.md` and `OPERATIONS.md`; clean up 11 LaTeX worklog files; record 3-minute demo video; write 12-week summary report; present results; submit FCAJ final report. | Internship report submitted 31/07/2026; clean repo ready for successor. |

## Budget

Estimated monthly AWS cost (beyond Free Tier):

- **EC2 t3.small** $15/month (730h × $0.0204/h, Ubuntu on-demand, `us-east-1`)
- **ECR storage** $0.10/GB/month × ~5GB image layers = $0.50
- **S3** for `videoplatform-deploy-artifacts` (compose files ~50KB total) < $0.01
- **S3** for `videoplatform-backup` (Postgres dump ~200MB/day × 30 = 6GB, $0.023/GB/month) = $0.14
- **CloudWatch Logs** $0.50/GB ingested at INFO level, 6 containers × ~50MB/day × 30 days = ~9GB = $4.50
- **CloudWatch Alarms + SNS** $0.10/alarm × 4 alarms + $0.06/1000 notifications = $0.50
- **Data transfer out** from EC2 to Internet (GraphQL responses) ~10GB/month × $0.09/GB = $0.90
- **CodeBuild** $0.005/build-min × 2 services × 30 builds/month × ~5 min = $1.50

**Total estimate: ~$23/month**, within AWS Free Tier for the first 12 months. Estimated annual cost: **~$280/year** after Free Tier expires.

Cost-control methods: (1) set CloudWatch log retention to 30 days; (2) use `t3.small` instead of `t3.medium` given low traffic; (3) set a budget alarm at $30/month; (4) stop EC2 outside working hours if not needed; (5) review AWS Cost Explorer monthly.

## Risks and Mitigation

| Risk | Impact | Mitigation |
|---|---|---|
| **Dockerfile or buildspec errors** (4 consecutive build bugs seen in week 10: `.dockerignore` matching, missing `COPY requirements.txt`, prisma schema comment-out, `cd $CODEBUILD_SRC_DIR`) | Build failure; extended debug time (~5 min per build attempt). | Audit Dockerfile + buildspec + dockerignore before first build; layer cache helps; test locally before pushing; write unit tests for entrypoint; use hadolint for Dockerfile. |
| **Exceeding Free Tier** due to oversized images or excessive logs | Unexpected costs ($50-100/month). | Strict `.dockerignore`; CloudWatch log retention 30 days; AWS Budget alarm at 50% and 100% threshold; compress Postgres backup before S3 upload; review Cost Explorer weekly. |
| **Secret leakage via image layer** (e.g. `FIREBASE_PRIVATE_KEY` if accidentally COPYed into Dockerfile) | Compromised production credentials for Firebase/DB. | Never COPY any `.env` into the image; secrets injected at runtime only via `deploy.sh` from Secrets Manager; ECR scan on push warns on key patterns; rotate secrets every 90 days; use IMDSv2 on EC2. |
| **EC2 instance failure** or `docker compose up` error | Service outage; users cannot upload or query videos. | `smoke-test.sh` post-deploy verifies 6 containers healthy, auto-rollback on failure; CloudWatch alarm restarts containers on 3 health check failures; daily Postgres backup, restore in ~30 min; manual rollback via ECR tag; future migration to ECS/EKS for auto-replace. |
| **Branch `feat/CI-CD` diverging from `main`** when multiple people push | Merge conflicts, lost commits. | All CI/CD changes must go through PR review; rebase `feat/CI-CD` onto `main` each sprint; once stable, merge and archive the branch. |
