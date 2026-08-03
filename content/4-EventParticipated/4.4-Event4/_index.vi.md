+++
title = "4.4. Sự kiện 4"
date = 2026-08-01
weight = 4
chapter = false
+++

Tên sự kiện: Agent Forge - Deepdive Day 1

Thời gian: 09:00 - 12:00, 1 tháng 8, 2026

Địa điểm: Tầng 26, Tòa nhà Bitexco Financial Tower, Thành phố Hồ Chí Minh

Vai trò: Người tham dự

### Tổng quan sự kiện

Agent Forge là một workshop chuyên sâu 3 ngày về xây dựng và triển khai AI agents sẵn sàng cho production bằng Amazon Bedrock AgentCore và Kiro IDE. Ngày 1 (hôm nay) tập trung vào các khái niệm cơ bản của Kiro và phát triển agent thực hành bằng vibe coding - một cách tiếp cận cách mạng mà developers mô tả tính năng bằng ngôn ngữ tự nhiên và AI tạo ra code hoàn chỉnh, đã test, sẵn sàng production.

**Vibe Coding là gì?**

Vibe coding đại diện cho tương lai của phát triển phần mềm:
- Mô tả những gì bạn muốn bằng tiếng Anh đơn giản
- Kiro IDE tạo code hoàn chỉnh, đã test ngay lập tức
- Không cần ghi nhớ cú pháp, không cần boilerplate, không cần debug
- Từ ý tưởng đến phần mềm hoạt động trong vài phút, không phải vài ngày

**Kiro IDE là gì?**

Kiro là môi trường phát triển được hỗ trợ bởi AI được thiết kế đặc biệt cho vibe coding với:
- Chuyển đổi ngôn ngữ tự nhiên sang code production
- Phát triển theo spec cho các tính năng phức tạp (Requirements → Design → Tasks → Code)
- Agent hooks cho tự động hóa workflow
- Tích hợp MCP (Model Context Protocol) cho các dịch vụ bên ngoài
- Triển khai full-stack: Frontend, Backend, Cloud deployment

**Amazon Bedrock AgentCore là gì?**

Một dịch vụ AWS được quản lý để triển khai AI agents lên production với:
- **Memory**: Nhớ preferences của người dùng qua các cuộc hội thoại (summary, preferences, semantic)
- **Gateway**: Kết nối an toàn với APIs và databases bên ngoài qua MCP
- **Runtime**: Hạ tầng tự động scale, được quản lý cho agents
- **Observability**: CloudWatch logs, traces, và GenAI dashboard
- **Policies**: Ủy quyền chi tiết dựa trên Cedar với ràng buộc tham số

### Nội dung Workshop: Lab 1 & Lab 2 (Một phần)

#### Lab 1: Kiro và Vibe Coding (09:00 – 10:00)

**Tổng quan**
Khám phá Kiro, một môi trường phát triển AI-native biến đổi phát triển phần mềm thông qua vibe coding.

**Khả năng cốt lõi**
- **Phát triển bằng Ngôn ngữ Tự nhiên**: "Xây dựng web application với đăng nhập và dashboard" → Kiro tạo UI responsive, serverless backend, database, auth system, triển khai lên production
- **Triển khai Full-Stack**: Frontend (Vite, React), Backend (NestJS, Lambda), Cloud (AWS CDK, ECR, S3)
- **Tự động hóa Thông minh**: Thêm tính năng như payments hoặc notifications → Kiro tích hợp APIs, cập nhật schemas, xử lý security
- **Cộng tác Lặp lại**: Feedback thời gian thực, smart debugging, cải tiến liên tục

**Hai Chế độ Phát triển**
1. **Vibe Coding** (Nhanh): Cho prototyping nhanh và khám phá - mô tả những gì bạn muốn, Kiro tạo code ngay lập tức
2. **Spec-Driven Development** (Production): Cho tính năng phức tạp - Thu thập requirements → Lập kế hoạch thiết kế → Sắp xếp tasks → Triển khai với full traceability

