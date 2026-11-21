# Questions — Module 19: Practice Exams and Questions

> Study strategy for the exam

---

**About the practice exam questions**

The README for this module contains the full 65-question practice exam. This file contains additional exam-strategy questions and meta-questions about the exam format.

---

**1.** On the SAA-C03 exam, a question asks you to choose the "most cost-effective" solution. Which elimination approach should you use?

- A) Always choose the option with Lambda (serverless is always cheaper)
- B) Eliminate options with idle resources (always-on EC2, over-provisioned instances), then compare the cost of the remaining ones
- C) Always choose Reserved Instances (largest discount)
- D) Choose the option with the fewest AWS services (fewer services = lower cost)

<details><summary>Answer</summary>

**B** — "Most cost-effective" varies by context: for intermittent workloads, Lambda/Fargate/Spot are cheaper. For stable 24/7 workloads, Reserved Instances. Eliminate first: (1) over-provisioning, (2) idle resources, (3) unnecessary redundancy. Then compare the remaining services by pricing model.

</details>

---

**2.** An exam question asks you to choose 2 correct answers out of 5 options. What is the best strategy?

- A) Select the first 2 options that seem correct
- B) Identify the 2 most obviously incorrect ones, eliminate them, and from the remaining 3 choose the 2 most correct
- C) Mark it as uncertain and come back later
- D) Look for keywords like "serverless" or "managed" in the options

<details><summary>Answer</summary>

**B** — Elimination strategy: first remove the clearly wrong options (wrong service for the context, violates an obvious security principle). From the remaining ones, choose the 2 that best meet ALL of the question's requirements. In multiple-response questions, all correct options must be selected to score.

</details>

---

**3.** The exam asks for a "highly available" architecture. What is the minimum criterion the answer must meet?

- A) Use the newest AWS service
- B) Have resources in at least 2 Availability Zones with automatic failover
- C) Use Reserved Instances in 3 regions
- D) Have daily data backups

<details><summary>Answer</summary>

**B** — High availability = tolerance to the failure of one AZ. Minimum: resources in 2+ AZs with automatic failover (ELB + multi-AZ ASG, RDS Multi-AZ, etc.). Backup is for recovery (not HA). Multi-region is for disaster recovery or global reach (different from simple HA).

</details>

---

**4.** A question describes an application that "sometimes has unpredictable traffic spikes" and needs a compute solution. Which service is the most likely answer?

- A) Fixed EC2 On-Demand
- B) EC2 Reserved Instances
- C) Lambda or ECS Fargate with Auto Scaling
- D) EC2 Spot Instances

<details><summary>Answer</summary>

**C** — "Unpredictable" and "spikes" = serverless/auto-scaling. Lambda scales to zero when there is no load and to thousands during a spike, with no manual configuration. Fargate with metric-based ASG as well. Reserved Instances are for predictable load; Spot can be interrupted during spikes.

</details>

---

**5.** On the exam, when the question mentions "least change to the existing application" (lift-and-shift), which answer category is most likely?

- A) Lambda and DynamoDB (native serverless)
- B) EC2 + RDS (VM equivalent + relational database)
- C) EKS + S3 (containers + storage)
- D) CloudFormation + CodePipeline (automation)

<details><summary>Answer</summary>

**B** — "Least possible change" = lift-and-shift = EC2 (equivalent VM), RDS (managed database, same engine), ELB (load balancer). The application does not need to be rewritten. Lambda/DynamoDB require refactoring. EKS requires containerization. The question is testing recognition of the "Rehost" pattern from the 7 Rs.

</details>

---

**6.** What is the most important tip when reading an SAA-C03 exam question?

- A) Read the last sentence of the question first to understand what is being asked
- B) Identify all explicitly mentioned requirements (availability, cost, security, performance) before looking at the options
- C) Eliminate the longest options (usually unnecessary complexity)
- D) Always choose the option with the most managed service

<details><summary>Answer</summary>

**B** — Read the full scenario and note the explicit requirements (e.g., "RTO < 1 hour", "no custom code", "lowest cost"). Each requirement can eliminate options. Some questions have distractors that meet some requirements but violate others. Identifying ALL requirements before evaluating the answers is the most effective strategy.

</details>

---

**7.** On exam day, how much time should you dedicate to the first pass through the 65 questions?

- A) Solve all questions meticulously on the first pass (without skipping)
- B) ~1.5 minutes per question on the first pass (~97 min); flag difficult questions; use the remaining time to review
- C) Skip the first 20 questions (usually the hardest)
- D) Answer the easiest ones first (the last 30 questions)

<details><summary>Answer</summary>

**B** — Time management: 130 minutes / 65 questions = 2 min/question. Reserve ~97 min for the first pass (~1.5 min/question). Questions you are confident about: answer and move on. Difficult questions: flag and skip. With the remaining 33 min, review the flagged ones. Never get stuck on a single question for 10 minutes.

</details>

---

