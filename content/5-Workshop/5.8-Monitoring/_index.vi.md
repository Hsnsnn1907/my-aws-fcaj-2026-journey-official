+++
title = "5.8. Giám sát"
date = 2026-08-03
weight = 8
chapter = false
+++

## Giám sát và Khả năng quan sát (Monitoring and Observability)

Phần này trình bày thiết lập giám sát cho pipeline CI/CD bao gồm CloudWatch logs, alarms và theo dõi chi phí.

### 7.1 CloudWatch Logs

Thiết lập logging toàn diện cho tất cả các thành phần trong pipeline:

#### CodeBuild Logs
- **Log group**: `/aws/codebuild/build-{api,search}-service`
- **Stream**: Build ID và phase
- **Retention**: 30 ngày
- **Nội dung**: Build output, commands, errors

#### SSM Command Logs
- **Log group**: `/aws/ssm/AWS-RunShellScript`
- **Stream**: Command ID và instance ID
- **Retention**: 30 ngày
- **Nội dung**: Command output, errors, duration

#### EC2 Application Logs
- **Docker logging**: Cấu hình Docker logging driver cho CloudWatch
- **EC2 system logs**: Giám sát `/var/log/syslog`, `/var/log/auth.log`
- **Application logs**: Container stdout/stderr

#### CloudWatch Logs Setup

```bash
# Tạo log groups
aws logs create-log-group --log-group-name /aws/codebuild/build-api-service
aws logs create-log-group --log-group-name /aws/codebuild/build-search-service
aws logs create-log-group --log-group-name /aws/ssm/AWS-RunShellScript

# Thiết lập retention policy
aws logs put-retention-policy --log-group-name /aws/codebuild/build-api-service --retention-in-days 30
aws logs put-retention-policy --log-group-name /aws/codebuild/build-search-service --retention-in-days 30
aws logs put-retention-policy --log-group-name /aws/ssm/AWS-RunShellScript --retention-in-days 30
```

### 7.2 CloudWatch Alarms

Thiết lập alarms để phát hiện sự cố chủ động:

#### EC2 Alarms

| Metric | Điều kiện | Hành động |
|--------|-----------|--------|
| CPUUtilization | > 80% trong 5 phút | Thông báo qua SNS |
| DiskSpaceUtilization | > 90% | Thông báo qua SNS |
| StatusCheckFailed | Bất kỳ | Thông báo qua SNS |

#### ECR Alarms

| Metric | Điều kiện | Hành động |
|--------|-----------|--------|
| ImageScanFindings | CRITICAL severity > 0 | Thông báo qua SNS |

#### CodeBuild Alarms

| Metric | Điều kiện | Hành động |
|--------|-----------|--------|
| BuildsFailed | > 3 liên tiếp | Thông báo qua SNS |

#### S3 và Network Alarms

| Metric | Điều kiện | Hành động |
|--------|-----------|--------|
| NetworkOut | > 1GB/giờ | Điều tra (khả năng rò rỉ dữ liệu) |
| BytesDownloaded | Spike bất thường | Điều tra |

#### Alarm Creation Commands

```bash
# CPU High Alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-High-CPU" \
  --alarm-description "CPU usage above 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/EC2 \
  --statistic Average \
  --period 300 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-037a4cd636a68eb7e \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:772706200692:deployment-notifications

# Disk Space Alarm
aws cloudwatch put-metric-alarm \
  --alarm-name "EC2-Disk-Space" \
  --alarm-description "Disk usage above 90%" \
  --metric-name DiskSpaceUtilization \
  --namespace System/Linux \
  --statistic Average \
  --period 300 \
  --threshold 90 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=InstanceId,Value=i-037a4cd636a68eb7e \
  --evaluation-periods 2 \
  --alarm-actions arn:aws:sns:us-east-1:772706200692:deployment-notifications
```

### 7.3 Giám sát chi phí (Cost Monitoring)

Theo dõi và tối ưu chi phí trên tất cả các dịch vụ AWS:

#### Phân tích chi phí hàng tháng

