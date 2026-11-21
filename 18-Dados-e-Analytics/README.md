# Module 12 — Data and Analytics

## AWS Analytics Ecosystem Overview

```
Ingestion → Storage → Processing → Analysis → Visualization
  Kinesis    S3 (Data Lake)   EMR / Glue     Athena    QuickSight
  MSK        Redshift DW      Spark          OpenSearch BI tools
  DataSync   OpenSearch       Lambda         Redshift
```

---

## Amazon Athena

Serverless SQL over data in S3:

- **Pay-per-query**: $5 per TB of data scanned
- No infrastructure to manage; reads formats: Parquet, ORC, Avro, JSON, CSV
- Uses **Glue Data Catalog** as its metastore
- Supports partitioning → **crucial for reducing cost** (avoids a full table scan)
- **Federated Query**: queries other data sources via Lambda connectors (DynamoDB, RDS, Redshift, CloudWatch)
- **Athena for Apache Spark**: interactive Spark notebooks (no EMR cluster)

### Optimizations to Reduce Cost and Latency

```sql
-- Partitioning (partition by date → scan only relevant partitions)
MSCK REPAIR TABLE vendas; -- updates partitions in the Glue Catalog

-- Columnar (Parquet/ORC scan only the required columns)
SELECT amount FROM vendas WHERE year=2024 AND month=01
-- Only scans the 'amount' column in the Jan/2024 partitions → 99% less data
```

---

## AWS Glue

Serverless ETL and data catalog:

| Component | Function |
|---|---|
| **Glue Data Catalog** | Central metastore: tables, schemas, S3 location |
| **Crawlers** | Automatically discover and register schemas (S3, RDS, DynamoDB, etc.) |
| **ETL Jobs** | Managed Python/Scala Spark jobs (automatic DPU scaling) |
| **Glue Studio** | Visual interface for ETL jobs |
| **DataBrew** | Visual data preparation (cleaning, normalization) without code |
| **Glue Streaming** | Real-time streaming ETL (Kinesis, Kafka, S3) |
| **Workflow** | Orchestration of ETL jobs with triggers and dependencies |

**DPU (Data Processing Unit)**: 4 vCPUs + 16 GB RAM. Jobs are billed per DPU-hour.

---

## Amazon EMR (Elastic MapReduce)

Managed cluster for big data with Apache Hadoop, Spark, Hive, Presto, HBase:

| Mode | Description |
|---|---|
| **EMR on EC2** | Cluster of EC2 instances (Master + Core + Task nodes) |
| **EMR on EKS** | Spark/Presto running as Kubernetes pods on EKS |
| **EMR Serverless** | Submit jobs without managing a cluster (auto-scaling workers) |

**Storage options:**
- **HDFS** (in-cluster, ephemeral): maximum throughput for processing
- **EMRFS → S3**: persistent storage; data survives cluster termination

**Nodes:**
- **Master** (Primary): coordinates the cluster (NameNode, YARN ResourceManager)
- **Core**: stores HDFS data + processes
- **Task**: processes only (no HDFS); ideal for Spot instances

---

## Amazon OpenSearch Service

Search + analytics engine based on Elasticsearch/OpenSearch:
- **Search**: full-text search, autocompletion, faceted search for applications
- **Log Analytics**: CloudWatch Logs/Kinesis Firehose → OpenSearch → Kibana/Dashboards
- **Real-time Analytics**: near real-time indexing for BI dashboards

| Deployment | Nodes | HA |
|---|---|---|
| Multi-AZ | Shard replicas in different AZs | YES (2-3 AZs) |
| Dedicated Master | Master nodes separate from data nodes | Recommended for prod |
| UltraWarm | Cheaper storage tier for "warm" data | Read-only; S3-backed |
| Cold Storage | Stores indexes in S3, loads when needed | Even cheaper |

---

## Amazon MSK (Managed Streaming for Apache Kafka)

