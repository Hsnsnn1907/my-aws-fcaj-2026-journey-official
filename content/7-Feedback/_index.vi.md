+++
title = "Chia sẻ và phản hồi"
date = 2026-07-27
weight = 7
chapter = false
+++

# Chia sẻ và phản hồi / Sharing and Feedback

## Cảm nhận về chương trình / Program Reflection

Chương trình thực tập FCAJ First Cloud AI Journey là một trong những trải nghiệm học tập giá trị nhất mà tôi từng tham gia. Điểm khác biệt lớn nhất so với các khóa học online là tính **thực chiến**: thay vì chỉ đọc docs và làm bài tập lý thuyết, tôi được giao một dự án production thật (VideoPlatform với backend microservice 2 service), phải tự thiết kế pipeline CI/CD, tự debug lỗi build, tự vận hành trên AWS Free Tier và chịu trách nhiệm về kết quả deploy. Chính áp lực "production-like" này giúp tôi học nhanh gấp 3-4 lần so với tự học thông thường, vì mỗi sai lầm đều ảnh hưởng đến demo cuối kỳ và feedback từ mentor.

Điều tôi ấn tượng nhất là **mentor FCAJ rất sẵn lòng chia sẻ**: mỗi tuần mentor đều dành 1-2 giờ để review code 1-1, chỉ ra điểm yếu trong thiết kế pipeline và gợi ý hướng cải thiện. Nhờ vậy, tôi không chỉ học được cách dùng AWS mà còn học được cách tư duy như một Cloud Engineer thực thụ: tại sao chọn ECR thay vì Docker Hub, tại sao tách Secrets Manager khỏi biến môi trường, tại sao nên dùng IAM Role thay vì access key, v.v. Cộng đồng FCAJ cũng rất supportive, các bạn cùng khóa hay chia sẻ blog hay từ AWS Blog và AWS re:Post, giúp tôi cập nhật được nhiều dịch vụ mới mà docs chính thức chưa cập nhật kịp.

## Mức độ hài lòng / Satisfaction Level

Tôi đánh giá mức độ hài lòng của mình với chương trình ở mức **9/10**. Lý do chính:

- **Nội dung thực tế, sát nhu cầu doanh nghiệp** (10/10): dự án VideoPlatform cover đầy đủ các AWS service mà một fresher Cloud Engineer cần biết (CodeBuild, ECR, S3, SSM, CloudWatch, Secrets Manager, IAM). Đây là điểm mà rất ít chương trình thực tập ở Việt Nam có thể mang lại.
- **Mentor chất lượng cao, tận tâm** (10/10): mentor không chỉ review code mà còn chia sẻ kinh nghiệm vận hành thực tế, các pitfall khi làm việc với AWS ở môi trường production.
- **Đội ngũ FCAJ tổ chức chuyên nghiệp** (9/10): quy trình đăng ký, lịch trình, điểm danh, nộp báo cáo đều rõ ràng; có nhóm chat hỗ trợ nhanh.
- **Cộng đồng supportive** (9/10): các bạn cùng khóa hay chia sẻ tài liệu, giúp đỡ lẫn nhau.
- **Hạ tầng AWS Free Tier hơi hẹp** (7/10): có 1-2 lần tôi gần vượt giới hạn S3 5GB khi chạy image Docker multi-stage, phải tối ưu `.dockerignore` và bật BuildKit layer cache. Nếu FCAJ cấp thêm budget AWS cho intern thì sẽ thoải mái hơn.
- **Buổi orientation hơi ngắn** (7/10): chỉ có 1 buổi 2 tiếng giới thiệu tổng quan, trong khi dự án thực tế khá phức tạp. Tôi mất 1-2 tuần đầu để hiểu được kiến trúc backend microservice.

Nhìn chung, 11 tuần thực tập tại FCAJ cho tôi nhiều giá trị hơn cả một khóa học AWS Solutions Architect Associate trị giá vài triệu đồng, bởi vì tôi không chỉ học lý thuyết mà còn áp dụng được vào thực tế.

## Điểm cần cải thiện / Suggested Improvements

Dựa trên trải nghiệm cá nhân, tôi xin đề xuất một số điểm FCAJ có thể cải thiện trong các kỳ thực tập tiếp theo:

