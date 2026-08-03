+++
title = "Giám sát"
date = 2026-08-03
weight = 8
chapter = false
+++

## Giám sát và Khả năng quan sát

### CloudWatch Logs

**CodeBuild logs**: `/aws/codebuild/build-{api,search}-service`
**SSM command logs**: `/aws/ssm/AWS-RunShellScript`
**EC2 system logs**: `/var/log/syslog`, `/var/log/auth.log`

### CloudWatch Alarms

| Metric | Điều kiện | Hành động |
|--------|-----------|-----------|
| CPU > 80% | 5 phút | SNS notification |
| Disk > 90% | - | SNS notification |
| Image scan findings | CRITICAL | SNS notification |
| Build failures | > 3 liên tiếp | SNS notification |
| Network out | > 1GB/giờ | Điều tra |

### Monitoring Chi phí

**Phân tích chi phí hàng tháng:**
- EC2 t2.medium (730 giờ): ~$17/tháng
- ECR storage (2 repos, ~5 GB): ~$0.50/tháng
- S3 storage + requests: ~$0.50/tháng
- Secrets Manager (2 secrets): ~$1/tháng
- CodeBuild: ~$0 (Free Tier)
- Data transfer: ~$3/tháng
- CloudWatch: ~$0.50/tháng
- **Tổng**: ~$22/tháng

**Tối ưu chi phí:**
- Sử dụng AWS Cost Explorer
- Thiết lập billing alarms tại ngưỡng $25/tháng
- Sử dụng AWS Budgets

### Các lệnh phân tích log

```bash
# Tìm lỗi trong CodeBuild logs
aws logs filter-log-events \
  --log-group-name /aws/codebuild/build-api-service \
  --filter-pattern "ERROR"

# Kiểm tra logs SSM
aws logs filter-log-events \
  --log-group-name /aws/ssm/AWS-RunShellScript \
  --filter-pattern "Error"

# Thống kê Docker containers
sudo docker stats --no-stream

# Kiểm tra network
netstat -an | grep ESTABLISHED
```

Với giám sát đã thiết lập, hãy đi đến Bảo mật.