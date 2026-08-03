+++
title = "3. Blogs Posted"
date = 2026-07-01
weight = 3
chapter = false
+++

| No. | Topic | Summary | Link |
|---|---|---|---|
| 1 | Building a CI/CD pipeline with AWS CodeBuild + ECR + S3 + SSM for Node.js & Python microservices | End-to-end guide on setting up a pipeline from GitHub push to EC2 deployment in 5 minutes, covering 2 parallel services; includes 4 common build errors and fixes. | [Facebook Post](https://www.facebook.com/groups/awsstudygroupfcj/posts/2224001141698179/?notif_id=1785108354168202&notif_t=tagged_with_story&ref=notif) |
| 2 | Optimizing cost & Docker image speed with multi-stage build + Amazon ECR scan on push | How to cut 70% of Node.js/Python image size using multi-stage builds, strict `.dockerignore`, and ECR scan on push to detect vulnerabilities early; includes real cost comparison between 1.2GB and 280MB images. | Placeholder |
| 3 | Zero-secret deployment with AWS Secrets Manager + IAM Role + IMDSv2: lessons from the `FIREBASE_PRIVATE_KEY` newline bug | Completely removing `.env` files from the deployment workflow, using Secrets Manager + EC2 IAM Roles + Python heredocs instead of `jq` to avoid corrupting multiline strings; includes real template code. | [Facebook Post](https://www.facebook.com/groups/awsstudygroupfcj/posts/2226205278144432/?notif_id=1785326007150392&notif_t=tagged_with_story&ref=notif) |