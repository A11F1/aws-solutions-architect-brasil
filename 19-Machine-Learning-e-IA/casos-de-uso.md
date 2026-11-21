# Real-World Use Cases — Machine Learning and AI (Module 13)

## Case 1 — Product Recommendation System

**Context:** An e-commerce company with 10 million users needs real-time personalized recommendations on the product page ("customers who bought X also bought Y") and in segment-personalized email campaigns.

**Requirements:**
- Real-time recommendations (< 200ms per request)
- Model retrained daily with new behavior data
- No dedicated ML team required — use a managed service
- Segmentation for batch marketing campaigns

**Architecture:**
```
INPUT DATA:
Clickstream → Kinesis → S3 (user-product interactions)
Orders      → S3 (purchase history)

TRAINING (daily — batch):
S3 (interactions) → Amazon Personalize
                  ├── Dataset Group
                  ├── Solution (USER_PERSONALIZATION algorithm)
                  └── Campaign (inference endpoint)

INFERENCE (real time):
App → Amazon Personalize Campaign → JSON recommendations
App → ElastiCache (cache result by user_id, TTL 1h)

BATCH (email campaigns):
EventBridge (daily) → Lambda → Personalize Batch Inference
                                    └── S3 (recommendations by segment)
                                              └── SES (personalized emails)
```

**Personalize vs SageMaker for this case:**
| Criterion | Personalize | SageMaker |
|---------|-------------|-----------|
| ML team required | No | Yes |
| Available algorithms | Recommendation/ranking | Any |
| Setup time | Hours | Days/weeks |
| Flexibility | Low | High |
| Cost | Per TPS + training hours | Per instance |

---

## Case 2 — Financial Document Analysis with Textract + Comprehend

**Context:** An insurance company needs to process 5,000 documents/day (claim forms, medical reports, invoices) to speed up policy analysis. The manual process takes 3 days per claim.

**Requirements:**
- Extract structured data from forms (fields, tables)
- Detect PII (national ID number, name, date of birth) for protection
- Classify document type (claim/report/invoice)
- Verify consistency of the extracted data

**Architecture:**
```
Documents (PDF/image) → S3 (claims-input)
                                │ S3 Event Notification
                                ▼
                           Lambda (orchestrator)
                           ├── Textract StartDocumentAnalysis (async)
                           │     (extracts forms + tables)
                           │     └── SNS callback → Lambda (result)
                           ├── Comprehend DetectPiiEntities
                           │     (detects ID number, name, date)
                           ├── Comprehend ClassifyDocument
                           │     (type: claim/report/invoice)
                           └── Comprehend DetectSentiment
                                 (customer urgency in free text)
                                │
                           DynamoDB (metadata + extracted fields)
                           S3 (structured JSON result)
                           Step Functions (human approval flow if confidence < 80%)
                           
A2I (Augmented AI) → Human review when Textract has low confidence
```

**Time Reduction:**
- Before: 3 days (manual) → After: 2 hours (automated)
- Automation rate: 85% of documents without human review
- A2I covers the 15% with low confidence

---

## Case 3 — Customer Service Chatbot with Lex v2 + Lambda

**Context:** A telecommunications company wants to reduce the volume of calls (200,000/month) to customer service by offering a chatbot that resolves the 5 most common problems: check invoice, reissue payment slip, cancel services, check data balance, change plan.

**Requirements:**
- Understand natural language in Portuguese
- Authenticate the user before sensitive operations
- Escalate complex calls to a human agent
- Integrate with the existing CRM system (REST API)

