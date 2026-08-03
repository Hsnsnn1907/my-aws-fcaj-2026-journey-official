+++
title = "Deploy Sản Xuất"
date = 2026-08-03
weight = 7
chapter = false
+++

## Deploy Sản Xuất (Tuần 11-12)

### Test Pipeline Đầy đủ

Push một thay đổi code để test:

```bash
git add api_service/src/video/video.service.ts
git commit -m "feat: add video transcoding queue"
git push origin feat/CI-CD
```

**Timeline triển khai:**
- Webhook trigger: < 5 giây
- Docker build: 2-3 phút
- Push ECR: 30 giây
- Sync S3: 5 giây
- SSM trigger: 10 giây
- EC2 deploy: 1 phút
- **Tổng**: ~4-5 phút

### Checklist Xác minh

- [ ] **Services khỏe**: `docker ps` hiển thị tất cả containers chạy
- [ ] **Logs sạch**: `docker logs <service>` không có lỗi
- [ ] **API trả về**: `curl http://98.81.144.110:3000/graphql`
- [ ] **Search service connected**: Kiểm tra consumer logs RabbitMQ
- [ ] **Migrations áp dụng**: Kiểm tra Prisma migrate logs
- [ ] **Secrets tải đúng**: Verify env vars trong containers

### Quy trình Rollback

Nếu deployment thất bại:

```bash
# Trên EC2
cd /app/compose
docker compose -f docker-compose.api-service.yml down
docker pull 772706200692.dkr.ecr.us-east-1.amazonaws.com/api-service:<previous-tag>
docker compose -f docker-compose.api-service.yml up -d
```

### Best Practices

1. **Giám sát trong deploy**: Theo dõi logs và metrics
2. **Health checks**: Cấu hình và sử dụng container health checks
3. **Graceful shutdown**: Cho phép containers hoàn thành request trước khi dừng
4. **Notifications**: Cấu hình alerts khi deploy thất bại

Với deploy sản xuất đã thiết lập, hãy đi đến Giám sát.