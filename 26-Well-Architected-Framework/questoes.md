# Questions — Module 16: Well-Architected Framework

> **SAA-C03 Domain**: All domains (cross-cutting framework)  
> **Difficulty**: Medium-High

---

**1.** A company wants to implement the principles of the Reliability pillar of the Well-Architected Framework. Which of the following actions is NOT a principle of this pillar?

- A) Scale horizontally to increase system availability
- B) Automatically recover from failure
- C) Minimize the environmental impact of workloads
- D) Stop guessing capacity by using Auto Scaling

<details><summary>Answer</summary>

**C** — Minimizing environmental impact belongs to the **Sustainability** pillar (the 6th pillar). The principles of the Reliability pillar are: automatically recover from failure, test recovery procedures, scale horizontally, stop guessing capacity, and manage change through automation.

</details>

---

**2.** A company has a critical application with an RPO of 1 hour and an RTO of 30 minutes. Which DR strategy is the most cost-effective for these requirements?

- A) Backup & Restore (hourly snapshots)
- B) Pilot Light with critical components running in a second Region
- C) Warm Standby with reduced capacity in a second Region
- D) Multi-site Active-Active

<details><summary>Answer</summary>

**B** — Pilot Light: keeps the database replicated (via Aurora Global DB or DMS CDC → RPO ~seconds) and AMIs ready in the second Region. In case of disaster, within ~10-15 minutes you can provision the components and redirect DNS (Route 53 failover). An RTO of 30 min and an RPO of 1h are achievable at a lower cost than Warm Standby.

</details>

---

**3.** A company is reviewing its workload with the AWS Well-Architected Tool and identified that it does not log all API calls. Under which pillar does this risk fall?

- A) Performance Efficiency
- B) Cost Optimization
- C) Security
- D) Operational Excellence

<details><summary>Answer</summary>

**C** — The Security pillar requires "enable traceability" — CloudTrail for APIs, CloudWatch Logs, Config for configurations. The absence of logging is a High Risk Item for security (it makes incident detection and investigation impossible).

</details>

---

**4.** A startup uses On-Demand EC2 to run its production workloads, which have been online 24/7 for 18 months. The CTO wants to reduce costs immediately. What is the main recommendation of the Cost Optimization pillar?

- A) Buy 1-year Reserved Instances or Savings Plans for the steady workload
- B) Migrate everything to Lambda (serverless)
- C) Move to Spot instances
- D) Reduce the number of Availability Zones

<details><summary>Answer</summary>

**A** — Steady 24/7 workloads are ideal for Reserved Instances or Savings Plans. 1-year RI/SP offer ~40% discount vs On-Demand. Spot (C) is interruptible (unsuitable for 24/7 production without fault tolerance). Lambda is not always cheaper for long-running workloads.

</details>

---

**5.** A company is developing a new application and wants to apply the "implement a strong identity foundation" principle of the Security pillar. Which practices represent this? (Select 2)

- A) Use IAM roles for all applications (no hardcoded access keys)
- B) Enable MFA for human users who access the AWS Console
- C) Configure CloudFront for all endpoints
- D) Use instances with more memory capacity

<details><summary>Answer</summary>

**A and B** — IAM roles (no long-term credentials) and MFA are the foundations of a "strong identity foundation" in the Security pillar. CloudFront is Performance Efficiency/Reliability. More memory is Performance Efficiency.

</details>

---

**6.** A company suffered a security incident. After the investigation, they realized they did not have enough logs to identify the root cause. Which Well-Architected Framework principle was neglected, and under which pillar?

- A) "Automate security best practices" — Pillar: Security
- B) "Enable traceability" — Pillar: Security
- C) "Refine operations procedures frequently" — Pillar: Operational Excellence
- D) "Anticipate failure" — Pillar: Reliability

<details><summary>Answer</summary>

**B** — "Enable traceability" is a design principle of the Security pillar. Traceability = CloudTrail logs (APIs), VPC Flow Logs (network), CloudWatch Logs (application), Config (configuration). Without these logs, forensic investigation is impossible.

</details>

---

**7.** A company has a development environment that runs 8 hours a day, 5 days a week. It currently uses On-Demand instances and spends $500/month. What does the Cost Optimization pillar recommend?

- A) Convert to 1-year Reserved Instances
- B) Use Spot Instances for development
- C) Create automation (Lambda + EventBridge) to start/stop the instances outside working hours
- D) Migrate the development environment to containers on Fargate

<details><summary>Answer</summary>

**C** — Non-production environments should run only when needed. Start/stop (8h × 5 days = 40h/week) vs 168h/week On-Demand → ~76% cost reduction. EventBridge Rules (schedule) → Lambda (start/stop EC2). Cost Optimization: adopt a consumption model (pay for what you use).

</details>

---

**8.** A company uses a database server with a very high specification (64 vCPUs), but average CPU usage is only 5%. What do the Performance Efficiency and Cost Optimization pillars recommend?

- A) Keep the high specification to ensure headroom
- B) Use AWS Compute Optimizer to identify the appropriate RDS instance type (rightsizing)
- C) Migrate to Lambda
- D) Add Read Replicas to distribute the load

