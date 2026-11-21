# Flashcards — Module 16: Well-Architected Framework

> **Format:** Q = Question | A = Answer  
> **Total:** 25 flashcards

---

**Q:** What are the 6 pillars of the AWS Well-Architected Framework?  
**A:** **(1) Operational Excellence** — run and monitor systems. **(2) Security** — protect data and systems. **(3) Reliability** — recover from failures, scale. **(4) Performance Efficiency** — use resources efficiently. **(5) Cost Optimization** — avoid unnecessary costs. **(6) Sustainability** — minimize environmental impact.

---

**Q:** What are the 4 Disaster Recovery strategies in order from lowest to highest cost/capability?  
**A:** **(1) Backup & Restore** (RPO/RTO: hours) → **(2) Pilot Light** (minimal core always on, RPO/RTO: tens of min) → **(3) Warm Standby** (scaled-down version always active, RTO: minutes) → **(4) Active-Active Multi-Site** (traffic in multiple Regions simultaneously, RTO: seconds/zero).

---

**Q:** What are RPO vs RTO and how do they affect the choice of DR strategy?  
**A:** **RPO** (Recovery Point Objective): how much data I can lose. **RTO** (Recovery Time Objective): how long I can be unavailable. Lower RPO/RTO = more expensive strategy: Backup&Restore (RPO/RTO hours) → Pilot Light → Warm Standby → Active-Active (RPO/RTO ≈ 0, most expensive).

---

**Q:** What is the Operational Excellence pillar and its design principles?  
**A:** Focus on continuously operating and improving processes. Principles: **Perform operations as code** (IaC), **Annotate documentation** (auto-generated), **Make frequent, small, reversible changes** (no big-bang deployments), **Refine operations procedures frequently** (Game Days), **Anticipate failure** (pre-mortem), **Learn from failures** (blameless post-mortems).

---

**Q:** What is the Security pillar and its "base of identity" principle?  
**A:** **Identity is the foundation of security.** Everything starts with "who can do what". Principles: Strong identity foundation (IAM, Organizations SCP), Enable traceability (CloudTrail, CloudWatch), Apply security at all layers (SG, WAF, KMS, encryption), Protect data in transit and at rest, Keep people away from data, Prepare for security events.

---

**Q:** What is the Reliability pillar and how does Auto Scaling contribute?  
**A:** Reliability = ability to recover from failures and grow. Auto Scaling contributes by: scaling horizontally (more instances, not bigger ones), automatically recovering from failures (replacing unhealthy instances), distributing across multiple AZs. Principles: Test recovery procedures, Stop guessing capacity, Manage change with automation.

---

**Q:** What is the Performance Efficiency pillar?  
**A:** Using computing resources efficiently. Principles: **Democratize advanced technologies** (use managed services instead of building), **Go global in minutes** (Regions, CloudFront), **Use serverless architectures** (no server management), **Experiment more often** (easy to test instance types), **Consider mechanical sympathy** (use the right service for the workload).

---

**Q:** What are the principles of the Cost Optimization pillar?  
**A:** Avoid unnecessary spending. Principles: **Implement cloud financial management** (organizational process), **Adopt a consumption model** (pay for what you use, shut down idle), **Measure overall efficiency** (output per cost), **Stop spending on undifferentiated heavy lifting** (use managed services), **Analyze and attribute expenditure** (tags, cost allocation).

---

**Q:** What is the Sustainability pillar and how does it reduce environmental impact?  
**A:** Minimize the environmental impact of workloads. Practices: use Regions with a lower carbon footprint, optimize utilization (no idle instances), use managed services (AWS optimizes shared hardware), serverless (on-demand resources), scaling down during low-usage periods, modernize hardware via new services (Graviton — better energy efficiency).

---

**Q:** What is the AWS Well-Architected Tool (WAT)?  
**A:** A tool in the AWS console for reviewing workloads against the 6 pillars. Process: create review → answer questions per pillar → receive "High Risk Issues" (HRI) and "Medium Risk Issues" (MRI) with recommendations. Creates an improvement plan. Free. Can be performed by AWS partners via the Well-Architected Partner Program.

---

**Q:** What does "Pilot Light" mean in a DR strategy?  
**A:** Keep the **minimal core** of the application always running in the DR Region. The core: database (replicated) and basic configurations. In case of disaster: "light the pilot" = scale up the remaining components (EC2, ELB) in the DR Region. RTO: minutes to tens of minutes. Lower cost than Warm Standby (only the essentials run continuously).

---

**Q:** What is "Warm Standby" and how does it differ from "Pilot Light"?  
**A:** **Warm Standby:** a complete (but scaled-down/under-capacity) version of the application running in the DR Region. **Pilot Light:** only the minimal core. Difference: Warm Standby has all components ready (small EC2, RDS replica) and only scales up on failover. Pilot Light needs to provision EC2 etc. on failover. Warm Standby = lower RTO, higher cost.

---

