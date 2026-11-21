# Use Cases — Introduction to SAA-C03

## Case 1 — SaaS Platform Starting an Architectural Review

### Scenario
A B2B SaaS company already runs on AWS, but the team realized it knows how to operate services without being able to justify architectural decisions. Leadership wants to prepare two architects for the SAA-C03 and, at the same time, review the platform's current design.

### Reasoning Architecture

```text
Business -> Requirements -> Constraints -> Service choice -> Trade-off -> Operations
   |            |              |               |                |            |
   +------------+--------------+---------------+----------------+------------+
```

### Justified Technical Decisions
- Study should start with core services such as IAM, VPC, EC2, S3, RDS, and Route 53, because they appear in a large share of scenarios.
- The group needs to review not only services, but also patterns: HA, DR, decoupling, and total cost.
- The best way to internalize this is to combine theory, questions, and real-world cases.

### Trade-offs Considered
- Studying isolated services is fast, but provides little context.
- Studying by scenario is slower, but brings the candidate closer to the real exam format.

## Case 2 — E-commerce Torn Between Robustness and Simplicity

### Scenario
An online store wants to grow before Black Friday. The technical team proposes several highly sophisticated solutions, but management wants to know which approach is adequate without inflating cost and operations. The goal of the study is to learn to distinguish what is necessary from what is excessive.

### Reasoning Architecture

```text
Users -> ALB -> ASG EC2 -> Aurora Multi-AZ
            |         |
            |         +-> CloudWatch
            |
            +-> CloudFront -> Static S3
```

### Justified Technical Decisions
- ALB and ASG address elasticity with less effort than manually managing multiple instances.
- Aurora Multi-AZ provides regional continuity without requiring multi-region from the start.
- CloudFront reduces latency and offloads static content.

### Trade-offs Considered
- Active-active multi-region would be more resilient, but may be overkill if the real requirement is only regional HA.
- Dedicated instances can offer isolation, but increase cost without an explicit need.

## Case 3 — Data Startup on a Tight Budget

### Scenario
A startup needs to process application events at a controlled cost. The SAA-C03 exam would use this context to assess whether the candidate understands when to prioritize managed services and on-demand pricing.

### Reasoning Architecture

```text
Application -> EventBridge -> Lambda -> DynamoDB
                                |
                                +-> CloudWatch Logs
```

### Justified Technical Decisions
- Lambda reduces idle time and patching for variable workloads.
- EventBridge facilitates event-driven integration with low coupling.
- DynamoDB fits well when access is predictable by key and scale is horizontal.

### Trade-offs Considered
- EC2 can offer more control, but requires constant administration.
- A relational database may be better in complex transactional scenarios, but here the access pattern favors NoSQL.

## Case 4 — Regulated Company Focused on Governance

### Scenario
A financial organization needs traceability, least privilege, and controlled encryption keys. It wants exam preparation to reinforce real-world secure architecture best practices.

### Reasoning Architecture

```text
Users -> IAM Identity Center -> AWS Accounts
                       |
                       +-> IAM Roles / SCPs
                       +-> KMS
                       +-> CloudTrail
```

### Justified Technical Decisions
- IAM Identity Center simplifies centralized access.
- SCPs help limit what accounts can do at the organizational level.
- KMS and CloudTrail appear as pillars of encryption and auditing.

### Trade-offs Considered
- More centralized governance reduces risk, but requires careful permission design.
- Overly broad policies speed things up in the short term and harm compliance.
