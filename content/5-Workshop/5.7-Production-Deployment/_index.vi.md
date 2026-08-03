+++
title = "5.7. Triển khai Production"
date = 2026-08-03
weight = 7
chapter = false
+++

## Triển khai Production (Tuần 11-12)

Phần này bao gồm toàn bộ quy trình deploy production, các quy trình xác minh và khả năng rollback cho CI/CD pipeline.

### 6.1 Kiểm thử Pipeline đầy đủ

Push một thay đổi code để kiểm thử pipeline deploy hoàn chỉnh:

```bash
# Thực hiện một thay đổi code
git add api_service/src/video/video.service.ts
git commit -m "feat: add video transcoding queue"
git push origin feat/CI-CD
```

**Dòng thời gian thực thi pipeline hoàn chỉnh:**

| Bước | Thời lượng | Hành động |
|------|----------|--------|
| 1 | < 5 giây | GitHub webhook kích hoạt CodeBuild |
| 2 | 2-3 phút | CodeBuild build Docker image |
| 3 | 30 giây | Push lên ECR |
| 4 | 5 giây | Đồng bộ compose files lên S3 |
| 5 | 10 giây | SSM kích hoạt EC2 deploy.sh |
| 6 | 1 phút | EC2 pull image và khởi động lại container |
| **Tổng** | **~4-5 phút** | Triển khai hoàn chỉnh |

**Tổng thời gian: ~4-5 phút** (so với deploy thủ công: 20-30 phút)

![Production Deployment](/my-aws-fcaj-2026-journey-official/images/5-Workshop/12_ec2_production_deployment.png)

### 6.2 Checklist xác minh

Sau khi deploy, xác minh tất cả thành phần đang hoạt động đúng:

#### Xác minh hạ tầng (Infrastructure)
- [ ] **Services healthy**: `docker ps` hiển thị tất cả container đang chạy
- [ ] **Không có crash loop**: Số lần restart container thấp (0-1)
- [ ] **Sử dụng tài nguyên**: CPU và memory nằm trong ngưỡng mong đợi
- [ ] **Kết nối mạng**: Các service có thể kết nối với nhau

#### Xác minh ứng dụng (Application)
- [ ] **API phản hồi**: `curl http://98.81.144.110:3000/graphql`
- [ ] **Search service đã kết nối**: Kiểm tra consumer logs để xác nhận kết nối RabbitMQ
- [ ] **Database migrations đã áp dụng**: Kiểm tra logs Prisma migrate
- [ ] **Secrets đã load đúng**: Xác minh biến môi trường bên trong container

#### Xác minh dịch vụ AWS (AWS Service)
- [ ] **CloudWatch metrics**: CPU, memory, network nằm trong ngưỡng mong đợi
- [ ] **S3 artifacts**: Xác minh compose files đã được đồng bộ đúng
- [ ] **ECR images**: Image mới nhất đã được push với tag chính xác
- [ ] **SSM commands**: Lịch sử command hiển thị thực thi thành công

#### Lệnh xác minh

```bash
# Kiểm tra tất cả container đang chạy
sudo docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"

# Xác minh API service đang phản hồi
curl -s http://localhost:3000/graphql -H "Content-Type: application/json" \
  -d '{"query":"{ __typename }"}'

# Kiểm tra health search service
curl http://localhost:8000/health

# Xem logs gần đây để tìm lỗi
sudo docker logs --tail 50 api_service
sudo docker logs --tail 50 search_service

# Xác minh biến môi trường đã được load
sudo docker exec api_service env | grep -E "(DATABASE_URL|JWT_SECRET)"

# Kiểm tra CloudWatch metrics trong console
# Xác minh các metric CPU, memory và network
```

### 6.3 Quy trình rollback

Nếu deploy thất bại hoặc phát hiện sự cố, thực thi rollback:

#### Các bước rollback nhanh

```bash
# Trên EC2 instance
cd /app/compose

# Xác định phiên bản làm việc trước đó
docker images 772706200692.dkr.ecr.us-east-1.amazonaws.com/api-service

# Hoặc sử dụng Docker Compose rollback
docker compose -f docker-compose.api-service.yml down
docker compose -f docker-compose.api-service.yml up -d

# Xác minh rollback thành công
sudo docker ps
curl http://localhost:3000/health
```

#### Rollback qua ECR Image trước đó

