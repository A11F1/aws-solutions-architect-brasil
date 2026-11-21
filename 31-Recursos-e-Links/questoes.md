# Questions — Module 21: Resources and Links

> **Study Orientation Quiz** — Know your sources and strategies  
> **Difficulty**: Basic-Medium

---

**1.** What is the official AWS resource for accessing free practice questions for the SAA-C03?

- A) AWS Certification Website → Practice Exam (paid)
- B) AWS Skill Builder — Official Exam Prep module (free, includes sample questions)
- C) Amazon.com → AWS Books
- D) GitHub AWS → practice-exams

<details><summary>Answer</summary>

**B** — AWS Skill Builder (skillbuilder.aws) offers: (1) free Sample Questions (20 questions) to calibrate your level; (2) Official Practice Exam (paid, 65 questions) with detailed feedback; (3) free and paid Courses (Individual and Team). It is the most reliable source because the questions are created by AWS itself.

</details>

---

**2.** Which AWS whitepaper covers the 6 pillars of well-architected design?

- A) "AWS Security Best Practices"
- B) "AWS Well-Architected Framework" (available at aws.amazon.com/architecture/well-architected)
- C) "Overview of AWS Services"
- D) "AWS Migration Acceleration Program"

<details><summary>Answer</summary>

**B** — The "AWS Well-Architected Framework" whitepaper describes the 6 pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, and Sustainability. It is required reading for the SAA-C03 because exam questions frequently cite the principles directly.

</details>

---

**3.** What is the advantage of Tutorials Dojo as a study resource for the SAA-C03?

- A) It is the most complete official AWS resource
- B) It offers practice exams with detailed explanations per option (including why each wrong answer is wrong) and high correlation with the real exam
- C) It is free with no limitations
- D) Created by the authors of the AWS exam themselves

<details><summary>Answer</summary>

**B** — Tutorials Dojo (tutorialsdojo.com): renowned for the quality of its explanations. For each question, it explains why each alternative option is incorrect — not just the correct one. Cheat sheets per service. Practice exams in timed and review modes. High reported correlation with the real exam. Created by Jon Bonso (certified AWS architect).

</details>

---

**4.** Which domain should you prioritize in the last 3 days before the SAA-C03 exam?

- A) Networking (VPC, Direct Connect, Route 53) — high complexity
- B) Design Secure Applications (30% of the exam) — the highest weight on the exam
- C) Machine Learning — frequent novelty
- D) Billing and Pricing — easy to memorize

<details><summary>Answer</summary>

**B** — Design Secure Applications is 30% of the exam — the largest domain. Review in the final days: IAM policies (conditions, boundaries), KMS vs CloudHSM, VPC security (stateful SG vs stateless NACL), S3 security (BPA, policies, ACLs, OAC), Cognito (User Pools vs Identity Pools), GuardDuty, Shield vs WAF, Secrets Manager vs Parameter Store.

</details>

---

**5.** Which FAQ is the most important to read before the SAA-C03 exam?

- A) The EC2 and S3 FAQs only (the most-tested services)
- B) The FAQs of the 3-5 core services: EC2, S3, RDS, IAM, VPC — they are the most detailed and cover the nuances that get tested
- C) The FAQs of all 200+ AWS services
- D) FAQs are not useful — focus on the courses

<details><summary>Answer</summary>

**B** — The AWS FAQs describe limits, special cases, and nuances that show up on the exam. Priority: EC2 (instance types, pricing, HA), S3 (storage classes, limits, security), RDS (Multi-AZ vs Read Replicas, backups, encryption), IAM (policy evaluation, roles, SCP), VPC (peering, endpoints, routing). FAQs for specific services such as Lambda, DynamoDB, and ELB are also valuable.

</details>

---

**6.** Which AWS tool is used to estimate the cost of an architecture before implementing it?

- A) AWS Cost Explorer (analyzes historical costs)
- B) AWS Pricing Calculator (calculator.aws) — estimates service costs before using them
- C) AWS Trusted Advisor (optimization recommendations)
- D) AWS Budgets (sets spending limits)

<details><summary>Answer</summary>

**B** — AWS Pricing Calculator (calculator.aws): a free tool for building architecture estimates. Add services, configure parameters (instance type, Region, expected usage), and see the monthly estimate. Unlike Cost Explorer (which analyzes spending that has already occurred). On the exam, when the question asks "how to estimate costs before deployment" → Pricing Calculator.

</details>

---

**7.** You need to understand how traffic enters and leaves a VPC. Which recommended AWS resource offers network architecture diagrams and examples?

- A) AWS EC2 User Guide
- B) AWS VPC User Guide (docs.aws.amazon.com/vpc/latest/userguide) + AWS Architecture Center
- C) AWS CLI Reference
- D) AWS SDK for Python (Boto3)

<details><summary>Answer</summary>

**B** — The AWS VPC User Guide is the most complete documentation on networking. The AWS Architecture Center (aws.amazon.com/architecture) has reference architecture diagrams for common use cases — excellent for visualizing multi-AZ HA, hybrid connectivity, and microservices patterns. Both are free and official.

</details>

---

**8.** An SAA-C03 candidate has 4 weeks to study. What is the most effective distribution?

- A) Weeks 1-4: watch the entire Stephane Maarek course with no practice exams
- B) Weeks 1-2: theory course; Week 3: practice exams with review of wrong answers; Week 4: focus on weak points + final practice exam
- C) Week 1: practice exam for a baseline; Weeks 2-3: study content; Week 4: practice exam
- D) Study every existing AWS service (200+) uniformly

<details><summary>Answer</summary>

