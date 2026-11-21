# Real-World Use Cases — Study Resources in Action (Module 21)

## Case 1 — Study Plan for Someone with 8 Weeks

**Context:** An IT professional with on-premises infrastructure experience (no prior AWS experience) plans to earn the SAA-C03 in 8 weeks, studying 2 hours per day.

**Week-by-Week Plan:**
```
WEEK 1 — FOUNDATIONS AND COMPUTE (14h):
Resources:
  ├── Udemy course by Stephane Maarek: Sections 1-5 (EC2, ELB, ASG)
  ├── AWS Free Tier: create an account + explore the console
  └── Skill Builder: "AWS Cloud Practitioner Essentials" (free)
  
Practice:
  └── Create an EC2 t3.micro, connect via SSH, install nginx, access it via browser
  
Goal: understand EC2, instance types, pricing, ELB, Auto Scaling

WEEK 2 — STORAGE AND DATABASES (14h):
Resources:
  ├── Maarek course: Sections 6-10 (EBS, EFS, S3, RDS, DynamoDB)
  └── AWS Docs: S3 FAQ + RDS FAQ (30min each)
  
Practice:
  └── Create an S3 bucket, versioning, lifecycle policy
  └── Create an RDS Free Tier instance, connect via DBeaver

WEEK 3 — NETWORKING AND SECURITY (14h):
Resources:
  ├── Maarek course: Sections 11-14 (VPC, Route 53, CloudFront, WAF)
  └── Whitepaper: "AWS Security Best Practices" (1h)
  
Practice:
  └── Create a VPC with 2 subnets (public+private), NAT GW, bastion host

WEEK 4 — SERVERLESS AND INTEGRATION (14h):
Resources:
  ├── Maarek course: Sections 15-19 (Lambda, SQS, SNS, API GW, Kinesis)
  └── Tutorial: "Build a Serverless App" (free AWS Labs)
  
Practice:
  └── Lambda + API GW + DynamoDB (complete todo app)

WEEK 5 — MICROSERVICES, MONITORING, AND ANALYTICS (14h):
Resources:
  ├── Maarek course: Sections 20-24 (ECS, CloudWatch, CloudTrail, Athena)
  └── AWS Architecture Center: Well-Architected examples

WEEK 6 — MIGRATION, COST, BEST PRACTICES (14h):
Resources:
  ├── Maarek course: Final sections
  └── Whitepaper: "AWS Well-Architected Framework"
  
WEEK 7 — FIRST PRACTICE EXAM (14h):
Resources:
  ├── Tutorials Dojo: 1 full practice exam (65 questions)
  └── Review ALL wrong answers (read the explanation carefully)
  
Goal: score > 65% on the first practice exam

WEEK 8 — INTENSIVE REVIEW + EXAM (14h):
Resources:
  ├── Tutorials Dojo: second full practice exam
  ├── Review cheatsheets for the most-missed services
  └── Flashcards in the days before the exam
  
SCHEDULE THE EXAM: at the start of week 7 (motivation to study!)
```

---

## Case 2 — Using Skill Builder for Hands-On Labs

**Context:** The candidate knows the theory but has never operated on AWS. Skill Builder offers hands-on labs with credits included.

**Skill Builder Lab Plan:**
```
AWS Skill Builder Individual ($29/month or $299/year): 
  Access to labs in a real sandbox (you never pay AWS costs in the labs)
  
PRIORITY LABS FOR SAA-C03:
├── "Introduction to Amazon EC2" (free on basic Skill Builder)
├── "Introduction to Amazon S3" 
├── "Creating an Amazon Virtual Private Cloud (VPC)"
├── "Introduction to Amazon RDS"
├── "Introduction to AWS Lambda"
├── "Introduction to Amazon DynamoDB"
├── "Introduction to Amazon CloudWatch"
├── "Introduction to AWS IAM"
└── "AWS Well-Architected Best Practices" (not a lab, it's a theory module)

EXAM READINESS: "AWS Certified Solutions Architect - Associate" 
  (free official course, 3h, covers all exam domains)
  Available at: skillbuilder.aws → search "SAA Exam Readiness"

FREE ON BASIC SKILL BUILDER (no payment):
├── Cloud Practitioner Essentials
├── AWS Technical Essentials  
├── Exam Readiness SAA-C03
└── Several introductory-level labs
```

---

## Case 3 — Strategy for Architecture Diagram Questions

**Context:** The SAA-C03 exam presents complex multi-service scenarios. How to structure your reasoning.

