+++
title = "Week 12 Worklog"
date = 2026-07-31
weight = 12
chapter = false
pre = "<b>1.11.</b>"
+++

## Week 12: Documentation, Knowledge Transfer & Final Report

### Completed Tasks

- **Rewrote `sever code/VideoPlatformServer/README.md`** (which had drifted from the current codebase):
  - Updated the project structure tree to match the new layout (`api_service` + `search_service` + `docker/`).
  - Added a port table: `api_service` 3000, `search_service` 8000, postgres 5434/5438, redis 6379/6380, qdrant 6333/6334, rabbitmq 5672/15672, gRPC 50051.
  - Added a "Quick start" section with docker-compose commands.
  - Added a "CI/CD flow" section detailing the pipeline from GitHub push to EC2 deploy.
  - Added a "Troubleshooting" section covering common issues (Redis hostname, shell-globbing typo, Prisma schema commented out, buildspec cwd).

- **Wrote `sever code/VideoPlatformServer/OPERATIONS.md`** — an operational guide for on-call engineers:
  - How to SSH into EC2, and where to find the sudo password.
  - How to view logs (`docker logs`, CloudWatch).
  - How to rollback (`aws ecr describe-images` → tag previous image → trigger deploy).
  - How to scale EC2, and how to restore Postgres backups from S3.

- **Cleaned up 11 LaTeX worklog files** (`week2.tex` through `week12.tex`) and consolidated them into a 12-week summary table (week 1 = orientation, 11 main worklog weeks).

- **Recorded a 3-minute demo video** showcasing the complete end-to-end deployment flow: GitHub push → CodeBuild → ECR → EC2 → smoke test pass.

- **Wrote the 12-week internship final report:**
  - Overview of the VideoPlatform project and technical stack.
  - Week-by-week implementation process (referencing worklogs).
  - Outcomes achieved: fully operational CI/CD, 2 services running on EC2, comprehensive monitoring/logging.
  - Lessons learned (real-world cloud pipelines, CI/CD debugging, multi-stage Dockerfiles).
  - Future development directions (auto-scaling groups, multi-region, blue-green deployment, Terraform/IaC).

- **Presented results** to the FCAJ group and mentor, gathered feedback, and recorded final evaluations.

- **Cleaned up GitHub repository:** Closed old branches (`feat/refactor`, `feat/semantic-search`), keeping `feat/CI-CD` as the clean base for the next maintainer.

- **Submitted final FCAJ report on 31/07/2026.**

### Task Summary

| Day | Task | Start Date | Completion Date |
|---|---|---|---|
| 52 | Rewrite `README.md` to match current code | 27/07/2026 | 27/07/2026 |
| 53 | Write `OPERATIONS.md`; clean up 11 LaTeX worklogs | 28/07/2026 | 28/07/2026 |
| 54 | Record demo video + write 12-week summary report | 29/07/2026 | 29/07/2026 |
| 55 | Presentation + feedback; cleanup GitHub branches | 30/07/2026 | 30/07/2026 |
| 56 | Submit final FCAJ report | 31/07/2026 | 31/07/2026 |

### Outcomes

- Comprehensive documentation is complete and ready for new member onboarding.
- The internship report was submitted exactly on deadline (31/07/2026), meeting all FCAJ requirements.
- The `VideoPlatformServer` repo is clean, with `feat/CI-CD` serving as a stable base branch for successors.
- 12-week summary: went from zero (orientation + AWS basics) to deploying a production-ready pipeline with full monitoring.

### Notes

*Personal retrospective: 12 weeks is just enough time to understand a real cloud pipeline end-to-end — from design to code to deployment and monitoring. The most valuable lessons: (1) "Works on my machine" is a persistent theme; (2) code review does not replace running the code; (3) shell-globbing bugs are a subtle class of errors requiring quotes for all version specifiers. Next steps: dive deeper into Terraform/IaC to replace manual setups with code, and learn Kubernetes to scale microservices correctly in true production scenarios.*

**End of FCAJ Internship: 18/05/2026 — 31/07/2026 (75 days, 11 main working weeks + 1 orientation week).**
