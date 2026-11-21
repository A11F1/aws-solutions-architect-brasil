# Questions — Module 15: Data Migration and Transfer

> **SAA-C03 Domain**: Design Resilient Architectures  
> **Difficulty**: Medium

---

**1.** A company needs to migrate 500 TB of data from a datacenter to AWS. The internet connection is 1 Gbps shared with the company. Transferring over the internet would take more than 45 days. Which service should be used?

- A) AWS DataSync with a transfer scheduler
- B) AWS Direct Connect for direct transfer
- C) AWS Snowball Edge Storage Optimized (multiple devices)
- D) S3 Transfer Acceleration

<details><summary>Answer</summary>

**C** — Snowball Edge Storage Optimized: 80 TB/device. For 500 TB, ~7 devices would be needed. DataSync over the internet (A) would take 45+ days with a shared 1 Gbps link. Direct Connect (B) requires months of provisioning and setup. Snow Family is ideal when network transfer would take weeks.

</details>

---

**2.** A company is migrating an Oracle 12c database to Amazon Aurora PostgreSQL. The team needs to convert the PL/SQL stored procedures that are incompatible with PostgreSQL. Which tool should be used?

- A) AWS DMS (Database Migration Service)
- B) AWS Schema Conversion Tool (SCT)
- C) AWS Glue for data transformation
- D) Amazon RDS for Oracle instead of Aurora

<details><summary>Answer</summary>

**B** — SCT (Schema Conversion Tool): converts the schema and procedural code (stored procedures, functions, triggers, PL/SQL packages) from Oracle to the PostgreSQL dialect. It identifies objects that cannot be converted automatically and generates a report. Then, use DMS to migrate the data.

</details>

---

**3.** A company wants to migrate 50 on-prem Windows servers to Amazon EC2 with minimal changes. The maintenance window is only 2 hours. Which service should be used?

- A) AWS Application Migration Service (MGN) with continuous replication before cutover
- B) AWS SMS (Server Migration Service) with scheduled cutover
- C) Manually create AMIs and launch EC2 instances
- D) AWS DMS to migrate the entire server

<details><summary>Answer</summary>

**A** — MGN installs a replication agent on the source servers and continuously replicates the disks to AWS. When ready for cutover, the maintenance window is only for the final delta (minutes, not hours). An RTO of ~30-60 minutes within a 2-hour window is achievable. SMS is the legacy service replaced by MGN.

</details>

---

**4.** A retail company has B2B partners that can only transfer files via SFTP. The company wants to receive these files directly in S3 without maintaining an SFTP server. Which service should be used?

- A) AWS DataSync with an SFTP endpoint
- B) AWS Transfer Family with the SFTP protocol and an S3 backend
- C) AWS Storage Gateway File Gateway
- D) S3 with pre-signed URLs for upload

<details><summary>Answer</summary>

**B** — Transfer Family: creates a managed SFTP endpoint (no server to manage). Partners keep using their existing SFTP clients without changes. Files are stored directly in S3. Authentication can be via IAM, Cognito, or a custom Lambda (for LDAP/AD).

</details>

---

**5.** A company wants to migrate a 2 TB on-prem MySQL database to Amazon RDS MySQL with downtime of less than 15 minutes. What is the strategy?

- A) Export a MySQL dump, upload it to S3, restore it into RDS (hours of downtime)
- B) DMS full load + CDC: migrate the initial data, then replicate changes, cutover with a minimal window
- C) Create an RDS Read Replica pointing to the on-prem MySQL and promote it
- D) AWS Database Migration Service with snapshot migration

<details><summary>Answer</summary>

**B** — DMS with CDC (Change Data Capture): first performs a full load of the database into RDS (no impact on production). Then enables CDC to replicate changes in real time. When both databases are in sync, the cutover window is minimal (only the accumulated lag, which can be seconds). Total downtime < 15 minutes.

</details>

---

**6.** A team wants to migrate on-prem NAS file data to Amazon EFS, ensuring that timestamps and Unix permissions are preserved. Which service should be used?

- A) AWS Snowball Edge + rsync
- B) AWS DataSync with metadata preservation
- C) FTP to S3 + copy script
- D) AWS Storage Gateway File Gateway

<details><summary>Answer</summary>

**B** — DataSync: preserves file metadata (modification, access, and creation timestamps; Unix permissions uid/gid/chmod). Supports NFS → EFS. Performs integrity verification (checksums) during and after the transfer. More reliable and faster than manual rsync.

</details>

---

**7.** A company wants to assess its on-prem server fleet before migration to identify dependencies between servers and utilization patterns. The environment is VMware. Which service and mode should be used?

- A) Application Discovery Service with Agent-based mode on each server
- B) Application Discovery Service Agentless with the VMware vCenter Connector
- C) AWS Migration Hub with manual analysis
- D) AWS MGN with pre-migration analysis

<details><summary>Answer</summary>

**B** — ADS Agentless: a VM appliance on VMware vCenter collects CPU, memory, storage, and network data from VMs without installing an agent on each server. Faster to deploy for an initial inventory. Agent-based (A) provides more detailed data (processes, network connections) but requires installation on each server.

</details>

---

