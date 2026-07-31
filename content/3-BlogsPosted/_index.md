+++
title = "Blogs Posted"
date = 2026-07-01
weight = 3
chapter = false
+++| No. | Topic | Summary | Link |
|---|---|---|---|
| 1 | Building a CI/CD pipeline with AWS CodeBuild + ECR + S3 + SSM for Node.js & Python microservices | End-to-end guide on setting up a pipeline from GitHub push to EC2 deployment in 5 minutes, covering 2 parallel services; includes 4 common build errors and fixes. | [Facebook Post](https://www.facebook.com/groups/awsstudygroupfcj/posts/2224001141698179/?notif_id=1785108354168202&notif_t=tagged_with_story&ref=notif) |
| 2 | Optimizing cost & Docker image speed with multi-stage build + Amazon ECR scan on push | How to cut 70% of Node.js/Python image size using multi-stage builds, strict `.dockerignore`, and ECR scan on push to detect vulnerabilities early; includes real cost comparison between 1.2GB and 280MB images. | Placeholder |
| 3 | Zero-secret deployment with AWS Secrets Manager + IAM Role + IMDSv2: lessons from the `FIREBASE_PRIVATE_KEY` newline bug | Completely removing `.env` files from the deployment workflow, using Secrets Manager + EC2 IAM Roles + Python heredocs instead of `jq` to avoid corrupting multiline strings; includes real template code. | [Facebook Post](https://www.facebook.com/groups/awsstudygroupfcj/posts/2226205278144432/?notif_id=1785326007150392&notif_t=tagged_with_story&ref=notif) |

## Blog 1: Building a CI/CD pipeline with AWS CodeBuild + ECR + S3 + SSM

**Introduction**
This blog shares practical experience building the CI/CD pipeline for the VideoPlatform project (a Node.js 18 + NestJS 11 `api_service` and a Python 3.10 + FastAPI `search_service`). The entire flow, from pushing code to GitHub to services running on EC2, takes about 5 minutes, entirely automated without manual SSH. It focuses on CodeBuild, ECR, S3, and SSM.

**Key points:**
- **4-step Architecture:** GitHub webhook $\rightarrow$ CodeBuild $\rightarrow$ ECR push $\rightarrow$ S3 sync $\rightarrow$ SSM `AWS-RunShellScript` trigger.
- **CodeBuild Ephemeral Env:** Billed by the minute (`$0.005/minute`), auto-scaled and auto-cleaned, heavily cost-effective compared to traditional self-hosted Jenkins.
- **Parallel CodeBuild Projects:** Using path-based webhook filtering (`^api_service/.*$`) to separately build microservices rather than rebuilding everything whenever a root level file changes.
- **Troubleshooting:**
  - **Bug 1:** `.dockerignore` conflicting with the build context.
  - **Bug 2:** Forgetting `COPY requirements.txt` before running pip in multi-stage builds.
  - **Bug 3:** Having `prisma/schema.prisma` commented out completely resulting in zero datasources.
  - **Bug 4:** Remaining inside the `search_service` directory during `post_build` ruining the S3 sync. Fixed by returning to `$CODEBUILD_SRC_DIR`.
  - **Docker Entrypoints vs Dockerfile RUN:** A TCP wait must happen in `entrypoint.sh` using `nc -z db 5432`, not blindly within `Dockerfile RUN`.
- Docker Layer Cache usage allows fixing quick steps iteratively in 1-2 minutes instead of 10-15 minute full rebuilds. Use `--progress=plain` and `docker run --rm -it $IMAGE_ID sh` to visually debug containers.

**Lessons learned** 
Never substitute live test pipelines with mere code review. 4 distinct bugs only surfaced during actual CodeBuild triggers at week 10. Run early and run often using local Layer Caches to cut 15 minute iterations into 1. Replicating the Jenkins deployment setup effectively in AWS-native tooling offers immense flexibility.

![Blog 1 Illustration](../images/blog_1.png)

## Blog 2: Optimizing cost & Docker image speed with multi-stage build

**Introduction**
My initial Node.js image was 1.2 GB full of `devDependencies`, Git history, and bloated compilers. This post covers how 3 techniques drastically cut the image down to 280 MB and allowed for seamless ECR scanning that caught 3 critical CVEs on the very first push.

**Key points:**
- **Node.js Multi-stage:** The `builder` stage utilizes `node:18-alpine` running `npm ci` and `npm run build`. The `runtime` stage simply copies `dist/` and isolates `npm ci --omit=dev`.
- **Python Multi-stage:** Uses a "wheel-based" pattern with `pip wheel` across stages effectively turning runtime installs into binary unzips and reducing compiler dependencies.
- **Strict `.dockerignore`:** Blocking 20+ known vulnerabilities, IDE files (`.vscode`, `.idea`), `.git`, and environment variables correctly. 
- **Amazon ECR Scan On Push:** Enhanced scanning detected vulnerabilities like `glibc CVE-2023-0682` allowing proactive patching natively through base image bumping instead of manual patching.
- **Cost Saving:** Reduced overall storage costs by 76% resulting in $0.063/month vs $0.27/month. Pull transfer time fell from 45s to 12s.
- **Debugger Tooling:** Recommended the `dive` open-source tool for exploring physical docker layer modifications.

## Blog 3: Zero-secret deployment with AWS Secrets Manager + IAM Role

**Introduction**
This post shares the transition from sending `.env` files via `scp` to achieving a flawless "zero-secret" state using AWS Secrets Manager after debugging a massive newline corruption issue during Firebase JSON parsing with `jq`.

**Key points:**
- **The Golden Rule:** Secrets NEVER belong in `Dockerfile`, Git, or standard `.env` EC2 files. They belong entirely in AWS Secrets Manager.
- **IAM Roles over Access Keys:** Ditch static long-term IAM Users. Employ IAM Roles generating temporary STS tokens.
- **`FIREBASE_PRIVATE_KEY` Bug:** JSON parsing with `jq` ruins exact newline escapement (`\n`). `jq` thinks it's literal text instead of a PEM-formatted sequence.  
- **Python Heredoc Solution:** Exchanged `jq` parsed scripting with `python3 - <<'PYEOF'` to guarantee standard exact payload transcription inside the EC2 setup script.
- **IMDSv2 Necessity:** A historical lookup at the **Capital One 2019 SSRF Attack** showing how crucial it is to mandate token-based HTTP PUT metadata calls (`IMDSv2`) rather than raw `IMDSv1` requests.
- **Rotation Configuration:** Using Lambdas to rotate sensitive databases automatically every 90 days.
