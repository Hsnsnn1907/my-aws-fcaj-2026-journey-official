+++
title = "3.2. Blog 2: Tối ưu chi phí & tăng tốc Docker image với multi-stage build"
date = 2026-07-15
weight = 2
chapter = false
+++

## Blog 2: Tối ưu chi phí & tăng tốc Docker image với multi-stage build

**Giới thiệu**
Từ 1.2GB image cho Node.js rút gọn xuống 280MB nhờ multi-stage build và `.dockerignore`, đồng bộ với công tác security scan của ECR quét dính 3 lỗ hổng critical ngay khi push.

**Những điểm chính cần biết:**
- **Node.js Multi-stage build:** Chuyển dist qua stage runtime và cài đặt `npm ci --omit=dev`.
- **Python Multi-stage build:** Sử dụng wheel-based pattern gói toàn cỗ `pip wheel` rút gón môi trường ở Runtime cực tinh gọn.
- **`.dockerignore`:** Cắt trên 20 pattern cấm (Secrets, Test artifacts, vcs..). Ngăn chặn shell globbing nhầm.
- **ECR scan on push:** Tự động kích hoạt tính năng vá lổ hổng Inspect (Basic và Enhanced Scan).
- **Cost Saving:** Tiết kiệm 76% phí duy trì Image. Thời gian deploy giảm mạnh.
- Khuyến nghị sử dụng cài đặt Tool Dive (`wagoodman/dive`) kiểm soát wasted space.