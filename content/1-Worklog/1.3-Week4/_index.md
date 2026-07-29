+++
title = "Week 4 Worklog"
date = 2026-06-05
weight = 4
chapter = false
pre = "<b>1.3.</b>"
+++

## Week 4: S3, Basic CloudWatch & AWS Hands-on Labs

### Completed Tasks

- Learned S3: buckets, objects, versions, lifecycle policies, IAM policies for S3; practiced `aws s3 mb`, `aws s3 cp`, `aws s3 sync`.
- Learned CloudWatch: metrics, logs, alarms; practiced creating an alarm for CPU > 80% on the EC2 instance created in week 3.
- Completed AWS Cloud Practitioner Essentials: 10 modules + final assessment (expected score 850+/1000).
- Hands-on lab: launched EC2 with user data bootstrap script, installed nginx, opened port 80.
- Read basic Docker documentation (containers, images, Dockerfile, docker compose) in preparation for microservice deployment in upcoming weeks.

### Key Tasks Completed

| Day | Task | Start Date | Completion Date |
|---|---|---|---|
| 12 | Learned S3: buckets, lifecycle, IAM policies | 01/06/2026 | 01/06/2026 |
| 13 | Practiced basic `aws s3` CLI commands | 02/06/2026 | 02/06/2026 |
| 14 | Learned CloudWatch: metrics, logs, alarms | 03/06/2026 | 03/06/2026 |
| 15 | Completed AWS Cloud Practitioner Essentials final assessment | 04/06/2026 | 04/06/2026 |
| 16 | Lab: launch EC2 + nginx + user data bootstrap; read Docker docs | 05/06/2026 | 05/06/2026 |

### Outcomes

- First S3 bucket `fcaj-intern-storage` created and test files uploaded successfully.
- CloudWatch alarm for EC2 CPU triggered email notification via SNS.
- Achieved internal AWS Cloud Practitioner certification (expected).
- Understood Docker architecture well enough to read and modify a Dockerfile for next week.

### Notes

*This week transitioned from "getting acquainted" to "mastering the fundamentals". The content covered is sufficient to start touching the real project in week 5. Note: S3 Free Tier has limits of 5GB storage + 15GB transfer out/month — avoid uploading large files during labs.*