**Analysis Framework:**
```
4-STEP METHOD:

STEP 1 — READ THE LAST PARAGRAPH FIRST:
(it usually contains the main constraint)
"The company needs the LOWEST-COST solution that meets the requirements."
→ This immediately eliminates all the more expensive solutions

STEP 2 — IDENTIFY KEYWORDS:
┌─────────────────────────────────────────────────────────┐
│ "serverless"       → Lambda, Fargate, Aurora Serverless  │
│ "least overhead"   → Managed services                    │
│ "real-time"        → Kinesis, EventBridge                │
│ "decouple"         → SQS, SNS                            │
│ "global"           → CloudFront, Route 53, DynamoDB GT   │
│ "auto failover"    → Multi-AZ, Route 53 Failover         │
│ "scale reads"      → Read Replica (NOT Multi-AZ)         │
│ "no data loss"     → Multi-AZ, synchronous               │
│ "compliance/audit" → CloudTrail + S3 + Object Lock       │
│ "sensitive data"   → Macie, KMS, Secrets Manager         │
└─────────────────────────────────────────────────────────┘

STEP 3 — ELIMINATE CLEARLY WRONG OPTIONS:
"Install software on EC2 instances" → red flag (more overhead)
"Manage manually" in an HA question → almost always wrong
"Share root credentials" → always wrong

STEP 4 — CHOOSE BETWEEN THE 2 REMAINING:
Apply the question's criterion (cost? performance? availability?)
If still in doubt: managed service > self-managed on AWS
```

---

## Case 4 — Tutorials Dojo vs the Real Exam

**Context:** The candidate scores 78% on Tutorials Dojo. Are they ready for the real exam?

**Practice Exam Calibration:**
```
PRACTICE EXAM SCORE × REAL EXAM CORRELATION (empirical):
Tutorials Dojo 75% → Real exam probably 70-75% (passing: 72%)
Tutorials Dojo 80% → Real exam probably 75-82% (comfortable pass)
Tutorials Dojo 85% → Real exam probably 80-88% (solid pass)

WHY PRACTICE EXAMS CAN BE HARDER:
Tutorials Dojo and ExamTopics have more technical/detailed questions
The real exam has more scenario questions than pure technical questions
Real exam: 50 scored + 15 unscored (new questions don't count)

INTERPRETING SCORE BY DOMAIN:
Domain 1 (Security 30%): score < 70% → review IAM, KMS, VPC
Domain 2 (Resilience 26%): score < 70% → review Multi-AZ, Auto Scaling, DR
Domain 3 (Performance 24%): score < 70% → review ElastiCache, Kinesis, CloudFront
Domain 4 (Cost 20%): score < 70% → review S3 tiers, Reserved, Spot

RECOMMENDED GOAL:
Take 2-3 full 65-question practice exams
Average score > 78% on the last 2 practice exams → schedule the exam
Don't take a practice exam the day before the exam (anxiety)
```

---

## Case 5 — Post-Certification: Next Career Steps

**Context:** The candidate just passed the SAA-C03. What to do next to maximize the return on the investment.

**Career Plan:**
```
IMMEDIATE (first 7 days):
├── Update LinkedIn:
│   Licenses and Certifications section:
│   Name: "AWS Certified Solutions Architect - Associate"
│   Issuer: Amazon Web Services (AWS)
│   Verification link: Credly badge
├── Accept the badge on Credly (email from AWS)
└── Update your resume/profile on job boards

NEXT CERTIFICATIONS (recommended order):
OPTION A — Broaden (other Associates before the Professional):
SAA-C03 → DVA-C02 (Developer) → SOA-C02 (SysOps) → SAP-C02 (Pro)

OPTION B — Go deeper directly:
SAA-C03 → SAP-C02 (Professional — highest Architect level)
           (SAA renews automatically when you pass SAP)

OPTION C — Specialization:
SAA-C03 → DOP-C02 (DevOps Pro) or ANS-C01 (Advanced Network)
           or SCS-C02 (Security Specialty)

FOR JOB SEEKERS:
AWS Partner Network (register a free account) → find openings
LinkedIn: "AWS Solutions Architect" → job alerts
GitHub: create a repository with AWS projects (practical evidence)
Blog: write about what you learned → visibility

FOR FREELANCERS:
Upwork/Toptal: certification increases hourly rate
AWS Marketplace: companies look for certified SAs for consulting
AWS Partner: a company can become a partner with 2+ certified SAs

STAYING RELEVANT:
AWS re:Invent (November): main conference, news of the year
AWS Blogs (aws.amazon.com/blogs): weekly service news
re:Post: answer community questions (solidify your knowledge)
```
