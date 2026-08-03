+++
title = "3.3. Blog 3: Zero-secret deployment với AWS Secrets Manager + IAM Role"
date = 2026-07-29
weight = 3
chapter = false
+++

## Blog 3: Zero-secret deployment với AWS Secrets Manager + IAM Role

**Giới thiệu**
Đạt trạng thái 'zero secret deployment' triệt để khi bỏ hoàn toàn tệp `.env` hardcode vào máy chủ, thay vì copy file. Mọi biến được inject dưới dạng Role thông qua IMDSv2. Ngăn ngừa toàn bộ rủi ro của `FIREBASE_PRIVATE_KEY`.

**Những điểm chính cần biết:**
- **Zero-secret:** Mọi secret sống trong Secrets Manager.
- **Tạm biệt Access Key:** Dùng IAM Role (gắn vào EC2 + SecretsManagerReadWrite) lấy STS token thay vì key vĩnh viễn.
- **`FIREBASE_PRIVATE_KEY` Bug:** Lỗi Parse Newline (0x0A) gây ra thông qua cơ chế `jq`.
- **Giải pháp Python Heredoc:** Khởi tạo Python Json loads trong Deploy sh. Python đọc \n an toàn hơn các tool bash thuần.
- **Khái niệm IMDSv2:** Tầm quan trọng của IMDSv2 từ vụ hack SSRF rực rỡ của **Capital One (2019)**. Require HttpToken qua PUT request.
- Các quy trình Rotating với Lambda Serverless và audit tracking tự động qua CloudWatch.