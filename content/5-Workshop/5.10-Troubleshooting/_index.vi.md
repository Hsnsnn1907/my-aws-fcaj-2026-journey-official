+++
title = "Khắc phục sự cố"
date = 2026-08-03
weight = 10
chapter = false
+++

## Hướng dẫn Khắc phục sự cố

### Vấn đề 1: CodeBuild lỗi "docker-entrypoint.sh không tìm thấy"

**Nguyên nhân**: `.dockerignore` loại bỏ entrypoint script
**Khắc phục**: Xóa `docker-entrypoint.sh` khỏi `.dockerignore`

### Vấn đề 2: Prisma migrations thất bại trong container

**Nguyên nhân**: `DATABASE_URL_HOST` và `DATABASE_URL_PORT` không được set
**Khắc phục**: Parse `DATABASE_URL` trong `deploy.sh`

### Vấn đề 3: S3 sync thất bại "docker/ không tồn tại"

**Nguyên nhân**: `cd search_service` trong BUILD phase vẫn tồn tại trong POST_BUILD
**Khắc phục**: Thêm `cd $CODEBUILD_SRC_DIR` trước `aws s3 sync`

### Vấn đề 4: EC2 deployment stuck "Pulling image..."

**Nguyên nhân**: Docker Hub rate limit
**Khắc phục**: Login vào ECR trước `docker compose pull`

### Vấn đề 5: SSM command hết giờ

**Nguyên nhân**: EC2 không đăng ký với SSM hoặc IAM role thiếu
**Khắc phục**: Đảm bảo EC2 có policy `AmazonSSMManagedInstanceCore`

### Vấn đề 6: Secrets Manager access denied

**Nguyên nhân**: EC2 IAM role thiếu quyền Secrets Manager
**Khắc phục**: Thêm policy `SecretsManagerReadWrite` vào EC2 role

### Các lệnh debug

```bash
# Kiểm tra build details
aws codebuild batch-get-builds --ids build-id

# Kiểm tra SSM command
aws ssm list-commands --instance-id i-037a4cd636a68eb7e

# Xem logs container trên EC2
sudo docker logs api_service --tail 100

# Kiểm tra mạng Docker
sudo docker network ls
```

Với kiến thức khắc phục sự cố, hãy đi đến Các bước tiếp theo.