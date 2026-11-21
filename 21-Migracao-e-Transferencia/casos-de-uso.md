# Real-World Use Cases — Migration and Transfer (Module 15)

## Case 1 — Lift-and-Shift of a Corporate Data Center

**Context:** A manufacturing company with 200 on-premises servers (Windows Server + SQL Server, Linux + Oracle) needs to migrate to AWS within 12 months. It wants minimal initial rework (lift-and-shift) with modernization later.

**Requirements:**
- Cutover window < 2 hours per server
- Zero data loss during migration
- Continuous replication before cutover
- Rollback plan in case of failure

**Migration Architecture with MGN:**
```
PHASE 1 — DISCOVERY (weeks 1-4):
AWS Application Discovery Service (discovery agent on the servers)
├── Maps dependencies between apps (which servers talk to each other)
├── Collects: CPU, memory, disk, network traffic
└── Migration Hub → groups them into migration waves

PHASE 2 — REPLICATION (weeks 5-20):
On-premises → AWS MGN (Application Migration Service)
        AWS Replication Agent installed on each server
                │ Continuous replication over TCP 1500
                ▼
        AWS Staging Area (replication server instances)
                │
                ▼
        EBS Volumes (copy of the on-premises disks)

PHASE 3 — TEST CUTOVER (weeks 21-23):
MGN → Launch Test Instance (launch template configured)
├── Validate the app works on AWS
├── Test connectivity with other services
└── Verify performance (same or better)

PHASE 4 — PRODUCTION CUTOVER:
├── Freeze on-premises traffic (30 min maintenance)
├── MGN finishes replication (delta sync)
├── Launch production instances
├── Update DNS (Route 53 → new IPs)
└── Monitor for 48h before decommissioning on-premises
```

**Rollback Plan:**
- MGN keeps staging instances for 90 days after cutover
- DNS can be reverted in < 5 minutes (low Route 53 TTL)
- On-premises is not decommissioned until full validation

---

## Case 2 — PostgreSQL Database Migration to Aurora with DMS

**Context:** A FinTech with an on-premises PostgreSQL database (500 GB, 100 transactions/second) needs to migrate to Amazon Aurora PostgreSQL without interrupting operations. The system processes payments 24/7.

**Requirements:**
- Zero downtime during migration
- Cutover window < 10 minutes
- Validate data integrity after migration
- Ability to roll back within 15 minutes

**Architecture with DMS (Change Data Capture):**
```
PHASE 1 — FULL LOAD:
PostgreSQL On-Prem ──── DMS Replication Instance ──── Aurora PostgreSQL
   (source endpoint)          (r5.xlarge)              (target endpoint)
                         Full Load Task
                         ├── Copies all tables
                         ├── Disables FK constraints (temporarily)
                         └── Creates indexes after load (faster)
                         
Estimated duration: 4-6 hours (500GB)

PHASE 2 — CDC REPLICATION (while Full Load is still running):
DMS → Change Data Capture (captures INSERT/UPDATE/DELETE)
     └── Applies changes to Aurora in real time
     └── Replication latency: < 1 second
     
CUTOVER (when replication lag ≈ 0):
1. Application: read-only mode (or 5-minute maintenance page)
2. DMS: verify lag = 0 (no pending data)
3. Change the app's connection string → Aurora endpoint
4. AWS SCT validated: same structure + matching checksums
5. Resume normal operation
6. Monitor for 24h = rollback available (DMS still active in the reverse direction)

ROLLBACK:
Pre-configured reverse DMS task (Aurora → on-prem PostgreSQL)
Change the connection string back in < 5 minutes
```

**Full Load vs CDC difference:**
| Mode | What it does | When to use |
|------|-----------|-------------|
| Full Load | Single complete copy | Traffic can be interrupted |
| CDC | Replicates incremental changes | Zero-downtime migration |
| Full Load + CDC | Copy + tracks changes | **Recommended default** |

---

## Case 3 — Transferring 500 TB with AWS Snowball Edge

**Context:** An oil company with a remote base (no fiber, 100 Mbps satellite link) needs to transfer 500 TB of historical seismic data to S3 and then process it with EMR.

**Requirements:**
- Transfer in < 30 days
- Data encrypted in transit
- No dependency on the internet link (slow link)
- Processing a subset of the data at the edge before shipping

**Feasibility Calculation:**
```
500 TB over the internet (100 Mbps):
  500 TB = 500 × 1024 × 8 Gbits = 4,096,000 Gbits
  @ 100 Mbps = 0.1 Gbps
  Time = 4,096,000 / 0.1 / 3600 / 24 = ~474 days
  
Snowball Edge Storage Optimized (80 TB usable):
  500 TB / 80 TB = 7 devices
  Packaging + shipping + ingestion: ~3 weeks
  ✓ MUCH FASTER
```