**Những gì đã học**
- Cách sử dụng Kiro cho phát triển AI agent nhanh
- Thiết lập AWS credentials và truy cập Bedrock
- Tích hợp MCP server cho external tools
- Sự khác biệt giữa approaches khám phá và production

#### Lab 2: Xây dựng & Triển khai AI Agents với AgentCore CLI (10:00 – 11:00+)

**Điều kiện tiên quyết**: Thiết lập Lab 1 (Kiro IDE, AWS credentials, MCP servers)

**Kịch bản**: Bạn làm việc tại một công ty thương mại điện tử cần tự động hóa quy trình trả hàng và hoàn tiền của khách hàng. Quy trình thủ công hiện tại chậm và dễ xảy ra lỗi. Xây dựng một trợ lý được hỗ trợ bởi AI xử lý tất cả thông qua hội thoại tự nhiên.

**Returns & Refunds Agent Có thể:**
- Tra cứu đơn hàng, sản phẩm và chi tiết tài khoản khách hàng từ DynamoDB
- Kiểm tra xem một mặt hàng có còn trong khung thời gian đủ điều kiện trả hàng không
- Tính toán số tiền hoàn lại chính xác dựa trên tình trạng sản phẩm và lý do trả hàng
- Truy xuất và áp dụng chính sách trả hàng theo quốc gia (US, UK, India) từ Knowledge Base
- Nhớ preferences của khách hàng qua các phiên

**Kiến trúc Mục tiêu**
- Strands agent được triển khai trên AgentCore Runtime (auto-scaling)
- Custom @tool functions cho kiểm tra điều kiện trả hàng và tính toán hoàn tiền
- Persistent memory cho recall cross-session
- Gateway kết nối với DynamoDB tables và Bedrock Knowledge Base
- Streamlit web UI với Cognito authentication
- Full CloudWatch observability (logs, traces, GenAI dashboard)

**Tiến độ Lab 2 Hoàn thành: Parts 1-2**

✅ **Part 1: Agent Đầu tiên trong 3 Lệnh (20 phút)**
- Scaffold Strands agent mới với `agentcore create`
- Test local với `agentcore dev`
- Triển khai lên AgentCore Runtime với `agentcore deploy`
- Gọi trên cloud với `agentcore invoke`

✅ **Part 2: Xây dựng Returns & Refunds Agent (25 phút)**
- Sử dụng Kiro để thêm system prompt theo domain
- Tạo @tool decorated functions:
  - `order_lookup`: Truy xuất đơn hàng theo ID
  - `customer_lookup`: Lấy preferences của khách hàng
  - `product_lookup`: Fetch chi tiết sản phẩm
  - `policy_retrieval`: Query chính sách trả hàng
- Tích hợp mock data để test
- Test và triển khai lại với logic agent đã cập nhật

**Các Part Còn lại (Chưa Hoàn thành Hôm nay)**
- Part 3: Thêm persistent memory cho recall cross-session
- Part 4: Kết nối với DynamoDB & Knowledge Base data thực
- Part 5: Xây dựng Streamlit web UI với Cognito
- Part 6: Khám phá CloudWatch observability
- Part 7: Cải tiến mở
- Part 8: So sánh AgentCore Harness
- Part 9: Đánh giá chất lượng agent
- Part 10: Bảo mật truy cập tool với policies

**Các Công nghệ Chính Được đề cập**
- **Kiro IDE**: Môi trường phát triển được hỗ trợ AI
- **Strands Agents SDK**: Framework Python @tool decorator nhẹ
- **AgentCore CLI**: Quản lý dự án (`create`, `dev`, `deploy`, `invoke`, `logs`, `traces`)
- **Amazon Bedrock**: Foundation models và managed agent runtime
- **Strands @tool**: Custom agent functions với input/output specs
- **DynamoDB**: Lưu trữ dữ liệu order, customer, product
- **Bedrock Knowledge Base**: Tài liệu chính sách trả hàng (US, UK, India)
- **Lambda**: Serverless compute cho truy cập dữ liệu
- **Cognito**: OAuth authentication cho web UI
- **Streamlit**: Python web framework cho chat UI
- **CloudWatch**: Logs, traces, và dashboards

