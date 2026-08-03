+++
title = "5. Workshop"
date = 2026-08-03
weight = 5
chapter = false
+++

# CI/CD Pipeline Implementation Workshop

{{< notice warning >}}
Lưu ý: Thông tin dưới đây chỉ mang tính tham khảo. Vui lòng không sao chép nguyên văn cho báo cáo của bạn, bao gồm cả cảnh báo này.
{{< /notice >}}

## Tổng quan (Overview)

Workshop kéo dài 12 tuần này ghi lại hành trình chuyển đổi CI/CD toàn diện từ quy trình deploy thủ công sang tự động hóa AWS-native cho backend của nền tảng streaming video. Quy trình thủ công tốn thời gian ban đầu đã phát triển thành một pipeline tự động hóa, hợp lý, có khả năng deploy hai microservice với bảo mật, giám sát và độ tin cậy cấp production.

Hành trình mang lại giá trị kinh doanh đáng kể bằng cách giảm thời gian deploy từ 20-30 phút xuống chỉ còn 4-5 phút cho mỗi service, loại bỏ hoàn toàn các lỗi cấu hình thủ công trước đây gây ra sự cố production. Bằng cách tập trung quản lý secrets trong AWS Secrets Manager và triển khai truy cập dựa trên IAM role, nền tảng đã đạt được bảo mật cấp doanh nghiệp mà không ảnh hưởng đến tốc độ phát triển.

Về mặt kỹ thuật, workshop này bao gồm việc đóng gói hóa service NestJS GraphQL API và FastAPI search service, tích hợp với 8 dịch vụ AWS (EC2, ECR, CodeBuild, S3, Secrets Manager, SSM, CloudWatch, IAM), và triển khai pipeline deploy đầy đủ từ commit code đến deploy production. Giải pháp xử lý các yêu cầu phức tạp bao gồm database migration, phụ thuộc giữa các service, và inject secret runtime.

Người tham gia sẽ có được kinh nghiệm thực hành với thiết kế kiến trúc AWS, Docker multi-stage builds, các best practice bảo mật production, chiến lược tối ưu chi phí, và khắc phục các sự cố deploy thực tế. Workshop này được thiết kế cho các developer và DevOps engineer muốn triển khai pipeline CI/CD mạnh mẽ trong môi trường AWS.

## Kiến trúc (Architecture)

![CI/CD Architecture](/my-aws-fcaj-2026-journey-official/images/5-Workshop/architecture_diagram.png)

### Tổng quan các thành phần (Components Overview)

| AWS Service | Vai trò trong Pipeline | Chi tiết cấu hình |
|-------------|----------------------|----------------------|
| **GitHub** | Source control | Branch: `feat/CI-CD`, Path filters: `^api_service/.*$` và `^search_service/.*$` |
| **CodeBuild** | Build automation | 2 project: `build-api-service`, `build-search-service`, Ubuntu 22.04, Docker privileged mode |
| **ECR** | Container registry | 2 private repo: `api-service`, `search-service`, bật scan-on-push |
| **S3** | Artifact storage | `videoplatform-deploy-artifacts-dsk`, đường dẫn compose files: `compose/docker-compose.*.yml` |
| **Secrets Manager** | Runtime secrets | 2 secret: `prod/backend/api-service`, `prod/backend/search-service`, định dạng JSON |
| **EC2** | Production host | t2.medium (2 vCPU, 4 GB RAM), Ubuntu 22.04, Public IP: 98.81.144.110 |
| **SSM** | Remote deployment trigger | Document `AWS-RunShellScript`, target EC2 instance: `i-037a4cd636a68eb7e` |
| **CloudWatch** | Monitoring | Build logs, SSM command logs, EC2 metrics, alarms cho ngưỡng CPU/Disk |

### Luồng dữ liệu (Data Flow)

1. **Code Push**: Developer push thay đổi lên branch `feat/CI-CD` trong GitHub repository
2. **Webhook Trigger**: GitHub webhook trigger project CodeBuild phù hợp dựa trên path filter
3. **Build Phase**: CodeBuild clone repository, build Docker image sử dụng multi-stage builds
4. **Registry Push**: Image sau khi build được push lên ECR repository với tag `latest` và git SHA
5. **Artifact Sync**: Docker Compose files được đồng bộ lên S3 bucket để deploy
6. **Deployment Trigger**: Lệnh SSM được gửi tới EC2 instance để thực thi deployment script
7. **EC2 Deployment**: EC2 pull image mới nhất từ ECR, lấy secrets từ Secrets Manager
8. **Container Launch**: Docker Compose khởi động container với các biến môi trường được inject
9. **Health Verification**: Các service được validate và CloudWatch monitoring bắt đầu
10. **Pipeline Complete**: Tổng thời gian deploy: 4-5 phút cho mỗi service

