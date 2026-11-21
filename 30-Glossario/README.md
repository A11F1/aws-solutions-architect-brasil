# Module 20 — AWS SAA-C03 Glossary

## About This Module

Dictionary of essential terms and concepts for the AWS Solutions Architect Associate (SAA-C03) exam. Organized by letter to make quick review easier.

---

## A

| Term | Definition |
|---|---|
| **ACL (Access Control List)** | List of access rules. In S3: legacy per-bucket/object permission configuration. In VPC: NACL — stateless firewall at the subnet level |
| **ACM (AWS Certificate Manager)** | Service to provision, manage, and deploy free SSL/TLS certificates for use with CloudFront, ALB, API Gateway |
| **ALB (Application Load Balancer)** | Layer 7 (HTTP/HTTPS) load balancer. Supports path-based and host-based routing, WebSocket, gRPC |
| **AMI (Amazon Machine Image)** | EC2 instance template containing the OS, applications, and settings. Region-specific (can be copied across regions) |
| **ARN (Amazon Resource Name)** | Unique identifier for AWS resources. Format: `arn:aws:service:region:account-id:resource` |
| **ASG (Auto Scaling Group)** | Group of EC2 instances that scales automatically based on scaling policies or demand |
| **Athena** | Serverless SQL query service over data in S3. Pay per TB scanned |
| **Aurora** | MySQL/PostgreSQL-compatible relational database built by AWS. 5x the performance of MySQL, 3x of PostgreSQL |
| **Availability Zone (AZ)** | Isolated data center within an AWS Region. Multiple AZs per Region ensure high availability |
| **AWS Organizations** | Centralized management of multiple AWS accounts with policies (SCPs) and consolidated billing |

---

## B

| Term | Definition |
|---|---|
| **Bastion Host** | EC2 instance in a public subnet used as an SSH access point to instances in private subnets. More secure alternative: SSM Session Manager |
| **Bedrock** | AWS Generative AI service. Accesses Foundation Models (Anthropic Claude, Amazon Titan, etc.) via API without managing infrastructure |
| **Blue/Green Deployment** | Deployment strategy with two identical environments; traffic is gradually or fully shifted from "blue" (current) to "green" (new) |
| **Bucket Policy** | Resource-based JSON policy applied directly to the S3 bucket. Controls access for any principal (cross-account) |
| **Burst** | Temporary capacity beyond the provisioned limit. EC2 (T instances): CPU burst credits. Lambda: 3,000 initial concurrent executions per Region |

---

## C

| Term | Definition |
|---|---|
| **CIDR (Classless Inter-Domain Routing)** | Notation for IP ranges. E.g.: `10.0.0.0/16` = 65,536 IPs. The VPC CIDR defines the address space of the virtual network |
| **CloudFormation** | AWS IaC (Infrastructure as Code). YAML/JSON templates that define and provision AWS resources as code |
| **CloudFront** | AWS global CDN (Content Delivery Network) with 400+ edge locations. Improves latency and reduces load on the origin |
| **CloudTrail** | Auditing service that records all API calls made in the AWS account. Essential for compliance and incident investigation |
| **CloudWatch** | Monitoring and observability service. Collects metrics and logs, creates alarms and dashboards |
| **CMK (Customer Managed Key)** | KMS key created and managed by the customer. Allows full control over usage policies and rotation |
| **Cognito** | Authentication and authorization service for web/mobile applications. User Pools (auth) + Identity Pools (AWS credentials federation) |
| **Cold Start** | Delay on the first invocation of a Lambda function when a new execution environment needs to be initialized |
| **Compliance** | Conformity with regulations and standards (PCI-DSS, HIPAA, SOC2, ISO 27001). AWS is responsible for the infrastructure; the customer for the application |
| **CRR (Cross-Region Replication)** | Automatic replication of S3 objects to a bucket in another Region. Requires versioning enabled |

---

## D