1. **Kéo dài buổi orientation lên 1-2 ngày**: bao gồm workshop về AWS account setup, AWS CLI cấu hình SSO/IAM, giới thiệu tổng quan kiến trúc dự án (kiến trúc microservice, gRPC, RabbitMQ, Prisma multi-schema). Hiện tại intern mới phải tự tìm hiểu khá nhiều, dễ nản trong tuần đầu.
2. **Cấp thêm AWS credit cho intern ngoài Free Tier**: cụ thể là 20-30 USD/kỳ để cover phần vượt Free Tier khi chạy nhiều Docker image hoặc ECR scan on push liên tục. Nhiều intern ngại tối ưu vì sợ tốn tiền cá nhân.
3. **Bổ sung chương trình "mini-talk" kỹ thuật 2 tuần/lần**: mời các cựu intern hoặc kỹ sư AWS Việt Nam chia sẻ về 1 chủ đề cụ thể (ví dụ: Terraform cơ bản, Kubernetes intro, Blue-Green deployment). Điều này giúp intern mở rộng tầm nhìn ngoài dự án thực tập.
4. **Đa dạng hóa stack dự án thực tập**: hiện tại dự án chính là VideoPlatform. Một số bạn quan tâm hơn về data engineering, một số quan tâm về DevOps. Có thể rotate intern qua 2-3 dự án nhỏ khác nhau (ví dụ: 1 dự án về Lambda + API Gateway, 1 dự án về ECS Fargate) để họ có cái nhìn rộng hơn.
5. **Cải thiện tài liệu "Onboarding cho intern mới"**: hiện tại phần lớn kiến thức nền tảng nằm rải rác trong các worklog cũ. Nên có 1 file `ONBOARDING.md` tổng hợp: cấu trúc repo, cách setup môi trường local, cách chạy từng service, các lỗi thường gặp và fix.
6. **Tăng cường feedback giữa kỳ**: hiện tại chủ yếu có review code hàng tuần và review cuối kỳ. Có thể thêm 1 buổi "mid-term review" sau 5-6 tuần để đánh giá tiến độ, điều chỉnh mục tiêu nếu cần, tránh trường hợp cuối kỳ mới phát hiện đi sai hướng.

## Giới thiệu chương trình / Recommendation

**Có, tôi chắc chắn sẽ giới thiệu chương trình FCAJ First Cloud AI Journey cho bạn bè và người quen** đang theo hướng Cloud Engineer / DevOps / Backend. Lý do cụ thể:

- **Giá trị thực tế cao**: nếu bạn muốn học AWS một cách nghiêm túc mà không có điều kiện đi làm part-time tại doanh nghiệp, FCAJ là lựa chọn tốt nhất hiện tại. Bạn sẽ được làm việc trên dự án production-like, không phải chỉ làm bài tập giả lập.
- **Miễn phí nhưng chất lượng không thua chương trình trả phí**: tôi đã tham gia 1 khóa AWS online trả phí trước đó nhưng FCAJ cho tôi nhiều kinh nghiệm thực tế hơn gấp nhiều lần. Đặc biệt, khi đi phỏng vấn xin việc, nhà tuyển dụng thường rất ấn tượng khi tôi kể về dự án VideoPlatform và pipeline CI/CD đã triển khai.
- **Cộng đồng FCAJ rất chất lượng**: các bạn cùng khóa đều có tinh thần học hỏi cao, hay share tài liệu hay. Sau kỳ thực tập, tôi vẫn giữ liên lạc với 4-5 bạn cùng nhóm để cùng trao đổi về Cloud và AI.
- **Cơ hội nghề nghiệp thực sự**: nhiều bạn cựu intern FCAJ hiện đang làm việc tại FPT, VNG, VinAI, hoặc các công ty outsource Nhật Bản. Tên FCAJ trong CV là một điểm cộng lớn.

Tuy nhiên, tôi sẽ **khuyến nghị có điều kiện**: chương trình phù hợp nhất với những bạn đã có nền tảng backend cơ bản (Node.js hoặc Python) và đã quen dòng lệnh Linux. Nếu bạn là người mới hoàn toàn, nên tự học trước về Docker, Git, cơ bản Linux trong 1-2 tháng rồi hãy đăng ký, vì nhịp thực tập khá nhanh và áp lực deadline khá cao.

Cuối cùng, tôi muốn gửi lời cảm ơn chân thành đến mentor, team lead, và toàn bộ đội ngũ FCAJ đã hỗ trợ tôi trong 11 tuần qua. Chương trình này thực sự đã thay đổi hướng đi nghề nghiệp của tôi.
