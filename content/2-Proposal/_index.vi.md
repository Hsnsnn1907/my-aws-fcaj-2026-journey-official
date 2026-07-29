+++
title = "Đề xuất dự án"
date = 2026-06-26
weight = 2
chapter = false
+++

# Đề xuất dự án / Project Proposal

## Tổng quan dự án / Project Overview

Dự án **VideoPlatform** là một nền tảng chia sẻ & tìm kiếm video ngữ nghĩa (semantic video search) cho phép người dùng upload video, hệ thống tự động xử lý (transcode, trích xuất frame, sinh caption bằng AI) và cho phép tìm kiếm theo nội dung thay vì chỉ theo tag/title. Trong khuôn khổ thực tập FCAJ, tôi đã thiết kế và triển khai **CI/CD pipeline hoàn chỉnh trên AWS** cho backend của dự án, gồm 2 microservice (`api_service` NestJS GraphQL gateway + `search_service` FastAPI vector search) chạy trên EC2, đóng gói qua Docker, build & deploy tự động qua AWS CodeBuild → ECR → S3 → SSM. Use case thực tế: nền tảng video cho cộng đồng content creator Việt Nam; giá trị cốt lõi là rút ngắn thời gian từ "code commit" đến "service chạy trên production" từ vài giờ thủ công xuống còn ~5 phút tự động, đồng thời đảm bảo reproducibility (image bất biến, secret an toàn qua Secrets Manager, rollback một lệnh).

## Mục tiêu / Objectives

- Thiết kế & triển khai CI/CD pipeline hoàn chỉnh cho backend VideoPlatform trên AWS (CodeBuild + ECR + S3 + EC2), thời gian từ git push đến deploy thành công ≤ 10 phút, idempotent và có rollback.
- Tái cấu trúc backend từ monolith NestJS thành 2 microservice tách biệt về trách nhiệm: `api_service` (gateway, GraphQL, video processing) và `search_service` (FastAPI + Qdrant vector DB + Prisma multi-schema Postgres), giao tiếp qua gRPC và RabbitMQ.
- Đảm bảo security best-practice: dùng IAM Role thay vì access key, secret qua AWS Secrets Manager, container image scan on push, SSH key-based + ufw firewall trên EC2, CloudWatch alarm cho CPU/Memory/health check.
- Hoàn thiện observability: logs centralized qua CloudWatch (retention 30 ngày), dashboard với 4 widget (CPU/Memory, request count, error rate, restart count), smoke-test script chạy post-deploy để verify 6 container cùng healthy.

## Vấn đề cần giải quyết / Problem Statement

Trước khi thực hiện dự án, team phát triển VideoPlatform đang gặp 4 vấn đề lớn:

1. Backend là monolith NestJS (`Server/`) xử lý đồng thời GraphQL + video upload + FFmpeg transcode + vector search + background job → CPU contention khi có nhiều video upload đồng thời, latency GraphQL tăng 5-10x.
2. Quy trình deploy thủ công — developer phải SSH vào EC2, `git pull`, `npm install`, `docker compose up`, dễ sai sót và mất 30-60 phút mỗi lần.
3. Không có monitoring — khi service crash, chỉ phát hiện khi user complain.
4. Secret (Firebase private key, DB password) được lưu trong `.env` file local và copy thủ công lên server, nguy cơ lộ cao.

Nhóm đã chọn giải pháp cloud (AWS) thay vì on-premise vì cần elasticity (scale theo traffic video), managed services (RDS, S3, ECR giảm operational overhead), và ecosystem CI/CD tích hợp sẵn (CodeBuild, CodePipeline).

## Kiến trúc giải pháp / Solution Architecture

![Sơ đồ kiến trúc giải pháp](/images/workshop/architecture-overview.svg)

Giải pháp gồm 5 thành phần AWS chính:

1. **GitHub** (source code) — webhook trigger CodeBuild khi push lên branch `feat/CI-CD` với path filter `^api_service/.*$` hoặc `^search_service/.*$`.
2. **AWS CodeBuild** (build) — 2 project riêng (`build-api-service`, `build-search-service`), chạy `buildspec-*.yml` đa phase (`install` → `pre_build` → `build` → `post_build`), build multi-stage Docker image và push lên ECR.
3. **Amazon ECR** (image registry) — 2 private repo (`videoplatform/api-service`, `videoplatform/search-service`) với scan on push + tag immutability.
4. **Amazon S3** (deploy artifacts) — bucket `videoplatform-deploy-artifacts-dsk` chứa `docker-compose.*.yml` được CodeBuild sync sau mỗi build.
5. **Amazon EC2** (compute) — 1 instance `t3.small` Ubuntu 22.04, IAM Role `EC2-Backend-Role` (S3 read + ECR pull + SSM), chạy 6 container qua Docker Compose v2: `postgres` (Prisma multi-schema `core`/`search`), `redis` (cache + BullMQ), `qdrant` (vector DB), `rabbitmq` (message broker), `api_service` (NestJS GraphQL gateway, port 3000), `search_service` (FastAPI, port 8000).

