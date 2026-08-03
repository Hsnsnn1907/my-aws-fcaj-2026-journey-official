+++
title = "4.4. Event 4"
date = 2026-08-01
weight = 4
chapter = false
+++

Event Name: Agent Forge - Deepdive Day 1

Date & Time: 09:00 - 12:00, August 1, 2026

Location: Level 26, Bitexco Financial Tower, Ho Chi Minh City

Role: Attendee

### Event Overview

Agent Forge is an exclusive 3-day deep dive workshop on building and scaling production-ready AI agents using Amazon Bedrock AgentCore and Kiro IDE. Day 1 (today) focuses on Kiro fundamentals and hands-on agent development using vibe coding - a revolutionary approach where developers describe features in natural language and AI generates complete, tested, production-ready code.

**What is Vibe Coding?**

Vibe coding represents the future of software development:
- Describe what you want in plain English
- Kiro IDE generates complete, tested code instantly
- No syntax memorization, no boilerplate, no debugging
- From idea to working software in minutes, not days

**What is Kiro IDE?**

Kiro is an AI-powered development environment designed specifically for vibe coding with:
- Natural language to production code transformation
- Spec-driven development for complex features (Requirements → Design → Tasks → Code)
- Agent hooks for workflow automation
- MCP (Model Context Protocol) integration for external services
- Full-stack implementation: Frontend, Backend, Cloud deployment

**What is Amazon Bedrock AgentCore?**

A managed AWS service for deploying AI agents to production with:
- **Memory**: Remember user preferences across conversations (summary, preferences, semantic)
- **Gateway**: Securely connect to external APIs and databases via MCP
- **Runtime**: Auto-scaling, managed infrastructure for agents
- **Observability**: CloudWatch logs, traces, and GenAI dashboard
- **Policies**: Cedar-based fine-grained authorization with parameter constraints

### Workshop Content: Lab 1 & Lab 2 (Partial)

#### Lab 1: Kiro and Vibe Coding (09:00 – 10:00)

**Overview**
Discover Kiro, an AI-native development environment that transforms software development through vibe coding.

**Core Capabilities**
- **Natural Language Development**: "Build a web application with user login and dashboard" → Kiro creates responsive UI, serverless backend, database, auth system, deploys to production
- **Full-Stack Implementation**: Frontend (Vite, React), Backend (NestJS, Lambda), Cloud (AWS CDK, ECR, S3)
- **Intelligent Automation**: Add features like payments or notifications → Kiro integrates APIs, updates schemas, handles security
- **Iterative Collaboration**: Real-time feedback, smart debugging, continuous improvement

**Two Development Modes**
1. **Vibe Coding** (Fast): For quick prototyping and exploration - describe what you want, Kiro generates code immediately
2. **Spec-Driven Development** (Production): For complex features - Requirements gathering → Design planning → Task sequencing → Implementation with full traceability

**What You Learned**
- How to use Kiro for rapid AI agent development
- Setting up AWS credentials and Bedrock access
- MCP server integration for external tools
- Difference between exploration and production approaches

#### Lab 2: Build & Deploy AI Agents with AgentCore CLI (10:00 – 11:00+)

**Prerequisite**: Lab 1 setup (Kiro IDE, AWS credentials, MCP servers)

**The Scenario**: You work at an e-commerce company needing to automate customer returns and refunds. Today's manual process is slow and error-prone. Build an AI-powered assistant that handles it all through natural conversation.

**The Returns & Refunds Agent Can:**
- Look up customer orders, products, and account details from DynamoDB
- Check whether an item is still within its return eligibility window
- Calculate correct refund amounts based on product condition and return reason
- Retrieve and apply country-specific return policies (US, UK, India) from Knowledge Base
- Remember customer preferences across sessions

**Target Architecture**
- Strands agent deployed on AgentCore Runtime (auto-scaling)
- Custom @tool functions for return eligibility and refund calculations
- Persistent memory for cross-session recall
- Gateway connecting to DynamoDB tables and Bedrock Knowledge Base
- Streamlit web UI with Cognito authentication
- Full CloudWatch observability (logs, traces, GenAI dashboard)

