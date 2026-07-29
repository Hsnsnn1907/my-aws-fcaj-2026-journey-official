+++
title = "Worklog Tuần 11"
date = 2026-07-24
weight = 11
chapter = false
pre = "<b>1.10.</b>"
+++

## Tuần 11: End-to-end verification trên EC2, monitoring + hardening

### Công việc đã làm

- **SSH vào EC2** `i-037a4cd636a68eb7e`, verify:
  - `docker ps` thấy 2 container `api-service` + `search-service` + 4 infra containers (postgres, redis, qdrant, rabbitmq) đang chạy.
  - `curl http://localhost:3000/graphql` trả response GraphQL hợp lệ.
  - `curl http://localhost:8000/health` trả `{"status":"ok"}` (search_service FastAPI).

- **Phát hiện và fix bug runtime** phát sinh trên EC2: `api_service` crash với `ECONNREFUSED 127.0.0.1:6379`. Nguyên nhân: `REDIS_URL=redis://localhost:6379` nhưng Redis chạy trong container riêng → phải dùng hostname `redis`. Fix: cập nhật `api_service/src/config/env.validation.ts` default `REDIS_URL=redis://redis:6379` khi chạy trong Docker.

- **Viết `script/smoke-test.sh`** chạy trên EC2 sau mỗi deploy: gọi `/health` của cả 3 service, fail nếu bất kỳ service nào không respond trong 30s.

- **Setup CloudWatch Logs:** Tạo log group `/ecs/api-service` và `/ecs/search-service`, retention 30 ngày.

- **Setup CloudWatch Alarms:**
  - CPU > 80% trong 5 phút → SNS notification.
  - Health check fail 3 lần liên tiếp → trigger SSM `docker restart` + alert.

- **Tạo CloudWatch Dashboard** với 4 widget: CPU/Memory utilization, request count, error rate 4xx/5xx, container restart count.

- **Security hardening EC2:** Tắt password auth, disable root SSH, cài `ufw` firewall (chỉ mở 22/80/443/3000/8000), enable `fail2ban`, `unattended-upgrades` cho security patches.

- **Setup Postgres backup hàng ngày** lên S3 `videoplatform-backup`: `pg_dump` + `aws s3 cp` qua cron job lúc 02:00 UTC.

### Tóm tắt công việc

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 47 | SSH EC2 verify 6 containers; phát hiện Redis hostname bug; fix | 20/07/2026 | 20/07/2026 |
| 48 | Viết `smoke-test.sh`; verify end-to-end GraphQL flow | 21/07/2026 | 21/07/2026 |
| 49 | Setup CloudWatch Logs + Alarms + Dashboard | 22/07/2026 | 22/07/2026 |
| 50 | Security hardening (SSH, ufw, fail2ban, auto-update) | 23/07/2026 | 23/07/2026 |
| 51 | Postgres backup cron job sang S3; verify restore từ backup test | 24/07/2026 | 24/07/2026 |

### Kết quả đạt được

- Toàn bộ pipeline CI/CD chạy thành công end-to-end: GitHub push → CodeBuild → ECR → S3 → SSM → EC2 deploy.
- 2 service (api + search) + 4 infra containers đều healthy trên EC2.
- Có monitoring, logging, alerts đầy đủ — có thể phát hiện sự cố trong vòng 5 phút.
- Có backup Postgres tự động, có thể khôi phục dữ liệu khi cần.

### Ghi chú

*Bug Redis hostname (`localhost` vs `redis`) là ví dụ điển hình của "works on my machine" — không phát hiện được khi dev local vì Redis có thể đang chạy ở host, không phải trong Docker. Khuyến nghị: luôn dùng Docker Compose cho local dev kể cả khi chỉ test 1 service, để khớp với production networking model. Cost estimate cho CloudWatch + SNS + S3 backup khoảng \$15-25/tháng với traffic hiện tại.*
