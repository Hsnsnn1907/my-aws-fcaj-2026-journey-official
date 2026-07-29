+++
title = "Worklog Tuần 8"
date = 2026-07-03
weight = 8
chapter = false
pre = "<b>1.7.</b>"
+++

## Tuần 8: AWS Console setup cho CI/CD pipeline

### Công việc đã làm

- Tạo ECR repositories cho 2 service trên AWS Console:
  - `videoplatform/api-service` (private).
  - `videoplatform/search-service` (private).
  - Cấu hình scan on push, tag immutability.
- Tạo S3 bucket `videoplatform-deploy-artifacts-dsk` dùng để ship compose files từ CodeBuild đến EC2 qua `aws s3 sync`.
- Tạo secret trong AWS Secrets Manager `videoplatform/backend-secrets` chứa:
  - `DATABASE_URL` (Postgres connection string).
  - `RABBITMQ_URL`.
  - `REDIS_URL`.
  - `QDRANT_URL`.
  - `FIREBASE_PRIVATE_KEY` (với `\n` escapes).
  - `FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, v.v.
- Tạo IAM role `EC2-Backend-Role` gắn vào EC2 (đã launch ở tuần 3) với policy:
  - `AmazonS3ReadOnlyAccess` (để pull compose files).
  - `AmazonEC2ContainerRegistryPowerUser` (để pull images).
  - `AmazonSSMFullAccess` (để nhận command từ CodeBuild).
  - `SecretsManagerReadWrite` (để đọc secret qua SSM parameter).
- Document IAM role ARN và secret ARN để dùng cho buildspec ở tuần 9.

### Công việc chính

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 32 | Tạo 2 ECR repo (api-service, search-service) với scan on push | 29/06/2026 | 29/06/2026 |
| 33 | Tạo S3 bucket `videoplatform-deploy-artifacts-dsk` | 30/06/2026 | 30/06/2026 |
| 34 | Tạo secret `videoplatform/backend-secrets` trong Secrets Manager | 01/07/2026 | 01/07/2026 |
| 35 | Attach IAM role `EC2-Backend-Role` vào EC2 instance | 02/07/2026 | 02/07/2026 |
| 36 | Document ARN + secret keys chuẩn bị cho buildspec tuần sau | 03/07/2026 | 03/07/2026 |

### Kết quả đạt được

- Toàn bộ AWS-side infrastructure cho CI/CD đã sẵn sàng: ECR repos chứa image, S3 bucket chứa compose files, Secrets Manager chứa credentials.
- IAM role đã attach cho EC2, EC2 có thể pull image từ ECR và đọc secret qua instance metadata.
- Không còn phải SSH vào EC2 để thiết lập thủ công -- mọi thứ được quản lý qua AWS Console + CLI.

### Ghi chú

*Bài học về `FIREBASE_PRIVATE_KEY`: phải dùng `\n` escapes (literal backslash-n) thay vì newline thật khi lưu vào JSON secret. Admin SDK parse chuỗi `"-----BEGIN PRIVATE KEY-----\nMIIE..."` thành PEM hợp lệ; nếu dùng newline thật, JSON parser fail hoặc env file bị corrupt. Đây là bug classic mà hầu hết các tutorial bỏ qua. Lưu lại vào `script/deploy.sh` ở tuần sau sẽ dùng Python heredoc thay vì `jq` để tránh newline corruption.*