| Term | Definition |
|---|---|
| **DAX (DynamoDB Accelerator)** | In-memory cache for DynamoDB with microsecond latency. Fully managed and compatible with the DynamoDB API |
| **DLQ (Dead Letter Queue)** | Queue (SQS) or topic (SNS) that receives messages that could not be processed after the maximum number of attempts |
| **DMS (Database Migration Service)** | Database migration service. Supports homogeneous and heterogeneous migrations with a CDC (Change Data Capture) option |
| **DynamoDB** | Fully managed NoSQL key-value database. Virtually unlimited scalability, millisecond latency, multi-Region with Global Tables |
| **DX (Direct Connect)** | Dedicated private network connection between on-premises and AWS. 1, 10, or 100 Gbps bandwidth with consistent latency |

---

## E

| Term | Definition |
|---|---|
| **EBS (Elastic Block Store)** | High-performance disk volumes for EC2. Types: gp2/gp3 (general-purpose SSD), io1/io2 (high-performance SSD), st1 (throughput HDD), sc1 (cold HDD) |
| **EC2 (Elastic Compute Cloud)** | AWS virtual compute service (VMs). Hundreds of instance types for different workloads |
| **ECR (Elastic Container Registry)** | Managed private Docker image registry. Integrated with ECS, EKS, and Lambda Container Image |
| **ECS (Elastic Container Service)** | AWS-managed container orchestrator. Runs tasks on EC2 or Fargate |
| **EFS (Elastic File System)** | Managed, elastic NFS file system for Linux. Multi-AZ and multi-instance simultaneously |
| **EKS (Elastic Kubernetes Service)** | AWS-managed Kubernetes. Reduces the operational complexity of the Kubernetes control plane |
| **ElastiCache** | Managed in-memory cache service. Two engines: Redis (persistence, Pub/Sub, sorted sets) and Memcached (simple, multi-threaded) |
| **Endpoint (VPC)** | Private connection between a VPC and AWS services without going out to the internet. Gateway (S3, DynamoDB — free) or Interface (PrivateLink — paid) |
| **ESM (Event Source Mapping)** | Configuration that allows Lambda to automatically receive events from Kinesis, DynamoDB Streams, SQS, MSK |
| **EventBridge** | Serverless event bus. Routes events between AWS services, SaaS, and custom applications via rules |

---

## F–G

| Term | Definition |
|---|---|
| **Fargate** | Serverless compute for containers (ECS/EKS). No servers to manage; pay for the vCPU/memory used by the task |
| **FIFO Queue** | SQS FIFO: guarantees delivery order and exactly-once delivery. Limited to 300 TPS (3,000 with batching) |
| **FSx** | Managed high-performance file systems: Lustre (HPC), Windows (SMB), NetApp ONTAP, OpenZFS |
| **Global Accelerator** | Networking service that uses the AWS private network (anycast) to route TCP/UDP traffic with the lowest global latency. 2 static anycast IPs |
| **Glue** | Serverless ETL service and data catalog. Crawlers discover schemas; Jobs process data with managed Spark |
| **GuardDuty** | Threat detection service using ML. Analyzes CloudTrail, VPC Flow Logs, DNS Logs to detect malicious behavior |

---

## H–I

| Term | Definition |
|---|---|
| **HA (High Availability)** | The system's ability to keep operating even when components fail. Usually requires resources in multiple AZs |
| **IAM (Identity and Access Management)** | Access control for AWS resources. Users, groups, roles, and policies define who can do what |
| **IOPS (I/O Operations Per Second)** | Storage performance measure. EBS gp3 baseline 3,000 IOPS; io2 Block Express up to 256,000 IOPS |
| **IRSA (IAM Roles for Service Accounts)** | EKS mechanism that associates IAM roles with Kubernetes service accounts. Pods obtain AWS credentials without hardcoded secrets |
| **Inspector** | Automated vulnerability assessment for EC2, Lambda functions, and ECR images. Uses the CVE database |

---

## J–K

| Term | Definition |
|---|---|
| **Jump Box** | See Bastion Host |
| **Kinesis Data Streams** | Real-time streaming service. Producers → Shards → Consumers. Retention up to 365 days; replay possible |
| **Kinesis Firehose** | Near-real-time data delivery to S3, Redshift, OpenSearch, Splunk. Fully managed with no consumer code |
| **KMS (Key Management Service)** | Managed encryption key service. Integrated with all AWS services that support encryption |

---

## L–M

