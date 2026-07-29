+++
title = "Week 8 Worklog"
date = 2026-07-03
weight = 8
chapter = false
pre = "<b>1.7.</b>"
+++

## Week 8: AWS Console Setup for CI/CD Pipeline

### Completed Tasks

- Created ECR repositories for 2 services on the AWS Console:
  - `videoplatform/api-service` (private).
  - `videoplatform/search-service` (private).
  - Configured scan on push and tag immutability.
- Created S3 bucket `videoplatform-deploy-artifacts-dsk` used to ship compose files from CodeBuild to EC2 via `aws s3 sync`.
- Created a secret in AWS Secrets Manager `videoplatform/backend-secrets` containing:
  - `DATABASE_URL` (Postgres connection string).
  - `RABBITMQ_URL`.
  - `REDIS_URL`.
  - `QDRANT_URL`.
  - `FIREBASE_PRIVATE_KEY` (with `\n` escapes).
  - `FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, etc.
- Created IAM role `EC2-Backend-Role` attached to EC2 (launched in Week 3) with policies:
  - `AmazonS3ReadOnlyAccess` (to pull compose files).
  - `AmazonEC2ContainerRegistryPowerUser` (to pull images).
  - `AmazonSSMFullAccess` (to receive commands from CodeBuild).
  - `SecretsManagerReadWrite` (to read secrets via SSM parameter).
- Documented IAM role ARN and secret ARN for use in the buildspec in Week 9.

### Key Tasks

| Day | Task | Start | Complete |
|---|---|---|---|
| 32 | Create 2 ECR repos (api-service, search-service) with scan on push | 29/06/2026 | 29/06/2026 |
| 33 | Create S3 bucket `videoplatform-deploy-artifacts-dsk` | 30/06/2026 | 30/06/2026 |
| 34 | Create secret `videoplatform/backend-secrets` in Secrets Manager | 01/07/2026 | 01/07/2026 |
| 35 | Attach IAM role `EC2-Backend-Role` to EC2 instance | 02/07/2026 | 02/07/2026 |
| 36 | Document ARNs + secret keys in preparation for buildspec next week | 03/07/2026 | 03/07/2026 |

### Outcomes

- All AWS-side infrastructure for CI/CD is ready: ECR repos for images, S3 bucket for compose files, Secrets Manager for credentials.
- IAM role attached to EC2; EC2 can pull images from ECR and read secrets via instance metadata.
- No more manual SSH into EC2 for setup -- everything is managed via AWS Console + CLI.

### Notes

*Lesson learned about `FIREBASE_PRIVATE_KEY`: you must use `\n` escapes (literal backslash-n) instead of actual newlines when storing in a JSON secret. The Admin SDK parses the string `"-----BEGIN PRIVATE KEY-----\nMIIE..."` into a valid PEM; if actual newlines are used, the JSON parser fails or the env file gets corrupted. This is a classic bug that most tutorials skip. Noted for the `script/deploy.sh` in the following week, which will use a Python heredoc instead of `jq` to avoid newline corruption.*
