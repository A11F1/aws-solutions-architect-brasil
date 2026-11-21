# Questions — Module 12: Data and Analytics

> **SAA-C03 Domain**: Design High-Performing Architectures  
> **Difficulty**: Medium

---

**1.** A company has raw data in S3 in JSON format (10 TB). Analysts frequently query only 3 of the 50 available columns. Which change would have the greatest impact on the cost and performance of Athena queries?

- A) Move the data to Redshift
- B) Convert the JSON files to Parquet format and partition by date
- C) Use S3 Intelligent-Tiering to reduce storage cost
- D) Enable S3 Transfer Acceleration

<details><summary>Answer</summary>

**B** — Parquet is columnar: Athena reads only the 3 required columns (vs all 50 in JSON). Partitioning avoids a full table scan. Combined: 80-95% reduction in query cost and time. Lower cost per TB scanned + much faster queries.

</details>

---

**2.** A Data Science team needs to run ETL on 100 GB of daily data in S3, transforming JSON files into Parquet and applying data cleaning. Which AWS service is most suitable for this without managing servers?

- A) AWS Glue (serverless ETL jobs with managed Spark)
- B) Amazon EMR with a permanent cluster
- C) Lambda for file-by-file processing
- D) AWS Batch with containers

<details><summary>Answer</summary>

**A** — Glue ETL jobs: serverless Spark, scales automatically, pay per DPU-hour only while the job runs. EMR requires cluster management. Lambda has a 15-minute limit and 10 GB of memory (insufficient for 100 GB of data). Glue is the standard for serverless ETL on AWS.

</details>

---

**3.** An analytics team wants to create a centralized catalog of the datasets stored in S3, making them searchable via Athena without manually configuring each table. Which service solves this automatically?

- A) Amazon Macie
- B) AWS Glue Crawlers + Glue Data Catalog
- C) AWS Lake Formation
- D) Amazon OpenSearch

<details><summary>Answer</summary>

**B** — Glue Crawlers: scan S3, infer schemas automatically, and create/update tables in the Glue Data Catalog. Athena uses the Glue Data Catalog as its metastore. Within minutes, all S3 datasets become queryable via Athena.

</details>

---

**4.** A company operates an on-premises Kafka cluster and wants to migrate to AWS without rewriting producers and consumers. Which AWS service should it use?

- A) Amazon Kinesis Data Streams
- B) Amazon SQS FIFO
- C) Amazon MSK (Managed Streaming for Apache Kafka)
- D) Amazon EventBridge

<details><summary>Answer</summary>

**C** — MSK is managed Kafka: compatible with native Kafka APIs. Existing producers and consumers keep working without modification. Kinesis has its own APIs (incompatible with Kafka).

</details>

---

**5.** A company has a Redshift cluster and analysts' queries are degrading performance for other users. Which Redshift feature solves this with minimal configuration?

- A) Increase the cluster size to ra3.16xlarge
- B) Enable Concurrency Scaling to automatically add capacity during query spikes
- C) Migrate to Redshift Serverless
- D) Create Redshift read replicas

<details><summary>Answer</summary>

**B** — Concurrency Scaling: when multiple users submit simultaneous queries and the queue grows, Redshift automatically provisions additional clusters to process the extra queries. The first 60 minutes per day of scaling are free. There are no read replicas in Redshift (unlike RDS).

</details>

---

**6.** An architect needs to build a pipeline that delivers clickstream events from a web app to S3 in Parquet format for later analysis with Athena. What is the simplest architecture?

- A) Kinesis Data Streams → Lambda → S3 (converting to Parquet in the Lambda)
- B) Kinesis Data Firehose with format conversion to Parquet (via Glue Data Catalog) → S3
- C) API Gateway → Lambda → Glue Job → S3
- D) MSK → Glue Streaming → S3

<details><summary>Answer</summary>

**B** — Kinesis Firehose supports native format conversion to Parquet/ORC using the Glue Data Catalog (which defines the schema). No-code configuration, auto-scaling, automatic compression, and buffered delivery to S3. The simplest, most managed solution.

</details>

---

**7.** A company needs business analysts to build interactive dashboards on data in S3 without moving data to another system. What is the best solution?

- A) Redshift + QuickSight
- B) Athena + QuickSight (with SPICE for interactive dashboards)
- C) OpenSearch + Kibana
- D) EMR + Jupyter Notebooks

<details><summary>Answer</summary>

**B** — Athena queries S3 directly (without moving data); QuickSight connects to Athena and can import the results into SPICE (in-memory cache) for ultra-fast dashboards. No infrastructure to manage. Redshift (A) would require moving the data.

</details>

---

**8.** A company has a self-managed Elasticsearch cluster running on EC2 for log analytics. They want to migrate to a managed service while keeping existing APIs and Kibana dashboards. Which service should they use?

- A) Amazon Kinesis Data Analytics
- B) Amazon OpenSearch Service (with OpenSearch Dashboards/Kibana support)
- C) Amazon Athena
- D) AWS Glue

