# Cheatsheet — Module 12: Data and Analytics

## Quick Comparison of Analytics Services

| Service | Type | Model | Best For |
|---|---|---|---|
| **Athena** | Interactive query | Serverless, pay-per-TB | Ad-hoc SQL on S3 |
| **Glue** | ETL | Serverless | ETL transformations, Catalog |
| **EMR** | Big Data Cluster | Managed EC2 | Custom Spark/Hive/Presto |
| **Redshift** | Data Warehouse | Cluster or Serverless | OLAP, BI, analytical SQL |
| **OpenSearch** | Search/Analytics | Cluster | Full-text search, logs, dashboards |
| **MSK** | Managed Kafka | Cluster | Native Kafka streaming |
| **Kinesis Data Streams** | Streaming | Shards | Customizable real-time streaming |
| **Kinesis Firehose** | Streaming delivery | Serverless | Automatic delivery to S3/Redshift |
| **QuickSight** | BI/Visualization | SaaS | Dashboards, SPICE |

---

## Athena — Cost and Performance Optimizations

| Technique | Impact | Implementation |
|---|---|---|
| **Columnar format** (Parquet/ORC) | Reduces scanned data by up to 90% | Convert CSV → Parquet with Glue |
| **Compression** (Snappy, GZIP) | Reduces file size | Configure in the ETL job |
| **Partitioning** | Scans only relevant partitions | `year=2024/month=01/day=15/` |
| **Bucketing** | Reduces data shuffle in joins | `CLUSTERED BY (user_id) INTO 64 BUCKETS` |
| **CTAS** (Create Table As Select) | Transforms in-place to Parquet | `CREATE TABLE ... WITH (format='PARQUET')` |

**Pricing:** $5.00 per TB scanned. Free: DDL queries, failed queries, queries with partitions (except scanned data).

---

## Kinesis Data Streams vs Kinesis Firehose

| Characteristic | KDS | KDF |
|---|---|---|
| Latency | Milliseconds (real-time) | Near real-time (60s-900s buffer) |
| Consumers | Custom (Lambda, App, KDA) | Automatic (S3, Redshift, OpenSearch) |
| Data retention | 1-365 days | None (delivers and discards) |
| Reprocessing | Yes (cursor for multiple consumers) | No |
| Throughput | 1 MB/s per shard (write), 2 MB/s (read) | Automatic |
| Code required | Yes (consumer code) | No (configuration) |
| Transformation | Not native | Optional Lambda |

---

## Glue — Components

| Component | Function |
|---|---|
| **Data Catalog** | Central repository of schemas/metadata |
| **Crawler** | Discovers schemas and populates the Catalog automatically |
| **ETL Job** | Spark code (Python/Scala) for transformation |
| **DataBrew** | Visual no-code transformation interface |
| **Studio** | Visual IDE for Glue jobs |
| **Workflow** | Orchestration of crawlers + jobs + events |
| **Streaming ETL** | Continuous ETL from Kinesis/Kafka |

---

## Redshift — Distribution and Sort Keys

| Type | When to Use |
|---|---|
| DISTKEY: join field | Co-locates related rows on the same nodes |
| DISTKEY: ALL | Replicates the whole table on every node (small dimension tables) |
| DISTKEY: EVEN | Round-robin distribution (no frequent join) |
| SORTKEY: filter/range field | Physically orders rows — more efficient range scans |

---

## Redshift Serverless vs Provisioned

| | Serverless | Provisioned |
|---|---|---|
| **Management** | Zero | Manage the cluster, resize |
| **Scalability** | Automatic | Manual or Elastic Resize |
| **Cost** | RPU-hours (pay-per-use) | Node-hour (regardless of usage) |
| **Best for** | Intermittent, dev/test | Predictable, continuous workloads |
| **Concurrency Scaling** | N/A | Automatic transient clusters |

---

## EMR — Node Types and Recommended Instances

| Node | Function | Recommended Instance | Spot OK? |
|---|---|---|---|
| **Master** | Coordinates the cluster (YARN RM, HDFS NN) | On-Demand or Reserved | No (SPOF) |
| **Core** | Processes + stores HDFS | On-Demand or Reserved | No (would lose HDFS data) |
| **Task** | Processes only (no HDFS) | **Spot** (ideal) | **Yes** — can be interrupted without losing data |

---

## Lake Formation — Access Controls

| Level | Example |
|---|---|
| **Database** | List/describe tables |
| **Table** | Select all columns |
| **Column** | Select only the authorized columns |
| **Row** | Row filters (Row-level filters) |
| **Cell** | Combination of column + row |

Fine-Grained Access Control (FGAC) replaces S3 bucket policy management for data lakes.

---

## Exam Tips — Common Patterns

| Clue in the Question | Likely Answer |
|---|---|
| "Ad-hoc SQL on S3" | Athena |
| "Reduce Athena cost" | Parquet + Partitioning |
| "Visual ETL without code" | AWS Glue DataBrew |
| "Automatic schema discovery" | Glue Crawler → Glue Data Catalog |
| "Migrate Kafka to AWS" | Amazon MSK |
| "Automatic stream delivery to S3" | Kinesis Data Firehose |
| "Real-time with a custom consumer" | Kinesis Data Streams |
| "Self-service BI for non-technical users" | QuickSight (SPICE) |
| "Full-text search + Kibana dashboards" | OpenSearch Service |
| "Historical data in S3 + SQL query" | Redshift Spectrum or Athena |
| "Granular column-level security in a data lake" | Lake Formation FGAC |
| "Stream processing with Apache Flink" | MSF (Managed Apache Flink) |
| "Oracle → Aurora heterogeneously" | SCT (schema) + DMS (data) |
