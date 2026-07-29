+++
title = "Week 11 Worklog"
date = 2026-07-24
weight = 11
chapter = false
pre = "<b>1.10.</b>"
+++

## Week 11: E2E Verification on EC2 & Monitoring

### Completed Tasks

- **SSH into EC2** `i-037a4cd636a68eb7e`, verified:
  - `docker ps` showed 2 containers `api-service` + `search-service` + 4 infrastructure containers (postgres, redis, qdrant, rabbitmq) running successfully.
  - `curl http://localhost:3000/graphql` returned a valid GraphQL response.
  - `curl http://localhost:8000/health` returned `{"status":"ok"}` (search_service FastAPI).

- **Identified and fixed runtime bug** on EC2: `api_service` crashed with `ECONNREFUSED 127.0.0.1:6379`. Root cause: `REDIS_URL=redis://localhost:6379` but Redis runs in a separate container → hostname must be `redis`. Fix: updated `api_service/src/config/env.validation.ts` default to `REDIS_URL=redis://redis:6379` when running in Docker.

- **Wrote `script/smoke-test.sh`** to run on EC2 after each deployment: calls `/health` on all 3 services, failing if any service does not respond within 30s.

- **Setup CloudWatch Logs:** Created log groups `/ecs/api-service` and `/ecs/search-service` with a 30-day retention period.

- **Setup CloudWatch Alarms:**
  - CPU > 80% for 5 minutes → triggers SNS notification.
  - Health check failing 3 consecutive times → triggers SSM `docker restart` + alert.

- **Created CloudWatch Dashboard** with 4 widgets: CPU/Memory utilization, request count, error rate 4xx/5xx, and container restart count.

- **EC2 Security Hardening:** Disabled password auth, disabled root SSH, installed `ufw` firewall (opening only 22/80/443/3000/8000), enabled `fail2ban` and `unattended-upgrades` for automated security patches.

- **Setup Postgres Daily Backup** to S3 `videoplatform-backup`: `pg_dump` + `aws s3 cp` via cron job at 02:00 UTC.

### Task Summary

| Day | Task | Start Date | Completion Date |
|---|---|---|---|
| 47 | SSH EC2, verify 6 containers; find and fix Redis hostname bug | 20/07/2026 | 20/07/2026 |
| 48 | Write `smoke-test.sh`; verify end-to-end GraphQL flow | 21/07/2026 | 21/07/2026 |
| 49 | Setup CloudWatch Logs + Alarms + Dashboard | 22/07/2026 | 22/07/2026 |
| 50 | Security hardening (SSH, ufw, fail2ban, auto-update) | 23/07/2026 | 23/07/2026 |
| 51 | Postgres backup cron job to S3; verify restore from test backup | 24/07/2026 | 24/07/2026 |

### Outcomes

- Entire CI/CD pipeline runs successfully end-to-end: GitHub push → CodeBuild → ECR → S3 → SSM → EC2 deploy.
- Both services (api + search) + 4 infra containers are healthy on EC2.
- Full monitoring, logging, and alerting in place — incidents can be detected within 5 minutes.
- Automated Postgres backups configured and tested for data restoration.

### Notes

*The Redis hostname bug (`localhost` vs `redis`) is a prime example of "works on my machine" — impossible to detect during local dev because Redis might be running on the host, not in Docker. Recommendation: always use Docker Compose for local dev even when testing just 1 service, to mirror the production networking model. Cost estimate for CloudWatch + SNS + S3 backup is around \$15-25/month with current traffic.*