<details><summary>Answer</summary>

**B** — Amazon OpenSearch Service is the managed successor of Elasticsearch Service. Compatible with Elasticsearch APIs and Kibana (via OpenSearch Dashboards). Migrating self-managed Elasticsearch clusters to OpenSearch Service is supported.

</details>

---

**9.** A data team needs to control which users can access which tables and columns in the data lake (S3 + Glue Catalog). Which service implements column-level permissions?

- A) S3 Bucket Policies
- B) Glue Data Catalog with IAM policies
- C) AWS Lake Formation with column-level security
- D) Amazon Macie

<details><summary>Answer</summary>

**C** — Lake Formation: manages data lake permissions with database/table/column/row granularity (Cell-Level Security). Integrates with Athena, Redshift Spectrum, and EMR. S3 Bucket Policies and IAM policies do not operate at the column level of a table.

</details>

---

**10.** A company stores historical data in S3 but needs to run ad-hoc SQL queries on that data combined with "hot" data in a Redshift cluster. Which Redshift feature should it use?

- A) Redshift Data Sharing
- B) Redshift Spectrum to query S3 data as external tables
- C) Migrate all historical data to Redshift
- D) Use Athena for all the data and abandon Redshift

<details><summary>Answer</summary>

**B** — Redshift Spectrum: creates external tables in Redshift pointing to S3 data (via Glue Catalog). Queries that reference external tables are distributed to thousands of Spectrum nodes over S3 — without moving data. Combines hot (Redshift) and cold (S3) data in a single SQL query.

</details>

---

**11.** An ML team needs to store computed features for use in both training (batch) and real-time (online) inference without inconsistencies. Which AWS service provides this?

- A) S3 to store CSV files of features
- B) SageMaker Feature Store with an offline store (S3) and an online store (low latency)
- C) DynamoDB for online features and S3 for offline features, kept separate
- D) ElastiCache Redis for all features

<details><summary>Answer</summary>

**B** — SageMaker Feature Store: keeps the offline store (S3, for training) and the online store (low latency, for real-time inference) in sync. Ensures consistency: the same feature definition is used in training and production, avoiding training/serving skew.

</details>

---

**12.** A company processes large volumes of data with Apache Spark on EMR. To reduce costs, they want to use Spot Instances for the processing nodes. Which node type should use Spot?

- A) Master node
- B) Core nodes
- C) Task nodes
- D) B and C

<details><summary>Answer</summary>

**C** — Task nodes are ideal for Spot: they process data but do not store HDFS data. If they are interrupted, the job can redistribute the work. Core nodes store HDFS data — if they are lost (Spot interruption), data is lost. Master node = never Spot (it coordinates the entire cluster).

</details>

---

**13.** A company wants to query data from a DynamoDB database and an S3 bucket in a single SQL query using Athena. Which feature should it use?

- A) DynamoDB Streams exported to S3 before the queries
- B) Athena Federated Query with a Lambda Data Source Connector for DynamoDB
- C) Glue ETL to move DynamoDB data to S3 before the queries
- D) AWS Glue DataBrew to unify the sources

<details><summary>Answer</summary>

**B** — Athena Federated Query: uses Lambda connectors to query non-S3 sources (DynamoDB, RDS, Redshift, CloudWatch, etc.) directly in SQL. The DynamoDB connector translates Athena queries into DynamoDB API calls. Queries across S3 and DynamoDB in a single SQL statement.

</details>

---

**14.** The analytics team needs to detect anomalies in sales metrics in real time (e.g., a sudden drop in conversion). Which combination of services implements this?

- A) Kinesis Data Streams → Kinesis Data Analytics (Apache Flink) → SNS for alerts
- B) S3 + Athena with queries scheduled by Lambda → SNS
- C) CloudWatch Metrics with Anomaly Detection → CloudWatch Alarm → SNS
- D) A and C are valid depending on the data source

<details><summary>Answer</summary>

**D** — Kinesis Analytics (A): for data already streaming (clickstream, apps). CloudWatch Anomaly Detection (C): for application and infrastructure metrics. The choice depends on the source: if the sales data comes from a stream, use Kinesis Analytics. If they are system metrics, use CloudWatch.

</details>

---

**15.** A company needs to migrate its on-prem Oracle data warehouse to Amazon Redshift. The Oracle schema uses stored procedures and has a table structure different from Redshift. What is the correct process?

- A) DMS directly from Oracle to Redshift (natively supported)
- B) SCT (Schema Conversion Tool) to convert the schema + procedures, then DMS to migrate the data
- C) Export CSV from Oracle → S3 → Redshift COPY command
- D) EMR to transform the data before loading into Redshift

<details><summary>Answer</summary>

**B** — Heterogeneous migration Oracle → Redshift: SCT first to convert DDL, stored procedures, and proprietary Oracle code to Redshift-compatible SQL (it also identifies what cannot be converted automatically). Then DMS to move the data (with support for full load and CDC for zero downtime).

</details>
