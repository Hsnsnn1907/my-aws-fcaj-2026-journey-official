+++
title = "Week 5 Worklog"
date = 2026-06-12
weight = 5
chapter = false
pre = "<b>1.4.</b>"
+++

## Week 5: Exploring the Codebase & Understanding the Project Stack

### Completed Tasks

- Cloned 2 project repositories to the local machine:
  - `VideoPlatformClient` (React 19 + Vite 7 + Tailwind 4 + Radix UI + GraphQL client).
  - `VideoPlatformServer` (NestJS 11 + GraphQL/Apollo 5 + Prisma 7 + BullMQ + Qdrant + S3 + FFmpeg).
- Analyzed the sub-project structure of the server:
  - `Server/` — NestJS gateway handling GraphQL, video uploads, transcoding (FFmpeg), vector search (Qdrant), and background jobs (BullMQ).
  - `Embed_Server/` — FastAPI Python service for text embedding.
  - `r2-worker/` — Cloudflare Worker handling uploads to Cloudflare R2.
  - `Infra/{Postgres,Qdrant,Redis}/docker-compose.yml` — local dev stack.
- Successfully built the first Docker image for `Server/`, ran the `docker compose up` local stack, and verified that Postgres + Redis + Qdrant are up and running.
- Learned fundamentals of gRPC + Protocol Buffers in preparation for the architecture refactoring next week.
- Researched RabbitMQ exchange/queue/binding models to design the message broker for microservices.

### Key Tasks Completed

| Day | Task | Start Date | Completion Date |
|---|---|---|---|
| 17 | Cloned `VideoPlatformClient` + `VideoPlatformServer` repositories | 08/06/2026 | 08/06/2026 |
| 18 | Analyzed structure of `Server/`, `Embed_Server/`, `r2-worker/` | 09/06/2026 | 09/06/2026 |
| 19 | Built Docker image for `Server/`; ran local stack | 10/06/2026 | 10/06/2026 |
| 20 | Learned gRPC + Protocol Buffers; wrote the first .proto file | 11/06/2026 | 11/06/2026 |
| 21 | Learned RabbitMQ: exchanges, queues, bindings | 12/06/2026 | 12/06/2026 |

### Outcomes

- Realized that the `Server/` monolith currently handles too many responsibilities (GraphQL + upload + transcode + vector search + background jobs).
- Local dev stack ran successfully: Postgres + Redis + Qdrant + Server are all healthy.
- Established a base knowledge of gRPC + RabbitMQ to begin planning the microservices migration.

### Notes

*Important observation: the current Server monolith has high coupling between concerns (auth, video processing, search). When scaling to thousands of users, FFmpeg transcoding could starve the GraphQL requests. It needs to be split into at least 2 services: api_service (Gateway + GraphQL) and search_service (Qdrant + embedding). Video processing could remain in api_service or be separated into its own worker depending on the actual load.*
