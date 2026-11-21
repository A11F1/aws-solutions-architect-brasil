# Real-World Use Cases — Multi-Service Architectures (Module 17)

## Case 1 — Multi-Tenant SaaS Platform with Tier-Based Isolation

**Context:** A B2B software company offers a management platform to customers of 3 sizes: Free (thousands of customers), Business (500 customers), Enterprise (50 large contracts). Each tier requires a different level of isolation and performance guarantee.

**Hybrid Architecture (Pool + Silo):**
```
ROUTING LAYER:
Route 53 → ALB → API Gateway

Per-Tenant Routing Logic:
├── ENTERPRISE (silo): dedicated stack
│     ALB → dedicated ECS → dedicated Aurora (no sharing)
│     DDoS: individual Shield Advanced
│     SLA: 99.99%
│
├── BUSINESS (bridge): shared infra, separate DB
│     ALB → shared ECS → Aurora Schema per tenant
│     Compute: shared, Storage: isolated
│     SLA: 99.9%
│
└── FREE (pool): everything shared
      ALB → shared ECS → DynamoDB (tenantId = partition key)
      SLA: 99%

TENANT IDENTIFICATION:
JWT token → API GW Authorizer (Lambda)
Lambda extracts tenantId → injects into X-Tenant-ID header
ECS app uses X-Tenant-ID to route to the correct queue / correct DB

NOISY NEIGHBOR PROTECTION (Free tier):
SQS per tenant-tier: Business has a queue separate from Free
Lambda reserved concurrency: Business = 100, Free = 20
DynamoDB: on-demand (scales per tenant without affecting others)
```

---

## Case 2 — Strangler Fig Migration of a Java Monolith to Microservices

**Context:** A legacy ERP system (monolithic Java, 15 years old) needs to be modernized gradually. Rewriting everything at once is not possible — high risk and prohibitive cost. Strategy: extract modules one by one with no downtime.

**Strangler Fig Process:**
```
INITIAL STATE:
Everything in the Java monolith:
CLI → Java Server → PostgreSQL Oracle

PHASE 1 — INSERT PROXY (Strangler):
CLI → API GW / ALB ← NEW ENTRY POINT
        ├── /api/estoque  → Java Monolith (legacy)
        ├── /api/clientes → Java Monolith (legacy)
        ├── /api/pedidos  → Java Monolith (legacy)
        └── /api/relatorio→ Java Monolith (legacy)

PHASE 2 — EXTRACT THE INVENTORY MODULE:
← Develop the inventory microservice in parallel
← Integration tests with real data
ALB Route: /api/estoque → NEW Lambda + DynamoDB
           (other /api/* still go to the monolith)

PHASE 3 — EXTRACT THE ORDERS MODULE:
/api/pedidos → NEW ECS Service + Aurora PostgreSQL
/api/estoque → Lambda + DynamoDB (already migrated)
/api/clientes, /api/relatorio → Monolith still

FINAL PHASE — "KILL" THE MONOLITH:
When the last module is extracted:
ALB has no more routes to the monolith
Java monolith decommissioned
100% microservices on AWS

ANTI-PATTERN AVOIDED:
❌ "Big Bang" — rewriting everything before going to production
✓ Incremental — each extraction validated in production before the next
```

---

## Case 3 — Order Processing Pipeline with the Saga Pattern

**Context:** An online marketplace with 3 independent microservices: Orders, Inventory, and Payment. When an order is created, all 3 must be consistent — if payment fails, inventory must be released.

**Choreographed Saga (Event-Driven):**
```
HAPPY PATH:
Order created → SQS (OrderCreated)
                     │
              Lambda InventoryService
              ├── Reserves inventory
              └── SQS (InventoryReserved)
                            │
                     Lambda PaymentService  
                     ├── Charges payment
                     └── SQS (PaymentApproved)
                                    │
                             Lambda OrderService
                             └── Confirms order (status = CONFIRMED)
                             └── SNS (OrderConfirmed) → notifies user

COMPENSATION (Rollback):
PaymentService fails → SQS (PaymentRejected)
                              │
                       Lambda InventoryService
                       └── Releases inventory (compensation)
                       └── SQS (InventoryReleased)
                                   │
                            Lambda OrderService
                            └── Cancels order (status = CANCELED)
                            └── SNS → notifies user
```