| Term | Definition |
|---|---|
| **Lambda** | Serverless compute. Runs code in response to events without provisioning servers. Pay per invocation + duration (GB-seconds) |
| **Lake Formation** | Data lake governance service on top of S3. Manages fine-grained (column/row) permissions for Glue + Athena |
| **Macie** | Data security service that uses ML to discover and protect sensitive data in S3 (PII, keys, credentials) |
| **MGN (Application Migration Service)** | Automated lift-and-shift service. Continuously replicates servers to AWS and performs cutover with minimal downtime |
| **MSK (Managed Streaming for Kafka)** | Managed Apache Kafka on AWS. Compatible with existing Kafka clients without code changes |
| **Multi-AZ** | High availability with resources in multiple Availability Zones. For RDS: automatic synchronous failover to a standby |

---

## N–O

| Term | Definition |
|---|---|
| **NAT Gateway** | Networking component that lets instances in private subnets initiate connections to the internet without being reachable from the internet |
| **NACL (Network Access Control List)** | Stateless firewall at the subnet level. Evaluates numbered rules in order; separate inbound and outbound rules |
| **NLB (Network Load Balancer)** | Layer 4 (TCP/UDP/TLS) load balancer. Extremely high performance (millions of RPS), static IP via EIP, ultra-low latency |
| **OAC (Origin Access Control)** | Current method (replaces OAI) for CloudFront to access S3 privately. The S3 bucket denies direct access from the internet |
| **OpenSearch** | Managed fork of Elasticsearch. Search, log analytics, and visualization with Kibana/OpenSearch Dashboards |

---

## P–Q

| Term | Definition |
|---|---|
| **Parameter Store** | Hierarchical configuration and secrets storage in Systems Manager. Free for standard (up to 4 KB/parameter) |
| **Placement Group** | Grouping of EC2 instances by proximity or distribution. Cluster (low latency), Spread (maximum availability), Partition (HBase/HDFS) |
| **PrivateLink** | AWS PrivateLink: exposes private services via Interface Endpoint (ENI) with no traffic going over the internet |
| **QuickSight** | AWS serverless BI. SPICE engine for in-memory caching; embeddable dashboards; automatic ML Insights |

---

## R

| Term | Definition |
|---|---|
| **RDS (Relational Database Service)** | Managed relational databases: MySQL, PostgreSQL, Oracle, SQL Server, MariaDB, and Aurora |
| **Read Replica** | Asynchronous copy of a database to scale reads. RDS: up to 5 replicas; Aurora: up to 15 replicas |
| **Rekognition** | Computer vision as a service. Image and video analysis: objects, faces, text, content moderation |
| **Reserved Instance (RI)** | 1- or 3-year commitment with up to 72% discount vs On-Demand. Standard RI: fixed instance; Convertible RI: can change type |
| **Route 53** | AWS managed DNS with 100% SLA. Supports 8 routing policies + health checks |
| **RPO (Recovery Point Objective)** | Maximum amount of data that can be lost in a disaster event (determines backup frequency) |
| **RTO (Recovery Time Objective)** | Maximum acceptable time to restore the system after a failure (determines DR strategy) |

---

## S

| Term | Definition |
|---|---|
| **S3 (Simple Storage Service)** | Durable object storage (11 9s), effectively unlimited. Buckets per Region; objects up to 5 TB |
| **SAM (Serverless Application Model)** | Simplified IaC framework for serverless Lambda/API GW/DynamoDB applications. Extension of CloudFormation |
| **Savings Plans** | Discount of up to 66% (Compute) or 72% (EC2) in exchange for a $/hour commitment for 1 or 3 years. More flexible than Reserved Instances |
| **SCT (Schema Conversion Tool)** | Desktop tool that converts a database schema between different engines in heterogeneous migration (together with DMS) |
| **Security Group (SG)** | Stateful firewall at the instance/ENI level. Allow rules only (no explicit Deny). Can reference other SGs by ID |
| **Shield** | DDoS protection. Standard (free, automatic for everyone); Advanced (paid, extra protection + DRT + free WAF) |
| **Spot Instance** | EC2 with up to 90% discount using spare capacity. Can be interrupted with a 2-minute warning |
| **SQS (Simple Queue Service)** | Managed message queue. Standard (high throughput, at-least-once) or FIFO (guaranteed order, exactly-once) |
| **SSM (Systems Manager)** | Suite of operational tools: Session Manager, Parameter Store, Patch Manager, Run Command, Automation |
| **SRR (Same-Region Replication)** | Replication of S3 objects between buckets in the same Region (e.g., consolidating logs, compliance) |
| **Step Functions** | Serverless workflow orchestrator. Coordinates Lambdas, ECS tasks, and other services in visual state machines |

