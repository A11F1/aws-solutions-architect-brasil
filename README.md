# 📚 AWS Certified Solutions Architect – Associate (SAA-C03)

## 🚦 Where to start

- **Beginner track (from scratch):** follow [Module 01](./01-Introducao-SAA-C03/README.md) → [Module 02](./02-IAM-e-Seguranca/README.md) → [Module 04](./04-Computacao-EC2/README.md) → [Module 05](./05-Alta-Disponibilidade-e-Escalabilidade/README.md) → [Module 06](./06-Amazon-S3-e-Armazenamento/README.md) → [Module 08](./08-VPC-e-Redes/README.md) → [Module 10](./10-Banco-de-Dados/README.md) and complete them in order through [Module 31](./31-Recursos-e-Links/README.md), including the corresponding labs each week.
- **Quick review:** review using [cheatsheets](./01-Introducao-SAA-C03/cheatsheet.md), [flashcards](./01-Introducao-SAA-C03/flashcards.md), and [questions](./29-Simulados-e-Questoes/README.md), prioritizing the modules with the greatest weight on the exam (security, resilience, performance, and cost).
- **Final stretch / last 14 days:** focus on [05](./05-Alta-Disponibilidade-e-Escalabilidade/README.md), [06](./06-Amazon-S3-e-Armazenamento/README.md), [08](./08-VPC-e-Redes/README.md), [10](./10-Banco-de-Dados/README.md), [13](./13-DNS-Route53-e-CloudFront/README.md), [14](./14-Desacoplamento-SQS-SNS-EventBridge/README.md), [17](./17-Serverless-Lambda-API-Gateway/README.md), [22](./22-Recuperacao-de-Desastres-e-Continuidade/README.md), [26](./26-Well-Architected-Framework/README.md) and finish with the [practice exams](./29-Simulados-e-Questoes/README.md).

> A complete study repository in English for anyone who wants to pass the exam with a real understanding of AWS architecture.

Published portal: https://thiago-code-lab.github.io/aws-solutions-architect-brasil/

## 🎯 About this Repository

This repository stands out by combining content in English with a focus on real architectural reasoning (trade-offs, use cases, and exam pitfalls), curated low-cost labs, a browsable portal on GitHub Pages, and practice questions in the style of the SAA-C03 exam.

