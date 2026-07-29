+++
title = "Week 9 Worklog"
date = 2026-07-10
weight = 9
chapter = false
pre = "<b>1.8.</b>"
+++

## Week 9: Branch Switch, Refactor & CI/CD Artifacts Creation

### Completed Tasks

- **Monday (06/07)**: Performed destructive branch switch on `VideoPlatformServer`:
  - `git fetch origin feat/CI-CD`
  - `git branch -m main feat/CI-CD` (renamed local main to feat/CI-CD)
  - `git reset --hard origin/feat/CI-CD` (HEAD = `5788515`)
  - `git branch -u origin/feat/CI-CD feat/CI-CD`
  - Note: 5 old commits on local `main` (`0f6c453`, `62d7e94`, ...) remain only in reflog for ~30 days.
- **Monday (cont.)**: Rewrote `structure.md` for the new layout (`api_service/` + `search_service/` + `proto/` + `docker/`); rewrote `README.md` for `VideoPlatformServer`; cleaned up stray file `search_service/=1.68` (shell-globbing bug); updated docs to remove cleanup items.
- **Tuesday (07/07)**: Analyzed 8 blockers from the CI/CD plan and devised a strategy:
  1. `docker-compose.api-service.yml` missing `api_service` service.
  2. `docker-compose.search-service.yml` missing `search_service` service.
  3. No `Dockerfile` for either service.
  4. `prisma migrate deploy` not running on container start.
  5. Build context too large (needs `.dockerignore`).
  6. EC2 has no git credentials (use S3 instead of `git pull`).
  7. `docker-compose` v1 deprecated (switch to v2).
  8. `jq` corrupts newlines in `FIREBASE_PRIVATE_KEY` (use Python heredoc).
- **Wednesday (08/07)**: Wrote all CI/CD artifacts:
  - `api_service/Dockerfile` (multi-stage Node 18-alpine).
  - `api_service/.dockerignore`, `api_service/docker-entrypoint.sh`.
  - `search_service/Dockerfile` (multi-stage python:3.10-slim).
  - `search_service/.dockerignore`, `search_service/docker-entrypoint.sh`.
  - `script/deploy.sh` (S3-based, no git pull).
  - `script/ec2-bootstrap.sh`, `script/sync-compose-to-s3.sh`.
  - `buildspec-api.yml`, `buildspec-search.yml`.
- **Thursday (09/07)**: Updated placeholder S3 bucket name in both scripts (`videoplatform-deploy-artifacts-dsk`). Commit `d3f106e` combining 11 new files + 3 modified, pushed to `origin/feat/CI-CD`.
- **Friday (10/07)**: Audited all CI/CD files on disk, discovered 6 issues (stray `api_service/=1.68`, stale guard, cosmetic newline, 2 important issues about load_dotenv/prisma generate, cleanup pattern). Commit `0972102` fixing 6 issues, pushed to GitHub.

### Key Tasks

| Day | Task | Start | Complete |
|---|---|---|---|
| 37 | Switch branch to `feat/CI-CD`; rewrite docs; delete stray file | 06/07/2026 | 06/07/2026 |
| 38 | Analyze 8 blockers; devise CI/CD strategy | 07/07/2026 | 07/07/2026 |
| 39 | Write Dockerfiles, entrypoints, `deploy.sh`, buildspecs | 08/07/2026 | 08/07/2026 |
| 40 | Update S3 bucket; commit `d3f106e`; push to GitHub | 09/07/2026 | 09/07/2026 |
| 41 | Audit 6 issues; commit `0972102`; push | 10/07/2026 | 10/07/2026 |

### Outcomes

- Local branch `feat/CI-CD` tracking `origin/feat/CI-CD` is synchronized.
- 2 major commits (`d3f106e`, `0972102`) are live on GitHub.
- All Dockerfiles + buildspecs + deploy scripts + entrypoints are ready for the first real build in Week 10.
- Pre-build audit helped catch 6 blockers early, preventing them from leaking into the pipeline.

### Notes

*Major lesson: the latent bug `sudo usermod -aG docker "$USER"` in `ec2-bootstrap.sh` -- when run with `sudo`, `$USER` resolves to `root`, so the docker group is added to root instead of `ubuntu`. Fixed with `REAL_USER=${SUDO_USER:-$(logname 2>/dev/null || echo ubuntu)}`. The shell-globbing bug when typing `pip install package==X.Y` also recurred in `api_service/` -- the pattern was blocked by adding `=*` to `.dockerignore` and `.gitignore`.*
