# Flashcards — Module 06: Databases

---

**Q:** What is the replication difference between RDS Multi-AZ and Read Replica?
**A:** Multi-AZ: **synchronous** replication → zero RPO, automatic failover (~1-2 min). Read Replica: **asynchronous** replication → small lag, for offloading reads, manual failover (promote to a standalone DB)

---

**Q:** How many copies of data does Aurora keep, and how?
**A:** 6 copies across 3 AZs (2 per AZ). Write quorum: 4/6. Read quorum: 3/6. Tolerates the loss of 1 AZ with no impact on reads; the loss of 2 AZs with no impact on writes

---

**Q:** What are the RTO/RPO of Aurora Global Database in case of a regional failure?
**A:** RPO < 1 second (replication with ~1s lag). RTO < 1 minute (promote a secondary Region manually or automatically via managed failover)

---

**Q:** What is RDS Proxy and when should you use it?
**A:** A managed proxy that does connection pooling between applications and RDS/Aurora. It reuses connections and avoids connection exhaustion. Critical for Lambda (stateless, opens a new connection on every invoke) and ECS (many tasks)

---

**Q:** How far back does RDS PITR (Point-In-Time Recovery) allow restoring?
**A:** Up to the last 35 days (configurable from 1 to 35 days). Restores to any second within that window using backups + transaction logs

---

**Q:** What are the 2 DynamoDB capacity modes and when should you use each?
**A:** Provisioned: you define RCUs and WCUs + Auto Scaling. Use for predictable/consistent load (cheaper). On-Demand: charges per request. Use for unpredictable load, new workloads, sudden spikes

---

**Q:** What are 1 RCU and 1 WCU in DynamoDB?
**A:** 1 RCU = 1 strongly consistent read of up to 4 KB (or 2 eventually consistent reads). 1 WCU = 1 write of up to 1 KB per second

---

**Q:** What is the latency of DAX vs DynamoDB directly?
**A:** DynamoDB: millisecond latency. DAX (DynamoDB Accelerator): **microseconds** (managed in-memory cache, compatible with the DynamoDB API)

---

**Q:** What is the difference between LSI and GSI in DynamoDB?
**A:** LSI: same partition key, different sort key, created ONLY at table creation, uses the table's throughput. GSI: different partition key, can be created at any time, independent throughput

---

**Q:** How long does DynamoDB Streams retain data?
**A:** 24 hours. Streams captures item-level changes (INSERT, MODIFY, REMOVE) with the before and/or after image. Integrates with Lambda (trigger) and DynamoDB Global Tables (multi-region replication)

---

**Q:** What is the "Lazy Loading" read pattern in ElastiCache?
**A:** Cache Miss: fetch from the database → write to the cache. Cache Hit: return from the cache. Pros: only stores requested data. Cons: a cache miss = 3 trips (cache, DB, cache) + risk of stale data

---

**Q:** What is the main difference between Redis and Memcached in ElastiCache?
**A:** Redis: persistence, replication, Multi-AZ, Pub/Sub, Sorted Sets, Geospatial, Cluster Mode (sharding). Memcached: multi-threading, no persistence, no replication, simple, pure cache. For HA and complex structures: Redis

---

**Q:** What is Redshift Enhanced VPC Routing?
**A:** It forces Redshift COPY and UNLOAD traffic to use the VPC (instead of the public internet/public S3 endpoints). It allows control via VPC Flow Logs, SGs, and NACLs. Recommended for security and compliance

---

**Q:** When should you use Amazon Neptune?
**A:** For graph data and queries: social networks (friendships), fraud detection (suspicious connections), knowledge graphs, recommendation engines. Supports Property Graph (Gremlin, openCypher) and RDF (SPARQL)

---

**Q:** When should you use Amazon QLDB?
**A:** A ledger database with an immutable, cryptographically verifiable history. For: financial records, supply chain tracking, audit logs where no data may be deleted or retroactively modified

---

**Q:** What workload is Aurora Serverless v2 ideal for?
**A:** Workloads with highly variable traffic (e.g., dev/test, applications with nightly or seasonal peaks). Scales from 0.5 to 128 ACUs in 0.5 ACU increments in ~1 second. Zero downtime during scaling

---

**Q:** What are the read endpoint and write endpoint of an Aurora Cluster?
**A:** Cluster Endpoint (write): points to the primary writer instance. Reader Endpoint: load balancer across all Read Replicas. Applications should use the correct endpoint to separate reads from writes

---

**Q:** What is ElastiCache Cluster Mode (sharding in Redis)?
**A:** It distributes key slots (keyspace partitions) across multiple node groups (shards). It allows higher total throughput (parallel writes across shards) and greater data capacity than a single node. Each shard has replicas for HA

---

**Q:** What is Amazon Keyspaces?
**A:** A fully managed, Apache Cassandra-compatible service on AWS. For lift-and-shift migration of Cassandra workloads without managing clusters. CQL (Cassandra Query Language) compatible

---

**Q:** What is Amazon Timestream used for?
**A:** A serverless time series database. For IoT data, DevOps metrics, application performance telemetry. Stores and analyzes trillions of events per day. Queries with built-in time-series analytics functions