**Q:** What is the availability table for 99%, 99.9%, 99.99%, 99.999%?  
**A:** **99%** = ~87.6h downtime/year (~7.3h/month). **99.9%** = ~8.7h/year (~43.8 min/month). **99.99%** = ~52 min/year (~4.4 min/month). **99.999% (Five Nines)** = ~5.26 min/year (~26 sec/month). Five Nines requires an active-active architecture with zero tolerance for a single failure.

---

**Q:** How do you compare On-Demand vs Reserved vs Savings Plans in the context of Cost Optimization?  
**A:** **On-Demand:** maximum flexibility, highest price. Use for: unpredictable workloads, spikes. **Reserved (1 or 3 years):** up to 72% discount. Use for: steady, predictable 24/7 workloads. **Savings Plans:** commitment of $ per hour (Compute or EC2/Fargate/Lambda). More flexible than Reserved. **Spot:** up to 90% off, interruptible. Use for: batch, fault-tolerant.

---

**Q:** What is a Well-Architected Framework Lens?  
**A:** An extension of the WAF for specific domains. Available: **Serverless Lens**, **SaaS Lens**, **Machine Learning Lens**, **Data Analytics Lens**, **Games Industry Lens**, **SAP Lens**, **High Performance Computing Lens**, etc. Each Lens has specific questions and best practices for that domain in addition to the 6 base pillars.

---

**Q:** Which Well-Architected principle says "design for failure"?  
**A:** The **Reliability** pillar: "Test recovery procedures" and "Design for failure". AWS: design assuming components WILL fail. Everything should have redundancy. Avoid SPOFs (Single Points of Failure). Multi-AZ, Auto Scaling, circuit breakers, dead letter queues — a resilient architecture treats failures as expected, not exceptional.

---

**Q:** How does the "Infrastructure as Code" principle fit into Well-Architected?  
**A:** The **Operational Excellence** pillar: "Perform operations as code". IaC (CloudFormation, Terraform, CDK): (1) reproducible — same state every time; (2) auditable — changes under version control; (3) reversible — rollback to a previous version; (4) testable — validate before production. Eliminates inconsistent manual configuration.

---

**Q:** What is the "Loose Coupling" principle in Well-Architected?  
**A:** The **Reliability** pillar: components with weak dependencies fail in isolation. Implementation: SQS between services (messages buffered if downstream fails), SNS for fan-out, API Gateway decouples clients from backends, ELB decouples clients from instances. The opposite: tight coupling (services call each other directly → a failure in one cascades).

---

**Q:** What are Game Days in the context of Operational Excellence?  
**A:** Planned simulations of failures and incidents to test response procedures. "Pre-mortems": imagine the system has failed and identify the causes. Goal: discover vulnerabilities and gaps in runbooks before they happen in production. A culture of learning through simulation. Related to Chaos Engineering (Netflix Chaos Monkey).

---

**Q:** What is the difference between "vertical scaling" and "horizontal scaling" in Well-Architected?  
**A:** **Vertical (Scale Up):** increase the instance size (m5.large → m5.xlarge). Physical limit. Requires downtime (usually). SPOF. **Horizontal (Scale Out):** add more instances of the same type. No practical limit. No downtime (gradual). Eliminates SPOF. Well-Architected favors horizontal: "Scale horizontally to increase aggregate system availability."

---

**Q:** What is the "consumption model" in the Cost Optimization pillar?  
**A:** Pay only for what you use — the opposite of the on-premises model (paying for peak capacity, even when idle). AWS: scale down when demand drops. Shut down dev environments at night/on weekends. Lambda/Fargate/DynamoDB On-Demand: pay only for actual execution. Goal: cost proportional to actual usage, not theoretical peak.

---

**Q:** What are the 3 types of security controls according to Well-Architected?  
**A:** **(1) Preventive:** prevents problems before they happen (IAM policies, SCP, SGs, encryption). **(2) Detective:** detects when something happens (CloudTrail, Config Rules, GuardDuty, Security Hub). **(3) Responsive:** responds after detection (runbooks, SNS/Lambda automation, incident response plans). A secure architecture needs all 3 types.

---

**Q:** What is "blast radius reduction" in the Security pillar?  
**A:** Limiting the potential impact of a security failure. Techniques: **Least privilege** (a compromised account has minimal access), **Accounts separation** (AWS Organizations — workloads in separate accounts, a failure in one does not affect others), **Network segmentation** (separate VPCs, private subnets), **Encryption per-resource** (a compromised key affects only the data under that key).

---

**Q:** How does the Sustainability pillar apply in practice on AWS?  
**A:** Practices: **(1)** Choose a Region with a lower carbon emission factor. **(2)** Use Graviton instances (ARM — better performance/watt). **(3)** Optimize utilization: avoid over-provisioning (Compute Optimizer). **(4)** Serverless: resources are released when not in use. **(5)** S3 Lifecycle: move cold data to Glacier (less energy). **(6)** Eliminate unnecessary workloads.

---

**Q:** What are Shared Responsibility Areas in Well-Architected?  
**A:** Well-Architected recognizes that some responsibilities are shared across teams: Dev, Ops, Security, and the Cloud Center of Excellence (CCoE). WAT reviews ideally involve: a solutions architect + a security engineer + a business representative. It is not solely the responsibility of ops or solely of dev. A culture of shared architecture.