**Lab 2 Progress Completed: Parts 1-2**

**Part 1: Your First Agent in 3 Commands (20 min)**
- Scaffolded new Strands agent with `agentcore create`
- Tested locally with `agentcore dev`
- Deployed to AgentCore Runtime with `agentcore deploy`
- Invoked in cloud with `agentcore invoke`

**Part 2: Build the Returns & Refunds Agent (25 min)**
- Used Kiro to add domain-specific system prompt
- Created @tool decorated functions:
  - `order_lookup`: Retrieve order by ID
  - `customer_lookup`: Get customer preferences
  - `product_lookup`: Fetch product details
  - `policy_retrieval`: Query return policies
- Integrated mock data for testing
- Tested and redeployed with updated agent logic

**Remaining Parts (Not Completed Today)**
- Part 3: Add persistent memory for cross-session recall
- Part 4: Connect to real DynamoDB & Knowledge Base data
- Part 5: Build Streamlit web UI with Cognito
- Part 6: Explore CloudWatch observability
- Part 7: Open-ended enhancements
- Part 8: AgentCore Harness comparison
- Part 9: Agent quality evaluation
- Part 10: Secure tool access with policies

**Key Technologies Covered**
- **Kiro IDE**: AI-powered development environment
- **Strands Agents SDK**: Lightweight Python @tool decorator framework
- **AgentCore CLI**: Project management (`create`, `dev`, `deploy`, `invoke`, `logs`, `traces`)
- **Amazon Bedrock**: Foundation models and managed agent runtime
- **Strands @tool**: Custom agent functions with input/output specs
- **DynamoDB**: Order, customer, product data storage
- **Bedrock Knowledge Base**: Return policy documents (US, UK, India)
- **Lambda**: Serverless compute for data access
- **Cognito**: OAuth authentication for web UI
- **Streamlit**: Python web framework for chat UI
- **CloudWatch**: Logs, traces, and dashboards

### Speakers
- **Nghĩa Trần** – Agentic SA, Amazon Web Services
- **Anh Phạm** – Cloud Consultant, C-Assistant Vietnam

### Key Highlights

**The Vibe Coding Revolution**
- Traditional development requires learning syntax, frameworks, debugging cycles
- Vibe coding only requires describing what you want in plain English
- Generated code is production-ready with tests and documentation
- Development cycles: days/weeks → minutes/hours

**Hands-On Agent Development**
- Went from zero to a working AI agent in under 3 hours
- Used Kiro to generate Strands SDK code from natural language descriptions
- Deployed to managed AWS infrastructure (AgentCore Runtime) with zero infrastructure configuration
- Experienced how AI handles complexity (dependencies, error handling, best practices)

**Architectural Insights**
- AgentCore separates concerns: Agent logic (code), Infrastructure (CLI), Deployment (runtime)
- MCP provides a standard protocol for tool integration
- Memory systems enable stateful, personalized agent interactions
- Gateway pattern secures access to backend resources

**Production-Ready Approach**
- Generated code includes error handling, logging, type hints
- Automated testing ensures code quality
- Managed deployment infrastructure handles scaling, availability
- Built-in observability provides production insights

### Key Takeaways

**Design Mindset: The AI-Assisted Future**
- Developers now orchestrate AI-generated components rather than writing every line
- Prompt engineering becomes as important as traditional programming skills
- Cloud-native architectures (Lambda, DynamoDB, Bedrock) are the foundation
- Security and observability must be integrated from day one

**Technical Architecture**
- Strands @tool decorator simplifies custom function definition
- AgentCore handles routing, scaling, authentication, observability
- MCP provides a universal interface for external tools and data
- Memory strategies (summary, preferences, semantic) enable context persistence

