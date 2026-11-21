# Cheatsheet — Module 11: Serverless, Lambda and API Gateway

## Critical Lambda Limits

| Parameter | Value |
|---|---|
| Maximum timeout | 15 minutes |
| Memory | 128 MB to 10,240 MB |
| Synchronous payload (request/response) | 6 MB each |
| Asynchronous payload | 256 KB |
| Package size (zip) | 50 MB (zip) / 250 MB (unzipped) |
| Concurrent executions per account/Region | 1,000 (default, can be increased) |
| Maximum layers per function | 5 |
| Environment variables | 4 KB total |
| Tmp storage (/tmp) | 512 MB to 10,240 MB |

---

## Lambda Invocation Types

| Type | Used By | Retry | DLQ |
|---|---|---|---|
| **Synchronous** | API GW, ALB, SDK, CLI | None (client retries) | N/A |
| **Asynchronous** | S3, SNS, EventBridge, SES | 2 automatic retries | SQS/SNS via Destinations |
| **Event Source Mapping** | SQS, Kinesis, DynamoDB Streams, MSK | Retry until expiry (Kinesis) or maxReceiveCount (SQS) | DLQ configured on the queue |

---

## Lambda Concurrency

| Type | Description | Cost |
|---|---|---|
| **On-demand** | Scales automatically with demand | Standard (pay-per-use) |
| **Reserved** | Guarantees exclusive capacity for the function; caps it at the defined maximum | Standard |
| **Provisioned** | Pre-warms instances (eliminates cold start) | $0.015/provisioned GB-hour |

**Throttling:** if `concurrent executions > reserved/account limit` → **HTTP 429** (sync) or the event goes to a retry queue (async).

---

## API Gateway — API Types

| Type | Features | Cost | Best For |
|---|---|---|---|
| **REST API** | Stages, caching, usage plans, API keys, full transform | Medium | Full production APIs |
| **HTTP API** | Native JWT/OAuth, no mapping templates | ~70% cheaper | Simple microservices |
| **WebSocket API** | Persistent bidirectional connections | Per connection + message | Chat, real-time updates |

---

## API Gateway Throttling

| Limit | Default Value |
|---|---|
| Request rate per account | 10,000 req/s |
| Burst | 5,000 concurrent requests |
| HTTP 429 | Too Many Requests (when exceeded) |

---

## API Gateway Integrations

| Type | Description |
|---|---|
| **Lambda Proxy** | Full event to Lambda; Lambda returns statusCode + headers + body |
| **Lambda Custom** | Mapping templates (VTL) transform the request/response |
| **HTTP Proxy** | Forwards the HTTP request without transformation |
| **AWS Service** | Calls the AWS API directly (e.g., SQS SendMessage without Lambda) |
| **Mock** | Returns a static response (for testing, CORS OPTIONS) |
| **VPC Link** | Accesses an NLB in a private VPC |

---

## API Gateway Authorizers

| Type | How It Works |
|---|---|
| **Lambda Authorizer Token** | Token in header → Lambda evaluates → returns IAM Policy (ALLOW/DENY) |
| **Lambda Authorizer Request** | Headers + query params → Lambda → IAM Policy; flexible |
| **Cognito User Pools** | Cognito JWT token validated natively by API GW |

Result cache: up to **3600 seconds** (configurable, default 300s).

---

## Lambda Execution Environment Lifecycle

```
INIT phase     → download code, start runtime, execute init code (outside handler)
INVOKE phase   → execute handler function
SHUTDOWN phase → cleanup (response to SIGTERM)
```

**Cold Start** occurs on: first invocation, after an idle period, a new deployment, and concurrency increases beyond the warm instances.

---

## Lambda + VPC — Connectivity Diagram

```
┌─────────────────────────────────────────────────────────────┐
│ AWS CLOUD                                                   │
│  ┌─────────────────┐     ┌─────────────────────────────┐   │
│  │   Lambda        │     │   VPC                       │   │
│  │   (hyperplane)  │─ENI─│  ┌──────────┐ ┌──────────┐ │   │
│  │                 │     │  │ Private  │ │ RDS / EC2│ │   │
│  └─────────────────┘     │  │ Subnet   │ │          │ │   │
│                          │  └──────────┘ └──────────┘ │   │
│                          │         ↓ NAT GW (if internet needed) │   │
│                          └─────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

**Without a NAT GW**: Lambda in a VPC has no internet access (SSM and DynamoDB need a VPC Endpoint or NAT GW).

---

## Lambda vs Other Compute — When to Use

| Scenario | Service |
|---|---|
| Event-driven, < 15 min, stateless | Lambda |
| Containerized, microservices, > 15 min | ECS Fargate |
| Needs GPU, HPC, specific software | EC2 |
| Long batch job (hours/days) | AWS Batch |
| Orchestrated multi-step workflow | Step Functions + Lambda |
| ML Training/Inference | SageMaker |

---

## Exam Tips — Common Patterns

| Clue in the Question | Likely Answer |
|---|---|
| "Cold start" + Java | Lambda SnapStart |
| "Consistent latency" serverless | Provisioned Concurrency |
| "Custom authentication" API GW | Lambda Authorizer |
| "JWT / OAuth" API GW | HTTP API + JWT Authorizer |
| "Lambda can't access RDS" | Check SG + VPC config + NAT GW |
| "Lambda needs 30 min" | Lambda does NOT fit → ECS Fargate or AWS Batch |
| "Gradual deploy 10% → 100%" | Lambda Alias with weights (canary) |
| "Multiple environments (dev/staging/prod)" | API GW Stages |
| "Share dependencies between functions" | Lambda Layers |
| "Lambda must not be invoked more than N times/s" | Reserved Concurrency |
