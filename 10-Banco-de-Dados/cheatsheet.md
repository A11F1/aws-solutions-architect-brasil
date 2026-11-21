# Cheatsheet — Module 06: Databases

## RDS Multi-AZ vs Read Replica Comparison

| | Multi-AZ | Read Replica |
|---|---|---|
| Purpose | High availability / DR | Read scalability |
| Replication | Synchronous | Asynchronous |
| RPO | ~0 (no data loss) | Small lag possible |
| RTO | ~1-2 minutes (DNS flip) | Manual (promotion to standalone) |
| Can serve reads? | NO (standby only) | YES (read-only) |
| Cross-region? | No (same Region) | Yes (up to 5 per DB) |
| Failover | Automatic | Manual |

## Aurora vs RDS

| | Aurora | RDS |
|---|---|---|
| Copies | 6 (across 3 AZs) | 2 (Multi-AZ: primary + standby) |
| Write quorum | 4/6 | 1 (primary synchronous to standby) |
| Read Replicas | Up to 15 | Up to 5 |
| RR failover | ~30 seconds | Manual promote |
| Global Database | Yes (< 1s lag) | No (manual DX/VPN only) |
| Serverless | v2 (0.5-128 ACU) | No (except Aurora) |
| Performance | 3x MySQL / 5x PostgreSQL | Standard |

## DynamoDB — Key Values

| Parameter | Value |
|---|---|
| 1 RCU (Eventually Consistent) | 2 reads of 4 KB/s |
| 1 RCU (Strongly Consistent) | 1 read of 4 KB/s |
| 1 WCU | 1 write of 1 KB/s |
| DAX latency | Microseconds |
| TTL | Free; eventual deletion within 48h |
| Streams retention | 24 hours |
| Global Tables | Active multi-region replication |

## DynamoDB Indexes

| | LSI | GSI |
|---|---|---|
| When to create | Only at table creation | Any time |
| Partition Key | Same as the table | Different |
| Sort Key | Different from the table | Any attribute |
| Throughput | Shared with the table | Independent |
| Consistency | Strong or Eventually | Eventually only |

## ElastiCache Redis vs Memcached

| | Redis | Memcached |
|---|---|---|
| Persistence | Yes (RDB, AOF) | No |
| Replication | Yes (Replica) | No |
| Multi-AZ | Yes | No |
| Cluster Mode | Yes (sharding) | Yes (multi-thread) |
| Structures | String, Hash, List, Set, Sorted Set, Geo, Streams | String only |
| Pub/Sub | Yes | No |
| Use case | Sessions, leaderboards, rich cache, queues | Simple cache, high-speed multi-thread |

## Caching Patterns

| Pattern | Cache Miss | Cache Hit | Pros | Cons |
|---|---|---|---|---|
| Lazy Loading | Fetch DB → write cache | Returns cache | Only stores what is needed | 3 round trips on a miss; data can become stale |
| Write-Through | Write DB + cache | Returns cache | Data always fresh | Write penalty; caches data that is never read |
| Write-Behind | Write cache → async DB | Returns cache | Lowest write latency | Risk of loss if the cache fails before persisting |

## Choosing a Database

| Need | Database |
|---|---|
| Relational OLTP (low latency) | RDS MySQL/PostgreSQL/Aurora |
| Relational OLTP (high performance) | Aurora |
| Relational OLAP / Data Warehouse | Redshift |
| NoSQL key-value / documents | DynamoDB |
| In-memory cache | ElastiCache Redis |
| Graphs | Amazon Neptune |
| Documents (MongoDB) | Amazon DocumentDB |
| Time series (IoT, metrics) | Amazon Timestream |
| Immutable ledger | Amazon QLDB |
| Managed Cassandra | Amazon Keyspaces |

## Quick Exam Tips
- The RDS Multi-AZ endpoint does not change after failover (same DNS CNAME)
- Aurora: 2 main endpoints — Cluster Endpoint (write) + Reader Endpoint (read LB)
- DynamoDB Global Tables requires On-Demand, or active replication via replica write capacity
- Redshift Spectrum: query S3 data directly from Redshift without COPY
- RDS Proxy: required when Lambda accesses RDS (connection pooling, IAM auth, Secrets Manager)
- ElastiCache Redis Cluster Mode: distributes the keyspace across multiple shards → higher capacity and write throughput
- Aurora Serverless v2: scales in ~1 second (does not require a restart like v1)
- PITR RDS retention: 1-35 days; Aurora: up to 35 days
