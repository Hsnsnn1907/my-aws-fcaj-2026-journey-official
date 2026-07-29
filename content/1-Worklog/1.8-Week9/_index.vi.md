+++
title = "Worklog Tuần 9"
date = 2026-07-10
weight = 9
chapter = false
pre = "<b>1.8.</b>"
+++

## Tuần 9: Branch switch, refactor & tạo CI/CD artifacts

### Công việc đã làm

- **Thứ 2 (06/07)**: Thực hiện destructive branch switch trên `VideoPlatformServer`:
  - `git fetch origin feat/CI-CD`
  - `git branch -m main feat/CI-CD` (đổi tên local main thành feat/CI-CD)
  - `git reset --hard origin/feat/CI-CD` (HEAD = `5788515`)
  - `git branch -u origin/feat/CI-CD feat/CI-CD`
  - Lưu ý: 5 commit cũ trên local `main` (`0f6c453`, `62d7e94`, ...) chỉ còn trong reflog ~30 ngày.
- **Thứ 2 (tiếp)**: Viết lại `structure.md` cho layout mới (`api_service/` + `search_service/` + `proto/` + `docker/`); viết lại `README.md` của `VideoPlatformServer`; dọn file stray `search_service/=1.68` (shell-globbing bug); cập nhật docs để xóa cleanup items.
- **Thứ 3 (07/07)**: Phân tích 8 blockers từ plan CI/CD, lên chiến lược:
  1. `docker-compose.api-service.yml` thiếu `api_service` service.
  2. `docker-compose.search-service.yml` thiếu `search_service` service.
  3. Chưa có `Dockerfile` cho 2 service.
  4. `prisma migrate deploy` chưa chạy khi container start.
  5. Build context quá lớn (cần `.dockerignore`).
  6. EC2 không có git credentials (dùng S3 thay `git pull`).
  7. `docker-compose` v1 deprecated (chuyển v2).
  8. `jq` corrupt newline trong `FIREBASE_PRIVATE_KEY` (dùng Python heredoc).
- **Thứ 4 (08/07)**: Viết toàn bộ CI/CD artifacts:
  - `api_service/Dockerfile` (multi-stage Node 18-alpine).
  - `api_service/.dockerignore`, `api_service/docker-entrypoint.sh`.
  - `search_service/Dockerfile` (multi-stage python:3.10-slim).
  - `search_service/.dockerignore`, `search_service/docker-entrypoint.sh`.
  - `script/deploy.sh` (S3-based, no git pull).
  - `script/ec2-bootstrap.sh`, `script/sync-compose-to-s3.sh`.
  - `buildspec-api.yml`, `buildspec-search.yml`.
- **Thứ 5 (09/07)**: Cập nhật placeholder S3 bucket name trong cả 2 script (`videoplatform-deploy-artifacts-dsk`). Commit `d3f106e` gộp 11 file mới + 3 modified, push lên `origin/feat/CI-CD`.
- **Thứ 6 (10/07)**: Audit toàn bộ CI/CD files trên đĩa, phát hiện 6 issues (stray `api_service/=1.68`, stale guard, cosmetic newline, 2 important về load_dotenv/prisma generate, cleanup pattern). Commit `0972102` fix 6 issues, push lên GitHub.

### Công việc chính

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 37 | Switch branch sang `feat/CI-CD`; viết lại docs; xóa stray file | 06/07/2026 | 06/07/2026 |
| 38 | Phân tích 8 blockers; lên chiến lược CI/CD | 07/07/2026 | 07/07/2026 |
| 39 | Viết Dockerfiles, entrypoints, `deploy.sh`, buildspecs | 08/07/2026 | 08/07/2026 |
| 40 | Cập nhật S3 bucket; commit `d3f106e`; push lên GitHub | 09/07/2026 | 09/07/2026 |
| 41 | Audit 6 issues; commit `0972102`; push | 10/07/2026 | 10/07/2026 |

### Kết quả đạt được

- Local branch `feat/CI-CD` tracking `origin/feat/CI-CD` đồng bộ.
- 2 commit lớn (`d3f106e`, `0972102`) đã live trên GitHub.
- Toàn bộ Dockerfile + buildspec + deploy script + entrypoint đã sẵn sàng cho lần build thật ở tuần 10.
- Audit trước khi trigger build giúp phát hiện sớm 6 blockers không bị lọt vào pipeline.

### Ghi chú

*Bài học lớn: latent bug `sudo usermod -aG docker "$USER"` trong `ec2-bootstrap.sh` -- khi chạy với `sudo`, `$USER` resolve thành `root`, nên docker group được add cho root chứ không phải `ubuntu`. Fix bằng `REAL_USER=${SUDO_USER:-$(logname 2>/dev/null || echo ubuntu)}`. Shell-globbing bug khi gõ `pip install package==X.Y` cũng lặp lại ở `api_service/` -- pattern đã được block bằng `=*` trong `.dockerignore` và `.gitignore`.*