SSM `AWS-RunShellScript` từ CodeBuild trigger `deploy.sh` trên EC2, script này fetch secret từ Secrets Manager, ghi `.env`, pull image mới từ ECR, sync compose từ S3, `docker compose up -d`.

**Data flow:** client upload video qua Cloudflare R2 (worker) → publish event qua RabbitMQ → `api_service` consume → trigger FFmpeg transcode + Prisma insert → publish kết quả qua RabbitMQ → `search_service` embed & index vào Qdrant → client query GraphQL → `api_service` gọi `search_service` qua gRPC (port 50051).

**Lý do chọn:** CodeBuild thay vì Jenkins vì zero-server-maintenance + native AWS integration; ECR thay vì DockerHub vì private + IAM-auth + nằm trong VPC; multi-service trên 1 EC2 thay vì ECS/EKS vì cost cho internship budget và traffic thấp; Prisma multi-schema thay vì 2 DB instance vì giảm overhead và đơn giản hóa Prisma migration.

## Timeline

| Giai đoạn / Phase | Công việc / Tasks | Deliverable |
|---|---|---|
| Phase 1 (Week 2-3) | Tìm hiểu AWS fundamentals (Compute, Storage, Networking, Database, Security); tạo tài khoản AWS Free Tier, cài đặt AWS CLI v2; launch EC2 instance đầu tiên; tạo IAM Role `EC2-Backend-Role`. | Báo cáo AWS Cloud Practitioner assessment; EC2 chạy ổn định với key-based SSH. |
| Phase 2 (Week 4-6) | Khám phá codebase `VideoPlatformClient` (React 19) + `VideoPlatformServer` (NestJS); khởi tạo `structure.md`, `workflow.md`, `memory.md`; thiết lập workflow làm việc với Claude (5 permanent rules). | Tài liệu `structure.md` + `workflow.md` làm canonical reference; quy trình push lên GitHub chuẩn hóa. |
| Phase 3 (Week 7-9) | Thiết kế kiến trúc 2 microservice (`api_service` + `search_service`), gRPC + RabbitMQ; tạo ECR repos, S3 bucket, Secrets Manager secret; switch local branch sang `feat/CI-CD`; viết `Dockerfile` multi-stage + `entrypoint` + `deploy.sh` + `buildspec-*.yml`; commit + push `d3f106e` + `0972102`. | Toàn bộ CI/CD artifacts (12 file mới + 4 modified) sống trên `origin/feat/CI-CD`; docker-compose có đầy đủ 6 service. |
| Phase 4 (Week 10) | Tạo 2 CodeBuild project trên AWS Console; bootstrap EC2 (fix `sudo $USER` bug); trigger `build-search-service` lần đầu; fix 4 build bug liên tiếp (.dockerignore, Dockerfile requirements.txt, prisma schema uncomment, buildspec `cd $CODEBUILD_SRC_DIR`). | 4 commits `f16d5ea`, `f4f2bd3`, `1c52793`, `1bf28e7` trên GitHub; `build-search-service` và `build-api-service` đều BUILD SUCCEEDED. |
| Phase 5 (Week 11) | End-to-end verify trên EC2: 6 container chạy healthy, GraphQL + FastAPI + MinIO response OK; fix Redis hostname bug (`localhost` → `redis`); viết `smoke-test.sh`; setup CloudWatch Logs/Alarms/Dashboard; security hardening EC2; Postgres backup hàng ngày lên S3. | CI/CD pipeline chạy ổn định từ GitHub push đến EC2 deploy trong ~5 phút; monitoring đầy đủ; backup tự động. |
| Phase 6 (Week 12) | Viết lại `README.md` và `OPERATIONS.md`; dọn 11 file worklog LaTeX; quay video demo 3 phút; viết báo cáo tổng kết 12 tuần; present kết quả; nộp báo cáo cuối kỳ FCAJ. | Báo cáo thực tập nộp ngày 31/07/2026; repo sạch sẵn sàng cho người kế thừa. |

## Ngân sách / Budget

