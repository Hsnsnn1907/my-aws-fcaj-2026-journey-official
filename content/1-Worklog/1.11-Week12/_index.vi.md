+++
title = "Worklog Tuần 12"
date = 2026-07-31
weight = 12
chapter = false
pre = "<b>1.11.</b>"
+++

## Tuần 12: Documentation, knowledge transfer & nộp báo cáo

### Công việc đã làm

- **Viết lại `sever code/VideoPlatformServer/README.md`** (đã drift so với code hiện tại):
  - Cập nhật project structure tree khớp với layout mới (`api_service` + `search_service` + `docker/`).
  - Thêm port table: `api_service` 3000, `search_service` 8000, postgres 5434/5438, redis 6379/6380, qdrant 6333/6334, rabbitmq 5672/15672, gRPC 50051.
  - Thêm "Quick start" section với docker-compose commands.
  - Thêm "CI/CD flow" section giải thích pipeline từ GitHub push → EC2 deploy.
  - Thêm "Troubleshooting" section với các lỗi thường gặp (Redis hostname, shell-globbing typo, prisma schema commented out, buildspec cwd).

- **Viết `sever code/VideoPlatformServer/OPERATIONS.md`** — hướng dẫn vận hành cho on-call:
  - Cách SSH vào EC2, sudo password ở đâu.
  - Cách xem logs (`docker logs`, CloudWatch).
  - Cách rollback (`aws ecr describe-images` → tag image trước đó → trigger deploy).
  - Cách scale EC2; cách restore Postgres backup từ S3.

- **Dọn dẹp 11 file worklog LaTeX** (`week2.tex` → `week12.tex`) và consolidate thành bảng tổng 12 tuần (tuần 1 = nhận việc, 11 tuần = worklog chính).

- **Quay video demo 3 phút** về flow deploy hoàn chỉnh: GitHub push → CodeBuild → ECR → EC2 → smoke test pass.

- **Viết báo cáo tổng kết 12 tuần thực tập:**
  - Tổng quan dự án VideoPlatform và stack kỹ thuật.
  - Quá trình thực hiện theo tuần (tham chiếu worklog).
  - Kết quả đạt được: CI/CD hoàn chỉnh, 2 service trên EC2, monitoring/logging đầy đủ.
  - Bài học kinh nghiệm (cloud pipeline thực tế, debugging CI/CD, multi-stage Dockerfile).
  - Hướng phát triển tiếp theo (auto-scaling group, multi-region, blue-green deploy, Terraform/IaC).

- **Present kết quả** trước FCAJ group + mentor, nhận feedback, ghi nhận đánh giá cuối kỳ.

- **Clean up GitHub:** đóng các branch cũ (`feat/refactor`, `feat/semantic-search`), giữ `feat/CI-CD` làm base cho người kế thừa.

- **Nộp báo cáo cuối kỳ FCAJ vào 31/07/2026.**

### Tóm tắt công việc

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 52 | Viết lại `README.md` khớp với code hiện tại | 27/07/2026 | 27/07/2026 |
| 53 | Viết `OPERATIONS.md`; dọn 11 file worklog LaTeX | 28/07/2026 | 28/07/2026 |
| 54 | Quay video demo + viết báo cáo tổng kết 12 tuần | 29/07/2026 | 29/07/2026 |
| 55 | Present kết quả + feedback; dọn GitHub branches | 30/07/2026 | 30/07/2026 |
| 56 | Nộp báo cáo cuối kỳ FCAJ | 31/07/2026 | 31/07/2026 |

### Kết quả đạt được

- Documentation đầy đủ, sẵn sàng cho người mới onboard.
- Báo cáo thực tập đã nộp đúng hạn 31/07/2026, đạt yêu cầu FCAJ.
- Repo `VideoPlatformServer` sạch sẽ, `feat/CI-CD` là base ổn định cho người kế thừa.
- Tổng kết 12 tuần: từ zero (orientation + AWS basics) đến deploy production-ready pipeline với monitoring đầy đủ.

### Ghi chú

*Retrospective cá nhân: 12 tuần vừa đủ để hiểu end-to-end một cloud pipeline thực tế — từ thiết kế, code, deploy, monitor. Những bài học giá trị nhất: (1) "works on my machine" là bài học xuyên suốt; (2) review code không thay thế được chạy thật; (3) shell-globbing bug là một class bug tinh vi cần quote mọi version specifier. Hướng tiếp theo: tìm hiểu sâu hơn về Terraform/IaC để replace phần setup thủ công bằng code, và học về Kubernetes nếu muốn scale microservices ra production thật.*

**Kết thúc kỳ thực tập FCAJ: 18/05/2026 — 31/07/2026 (75 ngày, 11 tuần làm việc chính + 1 tuần orientation).**
