+++
title = "3. Bài blog đã đăng"
date = 2026-07-01
weight = 3
chapter = false
+++

| No. | Chủ đề / Topic | Tóm tắt / Summary | Link |
|---|---|---|---|
| 1 | Xây dựng CI/CD pipeline với AWS CodeBuild + ECR + S3 + SSM cho microservice Node.js & Python | Hướng dẫn end-to-end cách thiết lập pipeline từ GitHub push đến EC2 deploy trong vòng 5 phút, gồm 2 service song song; kèm 4 lỗi build thường gặp và cách fix. | [Facebook Post](https://www.facebook.com/groups/awsstudygroupfcj/posts/2224001141698179/?notif_id=1785108354168202&notif_t=tagged_with_story&ref=notif) |
| 2 | Tối ưu chi phí & tăng tốc Docker image với multi-stage build + Amazon ECR scan on push | Cách cắt giảm 70% dung lượng image qua multi-stage build, dùng `.dockerignore` chặt, bật ECR scan on push để phát hiện lỗ hổng sớm; kèm so sánh chi phí. | Placeholder |
| 3 | Zero-secret deployment với AWS Secrets Manager + IAM Role + IMDSv2: bài học từ `FIREBASE_PRIVATE_KEY` | Cách loại bỏ hoàn toàn `.env` file khỏi deployment workflow, dùng Secrets Manager + EC2 IAM Role + Python heredoc thay vì `jq` để tránh corrupt secret. | [Facebook Post](https://www.facebook.com/groups/awsstudygroupfcj/posts/2226205278144432/?notif_id=1785326007150392&notif_t=tagged_with_story&ref=notif) |