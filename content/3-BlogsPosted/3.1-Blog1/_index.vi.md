+++
title = "3.1. Blog 1: Xây dựng CI/CD pipeline với AWS CodeBuild + ECR + S3 + SSM"
date = 2026-07-01
weight = 1
chapter = false
+++

## Blog 1: Xây dựng CI/CD pipeline với AWS CodeBuild + ECR + S3 + SSM

**Giới thiệu**
Blog này chia sẻ kinh nghiệm thực tế khi xây dựng pipeline CI/CD cho dự án VideoPlatform (Node.js 18 + NestJS 11 và Python 3.10 + FastAPI). Toàn bộ flow đến khi service chạy trên EC2 chỉ mất khoảng 5 phút.

**Những điểm chính cần biết:**
- **Kiến trúc pipeline 4 bước:** GitHub webhook $\rightarrow$ CodeBuild $\rightarrow$ ECR push $\rightarrow$ S3 sync $\rightarrow$ SSM trigger.
- **CodeBuild:** Môi trường ephemeral, trả phí theo phút ($0.005/minute), tiết kiệm hơn tự host Jenkins.
- **Hai CodeBuild project song song:** Dùng Webhook Filter path-based để build tách biệt 2 vi dịch vụ.
- **4 Loại Bug Điển Hình:**
  - Bug 1: `.dockerignore` chặn nhầm entrypoint.
  - Bug 2: Quên `COPY requirements.txt` ở stage 2.
  - Bug 3: `prisma/schema.prisma` bị comment vô tận.
  - Bug 4: Trôi dạt working directory ở `post_build`. Phải reset về `$CODEBUILD_SRC_DIR`.
  - Bug TCP Wait: Validate Database qua entrypoint shell `while ! nc -z db 5432; do sleep 0.5; done` chứ không đặt chặn trong `Dockerfile`.
- Mẹo sử dụng flag `--progress=plain` và `docker run --rm -it $IMAGE_ID sh` để debug.

**Bài học rút ra**
Điều quan trọng nhất là đừng cố review code thay thế việc chạy thật. 4 bug khác nhau chỉ lộ diện sau khi trigger CodeBuild lần đầu ở tuần 10. Luôn chạy thật càng sớm càng tốt. Tận dụng Docker layer cache để debug sửa từ 15 phút xuống 1 phút...

![Hình minh hoạ Blog 1](../../images/blog_1.png)