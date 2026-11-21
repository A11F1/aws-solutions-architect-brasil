# Flashcards — Module 19: Practice Exams and Questions

> **Format:** Q = Question | A = Answer  
> **Total:** 25 flashcards — exam strategy and pattern review

---

**Q:** What are the 4 SAA-C03 domains and their weights?  
**A:** **(1) Design Resilient Architectures: 26%** — HA, DR, decoupling. **(2) Design High-Performing Architectures: 24%** — scaling, caching, databases. **(3) Design Secure Applications and Architectures: 30%** — IAM, encryption, network security. **(4) Design Cost-Optimized Architectures: 20%** — EC2 pricing, storage tiers, serverless. Security is the largest domain.

---

**Q:** How do you identify on the exam when to use Lambda vs ECS vs EC2?  
**A:** **Lambda:** stateless, <15 min, responds to events, automatic scaling. **ECS/Fargate:** containerized, >15 min possible, microservices, needs Docker. **EC2:** full OS control, specific software (license, GPU), continuous load, SSH access, OS-level compliance. Clues: "without managing servers" → Lambda/Fargate; "lift-and-shift" → EC2.

---

**Q:** How do you identify when to use RDS vs DynamoDB on the exam?  
**A:** **RDS:** relational data, ACID transactions, complex SQL with JOINs, fixed schema, existing application that uses SQL. **DynamoDB:** high scale (millions of req/s), flexible schema, millisecond latency, serverless, key-value/document model, avoid JOINs. Exam clue: "relational" → RDS; "milliseconds at any scale" / "key-value" → DynamoDB.

---

**Q:** When should you use SQS vs SNS vs EventBridge on the exam?  
**A:** **SQS:** point-to-point queue, decoupling, durable asynchronous processing, throughput control. **SNS:** publish to multiple subscribers simultaneously (fan-out), push notifications (email, SMS, Lambda). **EventBridge:** routing based on event patterns, SaaS integration, scheduler, audit trail of events. SQS+SNS = fan-out with buffering.

---

**Q:** What clue in the question text indicates using ElastiCache?  
**A:** Keywords: "slow database queries", "repeated identical queries", "session data", "leaderboard", "real-time", "sub-millisecond latency", "hot data". Redis: when you need data structures (sets, sorted sets for leaderboards), pub/sub, optional persistence. Memcached: simple multi-threaded cache, no persistence needed.

---

**Q:** When should you use Route 53 Failover vs Latency vs Weighted Routing?  
**A:** **Failover:** active/passive DR — traffic goes to the primary, automatic failover to the secondary if the health check fails. **Latency:** routes users to the lowest-latency region (global performance). **Weighted:** A/B testing, gradual traffic distribution (canary). **Geolocation:** compliance — EU users go only to the EU region. **Multi-Value:** multiple healthy records returned (not a real load balancer).

---

**Q:** How do you interpret "lowest cost" with S3 storage on the exam?  
**A:** Think about the access pattern: **Standard** = frequent (any frequency). **Standard-IA** = infrequent access but fast retrieval when needed. **Glacier Instant** = occasional access (milliseconds). **Glacier Flexible** = rare access (<1/month), 1-12h retrieval. **Deep Archive** = less than annual access, 12-48h retrieval. Lower cost to store, higher cost to retrieve.

---

**Q:** What should you do when the question asks for the "most secure" way to access S3?  
**A:** S3 security hierarchy: (1) **Block Public Access** (account/bucket level). (2) **Bucket Policy** with granular Deny or Allow. (3) **IAM Policy** for users/roles. (4) **Pre-signed URLs** for temporary access. (5) **OAC** (Origin Access Control) for CloudFront. (6) **Object Lock** for WORM. For "most secure": Block Public Access + OAC for CloudFront, no ACLs.

---

**Q:** Why is VPC Peering not always the best solution for connecting VPCs?  
**A:** **VPC Peering limitations:** (1) Not transitive — A↔B and B↔C does not imply A↔C (you need an A↔C peering too). (2) CIDR ranges cannot overlap. (3) For N VPCs: N*(N-1)/2 peerings (scales poorly). **Alternative for hub-and-spoke:** AWS Transit Gateway — connects dozens/hundreds of VPCs and VPNs through a central hub. The exam tests: many VPCs = Transit Gateway; few VPCs = peering.

---

**Q:** How do you identify when to use CloudFront vs ALB on the exam?  
**A:** **CloudFront:** global content (assets, global API), static/dynamic content caching, DDoS protection (Shield Standard free), S3 origin, Lambda@Edge. **ALB:** regional load balancing within a VPC, path/host routing, target groups (EC2, Lambda, IP), WebSockets, regional WAF integration. Clue: "global" + "cache" → CloudFront; "regional" + "routing" → ALB.

---

**Q:** When is a SG (Security Group) more suitable than a NACL?  
**A:** **SG is better when:** instance-level control, stateful (automatic return traffic), accepts references to other SGs (e.g., "allow from the ALB's SG"). **NACL is better when:** you need explicit DENY rules (e.g., block a specific IP), subnet-level security, compliance requiring an ACL. Exam: "block malicious IP" → NACL Deny rule (SG has no Deny).

---

**Q:** What does "decoupled" mean in architecture in the context of the exam?  
**A:** Components that operate independently without direct synchronous calls. If one component fails or slows down, it does not cascade to others. Implementation: **SQS** (asynchronous buffering), **SNS** (pub/sub), **EventBridge** (event-driven). Exam clue: "tightly coupled" → solution with SQS/SNS for decoupling. "Independently scalable" → decoupled.

---