**Architecture:**
```
User (WhatsApp / App / Web)
       │
       ▼
Amazon Lex v2 (Bot in Portuguese)
  Intents:
  ├── ConsultarFatura → Lambda (queries CRM API)
  ├── EmitirBoleto    → Lambda (generates payment slip PDF → S3 → URL)
  ├── CancelarServico → Lambda (checks contract + calls CRM)
  ├── VerificarDados  → Lambda (queries telemetry system)
  └── AlterarPlano    → Lambda (plan options + Step Functions workflow)
  
  Authentication:
  ├── Slot: national ID + date of birth
  └── Lambda validator → queries DynamoDB (authenticated session)
  
  Fallback:
  └── Intent: TransferirAtendente → Amazon Connect (transfer)
  
Amazon Connect (integrated call center)
CloudWatch (metrics: intent recognition rate, transfer rate)
```

**Success Metrics:**
| KPI | Before | After |
|-----|-------|--------|
| Resolution without an agent | 0% | 65% |
| Average resolution time | 8 min | 90 sec |
| Cost per interaction | R$15 | R$0.12 |
| Satisfaction (CSAT) | 3.2/5 | 4.1/5 |

---

## Case 4 — Content Moderation with Rekognition

**Context:** A UGC (User Generated Content) platform with 500,000 image/video uploads per day needs to automatically remove inappropriate content (violence, nudity, drugs) before publishing.

**Requirements:**
- Analyze 100% of content before publishing
- False positives < 5% (valid content blocked)
- Human review for borderline cases (50-80% confidence)
- Audit trail of decisions (legal compliance)

**Architecture:**
```
User upload → S3 (content-pending)
                      │ Lambda trigger
                      ▼
                 Lambda (moderator)
                 ├── Rekognition DetectModerationLabels
                 │     (nudity, violence, drugs, etc.)
                 │
                 ├── If score > 80% (inappropriate content):
                 │     └── Move to S3 (content-blocked)
                 │         DynamoDB (log: blocked + reason)
                 │         SNS → Email to user (notification)
                 │
                 ├── If score 20-80% (uncertain):
                 │     └── A2I (Augmented AI) → Human review queue
                 │         Step Functions waits for human decision
                 │
                 └── If score < 20% (approved):
                       └── S3 (content-approved) → CloudFront

Audit: DynamoDB Streams → Lambda → S3 (immutable compliance log)
```

**Rekognition for Video:**
```
Video upload → S3 → Lambda → Rekognition StartContentModeration (async)
                                    │ SNS callback when complete
                                    ▼
                               Lambda (process result)
                               ├── Timestamps of inappropriate scenes
                               └── Decision: approve/reject/segment
```

---

## Case 5 — Demand Forecasting for Inventory Management

**Context:** A supermarket chain needs to forecast demand for 50,000 SKUs over the next 4 weeks to optimize replenishment orders and reduce waste.

**Requirements:**
- Forecast per SKU + store + week
- Account for seasonality, holidays, promotions
- Retrain the model monthly with new data
- Integrate forecasts with the ERP purchasing system

**Architecture:**
```
HISTORICAL DATA:
S3 (daily sales for the last 3 years, by SKU/store)
S3 (external data: holidays, promotions, weather)

AMAZON FORECAST:
├── Dataset Group (sales + metadata + related items)
├── Predictor (algorithm: AutoML selects the best)
│     (options: DeepAR+, Prophet, NPTS, CNN-QR)
└── Forecast (generates P10/P50/P90 for 28 days)

INFERENCE + INTEGRATION:
EventBridge (1x/week) → Lambda
     │
     ├── CreateForecastExportJob → S3 (forecasts.csv)
     ├── Lambda processes CSV → DynamoDB (suggested_purchases table)
     └── ERP REST API → reads DynamoDB → generates purchase orders

QuickSight (dashboard: forecast vs actual, MAPE accuracy)
SNS → Alert if MAPE > 15% (model degrading)
```

**Accuracy Metrics:**
| Metric | Typical Value | What it means |
|---------|-------------|----------------|
| MAPE | < 15% | Mean Absolute Percentage Error |
| WAPE | < 10% | Weighted APE (accounts for volume) |
| P50 | median forecast | 50% confidence |
| P90 | optimistic bound | For safety stock |