## Yêu cầu trước (Prerequisites)

- **AWS Free Tier account**: Tài khoản AWS đang hoạt động với quyền IAM để tạo resource
- **GitHub account**: Quyền truy cập repository và cấu hình webhook
- **Kiến thức kỹ thuật**: Hiểu biết cơ bản về Docker, Node.js, Python, Linux CLI
- **Công cụ local**: AWS CLI v2, Docker Desktop, Git, SSH client
- **Thời gian ước tính**: 12 tuần (bán thời gian, ~20 giờ/tuần cho triển khai và tài liệu)
- **Chi phí ước tính**: ~$23/tháng (EC2 t2.medium + ECR storage + S3 + Secrets Manager + data transfer tối thiểu)
- **AWS Region**: us-east-1 (North Virginia)
- **AWS Account ID**: 772706200692

---

## Các phần của Workshop (Workshop Sections)

Điều hướng qua các phần dưới đây để theo dõi toàn bộ workshop:

1. **Introduction** - Tổng quan workshop và những gì bạn sẽ học
2. **Prerequisites** - Thiết lập tài khoản AWS, công cụ và yêu cầu
3. **AWS Account Setup** - Khởi chạy EC2 instance và cấu hình bootstrap
4. **AWS Services Configuration** - Thiết lập ECR, Secrets Manager, S3 và IAM
5. **Dockerfiles and Scripts** - Container builds và tự động hóa deploy
6. **CodeBuild Setup** - Cấu hình build project và thiết lập webhook
7. **Production Deployment** - Kiểm thử và xác minh pipeline hoàn chỉnh
8. **Monitoring** - CloudWatch logs, alarms và theo dõi chi phí
9. **Security** - Best practices để bảo mật pipeline
10. **Troubleshooting** - Các sự cố thường gặp và giải pháp
11. **Next Steps** - Tối ưu chi phí và cải tiến trong tương lai

---

## Thành tựu chính (Key Achievements)

- **Thời gian deploy**: Giảm từ 20-30 phút → 4-5 phút (cải thiện 83%)
- **Tỷ lệ lỗi**: Loại bỏ các sai sót khi deploy thủ công và configuration drift
- **Bảo mật**: Quản lý secrets tập trung, IAM roles, không có credentials trong code, ECR scanning
- **Khả năng mở rộng**: Sẵn sàng mở rộng cho nhiều môi trường (staging, production, feature branches)
- **Observability**: CloudWatch monitoring cho tất cả thành phần với alarms có thể hành động
- **Chi phí**: ~$22/tháng (trong giới hạn Free Tier, ít hơn 60% so với các managed service tương đương)
- **Độ tin cậy**: Khả năng rollback tự động, health checks, quản lý phụ thuộc

**Thành tựu kỹ thuật:**
- Đóng gói hóa 2 microservice với multi-stage Docker builds
- Triển khai tích hợp 8 dịch vụ AWS với quyền IAM phù hợp
- Tạo pipeline tự động từ commit code đến deploy production
- Giải quyết 8 blocking issue và 6 audit finding trong quá trình triển khai
- Thiết lập các thực hành bảo mật và giám sát cấp production

![GitHub Repository](/my-aws-fcaj-2026-journey-official/images/5-Workshop/10_github_repository.png)

---

## Thông tin Repository (Repository Information)

**Repository:** [khiem918/VideoPlatformServer](https://github.com/khiem918/VideoPlatformServer)  
**AWS Account:** 772706200692 (us-east-1)  
**Production URL:** http://98.81.144.110:3000/graphql

---

**Cập nhật lần cuối:** August 3, 2026  
**Tác giả:** Ho Khang - FCAJ Internship 2026  
**Thời lượng:** 12 tuần (May 12 - July 31, 2026)  
**Số dịch vụ AWS sử dụng:** 8  
**Tổng số build thành công:** 42  
**Tỷ lệ deploy thành công:** 100% sau khi fix ở tuần 10