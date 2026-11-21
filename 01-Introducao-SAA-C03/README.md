# 01 Introduction to SAA-C03

## 📋 Table of Contents
- [Module Objectives](#-module-objectives)
- [Core Concepts](#-core-concepts)
- [Architecture and Components](#-architecture-and-components)
- [Important Settings for the Exam](#-important-settings-for-the-exam)
- [Service Comparison](#-service-comparison)
- [Exam Tips and Pitfalls](#-exam-tips-and-pitfalls)
- [For the Exam](#-for-the-exam)
- [Related Links](#-related-links)

## 🎯 Module Objectives

By the end of this module, you should be able to:

- understand exactly what the SAA-C03 exam measures and how the four domains show up in the questions
- tell apart the depth required in the SAA-C03 compared to the CLF-C02
- build an 8-week study plan without wasting time on low-return topics
- recognize the services most likely to appear on the exam and in which contexts they are tested
- think like a solutions architect: identify the requirement, constraint, risk, trade-off, and the decision best suited to the scenario

## 📚 Core Concepts

The AWS Certified Solutions Architect – Associate exam, code SAA-C03, is not a test of memorized definitions. It evaluates your ability to make architectural decisions in realistic scenarios. Instead of merely asking what a service does, AWS normally presents a business need, an operational constraint, and a technical requirement. From that, you need to pick the option that best balances availability, performance, security, and cost. This shift in focus is what separates the SAA-C03 from more introductory exams.

The four exam domains help organize this reasoning. The first is Design Resilient Architectures, at 30% of the exam. This is where multi-AZ, multi-region, decoupling, queues, replication, backup, disaster recovery, and fault tolerance come in. In many questions, the correct answer is not the most advanced technology, but the one that removes a single point of failure with the least operational complexity. That is why understanding Route 53, Load Balancers, Auto Scaling, S3 Replication, RDS Multi-AZ, Aurora, and patterns with SQS or EventBridge is essential.

The second domain is Design High-Performing Architectures, at 28%. This block measures your ability to select the most suitable compute, database, network, or storage option for throughput, latency, scalability, and access pattern. The exam commonly compares EC2 with Lambda, RDS with DynamoDB, CloudFront with Global Accelerator, gp3 with io2, ElastiCache Redis with Memcached, and also asks for performance tuning for S3, Kinesis, and Athena. The central point here is understanding the workload profile. AWS always rewards answers that align the type of service with the characteristics of the traffic or the application.

The third domain is Design Secure Applications and Architectures, at 24%. Knowing IAM at a superficial level is not enough. You need to master the differences between role, policy, resource-based policy, SCP, permission boundary, trust policy, and KMS key policy. You also need to know when to use Secrets Manager instead of Parameter Store, when to prefer a VPC Endpoint over public traffic, how CloudTrail and Config complement each other, and which service reduces the attack surface without adding too much operational burden. Many security questions are actually design questions: how to grant the minimum required access, how to isolate traffic, how to encrypt data at rest and in transit, and how to audit changes.

The fourth domain is Design Cost-Optimized Architectures, at 18%. Although it has the lowest weight, it cuts across all the others. AWS likes to ask which solution meets the requirement with the lowest operational cost, the least administration effort, or the best fit to actual consumption. Savings Plans, Spot instances, S3 Intelligent-Tiering, serverless, appropriate storage classes, automated shutdown, and managed services come up all the time. The common trap is choosing the most powerful architecture, yet one that is excessive for the stated need.

Compared with the CLF-C02, the most important difference is not the number of services, but the depth of the decision. In the CLF-C02, it was enough to know that Amazon S3 is object storage. In the SAA-C03, you need to know which class to use, how to configure lifecycle, when to enable versioning, when to use CRR, when to apply a pre-signed URL, when to prefer a VPC endpoint, and how this impacts security and cost. The same applies to EC2, VPC, IAM, RDS, DynamoDB, and CloudFront. The exam assumes you already know the catalog at a high level; now it wants you to design coherent solutions.

A practical way to study is to think in layers. The first layer is the foundation: IAM, VPC, EC2, S3, RDS, Route 53, and CloudWatch. The second is patterns: high availability, scalability, caching, decoupling, streaming, and observability. The third is decision-making: choosing between services that solve similar problems, evaluating management, elasticity, latency, compliance, and cost. When the exam mentions "minimal operational effort," for example, it is pointing to managed services. When it mentions "low global latency for HTTP," CloudFront tends to come into play. When it talks about "global TCP/UDP with a fixed IP," Global Accelerator makes more sense than CloudFront.

The exam also requires reasoning about trade-offs. There is no perfect answer. A multi-region architecture is more resilient, but more expensive and more complex. A relational database provides transactional consistency, but may not scale horizontally like a NoSQL database. A serverless solution reduces administration, but may require attention to cold starts, timeout limits, and VPC integration. The architect does not just ask "does it work?". They ask "does it work within the requirement, with acceptable risk, reasonable cost, and sustainable operation?". This is the mental model you should train from the first module.

Another important point is reading the question statement carefully. Many AWS options are technically valid, but one of them better meets the detail hidden in the scenario: a compliance requirement, a small team, latency between regions, predictable traffic, a need for auditing, or a tight budget. In general, the words that define the correct answer are: "most cost-effective", "least operational overhead", "highly available", "fault tolerant", "securely", "near real-time", "serverless", and "with minimal changes". Those who ignore these qualifiers fall into the traps.

For a realistic 8-week schedule, the best strategy is to alternate depth and review. Study one technical module at a time, finish with questions, and go back to weak topics the next day. At the end of each week, review using flashcards and cheatsheets. In the final stretch, use the case studies to connect services and simulate how the exam mixes topics. The SAA-C03 rarely tests a service in isolation; it tests the interaction between services in an architecture context.

The most recurring services deserve disproportionate attention, because they appear in a large part of the exam: EC2, Auto Scaling, ELB, S3, VPC, IAM, RDS, Aurora, DynamoDB, Lambda, API Gateway, SQS, SNS, EventBridge, Route 53, CloudFront, CloudWatch, CloudTrail, and KMS. If you master this group well, you already cover most of the decisions tested. Less frequent services such as Snow Family, Transfer Family, Neptune, QLDB, App Runner, or Bedrock come up more as contextual choices. The common mistake is spending too much energy on rare services and too little on the core blocks.

In summary, the SAA-C03 measures architectural maturity at the associate level. You do not need to be an expert in every service, but you need to quickly recognize which pattern AWS is trying to assess and choose the most balanced solution. This repository was structured exactly for that: consolidating fundamentals, training scenario reading, and turning the service catalog into architecture decisions.

## 🏗️ Architecture and Components

The diagram below represents how the exam usually connects the main architectural decision blocks:

```text
                 +-----------------------+
                 |  Business Requirements|
                 |  HA | Security | Cost |
                 +-----------+-----------+
                             |
                             v
                 +-----------------------+
                 |  Architectural Decision|
                 | Trade-offs/constraints |
                 +-----------+-----------+
                             |
      +----------------------+-----------------------+
      |                      |                       |
      v                      v                       v
+-------------+      +---------------+      +----------------+
| Compute     |      | Data / Storage|      | Network / Edge |
| EC2 Lambda  |      | S3 RDS Dynamo |      | VPC R53 CF ELB |
+------+------+      +-------+-------+      +--------+-------+
       |                     |                       |
       +---------------------+-----------------------+
                             |
                             v
                 +-----------------------+
                 | Security & Operations |
                 | IAM KMS CW CT Config  |
                 +-----------------------+
```

## ⚙️ Important Settings for the Exam

| Item | Value / Weight | Why it matters |
|---|---:|---|
| Questions | 65 | Sets the exam pace and time prioritization |
| Total time | 130 minutes | Average close to 2 minutes per question |
| Resilient domain | 30% | Highest weight, appears in HA, DR, and decoupling |
| High-Performing domain | 28% | Strong presence in EC2, storage, and databases |
| Secure domain | 24% | IAM, KMS, VPC, and auditing appear frequently |
| Cost-Optimized domain | 18% | Lowest formal weight, but present in almost every comparison |
| Default region for labs | us-east-1 | Greatest compatibility and least friction with examples |
| Keyword "least operational overhead" | High priority | Usually favors managed services |
| Keyword "multi-Region" | High priority | Tends to eliminate multi-AZ-only answers |
| Keyword "near real-time" | High priority | May indicate streaming, queues, or continuous replication |

## 🔄 Service Comparison

| Exam situation | Strongest option | Common alternative | Rule of thumb |
|---|---|---|---|
| Global HTTP application with caching | CloudFront | Global Accelerator | CloudFront is an edge cache and CDN; GA accelerates TCP/UDP or HTTP without caching |
| Event-driven API | EventBridge | SNS | EventBridge is better for rule-based routing and multiple producers |
| Relational database with managed HA | RDS Multi-AZ | EC2 self-managed DB | For the exam, AWS almost always prefers the managed service |
| Ultra-low latency with predictable key-value access | DynamoDB | RDS | Use DynamoDB when horizontal scalability is a priority |
| Compute with minimal administration and variable traffic | Lambda | EC2 | Lambda tends to win if the scenario tolerates serverless limits |
| Secret encryption with native rotation | Secrets Manager | Parameter Store | Secrets Manager is the strongest answer for sensitive, rotatable secrets |

## 💡 Exam Tips and Pitfalls

- Multi-AZ does not mean multi-region. If the requirement mentions a regional disaster, think beyond a single region.
- "More secure" does not always mean "more complex". In many scenarios, AWS favors a managed service with native IAM and KMS integration.
- "Lowest cost" is not the same as "cheapest in the short term". The exam considers the total cost of operation, maintenance, and risk.
- If an option requires manual server administration and a managed option that meets the requirement exists, the managed option usually wins.
- Pay close attention to the type of traffic. HTTP, gRPC, TCP, UDP, batch, stream, and queue are not the same thing.
- Keep the differences between high availability, scalability, fault tolerance, and disaster recovery clear. AWS uses these terms precisely.
- If the question stresses few changes to the existing application, discard answers that require major refactoring.

## 💡 For the Exam

- Prioritize deep study of EC2, ELB, ASG, S3, IAM, VPC, RDS, DynamoDB, Lambda, SQS, Route 53, CloudFront, CloudWatch, and KMS.
- Always train with the mental question: which requirement is driving the decision?
- When in doubt between two plausible options, choose the one that best balances security, availability, and the least operational effort.
- Take practice exams only after consolidating the core blocks; practice exams without a foundation become superficial memorization.

## 📎 Related Links

- [Module questions](./questoes.md)
- [Module flashcards](./flashcards.md)
- [Module cheatsheet](./cheatsheet.md)
- [Module use cases](./casos-de-uso.md)
- [Module hands-on lab](./lab.md)
- [Official links](./links.md)
- [Module 02: IAM and Security](../02-IAM-e-Seguranca/)
- [Module 16: Well-Architected Framework](../26-Well-Architected-Framework/)
- [Module 19: Practice Exams and Questions](../29-Simulados-e-Questoes/)
