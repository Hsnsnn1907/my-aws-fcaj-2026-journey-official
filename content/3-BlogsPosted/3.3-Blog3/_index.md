+++
title = "3.3. Blog 3: Zero-secret deployment with AWS Secrets Manager + IAM Role"
date = 2026-07-29
weight = 3
chapter = false
+++

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