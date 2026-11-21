# Flashcards — Module 12: Data and Analytics

> **Format:** Q = Question | A = Answer  
> **Total:** 25 flashcards

---

**Q:** What is Amazon Athena and what is its pricing model?  
**A:** A serverless interactive query service using standard SQL directly on S3 files. Model: **pay-per-query** — $5 per TB of data scanned. Optimizations to reduce cost: use a columnar format (Parquet/ORC) and partitioning (scan less data).

---

**Q:** Why does Parquet reduce Athena query costs?  
**A:** Parquet is **columnar**: Athena reads only the columns required by the query, ignoring the others. It also has native compression (Snappy, Gzip). Result: scans less data → lower cost ($5/TB scanned). CSV is row-based — it scans the entire row even if the query needs only 2 columns.

---

**Q:** What is the AWS Glue Data Catalog?  
**A:** A central metadata repository (schemas, tables, partitions) for data in S3, RDS, Redshift, etc. Works as the metastore for Athena and EMR. Glue Crawlers populate the Catalog automatically. Each data source has a "database" with "tables" in the Catalog.

---

**Q:** What is the difference between Glue ETL Jobs and AWS Glue DataBrew?  
**A:** **Glue ETL Jobs:** Python/Scala (Spark) code for complex custom transformations. **AWS Glue DataBrew:** a visual, no-code data preparation interface. More than 250 pre-built transformations (deduplicate, filter, format). For data analysts without programming skills.

---

**Q:** What are the 3 node types in an Amazon EMR cluster?  
**A:** **Master Node:** coordinates the cluster (YARN ResourceManager, HDFS NameNode). **Core Node:** processes tasks + stores HDFS data (YARN NodeManager). **Task Node:** only processes tasks, no HDFS storage — ideal for Spot Instances (losing a Task Node does not affect data).

---

**Q:** When should you use Amazon MSK instead of Amazon Kinesis?  
**A:** **MSK:** when you already use Apache Kafka on-premises and want to migrate without changing producers/consumers (fully managed Kafka); when you need the native Kafka protocol. **Kinesis:** a more integrated proprietary AWS service; no broker operations; better integration with Lambda, Firehose, etc. MSK = lift-and-shift Kafka; Kinesis = AWS-native.

---

**Q:** What is the difference between Redshift Multi-AZ and Redshift Cross-Region Snapshot?  
**A:** **Multi-AZ:** a Redshift cluster with replicas in multiple AZs — HA with automatic failover (available on RA3). **Cross-Region Snapshot:** a cluster backup copied to another Region for DR. Snapshots: automatic (1-35 days). For migration/restore in another Region.

---

**Q:** What is Redshift Concurrency Scaling?  
**A:** A feature that automatically adds transient clusters when concurrency demand exceeds the main cluster's capacity. Queries are routed to the scaling clusters. Free credits: 1 hour per day of active main cluster. Additional cost beyond the credits.

---

**Q:** What is SPICE in Amazon QuickSight?  
**A:** Super-fast Parallel In-memory Calculation Engine: an in-memory cache of the data imported into QuickSight. Dashboards are ultra-fast because they don't need to query the database on each interaction. Quota per user (GB). Scheduled or on-demand refresh to update the data.

---

**Q:** What are the use cases for Amazon Kinesis Data Firehose?  
**A:** Delivery of streaming data to **destinations**: S3, Redshift (via S3 staging), OpenSearch, Splunk, custom HTTP endpoints. Characteristics: near-real-time (buffering of 60s to 900s or 1-128 MB), optional transformation with Lambda, format conversion (JSON→Parquet), compression. No consumer required — automatic delivery.

---

**Q:** What is AWS Lake Formation and what problem does it solve?  
**A:** Simplifies building secure **Data Lakes** on S3. Problem: granular security in data lakes is complex. Solution: Fine-Grained Access Control (FGAC) — control by column, row-level filters, cell-level security. Centralizes permissions instead of individual policies per bucket/folder.

---

**Q:** What is the difference between OpenSearch Service and Amazon Kendra?  
**A:** **OpenSearch:** a search and analytics engine for logs, metrics, full-text search, visualizations (Kibana/OpenSearch Dashboards). **Kendra:** an AI-powered search engine for enterprise documents — understands natural language ("how many vacation days do I have?"), searches PDFs, Word, SharePoint. OpenSearch = technical/analytics; Kendra = intelligent document search.

---

**Q:** What is Redshift Spectrum?  
**A:** A Redshift extension that lets you query data directly in **S3** without loading it into Redshift. Uses independently scalable nodes (Spectrum layer). Accesses the Glue Data Catalog for metadata. Ideal for: historical/cold data in S3 + hot data in Redshift in a single SQL query.