**Applying to Work**
- Rapid prototyping: Describe a feature → minutes → working implementation
- Production deployment: CLI commands replace hours of infrastructure setup
- Team collaboration: Spec-driven mode ensures requirements capture before coding
- Continuous improvement: Observability data drives iterative optimization

### Event Experience

Participating in Agent Forge Day 1 was a paradigm-shifting experience that challenged everything I knew about software development. The hands-on format allowed me to immediately apply concepts rather than passively consuming theory.

**The Vibe Coding Revelation**
- Initial skepticism about "AI-generated code quality" completely dissolved after seeing tested, documented, production-grade output
- The speed from "idea" to "working code" was genuinely shocking - what would take me days with traditional development took minutes
- The feedback loop (describe → generate → run → refine) eliminated the tedious debugging that normally consumes development time

**Building Real Agents**
- Starting from a blank project and ending with a deployed AI agent demonstrated the complete workflow
- Integrating multiple AWS services (Bedrock, Lambda, DynamoDB) through natural language prompts simplified what traditionally requires extensive documentation
- Seeing the agent remember user context across conversations showed the power of managed memory systems

**Community and Learning**
- Collaborating with other attendees provided multiple perspectives on problem-solving
- Speaker expertise bridged theory and real-world production usage
- Q&A sessions revealed common patterns and best practices from practitioners already using these tools

**Technical Breakthroughs**
- Understanding the CLI-first, Kiro-assisted workflow clarified when to use which tool
- Grasping how AgentCore abstracts infrastructure complexity freed mental bandwidth for logic design
- Experiencing vibe coding vs spec-driven development provided a framework for selecting the right approach

### Some event photos

![Agent Forge Workshop Poster](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/event4_screenshot_1.png)

*The exclusive 3-day Agent Forge Deep Dive workshop promotion featuring speakers Nghĩa Trần (Agentic SA) and Anh Phạm (C-Assistant Vietnam), sold out with 150 attendees*

![Workshop Check-In Verification](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_8.png)

*Proof of attendance: Check-in confirmed at 09:00 AM at Bitexco Financial Tower Level 26*

![Hands-On Lab Session - Setup Phase](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_1.jpg)

*Participants working through Lab 1: Setting up Kiro IDE and AWS credentials on their EC2 instances*

![Active Workshop Environment](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_2.jpg)

*Full workshop room engaged in Lab 2: Building the Returns & Refunds AI agent using vibe coding*

![Speaker Technical Demonstration](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_3.jpg)

*Nghĩa Trần demonstrating AgentCore CLI commands and agent deployment workflow*

![Lab Environment with Multiple Screens](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_4.jpg)

*Attendees configuring Kiro IDE, testing vibe coding, and exploring AgentCore CLI commands*

![Kiro IDE Code Generation Demo](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_5.jpg)

*Live demonstration: Natural language prompts in Kiro generating complete Strands agent code with tests*

![Collaborative Problem Solving](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_6.jpg)

*Workshop participants collaborating on agent development, sharing approaches and debugging strategies*

![Intensive Hands-On Learning](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/confirm_event4_7.jpg)

*Full room engaged in Lab 2 Part 2: Building custom @tool functions and testing agent responses*

![Lu.ma Event Registration Portal](/my-aws-fcaj-2026-journey-official/images/4-EventParticipated/event4_screenshot_2.png)

*Event page showing "Sold Out" status - 150 registered attendees, highlighting the high demand for agentic AI training*

**Proof of participation** — Attendance recorded and verified through Lu.ma check-in system at Bitexco Financial Tower, Level 26, on August 1, 2026.

Overall, the Agent Forge Deep Dive Day 1 workshop provided far more than technical knowledge - it offered a glimpse into the actual future of software development. The combination of Kiro's vibe coding approach and AgentCore's production infrastructure demonstrated how AI is fundamentally transforming the developer experience from writing code to orchestrating intelligent systems. The hands-on experience of going from natural language descriptions to deployed, working agents in a single day was the most impactful learning experience of my internship so far.
