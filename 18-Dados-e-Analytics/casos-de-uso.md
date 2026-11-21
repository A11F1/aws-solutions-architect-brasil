# Real-World Use Cases — Data and Analytics (Module 12)

## Case 1 — Centralized Data Lake for a Retail Company

**Context:** A retail chain with 500 stores needs to centralize sales (POS), inventory (ERP), web behavior (clickstream), and external data (weather, holidays) for integrated analysis.

**Requirements:**
- Ingestion from multiple sources with different formats and frequencies
- Access isolation per department (finance can't see HR data)
- Ad hoc queries by analysts without provisioning clusters
- Cost proportional to the amount of data analyzed

**Architecture:**
```
DATA SOURCES:
POS (daily CSV batch) ───────────────────┐
ERP (nightly JSON batch) ────────────────┤ → S3 Raw Zone
Clickstream (real-time) → KDF ──────────┤    (raw data)
External APIs (scheduled Lambda) ────────┘
                                         │
                                    AWS Glue ETL
                                    (daily job)
                                         │
                                    S3 Processed Zone
                                    (Parquet + partitioned)
                                         │
                              ┌──────────┴──────────┐
                              │                     │
                           Athena                Redshift Spectrum
                        (ad hoc queries)         (BI dashboards)
                              │
                     Lake Formation
                     (column/table access control)
                     
Glue Data Catalog (centralized metadata)
CloudWatch → Glue Job alerts
```

**S3 Partitioning Strategy:**
```
s3://data-lake/processed/vendas/
  year=2024/
    month=01/
      day=15/
        loja=SP001/
          dados.parquet
```
→ Athena filters only the necessary partitions (reduces scan cost)

**Lake Formation Access Control:**
| Profile | Access |
|--------|--------|
| Financial Analyst | vendas.*, faturamento.* (no PII) |
| Marketing Analyst | clickstream.*, campanhas.* |
| Data Scientist | All tables (read-only) |
| ETL Service Role | All (raw read + processed write) |

---

## Case 2 — Real-Time Fraud Analysis Pipeline

**Context:** A fintech needs to detect fraudulent transactions in real time (<1 second) during payment processing.

**Requirements:**
- Analyze each transaction in < 500ms
- Enrich the transaction with the user's history
- Risk score via an ML model
- Store all transactions for future training

**Architecture:**
```
Payment App
     │ PUT record
     ▼
Kinesis Data Streams (shard per region)
     │ Lambda trigger (batch 100, window 1s)
     ▼
Lambda (fraud-scorer)
├── Reads history from DynamoDB (< 5ms with DAX)
├── Computes features (frequency, amount, geolocation)
├── Invokes SageMaker Endpoint (ML scoring)
└── If score > 0.8: publishes to SNS (fraud alert)
│
├── Kinesis Data Firehose (all transactions)
│        └── S3 (future training + compliance)
│
└── DynamoDB (updates user profile)

SNS → Lambda → block transaction + notify user
CloudWatch → Alarm (latency > 300ms or error rate > 1%)
```

**Kinesis vs SQS for this case:**
- **Kinesis:** multiple consumers of the same stream, ordering per shard, event replay
- **SQS:** 1 consumer per message, no guaranteed ordering (Standard), no replay

---

## Case 3 — Analytical Warehouse for Business Reports

**Context:** A logistics company needs performance dashboards (on-time deliveries, cost per route, driver productivity) for the board, with data refreshed every hour.

**Requirements:**
- Complex reports with JOINs across multiple tables
- Consolidated data from 5 years of history
- Refresh every 1 hour
- QuickSight for executive visualization

**Architecture:**
```
RDS Aurora (operational system)
     │ DMS (Change Data Capture - CDC)
     ▼
S3 (staging zone — Parquet)
     │ Redshift COPY (every 1h)
     ▼
Redshift Cluster (Warehousing)
├── Dimension tables: dim_motorista, dim_rota, dim_veiculo
├── Fact tables: fato_entrega, fato_custo
└── Materialized Views (pre-computed KPIs)
     │
     ▼
Amazon QuickSight (board dashboards)
     │
Redshift Spectrum → S3 (query historical data >2 years without moving it into Redshift)
```

**Redshift Distribution:**
| Table | Distribution Key | Sort Key |
|--------|-----------------|---------|
| fato_entrega | motorista_id (frequent JOIN) | data_entrega |
| fato_custo | rota_id | data |
| dim_motorista | ALL (small table) | — |

---

## Case 4 — Application Log Analysis with Athena

**Context:** A SaaS startup needs to analyze API logs (errors, latency, active users) without maintaining an Elasticsearch cluster. Volume: 10 GB of logs/day.

**Requirements:**
- Analysis of logs from the last 90 days
- Queries by date, endpoint, error code
- Controlled cost (Elasticsearch cluster ~$500/month)
- Error anomaly alerts

**Architecture:**
```
ALB Access Logs → S3 (raw-logs/alb/year=../month=../day=..)
CloudWatch Logs → Kinesis Data Firehose → Parquet Conversion → S3 (processed-logs/)
Lambda → Custom Logs → CloudWatch → KDF → S3

S3 (processed-logs, partitioned by date)
     │
Glue Crawler (updates partitions daily)
     │
Glue Data Catalog
     │
Athena (ad hoc queries)
     │
QuickSight (error rate, p99 latency dashboards)

CloudWatch Logs Insights → Quick queries (last few hours)
EventBridge → Lambda (alert if error rate > 5% in 5 min)
```

**Athena vs Elasticsearch Cost:**
| Item | Athena | Elasticsearch |
|------|--------|--------------|
| 10GB scan/query | $0.05 | — |
| 100 queries/day | ~$5/day | fixed cluster |
| Base cluster | $0 | ~$500/month |
| Break-even | favorable if < 100 complex queries/day | — |

---

## Case 5 — Real-Time ETL with Kinesis + Glue Streaming

**Context:** A marketplace company receives 50,000 events/minute of clicks, impressions, and conversions. It needs data available for campaign analysis with a maximum delay of 5 minutes.

**Requirements:**
- Ingestion-to-analysis latency: < 5 minutes
- Event enrichment (add product data from the catalog)
- Deduplication (the same event may arrive duplicated)
- Controlled cost (no always-on cluster)

**Architecture:**
```
App (JS/mobile tracker)
     │ POST /track
     ▼
API Gateway → Kinesis Data Streams (10 shards)
                     │
              Glue Streaming ETL
              ├── Reads from KDS every 60s
              ├── Enriches with catalog (DynamoDB lookup)
              ├── Deduplicates by event_id (5min window)
              ├── Converts to Parquet
              └── Writes to S3 (partitioned by hour)
                     │
              S3 (processed/events/)
                     │
              Athena (query by campaign/product — up to 5min ago)
              QuickSight (near-real-time dashboard)
              
EventBridge → Glue Job (trigger every 5min as a simpler alternative)
```

**Glue Streaming vs Kinesis Data Analytics (Flink):**
| Criterion | Glue Streaming | KDA / Flink |
|---------|---------------|-------------|
| Setup complexity | Low | High |
| Minimum latency | ~60s | Ms–seconds |
| Time windows | Basic | Advanced (sliding, tumbling) |
| Cost | DPU/hour | KPU/hour |
| When to use | Simple near-real-time ETL | CEP, real-time anomaly detection |