---

**Q:** How does partitioning work in Athena and why is it important?  
**A:** Partitioning: organizing data in S3 into hierarchical folders by a field (e.g., `s3://bucket/year=2024/month=01/day=15/`). In the query: `WHERE year=2024 AND month=01` → Athena scans only those partitions. Drastically reduces scanned data → lower cost and better performance.

---

**Q:** What are the differences between Kinesis Data Streams and Kinesis Data Firehose?  
**A:** **KDS:** real-time streaming, custom consumers (Lambda, apps, Kinesis Analytics), 1-365 day retention, configurable shards. Requires consumer code. **KDF:** managed delivery to destinations (S3, Redshift, OpenSearch), near-real-time (buffer), no consumer code, automatic compression/transformation. KDS = flexibility; KDF = delivery simplicity.

---

**Q:** What is AWS Glue Streaming ETL?  
**A:** An extension of Glue to process **streaming** data from Kinesis Data Streams, Kafka/MSK, or SQS. Processes micro-batches continuously (streaming ETL). Uses Apache Spark Structured Streaming. A managed alternative to Apache Flink for streaming transformations.

---

**Q:** How does Amazon EMR Serverless work?  
**A:** Runs Spark/Hive workloads without managing clusters. Submit a job → AWS automatically provisions the necessary resources → runs → deallocates. No nodes to manage. Pay per vCPU-second and GB-second. Ideal for intermittent or unpredictable workloads. No cold start from manual cluster provisioning.

---

**Q:** What are Amazon OpenSearch UltraWarm and Cold Tiers?  
**A:** Cost-effective storage tiers for historical data in OpenSearch: **UltraWarm:** less frequently accessed data in S3 (warm tier), cheaper than hot instances, still queryable in seconds. **Cold:** even cheaper, archived data, higher access latency. Migration: indexes are moved hot → warm → cold as they age.

---

**Q:** What is an AWS Glue Workflow?  
**A:** Orchestration of ETL pipelines in Glue. Lets you create execution graphs with triggers, crawlers, and jobs. Trigger types: on-demand, schedule (cron), event-based (completion of another job). Graphical visualization in the console. For complex multi-step ETL pipelines without Step Functions.

---

**Q:** When should you use Amazon EMR vs AWS Glue for ETL?  
**A:** **Glue:** simple-to-medium ETL, no infrastructure to manage, native integration with Catalog/S3/Redshift, ideal for non-Spark engineers. **EMR:** complex Spark/Hive/Presto workloads, full control over cluster configuration, multiple frameworks (local HDFS), HPC workloads, greater flexibility and performance for big data.

---

**Q:** What is Amazon Kinesis Data Analytics (now Amazon Managed Service for Apache Flink)?  
**A:** A managed service for Apache Flink applications — processes streams in real time with SQL or Java/Python/Scala. Input: KDS or MSK. Output: S3, KDS, KDF, Lambda, etc. Use cases: real-time anomaly detection, windowed aggregations (sliding/tumbling windows), joins of multiple streams.

---

**Q:** What is the Amazon Athena billing model?  
**A:** $5.00 per TB of data **scanned** (not per query time). No charge for failed queries. To reduce cost: (1) Compression — smaller data = fewer TB; (2) Columnar (Parquet/ORC) — scans only the necessary columns; (3) Partitioning — scans only the relevant partitions.

---

**Q:** What is Amazon QuickSight Q?  
**A:** QuickSight's NLP (Natural Language Processing) engine: lets you ask questions in natural language about the dashboards. E.g.: "what was the revenue in January?" → QuickSight interprets it and automatically generates a visualization. Requires topic training with your data.

---

**Q:** What is DMS (Database Migration Service) and when should you use SCT?  
**A:** **DMS:** database migration with minimal downtime (ongoing CDC replication). Supports homogeneous and heterogeneous migrations. **SCT (Schema Conversion Tool):** converts schema + code (stored procedures, views) between different engines (Oracle → PostgreSQL, SQL Server → Aurora). DMS = migrates data; SCT = converts incompatible schema. Homogeneous (MySQL→MySQL): DMS without SCT. Heterogeneous: DMS + SCT.

---

**Q:** What is Amazon Redshift Serverless?  
**A:** Redshift without the need to provision or manage clusters. AWS automatically provisions and scales capacity based on the workload. Pay-per-use (RPU-hours — Redshift Processing Units). Ideal for: intermittent workloads, dev/test environments, sporadic analytics. Vs cluster: no management, but cost can be higher for constant workloads.