| Dịch vụ | Chi phí ước tính | Ghi chú |
|---------|----------------|-------|
| EC2 t2.medium | ~$17/tháng | 730 giờ/tháng |
| ECR storage | ~$0.50/tháng | 2 repos, ~5 GB |
| S3 storage + requests | ~$0.50/tháng | Compose files, tối thiểu |
| Secrets Manager | ~$1/tháng | 2 secrets |
| CodeBuild | ~$0/tháng | Free Tier (20 builds/tháng) |
| Data transfer | ~$3/tháng | ECR pulls, API traffic |
| CloudWatch | ~$0.50/tháng | Logs và metrics |

**Tổng: ~$22-23/tháng**

#### Thiết lập giám sát chi phí

1. **AWS Cost Explorer**: Bật trong Billing & Cost Management
2. **Billing alarms**: Tạo CloudWatch alarms cho ngưỡng ngân sách
3. **AWS Budgets**: Thiết lập ngân sách hàng tháng với thông báo

```bash
# Đặt billing alarm ở ngưỡng $25
aws cloudwatch put-metric-alarm \
  --alarm-name "Billing-Alarm" \
  --alarm-description "Estimated charges above $25" \
  --metric-name EstimatedCharges \
  --namespace AWS/Billing \
  --statistic Maximum \
  --period 21600 \
  --threshold 25 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=Currency,Value=USD \
  --evaluation-periods 1 \
  --alarm-actions arn:aws:sns:us-east-1:772706200692:deployment-notifications
```

### 7.4 Monitoring Dashboard

Tạo CloudWatch dashboard để giám sát tập trung:

```json
{
  "widgets": [
    {
      "type": "metric",
      "x": 0, "y": 0,
      "width": 12, "height": 6,
      "properties": {
        "title": "EC2 CPU Utilization",
        "metrics": [["AWS/EC2", "CPUUtilization", "InstanceId", "i-037a4cd636a68eb7e"]],
        "period": 300,
        "stat": "Average",
        "region": "us-east-1"
      }
    },
    {
      "type": "metric",
      "x": 12, "y": 0,
      "width": 12, "height": 6,
      "properties": {
        "title": "EC2 Network Out",
        "metrics": [["AWS/EC2", "NetworkOut", "InstanceId", "i-037a4cd636a68eb7e"]],
        "period": 300,
        "stat": "Sum",
        "region": "us-east-1"
      }
    }
  ]
}
```

### 7.5 Log Analysis Commands

Các truy vấn phân tích log thường gặp:

```bash
# Tìm kiếm errors trong CodeBuild logs
aws logs filter-log-events \
  --log-group-name /aws/codebuild/build-api-service \
  --filter-pattern "ERROR" \
  --start-time $(date -d '24 hours ago' +%s)000

# Tìm kiếm SSM command output
aws logs filter-log-events \
  --log-group-name /aws/ssm/AWS-RunShellScript \
  --filter-pattern "Error" \
  --start-time $(date -d '7 days ago' +%s)000

# Kiểm tra EC2 system logs
aws ssm get-command-invocation \
  --command-id <command-id> \
  --instance-id i-037a4cd636a68eb7e \
  --output-format Text
```

### 7.6 Performance Monitoring

Theo dõi các metric hiệu năng ứng dụng:

```bash
# Giám sát Docker container stats
sudo docker stats --no-stream

# Kiểm tra resource usage của container
sudo docker ps -q | xargs sudo docker stats

# Giám sát disk I/O
iostat -x 1 5

# Network connections
netstat -an | grep ESTABLISHED
```

### 7.7 Alerting Configuration

Thiết lập SNS topic cho thông báo:

```bash
# Tạo SNS topic
aws sns create-topic --name deployment-notifications

# Subscribe để nhận thông báo
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:772706200692:deployment-notifications \
  --protocol email \
  --notification-endpoint your-email@example.com
```

### Best Practices cho Giám sát

1. **Log retention**: Đặt retention period phù hợp (khuyến nghị 30 ngày)
2. **Log encryption**: Bật mã hóa cho dữ liệu log nhạy cảm
3. **Alarm testing**: Kiểm thử alarms định kỳ để đảm bảo chúng kích hoạt
4. **Dashboard review**: Xem xét monitoring dashboard hàng ngày
5. **Cost alerts**: Đặt cảnh báo ở 50%, 80%, 100% ngân sách

### Bước tiếp theo

Sau khi đã cấu hình giám sát, tiếp tục đến phần Security để tìm hiểu các best practice bảo mật CI/CD pipeline và infrastructure.