**8.** A company discovered that 30% of its on-prem applications have low usage and will be discontinued within the next 6 months. What is the correct migration strategy for these applications?

- A) Rehost (lift-and-shift) to EC2
- B) Replatform to managed services
- C) Retire (do not migrate — eliminate)
- D) Retain (keep on-prem until discontinuation)

<details><summary>Answer</summary>

**C** — Retire: if the application will be discontinued in 6 months, it makes no sense to invest in migration. Eliminating it now reduces operational costs immediately. Retain (D) would be correct if there were a technical reason to keep it on-prem temporarily, but knowing they will be discontinued, retire them immediately.

</details>

---

**9.** A company wants to move its legacy on-prem CRM (without vendor support) to a modern cloud SaaS CRM. How would the 7 Rs classify this migration strategy?

- A) Rehost
- B) Replatform
- C) Repurchase
- D) Refactor

<details><summary>Answer</summary>

**C** — Repurchase: replace with a SaaS product (e.g., Salesforce, HubSpot, SAP). The code is not migrated; the data is migrated to the new SaaS system. The company abandons the legacy system and adopts a ready-made modern solution.

</details>

---

**10.** A company needs to migrate 10 PB of data from magnetic tapes to S3 for long-term retention. Which AWS service should be used?

- A) 100 Snowball Edge devices
- B) AWS Snowmobile (truck with 100 PB capacity)
- C) Dedicated Direct Connect + DataSync
- D) 125 Snowball Edge Storage Optimized devices

<details><summary>Answer</summary>

**D** — Snowball Edge Storage Optimized: 80 TB per device. For 10 PB: 10,000 TB / 80 TB = 125 devices. Snowmobile (B) is for migrations > 100 PB (exabyte-scale). For 10 PB, multiple Snowball Edges in parallel is the practical solution.

</details>

---

**11.** A company wants to migrate its on-prem Oracle RAC database to AWS. Oracle RAC uses proprietary features not available in RDS. What is the best option?

- A) RDS Oracle Multi-AZ
- B) Amazon RDS Custom for Oracle (access to the operating system and engine)
- C) Oracle on EC2 with EFS shared storage
- D) Aurora PostgreSQL with Oracle compatibility

<details><summary>Answer</summary>

**B** — RDS Custom for Oracle: a managed option that still allows access to the OS and engine customization (for customers who need specific patches, features such as RAC via workarounds, or external dependencies). More managed than plain Oracle on EC2, but more flexible than standard RDS Oracle.

</details>

---

**12.** During a DMS migration from SQL Server to Aurora MySQL, the team realizes that SQL Server triggers are not compatible. What is the role of SCT in this scenario?

- A) SCT automatically migrates the data like DMS
- B) SCT automatically converts the SQL Server T-SQL triggers to MySQL SQL, identifying incompatibilities for manual adjustment
- C) SCT only checks data type compatibility
- D) SCT is not needed for SQL Server → MySQL

<details><summary>Answer</summary>

**B** — SCT analyzes the SQL Server procedural code (T-SQL: stored procedures, triggers, functions) and converts it to MySQL SQL where possible. For code that cannot be converted automatically (proprietary T-SQL features), it generates a report with the required manual actions and indicates the conversion complexity per object.

</details>

---

**13.** A company operates a data center with 200 on-prem servers and wants to know how much it will cost to run them on AWS before committing to the migration. Which tool should be used?

- A) AWS Cost Explorer with forecasts
- B) AWS Migration Evaluator (Agentless Collector + TCO Analysis)
- C) AWS Compute Optimizer
- D) Trusted Advisor with cost analysis

<details><summary>Answer</summary>

**B** — AWS Migration Evaluator (formerly TSO Logic): collects utilization data from the on-prem environment (via agentless collector or data import) and provides a detailed TCO (Total Cost of Ownership) analysis comparing current costs vs AWS. The basis for the migration business case.

</details>

---

**14.** A company needs to move data daily from an on-prem NFS server to an S3 bucket in a scheduled and verified manner. Which service should be used?

- A) Storage Gateway File Gateway
- B) AWS DataSync with a scheduled task
- C) Direct Connect + S3 Transfer Acceleration
- D) AWS Snowcone with embedded DataSync agent

<details><summary>Answer</summary>

**B** — DataSync: install the agent on the on-prem server (or use the DataSync VM agent), connect to the NFS source and S3 destination, and create a task with a (daily) schedule. It verifies integrity with checksums and performs only incremental transfer after the first full sync. Simpler than managing Storage Gateway for migration.

</details>

---

**15.** A manufacturing company has old IoT equipment in remote locations with limited internet connectivity. They need to collect and process data locally before sending it to AWS. Which Snow device should be used?

- A) AWS Snowmobile
- B) AWS Snowball Edge Compute Optimized (edge computing + storage)
- C) AWS Snowcone with basic edge computing
- D) AWS Outposts

<details><summary>Answer</summary>

**B** — Snowball Edge Compute Optimized: 52 vCPUs, 208 GB RAM, 42 TB + 28 TB NVMe SSD storage, optional GPU. Runs EC2 instances (AMIs locally), Lambda functions, and SageMaker inference at the edge. Ideal for processing IoT data in remote locations and syncing with AWS when connectivity is available.

</details>
