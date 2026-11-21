# Real-World Use Cases — Practice Exams and Questions (Module 19)

## Case 1 — Analyzing a Hard High-Availability Question

**Context:** An exam question asks for the highest-availability architecture for a relational database with an automatic failover requirement of < 30 seconds and no data loss.

**Simulated Question:**
> A company needs a MySQL relational database with RTO < 30 seconds and RPO = 0 (zero data loss). Which configuration meets these requirements?
> A) RDS MySQL with two Read Replicas in different AZs
> B) RDS MySQL Multi-AZ with a synchronous replica in a separate AZ
> C) RDS MySQL with daily automated backup enabled
> D) EC2 MySQL with a Lambda failover script

**Analysis of Why Each Option Is Right or Wrong:**
```
A) ❌ Read Replicas are ASYNCHRONOUS → can have lag (RPO ≠ 0)
      Read Replicas do not automatically fail over for writes
      Read Replicas = scale reads, not HA

B) ✅ CORRECT:
   Multi-AZ = SYNCHRONOUS replica → RPO = 0 (no data loss)
   Automatic failover: 15-30 seconds
   The DNS endpoint automatically switches to the standby
   
C) ❌ Daily backup = RPO of up to 24h (very far from 0)
      Restoring from backup takes hours (RTO far too high)

D) ❌ Manual management = complexity, possible human error
      EC2 MySQL = you manage the OS, patches, HA
      Lambda failover = additional latency and possible failure

KEYWORDS IDENTIFIED: "automatic failover" + "no data loss"
→ Multi-AZ RDS (synchronous = RPO 0, automatic = RTO < 30s)
```

---

## Case 2 — Trap Question: Multi-AZ vs Read Replica

**Context:** An exam question mixes the two concepts to confuse candidates.

**Simulated Question:**
> An e-commerce application has slow reporting queries that impact customer transactions. The DBA identifies that heavy analytical queries run on the same database that processes orders. What is the simplest solution?
> A) Enable RDS Multi-AZ on the existing database
> B) Create a Read Replica and direct analytical queries to it
> C) Migrate to Aurora Serverless
> D) Add ElastiCache in front of RDS

**Analysis:**
```
A) ❌ Multi-AZ = HA/failover, does not improve performance
      The Multi-AZ replica is a STANDBY (does not accept reads)
      Enabling Multi-AZ does not solve the load problem

B) ✅ CORRECT:
   Read Replica = asynchronous copy for reads
   Direct analytical queries to the Read Replica endpoint
   The primary database stays dedicated to transactions (writes)
   
C) ⚠️ Works but is a bigger change (data migration, cost)
      "Simplest" points to Read Replica
      
D) ⚠️ ElastiCache helps with repeated/identical queries
      Analytical queries are generally unique (ad hoc)
      Does not solve complex queries that always change

LESSON: "performance problem" + "heavy read queries" → Read Replica
        "failover/availability" → Multi-AZ
```

---

## Case 3 — Analyzing a Cost-Efficiency Question

**Context:** A question asks you to choose S3 storage based on a specific access pattern.

**Simulated Question:**
> A company stores application logs in S3. Logs from the current month are accessed daily for debugging. Logs older than 30 days are rarely accessed but must be available within minutes when needed. Logs older than 1 year can tolerate hours of retrieval time. Which combination of S3 classes is the most cost-effective?
> A) Everything in S3 Standard
> B) S3 Standard for 30 days, then Glacier Deep Archive
> C) S3 Standard for 30 days, Standard-IA for 30-365 days, then Glacier Flexible Retrieval
> D) S3 Intelligent-Tiering for all objects

