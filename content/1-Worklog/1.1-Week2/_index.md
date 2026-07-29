+++
title = "Week 2 Worklog"
date = 2026-05-16
weight = 2
chapter = false
pre = "<b>1.1.</b>"
+++

## Week 2: Getting Acquainted with FCAJ & Setting Up the AWS Environment

### Completed Tasks

- Attended the orientation session with First Cloud AI Journey (FCAJ) members; read and acknowledged the internship rules and regulations.
- Studied an overview of AWS and its core service groups (Compute, Storage, Networking, Database, Security).
- Created an AWS Free Tier account, installed and configured AWS CLI v2 on the local machine (configured SSO/IAM user with read-only permissions to avoid key exposure).
- Practiced basic AWS CLI commands: `aws sts get-caller-identity`, `aws s3 ls`, `aws ec2 describe-instances`.

### Key Tasks Completed

| Day | Task | Start Date | Completion Date |
|---|---|---|---|
| 1 | Met FCAJ members; read internship regulations (Tuesday) | 12/05/2026 | 12/05/2026 |
| 2 | Studied AWS overview: Compute / Storage / Networking / Database (Wednesday) | 13/05/2026 | 13/05/2026 |
| 3 | Created AWS Free Tier account; installed & configured AWS CLI v2 (Thursday) | 14/05/2026 | 14/05/2026 |
| 4 | Practiced basic AWS CLI commands (sts, s3, ec2 describe) (Friday) | 15/05/2026 | 15/05/2026 |
| 5 | Completed AWS Cloud Practitioner quiz modules 1-2 (Saturday) | 16/05/2026 | 16/05/2026 |

### Outcomes

- AWS Free Tier account is operational and successfully verified with MFA.
- AWS CLI v2 installed successfully on the local machine; can query account information via `aws sts get-caller-identity`.
- Clear understanding of where the 4 core AWS service groups fit in the overall Cloud picture.

### Notes

*This first week was primarily about setting up foundations; no project code was touched. This was an onboarding week with a special Tue-Sat schedule (12-16/5/2026) due to starting mid-week; from Week 3 onward, the standard Mon-Fri schedule applies. Need to note Free Tier limits (EC2 750h/month, S3 5GB) to avoid unexpected charges when running CI/CD in subsequent weeks.*
