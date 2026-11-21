# Cheatsheet — Module 14: Monitoring, CloudWatch, and CloudTrail

## Overview of Monitoring Services

| Service | Answers | Best For |
|---|---|---|
| **CloudWatch Metrics** | "How is performance?" | Resource and application metrics |
| **CloudWatch Logs** | "What happened?" | App, Lambda, VPC Flow logs |
| **CloudTrail** | "Who did what?" | API call auditing |
| **AWS Config** | "Is it configured correctly?" | Compliance and configuration |
| **X-Ray** | "Where is the bottleneck?" | Distributed tracing |
| **GuardDuty** | "Is there suspicious activity?" | ML threat detection |
| **Security Hub** | "What is the security posture?" | Findings consolidation |
| **Trusted Advisor** | "What should be improved?" | Best-practice recommendations |

---

## CloudWatch — Important Metrics by Service

| Service | Built-in Metrics | Metrics Requiring the Agent |
|---|---|---|
| EC2 | CPUUtilization, NetworkIn/Out, DiskReadOps | **MemoryUtilization**, DiskSpaceUtilization |
| ELB | RequestCount, TargetResponseTime, HTTPCode_5XX | — |
| RDS | CPUUtilization, FreeStorageSpace, DatabaseConnections | — |
| Lambda | Invocations, Duration, Errors, Throttles, ConcurrentExecutions | — |
| API GW | Count, Latency, 4XXError, 5XXError | — |
| SQS | ApproximateNumberOfMessagesVisible, NumberOfMessagesSent | — |
| DynamoDB | ConsumedReadCapacityUnits, SystemErrors | — |

**EC2 Memory and Disk:** only via the CloudWatch Agent (installed on the instance).

---

## CloudWatch Alarms — States

| State | Meaning |
|---|---|
| **OK** | Metric within the threshold |
| **ALARM** | Metric violated the threshold for the configured period |
| **INSUFFICIENT_DATA** | Not enough data to evaluate (startup or no data) |

**Alarm Actions:** EC2 Action (reboot, stop, terminate, recover), Auto Scaling Action, SNS notification.

---

## CloudTrail — Event Types

| Type | What It Records | Enabled by Default | Extra Cost |
|---|---|---|---|
| **Management Events** | Create/delete resources, IAM changes, console login | **Yes** | No (1 free copy) |
| **Data Events** | S3 GetObject/PutObject, Lambda Invoke | **No** | Yes |
| **Insights Events** | Anomalous API activity | **No** | Yes |

**Retention in the console:** 90 days. For long term: enable a trail with an S3 destination (unlimited).

---

## AWS Config — Rule Types

| Type | Who Creates It | Examples |
|---|---|---|
| **Managed Rules** | AWS (~200 available) | `s3-bucket-public-read-prohibited`, `ec2-instance-no-public-ip`, `rds-storage-encrypted` |
| **Custom Rules (Lambda)** | You | Custom compliance logic |
| **Proactive Rules** | AWS (new) | Evaluates before creation (CloudFormation hook) |
| **Detective Rules** | AWS/Custom | Evaluates after create/change |

**Remediation:** SSM Automation Document executed automatically when NON_COMPLIANT.

---

## SSM Parameter Store — Hierarchy and Types

| Parameter Type | Encryption | Use |
|---|---|---|
| **String** | No | Non-secret settings |
| **StringList** | No | List of values (e.g., AMI IDs per region) |
| **SecureString** | KMS | Secrets, passwords, tokens |

**Hierarchy:** `/app/prod/database/host`, `/app/prod/database/password` — allows IAM policy by hierarchy.

| Tier | Limit | Cost | Max Size |
|---|---|---|---|
| Standard | 10,000 parameters | Free | 4 KB |
| Advanced | 100,000 parameters | $0.05/param/month | 8 KB |

---

## X-Ray — Key Concepts

| Concept | Definition |
|---|---|
| **Trace** | Set of segments representing an end-to-end request |
| **Segment** | Data from a single service/resource in the trace |
| **Subsegment** | Granularity within a segment (database calls, external HTTP) |
| **Service Map** | Graphical visualization of services + latencies + errors |
| **Annotations** | Indexed key-values to filter traces (e.g., `UserID`, `OrderID`) |
| **Metadata** | Non-indexed key-values for additional debug data |
| **Sampling Rule** | % of requests to trace; default 5% + 1/s |

**X-Ray Daemon:** lightweight process installed on EC2/ECS that collects and sends traces to the X-Ray API.

---

## CloudWatch vs CloudTrail vs Config — Decision

```
Question: "Who deleted the S3 bucket?"
→ CloudTrail (API call audit: who + when + from where)

Question: "Was the S3 bucket public yesterday?"
→ AWS Config (resource configuration history)

Question: "Is my API slow?"
→ CloudWatch (API GW latency metrics)

Question: "Where is the bottleneck in my microservices architecture?"
→ AWS X-Ray (distributed tracing)

Question: "Which instance is using the most CPU?"
→ CloudWatch (Top Contributor via Contributor Insights)

Question: "Is there a suspicious login from an unusual IP?"
→ GuardDuty (ML-based threats)
```

---

## Exam Tips — Common Patterns

| Clue in the Question | Likely Answer |
|---|---|
| "EC2 memory in CloudWatch" | Install the CloudWatch Agent |
| "Immutable CloudTrail audit log" | S3 Object Lock + Log File Validation |
| "Centralized multi-account compliance" | Config Aggregator + Organization |
| "Access EC2 without SSH, without bastion" | SSM Session Manager |
| "Auto-remediate non-compliant Config" | Config Rule + SSM Automation |
| "High latency in microservices" | X-Ray + Service Map |
| "Alert when Lambda has > 5 errors/min" | CloudWatch Alarm on the Errors metric |
| "Retain CloudTrail logs for 7 years" | S3 Lifecycle + Glacier Deep Archive |
| "Alert when an API call without MFA accesses sensitive S3" | Config Rule or CloudTrail + EventBridge |
| "Detect EC2 doing suspicious port scans" | GuardDuty |
| "Password/access key rotated automatically" | Secrets Manager (not Parameter Store) |
| "Detect AWS credentials exposed on GitHub" | GuardDuty FindingType: UnauthorizedAccess |
