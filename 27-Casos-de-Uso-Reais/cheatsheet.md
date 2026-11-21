# Cheatsheet — Real-World Use Cases (Module 17)

## Fundamental Architecture Patterns

| Pattern | Problem Solved | Key AWS Services | Exam Clue |
|--------|-------------------|-------------------|----------------|
| **Fan-Out** | 1 event → multiple processors | SNS → SQS (N queues) | "process in parallel by multiple systems" |
| **CQRS** | Reads and writes with distinct models | DynamoDB + ElastiCache / Aurora + Athena | "optimize reads without impacting writes" |
| **Event Sourcing** | Immutable state history | Kinesis / DynamoDB Streams | "complete audit trail / event replay" |
| **Strangler Fig** | Incremental monolith migration | ALB + Target Groups / API GW | "migrate with no downtime, route by route" |
| **Cache-Aside** | Reduce read latency | ElastiCache (Redis/Memcached) | "cache miss → DB, cache hit → fast" |
| **Saga** | Distributed transaction (rollback) | Step Functions + SQS + Lambda | "coordinate transactions across microservices" |
| **Bulkhead** | Isolate failures between components | Separate SQS queues per priority/tenant | "failure in one does not affect another" |
| **Circuit Breaker** | Avoid cascading failures | Lambda + DLQ + CloudWatch Alarm | "stop calls when downstream fails" |

---

## When to Use Each Integration Type

| Integration | When to Use | Service | Latency |
|------------|-------------|---------|----------|
| **Synchronous** | Immediate response needed | API GW → Lambda, ALB | < 30s |
| **Simple asynchronous** | Decoupling, fault tolerance | SQS → Lambda | seconds-minutes |
| **Fan-out** | 1 event → N consumers | SNS → multiple SQS | seconds |
| **Streaming** | High volume, real time | Kinesis Data Streams | ms-seconds |
| **Scheduled** | Periodic tasks | EventBridge Scheduler → Lambda | hourly |
| **Workflow** | Multi-step orchestration with retry | Step Functions | ms-hours |
| **Global Pub/Sub** | Multiple Regions, multiple protocols | SNS cross-region | seconds |

---

## Multi-Region: Active-Active vs Active-Passive

| Criterion | Active-Active | Active-Passive |
|----------|--------------|----------------|
| **Goal** | No loss of availability | Lower standby cost |
| **RPO** | ~0 | Minutes (depends on replication) |
| **RTO** | ~0 (automatic failover) | 1–10 min |
| **Cost** | 2x (double infrastructure) | ~1.2x (smaller standby) |
| **Complexity** | High (data conflicts) | Low |
| **Route 53 policy** | Latency / Geolocation | Failover |
| **Database** | Aurora Global (writer in both) | Aurora Global (1 writer) |
| **Use case** | Global SaaS, 24/7 finance | Corporate DR, regional backup |

---

## Architectures by Vertical

### High-Availability E-commerce
```
Customer → CloudFront → ALB (Multi-AZ)
                          ├── EC2 Auto Scaling (app)
                          └── ElastiCache (cart)
                               └── Aurora Multi-AZ (orders)
                                    └── DynamoDB (catalog/inventory)
SNS → SQS → Lambda (notifications)
EventBridge → Lambda (async post-order processing)
```

### IoT Pipeline
```
Devices → IoT Core (MQTT)
                   ├── IoT Rules → Kinesis Data Streams → Lambda (real-time)
                   ├── IoT Rules → S3 (raw data) → Glue → Athena (batch)
                   └── IoT Rules → DynamoDB (current device state)
CloudWatch → SNS (threshold alerts)
```

### Video Streaming Platform
```
Upload → S3 → Lambda → MediaConvert (transcoding)
                             └── S3 (multiple resolutions)
                                  └── CloudFront (global CDN)
DynamoDB (video metadata)
Cognito (user authentication)
```

### Modern Data Lake
```
Data → S3 (raw zone) → Glue ETL → S3 (processed zone)
                                         └── Athena (ad hoc query)
                                         └── Redshift Spectrum (BI)
Lake Formation (table/column-level access control)
Glue Data Catalog (centralized metadata)
```

---

## Multi-Tenant SaaS on AWS

| Strategy | Isolation | Cost | When to Use |
|------------|-----------|-------|-------------|
| **Silo** (per tenant) | Maximum (dedicated stack) | High | Compliance, large contracts |
| **Pool** (shared) | Logical (tag/ID) | Low | Many small tenants |
| **Hybrid** | Medium (tier-based) | Medium | Mix of premium/free customers |
| | | | |
| **DynamoDB** | Partition key = tenantId | Low | Data with a uniform pattern |
| **RDS** | Schema per tenant | Medium | SQL, mid-size tenants |
| **Aurora** | Database per tenant | High | Large enterprise customers |

---

## Security Patterns by Scenario

| Scenario | Correct Pattern | Services |
|---------|---------------|---------|
| Public API with auth | JWT/OAuth2 | Cognito User Pool + API GW |
| B2B API (company-to-company) | mTLS / API Key | API GW + ACM |
| Cross-account access | IAM Role + AssumeRole | STS |
| Secret in Lambda | Secrets Manager (not env var) | Secrets Manager + IAM |
| S3 data encryption | SSE-KMS (CMK) | KMS + S3 |
| Layer 7 DDoS protection | WAF + Shield Advanced | WAF + CloudFront |
| Audit who did what | CloudTrail + Athena query | CloudTrail + S3 + Athena |

---

## HIPAA / Compliance — Exam Patterns

| Requirement | AWS Solution |
|-----------|------------|
| PHI data at rest | KMS CMK (customer-managed) |
| PHI data in transit | TLS 1.2+ (ACM) |
| Immutable logs | CloudTrail + S3 Object Lock (WORM) |
| Least access necessary | Granular IAM + SCPs |
| BAA (Business Associate Agreement) | Sign with AWS for eligible services |
| Network isolation | VPC + Security Groups + NACLs |

---

## Dead Letter Queue (DLQ) — Fault Tolerance Pattern

```
SQS → Lambda (processes)
   ↓ maxReceiveCount exceeded
  DLQ (separate SQS) → CloudWatch Alarm → SNS → Email/PagerDuty
                               ↓
                         Manual analysis / analysis Lambda
```

- **maxReceiveCount:** 3–5 attempts before moving to the DLQ
- **SQS DLQ:** another SQS queue
- **Async Lambda DLQ:** SQS or SNS
- **Exam:** if the question mentions "reprocess failed messages" → DLQ

---

## Exam Tips

| Clue in the Question | Expected Answer |
|-----------------|------------------|
| "process in parallel by multiple consumers" | SNS → multiple SQS (fan-out) |
| "decouple producer from consumer" | SQS (FIFO if order matters) |
| "orchestrate a sequence of steps with retry/compensation" | Step Functions |
| "incremental migration without taking down the legacy system" | Strangler Fig (ALB rules) |
| "complete change history / replay" | Event Sourcing (Kinesis/DynamoDB Streams) |
| "reduce DB read latency" | ElastiCache (Cache-Aside) |
| "isolate failure of an upstream service" | SQS + DLQ + Circuit Breaker |
| "RTO ≈ 0, RPO ≈ 0, global" | Active-Active + Aurora Global + Route 53 Latency |
| "real-time IoT telemetry data" | IoT Core → Kinesis → Lambda |
| "multiple tenants, low cost" | Pool strategy (shared infra, tenantId partition key) |
| "notify several systems after an order event" | SNS topic with multiple subscriptions |
| "transaction across microservices with rollback" | Saga Pattern (Step Functions) |
