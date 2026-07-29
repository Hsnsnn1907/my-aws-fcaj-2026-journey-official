+++
title = "Week 10 Worklog"
date = 2026-07-17
weight = 10
chapter = false
pre = "<b>1.9.</b>"
+++

## Week 10: CodeBuild Setup & Fixing 4 Build Bugs in Succession

### Completed Tasks

- **Monday (13/07):** Created 2 CodeBuild projects on the AWS Console:
  - `build-api-service` with source = GitHub repo, webhook filter `^api_service/.*$`.
  - `build-search-service` with source = GitHub repo, webhook filter `^search_service/.*$`.

  Set environment variables as per the buildspec file header: `AWS_DEFAULT_REGION`, `AWS_ACCOUNT_ID`, `ECR_REPO_NAME`, `DEPLOY_S3_BUCKET`, `DEPLOY_S3_PREFIX`, `EC2_INSTANCE_ID`.

  Attached IAM roles `AmazonEC2ContainerRegistryPowerUser` + `AmazonSSMFullAccess` to both CodeBuild service roles.

- **Monday (continued):** SSH'd into EC2 `i-037a4cd636a68eb7e` to bootstrap (ran `ec2-bootstrap.sh`). Discovered `sudo $USER` bug carried over from Week 9; manually fixed with `sudo usermod -aG docker ubuntu` + `newgrp docker`.

- **Monday evening:** Triggered the first build for `build-search-service` — failed at step 14 (`COPY docker-entrypoint.sh` not found). Root cause: `search_service/.dockerignore` line 51 had pattern `docker-entrypoint.sh` (without `/` prefix) which matched the correct file too. Fix: removed that line from `search_service/.dockerignore`. → commit `f16d5ea`, pushed.

- **Tuesday (14/07):** Triggered `build-search-service` a second time — failed at step 12 (`Could not open requirements file`). Root cause: the Dockerfile runtime stage was missing `COPY requirements.txt .` (only had `COPY /wheels` from builder). Fix: added one line `COPY requirements.txt .`. → commit `f4f2bd3`, pushed.

- **Tuesday (continued):** Triggered a third time — failed at step 15 (`You don't have any datasource defined`). Root cause: `search_service/prisma/schema.prisma` was entirely commented out. Fix: uncommented all 22 lines. → commit `1c52793`, pushed.

- **Wednesday (15/07):** Triggered a fourth time — failed at post_build (`The user-provided path docker/ does not exist`). Root cause: `cd search_service` in the build phase did not reset in the post_build phase; `aws s3 sync docker/` ran inside `search_service/`. Fix: added `cd $CODEBUILD_SRC_DIR` before the S3 sync + SSM commands in both buildspecs. → commit `1bf28e7`, pushed.

- **Thursday (16/07):** Verified `build-search-service` passed completely — BUILD SUCCEEDED → POST_BUILD SUCCEEDED → ECR push OK → S3 sync OK → SSM triggered on EC2.

- **Friday (17/07):** Triggered `build-api-service` for the first time (also had `cd $CODEBUILD_SRC_DIR` fix from Wednesday) → SUCCEEDED.

### Task Summary

| Day | Task | Start Date | Completion Date |
|---|---|---|---|
| 42 | Created 2 CodeBuild projects; bootstrapped EC2; fixed `sudo $USER` bug | 13/07/2026 | 13/07/2026 |
| 43 | Fixed 3 build bugs (`.dockerignore`, `Dockerfile`, Prisma schema) | 14/07/2026 | 14/07/2026 |
| 44 | Fixed buildspec `cd $CODEBUILD_SRC_DIR`; triggered 4th build | 15/07/2026 | 15/07/2026 |
| 45 | Verified `build-search-service` passed completely | 16/07/2026 | 16/07/2026 |
| 46 | Triggered `build-api-service` → SUCCEEDED | 17/07/2026 | 17/07/2026 |

### Outcomes

- 4 consecutive commits on `feat/CI-CD`: `f16d5ea`, `f4f2bd3`, `1c52793`, `1bf28e7`.
- Each bug was identified and fixed within a single build cycle (thanks to Docker layer caching — only re-running from the relevant step, not rebuilding from scratch).
- Summary: 3 Dockerfile/dockerignore bugs dated back to Week 9, 1 schema.prisma bug predated Week 9, 1 buildspec bug from Week 9 — none caught earlier because previous builds had not reached those steps yet.

### Notes

*Key lesson: code review cannot replace actually running the code. Some bugs only surface when the build reaches the exact relevant step. Docker (and CodeBuild) layer caching is extremely useful for fast iteration — just fix and push, no need to rebuild from scratch. Best strategy: build one service at a time, fix until it passes completely, then move on to the next.*
