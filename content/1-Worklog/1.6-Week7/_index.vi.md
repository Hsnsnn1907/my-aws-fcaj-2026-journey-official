+++
title = "Worklog Tuần 7"
date = 2026-06-26
weight = 7
chapter = false
pre = "<b>1.6.</b>"
+++

## Tuần 7: Hoàn thiện workflow push & thiết kế microservices

### Công việc đã làm

- Bổ sung quy tắc "only push on user request" vào `structure.md` (mục Git & push rules mới ở cuối file) và `workflow.md`.
- Cập nhật Permanent rules trong `memory.md` từ 4 rules lên 5 rules, đảm bảo khớp với `workflow.md`.
- Khảo sát tính khả thi của việc tách `Server/` thành 2 service: `api_service/` (NestJS gateway + GraphQL) và `search_service/` (FastAPI + Qdrant + Postgres vector).
- Vẽ sơ đồ kiến trúc mới: 2 service giao tiếp qua gRPC, RabbitMQ làm message broker, mỗi service có Postgres riêng (multi-schema qua Prisma `multiSchema` preview feature).
- Lên kế hoạch 4 phần: (1) AWS Console setup (ECR, Secrets Manager, S3); (2) EC2 setup (IAM role + `deploy.sh`); (3) local code (Dockerfiles + buildspecs); (4) CodeBuild setup + chạy thử.
- Tạo checklist blocker trước khi viết Dockerfile để tránh sai sót (đặc biệt `FIREBASE_PRIVATE_KEY` newline handling).

### Công việc chính

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 27 | Bổ sung quy tắc "only push on user request" vào docs | 22/06/2026 | 22/06/2026 |
| 28 | Đồng bộ Permanent rules giữa `memory.md` và `workflow.md` | 23/06/2026 | 23/06/2026 |
| 29 | Khảo sát tách `Server/` thành `api_service/` + `search_service/` | 24/06/2026 | 24/06/2026 |
| 30 | Thiết kế sơ đồ gRPC + RabbitMQ + multi-schema Postgres | 25/06/2026 | 25/06/2026 |
| 31 | Lên plan 4 phần cho CI/CD + checklist blockers | 26/06/2026 | 26/06/2026 |

### Kết quả đạt được

- Quy trình push lên GitHub được chuẩn hóa chặt chẽ: assistant in ra `git` block, user paste, không tự động `git push`.
- Bản thiết kế kiến trúc mới được stakeholder duyệt, branch `feat/CI-CD` được tạo trên remote làm working branch.
- Xác định các dependency mới cần thêm: gRPC (`grpc-node`, `grpcio-tools`), RabbitMQ client, Prisma multi-schema.

### Ghi chú

*Cảnh báo rủi ro: tách monolith thành microservices cần đảm bảo tính tương thích ngược trong giai đoạn transition. Hai service dùng chung một Postgres instance với schema khác nhau (`core`, `search`) sẽ giúp giảm overhead so với việc tách hẳn 2 DB instance. Plan 4 phần đã được track chi tiết trong memory session#id2 (notes section).*