<details><summary>Answer</summary>

**B** — Compute Optimizer analyzes historical metrics (CPU, memory — with the CloudWatch agent, network) and recommends the ideal instance type. Over-provisioned = unnecessary cost with no performance benefit. Rightsizing is a core Cost Optimization principle.

</details>

---

**9.** A company notices that its manual deployment process (SSH + rsync of code) is slow and error-prone, causing frequent incidents. Which Well-Architected Framework pillar addresses this, and which AWS technology?

- A) Reliability — use Multi-AZ to avoid downtime during deploys
- B) Operational Excellence — implement CI/CD with CodePipeline + CodeDeploy (perform operations as code)
- C) Performance Efficiency — use Elastic Beanstalk for automatic deployment
- D) Security — use IAM to control who can deploy

<details><summary>Answer</summary>

**B** — Operational Excellence: "perform operations as code" and "make frequent, small, reversible changes". CI/CD automates deploys, reduces human error, and enables fast rollback. CodePipeline + CodeDeploy implements this principle directly.

</details>

---

**10.** A global application needs to serve content with minimal latency to users worldwide without maintaining multiple application clusters in each Region. Which principle of the Performance Efficiency pillar is applied?

- A) Use the right technology for each task
- B) Go global in minutes using CloudFront, Global Accelerator, and S3
- C) Stop guessing capacity
- D) Democratize advanced technologies

<details><summary>Answer</summary>

**B** — "Go global in minutes": CloudFront distributes content to 400+ edge locations without needing infrastructure in each Region. Global Accelerator routes traffic over the AWS private network to reduce latency. Performance without scaling the application horizontally per Region.

</details>

---

**11.** A company wants to measure whether its AWS workloads are aligned with the principles of the Well-Architected Framework in a formal, documented way. What is the correct process?

- A) Manually compare the architecture with the whitepapers
- B) Use the AWS Well-Architected Tool (WAT) to perform a formal review per workload
- C) Hire AWS Professional Services for an audit
- D) Use Trusted Advisor to check the 5 categories

<details><summary>Answer</summary>

**B** — AWS Well-Architected Tool: define the workload, answer questions about each pillar, generate a risk report (HRI and MRI — High/Medium Risk Items), create an Improvement Plan, and track progress. Free, and can be performed by the architecture team itself.

</details>

---

**12.** From the perspective of the Sustainability pillar, which of the following architectures has the lowest environmental impact for an intermittent workload?

- A) Always-on (24/7) On-Demand EC2 with high utilization
- B) AWS Lambda (serverless) for intermittent workloads and Fargate for containers
- C) EC2 Reserved Instances to guarantee availability
- D) EC2 with Auto Scaling with a minimum of 5 instances

<details><summary>Answer</summary>

**B** — Sustainability: maximize resource utilization. Lambda and Fargate charge only while running (zero idle consumption). Less hardware in use = smaller carbon footprint. Serverless aligns resources precisely with demand, eliminating idle resources.

</details>

---

**13.** A company has different environments (dev, staging, prod) and wants to implement the "use IaC" principle of the Operational Excellence pillar. What direct benefit does this bring?

- A) Dev/staging/prod environments become identical, eliminating "works on my machine" bugs
- B) Automatically reduces EC2 cost
- C) Increases the security of EC2 instances
- D) Improves RDS query performance

<details><summary>Answer</summary>

**A** — IaC (CloudFormation, CDK, Terraform) allows creating identical environments in a reproducible way. The same template provisions dev, staging, and prod with the same configurations, eliminating configuration drift and the famous "works in dev but not in prod" problem.

</details>

---

**14.** The security team wants to ensure that no SSH port (22) is open to 0.0.0.0/0 in any of the company's Security Groups. What is the most robust preventive approach under the Security pillar?

- A) Audit the SGs manually every week
- B) Config Rule `restricted-ssh` + automatic Remediation that removes the offending rule
- C) CloudTrail alert when someone adds a rule to the SG
- D) IAM Permission denying `AuthorizeSecurityGroupIngress` for all users

<details><summary>Answer</summary>

**B** — A Config Rule continuously detects SGs with port 22 open to 0.0.0.0/0. With Remediation configured, Config runs an SSM Automation that removes the offending rule automatically (or notifies for manual action). Automates security best practices — a principle of the Security pillar.

</details>

---

**15.** A company is evaluating the total cost of ownership (TCO) for a workload that consumes 1,000 hours of EC2 m5.large per month, consistently for 2 years. Which purchasing model does the Cost Optimization pillar recommend?

- A) On-Demand (no commitment, maximum flexibility)
- B) Spot Instances (maximum discount)
- C) 1-year Compute Savings Plans ($X per committed hour)
- D) 3-year Convertible Reserved Instances (maximum discount)

<details><summary>Answer</summary>

**C** — 1-year Compute Savings Plans: ~66% discount vs On-Demand, more flexible than RI (you can change instance type, OS, and Region within the Compute SP). For a predictable workload over 2 years, some commitment is clearly the way to go. 3 years (D) gives a bigger discount but less flexibility; 1 year is the typical recommended balance point.

</details>