**Q:** What is the difference between a KMS CMK and a KMS AWS Managed Key?  
**A:** **AWS Managed Key** (e.g., `aws/s3`): automatic annual rotation, each service has its own, no additional cost, no policy control. **Customer Managed Key (CMK):** you control the key policy, auditable in CloudTrail, can be disabled, cost (~$1/month), configurable rotation. Exam: "audit key usage" or "control who uses the key" → CMK. Simple encryption without control → AWS Managed Key.

---

**Q:** When should you use Secrets Manager vs SSM Parameter Store?  
**A:** **Secrets Manager:** native automatic rotation (Lambda, RDS, Redshift, DocumentDB), versioning, direct integration with RDS. More expensive (~$0.40/secret/month). **Parameter Store Standard:** free, no native automatic rotation, simple configuration (non-secret). **Parameter Store Advanced:** supports Parameter Policies (TTL, alerts). If the question mentions "automatic rotation" → Secrets Manager.

---

**Q:** What should you identify first when reading an exam question?  
**A:** Reading sequence: **(1)** Last sentence — it usually specifies the real requirement ("what is the most cost-effective solution?"). **(2)** Explicit constraints — "without managing servers", "least code change", "RPO < 1 hour". **(3)** Context — data type, volume, access frequency. **(4)** Elimination — which option DIRECTLY violates a requirement?

---

**Q:** How does the exam test Multi-AZ vs Multi-Region knowledge?  
**A:** **Multi-AZ:** protection against AZ failure (availability), same region, low latency, automatic failover (RDS Multi-AZ, ALB, Aurora). **Multi-Region:** protection against region failure (DR), higher latency, usually latency routing or failover with Route 53. Exam: "datacenter failure" → Multi-AZ. "region failure" / "disaster recovery" → Multi-Region.

---

**Q:** What is the difference between Active-Active and Active-Passive multi-region?  
**A:** **Active-Active:** both regions receive traffic simultaneously (Route 53 Latency/Weighted). No downtime on failover. Aurora Global Database (writes in the primary, reads in any). **Active-Passive:** only the primary region receives traffic. Manual or automatic failover to the secondary if the primary fails (Route 53 Failover). Active-Active = lower RTO, higher cost.

---

**Q:** Which clue leads you to choose Aurora over RDS MySQL?  
**A:** Clues for Aurora: "high-performance reads" (up to 15 Read Replicas, vs 5 for RDS), "automatic scaling" (Aurora Serverless), "failover <30s" (Aurora keeps data in a distributed cluster volume, vs Multi-AZ ~1-2 min), "multi-region" (Aurora Global Database), "zero-downtime patching". Cost: Aurora ~20% more expensive than RDS MySQL, but equivalent with more features.

---

**Q:** How do you recognize a Lambda cold start problem on the exam?  
**A:** Clues: "inconsistent latency", "slow first request", "sporadic latency spikes", "Java + Lambda". Exam solution: **Provisioned Concurrency** (eliminates cold start, pre-warmed instances) or **SnapStart** (Java 11+). NOT the solution: increasing memory (reduces duration but does not eliminate cold start), Arm64/Graviton (better cost/performance but does not fix cold start).

---

**Q:** What are the most tested IAM conditions on the exam?  
**A:** **aws:RequestedRegion** — restricts the region. **aws:PrincipalTag** — based on the user/role tag. **s3:prefix** — S3 prefix. **aws:SourceVpc/VPCe** — restricts access to resources within a VPC/VPC Endpoint. **aws:MultiFactorAuthPresent** — requires MFA. **StringLike/StringEquals** — comparison with wildcards. The exam tests: "only from the VPC" → `aws:SourceVpc` condition.

---

**Q:** What is an SCP (Service Control Policy) and when should you use it?  
**A:** A policy in AWS Organizations that defines the **maximum permission limit** for member accounts — even if an account has IAM Admin, an SCP can block actions (e.g., prohibit actions outside us-east-1). An SCP does not grant permissions — it only restricts. Use cases: multi-account compliance (prohibit unapproved regions, prohibit deleting CloudTrail). SCP + account IAM = effective permissions.

---

**Q:** How do you eliminate wrong options in exam security questions?  
**A:** Automatically eliminate: (1) **Hardcoded access keys** in code. (2) **Credentials in environment variables** when IAM Roles are possible. (3) **Unnecessarily open 0.0.0.0/0** ports. (4) S3 **Public Read ACL** when there is an alternative. (5) Sensitive data in **plaintext**. AWS security principle: assume role > access keys; least privilege; encrypt by default.

---

**Q:** What does the exam test about EBS vs EFS vs S3?  
**A:** **EBS:** block storage attached to one EC2, single-AZ (except io2 Multi-Attach), high IOPS, databases, OS. **EFS:** NFS file system shared across multiple EC2s, multi-AZ, scales automatically. **S3:** object storage, access via HTTP API, not mountable as a filesystem (EFS/EBS are), practically unlimited, ideal for unstructured data, backup, data lake.

---

**Q:** When should you use AWS Batch vs Lambda for batch processing?  
**A:** **Lambda:** short jobs (<15 min), event-driven, small. **AWS Batch:** long jobs (hours/days), heavy compute (HPC, ML training, rendering), needs GPU, large volume of parallel jobs. Manages: dynamically provisions EC2/Fargate, job queue, priorities. Ideal for workloads that exceed Lambda's limits.

---

**Q:** How does the exam test the principle of least privilege?  
**A:** Scenarios: **(1)** EC2 only needs to read one S3 bucket → IAM Role with `s3:GetObject` on that specific bucket (not `s3:*` or AdministratorAccess). **(2)** Lambda needs to write to DynamoDB → `dynamodb:PutItem` on the specific table (not full access). **(3)** Cross-account: trust policy + permission policy combined. Always: **the minimum necessary for the minimum time necessary**.