Managed Kafka on AWS:
- Creates and manages Kafka clusters (broker nodes, zookeeper)
- Compatible with native Kafka APIs (existing producers/consumers need zero changes)
- **MSK Connect** (managed Kafka Connect): source/sink connectors for S3, RDS, OpenSearch, etc.
- **MSK Serverless**: automatic capacity, no brokers to manage
- Configurable data retention; integrates with IAM and VPC
- **When to use MSK vs Kinesis?** MSK: migrating existing Kafka; Kafka ecosystem (connectors, streams); compliance. Kinesis: simpler AWS-native option, no Kafka expertise required.

---

## Amazon Redshift

OLAP Data Warehouse with MPP (Massively Parallel Processing):
- Clusters: 1 Leader Node + 1-128 Compute Nodes
- **Columnar storage**: each column stored separately → compression + efficient scans
- **Distribution styles**: EVEN, KEY, ALL (for small dimension tables)

| Feature | Description |
|---|---|
| Redshift Spectrum | Query S3 as external tables without COPY |
| Materialized Views | Incremental pre-computation of complex queries |
| Concurrency Scaling | Automatically adds cluster capacity for bursts of simultaneous queries |
| AQUA | Hardware-accelerated cache (ra3 nodes only) |
| Redshift Serverless | Automatic capacity per query (pay per RPU-hour) |
| Data Sharing | Read data across clusters without copying (same or cross-account) |

---

## Amazon QuickSight

Serverless BI and dashboards:
- Connects to: S3 (Athena), Redshift, RDS, OpenSearch, third-party (Salesforce, etc.)
- **SPICE** (Super-fast Parallel In-memory Calculation Engine): in-memory cache for ultra-fast dashboards
- **Embedding**: dashboards embedded in web applications with per-user row-level security
- **ML Insights**: automatic anomaly detection, forecasting, natural-language narratives
- Pay per user/session (no cluster to provision)

---

## AWS Lake Formation

Data Lake governance and security on top of S3 + Glue Catalog:
- **Fine-grained access control**: permissions per database/table/column/row (cell-level security)
- **Blueprint**: automates data lake creation from a data source
- **DataFilters**: share only subsets of tables across accounts
- **Tag-based access control (LF-TBAC)**: tags on resources and identities for access control

---

## Typical Analytics Pipeline

```
Raw Data (on-prem, RDS, APIs)
  ├── Kinesis Data Firehose → S3 (raw layer)
  ├── DMS → S3 (RDS/Oracle migration)
  └── DataSync → S3 (on-prem files)

S3 (raw/bronze)
  └── Glue Crawlers → Glue Catalog
        └── Glue ETL Jobs (Spark)
              └── S3 (processed/silver/gold - partitioned Parquet)
                    ├── Athena (ad-hoc queries, pay-per-scan)
                    ├── Redshift Spectrum (mixed DW queries)
                    └── QuickSight (BI dashboards via SPICE)

Streaming:
  Kinesis/Kafka → Glue Streaming / Flink (Kinesis Analytics) → OpenSearch (real-time dash)
```

---

## Exam Tips

- Athena: **Parquet + partitioning** = the standard combination to minimize cost
- The Glue Catalog is shared across Athena, Redshift Spectrum, EMR — a single metastore
- **EMR node types**: Master (coordinates), Core (HDFS + compute), Task (compute only = ideal for Spot)
- MSK has no maximum retention; Kinesis has a maximum retention of 365 days
- Redshift cluster vs Redshift Serverless: cluster for predictable load; serverless for variable or intermittent load
- OpenSearch: NOT serverless (in the standard cloud offering) — you manage nodes and replication; it uses compute instances
- Lake Formation FGA overrides IAM policies for Glue Catalog permissions
- AQUA (Redshift): hardware acceleration for aggregate queries; available only on ra3 nodes
- QuickSight SPICE: imported data, in-memory cache; configurable refresh (scheduled or manual)
- Athena Federated Query uses Lambda Data Source Connectors → runs queries on non-S3 sources