**B** — Proven methodology: (1) Solid theory on the core services; (2) Practice exams to identify gaps — feedback from wrong questions is the most efficient study; (3) Focused review of the weak domains; (4) Final practice exam to measure readiness. Theory without practice is no use; practice exams without reviewing the wrong answers are a waste of time.

</details>

---

**9.** What is the ideal resource for understanding EC2 pricing models (On-Demand vs Reserved vs Spot)?

- A) AWS Billing Console → Bills
- B) AWS EC2 Pricing page (aws.amazon.com/ec2/pricing) + AWS Compute Optimizer
- C) AWS Trusted Advisor → Cost Optimization
- D) CUR (Cost and Usage Report)

<details><summary>Answer</summary>

**B** — The EC2 pricing page has all the models with an interactive calculator. AWS Compute Optimizer analyzes actual utilization and suggests the right instance type. For the exam, know: On-Demand (maximum flexibility, highest price), Reserved (1 or 3 years, up to 72% off), Savings Plans (spend commitment, more flexible), Spot (up to 90% off, interruptible).

</details>

---

**10.** What is the AWS Architecture Center and how do you use it to study?

- A) Support center for certified AWS architects
- B) Library of reference architectures, diagrams, and whitepapers at aws.amazon.com/architecture — study the patterns of each industry
- C) Diagram design tool that competes with draw.io
- D) Part of the AWS Management Console for visualizing infrastructure

<details><summary>Answer</summary>

**B** — AWS Architecture Center (aws.amazon.com/architecture): a free library of "Reference Architectures" by use case (3-tier web app, serverless API, data lake, etc.) and industry (fintech, healthcare, media). To study: read the reference architectures relevant to the SAA-C03 to understand how services combine in real production patterns.

</details>

---

**11.** Which YouTube channel is most recommended for free AWS preparation videos?

- A) Official AWS channel → "AWS re:Invent" (technical but not certification-focused)
- B) Stephane Maarek, Neal Davis (Digital Cloud Training), Be a Better Dev — certification-focused content, free on YouTube
- C) Linux Foundation YouTube
- D) Microsoft Azure YouTube (for comparison)

<details><summary>Answer</summary>

**B** — For the SAA-C03: Stephane Maarek (free snippets from his Udemy course), Neal Davis's Digital Cloud Training (detailed tutorials), Be a Better Dev (practical demos). The official AWS channel has excellent content (re:Invent, re:Inforce) but is not exam-focused — it is more technical/in-depth on specific services.

</details>

---

**12.** Where can you find the updated version of the official SAA-C03 Exam Guide?

- A) Amazon.com → AWS Study Guides
- B) aws.amazon.com/certification/certified-solutions-architect-associate → "Exam Prep" → "Exam Guide PDF"
- C) AWS Skill Builder → My Courses
- D) AWS Console → Certification → Exam Guide

<details><summary>Answer</summary>

**B** — The official Exam Guide PDF is on the certification page at aws.amazon.com. It describes: (1) domains and weights, (2) expected tasks per domain, (3) in-scope services, (4) out-of-scope services. Required reading before you start studying — it guides what to prioritize based on the official weights.

</details>

---

**13.** What is the most important flashcard to memorize about S3 storage classes?

- A) S3 Standard → S3 Intelligent-Tiering → S3 Standard-IA → S3 One Zone-IA → S3 Glacier Instant → S3 Glacier Flexible → S3 Glacier Deep Archive (decreasing order of storage cost)
- B) S3 has only 2 classes: Standard and Glacier
- C) S3 Glacier is for monthly backups; S3 Standard for daily backups
- D) S3 Intelligent-Tiering only works for objects > 1 TB

<details><summary>Answer</summary>

**A** — The 7 S3 classes in order of storage cost (most expensive to cheapest): Standard → Intelligent-Tiering → Standard-IA → One Zone-IA → Glacier Instant → Glacier Flexible → Glacier Deep Archive. Inversely proportional: the cheaper the storage, the higher the retrieval cost and latency. The exam tests: which class for which use case (frequent access, infrequent access, archive).

</details>

---

**14.** When should you use the AWS Free Tier to practice before the exam?

- A) Never — it can generate unexpected charges
- B) For the hands-on labs in the study modules: create a VPC, Lambda, S3, DynamoDB — core services have a generous free tier; always set billing alerts
- C) Only for serverless services (Lambda, DynamoDB)
- D) The Free Tier is only for corporate accounts

<details><summary>Answer</summary>

**B** — The Free Tier (aws.amazon.com/free) offers: 12 months of t2.micro/t3.micro EC2, 5 GB S3, 25 GB DynamoDB, 1M Lambda requests/month, etc. To practice: create a new account, enable AWS Budgets with an alert at $1-5, and do the study's hands-on labs. Real practice beats purely theoretical study. Always clean up resources after the labs.

</details>

---

**15.** After passing the SAA-C03 exam, what is the next most recommended certification to continue the path?

- A) AWS Cloud Practitioner (it is a mandatory prerequisite)
- B) AWS Solutions Architect Professional (SAP-C02) or AWS Developer Associate (DVA-C02) depending on career focus
- C) Google Cloud Professional Architect
- D) Kubernetes CKA — complementary to AWS

<details><summary>Answer</summary>

**B** — Post-SAA-C03 paths: (1) Deepen architecture: SAP-C02 (Professional, harder, enterprise focus); (2) Development focus: DVA-C02 (Developer Associate, serverless, CI/CD, SDK); (3) Operations focus: SOA-C02 (SysOps Administrator Associate); (4) Specialties: AWS Security Specialty, Data Analytics Specialty. CCP (Cloud Practitioner) is an optional prerequisite, not mandatory.

</details>
