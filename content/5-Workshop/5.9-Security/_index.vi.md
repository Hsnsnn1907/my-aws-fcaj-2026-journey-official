+++
title = "Bảo mật"
date = 2026-08-03
weight = 9
chapter = false
+++

## Best Practices Bảo mật

### Quản lý Secrets

- ✅ Không có secrets trong code hoặc env vars
- ✅ Secrets lưu trong AWS Secrets Manager
- ✅ Truy cập dựa trên IAM role (không có long-term credentials)
- ✅ Secrets fetch tại runtime
- ✅ Mã hóa dữ liệu (AWS-managed keys)
- ✅ Rotation secrets (90 ngày)

### Bảo mật mạng

- ✅ EC2 trong default VPC
- ✅ Security groups: tối thiểu port exposure
- ✅ IMDSv2 bắt buộc
- ✅ ECR private repositories
- ✅ S3 bucket: block public access
- ✅ SSH restricted đến IPs cụ thể

### Bảo mật containers

- ✅ Multi-stage builds (giảm surface attack)
- ✅ Non-root user (tốt nhất có thể)
- ✅ ECR image scanning enabled
- ✅ Minimal base images (alpine, slim)
- ✅ Không có credentials trong images
- ✅ Quét vulnerability định kỳ

### IAM Least Privilege

- ✅ Roles riêng cho CodeBuild và EC2
- ✅ Policies resource-specific (không dùng `*` wildcard)
- ✅ Không có AWS access keys (chỉ IAM roles)
- ✅ Review policies hàng tháng
- ✅ MFA enabled cho tất cả IAM users

### Commands kiểm tra bảo mật

```bash
# Kiểm tra unused access keys
aws iam list-access-keys --user-name <username>

# Review security groups cho rules mở rộng
aws ec2 describe-security-groups --filters "Name=ip-permission.cidr,Values=0.0.0.0/0"

# Kiểm tra IAM policies
aws iam list-policies --scope AWS
```

Với best practices bảo mật đã áp dụng, hãy đi đến Khắc phục sự cố.