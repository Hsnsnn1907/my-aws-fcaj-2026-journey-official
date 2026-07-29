+++
title = "Worklog Tuần 6"
date = 2026-06-19
weight = 6
chapter = false
pre = "<b>1.5.</b>"
+++

## Tuần 6: Khám phá codebase chi tiết & khởi tạo canonical docs

### Công việc đã làm

- Đọc sâu toàn bộ cấu trúc dự án `VideoPlatform`:
  - `client code/VideoPlatformClient/` -- React 19 + Vite 7 + Tailwind 4 + Radix UI + GraphQL client.
  - `sever code/VideoPlatformServer/Server/` -- NestJS 11 + GraphQL/Apollo 5 + Prisma 7 + BullMQ + Qdrant + S3 + FFmpeg.
  - `sever code/VideoPlatformServer/Embed_Server/` -- FastAPI Python service.
  - `sever code/VideoPlatformServer/r2-worker/` -- Cloudflare Worker.
  - `sever code/VideoPlatformServer/Infra/{Postgres,Qdrant,Redis}/docker-compose.yml` -- local dev stack.
- Tạo `agent set up/docs/structure.md` -- cây thư mục đầy đủ, tech stack, sơ đồ data-flow.
- Tạo `agent set up/docs/workflow.md` -- session lifecycle, memory format, push-block conventions, permanent rules (5 rules).
- Khởi tạo `agent set up/claude memory/memory.md` với Permanent rules + session block `session#id1`.
- Đề xuất 5 permanent rules cho memory file: (1) never push `agent set up/`, (2) only push on user request, (3) always read memory first, (4) always increment session id, (5) always show push-block at session end.

### Công việc chính

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 22 | Đọc `package.json` client + server; liệt kê tech stack | 15/06/2026 | 15/06/2026 |
| 23 | Khảo sát `Server/`, `Embed_Server/`, `r2-worker/`, `Infra/` | 16/06/2026 | 16/06/2026 |
| 24 | Viết `structure.md` -- directory tree + data-flow diagram | 17/06/2026 | 17/06/2026 |
| 25 | Viết `workflow.md` (session lifecycle + 5 permanent rules) | 18/06/2026 | 18/06/2026 |
| 26 | Khởi tạo `memory.md` với Permanent rules + session#id1 | 19/06/2026 | 19/06/2026 |

### Kết quả đạt được

- Hiểu rõ monolith hiện tại `Server/` đang là NestJS gateway xử lý cả GraphQL, video upload, transcode (FFmpeg), vector search (Qdrant), background job (BullMQ).
- Tài liệu `structure.md` + `workflow.md` trở thành canonical reference cho mọi session sau.
- Quy trình làm việc với Claude được chuẩn hóa: mọi session đọc memory trước, append session block trước khi kết thúc.
- Đã có base để bắt đầu làm việc thật trên codebase từ tuần 7.

### Ghi chú

*Quy tắc quan trọng được thiết lập từ tuần này: thư mục `agent set up/` chứa config agent-team và KHÔNG BAO GIỜ được push lên GitHub. Mọi session chỉ commit/push khi user yêu cầu rõ ràng. Đây là rào chắn quan trọng nhất trong workflow vì tránh leak config nội bộ và memory ra public.*
