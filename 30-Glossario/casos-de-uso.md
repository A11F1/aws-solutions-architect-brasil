# Real-World Use Cases — The Glossary in Practice (Module 20)

## Case 1 — When "RTO" and "RPO" Determine the Architecture

**Context:** An exam question defines RTO and RPO as constraints. The candidate needs to know which DR strategy to choose based on those values.

**Application Scenarios:**
```
SCENARIO A: RPO = 0, RTO < 30s
(no data loss, near-instant recovery)
→ Architecture: Multi-Region Active-Active
→ Aurora Global Database (sync lag < 1s ≈ RPO≈0)  
→ Route 53 Latency Routing (automatic failover ≈ 60s) 
→ Cost: $$$$

SCENARIO B: RPO ≤ 15min, RTO ≤ 1h
→ Architecture: Warm Standby
→ Aurora Global Database (standby read replica ready)
→ Auto Scaling: min = 1, max = 20 (scale when needed)
→ Cost: $$$

SCENARIO C: RPO ≤ 1h, RTO ≤ 4h  
→ Architecture: Pilot Light
→ RDS automated backup (snapshots) + fast restore
→ EC2 AMIs ready, launched when needed
→ Cost: $$

SCENARIO D: RPO ≤ 24h, RTO can be days
→ Architecture: Backup & Restore
→ S3 + Glacier for backups
→ Manual restore scripts
→ Cost: $

MNEMONIC:
RPO = "how much data can I LOSE?" (looks to the past)
RTO = "how long can I be DOWN?" (looks to the future)
```

---

## Case 2 — Distinguishing "Durability" from "Availability"

**Context:** The question intentionally confuses the two terms.

**The Difference in Practice:**
```
SCENARIO: S3 Standard
Durability: 99.999999999% (11 nines)
  → Probability of LOSING an object: 0.000000001% per year
  → With 10 million objects, you expect to lose < 1 object in 10,000 years
  → "Will my data always exist?"

Availability: 99.99% 
  → Allowed downtime: ~52 min/year
  → "Can I access my data right now?"

ANALOGY:
Bank vault: high DURABILITY (the money doesn't vanish)
            AVAILABILITY: may be closed on a holiday

TRICK QUESTION:
"Which S3 class offers the lowest durability?"
→ S3 One Zone-IA: 99.999999999% (same durability!)
  Difference: only 1 AZ → lower AVAILABILITY if the AZ fails
  Durability doesn't change (data replicated within 1 AZ)

ANSWER: "One Zone-IA has lower AVAILABILITY (not durability)"
```

---

## Case 3 — Applying the "Principle of Least Privilege" in IAM

**Context:** The question asks for the minimal IAM policy for a specific case.

**Scenario:** A Lambda function needs to read from 1 specific DynamoDB table and write logs to CloudWatch.

**Policy Analysis:**
```
WRONG — Excessive permission:
{
  "Effect": "Allow",
  "Action": "dynamodb:*",
  "Resource": "*"
}
→ DynamoDB:* = full access to ANY table
→ Violates the Principle of Least Privilege

WRONG — More specific but still broad:
{
  "Effect": "Allow",
  "Action": ["dynamodb:GetItem", "dynamodb:Scan", "dynamodb:Query"],
  "Resource": "*"  // ← still "*"
}
→ Allows reading ALL tables

CORRECT — Minimum necessary:
{
  "Effect": "Allow",
  "Action": ["dynamodb:GetItem", "dynamodb:Query"],
  "Resource": "arn:aws:dynamodb:us-east-1:123456789:table/MinhaTabela"
}

For CloudWatch Logs (created automatically by Lambda if the role allows it):
{
  "Effect": "Allow",
  "Action": [
    "logs:CreateLogGroup",
    "logs:CreateLogStream",
    "logs:PutLogEvents"
  ],
  "Resource": "arn:aws:logs:us-east-1:123456789:log-group:/aws/lambda/MinhaFuncao:*"
}

NOTE: AWSLambdaBasicExecutionRole already includes basic logging — you can use the managed policy
      For DynamoDB: always specify the ARN of the specific table
```

---

## Case 4 — CIDR and VPC Subnets in Practice

**Context:** VPC questions often ask you to calculate available IPs or identify CIDR conflicts.

**Exercises:**
```
EXERCISE 1: How many usable IPs in a /24?
IPv4 /24 = 256 total addresses
AWS reserves 5 (network, GW, DNS+2, broadcast)
256 - 5 = 251 usable IPs

EXERCISE 2: Subnets for HA across 2 AZs with VPC 10.0.0.0/16:
VPC: 10.0.0.0/16 (65,536 addresses)
Subnet plan:
  Public AZ-a:  10.0.1.0/24 (251 usable)
  Public AZ-b:  10.0.2.0/24 (251 usable)
  Private AZ-a: 10.0.10.0/24 (251 usable)
  Private AZ-b: 10.0.11.0/24 (251 usable)
  DB AZ-a:      10.0.20.0/24 (251 usable)
  DB AZ-b:      10.0.21.0/24 (251 usable)

EXERCISE 3: VPC Peering — CIDR conflict:
VPC A: 10.0.0.0/16
VPC B: 10.0.0.0/24  ← CONFLICT! The /24 is inside the /16
→ VPC Peering does NOT work with overlapping CIDRs
Solution: use distinct CIDRs (e.g., VPC B: 172.16.0.0/16)

EXAM TRAP:
/28 = smallest subnet allowed on AWS
/28 = 16 total addresses, 16 - 5 = 11 usable
```

---

## Case 5 — Shared Responsibility Model in Real Scenarios

**Context:** The question asks who is responsible for security problems.

**Scenarios and Responsibilities:**
```
SCENARIO 1: EC2 with Windows Server was infected by ransomware
  CUSTOMER responsibility:
  ✓ Windows Server patches (customer)
  ✓ Antivirus installed on the OS (customer)
  ✓ Security Group configured correctly (customer)
  AWS responsibility:
  ✓ Isolated hypervisor (AWS)
  ✓ Physical hardware (AWS)
  → The CUSTOMER is responsible for the OS infection

SCENARIO 2: Physical disk failure on the server hosting EC2
  → AWS is responsible (physical hardware)
  → AWS replaces the hardware, EC2 migrates automatically (Live Migration)
  → Data on EBS is preserved (replicated within the AZ)

SCENARIO 3: S3 bucket accidentally made public
  → The CUSTOMER configured the wrong permission
  → AWS offered tools (Block Public Access, Macie, Config)
  → Customer chose not to use them → CUSTOMER responsibility

SCENARIO 4: RDS MySQL with an outdated OS
  → AWS manages the RDS OS → AWS is responsible for OS patches
  → Customer manages: data, database users, settings

SCENARIO 5: Lambda with code vulnerable to SQL Injection
  → The CUSTOMER wrote the code → CUSTOMER is responsible
  → AWS provides the (secure) execution platform
  → CODE security = developer's responsibility

MNEMONIC RULE:
AWS = "of the cloud" (hardware, physical network, hypervisor, OS of managed services)
Customer = "in the cloud" (EC2 OS, code, data, IAM config, virtual network)
```
