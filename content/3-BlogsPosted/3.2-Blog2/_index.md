+++
title = "3.2. Blog 2: Optimizing cost & Docker image speed with multi-stage build"
date = 2026-07-15
weight = 2
chapter = false
+++

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