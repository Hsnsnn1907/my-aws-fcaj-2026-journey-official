+++
title = "Week 7 Worklog"
date = 2026-06-26
weight = 7
chapter = false
pre = "<b>1.6.</b>"
+++

## Week 7: Push Workflow Finalization & Microservices Design

### Completed Tasks

- Added the "only push on user request" rule to `structure.md` (new Git & push rules section at the end of the file) and `workflow.md`.
- Updated Permanent rules in `memory.md` from 4 rules to 5 rules, ensuring alignment with `workflow.md`.
- Assessed the feasibility of splitting `Server/` into 2 services: `api_service/` (NestJS gateway + GraphQL) and `search_service/` (FastAPI + Qdrant + Postgres vector).
- Designed a new architecture diagram: 2 services communicating via gRPC, RabbitMQ as message broker, each service with its own Postgres schema (multi-schema via Prisma `multiSchema` preview feature).
- Created a 4-part plan: (1) AWS Console setup (ECR, Secrets Manager, S3); (2) EC2 setup (IAM role + `deploy.sh`); (3) local code (Dockerfiles + buildspecs); (4) CodeBuild setup + test run.
- Created a blocker checklist before writing Dockerfiles to prevent mistakes (especially `FIREBASE_PRIVATE_KEY` newline handling).

### Key Tasks

| Day | Task | Start | Complete |
|---|---|---|---|
| 27 | Add "only push on user request" rule to docs | 22/06/2026 | 22/06/2026 |
| 28 | Synchronize Permanent rules between `memory.md` and `workflow.md` | 23/06/2026 | 23/06/2026 |
| 29 | Assess splitting `Server/` into `api_service/` + `search_service/` | 24/06/2026 | 24/06/2026 |
| 30 | Design gRPC + RabbitMQ + multi-schema Postgres architecture | 25/06/2026 | 25/06/2026 |
| 31 | Create 4-part CI/CD plan + blocker checklist | 26/06/2026 | 26/06/2026 |

### Outcomes

- The GitHub push workflow was strictly standardized: the assistant outputs a `git` block, the user pastes it -- no automatic `git push`.
- The new architecture design was approved by stakeholders; the `feat/CI-CD` branch was created on the remote as the working branch.
- Identified new dependencies to add: gRPC (`grpc-node`, `grpcio-tools`), RabbitMQ client, Prisma multi-schema.

### Notes

*Risk warning: splitting the monolith into microservices requires maintaining backward compatibility during the transition period. Two services sharing a single Postgres instance with different schemas (`core`, `search`) reduces overhead compared to maintaining two separate DB instances. The 4-part plan has been tracked in detail within memory session#id2 (notes section).*