**Orchestrated Saga (Step Functions):**
```
Step Functions Express Workflow:
State 1: Task (ReserveInventory) → retry 3x, catch → CancelOrder
State 2: Task (ProcessPayment) → retry 2x, catch → ReleaseInventory → CancelOrder
State 3: Task (ConfirmOrder)
State 4: Task (NotifyUser)

Compensation (Catch):
ReleaseInventory → CancelOrder → NotifyFailure

Advantage vs Choreography:
- Full visibility of state at each step
- Configurable retry per step
- Easy debugging (X-Ray integrated)
```

---

## Case 4 — Event Sourcing for a Financial Audit System

**Context:** An investment bank needs a complete and immutable history of all changes to accounts and portfolios. Regulation requires 10-year traceability + the ability to "replay" for audits.

**Event Sourcing Architecture:**
```
WRITE MODEL:
App → API GW → Lambda (command handler)
                    │ Does NOT update state directly
                    │ WRITES an immutable EVENT
                    ▼
              DynamoDB (event store — events table)
              ├── PK: aggregateId (e.g.: account-123)
              ├── SK: version (1, 2, 3...)
              ├── eventType: "BalanceDebited", "BalanceCredited"
              ├── payload: {amount: 5000, currency: "BRL"}
              └── timestamp: ISO 8601
              
DynamoDB Streams → Lambda (projections updater)
                        └── ElastiCache (current balance — fast reads)
                        └── Athena/Redshift (analytical reports)

READ MODEL (CQRS):
App → API GW → Lambda (query handler)
                    └── ElastiCache (fast read of current state)

REPLAY FOR AUDIT:
Query: SELECT * FROM event_store WHERE accountId = 'conta-123' ORDER BY version
      → Replay all events → recompute state at any point in time
      
S3 (event store backup, compacted monthly, Object Lock 10 years)
```

---

## Case 5 — Live Streaming Platform with IVS + Lambda

**Context:** An online education platform wants to offer live classes with very low latency (< 5 seconds), real-time chat, polls during the broadcast, and automatic recording for later replay.

**Architecture:**
```
BROADCAST:
Teacher → OBS/Encoder (RTMP) → Amazon IVS (Interactive Video Service)
                                        │
                                  CloudFront CDN (global delivery)
                                        │
                                  Student players (< 5s latency)

REAL-TIME CHAT:
Student → API GW WebSocket
             ├── Lambda (onConnect) → DynamoDB (connectionIds)
             ├── Lambda (sendMessage) → DynamoDB + broadcast
             └── Lambda (onDisconnect) → removes connectionId
             
Broadcast to everyone in the room:
Lambda → API GW Management API → POST /connections/{connectionId}

POLLS:
Teacher → REST API → Lambda → DynamoDB (poll + answers)
IVS Timed Metadata event → Player displays the poll synchronized with the video

RECORDING AND REPLAY:
IVS → automatic S3 (full recording)
     └── MediaConvert (transcoding to multiple resolutions)
            └── S3 (adaptive HLS)
                └── CloudFront (CDN for replay)

METRICS:
IVS → CloudWatch (viewers, bitrate, buffering ratio)
Custom: Lambda → CloudWatch Metrics (messages/min, poll answers)
```

**IVS vs MediaLive:**
| Criterion | IVS | MediaLive |
|---------|-----|-----------|
| Latency | < 5s (ultra-low latency) | 15-30s (standard) |
| Configuration | Low (managed) | High (many parameters) |
| Interactivity | Native (timed metadata, chat) | Limited |
| Cost | Per hour + GB | Per hour (more expensive at scale) |
| When to use | Simple interactive lives | Broadcast TV broadcasting |