```bash
# Liệt kê các tag image có sẵn
aws ecr list-images --repository-name api-service --region us-east-1

# Pull image trước đó
docker pull 772706200692.dkr.ecr.us-east-1.amazonaws.com/api-service:<previous-tag>

# Cập nhật compose file với tag trước đó
# Chỉnh sửa docker-compose.api-service.yml để dùng tag cụ thể

# Triển khai lại
docker compose -f docker-compose.api-service.yml up -d
```

#### ECR Lifecycle Policy cho Rollback

Cấu hình lifecycle policy để giữ lại các image trước đó:

```json
{
  "rules": [
    {
      "rulePriority": 1,
      "description": "Keep last 10 images for rollback",
      "selection": {
        "tagStatus": "any",
        "countType": "imageCountMoreThan",
        "countNumber": 10
      },
      "action": {
        "type": "expire"
      }
    }
  ]
}
```

### 6.4 Chiến lược triển khai

#### Blue/Green Deployment (Cải tiến trong tương lai)

Cho triển khai zero-downtime:
1. Triển khai phiên bản mới song song với phiên bản hiện tại
2. Kiểm thử phiên bản mới với traffic production
3. Chuyển traffic sang phiên bản mới
4. Giữ phiên bản cũ để rollback nhanh

#### Canary Deployment (Cải tiến trong tương lai)

Chiến lược rollout từng bước:
1. Triển khai cho 10% traffic ban đầu
2. Giám sát metrics và lỗi
3. Tăng dần traffic
4. Triển khai đầy đủ hoặc rollback tự động

#### Blue/Green với Application Load Balancer

```yaml
# Cải tiến trong tương lai - cấu hình ALB
services:
  api_service_green:
    image: ${ECR_URL}/api-service:green
    # ...
  api_service_blue:
    image: ${ECR_URL}/api-service:blue
    # ...
```

### 6.5 Best Practice khi triển khai

1. **Giám sát trong quá trình deploy**: Theo dõi logs và metrics xuyên suốt
2. **Health checks**: Cấu hình và dựa vào health check của container
3. **Graceful shutdown**: Cho phép container hoàn thành request trước khi dừng
4. **Thiết lập thông báo**: Cấu hình alerts cho các lần deploy thất bại
5. **Tài liệu hóa**: Ghi lại mọi sự cố và cách khắc phục đã gặp

### 6.6 Checklist sau khi deploy

Sau mỗi lần deploy:

```bash
# 1. Xác minh tất cả service healthy
sudo docker-compose -f docker-compose.api-service.yml ps
sudo docker-compose -f docker-compose.search-service.yml ps

# 2. Kiểm tra lỗi trong logs
sudo docker logs api_service 2>&1 | grep -i error | tail -20
sudo docker logs search_service 2>&1 | grep -i error | tail -20

# 3. Kiểm thử các endpoint quan trọng
curl -f http://localhost:3000/health || exit 1
curl -f http://localhost:8000/health || exit 1

# 4. Xác minh kết nối database
sudo docker exec api_service npx prisma db ping

# 5. Kiểm tra CloudWatch xem có alarm nào không
# Xem xét: các metric CPU, Memory, Disk, Network
```

### Các sự cố triển khai thường gặp

| Sự cố | Nguyên nhân | Giải pháp |
|-------|-------|----------|
| Container không khởi động | Thiếu secret | Xác minh quyền truy cập Secrets Manager |
| Kết nối database thất bại | Sự cố mạng | Kiểm tra cấu hình Docker network |
| Port đã được sử dụng | Container cũ vẫn chạy | Dọn dẹp bằng `docker down` |
| Image pull thất bại | ECR authentication hết hạn | Xác thực lại với ECR |

### Tự động hóa triển khai

Thiết lập thông báo triển khai tự động:

```bash
# Thêm vào deploy.sh để gửi thông báo
if [ $? -eq 0 ]; then
    # Gửi thông báo thành công (Slack, SNS, v.v.)
    aws sns publish --topic-arn arn:aws:sns:us-east-1:772706200692:deployment-notifications \
        --message "Deployment successful: ${SERVICE}-service"
else
    # Gửi thông báo thất bại
    aws sns publish --topic-arn arn:aws:sns:us-east-1:772706200692:deployment-notifications \
        --message "Deployment FAILED: ${SERVICE}-service"
fi
```

### Bước tiếp theo

Sau khi cấu hình triển khai production, tiếp tục với Monitoring nơi bạn sẽ thiết lập CloudWatch logs, alarms và giám sát chi phí.