+++
title = "Worklog Tuần 3"
date = 2026-05-29
weight = 3
chapter = false
pre = "<b>1.2.</b>"
+++

## Tuần 3: EC2, IAM và triển khai instance đầu tiên

### Công việc đã làm

- Học kiến trúc EC2: AMI, instance type, EBS, key pair, security group, elastic IP.
- Học IAM: user, group, role, policy; phân biệt IAM User (long-term credential) vs IAM Role (temporary credential cho EC2).
- Tạo IAM Role `EC2-Backend-Role` với policy `AmazonS3ReadOnlyAccess` + `AmazonEC2ContainerRegistryPowerUser` + `AmazonSSMFullAccess` (sẽ dùng cho deploy script sau này).
- Launch EC2 instance `t3.small` Ubuntu 22.04, cấu hình Security Group mở port 22 (SSH), 80 (HTTP), 443 (HTTPS).
- Đọc docs VPC cơ bản (subnet, route table, internet gateway) để chuẩn bị cho việc triển khai nhiều service trong các tuần sau.

### Công việc chính

| Ngày | Công việc | Bắt đầu | Hoàn thành |
|---|---|---|---|
| 7 | Học EC2: AMI, instance type, EBS, key pair, security group | 25/05/2026 | 25/05/2026 |
| 8 | Học IAM: User/Group/Role/Policy; phân biệt credential model | 26/05/2026 | 26/05/2026 |
| 9 | Tạo IAM Role `EC2-Backend-Role` với policy cho S3/ECR/SSM | 27/05/2026 | 27/05/2026 |
| 10 | Launch EC2 `t3.small` Ubuntu 22.04, cấu hình Security Group | 28/05/2026 | 28/05/2026 |
| 11 | Đọc docs VPC: subnet, route table, internet gateway | 29/05/2026 | 29/05/2026 |

### Kết quả đạt được

- EC2 instance đầu tiên chạy ổn định, SSH thành công qua key pair.
- IAM Role đã attach thành công vào instance, có thể gọi AWS API từ EC2 mà không cần lưu access key trên disk (đây là best practice).
- Hiểu rõ tại sao dùng Role thay vì User credentials cho service chạy lâu dài trên EC2.

### Ghi chú 

*Bài học quan trọng: tuyệt đối không lưu AWS access key vào EC2 user data hoặc repo. Luôn dùng IAM Role + metadata service (IMDSv2). EC2 instance này sẽ trở thành backend deploy host cho các tuần tiếp theo.*
