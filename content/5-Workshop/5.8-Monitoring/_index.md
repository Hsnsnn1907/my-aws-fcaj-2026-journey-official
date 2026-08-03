+++
title = "5.8. Monitoring"
date = 2026-08-03
weight = 8
chapter = false
+++

## Monitoring and Observability

This section covers the monitoring setup for the CI/CD pipeline including CloudWatch logs, alarms, and cost monitoring.

### 7.1 CloudWatch Logs

Set up comprehensive logging across all pipeline components:

#### CodeBuild Logs
- **Log group**: `/aws/codebuild/build-{api,search}-service`
- **Stream**: Build ID and phase
- **Retention**: 30 days
- **Contents**: Build output, commands, errors

#### SSM Command Logs
- **Log group**: `/aws/ssm/AWS-RunShellScript`
- **Stream**: Command ID and instance ID
- **Retention**: 30 days
- **Contents**: Command output, errors, duration

#### EC2 Application Logs
- **Docker logging**: Configure Docker logging driver to CloudWatch
- **EC2 system logs**: Monitor `/var/log/syslog`, `/var/log/auth.log`
- **Application logs**: Container stdout/stderr

#### CloudWatch Logs Setup

```bash
# Create log groups
aws logs create-log-group --log-group-name /aws/codebuild/build-api-service
aws logs create-log-group --log-group-name /aws/codebuild/build-search-service
aws logs create-log-group --log-group-name /aws/ssm/AWS-RunShellScript

# Set retention policy
aws logs put-retention-policy --log-group-name /aws/codebuild/build-api-service --retention-in-days 30
aws logs put-retention-policy --log-group-name /aws/codebuild/build-search-service --retention-in-days 30
aws logs put-retention-policy --log-group-name /aws/ssm/AWS-RunShellScript --retention-in-days 30
```

### 7.2 CloudWatch Alarms

Set up alarms for proactive issue detection:

#### EC2 Alarms

| Metric | Condition | Action |
|--------|-----------|--------|
| CPUUtilization | > 80% for 5 minutes | Notify via SNS |
| DiskSpaceUtilization | > 90% | Notify via SNS |
| StatusCheckFailed | Any | Notify via SNS |

#### ECR Alarms

| Metric | Condition | Action |
|--------|-----------|--------|
| ImageScanFindings | CRITICAL severity > 0 | Notify via SNS |

#### CodeBuild Alarms

| Metric | Condition | Action |
|--------|-----------|--------|
| BuildsFailed | > 3 consecutive | Notify via SNS |

#### S3 and Network Alarms

| Metric | Condition | Action |
|--------|-----------|--------|
| NetworkOut | > 1GB/hour | Investigate (potential data leak) |
| BytesDownloaded | Unusual spike | Investigate |

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

### 7.3 Cost Monitoring

Track and optimize costs across all AWS services:

#### Monthly Cost Breakdown

| Service | Estimated Cost | Notes |
|---------|----------------|-------|
| EC2 t2.medium | ~$17/month | 730 hours/month |
| ECR storage | ~$0.50/month | 2 repos, ~5 GB |
| S3 storage + requests | ~$0.50/month | Compose files, minimal |
| Secrets Manager | ~$1/month | 2 secrets |
| CodeBuild | ~$0/month | Free Tier (20 builds/month) |
| Data transfer | ~$3/month | ECR pulls, API traffic |
| CloudWatch | ~$0.50/month | Logs and metrics |

**Total: ~$22-23/month**

#### Cost Monitoring Setup

1. **AWS Cost Explorer**: Enable in Billing & Cost Management
2. **Billing alarms**: Create CloudWatch alarms for budget thresholds
3. **AWS Budgets**: Set up monthly budget with notifications

```bash
# Set billing alarm at $25 threshold
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

Create a CloudWatch dashboard for centralized monitoring:

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

Common log analysis queries:

```bash
# Search for errors in CodeBuild logs
aws logs filter-log-events \
  --log-group-name /aws/codebuild/build-api-service \
  --filter-pattern "ERROR" \
  --start-time $(date -d '24 hours ago' +%s)000

# Search SSM command output
aws logs filter-log-events \
  --log-group-name /aws/ssm/AWS-RunShellScript \
  --filter-pattern "Error" \
  --start-time $(date -d '7 days ago' +%s)000

# Check EC2 system logs
aws ssm get-command-invocation \
  --command-id <command-id> \
  --instance-id i-037a4cd636a68eb7e \
  --output-format Text
```

### 7.6 Performance Monitoring

Track application performance metrics:

```bash
# Monitor Docker container stats
sudo docker stats --no-stream

# Check container resource usage
sudo docker ps -q | xargs sudo docker stats

# Monitor disk I/O
iostat -x 1 5

# Network connections
netstat -an | grep ESTABLISHED
```

### 7.7 Alerting Configuration

Set up SNS topic for notifications:

```bash
# Create SNS topic
aws sns create-topic --name deployment-notifications

# Subscribe to notifications
aws sns subscribe \
  --topic-arn arn:aws:sns:us-east-1:772706200692:deployment-notifications \
  --protocol email \
  --notification-endpoint your-email@example.com
```

### Monitoring Best Practices

1. **Log retention**: Set appropriate retention periods (30 days recommended)
2. **Log encryption**: Enable encryption for sensitive log data
3. **Alarm testing**: Test alarms periodically to ensure they fire
4. **Dashboard review**: Review monitoring dashboard daily
5. **Cost alerts**: Set alerts at 50%, 80%, 100% of budget

### Next Steps

With monitoring configured, proceed to Security for best practices on securing the CI/CD pipeline and infrastructure.