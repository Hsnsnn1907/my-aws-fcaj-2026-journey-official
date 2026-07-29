+++
title = "Worklog Tuần 2"
date = 2026-05-16
weight = 2
chapter = false
pre = "<b>1.1.</b>"
+++

## Tuần 2: Làm quen FCAJ & khởi tạo môi trường AWS

### Công việc đã làm

- Tham gia buổi orientation với các thành viên First Cloud AI Journey (FCAJ), đọc và ghi nhận nội quy / quy định thực tập.
- Tìm hiểu tổng quan AWS, các nhóm dịch vụ cốt lõi (Compute, Storage, Networking, Database, Security).
- Tạo tài khoản AWS Free Tier, cài đặt và cấu hình AWS CLI v2 trên máy local (cấu hình SSO/IAM user với quyền read-only để tránh lộ key).
- Thực hành lệnh AWS CLI cơ bản: `aws sts get-caller-identity`, `aws s3 ls`, `aws ec2 describe-instances`.

### Công việc chính

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 1 | Làm quen thành viên FCAJ; đọc nội quy thực tập (Thứ Ba) | 12/05/2026 | 12/05/2026 |
| 2 | Học tổng quan AWS: Compute / Storage / Networking / Database (Thứ Tư) | 13/05/2026 | 13/05/2026 |
| 3 | Tạo AWS Free Tier account; cài đặt & cấu hình AWS CLI v2 (Thứ Năm) | 14/05/2026 | 14/05/2026 |
| 4 | Thực hành AWS CLI cơ bản (sts, s3, ec2 describe) (Thứ Sáu) | 15/05/2026 | 15/05/2026 |
| 5 | Làm bài quiz AWS Cloud Practitioner module 1-2 (Thứ Bảy) | 16/05/2026 | 16/05/2026 |

### Kết quả đạt được

- Tài khoản AWS Free Tier hoạt động, đã xác thực thành công bằng MFA.
- AWS CLI v2 cài đặt thành công trên máy local, có thể truy vấn thông tin account qua `aws sts get-caller-identity`.
- Hiểu rõ vị trí của 4 nhóm dịch vụ AWS cốt lõi trong bức tranh tổng thể của Cloud.

### Ghi chú

*Tuần đầu tiên chủ yếu là setup nền tảng; chưa chạm vào code dự án. Đây là tuần onboarding với lịch đặc biệt Tue-Sat (12-16/5/2026) do bắt đầu giữa tuần; từ tuần 3 trở đi làm việc Mon-Fri chuẩn. Cần note lại các giới hạn Free Tier (EC2 750h/tháng, S3 5GB) để tránh phát sinh chi phí khi chạy CI/CD ở các tuần sau.*
