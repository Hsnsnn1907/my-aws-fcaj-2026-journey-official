+++
title = "Worklog Tuần 10"
date = 2026-07-17
weight = 10
chapter = false
pre = "<b>1.9.</b>"
+++

## Tuần 10: Setup CodeBuild, fix 4 build bugs liên tiếp

### Công việc đã làm

- **Thứ 2 (13/07):** Tạo 2 CodeBuild project trên AWS Console:
  - `build-api-service` với source = GitHub repo, webhook filter `^api_service/.*$`.
  - `build-search-service` với source = GitHub repo, webhook filter `^search_service/.*$`.

  Set env vars theo file header của buildspec: `AWS_DEFAULT_REGION`, `AWS_ACCOUNT_ID`, `ECR_REPO_NAME`, `DEPLOY_S3_BUCKET`, `DEPLOY_S3_PREFIX`, `EC2_INSTANCE_ID`.

  Attach IAM role `AmazonEC2ContainerRegistryPowerUser` + `AmazonSSMFullAccess` cho 2 CodeBuild service roles.

- **Thứ 2 (tiếp):** SSH EC2 `i-037a4cd636a68eb7e` để bootstrap (chạy `ec2-bootstrap.sh`). Phát hiện bug `sudo $USER` đã được viết từ tuần 9, fix thủ công bằng `sudo usermod -aG docker ubuntu` + `newgrp docker`.

- **Thứ 2 tối:** Trigger lần build đầu cho `build-search-service`, fail ở step 14 (`COPY docker-entrypoint.sh` not found). Phân tích: `search_service/.dockerignore` line 51 có pattern `docker-entrypoint.sh` (không có `/` prefix) match cả file đúng. Fix: xóa dòng đó khỏi `search_service/.dockerignore`. → commit `f16d5ea`, push.

- **Thứ 3 (14/07):** Trigger `build-search-service` lần 2, fail ở step 12 (`Could not open requirements file`). Nguyên nhân: Dockerfile runtime stage thiếu `COPY requirements.txt .` (chỉ có `COPY /wheels` từ builder). Fix: thêm 1 dòng `COPY requirements.txt .`. → commit `f4f2bd3`, push.

- **Thứ 3 (tiếp):** Trigger lần 3, fail ở step 15 (`You don't have any datasource defined`). Nguyên nhân: `search_service/prisma/schema.prisma` toàn bộ bị comment-out. Fix: uncomment toàn bộ 22 dòng. → commit `1c52793`, push.

- **Thứ 4 (15/07):** Trigger lần 4, fail ở post_build (`The user-provided path docker/ does not exist`). Nguyên nhân: `cd search_service` ở build phase không reset ở post_build phase; `aws s3 sync docker/` chạy trong `search_service/`. Fix: thêm `cd $CODEBUILD_SRC_DIR` trước S3 sync + SSM trong cả 2 buildspec. → commit `1bf28e7`, push.

- **Thứ 5 (16/07):** Verify build `build-search-service` pass hoàn toàn → BUILD SUCCEEDED → POST_BUILD SUCCEEDED → ECR push OK → S3 sync OK → SSM trigger EC2.

- **Thứ 6 (17/07):** Trigger `build-api-service` lần đầu (cũng đã fix `cd $CODEBUILD_SRC_DIR` từ thứ 4) → SUCCEEDED.

### Tóm tắt công việc

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 42 | Tạo 2 CodeBuild project; bootstrap EC2; fix `sudo $USER` bug | 13/07/2026 | 13/07/2026 |
| 43 | Fix 3 build bug (`.dockerignore`, `Dockerfile`, prisma schema) | 14/07/2026 | 14/07/2026 |
| 44 | Fix buildspec `cd $CODEBUILD_SRC_DIR`; trigger lần 4 | 15/07/2026 | 15/07/2026 |
| 45 | Verify `build-search-service` pass hoàn toàn | 16/07/2026 | 16/07/2026 |
| 46 | Trigger `build-api-service` → SUCCEEDED | 17/07/2026 | 17/07/2026 |

### Kết quả đạt được

- 4 commits liên tiếp trên `feat/CI-CD`: `f16d5ea`, `f4f2bd3`, `1c52793`, `1bf28e7`.
- Mỗi bug được phát hiện và fix trong vòng 1 lần build (nhờ layer cache của Docker — chỉ re-run từ step liên quan, không phải build lại từ đầu).
- Tổng kết: 3 bug Dockerfile/dockerignore có từ tuần 9, 1 bug schema.prisma có từ trước tuần 9, 1 bug buildspec có từ tuần 9 — không bị phát hiện sớm vì build trước đó chưa chạy đến step đó.

### Ghi chú

*Bài học lớn: review code không thể thay thế việc chạy thật. Một số bug chỉ lộ ra khi build chạm đến đúng step liên quan. Layer cache của Docker (và CodeBuild) cực kỳ hữu ích để iterate nhanh — chỉ cần fix và push, không cần build lại từ đầu. Chiến lược tốt: build từng service một, fix cho đến khi pass hoàn toàn, rồi mới sang service kia.*
