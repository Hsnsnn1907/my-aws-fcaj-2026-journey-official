+++
title = "Week 6 Worklog"
date = 2026-06-19
weight = 6
chapter = false
pre = "<b>1.5.</b>"
+++

## Week 6: Deep Codebase Exploration & Canonical Docs Initialization

### Completed Tasks

- Performed an in-depth read of the entire `VideoPlatform` project structure:
  - `client code/VideoPlatformClient/` -- React 19 + Vite 7 + Tailwind 4 + Radix UI + GraphQL client.
  - `sever code/VideoPlatformServer/Server/` -- NestJS 11 + GraphQL/Apollo 5 + Prisma 7 + BullMQ + Qdrant + S3 + FFmpeg.
  - `sever code/VideoPlatformServer/Embed_Server/` -- FastAPI Python service.
  - `sever code/VideoPlatformServer/r2-worker/` -- Cloudflare Worker.
  - `sever code/VideoPlatformServer/Infra/{Postgres,Qdrant,Redis}/docker-compose.yml` -- local dev stack.
- Created `agent set up/docs/structure.md` -- complete directory tree, tech stack, and data-flow diagram.
- Created `agent set up/docs/workflow.md` -- session lifecycle, memory format, push-block conventions, permanent rules (5 rules).
- Initialized `agent set up/claude memory/memory.md` with Permanent rules and session block `session#id1`.
- Proposed 5 permanent rules for the memory file: (1) never push `agent set up/`, (2) only push on user request, (3) always read memory first, (4) always increment session id, (5) always show push-block at session end.

### Key Tasks

| Day | Task | Start | Complete |
|---|---|---|---|
| 22 | Read `package.json` for client + server; list tech stack | 15/06/2026 | 15/06/2026 |
| 23 | Survey `Server/`, `Embed_Server/`, `r2-worker/`, `Infra/` | 16/06/2026 | 16/06/2026 |
| 24 | Write `structure.md` -- directory tree + data-flow diagram | 17/06/2026 | 17/06/2026 |
| 25 | Write `workflow.md` (session lifecycle + 5 permanent rules) | 18/06/2026 | 18/06/2026 |
| 26 | Initialize `memory.md` with Permanent rules + session#id1 | 19/06/2026 | 19/06/2026 |

### Outcomes

- Gained a clear understanding that the current `Server/` monolith is a NestJS gateway handling GraphQL, video upload, transcoding (FFmpeg), vector search (Qdrant), and background jobs (BullMQ).
- `structure.md` + `workflow.md` became the canonical reference for all subsequent sessions.
- The workflow with Claude was standardized: every session reads memory first and appends a session block before closing.
- Established the base to begin real codebase work from Week 7.

### Notes

*A critical rule was established this week: the `agent set up/` directory contains agent-team configs and must NEVER be pushed to GitHub. Every session only commits/pushes when the user explicitly requests it. This is the most important safeguard in the workflow to prevent leaking internal configs and memory to public repos.*