[![GitHub Pages](https://img.shields.io/website?url=https%3A%2F%2Fthiago-code-lab.github.io%2Faws-solutions-architect-brasil%2F&label=GitHub%20Pages)](https://thiago-code-lab.github.io/aws-solutions-architect-brasil/) [![Deploy GitHub Pages](https://github.com/thiago-code-lab/aws-solutions-architect-brasil/actions/workflows/deploy.yml/badge.svg)](https://github.com/thiago-code-lab/aws-solutions-architect-brasil/actions/workflows/deploy.yml) [![License](https://img.shields.io/github/license/thiago-code-lab/aws-solutions-architect-brasil)](./LICENSE) [![GitHub stars](https://img.shields.io/github/stars/thiago-code-lab/aws-solutions-architect-brasil?style=social)](https://github.com/thiago-code-lab/aws-solutions-architect-brasil/stargazers)

This material was structured for an intermediate-level student who already uses AWS day to day and already masters cloud fundamentals. The focus is not blind memorization: the goal is to develop the reasoning expected of a solutions architect, understanding the trade-offs between resilience, performance, security, and cost.

The repository was designed to work well on GitHub and GitHub Pages. Each module contains theory, exam-style questions, flashcards, a cheatsheet, architectural scenarios, labs, and a curated set of official links. The order of the modules follows the most heavily tested exam topics and a progression of complexity that makes sense for a 9-week review.

## 📊 Exam Structure and Domains

| Domain | Weight | What shows up in practice | Main modules |
|---|---:|---|---|
| Design Resilient Architectures | 30% | High availability, disaster recovery, decoupling, multi-AZ and multi-region | 05, 06, 08, 09, 13, 14, 21, 22, 24, 26 |
| Design High-Performing Architectures | 28% | Compute selection, caching, storage performance, networking, and databases | 04, 06, 08, 10, 11, 12, 13, 16, 18 |
| Design Secure Applications and Architectures | 24% | IAM, KMS, network isolation, encryption, audit trails, and governance | 02, 03, 08, 09, 14, 17, 20, 23, 24, 25 |
| Design Cost-Optimized Architectures | 18% | Rightsizing, storage classes, purchase models, serverless, and automation | 04, 06, 15, 17, 21, 23 |

## 🗺️ Study Map

### Week 1
- Module 01: exam overview, strategy, weights, and most-tested services
- Module 02: IAM, KMS, Organizations, Identity Center, and policies
- Module 03: guided IAM and security lab

### Week 2
- Module 04: EC2, EBS, placement groups, and purchase models
- Module 05: ELB, ASG, health checks, and scalability

### Week 3
- Module 06: S3, lifecycle, replication, classes, and security
- Module 07: advanced S3 lab
- Module 08: VPC, subnets, endpoints, routes, and network security
- Module 09: VPC and networking lab

### Week 4
- Module 10: databases for architecture (RDS, Aurora, DynamoDB, ElastiCache, Redshift)
- Module 11: RDS and relational databases lab
- Module 12: DynamoDB modeling and patterns
- Module 13: Route 53, CloudFront, OAC, WAF, and Global Accelerator

### Week 5
- Module 14: SQS, SNS, EventBridge, and Step Functions
- Module 15: SQS/SNS messaging lab
- Module 16: ECS, EKS, Fargate, ECR, and App Runner

### Week 6
- Module 17: Lambda, API Gateway, authorizers, and serverless observability
- Module 18: Kinesis, Firehose, Glue, Athena, EMR, and analytics

### Week 7
- Module 19: most-tested managed AI and ML services
- Module 20: CloudWatch, CloudTrail, Config, and X-Ray
- Module 21: DMS, MGN, DataSync, Snow Family, and Transfer Family

### Week 8
- Module 22: disaster recovery, continuity, RTO/RPO, and DR strategies
- Module 23: AWS Organizations, multi-account governance, and cost optimization
- Module 24: advanced networking and hybrid connectivity (TGW, DX, VPN, PrivateLink)
- Module 25: encryption with KMS and secrets management with Secrets Manager/SSM

### Week 9
- Module 26: Well-Architected Framework and DR patterns
- Module 27: real-world case studies
- Module 28: end-to-end labs
- Module 29: full practice exams
- Modules 30 and 31: glossary and review of official links

## 📁 Modules

| # | Module | Predominant domain | Status | Link |
|---|---|---|---|---|
| 01 | SAA-C03 Introduction | Foundation / Strategy | ✅ Complete | [README](./01-Introducao-SAA-C03/README.md) |
| 02 | IAM and Security | Security | ✅ Complete | [README](./02-IAM-e-Seguranca/README.md) |
| 03 | IAM and Security (Labs) | Security (hands-on) | ✅ Complete | [Lab](./03-IAM-e-Seguranca-Labs/lab.md) |
| 04 | EC2 Compute | High performance | ✅ Complete | [README](./04-Computacao-EC2/README.md) |
| 05 | High Availability and Scalability | Resilience | ✅ Complete | [README](./05-Alta-Disponibilidade-e-Escalabilidade/README.md) |
| 06 | Amazon S3 and Storage | Resilience / Cost | ✅ Complete | [README](./06-Amazon-S3-e-Armazenamento/README.md) |
| 07 | Advanced S3 (Labs) | Resilience (hands-on) | ✅ Complete | [Lab](./07-S3-Avancado-Labs/lab.md) |
| 08 | VPC and Networking | Security / Resilience | ✅ Complete | [README](./08-VPC-e-Redes/README.md) |
| 09 | VPC and Networking (Labs) | Security (hands-on) | ✅ Complete | [Lab](./09-VPC-e-Redes-Labs/lab.md) |
| 10 | Databases | High performance | ✅ Complete | [README](./10-Banco-de-Dados/README.md) |
| 11 | RDS and Relational Databases (Labs) | High performance (hands-on) | ✅ Complete | [Lab](./11-RDS-e-Bancos-Relacionais-Labs/lab.md) |
| 12 | DynamoDB (Labs) | High performance (hands-on) | ✅ Complete | [Lab](./12-DynamoDB/lab.md) |
| 13 | DNS, Route 53, and CloudFront | Resilience / Performance | ✅ Complete | [README](./13-DNS-Route53-e-CloudFront/README.md) |
| 14 | Decoupling: SQS, SNS, EventBridge | Resilience | ✅ Complete | [README](./14-Desacoplamento-SQS-SNS-EventBridge/README.md) |
| 15 | SQS and SNS (Labs) | Resilience (hands-on) | ✅ Complete | [Lab](./15-SQS-SNS-Mensageria-Labs/lab.md) |
| 16 | Containers: ECS, EKS, Fargate | High performance | ✅ Complete | [README](./16-Containers-ECS-EKS-Fargate/README.md) |
| 17 | Serverless: Lambda and API Gateway | Cost / Security | ✅ Complete | [README](./17-Serverless-Lambda-API-Gateway/README.md) |
| 18 | Data and Analytics | High performance | ✅ Complete | [README](./18-Dados-e-Analytics/README.md) |
| 19 | Machine Learning and AI | Managed services | ✅ Complete | [README](./19-Machine-Learning-e-IA/README.md) |
| 20 | Monitoring: CloudWatch and CloudTrail | Security / Operations | ✅ Complete | [README](./20-Monitoramento-CloudWatch-CloudTrail/README.md) |
| 21 | Migration and Transfer | Resilience / Cost | ✅ Complete | [README](./21-Migracao-e-Transferencia/README.md) |
| 22 | Disaster Recovery and Continuity | Resilience / Security | ✅ Complete | [README](./22-Recuperacao-de-Desastres-e-Continuidade/README.md) |
| 23 | AWS Organizations, Governance, and Costs | Security / Cost | ✅ Complete | [README](./23-AWS-Organizations-Governanca-e-Custos/README.md) |
| 24 | Advanced Networking and Hybrid Connectivity | Security / Performance | ✅ Complete | [README](./24-Redes-Avancadas-e-Conectividade-Hibrida/README.md) |
| 25 | Encryption, KMS, and Secrets Management | Security | ✅ Complete | [README](./25-Criptografia-KMS-e-Gestao-de-Segredos/README.md) |
| 26 | Well-Architected Framework | All domains | ✅ Complete | [README](./26-Well-Architected-Framework/README.md) |
| 27 | Real-World Use Cases | All domains | ✅ Complete | [README](./27-Casos-de-Uso-Reais/README.md) |
| 28 | Hands-On Labs | Practice | ✅ Complete | [README](./28-Labs-Praticos/README.md) |
| 29 | Practice Exams and Questions | All domains | ✅ Complete | [README](./29-Simulados-e-Questoes/README.md) |
| 30 | Glossary | Review | ✅ Complete | [README](./30-Glossario/README.md) |
| 31 | Resources and Links | Review | ✅ Complete | [README](./31-Recursos-e-Links/README.md) |

## 🧩 Module overview

| Module name | Week / position | Theory (Yes/No) | Lab (Yes/No) | Cheatsheet (Yes/No) | Practice questions (Yes/No) |
|---|---|---|---|---|---|
| [01 - SAA-C03 Introduction](./01-Introducao-SAA-C03/README.md) | Week 1 | Yes | Yes | Yes | Yes |
| [02 - IAM and Security](./02-IAM-e-Seguranca/README.md) | Week 1 | Yes | Yes | Yes | Yes |
| [03 - IAM and Security (Labs)](./03-IAM-e-Seguranca-Labs/lab.md) | Week 1 | No | Yes | No | No |
| [04 - EC2 Compute](./04-Computacao-EC2/README.md) | Week 2 | Yes | Yes | Yes | Yes |
| [05 - High Availability and Scalability](./05-Alta-Disponibilidade-e-Escalabilidade/README.md) | Week 2 | Yes | Yes | Yes | Yes |
| [06 - Amazon S3 and Storage](./06-Amazon-S3-e-Armazenamento/README.md) | Week 3 | Yes | No | Yes | Yes |
| [07 - Advanced S3 (Labs)](./07-S3-Avancado-Labs/lab.md) | Week 3 | No | Yes | No | No |
| [08 - VPC and Networking](./08-VPC-e-Redes/README.md) | Week 3 | Yes | No | Yes | Yes |
| [09 - VPC and Networking (Labs)](./09-VPC-e-Redes-Labs/lab.md) | Week 3 | No | Yes | No | No |
| [10 - Databases](./10-Banco-de-Dados/README.md) | Week 4 | Yes | No | Yes | Yes |
| [11 - RDS and Relational Databases (Labs)](./11-RDS-e-Bancos-Relacionais-Labs/lab.md) | Week 4 | No | Yes | No | No |
| [12 - DynamoDB (Labs)](./12-DynamoDB/lab.md) | Week 4 | No | Yes | No | No |
| [13 - DNS, Route 53, and CloudFront](./13-DNS-Route53-e-CloudFront/README.md) | Week 4 | Yes | No | Yes | Yes |
| [14 - Decoupling: SQS, SNS, EventBridge](./14-Desacoplamento-SQS-SNS-EventBridge/README.md) | Week 5 | Yes | No | Yes | Yes |
| [15 - SQS and SNS (Labs)](./15-SQS-SNS-Mensageria-Labs/lab.md) | Week 5 | No | Yes | No | No |
| [16 - Containers: ECS, EKS, Fargate](./16-Containers-ECS-EKS-Fargate/README.md) | Week 5 | Yes | No | Yes | Yes |
| [17 - Serverless: Lambda and API Gateway](./17-Serverless-Lambda-API-Gateway/README.md) | Week 6 | Yes | Yes | Yes | Yes |
| [18 - Data and Analytics](./18-Dados-e-Analytics/README.md) | Week 6 | Yes | Yes | Yes | Yes |
| [19 - Machine Learning and AI](./19-Machine-Learning-e-IA/README.md) | Week 7 | Yes | Yes | Yes | Yes |
| [20 - Monitoring: CloudWatch and CloudTrail](./20-Monitoramento-CloudWatch-CloudTrail/README.md) | Week 7 | Yes | Yes | Yes | Yes |
| [21 - Migration and Transfer](./21-Migracao-e-Transferencia/README.md) | Week 7 | Yes | Yes | Yes | Yes |
| [22 - Disaster Recovery and Continuity](./22-Recuperacao-de-Desastres-e-Continuidade/README.md) | Week 8 | Yes | Yes | Yes | Yes |
| [23 - AWS Organizations, Governance, and Costs](./23-AWS-Organizations-Governanca-e-Custos/README.md) | Week 8 | Yes | Yes | Yes | Yes |
| [24 - Advanced Networking and Hybrid Connectivity](./24-Redes-Avancadas-e-Conectividade-Hibrida/README.md) | Week 8 | Yes | Yes | Yes | Yes |
| [25 - Encryption, KMS, and Secrets Management](./25-Criptografia-KMS-e-Gestao-de-Segredos/README.md) | Week 8 | Yes | Yes | Yes | Yes |
| [26 - Well-Architected Framework](./26-Well-Architected-Framework/README.md) | Week 9 | Yes | Yes | Yes | Yes |
| [27 - Real-World Use Cases](./27-Casos-de-Uso-Reais/README.md) | Week 9 | Yes | Yes | Yes | Yes |
| [28 - Hands-On Labs](./28-Labs-Praticos/README.md) | Week 9 | Yes | Yes | Yes | Yes |
| [29 - Practice Exams and Questions](./29-Simulados-e-Questoes/README.md) | Week 9 | Yes | Yes | Yes | Yes |
| [30 - Glossary](./30-Glossario/README.md) | Week 9 | Yes | Yes | Yes | Yes |
| [31 - Resources and Links](./31-Recursos-e-Links/README.md) | Week 9 | Yes | Yes | Yes | Yes |

## 🚀 How to Use This Repository

1. Start with module 01 to align expectations, weights, and strategy.
2. Study modules 02 to 25 in order and use the lab modules to consolidate the critical topics.
3. At the end of each module, solve the questions before opening the answer key.
4. Use the flashcards for spaced review and the cheatsheet for quick pre-exam review.
5. Run the labs in us-east-1 whenever possible to stay consistent with the material.
6. Finish your preparation with the case studies, end-to-end labs, and the three full practice exams.

## ⚠️ Common exam pitfalls

| Frequent confusion | When to use each option | Typical exam pitfall |
|---|---|---|
| SNS vs SQS | SNS for pub/sub fanout; SQS for a decoupled queue with asynchronous processing and consumption control. | Confusing broadcast to multiple consumers with a point-to-point queue. |
| SQS Standard vs FIFO | Standard for high throughput and tolerance for duplicates/eventual ordering; FIFO for strict ordering and deduplication. | Choosing FIFO unnecessarily and losing scale, or choosing Standard when ordering is mandatory. |
| RDS Multi-AZ vs Read Replicas | Multi-AZ for high availability and failover; Read Replicas to scale reads and reporting. | Assuming Multi-AZ improves reads or that a replica replaces synchronous HA. |
| NAT Gateway vs VPC Endpoint | NAT Gateway for general private outbound access to the internet; VPC Endpoint for private access to AWS services without the internet. | Using NAT for S3/DynamoDB and paying more, even with an endpoint available. |
| ALB vs NLB vs CLB | ALB for HTTP/HTTPS layer 7; NLB for TCP/UDP/TLS layer 4 and low latency; CLB for legacy. | Ignoring the protocol requirement and sticky/session routing. |
| S3 Standard vs IA vs One Zone-IA vs Glacier | Standard for frequent access; IA for infrequent access; One Zone-IA for reproducible data; Glacier for archiving. | Choosing a cheap class without considering retrieval latency and minimum retention. |
| EBS vs EFS vs FSx | EBS for block storage on EC2; EFS for multi-AZ Linux NFS sharing; FSx for specialized file systems (Windows, Lustre, ONTAP, OpenZFS). | Treating EBS as shared by multiple instances without a specific architecture. |
| ASG vs scheduled scaling vs predictive scaling | ASG for automatic elasticity based on metrics; scheduled for predictable events; predictive for historical patterns. | Using only scheduled scaling for unpredictable load and losing resilience during spikes. |
| API Gateway vs ALB for Lambda | API Gateway for managed APIs with auth, throttling, and versioning; ALB for simple HTTP routing with Lambda integration. | Choosing ALB when the question asks for advanced API management features. |
| KMS CMK vs Secrets Manager vs SSM Parameter Store | KMS for encryption/keys; Secrets Manager for rotation and sensitive secrets; Parameter Store for parameters and simple secrets. | Thinking KMS alone solves the secret lifecycle. |

## 🤝 How to contribute

See [CONTRIBUTING.md](./CONTRIBUTING.md) for how to open issues, propose labs/questions, and submit pull requests in a standardized and objective way.

Contributions are welcome, especially to:

- fix technical details and service limits
- improve question scenarios and explanations of the options
- update recent changes to AWS services
- add clearer ASCII diagrams and more cost-effective labs

When contributing, prefer small, technically justified changes with official AWS links when the subject involves specific service behavior.

## 📦 Releases

For relevant content updates, see the Releases page on GitHub and the history in [CHANGELOG.md](./CHANGELOG.md).

## 📄 License

MIT

## ✅ Content quality and last review

- The content is based on official AWS documentation and relevant whitepapers.
- The examples and labs prioritize low-cost choices for hands-on study.
- The repository is updated periodically as AWS services and the exam blueprint change.

**Last global review:** April 8, 2026 (update this date whenever a broad review of the repository is completed).
