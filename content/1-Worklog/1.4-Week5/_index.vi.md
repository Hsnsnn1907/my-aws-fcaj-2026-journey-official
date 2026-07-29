+++
title = "Worklog Tuần 5"
date = 2026-06-12
weight = 5
chapter = false
pre = "<b>1.4.</b>"
+++

## Tuần 5: Khám phá codebase & hiểu tech stack dự án

### Công việc đã làm

- Clone 2 repo dự án về máy local:
  - `VideoPlatformClient` (React 19 + Vite 7 + Tailwind 4 + Radix UI + GraphQL client).
  - `VideoPlatformServer` (NestJS 11 + GraphQL/Apollo 5 + Prisma 7 + BullMQ + Qdrant + S3 + FFmpeg).
- Đọc hiểu cấu trúc sub-projects của server:
  - `Server/` — NestJS gateway xử lý cả GraphQL, video upload, transcode (FFmpeg), vector search (Qdrant), background job (BullMQ).
  - `Embed_Server/` — FastAPI Python service cho text embedding.
  - `r2-worker/` — Cloudflare Worker xử lý upload lên Cloudflare R2.
  - `Infra/{Postgres,Qdrant,Redis}/docker-compose.yml` — local dev stack.
- Build image Docker đầu tiên thành công cho `Server/`, chạy `docker compose up` local stack, verify Postgres + Redis + Qdrant đều lên.
- Học gRPC + Protocol Buffer cơ bản để chuẩn bị cho refactor kiến trúc ở tuần sau.
- Tìm hiểu RabbitMQ exchange/queue/binding model để thiết kế message broker cho microservices.

### Công việc chính

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 17 | Clone repo `VideoPlatformClient` + `VideoPlatformServer` | 08/06/2026 | 08/06/2026 |
| 18 | Đọc cấu trúc `Server/`, `Embed_Server/`, `r2-worker/` | 09/06/2026 | 09/06/2026 |
| 19 | Build Docker image cho `Server/`; chạy local stack | 10/06/2026 | 10/06/2026 |
| 20 | Học gRPC + Protocol Buffer; viết .proto đầu tiên | 11/06/2026 | 11/06/2026 |
| 21 | Học RabbitMQ: exchange, queue, binding | 12/06/2026 | 12/06/2026 |

### Kết quả đạt được

- Hiểu rõ monolith `Server/` đang xử lý quá nhiều trách nhiệm (GraphQL + upload + transcode + vector search + background job).
- Local dev stack chạy thành công: Postgres + Redis + Qdrant + Server đều healthy.
- Đã có base knowledge về gRPC + RabbitMQ để bắt đầu lên kế hoạch tách microservice.

### Ghi chú 

*Quan sát quan trọng: monolith Server hiện tại có coupling cao giữa các concern (auth, video processing, search). Khi scale lên vài nghìn user, FFmpeg transcode có thể starve các request GraphQL. Cần tách thành ít nhất 2 service: api_service (Gateway + GraphQL) và search_service (Qdrant + embedding). Video processing có thể giữ trong api_service hoặc tách thành worker riêng tùy tải thực tế.*
