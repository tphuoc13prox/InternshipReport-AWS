---
title: "Event 5"
date: 2026-11-14
weight: 5
chapter: false
pre: " <b> 4.5. </b> "
---

# Summary Report: "FCAJ x FPTU: Cloud Mastery 1"

### Event Objectives

- Explain the gap between an AI prototype and an AI application that runs reliably in production.
- Introduce the four pillars of production AI on AWS: security, scalability, reliability and cost.
- Present real enterprise case studies of AI agents running on Amazon Bedrock AgentCore.
- Introduce the main components of Amazon Bedrock AgentCore, including Runtime, Identity, Gateway, Memory and Observability.
- Show how the Strands Agents SDK and AgentCore can be combined to build and operate AI agents.

### Speakers

- **Khang Duong – Member, FCAJ**
- **Huyen – AI Engineer Fresher, FPT Software HCMC**
- **Nguyen Lam Anh – AI Engineer, FCAJ**
- **Nguyen Quoc Bao – Cloud Engineer, AWS First Cloud AI Journey**
- **Huynh An Khuong – AI Engineer, FPT Software**

### Event Information

- **Event:** FCAJ x FPTU: Cloud Mastery 1
- **Date:** November 14, 2026
- **Time:** 09:00 AM – 12:00 PM
- **Venue:** 26th Floor, Bitexco Financial Tower, Ho Chi Minh City
- **Organizers:** AWS First Cloud AI Journey (FCAJ) and FPT University (FPTU)
- **Role:** Attendee

### Key Highlights

#### From Prototype to Production

- Compared academic AI with enterprise AI using the analogy of a home kitchen and a restaurant chain: enterprise AI is measured by business value rather than accuracy alone, works with constantly changing data, and must serve millions of users 24/7 without downtime.
- Explained that the model is only one part of a production system, which also needs a data pipeline, monitoring, security, scalability, reliability and cost control.

#### The Four Pillars on AWS

- Security: AWS KMS for data privacy, AWS IAM for access control, and guardrails for AI-specific risks such as users tricking the model into mistakes.
- Scalability and reliability: AWS Lambda, Auto Scaling and Elastic Load Balancing to handle changing traffic, together with health checks, retries, failover and alarms.
- Cost efficiency through pay-per-use, right-sizing and caching, and the idea of using AI wisely: a larger model as the orchestrator, smaller and cheaper models for simpler tasks, and caching for repeated questions.

#### Enterprise Case Studies: Cox Automotive and Mobileye

- Cox Automotive identified 57 agentic AI opportunities but faced two problems: agents needed least-privilege access to sensitive data, and there was no standard environment to run them. By standardizing on AgentCore Runtime, AgentCore Identity and Amazon Bedrock Guardrails, the company moved 17 agent solutions into production.
- At Mobileye, 66% of support tickets were routine status inquiries. A hybrid design connected the on-premises ticket system to a support agent on AgentCore Runtime, with AWS Secrets Manager and MCP access to internal services, and reduced ticket response time from hours to about one minute.
- The session closed with three takeaways: build security and scale into the foundation rather than as a patch, agents only create value when they reach real systems and live data, and an agent cannot be trusted unless it is measured.

#### AI in Enterprise: Amazon Bedrock AgentCore

- Used the Duolingo case study to show why a shared agent platform helps when every team would otherwise rebuild the same setup.
- Walked through building an order-lookup agent with the Strands Agents SDK and running it on AgentCore Runtime, which provides managed compute, isolated sessions and scaling.
- Highlighted that teams still own their business rules, data, tools and testing, even when the runtime is managed.

#### The Core of Agent: Identity, Gateway, Memory and Observability

- AgentCore Identity separates inbound authentication (who the user is and whether they may call the agent) from outbound authentication (what the agent may access on the user's behalf).
- AgentCore Gateway gives agents a single, secured way to reach many tools, APIs and MCP servers, while AgentCore Memory provides short-term memory for the current conversation and long-term memory for insights such as user preferences and summaries.
- AgentCore Observability, integrated with Amazon CloudWatch, traces each step of an agent, including model calls, tool calls, latency and token usage, following the principle "you cannot fix what you cannot see".

#### Q&A, Mini Quiz and Wrap-up

- Participants asked the speakers questions about the topics covered during the sessions.
- Took part in a mini quiz at the end of the event.
- Completed the event feedback form and joined the group photo with speakers and participants.

### Key Takeaways

#### Production-Ready AI

- A working model is only the starting point; security, scalability, reliability and cost must be designed in from the beginning.

#### Agent Architecture

- Gained a clearer picture of how an agent framework such as Strands works together with a managed platform such as Amazon Bedrock AgentCore.
- Understood why identity, tool access and memory should be handled by shared, secure components instead of being rebuilt in every project.

#### Security for AI Agents

- Learned to treat inbound and outbound access separately and to give each agent only the permissions it needs.

#### Observability and Evaluation

- Recognized that agents often fail without throwing errors, so every run should be logged, traced and measured against targets set in advance.

### Applying to Work

- Add Amazon Bedrock Guardrails to the chatbot in our CPAWS-commodity-forecasting project to screen user inputs and model outputs before they reach the Lambda `/chat` response.
- Review the IAM role of the `/chat` Lambda function so that it only has the permissions it needs to invoke the Claude Haiku model on Amazon Bedrock.
- Log latency, token usage and errors for each chatbot request in Amazon CloudWatch, and define simple targets for response time and cost.
- Add a small set of test questions to the CI pipeline to check the chatbot's answers before each deployment.
- Explore the Strands Agents SDK and AgentCore Runtime if the chatbot later needs to call tools, such as looking up the latest commodity price forecasts.

### Event Experience

Participating in "FCAJ x FPTU: Cloud Mastery 1" provided me with a practical view of what it takes to move AI applications from a prototype to production on AWS.

The first session was easy to follow because it explained the four pillars of production AI through the example of a restaurant chain. It made me realize that a model that works in a demo is very different from a system that must stay secure, scalable and affordable for many users.

The case studies of Cox Automotive and Mobileye were particularly valuable because they showed concrete results from companies running AI agents in production. They also showed that most of the effort lies in security, integration with existing systems and measurement, not only in the model itself.

The later sessions on Amazon Bedrock AgentCore gave me a clear map of its components, from Runtime and Identity to Gateway, Memory and Observability. Since I am responsible for the chatbot in my team's project, the parts on guardrails, least-privilege access and observability were directly relevant to my own work.

Overall, the event strengthened my understanding of how enterprise AI agents are built and operated on AWS while also highlighting the importance of security, observability and careful design when bringing AI into production.

### Some Event Photos

![Event 5 – photo 1](/images/4-Event/event5_img1.jpg)

![Event 5 – photo 2](/images/4-Event/event5_img2.jpg)