**Process:**
```
AWS Console → Request 7 Snowball Edge Storage Optimized
                    │ Delivery in 2-5 business days
                    ▼
Remote Base:
  ├── Connect Snowball to the local network (10 GbE)
  ├── Snowball Client → transfer data to the device
  ├── (optional) Snowball Edge compute: local Lambda for pre-processing
  └── Automatic encryption (AES-256, key in KMS)
  
Shipment to AWS (carrier)
                    │
                    ▼
AWS Data Center → Automatic ingestion into S3
                    │
                    ▼
S3 (seismic data) → EMR (Spark processing)
                    → Athena (ad hoc analysis)
                    
Notification: SNS email when ingestion completes
Verification: automatic MD5 checksum by the service
```

---

## Case 4 — Heterogeneous Migration: Oracle → Aurora PostgreSQL

**Context:** An HR company wants to eliminate its Oracle license ($500K/year) by migrating to Aurora PostgreSQL. The database is 2 TB with stored procedures, PL/SQL packages, and Oracle-specific functions.

**Requirements:**
- Convert PL/SQL code to PL/pgSQL (natively incompatible)
- Identify incompatibilities before starting
- Migrate data with minimal downtime
- Document all conversions for audit

**Process with SCT + DMS:**
```
PHASE 1 — ASSESSMENT:
AWS SCT (Schema Conversion Tool)
├── Connects to on-premises Oracle
├── Generates a compatibility report:
│   ├── 70% of code converted automatically ✓
│   ├── 20% requires manual adjustment (Oracle-specific logic)
│   └── 10% must be rewritten (features with no equivalent)
└── Priority: fix the 30% manual items before continuing

PHASE 2 — CONVERSION:
SCT converts automatically:
├── Tables, indexes, views → CREATE TABLE/INDEX/VIEW
├── Basic PL/SQL → PL/pgSQL
└── Sequences → PostgreSQL Sequences

DBA team fixes manually:
├── CONNECT BY (hierarchy) → Recursive CTE
├── ROWNUM → ROW_NUMBER() OVER()
├── Oracle packages → PostgreSQL schemas + functions
└── Implicit cursors → Explicit PgSQL cursors

PHASE 3 — DATA MIGRATION:
DMS Full Load + CDC (same as case 2, but Oracle source → Aurora PG)
Difference: SCT must be applied before DMS

PHASE 4 — VALIDATION:
AmazonDMS data validation (table hashes compared automatically)
Regression tests (suite of critical queries must return the same result)
```

---

## Case 5 — Transfer Family for B2B Integration with Partners

**Context:** A logistics company needs to receive files from 50 partners via SFTP (the market's legacy standard) and process them automatically. It currently maintains 2 on-premises SFTP servers.

**Requirements:**
- Keep the SFTP protocol (partners won't change)
- Process files automatically on arrival
- Each partner sees only their own folder
- High availability (no server maintenance)

**Architecture:**
```
Partners (50 companies)
    │ SFTP (port 22)
    ▼
AWS Transfer Family (managed SFTP endpoint)
├── Custom DNS: sftp.empresa.com.br (Route 53)
├── Authentication: AWS Secrets Manager (SSH key per partner)
└── Storage: S3 (bucket per partner or prefix per partner)

S3 Structure:
  s3://logistica-parceiros/
    parceiro-A/incoming/   (partner A only accesses this prefix)
    parceiro-B/incoming/   (partner B only accesses this prefix)

Access Control:
IAM Policy per partner: 
  s3:PutObject only on /{parceiro}/{userid}/*
  s3:GetObject only on /{parceiro}/{userid}/*
  
AUTOMATIC PROCESSING:
S3 Event (s3:ObjectCreated) → Lambda (process-file)
├── Validates file format (CSV, EDI)
├── Moves to /processed/ after validation
├── Inserts records into DynamoDB
└── SNS → internal system updates delivery status

MONITORING:
CloudWatch Logs (each SFTP connection logged)
CloudTrail (who uploaded which file, and when)
CloudWatch Alarm (file not received from a critical partner in 24h)
```

**Benefits vs on-premises SFTP:**
| Item | On-premises | Transfer Family |
|------|------------|----------------|
| HA | Manual (2 VMs) | Native Multi-AZ |
| Scalability | Limited | Automatic |
| Maintenance | Manual patches | Zero |
| Cost | Servers + licenses | Per hour + per GB |
| Auditing | Manual logs | Native CloudTrail |