Ước tính chi phí AWS hàng tháng cho stack này (ngoài Free Tier):
- **EC2 t3.small** $15/tháng (730h × $0.0204/h, Ubuntu on-demand, region `us-east-1`)
- **ECR storage** $0.10/GB/tháng × ~5GB image layer = $0.50
- **S3** cho `videoplatform-deploy-artifacts` (compose files ~50KB total) < $0.01
- **S3** cho `videoplatform-backup` (Postgres dump ~200MB/ngày × 30 ngày = 6GB, $0.023/GB/tháng) = $0.14
- **CloudWatch Logs** $0.50/GB ingested, với logging level INFO và 6 container × ~50MB/ngày × 30 ngày = ~9GB = $4.50
- **CloudWatch Alarms + SNS** $0.10/alarm × 4 alarms + $0.06/1000 notifications = $0.50
- **Data transfer out** từ EC2 đến Internet (GraphQL response) ước ~10GB/tháng × $0.09/GB = $0.90
- **CodeBuild** $0.005/build-min × 2 service × 30 builds/tháng × ~5 min = $1.50

**Tổng ước tính: ~$23/tháng**, thu gọn trong AWS Free Tier cho 12 tháng đầu. Ước tính chi phí duy trì sau này: **~$280/năm** sau khi hết thời hạn dùng thử Free Tier. 

Cách kiểm soát chi phí: (1) set CloudWatch log retention 30 ngày (không giữ vĩnh viễn); (2) dùng `t3.small` thay vì `t3.medium` vì traffic thấp; (3) set budget alarm ở $30/tháng để cảnh báo sớm; (4) tắt EC2 ngoài giờ làm việc nếu không cần dev/test (start/stop thủ công); (5) review AWS Cost Explorer hàng tháng để phát hiện bất thường.

## Rủi ro / Risks

| Rủi ro / Risk | Tác động / Impact | Giảm thiểu / Mitigation |
|---|---|---|
| Rủi ro kỹ thuật: **Dockerfile hoặc buildspec sai** (4 bug build liên tiếp đã thấy trong tuần 10 — `.dockerignore` match file đúng, thiếu `COPY requirements.txt`, schema.prisma comment-out, `cd $CODEBUILD_SRC_DIR`). | Build fail, không deploy được service; thời gian debug kéo dài (mỗi lần build chạy ~5 phút). | (1) Audit Dockerfile + buildspec + dockerignore trước khi trigger build lần đầu; (2) layer cache giúp chỉ re-run step bị lỗi, không build lại từ đầu; (3) test local với cùng Docker context trước khi push; (4) viết unit test cho entrypoint script; (5) dùng CI/CD lint tool (hadolint cho Dockerfile). |
| Rủi ro chi phí: **Vượt Free Tier** do image quá lớn hoặc logs quá nhiều. | Phát sinh chi phí ngoài kế hoạch ($50-100/tháng). | (1) Set `.dockerignore` chặt (đã có cho `node_modules`, `.git`, `=*` stray files); (2) CloudWatch log retention 30 ngày; (3) AWS Budget alarm ở 50% và 100% ngưỡng; (4) compress Postgres backup trước khi upload S3; (5) review Cost Explorer hàng tuần. |
| Rủi ro bảo mật: **Lộ secret qua image layer** (ví dụ: `FIREBASE_PRIVATE_KEY` nếu COPY nhầm vào `Dockerfile` thay vì inject runtime). | Lộ credential production, có thể compromise toàn bộ hệ thống Firebase/DB. | (1) KHÔNG COPY bất kỳ file `.env` nào vào image (đã enforce qua `.dockerignore`); (2) secret chỉ inject runtime qua `deploy.sh` từ Secrets Manager; (3) `ECR scan on push` sẽ cảnh báo nếu detect key pattern; (4) rotate secret theo định kỳ (90 ngày); (5) dùng IMDSv2 trên EC2 thay vì v1. |
| Rủi ro downtime: **EC2 instance fail** hoặc `docker compose up` lỗi. | Service ngừng hoạt động, user không upload/query được video. | (1) `smoke-test.sh` post-deploy verify 6 container healthy, fail sẽ rollback tự động về image trước; (2) CloudWatch alarm restart container nếu health check fail 3 lần; (3) backup Postgres hàng ngày, restore trong ~30 phút nếu DB corrupt; (4) manual rollback: `aws ecr describe-images` → tag image cũ → trigger deploy; (5) trong tương lai chuyển sang ECS/EKS với auto-replace task khi fail. |
| Rủi ro coordination: **Branch `feat/CI-CD` diverge** với `main` nếu nhiều người cùng push. | Merge conflict, mất commit. | (1) Quy ước: mọi thay đổi CI/CD phải qua PR review; (2) rebase `feat/CI-CD` về `main` mỗi sprint; (3) sau khi CI/CD ổn định, merge `feat/CI-CD` vào `main` rồi archive branch cũ. |