**Analysis:**
```
Requirements mapping:
- 0-30 days: frequent access, daily debugging → S3 Standard
- 30-365 days: rarely accessed, available "within minutes" → Standard-IA
  (NOT Glacier — Glacier takes hours for Flexible or 12h for Deep Archive)
- > 1 year: retrieval can take hours → Glacier Flexible (1-5h) ✓

A) ❌ Cost too high, everything in Standard even old files
B) ❌ After 30 days it goes to Deep Archive (12h retrieval)
      But the 30-365 day requirement asks for "available within minutes"!
C) ✅ CORRECT — perfect mapping by period
D) ⚠️ Works automatically but:
      Per-object monitoring fee ($0.0025/1000 objects)
      For many small objects: can be more expensive than C
      "Most cost-effective" → C is more optimized when the pattern is known

LESSON: Glacier = hours (not minutes). Standard-IA = milliseconds (lower cost)
```

---

## Case 4 — Security Question: Which Service for Which Threat

**Context:** Security questions often confuse GuardDuty, WAF, Shield, Macie, and Inspector.

**Simulated Question:**
> A company detected that its S3 bucket with customer data contains national ID numbers and credit card numbers exposed without encryption. Which AWS service identifies and alerts on this sensitive data?
> A) AWS GuardDuty
> B) Amazon Macie
> C) AWS WAF
> D) AWS Config

**Analysis:**
```
A) ❌ GuardDuty = detects THREATS (anomalous behavior, mining, exfiltration)
      Analyzes: VPC Flow Logs, CloudTrail, DNS
      Does NOT analyze the CONTENT of data in S3
      
B) ✅ CORRECT — Amazon Macie:
   Uses ML to discover sensitive data in S3
   Identifies: PII (national ID, name, email), PCI (cards), credentials
   Generates findings: "S3BucketEncryptionDisabled" + "SensitiveData:S3Object/PII"
   
C) ❌ WAF = filters HTTP web requests (SQLi, XSS)
      Does NOT analyze the content of stored objects
      
D) ❌ AWS Config = resource CONFIGURATION compliance
      Checks whether the bucket has encryption enabled (configuration)
      Does NOT check the CONTENT of objects

SECURITY SERVICES MAPPING:
├── Sensitive data in S3 → Macie
├── Threats and anomalous behavior → GuardDuty
├── HTTP web attacks → WAF
├── DDoS → Shield
├── Resource misconfiguration → Config
├── Vulnerabilities in EC2/containers → Inspector
└── Centralizing findings → Security Hub
```

---

## Case 5 — Simulating a Review Strategy on Exam Day

**Context:** The candidate has 65 questions and 130 minutes. How to manage time and flagged questions.

**Execution Simulation:**
```
QUESTIONS 1-20 (40 min — ~2 min/question):
Q1: I know the answer → answer immediately (45s)
Q5: Torn between A and C → flag it, pick C (the one that seems better), file under "review"
Q12: Too long, too many details → flag it, skip to the next (saves time)
Q15: I'm certain → 30s

QUESTIONS 21-65 (continuing...):
Q33: Words "multi-AZ" + "performance" → Red flag (Multi-AZ vs Read Replica trap)
     Read carefully → it is about reads → Read Replica → answer
Q47: Question about NAT Gateway cost → I know NAT GW is billed per hour
     Answer with confidence
Q55: 3 services I have never seen combined → flag it, pick by eliminating the ones that seem wrong

REVIEW (flagged questions — 30 min):
Q5 (torn between A vs C): Re-read the question, spot "synchronous" → Multi-AZ → changed to B
Q12 (long): Now calmly, I read it → Kinesis vs SQS question, I spot "replay"
            → Kinesis. Answer.
Q55 (unfamiliar services): 2 options left, choose the one with managed services
                           → Principle: "managed > self-managed" on AWS

LESSONS FROM THE SIMULATION:
1. Long questions: read the LAST paragraph first (it usually has the key constraint)
2. Questions with lists: eliminate the clearly wrong ones
3. "Minimum cost" → Spot/Reserved/Serverless/S3-IA/Glacier
4. "Minimum operational overhead" → Managed services (RDS > EC2+MySQL)
5. "Maximum availability" → Multi-AZ + Multi-Region + Auto Scaling
6. Never spend > 3 minutes on a question — flag it and move on
```
