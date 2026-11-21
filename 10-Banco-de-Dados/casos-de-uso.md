# Use Cases — Module 06: Databases

## Case 1: E-commerce with RDS Multi-AZ and Read Replicas

**Scenario:** An e-commerce site with seasonal traffic spikes. The catalog is read 20x more than it is written. Automatic failover is required.

**Architecture:**
```
                ┌─ RDS PostgreSQL Primary (us-east-1a)
                │    ├── OLTP: orders, payments
ALB → App      ─┤    └── Multi-AZ Standby (us-east-1b) ← sync replica
Servers        │
                └─ RDS Read Replica x3 (us-east-1c)
                     └── OLTP reads: catalog, search

ElastiCache Redis ← Session cache + most-searched catalog items
```

**Key settings:**
- Primary endpoint: writes + critical reads (strong consistency)
- Read Replica endpoint (Reader): catalog reads (eventual consistency is ok)
- ElastiCache Redis: 1-hour cache for product pages
- RDS Multi-AZ: standby in a different AZ, failover ~1-2 min

---

## Case 2: Serverless Application with DynamoDB and DAX

**Scenario:** A real-time leaderboard API for a mobile game with 1 million concurrently active users.

**Architecture:**
```
Mobile App → API Gateway → Lambda
                              ├── DynamoDB Table (Players)
                              │     ├── PK: player_id
                              │     └── GSI: score-index (PK: game_id, SK: score)
                              └── DAX Cluster (cache layer)
                                    └── Latency < 1ms for top players
```

**DynamoDB Design:**
```
Table: GameLeaderboard
  PK: game_id#season
  SK: player_id
  GSI1: score-index (PK: game_id#season, SK: score) → top 100 per game

TTL: season_end_timestamp → automatically deletes data from the previous season
```

**Why DAX?** Lambda cold starts + 1M users → DynamoDB would give milliseconds, but DAX guarantees < 100µs for any query. The GSI allows efficient `query` by ranking without a scan.

---

## Case 3: Aurora Global Database for a Multi-Regional Application

**Scenario:** A global SaaS with users in the Americas (us-east-1) and Europe (eu-west-1). European regulation requires data to stay in Europe.

**Architecture:**
```
Americas (us-east-1) — PRIMARY
  └── Aurora PostgreSQL Cluster
        ├── Writer Instance
        ├── Reader Instance x2
        └── US App Servers (write + read)

                    ↓ Replication < 1 second (Global Database)

Europe (eu-west-1) — SECONDARY
  └── Aurora PostgreSQL Cluster (read-only)
        ├── Reader Instance x2 (local GDPR reads)
        └── EU App Servers (read only)

Failover:
  Aurora Managed Planned Failover → promotes EU to Primary in < 1 minute
  RPO < 1s | RTO < 1 minute
```

---

## Case 4: Analytics Pipeline with Redshift + Spectrum

**Scenario:** The BI team needs to join historical sales data in the DW with fresh data from an S3 data lake.

**Architecture:**
```
Historical Data (DW)           Fresh Data (Data Lake)
  └── Redshift Cluster             └── S3 (Parquet, partitioned by date)
        └── table: vendas_hist           └── Glue Catalog (file schema)

Unified query (Redshift Spectrum):
SELECT h.regiao, f.produto, SUM(h.receita) + SUM(f.receita) as total
FROM vendas_hist h
JOIN spectrum.vendas_fresh f ON h.produto_id = f.produto_id  ← S3 via Spectrum
WHERE f.data >= CURRENT_DATE - 7
GROUP BY 1, 2
```

**Benefit:** No daily ETL to load S3 data into the DW. Real-time joins between the DW and the Data Lake. Redshift MPP distributes the Spectrum processing across the nodes.

---

## Case 5: Fraud Prevention with DynamoDB + ElastiCache

**Scenario:** A banking system needs to check whether a card payment is being made from a suspicious IP or device in < 10ms.

**Architecture:**
```
Payment → API
  ├── ElastiCache Redis → Check: ip_blacklist, device_fingerprint (hits < 1ms)
  │         └── Cache Miss → DynamoDB → redis (write-through)
  │
  ├── DynamoDB: fraud_signals_table
  │     PK: signal_type (ip|device|account)
  │     SK: signal_value
  │     Attributes: risk_score, last_seen, blocked_until (TTL)
  │
  └── EventBridge: "fraud_detected" → SNS → Email/Lambda → Block Account
```

**Pattern used:** Write-through cache (when a new fraud signal is detected, write to Redis AND DynamoDB simultaneously). TTL on DynamoDB to expire old signals automatically. DAX could replace Redis+DynamoDB if all the logic stays in DynamoDB.