### Diễn giả
- **Nghĩa Trần** – Agentic SA, Amazon Web Services
- **Anh Phạm** – Cloud Consultant, C-Assistant Vietnam

### Điểm nổi bật chính

**Cuộc Cách mạng Vibe Coding**
- Phát triển truyền thống yêu cầu học cú pháp, frameworks, chu kỳ debugging
- Vibe coding chỉ yêu cầu mô tả những gì bạn muốn bằng tiếng Anh đơn giản
- Code được tạo ra sẵn sàng production với tests và documentation
- Chu kỳ phát triển: ngày/tuần → phút/giờ

**Phát triển Agent Thực hành**
- Từ zero đến một AI agent hoạt động trong vòng dưới 3 giờ
- Sử dụng Kiro để tạo Strands SDK code từ mô tả ngôn ngữ tự nhiên
- Triển khai lên hạ tầng AWS được quản lý (AgentCore Runtime) với zero cấu hình infrastructure
- Trải nghiệm cách AI xử lý độ phức tạp (dependencies, error handling, best practices)

**Insights Kiến trúc**
- AgentCore tách concerns: Agent logic (code), Infrastructure (CLI), Deployment (runtime)
- MCP cung cấp protocol chuẩn cho tool integration
- Memory systems cho phép tương tác agent có trạng thái, cá nhân hóa
- Gateway pattern bảo mật truy cập vào backend resources

**Cách tiếp cận Sẵn sàng Production**
- Code được tạo ra bao gồm error handling, logging, type hints
- Automated testing đảm bảo chất lượng code
- Managed deployment infrastructure xử lý scaling, availability
- Built-in observability cung cấp production insights

### Bài học rút ra

**Tư duy Thiết kế: Tương lai được hỗ trợ AI**
- Developers hiện orchestrate các components được tạo bởi AI thay vì viết từng dòng
- Prompt engineering trở nên quan trọng như kỹ năng lập trình truyền thống
- Kiến trúc cloud-native (Lambda, DynamoDB, Bedrock) là nền tảng
- Security và observability phải được tích hợp từ ngày đầu

**Kiến trúc Kỹ thuật**
- Strands @tool decorator đơn giản hóa định nghĩa custom function
- AgentCore xử lý routing, scaling, authentication, observability
- MCP cung cấp interface universal cho external tools và data
- Memory strategies (summary, preferences, semantic) cho phép context persistence

**Áp dụng vào Công việc**
- Rapid prototyping: Mô tả một tính năng → phút → triển khai hoạt động
- Production deployment: CLI commands thay thế hàng giờ thiết lập infrastructure
- Team collaboration: Spec-driven mode đảm bảo capture requirements trước khi coding
- Continuous improvement: Observability data thúc đẩy tối ưu hóa lặp lại

### Trải nghiệm sự kiện

Tham gia Agent Forge Day 1 là một trải nghiệm thay đổi paradigm thách thức mọi thứ tôi biết về phát triển phần mềm. Format thực hành cho phép tôi áp dụng ngay các khái niệm thay vì thụ động tiêu thụ lý thuyết.

**Khám phá Vibe Coding**
- Sự hoài nghi ban đầu về "chất lượng code được tạo bởi AI" hoàn toàn tan biến sau khi thấy output production-grade đã test, có documentation
- Tốc độ từ "ý tưởng" đến "code hoạt động" thực sự gây sốc - những gì sẽ mất vài ngày với phát triển truyền thống chỉ mất vài phút
- Vòng feedback (mô tả → tạo → chạy → tinh chỉnh) loại bỏ debugging tẻ nhạt thường tiêu tốn thời gian phát triển

