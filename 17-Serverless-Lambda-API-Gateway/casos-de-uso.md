# Real-World Use Cases — Serverless, Lambda and API Gateway (Module 11)

## Case 1 — Serverless REST API for E-commerce

**Context:** An e-commerce startup needs an API for a product catalog and order creation. It expects traffic spikes on holidays and shopping events (Black Friday, Christmas). Limited budget — it doesn't want to pay for idle capacity.

**Requirements:**
- Automatic scaling without manual intervention
- Cost proportional to usage (no traffic = no cost)
- Latency < 500ms for product reads
- JWT authentication for order routes

**Architecture:**
```
Mobile/Web Client
       │
       ▼
  CloudFront (cache GET /products)
       │
       ▼
 API Gateway (REST)
  ├── GET /products     → Lambda (no auth) → DynamoDB (catalog)
  ├── GET /products/{id}→ Lambda (no auth) → DynamoDB
  ├── POST /orders      → Lambda (JWT auth) → DynamoDB + SNS
  └── GET /orders/{id}  → Lambda (JWT auth) → DynamoDB
           │
     Cognito Authorizer (validates JWT)
           │
     DynamoDB (orders, TTL on session items)
           │
     SNS → SQS → Lambda (process payment async)
```

**Design Decisions:**
| Decision | Discarded Alternative | Reason |
|---------|----------------------|--------|
| Lambda + DynamoDB | EC2 + RDS | No idle capacity; DynamoDB scales horizontally |
| Cognito Authorizer | Lambda Authorizer | Less operational work; Cognito manages JWT rotation |
| CloudFront in front | API GW only | Caching GETs reduces Lambda invocations |
| SNS → SQS async | Sync Lambda for payment | Decouples; payment can take a long time without timeout |

**Cost Estimate (1M req/month):**
- API Gateway: ~$3.50
- Lambda: ~$0.20 (Free Tier covers it)
- DynamoDB On-Demand: ~$1.25
- **Total: < $5/month** vs EC2 t3.small ($13/month idle)

---

## Case 2 — Image Processing Pipeline

**Context:** A social media platform needs to resize and apply filters to images uploaded by users. Volume of 50,000 uploads/day with peaks of 500 uploads/minute.

**Requirements:**
- Process images asynchronously (the user doesn't wait)
- Ensure no image is lost (at-least-once)
- Multiple outputs: 100px thumbnail, 600px medium, original
- Store results on a global CDN

**Architecture:**
```
User uploads
       │
       ▼
  S3 (bucket: original-uploads)
       │ S3 Event Notification
       ▼
  SQS (buffer — guarantees at-least-once)
       │
       ▼
  Lambda (processImage)
  ├── Reads image from S3
  ├── Resize with Pillow Layer
  ├── Writes thumbnail/medium/original to S3 (bucket: processed)
  └── Updates DynamoDB (status = "processed")
       │
       ▼
  CloudFront → S3 (bucket: processed)
  
  DLQ (SQS) → SNS → Email (failure alerts)
  CloudWatch → Alarm → SNS (Lambda errors > 10)
```

**Points of Attention:**
- Lambda Layer with the Pillow library (max 250 MB unzipped)
- Reserved concurrency to avoid S3 throttling
- SQS visibility timeout > maximum Lambda time (e.g., Lambda 30s → SQS timeout 35s)
- DLQ configured with maxReceiveCount=3

---

## Case 3 — Webhook Fanout with SNS → Lambda

**Context:** A SaaS platform needs to notify multiple systems when a business event occurs (e.g., "customer closed a contract"): CRM, ERP, billing system, email marketing.

**Requirements:**
- Event published once → multiple systems receive it
- Failure in one system doesn't affect the others
- Each system has its own processing pace

**Architecture:**
```
App (publishes event)
       │
       ▼
  SNS Topic: contract-closed
  ├── Subscriber 1: SQS → Lambda (update CRM)
  ├── Subscriber 2: SQS → Lambda (update ERP)
  ├── Subscriber 3: SQS → Lambda (generate billing)
  └── Subscriber 4: SQS → Lambda (trigger email marketing)

  Each SQS has its own DLQ
  Each Lambda has a separate CloudWatch Log Group
```

**Why SQS between SNS and Lambda (and not SNS → Lambda directly)?**
- SQS guarantees persistence if Lambda fails
- SQS allows concurrency control (Lambda doesn't get overloaded)
- SQS allows a DLQ per subscriber (a CRM failure doesn't contaminate the ERP)
- SNS → Lambda directly: event loss if Lambda fails (no native retry)

---

## Case 4 — GraphQL API with AppSync + Lambda

**Context:** A social network mobile app needs real-time subscriptions (feed notifications), optimized queries, and custom resolvers for business logic.

**Requirements:**
- Real-time updates (native WebSocket)
- Direct resolvers to DynamoDB (no Lambda for simple CRUD)
- Lambda resolvers only for complex logic (recommend friends)
- Authentication with a Cognito User Pool

**Architecture:**
```
Mobile App
    │ GraphQL (HTTP/WebSocket)
    ▼
AWS AppSync
    ├── Query getUserProfile → DynamoDB Resolver (direct)
    ├── Query getFeed       → DynamoDB Resolver (direct)
    ├── Mutation createPost → DynamoDB Resolver + Pipeline Resolver
    ├── Query recommendations → Lambda Resolver (ML logic)
    └── Subscription onNewPost → automatic WebSocket push
    
Cognito User Pool (AppSync authentication)
DynamoDB (users, posts, follows)
Lambda (recommendation engine)
```

**AppSync vs API Gateway:**
| Criterion | AppSync | API GW REST |
|---------|---------|------------|
| Real-time subscriptions | Native WebSocket | Requires a separate API GW WebSocket |
| Optimized queries | GraphQL (1 request) | REST multiple endpoints |
| Direct resolvers to DB | DynamoDB, Aurora, Dynamo | Always via Lambda |
| Complexity | Higher initial setup | Simple for pure REST |

---

## Case 5 — Batch Processing with Lambda and Step Functions

**Context:** A financial analytics company needs to process 1 million daily transactions by 6 AM. Each transaction goes through 3 sequential steps: validation, enrichment (external API lookup), and score calculation.

**Requirements:**
- Process everything in < 4 hours
- Automatic retry on step failure
- Final report of rejected transactions
- Real-time visibility of progress

**Architecture:**
```
EventBridge Scheduler (02:00 UTC daily)
       │
       ▼
  Step Functions (Express Workflow)
  ┌─────────────────────────────────┐
  │ Map State (processes in parallel)│
  │   ├── Lambda: Validate Transaction│
  │   │     (retry: 3x, backoff 2s)  │
  │   ├── Lambda: Enrich Data        │
  │   │     (retry: 5x, backoff 10s) │
  │   └── Lambda: Calculate Score    │
  │         (retry: 2x, backoff 1s)  │
  └─────────────────────────────────┘
       │
       ▼
  DynamoDB (calculated scores)
  S3 (JSON report of rejected items)
  SNS → Email (completion notification)
  CloudWatch Logs (details of each execution)
```

**Why Step Functions and not chained SQS?**
- Step Functions: state visibility, configurable retry per step, native Map State for parallelism
- Chained SQS: works, but without visibility of the pipeline state, and retry is hard to configure per step
- Lambda timeout 15min: enough for 1 transaction; Step Functions orchestrates the batch

**Cost for 1M transactions/day:**
- Step Functions Express: $1.00 per 1M state executions
- Lambda: ~$0.20 per 1M invocations
- DynamoDB: ~$1.25 write
- **Total: < $3/day**
