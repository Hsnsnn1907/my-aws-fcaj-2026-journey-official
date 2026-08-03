+++
title = "7. Sharing and Feedback"
date = 2026-07-27
weight = 7
chapter = false
+++## Program Reflection

The FCAJ First Cloud AI Journey internship is one of the most valuable learning experiences I have ever joined. The biggest differentiator from online courses is its **real-world** nature: instead of just reading docs and doing theoretical exercises, I was assigned an actual production-like project (VideoPlatform with a 2-service microservice backend), had to design the CI/CD pipeline myself, debug build errors, operate on AWS Free Tier, and take responsibility for deploy outcomes. This "production-like" pressure made me learn 3-4 times faster than self-study, because every mistake affected the final demo and mentor feedback.

What impressed me most was how **approachable FCAJ mentors are**: every week, my mentor spent 1-2 hours doing 1-on-1 code review, pointing out weaknesses in pipeline design and suggesting improvements. Thanks to that, I learned not only how to use AWS but also how to think like a real Cloud Engineer: why to choose ECR over Docker Hub, why to separate Secrets Manager from environment variables, why to use IAM Role instead of access keys, etc. The FCAJ community is also very supportive; fellow interns often shared interesting blogs from AWS Blog and AWS re:Post, helping me keep up with many new services that official docs had not yet covered.

## Satisfaction Level

I rate my satisfaction with the program at **9/10**. Main reasons:

- **Practical, business-aligned content** (10/10): the VideoPlatform project covers all AWS services a fresher Cloud Engineer needs to know (CodeBuild, ECR, S3, SSM, CloudWatch, Secrets Manager, IAM). Very few internship programs in Vietnam can offer this.
- **High-quality, dedicated mentors** (10/10): mentors not only review code but also share real operational experience and pitfalls when working with AWS in production.
- **Professional FCAJ organization** (9/10): registration, scheduling, attendance, and report submission processes are all clear; there's a quick-support chat group.
- **Supportive community** (9/10): fellow interns frequently share resources and help each other.
- **AWS Free Tier is a bit tight** (7/10): 1-2 times I almost hit the 5GB S3 limit when running multi-stage Docker images and had to optimize `.dockerignore` and enable BuildKit layer cache. If FCAJ could allocate extra AWS budget for interns, that would be more comfortable.
- **Orientation session is a bit short** (7/10): only a single 2-hour overview, while the actual project is quite complex. I spent the first 1-2 weeks trying to understand the microservice backend architecture.

Overall, 11 weeks at FCAJ gave me more value than an entire AWS Solutions Architect Associate course costing several million VND, because I didn't just learn theory — I applied it in practice.

## Suggested Improvements

Based on my personal experience, I would like to suggest several improvements FCAJ could make for future internship cohorts:

1. **Extend orientation to 1-2 days**: include a workshop on AWS account setup, AWS CLI with SSO/IAM configuration, and an overview of the project architecture (microservice architecture, gRPC, RabbitMQ, Prisma multi-schema). Currently new interns have to figure out a lot on their own, which can be discouraging in the first week.
2. **Allocate extra AWS credits beyond Free Tier**: specifically 20-30 USD per cohort to cover over-limit usage when running multiple Docker images or continuous ECR scan on push. Many interns hesitate to optimize because they fear personal cost.
3. **Add a biweekly technical "mini-talk"**: invite former interns or Vietnamese AWS engineers to share on a specific topic (e.g. Terraform basics, Kubernetes intro, Blue-Green deployment). This would help interns broaden their view beyond the actual project.
4. **Diversify the project stack**: the main project is currently VideoPlatform. Some interns are more interested in data engineering, others in DevOps. Interns could rotate through 2-3 smaller projects (e.g. 1 about Lambda + API Gateway, 1 about ECS Fargate) for a broader perspective.
5. **Improve "Onboarding for new interns" documentation**: most foundational knowledge is currently scattered across old worklogs. A consolidated `ONBOARDING.md` would help: repo structure, local environment setup, how to run each service, common errors and fixes.
6. **Strengthen mid-term feedback**: currently we mostly have weekly code review and final review. Adding a "mid-term review" after 5-6 weeks would help assess progress, adjust goals if needed, and avoid the situation where the wrong direction is only discovered at the end of the program.

## Recommendation

**Yes, I will definitely recommend the FCAJ First Cloud AI Journey program to my friends and acquaintances** who are pursuing Cloud Engineer / DevOps / Backend paths. Specific reasons:

- **High practical value**: if you want to seriously learn AWS without being able to work part-time at a company, FCAJ is currently the best option. You'll work on a production-like project, not just simulated exercises.
- **Free but quality matches paid programs**: I had previously taken a paid AWS online course, but FCAJ gave me many times more practical experience. Especially during job interviews, recruiters were very impressed when I described the VideoPlatform project and the CI/CD pipeline I had implemented.
- **Quality FCAJ community**: fellow interns all have a strong learning spirit and share good resources. After the program, I still keep in touch with 4-5 group members to exchange ideas about Cloud and AI.
- **Real career opportunities**: many FCAJ alumni now work at FPT, VNG, VinAI, or Japanese outsourcing companies. FCAJ on the CV is a big plus.

However, I will give a **conditional recommendation**: the program fits best with people who already have a basic backend foundation (Node.js or Python) and are familiar with the Linux command line. If you are a complete beginner, I recommend self-studying Docker, Git, and basic Linux for 1-2 months before applying, because the internship pace is quite fast and deadlines are tight.

Finally, I would like to sincerely thank my mentor, team lead, and the entire FCAJ team for supporting me over the past 11 weeks. This program has truly changed my career direction.