**Xây dựng Agents Thực**
- Bắt đầu từ dự án trống và kết thúc với một AI agent đã triển khai chứng minh workflow hoàn chỉnh
- Tích hợp nhiều AWS services (Bedrock, Lambda, DynamoDB) thông qua natural language prompts đơn giản hóa những gì truyền thống yêu cầu documentation mở rộng
- Thấy agent nhớ context của người dùng qua conversations cho thấy sức mạnh của managed memory systems

**Cộng đồng và Học tập**
- Cộng tác với người tham dự khác cung cấp nhiều góc nhìn về problem-solving
- Chuyên môn của diễn giả kết nối lý thuyết và sử dụng production thực tế
- Q&A sessions tiết lộ patterns phổ biến và best practices từ practitioners đã sử dụng các tools này

**Đột phá Kỹ thuật**
- Hiểu workflow CLI-first, Kiro-assisted làm rõ khi nào sử dụng tool nào
- Nắm bắt cách AgentCore trừu tượng hóa độ phức tạp infrastructure giải phóng bandwidth tinh thần cho thiết kế logic
- Trải nghiệm vibe coding vs spec-driven development cung cấp framework để chọn cách tiếp cận đúng

### Một số hình ảnh sự kiện

![Poster Workshop Agent Forge](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/event4_screenshot_1.png)

*Workshop chuyên sâu Agent Forge 3 ngày độc quyền với diễn giả Nghĩa Trần (Agentic SA) và Anh Phạm (C-Assistant Vietnam), sold out với 150 người tham dự*

![Xác minh Check-In Workshop](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_8.png)

*Minh chứng tham dự: Check-in được xác nhận lúc 09:00 sáng tại Bitexco Financial Tower Tầng 26*

![Phiên Lab Thực hành - Giai đoạn Thiết lập](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_1.jpg)

*Người tham dự làm việc qua Lab 1: Thiết lập Kiro IDE và AWS credentials trên EC2 instances của họ*

![Môi trường Workshop Sôi động](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_2.jpg)

*Phòng workshop đầy tham gia vào Lab 2: Xây dựng Returns & Refunds AI agent bằng vibe coding*

![Demo Kỹ thuật của Diễn giả](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_3.jpg)

*Nghĩa Trần demo các lệnh AgentCore CLI và workflow triển khai agent*

![Môi trường Lab với Nhiều Màn hình](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_4.jpg)

*Người tham dự cấu hình Kiro IDE, test vibe coding, và khám phá các lệnh AgentCore CLI*

![Demo Tạo Code của Kiro IDE](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_5.jpg)

*Demo trực tiếp: Natural language prompts trong Kiro tạo complete Strands agent code với tests*

![Giải quyết Vấn đề Cộng tác](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_6.jpg)

*Người tham dự workshop cộng tác về phát triển agent, chia sẻ approaches và strategies debugging*

![Học tập Thực hành Chuyên sâu](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_7.jpg)

*Cả phòng tham gia vào Lab 2 Part 2: Xây dựng custom @tool functions và test agent responses*

![Portal Đăng ký Sự kiện Lu.ma](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/event4_screenshot_2.png)

*Trang sự kiện hiển thị trạng thái "Sold Out" - 150 người đăng ký, nói lên nhu cầu cao cho đào tạo agentic AI*

**Minh chứng tham gia** — Điểm danh được ghi nhận và xác minh thông qua hệ thống check-in Lu.ma tại Bitexco Financial Tower, Tầng 26, vào ngày 1 tháng 8, 2026.

Nhìn chung, workshop Agent Forge Deep Dive Day 1 cung cấp nhiều hơn kiến thức kỹ thuật - nó cho một cái nhìn về tương lai thực sự của phát triển phần mềm. Sự kết hợp của vibe coding approach của Kiro và production infrastructure của AgentCore chứng minh cách AI đang chuyển đổi cơ bản trải nghiệm developer từ viết code sang orchestrate các hệ thống thông minh. Trải nghiệm thực hành từ mô tả ngôn ngữ tự nhiên đến agents đã triển khai, hoạt động trong một ngày là trải nghiệm học tập có tác động nhất trong thời gian thực tập của tôi cho đến nay.