**8.** An exam question has the following options and asks for the "most secure" answer:
- A) Use IAM access keys hardcoded in the application code
- B) Use an IAM Role associated with the EC2 instance profile
- C) Store access keys in EC2 environment variables
- D) Use a ~/.aws/credentials file on the server

<details><summary>Answer</summary>

**B** — IAM Role via Instance Profile: temporary credentials automatically rotated by STS; never exposed in code, env vars, or files. Hardcoded access keys (A), env vars (C), and a credentials file (D) are all leak risks. The general rule: never use long-lived credentials when IAM Roles are possible.

</details>

---

**9.** Which SAA-C03 exam domain has the highest weight?

- A) Design Resilient Architectures (26%)
- B) Design High-Performing Architectures (24%)
- C) Design Secure Applications (30%)
- D) Design Cost-Optimized Architectures (20%)

<details><summary>Answer</summary>

**C** — Design Secure Applications is 30% of the exam — it is the heaviest domain. Focus on: IAM (roles, policies, conditions), KMS, VPC security (SG vs NACL), S3 (Block Public Access, OAC, pre-signed), Cognito, Secrets Manager, GuardDuty, Shield, WAF. 30% of the questions are about security.

</details>

---

**10.** What is the minimum passing score for SAA-C03?

- A) 65% (700/1000)
- B) 72% (720/1000)
- C) 75% (750/1000)
- D) 80% (800/1000)

<details><summary>Answer</summary>

**B** — Minimum score: **720/1000** (on a scale of 100-1000). It is not 72% of questions correct — the score uses a psychometric model (unscored pilot questions do not count). In practice, you need to get roughly 43-47 of the 65 questions right (there are unscored test questions that appear randomly).

</details>

---

**11.** A question describes: "the company needs a solution that processes data **in batch** once a day". Which service/combination is a candidate for the correct answer?

- A) Kinesis Data Streams + Lambda
- B) EventBridge Scheduler + Lambda (or ECS/Batch)
- C) SQS FIFO + Lambda
- D) Kinesis Data Firehose

<details><summary>Answer</summary>

**B** — "Once a day" = scheduled processing. EventBridge Scheduler (or a `schedule` type Rule) + Lambda/ECS Fargate/AWS Batch is the pattern. Kinesis is for continuous real-time streaming, not daily batch. AWS Batch is excellent for heavy-compute batch jobs.

</details>

---

**12.** You are taking the exam and a question asks you to choose between RDS Multi-AZ and Read Replicas. The requirement says "high availability for writes". What should you choose?

- A) Read Replicas (can be promoted to primary on failure)
- B) RDS Multi-AZ (automatic synchronous failover to standby; no loss of write data)
- C) Both, with Read Replicas pointing to the Multi-AZ standby
- D) Aurora instead of standard RDS

<details><summary>Answer</summary>

**B** — Multi-AZ: **synchronous** replication to a standby in another AZ. Automatic failover in 1-2 minutes. No loss of writes (synchronous). Read Replicas: **asynchronous** replication — can have lag; if the primary fails, promotion is manual (not automatic for RDS — only Aurora promotes automatically).

</details>

---

**13.** A question mentions "data transfer out cost". What is the architectural impact of this?

- A) Data transferred between AWS regions is free
- B) Upload to S3 is free; S3 → internet transfer has a cost (~$0.09/GB); using CloudFront reduces the transfer cost
- C) VPC Peering never has a data transfer cost
- D) Lambda has no data transfer cost

<details><summary>Answer</summary>

**B** — Data transfer pricing: UPLOAD to AWS = free. Download from AWS to the internet = ~$0.09/GB. CloudFront has a lower per-GB cost than S3 directly + includes edge caching. Intra-region transfer (same AZ) = generally free between services. AZ to AZ in the same region = has a cost ($0.01/GB). Between regions = always has a cost.

</details>

---

**14.** On the exam, when a question mentions a "startup with a limited budget" needing high availability, which approach does the exam usually favor?

- A) Multi-region active-active for maximum HA
- B) Multi-AZ with smaller instances + Auto Scaling (HA without over-provisioning)
- C) On-Demand instances in a single AZ (cheapest)
- D) 3-year Reserved Instances immediately

<details><summary>Answer</summary>

**B** — The exam frequently tests cost-effective high availability. Multi-AZ is sufficient for most HA cases. Smaller instances + Auto Scaling avoid expensive and unnecessary on-demand capacity. Multi-region is for DR (different from HA) and costs much more. Reserve only after growth stabilizes.

</details>

---

**15.** What is the purpose of "unscored" questions on the SAA-C03 exam?

- A) There are no unscored questions on SAA-C03
- B) They are pilot questions (testing new questions for future versions of the exam), randomly distributed among the 65
- C) They are the 5 easiest questions to guarantee a minimum score
- D) They are questions about new services that were recently announced

<details><summary>Answer</summary>

**B** — AWS includes "field trial" or "pre-operational" questions — new questions being tested for calibration. You do not know which ones are scored and which are not. Implication: do not skip any question (even ones that seem very difficult may be unscored; even easy ones are scored). Answer all of them.

</details>