---

## T–Z

| Term | Definition |
|---|---|
| **TGW (Transit Gateway)** | Centralized network hub that connects VPCs and on-premises in a hub-and-spoke topology. Transitive routing (unlike VPC Peering) |
| **Transcribe** | Managed Speech-to-Text. Supports streaming, speaker identification, custom vocabulary |
| **Transfer Family** | Managed SFTP/FTPS/FTP/AS2 server with an S3 or EFS backend. Clients don't need to change their existing tools |
| **Trusted Advisor** | Tool that recommends improvements in cost, security, performance, fault tolerance, and service limits |
| **VPC (Virtual Private Cloud)** | Isolated virtual network on AWS. You control the CIDR, subnets, route tables, security groups, and NACLs |
| **VPC Peering** | Connection between two VPCs (same or different account/Region) over the AWS network. Non-transitive (A↔B, B↔C ≠ A↔C) |
| **WAF (Web Application Firewall)** | Layer 7 web application firewall. Protects against SQL injection, XSS, and other web attacks via web ACLs and managed rules |
| **WORM (Write Once Read Many)** | Storage model where data cannot be modified after it is written. S3 Object Lock Compliance mode |
| **X-Ray** | Distributed tracing service. Visualizes latency, errors, and performance across the services of an application |

---

## Acronym Table

| Acronym | Meaning |
|---|---|
| ACL | Access Control List |
| ACM | AWS Certificate Manager |
| ALB | Application Load Balancer |
| AMI | Amazon Machine Image |
| ARN | Amazon Resource Name |
| ASG | Auto Scaling Group |
| CDC | Change Data Capture |
| CDN | Content Delivery Network |
| CMK | Customer Managed Key |
| CRR | Cross-Region Replication |
| DAX | DynamoDB Accelerator |
| DLQ | Dead Letter Queue |
| DMS | Database Migration Service |
| DX | Direct Connect |
| EBS | Elastic Block Store |
| ECR | Elastic Container Registry |
| ECS | Elastic Container Service |
| EFS | Elastic File System |
| EKS | Elastic Kubernetes Service |
| ELB | Elastic Load Balancing |
| ENI | Elastic Network Interface |
| ESM | Event Source Mapping |
| FGA | Fine-Grained Access Control |
| FM | Foundation Model (Bedrock) |
| FSx | Amazon FSx (file systems) |
| IAM | Identity and Access Management |
| IGW | Internet Gateway |
| IOPS | Input/Output Operations Per Second |
| IRSA | IAM Roles for Service Accounts |
| IaC | Infrastructure as Code |
| KMS | Key Management Service |
| MGN | AWS Application Migration Service |
| MPP | Massively Parallel Processing |
| MSK | Managed Streaming for Kafka |
| NACL | Network Access Control List |
| NLB | Network Load Balancer |
| NLP | Natural Language Processing |
| OAC | Origin Access Control |
| OAI | Origin Access Identity (legacy) |
| PII | Personally Identifiable Information |
| RI | Reserved Instance |
| RPO | Recovery Point Objective |
| RTO | Recovery Time Objective |
| SCP | Service Control Policy |
| SCT | Schema Conversion Tool |
| SG | Security Group |
| SNS | Simple Notification Service |
| SQS | Simple Queue Service |
| SRR | Same-Region Replication |
| SSE | Server-Side Encryption |
| SSM | AWS Systems Manager |
| TGW | Transit Gateway |
| TTS | Text-to-Speech |
| VGW | Virtual Private Gateway |
| VPC | Virtual Private Cloud |
| WAF | Web Application Firewall |
| WAT | Well-Architected Tool |
| WORM | Write Once Read Many |
