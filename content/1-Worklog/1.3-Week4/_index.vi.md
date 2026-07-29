+++
title = "Worklog Tuần 4"
date = 2026-06-05
weight = 4
chapter = false
pre = "<b>1.3.</b>"
+++

## Tuần 4: S3, CloudWatch cơ bản & AWS Hands-on Labs

### Công việc đã làm

- Học S3: bucket, object, version, lifecycle policy, IAM policy cho S3; thực hành `aws s3 mb`, `aws s3 cp`, `aws s3 sync`.
- Học CloudWatch: metrics, logs, alarms; thực hành tạo alarm cho CPU > 80% trên EC2 đã tạo ở tuần 3.
- Hoàn thành AWS Cloud Practitioner Essentials: 10 module + final assessment (dự kiến đạt 850+/1000).
- Làm lab thực hành: launch EC2 với user data bootstrap script, cài nginx, mở port 80.
- Đọc docs Docker cơ bản (container, image, Dockerfile, docker compose) để chuẩn bị cho việc deploy microservice trong các tuần sau.

### Công việc chính

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 12 | Học S3: bucket, lifecycle, IAM policy | 01/06/2026 | 01/06/2026 |
| 13 | Thực hành `aws s3` CLI cơ bản | 02/06/2026 | 02/06/2026 |
| 14 | Học CloudWatch: metrics, logs, alarms | 03/06/2026 | 03/06/2026 |
| 15 | Hoàn thành AWS Cloud Practitioner Essentials final assessment | 04/06/2026 | 04/06/2026 |
| 16 | Lab: launch EC2 + nginx + user data bootstrap; đọc docs Docker | 05/06/2026 | 05/06/2026 |

### Kết quả đạt được

- S3 bucket đầu tiên `fcaj-intern-storage` đã tạo và upload file test thành công.
- CloudWatch alarm cho CPU EC2 đã trigger email notification qua SNS.
- Đạt chứng chỉ nội bộ AWS Cloud Practitioner (dự kiến).
- Hiểu kiến trúc Docker đủ để đọc và sửa Dockerfile cho tuần sau.

### Ghi chú 

*Tuần này chuyển từ "làm quen" sang "nắm vững nền tảng". Nội dung cover đủ để bắt đầu đụng vào dự án thật ở tuần 5. Lưu ý: S3 free tier giới hạn 5GB storage + 15GB transfer out/tháng — không up file lớn trong các lab.*
